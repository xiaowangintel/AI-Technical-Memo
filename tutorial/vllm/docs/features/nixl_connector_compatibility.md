# nixl_connector_compatibility.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/nixl_connector_compatibility.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This page is a compatibility matrix documenting which vLLM features and model architectures are supported when using the NixlConnector for disaggregated prefilling. It provides a detailed table, footnotes, and configuration notes on what must or can differ between prefill (P) and decode (D) instances.

**CN:** 本页面是一个兼容性矩阵，记录使用 NixlConnector 进行解耦预填充时，哪些 vLLM 功能和模型架构受到支持。包含详细表格、脚注，以及关于预填充（P）实例和解码（D）实例之间哪些配置必须或可以不同的说明。

### Universally Supported Features / 普遍支持的功能

**EN:** Chunked prefill, APC (prefix caching), data parallelism, CUDA graphs, logprobs, prompt logprobs, prompt embeds, and multiple NIXL backends (UCX, GDS, LIBFABRIC, etc.) work with all model architectures.

**CN:** 分块预填充、APC（前缀缓存）、数据并行、CUDA 图、logprobs、提示词 logprobs、提示词嵌入，以及多种 NIXL 后端（UCX、GDS、LIBFABRIC 等）适用于所有模型架构。

### Compatibility Matrix / 兼容性矩阵

**EN:** Key findings: Dense Transformers and MoE fully support basic PD, speculative decoding, hetero TP, cross-layer blocks, and SWA. MLA models (DeepSeek-V2/V3) have partial hetero TP support (no head-splitting). Hybrid SSM/Mamba requires homogeneous TP. Encoder-Decoder is unsupported for all features.

**CN:** 关键结论：稠密 Transformer 和 MoE 完全支持基本 PD、投机解码、异构 TP、跨层块和 SWA；MLA 模型（DeepSeek-V2/V3）对异构 TP 有部分支持（无头分割）；混合 SSM/Mamba 需要同构 TP；编码器-解码器架构对所有功能均不支持。

### P/D Configuration Rules / P/D 配置规则

**EN:** Must match: vLLM version, model architecture/dtype/KV heads/head size/layers, attention backend, cache dtype. Can differ: tensor-parallel-size, block-size (with restrictions), number of KV cache blocks.

**CN:** 必须相同：vLLM 版本、模型架构/数据类型/KV 头/头大小/层数、注意力后端、缓存数据类型。可以不同：tensor-parallel-size、block-size（有限制）、KV 缓存块数量。

### KV Cache Layout / KV 缓存布局

**EN:** NixlConnector defaults to `HND` layout for optimal transfer performance. `NHD` is supported but does not allow hetero TP head-splitting. An experimental `HND ↔ NHD` permute option exists (not supported with HMA).

**CN:** NixlConnector 默认使用 `HND` 布局以获得最佳传输性能。`NHD` 受支持但不允许异构 TP 头分割。存在实验性的 `HND ↔ NHD` 转置选项（不支持 HMA）。

## Key Concepts / 关键概念

- **EN:** Compatibility hash — handshake check ensuring P and D agree on model spec, backend, and cache dtype; can be disabled at risk. **CN:** 兼容性哈希——握手检查，确保 P 和 D 在模型规格、后端和缓存数据类型上一致，可自行承担风险禁用。
- **EN:** Hetero TP — P and D can use different tensor-parallel sizes; support depends on model architecture (MLA has limited head-splitting). **CN:** 异构 TP——P 和 D 可使用不同的张量并行大小，支持情况取决于模型架构（MLA 头分割有限制）。
- **EN:** Cross-layer blocks — requires `FLASH_ATTN`/`FLASHINFER` backend and `HND` layout; must be explicitly enabled. **CN:** 跨层块——需要 `FLASH_ATTN`/`FLASHINFER` 后端和 `HND` 布局，必须显式启用。
- **EN:** HMA (Hybrid Model Attention) — used by Mamba/SSM hybrids; incompatible with hetero block size and `NHD ↔ HND` permute. **CN:** HMA（混合模型注意力）——用于 Mamba/SSM 混合模型，与异构块大小和 `NHD ↔ HND` 转置不兼容。
- **EN:** Quantized KV cache — FP8 KV requires matching `cache_dtype` on both P and D; mismatch fails the compatibility hash check. **CN:** 量化 KV 缓存——FP8 KV 要求 P 和 D 上的 `cache_dtype` 相同，不匹配将导致兼容性哈希检查失败。
