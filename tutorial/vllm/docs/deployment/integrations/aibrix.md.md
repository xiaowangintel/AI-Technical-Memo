# aibrix.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/integrations/aibrix.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document introduces AIBrix as a cloud-native control plane for deploying and operating vLLM on Kubernetes. It highlights scaling, routing, and LoRA adapter management, then directs readers to the dedicated AIBrix documentation for installation and day-two operations.
- **CN:** 本文档将 AIBrix 介绍为一个面向 Kubernetes 的云原生控制平面，用于部署和运维 vLLM。它强调扩缩容、流量路由以及 LoRA 适配器管理，并把读者引导到 AIBrix 专门文档中查看安装和运维细节。

## Key Concepts / 关键概念
- **EN:** AIBrix focuses on control-plane concerns rather than only raw model serving. **CN:** AIBrix 关注的是控制平面层面的问题，而不只是模型推理本身。
- **EN:** The integration is Kubernetes-centric from the outset. **CN:** 该集成从一开始就是面向 Kubernetes 的。
- **EN:** LoRA management is treated as part of the operational surface area. **CN:** LoRA 管理被视为运维能力的一部分。
