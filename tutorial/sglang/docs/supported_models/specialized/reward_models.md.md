# reward_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/specialized/reward_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: These models output a scalar reward score or classification result, often used in reinforcement learning or content moderation tasks. Example launch Command Supported models. / 该文档围绕 Reward 模型s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** These models output a scalar reward score or classification result, often used in reinforcement learning or content moderation tasks. `` They are executed with --is-embedding and some may require --trust-remote-code.
**CN:** 本节围绕 Overview 展开，概述了 These, may, score, often 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example launch Command
**EN:** ``shell python3 -m sglang.launch_server \ --model-path Qwen/Qwen2.5-Math-RM-72B \ # example HF/local path --is-embedding \ --host 0.0.0.0 \ --tp-size=4 \ # set for tensor parallelism --port 30000 \ ``
**CN:** 本节围绕 Example launch Command 展开，概述了 Qwen, path, host, port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported models
**EN:** This section provides a comparison table for Supported models, covering columns such as Model Family (Reward), Example HuggingFace Identifier, Description and examples such as **Llama (3.1 Reward / LlamaForSequenceClassification)**, **Gemma 2 (27B Reward / Gemma2ForSequenceClassification)**, **InternLM 2 (Reward / InternLM2ForRewardMode)**, **Qwen2.5 (Reward - Math / Qwen2ForRewardModel)**.
**CN:** 本节围绕 Supported 模型s 展开，概述了 Reward, RLHF, Qwen2.5, Llama 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Reward / **CN:** Reward
- **EN:** RLHF / **CN:** RLHF
- **EN:** Qwen2.5 / **CN:** Qwen2.5
- **EN:** Llama / **CN:** Llama
- **EN:** Gemma / **CN:** Gemma
- **EN:** model / **CN:** 模型
- **EN:** InternLM / **CN:** InternLM
- **EN:** scoring / **CN:** scoring

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
