# petit_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/petit_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides helper functions and kernel-facing utilities for petit utils quantization flows. / 该模块提供了面向 Petit 工具 量化流程的辅助函数与内核工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: module imports and setup
```python
from typing import Optional

import torch
```
**EN:** This block imports petit_kernel, torch, typing and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 petit_kernel, torch, typing 等依赖，并为当前量化实现准备模块命名空间。

### Lines 5-33: guarded import or fallback path
```python
try:
    from petit_kernel import mul_nvfp4_a16, process_nvfp4_scales, repack_nvfp4
except ImportError:

    def _check_petit_nvfp4_supported(
        quant_method: str, group_size: Optional[int]
    ) -> tuple[bool, Optional[str]]:
        return (
            False,
            "Petit is not installed. Please install it with `pip install petit-kernel`.",
        )

    def prepare_nvfp4_layer_for_petit(layer: torch.nn.Module) -> None:
        raise ValueError(
            "Petit is not installed. Please install it with `pip install petit-kernel`."
        )

    def apply_petit_nvfp4_linear(
        input: torch.Tensor,
        weight: torch.Tensor,
        weight_scale: torch.Tensor,
        weight_scale_2: torch.Tensor,
        size_n: int,
        size_k: int,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        raise ValueError(
            "Petit is not installed. Please install it with `pip install petit-kernel`."
        )
```
**EN:** This block uses exception handling to provide a fallback implementation when optional functionality is unavailable.
**CN:** 该代码块通过异常处理在可选功能不可用时提供后备实现。

### Lines 36-52: _check_petit_nvfp4_supported()
```python
def _check_petit_nvfp4_supported(
    quant_method: str, group_size: Optional[int]
) -> tuple[bool, Optional[str]]:
    if quant_method != "NVFP4":
        return (
            False,
            "Petit currently only supports: NVFP4"
            " quantizations in sglang. Please check the "
            "`hf_quant_config.json` file for your model's "
            "quant configuration.",
        )
    if group_size is not None and group_size != 16:
        return (
            False,
            "Petit currently only supports: group_size=16" " quantizations.",
        )
    return (True, None)
```
**EN:** This block defines `_check_petit_nvfp4_supported()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_check_petit_nvfp4_supported()`，用于实现量化栈中的可复用模块逻辑。

### Lines 55-58: verify_petit_nvfp4_supported()
```python
def verify_petit_nvfp4_supported(quant_method: str, group_size: Optional[int]) -> None:
    supported, error_msg = _check_petit_nvfp4_supported(quant_method, group_size)
    if not supported:
        raise ValueError(error_msg)
```
**EN:** This block defines `verify_petit_nvfp4_supported()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `verify_petit_nvfp4_supported()`，用于实现量化栈中的可复用模块逻辑。

### Lines 61-75: prepare_nvfp4_layer_for_petit()
```python
def prepare_nvfp4_layer_for_petit(layer: torch.nn.Module) -> None:
    # Repack weights to petit format
    part_size_n = layer.output_size_per_partition
    part_size_k = layer.input_size_per_partition
    qweight = layer.weight.view(torch.int32).contiguous()
    petit_qweight = repack_nvfp4(qweight, size_n=part_size_n, size_k=part_size_k)
    layer.weight = torch.nn.Parameter(petit_qweight, requires_grad=False)

    # Permute scales
    weight_scale = process_nvfp4_scales(
        scales=layer.weight_scale, size_k=part_size_k, size_n=part_size_n
    )
    layer.weight_scale = torch.nn.Parameter(weight_scale, requires_grad=False)

    return
```
**EN:** This block defines `prepare_nvfp4_layer_for_petit()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `prepare_nvfp4_layer_for_petit()`，用于实现量化栈中的可复用模块逻辑。

### Lines 78-104: apply_petit_nvfp4_linear()
```python
def apply_petit_nvfp4_linear(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    weight_scale_2: torch.Tensor,
    size_n: int,
    size_k: int,
    bias: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    reshaped_x = input.reshape(-1, input.shape[-1])
    out_shape = input.shape[:-1] + (size_n,)

    # TODO: Use auto-tuning to find the performant solution_id
    output = mul_nvfp4_a16(
        a=reshaped_x,
        b=weight,
        s=weight_scale,
        global_scale=weight_scale_2,
        size_m=reshaped_x.size(0),
        size_n=size_n,
        size_k=size_k,
        solution_id=-1,
    )
    if bias is not None:
        output.add_(bias)  # In-place add

    return output.reshape(out_shape)
```
**EN:** This block defines `apply_petit_nvfp4_linear()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `apply_petit_nvfp4_linear()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `verify_petit_nvfp4_supported()` : A public function that implements reusable module logic for the quantization stack. / `verify_petit_nvfp4_supported()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `prepare_nvfp4_layer_for_petit()` : A public function that implements reusable module logic for the quantization stack. / `prepare_nvfp4_layer_for_petit()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `apply_petit_nvfp4_linear()` : A public function that applies quantized computation to runtime inputs. / `apply_petit_nvfp4_linear()`：一个公开函数，用于将量化计算应用到运行时输入上。

## Dependencies / 依赖关系
- **External / 外部**: `petit_kernel`, `torch`, `typing`
- **Internal / 内部**: Minimal package-local imports. / 包内直接导入较少。
