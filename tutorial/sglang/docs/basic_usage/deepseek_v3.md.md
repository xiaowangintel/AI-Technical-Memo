# deepseek_v3.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/deepseek_v3.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang provides many optimizations specifically designed for the DeepSeek models, making it the inference engine recommended by the official DeepSeek team from Day 0. This document outlines current optimizations for DeepSeek. / 该文档围绕 DeepSeek V3/V3.1/R1 Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang provides many optimizations specifically designed for the DeepSeek models, making it the inference engine recommended by the official DeepSeek team from Day 0. This document outlines current optimizations for DeepSeek.
**CN:** 本节围绕 Overview 展开，概述了 DeepSeek, Day, Roadmap, optimizations 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch DeepSeek V3.1/V3/R1 with SGLang
**EN:** This section provides a comparison table for Launch DeepSeek V3.1/V3/R1 with SGLang, covering columns such as Weight Type, Configuration and examples such as **Full precision FP8**<br>*(recommended)*, 8 x B200, 8 x MI300X, 2 x 8 x H100/800/20.
**CN:** 本节围绕 Launch DeepSeek V3.1/V3/R1 with SGLang 展开，概述了 H200, Quantized, A100, B200 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Download Weights
**EN:** If you encounter errors when starting the server, ensure the weights have finished downloading. It's recommended to download them beforehand or restart multiple times until all weights are downloaded.
**CN:** 本节围绕 Download Weights 展开，概述了 weights, Please, DeepSeek, download 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch with one node of 8 x H200
**EN:** Please refer to the example.
**CN:** 本节围绕 Launch with one node of 8 x H200 展开，概述了 Please, refer, --launch, installation--launch 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Running examples on Multi-Node
**EN:** Deploying DeepSeek on GB200 NVL72 with PD and Large Scale EP (Part I, Part II) - Comprehensive guide on GB200 optimizations.
**CN:** 本节围绕 Running examples on Multi-Node 展开，概述了 Serving, H20, Part, Comprehensive 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-head Latent Attention (MLA) Throughput Optimizations
**EN:** **Description**: MLA is an innovative attention mechanism introduced by the DeepSeek team, aimed at improving inference efficiency.
**CN:** 本节围绕 Multi-head Latent Attention (MLA) Throughput Optimizations 展开，概述了 MLA, FP8, Torch.compile, FlashAttention3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data Parallelism Attention
**EN:** **Description**: This optimization involves data parallelism (DP) for the MLA attention mechanism of DeepSeek Series Models, which allows for a significant reduction in the KV cache size, enabling larger batch sizes.
**CN:** 本节围绕 Data 并行策略 Attention 展开，概述了 attention, GPUs, Data, cache 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-Node Tensor Parallelism
**EN:** **Description**: For users with limited memory on a single node, SGLang supports serving DeepSeek Series Models, including DeepSeek V3, across multiple nodes using tensor parallelism.
**CN:** 本节围绕 Multi-Node Tensor 并行策略 展开，概述了 DeepSeek, GPUs, node, Usage 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Block-wise FP8
**EN:** **Description**: SGLang implements block-wise FP8 quantization with two key optimizations: - **Activation**: E4M3 format using per-token-per-128-channel sub-vector scales with online casting.
**CN:** 本节围绕 Block-wise FP8 展开，概述了 DeepGEMM, FP8, DeepSeek, E4M3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-token Prediction
**EN:** **Description**: SGLang implements DeepSeek V3 Multi-Token Prediction (MTP) based on EAGLE speculative decoding.
**CN:** 本节围绕 Multi-令牌 Prediction 展开，概述了 batch, EAGLE, MTP, sizes 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reasoning Content for DeepSeek R1 & V3.1
**EN:** See Reasoning Parser and Thinking Parameter for DeepSeek V3.1.
**CN:** 本节围绕 Reasoning Content for DeepSeek R1 & V3.1 展开，概述了 See, V3.1, Parser, Thinking 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Function calling for DeepSeek Models
**EN:** Add arguments --tool-call-parser deepseekv3 and --chat-template ./examples/chat_template/tool_chat_template_deepseekv3.jinja(recommended) to enable this feature. Use a lower "temperature" value for better results.
**CN:** 本节围绕 Function calling for DeepSeek 模型s 展开，概述了 city, function, tool_calls, arguments 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Thinking Budget for DeepSeek R1
**EN:** In SGLang, we can implement thinking budget with CustomLogitProcessor. Launch a server with --enable-custom-logit-processor flag on.
**CN:** 本节围绕 Thinking Budget for DeepSeek R1 展开，概述了 DeepSeekR1ThinkingBudgetLogitProcessor, import, CustomLogitProcessor, --enable-custom-logit-processor 等要点，并说明相关配置、流程、示例或限制条件。

### Section: FAQ
**EN:** **Q: Model loading is taking too long, and I'm encountering an NCCL timeout. What should I do?** A: If you're experiencing extended model loading times and an NCCL timeout, you can try increasing the timeout duration.
**CN:** 本节围绕 FAQ 展开，概述了 NCCL, timeout, Add, Model 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** DeepSeek / **CN:** DeepSeek
- **EN:** FP8 / **CN:** FP8
- **EN:** MLA / **CN:** MLA
- **EN:** H200 / **CN:** H200
- **EN:** city / **CN:** city
- **EN:** batch / **CN:** batch
- **EN:** function / **CN:** function
- **EN:** arguments / **CN:** arguments

## Dependencies / 依赖关系
- `../platforms/amd_gpu.md`
- `../platforms/cpu_server.md`
- `../platforms/ascend/ascend_npu_deepseek_example.md`
- `../advanced_features/attention_backend.md`
- `bench_speculative.py`
- `//github.com/sgl-project/sglang/blob/main/scripts/playground/bench_speculative.py`
- `//github.com/sgl-project/sglang/blob/main/python/sglang/srt/server_args.py`
