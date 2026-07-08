# PrivateUse1 P0 链路

P0 阶段的目标是证明 PyTorch 能识别一个新设备，并完成最小 tensor 生命周期闭环：

```python
import torch
import torch_tpu

cpu = torch.arange(8, dtype=torch.float32)
tpu = cpu.to("tpu")
cloned = tpu.clone()
back = cloned.to("cpu")

assert torch.equal(back, cpu)
```

这条链路还不证明真实算子已经接入，但能验证设备注册、内存分配、数据搬运和回拷校验是否成立。

## 1. 加载扩展

`import torch_tpu` 会加载 Python 包和 C++ 扩展。C++ 扩展初始化时通常完成几件事：

- 注册 `PrivateUse1` backend 名称，例如 `"tpu"`。
- 注册 device guard。
- 注册 storage 创建函数。
- 注册 allocator。
- 暴露 Python 可调用的设备 API。

核心效果是：PyTorch 知道 `device="tpu"` 应该映射到 `c10::DeviceType::PrivateUse1`。

## 2. 注册设备后端

典型注册逻辑包括：

```cpp
c10::register_privateuse1_backend("tpu");
C10_REGISTER_GUARD_IMPL(PrivateUse1, TPUGuardImpl);
c10::SetStorageImplCreate(DeviceType::PrivateUse1, make_tpu_storage_impl);
c10::SetAllocator(DeviceType::PrivateUse1, &privateuse1_alloc, 0);
at::RegisterPrivateUse1HooksInterface(get_tpu_hooks());
```

这些注册完成后，PyTorch dispatcher 在遇到 `device="tpu"` 的 tensor 时，会走 PrivateUse1 对应的 kernel 或 fallback 路径。

## 3. 创建设备 tensor

执行 `cpu.to("tpu")` 时，PyTorch 需要做两件事：

1. 在目标设备上创建一个 tensor。
2. 把 CPU tensor 的数据拷贝到设备 tensor。

创建目标 tensor 通常会触发 `aten::empty` 的 PrivateUse1 实现：

```cpp
TORCH_LIBRARY_IMPL(aten, PrivateUse1, m) {
    m.impl("empty.memory_format", tpu_empty_memory_format);
    m.impl("empty_strided", tpu_empty_strided);
    m.impl("_copy_from", tpu_copy_from);
    m.impl("_copy_from_and_resize", tpu_copy_from_and_resize);
}
```

设备内存分配链路可以抽象为：

```text
aten::empty
  -> PrivateUse1 empty implementation
  -> device allocator
  -> runtime malloc wrapper
  -> device runtime / driver
```

## 4. 数据搬运方向

P0 最关键的是 `_copy_from`。它要根据源和目标设备判断拷贝方向：

| 源 | 目标 | 方向 |
| --- | --- | --- |
| CPU | 设备 | H2D |
| 设备 | 设备 | D2D |
| 设备 | CPU | D2H |

抽象链路是：

```text
tpu_copy_from
  -> 判断 src/dst device
  -> runtime memcpy wrapper
  -> device runtime memcpy
```

如果 H2D、D2D、D2H 任意一步有问题，最终 `torch.equal(back, cpu)` 都会失败。

## 5. P0 验收范围

P0 能证明：

- `import torch_tpu` 不崩溃。
- `torch.empty(..., device="tpu")` 能创建设备 tensor。
- CPU 到设备的数据拷贝可用。
- 设备到设备的 clone / copy 可用。
- 设备到 CPU 的回拷可用。
- 回拷结果能和 CPU reference 对齐。

P0 不能证明：

- `torch.add(tpu, tpu)` 这类真实算子已经走通。
- custom kernel launch 已经可用。
- dtype、broadcast、non-contiguous、layout padding 等复杂情况已经正确。

## 6. P1 Readiness 多验证了什么

P1 readiness 通常会直接验证算子层会依赖的 runtime 能力：

- stream create / synchronize。
- event create / record / synchronize。
- malloc / free。
- H2D、D2D、D2H memcpy。
- 多 shape / dtype 的 tensor copy / clone。

也就是说：

- P0 验证 PyTorch tensor 层闭环。
- P0.5 / P1 readiness 验证 P1 算子层会使用的 runtime 基线。
- P1 才开始验证 ATen op、参数打包、kernel launch 和数值正确性。
