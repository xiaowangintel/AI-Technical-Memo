# teacache.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/performance/cache/teacache.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: **Note**: This is one of two caching strategies available in SGLang. For an overview of all caching options, see caching. / 该文档围绕 Tea缓存 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** > **Note**: This is one of two caching strategies available in SGLang. > For an overview of all caching options, see caching.
**CN:** 本节围绕 Overview 展开，概述了 caching, Note, TeaCache, Temporal 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** TeaCache works by: 1. Tracking the L1 distance between modulated inputs across consecutive timesteps 2.
**CN:** 本节围绕 Overview 展开，概述了 distance, TeaCache, Tracking, Accumulating 等要点，并说明相关配置、流程、示例或限制条件。

### Section: L1 Distance Tracking
**EN:** This content focuses on L1 Distance Tracking and highlights previous, mean, rel_l1, current.
**CN:** 本节围绕 L1 Distance Tracking 展开，概述了 previous, mean, rel_l1, current 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Cache Decision
**EN:** If accumulated >= threshold: Force computation, reset accumulator - If accumulated < threshold: Skip computation, use cached residual
**CN:** 本节围绕 缓存 Decision 展开，概述了 Skip, Force, threshold, accumulated 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CFG Support
**EN:** This content focuses on CFG Support and highlights CFG, TeaCache, Wan, Flux.
**CN:** 本节围绕 CFG Support 展开，概述了 CFG, TeaCache, Wan, Flux 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration
**EN:** TeaCache is configured via TeaCacheParams in the sampling parameters: ``python from sglang.multimodal_gen.configs.sample.teacache import TeaCacheParams params = TeaCacheParams( teacache_thresh=0.1, # Threshold for accumulated L1 distance coefficients=[1.0, 0.0, 0.0], # Polynomial coefficients for L1 rescaling ) ``
**CN:** 本节围绕 配置 展开，概述了 TeaCacheParams, TeaCache, Threshold, Polynomial 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Parameters
**EN:** This section provides a comparison table for Parameters, covering columns such as Parameter, Type, Description and examples such as teacache_thresh, coefficients.
**CN:** 本节围绕 Parameters 展开，概述了 coefficients, Type, Lower, float 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Model-Specific Configurations
**EN:** Different models may have different optimal configurations. The coefficients are typically tuned per-model to balance speed and quality.
**CN:** 本节围绕 模型-Specific 配置s 展开，概述了 Different, may, tuned, speed 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** This section provides a comparison table for Supported Models, covering columns such as Model Family, CFG Cache Separation, Notes and examples such as Wan (wan2.1, wan2.2), Hunyuan (HunyuanVideo), Z-Image, Flux.
**CN:** 本节围绕 Supported 模型s 展开，概述了 Yes, supported, Wan, Full 等要点，并说明相关配置、流程、示例或限制条件。

### Section: References
**EN:** TeaCache: Accelerating Diffusion Models with Temporal Similarity
**CN:** 本节围绕 References 展开，概述了 TeaCache, Models, Temporal, Diffusion 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** TeaCache / **CN:** Tea缓存
- **EN:** CFG / **CN:** CFG
- **EN:** coefficients / **CN:** coefficients
- **EN:** distance / **CN:** distance
- **EN:** accumulated / **CN:** accumulated
- **EN:** TeaCacheParams / **CN:** Tea缓存Params
- **EN:** Yes / **CN:** Yes
- **EN:** caching / **CN:** caching

## Dependencies / 依赖关系
- `../index.md`
