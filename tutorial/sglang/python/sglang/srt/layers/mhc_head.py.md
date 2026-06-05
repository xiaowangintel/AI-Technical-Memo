# mhc_head.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/mhc_head.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime layer support for the SGLang SRT runtime. It exposes symbols such as `_hc_head_kernel` and `fused_hc_head` and connects them to backend-specific paths such as `Triton`. / 该模块为 SGLang 的 SRT 运行时实现了运行时层支持。它提供了 `_hc_head_kernel` 和 `fused_hc_head` 等符号，并把这些符号连接到 `Triton` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Module imports and dependency wiring
```python
"""Fused triton kernel for the DSV4 hc_head LM-head mixer.

Reference torch implementation (deepseek_v4.py DeepseekV4Model.hc_head):

    shape, dtype = x.size(), x.dtype
    x = x.flatten(1).float()
    rsqrt = torch.rsqrt(x.square().mean(-1, keepdim=True) + norm_eps)
    mixes = F.linear(x, hc_fn) * rsqrt
    pre = torch.sigmoid(mixes * hc_scale + hc_base) + hc_eps
    y = torch.sum(pre.unsqueeze(-1) * x.view(shape), dim=1)
    return y.to(dtype)

Shapes (DSV4-Pro, hc_mult=4, hidden_size=7168 typical):
    x      : (T, hc_mult, hidden_size)            bf16
    hc_fn  : (hc_mult, hc_mult * hidden_size)     fp32
    scale  : (1,)                                 fp32
    base   : (hc_mult,)                           fp32
    out y  : (T, hidden_size)                     bf16

This is a one-shot LM-head op (fires once per forward on the last PP rank), so
we use a 1-CTA-per-token design that does two passes over x without split-K.
"""

from __future__ import annotations

import torch
import triton
import triton.language as tl
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `torch`, `triton`, and `triton.language`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`torch`、`triton` 以及 `triton.language`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 31-89: Internal helper `_hc_head_kernel`
```python
@triton.jit
def _hc_head_kernel(
    x_ptr,
    fn_ptr,
    scale_ptr,
    base_ptr,
    y_ptr,
    hidden_size: tl.constexpr,
    HC_MULT: tl.constexpr,
    K_TOTAL: tl.constexpr,
    BLOCK_K: tl.constexpr,
    BLOCK_D: tl.constexpr,
    norm_eps: tl.constexpr,
    hc_eps: tl.constexpr,
):
    pid = tl.program_id(0).to(tl.int64)

    # ---------- Pass 1: sum_sq over flattened K dim, plus hc_mult inner products ----------
    sumsq = tl.zeros((), dtype=tl.float32)
    mix = tl.zeros((HC_MULT,), dtype=tl.float32)

    x_row = x_ptr + pid * K_TOTAL
    m_idx = tl.arange(0, HC_MULT)

    for k_off in tl.range(0, K_TOTAL, BLOCK_K):
        k_offs = k_off + tl.arange(0, BLOCK_K)
        k_mask = k_offs < K_TOTAL
        x_tile = tl.load(x_row + k_offs, mask=k_mask, other=0.0).to(tl.float32)

        sumsq += tl.sum(x_tile * x_tile, axis=0)

        fn_offs = m_idx[:, None] * K_TOTAL + k_offs[None, :]
        fn_mask = (m_idx[:, None] < HC_MULT) & k_mask[None, :]
        fn_tile = tl.load(fn_ptr + fn_offs, mask=fn_mask, other=0.0)
        mix += tl.sum(fn_tile * x_tile[None, :], axis=1)

    rsqrt = tl.rsqrt(sumsq / K_TOTAL + norm_eps)
    scale_v = tl.load(scale_ptr).to(tl.float32)
    base_v = tl.load(base_ptr + m_idx).to(tl.float32)

    # pre[m] = sigmoid(mix[m] * rsqrt * scale + base[m]) + hc_eps
    pre = tl.sigmoid(mix * rsqrt * scale_v + base_v) + hc_eps

    # ---------- Pass 2: y[d] = sum_m pre[m] * x[m, d]  for d in range(hidden_size) ----------
    y_row = y_ptr + pid * hidden_size

    for d_off in tl.range(0, hidden_size, BLOCK_D):
        d_offs = d_off + tl.arange(0, BLOCK_D)
        d_mask = d_offs < hidden_size

        x_offs = m_idx[:, None] * hidden_size + d_offs[None, :]
        x_mask = (m_idx[:, None] < HC_MULT) & d_mask[None, :]
        x_block = tl.load(x_row + x_offs, mask=x_mask, other=0.0).to(tl.float32)

        y_block = tl.sum(pre[:, None] * x_block, axis=0)

        tl.store(y_row + d_offs, y_block.to(y_ptr.dtype.element_ty), mask=d_mask)
```
**EN:** This block defines `_hc_head_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id.to`, `tl.zeros`, `tl.arange`, `tl.range`, and `tl.rsqrt`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `sumsq`, `mix`, `x_row`, and `m_idx` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_hc_head_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id.to`、`tl.zeros`、`tl.arange`、`tl.range` 以及 `tl.rsqrt`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`sumsq`、`mix`、`x_row` 以及 `m_idx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 90-151: Function `fused_hc_head` and its core logic
```python
def fused_hc_head(
    x: torch.Tensor,
    hc_fn: torch.Tensor,
    hc_scale: torch.Tensor,
    hc_base: torch.Tensor,
    norm_eps: float,
    hc_eps: float,
) -> torch.Tensor:
    """Fused (RMSNorm + Linear + Sigmoid-gate + weighted-sum) for the DSV4 hc_head.

    Args:
        x         : (T, hc_mult, hidden_size) bf16/fp16, must be contiguous
        hc_fn     : (hc_mult, hc_mult * hidden_size) fp32, contiguous
        hc_scale  : (1,) fp32 scalar
        hc_base   : (hc_mult,) fp32
        norm_eps  : RMS epsilon
        hc_eps    : additive epsilon after sigmoid

    Returns:
        y : (T, hidden_size) same dtype as x
    """
    assert x.is_contiguous(), "x must be contiguous"
    assert hc_fn.is_contiguous(), "hc_fn must be contiguous"
    assert hc_scale.dtype == torch.float32 and hc_base.dtype == torch.float32
    assert hc_fn.dtype == torch.float32
    assert x.dim() == 3, f"x must be 3D (T, hc_mult, hidden_size), got {x.shape}"

    T, hc_mult, hidden_size = x.shape
    assert hc_fn.shape == (hc_mult, hc_mult * hidden_size), (
        f"hc_fn shape {hc_fn.shape} does not match (hc_mult={hc_mult}, "
        f"hc_mult*hidden_size={hc_mult * hidden_size})"
    )
    assert hc_base.shape == (hc_mult,)
    assert hc_scale.numel() == 1

    y = torch.empty((T, hidden_size), dtype=x.dtype, device=x.device)

    if T == 0:
        return y

    BLOCK_K = 512
    BLOCK_D = 512

    hc_mult_pow2 = max(1, triton.next_power_of_2(hc_mult))

    grid = (T,)
    _hc_head_kernel[grid](
        x,
        hc_fn,
        hc_scale,
        hc_base,
        y,
        hidden_size=hidden_size,
        HC_MULT=hc_mult_pow2,
        K_TOTAL=hc_mult * hidden_size,
        BLOCK_K=BLOCK_K,
        BLOCK_D=BLOCK_D,
        norm_eps=norm_eps,
        hc_eps=hc_eps,
        num_warps=4,
    )
    return y
```
**EN:** This block defines `fused_hc_head` and contains the main logic for this step. It mainly invokes `x.is_contiguous`, `hc_fn.is_contiguous`, `torch.empty`, `max`, and `_hc_head_kernel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `T`, `hc_mult`, `hidden_size`, `y`, and `BLOCK_K` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_hc_head`，并承载这一阶段的核心逻辑。 它主要调用 `x.is_contiguous`、`hc_fn.is_contiguous`、`torch.empty`、`max` 以及 `_hc_head_kernel`，说明该流程会编排底层辅助函数或计算内核。 像 `T`、`hc_mult`、`hidden_size`、`y` 以及 `BLOCK_K` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_hc_head_kernel` and `fused_hc_head`. / **主要符号**：核心入口包括 `_hc_head_kernel` 和 `fused_hc_head`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。
- **Acceleration stack**: The module depends on external acceleration libraries and wraps them behind Python entry points. / **加速栈**：该模块依赖外部加速库，并通过 Python 入口对它们进行封装。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations` / **标准库**：`__future__.annotations`
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
