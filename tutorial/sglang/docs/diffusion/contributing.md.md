# contributing.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/contributing.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This guide outlines the requirements for contributing to the SGLang Diffusion module (sglang.multimodal_gen). Contributor Guides Support New Models: implementation guide for adding new diffusion pipelines CI Performance: update and regenerate perf baselines. / 该文档围绕 Contributing to SGLang Diffusion 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This guide outlines the requirements for contributing to the SGLang Diffusion module (sglang.multimodal_gen).
**CN:** 本节围绕 Overview 展开，概述了 sglang.multimodal_gen, module, outlines, Diffusion 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Contributor Guides
**EN:** Support New Models: implementation guide for adding new diffusion pipelines - CI Performance: update and regenerate perf baselines `` :maxdepth: 1 support_new_models ci_perf ``
**CN:** 本节围绕 Contributor Guides 展开，概述了 Performance, New, perf, Models 等要点，并说明相关配置、流程、示例或限制条件。

### Section: On AI-Assisted ("Vibe Coding") PRs
**EN:** Vibe-coded PRs are welcome — we judge code quality, not how it was produced. The bar is the same for all PRs: - **No over-commenting.** If the name says it all, skip the docstring.
**CN:** 本节围绕 On AI-Assisted ("Vibe Coding") PRs 展开，概述了 PRs, Don, Test, code 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Commit Message Convention
**EN:** We follow a structured commit message format to maintain a clean history.
**CN:** 本节围绕 Commit Message Convention 展开，概述了 diffusion, 
- , cli, add 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance Reporting
**EN:** For PRs that impact **latency**, **throughput**, or **memory usage**, you **should** provide a performance comparison report.
**CN:** 本节围绕 性能 Reporting 展开，概述了 PRs, usage, impact, memory 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How to Generate a Report
**EN:** **Baseline**: run the benchmark (for a single generation task) ``bash $ sglang generate --model-path <model> --prompt "A benchmark prompt" --perf-dump-path baseline.json ` 2.
**CN:** 本节围绕 How to Generate a Report 展开，概述了 prompt, benchmark, run, New 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CI-Based Change Protection
**EN:** See test for examples
**CN:** 本节围绕 CI-Based Change Protection 展开，概述了 PRs, See, new, model 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** PRs / **CN:** PRs
- **EN:** model / **CN:** 模型
- **EN:** diffusion / **CN:** diffusion
- **EN:** run / **CN:** run
- **EN:** prompt / **CN:** prompt
- **EN:** benchmark / **CN:** benchmark
- **EN:** Performance / **CN:** 性能
- **EN:** New / **CN:** New

## Dependencies / 依赖关系
- `support_new_models.md`
- `ci_perf.md`
- `testcase_configs.py`
- `python/sglang/multimodal_gen/benchmarks/compare_perf.py`
