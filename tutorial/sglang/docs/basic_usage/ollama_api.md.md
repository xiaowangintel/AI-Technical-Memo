# ollama_api.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/ollama_api.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang provides Ollama API compatibility, allowing you to use the Ollama CLI and Python library with SGLang as the inference backend. Prerequisites **Note**: You don't need the Ollama server installed - SGLang acts as the backend. / 该文档围绕 Ollama-Compatible API 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang provides Ollama API compatibility, allowing you to use the Ollama CLI and Python library with SGLang as the inference backend.
**CN:** 本节围绕 Overview 展开，概述了 Ollama, API, CLI, library 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prerequisites
**EN:** ``bash # Install the Ollama Python library (for Python client usage) pip install ollama ` > **Note**: You don't need the Ollama server installed - SGLang acts as the backend. You only need the ollama` CLI or Python library as the client.
**CN:** 本节围绕 Prerequisites 展开，概述了 ollama, CLI, Note, need 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Endpoints
**EN:** This section provides a comparison table for Endpoints, covering columns such as Endpoint, Method, Description and examples such as /, /api/tags, /api/chat, /api/generate.
**CN:** 本节围绕 Endpoints 展开，概述了 POST, HEAD, List, Chat 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 1. Launch SGLang Server
**EN:** ``bash python -m sglang.launch_server \ --model Qwen/Qwen2.5-1.5B-Instruct \ --port 30001 \ --host 0.0.0.0 ` > **Note**: The model name used with ollama run must match exactly what you passed to --model`.
**CN:** 本节围绕 1. Launch SGLang 服务端 展开，概述了 model, Note, --model, run 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 2. Use Ollama CLI
**EN:** This content focuses on 2. Use Ollama CLI and highlights OLLAMA_HOST, ollama, SSH, List.
**CN:** 本节围绕 2. Use Ollama CLI 展开，概述了 OLLAMA_HOST, ollama, SSH, List 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 3. Use Ollama Python Library
**EN:** This content focuses on 3. Use Ollama Python Library and highlights content, stream, Qwen, Tell.
**CN:** 本节围绕 3. Use Ollama Python Library 展开，概述了 content, stream, Qwen, Tell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Smart Router
**EN:** For intelligent routing between local Ollama (fast) and remote SGLang (powerful) using an LLM judge, see the Smart Router documentation.
**CN:** 本节围绕 Smart Router 展开，概述了 LLM, Ollama, see, fast 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Summary
**EN:** This section provides a comparison table for Summary, covering columns such as Component, Purpose and examples such as **Ollama API**, **SGLang Backend**, **Smart Router**.
**CN:** 本节围绕 Summary 展开，概述了 API, tasks, Purpose, Component 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Ollama / **CN:** Ollama
- **EN:** CLI / **CN:** CLI
- **EN:** POST / **CN:** POST
- **EN:** OLLAMA_HOST / **CN:** OLLAMA_HOST
- **EN:** model / **CN:** 模型
- **EN:** API / **CN:** API
- **EN:** Note / **CN:** Note
- **EN:** List / **CN:** List

## Dependencies / 依赖关系
- `//github.com/sgl-project/sglang/blob/main/python/sglang/srt/entrypoints/ollama/README.md`
