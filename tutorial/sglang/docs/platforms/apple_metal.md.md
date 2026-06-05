# apple_metal.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/apple_metal.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document describes how run SGLang on Apple Silicon using Metal (MLX). If you encounter issues or have questions, please open an issue. / 该文档围绕 Apple Silicon with Metal (MLX) 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document describes how run SGLang on Apple Silicon using Metal (MLX). If you encounter issues or have questions, please open an issue.
**CN:** 本节围绕 Overview 展开，概述了 MLX, Metal, run, open 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install SGLang
**EN:** You can install SGLang using one of the methods below.
**CN:** 本节围绕 Install SGLang 展开，概述了 one, below, install, methods 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install from Source
**EN:** ``bash # Use the default branch git clone https://github.com/sgl-project/sglang.git cd sglang # Install sglang python package pip install --upgrade pip rm -f python/pyproject.toml && mv python/pyproject_other.toml python/pyproject.toml uv pip install -e "python[all_mps]" ``
**CN:** 本节围绕 Install from Source 展开，概述了 pip, Install, python/pyproject.toml, git 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch of the Serving Engine
**EN:** Launch the server with: ``bash SGLANG_USE_MLX=1 python -m sglang.launch_server \ --model <MODEL_ID_OR_PATH> \ --disable-cuda-graph \ --host 0.0.0.0 ` **Key Parameters Explained:** 1.
**CN:** 本节围绕 Launch of the Serving Engine 展开，概述了 MLX, Disables, SGLANG_USE_MLX, --disable-cuda-graph 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmarking with Requests
**EN:** sglang.benchmark_one_batch calls the synchronous prefill/decode methods directly without going through the scheduler and the overlap code path.
**CN:** 本节围绕 Benchmarking with Requests 展开，概述了 overlap, code, path, scheduler 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Throughput Testing
**EN:** This content focuses on Throughput Testing and highlights SGLANG_USE_MLX, MODEL_ID_OR_PATH, throughput, model-path.
**CN:** 本节围绕 Throughput Testing 展开，概述了 SGLANG_USE_MLX, MODEL_ID_OR_PATH, throughput, model-path 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** SGLANG_USE_MLX / **CN:** SGLANG_USE_MLX
- **EN:** MLX / **CN:** MLX
- **EN:** MODEL_ID_OR_PATH / **CN:** 模型_ID_OR_PATH
- **EN:** --disable-cuda-graph / **CN:** --disable-cuda-graph
- **EN:** Metal / **CN:** Metal
- **EN:** Install / **CN:** Install
- **EN:** disable-cuda-graph / **CN:** disable-cuda-graph
- **EN:** --disable-overlap-schedule / **CN:** --disable-overlap-schedule

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
