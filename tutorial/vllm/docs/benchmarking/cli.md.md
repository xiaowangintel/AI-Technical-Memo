# cli.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/benchmarking/cli.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This is the master guide for vLLM benchmarking. It explains supported datasets, online serving benchmarks, offline throughput tests, and a broad set of specialized benchmark scripts for structured outputs, long-document QA, prefix caching, hashing, prioritization, multimodal workloads, embeddings, rerankers, and multimodal processor profiling.

**EN:** The document is highly example-driven. Its main role is to show which backend, endpoint, dataset format, and flags are appropriate for each evaluation scenario, while also recommending GuideLLM for more production-oriented server benchmarking.

**CN:** 这是 vLLM 基准测试的总指南。文档覆盖支持的数据集、在线服务压测、离线吞吐测试，以及结构化输出、长文档问答、前缀缓存、哈希、优先级调度、多模态、Embedding、Reranker、多模态处理器等专项基准脚本。

**CN:** 整篇文档以命令示例为核心，重点说明不同评测场景应该使用什么 backend、endpoint、数据集格式和参数，同时也提示生产级服务压测更适合使用 GuideLLM。

## Key Concepts / 关键概念
- **EN:** Dataset coverage — Lists many built-in and custom datasets for text, audio, image, video, code, math, and synthetic workloads.
- **CN:** 数据集覆盖面 — 列出文本、音频、图像、视频、代码、数学以及合成负载等多种内置或自定义数据集。
- **EN:** Backend/endpoint matching — Shows that benchmark correctness depends on pairing the right backend with the right API endpoint.
- **CN:** 后端与接口匹配 — 说明要得到正确结果，必须把合适的 backend 与对应 API endpoint 配对。
- **EN:** Specialized benchmarks — Documents targeted scripts for prefix caching, structured outputs, hashing, prioritization, and multimodal preprocessing.
- **CN:** 专项基准测试 — 介绍前缀缓存、结构化输出、哈希、优先级调度和多模态预处理等专项测试脚本。
- **EN:** Load patterns — Explains request rate, burstiness, and max concurrency as workload-shaping controls.
- **CN:** 负载模式 — 解释 request rate、burstiness 和 max concurrency 如何共同塑造负载形态。
