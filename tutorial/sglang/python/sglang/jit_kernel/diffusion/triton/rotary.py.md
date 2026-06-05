# rotary.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/triton/rotary.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import torch". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import torch”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
import torch
import triton  # type: ignore
import triton.language as tl  # type: ignore

from sglang.multimodal_gen.runtime.platforms import current_platform


@triton.autotune(
    configs=[
        triton.Config({"BLOCK_HEADS": 1, "BLOCK_HS_HALF": 32}, num_warps=2),
        triton.Config({"BLOCK_HEADS": 2, "BLOCK_HS_HALF": 32}, num_warps=2),
        triton.Config({"BLOCK_HEADS": 4, "BLOCK_HS_HALF": 32}, num_warps=4),
        triton.Config({"BLOCK_HEADS": 4, "BLOCK_HS_HALF": 64}, num_warps=4),
        triton.Config({"BLOCK_HEADS": 8, "BLOCK_HS_HALF": 64}, num_warps=8),
    ],
    key=["num_heads", "head_size"],
)
@triton.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 19-83: Function `_rotary_embedding_kernel`
```python
def _rotary_embedding_kernel(
    output_ptr,
    x_ptr,
    cos_ptr,
    sin_ptr,
    num_heads,
    head_size,
    num_tokens,
    stride_out_bt,
    stride_out_head,
    stride_x_bt,
    stride_x_head,
    stride_cos_row,
    stride_sin_row,
    BLOCK_HEADS: tl.constexpr,
    BLOCK_HS_HALF: tl.constexpr,
):
    bt_idx = tl.program_id(0)
    head_block_idx = tl.program_id(1)
    token_idx = bt_idx % num_tokens

    cos_row_ptr = cos_ptr + token_idx * stride_cos_row
    sin_row_ptr = sin_ptr + token_idx * stride_sin_row
    head_offsets = head_block_idx * BLOCK_HEADS + tl.arange(0, BLOCK_HEADS)
    head_mask = head_offsets < num_heads

    head_size_half = head_size // 2
    x_row_ptrs = x_ptr + bt_idx * stride_x_bt + head_offsets[:, None] * stride_x_head
    output_row_ptrs = (
        output_ptr + bt_idx * stride_out_bt + head_offsets[:, None] * stride_out_head
    )

    for block_start in range(0, head_size_half, BLOCK_HS_HALF):
        offsets_half = block_start + tl.arange(0, BLOCK_HS_HALF)
        half_mask = offsets_half < head_size_half
        mask = head_mask[:, None] & half_mask[None, :]

        cos_vals = tl.load(cos_row_ptr + offsets_half, mask=half_mask, other=0.0)
        sin_vals = tl.load(sin_row_ptr + offsets_half, mask=half_mask, other=0.0)

        offsets_x1 = 2 * offsets_half
        offsets_x2 = 2 * offsets_half + 1

        x1_vals = tl.load(x_row_ptrs + offsets_x1[None, :], mask=mask, other=0.0)
        x2_vals = tl.load(x_row_ptrs + offsets_x2[None, :], mask=mask, other=0.0)

        x1_fp32 = x1_vals.to(tl.float32)
        x2_fp32 = x2_vals.to(tl.float32)
        cos_fp32 = cos_vals.to(tl.float32)[None, :]
        sin_fp32 = sin_vals.to(tl.float32)[None, :]
        o1_vals = tl.fma(-x2_fp32, sin_fp32, x1_fp32 * cos_fp32)
        o2_vals = tl.fma(x1_fp32, sin_fp32, x2_fp32 * cos_fp32)

        tl.store(
            output_row_ptrs + offsets_x1[None, :],
            o1_vals.to(x1_vals.dtype),
            mask=mask,
        )
        tl.store(
            output_row_ptrs + offsets_x2[None, :],
            o2_vals.to(x2_vals.dtype),
            mask=mask,
        )
```
**EN:** This block defines `_rotary_embedding_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_rotary_embedding_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 84-141: Imports and module setup
```python
def apply_rotary_embedding(
    x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor, interleaved: bool = False
) -> torch.Tensor:
    output = torch.empty_like(x)

    if x.dim() > 3:
        bsz, num_tokens, num_heads, head_size = x.shape
    else:
        num_tokens, num_heads, head_size = x.shape
        bsz = 1

    assert head_size % 2 == 0, "head_size must be divisible by 2"

    x_reshaped = x.view(bsz * num_tokens, num_heads, head_size)
    output_reshaped = output.view(bsz * num_tokens, num_heads, head_size)

    if interleaved and cos.shape[-1] == head_size:
        cos = cos[..., ::2].contiguous()
        sin = sin[..., ::2].contiguous()
    else:
        cos = cos.contiguous()
        sin = sin.contiguous()

    _rotary_embedding_kernel[
        lambda META: (bsz * num_tokens, triton.cdiv(num_heads, META["BLOCK_HEADS"]))
    ](
        output_reshaped,
        x_reshaped,
        cos,
        sin,
        num_heads,
        head_size,
        num_tokens,
        output_reshaped.stride(0),
        output_reshaped.stride(1),
        x_reshaped.stride(0),
        x_reshaped.stride(1),
        cos.stride(0),
        sin.stride(0),
    )

    return output


if current_platform.is_npu():
    from .npu_fallback import apply_rotary_embedding_native

    apply_rotary_embedding = apply_rotary_embedding_native

if current_platform.is_mps():
    from .mps_fallback import apply_rotary_embedding_native

    apply_rotary_embedding = apply_rotary_embedding_native

if current_platform.is_cpu():
    from .torch_fallback import apply_rotary_embedding_native

    apply_rotary_embedding = apply_rotary_embedding_native
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language as tl`
- `sglang.multimodal_gen.runtime.platforms -> current_platform`
- `.npu_fallback -> apply_rotary_embedding_native`
- `.mps_fallback -> apply_rotary_embedding_native`
- `.torch_fallback -> apply_rotary_embedding_native`
