# nccl.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/training/weight_transfer/nccl.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document describes the NCCL backend for sending updated weights from a trainer to inference workers across separate GPUs or nodes. It focuses on distributed process-group setup, broadcast flow, and packed tensor optimization.
- **CN:** 本文介绍 NCCL 后端，用于在独立 GPU 或多节点之间把训练端更新后的权重广播到推理 worker。重点是分布式进程组初始化、广播流程以及打包张量优化。

## Key Concepts / 关键概念
- **EN:** NCCL is the preferred backend when training and inference are separated across GPUs, often with tensor parallel inference.
  **CN:** 当训练与推理分布在不同 GPU 上，尤其涉及张量并行推理时，NCCL 是首选后端。
- **EN:** Trainer and workers join a shared process group, with the trainer fixed at rank 0 and workers shifted by `rank_offset`.
  **CN:** 训练端与 worker 会加入同一个进程组，其中训练端固定为 rank 0，而 worker 通过 `rank_offset` 偏移。
- **EN:** Packed broadcasting combines small tensors into larger buffers to reduce NCCL calls and overlap stages on CUDA streams.
  **CN:** 打包广播会把小张量合并成大缓冲区，减少 NCCL 调用次数，并在 CUDA stream 上实现阶段重叠。
- **EN:** The inference side still follows the generic start/update/finish protocol, and metadata order must match trainer iteration order.
  **CN:** 推理端依然遵循通用的 start/update/finish 协议，且元数据顺序必须与训练端遍历顺序一致。
