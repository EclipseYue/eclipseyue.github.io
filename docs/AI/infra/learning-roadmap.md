# AI Infra 学习路线

这份路线面向个人长期学习和项目实施，目标不是泛泛补课，而是围绕“能读懂核心仓库、能定位框架问题、能把模型在新硬件或新服务栈上跑准跑快”建立能力闭环。

当前无法读取 `C:\Users\heiling\Downloads\框架Sprint Planning(2024.09.02-).pdf`，因此这里先基于已有公开笔记和现阶段方向整理。后续如果有 Sprint 文档文本版，可以再补充项目特定任务。

## 总目标

1. 吃透 PyTorch 到设备后端的执行路径，能解释 `torch.xxx` 如何走到 dispatcher、backend implementation、runtime 和 kernel。
2. 吃透 vLLM 的推理服务路径，能解释请求如何进入 scheduler、KV cache、attention kernel、sampling 和 OpenAI-compatible API。
3. 掌握大模型推理核心优化，包括量化、MoE、Paged Attention、Continuous Batching、Speculative Decoding 和 Prefix Cache。
4. 建立调试方法论：模型级 failure 能压缩到 op 级、kernel 级或 runtime 级最小复现。
5. 最终形成可展示产出：源码阅读笔记、最小实验、性能/正确性报告、面试项目表达。

## 技能地图

| 模块 | 必须吃透的技能点 | 验收标准 |
| --- | --- | --- |
| C++ / Python 扩展 | CPython extension、pybind11、CMake、ABI、动态链接、wheel 构建 | 能解释 `import torch_xxx` 时 C++ extension 如何初始化 |
| PyTorch 核心 | Tensor、Storage、Device、Stream、DispatchKey、ATen dispatcher、autograd 基本边界 | 能从 Python API 追到 C++ op 注册和 backend kernel |
| PyTorch 设备后端 | PrivateUse1、allocator、device guard、copy、fallback、op registration、out variant | 能实现并解释 `empty -> copy -> add/relu/sum -> cpu check` 最小链 |
| Runtime / Kernel | stream/event、memcpy、launch protocol、参数打包、shape/stride/layout、tiling、padding | 能把一次 op 调用拆成 descriptor、runtime launch、kernel 执行和回拷校验 |
| 推理服务 | tokenizer、prefill、decode、scheduler、batching、OpenAI API、流式返回、异常隔离 | 能搭建 vLLM 服务并解释请求生命周期 |
| KV Cache | KV layout、block table、Paged Attention、prefix cache、cache eviction、显存估算 | 能算出模型在给定 batch/seq 下的 KV cache 显存占用 |
| 量化 | FP16/BF16、INT8/INT4、per-tensor/per-channel/per-group、AWQ、GPTQ、SmoothQuant、KV cache quant | 能说明量化如何影响显存、带宽、吞吐、延迟和精度 |
| MoE | expert routing、top-k gating、capacity factor、expert parallel、all-to-all、load balancing、MoE quant | 能解释 MoE 推理为什么容易受通信、路由偏斜和小 batch 影响 |
| 分布式 | tensor parallel、pipeline parallel、data parallel、expert parallel、NCCL/RCCL、collective debug | 能判断单机多卡推理瓶颈是在通信、算子还是调度 |
| 性能分析 | TTFT、TPOT、throughput、P50/P99、GPU utilization、memory bandwidth、kernel timeline | 能写出一次推理压测报告并定位主要瓶颈 |
| 正确性调试 | CPU reference、数值容差、dump tensor、最小 shape、seed 固定、二分定位 | 能把模型输出异常压缩到具体 op、dtype、layout 或 kernel case |
| 工程交付 | 文档、测试矩阵、CI、版本兼容、回归用例、benchmark baseline | 每个阶段都有可复跑脚本和可复查记录 |

## 重点仓库

| 仓库 | 学习重点 | 阅读入口 |
| --- | --- | --- |
| `pytorch/pytorch` | dispatcher、ATen、TensorImpl、Storage、Device、PrivateUse1、op registration | `aten/`、`c10/`、`torch/csrc/`、`torchgen/` |
| `vllm-project/vllm` | LLM serving、scheduler、Paged Attention、KV cache、sampling、OpenAI API、quantization | `vllm/engine/`、`vllm/core/`、`vllm/worker/`、`vllm/model_executor/` |
| `sgl-project/sglang` | 另一套推理运行时设计，对比 scheduler、radix cache、server runtime | `python/sglang/srt/` |
| `huggingface/transformers` | 模型结构、generation、config/tokenizer、权重加载 | `src/transformers/models/`、`generation/` |
| `huggingface/accelerate` | 多设备加载、offload、分布式启动 | `src/accelerate/` |
| `AutoGPTQ/AutoGPTQ` | GPTQ 权重量化流程和推理侧权重格式 | quantizer、model wrappers |
| `mit-han-lab/llm-awq` | AWQ 量化思想、校准、group-wise quant | quantization scripts、kernels |
| `NVIDIA/TensorRT-LLM` | 工业级推理优化、plugin、engine build、quant、MoE | `cpp/`、`tensorrt_llm/`、examples |
| `flashinfer-ai/flashinfer` | attention、sampling、KV cache kernel 的现代实现 | kernels、Python bindings |
| `Dao-AILab/flash-attention` | attention kernel、tiling、IO-aware 优化 | csrc、hopper 相关实现 |
| `NVIDIA/nccl` | collective 通信和多卡性能问题背景 | docs、src collective 实现 |

## 学习阶段

### 阶段 0：补齐工程地基

周期：1 到 2 周。

学习内容：

- C++17 基础、模板、RAII、智能指针、宏和编译错误定位。
- Python extension、pybind11、CMake、动态链接、环境隔离。
- Linux 调试基础：`gdb`、`ldd`、`nm`、`objdump`、`perf`、日志和 core dump。

产出：

- 一个最小 pybind11 extension。
- 一篇“Python import C++ extension 发生了什么”的笔记。

### 阶段 1：PyTorch 设备后端主线

周期：3 到 5 周。

学习内容：

- `TensorImpl`、`Storage`、`Device`、`DispatchKey`。
- PrivateUse1 设备注册、allocator、device guard、stream、event。
- `empty`、`copy_`、`to(device)`、`clone`、`add`、`relu`、`sum` 的调用链。
- op schema、registration、fallback、out variant、dtype/shape/layout 处理。

实践顺序：

1. 追踪 `torch.empty(device="xxx")`。
2. 追踪 `x.to("cpu")` 和 `x.to("xxx")`。
3. 追踪 `torch.add(x, y)` 从 Python 到 backend implementation。
4. 为 `add -> relu -> sum` 写 CPU reference 对齐测试。
5. 记录每一步涉及的文件、函数、数据结构和失败模式。

产出：

- PyTorch dispatcher 源码阅读图。
- PrivateUse1 P0/P1 最小链笔记。
- 一个 op 级正确性排查模板。

### 阶段 2：vLLM 推理主线

周期：3 到 5 周。

学习内容：

- OpenAI-compatible server、engine、worker、model runner。
- prefill/decode 分离、scheduler、sequence group、block manager。
- Paged Attention、KV cache block table、显存预算。
- sampling、streaming、metrics、异常请求处理。

实践顺序：

1. 本地启动一个小模型 vLLM 服务。
2. 从 HTTP 请求追到 engine 和 scheduler。
3. 记录一次 prefill 和多次 decode 的关键对象变化。
4. 修改 batch/seq/concurrency，记录 TTFT、TPOT 和吞吐变化。
5. 对比 vLLM 和 SGLang 的 scheduler/cache 设计。

产出：

- vLLM 请求生命周期图。
- KV cache 显存估算表。
- 一份小模型压测报告。

### 阶段 3：量化专题

周期：2 到 4 周。

学习内容：

- 数值格式：FP32、TF32、FP16、BF16、FP8、INT8、INT4。
- 权重量化：per-channel、per-group、zero point、scale、packing。
- GPTQ、AWQ、SmoothQuant 的核心思想和适用场景。
- activation quant、KV cache quant、量化 kernel 和 dequant overhead。
- 量化对质量、显存、带宽、吞吐、延迟的影响。

实践顺序：

1. 用同一个模型比较 FP16、INT8、INT4 的显存和延迟。
2. 记录困惑度或任务指标变化。
3. 阅读 vLLM 里的 quantization config 和 kernel 调用路径。
4. 梳理 AWQ/GPTQ 权重格式如何进入推理框架。

产出：

- 量化方法对比表。
- 一份“为什么 INT4 不一定线性提速”的分析。

### 阶段 4：MoE 专题

周期：2 到 4 周。

学习内容：

- dense FFN 和 MoE FFN 的差异。
- router/gating、top-1/top-2、expert capacity、load balance loss。
- expert parallel、token dispatch、all-to-all、shared expert。
- MoE 推理中的 batch size、路由偏斜、专家并行和通信瓶颈。
- MoE 量化：expert weight quant、router 精度、hot expert 缓存。

实践顺序：

1. 读一个 Mixtral 或 Qwen-MoE 的模型结构。
2. 画出 token 从 hidden states 到 expert 输出的路径。
3. 在 vLLM 中追踪 MoE model executor 和相关 kernel。
4. 记录不同 batch/sequence 下专家负载分布。

产出：

- MoE 推理路径图。
- MoE 瓶颈清单：算子、通信、显存、调度。

### 阶段 5：Kernel 与性能分析

周期：长期迭代。

学习内容：

- GEMM、attention、softmax、layernorm、RMSNorm、RoPE、sampling。
- tiling、memory coalescing、shared memory/register pressure、occupancy。
- FlashAttention / Paged Attention 的 IO-aware 思路。
- profiler timeline、kernel launch overhead、CPU 调度开销。

产出：

- 一个 attention kernel 阅读笔记。
- 一份 vLLM 性能 profile 复盘。
- 一个 kernel 级最小复现模板。

## 每周执行模板

每周只追一个主问题，避免同时开太多坑。

| 时间 | 动作 | 产出 |
| --- | --- | --- |
| 周一 | 确定本周问题和验收标准 | 3 到 5 个具体问题 |
| 周二到周三 | 阅读源码和文档 | 调用链、关键类、关键函数 |
| 周四 | 做最小实验 | 可复跑脚本、日志、截图或表格 |
| 周五 | 写复盘 | 结论、坑点、下一步 |
| 周末 | 整理到个人网站 | 一篇文档或更新一张表 |

## 优先级建议

短期最该投入的是 PyTorch 后端和 vLLM，因为它们分别对应“框架底座”和“推理入口”。量化和 MoE 不要只看论文，要绑定 vLLM 或 TensorRT-LLM 的真实实现，否则容易停留在概念层。

推荐顺序：

1. PyTorch dispatcher + PrivateUse1 + 基础 op。
2. vLLM request lifecycle + scheduler + KV cache。
3. 量化在 vLLM 中的加载、权重格式和 kernel 路径。
4. MoE 模型结构、expert dispatch 和多卡通信。
5. FlashAttention / Paged Attention / sampling kernel。

## 面试和项目表达口径

每个专题都要沉淀为“问题 - 路径 - 结果 - 复盘”：

- 问题：为什么要做这个优化或适配。
- 路径：请求或 op 从上层 API 到底层 kernel 的链路。
- 结果：正确性、延迟、吞吐、显存、稳定性上的变化。
- 复盘：踩过的坑、如何定位、如何防止回归。

不要只说“了解 vLLM / PyTorch / MoE / 量化”。更好的表达是：能说清楚一个请求、一个 tensor、一个 op、一个 expert 或一个 quantized weight block 在系统里如何流动。
