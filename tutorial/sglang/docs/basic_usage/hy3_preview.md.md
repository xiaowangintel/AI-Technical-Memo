# hy3_preview.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/hy3_preview.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Hy3-preview is a large-scale language model (295B parameters, 21B active parameters) from Tencent Hunyuan team. SGLang supports serving Hy3-preview. / 该文档围绕 Hy3-preview Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Hy3-preview is a large-scale language model (295B parameters, 21B active parameters) from Tencent Hunyuan team. SGLang supports serving Hy3-preview.
**CN:** 本节围绕 Overview 展开，概述了 Hy3-preview, BF16, parameters, run 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Docker
**EN:** ``bash docker pull lmsysorg/sglang:hy3-preview ``
**CN:** 本节围绕 Docker 展开，概述了 pull, docker, lmsysorg/sglang:hy3-preview, bash
docker pull lmsysorg/sglang:hy3-preview
 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Build from Source
**EN:** ``bash # Install SGLang git clone https://github.com/sgl-project/sglang cd sglang pip3 install pip --upgrade pip3 install "transformers>=5.6.0" pip3 install -e "python" ``
**CN:** 本节围绕 Build from Source 展开，概述了 pip3, install, git, pip 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Hy3-preview with SGLang
**EN:** To serve the Hy3-preview model on 8 GPUs. On 8x96GB H20, SGLang can barely deploy the BF16 model and can only run small batch sizes or short requests.
**CN:** 本节围绕 Launch Hy3-preview with SGLang 展开，概述了 GPUs, Hy3-preview, model, H20 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE Speculative Decoding
**EN:** **Description**: SGLang supports Hy3-preview models with EAGLE speculative decoding. **Usage**: Add --speculative-algorithm, --speculative-num-steps, --speculative-eagle-topk, and --speculative-num-draft-tokens to enable this feature.
**CN:** 本节围绕 EAGLE 投机解码 展开，概述了 EAGLE, Hy3-preview, --speculative-algorithm, --speculative-num-steps 等要点，并说明相关配置、流程、示例或限制条件。

### Section: OpenAI Client Example
**EN:** First, install the OpenAI Python client: ``bash uv pip install -U openai ` You can use the OpenAI client as follows to verify thinking-mode responses.
**CN:** 本节围绕 OpenAI Client Example 展开，概述了 OpenAI, messages, content, Thinking 等要点，并说明相关配置、流程、示例或限制条件。

### Section: cURL Usage
**EN:** ``bash curl http://localhost:30000/v1/chat/completions \ -H "Content-Type: application/json" \ -d ' , ], "temperature": 1, "max_tokens": 4096 }' ``
**CN:** 本节围绕 cURL Usage 展开，概述了 role, Hello, content, Content-Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmarking Results
**EN:** For benchmarking, disable prefix caching by adding --disable-radix-cache to the server command. The following example runs the benchmark on 8 H20 GPUs with 96 GB memory each.
**CN:** 本节围绕 Benchmarking Results 展开，概述了 Total, ITL, token, throughput 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Hy3-preview / **CN:** Hy3-preview
- **EN:** Total / **CN:** Total
- **EN:** model / **CN:** 模型
- **EN:** OpenAI / **CN:** OpenAI
- **EN:** content / **CN:** content
- **EN:** messages / **CN:** messages
- **EN:** ITL / **CN:** ITL
- **EN:** GPUs / **CN:** GPUs

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
