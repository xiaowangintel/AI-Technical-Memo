# dp_for_multi_modal_encoder.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/dp_for_multi_modal_encoder.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: A typical VLM architecture involves two main components: an multi-modal encoder and a text decoder. / 该文档围绕 DP for Multi-Modal Encoder in SGLang 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** A typical VLM architecture involves two main components: an multi-modal encoder and a text decoder.
**CN:** 本节围绕 Overview 展开，概述了 ViT, LLM, parallel, data 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Command Example
**EN:** You can enable batch-level DP by setting mm-enable-dp-encoder, for example: `` python3 -m sglang.launch_server \ --model-path Qwen/Qwen2.5-VL-7B-Instruct \ --tp 2 \ --mm-enable-dp-encoder ``
**CN:** 本节围绕 Command Example 展开，概述了 mm-enable-dp-encoder, --tp, Qwen, enable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Known supported models
**EN:** Qwen2.5-VL (<https://github.com/sgl-project/sglang/pull/13126>) - Qwen3-VL (<https://github.com/sgl-project/sglang/pull/13724>) - InternVL (<https://github.com/sgl-project/sglang/pull/13925>) - GLM-4.5V & GLM-4.6V (<https://github.com/sgl-project/sglang/pull/14097>)
**CN:** 本节围绕 Known supported 模型s 展开，概述了 Qwen3-VL, InternVL, GLM-4.5V, GLM-4.6V 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** ViT / **CN:** ViT
- **EN:** LLM / **CN:** LLM
- **EN:** parallel / **CN:** parallel
- **EN:** data / **CN:** data
- **EN:** mm-enable-dp-encoder / **CN:** mm-enable-dp-encoder
- **EN:** VLM / **CN:** VLM
- **EN:** GPU / **CN:** GPU
- **EN:** TTFT / **CN:** TTFT

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
