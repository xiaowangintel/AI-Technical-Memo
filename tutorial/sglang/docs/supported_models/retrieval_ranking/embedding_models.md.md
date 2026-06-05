# embedding_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/retrieval_ranking/embedding_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang provides robust support for embedding models by integrating efficient serving mechanisms with its flexible programming interface. / 该文档围绕 Embedding 模型s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang provides robust support for embedding models by integrating efficient serving mechanisms with its flexible programming interface.
**CN:** 本节围绕 Overview 展开，概述了 embedding, models, may, flag 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Server
**EN:** ``shell python3 -m sglang.launch_server \ --model-path Qwen/Qwen3-Embedding-4B \ --is-embedding \ --host 0.0.0.0 \ --port 30000 ``
**CN:** 本节围绕 Launch 服务端 展开，概述了 Qwen, host, port, shell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Client Request
**EN:** ``python import requests url = "http://127.0.0.1:30000" payload = response = requests.post(url + "/v1/embeddings", json=payload).json() print("Embedding:", response["data"][0]["embedding"]) ``
**CN:** 本节围绕 Client Request 展开，概述了 url, France, payload, response 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multimodal Embedding Example
**EN:** This content focuses on Multimodal Embedding Example and highlights GME, url, text, image.
**CN:** 本节围绕 Multimodal Embedding Example 展开，概述了 GME, url, text, image 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Matryoshka Embedding Example
**EN:** Matryoshka Embeddings or Matryoshka Representation Learning (MRL) is a technique used in training embedding models. It allows user to trade off between performance and cost.
**CN:** 本节围绕 Matryoshka Embedding Example 展开，概述了 MRL, Matryoshka, off, user 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 1. Launch a Matryoshka‑capable model
**EN:** If the model config already includes matryoshka_dimensions or is_matryoshka then no override is needed.
**CN:** 本节围绕 1. Launch a Matryoshka‑capable 模型 展开，概述了 matryoshka_dimensions, Otherwise, is_matryoshka, --json-model-override-args 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 2. Make requests with different output dimensions
**EN:** ``python import requests url = "http://127.0.0.1:30000" # Request a truncated (Matryoshka) embedding by specifying a supported dimension.
**CN:** 本节围绕 2. Make requests with different output dimensions 展开，概述了 url, Request, Explain, payload 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** This section provides a comparison table for Supported Models, covering columns such as Model Family, Example Model, Chat Template and examples such as **E5 (Llama/Mistral based)**, **GTE-Qwen2**, **Qwen3-Embedding**, **BGE**.
**CN:** 本节围绕 Supported 模型s 展开，概述了 text, N/A, CLIP, Multimodal 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** embedding / **CN:** embedding
- **EN:** text / **CN:** text
- **EN:** model / **CN:** 模型
- **EN:** Matryoshka / **CN:** Matryoshka
- **EN:** url / **CN:** url
- **EN:** payload / **CN:** payload
- **EN:** N/A / **CN:** N/A
- **EN:** Qwen / **CN:** Qwen

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
