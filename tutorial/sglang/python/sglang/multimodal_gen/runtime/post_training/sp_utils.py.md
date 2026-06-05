# sp_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/post_training/sp_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for sp utils in the multimodal generation stack. Key symbols include `should_do_sp_collective`, `gather_stacked_latents_for_sp`, `all_reduce_if_sp_sharded`. / 该模块包含多模态生成体系中与 sp utils 相关的运行时支持代码。 关键符号包括 `should_do_sp_collective`, `gather_stacked_latents_for_sp`, `all_reduce_if_sp_sharded`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""Sequence Parallel helpers for post-training rollout code."""

from __future__ import annotations

import torch

from sglang.multimodal_gen.runtime.distributed import (
    get_local_torch_device,
    get_sp_world_size,
)
from sglang.multimodal_gen.runtime.distributed.communication_op import (
    sequence_model_parallel_all_gather,
    sequence_model_parallel_all_reduce,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 18-19: Function `should_do_sp_collective` / 函数 `should_do_sp_collective`
```python
def should_do_sp_collective(batch) -> bool:
    return get_sp_world_size() > 1 and getattr(batch, "did_sp_shard_latents", False)
```
**EN:** This function drives `should_do_sp_collective` with inputs such as `batch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `should_do_sp_collective`，主要处理 `batch` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 22-36: Function `gather_stacked_latents_for_sp` / 函数 `gather_stacked_latents_for_sp`
```python
def gather_stacked_latents_for_sp(
    pipeline_config,
    batch,
    stacked_latents: torch.Tensor,
) -> torch.Tensor:
    if not should_do_sp_collective(batch):
        return stacked_latents
    if stacked_latents.dim() < 2:
        return stacked_latents
    bsz, t_steps = stacked_latents.shape[0], stacked_latents.shape[1]
    flat_inputs = stacked_latents.flatten(0, 1).contiguous()
    gathered_flat_inputs = pipeline_config.gather_latents_for_sp(
        flat_inputs, batch=batch
    )
    return gathered_flat_inputs.unflatten(0, (bsz, t_steps))
```
**EN:** This function drives `gather_stacked_latents_for_sp` with inputs such as `pipeline_config`, `batch`, `stacked_latents`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `gather_stacked_latents_for_sp`，主要处理 `pipeline_config`, `batch`, `stacked_latents` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 39-44: Function `all_reduce_if_sp_sharded` / 函数 `all_reduce_if_sp_sharded`
```python
def all_reduce_if_sp_sharded(batch, tensor: torch.Tensor) -> torch.Tensor:
    if not should_do_sp_collective(batch):
        return tensor
    tensor = tensor.to(get_local_torch_device())
    sequence_model_parallel_all_reduce(tensor)
    return tensor
```
**EN:** This function drives `all_reduce_if_sp_sharded` with inputs such as `batch`, `tensor`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `all_reduce_if_sp_sharded`，主要处理 `batch`, `tensor` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 47-51: Function `all_gather_if_sp_sharded` / 函数 `all_gather_if_sp_sharded`
```python
def all_gather_if_sp_sharded(batch, x: torch.Tensor, dim: int = 0) -> torch.Tensor:
    if not should_do_sp_collective(batch):
        return x
    x = x.to(get_local_torch_device()).contiguous()
    return sequence_model_parallel_all_gather(x, dim=dim)
```
**EN:** This function drives `all_gather_if_sp_sharded` with inputs such as `batch`, `x`, `dim`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `all_gather_if_sp_sharded`，主要处理 `batch`, `x`, `dim` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 54-61: Function `maybe_trim_sp_rope_seq_for_batch` / 函数 `maybe_trim_sp_rope_seq_for_batch`
```python
def maybe_trim_sp_rope_seq_for_batch(batch, rope: torch.Tensor) -> torch.Tensor:
    raw = getattr(batch, "raw_latent_shape", None)
    if raw is None or len(raw) < 2:
        return rope
    target = int(raw[1])
    if rope.shape[0] > target:
        return rope[:target]
    return rope
```
**EN:** This function drives `maybe_trim_sp_rope_seq_for_batch` with inputs such as `batch`, `rope`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `maybe_trim_sp_rope_seq_for_batch`，主要处理 `batch`, `rope` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Symbol `should_do_sp_collective` anchors the module API / 符号 `should_do_sp_collective` 构成该模块的核心 API
- Symbol `gather_stacked_latents_for_sp` anchors the module API / 符号 `gather_stacked_latents_for_sp` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.communication_op`
- **External / 外部**: `__future__`, `torch`
