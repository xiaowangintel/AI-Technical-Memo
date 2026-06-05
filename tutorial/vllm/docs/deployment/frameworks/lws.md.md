# lws.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/lws.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide explains how to deploy vLLM with LeaderWorkerSet (LWS) for multi-node inference on Kubernetes. It highlights the hardware requirements, installation of the LWS API, YAML-based deployment, service access, and a distributed serving example.
- **CN:** 本指南介绍如何借助 LeaderWorkerSet（LWS）在 Kubernetes 上部署 vLLM 以实现多节点推理。内容强调硬件要求、LWS API 的安装、基于 YAML 的部署、服务访问方式以及分布式服务示例。

## Key Concepts / 关键概念
- **EN:** LWS targets AI/ML deployment patterns that need coordinated leader-worker groups. **CN:** LWS 面向需要协调 leader-worker 组的 AI/ML 部署模式。
- **EN:** The guide is specifically about distributed serving, not single-node usage. **CN:** 该指南专门针对分布式服务，而不是单节点使用。
- **EN:** Kubernetes-native manifests are the main mechanism for describing the deployment. **CN:** Kubernetes 原生清单是描述该部署的主要机制。
