# per_token_group_quant_8bit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/per_token_group_quant_8bit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING

import torch

from sglang.jit_kernel.utils import cache_once, load_jit, make_cpp_args
from sglang.kernel_api_logging import debug_kernel_api
from sglang.srt.utils.custom_op import register_custom_op

if TYPE_CHECKING:
    from tvm_ffi.module import Module

from sglang.jit_kernel.utils import CPP_DTYPE_MAP as OUTPUT_DTYPE_MAP


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-38: Function `_jit_per_token_group_quant_8bit_module`
```python
def _jit_per_token_group_quant_8bit_module(
    dtype: torch.dtype, output_type: torch.dtype
) -> Module:
    input_args = make_cpp_args(dtype)
    out_cpp = OUTPUT_DTYPE_MAP[output_type]
    return load_jit(
        "per_token_group_quant_8bit",
        cuda_files=["gemm/per_token_group_quant_8bit.cuh"],
        cuda_wrappers=[
            (
                "per_token_group_quant_8bit",
                f"per_token_group_quant_8bit<{input_args}, {out_cpp}>",
            )
        ],
    )


@register_custom_op(
    op_name="per_token_group_quant_8bit",
    mutates_args=["output_q", "output_s"],
)
```
**EN:** This block defines `_jit_per_token_group_quant_8bit_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_per_token_group_quant_8bit_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 39-76: Function `_per_token_group_quant_8bit_custom_op`
```python
def _per_token_group_quant_8bit_custom_op(
    input: torch.Tensor,
    output_q: torch.Tensor,
    output_s: torch.Tensor,
    group_size: int,
    eps: float,
    fp8_min: float,
    fp8_max: float,
    scale_ue8m0: bool = False,
) -> None:
    """
    Per-token-group quantization to 8-bit format.

    Args:
        input: Input tensor to quantize (float, half, or bfloat16).
        output_q: Output quantized tensor (e.g., fp8_e4m3 or int8).
        output_s: Output scale tensor.
        group_size: The size of the group for quantization.
        eps: A small value to avoid division by zero.
        fp8_min: The minimum value of the 8-bit data type.
        fp8_max: The maximum value of the 8-bit data type.
        scale_ue8m0: Whether to use UE8M0 format for scales.
    """
    module = _jit_per_token_group_quant_8bit_module(input.dtype, output_q.dtype)
    module.per_token_group_quant_8bit(
        input,
        output_q,
        output_s,
        group_size,
        eps,
        fp8_min,
        fp8_max,
        scale_ue8m0,
    )
    return None


@debug_kernel_api
```
**EN:** This block defines `_per_token_group_quant_8bit_custom_op`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_per_token_group_quant_8bit_custom_op`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 77-97: Function `per_token_group_quant_8bit`
```python
def per_token_group_quant_8bit(
    input: torch.Tensor,
    output_q: torch.Tensor,
    output_s: torch.Tensor,
    group_size: int,
    eps: float,
    fp8_min: float,
    fp8_max: float,
    scale_ue8m0: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    _per_token_group_quant_8bit_custom_op(
        input=input,
        output_q=output_q,
        output_s=output_s,
        group_size=group_size,
        eps=eps,
        fp8_min=fp8_min,
        fp8_max=fp8_max,
        scale_ue8m0=scale_ue8m0,
    )
    return output_q, output_s
```
**EN:** This block defines `per_token_group_quant_8bit`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `per_token_group_quant_8bit`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `sglang.kernel_api_logging -> debug_kernel_api`
- `sglang.srt.utils.custom_op -> register_custom_op`
- `tvm_ffi.module -> Module`
- `sglang.jit_kernel.utils -> CPP_DTYPE_MAP`
