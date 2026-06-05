# codex.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/integrations/codex.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page explains how to connect OpenAI Codex to vLLM using the OpenAI Responses API implemented by vLLM.
  **CN:** 本页说明如何利用 vLLM 实现的 OpenAI Responses API 将 OpenAI Codex 连接到 vLLM。
- **EN:** The guide centers on a TOML configuration flow: start a tool-calling-capable vLLM server, point Codex to `base_url`, and ensure the configured model name matches the served name.
  **CN:** 该指南以 TOML 配置流程为中心：先启动具备 tool calling 能力的 vLLM 服务，再把 Codex 指向 `base_url`，并确保配置中的模型名与服务端名称一致。

## Key Concepts / 关键概念
- **EN:** `~/.codex/config.toml` is the main integration file.
  **CN:** `~/.codex/config.toml` 是主要的集成配置文件。
- **EN:** `wire_api = "responses"` aligns Codex with vLLM's supported API style.
  **CN:** `wire_api = "responses"` 让 Codex 与 vLLM 支持的 API 风格保持一致。
- **EN:** Dummy API-key environment variables are acceptable because vLLM does not require auth by default.
  **CN:** 由于 vLLM 默认不要求认证，因此可使用任意占位 API key 环境变量。
