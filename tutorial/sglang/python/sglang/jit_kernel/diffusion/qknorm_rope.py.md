# qknorm_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/qknorm_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and module setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import torch

from sglang.jit_kernel.utils import (
    cache_once,
    is_arch_support_pdl,
    load_jit,
    make_cpp_args,
)
from sglang.srt.utils.custom_op import register_custom_op

if TYPE_CHECKING:
    from tvm_ffi.module import Module


logger = logging.getLogger(__name__)


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 24-40: Function `_jit_qknorm_rope_module`
```python
def _jit_qknorm_rope_module(
    head_dim: int,
    rope_dim: int,
    is_neox: bool,
    dtype: torch.dtype,
) -> Module:
    args = make_cpp_args(head_dim, rope_dim, is_neox, is_arch_support_pdl(), dtype)
    return load_jit(
        "qknorm_rope",
        *args,
        cuda_files=["diffusion/qknorm_rope.cuh"],
        cuda_wrappers=[("qknorm_rope", f"QKNormRopeKernel<{args}>::run")],
    )


@torch.compiler.assume_constant_result
@cache_once
```
**EN:** This block defines `_jit_qknorm_rope_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_qknorm_rope_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 41-80: Function `can_use_fused_inplace_qknorm_rope`
```python
def can_use_fused_inplace_qknorm_rope(
    head_dim: int,
    rope_dim: int,
    is_neox: bool,
    dtype: torch.dtype,
) -> bool:
    if head_dim not in (64, 128, 256):
        logger.warning(f"Unsupported head_dim={head_dim} for JIT fused QKNorm+RoPE")
        return False
    if rope_dim <= 0 or rope_dim > head_dim:
        logger.warning(
            f"Unsupported rope_dim={rope_dim} for head_dim={head_dim} in fused QKNorm+RoPE"
        )
        return False
    elems_per_thread = head_dim // 32
    if rope_dim % elems_per_thread != 0:
        logger.warning(
            "rope_dim=%s must be divisible by per-thread width=%s for fused QKNorm+RoPE",
            rope_dim,
            elems_per_thread,
        )
        return False
    if is_neox:
        rotary_lanes = rope_dim // elems_per_thread
        if rotary_lanes < 2 or rotary_lanes & (rotary_lanes - 1):
            logger.warning(
                "rope_dim=%s yields invalid rotary_lanes=%s for neox fused QKNorm+RoPE; rotary lane count must be a power of 2",
                rope_dim,
                rotary_lanes,
            )
            return False
    try:
        _jit_qknorm_rope_module(head_dim, rope_dim, is_neox, dtype)
        return True
    except Exception as e:
        logger.warning(f"Failed to load JIT fused QKNorm+RoPE kernel: {e}")
        return False


@register_custom_op(mutates_args=["q", "k"])
```
**EN:** This block defines `can_use_fused_inplace_qknorm_rope`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `can_use_fused_inplace_qknorm_rope`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 81-97: Function `fused_inplace_qknorm_rope`
```python
def fused_inplace_qknorm_rope(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    positions: torch.Tensor,
    *,
    is_neox: bool,
    eps: float = 1e-6,
    head_dim: int = 0,
    rope_dim: int = 0,
) -> None:
    head_dim = head_dim or q.size(-1)
    rope_dim = rope_dim or cos_sin_cache.size(-1)
    module = _jit_qknorm_rope_module(head_dim, rope_dim, is_neox, q.dtype)
    module.qknorm_rope(q, k, q_weight, k_weight, cos_sin_cache, positions, eps)
```
**EN:** This block defines `fused_inplace_qknorm_rope`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_inplace_qknorm_rope`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `logging`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> (`
- `sglang.srt.utils.custom_op -> register_custom_op`
- `tvm_ffi.module -> Module`
