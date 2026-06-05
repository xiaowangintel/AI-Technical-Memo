# gemma4_fused_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/gemma4_fused_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `_gemma_rmsnorm_residual_kernel`, `gemma_rmsnorm_residual_scalar`, `_gemma_dual_rmsnorm_residual_kernel`, and `_gemma_qkv_rmsnorm_kernel` and connects them to backend-specific paths such as `CUDA`, `Triton`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成。它提供了 `_gemma_rmsnorm_residual_kernel`、`gemma_rmsnorm_residual_scalar`、`_gemma_dual_rmsnorm_residual_kernel` 以及 `_gemma_qkv_rmsnorm_kernel` 等符号，并把这些符号连接到 `CUDA`、`Triton` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module imports and dependency wiring
```python
"""Fused triton kernels for Gemma4 decoder layer operations.

Fuses standard RMSNorm + residual-add (+ optional scalar multiply) into
a single kernel pass to reduce kernel launch overhead.
"""

from typing import Optional

import torch
import triton
import triton.language as tl
```
**EN:** This section prepares the module namespace. It imports `typing.Optional`, `torch`, `triton`, and `triton.language`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.Optional`、`torch`、`triton` 以及 `triton.language`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 14-53: Internal helper `_gemma_rmsnorm_residual_kernel`
```python
@triton.jit
def _gemma_rmsnorm_residual_kernel(
    X_ptr,
    W_ptr,
    Residual_ptr,
    Scalar_ptr,
    Out_ptr,
    stride_x,
    stride_r,
    stride_o,
    N,
    eps,
    HAS_SCALAR: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    """Fused kernel: out = rmsnorm(x, w) + residual [* scalar]

    When HAS_SCALAR is True, also multiplies by a scalar loaded from Scalar_ptr.
    """
    row = tl.program_id(0)
    cols = tl.arange(0, BLOCK_SIZE)
    mask = cols < N

    x = tl.load(X_ptr + row * stride_x + cols, mask=mask, other=0.0).to(tl.float32)
    w = tl.load(W_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    r = tl.load(Residual_ptr + row * stride_r + cols, mask=mask, other=0.0).to(
        tl.float32
    )

    var = tl.sum(x * x, axis=0) / N
    rrms = tl.rsqrt(var + eps)
    out = x * rrms * w + r

    if HAS_SCALAR:
        scalar = tl.load(Scalar_ptr).to(tl.float32)
        out = out * scalar

    tl.store(Out_ptr + row * stride_o + cols, out.to(x.dtype), mask=mask)
```
**EN:** This block defines `_gemma_rmsnorm_residual_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.rsqrt`, and `tl.store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `row`, `cols`, `mask`, `x`, and `w` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_gemma_rmsnorm_residual_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load.to`、`tl.rsqrt` 以及 `tl.store`，说明该流程会编排底层辅助函数或计算内核。 像 `row`、`cols`、`mask`、`x` 以及 `w` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 54-83: Function `gemma_rmsnorm_residual_scalar` and its core logic
```python
def gemma_rmsnorm_residual_scalar(
    x: torch.Tensor,
    weight: torch.Tensor,
    residual: torch.Tensor,
    scalar: torch.Tensor,
    eps: float = 1e-6,
) -> torch.Tensor:
    """Fused (rmsnorm(x) + residual) * scalar."""
    assert x.dim() == 2 and x.stride(-1) == 1, "Expected contiguous 2D input"
    M, N = x.shape
    BLOCK_SIZE = triton.next_power_of_2(N)
    out = torch.empty_like(x)

    _gemma_rmsnorm_residual_kernel[(M,)](
        x,
        weight,
        residual,
        scalar,
        out,
        x.stride(0),
        residual.stride(0),
        out.stride(0),
        N,
        eps,
        HAS_SCALAR=True,
        BLOCK_SIZE=BLOCK_SIZE,
    )
    return out
```
**EN:** This block defines `gemma_rmsnorm_residual_scalar` and contains the main logic for this step. It mainly invokes `triton.next_power_of_2`, `torch.empty_like`, `_gemma_rmsnorm_residual_kernel`, `x.stride`, and `residual.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `N`, `BLOCK_SIZE`, and `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `gemma_rmsnorm_residual_scalar`，并承载这一阶段的核心逻辑。 它主要调用 `triton.next_power_of_2`、`torch.empty_like`、`_gemma_rmsnorm_residual_kernel`、`x.stride` 以及 `residual.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`N`、`BLOCK_SIZE` 以及 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 84-134: Internal helper `_gemma_dual_rmsnorm_residual_kernel`
```python
@triton.jit
def _gemma_dual_rmsnorm_residual_kernel(
    X1_ptr,
    W1_ptr,
    X2_ptr,
    W2_ptr,
    W3_ptr,
    Residual_ptr,
    Scalar_ptr,
    Out_ptr,
    stride_x1,
    stride_x2,
    stride_r,
    stride_o,
    N,
    eps1,
    eps2,
    eps3,
    BLOCK_SIZE: tl.constexpr,
):
    """Fused: out = (rmsnorm(rmsnorm(x1,w1) + rmsnorm(x2,w2), w3) + residual) * scalar"""
    row = tl.program_id(0)
    cols = tl.arange(0, BLOCK_SIZE)
    mask = cols < N

    x1 = tl.load(X1_ptr + row * stride_x1 + cols, mask=mask, other=0.0).to(tl.float32)
    w1 = tl.load(W1_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    x2 = tl.load(X2_ptr + row * stride_x2 + cols, mask=mask, other=0.0).to(tl.float32)
    w2 = tl.load(W2_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    w3 = tl.load(W3_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    r = tl.load(Residual_ptr + row * stride_r + cols, mask=mask, other=0.0).to(
        tl.float32
    )

    var1 = tl.sum(x1 * x1, axis=0) / N
    norm1 = x1 * tl.rsqrt(var1 + eps1) * w1

    var2 = tl.sum(x2 * x2, axis=0) / N
    norm2 = x2 * tl.rsqrt(var2 + eps2) * w2

    combined = norm1 + norm2

    var3 = tl.sum(combined * combined, axis=0) / N
    norm3 = combined * tl.rsqrt(var3 + eps3) * w3

    scalar = tl.load(Scalar_ptr).to(tl.float32)
    out = (norm3 + r) * scalar

    tl.store(Out_ptr + row * stride_o + cols, out.to(x1.dtype), mask=mask)
```
**EN:** This block defines `_gemma_dual_rmsnorm_residual_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.store`, and `tl.sum`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `row`, `cols`, `mask`, `x1`, and `w1` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_gemma_dual_rmsnorm_residual_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load.to`、`tl.store` 以及 `tl.sum`，说明该流程会编排底层辅助函数或计算内核。 像 `row`、`cols`、`mask`、`x1` 以及 `w1` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 135-193: Internal helper `_gemma_qkv_rmsnorm_kernel`
```python
@triton.jit
def _gemma_qkv_rmsnorm_kernel(
    Q_ptr,
    K_ptr,
    V_ptr,
    Q_w_ptr,
    K_w_ptr,
    stride_q_m,
    stride_k_m,
    stride_v_m,
    NUM_Q_HEADS: tl.constexpr,
    NUM_KV_HEADS: tl.constexpr,
    HEAD_DIM: tl.constexpr,
    eps,
    HAS_KV: tl.constexpr,
    BLOCK: tl.constexpr,
):
    """Per-token fused RMSNorm of Q (with q_w), K (with k_w), V (no scale).

    Layout assumption: each tensor's last dim packs (num_heads, head_dim) contiguously
    so per-head offset is `h * HEAD_DIM`. The token (M) stride is taken from
    stride_*_m so the kernel works on strided views (e.g. slices of a larger
    qkv buffer produced by `qkv.split`) without requiring `.contiguous()` copies.
    V uses `weight=ones` semantics so the multiply-by-weight is omitted.
    """
    m = tl.program_id(0)
    cols = tl.arange(0, BLOCK)
    mask = cols < HEAD_DIM

    qw = tl.load(Q_w_ptr + cols, mask=mask, other=0.0).to(tl.float32)

    # Q heads
    for h in tl.static_range(NUM_Q_HEADS):
        off = m * stride_q_m + h * HEAD_DIM + cols
        x = tl.load(Q_ptr + off, mask=mask, other=0.0).to(tl.float32)
        rrms = tl.rsqrt(tl.sum(x * x, axis=0) / HEAD_DIM + eps)
        out = x * rrms * qw
        tl.store(Q_ptr + off, out.to(Q_ptr.dtype.element_ty), mask=mask)

    if HAS_KV:
        kw = tl.load(K_w_ptr + cols, mask=mask, other=0.0).to(tl.float32)

        # K heads
        for h in tl.static_range(NUM_KV_HEADS):
            off = m * stride_k_m + h * HEAD_DIM + cols
            x = tl.load(K_ptr + off, mask=mask, other=0.0).to(tl.float32)
            rrms = tl.rsqrt(tl.sum(x * x, axis=0) / HEAD_DIM + eps)
            out = x * rrms * kw
            tl.store(K_ptr + off, out.to(K_ptr.dtype.element_ty), mask=mask)

        # V heads (no scaling: V-norm uses weight=ones)
        for h in tl.static_range(NUM_KV_HEADS):
            off = m * stride_v_m + h * HEAD_DIM + cols
            x = tl.load(V_ptr + off, mask=mask, other=0.0).to(tl.float32)
            rrms = tl.rsqrt(tl.sum(x * x, axis=0) / HEAD_DIM + eps)
            out = x * rrms
            tl.store(V_ptr + off, out.to(V_ptr.dtype.element_ty), mask=mask)
```
**EN:** This block defines `_gemma_qkv_rmsnorm_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.static_range`, and `tl.rsqrt`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `m`, `cols`, `mask`, `qw`, and `off` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_gemma_qkv_rmsnorm_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load.to`、`tl.static_range` 以及 `tl.rsqrt`，说明该流程会编排底层辅助函数或计算内核。 像 `m`、`cols`、`mask`、`qw` 以及 `off` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 194-247: Function `gemma_qkv_rmsnorm` and its core logic
```python
def gemma_qkv_rmsnorm(
    q: torch.Tensor,
    k: Optional[torch.Tensor],
    v: Optional[torch.Tensor],
    q_weight: torch.Tensor,
    k_weight: Optional[torch.Tensor],
    num_q_heads: int,
    num_kv_heads: int,
    head_dim: int,
    eps: float = 1e-6,
) -> None:
    """In-place fused RMSNorm on Q, K, V for Gemma4 attention.

    All three norms compute `x * rsqrt(mean(x^2) + eps)` independently per head.
    Q is scaled by `q_weight`, K by `k_weight`, V by 1 (Gemma4's V-norm has
    `with_scale=False`).

    Inputs may be 2D `(M, num_heads * head_dim)` or strided views of a larger
    buffer (such as q/k/v slices from `qkv.split`). The kernel uses the actual
    `stride(0)` so no `.contiguous()` copy is required. Within a token, the
    last dim must be contiguous so heads pack as `h * head_dim` offsets.

    If k and v are both None (KV-shared layer), only Q is normalized.
    """
    assert q.is_cuda
    assert q.stride(-1) == 1, "Q's last dim must be contiguous"
    assert q_weight.shape[-1] == head_dim
    M = q.shape[0] if q.dim() >= 2 else 1
    BLOCK = triton.next_power_of_2(head_dim)

    has_kv = k is not None and v is not None
    if has_kv:
        assert k.is_cuda and v.is_cuda
        assert k.stride(-1) == 1 and v.stride(-1) == 1
        assert k_weight is not None and k_weight.shape[-1] == head_dim

    _gemma_qkv_rmsnorm_kernel[(M,)](
        q,
        k if has_kv else q,
        v if has_kv else q,
        q_weight,
        k_weight if has_kv else q_weight,
        q.stride(0),
        k.stride(0) if has_kv else 0,
        v.stride(0) if has_kv else 0,
        NUM_Q_HEADS=num_q_heads,
        NUM_KV_HEADS=num_kv_heads if has_kv else 0,
        HEAD_DIM=head_dim,
        eps=eps,
        HAS_KV=has_kv,
        BLOCK=BLOCK,
    )
```
**EN:** This block defines `gemma_qkv_rmsnorm` and contains the main logic for this step. It mainly invokes `triton.next_power_of_2`, `_gemma_qkv_rmsnorm_kernel`, `q.stride`, `q.dim`, and `k.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `BLOCK`, and `has_kv` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `gemma_qkv_rmsnorm`，并承载这一阶段的核心逻辑。 它主要调用 `triton.next_power_of_2`、`_gemma_qkv_rmsnorm_kernel`、`q.stride`、`q.dim` 以及 `k.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`BLOCK` 以及 `has_kv` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 248-285: Function `gemma_dual_rmsnorm_residual_scalar` and its core logic
```python
def gemma_dual_rmsnorm_residual_scalar(
    x1: torch.Tensor,
    weight1: torch.Tensor,
    x2: torch.Tensor,
    weight2: torch.Tensor,
    weight3: torch.Tensor,
    residual: torch.Tensor,
    scalar: torch.Tensor,
    eps1: float = 1e-6,
    eps2: float = 1e-6,
    eps3: float = 1e-6,
) -> torch.Tensor:
    """Fused (rmsnorm(rmsnorm(x1,w1) + rmsnorm(x2,w2), w3) + residual) * scalar."""
    assert x1.dim() == 2 and x1.stride(-1) == 1
    M, N = x1.shape
    BLOCK_SIZE = triton.next_power_of_2(N)
    out = torch.empty_like(x1)

    _gemma_dual_rmsnorm_residual_kernel[(M,)](
        x1,
        weight1,
        x2,
        weight2,
        weight3,
        residual,
        scalar,
        out,
        x1.stride(0),
        x2.stride(0),
        residual.stride(0),
        out.stride(0),
        N,
        eps1,
        eps2,
        eps3,
        BLOCK_SIZE=BLOCK_SIZE,
    )
    return out
```
**EN:** This block defines `gemma_dual_rmsnorm_residual_scalar` and contains the main logic for this step. It mainly invokes `triton.next_power_of_2`, `torch.empty_like`, `_gemma_dual_rmsnorm_residual_kernel`, `x1.stride`, and `x2.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `N`, `BLOCK_SIZE`, and `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `gemma_dual_rmsnorm_residual_scalar`，并承载这一阶段的核心逻辑。 它主要调用 `triton.next_power_of_2`、`torch.empty_like`、`_gemma_dual_rmsnorm_residual_kernel`、`x1.stride` 以及 `x2.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`N`、`BLOCK_SIZE` 以及 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_gemma_rmsnorm_residual_kernel`, `gemma_rmsnorm_residual_scalar`, `_gemma_dual_rmsnorm_residual_kernel`, `_gemma_qkv_rmsnorm_kernel`, and `gemma_qkv_rmsnorm`. / **主要符号**：核心入口包括 `_gemma_rmsnorm_residual_kernel`、`gemma_rmsnorm_residual_scalar`、`_gemma_dual_rmsnorm_residual_kernel`、`_gemma_qkv_rmsnorm_kernel` 以及 `gemma_qkv_rmsnorm`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。
- **Acceleration stack**: The module depends on external acceleration libraries and wraps them behind Python entry points. / **加速栈**：该模块依赖外部加速库，并通过 Python 入口对它们进行封装。

## Dependencies / 依赖关系
- **Standard library**: `typing.Optional` / **标准库**：`typing.Optional`
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
