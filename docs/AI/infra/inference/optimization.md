# 推理优化

推理优化的目标是在质量可接受的前提下降低延迟和成本。不同优化手段作用在不同层：模型结构、数值精度、运行时、调度和硬件。

## 优化方向

### 精度与量化

- FP16 / BF16：常用混合精度格式。
- INT8 / INT4：降低显存和带宽压力。
- AWQ / GPTQ：常见权重量化方法。

### 内存优化

- KV Cache 管理。
- Paged Attention。
- 权重共享和加载策略。
- CPU/GPU offload。

### 解码优化

- Continuous Batching。
- Speculative Decoding。
- Prefix Cache。
- Early Exit 或约束解码。

### 算子优化

- FlashAttention。
- Kernel Fusion。
- TensorRT / ONNX Runtime / vLLM / TGI。

## 取舍

- 量化可能影响质量。
- 更大的 batch 提高吞吐，但可能增加延迟。
- Prefix Cache 依赖请求模式。
- Speculative Decoding 依赖小模型命中率。
