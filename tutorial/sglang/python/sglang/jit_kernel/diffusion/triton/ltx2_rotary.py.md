# ltx2_rotary.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/triton/ltx2_rotary.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import torch". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import torch”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and module setup
```python
import torch
import triton
import triton.language as tl


@triton.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 7-60: Function `_ltx2_split_rotary_kernel`
```python
def _ltx2_split_rotary_kernel(
    out_ptr,
    x_ptr,
    cos_ptr,
    sin_ptr,
    seq_len: tl.constexpr,
    num_heads: tl.constexpr,
    head_dim: tl.constexpr,
    half_dim: tl.constexpr,
    stride_cos_b: tl.constexpr,
    stride_cos_h: tl.constexpr,
    stride_cos_t: tl.constexpr,
    stride_sin_b: tl.constexpr,
    stride_sin_h: tl.constexpr,
    stride_sin_t: tl.constexpr,
    BLOCK_HEADS: tl.constexpr,
    BLOCK_HALF: tl.constexpr,
):
    pid_bt = tl.program_id(0)
    head_block = tl.program_id(1)
    batch = pid_bt // seq_len
    token = pid_bt - batch * seq_len
    heads = head_block * BLOCK_HEADS + tl.arange(0, BLOCK_HEADS)
    offsets = tl.arange(0, BLOCK_HALF)
    mask = (heads[:, None] < num_heads) & (offsets[None, :] < half_dim)

    x_base = ((batch * seq_len + token) * num_heads + heads[:, None]) * head_dim
    cos_base = (
        batch * stride_cos_b + heads[:, None] * stride_cos_h + token * stride_cos_t
    )
    sin_base = (
        batch * stride_sin_b + heads[:, None] * stride_sin_h + token * stride_sin_t
    )

    x_first = tl.load(x_ptr + x_base + offsets[None, :], mask=mask, other=0.0)
    x_second = tl.load(
        x_ptr + x_base + half_dim + offsets[None, :], mask=mask, other=0.0
    )
    cos = tl.load(cos_ptr + cos_base + offsets[None, :], mask=mask, other=0.0)
    sin = tl.load(sin_ptr + sin_base + offsets[None, :], mask=mask, other=0.0)

    # Match the original PyTorch order: x * cos is written as BF16 first, then
    # addcmul_ computes the sine product in FP32 before the final BF16 store.
    out_first = (x_first * cos).to(tl.bfloat16).to(tl.float32) + (
        -x_second.to(tl.float32) * sin.to(tl.float32)
    )
    out_second = (x_second * cos).to(tl.bfloat16).to(tl.float32) + (
        x_first.to(tl.float32) * sin.to(tl.float32)
    )

    tl.store(out_ptr + x_base + offsets[None, :], out_first, mask=mask)
    tl.store(out_ptr + x_base + half_dim + offsets[None, :], out_second, mask=mask)
```
**EN:** This block defines `_ltx2_split_rotary_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_ltx2_split_rotary_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 61-102: Function `apply_ltx2_split_rotary_emb`
```python
def apply_ltx2_split_rotary_emb(
    x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor
) -> torch.Tensor:
    batch, seq_len, inner_dim = x.shape
    cos_batch, num_heads, cos_seq_len, half_dim = cos.shape
    head_dim = half_dim * 2
    if (
        cos_batch != batch
        or cos_seq_len != seq_len
        or inner_dim != num_heads * head_dim
        or sin.shape != cos.shape
    ):
        raise ValueError(
            "LTX2 split RoPE shape mismatch: "
            f"x={tuple(x.shape)}, cos={tuple(cos.shape)}, sin={tuple(sin.shape)}"
        )

    out = torch.empty_like(x)
    block_half = triton.next_power_of_2(half_dim)
    block_heads = min(16, triton.next_power_of_2(num_heads))
    num_warps = min(8, max(1, block_heads))
    grid = (batch * seq_len, triton.cdiv(num_heads, block_heads))
    _ltx2_split_rotary_kernel[grid](
        out,
        x,
        cos,
        sin,
        seq_len,
        num_heads,
        head_dim,
        half_dim,
        cos.stride(0),
        cos.stride(1),
        cos.stride(2),
        sin.stride(0),
        sin.stride(1),
        sin.stride(2),
        BLOCK_HEADS=block_heads,
        BLOCK_HALF=block_half,
        num_warps=num_warps,
    )
    return out
```
**EN:** This block defines `apply_ltx2_split_rotary_emb`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_ltx2_split_rotary_emb`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language as tl`
