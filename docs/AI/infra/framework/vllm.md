# vLLM

> vLLM 是面向大模型推理的高吞吐服务框架，“分页 KV Cache + continuous batching”是它最核心的两个设计。

## 核心设计

- **PagedAttention**：把 KV Cache 切成固定大小的 block（页），用 block table 维护逻辑到物理的映射，类似操作系统的虚拟内存分页。
  - 收益：显存碎片近乎为零；同一份 KV 可被多个序列共享（并行采样、beam search）。
  - 代价：引入间接寻址，attention kernel 需要按 block 读写。
- **Continuous batching**：以迭代（step）为单位调度，完成的请求立即退出、新请求立即加入，而不是等整批跑完。
  - 这是吞吐的关键：decode 阶段每步只算一个 token，静态 batch 的空闲浪费极大。
- **调度器**：按 token 预算与显存预算决定这一步跑哪些序列；chunked prefill 与 decode 如何混批是主要调优点。

## KV Cache 与显存

- KV Cache 占用与 `2 * layers * kv_heads * head_dim * seq_len * batch * dtype_size` 同阶，长上下文下可与权重相当。
- `gpu_memory_utilization`、`max_model_len`、`block_size`、`max_num_seqs` 共同决定可用并发。
- 前缀缓存命中后可直接复用已有 block，省掉这部分 prefill 计算。

## 服务与并行

- OpenAI 兼容 server：提供 `/v1/chat/completions` 等接口，便于替换已有调用方。
- 并行策略：TP（张量并行）是主力；PP（流水线并行）/ EP（专家并行）用于更大模型。
- attention backend 通常可切换（FlashAttention / FlashInfer / 自定义实现），不同 backend 对分页布局的支持程度不同。

## 性能关注点

- 吞吐与首 token 延迟存在取舍：chunked prefill 往往牺牲首 token 延迟换整体吞吐。
- prefill / decode 的资源配比，是引入 PD 分离的动机，见 [推理总览](../inference/index.md)。
- 显存碎片与 block 利用率，是超长 prompt 等长尾请求下的主要风险。

## 相关

- [框架分析](index.md)
- [SGLang](sglang.md)
- [推理总览](../inference/index.md)

!!! note "TODO"
    - 补一条“从启动参数到并发能力”的推导链。
    - 记录一次 prefix caching 命中率偏低的排查过程。
