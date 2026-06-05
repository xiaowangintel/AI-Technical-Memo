# evaluating_new_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/developer_guide/evaluating_new_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document provides commands for evaluating models' accuracy and performance. Before open-sourcing new models, we strongly suggest running these commands to verify whether the score matches your internal benchmark results. / 该文档围绕 Evaluating New 模型s with SGLang 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document provides commands for evaluating models' accuracy and performance. Before open-sourcing new models, we strongly suggest running these commands to verify whether the score matches your internal benchmark results.
**CN:** 本节围绕 Overview 展开，概述了 models, commands, Before, MiniMax 等要点，并说明相关配置、流程、示例或限制条件。

### Section: LLMs
**EN:** SGLang provides built-in scripts to evaluate common benchmarks. You may skip it if the model has forced thinking enabled.
**CN:** 本节围绕 LLMs 展开，概述了 port, --port, eval-name, --eval-name 等要点，并说明相关配置、流程、示例或限制条件。

### Section: VLMs
**EN:** **MMMU** ``bash python benchmark/mmmu/bench_sglang.py \ --port 30000 \ --concurrency 64 ` ` You can set max tokens by passing --extra-request-body ' '.
**CN:** 本节围绕 VLMs 展开，概述了 MVBench, VideoMME, MMMU, tip 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance
**EN:** Performance benchmarks measure **Latency** (Time To First Token - TTFT) and **Throughput** (tokens/second).
**CN:** 本节围绕 性能 展开，概述了 Time, TTFT, Latency, Throughput 等要点，并说明相关配置、流程、示例或限制条件。

### Section: LLMs
**EN:** **Latency-Sensitive Benchmark** This simulates a scenario with low concurrency (e.g., single user) to measure latency.
**CN:** 本节围绕 LLMs 展开，概述了 num-prompts, --num-prompts, max-concurrency, --max-concurrency 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reporting Results
**EN:** For each evaluation, please report: 1. **Metric Score**: Accuracy % (LLMs and VLMs); Latency (ms) and Throughput (tok/s) (LLMs only).
**CN:** 本节围绕 Reporting Results 展开，概述了 LLMs, GPU, VLMs, Model 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** port / **CN:** port
- **EN:** --port / **CN:** --port
- **EN:** LLMs / **CN:** LLMs
- **EN:** models / **CN:** 模型s
- **EN:** num-prompts / **CN:** num-prompts
- **EN:** --num-prompts / **CN:** --num-prompts
- **EN:** max-concurrency / **CN:** max-concurrency
- **EN:** --max-concurrency / **CN:** --max-concurrency

## Dependencies / 依赖关系
- `benchmark/hellaswag/bench_sglang.py`
- `benchmark/mmmu/bench_sglang.py`
