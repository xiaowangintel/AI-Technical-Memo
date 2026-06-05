# dp_dpa_smg_guide.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/dp_dpa_smg_guide.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This guide explains the difference between Data Parallelism (DP) and Data Parallelism Attention (DPA), how to enable each mode correctly, and how to use the SGLang Model Gateway (SMG) for production-grade DP deployments. / 该文档围绕 DP, DPA and SGLang DP Router 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This guide explains the difference between Data Parallelism (DP) and Data Parallelism Attention (DPA), how to enable each mode correctly, and how to use the SGLang Model Gateway (SMG) for production-grade DP deployments.
**CN:** 本节围绕 Overview 展开，概述了 DPA, SMG, Data, Parallelism 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data Parallelism (DP)
**EN:** **Data Parallelism (DP)** is the most common parallelism strategy that replicates the entire model across multiple GPU sets and processes different batches of requests in parallel. Each GPU set handles independent requests.
**CN:** 本节围绕 Data 并行策略 (DP) 展开，概述了 GPU, routing, requests, Data 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Key characteristics
**EN:** Each replica has a full copy of the model - Requests are distributed/scattered across replicas - No inter-replica communication during one request's inference (for simple DP)
**CN:** 本节围绕 Key characteristics 展开，概述了 Requests, one, full, copy 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data Parallelism Attention (DPA)
**EN:** **Data Parallelism Attention (DPA)**, also known as DP Attention, is an advanced parallelism strategy.
**CN:** 本节围绕 Data 并行策略 Attention (DPA) 展开，概述了 Attention, DPA, MLA, Qwen 等要点，并说明相关配置、流程、示例或限制条件。

### Section: The Problem with Tensor Parallelism for MLA Models
**EN:** The most common parallelism strategy for inference is **Tensor Parallelism (TP)**. However, TP might not be the most efficient strategy for certain models.
**CN:** 本节围绕 The Problem with Tensor 并行策略 for MLA 模型s 展开，概述了 GPUs, MLA, most, models 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How DPA Works
**EN:** DPA addresses these limitations by applying **data parallelism specifically to the attention component**.
**CN:** 本节围绕 How DPA Works 展开，概述了 DPA, All2All, width, table 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Key benefits of DPA
**EN:** **Significantly reduced KV cache memory**: Each DP replica only stores KV cache for its own batches 2. **Larger batch sizes**: Memory savings enable larger batch sizes 3.
**CN:** 本节围绕 Key benefits of DPA 展开，概述了 cache, batch, sizes, modes 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DPA with Expert Parallelism for MoE
**EN:** For MoE models like DeepSeek, DPA is **often** paired with Expert Parallelism (EP) for best throughput at scale. However, **DPA does not require EP**: you can enable DPA without EP if your deployment does not need expert sharding.
**CN:** 本节围绕 DPA with Expert 并行策略 for MoE 展开，概述了 DPA, GPU, GPUs, does 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Recommended setup for DeepSeek
**EN:** ``bash python -m sglang.launch_server \ --model-path deepseek-ai/DeepSeek-V3 \ --tp 8 \ --dp-size 8 \ --ep 8 \ --enable-dp-attention \ --moe-a2a-backend deepep \ --moe-runner-backend deep_gemm ` > **Note**: --dp-size must be explicitly set when using --enable-dp-attention.
**CN:** 本节围绕 Recommended setup for DeepSeek 展开，概述了 DPA, Note, EPLB, DeepEP 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Target Models
**EN:** To enable DPA, add --enable-dp-attention to your server launch command.
**CN:** 本节围绕 Target 模型s 展开，概述了 DPA, models, MLA, GQA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Activation Logic
**EN:** DPA is enabled explicitly via server arguments (CLI or config).
**CN:** 本节围绕 Activation Logic 展开，概述了 DPA, --dp-size, --enable-dp-attention, must 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Standard DP for MLA models
**EN:** Note that MLA models, of course, also support DP. Suppose you want to enable standard DP for MLA models.
**CN:** 本节围绕 Standard DP for MLA 模型s 展开，概述了 MLA, SMG, launch, DPA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Native DP Mode
**EN:** Native DP (built-in Data Parallelism) in SGLang creates multiple worker processes within a single SGLang instance, under the control of DataParallelController with the launching parameter of dp-size. It is only used in some ancient/outdated RL frameworks.
**CN:** 本节围绕 Native DP Mode 展开，概述了 Native, dp-size, DataParallelController, SMG 等要点，并说明相关配置、流程、示例或限制条件。

### Section: SMG-Based DP (Recommended)
**EN:** This section provides a comparison table for SMG-Based DP (Recommended), covering columns such as Feature, Native DP, SMG-Based DP and examples such as **Load Balancing**, **Cache Awareness**, **Throughput**, **Multi-Node Support**.
**CN:** 本节围绕 SMG-Based DP (Recommended) 展开，概述了 SMG, Basic, Yes, Rust 等要点，并说明相关配置、流程、示例或限制条件。

### Section: SMG's Performance
**EN:** This section provides a comparison table for SMG's Performance, covering columns such as Metric, Without Cache-Aware, With Cache-Aware SMG and examples such as Throughput (token/s), Cache Hit Rate.
**CN:** 本节围绕 SMG's 性能 展开，概述了 SMG, A100, GPUs, Metric 等要点，并说明相关配置、流程、示例或限制条件。

### Section: When to Use Each
**EN:** Details can be found at Load Balancing Router in RL.
**CN:** 本节围绕 When to Use Each 展开，概述了 SMG-Based, Note, high, Never 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Start For SMG
**EN:** Launch workers on each node `bash # Node 1 python -m sglang.launch_server \ --model-path meta-llama/Meta-Llama-3.1-8B-Instruct \ --port 8000 # Node 2 python -m sglang.launch_server \ --model-path meta-llama/Meta-Llama-3.1-8B-Instruct \ --port 8000 ` 2.
**CN:** 本节围绕 Quick Start For SMG 展开，概述了 SMG, Option, port, --port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Load Balancing Policies
**EN:** This section provides a comparison table for Load Balancing Policies, covering columns such as Policy, Description, Best For and examples such as cache_aware, round_robin, random, power_of_two.
**CN:** 本节围绕 Load Balancing Policies 展开，概述了 Recommended, load, Policy, workers 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Best Practices
**EN:** **Start with cache_aware policy** - It provides the best balance between cache locality and load distribution for most workloads 2.
**CN:** 本节围绕 Best Practices 展开，概述了 cache_aware, sglang_router.launch_server, --router-health-check-interval-secs, K8s 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Verifying Traffic Distribution
**EN:** After launching SMG, verify that traffic is being distributed correctly: **1. Check worker status:** ``bash curl http://localhost:30000/workers ` **2.
**CN:** 本节围绕 Verifying Traffic Distribution 展开，概述了 Check, metrics, SMG, Key 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reference
**EN:** This section provides a comparison table for Reference, covering columns such as Strategy, Use Case, Key Benefit and examples such as **Native DP** (--dp-size), **SMG-Based DP**, **DPA** (--dp-size N --enable-dp-attention), **DPA + EP**.
**CN:** 本节围绕 Reference 展开，概述了 DPA, SMG, Enable, DeepSeek 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** SMG / **CN:** SMG
- **EN:** DPA / **CN:** DPA
- **EN:** MLA / **CN:** MLA
- **EN:** models / **CN:** 模型s
- **EN:** dp-size / **CN:** dp-size
- **EN:** --dp-size / **CN:** --dp-size
- **EN:** Parallelism / **CN:** 并行策略
- **EN:** DeepSeek / **CN:** DeepSeek

## Dependencies / 依赖关系
- `expert_parallelism.md`
- `sgl_model_gateway.md`
- `./sglang_for_rl.md`
