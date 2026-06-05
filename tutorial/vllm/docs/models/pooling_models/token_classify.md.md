# token_classify.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/pooling_models/token_classify.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide introduces token classification, where vLLM returns a label distribution for each token rather than one result for the whole sequence.
  **CN:** 本指南介绍 token classification：vLLM 为每个 token 返回标签分布，而不是为整个序列只返回一个结果。
- **EN:** The document ties this task to concrete use cases such as NER and alignment, then maps it to pooling APIs and supported models.
  **CN:** 文档将该任务与 NER、对齐等具体场景关联起来，并进一步映射到 pooling API 和支持模型。

## Key Concepts / 关键概念
- **EN:** Use `LLM.encode(..., pooling_task="token_classify")` for offline inference.
  **CN:** 离线推理使用 `LLM.encode(..., pooling_task="token_classify")`。
- **EN:** Online access is provided through the generic `/pooling` endpoint.
  **CN:** 在线访问通过通用 `/pooling` 端点提供。
- **EN:** Granularity is the key distinction from sequence classification.
  **CN:** 与序列分类相比，关键区别在于输出粒度。
