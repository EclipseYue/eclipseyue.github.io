# 框架分析

推理框架是 AI Infra 中“把模型变成服务”的那一层。写 PyTorch 算子解决的是“一个算子怎么算对”，框架解决的是“一堆请求在有限显存和带宽下怎么排队、复用和调度”。

这也是把 vLLM / SGLang 从 [PyTorch 设备后端](../pytorch-backend/index.md) 里独立出来的原因：它们关心的不是单个 kernel 的数值，而是 **KV Cache 的生命周期** 与 **batch 的组织方式**。

## 目录

- [vLLM](vllm.md)：以 PagedAttention 与 continuous batching 为核心的高吞吐推理服务框架。
- [SGLang](sglang.md)：以 RadixAttention 前缀复用与结构化生成 DSL 为核心的推理框架。

## 为什么单独看框架

- 显存是硬约束：KV Cache 常常与权重同量级，“怎么存”比“怎么算”更决定吞吐。
- 请求长度差异极大：静态 batch 会浪费大量算力，动态调度才是常态。
- 复用无处不在：系统提示词、few-shot 前缀、多轮对话历史天然大量重复，缓存命中率可直接换算成吞吐。
- 这些问题无法靠“换一个更快的算子”解决，属于调度与内存管理层面的设计问题。

## vLLM vs SGLang 对照

| 维度 | vLLM | SGLang |
|---|---|---|
| 调度粒度 | 请求级 + 迭代级 continuous batching | 请求级 + 前缀树感知的批次组织 |
| KV Cache 管理 | PagedAttention，block 化、按页复用 | RadixAttention，按前缀树节点复用 |
| 前缀缓存核心结构 | block table / 哈希命中 | radix tree |
| 结构化输出 | 受限解码（guided decoding） | compressed FSM + 前端 DSL |
| 典型场景 | 通用高吞吐 API 服务 | 强前缀复用、多轮 / 结构化输出场景 |

> 两者边界在持续互相吸收：vLLM 也在做前缀复用与结构化输出，SGLang 也在做分页式内存管理。不宜把它们看作互斥选项。

!!! note "TODO"
    - 补一栏“实测关注指标”（吞吐 / 首 token 延迟 / 显存占用），用于横向对比。
    - 记录一次真实选型判断的依据。
