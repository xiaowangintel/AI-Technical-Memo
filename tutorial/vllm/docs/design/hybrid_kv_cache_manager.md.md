# hybrid_kv_cache_manager.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/hybrid_kv_cache_manager.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** Presents an early-stage design for managing KV cache in hybrid models that mix full attention, sliding-window attention, local attention, or Mamba-style state. **CN:** 提出一个早期设计，用于管理混合模型中的 KV cache，这些模型会组合 full attention、sliding-window attention、local attention 或 Mamba 状态。
- **EN:** Defines the memory vocabulary—block, page, block size, page size, and KV hidden size—and uses those terms to explain why one cache pool must still serve heterogeneous layers. **CN:** 定义 block、page、block size、page size 与 KV hidden size 等内存术语，并据此说明为何一个缓存池仍需服务异构层。
- **EN:** Introduces KV cache groups to keep a shared page size while handling different attention patterns, layer ratios, hidden sizes, and KV-sharing cases. **CN:** 引入 KV cache group，在保持统一 page size 的同时处理不同注意力模式、层比例、隐藏维度以及 KV 共享场景。
- **EN:** Describes hybrid prefix caching and the manager/coordinator layering that maps grouped cache logic onto the final memory layout. **CN:** 描述混合前缀缓存，以及如何通过 manager/coordinator 分层把分组后的缓存逻辑映射到最终内存布局。

## Key Concepts / 关键概念
- **EN:** KV cache group: a set of layers that must share one physical page size and allocation policy. **CN:** KV cache group：一组必须共享同一物理 page size 与分配策略的层。
- **EN:** Grouping heuristics: the design handles regular ratios, irregular patterns, differing `kv_hidden_size`, and shared-layer cases separately. **CN:** 分组启发式：该设计分别处理规则比例、不规则模式、不同 `kv_hidden_size` 以及共享层等情况。
- **EN:** Prefix-cache intersection: for hybrid models, cache hits are computed per group and then intersected into one reusable prefix. **CN:** 前缀缓存求交：对混合模型先按 group 计算缓存命中，再求交得到可复用的统一前缀。
- **EN:** Single-type managers: grouped layers of the same type can still reuse simpler allocation managers under a higher-level coordinator. **CN:** 单类型管理器：同类型的分组层仍可在更高层 coordinator 之下复用更简单的分配管理器。
- **EN:** Design caveat: the document explicitly marks the implementation as work in progress and evolving. **CN:** 设计提示：文档明确指出该实现仍在开发中，后续可能继续演进。
