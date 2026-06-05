# sweeps.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/benchmarking/sweeps.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This guide explains the `vllm bench sweep` family. It covers sweeping server and benchmark parameters, workload exploration, startup benchmarking, and plotting utilities for performance curves and Pareto frontiers.

**EN:** The document emphasizes experiment structure: base commands, JSON parameter grids, Cartesian products, resumability, and visualization dimensions that turn raw benchmark runs into tunable engineering decisions.

**CN:** 这份指南介绍 `vllm bench sweep` 命令族，包括服务参数扫参、工作负载探索、启动时间基准，以及用于绘制性能曲线和 Pareto 前沿的可视化工具。

**CN:** 文档重点不只是命令本身，还强调实验组织方式：基础命令、JSON 参数网格、笛卡尔积组合、断点续跑，以及如何把原始结果转成可调优的工程结论。

## Key Concepts / 关键概念
- **EN:** Parameter grids — Uses JSON files to define repeatable combinations for server-side and benchmark-side settings.
- **CN:** 参数网格 — 通过 JSON 文件定义可复现的服务端和基准端参数组合。
- **EN:** Workload explorer — Searches latency/throughput tradeoffs by varying request rate or concurrency.
- **CN:** 负载探索器 — 通过改变请求率或并发度来搜索延迟与吞吐之间的平衡点。
- **EN:** Plotting tools — Transforms sweep outputs into curves and Pareto charts for decision making.
- **CN:** 绘图工具 — 把扫参结果转成曲线图和 Pareto 图，支持配置决策。
