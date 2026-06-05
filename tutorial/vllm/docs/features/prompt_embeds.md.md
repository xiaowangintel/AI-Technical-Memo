# prompt_embeds.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/prompt_embeds.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This document explains how to feed prompt embeddings directly into vLLM instead of relying only on text-to-token-to-embedding conversion. It first defines prompt embeddings conceptually, then documents offline and online input paths.

**CN:** 本文介绍如何直接向 vLLM 传入 prompt embeddings，而不是只依赖“文本 -> token -> embedding”的传统流程。它先从概念层面解释 prompt embeddings，再说明离线和在线两种输入方式。

**EN:** For offline usage, the page points to `vllm.inputs.EmbedsPrompt` and requires a tensor shaped `(sequence_length, hidden_size)`, with linked examples using Hugging Face Transformers outputs. For online serving, support is gated by `--enable-prompt-embeds` and works with both Completions and Chat Completions APIs.

**CN:** 在离线场景中，文档指向 `vllm.inputs.EmbedsPrompt`，要求传入形状为 `(sequence_length, hidden_size)` 的张量，并链接了基于 Hugging Face Transformers 输出的示例。在线服务则需要通过 `--enable-prompt-embeds` 开启，并同时支持 Completions 与 Chat Completions API。

**EN:** The page spends meaningful effort on API semantics. In Completions, `prompt_embeds` is base64-encoded and no chat template is applied, so callers must embed the fully templated prompt themselves. In Chat Completions, embeddings are inserted as content parts among text segments, and the server applies the chat template around that content. A strong warning notes that malformed tensor shapes can crash the engine, so the feature should be exposed only to trusted users.

**CN:** 文档对 API 语义做了较细致说明。在 Completions 中，`prompt_embeds` 需要以 base64 编码传输，且不会自动套用 chat template，因此调用方必须自行对完整模板化提示词进行 embedding。在 Chat Completions 中，embedding 作为消息内容片段与文本交错出现，由服务端在模板渲染时插入。文档还给出强警告：错误的张量形状可能导致引擎崩溃，因此该功能只应向可信用户开放。

## Key Concepts / 关键概念
- **EN:** Direct embedding inputs bypass standard text tokenization flow. **CN:** 直接输入 embedding 可绕过标准文本分词流程。
- **EN:** Offline tensors must match `(sequence_length, hidden_size)`. **CN:** 离线张量必须满足 `(sequence_length, hidden_size)` 形状要求。
- **EN:** Online support requires `--enable-prompt-embeds`. **CN:** 在线支持需要开启 `--enable-prompt-embeds`。
- **EN:** Completions and Chat Completions handle templating differently. **CN:** Completions 与 Chat Completions 对模板处理方式不同。
- **EN:** Incorrect embedding shapes are a stability and trust boundary risk. **CN:** 错误 embedding 形状会带来稳定性与信任边界风险。
