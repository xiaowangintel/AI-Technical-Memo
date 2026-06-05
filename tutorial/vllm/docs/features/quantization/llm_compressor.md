# llm_compressor.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/llm_compressor.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This is the top-level introduction to LLM Compressor — the primary recommended library for quantizing models for deployment with vLLM. It explains the motivation for quantization, lists key features, and provides references to examples and the GitHub repository.

**CN:** 本文档是 LLM Compressor 的顶层介绍——这是为 vLLM 部署量化模型的首选推荐库。内容说明量化的动机、列出关键特性，并提供示例和 GitHub 仓库的参考链接。

### Why Quantize / 为什么量化

**EN:** Modern LLMs store billions of parameters in 16/32-bit float, requiring large GPU memory. Quantization reduces precision of weights and activations to smaller data types, lowering memory requirements while maintaining output quality.

**CN:** 现代 LLM 以 16/32 位浮点数存储数十亿参数，需要大量 GPU 内存。量化将权重和激活的精度降低为更小的数据类型，在保持输出质量的同时降低内存需求。

### Benefits / 优点

**EN:** (1) Reduced memory footprint — run larger models on smaller GPUs. (2) Lower inference costs — serve more concurrent users per GPU. (3) Faster inference — less memory bandwidth consumed, higher throughput especially for memory-bound workloads.

**CN:** (1) 减少内存占用——在更小的 GPU 上运行更大的模型。(2) 降低推理成本——每 GPU 服务更多并发用户。(3) 更快推理——消耗更少内存带宽，提升吞吐量，尤其对内存受限工作负载效果显著。

### Key Features / 关键特性

**EN:** Multiple algorithms (AWQ, GPTQ, AutoRound, RTN, QuIP, SpinQuant); multiple formats (FP8, INT8, INT4, NVFP4, MXFP4, mixed-precision); one-shot quantization; seamless vLLM integration via compressed-tensors format; HuggingFace compatibility.

**CN:** 多种算法（AWQ、GPTQ、AutoRound、RTN、QuIP、SpinQuant）；多种格式（FP8、INT8、INT4、NVFP4、MXFP4、混合精度）；一次性量化；通过 compressed-tensors 格式与 vLLM 无缝集成；HuggingFace 兼容。

### Integration / 集成

**EN:** LLM Compressor produces models in `compressed-tensors` format, which vLLM loads natively. No special flag is needed for most quantized models produced by this library.

**CN:** LLM Compressor 以 `compressed-tensors` 格式生成模型，vLLM 原生加载该格式，大多数由此库生成的量化模型无需特殊标志。

## Key Concepts / 关键概念

- **EN:** `compressed-tensors` format — the wire format used between LLM Compressor and vLLM for quantized model exchange. **CN:** `compressed-tensors` 格式——LLM Compressor 和 vLLM 之间交换量化模型使用的存储格式。
- **EN:** One-shot quantization — quantize with minimal calibration data in a single pass, without iterative fine-tuning. **CN:** 一次性量化——只需少量校准数据，单次完成量化，无需迭代微调。
- **EN:** KV cache and attention quantization — LLM Compressor also supports quantizing the KV cache and attention layers, not just linear weights. **CN:** KV 缓存和注意力量化——LLM Compressor 还支持量化 KV 缓存和注意力层，不仅限于线性权重。
- **EN:** Memory-bound workloads — quantization provides the biggest throughput gains for inference that is bottlenecked by memory bandwidth. **CN:** 内存受限工作负载——量化对内存带宽受限的推理提供最大的吞吐量收益。
- **EN:** vLLM's recommended entry point — the doc explicitly positions LLM Compressor as the starting point for all quantization workflows. **CN:** vLLM 推荐入口——文档明确将 LLM Compressor 定位为所有量化工作流的起点。
