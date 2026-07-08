# 训练总览

训练系统的目标是把数据、模型、算力和优化策略组织起来，让模型稳定收敛，并且在成本可接受的范围内产出可复用的 checkpoint。

## 基本流程

1. 数据准备：清洗、去重、标注、切分。
2. 样本构造：tokenize、packing、mask、负样本。
3. 训练配置：模型结构、batch size、学习率、精度格式。
4. 并行策略：数据并行、张量并行、流水线并行、ZeRO/FSDP。
5. 监控与恢复：loss、梯度、吞吐、显存、checkpoint。

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
