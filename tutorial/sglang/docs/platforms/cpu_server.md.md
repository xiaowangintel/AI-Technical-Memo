# cpu_server.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/cpu_server.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: The document addresses how to set up the SGLang environment and run LLM inference on CPU servers. SGLang is enabled and optimized on the CPUs equipped with Intel® AMX® Instructions, which are 4th generation or newer Intel® Xeon® Scalable Processors. / 该文档围绕 CPU 服务端s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** The document addresses how to set up the SGLang environment and run LLM inference on CPU servers. SGLang is enabled and optimized on the CPUs equipped with Intel® AMX® Instructions, which are 4th generation or newer Intel® Xeon® Scalable Processors.
**CN:** 本节围绕 Overview 展开，概述了 Intel, LLM, CPU, AMX 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Optimized Model List
**EN:** This section provides a comparison table for Optimized Model List, covering columns such as Model Name, BF16, W8A8_INT8 and examples such as DeepSeek-R1, DeepSeek-V3.1-Terminus, Llama-3.2-3B, Llama-3.1-8B.
**CN:** 本节围绕 Optimized 模型 List 展开，概述了 RedHatAI, DeepSeek-R1, DeepSeek-V3.1-Terminus, Qwen 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install Using Docker
**EN:** It is recommended to use Docker for setting up the SGLang environment. A Dockerfile is provided to facilitate the installation.
**CN:** 本节围绕 Install Using Docker 展开，概述了 docker, Dockerfile, host, Clone 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install From Source
**EN:** If you prefer to install SGLang in a bare metal environment, the setup process is as follows: Please install the required packages and libraries beforehand if they are not already present on your system.
**CN:** 本节围绕 Install From Source 展开，概述了 LD_LIBRARY_PATH, LD_PRELOAD, export, SGLANG_USE_CPU_ENGINE 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch of the Serving Engine
**EN:** Example command to launch SGLang serving: ``bash python -m sglang.launch_server \ --model <MODEL_ID_OR_PATH> \ --trust-remote-code \ --disable-overlap-schedule \ --device cpu \ --host 0.0.0.0 \ --tp 6 ` Notes: 1.
**CN:** 本节围绕 Launch of the Serving Engine 展开，概述了 rank, SNC, SGLANG_CPU_OMP_THREADS_BIND, numactl 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmarking with Requests
**EN:** You can benchmark the performance via the bench_serving script. Run the command in another terminal.
**CN:** 本节围绕 Benchmarking with Requests 展开，概述了 command, via, Run, curl 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example Usage Commands
**EN:** Large Language Models can range from fewer than 1 billion to several hundred billion parameters. Dense models larger than 20B are expected to run on flagship 6th Gen Intel® Xeon® processors with dual sockets and a total of 6 sub-NUMA clusters.
**CN:** 本节围绕 Example Usage Commands 展开，概述了 Xeon, Dense, Intel, fewer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example: Running DeepSeek-V3.1-Terminus
**EN:** The value 4` in the examples is illustrative.
**CN:** 本节围绕 Example: Running DeepSeek-V3.1-Terminus 展开，概述了 torch-compile-max-bs, --torch-compile-max-bs, DeepSeek-V3.1-Terminus, cpu 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example: Running Llama-3.2-3B
**EN:** For instance, use --tp 3 to utilize 1 socket with 3 sub-NUMA clusters on an Intel® Xeon® 6980P server.
**CN:** 本节围绕 Example: Running Llama-3.2-3B 展开，概述了 --tp, Llama-3.2-3B, command, torch-compile-max-bs 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Xeon / **CN:** Xeon
- **EN:** CPU / **CN:** CPU
- **EN:** rank / **CN:** rank
- **EN:** Intel / **CN:** Intel
- **EN:** command / **CN:** command
- **EN:** SNC / **CN:** SNC
- **EN:** LD_LIBRARY_PATH / **CN:** LD_LIBRARY_PATH
- **EN:** DeepSeek-V3.1-Terminus / **CN:** DeepSeek-V3.1-Terminus

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
