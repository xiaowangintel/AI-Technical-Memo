# quantized_kv_cache.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/quantized_kv_cache.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Quantized KV cache reduces the memory footprint of key-value cache storage by using lower-precision data types (FP8 or FP4) instead of the default model precision in BF16. / 该文档围绕 Quantized KV 缓存 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Quantized KV cache reduces the memory footprint of key-value cache storage by using lower-precision data types (FP8 or FP4) instead of the default model precision in BF16.
**CN:** 本节围绕 Overview 展开，概述了 cache, memory, Quantized, attention 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Formats
**EN:** SGLang supports the following quantized KV cache formats:
**CN:** 本节围绕 Supported Formats 展开，概述了 cache, formats, supports, following 等要点，并说明相关配置、流程、示例或限制条件。

### Section: FP8 Format
**EN:** OCP (Open Compute Project) specifies two common 8-bit floating point formats: - **E5M2** (5 exponent bits, 2 mantissa bits): Larger dynamic range (±57344.0), lower precision - **E4M3** (4 exponent bits, 3 mantissa bits): Higher precision, smaller dynamic range (±240.0)
**CN:** 本节围绕 FP8 Format 展开，概述了 bits, OCP, E5M2, E4M3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: FP4 Format
**EN:** `` FP4 quantization is currently experimental.
**CN:** 本节围绕 FP4 Format 展开，概述了 bit, FP4, OCP, blocks 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Enabling Quantized KV Cache
**EN:** This content focuses on Enabling Quantized KV Cache and highlights --kv-cache-dtype, cache, kv-cache-dtype, Enable.
**CN:** 本节围绕 Enabling Quantized KV 缓存 展开，概述了 --kv-cache-dtype, cache, kv-cache-dtype, Enable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Scaling Factors
**EN:** FP8 quantization requires scaling factors to properly quantize and dequantize the KV cache. `` Currently, only per-tensor (scalar) scaling factors are supported.
**CN:** 本节围绕 Scaling Factors 展开，概述了 scaling, factors, FP4, FP8 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory Savings
**EN:** FP4 with block size 16 supports approximately 1.78× more tokens than FP8, and approximately 3.56× more tokens than BF16. The relative token capacity between FP8 and BF16 can be derived from these ratios.
**CN:** 本节围绕 Memory Savings 展开，概述了 BF16, FP4, FP8, memory 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Accuracy Impact
**EN:** This section provides a comparison table for Accuracy Impact, covering columns such as Model, Dataset, KV16 and examples such as Qwen3-235B-A22B, Qwen3-235B-A22B, Qwen3-235B-A22B, DeepSeek-R1-0528.
**CN:** 本节围绕 Accuracy Impact 展开，概述了 FP4, accuracy, FP8, GPT-OSS-120B 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Best Practices
**EN:** **Use pre-quantized models**: Prefer models quantized offline with scaling factors included in the checkpoint.
**CN:** 本节围绕 Best Practices 展开，概述了 Check, Prefer, Choose, Verify 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** FP4 / **CN:** FP4
- **EN:** FP8 / **CN:** FP8
- **EN:** cache / **CN:** 缓存
- **EN:** BF16 / **CN:** BF16
- **EN:** accuracy / **CN:** accuracy
- **EN:** quantization / **CN:** 量化
- **EN:** scaling / **CN:** scaling
- **EN:** memory / **CN:** memory

## Dependencies / 依赖关系
- `attention_backend.md`
- `quantization.md`
- `server_arguments.md`
