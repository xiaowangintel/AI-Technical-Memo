# _fa3.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/attention/_fa3.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements attention-specific abstractions and helpers for modern sequence models.
- **Purpose (CN)**: 实现现代序列模型中的注意力相关抽象与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-41
```python
"""
PROTOTYPE!
Flash Attention 3 implementation.
For fp8: only supports forward pass right now.
For fp16/bf16: supports forward and backward pass.
"""
# mypy: allow-untyped-defs

from __future__ import annotations

import importlib
import warnings
from typing import TYPE_CHECKING


if TYPE_CHECKING:
    from collections.abc import Callable

from dataclasses import dataclass
from functools import cache
from typing_extensions import TypeVarTuple, Unpack

import torch
from torch.library import Library

from . import _registry


__all__ = [
    "register_flash_attention_fa3",
]


_FA3_CUDA_FWD: Callable | None = None  # Cache for torch.ops.flash_attn_3.fwd
_FA3_CUDA_BWD: Callable | None = None  # Cache for torch.ops.flash_attn_3.bwd


@dataclass
class _FA3Handle:
    library: Library | None
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 42-70
```python
    def remove(self) -> None:
        self.library = None
        # Clear the C++ flag
        torch._C._set_sdp_use_fa3(False)


@cache
def _get_device_major(device: torch.device) -> int:
    major, _ = torch.cuda.get_device_capability(device)
    return major


def register_flash_attention_fa3(
    module_path: str = "flash_attn_interface",
) -> _FA3Handle:
    """
    Register FA3 flash attention kernels with the PyTorch dispatcher.

    Args:
        module_path: Python module path to the FA3 implementation.
    """
    _fa3_import_module(module_path)

    # Expose FA3 registration status to C++
    torch._C._set_sdp_use_fa3(True)

    return _FA3Handle(_fa3_register_kernels())
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 71-111
```python
def _fa3_import_module(module_path: str) -> None:
    importlib.import_module(module_path)
    if not hasattr(torch.ops, "flash_attn_3"):
        raise RuntimeError(f"Module '{module_path}' does not expose FA3 kernels")
    if not hasattr(torch.ops.flash_attn_3, "fwd"):
        raise RuntimeError(
            f"Module '{module_path}' does not expose FA3 forward kernels"
        )
    if not hasattr(torch.ops.flash_attn_3, "bwd"):
        raise RuntimeError(
            f"Module '{module_path}' does not expose FA3 backward kernels"
        )
    global _FA3_CUDA_FWD, _FA3_CUDA_BWD
    _FA3_CUDA_FWD = torch.ops.flash_attn_3.fwd
    _FA3_CUDA_BWD = torch.ops.flash_attn_3.bwd


def _fa3_register_kernels() -> Library:
    lib = Library("aten", "IMPL", "CUDA")  # noqa: TOR901
    lib.impl(
        "_flash_attention_forward.quantized", _fa3_flash_attention_forward_impl, "CUDA"
    )
    lib.impl(
        "_scaled_dot_product_flash_attention.quantized",
        _fa3_scaled_dot_product_flash_attention_forward_impl,
        "CUDA",
    )
    lib.impl(
        "_flash_attention_forward", _fa3_flash_attention_forward_impl_default, "CUDA"
    )
    lib.impl(
        "_flash_attention_forward_no_dropout_inplace",
        _fa3_flash_attention_forward_no_dropout_inplace_impl,
        "CUDA",
    )
    lib.impl(
        "_scaled_dot_product_flash_attention",
        _fa3_scaled_dot_product_flash_attention_forward_impl_default,
        "CUDA",
    )
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 112-139
```python
    lib.impl("_flash_attention_backward", _fa3_flash_attention_backward_impl, "CUDA")
    lib.impl(
        "_scaled_dot_product_flash_attention_backward",
        _fa3_scaled_dot_product_flash_attention_backward_impl,
        "CUDA",
    )
    return lib


def _fa3_common_support_error(
    query: torch.Tensor,
    tensors: tuple[torch.Tensor, ...],
    dropout_p: float,
    cum_seq_q: torch.Tensor | None,
    q_descale: torch.Tensor | None,
    k_descale: torch.Tensor | None,
    v_descale: torch.Tensor | None,
) -> str | None:
    if dropout_p != 0.0:
        return "dropout_p must be 0"

    if not all(t.is_cuda for t in tensors):
        return "inputs must be CUDA tensors"
    if len({t.device for t in tensors}) != 1:
        return "inputs must share device"
    if query.dtype == torch.float8_e4m3fn and (
        q_descale is None or k_descale is None or v_descale is None
    ):
```
- **EN**: This module-level block helps implement attention-specific transformations and bookkeeping. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于实现注意力相关的变换与簿记逻辑。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 140-167
```python
        warnings.warn(
            "When using SDPA with fp8, descale tensor should always be used"
            " for accurate dequantization. Please use "
            "_scaled_dot_product_attention_quantized and "
            "provide the descale tensors.",
            UserWarning,
        )
    if cum_seq_q is None and query.dim() != 4:
        return "dense query must be 4D"
    if cum_seq_q is not None and query.dim() != 3:
        return "ragged query must be 3D"
    if not torch.cuda.is_available():
        return "CUDA not available"
    if _get_device_major(query.device) != 9:
        return "FA3 requires compute capability 9.0"
    return None


def _fa3_forward_support_error(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    dropout_p: float,
    return_debug_mask: bool,
    alibi_slopes: torch.Tensor | None,
    seqused_k: torch.Tensor | None,
    cum_seq_q: torch.Tensor | None,
    q_descale: torch.Tensor | None,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 168-200
```python
    k_descale: torch.Tensor | None,
    v_descale: torch.Tensor | None,
) -> str | None:
    if return_debug_mask:
        return "return_debug_mask must be False"
    if alibi_slopes is not None:
        return "alibi_slopes not supported"
    if seqused_k is not None:
        if seqused_k.dtype != torch.int32:
            return "seqused_k must be int32"
        if not seqused_k.is_cuda:
            return "seqused_k must be CUDA"
    supported_dtypes = (torch.float8_e4m3fn, torch.float16, torch.bfloat16)
    if not all(t.dtype in supported_dtypes for t in {query, key, value}):
        return f"inputs must be one of {supported_dtypes}"
    if len({t.dtype for t in {query, key, value}}) != 1:
        return "all inputs must have the same dtype"
    error = _fa3_common_support_error(
        query,
        (query, key, value),
        dropout_p,
        cum_seq_q,
        q_descale,
        k_descale,
        v_descale,
    )
    if error is not None:
        if error == "inputs must share device":
            return "query, key, value must be on same device"
        return error
    return None
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 201-241
```python
def _fa3_backward_support_error(
    grad_out: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    out: torch.Tensor,
    logsumexp: torch.Tensor,
    dropout_p: float,
    cum_seq_q: torch.Tensor | None,
    window_size_left: int | None,
    window_size_right: int | None,
) -> str | None:
    # FA3 backward ONLY supports fp16/bf16, NOT fp8
    if query.dtype == torch.float8_e4m3fn:
        return (
            "FA3 backward does not support fp8 - use inference only (torch.no_grad())"
        )
    if logsumexp.dtype != torch.float32:
        return "logsumexp dtype must be float32"
    supported_dtypes = (torch.float16, torch.bfloat16)
    if not all(t.dtype in supported_dtypes for t in {grad_out, query, key, value, out}):
        return f"inputs must be one of {supported_dtypes}"
    if len({t.dtype for t in {grad_out, query, key, value, out}}) != 1:
        return "all inputs must have the same dtype"
    error = _fa3_common_support_error(
        query,
        (grad_out, query, key, value, out, logsumexp),
        dropout_p,
        cum_seq_q,
        None,
        None,
        None,
    )
    if error is not None:
        return error
    return None


Ts = TypeVarTuple("Ts")
```
- **EN**: This module-level block helps connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 242-269
```python
def _transpose_dense(*tensors: Unpack[Ts]) -> tuple[Unpack[Ts]]:
    return tuple(t.transpose(1, 2) for t in tensors)  # type: ignore[attr-defined]


def _maybe_contiguous(x: torch.Tensor | None) -> torch.Tensor | None:
    """Ensure tensor is contiguous in the last dimension."""
    return x.contiguous() if x is not None and x.stride(-1) != 1 else x


def _fa3_run_forward(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    cu_seq_q: torch.Tensor | None,
    cu_seq_k: torch.Tensor | None,
    max_q: int,
    max_k: int,
    scale: float | None,
    is_causal: bool,
    window_size_left: int | None,
    window_size_right: int | None,
    seqused_k: torch.Tensor | None,
    out: torch.Tensor | None = None,
    q_descale: torch.Tensor | None = None,
    k_descale: torch.Tensor | None = None,
    v_descale: torch.Tensor | None = None,
    block_table: torch.Tensor | None = None,
    num_splits: int | None = None,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 270-297
```python
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Run the FA3 forward pass by calling the C++ kernel directly.
    """
    if _FA3_CUDA_FWD is None:
        raise RuntimeError("FA3 not registered")
    # Ensure contiguous in the last dimension
    q = _maybe_contiguous(query)
    k = _maybe_contiguous(key)
    v = (
        value.contiguous()
        if value.dtype == torch.float8_e4m3fn
        and value.stride(-1) != 1
        and value.stride(-3) != 1
        else _maybe_contiguous(value)
    )

    cu_seqlens_q = _maybe_contiguous(cu_seq_q)
    cu_seqlens_k = _maybe_contiguous(cu_seq_k)
    seqused_k = _maybe_contiguous(seqused_k)
    block_table = _maybe_contiguous(block_table)

    out, softmax_lse, out_accum, softmax_lse_accum = _FA3_CUDA_FWD(
        q,
        k,
        v,
        None,  # k_new
        None,  # v_new
```
- **EN**: This block continues `_fa3_run_forward` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa3_run_forward`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 298-331
```python
        None,  # qv
        out,  # out_ (pre-allocated output)
        cu_seqlens_q,  # cu_seqlens_q
        cu_seqlens_k,  # cu_seqlens_k
        None,  # cu_seqlens_k_new
        None,  # seqused_q
        seqused_k,  # seqused_k
        max_q,  # max_seqlen_q
        max_k,  # max_seqlen_k
        block_table,  # block_table,
        None,  # kv_batch_idx,
        None,  # leftpad_k,
        None,  # rotary_cos,
        None,  # rotary_sin,
        None,  # seqlens_rotary,
        q_descale,  # q_descale,
        k_descale,  # k_descale,
        v_descale,  # v_descale,
        scale,  # softmax_scale,
        is_causal,  # causal,
        window_size_left if window_size_left is not None else -1,  # window_size_left
        window_size_right if window_size_right is not None else -1,  # window_size_right
        0,  # attention_chunk,
        0.0,  # softcap,
        True,  # rotary_interleaved,
        None,  # scheduler_metadata,
        num_splits
        or (1 if torch.are_deterministic_algorithms_enabled() else 0),  # num_splits,
        None,  # pack_gqa,
        torch._C._get_sm_carveout_experimental() or 0,  # sm_margin,
    )
    return out, softmax_lse.contiguous()
```
- **EN**: This module-level block helps implement attention-specific transformations and bookkeeping. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于实现注意力相关的变换与簿记逻辑。 同时它还会计算并返回中间值或结果。

### Lines 332-359
```python
def _fa3_run_backward(
    grad_out: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    out: torch.Tensor,
    logsumexp: torch.Tensor,
    cu_seq_q: torch.Tensor | None,
    cu_seq_k: torch.Tensor | None,
    max_seqlen_q: int | None,
    max_seqlen_k: int | None,
    scale: float | None,
    is_causal: bool,
    window_size_left: int,
    window_size_right: int,
    deterministic: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    if _FA3_CUDA_BWD is None:
        raise RuntimeError("FA3 not registered")

    # Ensure contiguous
    dout = _maybe_contiguous(grad_out)
    q = query.contiguous() if query.stride(-1) != 1 else query
    k = key.contiguous() if key.stride(-1) != 1 else key
    v = value.contiguous() if value.stride(-1) != 1 else value
    o = _maybe_contiguous(out)
    lse = _maybe_contiguous(logsumexp)
```
- **EN**: Defines the `_fa3_run_backward` function; this block introduces logic that connect execution with differentiation-aware logic.
- **CN**: 定义`_fa3_run_backward` 函数；该代码块引入了用于将执行过程与可微分逻辑连接起来的逻辑。

### Lines 360-390
```python
    # Pre-allocate gradient tensors
    dq = torch.empty_like(q)
    dk = torch.empty_like(k)
    dv = torch.empty_like(v)
    _FA3_CUDA_BWD(
        dout,
        q,
        k,
        v,
        o,
        lse,
        dq,
        dk,
        dv,
        cu_seq_q,
        cu_seq_k,
        None,
        None,
        max_seqlen_q,
        max_seqlen_k,
        scale,
        is_causal,
        window_size_left,
        window_size_right,
        0.0,
        deterministic,
        torch._C._get_sm_carveout_experimental() or 0,
    )
    return dq, dk, dv
```
- **EN**: This module-level block helps connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果。

### Lines 391-418
```python
def _fa3_flash_attention_forward_impl(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    cum_seq_q: torch.Tensor | None,
    cum_seq_k: torch.Tensor | None,
    max_q: int,
    max_k: int,
    dropout_p: float,
    is_causal: bool,
    return_debug_mask: bool,
    q_descale: torch.Tensor | None = None,
    k_descale: torch.Tensor | None = None,
    v_descale: torch.Tensor | None = None,
    *,
    scale: float | None = None,
    window_size_left: int = -1,
    window_size_right: int = -1,
    seqused_k: torch.Tensor | None = None,
    alibi_slopes: torch.Tensor | None = None,
    out: torch.Tensor | None = None,
    block_table: torch.Tensor | None = None,
    compute_auxiliary: bool = True,
    num_splits: int | None = None,
):
    error = _fa3_forward_support_error(
        query,
        key,
```
- **EN**: Defines the `_fa3_flash_attention_forward_impl` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`_fa3_flash_attention_forward_impl` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 419-460
```python
        value,
        dropout_p,
        return_debug_mask,
        alibi_slopes,
        seqused_k,
        cum_seq_q,
        q_descale,
        k_descale,
        v_descale,
    )
    if error is not None:
        raise RuntimeError(f"FA3 flash_attention forward unsupported: {error}")
    out, lse = _fa3_run_forward(
        query,
        key,
        value,
        cum_seq_q,
        cum_seq_k,
        max_q,
        max_k,
        scale,
        is_causal,
        window_size_left,
        window_size_right,
        seqused_k,
        out,
        q_descale,
        k_descale,
        v_descale,
        block_table,
        num_splits,
    )
    if compute_auxiliary:
        rng_state = torch.zeros((2,), dtype=torch.uint64, device=query.device)
        philox_offset = torch.zeros((), dtype=torch.uint64, device=query.device)
        debug_mask = torch.empty(0, dtype=query.dtype, device=query.device)
    else:
        rng_state = None
        philox_offset = None
        debug_mask = None
    return out, lse, rng_state, philox_offset, debug_mask
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 461-488
```python

def _fa3_flash_attention_forward_no_dropout_inplace_impl(
    out: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    cum_seq_q: torch.Tensor | None,
    cum_seq_k: torch.Tensor | None,
    max_q: int,
    max_k: int,
    dropout_p: float,
    is_causal: bool,
    return_debug_mask: bool,
    *,
    scale: float | None = None,
    window_size_left: int = -1,
    window_size_right: int = -1,
    seqused_k: torch.Tensor | None = None,
    alibi_slopes: torch.Tensor | None = None,
    block_table: torch.Tensor | None = None,
    num_splits: int | None = None,
):
    _, lse, _, _, _ = _fa3_flash_attention_forward_impl(
        query,
        key,
        value,
        cum_seq_q,
        cum_seq_k,
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 489-516
```python
        max_q,
        max_k,
        dropout_p,
        is_causal,
        return_debug_mask,
        None,
        None,
        None,
        scale=scale,
        window_size_left=window_size_left,
        window_size_right=window_size_right,
        seqused_k=seqused_k,
        alibi_slopes=alibi_slopes,
        out=out,
        block_table=block_table,
        compute_auxiliary=False,
        num_splits=num_splits,
    )
    return lse


def _fa3_flash_attention_forward_impl_default(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    cum_seq_q: torch.Tensor | None,
    cum_seq_k: torch.Tensor | None,
    max_q: int,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 517-555
```python
    max_k: int,
    dropout_p: float,
    is_causal: bool,
    return_debug_mask: bool,
    *,
    scale: float | None = None,
    window_size_left: int = -1,
    window_size_right: int = -1,
    seqused_k: torch.Tensor | None = None,
    alibi_slopes: torch.Tensor | None = None,
    block_table: torch.Tensor | None = None,
    out: torch.Tensor | None = None,
    num_splits: int | None = None,
):
    return _fa3_flash_attention_forward_impl(
        query,
        key,
        value,
        cum_seq_q,
        cum_seq_k,
        max_q,
        max_k,
        dropout_p,
        is_causal,
        return_debug_mask,
        None,
        None,
        None,
        scale=scale,
        window_size_left=window_size_left,
        window_size_right=window_size_right,
        seqused_k=seqused_k,
        alibi_slopes=alibi_slopes,
        out=out,
        block_table=block_table,
        num_splits=num_splits,
    )
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 556-594
```python
def _fa3_flash_attention_backward_impl(
    grad_out: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    out: torch.Tensor,
    logsumexp: torch.Tensor,
    cum_seq_q: torch.Tensor | None,
    cum_seq_k: torch.Tensor | None,
    max_q: int,
    max_k: int,
    dropout_p: float,
    is_causal: bool,
    rng_state: torch.Tensor,
    unused: torch.Tensor,
    *,
    scale: float | None = None,
    window_size_left: int | None = None,
    window_size_right: int | None = None,
):
    """FA3 implementation of _flash_attention_backward."""
    error = _fa3_backward_support_error(
        grad_out,
        query,
        key,
        value,
        out,
        logsumexp,
        dropout_p,
        cum_seq_q,
        window_size_left,
        window_size_right,
    )

    if error is not None:
        raise RuntimeError(f"FA3 flash_attention backward unsupported: {error}")

    deterministic = torch.are_deterministic_algorithms_enabled()
```
- **EN**: Defines the `_fa3_flash_attention_backward_impl` function; this block introduces logic that implement attention-specific transformations and bookkeeping.
- **CN**: 定义`_fa3_flash_attention_backward_impl` 函数；该代码块引入了用于实现注意力相关的变换与簿记逻辑的逻辑。

### Lines 595-622
```python
    dq, dk, dv = _fa3_run_backward(
        grad_out,
        query,
        key,
        value,
        out,
        logsumexp,
        cum_seq_q,
        cum_seq_k,
        max_q,
        max_k,
        scale,
        is_causal,
        window_size_left if window_size_left is not None else -1,
        window_size_right if window_size_right is not None else -1,
        deterministic,
    )
    return dq, dk, dv


def _fa3_scaled_dot_product_flash_attention_forward_impl(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    q_descale: torch.Tensor | None = None,
    k_descale: torch.Tensor | None = None,
    v_descale: torch.Tensor | None = None,
    dropout_p: float = 0.0,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 623-651
```python
    is_causal: bool = False,
    return_debug_mask: bool = False,
    *,
    scale: float | None = None,
):
    error = _fa3_forward_support_error(
        query,
        key,
        value,
        dropout_p,
        return_debug_mask,
        None,
        None,
        None,
        q_descale,
        k_descale,
        v_descale,
    )
    if error is not None:
        raise RuntimeError(f"FA3 SDPA forward unsupported: {error}")
    q, k, v = _transpose_dense(query, key, value)

    # Pre-allocate output with query's strides (BHSD layout), then create
    # a BSHD view for the kernel. This ensures the returned output has
    # the same memory layout as the input query.
    out_dtype = torch.bfloat16 if query.dtype == torch.float8_e4m3fn else query.dtype
    out_bhsd = torch.empty_like(query, dtype=out_dtype)
    out_bshd = out_bhsd.transpose(1, 2)
```
- **EN**: This block continues `_fa3_scaled_dot_product_flash_attention_forward_impl` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa3_scaled_dot_product_flash_attention_forward_impl`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 652-685
```python
    max_q_flash = q.size(1)
    max_k_flash = k.size(1)
    _, lse, rng_state, philox_offset, debug_mask = _fa3_flash_attention_forward_impl(
        q,
        k,
        v,
        None,
        None,
        max_q_flash,
        max_k_flash,
        dropout_p,
        is_causal,
        return_debug_mask,
        scale=scale,
        out=out_bshd,
        q_descale=q_descale,
        k_descale=k_descale,
        v_descale=v_descale,
    )
    max_q = query.size(2)
    max_k = key.size(2)
    return (
        out_bhsd,
        lse,
        None,
        None,
        max_q,
        max_k,
        rng_state,
        philox_offset,
        debug_mask,
    )
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 686-713
```python
def _fa3_scaled_dot_product_flash_attention_forward_impl_default(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    dropout_p: float = 0.0,
    is_causal: bool = False,
    return_debug_mask: bool = False,
    *,
    scale: float | None = None,
):
    return _fa3_scaled_dot_product_flash_attention_forward_impl(
        query,
        key,
        value,
        None,
        None,
        None,
        dropout_p,
        is_causal,
        return_debug_mask,
        scale=scale,
    )


def _fa3_scaled_dot_product_flash_attention_backward_impl(
    grad_out: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 714-741
```python
    value: torch.Tensor,
    out: torch.Tensor,
    logsumexp: torch.Tensor,
    cum_seq_q: torch.Tensor | None,
    cum_seq_k: torch.Tensor | None,
    max_q: int,
    max_k: int,
    dropout_p: float,
    is_causal: bool,
    philox_seed: torch.Tensor,
    philox_offset: torch.Tensor,
    *,
    scale: float | None = None,
):
    """FA3 implementation of _scaled_dot_product_flash_attention_backward."""
    error = _fa3_backward_support_error(
        grad_out, query, key, value, out, logsumexp, dropout_p, None, None, None
    )
    if error is not None:
        raise RuntimeError(f"FA3 SDPA backward unsupported: {error}")

    # SDPA uses BHSD layout, FA3 uses BSHD - transpose
    grad_out_t, q_t, k_t, v_t, out_t = _transpose_dense(
        grad_out, query, key, value, out
    )

    dq, dk, dv = _fa3_flash_attention_backward_impl(
        grad_out_t,
```
- **EN**: This block continues `_fa3_scaled_dot_product_flash_attention_backward_impl` and works to implement attention-specific transformations and bookkeeping. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa3_scaled_dot_product_flash_attention_backward_impl`，用于实现注意力相关的变换与簿记逻辑。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 742-763
```python
        q_t,
        k_t,
        v_t,
        out_t,
        logsumexp,
        None,  # cum_seq_q (dense attention)
        None,  # cum_seq_k
        max_q,  # max_seqlen_q
        max_k,  # max_seqlen_k
        dropout_p,
        is_causal,
        philox_seed,
        philox_offset,
        scale=scale,
    )

    # Transpose gradients back to BHSD layout
    dq_out, dk_out, dv_out = _transpose_dense(dq, dk, dv)
    return dq_out, dk_out, dv_out


_registry.register_flash_attention_impl("FA3", register_fn=register_flash_attention_fa3)
```
- **EN**: This module-level block helps implement attention-specific transformations and bookkeeping. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于实现注意力相关的变换与簿记逻辑。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Attention mechanics / 注意力机制**
  - EN: The logic specializes in attention-style data movement, masking, or projection patterns.
  - CN: 该逻辑专门处理注意力风格的数据搬运、掩码或投影模式。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.library`, `.`
- **Standard library / 标准库**: `__future__`, `importlib`, `warnings`, `typing`, `dataclasses`, `functools`, `typing_extensions`, `collections.abc`
- **Primary symbols / 核心符号**: `__all__`, `_FA3_CUDA_FWD`, `_FA3_CUDA_BWD`, `_FA3Handle`, `_get_device_major`, `register_flash_attention_fa3`, `_fa3_import_module`, `_fa3_register_kernels`, `_fa3_common_support_error`, `_fa3_forward_support_error`, `_fa3_backward_support_error`, `_transpose_dense`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
