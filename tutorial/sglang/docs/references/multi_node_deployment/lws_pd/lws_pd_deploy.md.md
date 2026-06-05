# lws_pd_deploy.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/references/multi_node_deployment/lws_pd/lws_pd_deploy.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Prerequisites k8s >=1.26 lws installed on k8s. / 该文档围绕 LWS Based PD Deploy 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: 0. Prerequisites
**EN:** lws installed on k8s.
**CN:** 本节围绕 0. Prerequisites 展开，概述了 k8s, lws, installed 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 1. Image Preparation
**EN:** lmsysorg/sglang:deepep
**CN:** 本节围绕 1. Image Preparation 展开，概述了 lmsysorg/sglang:deepep 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 2. Deployment Manifest Files
**EN:** ***Notice: We will package all deployment files into Helm Chart format in the near future. Interested community members can contact us to contribute***
**CN:** 本节围绕 2. 部署 Manifest Files 展开，概述了 Notice, Interested, Helm, near 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prefill
**EN:** This content focuses on Prefill and highlights name, value, mountPath, path.
**CN:** 本节围绕 prefill 阶段 展开，概述了 name, value, mountPath, path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Decode
**EN:** To allow our users to directly experience the model API, we still need a load balancer to handle sequential calls between prefill and decode.
**CN:** 本节围绕 decode 阶段 展开，概述了 name, value, mountPath, path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Creating Service for Prefill and Decode
**EN:** This content focuses on Creating Service for Prefill and Decode and highlights Running, TCP, Service, Execute.
**CN:** 本节围绕 Creating Service for prefill 阶段 and decode 阶段 展开，概述了 Running, TCP, Service, Execute 等要点，并说明相关配置、流程、示例或限制条件。

### Section: FAQ
**EN:** The current deployment startup parameters may not be fully compatible with all RDMA scenarios. Different RDMA NCCL-related environment configurations may be needed in different network environments.
**CN:** 本节围绕 FAQ 展开，概述了 RDMA, may, EPLB, needed 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** name / **CN:** name
- **EN:** value / **CN:** value
- **EN:** Exists / **CN:** Exists
- **EN:** mountPath / **CN:** mountPath
- **EN:** path / **CN:** path
- **EN:** hostPath / **CN:** hostPath
- **EN:** LWS_WORKER_INDEX / **CN:** LWS_WORKER_INDEX
- **EN:** model / **CN:** 模型

## Dependencies / 依赖关系
- `lws-examples/p.yaml`
- `lws-examples/d.yaml`
- `lws-examples/p-svc.yaml`
- `kubectl apply -f p-svc.yaml`
- `lws-examples/d-svc.yaml`
- `kubectl apply -f d-svc.yaml`
- `lws-examples/lb.yaml`
- `kubectl apply -f lb.yaml`
- `prefill.yaml`
- `decode.yaml`
- `p.yaml`
- `d.yaml`
