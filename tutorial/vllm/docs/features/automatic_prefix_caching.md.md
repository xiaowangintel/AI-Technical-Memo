# automatic_prefix_caching.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/automatic_prefix_caching.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This document explains Automatic Prefix Caching (APC), a feature that reuses KV cache from earlier requests when new requests share the same prefix. The goal is to skip repeated prefill computation for shared prompt segments.

**CN:** 本文介绍 Automatic Prefix Caching（APC，自动前缀缓存）：当新请求与旧请求拥有相同前缀时，可复用已有 KV cache，从而跳过共享前缀的重复 prefill 计算。

**EN:** The workflow is simple: enable `enable_prefix_caching=True` in the engine and follow the linked offline example. The page then grounds the feature in two high-value scenarios—repeated queries over the same long document and multi-round chat sessions with growing history.

**CN:** 使用流程较直接：在引擎中设置 `enable_prefix_caching=True`，并参考链接的离线示例。文档随后用两个高价值场景说明其作用：对同一长文档反复提问，以及带有累积历史的多轮对话。

**EN:** The main caveat is scope: APC speeds up prefill only, not token decoding. Gains are limited when outputs are long or when requests do not share prefixes, so the benefit depends heavily on workload shape.

**CN:** 主要限制在于作用范围：APC 只优化 prefill，不会加速解码阶段。因此如果回答很长，或请求之间没有共享前缀，收益就会有限，效果高度依赖具体负载形态。

## Key Concepts / 关键概念
- **EN:** Reusing KV cache for shared prefixes. **CN:** 为共享前缀复用 KV cache。
- **EN:** Enablement via `enable_prefix_caching=True`. **CN:** 通过 `enable_prefix_caching=True` 启用。
- **EN:** Best fit for long-document QA and multi-turn chat. **CN:** 最适合长文档问答与多轮对话场景。
- **EN:** Improves prefill latency/throughput, not decode speed. **CN:** 提升的是 prefill 延迟与吞吐，而非解码速度。
