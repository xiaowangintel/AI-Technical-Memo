# expert_parallelism.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/expert_parallelism.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Expert Parallelism (EP) in SGLang distributes expert weights across multiple devices in Mixture-of-Experts (MoE) models, addressing memory bottlenecks and enabling efficient scaling for high-performance inference. / 该文档围绕 Expert 并行策略 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Expert Parallelism (EP) in SGLang distributes expert weights across multiple devices in Mixture-of-Experts (MoE) models, addressing memory bottlenecks and enabling efficient scaling for high-performance inference.
**CN:** 本节围绕 Overview 展开，概述了 MoE, GPU, GPUs, GEMMs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Backends and Selection Guidance
**EN:** SGLang's EP integrates diverse, highly efficient backends for different use cases, allowing fine-grained control over performance trade-offs. Users specify backends via command-line flags: - --moe-a2a-backend: Selects the backend for all-to-all communication.
**CN:** 本节围绕 Supported Backends and Selection Guidance 展开，概述了 Selects, MoE, Users, backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Backends for All-to-All Communication
**EN:** This section provides a comparison table for Backends for All-to-All Communication, covering columns such as Backend, Description, Use Cases and examples such as **none (default)**, deepep, mooncake, nixl.
**CN:** 本节围绕 Backends for All-to-All Communication 展开，概述了 DeepEP, NIXL-EP, mode, normal 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Backends for MoE Computation
**EN:** This section provides a comparison table for Backends for MoE Computation, covering columns such as Backend, Description, Use Cases and examples such as **auto (default)**, triton, deep_gemm, cutlass.
**CN:** 本节围绕 Backends for MoE Computation 展开，概述了 MoE, FlashInfer, FP4, GEMMs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Examples
**EN:** Launch with DeepEP and DeepGEMM for DeepSeek-V3: ``bash python -m sglang.launch_server --model-path deepseek-ai/DeepSeek-V3 --moe-a2a-backend deepep --moe-runner-backend deep_gemm --tp 8 --ep 8 ``
**CN:** 本节围绕 Examples 展开，概述了 DeepSeek-V3, Launch, DeepEP, DeepGEMM 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Extensible EP Framework
**EN:** SGLang's EP framework provides modular abstractions for easy integration of custom kernels, backends, and optimizations.
**CN:** 本节围绕 Extensible EP Framework 展开，概述了 MoE, core, easy, pass 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Framework Overview
**EN:** This section provides a comparison table for Framework Overview, covering columns such as v and examples such as pre-permute + grouped_gemm + post-permute.
**CN:** 本节围绕 Framework Overview 展开，概述了 DeepEP, via, bypass, FusedMoE 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Implementing New Backends
**EN:** To add a new backend: 1. For a new all-to-all dispatcher, implement a BaseDispatcher subclass with dispatch and combine methods.
**CN:** 本节围绕 Implementing New Backends 展开，概述了 new, Register, MoE, RunnerInput 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Examples
**EN:** For an example implementation, see moe_runner/triton.py, which demonstrates Triton-based grouped GEMMs with registered fused and permutation functions.
**CN:** 本节围绕 Examples 展开，概述了 GEMMs, Triton-based, see, fused 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Computation and Communication Overlap
**EN:** SGLang's EP employs advanced overlap techniques to hide communication latency behind computation, maximizing GPU utilization in MoE layers.
**CN:** 本节围绕 Computation and Communication Overlap 展开，概述了 GPU, MoE, hide, behind 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Two-Batch Overlap (TBO)
**EN:** TBO splits requests into micro-batches, interleaving attention computation with dispatch/combine operations. For details, see the Large-Scale EP Blog.
**CN:** 本节围绕 Two-Batch Overlap (TBO) 展开，概述了 Overlap, YieldOperation, TBO, Blog 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Single-Batch Overlap (SBO)
**EN:** SGLang introduces a dispatcher-hook system for Single-Batch Overlap (SBO), enabling the overlap of operations within a single batch—such as shared experts computation with communication—while decentralizing logic to enhance modularity.
**CN:** 本节围绕 Single-Batch Overlap (SBO) 展开，概述了 combine, SBO, MoE, These 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Workload Balancer
**EN:** SGLang integrates the Expert Parallelism Load Balancer (EPLB) from DeepSeek to address routing imbalances in MoE models.
**CN:** 本节围绕 Workload Balancer 展开，概述了 EPLB, MoE, GPU, Blog 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EP with Spectulative Decoding
**EN:** When utilizing speculative decoding with MTP on MoE architectures, use the --speculative-moe-runner-backend and --speculative-moe-a2a-backend arguments to customize the MoE layer behavior for the draft model.
**CN:** 本节围绕 EP with Spectulative Decoding 展开，概述了 MoE, model, draft, target 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Guidance on SGLang configuration in Ascend NPU
**EN:** moe-a2a-backend only supports deepep and ascend_fuseep backends, - deepep: The mechanism is consistent with the above description. ascend_fuseep: Offer a large fused operator which integrates all operations between dispatch and combine to boost MoE computation.
**CN:** 本节围绕 Guidance on SGLang 配置 in Ascend NPU 展开，概述了 --deepep-mode, deepep, deepep-mode, ascend_fuseep 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepEP Ascend Introduction
**EN:** DeepEP Ascend is the adapted version of the DeepEP communication library for Huawei Ascend NPUs, specifically designed for Mixture-of-Experts (MoE) model Expert Parallelism (EP).
**CN:** 本节围绕 DeepEP Ascend Introduction 展开，概述了 Enable, DEEPEP_NORMAL_LONG_SEQ_PER_ROUND_TOKENS, hidden_size, PADDING_BUFFSIZE 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** MoE / **CN:** MoE
- **EN:** DeepEP / **CN:** DeepEP
- **EN:** GEMMs / **CN:** GEMMs
- **EN:** dispatch / **CN:** dispatch
- **EN:** combine / **CN:** combine
- **EN:** communication / **CN:** communication
- **EN:** FlashInfer / **CN:** FlashInfer
- **EN:** FP4 / **CN:** FP4

## Dependencies / 依赖关系
- `//github.com/sgl-project/sglang/blob/main/benchmark/kernels/fused_moe_triton/README.md`
- `moe_runner/triton.py`
- `//github.com/sgl-project/sglang/blob/main/python/sglang/srt/layers/moe/moe_runner/triton.py`
