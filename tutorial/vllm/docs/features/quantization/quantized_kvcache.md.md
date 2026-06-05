# quantized_kvcache.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/quantized_kvcache.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document focuses on FP8 quantization for the KV cache, framing it as a memory optimization that enables more cached tokens, higher throughput, and longer context support.
- **CN:** 本文聚焦 KV Cache 的 FP8 量化，把它定位为一种显存优化手段，可容纳更多缓存 token、提升吞吐，并支持更长上下文。
- **EN:** It distinguishes two scaling strategies—per-tensor and per-attention-head—and explicitly states the backend caveat: per-head quantization currently requires the Flash Attention backend and llm-compressor-based calibration.
- **CN:** 文档区分了 per-tensor 与 per-attention-head 两种缩放策略，并明确给出后端限制：per-head 量化目前需要 Flash Attention 后端以及基于 llm-compressor 的校准流程。
- **EN:** The workflow section is practical and comparative: it shows no calibration, warmup-time random-token calibration, and recommended dataset calibration through llm-compressor, plus code snippets and `kv_cache_dtype` options.
- **CN:** 工作流部分很实用且带对比性：分别展示无校准、预热阶段随机 token 校准，以及推荐的 llm-compressor 数据集校准，并给出代码示例和 `kv_cache_dtype` 选项。

## Key Concepts / 关键概念
- **EN:** FP8 KV cache reduces cache memory pressure and is directly tied to serving efficiency for long-context workloads.
  **CN:** FP8 KV cache 可以降低缓存显存压力，并直接改善长上下文任务的服务效率。
- **EN:** When Flash Attention 3 is used with FP8 KV cache, attention stays in the quantized domain and queries are quantized too.
  **CN:** 当 Flash Attention 3 与 FP8 KV cache 一起使用时，注意力计算会停留在量化域内，查询张量也会被量化。
- **EN:** Calibration options trade convenience for accuracy: fixed scale 1.0 is simplest, warmup calibration is automatic, and dataset calibration is the recommended quality path.
  **CN:** 三种校准方式体现了便利性与精度的取舍：固定 1.0 最简单，预热校准最省事，而基于数据集的校准是推荐的高质量路径。
- **EN:** The doc links KV-cache quantization to external tooling by showing a full `llm-compressor` recipe for attention and cache calibration/export.
  **CN:** 文档通过完整的 `llm-compressor` 配方，把 KV-cache 量化与外部工具链的校准/导出流程连接起来。

