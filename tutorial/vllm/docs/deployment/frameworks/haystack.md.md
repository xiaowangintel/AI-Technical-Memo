# haystack.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/haystack.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page demonstrates how to connect Haystack pipelines to vLLM through an OpenAI-compatible API. It covers package installation, server startup, and an example chat pipeline so users can plug vLLM into a broader retrieval or orchestration stack.
- **CN:** 该页面演示了如何通过 OpenAI 兼容 API 将 Haystack 流水线接入 vLLM。内容包括依赖安装、服务启动以及一个聊天流水线示例，使用户能够将 vLLM 接入更完整的检索或编排体系。

## Key Concepts / 关键概念
- **EN:** Haystack uses vLLM as a model-serving backend inside application pipelines. **CN:** Haystack 在应用流水线中把 vLLM 作为模型服务后端。
- **EN:** The example-driven approach helps verify that chat generation works end to end. **CN:** 示例驱动的写法有助于验证聊天生成的端到端可用性。
- **EN:** The document targets orchestration scenarios beyond bare API serving. **CN:** 该文档面向的不只是裸 API 服务，还包括更高层的编排场景。
