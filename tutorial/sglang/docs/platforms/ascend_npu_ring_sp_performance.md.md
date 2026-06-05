# ascend_npu_ring_sp_performance.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/ascend_npu_ring_sp_performance.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This page reports Ring-SP performance on Ascend NPU with torch_npu==2.10.0. Baseline config: ulysses=1, ring=1 (short: u1r1) Ring-SP config: ulysses=1, ring=2 (short: u1r2) Benchmark Setup. / 该文档围绕 Ascend NPU Ring-SP 性能 (Wan2.1-T2V-1.3B) 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This page reports Ring-SP performance on Ascend NPU with torch_npu==2.10.0. Baseline config: ulysses=1, ring=1 (short: u1r1) - Ring-SP config: ulysses=1, ring=2 (short: u1r2)
**CN:** 本节围绕 Overview 展开，概述了 Ring-SP, u1r1, u1r2, ring 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmark Setup
**EN:** Model: Wan2.1-T2V-1.3B-Diffusers - Prompt: "a cat is playing piano" - Framework command: sglang generate - Runtime: torch_npu==2.10.0
**CN:** 本节围绕 Benchmark Setup 展开，概述了 Wan2.1-T2V-1.3B-Diffusers, Model, Prompt, Runtime 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Baseline (u1r1)
**EN:** ``bash sglang generate --model-path /nas/disk1/Wan2.1-T2V-1.3B-Diffusers \ --prompt "a cat is playing piano" --num-gpus 1 --ring-degree 1 \ --save-output ``
**CN:** 本节围绕 Baseline (u1r1) 展开，概述了 cat, piano, prompt, playing 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Ring-SP (u1r2)
**EN:** ``bash sglang generate --model-path /nas/disk1/Wan2.1-T2V-1.3B-Diffusers \ --prompt "a cat is playing piano" --num-gpus 2 --ring-degree 2 \ --save-output ``
**CN:** 本节围绕 Ring-SP (u1r2) 展开，概述了 cat, piano, prompt, playing 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmarks
**EN:** Benchmark Disclaimer These numbers are from one fixed setup and one prompt case. Actual performance may vary by model settings, environment, and workload.
**CN:** 本节围绕 Benchmarks 展开，概述了 one, Actual, may, case 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Stage Time Breakdown
**EN:** This section provides a comparison table for Stage Time Breakdown, covering columns such as Stage / Metric, u1r2 (s), u1r1 baseline (s) and examples such as InputValidation, TextEncoding, LatentPreparation, TimestepPreparation.
**CN:** 本节围绕 Stage Time Breakdown 展开，概述了 u1r2, u1r1, Stage, Total 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Summary
**EN:** With torch_npu==2.10.0, Ring-SP (u1r2) runs successfully on NPU for this case. End-to-end generation time improves from 266.50s to 141.86s (1.88x).
**CN:** 本节围绕 Summary 展开，概述了 DenoisingStage, NPU, u1r2, Ring-SP 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Ring-SP / **CN:** Ring-SP
- **EN:** u1r2 / **CN:** u1r2
- **EN:** u1r1 / **CN:** u1r1
- **EN:** Wan2.1-T2V-1.3B-Diffusers / **CN:** Wan2.1-T2V-1.3B-Diffusers
- **EN:** NPU / **CN:** NPU
- **EN:** Baseline / **CN:** Baseline
- **EN:** cat / **CN:** cat
- **EN:** Stage / **CN:** Stage

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
