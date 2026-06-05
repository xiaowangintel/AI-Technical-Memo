# usage_stats.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/usage/usage_stats.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page explains vLLM’s anonymous usage telemetry, why it exists, what kinds of environment/model metadata it includes, and how users can inspect or disable it. The tone is transparency-focused and emphasizes the absence of sensitive data.
- **CN:** 本文介绍 vLLM 的匿名使用遥测：为何收集、包含哪些环境与模型元数据，以及用户如何查看或关闭它。整体基调强调透明性，并指出数据不包含敏感信息。

## Key Concepts / 关键概念
- **EN:** Usage stats are collected by default to help the maintainers prioritize common hardware and workload patterns.
  **CN:** 默认会收集使用统计，以帮助维护者优先支持更常见的硬件与工作负载模式。
- **EN:** The documentation points readers to the source code for the authoritative list of collected fields.
  **CN:** 文档将读者引导到源码位置，以查看最新、权威的采集字段列表。
- **EN:** Users can inspect the local JSON record to see exactly what has been logged.
  **CN:** 用户可以查看本地 JSON 记录，明确知道实际被记录了哪些内容。
- **EN:** Opt-out is supported through environment variables or a local do-not-track file.
  **CN:** 用户可以通过环境变量或本地 do-not-track 文件来退出统计收集。
