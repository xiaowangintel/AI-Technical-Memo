# index.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/performance/index.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This section covers the main performance levers for SGLang Diffusion: attention backends, caching acceleration, and profiling. Overview Start Here Use Attention Backends to choose the best backend for your model and hardware. / 该文档围绕 性能 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This section covers the main performance levers for SGLang Diffusion: attention backends, caching acceleration, and profiling.
**CN:** 本节围绕 Overview 展开，概述了 main, covers, levers, caching 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** This section provides a comparison table for Overview, covering columns such as Optimization, Type, Description and examples such as **Cache-DiT**, **TeaCache**, **Attention Backends**, **Profiling**.
**CN:** 本节围绕 Overview 展开，概述了 Caching, SCM, Type, Kernel 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Start Here
**EN:** Use Attention Backends to choose the best backend for your model and hardware. Use Deployment Cookbook to choose CPU offload, FSDP, CFG parallelism, SP, and TP.
**CN:** 本节围绕 Start Here 展开，概述了 CPU, CFG, FSDP, choose 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Caching at a Glance
**EN:** Cache-DiT is block-level caching for diffusers pipelines and higher speedup-oriented tuning. TeaCache is timestep-level caching built into SGLang model families.
**CN:** 本节围绕 Caching at a Glance 展开，概述了 caching, TeaCache, Cache-DiT, built 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Current Baseline Snapshot
**EN:** For Ring SP benchmark details, see: - Ring SP Performance
**CN:** 本节围绕 Current Baseline Snapshot 展开，概述了 Ring, Performance, see, details 等要点，并说明相关配置、流程、示例或限制条件。

### Section: References
**EN:** Cache-DiT Repository - TeaCache Paper
**CN:** 本节围绕 References 展开，概述了 Paper, TeaCache, Cache-DiT, Repository 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Caching / **CN:** Caching
- **EN:** TeaCache / **CN:** Tea缓存
- **EN:** Cache-DiT / **CN:** 缓存-DiT
- **EN:** Profiling / **CN:** Profiling
- **EN:** Ring / **CN:** Ring
- **EN:** Performance / **CN:** 性能
- **EN:** SCM / **CN:** SCM
- **EN:** CPU / **CN:** CPU

## Dependencies / 依赖关系
- `attention_backends.md`
- `deployment_cookbook.md`
- `cache/index.md`
- `profiling.md`
- `cache/cache_dit.md`
- `cache/teacache.md`
- `ring_sp_performance.md`
