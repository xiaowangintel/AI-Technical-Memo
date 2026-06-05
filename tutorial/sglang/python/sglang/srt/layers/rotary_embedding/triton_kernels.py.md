# triton_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rotary_embedding/triton_kernels.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements rotary position embedding logic with embedding lookup and preprocessing for the SGLang SRT runtime. It exposes symbols such as `_triton_mrope_forward_fused`, `triton_mrope_fused`, `_triton_ernie45_rope_qk_fused`, and `triton_ernie45_rope_fused_inplace` and connects them to backend-specific paths such as `Triton`. / 该模块为 SGLang 的 SRT 运行时实现了旋转位置编码逻辑，并结合嵌入查找与预处理。它提供了 `_triton_mrope_forward_fused`、`triton_mrope_fused`、`_triton_ernie45_rope_qk_fused` 以及 `triton_ernie45_rope_fused_inplace` 等符号，并把这些符号连接到 `Triton` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Module imports and dependency wiring
```python
"""Triton JIT kernels for multimodal rotary positional embeddings."""

from __future__ import annotations

from typing import List

import torch
import triton
import triton.language as tl
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `typing.List`, `torch`, `triton`, and `triton.language`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`typing.List`、`torch`、`triton` 以及 `triton.language`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 12-110: Internal helper `_triton_mrope_forward_fused`
```python
@triton.jit
def _triton_mrope_forward_fused(
    q_ptr,
    k_ptr,
    cos_sin_cache_ptr,
    positions_ptr,
    q_stride,
    k_stride,
    positions_stride,
    n_qh: tl.constexpr,
    n_kh: tl.constexpr,
    hd: tl.constexpr,
    rd: tl.constexpr,
    pad_n_qh: tl.constexpr,
    pad_n_kh: tl.constexpr,
    pad_hd: tl.constexpr,
    mrope_section_t: tl.constexpr,
    mrope_section_h: tl.constexpr,
    mrope_section_w: tl.constexpr,
    is_interleaved: tl.constexpr,
    is_interleaved_glm: tl.constexpr,
    is_neox_style: tl.constexpr,
    axis_map_ptr,
):
    pid = tl.program_id(0)
    q_ptr = q_ptr + pid * q_stride
    k_ptr = k_ptr + pid * k_stride
    half_rd = rd // 2
    t = tl.load(positions_ptr + 0 * positions_stride + pid)
    h = tl.load(positions_ptr + 1 * positions_stride + pid)
    w = tl.load(positions_ptr + 2 * positions_stride + pid)
    t_cos = cos_sin_cache_ptr + t * rd
    h_cos = cos_sin_cache_ptr + h * rd
    w_cos = cos_sin_cache_ptr + w * rd
    t_sin = t_cos + half_rd
    h_sin = h_cos + half_rd
    w_sin = w_cos + half_rd
    cos_offsets = tl.arange(0, pad_hd // 2)
    if is_interleaved:
        if is_interleaved_glm:
            axes = tl.load(axis_map_ptr + cos_offsets, mask=cos_offsets < (pad_hd // 2))
            t_mask = axes == 0
            h_mask = axes == 1
            w_mask = axes == 2
        else:
            h_mask = ((cos_offsets % 3) == 1) & (cos_offsets <= 3 * mrope_section_h)
            w_mask = ((cos_offsets % 3) == 2) & (cos_offsets <= 3 * mrope_section_w)
            t_mask = ~(h_mask | w_mask)
    else:
        t_end = mrope_section_t
        h_end = t_end + mrope_section_h
        t_mask = cos_offsets < mrope_section_t
        h_mask = (t_end <= cos_offsets) & (cos_offsets < h_end)
        w_mask = (h_end <= cos_offsets) & (cos_offsets < half_rd)
    t_cos_row = tl.load(t_cos + cos_offsets, mask=t_mask, other=0)
    t_sin_row = tl.load(t_sin + cos_offsets, mask=t_mask, other=0)
    h_cos_row = tl.load(h_cos + cos_offsets, mask=h_mask, other=0)
    h_sin_row = tl.load(h_sin + cos_offsets, mask=h_mask, other=0)
    w_cos_row = tl.load(w_cos + cos_offsets, mask=w_mask, other=0)
    w_sin_row = tl.load(w_sin + cos_offsets, mask=w_mask, other=0)
    cos_row = t_cos_row + h_cos_row + w_cos_row
    sin_row = t_sin_row + h_sin_row + w_sin_row
    if is_neox_style:
        fhq = tl.arange(0, pad_n_qh)[:, None] * hd + tl.arange(0, pad_hd // 2)[None, :]
        fhk = tl.arange(0, pad_n_kh)[:, None] * hd + tl.arange(0, pad_hd // 2)[None, :]
        fqm = (tl.arange(0, pad_n_qh)[:, None] < n_qh) & (
            tl.arange(0, pad_hd // 2)[None, :] < rd // 2
        )
        fkm = (tl.arange(0, pad_n_kh)[:, None] < n_kh) & (
            tl.arange(0, pad_hd // 2)[None, :] < rd // 2
        )
        q1 = tl.load(q_ptr + fhq, mask=fqm, other=0).to(sin_row.dtype)
        k1 = tl.load(k_ptr + fhk, mask=fkm, other=0).to(sin_row.dtype)
        shq = fhq + (rd // 2)
        shk = fhk + (rd // 2)
        q2 = tl.load(q_ptr + shq, mask=fqm, other=0).to(sin_row.dtype)
        k2 = tl.load(k_ptr + shk, mask=fkm, other=0).to(sin_row.dtype)
        tl.store(q_ptr + fhq, q1 * cos_row - q2 * sin_row, mask=fqm)
        tl.store(q_ptr + shq, q2 * cos_row + q1 * sin_row, mask=fqm)
        tl.store(k_ptr + fhk, k1 * cos_row - k2 * sin_row, mask=fkm)
        tl.store(k_ptr + shk, k2 * cos_row + k1 * sin_row, mask=fkm)
    else:
        bq = tl.arange(0, pad_n_qh)[:, None] * hd
        bk = tl.arange(0, pad_n_kh)[:, None] * hd
        ei = 2 * tl.arange(0, pad_hd // 2)[None, :]
        oi = ei + 1
        im = tl.arange(0, pad_hd // 2)[None, :] < (rd // 2)
        qm = (tl.arange(0, pad_n_qh)[:, None] < n_qh) & im
        km = (tl.arange(0, pad_n_kh)[:, None] < n_kh) & im
        qe = tl.load(q_ptr + bq + ei, mask=qm, other=0).to(sin_row.dtype)
        qo = tl.load(q_ptr + bq + oi, mask=qm, other=0).to(sin_row.dtype)
        ke = tl.load(k_ptr + bk + ei, mask=km, other=0).to(sin_row.dtype)
        ko = tl.load(k_ptr + bk + oi, mask=km, other=0).to(sin_row.dtype)
        tl.store(q_ptr + bq + ei, qe * cos_row - qo * sin_row, mask=qm)
        tl.store(q_ptr + bq + oi, qo * cos_row + qe * sin_row, mask=qm)
        tl.store(k_ptr + bk + ei, ke * cos_row - ko * sin_row, mask=km)
        tl.store(k_ptr + bk + oi, ko * cos_row + ke * sin_row, mask=km)
```
**EN:** This block defines `_triton_mrope_forward_fused` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, `tl.arange`, `tl.load.to`, and `tl.store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `q_ptr`, `k_ptr`, `half_rd`, and `t` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_triton_mrope_forward_fused`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load`、`tl.arange`、`tl.load.to` 以及 `tl.store`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`q_ptr`、`k_ptr`、`half_rd` 以及 `t` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 111-155: Function `triton_mrope_fused` and its core logic
```python
def triton_mrope_fused(
    q: torch.Tensor,
    k: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    positions: torch.Tensor,
    mrope_section: List[int],
    head_size: int,
    rotary_dim: int,
    mrope_interleaved: bool,
    mrope_interleaved_glm: bool,
    is_neox_style: bool,
    axis_map: torch.Tensor,
) -> None:
    num_tokens, n_q_dim = q.shape
    n_k_dim = k.shape[1]
    n_qh = n_q_dim // head_size
    n_kh = n_k_dim // head_size
    pad_n_qh = triton.next_power_of_2(n_qh)
    pad_n_kh = triton.next_power_of_2(n_kh)
    pad_hd = triton.next_power_of_2(head_size)
    _triton_mrope_forward_fused[(num_tokens,)](
        q,
        k,
        cos_sin_cache,
        positions,
        q.stride(0),
        k.stride(0),
        positions.stride(0),
        n_qh,
        n_kh,
        head_size,
        rotary_dim,
        pad_n_qh,
        pad_n_kh,
        pad_hd,
        mrope_section[0],
        mrope_section[1],
        mrope_section[2],
        mrope_interleaved,
        mrope_interleaved_glm,
        is_neox_style,
        axis_map,
    )
```
**EN:** This block defines `triton_mrope_fused` and contains the main logic for this step. It mainly invokes `triton.next_power_of_2`, `_triton_mrope_forward_fused`, `q.stride`, `k.stride`, and `positions.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `n_q_dim`, `n_k_dim`, `n_qh`, and `n_kh` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `triton_mrope_fused`，并承载这一阶段的核心逻辑。 它主要调用 `triton.next_power_of_2`、`_triton_mrope_forward_fused`、`q.stride`、`k.stride` 以及 `positions.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`n_q_dim`、`n_k_dim`、`n_qh` 以及 `n_kh` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 156-230: Internal helper `_triton_ernie45_rope_qk_fused`
```python
@triton.jit
def _triton_ernie45_rope_qk_fused(
    q_ptr,
    k_ptr,
    cos_sin_cache_ptr,
    positions_ptr,
    q_stride0: tl.constexpr,
    k_stride0: tl.constexpr,
    pos_stride0: tl.constexpr,
    n_qh: tl.constexpr,
    n_kh: tl.constexpr,
    hd: tl.constexpr,
    rd: tl.constexpr,
    pad_n_qh: tl.constexpr,
    pad_n_kh: tl.constexpr,
    pad_hd: tl.constexpr,
    section_hw: tl.constexpr,
    is_neox_style: tl.constexpr,
):
    pid = tl.program_id(0)
    q_ptr = q_ptr + pid * q_stride0
    k_ptr = k_ptr + pid * k_stride0
    half_rd = rd // 2
    tpos = tl.load(positions_ptr + 0 * pos_stride0 + pid).to(tl.int32)
    hpos = tl.load(positions_ptr + 1 * pos_stride0 + pid).to(tl.int32)
    wpos = tl.load(positions_ptr + 2 * pos_stride0 + pid).to(tl.int32)
    ridx = tl.arange(0, pad_hd // 2)
    rmask = ridx < half_rd
    use_hw = ridx < section_hw
    use_h = (ridx & 1) == 0
    pos = tl.where(use_hw, tl.where(use_h, hpos, wpos), tpos)
    cos = tl.load(cos_sin_cache_ptr + pos * rd + ridx, mask=rmask, other=0.0)
    sin = tl.load(
        cos_sin_cache_ptr + pos * rd + (ridx + half_rd), mask=rmask, other=0.0
    )
    if is_neox_style:
        qh = tl.arange(0, pad_n_qh)[:, None]
        kh = tl.arange(0, pad_n_kh)[:, None]
        d = tl.arange(0, pad_hd // 2)[None, :]
        qm = (qh < n_qh) & (d < half_rd)
        km = (kh < n_kh) & (d < half_rd)
        qo0 = qh * hd + d
        ko0 = kh * hd + d
        qo1 = qo0 + half_rd
        ko1 = ko0 + half_rd
        q0 = tl.load(q_ptr + qo0, mask=qm, other=0.0).to(cos.dtype)
        q1 = tl.load(q_ptr + qo1, mask=qm, other=0.0).to(cos.dtype)
        k0 = tl.load(k_ptr + ko0, mask=km, other=0.0).to(cos.dtype)
        k1 = tl.load(k_ptr + ko1, mask=km, other=0.0).to(cos.dtype)
        cb = cos[None, :]
        sb = sin[None, :]
        tl.store(q_ptr + qo0, q0 * cb - q1 * sb, mask=qm)
        tl.store(q_ptr + qo1, q1 * cb + q0 * sb, mask=qm)
        tl.store(k_ptr + ko0, k0 * cb - k1 * sb, mask=km)
        tl.store(k_ptr + ko1, k1 * cb + k0 * sb, mask=km)
    else:
        qh = tl.arange(0, pad_n_qh)[:, None]
        kh = tl.arange(0, pad_n_kh)[:, None]
        p = tl.arange(0, pad_hd // 2)[None, :]
        qm = (qh < n_qh) & (p < half_rd)
        km = (kh < n_kh) & (p < half_rd)
        even = 2 * p
        odd = even + 1
        qe = tl.load(q_ptr + qh * hd + even, mask=qm, other=0.0).to(cos.dtype)
        qo = tl.load(q_ptr + qh * hd + odd, mask=qm, other=0.0).to(cos.dtype)
        ke = tl.load(k_ptr + kh * hd + even, mask=km, other=0.0).to(cos.dtype)
        ko = tl.load(k_ptr + kh * hd + odd, mask=km, other=0.0).to(cos.dtype)
        cb = cos[None, :]
        sb = sin[None, :]
        tl.store(q_ptr + qh * hd + even, qe * cb - qo * sb, mask=qm)
        tl.store(q_ptr + qh * hd + odd, qo * cb + qe * sb, mask=qm)
        tl.store(k_ptr + kh * hd + even, ke * cb - ko * sb, mask=km)
        tl.store(k_ptr + kh * hd + odd, ko * cb + ke * sb, mask=km)
```
**EN:** This block defines `_triton_ernie45_rope_qk_fused` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load.to`, `tl.arange`, `tl.where`, and `tl.load`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `q_ptr`, `k_ptr`, `half_rd`, and `tpos` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_triton_ernie45_rope_qk_fused`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load.to`、`tl.arange`、`tl.where` 以及 `tl.load`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`q_ptr`、`k_ptr`、`half_rd` 以及 `tpos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 231-272: Function `triton_ernie45_rope_fused_inplace` and its core logic
```python
def triton_ernie45_rope_fused_inplace(
    q: torch.Tensor,
    k: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    positions: torch.Tensor,
    mrope_section: list,
    head_size: int,
    rotary_dim: int,
    is_neox_style: bool,
) -> None:
    num_tokens = q.shape[0]
    n_qh = q.shape[1] // head_size
    n_kh = k.shape[1] // head_size
    rd = rotary_dim
    section_h, section_w, section_t = mrope_section
    assert section_h == section_w, "Ernie4.5 layout assumes section_h == section_w"
    assert section_h + section_w + section_t == rd // 2
    if cos_sin_cache.dtype != q.dtype or cos_sin_cache.device != q.device:
        cos_sin_cache = cos_sin_cache.to(device=q.device, dtype=q.dtype)
    pad_n_qh = triton.next_power_of_2(n_qh)
    pad_n_kh = triton.next_power_of_2(n_kh)
    pad_hd = triton.next_power_of_2(head_size)
    num_warps = 4 if (pad_n_qh * pad_hd) <= 8192 else 8
    _triton_ernie45_rope_qk_fused[(num_tokens,)](
        q,
        k,
        cos_sin_cache,
        positions,
        q.stride(0),
        k.stride(0),
        positions.stride(0),
        n_qh=n_qh,
        n_kh=n_kh,
        hd=head_size,
        rd=rd,
        pad_n_qh=pad_n_qh,
        pad_n_kh=pad_n_kh,
        pad_hd=pad_hd,
        section_hw=section_h + section_w,
        is_neox_style=is_neox_style,
        num_warps=num_warps,
    )
```
**EN:** This block defines `triton_ernie45_rope_fused_inplace` and contains the main logic for this step. It mainly invokes `triton.next_power_of_2`, `_triton_ernie45_rope_qk_fused`, `cos_sin_cache.to`, `q.stride`, and `k.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `n_qh`, `n_kh`, `rd`, and `section_h` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `triton_ernie45_rope_fused_inplace`，并承载这一阶段的核心逻辑。 它主要调用 `triton.next_power_of_2`、`_triton_ernie45_rope_qk_fused`、`cos_sin_cache.to`、`q.stride` 以及 `k.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`n_qh`、`n_kh`、`rd` 以及 `section_h` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_triton_mrope_forward_fused`, `triton_mrope_fused`, `_triton_ernie45_rope_qk_fused`, and `triton_ernie45_rope_fused_inplace`. / **主要符号**：核心入口包括 `_triton_mrope_forward_fused`、`triton_mrope_fused`、`_triton_ernie45_rope_qk_fused` 以及 `triton_ernie45_rope_fused_inplace`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。
- **Embedding pipeline**: Describes how IDs or features are mapped into model-space tensors. / **嵌入流水线**：说明 ID 或特征如何映射到模型空间张量。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations` and `typing.List` / **标准库**：`__future__.annotations` 和 `typing.List`
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
