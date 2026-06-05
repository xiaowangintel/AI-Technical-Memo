# kvcache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/kvcache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup
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


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 21-31: Function `_jit_kvcache_module`
```python
def _jit_kvcache_module(row_bytes: int) -> Module:
    args = make_cpp_args(row_bytes, is_arch_support_pdl())
    return load_jit(
        "kvcache",
        *args,
        cuda_files=["elementwise/kvcache.cuh"],
        cuda_wrappers=[("store_cache", f"StoreKVCacheKernel<{args}>::run")],
    )


@cache_once
```
**EN:** This block defines `_jit_kvcache_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_kvcache_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 32-50: Function `can_use_store_cache`
```python
def can_use_store_cache(size: int) -> bool:
    logger = logging.getLogger(__name__)
    if size % 4 != 0:
        logger.warning(
            f"Unsupported row_bytes={size} for JIT KV-Cache kernel:"
            " must be multiple of 4"
        )
        return False
    try:
        _jit_kvcache_module(size)
        return True
    except Exception as e:
        logger.warning(
            f"Failed to load JIT KV-Cache kernel " f"with row_bytes={size}: {e}"
        )
        return False


@register_custom_op(mutates_args=["k_cache", "v_cache"])
```
**EN:** This block defines `can_use_store_cache`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `can_use_store_cache`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 51-86: Function `store_cache`
```python
def store_cache(
    k: torch.Tensor,
    v: torch.Tensor,
    k_cache: torch.Tensor,
    v_cache: torch.Tensor,
    indices: torch.Tensor,
    *,
    row_bytes: int = 0,
    num_split: int = 0,  # can be tuned for performance
) -> None:
    """Store key and value tensors into KV cache at specified indices.

    Args:
        k (torch.Tensor): Key tensor of shape (batch_size, H * D).
        v (torch.Tensor): Value tensor of shape (batch_size, H * D).
        k_cache (torch.Tensor): Key cache tensor of shape (num_pages, H * D).
        v_cache (torch.Tensor): Value cache tensor of shape (num_pages, H * D).
        indices (torch.Tensor): Indices tensor of shape (batch_size,).
    """
    row_bytes = row_bytes or k.shape[-1] * k.element_size()
    module = _jit_kvcache_module(row_bytes)
    if num_split <= 0:
        if row_bytes % 2048 == 0:
            num_split = 4
        elif row_bytes % 1024 == 0:
            num_split = 2
        else:
            num_split = 1
    module.store_cache(
        k,
        v,
        k_cache,
        v_cache,
        indices,
        num_split,
    )
```
**EN:** This block defines `store_cache`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `store_cache`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

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
