# classify_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/retrieval_ranking/classify_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document describes the /v1/classify API endpoint implementation in SGLang, which is compatible with vLLM's classification API format. Overview The classification API allows you to classify text inputs using classification models. / 该文档围绕 Classification API 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document describes the /v1/classify API endpoint implementation in SGLang, which is compatible with vLLM's classification API format.
**CN:** 本节围绕 Overview 展开，概述了 API, vLLM, format, document 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** The classification API allows you to classify text inputs using classification models. This implementation follows the same format as vLLM's 0.7.0 classification API.
**CN:** 本节围绕 Overview 展开，概述了 API, classification, text, same 等要点，并说明相关配置、流程、示例或限制条件。

### Section: API Endpoint
**EN:** `` POST /v1/classify ``
**CN:** 本节围绕 API Endpoint 展开，概述了 POST, v1/classify, 
POST /v1/classify
 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Request Format
**EN:** This content focuses on Request Format and highlights text, model, input, classify.
**CN:** 本节围绕 Request Format 展开，概述了 text, model, input, classify 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Parameters
**EN:** model (string, required): The name of the classification model to use - input (string, required): The text to classify - user (string, optional): User identifier for tracking - rid (string, optional): Request ID for tracking - priority (integer, optional): Request priority
**CN:** 本节围绕 Parameters 展开，概述了 string, Request, model, priority 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Response Format
**EN:** ``json ], "usage": } ``
**CN:** 本节围绕 Response Format 展开，概述了 list, data, model, label 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Response Fields
**EN:** This content focuses on Response Fields and highlights Array, Total, Number, classification.
**CN:** 本节围绕 Response Fields 展开，概述了 Array, Total, Number, classification 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Using curl
**EN:** ``bash curl -v "http://127.0.0.1:8000/v1/classify" \ -H "Content-Type: application/json" \ -d ' ' ``
**CN:** 本节围绕 Using curl 展开，概述了 Loved, Content-Type, new, caf 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Using Python
**EN:** ``python import requests import json # Make classification request response = requests.post( "http://127.0.0.1:8000/v1/classify", headers= , json= ) # Parse response result = response.json() print(json.dumps(result, indent=2)) ``
**CN:** 本节围绕 Using Python 展开，概述了 Make, Loved, Parse, import 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** The classification API works with any classification model supported by SGLang, including:
**CN:** 本节围绕 Supported 模型s 展开，概述了 API, classification, works, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Classification Models (Multi-class)
**EN:** If id2label is not available, it falls back to LABEL_0, LABEL_1, etc., or Class_0, Class_1 as a last resort.
**CN:** 本节围绕 Classification 模型s (Multi-class) 展开，概述了 Multi-class, classification, id2label, LABEL_0 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reward Models (Single score)
**EN:** InternLM2ForRewardModel - Single reward score - Qwen2ForRewardModel - Single reward score - LlamaForSequenceClassificationWithNormal_Weights - Special reward model **Note**: The /classify endpoint in SGLang was originally designed for reward models but now supports all non-generative models.
**CN:** 本节围绕 Reward 模型s (Single score) 展开，概述了 Single, reward, Qwen2ForRewardModel, InternLM2ForRewardModel 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Error Handling
**EN:** The API returns appropriate HTTP status codes and error messages: - 400 Bad Request: Invalid request format or missing required fields - 500 Internal Server Error: Server-side processing error Error response format: ``json ``
**CN:** 本节围绕 Error Handling 展开，概述了 Error, format, Invalid, Server-side 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Implementation Details
**EN:** The classification API is implemented using: 1. **Rust Model Gateway**: Handles routing and request/response models in sgl-model-gateway/src/protocols/spec.rs 2.
**CN:** 本节围绕 Implementation Details 展开，概述了 Handles, API, Implements, classification 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Testing
**EN:** Use the provided test script to verify the implementation: ``bash python test_classify_api.py ``
**CN:** 本节围绕 Testing 展开，概述了 test, script, verify, provided 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Compatibility
**EN:** This implementation is compatible with vLLM's classification API format, allowing seamless migration from vLLM to SGLang for classification tasks.
**CN:** 本节围绕 Compatibility 展开，概述了 API, vLLM, classification, tasks 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** classification / **CN:** classification
- **EN:** API / **CN:** API
- **EN:** Multi-class / **CN:** Multi-class
- **EN:** model / **CN:** 模型
- **EN:** input / **CN:** input
- **EN:** Error / **CN:** Error
- **EN:** Single / **CN:** Single
- **EN:** Request / **CN:** Request

## Dependencies / 依赖关系
- `python/sglang/srt/entrypoints/http_server.py`
- `python/sglang/srt/entrypoints/openai/serving_classify.py`
- `test_classify_api.py`
