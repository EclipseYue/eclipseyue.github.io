# 推理总览

推理系统关注如何把训练好的模型变成在线或离线可用的服务。核心约束通常是延迟、吞吐、成本、稳定性和结果质量。

## 推理流程

1. 请求进入服务层。
2. tokenizer 处理输入。
3. 调度器组织 batch。
4. 模型执行 prefill 和 decode。
5. 后处理生成结果。
6. 记录日志、指标和异常。

## 大模型推理概念

- Prefill：处理输入 prompt，生成 KV Cache。
- Decode：逐 token 生成输出。
- KV Cache：缓存 attention 需要的历史 key/value。
- Continuous Batching：动态把请求合并成 batch。
- Speculative Decoding：用小模型预测，再由大模型验证。

## 推理系统关注点

- 首 token 延迟。
- 每 token 延迟。
- 最大并发。
- 显存占用。
- 输出质量与采样策略。
- 异常请求隔离。
