# litellm.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/litellm.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide shows how LiteLLM can sit in front of vLLM for both chat completions and embeddings. It explains that LiteLLM standardizes provider interactions, then demonstrates how to point LiteLLM requests at a hosted vLLM API base.
- **CN:** 本指南展示了 LiteLLM 如何位于 vLLM 之前，为聊天补全和向量嵌入提供统一调用层。文档先说明 LiteLLM 在多提供方之间的标准化价值，再演示如何把 LiteLLM 请求指向 vLLM 的 API 地址。

## Key Concepts / 关键概念
- **EN:** LiteLLM abstracts provider differences while vLLM supplies the actual model inference. **CN:** LiteLLM 抽象提供方差异，而 vLLM 负责实际模型推理。
- **EN:** The examples cover both text generation and embeddings, not only chat. **CN:** 示例同时覆盖文本生成和向量嵌入，而不只是聊天。
- **EN:** Router, fallback, and proxy features make LiteLLM relevant for gateway-style deployments. **CN:** 路由、回退和代理特性使 LiteLLM 适合网关式部署场景。
