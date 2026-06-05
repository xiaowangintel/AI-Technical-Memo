# deepseek_ocr.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/deepseek_ocr.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: DeepSeek OCR models are multimodal (image + text) models for OCR and document understanding. Launch server You can replace deepseek-ai/DeepSeek-OCR-2 with deepseek-ai/DeepSeek-OCR. / 该文档围绕 DeepSeek OCR (OCR-1 / OCR-2) 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** DeepSeek OCR models are multimodal (image + text) models for OCR and document understanding.
**CN:** 本节围绕 Overview 展开，概述了 OCR, models, text, image 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch server
**EN:** ``shell python -m sglang.launch_server \ --model-path deepseek-ai/DeepSeek-OCR-2 \ --trust-remote-code \ --host 0.0.0.0 \ --port 30000 ` > You can replace deepseek-ai/DeepSeek-OCR-2 with deepseek-ai/DeepSeek-OCR`.
**CN:** 本节围绕 Launch 服务端 展开，概述了 DeepSeek-OCR-2, deepseek-ai/DeepSeek-OCR-2, host, port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prompt examples
**EN:** ` ` <image> Free OCR.
**CN:** 本节围绕 Prompt examples 展开，概述了 image, Convert, Recommended, OCR 等要点，并说明相关配置、流程、示例或限制条件。

### Section: OpenAI-compatible request example
**EN:** ``python import requests url = "http://localhost:30000/v1/chat/completions" data = , }, ], } ], "max_tokens": 512, } response = requests.post(url, json=data) print(response.text) ``
**CN:** 本节围绕 OpenAI-compatible request example 展开，概述了 url, data, type, text 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** OCR / **CN:** OCR
- **EN:** image / **CN:** image
- **EN:** Convert / **CN:** Convert
- **EN:** url / **CN:** url
- **EN:** text / **CN:** text
- **EN:** document / **CN:** document
- **EN:** DeepSeek-OCR-2 / **CN:** DeepSeek-OCR-2
- **EN:** deepseek-ai/DeepSeek-OCR-2 / **CN:** deepseek-ai/DeepSeek-OCR-2

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
