# grouped_topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/grouped_topk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"Fused grouped top-k kernel for MoE routing (single-group, sigmoid scoring)."""". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""Fused grouped top-k kernel for MoE routing (single-group, sigmoid scoring)."""”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup
```python
"""Fused grouped top-k kernel for MoE routing (single-group, sigmoid scoring)."""

from __future__ import annotations

from typing import TYPE_CHECKING, Tuple

import torch

from sglang.jit_kernel.utils import cache_once, load_jit
from sglang.srt.utils.custom_op import register_custom_op

if TYPE_CHECKING:
    from tvm_ffi.module import Module


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 17-25: Function `_jit_grouped_topk_module`
```python
def _jit_grouped_topk_module() -> Module:
    return load_jit(
        "grouped_topk",
        cuda_files=["moe/grouped_topk.cuh"],
        cuda_wrappers=[("grouped_topk", "grouped_topk")],
    )


@register_custom_op(mutates_args=["topk_values", "topk_indices"])
```
**EN:** This block defines `_jit_grouped_topk_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_grouped_topk_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 26-50: Function `_jit_grouped_topk_op`
```python
def _jit_grouped_topk_op(
    scores: torch.Tensor,
    bias: torch.Tensor,
    topk_values: torch.Tensor,
    topk_indices: torch.Tensor,
    num_expert_group: int,
    topk_group: int,
    topk: int,
    renormalize: bool,
    scaling_factor: float,
) -> None:
    module = _jit_grouped_topk_module()
    module.grouped_topk(
        scores,
        bias,
        topk_values,
        topk_indices,
        num_expert_group,
        topk_group,
        topk,
        renormalize,
        scaling_factor,
    )
```
**EN:** This block defines `_jit_grouped_topk_op`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_grouped_topk_op`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 51-89: Function `grouped_topk`
```python
def grouped_topk(
    scores: torch.Tensor,
    bias: torch.Tensor,
    num_expert_group: int,
    topk_group: int,
    topk: int,
    renormalize: bool,
    scaling_factor: float,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Fused sigmoid + bias + top-k + renormalize for MoE routing.

    Replaces the naive PyTorch path that uses 3x torch.topk + scatter + masked_fill.
    Currently supports num_expert_group=1, topk_group=1, num_experts<=512, topk<=8.
    """
    num_tokens = scores.shape[0]

    topk_values = torch.empty(
        (num_tokens, topk), dtype=torch.float32, device=scores.device
    )
    topk_indices = torch.empty(
        (num_tokens, topk), dtype=torch.int32, device=scores.device
    )

    if num_tokens == 0:
        return topk_values, topk_indices

    _jit_grouped_topk_op(
        scores.contiguous(),
        bias.contiguous(),
        topk_values,
        topk_indices,
        num_expert_group,
        topk_group,
        topk,
        renormalize,
        scaling_factor,
    )
    return topk_values, topk_indices
```
**EN:** This block defines `grouped_topk`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `grouped_topk`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `sglang.srt.utils.custom_op -> register_custom_op`
- `tvm_ffi.module -> Module`
