# fused_qknorm_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/fused_qknorm_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Optional

import torch

from sglang.jit_kernel.utils import cache_once, load_jit
from sglang.srt.utils.custom_op import register_custom_op

if TYPE_CHECKING:
    from tvm_ffi.module import Module


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 16-36: Function `_jit_fused_qknorm_rope_module`
```python
def _jit_fused_qknorm_rope_module(head_dim: int, is_neox: bool, yarn: bool) -> Module:
    return load_jit(
        "fused_qknorm_rope",
        head_dim,
        int(is_neox),
        int(yarn),
        cuda_files=["elementwise/fused_qknorm_rope.cuh"],
        cuda_wrappers=[("fused_qk_norm_rope", "fused_qk_norm_rope")],
        extra_cuda_cflags=[
            "--use_fast_math",
            f"-DJIT_HEAD_DIM={head_dim}",
            f"-DJIT_INTERLEAVE={0 if is_neox else 1}",
            f"-DJIT_YARN={1 if yarn else 0}",
        ],
    )


@register_custom_op(
    op_name="fused_qk_norm_rope_out",
    mutates_args=["qkv"],
)
```
**EN:** This block defines `_jit_fused_qknorm_rope_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_fused_qknorm_rope_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 37-100: Function `fused_qk_norm_rope_out`
```python
def fused_qk_norm_rope_out(
    qkv: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    position_ids: torch.Tensor,
    num_heads_q: int,
    num_heads_k: int,
    num_heads_v: int,
    head_dim: int,
    eps: float,
    base: float,
    is_neox: bool,
    factor: float,
    low: float,
    high: float,
    attention_factor: float,
    rotary_dim: int,
) -> None:
    """
    Fused QK RMSNorm + RoPE applied in-place on the QKV tensor.

    Matches the call signature of ``sgl_kernel.fused_qk_norm_rope``.

    Args:
        qkv:              [num_tokens, (nq+nk+nv)*head_dim] bfloat16 — modified in-place
        q_weight:         [head_dim] bfloat16 — RMSNorm weights for Q
        k_weight:         [head_dim] bfloat16 — RMSNorm weights for K
        position_ids:     [num_tokens] int32
        num_heads_q:      number of query heads
        num_heads_k:      number of key heads
        num_heads_v:      number of value heads
        head_dim:         head dimension; must be 64, 128, or 256
        eps:              epsilon for RMSNorm
        base:             RoPE base frequency
        is_neox:          True → NeoX style, False → interleave (GPT-J) style
        factor:           YaRN scaling factor (1.0 = standard RoPE)
        low:              YaRN low-frequency threshold
        high:             YaRN high-frequency threshold
        attention_factor: scale applied to the rotary component
        rotary_dim:       number of elements per head to apply RoPE to
    """
    yarn = factor != 1.0
    module = _jit_fused_qknorm_rope_module(head_dim, is_neox, yarn)
    module.fused_qk_norm_rope(
        qkv,
        q_weight,
        k_weight,
        position_ids,
        num_heads_q,
        num_heads_k,
        num_heads_v,
        head_dim,
        eps,
        base,
        1 if is_neox else 0,
        factor,
        low,
        high,
        attention_factor,
        rotary_dim,
    )


@cache_once
```
**EN:** This block defines `fused_qk_norm_rope_out`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_qk_norm_rope_out`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 101-129: Function `can_use_fused_qk_norm_rope`
```python
def can_use_fused_qk_norm_rope(
    head_dim: int, is_neox: bool, dtype: torch.dtype, yarn: bool = False
) -> bool:
    """Return True if the JIT fused QK-Norm + RoPE kernel can be used.

    Args:
        head_dim: head dimension; supported values are 64, 128, 256
        dtype: tensor dtype; only bfloat16 is supported
        yarn: whether YaRN scaling is active (factor != 1.0); prebuilds the
              correct kernel variant so no extra JIT compile occurs on the
              first real call.
    """
    logger = logging.getLogger(__name__)
    if head_dim not in (64, 128, 256):
        logger.warning(
            f"Unsupported head_dim={head_dim} for JIT fused_qk_norm_rope kernel"
        )
        return False
    if dtype != torch.bfloat16:
        logger.warning(f"Unsupported dtype={dtype} for JIT fused_qk_norm_rope kernel")
        return False
    try:
        _jit_fused_qknorm_rope_module(head_dim, is_neox, yarn)
        return True
    except Exception as e:
        logger.warning(f"Failed to load JIT fused_qk_norm_rope kernel: {e}")
        return False
```
**EN:** This block defines `can_use_fused_qk_norm_rope`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `can_use_fused_qk_norm_rope`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 130-190: Function `fused_qk_norm_rope`
```python
def fused_qk_norm_rope(
    qkv: torch.Tensor,
    num_heads_q: int,
    num_heads_k: int,
    num_heads_v: int,
    head_dim: int,
    eps: float,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    base: float,
    is_neox: bool,
    position_ids: torch.Tensor,
    factor: float,
    low: float,
    high: float,
    attention_factor: float,
    rotary_dim: Optional[int] = None,
) -> None:
    """
    Fused QK RMSNorm + RoPE applied in-place on the QKV tensor.

    Matches the call signature of ``sgl_kernel.fused_qk_norm_rope``.

    Args:
        qkv:              [num_tokens, (nq+nk+nv)*head_dim] bfloat16 — modified in-place
        num_heads_q:      number of query heads
        num_heads_k:      number of key heads
        num_heads_v:      number of value heads
        head_dim:         head dimension; must be 64, 128, or 256
        eps:              epsilon for RMSNorm
        q_weight:         [head_dim] bfloat16 — RMSNorm weights for Q
        k_weight:         [head_dim] bfloat16 — RMSNorm weights for K
        base:             RoPE base frequency
        is_neox:          True → NeoX style, False → interleave (GPT-J) style
        position_ids:     [num_tokens] int32
        factor:           YaRN scaling factor (1.0 = standard RoPE)
        low:              YaRN low-frequency threshold
        high:             YaRN high-frequency threshold
        attention_factor: scale applied to the rotary component
        rotary_dim:       elements per head to rotate; defaults to head_dim
    """
    if rotary_dim is None:
        rotary_dim = head_dim
    fused_qk_norm_rope_out(
        qkv,
        q_weight,
        k_weight,
        position_ids,
        num_heads_q,
        num_heads_k,
        num_heads_v,
        head_dim,
        eps,
        base,
        is_neox,
        factor,
        low,
        high,
        attention_factor,
        rotary_dim,
    )
```
**EN:** This block defines `fused_qk_norm_rope`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_qk_norm_rope`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `logging`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `sglang.srt.utils.custom_op -> register_custom_op`
- `tvm_ffi.module -> Module`
