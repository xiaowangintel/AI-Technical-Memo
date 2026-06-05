# mla_kv_pack_quantize_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/mla_kv_pack_quantize_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"Fused ``cat(k_nope, broadcast(k_pe)) + FP8 quantize`` for K and ``FP8 quantize`` for V.". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""Fused ``cat(k_nope, broadcast(k_pe)) + FP8 quantize`` for K and ``FP8 quantize`` for V.”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
"""Fused ``cat(k_nope, broadcast(k_pe)) + FP8 quantize`` for K and ``FP8 quantize`` for V.

Dispatches between two Triton kernels per batch size; see ``_pick_kernel``.
"""

from __future__ import annotations

from typing import Optional, Tuple

import torch
import triton
import triton.language as tl

from sglang.jit_kernel.utils import is_arch_support_pdl


@triton.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-80: Function `_v0_kernel`
```python
def _v0_kernel(
    k_nope_ptr,
    k_pe_ptr,
    v_ptr,
    k_out_ptr,
    v_out_ptr,
    k_scale_inv,
    v_scale_inv,
    s_total,
    k_nope_stride_t,
    k_nope_stride_h,
    k_pe_stride_t,
    v_stride_t,
    v_stride_h,
    k_out_stride_t,
    k_out_stride_h,
    v_out_stride_t,
    v_out_stride_h,
    QK_NOPE: tl.constexpr,
    QK_ROPE: tl.constexpr,
    V_HEAD: tl.constexpr,
    FP8_DTYPE: tl.constexpr,
    BLOCK_S: tl.constexpr,
    ENABLE_PDL: tl.constexpr,
):
    pid_s = tl.program_id(0)
    pid_h = tl.program_id(1)
    t_idx = pid_s * BLOCK_S + tl.arange(0, BLOCK_S)
    t_mask = t_idx < s_total
    nope_idx = tl.arange(0, QK_NOPE)
    rope_idx = tl.arange(0, QK_ROPE)
    v_idx = tl.arange(0, V_HEAD)
    if ENABLE_PDL:
        tl.extra.cuda.gdc_wait()
    nope_off = (
        t_idx[:, None] * k_nope_stride_t + pid_h * k_nope_stride_h + nope_idx[None, :]
    )
    k_nope = tl.load(k_nope_ptr + nope_off, mask=t_mask[:, None])
    pe_off = t_idx[:, None] * k_pe_stride_t + rope_idx[None, :]
    k_pe = tl.load(k_pe_ptr + pe_off, mask=t_mask[:, None])
    v_off = t_idx[:, None] * v_stride_t + pid_h * v_stride_h + v_idx[None, :]
    v = tl.load(v_ptr + v_off, mask=t_mask[:, None])
    k_nope_fp8 = (k_nope.to(tl.float32) * k_scale_inv).to(FP8_DTYPE)
    k_pe_fp8 = (k_pe.to(tl.float32) * k_scale_inv).to(FP8_DTYPE)
    v_fp8 = (v.to(tl.float32) * v_scale_inv).to(FP8_DTYPE)
    k_out_base = t_idx[:, None] * k_out_stride_t + pid_h * k_out_stride_h
    tl.store(
        k_out_ptr + k_out_base + nope_idx[None, :], k_nope_fp8, mask=t_mask[:, None]
    )
    tl.store(
        k_out_ptr + k_out_base + QK_NOPE + rope_idx[None, :],
        k_pe_fp8,
        mask=t_mask[:, None],
    )
    v_out_off = (
        t_idx[:, None] * v_out_stride_t + pid_h * v_out_stride_h + v_idx[None, :]
    )
    tl.store(v_out_ptr + v_out_off, v_fp8, mask=t_mask[:, None])
    if ENABLE_PDL:
        tl.extra.cuda.gdc_launch_dependents()


@triton.jit
```
**EN:** This block defines `_v0_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_v0_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 81-147: Function `_v1_flat_kernel`
```python
def _v1_flat_kernel(
    k_nope_ptr,
    k_pe_ptr,
    v_ptr,
    k_out_ptr,
    v_out_ptr,
    k_scale_inv,
    v_scale_inv,
    s_total,
    num_heads,
    k_nope_stride_t,
    k_nope_stride_h,
    k_pe_stride_t,
    v_stride_t,
    v_stride_h,
    k_out_stride_t,
    k_out_stride_h,
    v_out_stride_t,
    v_out_stride_h,
    QK_NOPE: tl.constexpr,
    QK_ROPE: tl.constexpr,
    V_HEAD: tl.constexpr,
    FP8_DTYPE: tl.constexpr,
    BLOCK: tl.constexpr,
    ENABLE_PDL: tl.constexpr,
):
    if ENABLE_PDL:
        tl.extra.cuda.gdc_wait()
    pid = tl.program_id(0)
    pair_idx = pid * BLOCK + tl.arange(0, BLOCK)
    total = s_total * num_heads
    mask = pair_idx < total
    t_idx = pair_idx // num_heads
    h_idx = pair_idx % num_heads
    nope_idx = tl.arange(0, QK_NOPE)
    rope_idx = tl.arange(0, QK_ROPE)
    v_idx_ = tl.arange(0, V_HEAD)
    nope_off = (
        t_idx[:, None] * k_nope_stride_t
        + h_idx[:, None] * k_nope_stride_h
        + nope_idx[None, :]
    )
    k_nope = tl.load(k_nope_ptr + nope_off, mask=mask[:, None])
    pe_off = t_idx[:, None] * k_pe_stride_t + rope_idx[None, :]
    k_pe = tl.load(k_pe_ptr + pe_off, mask=mask[:, None])
    v_off = t_idx[:, None] * v_stride_t + h_idx[:, None] * v_stride_h + v_idx_[None, :]
    v = tl.load(v_ptr + v_off, mask=mask[:, None])
    k_nope_fp8 = (k_nope.to(tl.float32) * k_scale_inv).to(FP8_DTYPE)
    k_pe_fp8 = (k_pe.to(tl.float32) * k_scale_inv).to(FP8_DTYPE)
    v_fp8 = (v.to(tl.float32) * v_scale_inv).to(FP8_DTYPE)
    k_out_base = t_idx[:, None] * k_out_stride_t + h_idx[:, None] * k_out_stride_h
    tl.store(k_out_ptr + k_out_base + nope_idx[None, :], k_nope_fp8, mask=mask[:, None])
    tl.store(
        k_out_ptr + k_out_base + QK_NOPE + rope_idx[None, :],
        k_pe_fp8,
        mask=mask[:, None],
    )
    v_out_off = (
        t_idx[:, None] * v_out_stride_t
        + h_idx[:, None] * v_out_stride_h
        + v_idx_[None, :]
    )
    tl.store(v_out_ptr + v_out_off, v_fp8, mask=mask[:, None])
    if ENABLE_PDL:
        tl.extra.cuda.gdc_launch_dependents()
```
**EN:** This block defines `_v1_flat_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_v1_flat_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 148-170: Function `_pick_kernel`
```python
def _pick_kernel(s: int, num_heads: int) -> Tuple[str, dict]:
    """Tuned on GB300, DSv3 dims, BF16 -> FP8 e4m3."""
    if s <= 2:
        # Launch-overhead-bound; tighter (BLOCK_S, num_warps) just adds warp
        # setup cost without paying back in per-CTA work.
        return "v0", {"BLOCK_S": 1, "num_warps": 1, "num_stages": 2}
    if s <= 16:
        return "v0", {"BLOCK_S": 4, "num_warps": 2, "num_stages": 3}
    if s <= 32:
        return "v1_flat", {"BLOCK": 8, "num_warps": 8, "num_stages": 2}
    if s <= 192:
        return "v1_flat", {"BLOCK": 16, "num_warps": 8, "num_stages": 3}
    if s <= 1536:
        return "v0", {"BLOCK_S": 16, "num_warps": 4, "num_stages": 3}
    return "v1_flat", {"BLOCK": 16, "num_warps": 8, "num_stages": 3}


_FP8_DTYPE_MAP = {
    torch.float8_e4m3fn: tl.float8e4nv,
    torch.float8_e5m2: tl.float8e5,
}
```
**EN:** This block defines `_pick_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_pick_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 171-295: Function `mla_kv_pack_quantize_fp8`
```python
def mla_kv_pack_quantize_fp8(
    k_nope: torch.Tensor,
    k_pe: torch.Tensor,
    v: torch.Tensor,
    k_scale_inv: float = 1.0,
    v_scale_inv: float = 1.0,
    k_out: Optional[torch.Tensor] = None,
    v_out: Optional[torch.Tensor] = None,
    fp8_dtype: torch.dtype = torch.float8_e4m3fn,
    enable_pdl: Optional[bool] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Fused ``cat(k_nope, broadcast k_pe) + FP8 quantize`` for K and ``FP8 quantize`` for V.

    Shapes: ``k_nope [s, h, qk_nope]``, ``k_pe [s, 1, qk_rope]`` or ``[s, qk_rope]``,
    ``v [s, h, v_head]``. Returns ``(k_fp8 [s, h, qk_nope + qk_rope], v_fp8 [s, h, v_head])``.
    Strided views are supported as long as the inner dim is contiguous.
    """
    assert k_nope.dtype in (
        torch.bfloat16,
        torch.float16,
    ), f"k_nope must be bf16/fp16, got {k_nope.dtype}"
    assert (
        k_pe.dtype == k_nope.dtype and v.dtype == k_nope.dtype
    ), "k_nope, k_pe, v must share dtype"
    assert fp8_dtype in (torch.float8_e4m3fn, torch.float8_e5m2)

    s, num_heads, qk_nope = k_nope.shape
    qk_rope = k_pe.shape[-1]
    v_head = v.shape[-1]

    assert (
        v.shape[0] == s and v.shape[1] == num_heads
    ), f"v shape {tuple(v.shape)} mismatches k_nope {tuple(k_nope.shape)}"
    assert (
        k_pe.shape[0] == s
    ), f"k_pe first dim {k_pe.shape[0]} mismatches k_nope first dim {s}"
    assert k_nope.stride(-1) == 1, "k_nope must have stride-1 inner dim"
    assert v.stride(-1) == 1, "v must have stride-1 inner dim"
    assert k_pe.stride(-1) == 1, "k_pe must have stride-1 inner dim"

    if k_pe.dim() == 3:
        assert k_pe.shape[1] == 1, f"k_pe head dim must be 1, got {k_pe.shape[1]}"
        k_pe_2d = k_pe.squeeze(1)
    else:
        k_pe_2d = k_pe

    if k_out is None:
        k_out = torch.empty(
            (s, num_heads, qk_nope + qk_rope), dtype=fp8_dtype, device=k_nope.device
        )
    if v_out is None:
        v_out = torch.empty((s, num_heads, v_head), dtype=fp8_dtype, device=v.device)

    if enable_pdl is None:
        enable_pdl = is_arch_support_pdl()

    fp8_tl_dtype = _FP8_DTYPE_MAP[fp8_dtype]
    kernel_choice, cfg = _pick_kernel(s, num_heads)
    extra = {"launch_pdl": True} if enable_pdl else {}

    if kernel_choice == "v0":
        block_s = cfg["BLOCK_S"]
        grid = (triton.cdiv(s, block_s), num_heads)
        _v0_kernel[grid](
            k_nope,
            k_pe_2d,
            v,
            k_out,
            v_out,
            float(k_scale_inv),
            float(v_scale_inv),
            s,
            k_nope.stride(0),
            k_nope.stride(1),
            k_pe_2d.stride(0),
            v.stride(0),
            v.stride(1),
            k_out.stride(0),
            k_out.stride(1),
            v_out.stride(0),
# ...
```
**EN:** This block defines `mla_kv_pack_quantize_fp8`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `mla_kv_pack_quantize_fp8`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> Optional`
- `torch`
- `triton`
- `triton.language as tl`
- `sglang.jit_kernel.utils -> is_arch_support_pdl`
