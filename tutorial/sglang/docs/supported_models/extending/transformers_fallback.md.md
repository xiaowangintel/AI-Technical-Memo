# transformers_fallback.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/extending/transformers_fallback.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: sglang can fall back to using models that are available in transformers. This works for most decoder-style language models and support for vision-language models is coming soon!. / 该文档围绕 Transformers fallback in SGLang 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** sglang can fall back to using models that are available in transformers. This works for most decoder-style language models and support for vision-language models is coming soon!
**CN:** 本节围绕 Overview 展开，概述了 models, transformers, fall, back 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example launch Command
**EN:** By default, we will use sglang implementation if it is available. Otherwise, we will fall back to transformers one.
**CN:** 本节围绕 Example launch Command 展开，概述了 transformers, --model-impl, However, Otherwise 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quantization
**EN:** Transformers fall back has supported most of available quantization in SGLang (except GGUF). See Quantization page for more information about supported quantization in SGLang.
**CN:** 本节围绕 量化 展开，概述了 See, GGUF, supported, Transformers 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Remote code
**EN:** This fallback also means that any model on the hub that can be used in transformers with trust_remote_code=True that correctly implements attention can be used in production!
**CN:** 本节围绕 Remote code 展开，概述了 MyModel, ALL_ATTENTION_FUNCTIONS, PreTrainedModel, model 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** transformers / **CN:** transformers
- **EN:** MyModel / **CN:** My模型
- **EN:** ALL_ATTENTION_FUNCTIONS / **CN:** ALL_ATTENTION_FUNCTIONS
- **EN:** See / **CN:** See
- **EN:** PreTrainedModel / **CN:** PreTrained模型
- **EN:** fall / **CN:** fall
- **EN:** back / **CN:** back
- **EN:** model / **CN:** 模型

## Dependencies / 依赖关系
- `../../advanced_features/quantization.md`
