# profiling.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/performance/profiling.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This guide covers profiling techniques for multimodal generation pipelines in SGLang. PyTorch Profiler PyTorch Profiler provides detailed kernel execution time, call stack, and GPU utilization metrics. / 该文档围绕 Profiling Multimodal Generation 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This guide covers profiling techniques for multimodal generation pipelines in SGLang.
**CN:** 本节围绕 Overview 展开，概述了 covers, profiling, pipelines, techniques 等要点，并说明相关配置、流程、示例或限制条件。

### Section: PyTorch Profiler
**EN:** PyTorch Profiler provides detailed kernel execution time, call stack, and GPU utilization metrics.
**CN:** 本节围绕 PyTorch Profiler 展开，概述了 GPU, time, call, stack 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Denoising Stage Profiling
**EN:** This content focuses on Denoising Stage Profiling and highlights after, warmup, profile, step.
**CN:** 本节围绕 Denoising Stage Profiling 展开，概述了 after, warmup, profile, step 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Full Pipeline Profiling
**EN:** This content focuses on Full Pipeline Profiling and highlights profile, VAE, stages, pipeline.
**CN:** 本节围绕 Full Pipeline Profiling 展开，概述了 profile, VAE, stages, pipeline 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Output Location
**EN:** By default, trace files are saved in the ./logs/ directory.
**CN:** 本节围绕 Output Location 展开，概述了 Saved, output, path, trace 等要点，并说明相关配置、流程、示例或限制条件。

### Section: View Traces
**EN:** Load and visualize trace files at: - https://ui.perfetto.dev/ (recommended) - chrome://tracing (Chrome only) For large trace files, reduce --num-profiled-timesteps or avoid using --profile-all-stages.
**CN:** 本节围绕 View Traces 展开，概述了 Load, trace, files, Chrome 等要点，并说明相关配置、流程、示例或限制条件。

### Section: --perf-dump-path (Stage/Step Timing Dump)
**EN:** The dumped JSON contains a denoise_steps_ms field formatted as an array of objects, each with a step key (the step index) and a duration_ms key.
**CN:** 本节围绕 --perf-dump-path (Stage/Step Timing Dump) 展开，概述了 step, key, stage, PROMPT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Nsight Systems
**EN:** Nsight Systems provides low-level CUDA profiling with kernel details, register usage, and memory access patterns.
**CN:** 本节围绕 Nsight Systems 展开，概述了 CUDA, usage, Nsight, kernel 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Installation
**EN:** See the SGLang profiling guide for installation instructions.
**CN:** 本节围绕 安装 展开，概述了 See, profiling, installation, instructions 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Basic Profiling
**EN:** Profile the entire pipeline execution: ``bash nsys profile \ --trace-fork-before-exec=true \ --cuda-graph-trace=node \ --force-overwrite=true \ -o QwenImage \ sglang generate \ --model-path Qwen/Qwen-Image \ --prompt "A Logo With Bold Large Text: SGL Diffusion" \ --seed 0 ``
**CN:** 本节围绕 Basic Profiling 展开，概述了 Profile, QwenImage, SGL, Qwen 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Targeted Stage Profiling
**EN:** This content focuses on Targeted Stage Profiling and highlights --delay, --duration, delay, duration.
**CN:** 本节围绕 Targeted Stage Profiling 展开，概述了 --delay, --duration, delay, duration 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Notes
**EN:** This content focuses on Notes and highlights Reduce, --delay, Profile, Multiple.
**CN:** 本节围绕 Notes 展开，概述了 Reduce, --delay, Profile, Multiple 等要点，并说明相关配置、流程、示例或限制条件。

### Section: FAQ
**EN:** If you are profiling sglang generate with Nsight Systems and find that the generated profiler file did not capture any CUDA kernels, you can resolve this issue by increasing the model's inference steps to extend the execution time.
**CN:** 本节围绕 FAQ 展开，概述了 CUDA, did, find, time 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** profile / **CN:** profile
- **EN:** step / **CN:** step
- **EN:** denoising / **CN:** denoising
- **EN:** generate / **CN:** generate
- **EN:** --profile / **CN:** --profile
- **EN:** Parameters / **CN:** Parameters
- **EN:** --profile-all-stages / **CN:** --profile-all-stages
- **EN:** --num-profiled-timesteps / **CN:** --num-profiled-timesteps

## Dependencies / 依赖关系
- `//github.com/sgl-project/sglang/blob/main/docs/developer_guide/benchmark_and_profiling.md`
