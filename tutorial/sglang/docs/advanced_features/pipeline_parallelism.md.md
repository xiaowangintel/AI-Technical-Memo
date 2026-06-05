# pipeline_parallelism.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/pipeline_parallelism.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Why Pipeline Parallelism? As Large Language Models (LLMs) scale toward trillion-parameter architectures and "infinite" context windows, the underlying serving infrastructure must evolve toward more granular, cross-node parallelization strategies. / 该文档围绕 Pipeline 并行策略 for Long Context 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Why Pipeline Parallelism?
**EN:** As Large Language Models (LLMs) scale toward trillion-parameter architectures and "infinite" context windows, the underlying serving infrastructure must evolve toward more granular, cross-node parallelization strategies.
**CN:** 本节围绕 Why Pipeline 并行策略? 展开，概述了 ITL, LLMs, Time, TTFT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Implementation Refactoring based on Async Communication
**EN:** With Dynamic Chunked Prefill, pipeline parallelism has the potential to reduce the TTFT of long-context inputs. For each request, its input tokens can be partitioned into multiple chunks, each no longer than the chunked prefill size.
**CN:** 本节围绕 Implementation Refactoring based on Async Communication 展开，概述了 GPU, CPU, TTFT, P2PWork 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Why Dynamic Chunking
**EN:** Chunked prefill with a fixed size can cause bubbles in the pipeline, especially when the pp size is large.
**CN:** 本节围绕 Why Dynamic Chunking 展开，概述了 size, chunk, Runtime, next 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Chunked Prefill Size and Smoothing Factor
**EN:** When --enable-dynamic-chunking is enabled, each chunk size of a sequence is determined dynamically based on the quadratic model that predicts the next chunk size based on the estimated runtime of the initial chunk length.
**CN:** 本节围绕 Chunked prefill 阶段 Size and Smoothing Factor 展开，概述了 size, chunk, dynamic, prefill 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tuning the Chunked Prefill Size
**EN:** Optimizing the chunked prefill size is crucial for balancing pipeline efficiency and resource utilization. The ideal size depends on factors including model architecture, hardware configuration, and typical input lengths.
**CN:** 本节围绕 Tuning the Chunked prefill 阶段 Size 展开，概述了 size, optimal, ITL, Sizes 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Enable Dynamic Chunking and Adjust Smoothing Factor for Ultra-long ITL
**EN:** SGLang also offers a dynamic chunking solution that could further improve performance. This feature is currently an experimental feature that requires a certain amount of tuning experimentation and may not be suitable for all workloads.
**CN:** 本节围绕 Enable Dynamic Chunking and Adjust Smoothing Factor for Ultra-long ITL 展开，概述了 feature, workloads, performance, may 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Case Study on NVIDIA H20
**EN:** When evaluating pipeline parallelism with fixed chunked prefill sizes from 2K to 16K, experiment results show that a 4K chunk size delivered optimal prefill TTFT performance for the DeepSeek-V3.1, and a 6K chunk size delivered optimal prefill TTFT performance for the Qwen3-235B-A22B-FP8.
**CN:** 本节围绕 Case Study on NVIDIA H20 展开，概述了 prefill, size, DeepSeek-V3.1, MASTER_NODE_IP 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Best Practice for Pipeline Parallelism with PD Disaggregation
**EN:** Stay tuned for the latest updates on Pipeline Parallelism with PD Disaggregation.
**CN:** 本节围绕 Best Practice for Pipeline 并行策略 with PD Disaggregation 展开，概述了 Stay, Disaggregation, added, tuned 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** size / **CN:** size
- **EN:** chunk / **CN:** chunk
- **EN:** prefill / **CN:** prefill 阶段
- **EN:** chunked / **CN:** chunked
- **EN:** dynamic / **CN:** dynamic
- **EN:** optimal / **CN:** optimal
- **EN:** initial / **CN:** initial
- **EN:** chunking / **CN:** chunking

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
