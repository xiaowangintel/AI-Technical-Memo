# _fa4.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/attention/_fa4.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements attention-specific abstractions and helpers for modern sequence models.
- **Purpose (CN)**: 实现现代序列模型中的注意力相关抽象与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```python
"""UBER PROTOTYPE!!!"""
# mypy: allow-untyped-defs

from __future__ import annotations

import importlib
from dataclasses import dataclass
from functools import cache
from typing import Any, TYPE_CHECKING
from typing_extensions import TypeVarTuple, Unpack

from . import _registry


if TYPE_CHECKING:
    from types import ModuleType

import torch
from torch.library import Library


__all__ = [
    "register_flash_attention_fa4",
]
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 27-49
```python
_FA4_MODULE_PATH: str | None = None


@dataclass
class _FA4Handle:
    library: Library | None

    def remove(self) -> None:
        self.library = None


@cache
def _get_device_major(device: torch.device) -> int:
    major, _ = torch.cuda.get_device_capability(device)
    return major


def register_flash_attention_fa4(
    module_path: str = "flash_attn.cute.interface",
) -> _FA4Handle:
    """
    Register FA4 flash attention kernels with the PyTorch dispatcher.
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 50-67
```python
    Args:
        module_path: Python module path to the FA4 implementation.
    """
    global _FA4_MODULE_PATH
    _ = _fa4_import_module(module_path)
    _FA4_MODULE_PATH = module_path
    return _FA4Handle(_fa4_register_kernels())


@cache
def _fa4_import_module(module_path: str) -> ModuleType:
    module = importlib.import_module(module_path)
    if not hasattr(module, "_flash_attn_fwd") or not hasattr(module, "_flash_attn_bwd"):
        raise RuntimeError(f"Module '{module_path}' does not expose FA4 kernels")
    return module


def _fa4_register_kernels() -> Library:
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 68-88
```python
    lib = Library("aten", "IMPL", "CUDA")  # noqa: TOR901
    lib.impl("_flash_attention_forward", _fa4_flash_attention_forward_impl, "CUDA")
    lib.impl(
        "_flash_attention_forward_no_dropout_inplace",
        _fa4_flash_attention_forward_no_dropout_inplace_impl,
        "CUDA",
    )
    lib.impl("_flash_attention_backward", _fa4_flash_attention_backward_impl, "CUDA")
    lib.impl(
        "_scaled_dot_product_flash_attention",
        _fa4_scaled_dot_product_flash_attention_forward_impl,
        "CUDA",
    )
    lib.impl(
        "_scaled_dot_product_flash_attention_backward",
        _fa4_scaled_dot_product_flash_attention_backward_impl,
        "CUDA",
    )
    return lib
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 89-114
```python
def _fa4_common_support_error(
    query: torch.Tensor,
    tensors: tuple[torch.Tensor, ...],
    cum_seq_q: torch.Tensor | None,
    require_fp32: tuple[tuple[str, torch.Tensor], ...] = (),
) -> str | None:
    if not all(t.is_cuda for t in tensors):
        return "inputs must be CUDA tensors"
    if len({t.device for t in tensors}) != 1:
        return "inputs must share device"
    if query.dtype not in (torch.float16, torch.bfloat16):
        return "query dtype must be float16 or bfloat16"
    for name, tensor in require_fp32:
        if tensor.dtype != torch.float32:
            return f"{name} dtype must be float32"
    if cum_seq_q is None and query.dim() != 4:
        return "dense query must be 4D"
    if cum_seq_q is not None and query.dim() != 3:
        return "ragged query must be 3D"
    if not torch.cuda.is_available():
        return "CUDA not available"
    if _get_device_major(query.device) not in (9, 10):
        return "FA4 requires compute capability 9.0 or 10.0"
    return None
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 115-132
```python
def _fa4_forward_support_error(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    dropout_p: float,
    return_debug_mask: bool,
    alibi_slopes: torch.Tensor | None,
    seqused_k: torch.Tensor | None,
    cum_seq_q: torch.Tensor | None,
    block_table: torch.Tensor | None = None,
    num_splits: int | None = None,
) -> str | None:
    if dropout_p != 0.0:
        return "dropout_p must be 0"
    if return_debug_mask:
        return "return_debug_mask must be False"
    if alibi_slopes is not None:
        return "alibi_slopes not supported"
```
- **EN**: Defines the `_fa4_forward_support_error` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`_fa4_forward_support_error` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 133-154
```python
    if seqused_k is not None:
        if seqused_k.dtype != torch.int32:
            return "seqused_k must be int32"
        if not seqused_k.is_cuda:
            return "seqused_k must be CUDA"
    major = _get_device_major(query.device)
    if block_table is not None and major != 10:
        return f"paged KV (block_table) not supported on SM {major}0"
    if num_splits is not None and num_splits > 1 and major != 10:
        return f"SplitKV (num_splits > 1) not supported on SM {major}0"
    error = _fa4_common_support_error(
        query,
        (query, key, value),
        cum_seq_q,
    )
    if error is not None:
        if error == "inputs must share device":
            return "query, key, value must be on same device"
        return error
    return None
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 155-181
```python
def _fa4_backward_support_error(
    grad_out: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    out: torch.Tensor,
    logsumexp: torch.Tensor,
    dropout_p: float,
    cum_seq_q: torch.Tensor | None,
) -> str | None:
    if dropout_p != 0.0:
        return "dropout_p must be 0"
    error = _fa4_common_support_error(
        query,
        (grad_out, query, key, value, out, logsumexp),
        cum_seq_q,
        require_fp32=(("logsumexp", logsumexp),),
    )
    if error is not None:
        return error
    return None


def _aten_to_fa4_window_size(val: int | None) -> int | None:
    """need to convert -1 to None for FA4"""
    return None if val == -1 else val
```
- **EN**: This module-level block helps connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 182-199
```python

Ts = TypeVarTuple("Ts")


def _transpose_dense(*tensors: Unpack[Ts]) -> tuple[Unpack[Ts]]:
    return tuple(t.transpose(1, 2) for t in tensors)  # type: ignore[attr-defined]


def _fa4_run_forward(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    cu_seq_q: torch.Tensor | None,
    cu_seq_k: torch.Tensor | None,
    max_q: int | None,
    max_k: int | None,
    scale: float | None,
    is_causal: bool,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 200-217
```python
    window_size_left: int | None,
    window_size_right: int | None,
    seqused_k: torch.Tensor | None,
    out: torch.Tensor | None = None,
    block_table: torch.Tensor | None = None,
    num_splits: int | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    if _FA4_MODULE_PATH is None:
        raise RuntimeError("FA4 not registered")
    module = _fa4_import_module(_FA4_MODULE_PATH)

    kwargs: dict[str, Any] = {
        "softmax_scale": scale,
        "causal": is_causal,
        "window_size_left": _aten_to_fa4_window_size(window_size_left),
        "window_size_right": _aten_to_fa4_window_size(window_size_right),
        "return_lse": True,
        "cu_seqlens_q": cu_seq_q,
```
- **EN**: This block continues `_fa4_run_forward` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa4_run_forward`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 218-235
```python
        "cu_seqlens_k": cu_seq_k,
        "max_seqlen_q": max_q,
        "max_seqlen_k": max_k,
        "seqused_k": seqused_k.contiguous() if seqused_k is not None else None,
        "page_table": block_table,
        "num_splits": num_splits or 1,
        "out": out,
    }
    out, lse = module._flash_attn_fwd(query, key, value, **kwargs)
    return out, lse.contiguous()


def _fa4_run_backward(
    grad_out: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    out: torch.Tensor,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 236-253
```python
    logsumexp: torch.Tensor,
    cu_seq_q: torch.Tensor | None,
    cu_seq_k: torch.Tensor | None,
    scale: float | None,
    is_causal: bool,
    window_size_left: int | None,
    window_size_right: int | None,
    deterministic: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    if _FA4_MODULE_PATH is None:
        raise RuntimeError("FA4 not registered")
    module = _fa4_import_module(_FA4_MODULE_PATH)
    dq, dk, dv = module._flash_attn_bwd(
        query,
        key,
        value,
        out,
        grad_out,
```
- **EN**: This block continues `_fa4_run_backward` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa4_run_backward`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 254-271
```python
        logsumexp.contiguous(),
        softmax_scale=scale,
        causal=is_causal,
        window_size_left=_aten_to_fa4_window_size(window_size_left),
        window_size_right=_aten_to_fa4_window_size(window_size_right),
        cu_seqlens_q=cu_seq_q,
        cu_seqlens_k=cu_seq_k,
        deterministic=deterministic,
    )
    return dq, dk, dv


def _fa4_flash_attention_forward_impl(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    cum_seq_q: torch.Tensor | None,
    cum_seq_k: torch.Tensor | None,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 272-289
```python
    max_q: int,
    max_k: int,
    dropout_p: float,
    is_causal: bool,
    return_debug_mask: bool,
    *,
    scale: float | None = None,
    window_size_left: int | None = None,
    window_size_right: int | None = None,
    seqused_k: torch.Tensor | None = None,
    alibi_slopes: torch.Tensor | None = None,
    out: torch.Tensor | None = None,
    block_table: torch.Tensor | None = None,
    compute_auxiliary: bool = True,
    num_splits: int | None = None,
):
    error = _fa4_forward_support_error(
        query,
```
- **EN**: This block continues `_fa4_flash_attention_forward_impl` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `_fa4_flash_attention_forward_impl`，用于组织可复用的模块行为与状态。

### Lines 290-307
```python
        key,
        value,
        dropout_p,
        return_debug_mask,
        alibi_slopes,
        seqused_k,
        cum_seq_q,
        block_table,
        num_splits,
    )
    if error is not None:
        raise RuntimeError(f"FA4 flash_attention forward unsupported: {error}")
    out, lse = _fa4_run_forward(
        query,
        key,
        value,
        cum_seq_q,
        cum_seq_k,
```
- **EN**: This block continues `_fa4_flash_attention_forward_impl` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa4_flash_attention_forward_impl`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 308-329
```python
        max_q,
        max_k,
        scale,
        is_causal,
        window_size_left,
        window_size_right,
        seqused_k,
        out,
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
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 330-347
```python
def _fa4_flash_attention_forward_no_dropout_inplace_impl(
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
    window_size_left: int | None = None,
    window_size_right: int | None = None,
    seqused_k: torch.Tensor | None = None,
    alibi_slopes: torch.Tensor | None = None,
```
- **EN**: Defines the `_fa4_flash_attention_forward_no_dropout_inplace_impl` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`_fa4_flash_attention_forward_no_dropout_inplace_impl` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 348-374
```python
    block_table: torch.Tensor | None = None,
    num_splits: int | None = None,
):
    _, lse, _, _, _ = _fa4_flash_attention_forward_impl(
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
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 375-392
```python
def _fa4_flash_attention_backward_impl(
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
```
- **EN**: Defines the `_fa4_flash_attention_backward_impl` function; this block introduces logic that implement attention-specific transformations and bookkeeping.
- **CN**: 定义`_fa4_flash_attention_backward_impl` 函数；该代码块引入了用于实现注意力相关的变换与簿记逻辑的逻辑。

### Lines 393-410
```python
    window_size_right: int | None = None,
):
    error = _fa4_backward_support_error(
        grad_out,
        query,
        key,
        value,
        out,
        logsumexp,
        dropout_p,
        cum_seq_q,
    )
    if error is not None:
        raise RuntimeError(f"FA4 flash_attention backward unsupported: {error}")
    deterministic = torch.are_deterministic_algorithms_enabled()
    dq, dk, dv = _fa4_run_backward(
        grad_out,
        query,
```
- **EN**: This block continues `_fa4_flash_attention_backward_impl` and works to implement attention-specific transformations and bookkeeping. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa4_flash_attention_backward_impl`，用于实现注意力相关的变换与簿记逻辑。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 411-428
```python
        key,
        value,
        out,
        logsumexp,
        cum_seq_q,
        cum_seq_k,
        scale,
        is_causal,
        window_size_left,
        window_size_right,
        deterministic,
    )
    return dq, dk, dv


def _fa4_scaled_dot_product_flash_attention_forward_impl(
    query: torch.Tensor,
    key: torch.Tensor,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 429-455
```python
    value: torch.Tensor,
    dropout_p: float = 0.0,
    is_causal: bool = False,
    return_debug_mask: bool = False,
    *,
    scale: float | None = None,
):
    error = _fa4_forward_support_error(
        query,
        key,
        value,
        dropout_p,
        return_debug_mask,
        None,
        None,
        None,
    )
    if error is not None:
        raise RuntimeError(f"FA4 SDPA forward unsupported: {error}")
    q, k, v = _transpose_dense(query, key, value)

    # Pre-allocate output with query's strides (BHSD layout), then create
    # a BSHD view for the kernel. This ensures the returned output has
    # the same memory layout as the input query.
    out_bhsd = torch.empty_like(query)
    out_bshd = out_bhsd.transpose(1, 2)
```
- **EN**: This block continues `_fa4_scaled_dot_product_flash_attention_forward_impl` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa4_scaled_dot_product_flash_attention_forward_impl`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 456-473
```python
    max_q_flash = q.size(1)
    max_k_flash = k.size(1)
    _, lse, rng_state, philox_offset, debug_mask = _fa4_flash_attention_forward_impl(
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
    )
    max_q = query.size(2)
    max_k = key.size(2)
```
- **EN**: This block continues `_fa4_scaled_dot_product_flash_attention_forward_impl` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `_fa4_scaled_dot_product_flash_attention_forward_impl`，用于组织可复用的模块行为与状态。

### Lines 474-491
```python
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


def _fa4_scaled_dot_product_flash_attention_backward_impl(
    grad_out: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
```
- **EN**: This module-level block helps implement attention-specific transformations and bookkeeping. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于实现注意力相关的变换与簿记逻辑。 同时它还会计算并返回中间值或结果。

### Lines 492-509
```python
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
    error = _fa4_backward_support_error(
        grad_out,
        query,
        key,
        value,
```
- **EN**: This block continues `_fa4_scaled_dot_product_flash_attention_backward_impl` and works to connect execution with differentiation-aware logic.
- **CN**: 该代码块继续实现 `_fa4_scaled_dot_product_flash_attention_backward_impl`，用于将执行过程与可微分逻辑连接起来。

### Lines 510-527
```python
        out,
        logsumexp,
        dropout_p,
        None,
    )
    if error is not None:
        raise RuntimeError(f"FA4 SDPA backward unsupported: {error}")
    q, k, v, o, go = _transpose_dense(query, key, value, out, grad_out)
    max_q = query.size(2)
    max_k = key.size(2)
    dq, dk, dv = _fa4_flash_attention_backward_impl(
        go,
        q,
        k,
        v,
        o,
        logsumexp,
        None,
```
- **EN**: This block continues `_fa4_scaled_dot_product_flash_attention_backward_impl` and works to implement attention-specific transformations and bookkeeping. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_fa4_scaled_dot_product_flash_attention_backward_impl`，用于实现注意力相关的变换与簿记逻辑。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 528-541
```python
        None,
        max_q,
        max_k,
        dropout_p,
        is_causal,
        philox_seed,
        philox_offset,
        scale=scale,
    )
    dq, dk, dv = _transpose_dense(dq, dk, dv)
    return dq, dk, dv


_registry.register_flash_attention_impl("FA4", register_fn=register_flash_attention_fa4)
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
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `.`, `torch`, `torch.library`
- **Standard library / 标准库**: `__future__`, `importlib`, `dataclasses`, `functools`, `typing`, `typing_extensions`, `types`
- **Primary symbols / 核心符号**: `__all__`, `_FA4_MODULE_PATH`, `_FA4Handle`, `_get_device_major`, `register_flash_attention_fa4`, `_fa4_import_module`, `_fa4_register_kernels`, `_fa4_common_support_error`, `_fa4_forward_support_error`, `_fa4_backward_support_error`, `_aten_to_fa4_window_size`, `_transpose_dense`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
