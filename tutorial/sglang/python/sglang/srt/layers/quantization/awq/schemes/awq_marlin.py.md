# awq_marlin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/awq/schemes/awq_marlin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements scheme classes that describe how awq marlin weights are packed, loaded, and executed. / 该模块实现了执行方案类，用于描述 AWQ Marlin 权重如何打包、加载与执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.hardware_backend.gpu.quantization.awq_kernels import (
    AWQMarlinLinearKernel,
)
from sglang.srt.layers.parameter import GroupQuantScaleParameter, PackedvLLMParameter
from sglang.srt.layers.quantization.marlin_utils import verify_marlin_supports_shape

from .awq_scheme import AWQLinearSchemeBase
```
**EN:** This block imports __future__, torch, typing, .awq_scheme, sglang.srt.hardware_backend.gpu.quantization.awq_kernels, sglang.srt.layers.parameter, sglang.srt.layers.quantization.awq.awq and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, typing, .awq_scheme, sglang.srt.hardware_backend.gpu.quantization.awq_kernels, sglang.srt.layers.parameter, sglang.srt.layers.quantization.awq.awq 等依赖，并为当前量化实现准备模块命名空间。

### Lines 16-17: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.quantization.awq.awq import AWQMarlinConfig
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 19-19: initialize __all__
```python
__all__ = ["AWQMarlinLinearScheme"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 22-22: class AWQMarlinLinearScheme: definition
```python
class AWQMarlinLinearScheme(AWQLinearSchemeBase):
```
**EN:** This block declares `AWQMarlinLinearScheme`, a scheme class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply_weights.
**CN:** 该代码块声明 `AWQMarlinLinearScheme`，它是量化栈中的执行方案类，组织了 __init__, create_weights, process_weights_after_loading, apply_weights 等行为。

### Lines 23-25: AWQMarlinLinearScheme.__init__()
```python
    def __init__(self, quant_config: "AWQMarlinConfig"):
        self.quant_config = quant_config
        self.kernel = AWQMarlinLinearKernel(quant_config)
```
**EN:** This block defines `AWQMarlinLinearScheme.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQMarlinLinearScheme.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 27-71: AWQMarlinLinearScheme.create_weights() (part 1/2)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        params_dtype: torch.dtype,
        weight_loader,
        **kwargs,
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)

        group_size = (
            self.quant_config.group_size
            if self.quant_config.group_size != -1
            else input_size
        )

        verify_marlin_supports_shape(
            output_size_per_partition=output_size_per_partition,
            input_size_per_partition=input_size_per_partition,
            input_size=input_size,
            group_size=group_size,
        )

        qweight = PackedvLLMParameter(
            data=torch.empty(
                input_size_per_partition,
                output_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            input_dim=0,
            output_dim=1,
            packed_dim=1,
            packed_factor=self.quant_config.pack_factor,
            weight_loader=weight_loader,
        )

        num_groups = input_size_per_partition // group_size

        qzeros = PackedvLLMParameter(
            data=torch.empty(
                num_groups,
                output_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
```
**EN:** This segment of `AWQMarlinLinearScheme.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `AWQMarlinLinearScheme.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 72-97: AWQMarlinLinearScheme.create_weights() (part 2/2)
```python
            ),
            input_dim=0,
            output_dim=1,
            packed_dim=1,
            packed_factor=self.quant_config.pack_factor,
            weight_loader=weight_loader,
        )

        scales = GroupQuantScaleParameter(
            data=torch.empty(
                num_groups,
                output_size_per_partition,
                dtype=params_dtype,
            ),
            input_dim=0,
            output_dim=1,
            weight_loader=weight_loader,
        )

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("qzeros", qzeros)
        layer.register_parameter("scales", scales)

        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.num_groups = num_groups
```
**EN:** This segment of `AWQMarlinLinearScheme.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `AWQMarlinLinearScheme.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 99-100: AWQMarlinLinearScheme.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self.kernel.process_weights_after_loading(layer)
```
**EN:** This block defines `AWQMarlinLinearScheme.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `AWQMarlinLinearScheme.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 102-105: AWQMarlinLinearScheme.apply_weights()
```python
    def apply_weights(
        self, layer: torch.nn.Module, x: torch.Tensor, bias: Optional[torch.Tensor]
    ):
        return self.kernel.apply(layer, x, bias)
```
**EN:** This block defines `AWQMarlinLinearScheme.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `AWQMarlinLinearScheme.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `AWQMarlinLinearScheme`: A scheme class that structures file-level quantization behavior. / `AWQMarlinLinearScheme` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`, `typing`
- **Internal / 内部**: `.awq_scheme`, `sglang.srt.hardware_backend.gpu.quantization.awq_kernels`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.awq.awq`, `sglang.srt.layers.quantization.marlin_utils`
