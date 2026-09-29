# 基于PrivateUse1机制的特定后端插件化适配

> 目标：把一个新的加速器后端接入 PyTorch，并理清“算子如何被分派到该后端”这条主链路。

## 从dispatch讲起：如何分派到特定后端计算

PyTorch 的算子调用最终都会落到 `c10::Dispatcher`，它按 **dispatch key** 选出具体实现。

- dispatch key 是分层集合，常见几类：`Autograd*`（反向/自动微分）、`Backend*`（CPU/CUDA/Meta/...）、`Quantized`、`Sparse` 等。一次调用会按优先级取“最具体”的那个 key。
- 自定义设备没有原生 key，因此 PyTorch 预留了 **`PrivateUse1`** 作为第三方后端的占位 key，所有自定义后端复用它。
- 设备名与 key 的绑定：`torch.utils.rename_privateuse1_backend("xxx")` 会把 `PrivateUse1` 重命名成可读设备串，之后即可写 `torch.empty(..., device="xxx:0")`。
- 生成 Python 侧 API：`torch.utils.generate_methods_for_privateuse1_backend()` 用于补齐 `tensor.xxx()`、`tensor.to("xxx")` 这类方法。
- 两段式注册：
  - `TORCH_LIBRARY(my_ns, m) { m.def("my_op(...) -> Tensor"); }` —— 声明 schema（与后端无关）。
  - `TORCH_LIBRARY_IMPL(aten, PrivateUse1, m) { m.impl("add.Tensor", &my_add); }` —— 把实现挂到 `PrivateUse1` 这个 key 上。
- 没有逐 op 实现时可以注册兜底：`TORCH_LIBRARY_IMPL(_, PrivateUse1, m) { m.fallback(...); }`，用于报错提示、统一转译或走模拟实现。

```text
torch.add(a, b)
  -> Tensor 方法 / ATen 函数
  -> c10::Dispatcher（callBoxed / call）
  -> 按 dispatch key 查表（Autograd -> PrivateUse1 -> ...）
  -> PrivateUse1 上注册的 kernel
```

!!! note "TODO"
    - 补一张精简的 dispatch key 优先级表，只保留与自定义后端相关的几档。
    - 记录一次“算子落到 fallback 而非真实 kernel”的排查过程。

## 插件编译入口init

后端通常以 C++/Python 扩展（或预编译动态库）形式加载，注册发生在 **库被 `import` 时的静态初始化阶段**。

- 扩展骨架：`PYBIND11_MODULE(_my_backend, m)` 中调用 `TORCH_LIBRARY` / `TORCH_LIBRARY_IMPL` 宏；这些宏会生成静态初始化对象，库加载即执行注册。
- 因此“`import` 了就注册成功、没 `import` 就找不到后端”是预期行为，op 注册与设备注册都依赖这条时序。
- 注册内容通常包含三类：
  - op 实现：`m.impl(...)`；
  - allocator：把自定义 allocator 交给 `PrivateUse1` 设备类型（具体 API 名随版本变化，以所用版本头文件为准）；
  - 设备 guard 支撑：实现 `c10::impl::DeviceGuardImplInterface`，供 `DeviceGuard` / `OptionalDeviceGuard` / `InlineDeviceGuard` 使用。
- 常见坑：注册与静态初始化顺序耦合，容易出现“有时能加载、有时报后端不存在”。解法是把注册收拢到单一入口函数并显式调用。
- 插件化适配的价值：后端实现与 PyTorch 主线解耦，可按版本独立编译，不需要改动 PyTorch 源码树。

!!! note "TODO"
    - 补一个最小编译示例（`cmake` 或 `setup.py`，去掉任何本地真实路径）。
    - 记录“import 顺序导致注册失败”的具体现象与定位方法。

## 特定后端的内存实现

张量创建与数据搬运最终都落到 allocator，后端必须提供可用的内存实现。

- `c10::Allocator` 关键接口：`allocate(nbytes)`、`raw_allocate`（无 deleter 的裸分配）、`raw_delete`，返回 `c10::DataPtr`（数据指针 + deleter + device）。
- `c10::DataPtr` 的 deleter 决定释放时机；异步执行下“何时真正释放”必须由 stream 语义保证。
- 缓存分配器：`CachingAllocator` 在设备内存之上做块池化，避免每次分配都直连 driver。
- 能力收敛：PyTorch 2.x 起把通用内存管理能力向 `c10::DeviceAllocator` 抽象，要求后端提供 `initialized()`、`emptyCache()`、`recordStream()`、`getDeviceStats()`、`resetAccumulatedStats()`、`resetPeakStats()`、`getMemoryInfo()` 等能力（虚函数清单随版本演进，以所用版本头文件为准）。
- 与 tensor 的关系：`StorageImpl` 持有 `DataPtr`，`TensorImpl` 引用 `Storage`；`aten::empty`、`aten::empty_strided`、`aten::_copy_from` 是后端最先要打通的一组 op。
- `recordStream` 的意义：把某块内存与某条 stream 绑定，stream 未完成前不允许复用该内存，从而支撑 stream-safe 的延迟释放。

```text
aten::empty
  -> 选出 PrivateUse1 kernel
  -> allocator.allocate(nbytes) -> DataPtr
  -> StorageImpl / TensorImpl 建立
aten::_copy_from
  -> H2D / D2D / D2H（后端实现或复用 runtime）
```

!!! note "TODO"
    - 对照所用 PyTorch 版本头文件，抄一份 `DeviceAllocator` 的实际虚函数清单（去掉本地路径）。
    - 补 memory stats / memory snapshot 在后端的落地方式。
