# tpu.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/tpu.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang supports high-performance TPU inference through the SGLang-JAX backend, which is specifically optimized for Google Cloud TPUs. / 该文档围绕 TPU 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang supports high-performance TPU inference through the SGLang-JAX backend, which is specifically optimized for Google Cloud TPUs.
**CN:** 本节围绕 Overview 展开，概述了 TPU, SGLang-JAX, JAX-based, LLM 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported TPU Hardware
**EN:** This section provides a comparison table for Supported TPU Hardware, covering columns such as TPU Type, HBM Memory, Availability and examples such as TPU v6e, TPU v7.
**CN:** 本节围绕 Supported TPU Hardware 展开，概述了 TPU, Cloud, Google, Availability 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Software Requirements
**EN:** **Python:** 3.12 or higher - **JAX:** Latest version with TPU support - **Environment:** Google Cloud TPU VM or compatible TPU runtime - **Optional:** SkyPilot for simplified cloud deployment
**CN:** 本节围绕 Software Requirements 展开，概述了 TPU, JAX, Latest, Optional 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Feature Support Matrix
**EN:** This section provides a comparison table for Feature Support Matrix, covering columns such as Feature, Support Status, Description and examples such as High-Throughput Continuous Batching, Radix Tree KV Cache, FlashAttention Backend, Tensor Parallelism.
**CN:** 本节围绕 Feature 支持矩阵 展开，概述了 TPU, Attention, TPU-optimized, development 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Attention Backend Comparison
**EN:** This section provides a comparison table for Attention Backend Comparison, covering columns such as Backend, Paged Attention, Spec Decoding and examples such as FlashAttention (fa), Native.
**CN:** 本节围绕 注意力后端 Comparison 展开，概述了 FlashAttention, MLA, NOTE, Native 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Optimized Model List
**EN:** This section provides a comparison table for Optimized Model List, covering columns such as Model Family, Performance Status and examples such as Qwen 3, Qwen 3 MoE, Qwen 2, Qwen 2 MoE.
**CN:** 本节围绕 Optimized 模型 List 展开，概述了 Qwen, Needs, improvement, MoE 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 1: Using PyPI (Recommended)
**EN:** ``bash pip install sglang-jax ``
**CN:** 本节围绕 Method 1: Using PyPI (Recommended) 展开，概述了 pip, install, sglang-jax, bash
pip install sglang-jax
 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 2: From Source
**EN:** ``bash git clone https://github.com/sgl-project/sglang-jax cd sglang-jax uv venv --python 3.12 && source .venv/bin/activate uv pip install -e "python[all]" ``
**CN:** 本节围绕 Method 2: From Source 展开，概述了 git, pip, venv, clone 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 3: Using Docker
**EN:** **NOTE:** Docker support for TPU is currently under development. Please use PyPI or source installation methods.
**CN:** 本节围绕 Method 3: Using Docker 展开，概述了 TPU, NOTE, PyPI, Docker 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 4: Cloud TPU with SkyPilot
**EN:** SkyPilot provides simplified deployment on Google Cloud TPU: 1. Install SkyPilot and configure GCP access (see SkyPilot documentation) 2.
**CN:** 本节围绕 Method 4: Cloud TPU with SkyPilot 展开，概述了 SkyPilot, sglang-jax.sky.yaml, TPU, sglang-jax 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Basic Example: Qwen-7B
**EN:** JAX_COMPILATION_CACHE_DIR=/tmp/jit_cache - Enables JIT compilation caching to accelerate server startup on subsequent runs 2. tp-size=4 - Tensor parallelism size; match this to your TPU core count (typically 1, 4, or 8) 3.
**CN:** 本节围绕 Basic Example: Qwen-7B 展开，概述了 TPU, JAX_COMPILATION_CACHE_DIR, device, bfloat16 等要点，并说明相关配置、流程、示例或限制条件。

### Section: High-Performance Configuration: Qwen3-8B
**EN:** This content focuses on High-Performance Configuration: Qwen3-8B and highlights tpu, Qwen, dtype, device.
**CN:** 本节围绕 High-性能 配置: Qwen3-8B 展开，概述了 tpu, Qwen, dtype, device 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Advanced: Speculative Decoding (EAGLE3)
**EN:** See the Speculative Decoding documentation for detailed configuration guidance.
**CN:** 本节围绕 Advanced: 投机解码 (EAGLE3) 展开，概述了 Speculative, See, NOTE, Qwen3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-Node Distributed Serving
**EN:** This content focuses on Multi-Node Distributed Serving and highlights Node, MODEL_PATH, nnodes, python3.
**CN:** 本节围绕 Multi-Node Distributed Serving 展开，概述了 Node, MODEL_PATH, nnodes, python3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Throughput Testing
**EN:** Basic throughput benchmark: ``bash python3 -m sgl_jax.bench_serving \ --backend sgl-jax \ --dataset-name random \ --num-prompts=100 \ --random-input=512 \ --random-output=128 \ --max-concurrency=8 \ --random-range-ratio=1 \ --warmup-requests=0 ``
**CN:** 本节围绕 Throughput Testing 展开，概述了 Basic, random, python3, backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Latency Testing
**EN:** Measure single-batch latency: ``bash python3 -m sgl_jax.bench_one_batch_server \ --base-url http://127.0.0.1:30000 \ --model-path Qwen/Qwen-7B-Chat \ --batch-size=32 \ --input-len=256 \ --output-len=32 ``
**CN:** 本节围绕 Latency Testing 展开，概述了 Measure, Qwen, latency, python3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Comprehensive Benchmark Script
**EN:** This content focuses on Comprehensive Benchmark Script and highlights echo, done, backend, output_seq_len.
**CN:** 本节围绕 Comprehensive Benchmark Script 展开，概述了 echo, done, backend, output_seq_len 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory Optimization
**EN:** This content focuses on Memory Optimization and highlights Reduce, memory, --mem-fraction-static, Lower.
**CN:** 本节围绕 Memory Optimization 展开，概述了 Reduce, memory, --mem-fraction-static, Lower 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Throughput Optimization
**EN:** This content focuses on Throughput Optimization and highlights Enable, Qwen3, EAGLE3, Increase.
**CN:** 本节围绕 Throughput Optimization 展开，概述了 Enable, Qwen3, EAGLE3, Increase 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Latency Optimization
**EN:** To minimize time-to-first-token (TTFT) and inter-token latency: - Reduce --page-size to 1-4 - Lower --max-running-requests (16-32) for smaller batches - Reduce --chunked-prefill-size - Use conservative memory settings to avoid GC pauses
**CN:** 本节围绕 Latency Optimization 展开，概述了 Reduce, TTFT, Lower, --page-size 等要点，并说明相关配置、流程、示例或限制条件。

### Section: TPU-Specific Optimizations
**EN:** **JIT Compilation Cache:** ``bash export JAX_COMPILATION_CACHE_DIR=/tmp/jit_cache ` Always set this environment variable to cache compiled kernels and accelerate server startup. **Data Type Optimization:** Use --dtype=bfloat16 for TPU native optimization.
**CN:** 本节围绕 TPU-Specific Optimizations 展开，概述了 TPU, Always, TPUs, Match 等要点，并说明相关配置、流程、示例或限制条件。

### Section: OOM (Out of Memory) Errors
**EN:** If you encounter out-of-memory errors: 1. Reduce --mem-fraction-static from 0.8 to 0.5 or lower 2.
**CN:** 本节围绕 OOM (Out of Memory) Errors 展开，概述了 Lower, Reduce, Decrease, Increase 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Compilation Long-Time
**EN:** If the server takes too long to start: 1. Ensure JAX_COMPILATION_CACHE_DIR is properly set 2.
**CN:** 本节围绕 Compilation Long-Time 展开，概述了 JAX_COMPILATION_CACHE_DIR, JIT, first, Ensure 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Low Throughput
**EN:** If you're not achieving expected throughput: 1. Verify --tp-size matches your TPU core configuration 2.
**CN:** 本节围绕 Low Throughput 展开，概述了 TPU, Check, batch, Verify 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Connection Issues
**EN:** If clients cannot connect to the server: 1. Ensure --host=0.0.0.0 for external access (not just 127.0.0.1) 2.
**CN:** 本节围绕 Connection Issues 展开，概述了 Check, Ensure, Verify, server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Speculative Decoding
**EN:** SGLang-JAX supports EAGLE and EAGLE3 speculative decoding algorithms for Qwen3 and LLaMA model families. Speculative decoding can improve throughput by 20-40% without affecting output quality.
**CN:** 本节围绕 投机解码 展开，概述了 Speculative, See, EAGLE, Qwen3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Chunked Prefill
**EN:** Enable mixed prefill-decode batching for better TPU utilization: ``bash --chunked-prefill-size=2048 --enable-mixed-chunk `` This allows the scheduler to mix prefill operations with decode operations in the same batch, improving overall throughput.
**CN:** 本节围绕 Chunked prefill 阶段 展开，概述了 TPU, Enable, operations, mix 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Custom Attention Backends
**EN:** SGLang-JAX supports a plugin-based attention backend system. You can implement custom attention kernels optimized for specific use cases.
**CN:** 本节围绕 Custom 注意力后端s 展开，概述了 See, attention, SGLang-JAX, cases 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Environment Verification
**EN:** Verify your TPU setup before deploying: ``bash python -c "from sgl_jax import check_env; check_env.check_env()" `` This command checks: - Installed package versions - TPU device availability and specifications - System resources and configuration - Compatibility of settings
**CN:** 本节围绕 Environment Verification 展开，概述了 TPU, Verify, System, Installed 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Contributing
**EN:** We welcome contributions to improve TPU support in SGLang-JAX!
**CN:** 本节围绕 Contributing 展开，概述了 TPU, SGLang-JAX, welcome, improve 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Areas for Contribution
**EN:** **Check the Development Roadmap** to see planned features and find opportunities to contribute new functionality.
**CN:** 本节围绕 Areas for Contribution 展开，概述了 TPU, Bug, Check, Current 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How to Contribute
**EN:** Visit the sglang-jax repository 2. Read the Contribution Guide 3.
**CN:** 本节围绕 How to Contribute 展开，概述了 Read, Join, Visit, Report 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Testing on TPU
**EN:** For contributors who need TPU access for testing: - Refer to the TPU Resources Guide for information on accessing TPU hardware - Use SkyPilot with spot instances for cost-effective testing - Follow the Benchmark and Profiling Guide for performance validation
**CN:** 本节围绕 Testing on TPU 展开，概述了 TPU, Refer, Follow, testing 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Documentation
**EN:** SGLang-JAX Repository - SGLang-JAX Installation Guide - Qwen Models Quick Start - Benchmark and Profiling Guide - Speculative Decoding
**CN:** 本节围绕 Documentation 展开，概述了 Benchmark, SGLang-JAX, Qwen, Quick 等要点，并说明相关配置、流程、示例或限制条件。

### Section: External Resources
**EN:** JAX Documentation - Google Cloud TPU Documentation - SkyPilot Documentation
**CN:** 本节围绕 External Resources 展开，概述了 JAX, TPU, Cloud, Google 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** TPU / **CN:** TPU
- **EN:** Qwen / **CN:** Qwen
- **EN:** SkyPilot / **CN:** SkyPilot
- **EN:** Needs / **CN:** Needs
- **EN:** Speculative / **CN:** Speculative
- **EN:** SGLang-JAX / **CN:** SGLang-JAX
- **EN:** --mem-fraction-static / **CN:** --mem-fraction-static
- **EN:** --max-running-requests / **CN:** --max-running-requests

## Dependencies / 依赖关系
- `sglang-jax.sky.yaml`
- `//github.com/sgl-project/sglang-jax/blob/main/docs/features/speculative_decoding.md`
- `//github.com/sgl-project/sglang-jax/blob/main/docs/developer_guide/benchmark_and_profiling.md`
- `//github.com/sgl-project/sglang-jax/blob/main/docs/features/attention_backend.md`
- `//github.com/sgl-project/sglang-jax/blob/main/docs/developer_guide/contribution_guide.md`
- `//github.com/sgl-project/sglang-jax/blob/main/docs/developer_guide/tpu_resources_guide.md`
- `//github.com/sgl-project/sglang-jax/blob/main/docs/get_started/install.md`
- `//github.com/sgl-project/sglang-jax/blob/main/docs/basic_usage/qwen.md`
