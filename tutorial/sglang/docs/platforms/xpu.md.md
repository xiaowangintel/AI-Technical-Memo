# xpu.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/xpu.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: The document addresses how to set up the SGLang environment and run LLM inference on Intel GPU, see more context about Intel GPU support within PyTorch ecosystem. / 该文档围绕 XPU 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** The document addresses how to set up the SGLang environment and run LLM inference on Intel GPU, see more context about Intel GPU support within PyTorch ecosystem.
**CN:** 本节围绕 Overview 展开，概述了 Intel, Arc, LLM, GPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Optimized Model List
**EN:** This section provides a comparison table for Optimized Model List, covering columns such as Model Name, BF16 and examples such as Llama-3.2-3B, Llama-3.1-8B, Qwen2.5-1.5B.
**CN:** 本节围绕 Optimized 模型 List 展开，概述了 Qwen2.5-1.5B, Intel, Arc, LLMs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install From Source
**EN:** Currently SGLang XPU only supports installation from source. Please refer to "Getting Started on Intel GPU" to install XPU dependency.
**CN:** 本节围绕 Install From Source 展开，概述了 XPU, install, pip, CUDA-enabled 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install Using Docker
**EN:** The docker for XPU is under active development.
**CN:** 本节围绕 Install Using Docker 展开，概述了 XPU, Please, stay, tuned 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch of the Serving Engine
**EN:** This content focuses on Launch of the Serving Engine and highlights XPU, GPUs, backend, intel_xpu.
**CN:** 本节围绕 Launch of the Serving Engine 展开，概述了 XPU, GPUs, backend, intel_xpu 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmarking with Requests
**EN:** You can benchmark the performance via the bench_serving script. Run the command in another terminal.
**CN:** 本节围绕 Benchmarking with Requests 展开，概述了 via, command, Run, curl 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prefill-Decode (P/D) Disaggregation on Intel XPU [Experimental]
**EN:** This section provides a comparison table for Prefill-Decode (P/D) Disaggregation on Intel XPU [Experimental], covering columns such as Model, Notes and examples such as Qwen/Qwen3-0.6B, Qwen/Qwen2.5-7B-Instruct.
**CN:** 本节围绕 prefill 阶段-decode 阶段 (P/D) Disaggregation on Intel XPU [Experimental] 展开，概述了 XPU, Qwen, Start, UCX_POSIX_USE_PROC_LINK 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** XPU / **CN:** XPU
- **EN:** Intel / **CN:** Intel
- **EN:** Qwen / **CN:** Qwen
- **EN:** GPU / **CN:** GPU
- **EN:** install / **CN:** install
- **EN:** Arc / **CN:** Arc
- **EN:** Start / **CN:** Start
- **EN:** UCX_POSIX_USE_PROC_LINK / **CN:** UCX_POSIX_USE_PROC_LINK

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
