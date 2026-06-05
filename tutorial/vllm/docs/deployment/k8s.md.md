# k8s.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/k8s.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document is the native Kubernetes deployment guide for vLLM. It covers CPU and GPU variants, gRPC serving, troubleshooting, and probe-related failures, making it the most direct reference for running upstream vLLM on standard Kubernetes resources.
- **CN:** 本文档是 vLLM 的原生 Kubernetes 部署指南。它覆盖 CPU 与 GPU 两种部署方式、gRPC 服务、故障排查以及探针失败问题，是在标准 Kubernetes 资源上运行上游 vLLM 的直接参考。

## Key Concepts / 关键概念
- **EN:** The guide treats Kubernetes manifests as the canonical deployment interface. **CN:** 该指南将 Kubernetes 清单视为规范的部署接口。
- **EN:** Support for both CPU and GPU deployments broadens the operational scope. **CN:** 同时支持 CPU 和 GPU 部署，拓宽了运维适用范围。
- **EN:** Troubleshooting readiness and startup behavior is an important practical theme. **CN:** 对 readiness 和 startup 行为的排障是一个重要实践主题。
