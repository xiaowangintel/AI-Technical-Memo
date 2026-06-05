# deterministic_inference.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/deterministic_inference.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Why Deterministic Inference Matters Deterministic inference ensures consistent LLM outputs across runs, which is critical for: **Reinforcement Learning**: Ensures consistent logprobs across runs, reducing stochastic noise and making RL training more stable, reproducible, and debuggable. / 该文档围绕 Deterministic Inference 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Why Deterministic Inference Matters
**EN:** Deterministic inference ensures consistent LLM outputs across runs, which is critical for: - **Reinforcement Learning**: Ensures consistent logprobs across runs, reducing stochastic noise and making RL training more stable, reproducible, and debuggable.
**CN:** 本节围绕 Why Deterministic Inference Matters 展开，概述了 LLM, GPU, Even, runs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: The Root Cause of Non-Determinism
**EN:** The main source is **varying batch sizes**. Different batch sizes cause GPU kernels to split reduction operations differently, leading to different addition orders.
**CN:** 本节围绕 The Root Cause of Non-Determinism 展开，概述了 GPU, Due, batch, sizes 等要点，并说明相关配置、流程、示例或限制条件。

### Section: SGLang's Solution
**EN:** Building on Thinking Machines Lab's batch-invariant operators, SGLang achieves fully deterministic inference while maintaining compatibility with chunked prefill, CUDA graphs, radix cache, and non-greedy sampling.
**CN:** 本节围绕 SGLang's Solution 展开，概述了 CUDA, Building, inference, deterministic 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Backends
**EN:** This section provides a comparison table for Supported Backends, covering columns such as Attention Backend, CUDA Graph, Chunked Prefill and examples such as **FlashInfer**, **FlashAttention 3 (FA3)**, **Triton**.
**CN:** 本节围绕 Supported Backends 展开，概述了 Yes, FA3, Triton, FlashInfer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Basic Usage
**EN:** Enable deterministic inference by adding the --enable-deterministic-inference flag: ``bash python3 -m sglang.launch_server \ --model-path Qwen/Qwen3-8B \ --attention-backend fa3 \ --enable-deterministic-inference ``
**CN:** 本节围绕 Basic Usage 展开，概述了 --enable-deterministic-inference, Enable, enable-deterministic-inference, fa3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Server Arguments
**EN:** This section provides a comparison table for Server Arguments, covering columns such as Argument, Type/Default, Description and examples such as --enable-deterministic-inference, --attention-backend.
**CN:** 本节围绕 服务端 Arguments 展开，概述了 fa3, Enable, Choose, Argument 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example Configurations
**EN:** This content focuses on Example Configurations and highlights python3, Qwen3-8B, model-path, --model-path.
**CN:** 本节围绕 Example 配置s 展开，概述了 python3, Qwen3-8B, model-path, --model-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Deterministic Inference with Non-Greedy Sampling (Temperature > 0)
**EN:** SGLang supports deterministic inference even with non-greedy sampling by using sampling seeds.
**CN:** 本节围绕 Deterministic Inference with Non-Greedy Sampling (Temperature > 0) 展开，概述了 sampling, seed, same, response 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Verification
**EN:** This content focuses on Verification and highlights test, python3, test-mode, --test-mode.
**CN:** 本节围绕 Verification 展开，概述了 test, python3, test-mode, --test-mode 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Yes / **CN:** Yes
- **EN:** different / **CN:** different
- **EN:** sampling / **CN:** sampling
- **EN:** inference / **CN:** inference
- **EN:** deterministic / **CN:** deterministic
- **EN:** --enable-deterministic-inference / **CN:** --enable-deterministic-inference
- **EN:** seed / **CN:** seed
- **EN:** same / **CN:** same

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
