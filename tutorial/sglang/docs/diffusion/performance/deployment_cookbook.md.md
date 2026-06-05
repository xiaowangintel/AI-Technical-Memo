# deployment_cookbook.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/performance/deployment_cookbook.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This page gives practical defaults for choosing CPU offload, FSDP, CFG parallelism, SP, and TP. Quick Rule Use the simplest setting that fits your memory target: Base the decision on available memory on the selected GPU(s). / 该文档围绕 部署 Cookbook 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This page gives practical defaults for choosing CPU offload, FSDP, CFG parallelism, SP, and TP.
**CN:** 本节围绕 Overview 展开，概述了 CPU, CFG, FSDP, gives 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Rule
**EN:** This section provides a comparison table for Quick Rule, covering columns such as Goal, Recommended setting and examples such as Fastest single-GPU run when the model fits, Lower single-GPU memory usage, Faster multi-GPU Qwen/Wan CFG generation, Sequence length or video-shape scaling.
**CN:** 本节围绕 Quick Rule 展开，概述了 GPU, CPU, FSDP, offload 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance Modes
**EN:** This section provides a comparison table for Performance Modes, covering columns such as Mode, Meaning and examples such as manual, auto, speed, memory.
**CN:** 本节围绕 性能 Modes 展开，概述了 auto, offload, DiT, FSDP 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Interpreting The Levers
**EN:** **No offload** keeps model components resident on GPU. It is usually fastest when memory is sufficient.
**CN:** 本节围绕 Interpreting The Levers 展开，概述了 GPU, DiT, memory, CPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Current Benchmark Takeaways
**EN:** Observed regular-scale trends: - Z-Image: single-GPU no-offload was faster than FSDP/SP in the tested setting; keep FSDP off unless memory or parallelism requires it.
**CN:** 本节围绕 Current Benchmark Takeaways 展开，概述了 FSDP, GPU, offload, Wan 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** GPU / **CN:** GPU
- **EN:** FSDP / **CN:** FSDP
- **EN:** DiT / **CN:** DiT
- **EN:** offload / **CN:** offload
- **EN:** memory / **CN:** memory
- **EN:** CPU / **CN:** CPU
- **EN:** CFG / **CN:** CFG
- **EN:** auto / **CN:** auto

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
