# openai_api.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/api/openai_api.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: The SGLang diffusion HTTP server implements an OpenAI-compatible API for image and video generation, as well as LoRA adapter management. Prerequisites Python 3.11+ if you plan to use the OpenAI Python SDK. / 该文档围绕 SGLang Diffusion OpenAI API 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** The SGLang diffusion HTTP server implements an OpenAI-compatible API for image and video generation, as well as LoRA adapter management.
**CN:** 本节围绕 Overview 展开，概述了 LoRA, API, well, image 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prerequisites
**EN:** Python 3.11+ if you plan to use the OpenAI Python SDK.
**CN:** 本节围绕 Prerequisites 展开，概述了 SDK, plan, OpenAI, OpenAI Python SDK 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Serve
**EN:** Launch the server using the sglang serve command.
**CN:** 本节围绕 Serve 展开，概述了 Launch, serve, server, command 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Start the server
**EN:** ``bash SERVER_ARGS=( --model-path Wan-AI/Wan2.1-T2V-1.3B-Diffusers --text-encoder-cpu-offload --pin-cpu-memory --num-gpus 4 --ulysses-degree=2 --ring-degree=2 --port 30010 ) sglang serve "$ " ` - **--model-path**: Path to the model or model ID.
**CN:** 本节围绕 Start the 服务端 展开，概述了 model, SERVER_ARGS, port, T2V 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Image Generation
**EN:** The server implements an OpenAI-compatible Images API under the /v1/images namespace. **Edit an image** **Endpoint:** POST /v1/images/edits This endpoint accepts a multipart form upload with input images and a text prompt.
**CN:** 本节围绕 Image Generation 展开，概述了 POST, URL, Bearer, Authorization 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Video Generation
**EN:** The server implements a subset of the OpenAI Videos API under the /v1/videos namespace.
**CN:** 本节围绕 Video Generation 展开，概述了 Bearer, Authorization, POST, Endpoint 等要点，并说明相关配置、流程、示例或限制条件。

### Section: LoRA Management
**EN:** The server supports dynamic loading, merging, and unmerging of LoRA adapters.
**CN:** 本节围绕 LoRA Management 展开，概述了 LoRA, POST, strength, target 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example: Switching LoRAs
**EN:** Set LoRA A: ``bash curl -X POST http://localhost:30010/v1/set_lora -d ' ' ` 2. Generate with LoRA A...
**CN:** 本节围绕 Example: Switching LoRAs 展开，概述了 LoRA, POST, Generate, curl 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Adjust Output Quality
**EN:** The server supports adjusting output quality and compression levels for both image and video generation through the output-quality and output-compression parameters.
**CN:** 本节围绕 Adjust Output Quality 展开，概述了 quality, output-quality, output-compression, compression 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** LoRA / **CN:** LoRA
- **EN:** POST / **CN:** POST
- **EN:** Endpoint / **CN:** Endpoint
- **EN:** curl / **CN:** curl
- **EN:** Bearer / **CN:** Bearer
- **EN:** Authorization / **CN:** Authorization
- **EN:** Content-Type / **CN:** Content-Type
- **EN:** strength / **CN:** strength

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
