# prompt_embeds.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/prompt_embeds.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document explains *prompt embedding inputs* — a feature that allows users to bypass the tokenizer and inject pre-computed dense embeddings directly as model input. It covers offline inference, online serving via Completions and Chat Completions APIs, and an important security warning.

**CN:** 本文档介绍*提示词嵌入输入*功能，允许用户绕过分词器，直接将预计算的稠密嵌入作为模型输入注入。内容涵盖离线推理、通过 Completions 和 Chat Completions API 进行在线服务，以及重要的安全警告。

### Background / 背景

**EN:** Standard LLM input goes: text → token IDs (tokenizer) → embeddings (lookup table). Prompt embeds let users provide the final embedding tensor directly, enabling use of custom embeddings, embeddings from other models, or embeddings for tokens outside the vocabulary.

**CN:** 标准 LLM 输入流程：文本→token ID（分词器）→嵌入（查找表）。提示词嵌入让用户直接提供最终的嵌入张量，可使用自定义嵌入、来自其他模型的嵌入，或词表外 token 的嵌入。

### Offline Inference / 离线推理

**EN:** Use `vllm.inputs.EmbedsPrompt` with field `prompt_embeds` — a torch tensor of shape `(sequence_length, hidden_size)`. Compatible with HuggingFace Transformers input embeddings.

**CN:** 使用 `vllm.inputs.EmbedsPrompt` 及字段 `prompt_embeds`——形状为 `(sequence_length, hidden_size)` 的 torch 张量，与 HuggingFace Transformers 输入嵌入兼容。

### Online Completions API / 在线 Completions API

**EN:** Add `prompt_embeds` key in the JSON body with a base64-encoded torch tensor. Can be mixed with plain `prompt` text (embeds are returned first). The caller is responsible for applying the chat template before encoding — the server does NOT apply any template to `prompt_embeds`.

**CN:** 在 JSON 请求体中添加 `prompt_embeds` 键，值为 base64 编码的 torch 张量，可与普通 `prompt` 文本混合（嵌入始终优先返回）。调用方负责在编码前应用对话模板，服务端不对 `prompt_embeds` 应用任何模板。

### Online Chat Completions API / 在线 Chat Completions API

**EN:** Include `{"type": "prompt_embeds", "data": "<base64>"}` content parts in messages. Each part encodes only the content (not a full templated conversation) — the server wraps the chat template around it. Multiple parts can appear in any message at any position relative to text parts.

**CN:** 在消息中包含 `{"type": "prompt_embeds", "data": "<base64>"}` 内容部件。每个部件仅编码内容本身（而非完整的模板对话），服务端会将对话模板包裹在其外层。多个部件可出现在任意消息的任意位置（与文本部件的相对位置不受限制）。

### Security Warning / 安全警告

**EN:** Enable with `--enable-prompt-embeds` flag only. Incorrect embedding shapes may crash the engine. Only enable for trusted users.

**CN:** 仅通过 `--enable-prompt-embeds` 标志启用。嵌入形状不正确可能导致引擎崩溃，仅对可信用户启用。

## Key Concepts / 关键概念

- **EN:** `EmbedsPrompt` / `prompt_embeds` — the primary interface; shape `(seq_len, hidden_size)`. **CN:** `EmbedsPrompt` / `prompt_embeds`——主要接口，形状为 `(seq_len, hidden_size)`。
- **EN:** Base64-encoded tensor — the wire format for prompt embeddings in both Completions and Chat Completions APIs. **CN:** Base64 编码张量——提示词嵌入在 Completions 和 Chat Completions API 中的传输格式。
- **EN:** Template responsibility — Completions API: caller must embed the fully-templated conversation. Chat API: server applies the template, so embed only the content. **CN:** 模板责任——Completions API：调用方需嵌入已完整模板化的对话；Chat API：服务端应用模板，只需嵌入内容本身。
- **EN:** `--enable-prompt-embeds` — server flag required to activate online prompt embedding support. **CN:** `--enable-prompt-embeds`——激活在线提示词嵌入支持所需的服务端标志。
- **EN:** Mixing text and embeds — in Completions API, both `prompt` and `prompt_embeds` can appear in the same request; embeds are prepended. **CN:** 混合文本与嵌入——在 Completions API 中，`prompt` 和 `prompt_embeds` 可在同一请求中共存，嵌入始终在前。
