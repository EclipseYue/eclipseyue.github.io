# PyTorch 设备后端

这里整理 PyTorch 接入新设备后端时需要理解的核心链路。重点不是某个项目的临时命令，而是能复用到后续 AI Infra 工作中的结构性知识。

## 目录

- [PrivateUse1 P0 链路](privateuse1-p0-flow.md)：从 `import`、设备注册、allocator 到 H2D/D2D/D2H 拷贝闭环。
- [2D Padded Layout](tensor-layout-2d-padded.md)：逻辑 tensor 和设备物理 storage 之间的 layout 差异。
- [后端路径图](backend-paths.md)：从 PyTorch / vLLM / SGLang 到 runtime、custom kernel、模拟器或真实硬件的路径。
- [P1 Readiness 路线](p1-readiness-roadmap.md)：从 P0 tensor 闭环走向 P1 算子接入的最小验收路线。

## 学习顺序

1. 先看 PrivateUse1 如何让 PyTorch 识别一个新设备。
2. 再看 allocator、copy、storage desc 如何支撑 tensor 创建和数据搬运。
3. 接着理解 layout、padding 和非连续 tensor 为什么会影响 kernel 正确性。
4. 最后进入算子层：ATen dispatch、参数打包、runtime launch 和数值对齐。

## 记录边界

- 这里记录公开可沉淀的机制和方法，不保存内网地址、账号、token 或真实机器凭据。
- 项目特定路径只保留抽象链路，详细命令放到私有工作区。
- 如果某个问题来自真实项目，应抽象为“现象 - 定位路径 - 根因 - 可复用经验”。
