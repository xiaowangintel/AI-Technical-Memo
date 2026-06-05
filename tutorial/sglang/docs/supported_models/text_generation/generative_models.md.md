# generative_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/text_generation/generative_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: These models accept text input and produce text output (e.g., chat completions). They are primarily large language models (LLMs), some with mixture-of-experts (MoE) architectures for scaling. / 该文档围绕 Large Language 模型s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** These models accept text input and produce text output (e.g., chat completions). They are primarily large language models (LLMs), some with mixture-of-experts (MoE) architectures for scaling.
**CN:** 本节围绕 Overview 展开，概述了 MoE, LLMs, text, These 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example launch Command
**EN:** ``shell python3 -m sglang.launch_server \ --model-path meta-llama/Llama-3.2-1B-Instruct \ # example HF/local path --host 0.0.0.0 \ --port 30000 \ ``
**CN:** 本节围绕 Example launch Command 展开，概述了 path, host, port, shell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported models
**EN:** This section provides a comparison table for Supported models, covering columns such as Model Family (Variants), Example HuggingFace Identifier, Description and examples such as **DeepSeek** (v1, v2, v3/R1), **Kimi K2** (Thinking, Instruct), **Kimi Linear** (48B-A3B), **GPT-OSS**.
**CN:** 本节围绕 Supported 模型s 展开，概述了 MoE, models, model, series 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** MoE / **CN:** MoE
- **EN:** models / **CN:** 模型s
- **EN:** model / **CN:** 模型
- **EN:** series / **CN:** series
- **EN:** NVIDIA / **CN:** NVIDIA
- **EN:** reasoning / **CN:** reasoning
- **EN:** LLM / **CN:** LLM
- **EN:** open / **CN:** open

## Dependencies / 依赖关系
- `../../basic_usage/deepseek_v3.md`
- `../../advanced_features/separate_reasoning.ipynb`
- `../../basic_usage/llama4.md`
