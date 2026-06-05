# dstack.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/dstack.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide covers running vLLM on cloud GPUs with dstack. It centers on installing the dstack client, starting the control plane, preparing a project directory, and launching a reproducible service definition that exposes vLLM remotely.
- **CN:** 本指南介绍如何借助 dstack 在云 GPU 上运行 vLLM。重点包括安装 dstack 客户端、启动控制面、准备项目目录，以及发布可复现的服务定义以对外提供 vLLM。

## Key Concepts / 关键概念
- **EN:** dstack is framed as an open-source orchestration layer for multi-cloud LLM execution. **CN:** dstack 被定位为一个面向多云 LLM 运行的开源编排层。
- **EN:** The workflow depends on preconfigured cloud credentials, gateway access, and quotas. **CN:** 该流程依赖预先配置好的云凭据、网关访问和资源配额。
- **EN:** Both local project structure and remote endpoint exposure are part of the deployment story. **CN:** 本地项目结构和远程端点暴露共同构成了部署流程。
