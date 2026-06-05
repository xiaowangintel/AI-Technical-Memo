# communication_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/communication_op.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `communication_op`. It exposes primary entry points such as `tensor_model_parallel_all_reduce`, `tensor_model_parallel_quant_all_reduce`, `tensor_model_parallel_fused_allreduce_rmsnorm`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `communication_op` 的逻辑。 它对外提供的主要入口包括 `tensor_model_parallel_all_reduce`, `tensor_model_parallel_quant_all_reduce`, `tensor_model_parallel_fused_allreduce_rmsnorm`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/communication_op.py

from typing import Any, Dict, Optional, Tuple, Union

import torch
import torch.distributed

from .parallel_state import (
    get_attn_tp_group,
    get_moe_ep_group,
    get_moe_tp_group,
    get_tp_group,
)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 18-20: Function tensor_model_parallel_all_reduce
```python
def tensor_model_parallel_all_reduce(input_: torch.Tensor) -> torch.Tensor:
    """All-reduce the input tensor across model parallel group."""
    return get_tp_group().all_reduce(input_)
```
**EN:** This callable implements `tensor_model_parallel_all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "All-reduce the input tensor across model parallel group." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `tensor_model_parallel_all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 23-25: Function tensor_model_parallel_quant_all_reduce
```python
def tensor_model_parallel_quant_all_reduce(input_: torch.Tensor) -> torch.Tensor:
    """All-reduce the input tensor across model parallel group."""
    return get_tp_group().quant_all_reduce(input_)
```
**EN:** This callable implements `tensor_model_parallel_quant_all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "All-reduce the input tensor across model parallel group." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `tensor_model_parallel_quant_all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 28-40: Function tensor_model_parallel_fused_allreduce_rmsnorm
```python
def tensor_model_parallel_fused_allreduce_rmsnorm(
    input_: torch.Tensor,
    residual_inp_: torch.Tensor,
    weight_: torch.Tensor,
    eps: float,
) -> Optional[Tuple[torch.Tensor, torch.Tensor]]:
    """Fused TP all-reduce + RMSNorm.

    Policy and backend selection are owned by GroupCoordinator:
    it may dispatch to communicator-native fused APIs, custom fused kernels,
    or return None so callers can run generic fallback paths.
    """
    return get_tp_group().fused_allreduce_rmsnorm(input_, residual_inp_, weight_, eps)
```
**EN:** This callable implements `tensor_model_parallel_fused_allreduce_rmsnorm`. It takes `input_`, `residual_inp_`, `weight_`, `eps` and mainly reduces or aggregates values. The docstring states: "Fused TP all-reduce + RMSNorm." In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `tensor_model_parallel_fused_allreduce_rmsnorm`。它接收 `input_`, `residual_inp_`, `weight_`, `eps`，主要用于归约或聚合数值。 在这一范围内，它会管理模型权重或检查点。

### Lines 43-47: Function tensor_model_parallel_all_gather
```python
def tensor_model_parallel_all_gather(
    input_: torch.Tensor, dim: int = -1
) -> torch.Tensor:
    """All-gather the input tensor across model parallel group."""
    return get_tp_group().all_gather(input_, dim)
```
**EN:** This callable implements `tensor_model_parallel_all_gather`. It takes `input_`, `dim` and mainly implements tensor model parallel all gather. The docstring states: "All-gather the input tensor across model parallel group."
**CN:** 这一可调用对象实现了 `tensor_model_parallel_all_gather`。它接收 `input_`, `dim`，主要用于实现 tensor model parallel all gather 相关逻辑。

### Lines 50-54: Function tensor_model_parallel_gather
```python
def tensor_model_parallel_gather(
    input_: torch.Tensor, dst: int = 0, dim: int = -1
) -> Optional[torch.Tensor]:
    """Gather the input tensor across model parallel group."""
    return get_tp_group().gather(input_, dst, dim)
```
**EN:** This callable implements `tensor_model_parallel_gather`. It takes `input_`, `dst`, `dim` and mainly implements tensor model parallel gather. The docstring states: "Gather the input tensor across model parallel group."
**CN:** 这一可调用对象实现了 `tensor_model_parallel_gather`。它接收 `input_`, `dst`, `dim`，主要用于实现 tensor model parallel gather 相关逻辑。

### Lines 57-62: Function broadcast_tensor_dict
```python
def broadcast_tensor_dict(
    tensor_dict: Optional[Dict[Any, Union[torch.Tensor, Any]]] = None, src: int = 0
):
    if not torch.distributed.is_initialized():
        return tensor_dict
    return get_tp_group().broadcast_tensor_dict(tensor_dict, src)
```
**EN:** This callable implements `broadcast_tensor_dict`. It takes `tensor_dict`, `src` and mainly implements broadcast tensor dict. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `broadcast_tensor_dict`。它接收 `tensor_dict`, `src`，主要用于实现 broadcast tensor dict 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 65-67: Function attention_tensor_model_parallel_all_reduce
```python
def attention_tensor_model_parallel_all_reduce(input_: torch.Tensor) -> torch.Tensor:
    """All-reduce the input tensor across attention parallel group."""
    return get_attn_tp_group().all_reduce(input_)
```
**EN:** This callable implements `attention_tensor_model_parallel_all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "All-reduce the input tensor across attention parallel group." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `attention_tensor_model_parallel_all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 70-74: Function attention_tensor_model_parallel_quant_all_reduce
```python
def attention_tensor_model_parallel_quant_all_reduce(
    input_: torch.Tensor,
) -> torch.Tensor:
    """All-reduce the input tensor across attention parallel group."""
    return get_attn_tp_group().quant_all_reduce(input_)
```
**EN:** This callable implements `attention_tensor_model_parallel_quant_all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "All-reduce the input tensor across attention parallel group." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `attention_tensor_model_parallel_quant_all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 77-79: Function moe_tensor_model_parallel_all_reduce
```python
def moe_tensor_model_parallel_all_reduce(input_: torch.Tensor) -> torch.Tensor:
    """All-reduce the input tensor across moe parallel group."""
    return get_moe_tp_group().all_reduce(input_)
```
**EN:** This callable implements `moe_tensor_model_parallel_all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "All-reduce the input tensor across moe parallel group." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `moe_tensor_model_parallel_all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 82-84: Function moe_expert_parallel_all_reduce
```python
def moe_expert_parallel_all_reduce(input_: torch.Tensor) -> torch.Tensor:
    """All-reduce the input tensor across moe expert parallel group."""
    return get_moe_ep_group().all_reduce(input_)
```
**EN:** This callable implements `moe_expert_parallel_all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "All-reduce the input tensor across moe expert parallel group." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `moe_expert_parallel_all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

## Key Concepts / 关键概念
- `tensor_model_parallel_all_reduce`: reduces or aggregates values / 归约或聚合数值
- `tensor_model_parallel_quant_all_reduce`: reduces or aggregates values / 归约或聚合数值
- `tensor_model_parallel_fused_allreduce_rmsnorm`: reduces or aggregates values / 归约或聚合数值
- `tensor_model_parallel_all_gather`: implements tensor model parallel all gather / 实现 tensor model parallel all gather 相关逻辑
- `tensor_model_parallel_gather`: implements tensor model parallel gather / 实现 tensor model parallel gather 相关逻辑
- `broadcast_tensor_dict`: implements broadcast tensor dict / 实现 broadcast tensor dict 相关逻辑
- `attention_tensor_model_parallel_all_reduce`: reduces or aggregates values / 归约或聚合数值
- `attention_tensor_model_parallel_quant_all_reduce`: reduces or aggregates values / 归约或聚合数值
- `moe_tensor_model_parallel_all_reduce`: reduces or aggregates values / 归约或聚合数值
- `moe_expert_parallel_all_reduce`: reduces or aggregates values / 归约或聚合数值

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `.parallel_state`
