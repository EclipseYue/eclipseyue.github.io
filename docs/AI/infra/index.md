# AI Infra

AI Infra 记录训练、推理和模型工程化相关内容。这里关注的是“怎样把模型可靠、高效、可观测地跑起来”，包括数据管线、分布式训练、模型服务、推理优化和评测闭环。

## 目录

- PyTorch 后端
  - [后端总览](pytorch-backend/index.md)
  - [PrivateUse1 P0 链路](pytorch-backend/privateuse1-p0-flow.md)
  - [2D Padded Layout](pytorch-backend/tensor-layout-2d-padded.md)
  - [后端路径图](pytorch-backend/backend-paths.md)
  - [P1 Readiness 路线](pytorch-backend/p1-readiness-roadmap.md)
- 学习规划
  - [AI Infra 学习路线](learning-roadmap.md)
- 训练
  - [训练总览](training/index.md)
  - [数据与样本工程](training/data.md)
  - [分布式训练](training/distributed.md)
  - [训练调试与稳定性](training/debugging.md)
- 推理
  - [推理总览](inference/index.md)
  - [模型服务](inference/serving.md)
  - [推理优化](inference/optimization.md)
  - [评测与观测](inference/evaluation.md)
- 硬件
  - [AI 加速器指标](hardware/accelerator-metrics.md)

## 关注维度

- 吞吐：单位时间处理多少样本或 token。
- 延迟：单次请求从进入系统到返回结果的时间。
- 显存：参数、激活、KV Cache、batch 对资源的占用。
- 成本：GPU 时间、存储、网络、运维复杂度。
- 稳定性：训练是否收敛，服务是否可用，结果是否可复现。

## 和理论笔记的分工

- 理论原理解释模型为什么能工作。
- AI Infra 解释模型如何在真实资源约束下工作。
- AI Agent 解释模型如何被组织成可完成任务的系统。
