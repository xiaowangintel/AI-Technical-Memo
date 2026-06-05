# cache_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/deepseek_v4_ops/cache_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Triton kernels for DeepseekV4 paged K-cache management and sparse-attention index preparation. / 该模块位于 `attention/ops/deepseek_v4_ops` 子系统，主要围绕 `quantize_and_insert_k_kernel`, `quantize_and_insert_k_cache`, `_dequantize_and_gather_k_kernel` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
Triton kernels for DeepseekV4 paged K-cache management and sparse-attention index
preparation.

- quantize_and_insert_k_cache: quantize bf16 K to UE8M0 FP8 and insert into
  the paged cache.
- dequantize_and_gather_k_cache: gather and dequantize FP8 K from the paged
  cache for sparse/SWA prefill.
- compute_global_topk_indices_and_lens: map local topk indices to global KV
  cache slots and count valid entries.
- combine_topk_swa_indices: concatenate topk compressed indices with SWA
  window indices for sparse prefill.
"""

import torch

from vllm.triton_utils import tl, triton
from vllm.utils.import_utils import has_cutedsl
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `quantize_and_insert_k_kernel` function / `quantize_and_insert_k_kernel` 函数
```python
@triton.jit
def quantize_and_insert_k_kernel(
    # Input tensors
    k_ptr,  # [num_tokens, 512] bf16
    slot_mapping_ptr,  # [num_tokens] int64
    # Output tensor
    k_cache_ptr,  # [num_blocks, block_bytes] as uint8 (flattened view)
    # Dimensions
    num_tokens,
    input_dim: tl.constexpr,  # 512
    fp8_dim: tl.constexpr,  # 448
    bf16_dim: tl.constexpr,  # 64
    scale_dim: tl.constexpr,  # 8
    quant_block: tl.constexpr,  # 64 (quantization block size)
    cache_block_size: tl.constexpr,  # 64 (paged cache block size)
    token_data_size: tl.constexpr,  # 576 bytes per token data
    block_stride: tl.constexpr,  # total bytes per block (padded)
    fp8_max: tl.constexpr,
    n_quant_blocks: tl.constexpr,  # 8 (7 real + 1 padding)
):
    """
    Quantize K tensor and insert into paged K cache.

    K Cache block layout (block_size=64 tokens):
    - [0, 64*576): Token data, each token has 448 fp8 + 128 bf16
    - [64*576, 64*576 + 64*8): Scales, each token has 8 uint8 scales
    - [64*576 + 64*8, block_stride): Padding

    One program per token.
    """
    pid = tl.program_id(0)

    if pid >= num_tokens:
        return

    # Get slot mapping
    slot_idx = tl.load(slot_mapping_ptr + pid)
    if slot_idx == -1:
        return

    block_idx = slot_idx // cache_block_size
    pos_in_block = slot_idx % cache_block_size

    # Input pointer for this token
    input_row_ptr = k_ptr + pid * input_dim

    # int64: block_idx * block_stride can exceed 2^31 with many KV-cache blocks
    # (e.g. >= 57K at block_stride ~37K). Matches gather path below.
    cache_block_ptr = k_cache_ptr + block_idx.to(tl.int64) * block_stride

# ... omitted for brevity ...
            # Convert to fp8, then bitcast to uint8 for storage
            x_fp8 = x_clamped.to(tl.float8e4nv)
            x_uint8 = x_fp8.to(tl.uint8, bitcast=True)

            # Store as uint8 (1 byte each)
            tl.store(token_fp8_ptr + offsets, x_uint8, mask=mask)

            # UE8M0 scale encoding: stored_value = exponent + 127 (bias)
            # During dequant: scale = 2^(stored_value - 127)
            encoded_scale = exponent + 127.0
            encoded_scale = tl.maximum(tl.minimum(encoded_scale, 255.0), 0.0)
            tl.store(token_scale_ptr + qblock_idx, encoded_scale.to(tl.uint8))

    # Padding scale at index 7
    tl.store(token_scale_ptr + 7, tl.zeros((), dtype=tl.uint8))

    # ========== Store BF16 portion (last 64 elements, no quantization) ==========
    bf16_input_offset = fp8_dim

    # Process bf16 in chunks of 16
    bf16_out_ptr = token_bf16_ptr.to(tl.pointer_type(tl.bfloat16))
    for i in tl.static_range(bf16_dim // 16):
        chunk_offsets = i * 16 + tl.arange(0, 16)
        bf16_vals = tl.load(input_row_ptr + bf16_input_offset + chunk_offsets)
        tl.store(bf16_out_ptr + chunk_offsets, bf16_vals)
```
**EN:** This function implements `quantize_and_insert_k_kernel` within the module. The docstring frames it as: Quantize K tensor and insert into paged K cache. Key calls include `program_id`, `load`, `static_range`, `store`, `to`, `zeros`. The control flow contains 3 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `quantize_and_insert_k_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `static_range`, `store`, `to`, `zeros`。 控制流包含 3 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `quantize_and_insert_k_cache` function / `quantize_and_insert_k_cache` 函数
```python
def quantize_and_insert_k_cache(
    k: torch.Tensor,  # [num_tokens, 512] bf16
    k_cache: torch.Tensor,  # [num_blocks, block_bytes] uint8
    slot_mapping: torch.Tensor,  # [num_tokens] int64
    block_size: int = 64,
    is_ue8m0: bool = True,
):
    """
    Quantize K tensor and insert into paged K cache.

    K Cache block layout (block_size=64 tokens):
    - First 64 * 576 = 36864 bytes: Token data
      - Each token: 448 bytes (fp8) + 128 bytes (bf16)
    - Next 64 * 8 = 512 bytes: Scales
      - Each token: 8 bytes (uint8 scales, 7 real + 1 padding)
    - Padded to multiple of 576
    """
    assert k.dim() == 2 and k.shape[1] == 512, (
        f"K must be [num_tokens, 512], got {k.shape}"
    )
    assert k.dtype == torch.bfloat16, f"K must be bf16, got {k.dtype}"
    assert is_ue8m0, "Only support ue8m0 quantization."

    # NOTE: When using DP, slot_mapping.shape[0] can be less than k.shape[0] due to
    # padding. Always use slot_mapping.shape[0] as the token count.
    num_tokens = slot_mapping.shape[0]
    block_stride = k_cache.stride(0)  # bytes per block

    TOKEN_FP8_DIM = 448
    TOKEN_BF16_DIM = 64
    TOKEN_SCALE_DIM = 8
    QUANT_BLOCK_SIZE = 64
    FP8_MAX = 448.0
    TOKEN_DATA_SIZE = TOKEN_FP8_DIM + TOKEN_BF16_DIM * 2

    grid = (num_tokens,)

    quantize_and_insert_k_kernel[grid](
        k,
        slot_mapping,
        k_cache,
        num_tokens,
        input_dim=512,
        fp8_dim=TOKEN_FP8_DIM,
        bf16_dim=TOKEN_BF16_DIM,
        scale_dim=TOKEN_SCALE_DIM,
        quant_block=QUANT_BLOCK_SIZE,
        cache_block_size=block_size,
        token_data_size=TOKEN_DATA_SIZE,
        block_stride=block_stride,
        fp8_max=FP8_MAX,
        n_quant_blocks=8,
    )
```
**EN:** This function implements `quantize_and_insert_k_cache` within the module. The docstring frames it as: Quantize K tensor and insert into paged K cache. Key calls include `stride`, `dim`.
**CN:** 该函数会实现 `quantize_and_insert_k_cache`，其作用域位于the module。 关键调用包括 `stride`, `dim`。

### `_dequantize_and_gather_k_kernel` function / `_dequantize_and_gather_k_kernel` 函数
```python
@triton.jit
def _dequantize_and_gather_k_kernel(
    out_ptr,
    out_stride0,
    out_stride1,
    k_cache_ptr,
    seq_lens_ptr,
    block_table_ptr,
    offset,
    gather_lens_ptr,
    # Constants
    max_blocks_per_seq: tl.constexpr,
    fp8_dim: tl.constexpr,  # 448
    bf16_dim: tl.constexpr,  # 64
    scale_dim: tl.constexpr,  # 8
    quant_block: tl.constexpr,  # 64 (quantization block size)
    cache_block_size: tl.constexpr,  # 64 or 128 (paged cache block size)
    token_data_size: tl.constexpr,  # 576 bytes per token data
    block_stride: tl.constexpr,  # total bytes per block (padded) int32
    output_dim: tl.constexpr,  # 512
    fp8_max: tl.constexpr,
    n_quant_blocks: tl.constexpr,  # 7 real blocks
):
    batch_idx = tl.program_id(0)
    worker_id = tl.program_id(1)
    num_workers = tl.num_programs(1)

    seq_len = tl.load(seq_lens_ptr + batch_idx)
    if gather_lens_ptr is not None:  # noqa: SIM108
        gather_len = tl.load(gather_lens_ptr + batch_idx)
    else:
        # Gather all tokens
        gather_len = seq_len
    start_pos = seq_len - gather_len

    for i in range(worker_id, gather_len, num_workers):
        # Calculate the actual token index in the sequence
        pos = start_pos + i

        # Calculate which block and position within block
        block_in_seq = pos // cache_block_size
        pos_in_block = pos % cache_block_size

        # Get physical block index from block table
        block_table_row_ptr = block_table_ptr + batch_idx * max_blocks_per_seq
        physical_block_idx = tl.load(block_table_row_ptr + block_in_seq)  # int32

        # int64: physical_block_idx * block_stride can exceed 2^31 with many
        # KV-cache blocks (e.g. >= 57K at block_stride ~37K).
        cache_block_ptr = k_cache_ptr + physical_block_idx.to(tl.int64) * block_stride
# ... omitted for brevity ...
                x_float = x_fp8.to(tl.float32)

                # Load and decode UE8M0 scale
                # UE8M0: scale = 2^(stored_value - 127)
                encoded_scale = tl.load(token_scale_ptr + qblock_idx)
                exponent = encoded_scale.to(tl.float32) - 127.0
                scale = tl.exp2(exponent)

                # Dequantize: bf16_value = fp8_value * scale
                x_dequant = x_float * scale

                # Store as bf16
                tl.store(output_row_ptr + offsets, x_dequant.to(tl.bfloat16), mask=mask)

        # ========== Copy BF16 portion directly ==========
        bf16_output_offset = fp8_dim  # After 448 elements in output

        # Read bf16 from cache
        bf16_cache_ptr = token_bf16_ptr.to(tl.pointer_type(tl.bfloat16))

        # Process in chunks of 16
        for j in tl.static_range(bf16_dim // 16):
            chunk_offsets = j * 16 + tl.arange(0, 16)
            bf16_vals = tl.load(bf16_cache_ptr + chunk_offsets)
            tl.store(output_row_ptr + bf16_output_offset + chunk_offsets, bf16_vals)
```
**EN:** This function implements `_dequantize_and_gather_k_kernel` within the module. Key calls include `program_id`, `num_programs`, `load`, `range`, `static_range`, `to`. The control flow contains 2 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_dequantize_and_gather_k_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `num_programs`, `load`, `range`, `static_range`, `to`。 控制流包含 2 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `dequantize_and_gather_k_cache_triton` function / `dequantize_and_gather_k_cache_triton` 函数
```python
def dequantize_and_gather_k_cache_triton(
    # [num_reqs, max_num_tokens, head_size]
    out: torch.Tensor,
    # [num_blocks, block_size, head_bytes]
    k_cache: torch.Tensor,
    # [num_reqs]
    seq_lens: torch.Tensor,
    # [num_reqs]
    gather_lens: torch.Tensor | None,
    # [num_reqs, max_blocks_per_seq]
    block_table: torch.Tensor,
    block_size: int,
    offset: int,
) -> None:
    TOKEN_FP8_DIM = 448
    TOKEN_BF16_DIM = 64
    TOKEN_SCALE_DIM = 8
    QUANT_BLOCK_SIZE = 64
    FP8_MAX = 448.0
    TOKEN_DATA_SIZE = TOKEN_FP8_DIM + TOKEN_BF16_DIM * 2

    num_reqs = seq_lens.shape[0]
    NUM_WORKERS = 128
    _dequantize_and_gather_k_kernel[(num_reqs, NUM_WORKERS)](
        out,
        out.stride(0),
        out.stride(1),
        k_cache,
        seq_lens,
        block_table,
        offset,
        gather_lens,
        max_blocks_per_seq=block_table.shape[-1],
        fp8_dim=TOKEN_FP8_DIM,
        bf16_dim=TOKEN_BF16_DIM,
        scale_dim=TOKEN_SCALE_DIM,
        quant_block=QUANT_BLOCK_SIZE,
        cache_block_size=block_size,
        token_data_size=TOKEN_DATA_SIZE,
        block_stride=k_cache.stride(0),
        output_dim=512,
        fp8_max=FP8_MAX,
        n_quant_blocks=7,
    )
```
**EN:** This function implements `dequantize_and_gather_k_cache_triton` within the module. Key calls include `stride`.
**CN:** 该函数会实现 `dequantize_and_gather_k_cache_triton`，其作用域位于the module。 关键调用包括 `stride`。

### `dequantize_and_gather_k_cache` function / `dequantize_and_gather_k_cache` 函数
```python
def dequantize_and_gather_k_cache(
    # [num_reqs, max_num_tokens, head_size]
    out: torch.Tensor,
    # [num_blocks, block_size, head_bytes]
    k_cache: torch.Tensor,
    # [num_reqs]
    seq_lens: torch.Tensor,
    # [num_reqs]
    gather_lens: torch.Tensor | None,
    # [num_reqs, max_blocks_per_seq]
    block_table: torch.Tensor,
    block_size: int,
    offset: int,
) -> None:
    if has_cutedsl():
        # lazily import, otherwise some tests fail due to CUDA driver init failure.
        from .dequant_gather_k_cutedsl import dequantize_and_gather_k_cache_cutedsl

        dequantize_and_gather_k_cache_cutedsl(
            out, k_cache, seq_lens, gather_lens, block_table, block_size, offset
        )
        return

    dequantize_and_gather_k_cache_triton(
        out, k_cache, seq_lens, gather_lens, block_table, block_size, offset
    )
```
**EN:** This function implements `dequantize_and_gather_k_cache` within the module. Key calls include `has_cutedsl`, `dequantize_and_gather_k_cache_triton`, `dequantize_and_gather_k_cache_cutedsl`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `dequantize_and_gather_k_cache`，其作用域位于the module。 关键调用包括 `has_cutedsl`, `dequantize_and_gather_k_cache_triton`, `dequantize_and_gather_k_cache_cutedsl`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `compute_global_topk_indices_and_lens` function / `compute_global_topk_indices_and_lens` 函数
```python
def compute_global_topk_indices_and_lens(
    topk_indices: torch.Tensor,
    token_to_req_indices: torch.Tensor,
    block_table: torch.Tensor,
    block_size: int,
    is_valid_token: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Map local topk indices to global KV cache slots and count valid entries.

    Fuses three operations into a single kernel:
    1. Block-table lookup (local index → global slot id)
    2. Valid-entry counting (topk_lens per token)
    3. Masking padding tokens to length 0
    """
    num_tokens = topk_indices.shape[0]
    global_topk_indices = torch.empty_like(topk_indices)
    topk_lens = torch.empty(num_tokens, dtype=torch.int32, device=topk_indices.device)
    _compute_global_topk_indices_and_lens_kernel[(num_tokens,)](
        global_topk_indices,
        global_topk_indices.stride(0),
        topk_lens,
        topk_indices,
        topk_indices.stride(0),
        topk_indices.shape[-1],
        token_to_req_indices,
        block_table,
        block_table.stride(0),
        block_size,
        is_valid_token,
        TRITON_BLOCK_SIZE=1024,
    )
    return global_topk_indices, topk_lens
```
**EN:** This function computes derived values within the module. The docstring frames it as: Map local topk indices to global KV cache slots and count valid entries. Key calls include `empty_like`, `empty`, `stride`.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `empty_like`, `empty`, `stride`。

### `_compute_global_topk_indices_and_lens_kernel` function / `_compute_global_topk_indices_and_lens_kernel` 函数
```python
@triton.jit
def _compute_global_topk_indices_and_lens_kernel(
    global_topk_indices_ptr,
    global_topk_indices_stride,
    topk_lens_ptr,
    topk_indices_ptr,
    topk_indices_stride,
    topk,
    token_to_req_indices_ptr,
    block_table_ptr,
    block_table_stride,
    block_size,
    is_valid_token_ptr,
    TRITON_BLOCK_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(0)
    is_valid_token = tl.load(is_valid_token_ptr + token_idx)
    req_idx = tl.load(token_to_req_indices_ptr + token_idx)

    count = tl.zeros((), dtype=tl.int32)
    for i in range(0, topk, TRITON_BLOCK_SIZE):
        offset = i + tl.arange(0, TRITON_BLOCK_SIZE)
        mask = offset < topk

        local_idx = tl.load(
            topk_indices_ptr + token_idx * topk_indices_stride + offset,
            mask=mask,
            other=-1,
        )
        is_valid = local_idx >= 0

        block_indices = local_idx // block_size
        block_numbers = tl.load(
            block_table_ptr + req_idx * block_table_stride + block_indices,
            mask=mask & is_valid,
        )
        block_offsets = local_idx % block_size

        slot_ids = block_numbers * block_size + block_offsets
        slot_ids = tl.where(is_valid, slot_ids, -1)
        tl.store(
            global_topk_indices_ptr + token_idx * global_topk_indices_stride + offset,
            slot_ids,
            mask=mask,
        )
        count += tl.sum(is_valid.to(tl.int32), axis=0)

    # Zero out length for padding tokens.
    tl.store(topk_lens_ptr + token_idx, tl.where(is_valid_token, count, 0))
```
**EN:** This function implements `_compute_global_topk_indices_and_lens_kernel` within the module. Key calls include `program_id`, `load`, `zeros`, `range`, `store`, `where`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_compute_global_topk_indices_and_lens_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `zeros`, `range`, `store`, `where`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
_SPARSE_PREFILL_TOPK_ALIGNMENT = 128
```
**EN:** Defines module-level constants or aliases such as `_SPARSE_PREFILL_TOPK_ALIGNMENT`, which are reused by later definitions.
**CN:** 定义 `_SPARSE_PREFILL_TOPK_ALIGNMENT` 等模块级常量或别名，供后续定义复用。

### `combine_topk_swa_indices` function / `combine_topk_swa_indices` 函数
```python
def combine_topk_swa_indices(
    topk_indices: torch.Tensor,
    query_start_loc: torch.Tensor,
    seq_lens: torch.Tensor,
    gather_lens: torch.Tensor,
    window_size: int,
    compress_ratio: int,
    topk: int,
    M: int,
    N: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    num_tokens = topk_indices.shape[0]
    num_reqs = seq_lens.shape[0]
    combined_topk = (
        (topk + window_size + _SPARSE_PREFILL_TOPK_ALIGNMENT - 1)
        // _SPARSE_PREFILL_TOPK_ALIGNMENT
        * _SPARSE_PREFILL_TOPK_ALIGNMENT
    )
    combined_indices = torch.full(
        (num_tokens, combined_topk),
        fill_value=-1,
        dtype=torch.int32,
        device=topk_indices.device,
    )
    combined_lens = torch.empty(
        num_tokens, dtype=torch.int32, device=topk_indices.device
    )

    NUM_WORKERS = 128
    _combine_topk_swa_indices_kernel[(num_reqs, NUM_WORKERS)](
        combined_indices,
        combined_indices.stride(0),
        combined_lens,
        topk_indices,
        topk_indices.stride(0),
        query_start_loc,
        seq_lens,
        gather_lens,
        M,
        N,
        TOP_K=topk,
        COMPRESS_RATIO=compress_ratio,
        WINDOW_SIZE=window_size,
        PADDED_TOP_K=triton.next_power_of_2(topk_indices.shape[-1]),
    )
    return combined_indices, combined_lens
```
**EN:** This function implements `combine_topk_swa_indices` within the module. Key calls include `full`, `empty`, `stride`, `next_power_of_2`.
**CN:** 该函数会实现 `combine_topk_swa_indices`，其作用域位于the module。 关键调用包括 `full`, `empty`, `stride`, `next_power_of_2`。

### `_combine_topk_swa_indices_kernel` function / `_combine_topk_swa_indices_kernel` 函数
```python
@triton.jit
def _combine_topk_swa_indices_kernel(
    combined_indices_ptr,
    combined_indices_stride,
    combined_lens_ptr,
    topk_indices_ptr,
    topk_indices_stride,
    query_start_loc_ptr,
    seq_lens_ptr,
    gather_lens_ptr,
    M,
    N,
    TOP_K: tl.constexpr,
    COMPRESS_RATIO: tl.constexpr,
    WINDOW_SIZE: tl.constexpr,
    PADDED_TOP_K: tl.constexpr,
):
    batch_idx = tl.program_id(0)
    worker_id = tl.program_id(1)
    num_workers = tl.num_programs(1)

    # query_start_loc is a global tensor; rebase to chunk-local offsets
    # by subtracting the chunk's starting value.
    base = tl.load(query_start_loc_ptr)
    query_start = tl.load(query_start_loc_ptr + batch_idx) - base
    query_end = tl.load(query_start_loc_ptr + batch_idx + 1) - base
    query_len = query_end - query_start
    seq_len = tl.load(seq_lens_ptr + batch_idx)
    gather_len = tl.load(gather_lens_ptr + batch_idx)
    start_pos = seq_len - query_len
    # The SWA portion of the gathered buffer starts from position
    # (seq_len - gather_len), not position 0. We need this offset
    # to correctly index into the gathered buffer.
    gather_start = seq_len - gather_len

    for token_idx in range(query_start + worker_id, query_end, num_workers):
        # topk_len is fully determined by the query token's absolute position:
        # both the C4A indexer and the C128A metadata builder emit
        # min((pos + 1) // compress_ratio, topk_tokens) valid entries.
        # Caller passes TOP_K=0 for SWA-only layers to zero this out.
        token_idx_in_query = token_idx - query_start
        pos = start_pos + token_idx_in_query
        topk_len = tl.minimum((pos + 1) // COMPRESS_RATIO, TOP_K)
        swa_len = tl.minimum(pos + 1, WINDOW_SIZE)

        offset = tl.arange(0, PADDED_TOP_K)
        mask = offset < topk_len
        topk_indices = tl.load(
            topk_indices_ptr + token_idx * topk_indices_stride + offset,
            mask=mask,
        )
        tl.store(
            combined_indices_ptr + token_idx * combined_indices_stride + offset,
            topk_indices + M * batch_idx,
            mask=mask,
        )
        offset = tl.arange(0, WINDOW_SIZE)
        # Index into gathered buffer: N + (position - gather_start)
        # For positions [pos - swa_len + 1, pos], the buffer indices are:
        # [N + pos - swa_len + 1 - gather_start, N + pos - gather_start]
        tl.store(
            combined_indices_ptr
            + token_idx * combined_indices_stride
            + topk_len
            + offset,
            M * batch_idx + N + offset + pos - swa_len + 1 - gather_start,
            mask=offset < swa_len,
        )

        combined_len = topk_len + swa_len
        tl.store(combined_lens_ptr + token_idx, combined_len)
```
**EN:** This function implements `_combine_topk_swa_indices_kernel` within the module. Key calls include `program_id`, `num_programs`, `load`, `range`, `minimum`, `arange`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_combine_topk_swa_indices_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `num_programs`, `load`, `range`, `minimum`, `arange`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `quantize_and_insert_k_kernel`: top-level helper or orchestration entry point. / `quantize_and_insert_k_kernel`：顶层辅助函数或编排入口。
- `quantize_and_insert_k_cache`: top-level helper or orchestration entry point. / `quantize_and_insert_k_cache`：顶层辅助函数或编排入口。
- `_dequantize_and_gather_k_kernel`: top-level helper or orchestration entry point. / `_dequantize_and_gather_k_kernel`：顶层辅助函数或编排入口。
- `dequantize_and_gather_k_cache_triton`: top-level helper or orchestration entry point. / `dequantize_and_gather_k_cache_triton`：顶层辅助函数或编排入口。
- `dequantize_and_gather_k_cache`: top-level helper or orchestration entry point. / `dequantize_and_gather_k_cache`：顶层辅助函数或编排入口。
- `compute_global_topk_indices_and_lens`: top-level helper or orchestration entry point. / `compute_global_topk_indices_and_lens`：顶层辅助函数或编排入口。
- `_compute_global_topk_indices_and_lens_kernel`: top-level helper or orchestration entry point. / `_compute_global_topk_indices_and_lens_kernel`：顶层辅助函数或编排入口。
- `combine_topk_swa_indices`: top-level helper or orchestration entry point. / `combine_topk_swa_indices`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.triton_utils`, `vllm.utils.import_utils`, `.dequant_gather_k_cutedsl`
