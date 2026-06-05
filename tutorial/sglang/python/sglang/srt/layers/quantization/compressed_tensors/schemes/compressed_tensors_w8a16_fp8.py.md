# compressed_tensors_w8a16_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_w8a16_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w8a16 fp8 quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 w8a16 FP8 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: module imports and setup
```python
# Adapted from https://github.com/vllm-project/vllm/tree/main/vllm/model_executor/layers/quantization/compressed_tensors
# SPDX-License-Identifier: Apache-2.0

# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from typing import Callable, List, Optional

import torch
from compressed_tensors.quantization import QuantizationStrategy

from sglang.srt.layers.parameter import (
    ChannelQuantScaleParameter,
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    CompressedTensorsLinearScheme,
)
from sglang.srt.layers.quantization.marlin_utils_fp8 import (
    apply_fp8_marlin_linear,
    prepare_fp8_layer_for_marlin,
)
from sglang.srt.layers.quantization.utils import convert_to_channelwise
```
**EN:** This block imports compressed_tensors.quantization, torch, typing, sglang.srt.layers.parameter, sglang.srt.layers.quantization.compressed_tensors.schemes, sglang.srt.layers.quantization.marlin_utils_fp8, sglang.srt.layers.quantization.utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 compressed_tensors.quantization, torch, typing, sglang.srt.layers.parameter, sglang.srt.layers.quantization.compressed_tensors.schemes, sglang.srt.layers.quantization.marlin_utils_fp8, sglang.srt.layers.quantization.utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 24-24: initialize __all__
```python
__all__ = ["CompressedTensorsW8A16Fp8"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 26-26: initialize SUPPORTED_STRATEGIES
```python
SUPPORTED_STRATEGIES = [QuantizationStrategy.CHANNEL, QuantizationStrategy.TENSOR]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as SUPPORTED_STRATEGIES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 SUPPORTED_STRATEGIES。

### Lines 29-29: class CompressedTensorsW8A16Fp8: definition
```python
class CompressedTensorsW8A16Fp8(CompressedTensorsLinearScheme):
```
**EN:** This block declares `CompressedTensorsW8A16Fp8`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, process_weights_after_loading, create_weights.
**CN:** 该代码块声明 `CompressedTensorsW8A16Fp8`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, process_weights_after_loading, create_weights 等行为。

### Lines 30-32: CompressedTensorsW8A16Fp8.__init__()
```python
    def __init__(self, strategy: str, is_static_input_scheme: bool):
        self.strategy = strategy
        self.is_static_input_scheme = is_static_input_scheme
```
**EN:** This block defines `CompressedTensorsW8A16Fp8.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsW8A16Fp8.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 34-37: CompressedTensorsW8A16Fp8.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # ampere and up
        return 80
```
**EN:** This block defines `CompressedTensorsW8A16Fp8.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsW8A16Fp8.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 42-62: CompressedTensorsW8A16Fp8.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer) -> None:
        if self.strategy == QuantizationStrategy.TENSOR:
            ws_channelwise = convert_to_channelwise(
                layer.weight_scale, layer.logical_widths
            )
            layer.weight_scale = torch.nn.Parameter(ws_channelwise, requires_grad=False)
        else:
            # required by torch.compile to be torch.nn.Parameter
            layer.weight_scale = torch.nn.Parameter(
                layer.weight_scale.data, requires_grad=False
            )

        # Weights must be transposed for marlin
        layer.weight = torch.nn.Parameter(layer.weight.t(), requires_grad=False)

        if self.is_static_input_scheme:
            # required by torch.compile to be torch.nn.Parameter
            layer.input_scale = torch.nn.Parameter(
                layer.input_scale.data, requires_grad=False
            )
        prepare_fp8_layer_for_marlin(layer, size_k_first=True)
```
**EN:** This block defines `CompressedTensorsW8A16Fp8.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `CompressedTensorsW8A16Fp8.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 64-120: CompressedTensorsW8A16Fp8.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size: int,
        output_partition_sizes: List[int],
        input_size_per_partition: int,
        params_dtype: torch.dtype,
        weight_loader: Callable,
        **kwargs,
    ):
        output_size_per_partition = sum(output_partition_sizes)
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype

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
                data=torch.empty((sum(output_partition_sizes), 1), dtype=torch.float32),
                output_dim=0,
                weight_loader=weight_loader,
            )
        elif self.strategy == QuantizationStrategy.TENSOR:
            weight_scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )
        else:
            raise ValueError(
                f"Unsupported weight strategy={self.strategy}, "
                f"supported strategies are {SUPPORTED_STRATEGIES}"
            )

        weight_scale[:] = torch.finfo(torch.float32).min
        layer.register_parameter("weight_scale", weight_scale)

        # INPUT SCALE (to deal with converted checkpoints)
        if self.is_static_input_scheme:
            input_scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )
            layer.register_parameter("input_scale", input_scale)
```
**EN:** This block defines `CompressedTensorsW8A16Fp8.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsW8A16Fp8.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 122-136: CompressedTensorsW8A16Fp8.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return apply_fp8_marlin_linear(
            input=x,
            weight=layer.weight,
            weight_scale=layer.weight_scale,
            workspace=layer.workspace,
            size_n=layer.output_size_per_partition,
            size_k=layer.input_size_per_partition,
            bias=bias,
        )
```
**EN:** This block defines `CompressedTensorsW8A16Fp8.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsW8A16Fp8.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `SUPPORTED_STRATEGIES`: A module-level constant or registry. / `SUPPORTED_STRATEGIES`：模块级常量或注册表。
- `CompressedTensorsW8A16Fp8`: A scheme class that structures file-level quantization behavior. / `CompressedTensorsW8A16Fp8` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `compressed_tensors.quantization`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.layers.quantization.marlin_utils_fp8`, `sglang.srt.layers.quantization.utils`
