# triton_unified_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/triton_unified_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_cast_kv_tile`, `_load_q_td`, `_load_kv_tile_td` for the V1 `attention/ops` subsystem. / 为 V1 的 `attention/ops` 子系统实现 `_cast_kv_tile`, `_load_q_td`, `_load_kv_tile_td`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from typing import Any

import torch

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.v1.attention.ops.triton_attention_helpers import (
    apply_alibi_to_score,
    apply_softcap,
    cdiv_fn,
    compute_kv_seq_mask,
    compute_tile_loop_bounds,
    find_seq_idx,
    init_softmax_M,
    load_qq_bias_tile,
    resolve_seq_and_query_len,
    softmax_step,
    store_segm_reduce_scalars,
)
from vllm.v1.kv_cache_interface import KVQuantMode

logger = init_logger(__name__)
is_batch_invariant = envs.VLLM_BATCH_INVARIANT
float8_info = torch.finfo(current_platform.fp8_dtype())
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `is_batch_invariant`, `float8_info`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `is_batch_invariant`, `float8_info`。

### `_cast_kv_tile` function / `_cast_kv_tile` 函数
```python
@triton.jit
def _cast_kv_tile(data, Q, tensor_scale, KV_QUANT_MODE: tl.constexpr):
    """Cast a loaded KV tile to Q's dtype, dequantizing if needed.

    Modes handled inside the core kernel:

    - ``KV_QUANT_MODE == 0`` (NONE) and ``2`` (INT8 per-token-head) and
      ``3`` (FP8 per-token-head): plain cast.  Per-token-head modes apply
      their scales separately on S/P inside the loop.
    - ``KV_QUANT_MODE == 1`` (FP8 per-tensor): dequantize using the
      tensor-wide scale.
    """
    if KV_QUANT_MODE == 1:
        if Q.dtype.is_fp8():
            return data.to(Q.dtype)
        return (data.to(tl.float32) * tl.load(tensor_scale)).to(Q.dtype)
    return data.to(Q.dtype)
```
**EN:** This function implements `_cast_kv_tile` within the module. The docstring frames it as: Cast a loaded KV tile to Q's dtype, dequantizing if needed. Key calls include `to`, `is_fp8`, `load`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_cast_kv_tile`，其作用域位于the module。 关键调用包括 `to`, `is_fp8`, `load`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_load_q_td` function / `_load_q_td` 函数
```python
@triton.jit
def _load_q_td(
    query_ptr,
    q_block_local_len,
    query_stride_0: tl.int64,
    query_stride_1: tl.int64,
    cur_batch_in_all_start_index,
    q_block_local_idx,
    kv_head_idx,
    num_queries_per_kv: tl.constexpr,
    BLOCK_Q: tl.constexpr,
    BLOCK_M: tl.constexpr,
    HEAD_SIZE: tl.constexpr,
    HEAD_SIZE_PADDED: tl.constexpr,
):
    """Load Q via a 2D tensor descriptor.

    Caller guarantees (via the wrapper's ``use_td_qo`` gate):
      * ``HEAD_SIZE == HEAD_SIZE_PADDED`` (head_size is a power of 2),
      * ``num_queries_per_kv`` is a power of 2,
      * the ``num_queries_per_kv`` heads of the current KV group are
        contiguous in memory (``query_stride_1 == HEAD_SIZE``, which is
        the default vLLM query layout).

    Under those preconditions the inner two axes are flattened into one
    row of size ``num_queries_per_kv * HEAD_SIZE`` with stride 1, which
    avoids the non-power-of-2 ``block_shape`` error from the Triton
    tensor-descriptor validator.  Returns (BLOCK_M, HEAD_SIZE_PADDED).
    """
    q_base = (
        query_ptr
        + (cur_batch_in_all_start_index + q_block_local_idx * BLOCK_Q) * query_stride_0
        + (kv_head_idx * num_queries_per_kv) * query_stride_1
    )
    q_desc = tl.make_tensor_descriptor(
        base=q_base,
        shape=(q_block_local_len, num_queries_per_kv * HEAD_SIZE),
        strides=(query_stride_0, 1),
        block_shape=(BLOCK_Q, num_queries_per_kv * HEAD_SIZE_PADDED),
    )
    return q_desc.load([0, 0]).reshape(BLOCK_M, HEAD_SIZE_PADDED)
```
**EN:** This function implements `_load_q_td` within the module. The docstring frames it as: Load Q via a 2D tensor descriptor. Key calls include `make_tensor_descriptor`, `reshape`, `load`.
**CN:** 该函数会实现 `_load_q_td`，其作用域位于the module。 关键调用包括 `make_tensor_descriptor`, `reshape`, `load`。

### `_load_kv_tile_td` function / `_load_kv_tile_td` 函数
```python
@triton.jit
def _load_kv_tile_td(
    cache_ptr,
    physical_block_idx_scalar,
    kv_head_idx,
    offset_in_block,
    stride_cache_0: tl.int64,
    stride_cache_1: tl.int64,
    stride_cache_2: tl.int64,
    stride_cache_3: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
    TILE_SIZE: tl.constexpr,
    HEAD_SIZE: tl.constexpr,
    HEAD_SIZE_PADDED: tl.constexpr,
):
    """Load a KV cache tile via tensor descriptor.

    Returns shape (TILE_SIZE, HEAD_SIZE_PADDED). Caller transposes for K.
    Tensor descriptors zero-pad reads beyond the shape boundary, so
    ``HEAD_SIZE_PADDED > HEAD_SIZE`` is handled correctly.
    """
    base = (
        cache_ptr
        + physical_block_idx_scalar * stride_cache_0
        + kv_head_idx * stride_cache_2
    )
    desc = tl.make_tensor_descriptor(
        base=base,
        shape=(BLOCK_SIZE, HEAD_SIZE),
        strides=(stride_cache_1, stride_cache_3),
        block_shape=(TILE_SIZE, HEAD_SIZE_PADDED),
    )
    return desc.load([offset_in_block, 0])
```
**EN:** This function implements `_load_kv_tile_td` within the module. The docstring frames it as: Load a KV cache tile via tensor descriptor. Key calls include `make_tensor_descriptor`, `load`.
**CN:** 该函数会实现 `_load_kv_tile_td`，其作用域位于the module。 关键调用包括 `make_tensor_descriptor`, `load`。

### `_store_output_td` function / `_store_output_td` 函数
```python
@triton.jit
def _store_output_td(
    base_ptr,
    acc,
    q_block_local_len,
    stride_token: tl.int64,
    stride_head: tl.int64,
    num_queries_per_kv: tl.constexpr,
    BLOCK_Q: tl.constexpr,
    HEAD_SIZE: tl.constexpr,
    HEAD_SIZE_PADDED: tl.constexpr,
):
    """Store an output tile via a tensor descriptor.

    The 2D and 3D epilogues differ only in ``base_ptr`` and the
    ``(stride_token, stride_head)`` pair: 2D writes directly to the
    flat output buffer, 3D writes to a single per-segment slice of
    ``segm_output_ptr``.  Descriptor shape / block_shape / reshape
    are the same in both modes, so share one helper.
    """
    acc = acc.to(base_ptr.dtype.element_ty)
    output_desc = tl.make_tensor_descriptor(
        base=base_ptr,
        shape=(q_block_local_len, num_queries_per_kv, HEAD_SIZE),
        strides=(stride_token, stride_head, 1),
        block_shape=(BLOCK_Q, num_queries_per_kv, HEAD_SIZE_PADDED),
    )
    output_desc.store(
        [0, 0, 0],
        acc.reshape(BLOCK_Q, num_queries_per_kv, HEAD_SIZE_PADDED),
    )
```
**EN:** This function implements `_store_output_td` within the module. The docstring frames it as: Store an output tile via a tensor descriptor. Key calls include `to`, `make_tensor_descriptor`, `store`, `reshape`.
**CN:** 该函数会实现 `_store_output_td`，其作用域位于the module。 关键调用包括 `to`, `make_tensor_descriptor`, `store`, `reshape`。

### `kernel_unified_attention` function / `kernel_unified_attention` 函数
```python
@triton.jit
def kernel_unified_attention(
    # Output destination for the 2D path.  In 3D mode per-segment partials
    # go to the ``segm_*`` tensors (see bottom of signature) and
    # ``output_ptr`` is unused (callers may pass any non-null pointer).
    output_ptr,
    # Inputs
    query_ptr,
    key_cache_ptr,
    value_cache_ptr,
    sink_ptr,
    block_tables_ptr,
    seq_lens_ptr,
    alibi_slopes_ptr,
    qq_bias_ptr,
    # Scalars
    scale,
    k_scale,
    v_scale,
    out_scale,
    softcap,
    num_query_heads: tl.constexpr,  # int
    num_queries_per_kv: tl.constexpr,  # int
    block_table_stride: tl.int64,  # int
    query_stride_0: tl.int64,  # int
    query_stride_1: tl.int64,  # int, should be equal to head_size
    output_stride_0: tl.int64,  # int
    output_stride_1: tl.int64,  # int, should be equal to head_size
    qq_bias_stride_0: tl.int64,  # int
    BLOCK_SIZE: tl.constexpr,  # int
    TILE_SIZE: tl.constexpr,  # int must be power of 2
    HEAD_SIZE: tl.constexpr,  # int
    HEAD_SIZE_PADDED: tl.constexpr,  # int, must be power of 2
    USE_ALIBI_SLOPES: tl.constexpr,  # bool
    USE_ALIBI_SQRT: tl.constexpr,  # bool
    USE_QQ_BIAS: tl.constexpr,  # bool
    USE_SOFTCAP: tl.constexpr,  # bool
    USE_SINKS: tl.constexpr,  # bool
    SLIDING_WINDOW: tl.constexpr,  # int
    USE_MM_PREFIX: tl.constexpr,  # bool
    MAX_MM_RANGES: tl.constexpr,  # int
    mm_prefix_range_ptr,
    stride_k_cache_0: tl.int64,  # int
    stride_k_cache_1: tl.int64,  # int
    stride_k_cache_2: tl.int64,  # int
    stride_k_cache_3: tl.constexpr,  # int
    stride_v_cache_0: tl.int64,  # int
    stride_v_cache_1: tl.int64,  # int
    stride_v_cache_2: tl.int64,  # int
    stride_v_cache_3: tl.constexpr,  # int
# ... omitted for brevity ...
                * output_stride_0
                + (kv_head_idx * num_queries_per_kv) * output_stride_1
            )
            _store_output_td(
                output_base,
                acc,
                q_block_local_len,
                output_stride_0,
                output_stride_1,
                num_queries_per_kv,
                BLOCK_Q,
                HEAD_SIZE,
                HEAD_SIZE_PADDED,
            )
        else:
            output_offset = (
                query_offset_0[:, None] * output_stride_0
                + query_offset_1[:, None] * output_stride_1
                + offs_d[None, :]
            )
            tl.store(
                output_ptr + output_offset,
                acc,
                mask=dim_mask[None, :] & query_mask_0[:, None] & query_mask_1[:, None],
            )
```
**EN:** This function implements `kernel_unified_attention` within the module. Key calls include `program_id`, `resolve_seq_and_query_len`, `minimum`, `arange`, `to`, `init_softmax_M`. The control flow contains 20 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `kernel_unified_attention`，其作用域位于the module。 关键调用包括 `program_id`, `resolve_seq_and_query_len`, `minimum`, `arange`, `to`, `init_softmax_M`。 控制流包含 20 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `reduce_segments` function / `reduce_segments` 函数
```python
@triton.jit
def reduce_segments(
    output_ptr,  # [num_tokens, num_query_heads, head_size]
    segm_output_ptr,
    # [num_tokens, num_query_heads, max_num_segments, head_size]
    segm_max_ptr,  # [num_tokens, num_query_heads, max_num_segments]
    segm_expsum_ptr,  # [num_tokens, num_query_heads, max_num_segments]
    seq_lens_ptr,  # [num_seqs]
    num_seqs,  # int
    num_query_heads: tl.constexpr,  # int
    out_scale_inv,  # float32
    output_stride_0: tl.int64,  # int
    output_stride_1: tl.int64,  # int, should be equal to head_size
    block_table_stride: tl.int64,  # int
    TILE_SIZE: tl.constexpr,  # int
    HEAD_SIZE: tl.constexpr,  # int, must be power of 2
    HEAD_SIZE_PADDED: tl.constexpr,  # int, must be power of 2
    query_start_len_ptr,  # [num_seqs+1]
    BLOCK_Q: tl.constexpr,  # int
    NUM_SEGMENTS_PER_SEQ: tl.constexpr,  # int
    USE_FP8: tl.constexpr,  # bool
    FP8_MIN: tl.constexpr = float8_info.min,
    FP8_MAX: tl.constexpr = float8_info.max,
):
    query_token_idx = tl.program_id(0)
    query_head_idx = tl.program_id(1)

    seq_idx = find_seq_idx(
        query_start_len_ptr, query_token_idx, num_seqs, BLOCK_Q, False
    )

    # sequence len for this particular sequence
    seq_len = tl.load(seq_lens_ptr + seq_idx)

    # number of segments for this particular sequence
    num_segments = NUM_SEGMENTS_PER_SEQ
    tiles_per_segment = cdiv_fn(seq_len, num_segments * TILE_SIZE)

    # create masks for subsequent loads
    act_num_segments = cdiv_fn(seq_len, tiles_per_segment * TILE_SIZE)
    segm_mask = tl.arange(0, NUM_SEGMENTS_PER_SEQ) < tl.full(
        [NUM_SEGMENTS_PER_SEQ], act_num_segments, dtype=tl.int32
    )
    dim_mask = tl.where(tl.arange(0, HEAD_SIZE_PADDED) < HEAD_SIZE, 1, 0).to(tl.int1)

    # load segment maxima
    segm_offset = (
        query_token_idx.to(tl.int64) * (num_query_heads * NUM_SEGMENTS_PER_SEQ)
        + query_head_idx * NUM_SEGMENTS_PER_SEQ
        + tl.arange(0, NUM_SEGMENTS_PER_SEQ)
    )
    segm_max = tl.load(segm_max_ptr + segm_offset, mask=segm_mask, other=float("-inf"))
    overall_max = tl.max(segm_max)

    # load and rescale segment exp sums
    segm_expsum = tl.load(segm_expsum_ptr + segm_offset, mask=segm_mask, other=0.0)
    segm_expsum = segm_expsum * tl.exp(segm_max - overall_max)
    overall_expsum = tl.sum(segm_expsum)

    # load, rescale, and add segment attention outputs
    segm_output_offset = (
        query_token_idx.to(tl.int64)
        * (num_query_heads * NUM_SEGMENTS_PER_SEQ * HEAD_SIZE_PADDED)
        + query_head_idx * (NUM_SEGMENTS_PER_SEQ * HEAD_SIZE_PADDED)
        + tl.arange(0, NUM_SEGMENTS_PER_SEQ)[:, None] * HEAD_SIZE_PADDED
        + tl.arange(0, HEAD_SIZE_PADDED)[None, :]
    )
    segm_output = tl.load(
        segm_output_ptr + segm_output_offset,
        mask=segm_mask[:, None] & dim_mask[None, :],
        other=0.0,
    )
    segm_output *= tl.exp(segm_max - overall_max)[:, None]
    acc_sum = tl.sum(segm_output, axis=0)
    # safely divide by overall_expsum, returning 0.0 if overall_expsum is 0
    acc = tl.where(overall_expsum == 0.0, 0.0, acc_sum / overall_expsum)

    if USE_FP8:
        acc = acc * tl.load(out_scale_inv)
        acc = tl.clamp(acc, FP8_MIN, FP8_MAX)

    # write result
    output_offset = (
        query_token_idx * output_stride_0
        + query_head_idx * output_stride_1
        + tl.arange(0, HEAD_SIZE_PADDED)
    )
    tl.store(output_ptr + output_offset, acc, mask=dim_mask)
```
**EN:** This function implements `reduce_segments` within the module. Key calls include `program_id`, `find_seq_idx`, `load`, `cdiv_fn`, `to`, `max`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `reduce_segments`，其作用域位于the module。 关键调用包括 `program_id`, `find_seq_idx`, `load`, `cdiv_fn`, `to`, `max`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_is_gemma3_attention` function / `_is_gemma3_attention` 函数
```python
def _is_gemma3_attention(head_size: int, sliding_window: int) -> bool:
    """Detect Gemma3 models via unique (head_size, sliding_window) signature.

    Gemma3 models are the only ones using sliding_window=1024 with
    head_size 128 (27B) or 256 (1B, 4B, 12B). Other SWA models use
    different window sizes (Mistral=4096, Phi-3=2047).
    """
    return sliding_window == 1024 and head_size in (128, 256)
```
**EN:** This function implements `_is_gemma3_attention` within the module. The docstring frames it as: Detect Gemma3 models via unique (head_size, sliding_window) signature.
**CN:** 该函数会实现 `_is_gemma3_attention`，其作用域位于the module。

### `_get_tile_size` function / `_get_tile_size` 函数
```python
def _get_tile_size(
    head_size: int,
    sliding_window: int,
    element_size: int,
    is_prefill: bool,
) -> int:
    """Select tile size with Gemma3-specific optimization."""
    if _is_gemma3_attention(head_size, sliding_window):
        # Gemma3: use 32 for decode (default is 16)
        return 32

    # Default behavior
    if is_prefill:
        return 32
    # Note: tile size must be at least 32 for fp8 (element_size == 1).
    return 16 if element_size >= 2 else 32
```
**EN:** This function implements `_get_tile_size` within the module. The docstring frames it as: Select tile size with Gemma3-specific optimization. Key calls include `_is_gemma3_attention`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_get_tile_size`，其作用域位于the module。 关键调用包括 `_is_gemma3_attention`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `unified_attention` function / `unified_attention` 函数
```python
def unified_attention(
    q,
    k,
    v,
    out,
    cu_seqlens_q,
    max_seqlen_q,
    seqused_k,
    max_seqlen_k,
    softmax_scale,
    causal,
    window_size,
    block_table,
    softcap,
    q_descale,
    k_descale,
    v_descale,
    seq_threshold_3D=None,
    num_par_softmax_segments=None,
    softmax_segm_output=None,
    softmax_segm_max=None,
    softmax_segm_expsum=None,
    alibi_slopes=None,
    output_scale=None,
    qq_bias=None,
    # Optional tensor for sinks
    sinks=None,
    # Optional tensor for prefix lengths (PrefixLM support)
    mm_prefix_range=None,
    use_alibi_sqrt=False,
    # KV cache quantization mode and per-token-head scale caches.
    kv_quant_mode: KVQuantMode = KVQuantMode.NONE,
    k_scale_cache=None,  # [num_blocks, block_size, num_kv_heads] float32
    v_scale_cache=None,  # [num_blocks, block_size, num_kv_heads] float32
    # Chunked attention: restrict attention to aligned blocks with lookback.
    chunk_lookback=-1,
    # Tensor-descriptor mode: use ``tl.make_tensor_descriptor`` for Q/K/V
    # loads and output stores.  Enables HW 2D block reads on Intel Xe2/Xe3.
    # The non-TD branch is dead-code-eliminated at Triton compile time so
    # disabling this flag costs nothing.
    use_td: bool = False,
):
    assert causal, "Only causal attention is supported"
    assert q_descale is None, "Q scales not supported"

    if sinks is not None:
        assert sinks.shape[0] == q.shape[1], "Sinks must be num_query_heads size"

    use_per_token_head_scales = kv_quant_mode in (
        KVQuantMode.INT8_PER_TOKEN_HEAD,
# ... omitted for brevity ...
        USE_TD=use_td,
        USE_TD_QO=use_td_qo,
    )

    if use_3d:
        reduce_segments[(q.shape[0], num_query_heads)](
            output_ptr=out,
            segm_output_ptr=softmax_segm_output,
            segm_max_ptr=softmax_segm_max,
            segm_expsum_ptr=softmax_segm_expsum,
            seq_lens_ptr=seqused_k,
            num_seqs=num_seqs,
            num_query_heads=num_query_heads,
            out_scale_inv=1 / output_scale if output_scale is not None else 1.0,
            output_stride_0=out.stride(0),
            output_stride_1=out.stride(1),
            block_table_stride=block_table.stride(0),
            TILE_SIZE=TILE_SIZE_DECODE,
            HEAD_SIZE=head_size,
            HEAD_SIZE_PADDED=head_size_padded,
            query_start_len_ptr=cu_seqlens_q,
            BLOCK_Q=BLOCK_Q,
            NUM_SEGMENTS_PER_SEQ=num_par_softmax_segments,
            USE_FP8=output_scale is not None,
        )
```
**EN:** This function implements `unified_attention` within the module. Key calls include `len`, `_get_tile_size`, `next_power_of_2`, `element_size`, `min`, `stride`. The control flow contains 20 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `unified_attention`，其作用域位于the module。 关键调用包括 `len`, `_get_tile_size`, `next_power_of_2`, `element_size`, `min`, `stride`。 控制流包含 20 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `_cast_kv_tile`: top-level helper or orchestration entry point. / `_cast_kv_tile`：顶层辅助函数或编排入口。
- `_load_q_td`: top-level helper or orchestration entry point. / `_load_q_td`：顶层辅助函数或编排入口。
- `_load_kv_tile_td`: top-level helper or orchestration entry point. / `_load_kv_tile_td`：顶层辅助函数或编排入口。
- `_store_output_td`: top-level helper or orchestration entry point. / `_store_output_td`：顶层辅助函数或编排入口。
- `kernel_unified_attention`: top-level helper or orchestration entry point. / `kernel_unified_attention`：顶层辅助函数或编排入口。
- `reduce_segments`: top-level helper or orchestration entry point. / `reduce_segments`：顶层辅助函数或编排入口。
- `_is_gemma3_attention`: top-level helper or orchestration entry point. / `_is_gemma3_attention`：顶层辅助函数或编排入口。
- `_get_tile_size`: top-level helper or orchestration entry point. / `_get_tile_size`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.logger`, `vllm.platforms`, `vllm.triton_utils`, `vllm.v1.attention.ops.triton_attention_helpers`, `vllm.v1.kv_cache_interface`
