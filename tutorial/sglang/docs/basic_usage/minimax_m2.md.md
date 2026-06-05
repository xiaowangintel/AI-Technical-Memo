# minimax_m2.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/minimax_m2.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: MiniMax-M2.5, MiniMax-M2.1, and MiniMax-M2 are advanced large language models created by MiniMax. The MiniMax-M2 series redefines efficiency for agents. / 该文档围绕 MiniMax M2.5/M2.1/M2 Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** MiniMax-M2.5, MiniMax-M2.1, and MiniMax-M2 are advanced large language models created by MiniMax. The MiniMax-M2 series redefines efficiency for agents.
**CN:** 本节围绕 Overview 展开，概述了 MiniMax-M2, models, billion, MiniMaxAI 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** This guide applies to the following models. You only need to update the model name during deployment.
**CN:** 本节围绕 Supported 模型s 展开，概述了 MiniMaxAI, MiniMax-M2, following, MiniMax-M2.5 等要点，并说明相关配置、流程、示例或限制条件。

### Section: System Requirements
**EN:** The following are recommended configurations; actual requirements should be adjusted based on your use case: - 4x 96GB GPUs: Supported context length of up to 400K tokens. 8x 144GB GPUs: Supported context length of up to 3M tokens.
**CN:** 本节围绕 System Requirements 展开，概述了 GPUs, Supported, length, tokens 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Deployment with Python
**EN:** This content focuses on Deployment with Python and highlights GPU, host, port, --host.
**CN:** 本节围绕 部署 with Python 展开，概述了 GPU, host, port, --host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: AMD GPUs (MI300X/MI325X/MI355X)
**EN:** This content focuses on AMD GPUs (MI300X/MI325X/MI355X) and highlights GPU, SGLANG_USE_AITER, host, port.
**CN:** 本节围绕 AMD GPUs (MI300X/MI325X/MI355X) 展开，概述了 GPU, SGLANG_USE_AITER, host, port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Testing Deployment
**EN:** After startup, you can test the SGLang OpenAI-compatible API with the following command: ``bash curl http://localhost:8000/v1/chat/completions \ -H "Content-Type: application/json" \ -d ' ]}, ]} ] }' ``
**CN:** 本节围绕 Testing 部署 展开，概述了 text, Who, role, type 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** MiniMaxAI / **CN:** MiniMaxAI
- **EN:** MiniMax-M2 / **CN:** MiniMax-M2
- **EN:** GPU / **CN:** GPU
- **EN:** MiniMax-M2.5 / **CN:** MiniMax-M2.5
- **EN:** GPUs / **CN:** GPUs
- **EN:** Supported / **CN:** Supported
- **EN:** deployment / **CN:** 部署
- **EN:** MiniMax-M2.1 / **CN:** MiniMax-M2.1

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
