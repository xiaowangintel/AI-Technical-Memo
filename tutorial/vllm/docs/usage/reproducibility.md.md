# reproducibility.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/usage/reproducibility.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document explains that vLLM favors performance over strict reproducibility by default, then outlines the limited mechanisms available to make behavior more repeatable. It distinguishes offline and online constraints and clarifies how the global seed behaves in V1.
- **CN:** 本文说明 vLLM 默认优先性能而非严格可复现性，并介绍可用于提升结果一致性的有限手段。它区分了离线与在线场景的约束，并解释了 V1 中全局 seed 的行为。

## Key Concepts / 关键概念
- **EN:** Offline users can disable V1 multiprocessing for deterministic scheduling or use batch invariance; online mode only supports batch invariance.
  **CN:** 离线用户可以关闭 V1 多进程以获得确定性调度，或启用 batch invariance；在线模式则只能使用 batch invariance。
- **EN:** Even with these settings, reproducibility is only expected on the same hardware and vLLM version.
  **CN:** 即使使用这些设置，也通常只在相同硬件和相同 vLLM 版本上才能期待可复现。
- **EN:** In V1, `seed` defaults to `0`, so workers keep consistent sampling behavior across a run.
  **CN:** 在 V1 中，`seed` 默认值为 `0`，因此各 worker 在一次运行中会保持一致的采样行为。
- **EN:** Changing multiprocessing mode can also affect the random state of user code when construction happens in the same process.
  **CN:** 如果模型构造发生在同一进程中，改变多进程模式还可能影响用户代码自身的随机状态。
