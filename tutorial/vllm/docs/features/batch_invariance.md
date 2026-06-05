# batch_invariance.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/batch_invariance.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes vLLM's *Batch Invariance* feature (beta), which ensures that model outputs are deterministic and independent of batch size or request order. It covers motivation, hardware requirements, enabling instructions, tested models, and implementation details.

**CN:** 本文档介绍 vLLM 的*批次不变性*功能（Beta 阶段），确保模型输出具有确定性，且与批次大小或请求顺序无关。内容涵盖动机、硬件要求、启用方法、已测试模型及实现细节。

### Motivation / 动机

**EN:** Key use cases include framework/model debugging (deterministic outputs simplify issue isolation), RL training (reproducible rollouts), and large-scale inference pipelines that require consistency guarantees for testing and validation.

**CN:** 核心使用场景包括：框架/模型调试（确定性输出简化问题定位）、强化学习训练（可复现的推演结果），以及需要一致性保证的大规模推理系统。

### Enabling / 启用方式

**EN:** Set the environment variable `VLLM_BATCH_INVARIANT=1` before launching vLLM. Works for both online serving (`vllm serve`) and offline inference (`LLM(...)` API).

**CN:** 在启动 vLLM 之前设置环境变量 `VLLM_BATCH_INVARIANT=1`，适用于在线服务（`vllm serve`）和离线推理（`LLM(...)` API）两种模式。

### Hardware Requirements / 硬件要求

**EN:** Requires NVIDIA GPUs with compute capability 9.0+ (H100, H200, B100, B200). Lower-capability GPUs are not supported.

**CN:** 需要算力等级 ≥ 9.0 的 NVIDIA GPU（H100、H200、B100、B200），不支持低算力 GPU。

### Implementation Details / 实现细节

**EN:** When enabled, vLLM switches to deterministic kernel implementations for attention and disables non-deterministic optimisations such as custom all-reduce in tensor-parallel mode. This may reduce throughput compared to default mode.

**CN:** 启用后，vLLM 切换到注意力机制的确定性内核实现，并禁用张量并行模式中的自定义 all-reduce 等非确定性优化，吞吐量可能低于默认模式。

### Tested Models / 已测试模型

**EN:** DeepSeek-V3/R1 series, Qwen3 dense and MoE, Qwen2.5, Llama 3, GPT-OSS 20B/120B, Mistral-7B.

**CN:** DeepSeek-V3/R1 系列、Qwen3 稠密/MoE、Qwen2.5、Llama 3、GPT-OSS 20B/120B、Mistral-7B。

## Key Concepts / 关键概念

- **EN:** Determinism — same inputs always produce the same outputs regardless of concurrent requests or batch composition. **CN:** 确定性——相同输入在任意批次组成下始终产生相同输出。
- **EN:** `VLLM_BATCH_INVARIANT=1` — the single environment variable that activates the feature. **CN:** `VLLM_BATCH_INVARIANT=1`——激活该功能的唯一环境变量。
- **EN:** Performance trade-off — determinism disables certain throughput-boosting non-deterministic kernels. **CN:** 性能权衡——确定性模式会禁用部分提升吞吐量的非确定性内核。
- **EN:** Tensor-parallel compatibility — custom all-reduce is disabled to preserve numerical consistency. **CN:** 张量并行兼容性——自定义 all-reduce 被禁用以保持数值一致性。
- **EN:** Beta status — active development; track progress at GitHub issue #27433. **CN:** Beta 状态——功能仍在积极开发，进度跟踪见 GitHub issue #27433。
