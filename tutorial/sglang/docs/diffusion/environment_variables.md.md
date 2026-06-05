# environment_variables.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/environment_variables.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Runtime Platform-Specific Apple MPS ROCm (AMD GPUs). / 该文档围绕 Environment Variables 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Runtime
**EN:** This section provides a comparison table for Runtime, covering columns such as Environment Variable, Default, Description and examples such as SGLANG_DIFFUSION_TARGET_DEVICE, SGLANG_DIFFUSION_ATTENTION_BACKEND, SGLANG_DIFFUSION_ATTENTION_CONFIG, SGLANG_DIFFUSION_STAGE_LOGGING.
**CN:** 本节围绕 Runtime 展开，概述了 cuda, fork, Root, Enable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Apple MPS
**EN:** This section provides a comparison table for Apple MPS, covering columns such as Environment Variable, Default, Description and examples such as SGLANG_USE_MLX.
**CN:** 本节围绕 Apple MPS 展开，概述了 SGLANG_USE_MLX, MLX, MPS, Metal 等要点，并说明相关配置、流程、示例或限制条件。

### Section: ROCm (AMD GPUs)
**EN:** This section provides a comparison table for ROCm (AMD GPUs), covering columns such as Environment Variable, Default, Description and examples such as SGLANG_USE_ROCM_VAE, SGLANG_USE_ROCM_CUDNN_BENCHMARK.
**CN:** 本节围绕 ROCm (AMD GPUs) 展开，概述了 VAE, ROCm, SGLANG_USE_ROCM_VAE, SGLANG_USE_ROCM_CUDNN_BENCHMARK 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quantization
**EN:** This section provides a comparison table for Quantization, covering columns such as Environment Variable, Default, Description and examples such as SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND.
**CN:** 本节围绕 量化 展开，概述了 SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND, NVFP4, Description, FP4 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Caching Acceleration
**EN:** These variables configure caching acceleration for Diffusion Transformer (DiT) models. SGLang supports multiple caching strategies - see caching documentation for an overview.
**CN:** 本节围绕 Caching Acceleration 展开，概述了 caching, DiT, These, see 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Cache-DiT Configuration
**EN:** This section provides a comparison table for Cache-DiT Configuration, covering columns such as Environment Variable, Default, Description and examples such as SGLANG_CACHE_DIT_ENABLED, SGLANG_CACHE_DIT_FN, SGLANG_CACHE_DIT_BN, SGLANG_CACHE_DIT_WARMUP.
**CN:** 本节围绕 缓存-DiT 配置 展开，概述了 SCM, compute, TaylorSeer, SGLANG_CACHE_DIT_FN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Cache-DiT Secondary Transformer
**EN:** This section provides a comparison table for Cache-DiT Secondary Transformer, covering columns such as Environment Variable, Default, Description and examples such as SGLANG_CACHE_DIT_SECONDARY_FN, SGLANG_CACHE_DIT_SECONDARY_BN, SGLANG_CACHE_DIT_SECONDARY_WARMUP, SGLANG_CACHE_DIT_SECONDARY_RDT.
**CN:** 本节围绕 缓存-DiT Secondary Transformer 展开，概述了 primary, TaylorSeer, SGLANG_CACHE_DIT_SECONDARY_FN, SGLANG_CACHE_DIT_SECONDARY_BN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Cloud Storage
**EN:** This section provides a comparison table for Cloud Storage, covering columns such as Environment Variable, Default, Description and examples such as SGLANG_CLOUD_STORAGE_TYPE, SGLANG_S3_BUCKET_NAME, SGLANG_S3_ENDPOINT_URL, SGLANG_S3_REGION_NAME.
**CN:** 本节围绕 Cloud Storage 展开，概述了 AWS, SGLANG_S3_BUCKET_NAME, SGLANG_S3_REGION_NAME, SGLANG_S3_ENDPOINT_URL 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CUDA Crash Debugging
**EN:** This section provides a comparison table for CUDA Crash Debugging, covering columns such as Environment Variable, Default, Description and examples such as SGLANG_KERNEL_API_LOGLEVEL, SGLANG_KERNEL_API_LOGDEST, SGLANG_KERNEL_API_DUMP_DIR, SGLANG_KERNEL_API_DUMP_INCLUDE.
**CN:** 本节围绕 CUDA Crash Debugging 展开，概述了 API, kernel, PID, CUDA 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Description / **CN:** Description
- **EN:** API / **CN:** API
- **EN:** Environment / **CN:** Environment
- **EN:** Enable / **CN:** Enable
- **EN:** primary / **CN:** primary
- **EN:** Variable / **CN:** Variable
- **EN:** Environment Variable / **CN:** Environment Variable
- **EN:** kernel / **CN:** kernel

## Dependencies / 依赖关系
- `performance/cache/index.md`
- `performance/cache/cache_dit.md`
