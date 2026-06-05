# multimodal_inputs.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/multimodal_inputs.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page is a broad integration guide for passing multimodal inputs into vLLM, covering offline inference and OpenAI-compatible online serving across image, video, audio, embedding, and cached-input workflows.
  **CN:** 本文是一份较全面的多模态接入指南，覆盖在 vLLM 中传入多模态输入的离线推理与 OpenAI 兼容在线服务，包括图像、视频、音频、embedding 与缓存输入等流程。
- **EN:** The offline section defines the core request schema (`prompt` plus `multi_modal_data`) and then walks modality by modality through concrete examples such as single/multi-image prompts, chat-style image content, video tensors, audio tuples, and precomputed embeddings.
  **CN:** 离线部分先定义核心请求结构（`prompt` 与 `multi_modal_data`），随后按模态逐一给出示例，包括单图/多图 prompt、chat 风格图像内容、视频张量、音频二元组以及预计算 embedding。
- **EN:** Several advanced media-processing details are documented, including RGBA background replacement, Moondream3 prompt recipes, long-audio chunking with quiet-point splitting, and automatic mono conversion for supported audio models.
  **CN:** 文档还记录了若干高级媒体处理细节，如 RGBA 背景色替换、Moondream3 专用 prompt 模板、基于静音点的长音频切分，以及对受支持音频模型的自动单声道归一化。
- **EN:** Caching is treated as an explicit workflow: users may provide `multi_modal_uuids` for stable media identities, reuse cached media across requests, and even omit raw media when cache hits are expected.
  **CN:** 缓存被当作一个明确工作流来说明：用户可提供 `multi_modal_uuids` 作为稳定媒体标识，在请求间复用缓存，甚至在预期命中缓存时省略原始媒体数据。
- **EN:** The online serving half explains how the Chat Completions API represents media (`image_url`, `video_url`, `input_audio`, embedding payloads), emphasizes that chat templates are mandatory, and shows server-side knobs such as allowed local paths, fetch timeouts, frame recovery, and `media_io_kwargs` metadata for client-extracted video frames.
  **CN:** 在线服务部分说明了 Chat Completions API 如何表示媒体（`image_url`、`video_url`、`input_audio`、embedding 载荷），强调 chat template 是必需项，并展示了服务端控制项，如允许的本地路径、抓取超时、视频帧恢复，以及向客户端提取的视频帧传递 `media_io_kwargs` 元数据。
- **EN:** The document repeatedly highlights operational and security caveats: multimodal support is still evolving, remote media access should be domain-restricted to reduce SSRF risk, redirect following can be disabled, and embedding input should only be enabled for trusted users because malformed shapes may crash the engine.
  **CN:** 文档多次强调运维与安全注意事项：多模态支持仍在快速迭代，应限制远程媒体访问域名以降低 SSRF 风险，可关闭重定向跟随；而 embedding 输入仅应对可信用户开放，因为错误形状可能导致引擎崩溃。

## Key Concepts / 关键概念
- **EN:** Core offline schema: `prompt` + `multi_modal_data`.  
  **CN:** 核心离线结构：`prompt` + `multi_modal_data`。
- **EN:** Supported modalities span image, video, audio, and precomputed embeddings.  
  **CN:** 支持的模态涵盖图像、视频、音频和预计算 embedding。
- **EN:** `multi_modal_uuids` enable stable caching and optional omission of already-cached media.  
  **CN:** `multi_modal_uuids` 支持稳定缓存，并允许省略已缓存媒体。
- **EN:** Online multimodal use depends on chat templates and OpenAI-style content blocks.  
  **CN:** 在线多模态使用依赖 chat template 与 OpenAI 风格内容块。
- **EN:** `media_io_kwargs` carries processing controls such as RGBA background color, frame recovery, or preserved video metadata.  
  **CN:** `media_io_kwargs` 可传递处理控制信息，如 RGBA 背景色、帧恢复或保留的视频元数据。
- **EN:** Security posture matters: restrict media domains, control local file access, and treat embedding inputs as trusted-only.  
  **CN:** 安全策略很重要：限制媒体域名、控制本地文件访问，并将 embedding 输入视为仅限可信用户的能力。
