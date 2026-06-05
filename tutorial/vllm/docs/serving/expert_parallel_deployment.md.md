# expert_parallel_deployment.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/expert_parallel_deployment.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide explains expert parallelism for Mixture-of-Experts models, where experts are distributed across GPUs to improve locality and throughput.
  **CN:** 本指南解释了面向 MoE 模型的 expert parallelism：将专家分布到不同 GPU 上，以提升局部性和吞吐。
- **EN:** It connects architecture to deployment math by showing how expert parallelism interacts with tensor parallelism, data parallelism, and all-to-all communication backends.
  **CN:** 它把架构与部署计算联系起来，说明 expert parallelism 如何与 tensor parallelism、data parallelism 以及 all-to-all 通信后端协同。

## Key Concepts / 关键概念
- **EN:** EP is typically paired with DP, while TP influences how attention weights are handled.
  **CN:** EP 通常与 DP 配合使用，而 TP 会影响 attention 权重的处理方式。
- **EN:** `EP_SIZE` is derived from tensor and data parallel sizing in the deployment formulas.
  **CN:** `EP_SIZE` 会在部署公式中由 tensor parallel 与 data parallel 的规模推导得出。
- **EN:** Backend choice via `--all2all-backend` is an important operational decision.
  **CN:** 通过 `--all2all-backend` 选择后端是重要的运维决策。
