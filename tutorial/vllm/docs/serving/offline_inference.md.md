# offline_inference.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/offline_inference.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document explains programmatic offline inference with the `vllm.LLM` class instead of serving a model behind HTTP.
  **CN:** 该文档说明如何通过 `vllm.LLM` 类进行编程式离线推理，而不是通过 HTTP 服务模型。
- **EN:** It combines beginner-friendly examples with scaling pointers, including the difference between generative and pooling outputs and the Ray Data LLM API for large datasets.
  **CN:** 它把入门示例与扩展思路结合起来，既解释生成式与 pooling 输出的差异，也介绍面向大规模数据集的 Ray Data LLM API。

## Key Concepts / 关键概念
- **EN:** `LLM(...)` is the core entry point for offline inference.
  **CN:** `LLM(...)` 是离线推理的核心入口。
- **EN:** The page shows a minimal example using `facebook/opt-125m`.
  **CN:** 页面给出了使用 `facebook/opt-125m` 的最小示例。
- **EN:** Ray Data integration enables scalable offline execution over large datasets.
  **CN:** Ray Data 集成支持在大型数据集上扩展离线执行。
