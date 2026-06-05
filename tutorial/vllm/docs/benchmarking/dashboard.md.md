# dashboard.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/benchmarking/dashboard.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This page explains how vLLM performance results are collected, published, and compared. It describes the public dashboard, manual benchmark triggering in CI containers, runtime environment variables, and helper scripts that convert and compare JSON benchmark outputs.

**EN:** It also documents the continuous benchmarking loop, where scheduled workflows track serving, throughput, and latency trends over time to catch regressions or improvements.

**CN:** 这页说明 vLLM 的性能结果如何被采集、发布和对比。内容包括公开性能面板、如何在 CI 镜像中手动触发基准测试、可用的运行时环境变量，以及把 JSON 结果转成表格或做对比的辅助脚本。

**CN:** 它还介绍了持续基准测试机制：定时工作流会持续跟踪服务性能、吞吐量和延迟变化，用于发现性能回退或改进。

## Key Concepts / 关键概念
- **EN:** Performance dashboard — Publishes benchmark results so performance changes are visible across commits and merges.
- **CN:** 性能面板 — 把基准测试结果公开展示，便于按提交或合并观察性能变化。
- **EN:** Benchmark environment variables — Lets users customize benchmark JSONs, remote targets, and adaptive concurrency search thresholds.
- **CN:** 基准环境变量 — 允许用户定制基准 JSON、远端目标以及自适应并发搜索阈值。
- **EN:** Result comparison — Provides scripts to compare multiple benchmark result sets and summarize valid concurrency under SLA limits.
- **CN:** 结果对比 — 提供脚本对比多组基准结果，并在 SLA 约束下总结可用并发。
