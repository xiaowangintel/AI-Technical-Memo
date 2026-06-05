# embed.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/pooling_models/embed.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide explains embedding models as encoders that turn text or multimodal inputs into vector representations for retrieval and similarity workflows.
  **CN:** 本指南将 embedding 模型描述为编码器，可把文本或多模态输入转换为向量表示，用于检索与相似度场景。
- **EN:** The page is structured around API mapping: it relates embedding usage to offline methods and multiple online endpoints, including Cohere-style and OpenAI-compatible embeddings APIs.
  **CN:** 该页面围绕 API 映射展开：把 embedding 用法关联到离线方法以及多个在线端点，包括 Cohere 风格和 OpenAI 兼容的 embeddings API。

## Key Concepts / 关键概念
- **EN:** Use `LLM.embed(...)` or `LLM.encode(..., pooling_task="embed")` offline.
  **CN:** 离线可使用 `LLM.embed(...)` 或 `LLM.encode(..., pooling_task="embed")`。
- **EN:** Online embedding endpoints include `/v2/embed`, `/v1/embeddings`, and `/pooling`.
  **CN:** 在线 embedding 端点包括 `/v2/embed`、`/v1/embeddings` 与 `/pooling`。
- **EN:** Embedding vectors also support downstream similarity and scoring workflows.
  **CN:** embedding 向量还支持下游相似度与评分工作流。
