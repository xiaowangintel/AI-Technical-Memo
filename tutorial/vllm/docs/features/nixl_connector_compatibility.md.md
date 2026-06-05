# nixl_connector_compatibility.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/nixl_connector_compatibility.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page is a compatibility matrix for disaggregated prefilling with NixlConnector. It is explicitly status-oriented: supported, partial, unsupported, unknown, and work-in-progress states are shown across model architectures and capabilities.
- **CN:** 本页是面向 NixlConnector 解耦 prefill 的兼容性矩阵。它明确以状态为中心，使用“支持 / 部分支持 / 不支持 / 未知 / 开发中”等标记，覆盖不同模型架构与能力组合。
- **EN:** The matrix shows broad support for dense, MLA, sparse MLA, and MoE models in basic PD mode, while multimodal remains mostly unknown and encoder-decoder is fully unsupported. Footnotes add important constraints, such as speculation config matching, cross-layer blocks requiring `FLASH_ATTN` or `FLASHINFER` plus `HND`, and heterogeneous block-size limits.
- **CN:** 矩阵显示：在基础 PD 模式下，dense、MLA、sparse MLA 与 MoE 的支持度较高；多模态大多仍是未知；encoder-decoder 则完全不支持。脚注补充了关键限制，例如 speculative 配置必须一致、cross-layer blocks 需要 `FLASH_ATTN` 或 `FLASHINFER` 且使用 `HND` 布局，以及异构 block size 的支持边界。
- **EN:** The configuration notes are especially actionable: P and D must match on vLLM/NIXL versions, model characteristics, attention backend, and `cache_dtype`, while TP size, block size, and block counts can differ within documented restrictions. The page also clarifies quantized-KV support: static and packed-layout scales are supported, but dynamic quantization is not.
- **CN:** 配置说明部分非常实用：P 与 D 必须在 vLLM / NIXL 版本、模型特征、attention backend 和 `cache_dtype` 上保持一致；而 TP size、block size 和 block 数量则可在文档约束下不同。文档还明确量化 KV 的支持边界：静态量化与 packed-layout scale 支持，动态量化不支持。

## Key Concepts / 关键概念
- **EN:** The document is a capability matrix, not a step-by-step usage guide.  
  **CN:** 该文档是能力矩阵，而不是逐步操作指南。
- **EN:** Encoder-decoder models are explicitly marked unsupported with NixlConnector PD.  
  **CN:** 在 NixlConnector 的 PD 场景下，encoder-decoder 模型被明确标记为不支持。
- **EN:** Compatibility hash checks protect handshake safety across P and D instances.  
  **CN:** 兼容性哈希检查用于保障 P / D 实例握手时的安全一致性。
- **EN:** Quantized KV support is selective: static yes, dynamic no.  
  **CN:** 量化 KV 只部分支持：静态支持，动态不支持。
