# rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup
```python
from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional

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

### Lines 21-29: Function `_jit_rotary_embedding_module`
```python
def _jit_rotary_embedding_module() -> Module:
    return load_jit(
        "rotary_embedding",
        cuda_files=["elementwise/pos_enc.cuh"],
        cuda_wrappers=[("rotary_embedding", "RotaryEmbeddingKernel::run")],
    )


@cache_once
```
**EN:** This block defines `_jit_rotary_embedding_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_rotary_embedding_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 30-46: Function `_jit_fused_rope_module`
```python
def _jit_fused_rope_module(is_neox: bool, rope_dim: int, dtype: torch.dtype) -> Module:
    args = make_cpp_args(is_neox, rope_dim, is_arch_support_pdl(), dtype)
    return load_jit(
        "fused_rope",
        *args,
        cuda_files=["elementwise/rope.cuh"],
        cuda_wrappers=[
            ("run_rope", f"FusedRopeKernel<{args}>::run"),
            ("run_rope_store", f"FusedRopeKernel<{args}>::run_fused"),
        ],
    )


@register_custom_op(
    op_name="rotary_embedding_with_key",
    mutates_args=["query", "key"],
)
```
**EN:** This block defines `_jit_fused_rope_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_fused_rope_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 47-62: Function `rotary_embedding_with_key`
```python
def rotary_embedding_with_key(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool = True,
) -> None:
    module = _jit_rotary_embedding_module()
    module.rotary_embedding(positions, query, key, head_size, cos_sin_cache, is_neox)


@register_custom_op(
    op_name="rotary_embedding_without_key",
    mutates_args=["query"],
)
```
**EN:** This block defines `rotary_embedding_with_key`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `rotary_embedding_with_key`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 63-73: Function `rotary_embedding_without_key`
```python
def rotary_embedding_without_key(
    positions: torch.Tensor,
    query: torch.Tensor,
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool = True,
) -> None:
    module = _jit_rotary_embedding_module()
    module.rotary_embedding(positions, query, None, head_size, cos_sin_cache, is_neox)
```
**EN:** This block defines `rotary_embedding_without_key`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `rotary_embedding_without_key`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 74-93: Function `rotary_embedding`
```python
def rotary_embedding(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: Optional[torch.Tensor],
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool = True,
):
    if key is None:
        rotary_embedding_without_key(
            positions, query, head_size, cos_sin_cache, is_neox
        )
    else:
        rotary_embedding_with_key(
            positions, query, key, head_size, cos_sin_cache, is_neox
        )
    return query, key


@dataclass
```
**EN:** This block defines `rotary_embedding`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `rotary_embedding`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 94-112: Class `FusedSetKVBufferArg`
```python
class FusedSetKVBufferArg:
    """
    value : Optional[torch.Tensor]
        Value tensor, shape: ``(nnz, num_v_heads * head_size)``.
    k_buffer : Optional[torch.Tensor]
        Buffer for keys, shape: ``(nnz, num_k_heads * head_size)``.
    v_buffer : Optional[torch.Tensor]
        Buffer for values, shape: ``(nnz, num_v_heads * head_size)``.
    cache_loc : Optional[torch.Tensor]
        Cache location tensor, used for indexing kv cache.
    """

    value: torch.Tensor
    k_buffer: torch.Tensor
    v_buffer: torch.Tensor
    cache_loc: torch.Tensor


@register_custom_op(mutates_args=["q", "k"])
```
**EN:** This block declares the `FusedSetKVBufferArg` class and establishes the behavior or state it encapsulates. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `FusedSetKVBufferArg` 类，并建立其封装的行为或状态。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 113-140: Function `apply_rope_inplace`
```python
def apply_rope_inplace(
    q: torch.Tensor,
    k: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    positions: torch.Tensor,
    *,
    is_neox: bool,
    rope_dim: int = 0,
) -> None:
    """
    Fused inplace rotary position embedding for query and key tensors.

    Args:
        q: Query tensor of shape [num_tokens, num_qo_heads, rope_dim].
        k: Key tensor of shape [num_tokens, num_kv_heads, rope_dim].
        cos_sin_cache: Cosine/sine cache of shape [max_position, rope_dim],
            where the first half along dim=-1 is cos and the second half is sin.
            Must be float32.
        positions: Position indices of shape [num_tokens], int32 or int64.
        is_neox: Whether to use GPT-NeoX style (True) or GPT-J interleaved style (False).
        rope_dim: Rotary embedding dimension. Defaults to cos_sin_cache.size(-1).
    """
    rope_dim = rope_dim or cos_sin_cache.size(-1)
    module = _jit_fused_rope_module(is_neox, rope_dim, q.dtype)
    module.run_rope(q, k, cos_sin_cache, positions)


@register_custom_op(mutates_args=["q", "k", "k_cache", "v_cache"])
```
**EN:** This block defines `apply_rope_inplace`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_rope_inplace`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 141-178: Function `apply_rope_inplace_with_kvcache`
```python
def apply_rope_inplace_with_kvcache(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    k_cache: torch.Tensor,
    v_cache: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    positions: torch.Tensor,
    out_loc: torch.Tensor,
    *,
    is_neox: bool,
    rope_dim: int = 0,
) -> None:
    """
    Fused inplace RoPE + KV cache store.

    Applies rotary position embedding to q and k inplace. The rotated k is also
    stored in k_cache. The original v is also stored in v_cache.

    Args:
        q: Query tensor of shape [num_tokens, num_qo_heads, head_dim].
        k: Key tensor of shape [num_tokens, num_kv_heads, head_dim].
        v: Value tensor of shape [num_tokens, num_kv_heads, head_dim].
        k_cache: Key cache of shape [cache_size, num_kv_heads * head_dim].
        v_cache: Value cache of shape [cache_size, num_kv_heads * head_dim].
        cos_sin_cache: Cosine/sine cache of shape [max_position, rope_dim], float32.
        positions: Position indices of shape [num_tokens], int32 or int64.
        out_loc: Cache write locations of shape [num_tokens], same dtype as positions.
        is_neox: Whether to use GPT-NeoX style (True) or GPT-J interleaved (False).
        rope_dim: Rotary embedding dimension. Defaults to cos_sin_cache.size(-1).
    """
    rope_dim = rope_dim or cos_sin_cache.size(-1)
    v = v.view_as(k)
    module = _jit_fused_rope_module(is_neox, rope_dim, q.dtype)
    module.run_rope_store(q, k, v, k_cache, v_cache, cos_sin_cache, positions, out_loc)


# NOTE: this name is intentionally set as the old kernel in `sgl_kernel`
```
**EN:** This block defines `apply_rope_inplace_with_kvcache`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_rope_inplace_with_kvcache`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 179-221: Function `apply_rope_with_cos_sin_cache_inplace`
```python
def apply_rope_with_cos_sin_cache_inplace(
    q: torch.Tensor,
    k: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    positions: torch.Tensor,
    *,
    is_neox: bool,
    rope_dim: int = 0,
    fused_args: Optional[FusedSetKVBufferArg] = None,
) -> None:
    """
    Apply RoPE to q and k inplace, with optional fused kv cache store.

    If `fused_args` is provided, it will perform fused RoPE and KV cache store.
    Otherwise, it will only apply RoPE inplace.

    Args:
        q: Query tensor of shape [num_tokens, num_qo_heads, head_dim].
        k: Key tensor of shape [num_tokens, num_kv_heads, head_dim].
        cos_sin_cache: Cosine/sine cache of shape [max_position, rope_dim], float32.
        positions: Position indices of shape [num_tokens], int32 or int64.
        is_neox: Whether to use GPT-NeoX style (True) or GPT-J interleaved (False).
        rope_dim: Rotary embedding dimension. Defaults to cos_sin_cache.size(-1).
        fused_args: Optional arguments for fused RoPE + KV cache store. If None,
            only RoPE will be applied inplace without touching kv cache.
    """
    if fused_args is not None:
        apply_rope_inplace_with_kvcache(
            q,
            k,
            fused_args.value,
            fused_args.k_buffer,
            fused_args.v_buffer,
            cos_sin_cache,
            positions,
            fused_args.cache_loc,
            is_neox=is_neox,
            rope_dim=rope_dim,
        )
    else:
        apply_rope_inplace(
            q, k, cos_sin_cache, positions, is_neox=is_neox, rope_dim=rope_dim
        )
```
**EN:** This block defines `apply_rope_with_cos_sin_cache_inplace`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_rope_with_cos_sin_cache_inplace`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `dataclasses -> dataclass`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> (`
- `sglang.srt.utils.custom_op -> register_custom_op`
- `tvm_ffi.module -> Module`
