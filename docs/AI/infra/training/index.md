# 训练总览

训练系统的目标是把数据、模型、算力和优化策略组织起来，让模型稳定收敛，并且在成本可接受的范围内产出可复用的 checkpoint。

## 基本流程

1. 数据准备：清洗、去重、标注、切分。
2. 样本构造：tokenize、packing、mask、负样本。
3. 训练配置：模型结构、batch size、学习率、精度格式。
4. 并行策略：数据并行、张量并行、流水线并行、ZeRO/FSDP。
5. 监控与恢复：loss、梯度、吞吐、显存、checkpoint。

## 分布式训练索引

- [分布式训练](distributed.md)：总入口，先按“模型是否放得下、单步是否算得动、通信是否拖慢”判断需要的数据并行、张量并行、流水线并行或 ZeRO/FSDP。
- 并行策略：DP 解决样本切分，TP 解决单层矩阵过大，PP 解决层数和显存压力，ZeRO/FSDP 解决参数、梯度和优化器状态分片。
- 通信与网络：关注 all-reduce、all-gather、reduce-scatter、send/recv 的时间占比，以及 NVLink、IB、以太网拓扑对训练吞吐的影响。
- Checkpoint：记录不同并行策略下的保存、恢复、reshard 和权重合并方式，避免训练恢复和推理部署被格式绑定。
- 排障：rank hang、NCCL timeout、吞吐抖动、显存碎片和 loss 行为变化都优先回到分布式训练页建立索引。

## 常见训练类型

- 预训练：从大规模通用语料中学习基础能力。
- 继续预训练：面向领域数据强化模型知识。
- SFT：用指令数据训练模型遵循任务。
- Preference Tuning：用偏好数据对齐输出风格和行为。
- LoRA / Adapter：低成本参数高效微调。

## 训练排查入口

- loss 不降：检查数据、学习率、标签、mask、梯度。
- loss 爆炸：检查初始化、混合精度、梯度裁剪。
- 吞吐低：检查 dataloader、通信、算子、batch 组织。
- 显存爆：检查 sequence length、activation checkpoint、并行策略。
