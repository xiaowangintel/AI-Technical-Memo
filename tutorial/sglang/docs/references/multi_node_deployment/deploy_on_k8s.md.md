# deploy_on_k8s.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/references/multi_node_deployment/deploy_on_k8s.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document is for deploying a RoCE network-based SGLang two-node inference service on a Kubernetes (K8S) cluster. LeaderWorkerSet (LWS) is a Kubernetes API that aims to address common deployment patterns of AI/ML inference workloads. / 该文档围绕 Deploy On Kubernetes 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document is for deploying a RoCE network-based SGLang two-node inference service on a Kubernetes (K8S) cluster. LeaderWorkerSet (LWS) is a Kubernetes API that aims to address common deployment patterns of AI/ML inference workloads.
**CN:** 本节围绕 Overview 展开，概述了 Kubernetes, LWS, inference, K8S 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prerequisites
**EN:** At least two Kubernetes nodes, each with two H20 systems and eight GPUs, are required. Make sure your K8S cluster has LWS correctly installed.
**CN:** 本节围绕 Prerequisites 展开，概述了 LWS, LWS_WORKER_INDEX, H20, K8S 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Basic example
**EN:** For the basic example documentation, refer to Deploy Distributed Inference Service with SGLang and LWS on GPUs. However, that document only covers the basic NCCL socket mode.
**CN:** 本节围绕 Basic example 展开，概述了 LWS, GPUs, NCCL, RDMA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: RDMA RoCE case
**EN:** `text NAME READY STATUS RESTARTS AGE sglang-0 0/1 Running 0 9s sglang-0-1 1/1 Running 0 9s ` Wait for the sglang leader (sglang-0) status to change to 1/1, which indicates it is Ready.
**CN:** 本节围绕 RDMA RoCE case 展开，概述了 name, INFO, NDR, lid 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Debug
**EN:** * Set NCCL_DEBUG=TRACE to check if it is a NCCL communication problem. This should resolve most NCCL-related issues.
**CN:** 本节围绕 Debug 展开，概述了 ACTIVE, LINK_UP, Ethernet, ConnectX-7 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Keys to success
**EN:** * In the YAML configuration above, pay attention to the NCCL environment variable. For older versions of NCCL, you should check the NCCL_IB_GID_INDEX environment setting.
**CN:** 本节围绕 Keys to success 展开，概述了 NCCL, Docker, environment, but 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Remaining issues
**EN:** * In Kubernetes, Docker, or Containerd environments, we use hostNetwork to prevent performance degradation. * We utilize privileged mode, which isn’t secure.
**CN:** 本节围绕 Remaining issues 展开，概述了 GPU, Docker, Kubernetes, Containerd 等要点，并说明相关配置、流程、示例或限制条件。

### Section: TODO
**EN:** * Integrated with k8s-rdma-shared-dev-plugin.
**CN:** 本节围绕 TODO 展开，概述了 Integrated, Mellanox, k8s-rdma-shared-dev-plugin 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** ACTIVE / **CN:** ACTIVE
- **EN:** Ethernet / **CN:** Ethernet
- **EN:** name / **CN:** name
- **EN:** state / **CN:** state
- **EN:** LINK_UP / **CN:** LINK_UP
- **EN:** ConnectX-7 / **CN:** ConnectX-7
- **EN:** controller / **CN:** controller
- **EN:** Mellanox / **CN:** Mellanox

## Dependencies / 依赖关系
- `lws.yaml`
- `kubectl apply -f lws.yaml`
- `//github.com/kubernetes-sigs/lws/blob/main/site/content/en/docs/installation/_index.md`
