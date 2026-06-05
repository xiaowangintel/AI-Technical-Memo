# deepseek_v32.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/deepseek_v32.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: DeepSeek-V3.2 model family equips DeepSeek-V3.1-Terminus with DeepSeek Sparse Attention (DSA) through continued training. / 该文档围绕 DeepSeek V3.2/GLM-5 Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** DeepSeek-V3.2 model family equips DeepSeek-V3.1-Terminus with DeepSeek Sparse Attention (DSA) through continued training.
**CN:** 本节围绕 Overview 展开，概述了 DeepSeek-V3.2, DSA, DeepSeek-V3.2-Exp, GLM-5 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Docker
**EN:** ``bash # H200/B200 docker pull lmsysorg/sglang:latest # MI350/MI355 docker pull lmsysorg/sglang:v0.5.8-rocm700-mi35x # MI300 # v0.5.8-rocm700-mi30x does not include PR #17504.
**CN:** 本节围绕 Docker 展开，概述了 pull, docker, NPUs, MI300 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Build From Source
**EN:** ``bash # Install SGLang git clone https://github.com/sgl-project/sglang cd sglang pip3 install pip --upgrade pip3 install -e "python" ``
**CN:** 本节围绕 Build From Source 展开，概述了 pip3, install, git, pip 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch DeepSeek V3.2/GLM-5 with SGLang
**EN:** This content focuses on Launch DeepSeek V3.2/GLM-5 with SGLang and highlights Launch, DeepSeek-V3.2-Exp, model, --model.
**CN:** 本节围绕 Launch DeepSeek V3.2/GLM-5 with SGLang 展开，概述了 Launch, DeepSeek-V3.2-Exp, model, --model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration Tips
**EN:** **DP Attention**: To enable DP Attention, please include --enable-dp-attention --dp <dp-size> in command. DP Attention is better for large concurrency scenarios.
**CN:** 本节围绕 配置 Tips 展开，概述了 Hopper, attention, flashmla_kv, flashmla_sparse 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-token Prediction
**EN:** SGLang implements Multi-Token Prediction (MTP) for DeepSeek V3.2 based on EAGLE speculative decoding. With this optimization, the decoding speed can be improved significantly on small batch sizes.
**CN:** 本节围绕 Multi-令牌 Prediction 展开，概述了 EAGLE, batch, speculative-num-steps, speculative-eagle-topk 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Function Calling and Reasoning Parser
**EN:** The usage of function calling and reasoning parser is the same as DeepSeek V3.1. Please refer to Reasoning Parser and Tool Parser documents.
**CN:** 本节围绕 Function Calling and Reasoning Parser 展开，概述了 parser, calling, reasoning, tp-size 等要点，并说明相关配置、流程、示例或限制条件。

### Section: NVFP4 Checkpoint
**EN:** To launch deepseek v3.2 NVFP4 checkpoint on Blackwell devices, the user needs to specify the quantization method as modelopt_fp4, and moe runner backend as one of flashinfer_trtllm(recommended), flashinfer_cutlass and flashinfer_cutedsl.
**CN:** 本节围绕 NVFP4 Checkpoint 展开，概述了 modelopt_fp4, flashinfer_trtllm, FP8, NVFP4 等要点，并说明相关配置、流程、示例或限制条件。

### Section: PD Disaggregation
**EN:** Additionally, you can also find startup commands for DeepEP-based EP parallelism in the aforementioned documentation.
**CN:** 本节围绕 PD Disaggregation 展开，概述了 PORT, HOST, LOCAL_IP, DIST_PORT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Accuracy Test with gsm8k
**EN:** The results are very close to the 8 shots results: ` Accuracy: 0.956 Invalid: 0.000 Latency: 29.545 s Output throughput: 4418.617 token/s ``
**CN:** 本节围绕 Accuracy Test with gsm8k 展开，概述了 Output, Invalid, Latency, Accuracy 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Accuracy Test with gpqa-diamond
**EN:** This content focuses on Accuracy Test with gpqa-diamond and highlights Repeat, Scores, mean, DeepSeek.
**CN:** 本节围绕 Accuracy Test with gpqa-diamond 展开，概述了 Repeat, Scores, mean, DeepSeek 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Accuracy Test with aime 2025
**EN:** This section provides a comparison table for Accuracy Test with aime 2025, covering columns such as evaluation_mode, num_entries, avg_tokens and examples such as pass@1[avg-of-4], majority@4, pass@4, evaluation_mode.
**CN:** 本节围绕 Accuracy Test with aime 2025 展开，概述了 BACKEND, PORT, pass, MODEL 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DSA long sequence context parallel optimization(experimental)
**EN:** **Note: This feature is only verified on Hopper machines** For context parallel in DeepSeek V3.2 model, we provide two different modes of splitting tokens, which can be controlled with argument --nsa-prefill-cp-mode.
**CN:** 本节围绕 DSA long sequence context parallel optimization(experimental) 展开，概述了 Note, Hopper, --nsa-prefill-cp-mode, two 等要点，并说明相关配置、流程、示例或限制条件。

### Section: In sequence splitting
**EN:** The first mode can be enabled by --nsa-prefill-cp-mode in-seq-split. This mode implements context parallel for DSA by splitting the sequence uniformly between context parallel ranks.
**CN:** 本节围绕 In sequence splitting 展开，概述了 mode, context, parallel, splitting 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Round robin splitting (default setting)
**EN:** This mode can be enabled by specifying the parameter --nsa-prefill-cp-mode round-robin-split, which distributes tokens across ranks based on token_idx % cp_size.
**CN:** 本节围绕 Round robin splitting (default setting) 展开，概述了 MoE, CP8, fused, DeepEP 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Pipeline Parallel + Context Parallel (PP + CP)
**EN:** This mode combines Pipeline Parallelism (PP) and Context Parallelism (CP) to scale across multiple nodes, which can achieve better throughput and Time To First Token (TTFT). Note that this method has only been tested on H20 96G.
**CN:** 本节围绕 Pipeline Parallel + Context Parallel (PP + CP) 展开，概述了 Node, round-robin-split, --tp, host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: HiSparse: Hierarchical Sparse Attention for DSA (experimental)
**EN:** HiSparse reduces per-request GPU memory during decode by keeping only a small "hot" KV buffer on GPU while storing complete KV data in CPU pinned memory. A CUDA kernel dynamically swaps in the top-k most relevant KV entries from host memory on each decode step.
**CN:** 本节围绕 HiSparse: Hierarchical Sparse Attention for DSA (experimental) 展开，概述了 HiSparse, GPU, decode, memory 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** DeepSeek-V3.2-Exp / **CN:** DeepSeek-V3.2-Exp
- **EN:** model / **CN:** 模型
- **EN:** sglang.launch_server / **CN:** sglang.launch_服务端
- **EN:** attention / **CN:** attention
- **EN:** Hopper / **CN:** Hopper
- **EN:** DeepSeek-V3.2 / **CN:** DeepSeek-V3.2
- **EN:** deepseek-ai/DeepSeek-V3.2-Exp / **CN:** deepseek-ai/DeepSeek-V3.2-Exp
- **EN:** --tp / **CN:** --tp

## Dependencies / 依赖关系
- `../advanced_features/dp_dpa_smg_guide.md`
- `../references/multi_node_deployment/rbg_pd/deepseekv32_pd.md`
- `../advanced_features/hisparse_guide.md`
- `bench_speculative.py`
- `//github.com/sgl-project/sglang/blob/main/scripts/playground/bench_speculative.py`
- `references/multi_node_deployment/rbg_pd/deepseekv32_pd.md`
- `benchmark/gsm8k/bench_sglang.py`
