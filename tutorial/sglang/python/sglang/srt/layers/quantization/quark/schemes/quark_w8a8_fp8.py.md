# quark_w8a8_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/quark/schemes/quark_w8a8_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for quark w8a8 fp8 quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 Quark W8A8 FP8 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0

from typing import Any, Callable, Optional, cast

import torch
from torch.nn import Parameter

from sglang.srt.layers.parameter import (
    ChannelQuantScaleParameter,
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
from sglang.srt.layers.quantization.fp8_utils import (
    apply_fp8_linear,
    cutlass_fp8_supported,
    normalize_e4m3fn_to_e4m3fnuz,
)
from sglang.srt.layers.quantization.quark.schemes import QuarkLinearScheme
from sglang.srt.layers.quantization.utils import requantize_with_max_scale
from sglang.srt.utils import get_bool_env_var, is_hip, set_weight_attrs
```
**EN:** This block imports aiter.ops.shuffle, torch, torch.nn, typing, sglang.srt.layers.parameter, sglang.srt.layers.quantization.fp8_kernel, sglang.srt.layers.quantization.fp8_utils, sglang.srt.layers.quantization.quark.schemes and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 aiter.ops.shuffle, torch, torch.nn, typing, sglang.srt.layers.parameter, sglang.srt.layers.quantization.fp8_kernel, sglang.srt.layers.quantization.fp8_utils, sglang.srt.layers.quantization.quark.schemes 等依赖，并为当前量化实现准备模块命名空间。

### Lines 23-23: initialize __all__
```python
__all__ = ["QuarkW8A8Fp8"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 25-25: initialize _is_fp8_fnuz
```python
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_fp8_fnuz.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_fp8_fnuz。

### Lines 26-26: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 27-27: initialize _use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _use_aiter.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _use_aiter。

### Lines 28-29: conditional logic for _use_aiter
```python
if _use_aiter:
    from aiter.ops.shuffle import shuffle_weight
```
**EN:** This block applies conditional logic controlled by `_use_aiter`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_use_aiter` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 32-33: class QuarkW8A8Fp8: definition
```python
class QuarkW8A8Fp8(QuarkLinearScheme):
```
**EN:** This block declares `QuarkW8A8Fp8`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, process_weights_after_loading, create_weights.
**CN:** 该代码块声明 `QuarkW8A8Fp8`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, process_weights_after_loading, create_weights 等行为。

### Lines 34-48: QuarkW8A8Fp8.__init__()
```python
    def __init__(
        self, weight_config: dict[str, Any], input_config: Optional[dict[str, Any]]
    ):
        self.cutlass_fp8_supported = cutlass_fp8_supported()
        self.weight_qscheme = cast(str, weight_config.get("qscheme"))
        self.is_static_input_scheme: bool = False
        self.input_qscheme: Optional[str] = None
        if input_config is not None:
            self.is_static_input_scheme = not cast(bool, input_config.get("is_dynamic"))
            self.input_qscheme = cast(str, input_config.get("qscheme"))

        self.per_token = (
            not self.is_static_input_scheme and self.input_qscheme == "per_channel"
        )
        self.out_dtype = torch.get_default_dtype()
```
**EN:** This block defines `QuarkW8A8Fp8.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkW8A8Fp8.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 50-53: QuarkW8A8Fp8.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # lovelace and up
        return 89
```
**EN:** This block defines `QuarkW8A8Fp8.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkW8A8Fp8.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 55-115: QuarkW8A8Fp8.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer) -> None:
        # If per tensor, when we have a fused module (e.g. QKV) with per
        # tensor scales (thus N scales being passed to the kernel),
        # requantize so we can always run per tensor
        if self.weight_qscheme == "per_tensor":
            if _is_fp8_fnuz:
                input_scale = getattr(layer, "input_scale", None)
                weight, max_w_scale, input_scale = normalize_e4m3fn_to_e4m3fnuz(
                    weight=layer.weight,
                    weight_scale=layer.weight_scale,
                    input_scale=input_scale,
                )
                if input_scale is not None:
                    layer.input_scale = Parameter(input_scale, requires_grad=False)
            else:
                max_w_scale = layer.weight_scale
                weight = layer.weight

            max_w_scale, weight = requantize_with_max_scale(
                weight=weight,
                weight_scale=max_w_scale,
                logical_widths=layer.logical_widths,
            )

            layer.weight = Parameter(weight.t(), requires_grad=False)
            layer.weight_scale = Parameter(max_w_scale, requires_grad=False)

        # If channelwise, scales are already lined up, so just transpose.
        elif self.weight_qscheme == "per_channel":
            weight = layer.weight

            if _is_fp8_fnuz:
                input_scale = getattr(layer, "input_scale", None)
                weight, weight_scale, input_scale = normalize_e4m3fn_to_e4m3fnuz(
                    weight=weight,
                    weight_scale=layer.weight_scale,
                    input_scale=input_scale,
                )
                if input_scale is not None:
                    layer.input_scale = Parameter(input_scale, requires_grad=False)
            else:
                weight_scale = layer.weight_scale.data
            if self.per_token:
                weight_scale = weight_scale.view(-1, 1)
            if _use_aiter:
                layer.weight = Parameter(
                    shuffle_weight(weight, (16, 16)).t(), requires_grad=False
                )
            else:
                layer.weight = Parameter(weight.t(), requires_grad=False)
            # required by torch.compile to be torch.nn.Parameter
            layer.weight_scale = Parameter(weight_scale, requires_grad=False)

        else:
            raise ValueError(f"Unknown quantization scheme {self.weight_qscheme}")

        # INPUT SCALE
        if self.is_static_input_scheme:
            layer.input_scale = Parameter(layer.input_scale.max(), requires_grad=False)
        else:
            layer.input_scale = None
```
**EN:** This block defines `QuarkW8A8Fp8.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `QuarkW8A8Fp8.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 117-169: QuarkW8A8Fp8.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        output_partition_sizes: list[int],
        input_size_per_partition: int,
        params_dtype: torch.dtype,
        weight_loader: Callable,
        **kwargs,
    ):
        output_size_per_partition = sum(output_partition_sizes)
        layer.logical_widths = output_partition_sizes

        # WEIGHT
        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                dtype=torch.float8_e4m3fn,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        # WEIGHT SCALE
        if self.weight_qscheme == "per_channel":
            weight_scale = ChannelQuantScaleParameter(
                data=torch.empty((sum(output_partition_sizes)), dtype=torch.float32),
                output_dim=0,
                weight_loader=weight_loader,
            )
        else:
            assert self.weight_qscheme == "per_tensor"
            weight_scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )
            set_weight_attrs(weight_scale, {"needs_scalar_to_array": True})

        # min requirement for fp8 kernels
        weight_scale[:] = torch.finfo(torch.float32).min
        layer.register_parameter("weight_scale", weight_scale)

        # INPUT SCALE
        if self.is_static_input_scheme:
            input_scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )
            input_scale[:] = torch.finfo(torch.float32).min
            set_weight_attrs(input_scale, {"needs_scalar_to_array": True})
            layer.register_parameter("input_scale", input_scale)
```
**EN:** This block defines `QuarkW8A8Fp8.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuarkW8A8Fp8.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 171-186: QuarkW8A8Fp8.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        return apply_fp8_linear(
            x,
            layer.weight,
            layer.weight_scale,
            input_scale=layer.input_scale,
            bias=bias,
            cutlass_fp8_supported=self.cutlass_fp8_supported,
            use_per_token_if_dynamic=self.per_token,
        )
```
**EN:** This block defines `QuarkW8A8Fp8.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuarkW8A8Fp8.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `QuarkW8A8Fp8`: A scheme class that structures file-level quantization behavior. / `QuarkW8A8Fp8` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `aiter.ops.shuffle`, `torch`, `torch.nn`, `typing`
- **Internal / 内部**: `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.quark.schemes`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
