# compressed_tensors_w8a8_int8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_w8a8_int8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w8a8 int8 quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 W8A8 INT8 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: module imports and setup
```python
# Adapted from https://github.com/vllm-project/vllm/tree/main/vllm/model_executor/layers/quantization/compressed_tensors
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from typing import Callable, Optional

import torch
from compressed_tensors.quantization import QuantizationStrategy
from torch.nn import Parameter

from sglang.srt.hardware_backend.npu.quantization.linear_method_npu import (
    NPUW8A8Int8DynamicLinearMethod,
)
from sglang.srt.layers.parameter import (
    ChannelQuantScaleParameter,
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    CompressedTensorsLinearScheme,
)
from sglang.srt.layers.quantization.int8_kernel import per_token_quant_int8
from sglang.srt.layers.quantization.utils import requantize_with_max_scale
from sglang.srt.utils import is_cuda
```
**EN:** This block imports compressed_tensors.quantization, sgl_kernel, torch, torch.nn, sglang.srt.hardware_backend.npu.quantization.linear_method_npu, sglang.srt.layers.parameter, sglang.srt.layers.quantization.compressed_tensors.schemes, sglang.srt.layers.quantization.int8_kernel and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 compressed_tensors.quantization, sgl_kernel, torch, torch.nn, sglang.srt.hardware_backend.npu.quantization.linear_method_npu, sglang.srt.layers.parameter, sglang.srt.layers.quantization.compressed_tensors.schemes, sglang.srt.layers.quantization.int8_kernel 等依赖，并为当前量化实现准备模块命名空间。

### Lines 26-26: initialize __all__
```python
__all__ = ["CompressedTensorsW8A8Int8", "NPUCompressedTensorsW8A8Int8"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 28-28: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 29-30: conditional logic for _is_cuda
```python
if _is_cuda:
    from sgl_kernel import int8_scaled_mm
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 33-34: class CompressedTensorsW8A8Int8: definition
```python
class CompressedTensorsW8A8Int8(CompressedTensorsLinearScheme):
```
**EN:** This block declares `CompressedTensorsW8A8Int8`, a scheme class for the quantization stack. It organizes behaviors such as __init__, create_weights, get_min_capability, process_weights_after_loading.
**CN:** 该代码块声明 `CompressedTensorsW8A8Int8`，它是量化栈中的执行方案类，组织了 __init__, create_weights, get_min_capability, process_weights_after_loading 等行为。

### Lines 35-40: CompressedTensorsW8A8Int8.__init__()
```python
    def __init__(
        self, strategy: str, is_static_input_scheme: bool, input_symmetric: bool
    ):
        self.strategy = strategy
        self.is_static_input_scheme = is_static_input_scheme
        self.input_symmetric = input_symmetric
```
**EN:** This block defines `CompressedTensorsW8A8Int8.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Int8.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 42-95: CompressedTensorsW8A8Int8.create_weights()
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
                output_size_per_partition, input_size_per_partition, dtype=torch.int8
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )

        layer.register_parameter("weight", weight)

        # WEIGHT SCALE
        if self.strategy == QuantizationStrategy.CHANNEL:
            weight_scale = ChannelQuantScaleParameter(
                data=torch.empty((sum(output_partition_sizes), 1), dtype=torch.float32),
                output_dim=0,
                weight_loader=weight_loader,
            )
        else:
            assert self.strategy == QuantizationStrategy.TENSOR
            weight_scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )
        layer.register_parameter("weight_scale", weight_scale)

        # INPUT SCALE
        if self.is_static_input_scheme:
            input_scale = PerTensorScaleParameter(
                data=torch.empty(1, dtype=torch.float32), weight_loader=weight_loader
            )
            layer.register_parameter("input_scale", input_scale)

            if not self.input_symmetric:
                # Note: compressed-tensors stores the zp using the same dtype
                # as the weights
                # AZP loaded as int8 but used as int32
                input_zero_point = PerTensorScaleParameter(
                    data=torch.empty(1, dtype=torch.int8), weight_loader=weight_loader
                )
                layer.register_parameter("input_zero_point", input_zero_point)
```
**EN:** This block defines `CompressedTensorsW8A8Int8.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Int8.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 97-100: CompressedTensorsW8A8Int8.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # ampere and up
        return 80
```
**EN:** This block defines `CompressedTensorsW8A8Int8.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Int8.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 102-169: CompressedTensorsW8A8Int8.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer) -> None:
        # If per tensor, when we have a fused module (e.g. QKV) with per
        # tensor scales (thus N scales being passed to the kernel),
        # requantize so we can always run per channel
        if self.strategy == QuantizationStrategy.TENSOR:
            max_w_scale, weight = requantize_with_max_scale(
                weight=layer.weight,
                weight_scale=layer.weight_scale,
                logical_widths=layer.logical_widths,
            )

            layer.weight = Parameter(weight.t(), requires_grad=False)
            layer.weight_scale = Parameter(max_w_scale, requires_grad=False)

        # If channelwise, scales are already lined up, so just transpose.
        elif self.strategy == QuantizationStrategy.CHANNEL:
            weight = layer.weight
            weight_scale = layer.weight_scale.data

            layer.weight = Parameter(weight.t(), requires_grad=False)
            # required by torch.compile to be torch.nn.Parameter
            layer.weight_scale = Parameter(weight_scale, requires_grad=False)

        else:
            raise ValueError(f"Unknown quantization strategy {self.strategy}")

        # INPUT SCALE
        if self.is_static_input_scheme and hasattr(layer, "input_scale"):
            if self.input_symmetric:
                layer.input_scale = Parameter(
                    layer.input_scale.max(), requires_grad=False
                )
            else:
                input_scale = layer.input_scale
                input_zero_point = layer.input_zero_point

                # reconstruct the ranges
                int8_traits = torch.iinfo(torch.int8)
                azps = input_zero_point.to(dtype=torch.int32)
                range_max = (input_scale * (int8_traits.max - azps)).max()
                range_min = (input_scale * (int8_traits.min - azps)).min()

                scale = (range_max - range_min) / (int8_traits.max - int8_traits.min)

                # AZP loaded as int8 but used as int32
                azp = (int8_traits.min - range_min / scale).to(dtype=torch.int32)

                layer.input_scale = Parameter(scale, requires_grad=False)
                layer.input_zero_point = Parameter(azp, requires_grad=False)
        else:
            layer.input_scale = None
            layer.input_zero_point = None

        # azp_adj is the AZP adjustment term, used to account for weights.
        # It does not depend on scales or azp, so it is the same for
        # static and dynamic quantization.
        # For more details, see csrc/quantization/cutlass_w8a8/Epilogues.md
        # https://github.com/vllm-project/vllm/blob/8d59dbb00044a588cab96bcdc028006ed922eb06/csrc/quantization/cutlass_w8a8/Epilogues.md
        if not self.input_symmetric:
            weight = layer.weight
            azp_adj = weight.sum(dim=0, keepdim=True, dtype=torch.int32)
            if self.is_static_input_scheme:
                # cutlass_w8a8 requires azp to be folded into azp_adj
                # in the per-tensor case
                azp_adj = layer.input_zero_point * azp_adj
            layer.azp_adj = Parameter(azp_adj, requires_grad=False)
        else:
            layer.azp_adj = None
```
**EN:** This block defines `CompressedTensorsW8A8Int8.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Int8.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 171-179: CompressedTensorsW8A8Int8.apply_weights()
```python
    def apply_weights(
        self, layer: torch.nn.Module, x: torch.Tensor, bias: Optional[torch.Tensor]
    ) -> torch.Tensor:
        # TODO: add cutlass_scaled_mm_azp support
        x_q, x_scale = per_token_quant_int8(x)

        return int8_scaled_mm(
            x_q, layer.weight, x_scale, layer.weight_scale, out_dtype=x.dtype, bias=bias
        )
```
**EN:** This block defines `CompressedTensorsW8A8Int8.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Int8.apply_weights()`，用于将量化计算应用到运行时输入上。

### Lines 182-183: class NPUCompressedTensorsW8A8Int8: definition
```python
class NPUCompressedTensorsW8A8Int8(CompressedTensorsW8A8Int8):
```
**EN:** This block declares `NPUCompressedTensorsW8A8Int8`, a supporting class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, process_weights_after_loading, apply_weights.
**CN:** 该代码块声明 `NPUCompressedTensorsW8A8Int8`，它是量化栈中的支撑类，组织了 __init__, get_min_capability, process_weights_after_loading, apply_weights 等行为。

### Lines 184-194: NPUCompressedTensorsW8A8Int8.__init__()
```python
    def __init__(
        self, strategy: str, is_static_input_scheme: bool, input_symmetric: bool
    ):
        super().__init__(strategy, is_static_input_scheme, input_symmetric)
        # TODO: Currently, NPU kernel for static quant requires quant_bias field,
        # which can't be replicated in compressed-tensors.
        if self.is_static_input_scheme:
            raise NotImplementedError(
                "Static compressed-tensors scheme is not yet supported on NPU."
            )
        self.kernel = NPUW8A8Int8DynamicLinearMethod()
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 196-198: NPUCompressedTensorsW8A8Int8.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return NotImplementedError
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 200-201: NPUCompressedTensorsW8A8Int8.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer):
        return self.kernel.process_weights_after_loading(layer)
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 203-204: NPUCompressedTensorsW8A8Int8.apply_weights()
```python
    def apply_weights(self, layer, x, bias):
        return self.kernel.apply(layer, x, bias)
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `CompressedTensorsW8A8Int8`: A scheme class that structures file-level quantization behavior. / `CompressedTensorsW8A8Int8` 是一个执行方案类，用于组织该文件中的量化行为。
- `NPUCompressedTensorsW8A8Int8`: A supporting class that structures file-level quantization behavior. / `NPUCompressedTensorsW8A8Int8` 是一个支撑类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `compressed_tensors.quantization`, `sgl_kernel`, `torch`, `torch.nn`, `typing`
- **Internal / 内部**: `sglang.srt.hardware_backend.npu.quantization.linear_method_npu`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.layers.quantization.int8_kernel`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
