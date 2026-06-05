# context_extension.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/context_extension.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This document explains how to extend a model's context length in vLLM, with emphasis on the newer `--hf-overrides` approach using `rope_parameters`. It explicitly notes that the older `--rope-scaling` flag is no longer supported.

**CN:** 本文说明如何在 vLLM 中扩展模型上下文长度，重点介绍通过 `rope_parameters` 配合 `--hf-overrides` 的新方法，并明确指出旧版 `--rope-scaling` 参数已不再支持。

**EN:** The structure covers two workflows: an offline Python example and an OpenAI-compatible serving setup. The online example shows a Qwen model configured with YARN-related RoPE parameters plus `--max-model-len`, then follows with a client-side chat request.

**CN:** 文档结构包含两条工作流：离线 Python 示例，以及兼容 OpenAI 的在线服务方式。在线示例展示了如何为 Qwen 模型设置基于 YARN 的 RoPE 参数和 `--max-model-len`，随后再通过客户端发起聊天请求。

**EN:** It also clarifies integration details: some parameters come from Hugging Face's RoPE definitions (`rope_type`, `factor`, `original_max_position_embeddings`), while `max_model_len` is vLLM-specific and affects KV-cache preallocation and serving-time request limits.

**CN:** 文档还澄清了集成边界：部分参数来自 Hugging Face 对 RoPE 的定义（如 `rope_type`、`factor`、`original_max_position_embeddings`），而 `max_model_len` 则是 vLLM 特有参数，会影响 KV cache 预分配和服务时的请求长度上限。

## Key Concepts / 关键概念
- **EN:** Prefer `--hf-overrides` with `rope_parameters` over deprecated flags. **CN:** 优先使用 `--hf-overrides` 与 `rope_parameters`，不要再用已弃用参数。
- **EN:** Supports both offline scripts and OpenAI-compatible serving. **CN:** 同时支持离线脚本和兼容 OpenAI 的服务模式。
- **EN:** YARN is presented as a concrete context-extension method. **CN:** 文档以 YARN 作为具体的上下文扩展示例。
- **EN:** `max_model_len` is a vLLM runtime control, not just a model config value. **CN:** `max_model_len` 是 vLLM 的运行时控制项，而不只是模型配置值。
