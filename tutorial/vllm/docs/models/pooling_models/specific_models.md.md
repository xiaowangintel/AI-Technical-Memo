# specific_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/pooling_models/specific_models.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page provides concrete examples for special pooling models, especially ColBERT-style late-interaction architectures across several encoder backbones.
  **CN:** 本页给出了特殊 pooling 模型的具体示例，尤其是多种编码器骨干上的 ColBERT 风格 late-interaction 架构。
- **EN:** It is example-driven: instead of defining generic APIs, it highlights retrieval and reranking scripts and points out when non-BERT backbones need explicit conversion choices.
  **CN:** 它以示例驱动：不是泛化定义 API，而是突出检索与重排脚本，并指出非 BERT 骨干在何时需要显式转换设置。

## Key Concepts / 关键概念
- **EN:** ColBERT keeps token-level representations and uses MaxSim late interaction for ranking.
  **CN:** ColBERT 保留 token 级表示，并使用 MaxSim late interaction 进行排序。
- **EN:** The page compares multiple backbones such as ModernBERT, XLM-R, and LFM2-derived variants.
  **CN:** 页面比较了 ModernBERT、XLM-R 以及 LFM2 派生等多种骨干。
- **EN:** Example scripts cover both multi-vector retrieval and reranking workflows.
  **CN:** 示例脚本同时覆盖多向量检索与 reranking 工作流。
