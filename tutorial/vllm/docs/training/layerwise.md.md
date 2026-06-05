# layerwise.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/training/layerwise.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document introduces layerwise reloading, a mechanism for replacing weights in existing destinations without recompiling CUDA graphs. It is positioned for post-training and QeRL-style workflows, including online quantization and incremental processing.
- **CN:** 本文介绍逐层重载机制：在不重新编译 CUDA graph 的前提下，将新权重写入既有权重位置。它主要面向后训练与 QeRL 类流程，并覆盖在线量化与增量处理。

## Key Concepts / 关键概念
- **EN:** The workflow breaks weight updates into transfer, fuse, process, shard, and copy stages.
  **CN:** 该流程将权重更新拆分为传输、融合、处理、分片和拷贝五个阶段。
- **EN:** Online quantization is treated as a processing step, allowing full-precision checkpoints to be quantized during load.
  **CN:** 在线量化被视为“处理”步骤，因此可以在加载过程中把全精度检查点即时量化。
- **EN:** vLLM exposes high-level weight transfer, mid-level `reload_weights`, and low-level lifecycle hooks for custom integrations.
  **CN:** vLLM 同时暴露高层权重传输接口、中层 `reload_weights` 接口，以及低层生命周期钩子以便自定义集成。
- **EN:** Loading weights out of layer order increases temporary buffering and can trigger avoidable device-memory warnings.
  **CN:** 若不按层顺序加载权重，会增加临时缓冲占用，并可能触发本可避免的显存警告。
