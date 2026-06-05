# environment_variables.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/references/environment_variables.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang supports various environment variables that can be used to configure its runtime behavior. This document provides a comprehensive list and aims to stay updated over time. / 该文档围绕 Environment Variables 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang supports various environment variables that can be used to configure its runtime behavior. This document provides a comprehensive list and aims to stay updated over time.
**CN:** 本节围绕 Overview 展开，概述了 SGL_, SGLANG_, Note, While 等要点，并说明相关配置、流程、示例或限制条件。

### Section: General Configuration
**EN:** This section provides a comparison table for General Configuration, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_USE_MODELSCOPE, SGLANG_HOST_IP, SGLANG_PORT, SGLANG_LOGGING_CONFIG_PATH.
**CN:** 本节围绕 General 配置 展开，概述了 Timeout, seconds, SGLANG_PORT, SGLANG_HOST_IP 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance Tuning
**EN:** This section provides a comparison table for Performance Tuning, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_ENABLE_TORCH_INFERENCE_MODE, SGLANG_ENABLE_TORCH_COMPILE, SGLANG_SET_CPU_AFFINITY, SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN.
**CN:** 本节围绕 性能 Tuning 展开，概述了  |
| , GPU, Enable, --scheduler-recv-interval 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepGEMM Configuration (Advanced Optimization)
**EN:** This section provides a comparison table for DeepGEMM Configuration (Advanced Optimization), covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_ENABLE_JIT_DEEPGEMM, SGLANG_JIT_DEEPGEMM_PRECOMPILE, SGLANG_JIT_DEEPGEMM_COMPILE_WORKERS, SGLANG_IN_DEEPGEMM_PRECOMPILE_STAGE.
**CN:** 本节围绕 DeepGEMM 配置 (Advanced Optimization) 展开，概述了 DeepGEMM, "false", kernels, Enable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepEP Configuration
**EN:** This section provides a comparison table for DeepEP Configuration, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK, SGLANG_FLASHINFER_NUM_MAX_DISPATCH_TOKENS_PER_RANK, SGLANG_DEEPEP_LL_COMBINE_SEND_NUM_SMS, SGLANG_BLACKWELL_OVERLAP_SHARED_EXPERTS_OUTSIDE_SBO.
**CN:** 本节围绕 DeepEP 配置 展开，概述了 GPU, DeepEP, SGLANG_DEEPEP_LL_COMBINE_SEND_NUM_SMS, SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK 等要点，并说明相关配置、流程、示例或限制条件。

### Section: MORI Configuration
**EN:** This section provides a comparison table for MORI Configuration, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_MORI_DISPATCH_DTYPE, SGLANG_MORI_FP8_COMB, SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK, SGLANG_MORI_DISPATCH_INTER_KERNEL_SWITCH_THRESHOLD.
**CN:** 本节围绕 MORI 配置 展开，概述了 SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK, Number, InterNodeV1, InterNodeV1LL 等要点，并说明相关配置、流程、示例或限制条件。

### Section: NSA Backend Configuration (For DeepSeek V3.2)
**EN:** This section provides a comparison table for NSA Backend Configuration (For DeepSeek V3.2), covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_NSA_FUSE_TOPK, SGLANG_NSA_ENABLE_MTP_PRECOMPUTE_METADATA, SGLANG_USE_FUSED_METADATA_COPY, SGLANG_NSA_PREFILL_DENSE_ATTN_KV_LEN_THRESHOLD.
**CN:** 本节围绕 NSA Backend 配置 (For DeepSeek V3.2) 展开，概述了 topk, metadata, Environment, SGLANG_NSA_FUSE_TOPK 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory Management
**EN:** This section provides a comparison table for Memory Management, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_DEBUG_MEMORY_POOL, SGLANG_CLIP_MAX_NEW_TOKENS_ESTIMATION, SGLANG_DETOKENIZER_MAX_STATES, SGLANG_ENABLE_TP_MEMORY_INBALANCE_CHECK.
**CN:** 本节围绕 Memory Management 展开，概述了 NVLINK, Enable, memory, BAREX 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Model-Specific Options
**EN:** This section provides a comparison table for Model-Specific Options, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_USE_AITER, SGLANG_ROCM_USE_MULTI_STREAM, SGLANG_MOE_PADDING, SGLANG_CUTLASS_MOE (deprecated).
**CN:** 本节围绕 模型-Specific Options 展开，概述了 HIP, MoE, stream, SGLANG_USE_AITER 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quantization
**EN:** This section provides a comparison table for Quantization, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_INT4_WEIGHT, SGLANG_FORCE_FP8_MARLIN, SGLANG_NVFP4_CKPT_FP8_GEMM_IN_ATTN, SGLANG_MOE_NVFP4_DISPATCH.
**CN:** 本节围绕 量化 展开，概述了 FP8, BF16, Quantize, moe 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Distributed Computing
**EN:** This section provides a comparison table for Distributed Computing, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_BLOCK_NONZERO_RANK_CHILDREN, SGLANG_IS_FIRST_RANK_ON_NODE, SGLANG_PP_LAYER_PARTITION, SGLANG_ONE_VISIBLE_DEVICE_PER_PROCESS.
**CN:** 本节围绕 Distributed Computing 展开，概述了 SGLANG_PP_LAYER_PARTITION, SGLANG_BLOCK_NONZERO_RANK_CHILDREN, SGLANG_ONE_VISIBLE_DEVICE_PER_PROCESS, rank 等要点，并说明相关配置、流程、示例或限制条件。

### Section: PD Disaggregation — Staging Buffer (Heterogeneous TP)
**EN:** This section provides a comparison table for PD Disaggregation — Staging Buffer (Heterogeneous TP), covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_DISAGG_STAGING_BUFFER, SGLANG_DISAGG_STAGING_BUFFER_SIZE_MB, SGLANG_DISAGG_STAGING_POOL_SIZE_MB, SGLANG_STAGING_USE_TORCH.
**CN:** 本节围绕 PD Disaggregation — Staging Buffer (Heterogeneous TP) 展开，概述了 RDMA, buffer, staging, SGLANG_STAGING_USE_TORCH 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Testing & Debugging (Internal/CI)
**EN:** This section provides a comparison table for Testing & Debugging (Internal/CI), covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_IS_IN_CI, SGLANG_IS_IN_CI_AMD, SGLANG_TEST_RETRACT, SGLANG_TEST_RETRACT_NO_PREFILL_BS.
**CN:** 本节围绕 Testing & Debugging (Internal/CI) 展开，概述了 API, kernel, SGLANG_TEST_RETRACT_NO_PREFILL_BS, PID 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profiling & Benchmarking
**EN:** This section provides a comparison table for Profiling & Benchmarking, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_TORCH_PROFILER_DIR, SGLANG_PROFILE_WITH_STACK, SGLANG_PROFILE_RECORD_SHAPES, SGLANG_OTLP_EXPORTER_SCHEDULE_DELAY_MILLIS.
**CN:** 本节围绕 Profiling & Benchmarking 展开，概述了 PyTorch, profiler, SGLANG_TORCH_PROFILER_DIR, SGLANG_PROFILE_WITH_STACK 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Storage & Caching
**EN:** This section provides a comparison table for Storage & Caching, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_WAIT_WEIGHTS_READY_TIMEOUT, SGLANG_DISABLE_OUTLINES_DISK_CACHE, SGLANG_USE_CUSTOM_TRITON_KERNEL_CACHE, SGLANG_HICACHE_DECODE_OFFLOAD_STRIDE.
**CN:** 本节围绕 Storage & Caching 展开，概述了 --page-size, page-size, cache, SGLANG_WAIT_WEIGHTS_READY_TIMEOUT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Function Calling / Tool Use
**EN:** This section provides a comparison table for Function Calling / Tool Use, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_TOOL_STRICT_LEVEL.
**CN:** 本节围绕 Function Calling / Tool Use 展开，概述了 Level, strict, validation, SGLANG_TOOL_STRICT_LEVEL 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Description / **CN:** Description
- **EN:** Enable / **CN:** Enable
- **EN:**  |
|  / **CN:**  |
| 
- **EN:** Environment / **CN:** Environment
- **EN:** GPU / **CN:** GPU
- **EN:** Value / **CN:** Value
- **EN:** Variable / **CN:** Variable
- **EN:** Default Value / **CN:** Default Value

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
