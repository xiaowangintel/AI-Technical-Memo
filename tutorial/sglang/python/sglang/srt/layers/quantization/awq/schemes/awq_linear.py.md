# awq_linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/awq/schemes/awq_linear.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements scheme classes that describe how awq linear weights are packed, loaded, and executed. / 该模块实现了执行方案类，用于描述 AWQ 线性 权重如何打包、加载与执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
from __future__ import annotations

from typing import TYPE_CHECKING, List, Optional

import torch

from sglang.srt.layers.parameter import GroupQuantScaleParameter, PackedvLLMParameter

from .awq_scheme import AWQLinearSchemeBase
```
**EN:** This block imports __future__, torch, typing, .awq_scheme, sglang.srt.hardware_backend.gpu.quantization.awq_kernels, sglang.srt.hardware_backend.npu.quantization.awq_kernels, sglang.srt.layers.parameter and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, typing, .awq_scheme, sglang.srt.hardware_backend.gpu.quantization.awq_kernels, sglang.srt.hardware_backend.npu.quantization.awq_kernels, sglang.srt.layers.parameter 等依赖，并为当前量化实现准备模块命名空间。

### Lines 12-13: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.quantization.awq.awq import AWQConfig
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 15-15: initialize __all__
```python
__all__ = ["AWQLinearScheme", "AWQAscendLinearScheme"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 18-18: class AWQLinearScheme: definition
```python
class AWQLinearScheme(AWQLinearSchemeBase):
```
**EN:** This block declares `AWQLinearScheme`, a scheme class for the quantization stack. It organizes behaviors such as __init__, _init_kernel, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `AWQLinearScheme`，它是量化栈中的执行方案类，组织了 __init__, _init_kernel, create_weights, process_weights_after_loading 等行为。

### Lines 19-21: AWQLinearScheme.__init__()
```python
    def __init__(self, quant_config: "AWQConfig"):
        self.quant_config = quant_config
        self.kernel = self._init_kernel(quant_config)
```
**EN:** This block defines `AWQLinearScheme.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQLinearScheme.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 23-28: AWQLinearScheme._init_kernel()
```python
    def _init_kernel(self, quant_config: "AWQConfig"):
        from sglang.srt.hardware_backend.gpu.quantization.awq_kernels import (
            AWQLinearKernel,
        )

        return AWQLinearKernel(quant_config)
```
**EN:** This block defines `AWQLinearScheme._init_kernel()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `AWQLinearScheme._init_kernel()`，用于实现量化栈中的可复用模块逻辑。

### Lines 30-93: AWQLinearScheme.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: List[int],
        params_dtype: torch.dtype,
        weight_loader,
        **kwargs,
    ):
        if input_size_per_partition % self.quant_config.group_size != 0:
            raise ValueError(
                "The input size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size."
            )

        output_size_per_partition = sum(output_partition_sizes)
        if output_size_per_partition % self.quant_config.pack_factor != 0:
            raise ValueError(
                "The output size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size."
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

        qzeros = PackedvLLMParameter(
            data=torch.empty(
                input_size_per_partition // self.quant_config.group_size,
                output_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            input_dim=0,
            output_dim=1,
            packed_dim=1,
            packed_factor=self.quant_config.pack_factor,
            weight_loader=weight_loader,
        )

        scales = GroupQuantScaleParameter(
            data=torch.empty(
                input_size_per_partition // self.quant_config.group_size,
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
```
**EN:** This block defines `AWQLinearScheme.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `AWQLinearScheme.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 95-96: AWQLinearScheme.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self.kernel.process_weights_after_loading(layer)
```
**EN:** This block defines `AWQLinearScheme.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `AWQLinearScheme.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 98-101: AWQLinearScheme.apply_weights()
```python
    def apply_weights(
        self, layer: torch.nn.Module, x: torch.Tensor, bias: Optional[torch.Tensor]
    ):
        return self.kernel.apply(layer, x, bias)
```
**EN:** This block defines `AWQLinearScheme.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `AWQLinearScheme.apply_weights()`，用于将量化计算应用到运行时输入上。

### Lines 104-104: class AWQAscendLinearScheme: definition
```python
class AWQAscendLinearScheme(AWQLinearScheme):
```
**EN:** This block declares `AWQAscendLinearScheme`, a scheme class for the quantization stack. It organizes behaviors such as _init_kernel.
**CN:** 该代码块声明 `AWQAscendLinearScheme`，它是量化栈中的执行方案类，组织了 _init_kernel 等行为。

### Lines 105-110: AWQAscendLinearScheme._init_kernel()
```python
    def _init_kernel(self, quant_config: "AWQConfig"):
        from sglang.srt.hardware_backend.npu.quantization.awq_kernels import (
            AWQAscendLinearKernel,
        )

        return AWQAscendLinearKernel(quant_config)
```
**EN:** This block defines `AWQAscendLinearScheme._init_kernel()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `AWQAscendLinearScheme._init_kernel()`，用于实现量化栈中的可复用模块逻辑。

## Key Concepts / 关键概念
- `AWQLinearScheme`: A scheme class that structures file-level quantization behavior. / `AWQLinearScheme` 是一个执行方案类，用于组织该文件中的量化行为。
- `AWQAscendLinearScheme`: A scheme class that structures file-level quantization behavior. / `AWQAscendLinearScheme` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`, `typing`
- **Internal / 内部**: `.awq_scheme`, `sglang.srt.hardware_backend.gpu.quantization.awq_kernels`, `sglang.srt.hardware_backend.npu.quantization.awq_kernels`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.awq.awq`
