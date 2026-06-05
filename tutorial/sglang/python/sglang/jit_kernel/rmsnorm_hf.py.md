# rmsnorm_hf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/rmsnorm_hf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"RMSNorm with HF LlamaRMSNorm semantics (cast to dtype before weight multiply)."""". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""RMSNorm with HF LlamaRMSNorm semantics (cast to dtype before weight multiply)."""”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and module setup
```python
"""RMSNorm with HF LlamaRMSNorm semantics (cast to dtype before weight multiply)."""

from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.jit_kernel.utils import (
    cache_once,
    is_arch_support_pdl,
    load_jit,
    make_cpp_args,
)

if TYPE_CHECKING:
    from tvm_ffi.module import Module

_CTA_BLOCK_SIZE = 512
_WARP_SIZE = 32
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 23-35: Function `is_supported_rmsnorm_hf_hidden_size`
```python
def is_supported_rmsnorm_hf_hidden_size(hidden_size: int) -> bool:
    """Return True iff the JIT rmsnorm_hf kernel supports this hidden size.

    Two launch configs cover the practical range:
      - Warp kernel: ``[32, 512)`` in multiples of 32 (q/k RMSNorm head dims).
      - CTA kernel: ``>= 512`` in multiples of 512 (token RMSNorms).
    """
    if _WARP_SIZE <= hidden_size < _CTA_BLOCK_SIZE and hidden_size % _WARP_SIZE == 0:
        return True
    return hidden_size >= _CTA_BLOCK_SIZE and hidden_size % _CTA_BLOCK_SIZE == 0


@cache_once
```
**EN:** This block defines `is_supported_rmsnorm_hf_hidden_size`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `is_supported_rmsnorm_hf_hidden_size`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 36-48: Function `_jit_rmsnorm_hf_module`
```python
def _jit_rmsnorm_hf_module(hidden_size: int, dtype: torch.dtype) -> Module:
    args = make_cpp_args(hidden_size, is_arch_support_pdl(), dtype)
    kernel_cls = (
        "HFRMSNormWarpKernel" if hidden_size < _CTA_BLOCK_SIZE else "HFRMSNormKernel"
    )
    return load_jit(
        "rmsnorm_hf",
        *args,
        cuda_files=["elementwise/rmsnorm_hf.cuh"],
        cuda_wrappers=[("rmsnorm_hf", f"{kernel_cls}<{args}>::run")],
    )
```
**EN:** This block defines `_jit_rmsnorm_hf_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_rmsnorm_hf_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 49-79: Function `rmsnorm_hf`
```python
def rmsnorm_hf(
    input: torch.Tensor,
    weight: torch.Tensor,
    eps: float = 1e-6,
    out: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    """RMSNorm: ``out = weight * cast_dtype(rsqrt(mean(x^2) + eps) * x)``.

    ``input`` must be 2D ``(num_tokens, hidden_size)``; callers with
    higher-rank tensors should reshape first. ``hidden_size`` must satisfy
    :func:`is_supported_rmsnorm_hf_hidden_size`. Empty inputs return an empty
    output without launching the kernel.
    """
    if input.dtype not in (torch.float16, torch.bfloat16):
        raise RuntimeError(f"rmsnorm_hf: input must be fp16 or bf16, got {input.dtype}")
    if input.dim() != 2:
        raise RuntimeError(f"rmsnorm_hf: input must be 2D, got {input.dim()}D")
    hidden_size = input.size(-1)
    if not is_supported_rmsnorm_hf_hidden_size(hidden_size):
        raise RuntimeError(
            f"rmsnorm_hf: unsupported hidden_size={hidden_size} "
            f"(must be a multiple of {_WARP_SIZE} in [{_WARP_SIZE}, {_CTA_BLOCK_SIZE}) "
            f"or a multiple of {_CTA_BLOCK_SIZE})"
        )
    if out is None:
        out = torch.empty_like(input)
    if input.numel() == 0:
        return out
    module = _jit_rmsnorm_hf_module(hidden_size, input.dtype)
    module.rmsnorm_hf(input, weight, out, eps)
    return out
```
**EN:** This block defines `rmsnorm_hf`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `rmsnorm_hf`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> (`
- `tvm_ffi.module -> Module`
