# modelslim_w4a4_int4_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/modelslim/schemes/modelslim_w4a4_int4_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for modelslim w4a4 int4 moe quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 ModelSlim w4a4 INT4 MoE 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any, Dict

import torch

from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
    NPUW4A4Int4DynamicMoEMethod,
)
from sglang.srt.layers.quantization.modelslim.schemes import ModelSlimMoEScheme
from sglang.srt.utils import set_weight_attrs
```
**EN:** This block imports __future__, logging, torch, typing, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.token_dispatcher and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, torch, typing, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.token_dispatcher 等依赖，并为当前量化实现准备模块命名空间。

### Lines 14-19: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe import MoeRunnerConfig
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 21-21: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 23-25: initialize __all__
```python
__all__ = [
    "ModelSlimW4A4Int4MoE",
]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 28-29: class ModelSlimW4A4Int4MoE: definition
```python
class ModelSlimW4A4Int4MoE(ModelSlimMoEScheme):
```
**EN:** This block declares `ModelSlimW4A4Int4MoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `ModelSlimW4A4Int4MoE`，它是量化栈中的执行方案类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 30-36: ModelSlimW4A4Int4MoE.__init__()
```python
    def __init__(
        self,
        quant_config: Dict[str, Any],
        prefix: str = None,
    ):
        self.quant_config = quant_config
        self.kernel = NPUW4A4Int4DynamicMoEMethod()
```
**EN:** This block defines `ModelSlimW4A4Int4MoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4MoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 38-106: ModelSlimW4A4Int4MoE.create_weights()
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

        # weight
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
                dtype=torch.int8,
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
                dtype=torch.int8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)
        # scale
        w13_weight_scale = torch.nn.Parameter(
            torch.empty(
                num_experts, 2 * intermediate_size_per_partition, 1, dtype=torch.float32
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)
        w2_weight_scale = torch.nn.Parameter(
            torch.empty(num_experts, hidden_size, 1, dtype=torch.float32),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)
        # offset
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
```
**EN:** This block defines `ModelSlimW4A4Int4MoE.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4MoE.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 108-109: ModelSlimW4A4Int4MoE.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self.kernel.process_weights_after_loading(layer)
```
**EN:** This block defines `ModelSlimW4A4Int4MoE.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4MoE.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 111-114: ModelSlimW4A4Int4MoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: "MoeRunnerConfig"
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `ModelSlimW4A4Int4MoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4MoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 116-121: ModelSlimW4A4Int4MoE.apply_weights()
```python
    def apply_weights(
        self,
        layer,
        dispatch_output: "StandardDispatchOutput",
    ) -> "CombineInput":
        return self.kernel.apply(layer, dispatch_output)
```
**EN:** This block defines `ModelSlimW4A4Int4MoE.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4MoE.apply_weights()`，用于将量化计算应用到运行时输入上。

### Lines 123-143: ModelSlimW4A4Int4MoE.apply_without_routing_weights()
```python
    def apply_without_routing_weights(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        logger.warning_once(
            "Warning: Performance may be reduced, because DeepEP Dispatcher does not support 4-bit quantization, "
            "switching to the bf16 dispatcher, quantization will be performed separately..."
        )
        return self.kernel.apply_without_routing_weights(
            layer,
            hidden_states,
            hidden_states_scale,
            group_list_type,
            group_list,
            output_dtype,
        )
```
**EN:** This block defines `ModelSlimW4A4Int4MoE.apply_without_routing_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `ModelSlimW4A4Int4MoE.apply_without_routing_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `ModelSlimW4A4Int4MoE`: A scheme class that structures file-level quantization behavior. / `ModelSlimW4A4Int4MoE` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.modelslim.schemes`, `sglang.srt.utils`
