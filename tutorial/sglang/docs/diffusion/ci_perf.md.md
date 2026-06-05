# ci_perf.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/ci_perf.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Perf Baseline Generation Script python/sglang/multimodal_gen/test/scripts/gen_perf_baselines.py starts a local diffusion server, issues requests for selected test cases, aggregates stage/denoise-step/E2E timings from the perf log, and writes the results back to the scenarios section of perf_baselines.json. / 该文档围绕 CI 性能 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Perf Baseline Generation Script
**EN:** python/sglang/multimodal_gen/test/scripts/gen_perf_baselines.py starts a local diffusion server, issues requests for selected test cases, aggregates stage/denoise-step/E2E timings from the perf log, and writes the results back to the scenarios section of perf_baselines.json.
**CN:** 本节围绕 Perf Baseline Generation Script 展开，概述了 scenarios, perf_baselines.json, E2E, log 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** This content focuses on Usage and highlights python/sglang/multimodal_gen/test/scripts/gen_perf_baselines.py, Run, case, Update.
**CN:** 本节围绕 Usage 展开，概述了 python/sglang/multimodal_gen/test/scripts/gen_perf_baselines.py, Run, case, Update 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** python/sglang/multimodal_gen/test/scripts/gen_perf_baselines.py / **CN:** python/sglang/multimodal_gen/test/scripts/gen_perf_baselines.py
- **EN:** scenarios / **CN:** scenarios
- **EN:** Run / **CN:** Run
- **EN:** case / **CN:** case
- **EN:** Select / **CN:** Select
- **EN:** Specify / **CN:** Specify
- **EN:** timeout / **CN:** timeout
- **EN:** baseline / **CN:** baseline

## Dependencies / 依赖关系
- `python/sglang/multimodal_gen/test/scripts/gen_perf_baselines.py`
