# compressed_tensors_w4a8_int8_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_w4a8_int8_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w4a8 int8 moe quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 w4a8 INT8 MoE 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import torch

from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
    NPUW4A8Int8DynamicMoEMethod,
)
from sglang.srt.layers.moe import MoeRunnerConfig
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    CompressedTensorsMoEScheme,
)
from sglang.srt.utils import set_weight_attrs
```
**EN:** This block imports __future__, logging, torch, typing, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.token_dispatcher and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, torch, typing, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.token_dispatcher 等依赖，并为当前量化实现准备模块命名空间。

### Lines 17-21: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 23-23: initialize __all__
```python
__all__ = ["NPUCompressedTensorsW4A8Int8DynamicMoE"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 26-26: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 29-31: class NPUCompressedTensorsW4A8Int8DynamicMoE: definition
```python
class NPUCompressedTensorsW4A8Int8DynamicMoE(CompressedTensorsMoEScheme):

    ### TODO: Get rid of code duplication with python/sglang/srt/modelslim/modelslim_moe.py @OrangeRedeng @TamirBaydasov
```
**EN:** This block declares `NPUCompressedTensorsW4A8Int8DynamicMoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, create_weights, _init_activation_clip_params, _init_extra_scale_params.
**CN:** 该代码块声明 `NPUCompressedTensorsW4A8Int8DynamicMoE`，它是量化栈中的执行方案类，组织了 __init__, create_weights, _init_activation_clip_params, _init_extra_scale_params 等行为。

### Lines 32-41: NPUCompressedTensorsW4A8Int8DynamicMoE.__init__()
```python
    def __init__(self, quantization_config) -> None:
        self.group_size = 0
        self.is_per_channel_weight = self.group_size == 0
        self.tp_size = 1
        self.activation_use_clip = (
            quantization_config.get("config_groups", {})
            .get("group_1", {})
            .get("activation_use_clip", False)
        )
        self.kernel = NPUW4A8Int8DynamicMoEMethod()
```
**EN:** This block defines `NPUCompressedTensorsW4A8Int8DynamicMoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A8Int8DynamicMoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 43-87: NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights() (part 1/3)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ) -> None:
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        self.num_experts = num_experts
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.CHANNEL.value}
        )

        # >> weight
        w13_output_size = intermediate_size_per_partition
        w2_output_size = hidden_size // 2
        w13_weight = torch.nn.Parameter(
            torch.empty(num_experts, w13_output_size, hidden_size, dtype=torch.int8),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)
        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                w2_output_size,
                intermediate_size_per_partition,
                dtype=torch.int8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # >> scale
        weight_scale_dtype = torch.int64 if self.activation_use_clip else torch.float32
        w13_weight_scale = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                1,
                dtype=weight_scale_dtype,
```
**EN:** This segment of `NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 88-132: NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights() (part 2/3)
```python
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)

        w2_weight_scale = torch.nn.Parameter(
            torch.empty(num_experts, hidden_size, 1, dtype=weight_scale_dtype),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        # >> offset
        w13_weight_offset = torch.nn.Parameter(
            torch.empty(
                num_experts, 2 * intermediate_size_per_partition, 1, dtype=torch.float32
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_offset", w13_weight_offset)
        set_weight_attrs(w13_weight_offset, extra_weight_attrs)

        w2_weight_offset = torch.nn.Parameter(
            torch.empty(num_experts, hidden_size, 1, dtype=torch.float32),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_offset", w2_weight_offset)
        set_weight_attrs(w2_weight_offset, extra_weight_attrs)

        # >>> special param for w4a8
        if self.activation_use_clip:
            self._init_activation_clip_params(
                layer,
                num_experts,
                hidden_size,
                intermediate_size_per_partition,
                extra_weight_attrs,
            )
        else:
            self._init_extra_scale_params(
                layer,
                num_experts,
                hidden_size,
                intermediate_size_per_partition,
```
**EN:** This segment of `NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 133-134: NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights() (part 3/3)
```python
                extra_weight_attrs,
            )
```
**EN:** This segment of `NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `NPUCompressedTensorsW4A8Int8DynamicMoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 136-171: NPUCompressedTensorsW4A8Int8DynamicMoE._init_activation_clip_params()
```python
    def _init_activation_clip_params(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        extra_weight_attrs: dict,
    ) -> None:
        """
        Initializes bias and alpha parameters for quantization schemes that use activation clipping.

        This helper registers `w13_bias`, `w2_bias`, and `w2_alpha`, which are required to
        shift and scale the activations or outputs to compensate for the precision loss
        introduced by clamping activations.
        """
        w13_bias = torch.nn.Parameter(
            torch.ones(
                num_experts, 2 * intermediate_size_per_partition, dtype=torch.float
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_bias", w13_bias)
        set_weight_attrs(w13_bias, extra_weight_attrs)

        w2_bias = torch.nn.Parameter(
            torch.ones(num_experts, hidden_size, dtype=torch.float),
            requires_grad=False,
        )
        layer.register_parameter("w2_bias", w2_bias)
        set_weight_attrs(w2_bias, extra_weight_attrs)

        w2_alpha = torch.nn.Parameter(
            torch.ones(num_experts, dtype=torch.float), requires_grad=False
        )
        layer.register_parameter("w2_alpha", w2_alpha)
        set_weight_attrs(w2_alpha, extra_weight_attrs)
```
**EN:** This block defines `NPUCompressedTensorsW4A8Int8DynamicMoE._init_activation_clip_params()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A8Int8DynamicMoE._init_activation_clip_params()`，用于实现量化栈中的可复用模块逻辑。

### Lines 173-217: NPUCompressedTensorsW4A8Int8DynamicMoE._init_extra_scale_params() (part 1/2)
```python
    def _init_extra_scale_params(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        extra_weight_attrs: dict,
    ) -> None:
        """
        Initializes additional scaling, offset, and bias parameters for quantization schemes without activation clipping.

        This method registers the following parameters:
        1. Scale Biases: `w13_scale_bias` and `w2_scale_bias`.
        2. Secondary Quantization Params (initialized only for grouped quantization):
            `w13_weight_scale_second`, `w13_weight_offset_second`,
            `w2_weight_scale_second`, and `w2_weight_offset_second`.
        """
        if not self.is_per_channel_weight:
            w13_weight_scale_second = torch.nn.Parameter(
                torch.empty(
                    num_experts,
                    2 * intermediate_size_per_partition,
                    hidden_size // self.group_size,
                    dtype=torch.float32,
                ),
                requires_grad=False,
            )
            layer.register_parameter("w13_weight_scale_second", w13_weight_scale_second)
            set_weight_attrs(w13_weight_scale_second, extra_weight_attrs)

            w13_weight_offset_second = torch.nn.Parameter(
                torch.empty(
                    num_experts,
                    2 * intermediate_size_per_partition,
                    hidden_size // self.group_size,
                    dtype=torch.float32,
                ),
                requires_grad=False,
            )
            layer.register_parameter(
                "w13_weight_offset_second", w13_weight_offset_second
            )
            set_weight_attrs(w13_weight_offset_second, extra_weight_attrs)

            w2_weight_scale_second = torch.nn.Parameter(
```
**EN:** This segment of `NPUCompressedTensorsW4A8Int8DynamicMoE._init_extra_scale_params()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `NPUCompressedTensorsW4A8Int8DynamicMoE._init_extra_scale_params()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 218-257: NPUCompressedTensorsW4A8Int8DynamicMoE._init_extra_scale_params() (part 2/2)
```python
                torch.empty(
                    num_experts,
                    hidden_size,
                    intermediate_size_per_partition // self.group_size,
                    dtype=torch.float32,
                ),
                requires_grad=False,
            )
            layer.register_parameter("w2_weight_scale_second", w2_weight_scale_second)
            set_weight_attrs(w2_weight_scale_second, extra_weight_attrs)

            w2_weight_offset_second = torch.nn.Parameter(
                torch.empty(
                    num_experts,
                    hidden_size,
                    intermediate_size_per_partition // self.group_size,
                    dtype=torch.float32,
                ),
                requires_grad=False,
            )
            layer.register_parameter("w2_weight_offset_second", w2_weight_offset_second)
            set_weight_attrs(w2_weight_offset_second, extra_weight_attrs)

        w13_scale_bias = torch.nn.Parameter(
            torch.empty(
                num_experts, 2 * intermediate_size_per_partition, 1, dtype=torch.float32
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_scale_bias", w13_scale_bias)
        set_weight_attrs(w13_scale_bias, extra_weight_attrs)

        w2_scale_bias = torch.nn.Parameter(
            torch.empty(
                num_experts, hidden_size, 16 // self.tp_size, dtype=torch.float32
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_scale_bias", w2_scale_bias)
        set_weight_attrs(w2_scale_bias, extra_weight_attrs)
```
**EN:** This segment of `NPUCompressedTensorsW4A8Int8DynamicMoE._init_extra_scale_params()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `NPUCompressedTensorsW4A8Int8DynamicMoE._init_extra_scale_params()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 259-262: NPUCompressedTensorsW4A8Int8DynamicMoE.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self.kernel.process_weights_after_loading(
            layer, self.is_per_channel_weight, self.activation_use_clip
        )
```
**EN:** This block defines `NPUCompressedTensorsW4A8Int8DynamicMoE.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A8Int8DynamicMoE.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 264-267: NPUCompressedTensorsW4A8Int8DynamicMoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `NPUCompressedTensorsW4A8Int8DynamicMoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A8Int8DynamicMoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 269-275: NPUCompressedTensorsW4A8Int8DynamicMoE.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        return self.kernel.apply(layer, dispatch_output)
```
**EN:** This block defines `NPUCompressedTensorsW4A8Int8DynamicMoE.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A8Int8DynamicMoE.apply_weights()`，用于将量化计算应用到运行时输入上。

### Lines 277-293: NPUCompressedTensorsW4A8Int8DynamicMoE.apply_weights_with_router_logits()
```python
    def apply_weights_with_router_logits(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        return self.kernel.apply_without_routing_weights(
            layer,
            hidden_states,
            hidden_states_scale,
            group_list_type,
            group_list,
            output_dtype,
        )
```
**EN:** This block defines `NPUCompressedTensorsW4A8Int8DynamicMoE.apply_weights_with_router_logits()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A8Int8DynamicMoE.apply_weights_with_router_logits()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `NPUCompressedTensorsW4A8Int8DynamicMoE`: A scheme class that structures file-level quantization behavior. / `NPUCompressedTensorsW4A8Int8DynamicMoE` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.utils`
