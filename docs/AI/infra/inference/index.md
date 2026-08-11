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

## PD 分离与 Mooncake

- PD 分离：把 prefill 和 decode 拆到不同执行池。prefill 侧更关注 prompt 长度、首 token 延迟和大块计算吞吐，decode 侧更关注小步迭代、并发调度和 KV Cache 命中。
- 资源拆分：prefill 节点适合吃满大 batch 和算力，decode 节点适合维持稳定 token 流水线，调度器需要在两类节点之间传递请求状态。
- KV Cache 流转：PD 分离的关键成本不是只看算子时间，还要看 KV Cache 在 GPU、CPU、网络和远端缓存之间的搬运代价。
- Mooncake：可作为 PD 分离下的缓存与传输索引主题，关注 KV Cache 跨节点复用、远端缓存命中、传输带宽管理和服务侧编排。
- 排查入口：首 token 延迟高先看 prefill 排队和 cache 传输，decode 吞吐低先看 batch 组织、KV Cache 访问和网络回压。

## 推理系统关注点

- 首 token 延迟。
- 每 token 延迟。
- 最大并发。
- 显存占用。
- 输出质量与采样策略。
- 异常请求隔离。
