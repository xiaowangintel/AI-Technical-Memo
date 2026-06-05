# index_cache.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/index_cache.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This doc describes IndexCache for DeepSeek-V3.2 DSA models. Its purpose is specific: reduce redundant top-k computation by caching top-k indices and reusing them across layers instead of recalculating them every time.
- **CN:** 本文档介绍了面向 DeepSeek-V3.2 DSA 模型的 IndexCache。其目标非常明确：缓存 top-k 索引并在层之间复用，从而减少每层重复执行 top-k 计算的开销。
- **EN:** The usage guidance is concise and centered on `--hf-overrides`. The two main controls are `use_index_cache` and either `index_topk_freq` or `index_topk_pattern`; the latter uses an `F`/`S` per-layer pattern where `F` means compute and `S` means reuse.
- **CN:** 使用说明很简洁，核心围绕 `--hf-overrides`。两个主要控制项是 `use_index_cache` 以及 `index_topk_freq` 或 `index_topk_pattern`；其中后者采用逐层的 `F` / `S` 模式，`F` 表示重新计算，`S` 表示复用。
- **EN:** The doc also sets boundaries clearly: this is intended for DeepSeek-V3.2 or compatible DSA models, and the feature is only active when `use_index_cache` is explicitly enabled.
- **CN:** 文档也清楚划定了边界：该特性面向 DeepSeek-V3.2 或兼容的 DSA 模型，且只有显式开启 `use_index_cache` 后才会生效。

## Key Concepts / 关键概念
- **EN:** IndexCache avoids repeated top-k work in DSA layers.  
  **CN:** IndexCache 用于避免 DSA 层中的重复 top-k 计算。
- **EN:** `index_topk_freq` controls compute frequency by layer interval.  
  **CN:** `index_topk_freq` 通过层间隔来控制重新计算频率。
- **EN:** `index_topk_pattern` provides explicit `F`/`S` per-layer behavior.  
  **CN:** `index_topk_pattern` 提供逐层显式的 `F` / `S` 行为定义。
- **EN:** The feature requires DeepSeek-V3.2 or another compatible DSA model.  
  **CN:** 该特性要求使用 DeepSeek-V3.2 或其他兼容的 DSA 模型。
