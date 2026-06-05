# skypilot.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/skypilot.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide shows how to run and scale vLLM with SkyPilot across clouds or Kubernetes. It covers prerequisites such as model access and environment checks, then walks through launching a serving configuration, scaling replicas, and optionally attaching a GUI endpoint.
- **CN:** 本指南展示了如何使用 SkyPilot 在多云或 Kubernetes 环境中运行并扩展 vLLM。内容包括模型访问申请和环境检查等前提条件，随后讲解启动服务配置、扩展副本，以及可选地连接图形界面端点。

## Key Concepts / 关键概念
- **EN:** SkyPilot is used as a cross-environment launcher and scaler for vLLM services. **CN:** SkyPilot 被用作 vLLM 服务的跨环境启动与扩缩容工具。
- **EN:** Authentication and model access prerequisites matter before deployment begins. **CN:** 在开始部署前，认证和模型访问权限等前置条件很重要。
- **EN:** The guide bridges infrastructure launch with user-facing demo access through a GUI. **CN:** 该指南把基础设施启动与面向用户的 GUI 访问连接起来。
