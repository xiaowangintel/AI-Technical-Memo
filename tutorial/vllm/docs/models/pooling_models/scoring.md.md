# scoring.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/pooling_models/scoring.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide explains scoring and reranking models as pairwise similarity systems spanning cross-encoder, late-interaction, and bi-encoder designs.
  **CN:** 本指南将 scoring/reranking 模型解释为成对相似度系统，涵盖 cross-encoder、late-interaction 与 bi-encoder 三种设计。
- **EN:** The page is both conceptual and operational: it describes score types, supported model families, request formats, templates, overrides, and the `/score` and `/rerank` APIs.
  **CN:** 该页面兼具概念与操作说明：既介绍 score type，也覆盖支持模型家族、请求格式、模板、覆盖参数，以及 `/score` 与 `/rerank` API。

## Key Concepts / 关键概念
- **EN:** `LLM.score` is the main offline API for similarity scoring.
  **CN:** `LLM.score` 是相似度评分的主要离线 API。
- **EN:** Online inference is exposed through `/score`, `/rerank`, `/v1/rerank`, and `/v2/rerank`.
  **CN:** 在线推理通过 `/score`、`/rerank`、`/v1/rerank` 与 `/v2/rerank` 暴露。
- **EN:** Some models need task-specific prompt templates or `hf_overrides` to reproduce official behavior.
  **CN:** 某些模型需要特定提示模板或 `hf_overrides` 才能复现官方行为。
