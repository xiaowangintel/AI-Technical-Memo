# transcription.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/model/transcription.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This guide explains how to add ASR or translation support by implementing `SupportsTranscription`, including capability declaration, API behavior configuration, prompt construction, and optional language/token helpers.
- **CN:** 本文说明如何通过实现 `SupportsTranscription` 来为模型增加语音识别或翻译支持，涵盖能力声明、API 行为配置、prompt 构造以及可选的语言和 token 辅助接口。
- **EN:** It also documents server-side audio preprocessing and chunking, automatic task exposure, in-tree examples, and endpoint-level curl tests, so it functions as both an interface reference and an integration checklist.
- **CN:** 文档还覆盖了服务端音频预处理与切块、任务自动暴露、仓库内示例以及 curl 级别接口测试，因此既是接口说明，也是接入检查清单。

## Key Concepts / 关键概念
- **EN:** `SupportsTranscription` — Implementing this interface is the core requirement for transcription and translation support.
- **CN:** `SupportsTranscription` — 实现该接口是支持转写和翻译能力的核心前提。
- **EN:** `SpeechToTextConfig` — This configuration controls sampling rate, chunk size, and server-side chunking behavior.
- **CN:** `SpeechToTextConfig` — 该配置控制采样率、切块大小以及服务端切块行为。
- **EN:** Prompt patterns — The doc distinguishes multimodal LLM prompts from encoder-decoder audio-only prompts.
- **CN:** Prompt 模式 — 文档区分了多模态 LLM 的 prompt 构造方式与纯音频 encoder-decoder 的方式。
- **EN:** Audio chunking — The server can resample audio and split long clips, optionally using low-energy boundaries.
- **CN:** 音频切块 — 服务端可以对音频重采样，并在长音频上切块，还可利用低能量区间减少切词错误。
- **EN:** Automatic task exposure — Once the model is registered and implements the interface, vLLM exposes transcription endpoints automatically.
- **CN:** 任务自动暴露 — 模型一旦完成注册并实现该接口，vLLM 会自动暴露对应的转写接口。
