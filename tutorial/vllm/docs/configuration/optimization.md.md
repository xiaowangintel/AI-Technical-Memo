# optimization.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/configuration/optimization.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This is the main performance tuning guide for vLLM V1. It discusses optimization levels, preemption behavior, chunked prefill tuning, multiple parallelism strategies, NUMA binding, tokenizer acceleration, API-server scale-out, multimodal caching, CPU sizing for GPU deployments, and attention backend selection.

**EN:** The document is valuable because it ties system behavior to concrete hardware and workload tradeoffs. Instead of presenting flags in isolation, it explains why a setting improves TTFT, ITL, throughput, CPU efficiency, cache reuse, or memory headroom.

**CN:** 这是 vLLM V1 的核心性能调优指南，内容涵盖优化等级、抢占机制、chunked prefill 调优、多种并行策略、NUMA 绑定、tokenizer 加速、API server 扩展、多模态缓存、GPU 部署下的 CPU 规划，以及 attention backend 选择。

**CN:** 这份文档的价值在于它把系统行为和硬件/负载权衡联系起来：并不是孤立地罗列参数，而是解释某个设置为什么会改善 TTFT、ITL、吞吐、CPU 利用率、缓存复用或内存余量。

## Key Concepts / 关键概念
- **EN:** Chunked prefill tuning — Explains how `max_num_batched_tokens` changes TTFT, ITL, and throughput.
- **CN:** Chunked prefill 调优 — 解释 `max_num_batched_tokens` 如何影响 TTFT、ITL 和吞吐。
- **EN:** Parallelism strategies — Compares TP, PP, EP, DP, and batch-level DP for different scaling goals.
- **CN:** 并行策略 — 比较 TP、PP、EP、DP 以及 batch-level DP 在不同扩展目标下的作用。
- **EN:** NUMA and CPU placement — Shows that CPU locality and process affinity can materially affect GPU-serving performance.
- **CN:** NUMA 与 CPU 绑定 — 说明 CPU 局部性和进程亲和性会显著影响 GPU 服务性能。
- **EN:** Multimodal caching — Separates processor caching from IPC caching and explains cache-placement tradeoffs.
- **CN:** 多模态缓存 — 区分 processor caching 与 IPC caching，并解释缓存放置的取舍。
- **EN:** CPU provisioning — Warns that insufficient CPU cores can bottleneck otherwise capable GPU deployments.
- **CN:** CPU 资源规划 — 提醒 CPU 核数不足会成为 GPU 部署的真实瓶颈。
