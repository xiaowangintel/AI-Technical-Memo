# compressed_tensors_w8a8_int8_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_w8a8_int8_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w8a8 int8 moe quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 W8A8 INT8 MoE 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import torch
from compressed_tensors.quantization import QuantizationStrategy

from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
    NPUW8A8Int8DynamicMoEMethod,
)
from sglang.srt.layers.moe import MoeRunnerConfig
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    CompressedTensorsMoEScheme,
)
from sglang.srt.utils import set_weight_attrs
```
**EN:** This block imports __future__, compressed_tensors.quantization, logging, torch, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.token_dispatcher and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, compressed_tensors.quantization, logging, torch, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.token_dispatcher 等依赖，并为当前量化实现准备模块命名空间。

### Lines 18-22: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 24-24: initialize __all__
```python
__all__ = ["NPUCompressedTensorsW8A8Int8DynamicMoE"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 26-26: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 29-30: class NPUCompressedTensorsW8A8Int8DynamicMoE: definition
```python
class NPUCompressedTensorsW8A8Int8DynamicMoE(CompressedTensorsMoEScheme):
```
**EN:** This block declares `NPUCompressedTensorsW8A8Int8DynamicMoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `NPUCompressedTensorsW8A8Int8DynamicMoE`，它是量化栈中的执行方案类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 31-53: NPUCompressedTensorsW8A8Int8DynamicMoE.__init__()
```python
    def __init__(self, weight_quant, input_quant):
        self.weight_quant = weight_quant
        self.input_quant = input_quant
        self.kernel = NPUW8A8Int8DynamicMoEMethod()

        self.static_input_scales = not self.input_quant.dynamic
        per_channel = (
            self.weight_quant.strategy == QuantizationStrategy.CHANNEL
            and self.input_quant.strategy == QuantizationStrategy.TOKEN
        )
        if not per_channel:
            raise ValueError(
                "For INT8 Fused MoE layers, we require channelwise, "
                "dynamic per token quantization. Found "
                f"{self.weight_quant}, {self.input_quant}"
            )

        self.static_input_scales = not self.input_quant.dynamic
        if self.static_input_scales:
            raise ValueError(
                "For INT8 Fused MoE layers, we require channelwise, "
                "dynamic per token quantization. Found static input scales."
            )
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8DynamicMoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8DynamicMoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 55-118: NPUCompressedTensorsW8A8Int8DynamicMoE.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):

        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        params_dtype = torch.int8

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # WEIGHT_SCALES
        assert self.weight_quant.strategy == QuantizationStrategy.CHANNEL
        w13_weight_scale = torch.nn.Parameter(
            torch.ones(
                num_experts, 2 * intermediate_size_per_partition, 1, dtype=torch.float32
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        w2_weight_scale = torch.nn.Parameter(
            torch.ones(num_experts, hidden_size, 1, dtype=torch.float32),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
        # Add PER-CHANNEL quantization for FusedMoE.weight_loader.
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.CHANNEL.value}
        )
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        # INPUT_SCALES
        assert not self.static_input_scales
        layer.w13_input_scale = None
        layer.w2_input_scale = None
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8DynamicMoE.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8DynamicMoE.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 120-121: NPUCompressedTensorsW8A8Int8DynamicMoE.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self.kernel.process_weights_after_loading(layer)
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8DynamicMoE.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8DynamicMoE.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 123-126: NPUCompressedTensorsW8A8Int8DynamicMoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8DynamicMoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8DynamicMoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 128-134: NPUCompressedTensorsW8A8Int8DynamicMoE.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        return self.kernel.apply(layer, dispatch_output)
```
**EN:** This block defines `NPUCompressedTensorsW8A8Int8DynamicMoE.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW8A8Int8DynamicMoE.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `NPUCompressedTensorsW8A8Int8DynamicMoE`: A scheme class that structures file-level quantization behavior. / `NPUCompressedTensorsW8A8Int8DynamicMoE` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `compressed_tensors.quantization`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.utils`
