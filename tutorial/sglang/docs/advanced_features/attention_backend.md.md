# attention_backend.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/attention_backend.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang supports a large variety of attention backends. Each of them has different pros and cons. / 该文档围绕 注意力后端 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang supports a large variety of attention backends. Each of them has different pros and cons.
**CN:** 本节围绕 Overview 展开，概述了 model, backends, based, backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Support Matrix
**EN:** The support matrix is split into two parts: MHA (standard attention) and MLA (multi-head latent attention). For an explanation of the key differences between MHA and MLA, please see the SGLang documentation on DeepSeek MLA and the original DeepSeek MLA paper.
**CN:** 本节围绕 支持矩阵 展开，概述了 MLA, MHA, DeepSeek, attention 等要点，并说明相关配置、流程、示例或限制条件。

### Section: MHA Backends
**EN:** This section provides a comparison table for MHA Backends, covering columns such as **Backend**, **Page Size > 1 (native)**, **FP8 KV Cache** and examples such as **FlashInfer**, **FA3 (FlashAttention 3)**, **FA4 (FlashAttention 4)**, **Triton**.
**CN:** 本节围绕 MHA Backends 展开，概述了 FlashAttention, Spec, ROCm, Cache 等要点，并说明相关配置、流程、示例或限制条件。

### Section: MLA Backends
**EN:** This section provides a comparison table for MLA Backends, covering columns such as **Backend**, **Native Page Sizes**, **FP8 KV Cache** and examples such as **FlashInfer MLA**, **FlashMLA**, **Cutlass MLA**, **TRTLLM MLA (Blackwell)**.
**CN:** 本节围绕 MLA Backends 展开，概述了 FA4, MLA, page_size, Spec 等要点，并说明相关配置、流程、示例或限制条件。

### Section: GDN Attention Backends
**EN:** This section provides a comparison table for GDN Attention Backends, covering columns such as **Backend**, **Decode**, **Prefill / Extend** and examples such as **Triton (CUDA)**, **Triton (AMD/ROCm)**, **Triton (NPU)**, **Triton (CPU)**.
**CN:** 本节围绕 GDN 注意力后端s 展开，概述了 GDN, Triton, CUDA, NPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DSA Attention Backend (NSA)
**EN:** This section provides a comparison table for DSA Attention Backend (NSA), covering columns such as **Sub-backend**, **Prefill**, **Decode** and examples such as **flashmla_sparse**, **flashmla_kv**, **flashmla_auto**, **fa3**.
**CN:** 本节围绕 DSA 注意力后端 (NSA) 展开，概述了 Blackwell, Hopper, decode, bf16 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hybrid attention (different backends for prefill vs decode) (Experimental)
**EN:** `` Hybrid attention is an experimental feature. ` You can mix-and-match attention backends for prefill and decode.
**CN:** 本节围绕 Hybrid attention (different backends for prefill 阶段 vs decode 阶段) (Experimental) 展开，概述了 backend, prefill, attention, decode 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Attention Backend Selection Guide (CUDA)
**EN:** If the --attention-backend argument is not specified, SGLang automatically selects the best backend based on the hardware (CUDA) and model architecture.
**CN:** 本节围绕 注意力后端 Selection Guide (CUDA) 展开，概述了 CUDA, --attention-backend, best, based 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Automatic Selection Logic
**EN:** MHA Models (e.g., Llama, Qwen)** - **Hopper (e.g., H100, H200)**: Defaults to fa3 if using CUDA 12.3+ and the model configuration is supported. **Blackwell (e.g., B200)**: Defaults to trtllm_mha, unless using speculative decoding with topk > 1.
**CN:** 本节围绕 Automatic Selection Logic 展开，概述了 Defaults, fa3, e.g, CUDA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Command for Different Attention Backends
**EN:** This content focuses on Launch Command for Different Attention Backends and highlights python3, sglang.launch_server, model, --model.
**CN:** 本节围绕 Launch Command for Different 注意力后端s 展开，概述了 python3, sglang.launch_server, model, --model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Steps to add a new attention backend
**EN:** To add a new attention backend, you can learn from the existing backends (python/sglang/srt/layers/attention/triton_backend.py, python/sglang/srt/layers/attention/flashattention_backend.py) and follow the steps below.
**CN:** 本节围绕 Steps to add a new 注意力后端 展开，概述了 called, Run, cuda, once 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** --attention-backend / **CN:** --attention-backend
- **EN:** model / **CN:** 模型
- **EN:** MLA / **CN:** MLA
- **EN:** attention-backend / **CN:** attention-backend
- **EN:** Blackwell / **CN:** Blackwell
- **EN:** python3 / **CN:** python3
- **EN:** attention / **CN:** attention
- **EN:** sglang.launch_server / **CN:** sglang.launch_服务端

## Dependencies / 依赖关系
- `../basic_usage/deepseek_v3.md`
- `../basic_usage/deepseek_v32.md`
- `python/sglang/srt/layers/attention/hybrid_attn_backend.py`
- `python/sglang/srt/layers/attention/triton_backend.py`
- `python/sglang/srt/layers/attention/flashattention_backend.py`
