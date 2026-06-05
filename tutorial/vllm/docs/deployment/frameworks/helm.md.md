# helm.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/helm.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This is a full Helm-based deployment guide for running vLLM on Kubernetes. It explains prerequisites, chart installation and removal, the chart architecture, values customization, and operational variants such as S3-backed model download or custom init containers.
- **CN:** 这是一份使用 Helm 在 Kubernetes 上部署 vLLM 的完整指南。文档说明了前置条件、Chart 的安装与卸载、架构设计、values 定制方式，以及基于 S3 下载模型或自定义 init container 等运维变体。

## Key Concepts / 关键概念
- **EN:** Helm provides repeatable Kubernetes deployment with override-friendly configuration. **CN:** Helm 提供了可重复、可覆盖配置的 Kubernetes 部署方式。
- **EN:** The guide covers not just installation but also chart structure and value semantics. **CN:** 该指南不仅讲安装，还覆盖 Chart 结构及 values 的语义。
- **EN:** Model bootstrap options such as S3 download and init containers are treated as first-class deployment concerns. **CN:** 模型初始化方案，如 S3 下载和 init container，被视为一等部署关注点。
