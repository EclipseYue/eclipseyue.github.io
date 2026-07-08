# P1 Readiness 路线

P1 的目标是从“设备和 tensor 能跑”推进到“基础算子能通过后端路径执行并对齐 CPU reference”。实际推进时不要一口气接完整模型，而要从低风险 op 开始扩张。

## 当前阶段划分

| 阶段 | 验证目标 | 价值 |
| --- | --- | --- |
| P0 | import、device API、empty、copy、clone、to cpu | 证明 PyTorch tensor 层闭环 |
| P0.5 | stream、event、malloc/free、H2D/D2D/D2H runtime smoke | 证明 P1 所需 runtime 能力 |
| P1-T03 最小链 | add / relu / sum 等基础 op | 证明 ATen dispatch 到后端 op 的路径 |
| P1-T04+ | codegen、更多 dtype/shape/layout、真实 kernel path | 从 smoke 走向可持续算子体系 |

## 推荐推进顺序

| 轮次 | 目标 | 完成后价值 |
| --- | --- | --- |
| 1 | `sub` / `div` binary smoke | 扩展基础二元运算面 |
| 2 | `neg` / `abs` / `sqrt` unary smoke | 扩展一元运算路径 |
| 3 | `relu` | 支撑常见激活函数 |
| 4 | `sum` | 支撑 reduction 路径 |
| 5 | `a + b -> relu -> sum` 链式测试 | 达到最小验收链 |
| 6 | shape / dtype / out variant | 从单点 smoke 变成回归测试矩阵 |
| 7 | 从 host-assisted 切到真实 custom kernel | 验证真实后端执行路径 |
| 8 | codegen / unsupported list / 文档同步 | 进入可持续维护状态 |

## 最小验收链

```python
a = torch.randn(3, 4, device="tpu")
b = torch.randn(3, 4, device="tpu")
c = a + b
c = torch.relu(c)
c = c.sum()
```

这个链路覆盖：

- binary op
- unary / activation op
- reduction op
- tensor 留在设备侧连续执行
- 最终 D2H 回拷并与 CPU reference 对齐

## TDD 切分方式

每次只新增一个公开行为测试：

1. 写 Python API 层测试，例如 `torch.sub(tpu, tpu)`。
2. RED 失败点应是 op 未注册或后端实现缺失。
3. GREEN 只做最小 PrivateUse1 后端实现。
4. 先保持 float32、contiguous、same-shape 小范围。
5. 跑 basic ops、readiness 和 SDK leak 检查。
6. 最后记录数据流、端口函数和新增文件。

## 什么时候切到真实 kernel

host-assisted correctness smoke 的价值是尽快验证 dispatcher、copy 和 result check 链路。但它不能代表真实 custom kernel 性能和正确性。

可以考虑切换到真实 kernel 的条件：

- P0 / P0.5 全部通过。
- 至少一个 op 的 CPU reference 路径稳定。
- 参数打包结构明确。
- runtime launch 能被测试验证。
- 现有 smoke 测试能作为回归保留。

切换时要明确标注：

- 哪些 op 仍是 host-assisted。
- 哪些 op 已经走真实 custom kernel。
- 测试如何证明 launch 被使用。
- 失败时如何退回到最小复现。
