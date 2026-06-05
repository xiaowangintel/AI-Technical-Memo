# qwen3_5.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/qwen3_5.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Qwen 3.5 is Alibaba's latest generation LLM featuring a hybrid attention architecture, advanced MoE with shared experts, and native multimodal capabilities. / 该文档围绕 Qwen 3.5 Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Qwen 3.5 is Alibaba's latest generation LLM featuring a hybrid attention architecture, advanced MoE with shared experts, and native multimodal capabilities.
**CN:** 本节围绕 Overview 展开，概述了 MoE, LLM, Key, Qwen 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Dense Model
**EN:** To serve Qwen/Qwen3.5-397B-A17B on 8 GPUs: ``bash python3 -m sglang.launch_server \ --model-path Qwen/Qwen3.5-397B-A17B \ --tp 8 \ --trust-remote-code ``
**CN:** 本节围绕 Dense 模型 展开，概述了 Qwen/Qwen3.5-397B-A17B, Qwen, GPUs, Qwen3.5-397B-A17B 等要点，并说明相关配置、流程、示例或限制条件。

### Section: AMD GPU (MI300X / MI325X / MI35X)
**EN:** On AMD Instinct GPUs, use the triton attention backend.
**CN:** 本节围绕 AMD GPU (MI300X / MI325X / MI35X) 展开，概述了 SGLANG_USE_AITER, AMD, triton, attention 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration Tips
**EN:** attention-backend: Use triton on AMD GPUs for Qwen 3.5. The hybrid attention architecture (Gated Delta Networks + full attention) works best with the Triton backend on ROCm.
**CN:** 本节围绕 配置 Tips 展开，概述了 Triton, attention, GDNAttnBackend, GDN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reasoning and Tool Calling
**EN:** Qwen 3.5 supports reasoning and tool calling via the Qwen3 parsers: ``bash python3 -m sglang.launch_server \ --model-path Qwen/Qwen3.5-397B-A17B \ --tp 8 \ --trust-remote-code \ --reasoning-parser qwen3 \ --tool-call-parser qwen3_coder ``
**CN:** 本节围绕 Reasoning and Tool Calling 展开，概述了 Qwen, Qwen3, via, --tp 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Accuracy Evaluation
**EN:** You can evaluate the model accuracy using lm-eval: ``bash pip install lm-eval[api] lm_eval --model local-completions \ --model_args ' ' \ --tasks gsm8k \ --batch_size auto \ --num_fewshot 5 \ --trust_remote_code ``
**CN:** 本节围绕 Accuracy Evaluation 展开，概述了 model, lm-eval, --model, pip 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Additional Resources
**EN:** AMD Day 0 Support for Qwen 3.5 on AMD Instinct GPUs - HuggingFace Model Card
**CN:** 本节围绕 Additional Resources 展开，概述了 Qwen, AMD, Support, Day 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Qwen / **CN:** Qwen
- **EN:** attention / **CN:** attention
- **EN:** AMD / **CN:** AMD
- **EN:** MoE / **CN:** MoE
- **EN:** Qwen3.5-397B-A17B / **CN:** Qwen3.5-397B-A17B
- **EN:** Qwen/Qwen3.5-397B-A17B / **CN:** Qwen/Qwen3.5-397B-A17B
- **EN:** GPUs / **CN:** GPUs
- **EN:** model / **CN:** 模型

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
