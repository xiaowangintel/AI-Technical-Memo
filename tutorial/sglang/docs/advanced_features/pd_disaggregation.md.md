# pd_disaggregation.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/pd_disaggregation.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Why and What is PD Disaggregation? Large Language Model (LLM) inference comprises two distinct phases: **Prefill** and **Decode**. / 该文档围绕 PD Disaggregation 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Why and What is PD Disaggregation?
**EN:** Large Language Model (LLM) inference comprises two distinct phases: **Prefill** and **Decode**.
**CN:** 本节围绕 Why and What is PD Disaggregation? 展开，概述了 Prefill, Decode, LLM, phase 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Issues with Unified Scheduling
**EN:** The conventional unified engine, which processes prefill and decode batches together, results in two significant problems: 1.
**CN:** 本节围绕 Issues with Unified Scheduling 展开，概述了 decode, prefill, batches, two 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profiling in PD Disaggregation Mode
**EN:** When you need to profile prefill or decode workers in PD disaggregation mode, please refer to the Profile In PD Disaggregation Mode section in the Benchmark and Profiling guide.
**CN:** 本节围绕 Profiling in PD Disaggregation Mode 展开，概述了 Due, decode, Profile, prefill 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Router Integration
**EN:** For deploying PD disaggregation at scale with load balancing and fault tolerance, SGLang provides a router. The router can distribute requests between prefill and decode instances using various routing policies.
**CN:** 本节围绕 Router Integration 展开，概述了 Router, routing, disaggregation, see 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Requirements
**EN:** ``bash uv pip install mooncake-transfer-engine ``
**CN:** 本节围绕 Requirements 展开，概述了 pip, install, mooncake-transfer-engine, bash
uv pip install mooncake-transfer-engine
 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Llama Single Node
**EN:** This content focuses on Llama Single Node and highlights port, --port, decode, prefill.
**CN:** 本节围绕 Llama Single Node 展开，概述了 port, --port, decode, prefill 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek Multi-Node
**EN:** This content focuses on DeepSeek Multi-Node and highlights host, port, --host, --port.
**CN:** 本节围绕 DeepSeek Multi-Node 展开，概述了 host, port, --host, --port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Advanced Configuration
**EN:** This section provides a comparison table for Advanced Configuration, covering columns such as Variable, Description, Default and examples such as **SGLANG_DISAGGREGATION_THREAD_POOL_SIZE**, **SGLANG_DISAGGREGATION_QUEUE_SIZE**, **SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT**, **SGLANG_DISAGGREGATION_BOOTSTRAP_ENTRY_CLEANUP_INTERVAL**.
**CN:** 本节围绕 Advanced 配置 展开，概述了 export, transfer, SGLANG_MOONCAKE_CUSTOM_MEM_POOL, NVLINK 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Heterogeneous TP with GPU Staging Buffer
**EN:** When prefill and decode use different tensor parallelism (TP) sizes (e.g., prefill TP=4, decode DP attention with TP=1), the KV cache memory layout differs between the two sides.
**CN:** 本节围绕 Heterogeneous TP with GPU Staging Buffer 展开，概述了 decode, buffer, prefill, staging 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Environment Variables
**EN:** This section provides a comparison table for Environment Variables, covering columns such as Variable, Description, Default and examples such as **SGLANG_DISAGG_STAGING_BUFFER**, **SGLANG_DISAGG_STAGING_BUFFER_SIZE_MB**, **SGLANG_DISAGG_STAGING_POOL_SIZE_MB**.
**CN:** 本节围绕 Environment Variables 展开，概述了 buffer, SGLANG_DISAGG_STAGING_BUFFER, SGLANG_DISAGG_STAGING_POOL_SIZE_MB, SGLANG_DISAGG_STAGING_BUFFER_SIZE_MB 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage Example
**EN:** This content focuses on Usage Example and highlights MODEL_PATH, port, --port, decode.
**CN:** 本节围绕 Usage Example 展开，概述了 MODEL_PATH, port, --port, decode 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Requirements
**EN:** ``bash pip install nixl ` Or build from source - may be required if you already have UCX installed. `bash git clone https://github.com/ai-dynamo/nixl.git cd nixl pip install .
**CN:** 本节围绕 Requirements 展开，概述了 pip, UCX, nixl, Install 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Llama Single Node
**EN:** This content focuses on Llama Single Node and highlights port, --port, nixl, decode.
**CN:** 本节围绕 Llama Single Node 展开，概述了 port, --port, nixl, decode 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek Multi-Node
**EN:** This content focuses on DeepSeek Multi-Node and highlights nixl, host, port, --host.
**CN:** 本节围绕 DeepSeek Multi-Node 展开，概述了 nixl, host, port, --host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Advanced Configuration
**EN:** #### NIXL Backend Selection By default, NIXL uses the **UCX** backend for KV cache transfers. You can select a different NIXL plugin backend depending on your infrastructure using the environment variable SGLANG_DISAGGREGATION_NIXL_BACKEND.
**CN:** 本节围绕 Advanced 配置 展开，概述了 NIXL, SGLANG_DISAGGREGATION_NIXL_BACKEND, LIBFABRIC, UCX 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** Use ascend backend with memfabric_hybrid and ASCEND_MF_STORE_URL being set ``bash pip install memfabric-hybrid==1.0.0 export ASCEND_MF_STORE_URL="tcp://xxx.xx.xxx.xxx:xxxx" ` Use mooncake backend, more details can be found in mooncake section.
**CN:** 本节围绕 Usage 展开，概述了 ASCEND_NPU_PHY_ID, ASCEND_MF_STORE_URL, export, backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Llama Single Node
**EN:** This content focuses on Llama Single Node and highlights port, --port, ascend, decode.
**CN:** 本节围绕 Llama Single Node 展开，概述了 port, --port, ascend, decode 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek Multi-Node
**EN:** This content focuses on DeepSeek Multi-Node and highlights host, port, --host, --port.
**CN:** 本节围绕 DeepSeek Multi-Node 展开，概述了 host, port, --host, --port 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** prefill / **CN:** prefill 阶段
- **EN:** decode / **CN:** decode 阶段
- **EN:** port / **CN:** port
- **EN:** --port / **CN:** --port
- **EN:** model-path / **CN:** 模型-path
- **EN:** --model-path / **CN:** --模型-path
- **EN:** disaggregation-mode / **CN:** disaggregation-mode
- **EN:** sglang.launch_server / **CN:** sglang.launch_服务端

## Dependencies / 依赖关系
- `../advanced_features/sgl_model_gateway.md`
