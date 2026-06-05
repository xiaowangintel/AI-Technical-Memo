# custom_chat_template.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/references/custom_chat_template.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: **NOTE**: There are two chat template systems in SGLang project. This document is about setting a custom chat template for the OpenAI-compatible API server (defined at conversation.py). / 该文档围绕 Custom Chat Template 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** **NOTE**: There are two chat template systems in SGLang project. This document is about setting a custom chat template for the OpenAI-compatible API server (defined at conversation.py).
**CN:** 本节围绕 Overview 展开，概述了 chat, template, server, NOTE 等要点，并说明相关配置、流程、示例或限制条件。

### Section: JSON Format
**EN:** This section provides a comparison table for JSON Format, covering columns such as "system": "<, im_start, >system", and examples such as "user": "<, "assistant": "<, "sep": "<, "stop_str": ["<.
**CN:** 本节围绕 JSON Format 展开，概述了 im_start, user, CHATML, system 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Jinja Format
**EN:** You can also use the Jinja template format as defined by Hugging Face Transformers. ``bash python -m sglang.launch_server \ --model-path meta-llama/Llama-2-7b-chat-hf \ --port 30000 \ --chat-template ./my_model_template.jinja ``
**CN:** 本节围绕 Jinja Format 展开，概述了 Jinja, Face, port, --port 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** template / **CN:** template
- **EN:** chat / **CN:** chat
- **EN:** defined / **CN:** defined
- **EN:** im_start / **CN:** im_start
- **EN:** port / **CN:** port
- **EN:** Jinja / **CN:** Jinja
- **EN:** --port / **CN:** --port
- **EN:** server / **CN:** 服务端

## Dependencies / 依赖关系
- `conversation.py`
- `//github.com/sgl-project/sglang/blob/main/python/sglang/srt/parser/conversation.py`
- `chat_template.py`
- `//github.com/sgl-project/sglang/blob/main/python/sglang/lang/chat_template.py`
