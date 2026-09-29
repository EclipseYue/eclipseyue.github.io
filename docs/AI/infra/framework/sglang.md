# SGLang

> SGLang 的特色是“前缀复用 + 结构化生成”：用 RadixAttention 把共享前缀的 KV 复用到极致，再用前端 DSL 描述生成流程。

## 核心设计

- **RadixAttention**：用 radix tree（前缀树）管理 KV Cache，树节点对应一段共享的 token 前缀。
  - 多轮对话、系统提示词、few-shot 示例天然共享前缀，命中后直接复用节点，无需重算 prefill。
  - 按序列管理 KV 的方式无法表达“跨请求共享”，这是它的主要差异点。
- **Radix Tree 调度**：调度器按树上可复用的前缀组织 batch，优先匹配命中，减少重复计算。
- **前缀缓存**：命中粒度比 block 化方案更细，较短的共享前缀也能复用。

## 前端与结构化生成

- **Frontend DSL**：用 Python 语法描述“生成 → 分支 → 约束”的流程（类似可编程 prompt），把多步调用收敛成一段程序。
- **Structured generation**：用 compressed FSM（压缩有限状态机）在解码时约束输出，直接产出 JSON / 正则 / 语法合法的结果，而不是事后解析再重试。
- 这两点使它在“前端编排 + 强约束输出”的场景中比通用服务框架更顺手。

## 与 vLLM 的差异

- KV Cache 组织：radix tree 对比分页 block table。
- 强项场景：强前缀复用（多轮对话、Agent 反复携带长上下文）、结构化输出。
- 共同点：都做 continuous batching、都支持 PD 分离，并且都在吸收对方的能力。

## 相关

- [框架分析](index.md)
- [vLLM](vllm.md)
- [推理总览](../inference/index.md)

!!! note "TODO"
    - 补一次 RadixAttention 命中 / 未命中的具体对比。
    - 记录 compressed FSM 在复杂 schema 下的性能表现。
