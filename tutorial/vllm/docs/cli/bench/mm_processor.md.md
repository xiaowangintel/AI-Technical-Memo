# mm_processor.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/cli/bench/mm_processor.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This command page documents `vllm bench mm-processor`, a benchmark for profiling multimodal preprocessing and encoder latency. It explains the purpose of the tool, a quick-start command, measured stages, and where to find fuller examples.

**EN:** Compared with the generic benchmark CLI wrappers, this page contains meaningful narrative content because the benchmark reports stage-level metrics that are useful for diagnosing vision-language bottlenecks.

**CN:** 该页面介绍 `vllm bench mm-processor`，这是一个用于分析多模态预处理与编码器延迟的基准工具。文档说明了用途、快速上手示例、测量阶段，以及更完整示例的入口。

**CN:** 和其他更薄的 CLI 包装页相比，这一页包含更实质性的说明，因为它关注的是阶段级指标，适合定位视觉语言模型的预处理瓶颈。

## Key Concepts / 关键概念
- **EN:** Per-stage latency — Breaks total request time into hashing, cache lookup, HF processing, prompt updates, and encoder forward.
- **CN:** 分阶段延迟 — 把总请求时间拆成哈希、缓存查询、HF 处理、提示更新和编码器前向等阶段。
- **EN:** Multimodal profiling — Targets the preprocessing pipeline for vision-language inputs rather than pure decoding throughput.
- **CN:** 多模态剖析 — 关注视觉语言输入的预处理链路，而不是纯解码吞吐。
- **EN:** Synthetic and HF data — Supports both random multimodal data and Hugging Face datasets.
- **CN:** 合成与 HF 数据 — 同时支持随机多模态数据和 Hugging Face 数据集。
