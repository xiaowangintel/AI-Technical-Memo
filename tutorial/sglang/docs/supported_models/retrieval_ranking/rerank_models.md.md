# rerank_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/retrieval_ranking/rerank_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang offers comprehensive support for rerank models by incorporating optimized serving frameworks with a flexible programming interface. / 该文档围绕 Rerank 模型s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang offers comprehensive support for rerank models by incorporating optimized serving frameworks with a flexible programming interface.
**CN:** 本节围绕 Overview 展开，概述了 models, --is-embedding, rerank, run 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported rerank models
**EN:** This section provides a comparison table for Supported rerank models, covering columns such as Model Family (Rerank), Example HuggingFace Identifier, Chat Template and examples such as **BGE-Reranker (BgeRerankModel)**, **Qwen3-Reranker (decoder-only yes/no)**, **Qwen3-VL-Reranker (multimodal yes/no)**.
**CN:** 本节围绕 Supported rerank 模型s 展开，概述了 Launch, yes/no, --is-embedding, reranker 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Command
**EN:** ``shell python3 -m sglang.launch_server \ --model-path BAAI/bge-reranker-v2-m3 \ --host 0.0.0.0 \ --disable-radix-cache \ --chunked-prefill-size -1 \ --attention-backend triton \ --is-embedding \ --port 30000 ``
**CN:** 本节围绕 Launch Command 展开，概述了 BAAI, host, port, shell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example Client Request
**EN:** Defaults to returning all documents. If specified value is greater than the total number of documents, all documents will be returned.
**CN:** 本节围绕 Example Client Request 展开，概述了 documents, item, Score, panda 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Command
**EN:** ``shell python3 -m sglang.launch_server \ --model-path Qwen/Qwen3-Reranker-0.6B \ --trust-remote-code \ --disable-radix-cache \ --host 0.0.0.0 \ --port 8001 \ --chat-template examples/chat_template/qwen3_reranker.jinja ` ` Qwen3-Reranker uses decoder-only logprob scoring (yes/no).
**CN:** 本节围绕 Launch Command 展开，概述了 --is-embedding, Qwen3-Reranker, Qwen, host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example Client Request (supports optional instruct, top_n, and return_documents)
**EN:** Defaults to returning all documents. If specified value is greater than the total number of documents, all documents will be returned.
**CN:** 本节围绕 Example Client Request (supports optional instruct, top_n, and return_documents) 展开，概述了 documents, query, Defaults, required 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Response Format
**EN:** If top_n is not specified or is greater than the total number of documents, all documents are returned. Example (with return_documents: true): ``json [ , , ] ` Example (with return_documents: false): `json [ , , ] ` Example (with top_n: 2): `json [ , ] ``
**CN:** 本节围绕 Response Format 展开，概述了 score, document, top_n, documents 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Common Pitfalls
**EN:** **--chat-template is required.** Without --chat-template examples/chat_template/qwen3_reranker.jinja, the server does not recognize the model as a decoder-only reranker and returns a 400 error: "This model does not appear to be an embedding model by default.
**CN:** 本节围绕 Common Pitfalls 展开，概述了 --is-embedding, is-embedding, score, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-VL-Reranker (multimodal decoder-only rerank)
**EN:** Qwen3-VL-Reranker extends the Qwen3-Reranker to support multimodal content, allowing reranking of documents containing text, images, and videos.
**CN:** 本节围绕 Qwen3-VL-Reranker (multimodal decode 阶段r-only rerank) 展开，概述了 Qwen3-Reranker, Qwen3-VL-Reranker, text, images 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Command
**EN:** ``shell python3 -m sglang.launch_server \ --model-path Qwen/Qwen3-VL-Reranker-2B \ --trust-remote-code \ --disable-radix-cache \ --host 0.0.0.0 \ --port 30000 \ --chat-template examples/chat_template/qwen3_vl_reranker.jinja ` ` Qwen3-VL-Reranker uses decoder-only logprob scoring (yes/no) like Qwen3-Reranker.
**CN:** 本节围绕 Launch Command 展开，概述了 --is-embedding, Qwen3-Reranker, Qwen3-VL-Reranker, Qwen 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Text-Only Reranking (backward compatible)
**EN:** ``python import requests url = "http://127.0.0.1:30000/v1/rerank" payload = response = requests.post(url, json=payload) results = response.json() for item in results: print(f"Score: - ...") ``
**CN:** 本节围绕 Text-Only Reranking (backward compatible) 展开，概述了 learning, item, url, Deep 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Image Reranking (text query, image/mixed documents)
**EN:** This content focuses on Image Reranking (text query, image/mixed documents) and highlights Document, url, Text, image_url.
**CN:** 本节围绕 Image Reranking (text query, image/mixed documents) 展开，概述了 Document, url, Text, image_url 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multimodal Query Reranking (query with image)
**EN:** This content focuses on Multimodal Query Reranking (query with image) and highlights url, Find, image_url, text.
**CN:** 本节围绕 Multimodal Query Reranking (query with image) 展开，概述了 url, Find, image_url, text 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Request Parameters (Multimodal)
**EN:** This content focuses on Request Parameters (Multimodal) and highlights text, documents, type, query.
**CN:** 本节围绕 Request Parameters (Multimodal) 展开，概述了 text, documents, type, query 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Common Pitfalls
**EN:** Always use --chat-template examples/chat_template/qwen3_vl_reranker.jinja for Qwen3-VL-Reranker. Do NOT launch with --is-embedding.
**CN:** 本节围绕 Common Pitfalls 展开，概述了 Qwen3-VL-Reranker-2B, Note, Always, Currently 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** documents / **CN:** documents
- **EN:** score / **CN:** score
- **EN:** --is-embedding / **CN:** --is-embedding
- **EN:** query / **CN:** query
- **EN:** text / **CN:** text
- **EN:** return_documents / **CN:** return_documents
- **EN:** url / **CN:** url
- **EN:** item / **CN:** item

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
