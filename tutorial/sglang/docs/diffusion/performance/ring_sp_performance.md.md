# ring_sp_performance.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/performance/ring_sp_performance.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This page reports Ring-SP performance for Wan2.2-TI2V-5B-Diffusers using: Parallel config: sp=2, ulysses=1, ring=2 (short: u1r2) Baseline config: sp=1, ulysses=1, ring=1 (short: u1r1) Benchmark Setup. / 该文档围绕 Ring SP Benchmark: Wan2.2-TI2V-5B (u1r2 vs Baseline) 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This page reports Ring-SP performance for Wan2.2-TI2V-5B-Diffusers using: - Parallel config: sp=2, ulysses=1, ring=2 (short: u1r2) - Baseline config: sp=1, ulysses=1, ring=1 (short: u1r1)
**CN:** 本节围绕 Overview 展开，概述了 Wan2.2-TI2V-5B-Diffusers, u1r2, u1r1, ring 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmark Setup
**EN:** Model: Wan2.2-TI2V-5B-Diffusers - GPU: 48G RTX40 series * 2
**CN:** 本节围绕 Benchmark Setup 展开，概述了 Wan2.2-TI2V-5B-Diffusers, GPU, Model, RTX40 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Ring SP (u1r2)
**EN:** ``bash sglang serve \ --model-type diffusion \ --model-path /model/HuggingFace/Wan-AI/Wan2.2-TI2V-5B-Diffusers \ --num-gpus 2 --sp-degree 2 --ulysses-degree 1 --ring-degree 2 \ --port 8898 ``
**CN:** 本节围绕 Ring SP (u1r2) 展开，概述了 port, serve, --port, Wan-AI 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Baseline (u1r1)
**EN:** ``bash sglang serve \ --model-type diffusion \ --model-path /model/HuggingFace/Wan-AI/Wan2.2-TI2V-5B-Diffusers \ --num-gpus 1 --sp-degree 1 --ulysses-degree 1 --ring-degree 1 \ --port 8898 ``
**CN:** 本节围绕 Baseline (u1r1) 展开，概述了 port, serve, --port, Wan-AI 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmark Disclaimer
**EN:** These benchmarks are provided for reference under one specific setup and command configuration. Actual performance may vary with model settings, runtime environment, and request patterns.
**CN:** 本节围绕 Benchmark Disclaimer 展开，概述了 These, Actual, one, may 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Stage Time Breakdown
**EN:** This section provides a comparison table for Stage Time Breakdown, covering columns such as Stage / Metric, u1r2 (s), u1r1 baseline (s) and examples such as InputValidation, TextEncoding, LatentPreparation, TimestepPreparation.
**CN:** 本节围绕 Stage Time Breakdown 展开，概述了 u1r2, u1r1, Stage, Total 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory Usage
**EN:** This section provides a comparison table for Memory Usage, covering columns such as Memory Metric, u1r2 (GB), u1r1 baseline (GB) and examples such as Peak GPU Memory, Peak Allocated, Memory Overhead, Overhead Ratio.
**CN:** 本节围绕 Memory Usage 展开，概述了 Memory, u1r2, u1r1, Peak 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Summary
**EN:** End-to-end latency improves from 90.63s to 63.74s (1.42x). Main gains come from Denoising (1.36x) and Decoding (1.75x).
**CN:** 本节围绕 Summary 展开，概述了 Decoding, Denoising, Main, Peak 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** u1r2 / **CN:** u1r2
- **EN:** Wan2.2-TI2V-5B-Diffusers / **CN:** Wan2.2-TI2V-5B-Diffusers
- **EN:** u1r1 / **CN:** u1r1
- **EN:** Baseline / **CN:** Baseline
- **EN:** Memory / **CN:** Memory
- **EN:** Decoding / **CN:** Decoding
- **EN:** Denoising / **CN:** Denoising
- **EN:** GPU / **CN:** GPU

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
