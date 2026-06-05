# norm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/norm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and module setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Optional

import torch

from sglang.jit_kernel.utils import (
    cache_once,
    is_arch_support_pdl,
    load_jit,
    make_cpp_args,
)
from sglang.kernel_api_logging import debug_kernel_api

if TYPE_CHECKING:
    from tvm_ffi.module import Module


logger = logging.getLogger(__name__)


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 24-38: Function `_jit_qknorm_module`
```python
def _jit_qknorm_module(head_dim: int, dtype: torch.dtype) -> Module:
    args = make_cpp_args(head_dim, is_arch_support_pdl(), dtype)
    return load_jit(
        "qknorm",
        *args,
        cuda_files=["elementwise/qknorm.cuh"],
        cuda_wrappers=[("qknorm", f"QKNormKernel<{args}>::run")],
    )


_RMSNORM_WARP_SIZES = frozenset({64, 128, 256})
_RMSNORM_MAX_HIDDEN_SIZE = 16384
_RMSNORM_HALF_BLOCK_MIN_SIZE = 2048
```
**EN:** This block defines `_jit_qknorm_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_qknorm_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 39-45: Function `_is_supported_rmsnorm_hidden_size`
```python
def _is_supported_rmsnorm_hidden_size(d: int) -> bool:
    return d in _RMSNORM_WARP_SIZES or (
        (d > 256 and d % 256 == 0 and d <= 8192)
        or (d >= 8192 and d % 512 == 0 and d <= 16384)
    )
```
**EN:** This block defines `_is_supported_rmsnorm_hidden_size`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_is_supported_rmsnorm_hidden_size`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 46-55: Function `_rmsnorm_kernel_class`
```python
def _rmsnorm_kernel_class(hidden_size: int) -> str:
    if hidden_size in _RMSNORM_WARP_SIZES:
        return "RMSNormWarpKernel"
    if hidden_size >= _RMSNORM_HALF_BLOCK_MIN_SIZE:
        if hidden_size % 512 == 0:
            return "RMSNormHalfKernel"
    return "RMSNormKernel"


@cache_once
```
**EN:** This block defines `_rmsnorm_kernel_class`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_rmsnorm_kernel_class`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 56-67: Function `_jit_rmsnorm_module`
```python
def _jit_rmsnorm_module(hidden_size: int, dtype: torch.dtype) -> Module:
    args = make_cpp_args(hidden_size, is_arch_support_pdl(), dtype)
    kernel_class = f"{_rmsnorm_kernel_class(hidden_size)}<{args}>"
    return load_jit(
        "rmsnorm",
        *args,
        cuda_files=["elementwise/rmsnorm.cuh"],
        cuda_wrappers=[("rmsnorm", f"{kernel_class}::run")],
    )


@cache_once
```
**EN:** This block defines `_jit_rmsnorm_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_rmsnorm_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 68-78: Function `_jit_fused_add_rmsnorm_module`
```python
def _jit_fused_add_rmsnorm_module(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype)
    return load_jit(
        "fused_add_rmsnorm",
        *args,
        cuda_files=["elementwise/fused_add_rmsnorm.cuh"],
        cuda_wrappers=[("fused_add_rmsnorm", f"FusedAddRMSNormKernel<{args}>::run")],
    )


@cache_once
```
**EN:** This block defines `_jit_fused_add_rmsnorm_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_fused_add_rmsnorm_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 79-92: Function `_jit_qknorm_across_heads_module`
```python
def _jit_qknorm_across_heads_module(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype)
    return load_jit(
        "qknorm_across_heads",
        *args,
        cuda_files=["elementwise/qknorm_across_heads.cuh"],
        cuda_wrappers=[
            ("qknorm_across_heads", f"QKNormAcrossHeadsKernel<{args}>::run")
        ],
    )


@torch.compiler.assume_constant_result
@cache_once
```
**EN:** This block defines `_jit_qknorm_across_heads_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_qknorm_across_heads_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 93-105: Function `can_use_fused_inplace_qknorm`
```python
def can_use_fused_inplace_qknorm(head_dim: int, dtype: torch.dtype) -> bool:
    if head_dim not in [64, 128, 256, 512, 1024]:
        logger.warning(f"Unsupported head_dim={head_dim} for JIT QK-Norm kernel")
        return False
    try:
        _jit_qknorm_module(head_dim, dtype)
        return True
    except Exception as e:
        logger.warning(f"Failed to load JIT QK-Norm kernel: {e}")
        return False


@debug_kernel_api
```
**EN:** This block defines `can_use_fused_inplace_qknorm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `can_use_fused_inplace_qknorm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 106-120: Function `fused_inplace_qknorm`
```python
def fused_inplace_qknorm(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    eps: float = 1e-6,
    *,
    head_dim: int = 0,
) -> None:
    head_dim = head_dim or q.size(-1)
    module = _jit_qknorm_module(head_dim, q.dtype)
    module.qknorm(q, k, q_weight, k_weight, eps)


@debug_kernel_api
```
**EN:** This block defines `fused_inplace_qknorm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_inplace_qknorm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 121-139: Function `rmsnorm`
```python
def rmsnorm(
    input: torch.Tensor,
    weight: torch.Tensor,
    out: Optional[torch.Tensor] = None,
    eps: float = 1e-6,
) -> None:
    out = out if out is not None else input
    hidden_size = input.size(-1)
    if not _is_supported_rmsnorm_hidden_size(hidden_size):
        raise RuntimeError(
            f"jit rmsnorm: unsupported hidden_size={hidden_size}. "
            f"Supported: {sorted(_RMSNORM_WARP_SIZES)}, and multiples of 256 in "
            f"(256, {_RMSNORM_MAX_HIDDEN_SIZE}]."
        )
    module = _jit_rmsnorm_module(hidden_size, input.dtype)
    module.rmsnorm(input, weight, out, eps)


@debug_kernel_api
```
**EN:** This block defines `rmsnorm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `rmsnorm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 140-150: Function `fused_add_rmsnorm`
```python
def fused_add_rmsnorm(
    input: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float = 1e-6,
) -> None:
    module = _jit_fused_add_rmsnorm_module(input.dtype)
    module.fused_add_rmsnorm(input, residual, weight, eps)


@debug_kernel_api
```
**EN:** This block defines `fused_add_rmsnorm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_add_rmsnorm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 151-169: Function `fused_inplace_qknorm_across_heads`
```python
def fused_inplace_qknorm_across_heads(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    eps: float = 1e-6,
) -> None:
    """
    Fused inplace QK normalization across all heads.

    Args:
        q: Query tensor of shape [batch_size, num_heads * head_dim]
        k: Key tensor of shape [batch_size, num_heads * head_dim]
        q_weight: Query weight tensor of shape [num_heads * head_dim]
        k_weight: Key weight tensor of shape [num_heads * head_dim]
        eps: Epsilon for numerical stability
    """
    module = _jit_qknorm_across_heads_module(q.dtype)
    module.qknorm_across_heads(q, k, q_weight, k_weight, eps)
```
**EN:** This block defines `fused_inplace_qknorm_across_heads`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_inplace_qknorm_across_heads`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `logging`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> (`
- `sglang.kernel_api_logging -> debug_kernel_api`
- `tvm_ffi.module -> Module`
