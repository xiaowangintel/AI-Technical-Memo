# retrieval_augmented_generation.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/retrieval_augmented_generation.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide explains retrieval-augmented generation (RAG) and shows two concrete stacks built on vLLM: LangChain + Milvus and LlamaIndex + Milvus. It combines conceptual framing with package installation and deployment steps so readers can build grounded generation pipelines.
- **CN:** 本指南解释了检索增强生成（RAG）的基本思想，并给出两套基于 vLLM 的具体方案：LangChain + Milvus，以及 LlamaIndex + Milvus。文档把概念说明与依赖安装、部署步骤结合起来，帮助读者构建有外部知识支撑的生成式流水线。

## Key Concepts / 关键概念
- **EN:** RAG augments model responses with retrieved documents instead of relying only on pretraining. **CN:** RAG 通过检索文档增强模型回答，而不只依赖预训练知识。
- **EN:** vLLM is the serving component inside a broader retrieval and orchestration stack. **CN:** vLLM 是更大检索与编排体系中的模型服务组件。
- **EN:** Milvus is used as the vector store across both illustrated integration patterns. **CN:** 在两种示例集成模式中，Milvus 都被用作向量数据库。
