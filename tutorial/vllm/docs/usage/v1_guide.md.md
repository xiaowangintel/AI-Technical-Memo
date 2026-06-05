# v1_guide.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/usage/v1_guide.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This living guide summarizes the architectural shift from vLLM V0 to V1 and highlights user-visible differences, supported platforms, and removed features. It is both a migration note and a status dashboard for the newer core engine.
- **CN:** 这份持续更新的指南总结了 vLLM 从 V0 到 V1 的架构转变，并突出用户可见的行为差异、支持平台以及已移除特性。它既是迁移说明，也是新核心引擎的状态看板。

## Key Concepts / 关键概念
- **EN:** V1 keeps proven model/kernel pieces from V0 but re-architects the scheduler, cache manager, workers, sampler, and API server.
  **CN:** V1 保留了 V0 中成熟的模型与内核部分，但对调度器、缓存管理器、worker、采样器和 API server 进行了重构。
- **EN:** Key goals are simplicity, modularity, low CPU overhead, unified optimizations, and zero-config defaults.
  **CN:** 其核心目标是简洁、模块化、低 CPU 开销、统一优化，以及尽量做到零配置默认可用。
- **EN:** Behavior changes include default chunked prefill, different logprob semantics, and prompt-logprob interactions with prefix caching.
  **CN:** 行为变化包括默认启用 chunked prefill、logprob 语义调整，以及 prompt logprob 与 prefix caching 的交互差异。
- **EN:** The guide also tracks support matrices and documents removed features such as `best_of`, per-request logits processors, and GPU↔CPU KV swapping.
  **CN:** 指南还维护了支持矩阵，并记录了已移除特性，例如 `best_of`、按请求 logits processor，以及 GPU↔CPU KV 交换。
