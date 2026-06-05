# triton_attention_helpers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/triton_attention_helpers.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Shared ``@triton.jit`` helpers used by the unified attention kernel and ``reduce_segments``. / 该模块位于 `attention/ops` 子系统，主要围绕 `cdiv_fn`, `apply_softcap`, `resolve_seq_and_query_len` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Shared ``@triton.jit`` helpers used by the unified attention kernel
and ``reduce_segments``.

These are plain attention-loop helpers — mask building, ALiBi / QQ-bias
score post-processing, online-softmax bookkeeping, tile-loop bounds,
sequence lookup — extracted so the 2D and 3D paths of the unified
kernel (and any future consumer) share a single implementation.
"""

from __future__ import annotations

from vllm.triton_utils import tl, triton
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `cdiv_fn` function / `cdiv_fn` 函数
```python
@triton.jit
def cdiv_fn(x, y):
    """Ceiling division.  Kept as a helper to keep kernel bodies terse."""
    return (x + y - 1) // y
```
**EN:** This function implements `cdiv_fn` within the module. The docstring frames it as: Ceiling division.
**CN:** 该函数会实现 `cdiv_fn`，其作用域位于the module。

### `apply_softcap` function / `apply_softcap` 函数
```python
@triton.jit
def apply_softcap(S, x):
    """Softcap (aka tanh-style clamp) used to bound attention scores.

    ``x * tanh(S / x)`` rewritten to avoid a direct ``tanh`` call.
    """
    Sdiv = S / x
    p1 = tl.exp(Sdiv)
    p2 = tl.exp(-Sdiv)
    return x * (p1 - p2) / (p1 + p2)
```
**EN:** This function implements `apply_softcap` within the module. The docstring frames it as: Softcap (aka tanh-style clamp) used to bound attention scores. Key calls include `exp`.
**CN:** 该函数会实现 `apply_softcap`，其作用域位于the module。 关键调用包括 `exp`。

### `resolve_seq_and_query_len` function / `resolve_seq_and_query_len` 函数
```python
@triton.jit
def resolve_seq_and_query_len(
    query_start_len_ptr,
    seq_lens_ptr,
    q_block_global_idx,
    num_seqs,
    BLOCK_Q: tl.constexpr,
):
    """Resolve the (sequence, q-block-within-sequence) pair and load the
    per-sequence lengths.

    Shared across every attention kernel — the ``q_block_global_idx``
    program id indexes into the flattened ``(seq, q_block_in_seq)``
    space, and a binary search over ``query_start_len_ptr`` recovers
    the (seq, local-q-block) pair.

    Returns ``(seq_idx, q_block_local_idx, cur_batch_in_all_start_index,
    cur_batch_query_len, seq_len)``.  Callers must still early-return
    when ``q_block_local_idx * BLOCK_Q >= cur_batch_query_len`` (Triton
    helpers cannot return from the caller).
    """
    # find_seq_idx is defined below; forward use is fine inside @triton.jit.
    seq_idx = find_seq_idx(
        query_start_len_ptr, q_block_global_idx, num_seqs, BLOCK_Q, True
    )
    q_block_start_idx = tl.load(query_start_len_ptr + seq_idx) // BLOCK_Q + seq_idx
    q_block_local_idx = q_block_global_idx - q_block_start_idx
    cur_start = tl.load(query_start_len_ptr + seq_idx)
    cur_stop = tl.load(query_start_len_ptr + seq_idx + 1)
    cur_batch_query_len = cur_stop - cur_start
    seq_len = tl.load(seq_lens_ptr + seq_idx)
    return seq_idx, q_block_local_idx, cur_start, cur_batch_query_len, seq_len
```
**EN:** This function resolves configuration or runtime choices within the module. The docstring frames it as: Resolve the (sequence, q-block-within-sequence) pair and load the per-sequence lengths. Key calls include `find_seq_idx`, `load`.
**CN:** 该函数会解析配置或运行期选择，其作用域位于the module。 关键调用包括 `find_seq_idx`, `load`。

### `find_seq_idx` function / `find_seq_idx` 函数
```python
@triton.jit
def find_seq_idx(
    query_start_len_ptr,
    target_idx,
    num_seqs,
    BLOCK_Q: tl.constexpr,
    use_q_block_mode: tl.constexpr,
):
    """Binary search over the cumulative query-length prefix.

    When ``use_q_block_mode`` is True, the prefix values are reshaped
    into units of ``BLOCK_Q`` plus one entry per boundary — matching
    the q-block grid laid out by the attention kernels.  When False
    we search the plain cumulative-length prefix (used by
    ``reduce_segments`` which iterates over raw query tokens).
    """
    left: tl.int32 = 0
    right = num_seqs
    while left < right:
        mid = (left + right) // 2
        val = tl.load(query_start_len_ptr + mid)
        mid_val = val // BLOCK_Q + mid if use_q_block_mode else val

        if mid_val <= target_idx:
            left = mid + 1
        else:
            right = mid

    return left - 1
```
**EN:** This function implements `find_seq_idx` within the module. The docstring frames it as: Binary search over the cumulative query-length prefix. Key calls include `load`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `find_seq_idx`，其作用域位于the module。 关键调用包括 `load`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `init_softmax_M` function / `init_softmax_M` 函数
```python
@triton.jit
def init_softmax_M(
    sink_ptr,
    query_offset_1,
    query_mask_1,
    segm_idx_or_0,
    BLOCK_M: tl.constexpr,
    USE_SINKS: tl.constexpr,
    IS_3D: tl.constexpr,
):
    """Initial row-max ``M`` for the online softmax.

    Without sinks: ``-inf``.  With sinks: load the per-head sink bias
    once.  In 3D mode only segment 0 loads — ``reduce_segments`` adds
    the sink contribution exactly once across segments, so other
    segments must start from ``-inf``.

    ``segm_idx_or_0`` is the 3D segment index or 0 for 2D (caller
    passes ``0`` when ``IS_3D`` is False).
    """
    M = tl.full([BLOCK_M], float("-inf"), dtype=tl.float32)
    if USE_SINKS:
        load_sinks = (not IS_3D) or (segm_idx_or_0 == 0)
        if load_sinks:
            M = tl.load(
                sink_ptr + query_offset_1,
                mask=query_mask_1,
                other=float("-inf"),
            ).to(tl.float32)
    return M
```
**EN:** This function implements `init_softmax_M` within the module. The docstring frames it as: Initial row-max ``M`` for the online softmax. Key calls include `full`, `float`, `to`, `load`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `init_softmax_M`，其作用域位于the module。 关键调用包括 `full`, `float`, `to`, `load`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `compute_tile_loop_bounds` function / `compute_tile_loop_bounds` 函数
```python
@triton.jit
def compute_tile_loop_bounds(
    context_len,
    seq_len,
    cur_batch_query_len,
    q_block_local_idx,
    segm_idx_or_0,
    tiles_per_segment_or_0,
    TILE_SIZE: tl.constexpr,
    BLOCK_M: tl.constexpr,
    BLOCK_Q: tl.constexpr,
    num_queries_per_kv: tl.constexpr,
    SLIDING_WINDOW: tl.constexpr,
    USE_MM_PREFIX: tl.constexpr,
    IS_3D: tl.constexpr,
    CHUNK_LOOKBACK: tl.constexpr = -1,
    CHUNK_SIZE: tl.constexpr = -1,
):
    """Compute the tile-loop bounds ``(loop_lo, loop_hi)`` and the
    derived ``max_seq_prefix_len`` used for per-tile masking.

    Combines three concerns into one helper:

    1. Longest prefix spanned by any query token in this q-block.
       Clamped to ``seq_len`` (causal) or extended to it when
       mm_prefix is active (bidirectional ranges can reach past the
       causal prefix).
    2. Sliding-window pruning: narrows ``[tile_start, tile_end)`` to
       only tiles that can contain an allowed key under SWA.
    3. 3D scoping: when ``IS_3D`` is True, further narrows to the
       segment's slice via ``(segm_idx * tiles_per_segment,
       (segm_idx + 1) * tiles_per_segment)``.
    """
    # compute the length of the longest sequence prefix spanned by any
    # query token in the current q_block (q_block_local_idx)
    max_seq_prefix_len = (
        context_len
        + q_block_local_idx * BLOCK_Q
        + (BLOCK_M - 1) // num_queries_per_kv
        + 1
    )
    if USE_MM_PREFIX:
        # image bidirectional attention ranges require a full range
        # including q_block padding to make sure doc mask is correct
        max_seq_prefix_len = tl.maximum(max_seq_prefix_len, seq_len)
    else:
        max_seq_prefix_len = tl.minimum(max_seq_prefix_len, seq_len)

    num_tiles = cdiv_fn(max_seq_prefix_len, TILE_SIZE)

    # ---- Sliding-window tile pruning --------------------
    # Default: keep previous global behavior
    tile_start = 0
    tile_end = num_tiles
    # TODO(Isotr0py): sliding window pruning with image bidirectional mask
    if SLIDING_WINDOW > 0 and not USE_MM_PREFIX:
        # Query rows covered by this Q-block
        qpos_lo = q_block_local_idx * BLOCK_Q
        qpos_hi = tl.minimum(
            qpos_lo + (BLOCK_M - 1) // num_queries_per_kv,
            cur_batch_query_len - 1,
        )
        # For sliding window, each query position q can only attend to
        # keys in the range [q_abs - SLIDING_WINDOW + 1, q_abs]
        # where q_abs = context_len + q
        # The union of allowed key positions for this Q-block is:
        # [context_len + qpos_lo - SLIDING_WINDOW + 1, context_len + qpos_hi]
        q_abs = context_len + qpos_lo
        if CHUNK_LOOKBACK > -1:
            # Chunked attention: align lower bound to the start of the
            # lookback'th previous chunk.
            first_allowed_key = ((q_abs // CHUNK_SIZE) - CHUNK_LOOKBACK) * CHUNK_SIZE
        else:
            first_allowed_key = q_abs - SLIDING_WINDOW + 1
        last_allowed_key = context_len + qpos_hi
        # Convert to tile indices and clamp
        tile_start = tl.maximum(0, first_allowed_key // TILE_SIZE)
        tile_end = tl.minimum((last_allowed_key // TILE_SIZE) + 1, num_tiles)

    if IS_3D:
        loop_lo = max(segm_idx_or_0 * tiles_per_segment_or_0, tile_start)
        loop_hi = min((segm_idx_or_0 + 1) * tiles_per_segment_or_0, tile_end)
    else:
        loop_lo = tile_start
        loop_hi = tile_end

    return loop_lo, loop_hi, max_seq_prefix_len
```
**EN:** This function computes derived values within the module. The docstring frames it as: Compute the tile-loop bounds ``(loop_lo, loop_hi)`` and the derived ``max_seq_prefix_len`` used for per-tile masking. Key calls include `cdiv_fn`, `maximum`, `minimum`, `max`, `min`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `cdiv_fn`, `maximum`, `minimum`, `max`, `min`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `store_segm_reduce_scalars` function / `store_segm_reduce_scalars` 函数
```python
@triton.jit
def store_segm_reduce_scalars(
    segm_max_ptr,
    segm_expsum_ptr,
    query_offset_0,
    query_offset_1,
    segm_idx,
    M,
    L,
    query_mask_0,
    query_mask_1,
    num_query_heads: tl.constexpr,
    NUM_SEGMENTS_PER_SEQ: tl.constexpr,
):
    """Store per-segment ``M`` and ``L`` for ``reduce_segments`` to
    combine into the final softmax.

    Shared across every 3D attention epilogue; the per-token output
    stripes are mode-specific (flat / 2-stream split / 4-stream split)
    and stay inlined.
    """
    segm_offset = (
        query_offset_0.to(tl.int64) * (num_query_heads * NUM_SEGMENTS_PER_SEQ)
        + query_offset_1 * NUM_SEGMENTS_PER_SEQ
        + segm_idx
    )
    tl.store(segm_max_ptr + segm_offset, M, mask=query_mask_0 & query_mask_1)
    tl.store(segm_expsum_ptr + segm_offset, L, mask=query_mask_0 & query_mask_1)
```
**EN:** This function implements `store_segm_reduce_scalars` within the module. The docstring frames it as: Store per-segment ``M`` and ``L`` for ``reduce_segments`` to combine into the final softmax. Key calls include `store`, `to`.
**CN:** 该函数会实现 `store_segm_reduce_scalars`，其作用域位于the module。 关键调用包括 `store`, `to`。

### `compute_kv_seq_mask` function / `compute_kv_seq_mask` 函数
```python
@triton.jit
def compute_kv_seq_mask(
    query_abs_pos,
    seq_offset,
    seq_idx,
    mm_prefix_range_ptr,
    SLIDING_WINDOW: tl.constexpr,
    USE_MM_PREFIX: tl.constexpr,
    MAX_MM_RANGES: tl.constexpr,
    CHUNK_LOOKBACK: tl.constexpr = -1,
    CHUNK_SIZE: tl.constexpr = -1,
):
    """Build the KV mask for one tile.

    Causal (key <= query) by default; AND-ed with either chunked
    attention (``CHUNK_LOOKBACK >= 0``) or sliding window
    (``SLIDING_WINDOW > 0``); OR-ed with the bidirectional ranges from
    ``mm_prefix_range`` when PrefixLM / multimodal attention is active.
    Order matches FlexAttention: ``(causal AND window) OR mm_prefix``.
    Chunked attention takes precedence over sliding window when both
    are non-default — the launcher zeros ``CHUNK_LOOKBACK`` whenever
    sliding window is disabled.
    """
    # Compute attention mask: causal by default (key <= query)
    seq_mask = seq_offset[None, :] <= query_abs_pos

    # Apply sliding window / chunked attention to base mask
    # BEFORE mm_prefix OR.
    # Order must match FlexAttention:
    #   (causal AND sliding_window) OR mm_prefix
    if CHUNK_LOOKBACK > -1:
        seq_mask = seq_mask & (
            (query_abs_pos // CHUNK_SIZE - seq_offset[None, :] // CHUNK_SIZE)
            <= CHUNK_LOOKBACK
        )
    elif SLIDING_WINDOW > 0:
        seq_mask = seq_mask & ((query_abs_pos - seq_offset) < SLIDING_WINDOW)

    # PrefixLM: extend mask with bidirectional ranges for multimodal tokens.
    # Applied AFTER sliding window so mm_prefix ranges override SW restriction.
    if USE_MM_PREFIX:
        for i in range(MAX_MM_RANGES):
            range_start = tl.load(
                mm_prefix_range_ptr + seq_idx * MAX_MM_RANGES * 2 + i * 2
            )
            range_end = tl.load(
                mm_prefix_range_ptr + seq_idx * MAX_MM_RANGES * 2 + i * 2 + 1
            )
            is_valid = range_start < range_end
            q_in_range = (
                (query_abs_pos >= range_start) & (query_abs_pos <= range_end) & is_valid
            )
            k_in_range = (
                (seq_offset[None, :] >= range_start)
                & (seq_offset[None, :] <= range_end)
                & is_valid
            )
            seq_mask |= q_in_range & k_in_range
    return seq_mask
```
**EN:** This function computes derived values within the module. The docstring frames it as: Build the KV mask for one tile. Key calls include `range`, `load`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `range`, `load`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `apply_alibi_to_score` function / `apply_alibi_to_score` 函数
```python
@triton.jit
def apply_alibi_to_score(
    S,
    alibi_slope,
    seq_offset,
    context_len,
    query_pos,
    USE_ALIBI_SQRT: tl.constexpr,
):
    """Add the ALiBi positional bias (linear or sqrt variant) to S in-place."""
    if USE_ALIBI_SQRT:
        relative_pos = seq_offset - (context_len + query_pos[:, None])
        alibi_offset = tl.where(
            relative_pos <= 0,
            -tl.sqrt((-relative_pos).to(tl.float32)),
            0.0,
        )
    else:
        alibi_offset = seq_offset - context_len
    return S + alibi_slope[:, None] * alibi_offset
```
**EN:** This function implements `apply_alibi_to_score` within the module. The docstring frames it as: Add the ALiBi positional bias (linear or sqrt variant) to S in-place. Key calls include `where`, `sqrt`, `to`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `apply_alibi_to_score`，其作用域位于the module。 关键调用包括 `where`, `sqrt`, `to`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `load_qq_bias_tile` function / `load_qq_bias_tile` 函数
```python
@triton.jit
def load_qq_bias_tile(
    qq_bias_row_ptrs,
    seq_offset,
    context_len,
    qq_bias_stride_0,
):
    """Load the qq-bias slice for keys that correspond to query rows."""
    key_rel_pos = seq_offset - context_len
    is_query_key = key_rel_pos >= 0 and key_rel_pos < qq_bias_stride_0
    return tl.load(
        qq_bias_row_ptrs + key_rel_pos[None, :],
        mask=is_query_key[None, :],
        other=0.0,
    )
```
**EN:** This function loads external or cached state within the module. The docstring frames it as: Load the qq-bias slice for keys that correspond to query rows. Key calls include `load`.
**CN:** 该函数会加载外部或缓存状态，其作用域位于the module。 关键调用包括 `load`。

### `softmax_step` function / `softmax_step` 函数
```python
@triton.jit
def softmax_step(S, M, L):
    """Online softmax update for one tile.

    Returns ``(M_new, L_new, P, alpha)``.  Caller is responsible for
    rescaling its accumulator(s) by ``alpha[:, None]`` — done outside so
    kernels with a different number / shape of accumulators can reuse
    the same step.
    """
    # compute running maximum
    # m_j : (BLOCK_M,)
    m_j = tl.maximum(M, tl.max(S, axis=1))
    # For sliding window there's a chance the max is -inf due to masking of
    # the entire row. In this case we need to set m_j 0 to avoid NaN
    m_j = tl.where(m_j > float("-inf"), m_j, 0.0)
    # P : (BLOCK_M, TILE_SIZE)
    P = tl.exp(S - m_j[:, None])
    # l_j : (BLOCK_M,)
    l_j = tl.sum(P, axis=1)
    # alpha : (BLOCK_M, )
    alpha = tl.exp(M - m_j)
    # update constants
    L_new = L * alpha + l_j
    return m_j, L_new, P, alpha
```
**EN:** This function implements `softmax_step` within the module. The docstring frames it as: Online softmax update for one tile. Key calls include `maximum`, `where`, `exp`, `sum`, `max`, `float`.
**CN:** 该函数会实现 `softmax_step`，其作用域位于the module。 关键调用包括 `maximum`, `where`, `exp`, `sum`, `max`, `float`。

## Key Concepts / 关键概念
- `cdiv_fn`: top-level helper or orchestration entry point. / `cdiv_fn`：顶层辅助函数或编排入口。
- `apply_softcap`: top-level helper or orchestration entry point. / `apply_softcap`：顶层辅助函数或编排入口。
- `resolve_seq_and_query_len`: top-level helper or orchestration entry point. / `resolve_seq_and_query_len`：顶层辅助函数或编排入口。
- `find_seq_idx`: top-level helper or orchestration entry point. / `find_seq_idx`：顶层辅助函数或编排入口。
- `init_softmax_M`: top-level helper or orchestration entry point. / `init_softmax_M`：顶层辅助函数或编排入口。
- `compute_tile_loop_bounds`: top-level helper or orchestration entry point. / `compute_tile_loop_bounds`：顶层辅助函数或编排入口。
- `store_segm_reduce_scalars`: top-level helper or orchestration entry point. / `store_segm_reduce_scalars`：顶层辅助函数或编排入口。
- `compute_kv_seq_mask`: top-level helper or orchestration entry point. / `compute_kv_seq_mask`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `__future__`
- Internal vLLM / 内部依赖: `vllm.triton_utils`
