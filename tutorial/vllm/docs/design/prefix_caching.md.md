# prefix_caching.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/prefix_caching.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** The document presents automatic prefix caching as a nearly free optimization that reuses KV-cache blocks when a new request shares the same prefix as an earlier request. **CN:** 文档将自动前缀缓存描述为一种几乎“白赚”的优化：当新请求与旧请求共享前缀时，可直接复用已有 KV cache block。
- **EN:** vLLM uses a hash-based design: each full KV block is identified by its parent hash, the block’s own tokens, and extra hashes such as LoRA IDs, multimodal hashes, or cache salts. **CN:** vLLM 采用基于哈希的设计：每个完整 KV block 由父块哈希、当前 block 的 token，以及 LoRA ID、多模态哈希、cache salt 等额外哈希共同标识。
- **EN:** The document highlights important correctness and security details, including full-block-only caching, the switch to `sha256` by default, reproducible CBOR variants, and the tradeoff of faster but less secure `xxhash` modes. **CN:** 文档强调了正确性与安全性细节，包括只缓存完整 block、默认切换到 `sha256`、可复现的 CBOR 变体，以及更快但安全性较弱的 `xxhash` 模式取舍。
- **EN:** It also explains multimodal support and cache isolation: image hashes distinguish placeholder spans, while per-request `cache_salt` limits reuse to trusted groups and reduces timing-leak risk. **CN:** 文档还解释了多模态支持与缓存隔离：图像哈希用于区分占位符片段，而每请求 `cache_salt` 可将复用限制在受信任组内，降低基于时延的泄露风险。
- **EN:** On the implementation side, the v1 KV cache manager uses a preallocated block pool, a doubly linked free queue, cache/request mappings, and LRU eviction, with an append-only block table that can temporarily create duplicate cached blocks. **CN:** 在实现层面，v1 的 KV cache manager 使用预分配 block 池、双向链表空闲队列、cache/request 映射与 LRU 淘汰；由于 block table 只追加不回写，系统会暂时出现重复缓存块。
- **EN:** The closing example walks through allocation, cache hits, freeing, and eviction over time to make the data-structure behavior concrete. **CN:** 文档最后通过时间线示例串联了分配、命中、释放和淘汰过程，使数据结构行为更直观。

## Key Concepts / 关键概念
- **EN:** **Block hash chaining:** every block depends on the previous block’s hash, so reuse is tied to the exact prefix path rather than isolated local tokens. **CN:** **块哈希链：** 每个 block 都依赖前一个 block 的哈希，因此复用绑定的是完整前缀路径，而不只是局部 token。
- **EN:** **Extra hashes:** multimodal inputs, LoRA settings, and cache salts are folded into the key so logically different requests do not collide. **CN:** **额外哈希：** 多模态输入、LoRA 配置和 cache salt 都会进入键中，避免逻辑上不同的请求发生碰撞。
- **EN:** **Free queue and LRU eviction:** free blocks are tracked in a doubly linked list so touched blocks can be moved efficiently and least-recently-used cached blocks can be evicted. **CN:** **空闲队列与 LRU 淘汰：** 空闲块通过双向链表维护，从而能高效移动被触碰的块，并淘汰最近最少使用的缓存块。
- **EN:** **Append-only block tables:** vLLM v1 may keep duplicate cached blocks temporarily because running requests are not allowed to rewrite their block tables. **CN:** **只追加的 block table：** vLLM v1 可能暂时保留重复缓存块，因为运行中的请求不允许重写自己的 block table。
- **EN:** **Cache isolation:** `cache_salt` provides a practical way to share cache only within an agreed trust boundary. **CN:** **缓存隔离：** `cache_salt` 提供了一种实用机制，使缓存只在约定好的信任边界内共享。
