# triton_reshape_and_cache_flash.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/triton_reshape_and_cache_flash.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `reshape_and_cache_kernel_flash`, `_reshape_cache_per_token_head`, `triton_reshape_and_cache_flash_per_token_head_quant` for the V1 `attention/ops` subsystem. / 为 V1 的 `attention/ops` 子系统实现 `reshape_and_cache_kernel_flash`, `_reshape_cache_per_token_head`, `triton_reshape_and_cache_flash_per_token_head_quant`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch

from vllm.model_executor.layers.quantization.utils.quant_utils import (
    FP8_DTYPE,
    get_fp8_min_max,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.torch_utils import is_quantized_kv_cache

FP8_MIN, FP8_MAX = get_fp8_min_max()
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `reshape_and_cache_kernel_flash` function / `reshape_and_cache_kernel_flash` 函数
```python
@triton.jit
def reshape_and_cache_kernel_flash(
    key_ptr,  # [num_tokens, num_heads, head_size]
    value_ptr,  # [num_tokens, num_heads, head_size]
    key_cache_ptr,  # [num_blocks, block_size, num_heads, head_size]
    value_cache_ptr,  # [num_blocks, block_size, num_heads, head_size]
    slot_mapping_ptr,  # [num_tokens]
    k_scale,  # float32
    v_scale,  # float32
    # strides
    key_stride: tl.int64,
    value_stride: tl.int64,
    block_stride: tl.int64,
    head_stride: tl.int64,
    dim_stride_k: tl.int64,
    dim_stride_v: tl.int64,
    page_stride: tl.int64,
    num_heads: tl.constexpr,
    head_size: tl.constexpr,
    block_size: tl.constexpr,
    x: tl.constexpr,
    USE_HEAD_MAJOR_LAYOUT: tl.constexpr,
    # FP8 flags
    FP8_KV_CACHE: tl.constexpr,
    # tune parameters
    TILE_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(axis=0)
    slot_idx = tl.load(slot_mapping_ptr + token_idx).to(tl.int64)
    if slot_idx < 0:
        # Padding token that should be ignored.
        return

    block_idx = slot_idx // block_size
    block_offset = slot_idx % block_size

    tile_i = tl.program_id(axis=1)
    tile_offs = tl.arange(0, TILE_SIZE)
    tile_pos = tile_i * TILE_SIZE + tile_offs
    src_key_idx = token_idx * key_stride
    src_value_idx = token_idx * value_stride

    if USE_HEAD_MAJOR_LAYOUT:
        # Decompose the tile index back into head and dim coordinates.
        cur_head = tile_pos // head_size
        cur_dim = tile_pos % head_size
        # Value addressing (4D): [Block, Head, Dim, Slot]
        tgt_idx_v = (
            block_idx * block_stride
            + cur_head * head_stride
# ... omitted for brevity ...
    # [TILE_SIZE]
    value_load = tl.load(
        value_ptr + src_value_idx + tile_pos, mask=tile_pos < (num_heads * head_size)
    )
    if FP8_KV_CACHE:
        if value_load.dtype.is_fp8():
            value_tile = value_load
        else:
            # tl.store will do the correct implicit cast to fp8,
            #  based on the value_cache_ptr.dtype.element_ty
            value_tile = value_load / tl.load(v_scale)
    else:
        value_tile = value_load

    tl.store(
        key_cache_ptr + tgt_idx_k,
        key_tile,
        mask=tile_pos < (num_heads * head_size),
    )
    tl.store(
        value_cache_ptr + tgt_idx_v,
        value_tile,
        mask=tile_pos < (num_heads * head_size),
    )
    return
```
**EN:** This function implements `reshape_and_cache_kernel_flash` within the module. Key calls include `program_id`, `to`, `arange`, `load`, `store`, `is_fp8`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `reshape_and_cache_kernel_flash`，其作用域位于the module。 关键调用包括 `program_id`, `to`, `arange`, `load`, `store`, `is_fp8`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `_reshape_cache_per_token_head` function / `_reshape_cache_per_token_head` 函数
```python
@triton.jit
def _reshape_cache_per_token_head(
    key_ptr,  # [num_tokens, num_kv_heads, head_size]
    value_ptr,  # [num_tokens, num_kv_heads, head_size_v]
    key_cache_ptr,  # [num_blocks, block_size, num_kv_heads, head_size]
    value_cache_ptr,  # [num_blocks, block_size, num_kv_heads, head_size_v]
    k_scale_cache_ptr,  # [num_blocks, block_size, num_kv_heads] float32
    v_scale_cache_ptr,  # [num_blocks, block_size, num_kv_heads] float32
    slot_mapping_ptr,  # [num_tokens]
    stride_key_tok: tl.int64,
    stride_key_head: tl.int64,
    stride_val_tok: tl.int64,
    stride_val_head: tl.int64,
    stride_kc_blk: tl.int64,  # key_cache stride over blocks
    stride_kc_slot: tl.int64,  # key_cache stride over slots
    stride_kc_head: tl.int64,  # key_cache stride over heads
    stride_vc_blk: tl.int64,
    stride_vc_slot: tl.int64,
    stride_vc_head: tl.int64,
    stride_ks_blk: tl.int64,  # k_scale_cache stride[0] (blocks)
    stride_ks_slot: tl.int64,  # k_scale_cache stride[1] (slots)
    stride_ks_head: tl.int64,  # k_scale_cache stride[2] (heads)
    stride_vs_blk: tl.int64,  # v_scale_cache stride[0] (blocks)
    stride_vs_slot: tl.int64,  # v_scale_cache stride[1] (slots)
    stride_vs_head: tl.int64,  # v_scale_cache stride[2] (heads)
    block_size: tl.constexpr,
    head_size: tl.constexpr,
    head_size_v: tl.constexpr,
    HEAD_SIZE_PADDED: tl.constexpr,  # next_power_of_2(max(head_size, head_size_v))
    QUANT_MAX: tl.constexpr = 127.0,
    QUANT_MIN: tl.constexpr = -128.0,
):
    tok = tl.program_id(0)
    head = tl.program_id(1)

    slot = tl.load(slot_mapping_ptr + tok).to(tl.int64)
    if slot < 0:
        return

    blk = slot // block_size
    slot_in_blk = slot % block_size

    dim_offs = tl.arange(0, HEAD_SIZE_PADDED)

    # ---- Key: load one head → absmax → quantize → store -------------------
    k_mask = dim_offs < head_size
    k_h = tl.load(
        key_ptr + tok * stride_key_tok + head * stride_key_head + dim_offs,
        mask=k_mask,
        other=0.0,
# ... omitted for brevity ...
    v_h = tl.load(
        value_ptr + tok * stride_val_tok + head * stride_val_head + dim_offs,
        mask=v_mask,
        other=0.0,
    ).to(tl.float32)

    v_scale = tl.maximum(tl.max(tl.abs(v_h)) / QUANT_MAX, 1e-6)
    tl.store(
        v_scale_cache_ptr
        + blk * stride_vs_blk
        + slot_in_blk * stride_vs_slot
        + head * stride_vs_head,
        v_scale,
    )

    v_q = tl.clamp(v_h * (1.0 / v_scale), QUANT_MIN, QUANT_MAX)
    tl.store(
        value_cache_ptr
        + blk * stride_vc_blk
        + slot_in_blk * stride_vc_slot
        + head * stride_vc_head
        + dim_offs,
        v_q,
        mask=v_mask,
    )
```
**EN:** This function implements `_reshape_cache_per_token_head` within the module. Key calls include `program_id`, `to`, `arange`, `maximum`, `store`, `clamp`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_reshape_cache_per_token_head`，其作用域位于the module。 关键调用包括 `program_id`, `to`, `arange`, `maximum`, `store`, `clamp`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### Module constants / 模块常量
```python
_PER_TOKEN_HEAD_QUANT_PARAMS: dict[torch.dtype, tuple[float, float]] = {
    torch.int8: (127.0, -128.0),
    FP8_DTYPE: (FP8_MAX, FP8_MIN),
}
```
**EN:** Defines module-level constants or aliases such as `_PER_TOKEN_HEAD_QUANT_PARAMS`, which are reused by later definitions.
**CN:** 定义 `_PER_TOKEN_HEAD_QUANT_PARAMS` 等模块级常量或别名，供后续定义复用。

### `triton_reshape_and_cache_flash_per_token_head_quant` function / `triton_reshape_and_cache_flash_per_token_head_quant` 函数
```python
def triton_reshape_and_cache_flash_per_token_head_quant(
    key: torch.Tensor,  # [num_tokens, num_kv_heads, head_size]
    value: torch.Tensor,  # [num_tokens, num_kv_heads, head_size_v]
    key_cache: torch.Tensor,  # [num_blocks, block_size, num_kv_heads, head_size]
    value_cache: torch.Tensor,  # [num_blocks, block_size, num_kv_heads, head_size_v]
    k_scale_cache: torch.Tensor,  # [num_blocks, block_size, num_kv_heads] float32
    v_scale_cache: torch.Tensor,  # [num_blocks, block_size, num_kv_heads] float32
    slot_mapping: torch.Tensor,  # [num_tokens]
):
    """Quantize key/value per (token, head) and write to paged cache.

    Computes one scale = absmax / QUANT_MAX per (token, head), stores
    quantized data in key_cache/value_cache, and stores the float32
    scale in k_scale_cache/v_scale_cache.

    The quantization range (QUANT_MAX, QUANT_MIN) is derived from the
    cache tensor dtype so the same code path works for int8 and fp8.
    """
    cache_dtype = key_cache.dtype
    quant_params = _PER_TOKEN_HEAD_QUANT_PARAMS.get(cache_dtype)
    if quant_params is None:
        raise ValueError(
            f"Per-token-head quantization not supported for cache dtype "
            f"{cache_dtype}.  Supported: {list(_PER_TOKEN_HEAD_QUANT_PARAMS)}"
        )
    quant_max, quant_min = quant_params

    num_tokens, num_kv_heads, head_size = key.shape
    head_size_v = value.shape[2]
    head_size_padded = triton.next_power_of_2(max(head_size, head_size_v))

    block_size = key_cache.shape[1]

    if current_platform.is_rocm() or current_platform.is_xpu():
        num_warps = 4
    else:
        num_warps = min(16, max(1, head_size_padded // 32))

    _reshape_cache_per_token_head[(num_tokens, num_kv_heads)](
        key_ptr=key,
        value_ptr=value,
        key_cache_ptr=key_cache,
        value_cache_ptr=value_cache,
        k_scale_cache_ptr=k_scale_cache,
        v_scale_cache_ptr=v_scale_cache,
        slot_mapping_ptr=slot_mapping,
        stride_key_tok=key.stride(0),
        stride_key_head=key.stride(1),
        stride_val_tok=value.stride(0),
        stride_val_head=value.stride(1),
        stride_kc_blk=key_cache.stride(0),
        stride_kc_slot=key_cache.stride(1),
        stride_kc_head=key_cache.stride(2),
        stride_vc_blk=value_cache.stride(0),
        stride_vc_slot=value_cache.stride(1),
        stride_vc_head=value_cache.stride(2),
        stride_ks_blk=k_scale_cache.stride(0),
        stride_ks_slot=k_scale_cache.stride(1),
        stride_ks_head=k_scale_cache.stride(2),
        stride_vs_blk=v_scale_cache.stride(0),
        stride_vs_slot=v_scale_cache.stride(1),
        stride_vs_head=v_scale_cache.stride(2),
        block_size=block_size,
        head_size=head_size,
        head_size_v=head_size_v,
        HEAD_SIZE_PADDED=head_size_padded,
        QUANT_MAX=quant_max,
        QUANT_MIN=quant_min,
        num_warps=num_warps,
    )
```
**EN:** This function implements `triton_reshape_and_cache_flash_per_token_head_quant` within the module. The docstring frames it as: Quantize key/value per (token, head) and write to paged cache. Key calls include `get`, `next_power_of_2`, `ValueError`, `max`, `is_rocm`, `is_xpu`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `triton_reshape_and_cache_flash_per_token_head_quant`，其作用域位于the module。 关键调用包括 `get`, `next_power_of_2`, `ValueError`, `max`, `is_rocm`, `is_xpu`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `triton_reshape_and_cache_flash` function / `triton_reshape_and_cache_flash` 函数
```python
def triton_reshape_and_cache_flash(
    key: torch.Tensor,  # [num_tokens, num_heads, head_size]
    value: torch.Tensor,  # [num_tokens, num_heads, head_size]
    # [num_blocks, block_size, num_heads, head_size]
    key_cache: torch.Tensor,
    # [num_blocks, block_size, num_heads, head_size]
    value_cache: torch.Tensor,
    slot_mapping: torch.Tensor,  # [num_tokens]
    kv_cache_dtype: str,  # "auto", "fp8"
    k_scale: torch.Tensor,  # float32
    v_scale: torch.Tensor,  # float32
):
    num_heads = key.shape[1]
    head_size = key.shape[2]

    use_head_major_layout = key_cache.ndim == 5
    if use_head_major_layout:
        block_size = key_cache.shape[3]
        x = key_cache.shape[4]
        head_stride = key_cache.stride(1)
        dim_stride_k = key_cache.stride(2)
        dim_stride_v = value_cache.stride(2)
    else:
        block_size = key_cache.shape[1]
        x = 1
        dim_stride_k = 0
        dim_stride_v = 0
        head_stride = key_cache.stride()[2]
    n = num_heads * head_size
    key_stride = key.stride()[0]
    value_stride = value.stride()[0]
    block_stride = key_cache.stride()[0]
    page_stride = key_cache.stride()[1]

    assert kv_cache_dtype == "auto" or is_quantized_kv_cache(kv_cache_dtype), (
        f"unsupported kv_cache_dtype (str), got {kv_cache_dtype}."
    )
    kv_cache_torch_dtype = (
        current_platform.fp8_dtype()
        if is_quantized_kv_cache(kv_cache_dtype)
        else key_cache.dtype
    )

    if key_cache.dtype != kv_cache_torch_dtype and is_quantized_kv_cache(
        kv_cache_dtype
    ):
        # to avoid erounous implicit cast in triton kernel (tl.store to uint8)
        # (e.g. explicit cast to fp8e4m3fnuz is not supported in triton 3.4)
        key_cache = key_cache.view(kv_cache_torch_dtype)
        value_cache = value_cache.view(kv_cache_torch_dtype)
# ... omitted for brevity ...
        value_ptr=value,
        key_cache_ptr=key_cache,
        value_cache_ptr=value_cache,
        slot_mapping_ptr=slot_mapping,
        k_scale=k_scale,
        v_scale=v_scale,
        # strides
        key_stride=key_stride,
        value_stride=value_stride,
        block_stride=block_stride,
        head_stride=head_stride,
        dim_stride_k=dim_stride_k,
        dim_stride_v=dim_stride_v,
        page_stride=page_stride,
        num_heads=num_heads,
        head_size=head_size,
        block_size=block_size,
        x=x,
        USE_HEAD_MAJOR_LAYOUT=use_head_major_layout,
        FP8_KV_CACHE=FP8_KV_CACHE,
        # autotune parameters
        TILE_SIZE=TILE_SIZE,
        num_warps=num_warps,
        num_stages=num_stages,
    )
```
**EN:** This function implements `triton_reshape_and_cache_flash` within the module. Key calls include `is_quantized_kv_cache`, `min`, `stride`, `fp8_dtype`, `view`, `next_power_of_2`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `triton_reshape_and_cache_flash`，其作用域位于the module。 关键调用包括 `is_quantized_kv_cache`, `min`, `stride`, `fp8_dtype`, `view`, `next_power_of_2`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `reshape_and_cache_kernel_flash_diffkv` function / `reshape_and_cache_kernel_flash_diffkv` 函数
```python
@triton.jit
def reshape_and_cache_kernel_flash_diffkv(
    key_ptr,  # [num_tokens, num_heads, head_size]
    value_ptr,  # [num_tokens, num_heads, head_size_v]
    kv_cache_ptr,  # [num_blocks, block_size, num_heads, head_size + head_size_v]
    slot_mapping_ptr,  # [num_tokens]
    k_scale,  # float32
    v_scale,  # float32
    # strides
    key_stride: tl.int64,
    value_stride: tl.int64,
    block_stride: tl.int64,
    page_stride: tl.int64,
    num_heads: tl.constexpr,
    head_size_k: tl.constexpr,
    head_size_v: tl.constexpr,
    block_size: tl.constexpr,
    # FP8 flags
    FP8_KV_CACHE: tl.constexpr,
    # tune parameters
    TILE_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(axis=0)
    slot_idx = tl.load(slot_mapping_ptr + token_idx).to(tl.int64)
    if slot_idx < 0:
        # Padding token that should be ignored.
        return

    tile_i = tl.program_id(axis=1)
    tile_offs = tl.arange(0, TILE_SIZE)

    block_idx = slot_idx // block_size
    block_offset = slot_idx % block_size

    src_key_idx = token_idx * key_stride + tile_i * head_size_k
    src_value_idx = token_idx * value_stride + tile_i * head_size_v

    tgt_idx = (
        block_idx * block_stride
        + block_offset * page_stride
        + tile_i * (head_size_k + head_size_v)
    )

    # [TILE_SIZE]
    key_load = tl.load(key_ptr + src_key_idx + tile_offs, mask=tile_offs < head_size_k)
    if FP8_KV_CACHE:
        # tl.store will do the correct implicit cast to fp8,
        # based on the key_cache_ptr.dtype.element_ty
        key_tile = key_load if key_load.dtype.is_fp8() else key_load / tl.load(k_scale)
    else:
        key_tile = key_load

    # [TILE_SIZE]
    value_load = tl.load(
        value_ptr + src_value_idx + tile_offs, mask=tile_offs < head_size_v
    )
    if FP8_KV_CACHE:
        if value_load.dtype.is_fp8():
            value_tile = value_load
        else:
            # tl.store will do the correct implicit cast to fp8,
            #  based on the value_cache_ptr.dtype.element_ty
            value_tile = value_load / tl.load(v_scale)
    else:
        value_tile = value_load

    tl.store(
        kv_cache_ptr + tgt_idx + tile_offs,
        key_tile,
        mask=tile_offs < head_size_k,
    )
    tl.store(
        kv_cache_ptr + tgt_idx + head_size_k + tile_offs,
        value_tile,
        mask=tile_offs < head_size_v,
    )
    return
```
**EN:** This function implements `reshape_and_cache_kernel_flash_diffkv` within the module. Key calls include `program_id`, `to`, `arange`, `load`, `store`, `is_fp8`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `reshape_and_cache_kernel_flash_diffkv`，其作用域位于the module。 关键调用包括 `program_id`, `to`, `arange`, `load`, `store`, `is_fp8`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `triton_reshape_and_cache_flash_diffkv` function / `triton_reshape_and_cache_flash_diffkv` 函数
```python
def triton_reshape_and_cache_flash_diffkv(
    key: torch.Tensor,  # [num_tokens, num_heads, head_size]
    value: torch.Tensor,  # [num_tokens, num_heads, head_size_v]
    # [num_blocks, block_size, num_heads, head_size + head_size_v]
    kv_cache: torch.Tensor,
    slot_mapping: torch.Tensor,  # [num_tokens]
    kv_cache_dtype: str,  # "auto", "fp8"
    k_scale: torch.Tensor,  # float32
    v_scale: torch.Tensor,  # float32
):
    num_heads = key.shape[1]
    head_size_k = key.shape[2]
    head_size_v = value.shape[2]
    block_size = kv_cache.shape[1]

    k_stride = key.stride()[0]
    v_stride = value.stride()[0]
    block_stride = kv_cache.stride()[0]
    page_stride = kv_cache.stride()[1]

    assert kv_cache_dtype == "auto" or is_quantized_kv_cache(kv_cache_dtype), (
        f"unsupported kv_cache_dtype (str), got {kv_cache_dtype}."
    )
    kv_cache_torch_dtype = (
        current_platform.fp8_dtype()
        if is_quantized_kv_cache(kv_cache_dtype)
        else kv_cache.dtype
    )

    if kv_cache.dtype != kv_cache_torch_dtype and is_quantized_kv_cache(kv_cache_dtype):
        # to avoid erounous implicit cast in triton kernel (tl.store to uint8)
        # (e.g. explicit cast to fp8e4m3fnuz is not supported in triton 3.4)
        kv_cache = kv_cache.view(kv_cache_torch_dtype)
    assert kv_cache_dtype != torch.uint8, (
        "explicit fp8 cast and store to "
        "uint8 is not supported by triton reshape_and_cache_flash_diffkv"
    )

    FP8_KV_CACHE = is_quantized_kv_cache(kv_cache_dtype)
    assert (not FP8_KV_CACHE) or kv_cache_torch_dtype in [
        torch.float8_e4m3fn,
        torch.float8_e5m2,
        torch.uint8,
        torch.float8_e4m3fnuz,
    ], (
        "unsupported dtype of KV cache tensor, got "
        "{kv_cache_torch_dtype}. Supported kv cache dtypes: fp8e4m3fn, "
        "fp8e5m2, uint8, bfloat16, float16, float32, fp8e4m3fnuz."
    )

    # heuristics instead of autotuning
    TILE_SIZE = max(head_size_k, head_size_v)
    TILE_SIZE = triton.next_power_of_2(TILE_SIZE)
    if current_platform.is_rocm() or current_platform.is_xpu():
        num_stages = 4
        num_warps = 8
    else:  # cuda
        num_stages = 10
        num_warps = 16

    # TODO(ngl): maybe replace with static launch grid to avoid overhead if
    #   using cudagraphs
    grid = lambda meta: (
        slot_mapping.shape[0],
        num_heads,
    )

    reshape_and_cache_kernel_flash_diffkv[grid](
        key_ptr=key,
        value_ptr=value,
        kv_cache_ptr=kv_cache,
        slot_mapping_ptr=slot_mapping,
        k_scale=k_scale,
        v_scale=v_scale,
        # strides
        key_stride=k_stride,
        value_stride=v_stride,
        block_stride=block_stride,
        page_stride=page_stride,
        num_heads=num_heads,
        head_size_k=head_size_k,
        head_size_v=head_size_v,
        block_size=block_size,
        # FP8 flags
        FP8_KV_CACHE=FP8_KV_CACHE,
        # autotune parameters
        TILE_SIZE=TILE_SIZE,
        num_warps=num_warps,
        num_stages=num_stages,
    )
```
**EN:** This function implements `triton_reshape_and_cache_flash_diffkv` within the module. Key calls include `is_quantized_kv_cache`, `max`, `next_power_of_2`, `stride`, `fp8_dtype`, `view`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `triton_reshape_and_cache_flash_diffkv`，其作用域位于the module。 关键调用包括 `is_quantized_kv_cache`, `max`, `next_power_of_2`, `stride`, `fp8_dtype`, `view`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `reshape_and_cache_kernel_flash`: top-level helper or orchestration entry point. / `reshape_and_cache_kernel_flash`：顶层辅助函数或编排入口。
- `_reshape_cache_per_token_head`: top-level helper or orchestration entry point. / `_reshape_cache_per_token_head`：顶层辅助函数或编排入口。
- `triton_reshape_and_cache_flash_per_token_head_quant`: top-level helper or orchestration entry point. / `triton_reshape_and_cache_flash_per_token_head_quant`：顶层辅助函数或编排入口。
- `triton_reshape_and_cache_flash`: top-level helper or orchestration entry point. / `triton_reshape_and_cache_flash`：顶层辅助函数或编排入口。
- `reshape_and_cache_kernel_flash_diffkv`: top-level helper or orchestration entry point. / `reshape_and_cache_kernel_flash_diffkv`：顶层辅助函数或编排入口。
- `triton_reshape_and_cache_flash_diffkv`: top-level helper or orchestration entry point. / `triton_reshape_and_cache_flash_diffkv`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.torch_utils`
