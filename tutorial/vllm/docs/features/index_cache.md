# index_cache.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/index_cache.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes *IndexCache*, an optimisation for DeepSeek-V3.2 (DSA) models that reduces redundant top-k token selection computations across layers by caching and reusing indices from earlier layers.

**CN:** 本文档介绍 *IndexCache*，这是针对 DeepSeek-V3.2（DSA）模型的优化，通过缓存并复用早期层的索引来减少各层冗余的 top-k token 选择计算。

### Background / 背景

**EN:** DeepSeek-V3.2 uses DeepSeek Sparse Attention (DSA), which computes per-layer top-k token selection. For deep models with many DSA layers, this is expensive. IndexCache skips redundant computations by sharing indices from a "Full" layer to subsequent "Shared" layers.

**CN:** DeepSeek-V3.2 使用 DeepSeek 稀疏注意力（DSA），每层均需计算 top-k token 选择。对于拥有大量 DSA 层的深度模型，此开销较大。IndexCache 通过将"Full"层的索引共享给后续"Shared"层来跳过冗余计算。

### Configuration / 配置

**EN:** Enabled via `--hf-overrides '{"use_index_cache": true, "index_topk_freq": N}'`. Two tuning knobs: `index_topk_freq` (compute full top-k every N layers) and `index_topk_pattern` (explicit per-layer `F`/`S` string for fine-grained control, overrides `index_topk_freq`).

**CN:** 通过 `--hf-overrides '{"use_index_cache": true, "index_topk_freq": N}'` 启用。两个调优旋钮：`index_topk_freq`（每 N 层完整计算一次 top-k）和 `index_topk_pattern`（逐层显式指定 `F`/`S` 字符串，覆盖 `index_topk_freq`）。

### How It Works / 工作原理

**EN:** Layers marked `F` (Full) compute and store top-k indices. Layers marked `S` (Shared) receive the cached indices from the previous `F` layer and skip recomputation. This reduces total top-k computation roughly by `(1 - 1/index_topk_freq)`.

**CN:** 标记为 `F`（Full）的层计算并存储 top-k 索引；标记为 `S`（Shared）的层接收上一个 `F` 层的缓存索引并跳过重新计算。总体上减少约 `(1 - 1/index_topk_freq)` 的 top-k 计算量。

### Requirements / 要求

**EN:** Only works with DeepSeek-V3.2 or compatible DSA models. Must be enabled via `--hf-overrides`.

**CN:** 仅适用于 DeepSeek-V3.2 或兼容的 DSA 模型，必须通过 `--hf-overrides` 启用。

## Key Concepts / 关键概念

- **EN:** DSA (DeepSeek Sparse Attention) — a sparse attention mechanism that selects the top-k tokens per layer, driving the need for IndexCache. **CN:** DSA（DeepSeek 稀疏注意力）——一种每层选择 top-k token 的稀疏注意力机制，是 IndexCache 的需求来源。
- **EN:** `F` layer — computes and stores top-k indices for downstream `S` layers to reuse. **CN:** `F` 层——计算并存储 top-k 索引，供下游 `S` 层复用。
- **EN:** `S` layer — receives cached indices from the nearest upstream `F` layer, skipping recomputation. **CN:** `S` 层——接收最近上游 `F` 层的缓存索引，跳过重新计算。
- **EN:** `index_topk_freq=4` — computes full top-k on 1/4 of layers, reusing indices on the other 3/4. **CN:** `index_topk_freq=4`——在 1/4 的层上完整计算 top-k，其余 3/4 层复用索引。
- **EN:** `index_topk_pattern` — character-per-DSA-layer string (e.g., `"FFSFSS..."`) for custom F/S assignment. **CN:** `index_topk_pattern`——每个 DSA 层对应一个字符（如 `"FFSFSS..."`）的自定义 F/S 分配字符串。
