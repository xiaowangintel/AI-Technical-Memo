# deepseekv32_pd.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/references/multi_node_deployment/rbg_pd/deepseekv32_pd.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Prerequisites k8s >=1.26 lws installed on k8s. rbg installed on k8s. / 该文档围绕 DeepSeekV32-Exp RBG Based PD Deploy 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: 0. Prerequisites
**EN:** lws installed on k8s. rbg installed on k8s.
**CN:** 本节围绕 0. Prerequisites 展开，概述了 k8s, installed, lws, rbg 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 1. Image Preparation
**EN:** lmsysorg/sglang:latest
**CN:** 本节围绕 1. Image Preparation 展开，概述了 lmsysorg/sglang:latest 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 2. All In One manifest file
**EN:** This content focuses on 2. All In One manifest file and highlights name, value, path, LWS_WORKER_INDEX.
**CN:** 本节围绕 2. All In One manifest file 展开，概述了 name, value, path, LWS_WORKER_INDEX 等要点，并说明相关配置、流程、示例或限制条件。

### Section: FAQ
**EN:** The current deployment startup parameters may not be fully compatible with all RDMA scenarios. Different RDMA NCCL-related environment configurations may be needed in different network environments.
**CN:** 本节围绕 FAQ 展开，概述了 RDMA, may, Please, code 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** name / **CN:** name
- **EN:** value / **CN:** value
- **EN:** path / **CN:** path
- **EN:** LWS_WORKER_INDEX / **CN:** LWS_WORKER_INDEX
- **EN:** hostPath / **CN:** hostPath
- **EN:** mountPath / **CN:** mountPath
- **EN:** spec / **CN:** spec
- **EN:** port / **CN:** port

## Dependencies / 依赖关系
- `rbg-dsv32.yml`
