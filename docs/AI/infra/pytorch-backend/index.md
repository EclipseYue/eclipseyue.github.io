# PyTorch 设备后端

这里整理 PyTorch 接入新设备后端时需要理解的核心链路。重点不是某个项目的临时命令，而是能复用到后续 AI Infra 工作中的结构性知识。

## 目录

- [PrivateUse1 插件化适配](pytorch-npu.md)：从 dispatch 分派、插件编译入口 init 到特定后端的内存实现。
- [PyTorch 后端解读](pytorch2.x.md)：从简单计算到算子、`Stream`/`Event` 机制、`eager`/`Graph` 两种执行机制。

> 推理侧框架（vLLM / SGLang）已独立为 [框架分析](../framework/index.md)。

## 学习顺序

1. 先看 dispatch 如何把算子分派到特定后端，以及 `PrivateUse1` 在其中的位置。
2. 再看插件编译入口 init 如何在 `import` 时完成 op 与 allocator 的注册。
3. 接着理解特定后端的内存实现：allocator、storage 与 `DeviceAllocator` 抽象。
4. 最后进入执行层：算子落地、`Stream`/`Event` 同步，以及 `eager` 与 `Graph` 两种执行机制。

## 记录边界

- 这里记录公开可沉淀的机制和方法，不保存内网地址、账号、token 或真实机器凭据。
- 项目特定路径只保留抽象链路，详细命令放到私有工作区。
- 如果某个问题来自真实项目，应抽象为“现象 - 定位路径 - 根因 - 可复用经验”。
