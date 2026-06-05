# supported_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/supported_models.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page is the top-level catalog of model architectures supported by vLLM across both generative and pooling workloads.
  **CN:** 本页是 vLLM 支持模型架构的总目录，覆盖生成式与 pooling 两类工作负载。
- **EN:** It combines architecture coverage with implementation choices such as native vLLM support, Transformers fallback, and custom-model pathways.
  **CN:** 它将架构覆盖范围与实现方式结合起来，包括原生 vLLM 支持、Transformers 回退路径以及自定义模型接入方式。

## Key Concepts / 关键概念
- **EN:** Support is organized by model architecture rather than only by checkpoint name.
  **CN:** 支持情况按模型架构组织，而不只是按检查点名称。
- **EN:** Flags like `--model-impl transformers` and `--trust-remote-code` shape compatibility behavior.
  **CN:** `--model-impl transformers` 与 `--trust-remote-code` 等参数会影响兼容行为。
- **EN:** The page is the primary lookup table when evaluating whether a model can run on vLLM.
  **CN:** 在评估模型是否可在 vLLM 上运行时，该页面是首要查询表。
