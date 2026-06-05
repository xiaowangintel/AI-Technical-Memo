# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/README.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This is the index and overview page for vLLM's quantization documentation. It links to all supported quantization formats, provides a hardware compatibility matrix, explains out-of-tree quantization plugin registration, and points to the `@register_quantization_config` decorator API.

**CN:** 本文档是 vLLM 量化文档的索引和概览页面，链接到所有支持的量化格式，提供硬件兼容性矩阵，介绍树外量化插件注册机制，并指向 `@register_quantization_config` 装饰器 API。

### Supported Formats Linked / 链接的支持格式

**EN:** AutoAWQ, BitsAndBytes, GGUF, GPTQModel, Intel Neural Compressor, INT4 W4A16, INT8 W8A8, FP8 W8A8, NVIDIA Model Optimizer, Online Quantization, AMD Quark, Quantized KV Cache, TorchAO, FP8 ViT Encoder Attention.

**CN:** AutoAWQ、BitsAndBytes、GGUF、GPTQModel、Intel Neural Compressor、INT4 W4A16、INT8 W8A8、FP8 W8A8、NVIDIA Model Optimizer、在线量化、AMD Quark、量化 KV 缓存、TorchAO、FP8 ViT 编码器注意力。

### Hardware Compatibility Matrix / 硬件兼容性矩阵

**EN:** Table showing support for AWQ, GPTQ, Marlin, INT8, FP8, BnB, DeepSpeedFP, GGUF across Volta/Turing/Ampere/Ada/Hopper/AMD GPU/Intel GPU/x86 CPU. Notable: FP8 W8A8 only on Ada+ and AMD GPU; INT8 not on Blackwell; Marlin not on Volta; Intel Gaudi migrated to vLLM-Gaudi project.

**CN:** 展示 AWQ、GPTQ、Marlin、INT8、FP8、BnB、DeepSpeedFP、GGUF 在 Volta/Turing/Ampere/Ada/Hopper/AMD GPU/Intel GPU/x86 CPU 上支持情况的表格。要点：FP8 W8A8 仅在 Ada+ 和 AMD GPU 上支持；INT8 在 Blackwell 上不支持；Marlin 在 Volta 上不支持；Intel Gaudi 已迁移至 vLLM-Gaudi 项目。

### Out-of-Tree Plugin Registration / 树外插件注册

**EN:** Custom quantization methods can be registered using `@register_quantization_config` on a `QuantizationConfig` subclass. Requires implementing: `get_name()`, `get_supported_act_dtypes()`, `get_min_capability()`, `get_config_filenames()`, `from_config()`, `get_quant_method()`. Dispatch to `LinearBase` (via `UnquantizedLinearMethod`) or `FusedMoE` (via `FusedMoEMethodBase`) based on layer type.

**CN:** 使用 `@register_quantization_config` 装饰 `QuantizationConfig` 子类来注册自定义量化方法，需要实现：`get_name()`、`get_supported_act_dtypes()`、`get_min_capability()`、`get_config_filenames()`、`from_config()`、`get_quant_method()`，根据层类型分发到 `LinearBase`（通过 `UnquantizedLinearMethod`）或 `FusedMoE`（通过 `FusedMoEMethodBase`）。

## Key Concepts / 关键概念

- **EN:** Quantization index — this README serves as the entry point; always start here to discover the right tool for your hardware and model. **CN:** 量化索引——本 README 作为入口点，总是从这里开始发现适合您硬件和模型的正确工具。
- **EN:** Hardware matrix — critical reference for choosing a quantization method that is actually supported on your GPU. **CN:** 硬件矩阵——选择在您的 GPU 上实际支持的量化方法的关键参考。
- **EN:** `@register_quantization_config` — the extension point for adding custom/third-party quantization without modifying vLLM core. **CN:** `@register_quantization_config`——无需修改 vLLM 核心即可添加自定义/第三方量化的扩展点。
- **EN:** `get_min_capability()` — returns the minimum GPU compute capability; return -1 for hardware-agnostic methods. **CN:** `get_min_capability()`——返回最低 GPU 算力要求，与硬件无关的方法返回 -1。
- **EN:** Intel Gaudi — all Intel Gaudi quantization support moved to the separate `vllm-gaudi` project; not covered by this matrix. **CN:** Intel Gaudi——所有 Intel Gaudi 量化支持已迁移至独立的 `vllm-gaudi` 项目，不在此矩阵覆盖范围内。
