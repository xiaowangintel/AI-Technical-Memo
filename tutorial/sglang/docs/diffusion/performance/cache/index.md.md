# index.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/performance/cache/index.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang provides two complementary caching strategies for Diffusion Transformer (DiT) models. Both reduce denoising cost by skipping redundant computation, but they operate at different levels. / 该文档围绕 Caching Acceleration 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang provides two complementary caching strategies for Diffusion Transformer (DiT) models. Both reduce denoising cost by skipping redundant computation, but they operate at different levels.
**CN:** 本节围绕 Overview 展开，概述了 DiT, Both, two, but 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** This section provides a comparison table for Overview, covering columns such as Strategy, Scope, Mechanism and examples such as **Cache-DiT**, **TeaCache**.
**CN:** 本节围绕 Overview 展开，概述了 Skip, Scope, Simple, Strategy 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Cache-DiT
**EN:** Cache-DiT provides block-level caching with advanced strategies like DBCache and TaylorSeer. It can achieve up to **1.69x speedup**.
**CN:** 本节围绕 缓存-DiT 展开，概述了 See, DBCache, Cache-DiT, TaylorSeer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Start
**EN:** ``bash SGLANG_CACHE_DIT_ENABLED=true \ sglang generate --model-path Qwen/Qwen-Image \ --prompt "A beautiful sunset over the mountains" ``
**CN:** 本节围绕 Quick Start 展开，概述了 SGLANG_CACHE_DIT_ENABLED, Qwen, prompt, sunset 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Key Features
**EN:** **DBCache**: Dynamic block-level caching based on residual differences - **TaylorSeer**: Taylor expansion-based calibration for optimized caching - **SCM**: Step-level computation masking for additional speedup
**CN:** 本节围绕 Key Features 展开，概述了 SCM, Taylor, DBCache, Dynamic 等要点，并说明相关配置、流程、示例或限制条件。

### Section: TeaCache
**EN:** TeaCache (Temporal similarity-based caching) accelerates diffusion inference by detecting when consecutive denoising steps are similar enough to skip computation entirely. See teacache.md for detailed documentation.
**CN:** 本节围绕 Tea缓存 展开，概述了 See, TeaCache, Temporal, teacache.md 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Overview
**EN:** Tracks L1 distance between modulated inputs across timesteps - When accumulated distance is below threshold, reuses cached residual - Supports CFG with separate positive/negative caches
**CN:** 本节围绕 Quick Overview 展开，概述了 Tracks, distance, CFG, below 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** Wan (wan2.1, wan2.2) - Hunyuan (HunyuanVideo) - Z-Image For Flux and Qwen models, TeaCache is automatically disabled when CFG is enabled. `` :maxdepth: 1 cache_dit teacache ``
**CN:** 本节围绕 Supported 模型s 展开，概述了 Wan, CFG, Qwen, Hunyuan 等要点，并说明相关配置、流程、示例或限制条件。

### Section: References
**EN:** Cache-DiT Repository - TeaCache Paper
**CN:** 本节围绕 References 展开，概述了 Paper, TeaCache, Cache-DiT, Repository 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** TeaCache / **CN:** Tea缓存
- **EN:** Cache-DiT / **CN:** 缓存-DiT
- **EN:** caching / **CN:** caching
- **EN:** See / **CN:** See
- **EN:** Skip / **CN:** Skip
- **EN:** DBCache / **CN:** DB缓存
- **EN:** TaylorSeer / **CN:** TaylorSeer
- **EN:** CFG / **CN:** CFG

## Dependencies / 依赖关系
- `cache_dit.md`
- `teacache.md`
