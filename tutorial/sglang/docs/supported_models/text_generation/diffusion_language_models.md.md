# diffusion_language_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/text_generation/diffusion_language_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Diffusion language models have shown promise for non-autoregressive text generation with parallel decoding capabilities. Unlike auto-regressive language models, different diffusion language models require different decoding strategies. / 该文档围绕 Diffusion Language 模型s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Diffusion language models have shown promise for non-autoregressive text generation with parallel decoding capabilities. Unlike auto-regressive language models, different diffusion language models require different decoding strategies.
**CN:** 本节围绕 Overview 展开，概述了 models, language, Unlike, decoding 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example Launch Command
**EN:** SGLang supports different DLLM algorithms such as LowConfidence and JointThreshold.
**CN:** 本节围绕 Example Launch Command 展开，概述了 LowConfidence, JointThreshold, DLLM, path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example Configuration File
**EN:** Depending on the algorithm selected, the configuration parameters vary. edit_threshold: 0.0 # Max extra T2T steps after all masks are removed.
**CN:** 本节围绕 Example 配置 File 展开，概述了 Range, threshold, T2T, but 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example Client Code Snippet
**EN:** This section provides a comparison table for Example Client Code Snippet, covering columns such as "<role>SYSTEM</role>detailed thinking off<, role_end, ><role>HUMAN</role> Write a brief introduction of the great wall < and examples such as "<role>SYSTEM</role>detailed thinking off<, "<role>SYSTEM</role>detailed thinking off<.
**CN:** 本节围绕 Example Client Code Snippet 展开，概述了 role, HUMAN, Write, SYSTEM 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** This section provides a comparison table for Supported Models, covering columns such as Model Family, Example Model, Description and examples such as **LLaDA2.0 (mini, flash)**, **SDAR (JetLM)**, **SDAR (JetLM)**.
**CN:** 本节围绕 Supported 模型s 展开，概述了 SDAR, JetLM, MoE, Chat 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** role / **CN:** role
- **EN:** SDAR / **CN:** SDAR
- **EN:** LowConfidence / **CN:** LowConfidence
- **EN:** language / **CN:** language
- **EN:** Range / **CN:** Range
- **EN:** HUMAN / **CN:** HUMAN
- **EN:** Write / **CN:** Write
- **EN:** JetLM / **CN:** JetLM

## Dependencies / 依赖关系
- `./config.yaml`
