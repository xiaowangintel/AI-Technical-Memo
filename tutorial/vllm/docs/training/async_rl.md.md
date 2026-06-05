# async_rl.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/training/async_rl.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page explains how vLLM supports asynchronous RL by overlapping rollout generation with trainer updates. It centers on pause/resume controls that let weights be synchronized safely while requests may still be in flight.
- **CN:** 本文说明 vLLM 如何通过将 rollout 生成与训练更新重叠来支持异步强化学习。核心是暂停/恢复控制，使得即使仍有请求在飞行中，也能安全地完成权重同步。

## Key Concepts / 关键概念
- **EN:** One-off pipelining runs generation and training as parallel coroutines to improve GPU utilization and throughput.
  **CN:** 一次性流水线化将生成与训练拆成并行协程，以提升 GPU 利用率和整体吞吐。
- **EN:** `pause_generation` supports `abort`, `wait`, and `keep`, which trade off immediacy, completeness, and request continuity.
  **CN:** `pause_generation` 提供 `abort`、`wait`、`keep` 三种模式，在即时性、完整性和请求连续性之间做权衡。
- **EN:** `clear_cache` determines whether resumed generation reuses stale KV cache or recomputes with the new weights.
  **CN:** `clear_cache` 决定恢复后是继续复用旧的 KV cache，还是使用新权重重新计算。
- **EN:** The same behavior is exposed through HTTP endpoints and has special considerations for internal vs. external data-parallel load balancing.
  **CN:** 相同行为也通过 HTTP 端点提供，并且对内部与外部数据并行负载均衡场景有不同要求。
