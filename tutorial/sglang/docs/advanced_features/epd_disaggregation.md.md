# epd_disaggregation.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/epd_disaggregation.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Why and What is EPD Disaggregation? In modern Vision-Language Model (VLM) inference, request execution naturally decomposes into three distinct stages: Encoder, Prefill, and Decode. / 该文档围绕 EPD Disaggregation 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Why and What is EPD Disaggregation?
**EN:** In modern Vision-Language Model (VLM) inference, request execution naturally decomposes into three distinct stages: Encoder, Prefill, and Decode.
**CN:** 本节围绕 Why and What is EPD Disaggregation? 展开，概述了 Decode, Prefill, Encoder, EPD 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** You can launch a language-only model using --language-only, or an encoder-only model using --encoder-only. When launching a language-only model, you must additionally specify the encoder service endpoints via --encoder-urls.
**CN:** 本节围绕 Usage 展开，概述了 model, language-only, encoder, encoder-only 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Encoder transfer with Mooncake
**EN:** encoder-transfer-backend mooncake controls **how encoder outputs are transferred** between encoder and language/prefill services. It is an encoder transfer option and can be used independently of the global multimodal embedding cache.
**CN:** 本节围绕 Encoder transfer with Mooncake 展开，概述了 encoder, mooncake, encoder-transfer-backend, --encoder-transfer-backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Global multimodal embedding cache with Mooncake
**EN:** SGLang also supports a Mooncake-backed **global multimodal embedding cache** for EPD workloads. When enabled on encoder servers, repeated image inputs can reuse previously computed ViT embeddings across instances instead of running the vision encoder again.
**CN:** 本节围绕 Global multimodal embedding 缓存 with Mooncake 展开，概述了 Qwen, port, --port, encoder 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** encoder / **CN:** encoder
- **EN:** Mooncake / **CN:** Mooncake
- **EN:** --encoder-transfer-backend / **CN:** --encoder-transfer-backend
- **EN:** Qwen / **CN:** Qwen
- **EN:** port / **CN:** port
- **EN:** --port / **CN:** --port
- **EN:** model-path / **CN:** 模型-path
- **EN:** --model-path / **CN:** --模型-path

## Dependencies / 依赖关系
- `hicache_best_practices.md`
- `../../python/sglang/srt/mem_cache/storage/mooncake_store/README.md`
