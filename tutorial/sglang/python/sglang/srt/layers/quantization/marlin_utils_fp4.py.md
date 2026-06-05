# marlin_utils_fp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/marlin_utils_fp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides helper functions and kernel-facing utilities for marlin utils fp4 quantization flows. / 该模块提供了面向 Marlin 工具 FP4 量化流程的辅助函数与内核工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and setup
```python
from __future__ import annotations

import torch

from sglang.srt.layers.quantization.marlin_utils import (
    marlin_make_workspace,
    marlin_permute_bias,
    marlin_permute_scales,
)
from sglang.srt.utils import is_cuda
```
**EN:** This block imports __future__, torch, sglang.jit_kernel.gptq_marlin_repack, sglang.srt.layers.quantization.marlin_utils, sglang.srt.utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, sglang.jit_kernel.gptq_marlin_repack, sglang.srt.layers.quantization.marlin_utils, sglang.srt.utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 12-12: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 14-15: conditional logic for _is_cuda
```python
if _is_cuda:
    from sglang.jit_kernel.gptq_marlin_repack import gptq_marlin_repack
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 18-33: mxfp4_marlin_process_scales()
```python
def mxfp4_marlin_process_scales(
    marlin_scales: torch.Tensor,
    input_dtype: torch.dtype | None = None,
) -> torch.Tensor:
    if input_dtype is None or input_dtype.itemsize == 2:
        marlin_scales = marlin_scales.view(-1, 4)[:, [0, 2, 1, 3]].view(
            marlin_scales.size(0), -1
        )
    marlin_scales = marlin_scales.to(torch.float8_e8m0fnu)
    if input_dtype == torch.float8_e4m3fn:
        marlin_scales = marlin_scales.view(torch.uint8)
        assert marlin_scales.max() <= 249
        # exponent_bias (fp4->fp8) = 2 ** 3 - 2 ** 1 = 6
        marlin_scales = marlin_scales + 6
        marlin_scales = marlin_scales.view(torch.float8_e8m0fnu)
    return marlin_scales
```
**EN:** This block defines `mxfp4_marlin_process_scales()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `mxfp4_marlin_process_scales()`，用于实现量化栈中的可复用模块逻辑。

### Lines 36-52: _normalize_scale_tensor()
```python
def _normalize_scale_tensor(
    scales: torch.Tensor, target_dtype: torch.dtype
) -> torch.Tensor:
    # The kernel consumes E8M0 exponents. Regardless of the placeholder dtype
    # the loader used, we want the *numerical* value 2**e in ``target_dtype``.
    # float32/bfloat16/float16 containers hold the numerical 2**e directly
    # (they were filled via a dtype-promoting copy from uint8/e8m0).
    # uint8/int8 containers hold the raw E8M0 byte and must be reinterpreted.
    if scales.dtype == torch.float8_e8m0fnu:
        return scales.to(target_dtype)
    if scales.dtype == torch.uint8:
        return scales.view(torch.float8_e8m0fnu).to(target_dtype)
    if scales.dtype == torch.int8:
        return scales.view(torch.uint8).view(torch.float8_e8m0fnu).to(target_dtype)
    if scales.dtype in (torch.float32, torch.bfloat16, torch.float16):
        return scales.to(target_dtype)
    raise TypeError(f"Unsupported MXFP4 scale dtype for Marlin: {scales.dtype}")
```
**EN:** This block defines `_normalize_scale_tensor()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_normalize_scale_tensor()`，用于实现量化栈中的可复用模块逻辑。

### Lines 55-60: _get_optional_param()
```python
def _get_optional_param(layer: torch.nn.Module, *names: str) -> torch.Tensor | None:
    for name in names:
        value = getattr(layer, name, None)
        if value is not None:
            return value
    return None
```
**EN:** This block defines `_get_optional_param()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_get_optional_param()`，用于实现量化栈中的可复用模块逻辑。

### Lines 63-107: prepare_moe_mxfp4_layer_for_marlin() (part 1/3)
```python
def prepare_moe_mxfp4_layer_for_marlin(layer: torch.nn.Module) -> None:
    group_size = 32
    w13 = layer.w13_weight.data
    w2 = layer.w2_weight.data
    w13_scale = _get_optional_param(layer, "w13_weight_scale", "w13_weight_scale_inv")
    w2_scale = _get_optional_param(layer, "w2_weight_scale", "w2_weight_scale_inv")
    w13_bias = _get_optional_param(layer, "w13_weight_bias", "w13_bias")
    w2_bias = _get_optional_param(layer, "w2_weight_bias", "w2_bias")

    if w13_scale is None or w2_scale is None:
        raise ValueError("MXFP4 Marlin requires w13/w2 weight scales.")

    w13_scale_data = w13_scale.data if hasattr(w13_scale, "data") else w13_scale
    w2_scale_data = w2_scale.data if hasattr(w2_scale, "data") else w2_scale
    w13_bias_data = w13_bias.data if hasattr(w13_bias, "data") else w13_bias
    w2_bias_data = w2_bias.data if hasattr(w2_bias, "data") else w2_bias

    num_experts = w13.shape[0]
    intermediate_size = w13.shape[1] // 2
    hidden_size = w13.shape[2] * 2
    param_dtype = getattr(
        layer,
        "orig_dtype",
        w13_bias_data.dtype if w13_bias_data is not None else torch.bfloat16,
    )

    device = w13.device
    layer.workspace = marlin_make_workspace(device, 4)
    perm = torch.empty(0, dtype=torch.int, device=device)

    def _repack_weight(weight: torch.Tensor, is_w13: bool) -> torch.Tensor:
        if is_w13:
            size_n, size_k = intermediate_size * 2, hidden_size
        else:
            size_n, size_k = hidden_size, intermediate_size
        assert weight.shape == (num_experts, size_n, size_k // 2)

        tensor_list = []
        for i in range(num_experts):
            qweight = weight[i].view(torch.int32).T.contiguous()
            marlin_qweight = gptq_marlin_repack(
                b_q_weight=qweight,
                perm=perm,
                size_k=size_k,
                size_n=size_n,
```
**EN:** This segment of `prepare_moe_mxfp4_layer_for_marlin()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `prepare_moe_mxfp4_layer_for_marlin()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 108-152: prepare_moe_mxfp4_layer_for_marlin() (part 2/3)
```python
                num_bits=4,
            )
            tensor_list.append(marlin_qweight)
        return torch.stack(tensor_list)

    def _permute_scales(scales: torch.Tensor, is_w13: bool) -> torch.Tensor:
        scales = _normalize_scale_tensor(scales, param_dtype)

        if is_w13:
            size_n, size_k = intermediate_size * 2, hidden_size
        else:
            size_n, size_k = hidden_size, intermediate_size

        tensor_list = []
        for i in range(num_experts):
            scale = scales[i].T.contiguous()
            marlin_scales = marlin_permute_scales(
                s=scale,
                size_k=size_k,
                size_n=size_n,
                group_size=group_size,
            )
            tensor_list.append(
                mxfp4_marlin_process_scales(
                    marlin_scales,
                    input_dtype=param_dtype,
                )
            )
        return torch.stack(tensor_list)

    def _permute_bias(bias: torch.Tensor | None) -> torch.Tensor | None:
        if bias is None:
            return None
        tensor_list = []
        for i in range(num_experts):
            tensor_list.append(marlin_permute_bias(bias[i].to(param_dtype)))
        return torch.stack(tensor_list)

    w13_marlin = _repack_weight(w13, True)
    w2_marlin = _repack_weight(w2, False)
    w13_scale_marlin = _permute_scales(w13_scale_data, True)
    w2_scale_marlin = _permute_scales(w2_scale_data, False)

    layer.w13_weight = torch.nn.Parameter(w13_marlin, requires_grad=False)
    layer.w2_weight = torch.nn.Parameter(w2_marlin, requires_grad=False)
```
**EN:** This segment of `prepare_moe_mxfp4_layer_for_marlin()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `prepare_moe_mxfp4_layer_for_marlin()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 153-163: prepare_moe_mxfp4_layer_for_marlin() (part 3/3)
```python
    layer.w13_weight_scale = torch.nn.Parameter(w13_scale_marlin, requires_grad=False)
    layer.w2_weight_scale = torch.nn.Parameter(w2_scale_marlin, requires_grad=False)

    if w13_bias_data is not None:
        layer.w13_weight_bias = torch.nn.Parameter(
            _permute_bias(w13_bias_data), requires_grad=False
        )
    if w2_bias_data is not None:
        layer.w2_weight_bias = torch.nn.Parameter(
            _permute_bias(w2_bias_data), requires_grad=False
        )
```
**EN:** This segment of `prepare_moe_mxfp4_layer_for_marlin()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `prepare_moe_mxfp4_layer_for_marlin()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

## Key Concepts / 关键概念
- `mxfp4_marlin_process_scales()` : A public function that implements reusable module logic for the quantization stack. / `mxfp4_marlin_process_scales()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `prepare_moe_mxfp4_layer_for_marlin()` : A public function that implements reusable module logic for the quantization stack. / `prepare_moe_mxfp4_layer_for_marlin()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`
- **Internal / 内部**: `sglang.jit_kernel.gptq_marlin_repack`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.utils`
