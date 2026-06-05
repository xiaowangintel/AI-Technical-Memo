# token_embed.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/pooling_models/token_embed.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page explains token embeddings as per-token vector outputs, in contrast to sequence embeddings that return one vector per input.
  **CN:** 本页说明 token embedding 会为每个 token 输出向量，这与每个输入只返回一个向量的序列 embedding 不同。
- **EN:** Its main application framing is late-interaction retrieval, and the doc maps that idea to `token_embed` configuration, supported models, and pooling APIs.
  **CN:** 其主要应用定位是 late-interaction 检索，文档进一步把这一思路映射到 `token_embed` 配置、支持模型与 pooling API。

## Key Concepts / 关键概念
- **EN:** Offline inference uses `LLM.encode(..., pooling_task="token_embed")`.
  **CN:** 离线推理使用 `LLM.encode(..., pooling_task="token_embed")`。
- **EN:** `token_embed` is the foundation for late-interaction / MaxSim scoring.
  **CN:** `token_embed` 是 late-interaction / MaxSim 评分的基础。
- **EN:** The generic `/pooling` endpoint can expose token-level embeddings online.
  **CN:** 通用 `/pooling` 端点可在线暴露 token 级 embedding。
