# data_parallel_deployment.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/data_parallel_deployment.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page describes data parallel deployment, where full model replicas run on separate GPUs or nodes to serve independent request batches.
  **CN:** 本页介绍 data parallel 部署：完整模型副本运行在不同 GPU 或节点上，以处理彼此独立的请求批次。
- **EN:** It is operationally focused, covering launch topology, internal load balancing, Ray-based multi-node startup, and combinations with tensor or expert parallelism.
  **CN:** 它偏重运维实践，覆盖启动拓扑、内部负载均衡、基于 Ray 的多节点启动，以及与 tensor/expert parallelism 的组合方式。

## Key Concepts / 关键概念
- **EN:** Data parallelism replicates weights instead of sharding them.
  **CN:** data parallelism 通过复制权重而非切分权重来扩展。
- **EN:** The guide documents single-command multi-node launch patterns and DP-specific arguments.
  **CN:** 该指南记录了单命令多节点启动模式以及 DP 专用参数。
- **EN:** Dense models and MoE models can both use this deployment style.
  **CN:** 稠密模型与 MoE 模型都可以使用这种部署方式。
