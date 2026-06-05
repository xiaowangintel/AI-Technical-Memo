# cuda_graph_for_multi_modal_encoder.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/cuda_graph_for_multi_modal_encoder.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Motivation In multimodal reasoning services, the visual encoder (ViT / Vision Transformer) typically has a few characteristic traits: Many layers, fragmented operators: Each layer includes LN, QKV projections, attention, MLP, residual connections, etc., resulting in extremely frequent kernel launches. / 该文档围绕 Cuda Graph for Multi-Modal Encoder in SGLang 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Motivation
**EN:** In multimodal reasoning services, the visual encoder (ViT / Vision Transformer) typically has a few characteristic traits: Many layers, fragmented operators: Each layer includes LN, QKV projections, attention, MLP, residual connections, etc., resulting in extremely frequent kernel launches.
**CN:** 本节围绕 Motivation 展开，概述了 ViT, GPU, batch, CUDA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Design and Restrictions
**EN:** The new CUDA Graph enabled ViT logic is built on ViTCudaGraphRunner. This runner captures the "blocks + merger + deepstack merger (optional)" part of a vision transformer into a CUDA graph and replays it for identical shapes.
**CN:** 本节围绕 Design and Restrictions 展开，概述了 CUDA, ViT, See, merger 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Dynamic inputs to fit static constraints of CUDA Graph
**EN:** Variable sequence length S is very common in ViT. While CUDA Graph requires fixed shapes.
**CN:** 本节围绕 Dynamic inputs to fit static constraints of CUDA Graph 展开，概述了 VRAM, many, graph, Variable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Stable addresses
**EN:** Everything "parameter-like" becomes a static buffer: - block_input / block_ws / block_output - cu_full_len / cu_window_len and their kk variants - sin_cos_ws In this way to solve the underlying requirement: during replay, not allowed to swap tensors, can only modify tensor contents.
**CN:** 本节围绕 Stable addresses 展开，概述了 Everything, way, swap, solve 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Attention backend arguments
**EN:** Attention backend arguments are fixed inside the graph: TritonAttn expects [cu_seqlens, cu_seqlens_kk, max_len] FA3 expects [cu_seqlens, max_len] max_len is frozen as an int constant.
**CN:** 本节围绕 注意力后端 arguments 展开，概述了 cu_seqlens, max_len, FA3, during 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Rotary buffer management
**EN:** The feature reallocates a larger sin_cos_ws when seq_len increases. The max_content_len is used to make sure the maximum size of the allocated rotary buffer.
**CN:** 本节围绕 Rotary buffer management 展开，概述了 make, sure, size, larger 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Command Example
**EN:** This content focuses on Command Example and highlights SGLANG_VIT_ENABLE_CUDA_GRAPH, ViT, CUDA, Graph.
**CN:** 本节围绕 Command Example 展开，概述了 SGLANG_VIT_ENABLE_CUDA_GRAPH, ViT, CUDA, Graph 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Known supported models
**EN:** Qwen2.5-VL (https://github.com/sgl-project/sglang/pull/14422) - Qwen3-VL (https://github.com/sgl-project/sglang/pull/15320)
**CN:** 本节围绕 Known supported 模型s 展开，概述了 Qwen3-VL, Qwen2.5-VL 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** ViT / **CN:** ViT
- **EN:** CUDA / **CN:** CUDA
- **EN:** Graph / **CN:** Graph
- **EN:** SGLANG_VIT_ENABLE_CUDA_GRAPH / **CN:** SGLANG_VIT_ENABLE_CUDA_GRAPH
- **EN:** CUDA Graph / **CN:** CUDA Graph
- **EN:** shapes / **CN:** shapes
- **EN:** GPU / **CN:** GPU
- **EN:** batch / **CN:** batch

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
