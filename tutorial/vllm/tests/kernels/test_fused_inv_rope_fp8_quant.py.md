# test_fused_inv_rope_fp8_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/test_fused_inv_rope_fp8_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / test_fused_inv_rope_fp8_quant, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / test_fused_inv_rope_fp8_quant 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-19)
```python
"""
Unit tests for the fused inverse RoPE + block-scaled FP8 quantization kernel.

Tests compare the fused kernel against a reference implementation built from
the existing separate operations (inverse RoPE via rotate_neox + FP8 quant
via per_token_group_quant_fp8).

The reference faithfully reproduces the exact flow in deepseek_v4_attention.py:295-310:
  1. Apply inverse RoPE (NeoX style, last rope_dim=64 dims of each head)
  2. Reshape [T, H, head_dim] -> [T, G, D]
  3. Transpose+flatten to [G*T, D], quantize, reshape back
  4. Return o_fp8 and o_scale with strides (D, T*D, 1) and (S, T*S, 1)
     (non-contiguous [T, G, ...] view backed by contiguous [G, T, ...] memory)

Usage:
    pytest tests/kernels/test_fused_inv_rope_fp8_quant.py -v
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 21-24)
```python
import pytest
import torch

from vllm.v1.attention.ops.deepseek_v4_ops import fused_inv_rope_fp8_quant
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.v1.attention.ops.deepseek_v4_ops.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.v1.attention.ops.deepseek_v4_ops。

### Constants and module state (lines 27-33)
```python
HEAD_DIM = 512
NOPE_DIM = 448
ROPE_DIM = 64
QUANT_GROUP_SIZE = 128
FP8_MAX = 448.0  # torch.finfo(torch.float8_e4m3fn).max
FP8_DTYPE = torch.float8_e4m3fn
EPS = 1e-10
```
**EN:** This block centralizes shared constants and parameter grids, including HEAD_DIM, NOPE_DIM, ROPE_DIM, QUANT_GROUP_SIZE, FP8_MAX. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 HEAD_DIM、NOPE_DIM、ROPE_DIM、QUANT_GROUP_SIZE、FP8_MAX。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `assert_dequant_close` (lines 41-73)
```python
def assert_dequant_close(
    fp8_a: torch.Tensor,
    scale_a: torch.Tensor,
    fp8_b: torch.Tensor,
    scale_b: torch.Tensor,
    msg: str = "",
):
    """Compare two FP8-quantized tensors via their dequantized values.

    Uses cosine-similarity-based diff (same as deep_gemm calc_diff).
    Both fused and reference paths rotate in fp32 using an fp32
    cos_sin_cache, so differences are only fp32 ordering ULPs that can
    occasionally shift FP8 values at quantization boundaries.
    """
    S = scale_a.shape[-1]
    shape = fp8_a.shape

    dq_a = fp8_a.float() * scale_a.unsqueeze(-1).expand(
        *shape[:-1], S, QUANT_GROUP_SIZE
    ).reshape(shape)
    dq_b = fp8_b.float() * scale_b.unsqueeze(-1).expand(
        *shape[:-1], S, QUANT_GROUP_SIZE
    ).reshape(shape)

    # Cosine diff: 1 - cos_sim (0 = identical, higher = worse)
    dq_a_flat = dq_a.flatten().float()
    dq_b_flat = dq_b.flatten().float()
    cos_sim = torch.nn.functional.cosine_similarity(
        dq_a_flat.unsqueeze(0), dq_b_flat.unsqueeze(0)
    ).item()
    diff = 1.0 - cos_sim

    assert diff < 1e-4, f"Dequant diff too large: {diff:.8f} (expected < 1e-4). {msg}"
```
**EN:** This helper function implements the shared logic for assert dequant close. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 assert dequant close 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `rotate_gptj` (lines 76-85)
```python
def rotate_gptj(x: torch.Tensor) -> torch.Tensor:
    """GPT-J style rotation: interleaved pairs, negate-swap.

    Matches vllm/model_executor/layers/rotary_embedding/common.py:23-27.
    DeepseekV4 uses is_neox_style=False, so this is the correct rotation.
    """
    x1 = x[..., ::2]
    x2 = x[..., 1::2]
    x = torch.stack((-x2, x1), dim=-1)
    return x.flatten(-2)
```
**EN:** This helper function implements the shared logic for rotate gptj. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 rotate gptj 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_cos_sin_cache` (lines 88-111)
```python
def make_cos_sin_cache(
    max_pos: int,
    rope_dim: int = ROPE_DIM,
    dtype: torch.dtype = torch.float32,
    device: str = "cuda",
) -> torch.Tensor:
    """Create a synthetic cos_sin_cache matching the layout used by
    DeepseekV4ScalingRotaryEmbedding._compute_cos_sin_cache.

    Shape: [max_pos, rope_dim] where first half is cos, second half is sin.
    The fused kernel requires fp32; callers can override dtype if passing
    the cache into the bf16-only paths.
    """
    half = rope_dim // 2
    # Use random but bounded frequencies so cos/sin are well-behaved
    inv_freq = 1.0 / (
        10000.0 ** (torch.arange(0, half, device=device, dtype=torch.float32) / half)
    )
    t = torch.arange(max_pos, device=device, dtype=torch.float32)
    freqs = torch.outer(t, inv_freq)  # [max_pos, half]
    cos = freqs.cos()
    sin = freqs.sin()
    cache = torch.cat((cos, sin), dim=-1)  # [max_pos, rope_dim]
    return cache.to(dtype)
```
**EN:** This helper function implements the shared logic for cos sin cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 cos sin cache 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `reference_inv_rope` (lines 114-151)
```python
def reference_inv_rope(
    o: torch.Tensor,
    positions: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    nope_dim: int = NOPE_DIM,
    rope_dim: int = ROPE_DIM,
) -> torch.Tensor:
    """Apply inverse RoPE to the last rope_dim dimensions of each head.

    Matches the GPT-J inverse rotation in pos_encoding_kernels.cu, which
    promotes the cache to fp32 and performs the rotation in fp32. The
    result is cast back to the input dtype.

    Args:
        o: [T, H, head_dim] bf16
        positions: [T] int64
        cos_sin_cache: [max_pos, rope_dim] fp32

    Returns:
        o with inverse RoPE applied on the rope portion (bf16).
    """
    assert cos_sin_cache.dtype == torch.float32
    cos_sin = cos_sin_cache[positions]  # [T, rope_dim] fp32
    half = rope_dim // 2
    cos = cos_sin[:, :half]
    sin = cos_sin[:, half:]

    # GPT-J style: repeat_interleave (not repeat) to match interleaved pairs
    cos = cos.repeat_interleave(2, dim=-1).unsqueeze(1)
    sin = sin.repeat_interleave(2, dim=-1).unsqueeze(1)
    sin = -sin  # inverse

    o_pass = o[..., :nope_dim]
    o_rot_f32 = o[..., nope_dim:].float()
    o_rot_f32 = o_rot_f32 * cos + rotate_gptj(o_rot_f32) * sin
    o_rot = o_rot_f32.to(o.dtype)

    return torch.cat([o_pass, o_rot], dim=-1)
```
**EN:** This helper function implements the shared logic for reference inv rope. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 reference inv rope 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `_ref_ue8m0_quant_block` (lines 154-173)
```python
def _ref_ue8m0_quant_block(x_f32: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
    """Per-block UE8M0 FP8 quantization in pure float32.

    Matches the Triton kernel logic exactly:
      absmax -> 2^ceil(log2(absmax / fp8_max)) -> clamp(x / scale) -> fp8

    Args:
        x_f32: [..., quant_group_size] float32 — one or more 128-element blocks.

    Returns:
        x_fp8: same shape, float8_e4m3fn
        scales: [...] float32, one scale per block
    """
    assert x_f32.shape[-1] == QUANT_GROUP_SIZE
    absmax = x_f32.abs().amax(dim=-1, keepdim=True).clamp(min=EPS)
    scale_raw = absmax * (1.0 / FP8_MAX)
    scale = torch.exp2(torch.ceil(torch.log2(scale_raw)))
    x_scaled = (x_f32 / scale).clamp(-FP8_MAX, FP8_MAX)
    x_fp8 = x_scaled.to(FP8_DTYPE)
    return x_fp8, scale.squeeze(-1)
```
**EN:** This helper function implements the shared logic for ref ue8m0 quant block. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 ref ue8m0 quant block 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `reference_inv_rope_fp8_quant` (lines 176-247)
```python
def reference_inv_rope_fp8_quant(
    o: torch.Tensor,
    positions: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    n_groups: int,
    heads_per_group: int,
    nope_dim: int = NOPE_DIM,
    rope_dim: int = ROPE_DIM,
    quant_group_size: int = QUANT_GROUP_SIZE,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Full reference: inverse RoPE in fp32 + UE8M0 FP8 quant in fp32.

    Mimics the Triton kernel's precision path exactly:
      Load bf16 -> cast to fp32 -> apply inverse RoPE with fp32 cos/sin ->
      UE8M0 quant in fp32 -> write fp8 + scale

    Returns:
        o_fp8: [T, G, D] FP8 with strides (D, T*D, 1)
        o_scale: [T, G, S] FP32 with strides (S, T*S, 1)
    """
    assert cos_sin_cache.dtype == torch.float32
    T, _H, head_dim = o.shape
    d = heads_per_group * head_dim
    S = d // quant_group_size
    half_rope = rope_dim // 2
    chunks_per_head = head_dim // quant_group_size

    # Reshape [T, H, head_dim] -> [T, G, heads_per_group, head_dim]
    o_4d = o.view(T, n_groups, heads_per_group, head_dim)

    # Lookup cos/sin directly in fp32
    cos_sin = cos_sin_cache[positions]  # [T, rope_dim] fp32
    cos = cos_sin[:, :half_rope]  # [T, half_rope] fp32
    sin = cos_sin[:, half_rope:]  # [T, half_rope] fp32

    # Allocate outputs in [G, T, ...] contiguous layout
    fp8_buf = torch.empty(n_groups, T, d, dtype=FP8_DTYPE, device=o.device)
    scale_buf = torch.empty(n_groups, T, S, dtype=torch.float32, device=o.device)

    # Process each quant block, matching the Triton kernel's per-program logic
    for g in range(n_groups):
        for qb in range(S):
            head_in_group = qb // chunks_per_head
            chunk_in_head = qb % chunks_per_head
            offset = chunk_in_head * quant_group_size

            # Load 128 bf16 elements and promote to fp32 for rotation+quant
            block = o_4d[:, g, head_in_group, offset : offset + quant_group_size]
            x = block.float()

            # Apply inverse RoPE in fp32 if this is the last chunk
            # GPT-J style: interleaved pairs (even=x, odd=y)
            if chunk_in_head == chunks_per_head - 1:
                rope_start = nope_dim % quant_group_size  # 64
                rope_region = x[:, rope_start:].clone()
                x_vals = rope_region[:, ::2]
                y_vals = rope_region[:, 1::2]
                x_new = x_vals * cos + y_vals * sin
                y_new = y_vals * cos - x_vals * sin
                x = x.clone()
                x[:, rope_start::2] = x_new
                x[:, rope_start + 1 :: 2] = y_new

            # UE8M0 quant in fp32
            x_fp8, scale = _ref_ue8m0_quant_block(x)

            # Write to [G, T, D] contiguous memory
            fp8_buf[g, :, qb * quant_group_size : (qb + 1) * quant_group_size] = x_fp8
            scale_buf[g, :, qb] = scale

    # Return transposed views
    return fp8_buf.transpose(0, 1), scale_buf.transpose(0, 1)
```
**EN:** This helper function implements the shared logic for reference inv rope FP8 quant. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 reference inv rope FP8 quant 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_correctness` (lines 255-319)
```python
@pytest.mark.parametrize("num_tokens", [1, 7, 32, 128])
@pytest.mark.parametrize(
    "num_heads,n_groups",
    [(64, 8), (32, 4), (128, 8)],
    ids=["H64_G8", "H32_G4", "H128_G8"],
)
@pytest.mark.parametrize("seed", [0, 42])
@torch.inference_mode()
def test_correctness(num_tokens, num_heads, n_groups, seed):
    """Compare fused kernel against reference for FP8 values and scales."""
    torch.manual_seed(seed)

    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"

    # Create inputs
    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.randint(
        0, max_pos, (num_tokens,), device=device, dtype=torch.long
    )
    cos_sin_cache = make_cos_sin_cache(
        max_pos, ROPE_DIM, dtype=torch.float32, device=device
    )

    # Reference
    ref_fp8, ref_scale = reference_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    # Fused kernel
    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    # Check shapes
    d = heads_per_group * HEAD_DIM
    S = d // QUANT_GROUP_SIZE
    assert ref_fp8.shape == (num_tokens, n_groups, d)
    assert fused_fp8.shape == (num_tokens, n_groups, d)
    assert ref_scale.shape == (num_tokens, n_groups, S)
    assert fused_scale.shape == (num_tokens, n_groups, S)

    # Scales: exact match (both use identical UE8M0 algorithm)
    # Scales may differ by one UE8M0 step (factor of 2) if fp32 rotation
    # ordering shifts absmax across a power-of-2 boundary. Check ratio is
    # close to 1.
    scale_ratio = fused_scale / ref_scale.clamp(min=1e-30)
    assert scale_ratio.max() <= 2.0 and scale_ratio.min() >= 0.5, (
        f"Scale ratio out of [0.5, 2]: min={scale_ratio.min():.4f} "
        f"max={scale_ratio.max():.4f}"
    )

    # Compare via dequant (Triton vs PyTorch fp32 may differ by ULPs)
    assert_dequant_close(ref_fp8, ref_scale, fused_fp8, fused_scale)
```
**EN:** This pytest case verifies correctness. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, num_heads, n_groups, seed. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 correctness 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、num_heads、n_groups、seed 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

### Function `test_output_strides` (lines 322-369)
```python
@pytest.mark.parametrize("num_tokens", [1, 7, 32, 128])
@pytest.mark.parametrize(
    "num_heads,n_groups",
    [(64, 8), (128, 8)],
    ids=["H64_G8", "H128_G8"],
)
@torch.inference_mode()
def test_output_strides(num_tokens, num_heads, n_groups):
    """Verify fused output layout:
    - FP8: logical [T, G, D] backed by contiguous [G, T, D].
    - Scale: MN-major TMA-aligned (column-major: T-stride=1).
    """

    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"

    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.randint(
        0, max_pos, (num_tokens,), device=device, dtype=torch.long
    )
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    # FP8: logical [T, G, D] backed by [G, T, D] row-major
    d = heads_per_group * HEAD_DIM
    expected_fp8_stride = (d, num_tokens * d, 1)
    assert fused_fp8.stride() == expected_fp8_stride, (
        f"FP8 stride mismatch: got {fused_fp8.stride()}, expected {expected_fp8_stride}"
    )

    # Scale: MN-major TMA-aligned layout. After fp8_einsum permutes
    # [T,G,S] -> [G,T,S], T-dim should have stride 1.
    # Our output is [T,G,S] = transpose of [G,T,S].
    # So fused_scale.permute(1,0,2) should have T-stride=1.
    perm = fused_scale.permute(1, 0, 2)  # [G, T, S]
    assert perm.stride(1) == 1 or num_tokens == 1, (
        f"Scale T-stride (after permute to [G,T,S]) should be 1, got {perm.stride(1)}"
    )
```
**EN:** This pytest case verifies output strides. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, num_heads, n_groups. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 output strides 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、num_heads、n_groups 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_per_group_contiguity` (lines 372-404)
```python
@pytest.mark.parametrize("num_tokens", [1, 7, 32, 128])
@torch.inference_mode()
def test_per_group_contiguity(num_tokens):
    """FP8 per-group slices must be contiguous. Scale per-group slices
    are column-major (T-stride=1) — not row-major contiguous, which is
    correct for TMA loads."""
    num_heads, n_groups = 64, 8
    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"

    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.randint(
        0, max_pos, (num_tokens,), device=device, dtype=torch.long
    )
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    for g in range(n_groups):
        fp8_slice = fused_fp8[:, g, :]
        assert fp8_slice.is_contiguous(), (
            f"o_fp8[:, {g}, :] is not contiguous: "
            f"shape={list(fp8_slice.shape)}, stride={list(fp8_slice.stride())}"
        )
```
**EN:** This pytest case verifies per group contiguity. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 per group contiguity 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 num_tokens 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_scales_are_power_of_two` (lines 407-436)
```python
@torch.inference_mode()
def test_scales_are_power_of_two():
    """Verify all scales are exact powers of 2 (UE8M0 property)."""
    num_tokens, num_heads, n_groups = 32, 64, 8
    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"

    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.randint(
        0, max_pos, (num_tokens,), device=device, dtype=torch.long
    )
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    _, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    # log2 of a power-of-two is an exact integer
    log2_scales = torch.log2(fused_scale)
    residual = (log2_scales - log2_scales.round()).abs()
    assert residual.max() < 1e-5, (
        f"Not all scales are powers of 2: max log2 residual = {residual.max().item()}"
    )
```
**EN:** This pytest case verifies scales are power of two. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 scales are power of two 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_nope_dims_unchanged` (lines 439-502)
```python
@torch.inference_mode()
def test_nope_dims_unchanged():
    """Nope dimensions (first 448 per head) should only be quantized,
    not rotated. Verify by dequantizing and comparing against
    quantize-only reference (no RoPE)."""
    num_tokens, num_heads, n_groups = 16, 64, 8
    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"
    torch.manual_seed(0)

    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.randint(
        0, max_pos, (num_tokens,), device=device, dtype=torch.long
    )
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    # Fused kernel result
    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    # Reference: quantize without RoPE (identity rotation)
    # Create a zero-sin cache so RoPE is identity
    zero_cache = torch.zeros_like(cos_sin_cache)
    half = ROPE_DIM // 2
    zero_cache[:, :half] = 1.0  # cos = 1
    # sin = 0 (already zero)

    norope_fp8, norope_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        zero_cache,
        n_groups,
        heads_per_group,
    )

    # Extract nope quant blocks only (first 3 of every 4 blocks per head)
    chunks_per_head = HEAD_DIM // QUANT_GROUP_SIZE  # 4

    for h in range(heads_per_group):
        for c in range(chunks_per_head - 1):  # skip last chunk (has rope)
            qb = h * chunks_per_head + c
            start = qb * QUANT_GROUP_SIZE
            end = start + QUANT_GROUP_SIZE

            fused_nope = fused_fp8[:, :, start:end].view(torch.uint8)
            norope_nope = norope_fp8[:, :, start:end].view(torch.uint8)
            assert torch.equal(fused_nope, norope_nope), (
                f"Nope block (head={h}, chunk={c}) differs between "
                f"fused and no-rope reference"
            )

            fused_s = fused_scale[:, :, qb]
            norope_s = norope_scale[:, :, qb]
            assert torch.equal(fused_s, norope_s), (
                f"Nope scale (head={h}, chunk={c}) differs"
            )
```
**EN:** This pytest case verifies nope dims unchanged. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 nope dims unchanged 的行为。 结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

### Function `test_single_token` (lines 505-534)
```python
@torch.inference_mode()
def test_single_token():
    """Edge case: single token."""
    num_tokens, num_heads, n_groups = 1, 64, 8
    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"

    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.tensor([42], device=device, dtype=torch.long)
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    ref_fp8, ref_scale = reference_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )
    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    assert_dequant_close(ref_fp8, ref_scale, fused_fp8, fused_scale)
```
**EN:** This pytest case verifies single token. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 pytest 用例验证 single token 的行为。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_zero_positions` (lines 537-566)
```python
@torch.inference_mode()
def test_zero_positions():
    """Edge case: all positions are 0."""
    num_tokens, num_heads, n_groups = 16, 64, 8
    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"

    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.zeros(num_tokens, device=device, dtype=torch.long)
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    ref_fp8, ref_scale = reference_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )
    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    assert_dequant_close(ref_fp8, ref_scale, fused_fp8, fused_scale)
```
**EN:** This pytest case verifies zero positions. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 pytest 用例验证 zero positions 的行为。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_large_values` (lines 569-602)
```python
@torch.inference_mode()
def test_large_values():
    """Edge case: values near FP8 saturation to test clamping."""
    num_tokens, num_heads, n_groups = 8, 64, 8
    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"

    # Create inputs with large values that will saturate FP8
    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    o = o * 1000.0  # scale up to force saturation
    positions = torch.randint(
        0, max_pos, (num_tokens,), device=device, dtype=torch.long
    )
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    ref_fp8, ref_scale = reference_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )
    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    assert_dequant_close(ref_fp8, ref_scale, fused_fp8, fused_scale)
```
**EN:** This pytest case verifies large values. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 pytest 用例验证 large values 的行为。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_dequant_numerical_accuracy` (lines 605-658)
```python
@torch.inference_mode()
def test_dequant_numerical_accuracy():
    """Verify dequantized values are close to the original (after inv RoPE)."""
    num_tokens, num_heads, n_groups = 32, 64, 8
    heads_per_group = num_heads // n_groups
    max_pos = 4096
    device = "cuda"
    torch.manual_seed(0)

    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.randint(
        0, max_pos, (num_tokens,), device=device, dtype=torch.long
    )
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    # Get the post-inv-RoPE values (ground truth before quantization)
    o_after_rope = reference_inv_rope(o.clone(), positions, cos_sin_cache)
    d = heads_per_group * HEAD_DIM
    o_after_rope = o_after_rope.view(num_tokens, n_groups, d)

    # Get fused quantized output
    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    # Dequantize: broadcast scale [T, G, S] to [T, G, D] via repeat
    S = d // QUANT_GROUP_SIZE
    scale_expanded = (
        fused_scale.unsqueeze(-1)
        .expand(num_tokens, n_groups, S, QUANT_GROUP_SIZE)
        .reshape(num_tokens, n_groups, d)
    )
    dequant = fused_fp8.float() * scale_expanded

    # Check relative error.
    # FP8 e4m3 with UE8M0 (power-of-two scales that round UP) quantizes more
    # coarsely than optimal scaling. Both paths rotate in fp32, so the bulk
    # of the error comes from UE8M0 quantization itself (~10-12% typical).
    o_gt = o_after_rope.transpose(0, 1).contiguous().transpose(0, 1)
    dequant_contig = dequant.transpose(0, 1).contiguous().transpose(0, 1)

    abs_err = (dequant_contig.float() - o_gt.float()).abs()
    rel_err = abs_err / (o_gt.float().abs().clamp(min=1e-6))
    mean_rel_err = rel_err.mean().item()

    assert mean_rel_err < 0.15, (
        f"Mean relative error too high: {mean_rel_err:.4f} (expected < 0.15)"
    )
```
**EN:** This pytest case verifies dequant numerical accuracy. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 dequant numerical accuracy 的行为。 结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

### Function `_unfused_inv_rope_fp8_quant` (lines 661-706)
```python
def _unfused_inv_rope_fp8_quant(
    o: torch.Tensor,
    positions: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    n_groups: int,
    heads_per_group: int,
    nope_dim: int = NOPE_DIM,
    rope_dim: int = ROPE_DIM,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Unfused path matching deepseek_v4_attention.py:295-310.

    Uses the production CUDA RoPE kernel + per_token_group_quant_fp8.
    """
    from vllm import _custom_ops as ops
    from vllm.model_executor.layers.quantization.utils.fp8_utils import (
        per_token_group_quant_fp8,
    )

    head_dim = o.shape[-1]
    rope_dim_offset = head_dim - rope_dim

    # Step 1: In-place CUDA RoPE (same as production)
    ops.rotary_embedding(
        positions,
        o,
        None,
        head_dim,
        cos_sin_cache,
        False,  # is_neox=False for DeepseekV4 (GPT-J style)
        rope_dim_offset=rope_dim_offset,
        inverse=True,
    )

    # Step 2: Reshape + quant + reshape (same as production)
    T = o.shape[0]
    d = heads_per_group * head_dim
    o = o.view(T, n_groups, -1)
    o_flat = o.transpose(0, 1).contiguous().reshape(-1, d)
    o_fp8, o_scale = per_token_group_quant_fp8(
        o_flat,
        group_size=QUANT_GROUP_SIZE,
        use_ue8m0=True,
    )
    o_fp8 = o_fp8.view(n_groups, T, d).transpose(0, 1)
    o_scale = o_scale.view(n_groups, T, -1).transpose(0, 1)
    return o_fp8, o_scale
```
**EN:** This helper function implements the shared logic for unfused inv rope FP8 quant. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 unfused inv rope FP8 quant 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_einsum_end_to_end` (lines 714-817)
```python
@pytest.mark.parametrize("num_tokens", [1, 7, 32, 128, 1024])
@pytest.mark.parametrize(
    "num_heads,n_groups",
    [(64, 8)],
    ids=["H64_G8"],
)
@torch.inference_mode()
def test_einsum_end_to_end(num_tokens, num_heads, n_groups):
    """End-to-end: fused inv_rope+quant → fp8_einsum must match
    unfused CUDA_rope+quant → fp8_einsum bitwise.

    This catches stride/layout bugs that only manifest when the einsum
    kernel actually consumes the quantized activations.
    """
    from deep_gemm.utils.math import ceil_div

    from vllm.utils.deep_gemm import (
        fp8_einsum,
        per_block_cast_to_fp8,
        transform_sf_into_required_layout,
    )

    heads_per_group = num_heads // n_groups
    d = heads_per_group * HEAD_DIM
    o_lora_rank = 1024
    max_pos = 4096
    device = "cuda"
    torch.manual_seed(0)

    o = torch.randn(
        num_tokens, num_heads, HEAD_DIM, device=device, dtype=torch.bfloat16
    )
    positions = torch.randint(
        0, max_pos, (num_tokens,), device=device, dtype=torch.long
    )
    cos_sin_cache = make_cos_sin_cache(max_pos, device=device)

    # -- Weight quantization (shared between both paths) --
    w = torch.randn(n_groups, o_lora_rank, d, device=device, dtype=torch.bfloat16)
    w_fp8 = torch.empty_like(w, dtype=torch.float8_e4m3fn)
# ... excerpt ...
        n_groups,
        heads_per_group,
    )
    z_ref = torch.empty(
        num_tokens, n_groups, o_lora_rank, device=device, dtype=torch.bfloat16
    )
    fp8_einsum(
        "bhr,hdr->bhd", (ref_fp8, ref_scale), (w_fp8, w_scale_t), z_ref, recipe=recipe
    )

    # -- FUSED path --
    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )
    z_fused = torch.empty(
        num_tokens, n_groups, o_lora_rank, device=device, dtype=torch.bfloat16
    )
    fp8_einsum(
        "bhr,hdr->bhd",
        (fused_fp8, fused_scale),
        (w_fp8, w_scale_t),
        z_fused,
        recipe=recipe,
    )

    # -- Checks --
    # Einsum output: Triton and CUDA both rotate in fp32 now, so diffs
    # come from fp32 ordering and UE8M0 boundary shifts only.
    # Use relative diff (same metric as test_fp8_einsum.py).
    from deep_gemm.testing import calc_diff

    z_diff = calc_diff(z_fused, z_ref)
    assert z_diff < 0.01, (
        f"Einsum output diff too large: {z_diff:.6f} (expected < 0.01)"
    )
```
**EN:** This pytest case verifies einsum end to end. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, num_heads, n_groups. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 einsum end to end 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、num_heads、n_groups 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_with_real_deepseek_v4_rope` (lines 820-908)
```python
@pytest.mark.parametrize("num_tokens", [1, 32, 256])
@torch.inference_mode()
def test_with_real_deepseek_v4_rope(num_tokens, default_vllm_config):
    """Test with real DeepseekV4ScalingRotaryEmbedding (GPT-J style,
    mscale=0, YaRN scaling) matching the production config."""

    num_heads = 64
    n_groups = 8
    heads_per_group = num_heads // n_groups
    device = "cuda"
    torch.manual_seed(0)

    # Build YaRN-scaled cos_sin_cache matching real DeepSeek V3/V4 config
    # (mscale=0 → mscale=1.0, so no magnitude scaling)
    from vllm.model_executor.layers.rotary_embedding.common import (
        yarn_find_correction_range,
        yarn_linear_ramp_mask,
    )

    scaling_factor = 16
    base = 10000.0
    max_pos = 65536
    beta_fast, beta_slow = 32, 1

    pos_freqs = base ** (
        torch.arange(0, ROPE_DIM, 2, dtype=torch.float32, device=device) / ROPE_DIM
    )
    inv_freq_extra = 1.0 / pos_freqs
    inv_freq_interp = 1.0 / (scaling_factor * pos_freqs)
    low, high = yarn_find_correction_range(
        beta_fast, beta_slow, ROPE_DIM, base, max_pos
    )
    mask = 1 - yarn_linear_ramp_mask(low, high, ROPE_DIM // 2, dtype=torch.float32).to(
        device
    )
    inv_freq = inv_freq_interp * (1 - mask) + inv_freq_extra * mask
    t = torch.arange(max_pos * scaling_factor, device=device, dtype=torch.float32)
    freqs = torch.outer(t, inv_freq)
    # mscale=0 → yarn_get_mscale returns 1.0
    cos_sin_cache = torch.cat([freqs.cos(), freqs.sin()], dim=-1)  # fp32
# ... excerpt ...
    )

    o_unfused = o.clone()
    ops.rotary_embedding(
        positions,
        o_unfused,
        None,
        HEAD_DIM,
        cos_sin_cache,
        False,  # is_neox=False (GPT-J style)
        rope_dim_offset=NOPE_DIM,
        inverse=True,
    )
    d = heads_per_group * HEAD_DIM
    T = num_tokens
    o_unfused = o_unfused.view(T, n_groups, d)
    o_flat = o_unfused.transpose(0, 1).contiguous().reshape(-1, d)
    ref_fp8, ref_scale = per_token_group_quant_fp8(
        o_flat,
        group_size=QUANT_GROUP_SIZE,
        use_ue8m0=True,
    )
    ref_fp8 = ref_fp8.view(n_groups, T, d).transpose(0, 1)
    ref_scale = ref_scale.view(n_groups, T, -1).transpose(0, 1)

    # FUSED: use the real YaRN-scaled cos_sin_cache
    fused_fp8, fused_scale = fused_inv_rope_fp8_quant(
        o.clone(),
        positions,
        cos_sin_cache,
        n_groups,
        heads_per_group,
    )

    # Scales must match exactly (same UE8M0 algorithm)
    # Compare via dequant (Triton bf16 rotation may differ from CUDA by 1 ULP)
    assert_dequant_close(
        ref_fp8, ref_scale, fused_fp8, fused_scale, msg="Real DeepSeek V4 rope"
    )
```
**EN:** This pytest case verifies with real deepseek v4 rope. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, default_vllm_config. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 with real deepseek v4 rope 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、default_vllm_config 等 fixture 或输入；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.v1.attention.ops.deepseek_v4_ops -> fused_inv_rope_fp8_quant`
