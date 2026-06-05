# multimodal.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/model/multimodal.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This is the most detailed multimodal integration guide in the set: it explains how to retrofit a basic model for multimodal embeddings, define processing metadata, generate worst-case dummy inputs, mirror Hugging Face processor behavior, and register the processor stack.
- **CN:** 这是整组文档中最细致的多模态接入指南：它解释了如何把基础模型改造成支持多模态 embedding、定义处理元数据、构造最坏情况 dummy 输入、复现 Hugging Face processor 行为，并完成整套处理器注册。
- **EN:** A major strength of the document is that it reasons from real HF implementations such as LLaVA and Fuyu to derive placeholder counts, prompt updates, and field schemas, which helps contributors translate external processor logic into vLLM abstractions.
- **CN:** 本文的重要价值在于它基于 LLaVA、Fuyu 等真实 HF 实现推导占位符数量、prompt 更新和字段模式，帮助贡献者把外部 processor 逻辑映射到 vLLM 抽象层。

## Key Concepts / 关键概念
- **EN:** `SupportsMultiModal` — The model interface marks a class as able to accept multimodal data and participate in the embedding pipeline.
- **CN:** `SupportsMultiModal` — 该接口用于标记模型能够接收多模态数据，并接入统一的 embedding 流程。
- **EN:** Processing information — `BaseProcessingInfo` captures modality limits and other facts the HF processor path depends on.
- **CN:** 处理信息 — `BaseProcessingInfo` 用于描述模态上限等 HF 处理流程所依赖的基础信息。
- **EN:** Dummy inputs for profiling — Dummy text and multimodal samples are chosen to approximate worst-case memory use.
- **CN:** 用于分析的 dummy 输入 — dummy 文本和多模态样本会按最坏内存占用场景来构造。
- **EN:** Prompt updates — Prompt replacement or insertion logic keeps token inputs consistent with what the HF processor would have produced.
- **CN:** Prompt 更新 — 通过替换或插入 token，使 token 输入与 HF processor 的结果保持一致。
- **EN:** Processor registration — The model, processing info, dummy builder, and processor class must be registered together in the multimodal registry.
- **CN:** 处理器注册 — 模型、处理信息、dummy 构造器和 processor 类需要一起注册到多模态注册表中。
