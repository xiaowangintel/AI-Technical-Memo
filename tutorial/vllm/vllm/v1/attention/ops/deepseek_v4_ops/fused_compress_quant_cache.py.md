# fused_compress_quant_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/deepseek_v4_ops/fused_compress_quant_cache.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fused compressor + FP8/MXFP4 UE8M0 quantization + KV cache insert kernels. / 该模块位于 `attention/ops/deepseek_v4_ops` 子系统，主要围绕 `_fused_kv_compress_norm_rope_insert_sparse_attn`, `_fused_kv_compress_norm_rope_insert_indexer_attn`, `_fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
Fused compressor + FP8/MXFP4 UE8M0 quantization + KV cache insert kernels.

Three specialized kernels:
  - _fused_kv_compress_norm_rope_insert_sparse_attn:
        head=512, nope=448 FP8 + rope=64 bf16
  - _fused_kv_compress_norm_rope_insert_indexer_attn:
        head=128, all FP8, 1 block/token
  - _fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn:
        head=128, MXFP4 (block=32), 4 ue8m0 bytes

RoPE is register-based via tl.reshape -> tl.split -> tl.interleave (or the
even/odd halves are consumed directly for MXFP4, no interleave needed).
FP8 UE8M0 quant uses tl.reshape to tile [N_QUANT_BLOCKS, QUANT_BLOCK] for
per-block absmax entirely in registers. MXFP4 does the same tiling on the
even/odd halves, producing (N_QUANT_BLOCKS, MXFP4_BLOCK/2) packed nibbles
and N_QUANT_BLOCKS ue8m0 bytes.
"""

from vllm.triton_utils import tl, triton

from .fused_indexer_q import _fp32x2_to_fp4x2
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `_fused_kv_compress_norm_rope_insert_sparse_attn` function / `_fused_kv_compress_norm_rope_insert_sparse_attn` 函数
```python
@triton.jit
def _fused_kv_compress_norm_rope_insert_sparse_attn(
    # ── state cache (compressor internal state) ──
    state_cache_ptr,
    state_cache_stride0,
    state_cache_stride1,
    # ── metadata ──
    token_to_req_indices_ptr,
    positions_ptr,
    slot_mapping_ptr,
    block_table_ptr,
    block_table_stride,
    block_size,
    # ── RMSNorm ──
    rms_norm_weight_ptr,
    rms_norm_eps,
    # ── RoPE ──
    cos_sin_cache_ptr,
    cos_sin_stride,
    # ── KV cache output ──
    k_cache_ptr,
    kv_slot_mapping_ptr,
    kv_cache_block_size,
    # ── constexprs ──
    HEAD_SIZE: tl.constexpr,
    TRITON_BLOCK_SIZE: tl.constexpr,
    STATE_WIDTH: tl.constexpr,
    COMPRESS_RATIO: tl.constexpr,
    OVERLAP: tl.constexpr,
    ROPE_HEAD_DIM: tl.constexpr,
    FP8_MAX: tl.constexpr,  # 448.0
    QUANT_BLOCK: tl.constexpr,  # 64 for DeepseekV4
    TOKEN_STRIDE: tl.constexpr,  # 576 for DeepseekV4
    SCALE_DIM: tl.constexpr,  # 8 for DeepseekV4 (7 real + 1 pad)
    KV_BLOCK_STRIDE: tl.constexpr,
):
    """Fused compress → RMSNorm → FP8 quant (nope) → RoPE → bf16 store (rope).

    One program per token; early-exits for non-boundary positions.

    Cache block layout (``block_size`` tokens):
      [0, bs*576):       token data (448 fp8 + 128 bf16 each)
      [bs*576, +bs*8):   uint8 UE8M0 scales (7 real + 1 pad each)
    """
    token_idx = tl.program_id(0)

    slot_id = tl.load(slot_mapping_ptr + token_idx)
    if slot_id < 0:
        return

# ... omitted for brevity ...
    NUM_PAIRS: tl.constexpr = TRITON_BLOCK_SIZE // 2
    NOPE_PAIRS: tl.constexpr = NOPE_HEAD_DIM // 2

    pair_2d = tl.reshape(normed, (NUM_PAIRS, 2))
    even, odd = tl.split(pair_2d)  # each [NUM_PAIRS] fp32

    pair_idx = tl.arange(0, NUM_PAIRS)
    rope_pair_local = pair_idx - NOPE_PAIRS
    is_rope_pair = rope_pair_local >= 0
    cs_idx = tl.maximum(rope_pair_local, 0)

    compressed_pos = (position // COMPRESS_RATIO) * COMPRESS_RATIO
    cache_base = cos_sin_cache_ptr + compressed_pos * cos_sin_stride
    cos_v = tl.load(cache_base + cs_idx, mask=is_rope_pair, other=1.0)
    sin_v = tl.load(cache_base + HALF_ROPE + cs_idx, mask=is_rope_pair, other=0.0)

    new_even = even * cos_v - odd * sin_v
    new_odd = odd * cos_v + even * sin_v
    result = tl.interleave(new_even, new_odd)  # [TRITON_BLOCK_SIZE] fp32

    # Store rotated rope portion as bf16 into the cache's bf16 area.
    bf16_ptr = (fp8_ptr + NOPE_HEAD_DIM).to(tl.pointer_type(tl.bfloat16))
    rope_local = block - NOPE_HEAD_DIM
    is_rope = (block >= NOPE_HEAD_DIM) & mask
    tl.store(bf16_ptr + rope_local, result.to(tl.bfloat16), mask=is_rope)
```
**EN:** This function implements `_fused_kv_compress_norm_rope_insert_sparse_attn` within the module. The docstring frames it as: Fused compress → RMSNorm → FP8 quant (nope) → RoPE → bf16 store (rope). Key calls include `program_id`, `load`, `arange`, `to`, `softmax`, `sum`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_fused_kv_compress_norm_rope_insert_sparse_attn`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `arange`, `to`, `softmax`, `sum`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `_fused_kv_compress_norm_rope_insert_indexer_attn` function / `_fused_kv_compress_norm_rope_insert_indexer_attn` 函数
```python
@triton.jit
def _fused_kv_compress_norm_rope_insert_indexer_attn(
    # ── state cache (compressor internal state) ──
    state_cache_ptr,
    state_cache_stride0,
    state_cache_stride1,
    # ── metadata ──
    token_to_req_indices_ptr,
    positions_ptr,
    slot_mapping_ptr,
    block_table_ptr,
    block_table_stride,
    block_size,
    # ── RMSNorm ──
    rms_norm_weight_ptr,
    rms_norm_eps,
    # ── RoPE ──
    cos_sin_cache_ptr,
    cos_sin_stride,
    # ── KV cache output ──
    k_cache_ptr,
    kv_slot_mapping_ptr,
    kv_cache_block_size,
    # ── constexprs ──
    HEAD_SIZE: tl.constexpr,
    TRITON_BLOCK_SIZE: tl.constexpr,
    STATE_WIDTH: tl.constexpr,
    COMPRESS_RATIO: tl.constexpr,
    OVERLAP: tl.constexpr,
    ROPE_HEAD_DIM: tl.constexpr,
    FP8_MAX: tl.constexpr,  # 448.0
    QUANT_BLOCK: tl.constexpr,  # 128 for indexer
    TOKEN_STRIDE: tl.constexpr,  # 128 for indexer
    SCALE_DIM: tl.constexpr,  # 4 for indexer (1 float32)
    KV_BLOCK_STRIDE: tl.constexpr,
):
    """Fused compress → RMSNorm → RoPE → FP8 quant → store.

    One program per token; early-exits for non-boundary positions.

    Cache block layout:
      [0, bs*128):       FP8 data (128 bytes/token)
      [bs*128, +bs*4):   float32 scales (4 bytes/token)

    For head_dim=128 we have exactly one quant block, so we skip the
    [N_QUANT_BLOCKS, QUANT_BLOCK] reshape entirely and use a flat
    ``tl.max`` reduction.
    """
    token_idx = tl.program_id(0)

# ... omitted for brevity ...

    # ── FP8 UE8M0 quant: single block, flat reduction ────────────────
    tl.static_assert(
        TRITON_BLOCK_SIZE == QUANT_BLOCK,
        "Indexer expects one quant block (QUANT_BLOCK == TRITON_BLOCK_SIZE)",
    )
    INV_FP8_MAX: tl.constexpr = 1.0 / FP8_MAX

    result_bf16 = result.to(tl.bfloat16).to(tl.float32)
    absmax = tl.max(tl.abs(result_bf16), axis=0)  # scalar
    absmax = tl.maximum(absmax, 1e-4)
    raw_scale = absmax * INV_FP8_MAX
    exponent = tl.ceil(tl.log2(raw_scale))
    inv_scale = tl.exp2(-exponent)

    x_scaled = result_bf16 * inv_scale
    x_clamped = tl.clamp(x_scaled, -FP8_MAX, FP8_MAX)
    x_fp8 = x_clamped.to(tl.float8e4nv)
    x_uint8 = x_fp8.to(tl.uint8, bitcast=True)

    tl.store(fp8_ptr + block, x_uint8, mask=mask)

    # Single float32 scale
    scale_val = tl.exp2(exponent)
    tl.store(scale_ptr.to(tl.pointer_type(tl.float32)), scale_val)
```
**EN:** This function implements `_fused_kv_compress_norm_rope_insert_indexer_attn` within the module. The docstring frames it as: Fused compress → RMSNorm → RoPE → FP8 quant → store. Key calls include `program_id`, `load`, `arange`, `to`, `softmax`, `sum`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_fused_kv_compress_norm_rope_insert_indexer_attn`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `arange`, `to`, `softmax`, `sum`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `_fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn` function / `_fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn` 函数
```python
@triton.jit
def _fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn(
    # ── state cache (compressor internal state) ──
    state_cache_ptr,
    state_cache_stride0,
    state_cache_stride1,
    # ── metadata ──
    token_to_req_indices_ptr,
    positions_ptr,
    slot_mapping_ptr,
    block_table_ptr,
    block_table_stride,
    block_size,
    # ── RMSNorm ──
    rms_norm_weight_ptr,
    rms_norm_eps,
    # ── RoPE ──
    cos_sin_cache_ptr,
    cos_sin_stride,
    # ── KV cache output ──
    k_cache_ptr,
    kv_slot_mapping_ptr,
    kv_cache_block_size,
    # ── constexprs ──
    HEAD_SIZE: tl.constexpr,
    TRITON_BLOCK_SIZE: tl.constexpr,
    STATE_WIDTH: tl.constexpr,
    COMPRESS_RATIO: tl.constexpr,
    OVERLAP: tl.constexpr,
    ROPE_HEAD_DIM: tl.constexpr,
    FP8_MAX: tl.constexpr,  # unused for MXFP4 (kept for signature parity)
    QUANT_BLOCK: tl.constexpr,  # 32 for MXFP4
    TOKEN_STRIDE: tl.constexpr,  # HEAD_SIZE // 2 = 64 packed bytes/token
    SCALE_DIM: tl.constexpr,  # HEAD_SIZE // QUANT_BLOCK = 4 ue8m0 bytes/token
    KV_BLOCK_STRIDE: tl.constexpr,
):
    """Fused compress → RMSNorm → RoPE → MXFP4 quant → store.

    One program per token; early-exits for non-boundary positions.

    Cache block layout (``block_size`` tokens per cache block):
      [0, bs*TOKEN_STRIDE):        packed MXFP4 nibbles (2 values/byte)
      [bs*TOKEN_STRIDE, +bs*SCALE_DIM): ue8m0 scale bytes (one per 32-elem block)

    MXFP4 format:
      - E2M1 4-bit values packed two per byte (low nibble first, then high).
      - Per-32-element block scale = 2^ceil(log2(amax / 6.0)), stored ue8m0
        (byte = exponent + 127).
      - Max representable magnitude = 6.0.
    """
# ... omitted for brevity ...
    tl.static_assert(SCALE_DIM == N_QUANT_BLOCKS)

    even_2d = tl.reshape(new_even, (N_QUANT_BLOCKS, HALF_BLOCK))
    odd_2d = tl.reshape(new_odd, (N_QUANT_BLOCKS, HALF_BLOCK))

    amax = tl.maximum(
        tl.max(tl.abs(even_2d), axis=1),
        tl.max(tl.abs(odd_2d), axis=1),
    )
    amax = tl.maximum(amax, 6.0 * (2**-126))

    # ue8m0 block scale: 2^ceil(log2(amax / 6.0)), stored as (exp + 127) byte.
    log2_ratio = tl.ceil(tl.log2(amax * (1.0 / 6.0)))
    log2_ratio = tl.minimum(tl.maximum(log2_ratio, -127.0), 127.0)
    inv_scale = tl.exp2(-log2_ratio)
    ue8m0 = (log2_ratio + 127.0).to(tl.uint8)  # [N_QUANT_BLOCKS]

    inv_scale_col = tl.reshape(inv_scale, (N_QUANT_BLOCKS, 1))
    packed = _fp32x2_to_fp4x2(
        even_2d * inv_scale_col, odd_2d * inv_scale_col
    )  # (N_BLOCKS, HALF_BLOCK) uint8
    packed_flat = tl.reshape(packed, (TOKEN_STRIDE,))

    tl.store(val_ptr + tl.arange(0, TOKEN_STRIDE), packed_flat)
    tl.store(scale_ptr + tl.arange(0, SCALE_DIM), ue8m0)
```
**EN:** This function implements `_fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn` within the module. The docstring frames it as: Fused compress → RMSNorm → RoPE → MXFP4 quant → store. Key calls include `program_id`, `load`, `arange`, `to`, `softmax`, `sum`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `arange`, `to`, `softmax`, `sum`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `_fused_kv_compress_norm_rope_insert_sparse_attn`: top-level helper or orchestration entry point. / `_fused_kv_compress_norm_rope_insert_sparse_attn`：顶层辅助函数或编排入口。
- `_fused_kv_compress_norm_rope_insert_indexer_attn`: top-level helper or orchestration entry point. / `_fused_kv_compress_norm_rope_insert_indexer_attn`：顶层辅助函数或编排入口。
- `_fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn`: top-level helper or orchestration entry point. / `_fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Internal vLLM / 内部依赖: `vllm.triton_utils`, `.fused_indexer_q`
