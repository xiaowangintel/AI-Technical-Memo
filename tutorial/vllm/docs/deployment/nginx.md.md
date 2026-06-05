# nginx.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/nginx.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide shows how to place Nginx in front of multiple vLLM Docker containers to load balance requests. It walks through building the Nginx image, preparing configuration, creating a Docker network, launching backend containers, and verifying server readiness before traffic is sent.
- **CN:** 本指南展示了如何在多个 vLLM Docker 容器前放置 Nginx 来实现负载均衡。文档依次讲解构建 Nginx 镜像、准备配置、创建 Docker 网络、启动后端容器，以及在流量切入前验证服务已就绪。

## Key Concepts / 关键概念
- **EN:** Nginx is used as a lightweight traffic distribution layer for multiple vLLM instances. **CN:** Nginx 被用作多个 vLLM 实例前的轻量流量分发层。
- **EN:** Container networking and shared model cache handling are practical deployment concerns. **CN:** 容器网络和共享模型缓存处理是实际部署中的关键问题。
- **EN:** Readiness verification is necessary to avoid routing traffic to still-loading models. **CN:** 就绪性验证是必要的，以避免把流量转发给仍在加载模型的实例。
