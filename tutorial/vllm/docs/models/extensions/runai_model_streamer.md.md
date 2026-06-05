# runai_model_streamer.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/extensions/runai_model_streamer.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page describes Run:ai Model Streamer as a concurrent tensor reader that streams safetensors weights into GPU memory, with support for local paths and multiple object stores.
  **CN:** 本页将 Run:ai Model Streamer 描述为一个并发张量读取器，可将 safetensors 权重流式传入 GPU 内存，并支持本地路径及多种对象存储。
- **EN:** Beyond the basic `runai_streamer` loader, the doc spends significant effort on operational tuning such as distributed streaming, concurrency, memory limits, and sharded checkpoint patterns.
  **CN:** 除基础的 `runai_streamer` 加载器外，文档还重点介绍了运维层面的调优选项，如分布式流式加载、并发度、内存上限以及分片检查点命名模式。

## Key Concepts / 关键概念
- **EN:** Install optional support with `pip3 install vllm[runai]`.
  **CN:** 通过 `pip3 install vllm[runai]` 安装可选支持。
- **EN:** Supports `--load-format runai_streamer` and `--load-format runai_streamer_sharded`.
  **CN:** 支持 `--load-format runai_streamer` 与 `--load-format runai_streamer_sharded`。
- **EN:** `--model-loader-extra-config` controls parameters such as `distributed`, `concurrency`, `memory_limit`, and shard filename `pattern`.
  **CN:** `--model-loader-extra-config` 可控制 `distributed`、`concurrency`、`memory_limit` 以及分片文件名 `pattern` 等参数。
