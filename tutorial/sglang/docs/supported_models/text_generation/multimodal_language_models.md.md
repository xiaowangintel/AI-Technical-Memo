# multimodal_language_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/text_generation/multimodal_language_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: These models accept multi-modal inputs (e.g., images and text) and generate text output. They augment language models with multimodal encoders. / 该文档围绕 Multimodal Language 模型s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** These models accept multi-modal inputs (e.g., images and text) and generate text output. They augment language models with multimodal encoders.
**CN:** 本节围绕 Overview 展开，概述了 text, These, models, e.g 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example launch Command
**EN:** ``shell python3 -m sglang.launch_server \ --model-path meta-llama/Llama-3.2-11B-Vision-Instruct \ # example HF/local path --host 0.0.0.0 \ --port 30000 \ `` > See the OpenAI APIs section for how to send multimodal requests.
**CN:** 本节围绕 Example launch Command 展开，概述了 See, path, host, port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported models
**EN:** This section provides a comparison table for Supported models, covering columns such as Model Family (Variants), Example HuggingFace Identifier, Description and examples such as **Qwen-VL**, **DeepSeek-VL2**, **DeepSeek-OCR / OCR-2**, **Janus-Pro** (1B, 7B).
**CN:** 本节围绕 Supported 模型s 展开，概述了 multimodal, model, image, text 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Audio Transcription
**EN:** SGLang supports audio-only ASR models via the OpenAI-compatible /v1/audio/transcriptions endpoint. Upload an audio file and receive a transcription.
**CN:** 本节围绕 Audio Transcription 展开，概述了 ASR, Upload, OpenAI-compatible, via 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Command
**EN:** ``shell sglang serve \ --model-path Qwen/Qwen3-ASR-1.7B \ --served-model-name qwen3-asr \ --trust-remote-code \ --host 0.0.0.0 --port 30000 ``
**CN:** 本节围绕 Launch Command 展开，概述了 Qwen, host, port, shell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example Request
**EN:** This section provides a comparison table for Example Request, covering columns such as Model Family, Example Identifier, Notes and examples such as **Whisper**, **Qwen3-ASR** (0.6B, 1.7B).
**CN:** 本节围绕 Example Request 展开，概述了 Notes, model, OpenAI, Whisper 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Video Input Support
**EN:** This section provides a comparison table for Video Input Support, covering columns such as Model Family, Example Identifier, Video notes and examples such as **Qwen-VL** (Qwen2-VL, Qwen2.5-VL, Qwen3-VL, Qwen3-Omni), **GLM-4v** (4.5V, 4.1V, MOE), **NVILA** (Full & Lite), **LLaVA video variants** (LLaVA-NeXT-Video, LLaVA-OneVision).
**CN:** 本节围绕 Video Input Support 展开，概述了 video, video_data, EVS, Qwen 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance Optimization
**EN:** This content focuses on Performance Optimization and highlights GPU, memory, --keep-mm-feature-on-device, latency.
**CN:** 本节围绕 性能 Optimization 展开，概述了 GPU, memory, --keep-mm-feature-on-device, latency 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multimodal Inputs Limitation
**EN:** **Use --mm-process-config ' ,"video": }'**: To set image, video, and audio input limits. This can reduce GPU memory usage, improve inference speed, and help to avoid OOM, but may impact model performance, thus set a proper value based on your specific use case.
**CN:** 本节围绕 Multimodal Inputs Limitation 展开，概述了 HuggingFace, GPU, OOM, Refer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Bidirectional Attention in Multimodal Model Serving
**EN:** **Note for serving the Gemma-3 multimodal model**: As mentioned in Welcome Gemma 3: Google's all new multimodal, multilingual, long context open LLM , Gemma-3 employs bidirectional attention between image tokens during the prefill phase.
**CN:** 本节围绕 Bidirectional Attention in Multimodal 模型 Serving 展开，概述了 attention, bidirectional, Graph, Gemma-3 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** model / **CN:** 模型
- **EN:** multimodal / **CN:** multimodal
- **EN:** text / **CN:** text
- **EN:** video / **CN:** video
- **EN:** Qwen / **CN:** Qwen
- **EN:** image / **CN:** image
- **EN:** models / **CN:** 模型s
- **EN:** GPU / **CN:** GPU

## Dependencies / 依赖关系
- `../../../python/sglang/srt/multimodal/evs/README.md`
