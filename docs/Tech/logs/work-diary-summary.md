# 工作日报摘要

这页从 Obsidian 工作日报中整理而来，只保留阶段性任务和经验，不保留任何账号、密码、token、内网地址或机器信息。

## 2026-04：入职与环境搭建

主要内容：

- 完成入职、阅读文档、明确 PyTorch 插件化适配方向。
- 配置 Git、shell、CMake、Python 环境和 AI 开发工具。
- 学习 AI 加速器软件栈基础概念。
- 调研 PyTorch 设备插件、昇腾 `torch_npu` 和相关开源生态。
- 开始搭建 LLVM、runtime、vLLM 等开发环境。

阶段经验：

- root 和普通用户环境差异会导致 Git、SSH、Conda、代理配置不一致。
- 大仓库 clone 失败时，先排查网络、SSH keepalive 和压缩参数。
- 新终端环境变量不继承旧终端状态，重要任务应记录启动命令。

## 2026-05：P0 runtime 迁移与验证

主要内容：

- 阅读迁移计划和插件化需求。
- 推进 PyTorch 设备插件 P0 阶段。
- 围绕 C++ extension 初始化、device guard、allocator、storage、copy kernel 做最小闭环。
- 修复迁移期 C++ 兼容问题。
- 验证 minimal / full-lite 构建和 P0 smoke。

阶段经验：

- P0 边界未稳定前，不宜过早拆太多文件。
- `import`、`empty(device=...)`、`to(device)`、`clone()` 是最小闭环中的关键节点。
- Python `try/except` 捕获不到 C++ abort，底层 probing 需要谨慎处理。
- 先让 fallback smoke 稳定，再接真实 driver / hardware 路径。

## 2026-07：P0 收口与 P1 readiness

主要内容：

- 整理工程经验、提交记录和日报。
- 收口 P0 验收入口。
- 新增 P0.5 / P1 readiness runtime baseline。
- 验证 stream、event、malloc/free、H2D/D2D/D2H memcpy。
- 为 P1 基础算子接入准备测试和运行时能力。

阶段结论：

- P0 证明设备接入、tensor 创建、内存分配和数据搬运闭环。
- P1 readiness 证明算子层会依赖的 runtime 能力。
- P1 下一步是 ATen op 注册、参数打包、kernel launch 和数值正确性。

## 可复用复盘模板

```text
日期：
任务：
背景：
完成内容：
验证命令：
遇到的问题：
根因：
后续动作：
```

## 敏感信息处理规则

- 不把账号、密码、token、内网 URL、机器 IP 写入公开站点。
- 命令中的真实路径和私有仓库名按需要泛化。
- 如果经验依赖内部环境，只记录通用排查思路。
- 对外发布前用关键词扫描一遍敏感信息。
