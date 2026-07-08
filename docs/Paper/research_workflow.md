# 论文调研与实验工作流

这页整理写论文时可复用的工作流。原始 Obsidian 笔记中有长篇外部转载和工具提示词，这里只保留抽象后的方法，不复制原文。

## 总体流程

1. 明确研究问题。
2. 做文献调研。
3. 提出可证伪假设。
4. 设计实验。
5. 实现真实可运行的实验代码。
6. 跑小规模 pilot，估算时间和资源。
7. 扩展实验并记录日志。
8. 写论文初稿。
9. 做一致性审查。
10. 归档代码、结果和复现实验说明。

## 研究问题约束

一篇论文要聚焦 1 到 2 个核心创新点。不要把环境配置、依赖安装、调试日志包装成研究贡献。

每一节都要回到核心问题：

- Abstract：说明问题、方法、结果和贡献。
- Introduction：解释为什么这个问题重要，现有方法缺口是什么。
- Method：描述技术方法，不写成操作日志。
- Experiments：报告可复现的量化结果。
- Discussion：说明适用边界、局限和未来工作。

## 实验真实性红线

- 不伪造 loss 曲线。
- 不用随机数假装实验结果。
- 不硬编码指标。
- 不用固定 for 循环冒充收敛。
- 遇到 NaN / Inf 要追根因，不用 `try/except` 或 `nan_to_num` 掩盖。

实验代码至少应包含：

- 明确的 objective / loss。
- 真实的算法更新逻辑。
- 收敛停止准则。
- deterministic seed。
- `results.json`。
- 可读的日志和指标输出。

## Pilot 先行

大实验前先跑一个最小条件：

```text
TIME_ESTIMATE: <single_condition_seconds>
estimated_total: <seconds>
```

如果实验组合太多：

- 降低 seed 数量。
- 限制最大迭代次数。
- 加 time guard。
- 在资源预算 80% 时保存部分结果并优雅停止。

## 写作结构

推荐结构：

- Introduction
- Related Work
- Methodology
- Experiments
- Discussion
- Conclusion
- References
- Appendix

正文页数有限时，核心贡献、方法和主要实验必须留在正文；实现细节、额外 ablation 和完整超参可以放附录。

## 一致性审查

提交前逐项检查：

- 论文声称的数据集是否真的在实验日志中出现。
- 论文声称的 baseline 是否真的实现并调参。
- 论文声称的 ablation 是否真的有结果。
- 图表数字是否来自 `results.json`。
- 统计检验是否真的在代码中实现。
- 文献引用是否真实、相关且可追溯。

如果方法、实验和结果不一致，应退回实验阶段，而不是修改措辞掩盖。
