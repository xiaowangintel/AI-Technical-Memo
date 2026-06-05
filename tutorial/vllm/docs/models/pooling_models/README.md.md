# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/pooling_models/README.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This is the conceptual hub for pooling models in vLLM, positioning them as NLU-oriented, non-generative workloads that reuse transformer infrastructure.
  **CN:** 这是 vLLM 中 pooling 模型的概念总览，强调它们面向 NLU 风格的非生成任务，并复用 Transformer 基础设施。
- **EN:** The document defines task taxonomy, pooling types, offline/online APIs, runner settings, model conversion, and task-specific subpages, while also noting that current support is mainly for convenience.
  **CN:** 文档定义了任务分类、pooling 类型、离线/在线 API、runner 设置、模型转换以及各类子页面，同时提醒目前该支持主要出于便利性。

## Key Concepts / 关键概念
- **EN:** Sequence-wise tasks and token-wise tasks are separated by output granularity.
  **CN:** 序列级任务与 token 级任务按输出粒度区分。
- **EN:** Core pooling tasks are `classify`, `embed`, `token_classify`, and `token_embed`.
  **CN:** 核心 pooling 任务包括 `classify`、`embed`、`token_classify` 和 `token_embed`。
- **EN:** `--runner pooling`, `--convert <type>`, and `/pooling` are the major integration primitives.
  **CN:** `--runner pooling`、`--convert <type>` 与 `/pooling` 是主要集成原语。
