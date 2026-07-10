# 大模型推理与训练技术栈学习指南

> 基于多期 Sprint Planning 文档（2024.09-2025.09）提炼的核心技能、关键仓库及系统化学习路径。

## 一、核心技能点（按技术领域分类）

### 1. 模型推理与部署

| 技能点 | 说明 | 关联任务 |
| --- | --- | --- |
| **vLLM 推理框架** | 掌握 vLLM 架构、调度策略、KV Cache 管理、continuous batching | 版本升级、参数调优（chunked-prefill, prefix-caching）、多卡 TP/PP/EP |
| **SGLang 推理** | 熟悉 SGLang 的 speculative decoding、融合算子、后端适配 | deepseek MTP、flash attention 后端适配 |
| **Prefix Caching** | 理解缓存命中率优化、cache hit 后跳过 forward 逻辑 | prefix-caching 优化（提升命中率 10%） |
| **Paged Attention** | 分页注意力机制、内存管理、性能分析（nsys） | paged_attn 占比优化、多卡场景分析 |
| **Speculative Decoding** | 草稿模型 + 目标模型验证、融合算子、accept rate 对齐 | deepseek MTP、EAGLE、Medusa |
| **Chunked Prefill** | 分块预填充策略，平衡吞吐与延迟 | 参数 enable-chunked-prefill 测试与性能分析 |

### 2. 模型训练与微调

| 技能点 | 说明 | 关联任务 |
| --- | --- | --- |
| **DeepSpeed** | ZeRO 1/2/3 优化、混合精度、通信策略 | llama-factory + deepspeed 多卡训练 |
| **Megatron-LM** | 张量并行、流水线并行、序列并行、DP/TP/PP 组合 | llama2 训练、MFU/HFU 计算 |
| **NeMo** | NVIDIA 大模型框架，支持 LoRA、SFT、并行策略 | nemo-run、docker/slurm executor 调试 |
| **llama-factory** | 统一微调框架，支持多种模型和 PEFT 方法 | qwen/deepseek 模型微调 |
| **Transformers 库** | Hugging Face 生态，模型加载、训练、推理 | rebase、版本适配 |
| **混合精度训练** | bf16/fp32 精度训练，精度毛刺问题定位 | long-running SFT 精度稳定性 |

### 3. 并行与分布式策略

| 技能点 | 说明 | 关联任务 |
| --- | --- | --- |
| **TP（张量并行）** | 层内参数切分 | qwen2.5-72B TP4 优化 |
| **DP（数据并行）** | 数据批次切分 | TP+DP 组合（如 TP4+DP2） |
| **PP（流水线并行）** | 层间切分，1F1B 调度 | Megatron TP2+PP2/PP4 训练 |
| **EP（专家并行）** | MoE 模型专家切分 | deepseek-r1 671B TP+EP |
| **通信原语** | allreduce、gather、scatter、broadcast、reduce_scatter | DLCCL 调试、ring_exchange |
| **MoE 优化** | 列优先 linear 计算、fused_moe、moe_align_block_size | deepseek 671B 性能优化 |

### 4. 量化与压缩

| 技能点 | 说明 | 关联任务 |
| --- | --- | --- |
| **AWQ（激活感知权重量化）** | 4-bit 量化，精度与性能权衡 | DeepSeek-V2-Lite-Chat-AWQ 测试 |
| **GPTQ** | 训练后量化（PTQ） | gptq_gemm 算子插入与测试 |
| **INT8 量化** | 8-bit 矩阵乘 | matmul int8 类型优化 |

### 5. 算子级优化

| 技能点 | 说明 | 关联任务 |
| --- | --- | --- |
| **融合算子** | fused_rms_norm、fused_adamw、fused_rope、fused_moe | 替换 TransformerEngine 对应实现 |
| **内存优化** | pin_memory、异步拷贝（synmemcpyasync）、CMA 内存管理 | copy_stride_sr、pin_memory async copy |
| **Kernel 优化** | matmul 分块大小、bank conflict 解决、列优先布局 | matmul_padding、列优先 linear |
| **Attention 变体** | MLA（Multi-head Latent Attention）、duo-attention、sliding window | deepseek MLA、gemma3 sliding window |
| **自定义算子** | paged_attention_v2、reshape_and_cache、gptq_gemm、moe_align_block_size | 算子插入与测试、CI 集成 |

### 6. 性能分析与 Profiling

| 技能点 | 说明 | 关联任务 |
| --- | --- | --- |
| **Nsight Systems** | GPU 耗时分析，CUDA kernel 占比 | paged_attn 占比分析、多卡 profiling |
| **PyTorch Profiler** | 算子级别耗时、内存占用 | vLLM 调度开销分析 |
| **MFU / HFU** | 模型浮点利用率、硬件浮点利用率 | Megatron 添加 MFU/HFU 指标 |
| **Benchmark** | throughput、latency、serving 测试 | benchmark_throughput、benchmark_serving 跑通 |

### 7. 编译器与图优化

| 技能点 | 说明 | 关联任务 |
| --- | --- | --- |
| **IREE** | 机器学习编译器，图模式编译 | vLLM v1 接入 IREE bf16，graph 优化 |
| **torch.compile** | PyTorch 动态图编译 | vLLM v1 支持 torch.compile 和 dynamo |
| **torch.dynamo** | 图捕获与转换 | 接入 dynamo + iree 图模式框架 |

### 8. 模型适配与多模态

| 技能点 | 说明 | 关联任务 |
| --- | --- | --- |
| **大语言模型** | Qwen、DeepSeek、Llama、Gemma、ChatGLM、Baichuan | 适配、推理、训练 |
| **多模态模型** | Qwen2.5-VL（视频输入）、GLM-4.5V、SAM、GroundingDINO | 视频输入优化、conv3d 融合 |
| **文生图/视频** | FLUX.1、Wan2.1、Stable Diffusion | FLUX 适配、Wan2 优化 |
| **语音模型** | CosyVoice、FunASR、Fish-Speech | 数字人适配、benchmark |
| **目标检测** | YOLO v8/v9/v11、D-FINE | 适配与性能优化 |

## 二、关键代码仓库与工具（需深入吃透）

### 推理框架

| 仓库 | 用途 | 学习重点 |
| --- | --- | --- |
| **[vLLM](https://github.com/vllm-project/vllm)** | 高性能 LLM 推理 | 调度器、KV Cache 管理、Paged Attention、多卡并行、prefix-caching、chunked-prefill |
| **[SGLang](https://github.com/sgl-project/sglang)** | 结构化生成语言 | Speculative decoding、融合算子、flash attention 后端 |
| **[Hugging Face Transformers](https://github.com/huggingface/transformers)** | 模型加载与基础推理 | 模型结构、tokenizer、generation 配置 |

### 训练框架

| 仓库 | 用途 | 学习重点 |
| --- | --- | --- |
| **[DeepSpeed](https://github.com/microsoft/DeepSpeed)** | 大规模分布式训练 | ZeRO 优化、混合精度、通信策略、MoE 支持 |
| **[Megatron-LM](https://github.com/NVIDIA/Megatron-LM)** | 超大规模训练框架 | TP/PP/DP 组合、序列并行、1F1B 调度、MFU 计算 |
| **[NeMo](https://github.com/NVIDIA/NeMo)** | 端到端大模型开发 | 分布式训练、LoRA、SFT、docker/slurm executor |
| **[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory)** | 统一微调平台 | 多模型支持、PEFT 方法、deepspeed 集成 |

### 编译与优化

| 仓库 | 用途 | 学习重点 |
| --- | --- | --- |
| **[IREE](https://github.com/iree-org/iree)** | ML 编译器 | 图捕获、算子编译、bf16 支持、capture 调试 |
| **[PyTorch](https://github.com/pytorch/pytorch)** | 基础框架 | torch.compile、dynamo、分布式通信包、算子实现 |

### 通信库与底层

| 仓库 | 用途 | 学习重点 |
| --- | --- | --- |
| **DLCCL（内部）** | 自研通信库 | allreduce、gather、scatter、ring 拓扑、超时调试 |
| **Flash Attention** | 高效注意力实现 | FlashInfer、paged_attn、sdpa 融合 |

### 基准测试与工具

- **nsys**（NVIDIA Nsight Systems）：GPU 性能分析。
- **PyTorch Profiler**：算子耗时统计。
- **Benchmark 脚本**：throughput/latency/serving 测试。

## 三、学习指南规划（分阶段实施）

### 第一阶段：基础理论铺垫（2-3 周）

目标：建立大模型推理与训练的核心知识体系。

1. **Transformer 深度理解**
   - 阅读《Attention Is All You Need》。
   - 手写简易 Transformer（PyTorch）。
   - 理解 Multi-Head Attention、FFN、LayerNorm、Positional Encoding。

2. **分布式训练基础**
   - 数据并行、模型并行（张量/流水线）原理。
   - DeepSpeed ZeRO 1/2/3 机制。
   - 通信原语（allreduce、broadcast、gather/scatter）。

3. **推理性能基础**
   - KV Cache 原理、Paged Attention 论文（vLLM 核心）。
   - Continuous Batching 与静态 batching 对比。
   - Speculative Decoding 论文（DeepMind 和 Google 相关工作）。

4. **MoE 与量化基础**
   - MoE 架构（GShard、Switch Transformer）。
   - AWQ/GPTQ 量化原理。

### 第二阶段：推理框架专项（3-4 周）

目标：精通 vLLM 和 SGLang，能独立完成模型适配和性能调优。

1. **vLLM 源码剖析**
   - 核心模块：scheduler、kv_cache_manager、worker、model_runner。
   - 并行策略：TP/PP 实现，`initialize_model_parallel` 流程。
   - 新增模型适配流程（如 Qwen-Image、DeepSeek-R1）。
   - 参数调优：`--enable-chunked-prefill`、`--prefix-caching`、`--max-num-seqs`。
   - 性能分析：使用 nsys 定位 paged_attn 耗时瓶颈。

2. **SGLang 实践**
   - 对比 vLLM 差异，理解其 speculative decoding 实现。
   - 融合算子（如 `fused_moe`）的插入与测试。

3. **实战任务（仿 Sprint）**
   - 在 4 卡/8 卡上跑通 DeepSeek-R1-Distill-Llama-70B 并调优。
   - 实现 prefix-caching 优化，命中率提升 10%。
   - 对比 A100 vs 自研硬件（DLC）推理性能。

### 第三阶段：训练框架专项（3-4 周）

目标：掌握 DeepSpeed + Megatron + NeMo，能开展大模型预训练与微调。

1. **DeepSpeed 深入**
   - 配置 ZeRO 1/2/3，理解 offload 机制。
   - 与 llama-factory 集成，跑通 qwen2-7b、llama2-70b 微调。
   - 调试 deepspeed + dlccl 通信问题（卡死、NaN）。

2. **Megatron-LM 掌握**
   - 构建 GPT 模型，配置 TP/PP/DP 组合。
   - 理解 pipeline 调度（1F1B）和 interleaved 调度。
   - 添加 MFU/HFU 指标，分析训练效率。
   - 调试常见问题：reshape_offset 卡死、allreduce 结果错误、NaN 定位。

3. **NeMo 框架**
   - 跑通 nemo-run（docker/slurm executor）。
   - 实现 NeMo + Megatron Core 的混合并行训练。
   - 融合算子替换（fused_adamw、fused_rms_norm）。

4. **实战任务**
   - 8 卡 llama2-70b 训练，与 A100 性能对比。
   - qwen2.5-32b 微调，解决 loss 异常与精度毛刺。
   - 多机多卡训练（如 4 机 32 卡 DeepSeek-R1）。

### 第四阶段：算子开发与优化（2-3 周）

目标：能独立开发/插入自定义算子，优化 kernel 性能。

1. **PyTorch 算子扩展**
   - 实现 C++/CUDA 扩展，注册自定义算子。
   - 处理连续内存（contiguous）和非连续 tensor 问题。
   - 常用算子：matmul、sdpa、reshape_and_cache、gptq_gemm。

2. **融合算子设计**
   - 融合 RMSNorm + Matmul 或 Attention 中的多个操作。
   - 实现 fused_moe、fused_rope，参考 FlashAttention 思路。

3. **性能调优**
   - 使用 nsys 和 nvprof 分析 kernel 耗时。
   - 分块大小调优（避免 bank conflict）。
   - 异步拷贝（`cudaMemcpyAsync`）与 pin_memory 优化。

4. **实战任务**
   - 为 vLLM 插入缺失算子（如 `moe_align_block_size`）。
   - 优化 `copy_stride_sr` 提升多卡通信效率。
   - 添加 `index_put` 算子支持 new 数据类型。

### 第五阶段：分布式通信与编译器（2 周）

目标：理解通信库内部机制，能利用编译器进行图优化。

1. **DLCCL 调试**
   - 理解集合通信超时、卡死原因（如 send/recv rank 不匹配）。
   - 测试 ring/tree/hypercube 拓扑性能。
   - 修复 broadcast_object_list、all_gather_object 相关问题。

2. **IREE + torch.compile**
   - 配置 IREE 编译流程，捕获子图。
   - 修复 capture stream 不一致、replay 卡死等问题。
   - 对比 eager 模式与 compiled 模式性能收益。

3. **实战任务**
   - 使 vLLM v1 支持 torch.compile + IREE 后端（bf16）。
   - 为 dynamo 接入增加弱引用 tensor 支持。

### 第六阶段：模型适配与多模态拓展（2 周）

目标：快速适配新模型，包括多模态和文生图/视频。

1. **多模态模型适配流程**
   - 理解视觉编码器（CLIP、SigLIP）与 LLM 拼接方式。
   - 处理视频输入的 conv3d 和 reshape 操作。
   - 常见适配模型：Qwen2.5-VL、GLM-4.5V、SAM。

2. **文生图/视频模型**
   - FLUX.1、Wan2.1 的 diffusers 管道。
   - 解决 matmul 形状不支持、batch size 限制问题。

3. **语音模型**
   - CosyVoice、FunASR 的推理与 benchmark。

4. **实战任务**
   - 将 GroundingDINO、SAM 添加到 ModelZoo。
   - 适配 Qwen-Image 并加入 CI 用例。

### 第七阶段：CI 与自动化测试（1 周）

目标：建立完整的模型测试与回归体系。

1. **CI 流水线搭建**
   - 集成模型推理/训练测试，自动检测算子缺失/精度问题。
   - 多卡环境下的 benchmark 自动运行。

2. **性能回归监控**
   - 对比每日性能数据（throughput/latency）。
   - 通过 CI 触发 profiling，自动上传报告。

## 四、推荐学习资源

### 论文必读

- vLLM: Efficient Memory Management for Large Language Model Serving。
- FlashAttention: Fast and Memory-Efficient Exact Attention。
- GShard: Scaling Giant Models with Conditional Computation。
- AWQ: Activation-aware Weight Quantization。
- DeepSpeed ZeRO: Memory Optimization。

### 官方文档

- vLLM 官方文档与博客。
- PyTorch Distributed Tutorials。
- DeepSpeed 文档与 examples。
- Megatron-LM 用户指南。

### 源码阅读顺序建议

1. vLLM 的 `scheduler.py` 和 `worker.py`。
2. DeepSpeed 的 `zero_stage` 相关代码。
3. Megatron-LM 的 `pretrain_gpt.py` 和 `pipeline_parallel`。
4. PyTorch `torch/distributed` 下的通信实现。

## 五、最终验证项目（综合实战）

完成以下任务作为结业考核，模拟真实 Sprint：

1. **从零适配一个新模型**：如 Qwen3-235B 到 vLLM 推理，支持 TP+EP，实现 benchmark throughput 不低于 A100 的 90%。
2. **训练任务**：用 Megatron 在 8 卡上训练 llama2-70b，完成 100 次迭代，loss 收敛，MFU 达到 45% 以上。
3. **性能优化**：将 prefix-caching 命中率从 60% 提升至 75%，并量化其对 latency 的改进。
4. **算子贡献**：为 vLLM 或 PyTorch 社区提交一个融合算子 PR（如 `fused_moe`）。

本指南涵盖从理论学习到实战落地的完整路径。建议按阶段推进，每个阶段完成后进行复盘总结。文档中的 Sprint 任务可作为练习题，直接对标工业界真实需求。
