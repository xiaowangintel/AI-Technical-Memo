# gpt_oss.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/gpt_oss.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Please refer to https://github.com/sgl-project/sglang/issues/8833. Responses API & Built-in Tools Responses API GPT‑OSS is compatible with the OpenAI Responses API. / 该文档围绕 GPT OSS Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Please refer to https://github.com/sgl-project/sglang/issues/8833.
**CN:** 本节围绕 Overview 展开，概述了 Please, refer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Responses API
**EN:** GPT‑OSS is compatible with the OpenAI Responses API. Use client.responses.create(...) with model, instructions, input, and optional tools to enable built‑in tool use.
**CN:** 本节围绕 Responses API 展开，概述了 instructions, GPT, OSS, low 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Built-in Tools
**EN:** GPT‑OSS can call built‑in tools for web search and Python execution. You can use the demo tool server or connect to external MCP tool servers.
**CN:** 本节围绕 Built-in Tools 展开，概述了 Exa, EXA_API_KEY, GPT, OSS 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tool & Reasoning Parser
**EN:** We support OpenAI Reasoning and Tool Call parser, as well as our SGLang native api for tool call and reasoning. Refer to reasoning parser and tool parser for more details.
**CN:** 本节围绕 Tool & Reasoning Parser 展开，概述了 parser, tool, Refer, reasoning 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Notes
**EN:** Use **Python 3.12** for the demo tools. And install the required gpt-oss packages.
**CN:** 本节围绕 Notes 展开，概述了 Docker, demo, EXA_API_KEY, tool 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Speculative Decoding
**EN:** SGLang supports speculative decoding for GPT-OSS models using EAGLE3 algorithm. This can significantly improve decoding speed, especially for small batch sizes.
**CN:** 本节围绕 投机解码 展开，概述了 EAGLE3, decoding, Add, Usage 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Demo
**EN:** This section provides a comparison table for Quick Demo, covering columns such as Date (2025), Source, Key news points and examples such as **May 13**, **Aug 18**, **Aug 20**.
**CN:** 本节围绕 Quick Demo 展开，概述了 Nvidia, Aug, price, tool 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** tool / **CN:** tool
- **EN:** Nvidia / **CN:** Nvidia
- **EN:** tools / **CN:** tools
- **EN:** Aug / **CN:** Aug
- **EN:** price / **CN:** price
- **EN:** EAGLE3 / **CN:** EAGLE3
- **EN:** EXA_API_KEY / **CN:** EXA_API_KEY
- **EN:** model / **CN:** 模型

## Dependencies / 依赖关系
- `../advanced_features/separate_reasoning.ipynb`
- `../advanced_features/tool_parser.ipynb`
- `browser_server.py`
- `python_server.py`
