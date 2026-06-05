# metrics.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/usage/metrics.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page explains how to obtain production metrics from the vLLM API server through a Prometheus-style `/metrics` endpoint. It also documents metric families, optional MFU instrumentation, and the project’s deprecation policy for metric names.
- **CN:** 本文说明如何通过 Prometheus 风格的 `/metrics` 端点，从 vLLM API 服务获取生产监控指标。同时也介绍了指标分类、可选的 MFU 监控，以及指标名称的弃用策略。

## Key Concepts / 关键概念
- **EN:** Metrics are exposed by the OpenAI-compatible server and can be queried over HTTP after starting `vllm serve`.
  **CN:** 启动 `vllm serve` 后，可通过 HTTP 从 OpenAI 兼容服务拉取这些监控指标。
- **EN:** The documentation groups metrics into general, speculative decoding, NIXL KV connector, and performance categories.
  **CN:** 文档将指标划分为通用指标、推测解码指标、NIXL KV 连接器指标以及性能指标等类别。
- **EN:** MFU metrics are opt-in and require `--enable-mfu-metrics`.
  **CN:** MFU 指标是可选项，需要显式开启 `--enable-mfu-metrics`。
- **EN:** Deprecated metrics follow a staged policy: hidden in the next release, optionally re-enabled, then removed later.
  **CN:** 弃用指标遵循分阶段策略：下一版本先隐藏、可临时重新启用、再在后续版本中删除。
