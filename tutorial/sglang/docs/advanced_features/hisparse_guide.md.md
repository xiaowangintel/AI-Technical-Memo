# hisparse_guide.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/hisparse_guide.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: HiSparse reduces per-request GPU memory consumption during the decode phase by maintaining only a small "hot" KV buffer on GPU while keeping complete KV data in CPU pinned memory. / 该文档围绕 HiSparse: Hierarchical Sparse Attention 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** HiSparse reduces per-request GPU memory consumption during the decode phase by maintaining only a small "hot" KV buffer on GPU while keeping complete KV data in CPU pinned memory.
**CN:** 本节围绕 Overview 展开，概述了 GPU, HiSparse, memory, decode 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Why HiSparse?
**EN:** In long-context LLM inference, each decoding request holds a full-length KV cache on GPU, limiting the number of concurrent requests a decode instance can serve.
**CN:** 本节围绕 Why HiSparse? 展开，概述了 HiSparse, GPU, request, LLM 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Decode Workflow
**EN:** Each decode step follows this flow: 1. **Forward decode** — generate the next token 2.
**CN:** 本节围绕 decode 阶段 工作流 展开，概述了 device, host, token, attention 等要点，并说明相关配置、流程、示例或限制条件。

### Section: PD Disaggregation Integration (Direct-to-Host)
**EN:** In PD disaggregation mode, the prefill instance transfers KV cache directly into the decode instance's host pool via RDMA, bypassing the GPU entirely on the decode side.
**CN:** 本节围绕 PD Disaggregation Integration (Direct-to-Host) 展开，概述了 GPU, RDMA, DMA, CPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Server Arguments
**EN:** This section provides a comparison table for Server Arguments, covering columns such as Argument, Type / Default, Description and examples such as --enable-hisparse, --hisparse-config.
**CN:** 本节围绕 服务端 Arguments 展开，概述了 HiSparse, Type, Argument, Description 等要点，并说明相关配置、流程、示例或限制条件。

### Section: HiSparse Config Parameters
**EN:** This section provides a comparison table for HiSparse Config Parameters, covering columns such as Parameter, Type / Default, Description and examples such as top_k, device_buffer_size, host_to_device_ratio.
**CN:** 本节围绕 HiSparse Config Parameters 展开，概述了 Number, int, top_k, --hisparse-config 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Deployment
**EN:** HiSparse currently requires **PD disaggregation mode** and is enabled only on the **decode instance**.
**CN:** 本节围绕 部署 展开，概述了 HiSparse, mode, decode, enabled 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prefill Instance
**EN:** This content focuses on Prefill Instance and highlights port, host, --port, --host.
**CN:** 本节围绕 prefill 阶段 Instance 展开，概述了 port, host, --port, --host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Decode Instance (with HiSparse)
**EN:** This content focuses on Decode Instance (with HiSparse) and highlights port, host, top_k, --port.
**CN:** 本节围绕 decode 阶段 Instance (with HiSparse) 展开，概述了 port, host, top_k, --port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmark
**EN:** This content focuses on Benchmark and highlights host, port, model, --host.
**CN:** 本节围绕 Benchmark 展开，概述了 host, port, model, --host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Key Notes
**EN:** The prefill instance does not need --enable-hisparse; it is unaware of HiSparse. On the decode instance, the following flags are **required** for HiSparse: - --kv-cache-dtype bfloat16 — currently only bfloat16 KV cache is supported (more dtypes planned).
**CN:** 本节围绕 Key Notes 展开，概述了 HiSparse, host_to_device_ratio, --enable-hisparse, host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Acknowledgments
**EN:** We would like to thank the SGLang team and community for the implementation and generous support, especially Zhiqiang Xie, Zhangheng Huang, Tingwei Huang, Shangming Cai, Teng Ma, and many others.
**CN:** 本节围绕 Acknowledgments 展开，概述了 team, Teng, thank, Huang 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** HiSparse / **CN:** HiSparse
- **EN:** GPU / **CN:** GPU
- **EN:** host / **CN:** host
- **EN:** decode / **CN:** decode 阶段
- **EN:** memory / **CN:** memory
- **EN:** instance / **CN:** instance
- **EN:** host_to_device_ratio / **CN:** host_to_device_ratio
- **EN:** device / **CN:** device

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
