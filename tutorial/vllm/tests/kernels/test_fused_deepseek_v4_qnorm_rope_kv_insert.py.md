# test_fused_deepseek_v4_qnorm_rope_kv_insert.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/test_fused_deepseek_v4_qnorm_rope_kv_insert.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / test_fused_deepseek_v4_qnorm_rope_kv_insert, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / test_fused_deepseek_v4_qnorm_rope_kv_insert 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-17)
```python
"""
Standalone unit test for the horizontally-fused DeepseekV4-MLA kernel:

  fused_deepseek_v4_qnorm_rope_kv_rope_quant_insert
    - Q side:  per-head RMSNorm (no weight) + GPT-J RoPE on last 64 dims
    - KV side: GPT-J RoPE on last 64 + UE8M0 FP8 quant + paged cache insert

We compare against:
  - PyTorch reference for RMSNorm + GPT-J RoPE on Q
  - Existing Triton `quantize_and_insert_k_cache` + round-trip via
    `dequantize_and_gather_k_cache` for KV

The kernel is imported via
`torch.ops._C.fused_deepseek_v4_qnorm_rope_kv_rope_quant_insert`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 19-25)
```python
import pytest
import torch

from vllm.v1.attention.ops.deepseek_v4_ops import (
    dequantize_and_gather_k_cache,
    quantize_and_insert_k_cache,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.v1.attention.ops.deepseek_v4_ops.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.v1.attention.ops.deepseek_v4_ops。

### Constants and module state (lines 28-33)
```python
HEAD_DIM = 512
ROPE_DIM = 64
NOPE_DIM = HEAD_DIM - ROPE_DIM  # 448
QUANT_BLOCK = 64
FP8_MAX = 448.0
HEAD_BYTES = NOPE_DIM + ROPE_DIM * 2 + 8  # 448 + 128 + 8 = 584
```
**EN:** This block centralizes shared constants and parameter grids, including HEAD_DIM, ROPE_DIM, NOPE_DIM, QUANT_BLOCK, FP8_MAX. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 HEAD_DIM、ROPE_DIM、NOPE_DIM、QUANT_BLOCK、FP8_MAX。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `make_cos_sin_cache` (lines 39-51)
```python
def make_cos_sin_cache(max_pos: int, rope_dim: int, dtype, device):
    """Build a cos||sin cache matching DeepseekV4ScalingRotaryEmbedding layout.
    cos_sin_cache[pos, :rope_dim/2] = cos(theta), [rope_dim/2:] = sin(theta).
    """
    base = 10000.0
    inv_freq = 1.0 / (
        base
        ** (torch.arange(0, rope_dim, 2, dtype=torch.float32, device=device) / rope_dim)
    )
    t = torch.arange(max_pos, dtype=torch.float32, device=device)
    freqs = torch.einsum("i,j -> ij", t, inv_freq)  # [max_pos, rope_dim/2]
    cache = torch.cat((freqs.cos(), freqs.sin()), dim=-1)  # [max_pos, rope_dim]
    return cache.to(dtype)
```
**EN:** This helper function implements the shared logic for cos sin cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 cos sin cache 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `apply_rope_gptj_last_k` (lines 54-97)
```python
def apply_rope_gptj_last_k(
    x: torch.Tensor, positions: torch.Tensor, cos_sin_cache: torch.Tensor
) -> torch.Tensor:
    """GPT-J-style (interleaved-pair) RoPE on the LAST rope_dim elements.

    x: [..., head_dim] float32
    positions: [num_tokens] int64 (positions[i] corresponds to x[i, ...])
    cos_sin_cache: [max_pos, rope_dim] float (cos|sin layout)

    Returns rotated x (same shape/dtype).
    """
    rope_dim = cos_sin_cache.shape[-1]
    half = rope_dim // 2
    head_dim = x.shape[-1]
    nope_dim = head_dim - rope_dim

    # Gather cos/sin for each token position: [num_tokens, rope_dim]
    cs = cos_sin_cache[positions].to(torch.float32)  # [N, rope_dim]
    cos = cs[..., :half]  # [N, half]
    sin = cs[..., half:]  # [N, half]

    # Reshape leading dims so we can broadcast: x shape [..., head_dim].
    # Bring token dim to front; assume x is [num_tokens, ..., head_dim].
    # We rely on positions being per-token and all other dims sharing the same pos.
    rope = x[..., nope_dim:].float()  # [..., rope_dim]
    # Make rope pairs: reshape last dim to [half, 2]
    shape = rope.shape
    rope = rope.reshape(*shape[:-1], half, 2)
    even = rope[..., 0]  # [..., half]
    odd = rope[..., 1]

    # Broadcast cos/sin over any heads dim in between.  cos/sin are [N, half].
    # Add singleton dims for intermediate axes.
    for _ in range(rope.ndim - 3):
        cos = cos.unsqueeze(1)
        sin = sin.unsqueeze(1)

    new_even = even * cos - odd * sin
    new_odd = even * sin + odd * cos
    rope_rotated = torch.stack((new_even, new_odd), dim=-1).reshape(shape)

    out = x.clone().float()
    out[..., nope_dim:] = rope_rotated
    return out.to(x.dtype)
```
**EN:** This helper function implements the shared logic for apply rope gptj last k. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 apply rope gptj last k 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `rmsnorm_no_weight` (lines 100-106)
```python
def rmsnorm_no_weight(x: torch.Tensor, eps: float) -> torch.Tensor:
    """RMSNorm with no learnable weight, matching
    `RMSNorm(head_dim, has_weight=False)`."""
    orig_dtype = x.dtype
    xf = x.float()
    variance = xf.pow(2).mean(dim=-1, keepdim=True)
    return (xf * torch.rsqrt(variance + eps)).to(orig_dtype)
```
**EN:** This helper function implements the shared logic for rmsnorm no weight. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 rmsnorm no weight 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_op_available` (lines 112-113)
```python
def _op_available() -> bool:
    return hasattr(torch.ops._C, "fused_deepseek_v4_qnorm_rope_kv_rope_quant_insert")
```
**EN:** This helper function implements the shared logic for op available. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 op available 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 116-119)
```python
pytestmark = pytest.mark.skipif(
    not torch.cuda.is_available() or not _op_available(),
    reason="CUDA not available or fused DeepseekV4 op not built in",
)
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `_call_fused` (lines 122-125)
```python
def _call_fused(q, kv, k_cache, slot_mapping, positions, cos_sin_cache, eps, bs):
    torch.ops._C.fused_deepseek_v4_qnorm_rope_kv_rope_quant_insert(
        q, kv, k_cache, slot_mapping, positions, cos_sin_cache, eps, bs
    )
```
**EN:** This helper function implements the shared logic for call fused. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 call fused 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_q_path_matches_reference` (lines 131-159)
```python
@pytest.mark.parametrize("num_tokens", [1, 4, 17, 64])
@pytest.mark.parametrize("n_heads", [8, 64])
def test_q_path_matches_reference(num_tokens: int, n_heads: int):
    torch.manual_seed(0)
    device = "cuda"
    dtype = torch.bfloat16
    eps = 1e-6
    max_pos = 4096

    q = torch.randn(num_tokens, n_heads, HEAD_DIM, dtype=dtype, device=device)
    positions = torch.arange(num_tokens, dtype=torch.int64, device=device)
    cos_sin_cache = make_cos_sin_cache(max_pos, ROPE_DIM, torch.float32, device)

    # Reference: RMSNorm (no weight) per head, then GPT-J RoPE on last 64.
    q_ref = rmsnorm_no_weight(q, eps)
    q_ref = apply_rope_gptj_last_k(q_ref, positions, cos_sin_cache)

    # Fused call with dummy KV tensors (KV branch will write slot_mapping=-1 → noop).
    num_blocks = 2
    bs = 16
    kv = torch.zeros(num_tokens, HEAD_DIM, dtype=dtype, device=device)
    k_cache = torch.zeros(
        num_blocks, bs, HEAD_BYTES, dtype=torch.uint8, device=device
    ).view(num_blocks, -1)
    slot_mapping = torch.full((num_tokens,), -1, dtype=torch.int64, device=device)
    q_fused = q.clone()
    _call_fused(q_fused, kv, k_cache, slot_mapping, positions, cos_sin_cache, eps, bs)

    torch.testing.assert_close(q_fused, q_ref, rtol=1e-2, atol=1e-2)
```
**EN:** This pytest case verifies q path matches reference. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, n_heads. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 q path matches reference 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、n_heads 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `_ue8m0_per_block_scales` (lines 165-173)
```python
def _ue8m0_per_block_scales(kv_roped_nope_f32: torch.Tensor, qblock: int):
    """Return per-token per-block max scale (used to bound FP8 error)."""
    n_tok, nope = kv_roped_nope_f32.shape
    n_blocks = nope // qblock
    blocks = kv_roped_nope_f32.view(n_tok, n_blocks, qblock)
    absmax = blocks.abs().amax(dim=-1).clamp(min=1e-4)
    raw = absmax / FP8_MAX
    exponent = torch.ceil(torch.log2(raw))
    return torch.pow(2.0, exponent)  # [n_tok, n_blocks]
```
**EN:** This helper function implements the shared logic for ue8m0 per block scales. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 ue8m0 per block scales 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_kv_path_matches_reference` (lines 176-258)
```python
@pytest.mark.parametrize("num_tokens", [1, 4, 17, 64])
@pytest.mark.parametrize("block_size", [16, 64])
def test_kv_path_matches_reference(num_tokens: int, block_size: int):
    torch.manual_seed(1)
    device = "cuda"
    dtype = torch.bfloat16
    eps = 1e-6
    max_pos = 4096

    kv = torch.randn(num_tokens, HEAD_DIM, dtype=dtype, device=device)
    positions = torch.arange(num_tokens, dtype=torch.int64, device=device)
    cos_sin_cache = make_cos_sin_cache(max_pos, ROPE_DIM, torch.float32, device)

    num_blocks = (num_tokens + block_size - 1) // block_size + 1
    slot_mapping = torch.arange(num_tokens, dtype=torch.int64, device=device)

    # ── Reference path: RoPE on kv, then existing Triton quant+insert ──────
    kv_ref = apply_rope_gptj_last_k(kv, positions, cos_sin_cache)
    k_cache_ref = torch.zeros(
        num_blocks, block_size * HEAD_BYTES, dtype=torch.uint8, device=device
    )
    quantize_and_insert_k_cache(
        kv_ref, k_cache_ref, slot_mapping, block_size=block_size
    )

    # ── Fused path (dummy q, single head) ──────────────────────────────────
    k_cache_fused = torch.zeros_like(k_cache_ref)
    q_dummy = torch.zeros(num_tokens, 1, HEAD_DIM, dtype=dtype, device=device)
    _call_fused(
        q_dummy,
        kv,
        k_cache_fused,
        slot_mapping,
        positions,
        cos_sin_cache,
        eps,
        block_size,
    )

    # ── Round-trip compare via dequant+gather ──────────────────────────────
# ... excerpt ...
            num_reqs, num_tokens, HEAD_DIM, dtype=torch.bfloat16, device=device
        )
        seq_lens = torch.tensor([num_tokens], dtype=torch.int32, device=device)
        block_table = torch.arange(
            max_blocks, dtype=torch.int32, device=device
        ).unsqueeze(0)
        # gather_lens arg is None (use seq_lens)
        k_cache_3d = k_cache_2d.view(num_blocks, block_size, HEAD_BYTES)
        dequantize_and_gather_k_cache(
            out, k_cache_3d, seq_lens, None, block_table, block_size, offset=0
        )
        return out[0, :num_tokens]

    recovered_ref = _dequant(k_cache_ref)
    recovered_fused = _dequant(k_cache_fused)

    # NoPE: per-block UE8M0 FP8 error bound (half-ULP at max = 16 * scale).
    scales = _ue8m0_per_block_scales(kv_ref[:, :NOPE_DIM].float(), QUANT_BLOCK)
    for t in range(num_tokens):
        max_allowed = 16.0 * scales[t].max().item()
        diff_ref = (
            (recovered_ref[t, :NOPE_DIM] - kv_ref[t, :NOPE_DIM]).abs().max().item()
        )
        diff_fused = (
            (recovered_fused[t, :NOPE_DIM] - kv_ref[t, :NOPE_DIM]).abs().max().item()
        )
        assert diff_ref <= max_allowed, (
            f"ref NoPE token {t} diff {diff_ref} > {max_allowed}"
        )
        assert diff_fused <= max_allowed, (
            f"fused NoPE token {t} diff {diff_fused} > {max_allowed}"
        )

    # RoPE region: bf16 stored exactly → zero diff.
    rope_diff = (recovered_fused[:, NOPE_DIM:] - kv_ref[:, NOPE_DIM:]).abs().max()
    assert rope_diff.item() == 0.0, f"RoPE portion not exact: {rope_diff.item()}"

    # Exact byte equality of the two cache buffers — strong parity.
    torch.testing.assert_close(k_cache_fused, k_cache_ref, rtol=0, atol=0)
```
**EN:** This pytest case verifies KV path matches reference. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, block_size. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 KV path matches reference 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、block_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_kv_path_with_dp_padding` (lines 264-309)
```python
@pytest.mark.parametrize("num_tokens", [4, 17])
@pytest.mark.parametrize("pad", [1, 5])
@pytest.mark.parametrize("block_size", [16, 64])
def test_kv_path_with_dp_padding(num_tokens: int, pad: int, block_size: int):
    """slot_mapping.size(0) < q.size(0): the kernel must skip padded
    tokens in the KV branch while still running Q-norm+RoPE on all rows."""
    torch.manual_seed(3)
    device = "cuda"
    dtype = torch.bfloat16
    eps = 1e-6
    max_pos = 4096
    total = num_tokens + pad

    kv = torch.randn(total, HEAD_DIM, dtype=dtype, device=device)
    positions = torch.arange(total, dtype=torch.int64, device=device)
    cos_sin_cache = make_cos_sin_cache(max_pos, ROPE_DIM, torch.float32, device)

    num_blocks = (num_tokens + block_size - 1) // block_size + 1
    slot_mapping = torch.arange(num_tokens, dtype=torch.int64, device=device)

    # Reference: only the first num_tokens kv rows get inserted.
    kv_ref = apply_rope_gptj_last_k(
        kv[:num_tokens], positions[:num_tokens], cos_sin_cache
    )
    k_cache_ref = torch.zeros(
        num_blocks, block_size * HEAD_BYTES, dtype=torch.uint8, device=device
    )
    quantize_and_insert_k_cache(
        kv_ref, k_cache_ref, slot_mapping, block_size=block_size
    )

    # Fused: pass full-sized q/kv/positions, shorter slot_mapping.
    q_dummy = torch.zeros(total, 1, HEAD_DIM, dtype=dtype, device=device)
    k_cache_fused = torch.zeros_like(k_cache_ref)
    _call_fused(
        q_dummy,
        kv,
        k_cache_fused,
        slot_mapping,
        positions,
        cos_sin_cache,
        eps,
        block_size,
    )

    torch.testing.assert_close(k_cache_fused, k_cache_ref, rtol=0, atol=0)
```
**EN:** This pytest case verifies KV path with dp padding. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, pad, block_size. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 KV path with dp padding 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、pad、block_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_combined_q_and_kv` (lines 315-359)
```python
@pytest.mark.parametrize("num_tokens", [1, 4, 17])
@pytest.mark.parametrize("n_heads", [8, 64])
@pytest.mark.parametrize("block_size", [16, 64])
def test_combined_q_and_kv(num_tokens: int, n_heads: int, block_size: int):
    torch.manual_seed(2)
    device = "cuda"
    dtype = torch.bfloat16
    eps = 1e-6
    max_pos = 4096

    q = torch.randn(num_tokens, n_heads, HEAD_DIM, dtype=dtype, device=device)
    kv = torch.randn(num_tokens, HEAD_DIM, dtype=dtype, device=device)
    positions = torch.arange(num_tokens, dtype=torch.int64, device=device)
    cos_sin_cache = make_cos_sin_cache(max_pos, ROPE_DIM, torch.float32, device)

    num_blocks = (num_tokens + block_size - 1) // block_size + 1
    slot_mapping = torch.arange(num_tokens, dtype=torch.int64, device=device)

    # Reference.
    q_ref = rmsnorm_no_weight(q, eps)
    q_ref = apply_rope_gptj_last_k(q_ref, positions, cos_sin_cache)
    kv_ref = apply_rope_gptj_last_k(kv, positions, cos_sin_cache)
    k_cache_ref = torch.zeros(
        num_blocks, block_size * HEAD_BYTES, dtype=torch.uint8, device=device
    )
    quantize_and_insert_k_cache(
        kv_ref, k_cache_ref, slot_mapping, block_size=block_size
    )

    # Fused single call.
    q_fused = q.clone()
    k_cache_fused = torch.zeros_like(k_cache_ref)
    _call_fused(
        q_fused,
        kv,
        k_cache_fused,
        slot_mapping,
        positions,
        cos_sin_cache,
        eps,
        block_size,
    )

    torch.testing.assert_close(q_fused, q_ref, rtol=1e-2, atol=1e-2)
    torch.testing.assert_close(k_cache_fused, k_cache_ref, rtol=0, atol=0)
```
**EN:** This pytest case verifies combined q and KV. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, n_heads, block_size. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 combined q and KV 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、n_heads、block_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.v1.attention.ops.deepseek_v4_ops -> dequantize_and_gather_k_cache, quantize_and_insert_k_cache`
