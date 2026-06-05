# quant_k_cache.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/nsa/quant_k_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages quant k cache logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 quant k cache 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-3: imports
```python
import torch
import triton
import triton.language as tl
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 6-7: function quantize_k_cache
```python
def quantize_k_cache(cache_k):
    return _quantize_k_cache_fast_wrapped(cache_k)
```
**EN:** Implements the quantize k cache routine used by this attention module.
**CN:** 实现该注意力模块使用的 quantize k cache 例程。

### Lines 10-55: function quantize_k_cache_separate
```python
def quantize_k_cache_separate(
    k_nope: torch.Tensor,
    k_rope: torch.Tensor,
    tile_size: int = 128,
):
    """
    Quantize k_nope and k_rope separately without concat, returns two tensors.

    This avoids the concat operation and enables direct reuse of set_mla_kv_buffer_triton
    by returning two separate byte tensors for the nope and rope parts.

    Args:
        k_nope: (num_tokens, dim_nope) or (num_tokens, 1, dim_nope)
                Must have dim_nope=512 for FP8 MLA quantization
        k_rope: (num_tokens, dim_rope) or (num_tokens, 1, dim_rope)
                Must have dim_rope=64 for FP8 MLA quantization
        tile_size: quantization tile size (default 128)

    Returns:
        Tuple of (nope_part, rope_part) where:
        - nope_part: (num_tokens, 1, 528) as uint8 view, contains [nope_fp8(512) | scales(16)]
        - rope_part: (num_tokens, 1, 128) as uint8 view, contains [rope_bf16_bytes(128)]

        These two tensors can be directly passed to set_mla_kv_buffer_triton(kv_buffer, loc, nope_part, rope_part)
    """
    # Squeeze middle dimension if present
    k_nope_2d = k_nope.squeeze(1) if k_nope.ndim == 3 else k_nope
    k_rope_2d = k_rope.squeeze(1) if k_rope.ndim == 3 else k_rope

    num_tokens = k_nope_2d.shape[0]
    dim_nope = k_nope_2d.shape[1]
    dim_rope = k_rope_2d.shape[1]

    # Validate dimensions for FP8 MLA
    if dim_nope != 512:
        raise ValueError(f"Expected dim_nope=512 for FP8 MLA, got {dim_nope}")
    if dim_rope != 64:
        raise ValueError(f"Expected dim_rope=64 for FP8 MLA, got {dim_rope}")
    if k_rope_2d.shape[0] != num_tokens:
        raise ValueError(
            f"k_nope and k_rope must have same num_tokens, got {num_tokens} vs {k_rope_2d.shape[0]}"
        )

    return _quantize_k_cache_fast_separate(
        k_nope=k_nope_2d, k_rope=k_rope_2d, group_size=tile_size
    )
```
**EN:** Implements the quantize k cache separate routine used by this attention module.
**CN:** 实现该注意力模块使用的 quantize k cache separate 例程。

### Lines 59-109: function _quantize_k_cache_ref
```python
def _quantize_k_cache_ref(
    input_k_cache: torch.Tensor,  # (num_blocks, block_size, h_k, d)
    dv: int = 512,
    tile_size: int = 128,
) -> torch.Tensor:
    """
    Quantize the k-cache
    Return a tensor with shape (num_blocks, block_size, h_k, dv + 4(dv/tile_size) + t(d-dv)) of dtype uint8_t, where t = input_k_cache.element_size()
    For more detail about the layout of K/V, please refer to comments in flash_mla_interface.py or README.md
    """
    assert dv % tile_size == 0
    num_tiles = dv // tile_size
    num_blocks, block_size, h_k, d = input_k_cache.shape
    assert h_k == 1
    input_k_cache = input_k_cache.squeeze(2)  # [num_blocks, block_size, d]
    input_elem_size = input_k_cache.element_size()

    result = torch.empty(
        (num_blocks, block_size, dv + num_tiles * 4 + input_elem_size * (d - dv)),
        dtype=torch.float8_e4m3fn,
        device=input_k_cache.device,
    )
    result_k_nope_part = result[..., :dv]
    result_k_scale_factor = result[..., dv : dv + num_tiles * 4].view(torch.float32)
    result_k_rope_part = result[..., dv + num_tiles * 4 :].view(input_k_cache.dtype)
    result_k_rope_part[:] = input_k_cache[..., dv:]

    for tile_idx in range(0, num_tiles):
        cur_scale_factors_inv = (
            torch.abs(
                input_k_cache[..., tile_idx * tile_size : (tile_idx + 1) * tile_size]
            )
# ... omitted 7 lines ...
        cur_quantized_nope = (
            input_k_cache[
                ..., tile_idx * tile_size : (tile_idx + 1) * tile_size
            ].float()
            / cur_scale_factors_inv.float()
        ).to(torch.float8_e4m3fn)
        result_k_nope_part[..., tile_idx * tile_size : (tile_idx + 1) * tile_size] = (
            cur_quantized_nope
        )

    result = result.view(num_blocks, block_size, 1, -1)
    return result
```
**EN:** Implements the quantize k cache ref routine used by this attention module.
**CN:** 实现该注意力模块使用的 quantize k cache ref 例程。

### Lines 112-130: function _quantize_k_cache_fast_wrapped
```python
def _quantize_k_cache_fast_wrapped(
    input_k_cache: torch.Tensor,
    dv: int = 512,
    tile_size: int = 128,
) -> torch.Tensor:
    # TODO the final API may be 2D instead of 4D, thus we convert them here
    num_blocks, block_size, _, dim_nope_and_rope = input_k_cache.shape
    assert dv == 512
    assert dim_nope_and_rope == 512 + 64
    assert tile_size == 128
    input_k_cache = input_k_cache.view((-1, dim_nope_and_rope))

    # TODO deliberately split into two tensors, then upstream can provide the two tensors instead of concat into one
    k_nope = input_k_cache[:, :dv]
    k_rope = input_k_cache[:, dv:]

    output = _quantize_k_cache_fast(k_nope=k_nope, k_rope=k_rope)

    return output.view(num_blocks, block_size, 1, -1)
```
**EN:** Implements the quantize k cache fast wrapped routine used by this attention module.
**CN:** 实现该注意力模块使用的 quantize k cache fast wrapped 例程。

### Lines 133-187: function _quantize_k_cache_fast
```python
def _quantize_k_cache_fast(k_nope, k_rope, group_size: int = 128):
    """
    :param k_nope: (num_tokens, dim_nope 512)
    :param k_rope: (num_tokens, dim_rope 64)
    """

    assert k_nope.dtype == torch.bfloat16
    assert k_rope.dtype == torch.bfloat16

    num_tokens, dim_nope = k_nope.shape
    num_tokens_, dim_rope = k_rope.shape
    assert num_tokens == num_tokens_
    assert dim_nope == 512
    assert dim_rope == 64
    assert k_nope.dtype == k_rope.dtype
    num_tiles = dim_nope // group_size

    assert k_nope.stride(1) == 1
    assert k_rope.stride(1) == 1

    output = torch.empty(
        (num_tokens, dim_nope + num_tiles * 4 + k_rope.element_size() * dim_rope),
        dtype=torch.float8_e4m3fn,
        device=k_nope.device,
    )
    output_nope_q = output[..., :dim_nope]
    output_nope_s = output[..., dim_nope : dim_nope + num_tiles * 4].view(torch.float32)
    output_rope = output[..., dim_nope + num_tiles * 4 :].view(torch.bfloat16)

    num_blocks_per_token = triton.cdiv(dim_nope + dim_rope, group_size)
    assert num_blocks_per_token == 5

# ... omitted 11 lines ...
        output_rope.stride(0),
        k_nope.stride(0),
        k_rope.stride(0),
        NUM_NOPE_BLOCKS=NUM_NOPE_BLOCKS,
        GROUP_SIZE=group_size,
        DIM_NOPE=dim_nope,
        DIM_ROPE=dim_rope,
        FP8_MIN=torch.finfo(torch.float8_e4m3fn).min,
        FP8_MAX=torch.finfo(torch.float8_e4m3fn).max,
    )

    return output
```
**EN:** Implements the quantize k cache fast routine used by this attention module.
**CN:** 实现该注意力模块使用的 quantize k cache fast 例程。

### Lines 190-264: function _quantize_k_cache_fast_separate
```python
def _quantize_k_cache_fast_separate(k_nope, k_rope, group_size: int = 128):
    """
    Quantize k_nope and k_rope in a single Triton kernel, directly outputting two separate tensors.

    This avoids packing/unpacking and enables direct use with set_mla_kv_buffer_triton.

    :param k_nope: (num_tokens, dim_nope 512) bfloat16
    :param k_rope: (num_tokens, dim_rope 64) bfloat16
    :param group_size: quantization tile size (default 128, kernel is tuned for this value)
    :return: Tuple of (nope_part_u8, rope_part_u8)
        - nope_part_u8: (num_tokens, 1, nope_part_bytes) uint8, layout [nope_fp8(dim_nope) | scales(num_tiles*4)]
        - rope_part_u8: (num_tokens, 1, rope_part_bytes) uint8, layout [rope_bf16_bytes(dim_rope*2)]
    """
    num_tokens, dim_nope = k_nope.shape
    num_tokens_, dim_rope = k_rope.shape

    assert num_tokens == num_tokens_, f"k_nope and k_rope must have same num_tokens"

    # Ensure contiguous tensors for kernel
    k_nope = k_nope.contiguous()
    k_rope = k_rope.contiguous()

    num_tiles = dim_nope // group_size

    # Calculate byte sizes based on validated dimensions
    # nope_part: [FP8 quantized data (dim_nope bytes)] + [FP32 scales (num_tiles * 4 bytes)]
    # rope_part: [BF16 raw data (dim_rope * 2 bytes)]
    nope_part_bytes = (
        dim_nope + num_tiles * 4
    )  # e.g., 512 + 4*4 = 528 for dim_nope=512, group_size=128
    rope_part_bytes = (
        dim_rope * k_rope.element_size()
# ... omitted 31 lines ...
        k_nope.stride(0),
        k_rope.stride(0),
        NUM_NOPE_BLOCKS=NUM_NOPE_BLOCKS,
        GROUP_SIZE=group_size,
        DIM_NOPE=dim_nope,
        DIM_ROPE=dim_rope,
        FP8_MIN=torch.finfo(torch.float8_e4m3fn).min,
        FP8_MAX=torch.finfo(torch.float8_e4m3fn).max,
    )

    # Add middle dimension for compatibility with set_mla_kv_buffer_triton
    return nope_part_u8.unsqueeze(1), rope_part_u8.unsqueeze(1)
```
**EN:** Implements the quantize k cache fast separate routine used by this attention module.
**CN:** 实现该注意力模块使用的 quantize k cache fast separate 例程。

### Lines 267-324: function _quantize_k_cache_fast_kernel
```python
@triton.jit
def _quantize_k_cache_fast_kernel(
    output_nope_q_ptr,
    output_nope_s_ptr,
    output_rope_ptr,
    k_nope_ptr,
    k_rope_ptr,
    output_nope_q_stride_0: int,
    output_nope_s_stride_0: int,
    output_rope_stride_0: int,
    k_nope_stride_0: int,
    k_rope_stride_0: int,
    NUM_NOPE_BLOCKS: tl.constexpr,
    GROUP_SIZE: tl.constexpr,
    DIM_NOPE: tl.constexpr,
    DIM_ROPE: tl.constexpr,
    FP8_MIN: tl.constexpr,
    FP8_MAX: tl.constexpr,
):
    token_id = tl.program_id(0)
    raw_block_id = tl.program_id(1)

    if raw_block_id < NUM_NOPE_BLOCKS:
        # a. quant nope
        effective_block_id = raw_block_id

        offs = effective_block_id * GROUP_SIZE + tl.arange(0, GROUP_SIZE)
        mask = offs < DIM_NOPE
        ptr = k_nope_ptr + token_id * k_nope_stride_0 + offs

        y = tl.load(ptr, mask=mask, other=0.0).to(tl.float32)

# ... omitted 14 lines ...
    else:
        # b. copy rope
        effective_block_id = raw_block_id - NUM_NOPE_BLOCKS

        offs = effective_block_id * GROUP_SIZE + tl.arange(0, GROUP_SIZE)
        mask = offs < DIM_ROPE

        src_ptr = k_rope_ptr + token_id * k_rope_stride_0 + offs
        dst_ptr = output_rope_ptr + token_id * output_rope_stride_0 + offs

        data = tl.load(src_ptr, mask=mask)
        tl.store(dst_ptr, data, mask=mask)
```
**EN:** Implements the quantize k cache fast kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 quantize k cache fast kernel 例程。

### Lines 327-449: conditional branch
```python
if __name__ == "__main__":
    import dequant_k_cache

    for num_blocks, block_size in [
        (1, 1),
        (10, 64),
    ]:
        dim_nope_and_rope = 512 + 64

        input_k_cache = torch.randn(
            (num_blocks, block_size, 1, dim_nope_and_rope),
            dtype=torch.bfloat16,
            device="cuda",
        )

        ref_quant = _quantize_k_cache_ref(input_k_cache)
        actual_quant = _quantize_k_cache_fast_wrapped(input_k_cache)

        ref_ref_dequant = dequant_k_cache._dequantize_k_cache_slow(ref_quant)
        ref_actual_dequant = dequant_k_cache._dequantize_k_cache_fast_wrapped(ref_quant)
        actual_actual_dequant = dequant_k_cache._dequantize_k_cache_fast_wrapped(
            actual_quant
        )

        print(f"{ref_ref_dequant=}")
        print(f"{actual_actual_dequant=}")
        print(f"{actual_actual_dequant - ref_ref_dequant=}")
        print(f"{torch.mean(ref_ref_dequant - actual_actual_dequant)=}")

        # TODO too different?
        torch.testing.assert_close(
            ref_ref_dequant, ref_actual_dequant, atol=0.2, rtol=0.2
# ... omitted 79 lines ...

        actual_quant = _quantize_k_cache_fast_wrapped(input_k_cache)

        page_table_1 = torch.arange(
            num_blocks * block_size, dtype=torch.int32, device="cuda"
        )

        def run_ans():
            return dequant_k_cache.dequantize_k_cache_paged(actual_quant, page_table_1)

        ans_time: float = triton.testing.do_bench(run_ans, warmup=10, rep=20) / 1000  # type: ignore
        print(f"seq_kv: {num_blocks * block_size}, time: {ans_time * 1e6: 4.0f} us")
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language`
- `dequant_k_cache`
