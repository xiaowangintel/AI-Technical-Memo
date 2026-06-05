# instanttensor.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/extensions/instanttensor.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide explains how InstantTensor speeds up safetensors loading on CUDA through distributed loading, pipelined prefetching, direct I/O, and optional GPUDirect Storage.
  **CN:** 本指南说明 InstantTensor 如何通过分布式加载、流水线预取、直接 I/O 和可选的 GPUDirect Storage 来加速 CUDA 上的 safetensors 加载。
- **EN:** It is both a setup guide and a performance note: the page shows installation, the `--load-format instanttensor` flag, and benchmark tables with large load-time speedups.
  **CN:** 它同时兼具配置指南和性能说明：页面展示了安装方式、`--load-format instanttensor` 标志，以及具有明显提速效果的基准测试表。

## Key Concepts / 关键概念
- **EN:** Install with `pip install instanttensor` before serving.
  **CN:** 在启动服务前使用 `pip install instanttensor` 安装。
- **EN:** Use `vllm serve ... --load-format instanttensor` to activate the loader.
  **CN:** 使用 `vllm serve ... --load-format instanttensor` 启用该加载器。
- **EN:** Benchmark-driven positioning: the doc highlights much faster checkpoint load throughput than baseline safetensors.
  **CN:** 该文档以基准测试为依据，强调其检查点加载吞吐量显著快于基础 safetensors。
