# faq.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/usage/faq.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This FAQ addresses a few recurring user questions rather than serving as a full troubleshooting guide. The answers clarify deployment limitations, embedding-model choices, and why outputs may differ across runs.
- **CN:** 这份 FAQ 处理的是一些高频问题，而不是完整的故障排查手册。回答重点澄清了部署限制、embedding 模型选择，以及为何多次运行输出可能不同。

## Key Concepts / 关键概念
- **EN:** Serving multiple models behind one OpenAI-compatible port is not directly supported; users should route across multiple server instances.
  **CN:** 在单个 OpenAI 兼容端口后同时提供多个模型当前并不直接支持，通常需要在多个服务实例前加一层路由。
- **EN:** Dedicated embedding models are recommended over repurposed text-generation models for offline embeddings.
  **CN:** 离线 embedding 更推荐使用专门训练的 embedding 模型，而不是复用文本生成模型。
- **EN:** Output variation can happen because batching differences and numeric instability change logits and sampled tokens.
  **CN:** 由于批处理差异和数值不稳定性会影响 logits 与采样结果，因此输出可能在不同运行间发生变化。
- **EN:** Using higher precision or request seeds can reduce variance, but does not fully guarantee identical outputs.
  **CN:** 使用更高精度或请求种子可以降低波动，但不能完全保证结果一致。
