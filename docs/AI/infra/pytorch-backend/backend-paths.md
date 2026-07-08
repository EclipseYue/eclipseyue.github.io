# 后端路径图

AI 加速器软件栈不是单仓库闭环。一个 PyTorch API 最终能在设备上执行，通常要穿过框架后端、参数打包、runtime、custom kernel、模拟器或真实硬件。

## P0 受控小路径

P0 阶段只关注最小 tensor 闭环：

```text
Python import torch_tpu
  -> C++ extension initialization
  -> PrivateUse1 backend register
  -> guard / allocator / storage / hooks
  -> aten::empty / _copy_from
  -> H2D / D2D / D2H copy
  -> CPU reference check
```

这条路径的核心问题是：PyTorch 能不能认识新设备，并完成 tensor 创建和数据搬运。

## P1 算子路径

P1 开始进入算子层：

```text
PyTorch API / ATen op
  -> ATen dispatcher
  -> PrivateUse1 backend implementation
  -> OpPreparation / shape / dtype / layout handling
  -> parameter descriptor packing
  -> runtime launch wrapper
  -> custom kernel
  -> simulator or real hardware
  -> D2H result check
```

P1 的关键变化是：不再只验证 copy，而是验证 `torch.add`、`torch.relu`、`torch.sum` 这类 op 是否能走到后端实现并返回正确结果。

## 完整生态路径

完整 AI Infra 软件栈通常还包括 vLLM / SGLang 等上层框架：

```text
PyTorch / vLLM / SGLang
  -> PyTorch device backend or framework custom op
  -> argument packing
  -> runtime / launch protocol
  -> custom kernel repository
  -> simulator or real hardware
  -> collective communication when multi-card
```

这个路径里每层都有不同的失败方式：

- PyTorch dispatcher：op 未注册、dispatch key 错误、fallback 路径不符合预期。
- 参数打包：shape、stride、dtype、layout、scalar 元数据错误。
- runtime：stream/event/memcpy/async error 没处理好。
- custom kernel：padding、tiling、数值精度、边界 shape 错误。
- 上层框架：KV cache layout、batch 调度、采样策略、模型输入输出不一致。

## 调试原则

1. 先确认处在哪一层失败，不要直接跳到 kernel。
2. 能用 CPU reference 的地方优先做对齐。
3. 模型级 failure 要尽量压缩到 op 级复现。
4. 区分“预期数值差异”和“真正正确性 bug”。
5. 所有最小复现都要记录 shape、dtype、layout、设备路径和环境开关。
