# w8a8_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/w8a8_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for w8a8 fp8 quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 W8A8 FP8 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: module imports and setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Any, Dict, List, Optional

import torch
from torch.nn.parameter import Parameter

from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
from sglang.srt.layers.parameter import ChannelQuantScaleParameter, ModelWeightParameter
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.fp8_kernel import (
    fp8_dtype,
    is_fp8_fnuz,
    per_token_group_quant_fp8,
)
from sglang.srt.layers.quantization.fp8_utils import (
    apply_fp8_linear,
    cutlass_fp8_supported,
    input_to_float8,
    normalize_e4m3fn_to_e4m3fnuz,
)
from sglang.srt.utils import set_weight_attrs
```
**EN:** This block imports __future__, torch, torch.nn.parameter, typing, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner.triton and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, torch.nn.parameter, typing, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner.triton 等依赖，并为当前量化实现准备模块命名空间。

### Lines 30-34: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 36-36: initialize _is_fp8_fnuz
```python
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_fp8_fnuz.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_fp8_fnuz。

### Lines 39-57: class W8A8Fp8Config: definition
```python
class W8A8Fp8Config(QuantizationConfig):
    """Config class for W8A8 FP8 Quantization.

    Weight Quantization:
    - Method: Static quantization
    - Granularity: Per-channel
    - Type: Symmetric

    Activation Quantization:
    - Method: Dynamic quantization
    - Granularity: Per-token
    - Type: Symmetric

    Note:
    - For models without offline quantization, weights will be quantized during model loading
    - If CUTLASS is supported: Per-channel weight quantization is used
    - If CUTLASS is not supported: Falls back to per-tensor weight quantization
    """
```
**EN:** This block declares `W8A8Fp8Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_supported_act_dtypes, get_min_capability, get_name.
**CN:** 该代码块声明 `W8A8Fp8Config`，它是量化栈中的配置类，组织了 __init__, get_supported_act_dtypes, get_min_capability, get_name 等行为。

### Lines 58-59: W8A8Fp8Config.__init__()
```python
    def __init__(self, is_checkpoint_fp8_serialized: bool = False):
        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized
```
**EN:** This block defines `W8A8Fp8Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `W8A8Fp8Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 61-63: W8A8Fp8Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.float16, torch.bfloat16]
```
**EN:** This block defines `W8A8Fp8Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Fp8Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 65-67: W8A8Fp8Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 89
```
**EN:** This block defines `W8A8Fp8Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Fp8Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 69-71: W8A8Fp8Config.get_name()
```python
    @classmethod
    def get_name(self) -> str:
        return "w8a8_fp8"
```
**EN:** This block defines `W8A8Fp8Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Fp8Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 73-75: W8A8Fp8Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return []
```
**EN:** This block defines `W8A8Fp8Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Fp8Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 77-83: W8A8Fp8Config.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> W8A8Fp8Config:
        quant_method = cls.get_from_keys(config, ["quant_method"])
        is_checkpoint_fp8_serialized = (
            "compressed-tensors" in quant_method or "w8a8_fp8" in quant_method
        )
        return cls(is_checkpoint_fp8_serialized=is_checkpoint_fp8_serialized)
```
**EN:** This block defines `W8A8Fp8Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `W8A8Fp8Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 85-97: W8A8Fp8Config.get_quant_method()
```python
    def get_quant_method(
        self,
        layer: torch.nn.Module,
        prefix: str,
    ) -> Optional[QuantizeMethodBase]:
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if isinstance(layer, LinearBase):
            return W8A8Fp8LinearMethod(self)
        elif isinstance(layer, FusedMoE):
            return W8A8FP8MoEMethod(self)
        return None
```
**EN:** This block defines `W8A8Fp8Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Fp8Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 99-100: W8A8Fp8Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `W8A8Fp8Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Fp8Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 103-104: class W8A8Fp8LinearMethod: definition
```python
class W8A8Fp8LinearMethod(LinearMethodBase):
```
**EN:** This block declares `W8A8Fp8LinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, process_weights_after_loading, create_weights, apply.
**CN:** 该代码块声明 `W8A8Fp8LinearMethod`，它是量化栈中的运行方法类，组织了 __init__, process_weights_after_loading, create_weights, apply 等行为。

### Lines 105-107: W8A8Fp8LinearMethod.__init__()
```python
    def __init__(self, quantization_config: W8A8Fp8Config):
        self.cutlass_fp8_supported = cutlass_fp8_supported()
        self.quantization_config = quantization_config
```
**EN:** This block defines `W8A8Fp8LinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `W8A8Fp8LinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 109-139: W8A8Fp8LinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        weight = layer.weight

        if self.quantization_config.is_checkpoint_fp8_serialized:
            weight_scale = layer.weight_scale.detach()
            # If checkpoint offline quantized with w8a8_fp8, load the weight and weight_scale directly.
            if _is_fp8_fnuz:
                weight, weight_scale, _ = normalize_e4m3fn_to_e4m3fnuz(
                    weight=weight, weight_scale=weight_scale
                )

            layer.weight = Parameter(weight.t(), requires_grad=False)
            layer.weight_scale = Parameter(weight_scale, requires_grad=False)
        else:
            # If checkpoint not offline quantized, quantize the weights with per-channel quantization.
            if self.cutlass_fp8_supported:
                # if cutlass supported, we use cutlass_scaled_mm
                # which requires per-channel quantization on weight
                qweight, weight_scale = per_token_group_quant_fp8(
                    layer.weight, layer.weight.shape[-1]
                )
                weight_scale = weight_scale.t().contiguous()
            else:
                # if cutlass not supported, we fall back to use torch._scaled_mm
                # which requires per tensor quantization on weight
                qweight, weight_scale = input_to_float8(layer.weight, dtype=fp8_dtype)

            # Update the layer with the new values.
            layer.weight = Parameter(qweight.t(), requires_grad=False)
            layer.weight_scale = Parameter(weight_scale, requires_grad=False)
            layer.input_scale = None
```
**EN:** This block defines `W8A8Fp8LinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `W8A8Fp8LinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 141-180: W8A8Fp8LinearMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: List[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        weight_dtype = (
            torch.float8_e4m3fn
            if self.quantization_config.is_checkpoint_fp8_serialized
            else params_dtype
        )

        weight_loader = extra_weight_attrs.get("weight_loader")
        self.logical_widths = output_partition_sizes

        weight = ModelWeightParameter(
            data=torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition,
                dtype=weight_dtype,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        if self.quantization_config.is_checkpoint_fp8_serialized:
            weight_scale = ChannelQuantScaleParameter(
                data=torch.empty((sum(output_partition_sizes), 1), dtype=torch.float32),
                output_dim=0,
                weight_loader=weight_loader,
            )
            layer.register_parameter("weight_scale", weight_scale)
        else:
            layer.weight_scale = None
```
**EN:** This block defines `W8A8Fp8LinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `W8A8Fp8LinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 182-194: W8A8Fp8LinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ):
        return apply_fp8_linear(
            x,
            layer.weight,
            layer.weight_scale,
            bias=bias,
            cutlass_fp8_supported=self.cutlass_fp8_supported,
        )
```
**EN:** This block defines `W8A8Fp8LinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `W8A8Fp8LinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 197-207: class W8A8FP8MoEMethod: definition
```python
class W8A8FP8MoEMethod(FusedMoEMethodBase):
    """MoE method for FP8.
    Supports loading FP8 checkpoints with static weight scale and
    dynamic/static activation scale.
    Also supports loading quantized FP16/BF16 model checkpoints with dynamic
    activation scaling. The weight scaling factor will be initialized after
    the model weights are loaded.
    Args:
        quant_config: The quantization config.
    """
```
**EN:** This block declares `W8A8FP8MoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `W8A8FP8MoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 208-209: W8A8FP8MoEMethod.__init__()
```python
    def __init__(self, quant_config: W8A8Fp8Config):
        self.quant_config = quant_config
```
**EN:** This block defines `W8A8FP8MoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `W8A8FP8MoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 211-271: W8A8FP8MoEMethod.create_weights()
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

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
                dtype=fp8_dtype,
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
                dtype=fp8_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        w13_weight_scale = torch.nn.Parameter(
            torch.ones(
                num_experts, 2 * intermediate_size_per_partition, 1, dtype=torch.float32
            ),
            requires_grad=False,
        )
        w2_weight_scale = torch.nn.Parameter(
            torch.ones(num_experts, hidden_size, 1, dtype=torch.float32),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        layer.register_parameter("w2_weight_scale", w2_weight_scale)

        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.CHANNEL.value}
        )

        set_weight_attrs(w13_weight_scale, extra_weight_attrs)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        w13_input_scale = None
        layer.register_parameter("w13_input_scale", w13_input_scale)

        w2_input_scale = None
        layer.register_parameter("w2_input_scale", w2_input_scale)
```
**EN:** This block defines `W8A8FP8MoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `W8A8FP8MoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 273-281: W8A8FP8MoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.w13_weight = Parameter(layer.w13_weight, requires_grad=False)
        layer.w2_weight = Parameter(layer.w2_weight, requires_grad=False)
        layer.w13_weight_scale = Parameter(
            layer.w13_weight_scale.data, requires_grad=False
        )
        layer.w2_weight_scale = Parameter(
            layer.w2_weight_scale.data, requires_grad=False
        )
```
**EN:** This block defines `W8A8FP8MoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `W8A8FP8MoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 283-287: W8A8FP8MoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.TRITON, moe_runner_config)
```
**EN:** This block defines `W8A8FP8MoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `W8A8FP8MoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 289-299: W8A8FP8MoEMethod.get_triton_quant_info()
```python
    def get_triton_quant_info(self, layer: torch.nn.Module) -> TritonMoeQuantInfo:
        return TritonMoeQuantInfo(
            w13_weight=layer.w13_weight,
            w2_weight=layer.w2_weight,
            use_fp8_w8a8=True,
            per_channel_quant=True,
            w13_scale=layer.w13_weight_scale,
            w2_scale=layer.w2_weight_scale,
            a13_scale=layer.w13_input_scale,
            a2_scale=layer.w2_input_scale,
        )
```
**EN:** This block defines `W8A8FP8MoEMethod.get_triton_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8FP8MoEMethod.get_triton_quant_info()`，用于为调用方获取或计算派生值。

### Lines 301-308: W8A8FP8MoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        quant_info = self.get_triton_quant_info(layer)
        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `W8A8FP8MoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `W8A8FP8MoEMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `W8A8Fp8Config`: A configuration class that structures file-level quantization behavior. / `W8A8Fp8Config` 是一个配置类，用于组织该文件中的量化行为。
- `W8A8Fp8LinearMethod`: A runtime method class that structures file-level quantization behavior. / `W8A8Fp8LinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `W8A8FP8MoEMethod`: A runtime method class that structures file-level quantization behavior. / `W8A8FP8MoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`, `torch.nn.parameter`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.utils`
