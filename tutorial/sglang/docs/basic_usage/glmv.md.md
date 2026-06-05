# glmv.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/glmv.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Launch commands for SGLang Below are suggested launch commands tailored for different hardware / precision modes FP8 (quantised) mode For high memory-efficiency and latency optimized deployments (e.g., on H100, H200) where FP8 checkpoint is supported:. / 该文档围绕 GLM-4.6V / GLM-4.5V Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Launch commands for SGLang
**EN:** Below are suggested launch commands tailored for different hardware / precision modes
**CN:** 本节围绕 Launch commands for SGLang 展开，概述了 Below, modes, launch, commands 等要点，并说明相关配置、流程、示例或限制条件。

### Section: FP8 (quantised) mode
**EN:** For high memory-efficiency and latency optimized deployments (e.g., on H100, H200) where FP8 checkpoint is supported: ``bash python3 -m sglang.launch_server \ --model-path zai-org/GLM-4.6V-FP8 \ --tp 2 \ --ep 2 \ --host 0.0.0.0 \ --port 30000 \ --keep-mm-feature-on-device ``
**CN:** 本节围绕 FP8 (quantised) mode 展开，概述了 FP8, H100, H200, e.g 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Non-FP8 (BF16 / full precision) mode
**EN:** For deployments on A100/H100 where BF16 is used (or FP8 snapshot not used): ``bash python3 -m sglang.launch_server \ --model-path zai-org/GLM-4.6V \ --tp 4 \ --ep 4 \ --host 0.0.0.0 \ --port 30000 ``
**CN:** 本节围绕 Non-FP8 (BF16 / full precision) mode 展开，概述了 FP8, BF16, --tp, --ep 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hardware-specific notes / recommendations
**EN:** On H100 with FP8: Use the FP8 checkpoint for best memory efficiency. On A100 / H100 with BF16 (non-FP8): It’s recommended to use --mm-max-concurrent-calls to control parallel throughput and GPU memory usage during image/video inference.
**CN:** 本节围绕 Hardware-specific notes / recommendations 展开，概述了 FP8, H100, GPU, A100 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Image input:
**EN:** ``python import requests url = f"http://localhost:30000/v1/chat/completions" data = , , }, ], } ], "max_tokens": 300, } response = requests.post(url, json=data) print(response.text) ``
**CN:** 本节围绕 Image input: 展开，概述了 url, data, type, text 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Video Input:
**EN:** ``python import requests url = f"http://localhost:30000/v1/chat/completions" data = , , }, ], } ], "max_tokens": 300, } response = requests.post(url, json=data) print(response.text) ``
**CN:** 本节围绕 Video Input: 展开，概述了 url, data, type, text 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Important Server Parameters and Flags
**EN:** When launching the model server for **multimodal support**, you can use the following command-line arguments to fine-tune performance and behavior: - --mm-attention-backend: Specify multimodal attention backend.
**CN:** 本节围绕 Important 服务端 Parameters and Flags 展开，概述了 multimodal, GPU, data, server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example usage with the above optimizations:
**EN:** This content focuses on Example usage with the above optimizations: and highlights fa3, SGLANG_VLM_CACHE_SIZE_MB, SGLANG_USE_CUDA_IPC_TRANSPORT, host.
**CN:** 本节围绕 Example usage with the above optimizations: 展开，概述了 fa3, SGLANG_VLM_CACHE_SIZE_MB, SGLANG_USE_CUDA_IPC_TRANSPORT, host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Thinking Budget for GLM-4.5V / GLM-4.6V
**EN:** In SGLang, we can implement thinking budget with CustomLogitProcessor. Launch a server with the --enable-custom-logit-processor flag.
**CN:** 本节围绕 Thinking Budget for GLM-4.5V / GLM-4.6V 展开，概述了 GLM-4.6, CustomLogitProcessor, Glm4MoeThinkingBudgetLogitProcessor, Then 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** FP8 / **CN:** FP8
- **EN:** GLM-4.6V / **CN:** GLM-4.6V
- **EN:** H100 / **CN:** H100
- **EN:** data / **CN:** data
- **EN:** GPU / **CN:** GPU
- **EN:** url / **CN:** url
- **EN:** BF16 / **CN:** BF16
- **EN:** memory / **CN:** memory

## Dependencies / 依赖关系
- `./glm45.md`
- `glm45.md`
