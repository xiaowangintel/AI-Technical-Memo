# openai_compatible_server.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/openai_compatible_server.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This is the main reference for vLLM's HTTP server that emulates multiple OpenAI-style APIs, including completions, chat, responses, embeddings, and more.
  **CN:** 这是 vLLM HTTP 服务器的主要参考文档，介绍其如何模拟多个 OpenAI 风格 API，包括 completions、chat、responses、embeddings 等。
- **EN:** The page is both a startup guide and an API reference: it covers `vllm serve`, endpoint-specific behavior, chat templates, extra parameters, and non-standard vLLM extensions.
  **CN:** 该页面既是启动指南也是 API 参考：涵盖 `vllm serve`、端点特定行为、chat template、额外参数以及 vLLM 的非标准扩展能力。

## Key Concepts / 关键概念
- **EN:** `vllm serve` launches the OpenAI-compatible HTTP interface.
  **CN:** `vllm serve` 用于启动 OpenAI 兼容 HTTP 接口。
- **EN:** Requests can use OpenAI-style payloads plus vLLM-specific extras through fields like `extra_body`.
  **CN:** 请求既可使用 OpenAI 风格负载，也可通过 `extra_body` 等字段传入 vLLM 特有参数。
- **EN:** The doc spans many endpoints, so it functions as a central serving reference.
  **CN:** 由于覆盖端点众多，该文档实际上是一个中心化服务参考页。
