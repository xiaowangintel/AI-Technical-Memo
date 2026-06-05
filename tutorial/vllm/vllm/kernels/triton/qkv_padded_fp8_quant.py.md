# qkv_padded_fp8_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/kernels/triton/qkv_padded_fp8_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements Triton helpers for padded QKV FP8 quantization. / 实现用于带 padding 的 QKV FP8 量化 Triton 辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-7)
```python
"""Stride-aware FP8 quantization with head_dim padding for ViT attention.

Reads directly from non-contiguous QKV views using 3D strides and pads
head_dim to a multiple of 16 for cuDNN compatibility.
"""
```
**EN:** The opening docstring explains the file's role and design intent, giving readers context before the concrete kernel code starts.
**CN:** 开头的文档字符串说明了文件职责与设计意图，让读者在进入具体内核实现前先获得整体上下文。

### Imports (lines 9-17)
```python
import torch

from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    get_fp8_min_max,
)
from vllm.platforms import current_platform
from vllm.triton_utils import HAS_TRITON, tl, triton
from vllm.utils.math_utils import round_up
```
**EN:** This import block loads `torch`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.math_utils`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `qkv_padded_fp8_quant.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.math_utils`，为 `qkv_padded_fp8_quant.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 19-19)
```python
_FP8_MIN, _FP8_MAX = get_fp8_min_max()
```
**EN:** This block defines module constants (`_FP8_MIN`, `_FP8_MAX`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`_FP8_MIN`, `_FP8_MAX`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Function `_quantize_pad_fp8_kernel` (lines 23-76)
```python
def _quantize_pad_fp8_kernel(
    x_ptr,
    y_ptr,
    scale_ptr,
    stride_xs,
    stride_xh,
    stride_xd,
    stride_ys,
    stride_yh,
    stride_yd,
    num_heads,
    n_rows,
    n_cols,
    n_cols_padded,
    fp8_min,
    fp8_max,
    SKIP_SCALE: tl.constexpr,
    BLOCK_M: tl.constexpr,
    BLOCK_N: tl.constexpr,
):
    pid_m = tl.program_id(0)
    pid_n = tl.program_id(1)

    offs_m = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_n = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    mask_m = offs_m < n_rows
    mask_out = mask_m[:, None] & (offs_n[None, :] < n_cols_padded)
    mask_in = mask_m[:, None] & (offs_n[None, :] < n_cols)

    # Decompose flattened row into (token, head) for 3D stride indexing.
    s = offs_m // num_heads
    h = offs_m % num_heads

    x_ptrs = (
        x_ptr
        + s[:, None] * stride_xs
        + h[:, None] * stride_xh
        + offs_n[None, :] * stride_xd
    )
    x = tl.load(x_ptrs, mask=mask_in, other=0.0).to(tl.float32)
    if SKIP_SCALE:
        x_q = x
    else:
        scale = tl.load(scale_ptr)
        x_q = x / scale
    x_q = tl.clamp(x_q, fp8_min, fp8_max).to(y_ptr.dtype.element_ty)

    y_ptrs = (
        y_ptr
        + s[:, None] * stride_ys
        + h[:, None] * stride_yh
        + offs_n[None, :] * stride_yd
    )
    tl.store(y_ptrs, x_q, mask=mask_out)
```
**EN:** This helper implements `_quantize_pad_fp8_kernel`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `program_id`, `to`, `store`, `arange`.
**CN:** 该函数 `_quantize_pad_fp8_kernel` 封装了此模块中的一段关键运行时逻辑，重点处理 quantize pad fp 8 kernel 相关工作。 它内部会调用 `program_id`, `to`, `store`, `arange` 等例程。

### Function `_get_fp8_pad_quant_config` (lines 79-84)
```python
def _get_fp8_pad_quant_config(padded_head_dim: int) -> tuple[int, int, int]:
    block_n = triton.next_power_of_2(padded_head_dim)
    block_n = max(16, min(block_n, 128))
    block_m = 16
    num_warps = 4
    return block_m, block_n, num_warps
```
**EN:** This lookup helper implements `_get_fp8_pad_quant_config`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `next_power_of_2`, `max`, `min`.
**CN:** 该函数 `_get_fp8_pad_quant_config` 封装了此模块中的一段关键运行时逻辑，重点处理 get fp 8 pad quant config 相关工作。 它内部会调用 `next_power_of_2`, `max`, `min` 等例程。

### Function `quantize_fp8_pad_head_dim_triton` (lines 87-151)
```python
def quantize_fp8_pad_head_dim_triton(
    tensor: torch.Tensor,
    scale: torch.Tensor,
    skip_scale: bool = False,
    block_m: int | None = None,
    block_n: int | None = None,
    num_warps: int | None = None,
) -> torch.Tensor:
    """Quantize a 3D/4D tensor to FP8, padding head_dim to a multiple of 16.

    Reads directly from the input using its 3D strides, so non-contiguous
    views (e.g. Q/K/V slices from an interleaved QKV buffer) are handled
    without an extra copy.  Output is always a fresh contiguous tensor
    with shape (S, H, padded_D).
    """
    if not HAS_TRITON:
        raise RuntimeError("Triton is required to quantize with head_dim padding.")

    original_shape = tensor.shape
    if tensor.dim() == 4:
        tensor = tensor.view(-1, tensor.shape[-2], tensor.shape[-1])
    assert tensor.dim() == 3, f"Expected 3D input (S, H, D), got {tensor.dim()}D"
    S, H, D = tensor.shape
    padded_head_dim = round_up(D, 16)
    out_dtype = current_platform.fp8_dtype()
    output = torch.empty(
        (S, H, padded_head_dim),
        device=tensor.device,
        dtype=out_dtype,
    )

    scale_1d = scale.reshape(-1)
    n_rows = S * H

    if block_m is None or block_n is None or num_warps is None:
        block_m, block_n, num_warps = _get_fp8_pad_quant_config(padded_head_dim)

    grid = (
        triton.cdiv(n_rows, block_m),
        triton.cdiv(padded_head_dim, block_n),
    )

    _quantize_pad_fp8_kernel[grid](
        tensor,
        output,
        scale_1d,
        tensor.stride(0),
        tensor.stride(1),
        tensor.stride(2),
        output.stride(0),
        output.stride(1),
        output.stride(2),
        H,
        n_rows,
        D,
        padded_head_dim,
        _FP8_MIN,
        _FP8_MAX,
        SKIP_SCALE=skip_scale,
        BLOCK_M=block_m,
        BLOCK_N=block_n,
        num_warps=num_warps,
    )

    return output.view((*original_shape[:-1], padded_head_dim))
```
**EN:** This execution helper implements `quantize_fp8_pad_head_dim_triton`. Quantize a 3D/4D tensor to FP8, padding head_dim to a multiple of 16. Internally it relies on calls such as `round_up`, `fp8_dtype`, `empty`, `reshape`.
**CN:** 该函数 `quantize_fp8_pad_head_dim_triton` 封装了此模块中的一段关键运行时逻辑，重点处理 quantize fp 8 pad head dim triton 相关工作。 它内部会调用 `round_up`, `fp8_dtype`, `empty`, `reshape` 等例程。

### Function `quantize_fp8_maybe_pad_head_dim` (lines 154-180)
```python
def quantize_fp8_maybe_pad_head_dim(
    tensor: torch.Tensor,
    scale: torch.Tensor,
    fp8_quant: QuantFP8,
    skip_scale: bool = False,
) -> torch.Tensor:
    """Quantize a 3D/4D tensor to FP8, padding head_dim to a multiple of 16
    only when needed.

    Accepts (S, H, D) or (B, S, H, D) input. Uses ``fp8_quant`` (a
    :class:`QuantFP8` CustomOp) when head_dim is already aligned to 16
    (no padding); otherwise falls back to a stride-aware Triton kernel
    that pads head_dim to a multiple of 16.
    """
    head_dim = tensor.shape[-1]
    if head_dim % 16 != 0:
        return quantize_fp8_pad_head_dim_triton(tensor, scale, skip_scale=skip_scale)

    if skip_scale:
        return tensor.to(current_platform.fp8_dtype())

    # QuantFP8 expects 2D: flatten all dims except (H, D).
    orig_shape = tensor.shape
    total_tokens = tensor.numel() // (orig_shape[-1] * orig_shape[-2])
    tensor_2d = tensor.reshape(total_tokens, -1)
    fp8_tensor, _ = fp8_quant(tensor_2d, scale=scale)
    return fp8_tensor.reshape(orig_shape)
```
**EN:** This execution helper implements `quantize_fp8_maybe_pad_head_dim`. Quantize a 3D/4D tensor to FP8, padding head_dim to a multiple of 16 only when needed. Internally it relies on calls such as `reshape`, `fp8_quant`, `quantize_fp8_pad_head_dim_triton`, `to`.
**CN:** 该函数 `quantize_fp8_maybe_pad_head_dim` 封装了此模块中的一段关键运行时逻辑，重点处理 quantize fp 8 maybe pad head dim 相关工作。 它内部会调用 `reshape`, `fp8_quant`, `quantize_fp8_pad_head_dim_triton`, `to` 等例程。

## Key Concepts / 关键概念
- **Triton kernels / Triton 内核**
  - **EN:** The implementation relies on Triton-generated GPU kernels for low-level tensor work.
  - **CN:** 该实现依赖 Triton 生成的 GPU 内核完成底层张量计算。
- **FP8 quantization / FP8 量化**
  - **EN:** The module handles FP8 data, scales, or kernels for low-precision inference.
  - **CN:** 该模块处理 FP8 数据、缩放因子或相关低精度推理内核。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.math_utils`
- **External / 外部依赖**: `torch`
