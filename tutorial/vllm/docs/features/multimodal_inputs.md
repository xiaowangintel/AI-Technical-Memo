# multimodal_inputs.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/multimodal_inputs.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document teaches how to pass multi-modal inputs (images, video, audio, embeddings) to vLLM's multi-modal language models. It covers offline inference, online (OpenAI-compatible) serving, and security guidance for production deployments.

**CN:** 本文档介绍如何向 vLLM 的多模态语言模型传入多模态输入（图像、视频、音频、嵌入），内容涵盖离线推理、在线（OpenAI 兼容）服务，以及面向生产部署的安全指导。

### Offline Inference Schema / 离线推理模式

**EN:** Use `vllm.inputs.PromptType` with a `multi_modal_data` dict. Supported keys include `image` (single PIL image or list), `video` (list of PIL frames or URL), `audio` (tuple of numpy array + sample rate), and `embeds`. Prompts should follow the HuggingFace format for the specific model.

**CN:** 使用 `vllm.inputs.PromptType`，配合 `multi_modal_data` 字典。支持的键包括 `image`（单张 PIL 图像或列表）、`video`（PIL 帧列表或 URL）、`audio`（numpy 数组 + 采样率元组）和 `embeds`（嵌入）。提示词应遵循该模型的 HuggingFace 格式。

### `LLM.chat` Method / `LLM.chat` 方法

**EN:** Supports image URLs, PIL Image objects, and pre-computed embeddings directly in message content using typed dict entries (`image_url`, `image_pil`, `image_embeds`).

**CN:** 支持在消息内容中直接使用类型化字典条目（`image_url`、`image_pil`、`image_embeds`）传入图像 URL、PIL 图像对象和预计算嵌入。

### Online Serving / 在线服务

**EN:** OpenAI-compatible API accepts images as base64 data or URLs. For chat, use the standard `image_url` content type. The server also accepts multi-modal embeddings via the prompt embeddings feature.

**CN:** OpenAI 兼容 API 支持以 base64 数据或 URL 形式传入图像。在对话场景中，使用标准 `image_url` 内容类型，服务端还可通过提示词嵌入功能接受多模态嵌入。

### Security / 安全

**EN:** Use `--allowed-media-domains` to restrict which domains vLLM can fetch media from (prevents SSRF). Set `VLLM_MEDIA_URL_ALLOW_REDIRECTS=0` to block redirect-based bypass. Critical in containerised environments with access to internal networks.

**CN:** 使用 `--allowed-media-domains` 限制 vLLM 可获取媒体的域名（防止 SSRF）。设置 `VLLM_MEDIA_URL_ALLOW_REDIRECTS=0` 阻止基于重定向的绕过攻击。在容器化环境中尤为重要，因为 vLLM Pod 可能可以访问内部网络。

### `limit_mm_per_prompt` / 每提示词媒体限制

**EN:** Use `limit_mm_per_prompt={"image": N}` to cap the number of media items accepted per prompt, preventing resource abuse.

**CN:** 使用 `limit_mm_per_prompt={"image": N}` 限制每个提示词接受的媒体条目数量，防止资源滥用。

## Key Concepts / 关键概念

- **EN:** `multi_modal_data` dict — the main interface for offline multi-modal input; maps modality name to data. **CN:** `multi_modal_data` 字典——离线多模态输入的主要接口，将模态名称映射到数据。
- **EN:** `MultiModalDataDict` — the schema definition for valid `multi_modal_data` keys and value types. **CN:** `MultiModalDataDict`——定义有效 `multi_modal_data` 键和值类型的模式。
- **EN:** Multi-image support — pass a list to the `image` key and set `limit_mm_per_prompt` accordingly. **CN:** 多图像支持——向 `image` 键传入列表，并相应设置 `limit_mm_per_prompt`。
- **EN:** SSRF protection — `--allowed-media-domains` and redirect blocking are essential for deployments in networked environments. **CN:** SSRF 防护——`--allowed-media-domains` 和禁止重定向是联网环境部署的必要安全措施。
- **EN:** Active development — multi-modal API is evolving; track RFC #4194 for upcoming changes. **CN:** 持续演进——多模态 API 仍在积极开发，跟踪 RFC #4194 了解即将到来的变更。
