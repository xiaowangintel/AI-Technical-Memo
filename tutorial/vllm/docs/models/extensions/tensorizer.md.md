# tensorizer.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/extensions/tensorizer.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document presents Tensorizer as a two-stage workflow: first serialize model weights, then deserialize them directly to GPU from disk, HTTP(S), or S3 for faster startup and lower CPU memory use.
  **CN:** 该文档将 Tensorizer 描述为一个两阶段工作流：先序列化模型权重，再从磁盘、HTTP(S) 或 S3 直接反序列化到 GPU，以获得更快启动速度和更低 CPU 内存占用。
- **EN:** The guide is practical rather than theoretical, covering serialization commands, serving via `vllm serve` or `LLM`, LoRA-aware artifacts, and advanced serializer/deserializer kwargs.
  **CN:** 该指南更偏实践而非原理说明，涵盖序列化命令、通过 `vllm serve` 或 `LLM` 提供服务、支持 LoRA 的产物，以及高级序列化/反序列化参数传递。

## Key Concepts / 关键概念
- **EN:** Install the extension with `pip install vllm[tensorizer]`.
  **CN:** 使用 `pip install vllm[tensorizer]` 安装该扩展。
- **EN:** Serve tensorized checkpoints through `--load-format tensorizer`.
  **CN:** 通过 `--load-format tensorizer` 加载并提供 tensorized 检查点。
- **EN:** `model_loader_extra_config` can forward `serialization_kwargs` and `deserialization_kwargs` for fine-grained control.
  **CN:** `model_loader_extra_config` 可透传 `serialization_kwargs` 与 `deserialization_kwargs` 进行细粒度控制。
