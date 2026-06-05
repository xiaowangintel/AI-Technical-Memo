# compressed_tensors_w8a8_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_w8a8_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w8a8 fp8 quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 W8A8 FP8 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: module imports and setup
```python
# Adapted from https://github.com/vllm-project/vllm/tree/main/vllm/model_executor/layers/quantization/compressed_tensors
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from typing import Callable, Optional

import torch
from compressed_tensors.quantization import QuantizationArgs, QuantizationStrategy
from torch.nn import Parameter

from sglang.srt.layers.parameter import (
    BlockQuantScaleParameter,
    ChannelQuantScaleParameter,
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    CompressedTensorsLinearScheme,
)
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
from sglang.srt.layers.quantization.fp8_utils import (
    apply_fp8_linear,
    apply_fp8_ptpc_linear,
    dispatch_w8a8_block_fp8_linear,
    normalize_e4m3fn_to_e4m3fnuz,
    validate_fp8_block_shape,
)
from sglang.srt.layers.quantization.utils import requantize_with_max_scale
from sglang.srt.utils import get_bool_env_var, is_hip
```
**EN:** This block imports aiter.ops.shuffle, compressed_tensors.quantization, torch, torch.nn, sglang.srt.layers.parameter, sglang.srt.layers.quantization.compressed_tensors.schemes, sglang.srt.layers.quantization.fp8_kernel, sglang.srt.layers.quantization.fp8_utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 aiter.ops.shuffle, compressed_tensors.quantization, torch, torch.nn, sglang.srt.layers.parameter, sglang.srt.layers.quantization.compressed_tensors.schemes, sglang.srt.layers.quantization.fp8_kernel, sglang.srt.layers.quantization.fp8_utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 31-31: initialize __all__
```python
__all__ = ["CompressedTensorsW8A8Fp8"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 33-33: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 34-34: initialize _use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _use_aiter.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _use_aiter。

### Lines 35-36: conditional logic for _use_aiter
```python
if _use_aiter:
    from aiter.ops.shuffle import shuffle_weight
```
**EN:** This block applies conditional logic controlled by `_use_aiter`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_use_aiter` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 39-43: initialize strategy_to_parameter_type
```python
strategy_to_parameter_type = {
    QuantizationStrategy.BLOCK: BlockQuantScaleParameter,
    QuantizationStrategy.CHANNEL: ChannelQuantScaleParameter,
    QuantizationStrategy.TENSOR: PerTensorScaleParameter,
}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as strategy_to_parameter_type.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 strategy_to_parameter_type。

### Lines 46-46: class CompressedTensorsW8A8Fp8: definition
```python
class CompressedTensorsW8A8Fp8(CompressedTensorsLinearScheme):
```
**EN:** This block declares `CompressedTensorsW8A8Fp8`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `CompressedTensorsW8A8Fp8`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, create_weights, process_weights_after_loading 等行为。

### Lines 47-53: CompressedTensorsW8A8Fp8.__init__()
```python
    def __init__(self, weight_quant: QuantizationArgs, is_static_input_scheme: bool):
        self.weight_quant = weight_quant
        self.strategy = self.weight_quant.strategy
        self.is_static_input_scheme = is_static_input_scheme
        self.weight_block_size = self.weight_quant.block_structure
        if self.weight_block_size is not None:
            self.w8a8_block_fp8_linear = dispatch_w8a8_block_fp8_linear()
```
**EN:** This block defines `CompressedTensorsW8A8Fp8.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Fp8.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 55-58: CompressedTensorsW8A8Fp8.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # lovelace and up
        return 89
```
**EN:** This block defines `CompressedTensorsW8A8Fp8.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Fp8.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 60-104: CompressedTensorsW8A8Fp8.create_weights() (part 1/2)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        weight_loader: Callable,
        **kwargs,
    ):
        output_size_per_partition = sum(output_partition_sizes)
        layer.logical_widths = output_partition_sizes
        layer.weight_block_size = None
        layer.orig_dtype = params_dtype

        if self.strategy == QuantizationStrategy.BLOCK:
            assert self.weight_block_size is not None
            layer.weight_block_size = self.weight_block_size
            # Validate block quantization shapes
            validate_fp8_block_shape(
                layer,
                input_size,
                output_size,
                input_size_per_partition,
                output_partition_sizes,
                self.weight_block_size,
            )

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
        if self.strategy == QuantizationStrategy.CHANNEL:
            weight_scale = ChannelQuantScaleParameter(
```
**EN:** This segment of `CompressedTensorsW8A8Fp8.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 105-141: CompressedTensorsW8A8Fp8.create_weights() (part 2/2)
```python
                data=torch.empty((sum(output_partition_sizes), 1), dtype=torch.float32),
                output_dim=0,
                weight_loader=weight_loader,
            )
            weight_scale[:] = torch.finfo(torch.float32).min
        elif self.strategy == QuantizationStrategy.TENSOR:
            weight_scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )
            weight_scale[:] = torch.finfo(torch.float32).min
        elif self.strategy == QuantizationStrategy.BLOCK:
            assert layer.weight_block_size is not None
            block_n, block_k = layer.weight_block_size[0], layer.weight_block_size[1]
            output_size_per_partition = sum(output_partition_sizes)
            weight_scale = BlockQuantScaleParameter(
                data=torch.empty(
                    (output_size_per_partition + block_n - 1) // block_n,
                    (input_size_per_partition + block_k - 1) // block_k,
                    dtype=torch.float32,
                ),
                input_dim=1,
                output_dim=0,
                weight_loader=weight_loader,
            )
            weight_scale.format_ue8m0 = False
            weight_scale[:] = torch.finfo(torch.float32).min

        layer.register_parameter("weight_scale", weight_scale)
        # INPUT SCALE
        if self.is_static_input_scheme:
            input_scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )
            input_scale[:] = torch.finfo(torch.float32).min
            layer.register_parameter("input_scale", input_scale)
```
**EN:** This segment of `CompressedTensorsW8A8Fp8.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 143-208: CompressedTensorsW8A8Fp8.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer) -> None:
        if self.strategy == QuantizationStrategy.TENSOR:
            max_w_scale, weight = requantize_with_max_scale(
                weight=layer.weight,
                weight_scale=layer.weight_scale,
                logical_widths=layer.logical_widths,
            )

            if is_fp8_fnuz():
                input_scale = getattr(layer, "input_scale", None)

                weight, max_w_scale, input_scale = normalize_e4m3fn_to_e4m3fnuz(
                    weight=weight, weight_scale=max_w_scale, input_scale=input_scale
                )
                if input_scale is not None:
                    layer.input_scale = Parameter(input_scale, requires_grad=False)
            layer.weight = Parameter(weight.t(), requires_grad=False)
            layer.weight_scale = Parameter(max_w_scale, requires_grad=False)

        elif self.strategy == QuantizationStrategy.CHANNEL:
            weight = layer.weight

            if is_fp8_fnuz():
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

            if _use_aiter:
                # keep the weight as (N, K)
                layer.weight = Parameter(
                    shuffle_weight(weight, (16, 16)), requires_grad=False
                )
            else:
                layer.weight = Parameter(weight.t(), requires_grad=False)

            # required by torch.compile to be torch.nn.Parameter
            layer.weight_scale = Parameter(weight_scale, requires_grad=False)

        elif self.strategy == QuantizationStrategy.BLOCK:
            assert self.is_static_input_scheme is False
            weight = layer.weight
            weight_scale = layer.weight_scale

            if is_fp8_fnuz():
                weight, weight_scale, _ = normalize_e4m3fn_to_e4m3fnuz(
                    weight=weight, weight_scale=weight_scale
                )
            layer.weight = Parameter(weight.data, requires_grad=False)
            layer.weight_scale = Parameter(weight_scale.data, requires_grad=False)

        else:
            raise ValueError(f"Unknown quantization strategy {self.strategy}")

        # INPUT SCALE
        if self.is_static_input_scheme and hasattr(layer, "input_scale"):
            layer.input_scale = Parameter(layer.input_scale.max(), requires_grad=False)
        else:
            layer.input_scale = None
```
**EN:** This block defines `CompressedTensorsW8A8Fp8.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Fp8.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 210-245: CompressedTensorsW8A8Fp8.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if self.weight_block_size is not None:
            return self.w8a8_block_fp8_linear(
                input=x,
                weight=layer.weight,
                block_size=self.weight_block_size,
                weight_scale=layer.weight_scale,
                input_scale=layer.input_scale,
                bias=bias,
            )

        if _use_aiter and self.strategy == QuantizationStrategy.CHANNEL:
            return apply_fp8_ptpc_linear(
                input=x,
                weight=layer.weight,
                weight_scale=layer.weight_scale,
                input_scale=layer.input_scale,
                bias=bias,
                use_per_token_if_dynamic=True,
                compressed_tensor_quant=True,
            )
        else:
            return apply_fp8_linear(
                input=x,
                weight=layer.weight,
                weight_scale=layer.weight_scale,
                input_scale=layer.input_scale,
                bias=bias,
                use_per_token_if_dynamic=True,
                compressed_tensor_quant=True,
            )
```
**EN:** This block defines `CompressedTensorsW8A8Fp8.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Fp8.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `CompressedTensorsW8A8Fp8`: A scheme class that structures file-level quantization behavior. / `CompressedTensorsW8A8Fp8` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `aiter.ops.shuffle`, `compressed_tensors.quantization`, `torch`, `torch.nn`, `typing`
- **Internal / 内部**: `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
