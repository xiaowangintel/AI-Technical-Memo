# batch_invariance.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/batch_invariance.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This page documents batch invariance, a beta feature that makes model outputs deterministic regardless of batch size or request ordering. It frames the feature around reproducibility-sensitive use cases such as framework debugging, model debugging, RL rollouts, and large-scale inference validation.

**CN:** 本页说明 batch invariance（批不变性）这一 beta 功能：无论 batch 大小或请求顺序如何变化，模型输出都保持确定性。文档将其定位于对可复现性要求较高的场景，如框架调试、模型调试、强化学习 rollout，以及大规模推理系统验证。

**EN:** The document gives a full enablement workflow: set `VLLM_BATCH_INVARIANT=1`, then use either server mode or offline generation, both with seeded sampling examples. It also records concrete deployment constraints, especially the requirement for NVIDIA GPUs with compute capability 9.0+.

**CN:** 文档给出了完整启用流程：设置 `VLLM_BATCH_INVARIANT=1`，然后在服务模式或离线推理中使用，并配有带随机种子的示例。同时明确了部署约束，尤其要求 NVIDIA 计算能力 9.0 及以上的 GPU。

**EN:** Beyond usage, the page lists validated model families, explains that deterministic kernels and consistent numerical behavior are enforced, and warns that some optimizations are disabled. The trade-off is clear: stronger reproducibility can cost performance, and coverage is still expanding.

**CN:** 除了使用方法，页面还列出了已验证的模型系列，解释了其通过确定性 kernel 与一致的数值行为来实现目标，并说明会关闭部分可能引入非确定性的优化。因此其权衡也很明确：更强的可复现性通常会带来性能损失，而且支持范围仍在持续扩展。

## Key Concepts / 关键概念
- **EN:** Deterministic outputs independent of batch composition. **CN:** 输出结果不受 batch 组成影响而保持确定性。
- **EN:** Enabled through the `VLLM_BATCH_INVARIANT` environment variable. **CN:** 通过 `VLLM_BATCH_INVARIANT` 环境变量启用。
- **EN:** Requires modern NVIDIA GPUs (compute capability 9.0+). **CN:** 需要较新的 NVIDIA GPU（计算能力 9.0+）。
- **EN:** Reproducibility is prioritized over peak performance. **CN:** 该特性优先保证可复现性，而非峰值性能。
- **EN:** Model support is explicit but still growing. **CN:** 模型支持已有明确列表，但仍在扩展中。
