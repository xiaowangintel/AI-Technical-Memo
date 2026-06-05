# w4afp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/w4afp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for w4afp8 quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 W4AFP8 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any, Dict, List, Optional

import torch
from torch.nn import Module
from torch.nn.parameter import Parameter

from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.fp8 import Fp8LinearMethod
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.layers.quantization.utils import is_layer_skipped
from sglang.srt.utils import set_weight_attrs
```
**EN:** This block imports __future__, logging, torch, torch.nn, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.cutlass_w4a8_moe, sglang.srt.layers.moe.ep_moe.layer and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, torch, torch.nn, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.cutlass_w4a8_moe, sglang.srt.layers.moe.ep_moe.layer 等依赖，并为当前量化实现准备模块命名空间。

### Lines 20-28: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe import MoeRunnerConfig
    from sglang.srt.layers.moe.ep_moe.layer import DeepEPMoE
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        DeepEPLLDispatchOutput,
        DeepEPNormalDispatchOutput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 30-30: initialize ACTIVATION_SCHEMES
```python
ACTIVATION_SCHEMES = ["static", "dynamic"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as ACTIVATION_SCHEMES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 ACTIVATION_SCHEMES。

### Lines 32-32: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 35-37: class W4AFp8Config: definition
```python
class W4AFp8Config(QuantizationConfig):
    """Config class for MIXED_PRECISION W4AFp8."""
```
**EN:** This block declares `W4AFp8Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_name, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `W4AFp8Config`，它是量化栈中的配置类，组织了 __init__, get_name, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 38-59: W4AFp8Config.__init__()
```python
    def __init__(
        self,
        is_checkpoint_fp8_serialized: bool = True,
        is_checkpoint_w4afp8_serialized: bool = True,
        linear_activation_scheme: str = "dynamic",
        moe_activation_scheme: str = "static",
        ignored_layers: Optional[List[str]] = None,
        weight_block_size: Optional[List[int]] = None,
        group_size: int = 128,
    ) -> None:
        super().__init__()
        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized
        self.is_checkpoint_w4afp8_serialized = is_checkpoint_w4afp8_serialized
        if is_checkpoint_w4afp8_serialized:
            logger.warning("Detected w4afp8 checkpoint. Please note that")
        if moe_activation_scheme not in ACTIVATION_SCHEMES:
            raise ValueError(f"Unsupported activation scheme {moe_activation_scheme}")
        self.linear_activation_scheme = linear_activation_scheme
        self.moe_activation_scheme = moe_activation_scheme
        self.ignored_layers = ignored_layers or []
        self.weight_block_size = [128, 128]
        self.group_size = group_size
```
**EN:** This block defines `W4AFp8Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `W4AFp8Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 61-63: W4AFp8Config.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "w4afp8"
```
**EN:** This block defines `W4AFp8Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W4AFp8Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 65-67: W4AFp8Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.bfloat16, torch.float8_e4m3fn]
```
**EN:** This block defines `W4AFp8Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W4AFp8Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 69-71: W4AFp8Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 90
```
**EN:** This block defines `W4AFp8Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W4AFp8Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 73-75: W4AFp8Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return []
```
**EN:** This block defines `W4AFp8Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W4AFp8Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 77-91: W4AFp8Config.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> W4AFp8Config:
        quant_method = cls.get_from_keys(config, ["quant_method"])
        is_checkpoint_fp8_serialized = "fp8" in quant_method
        is_checkpoint_w4afp8_serialized = "w4afp8" in quant_method
        linear_activation_scheme = "dynamic"
        moe_activation_scheme = "static"
        weight_block_size = [128, 128]
        return cls(
            is_checkpoint_fp8_serialized=is_checkpoint_fp8_serialized,
            is_checkpoint_w4afp8_serialized=is_checkpoint_w4afp8_serialized,
            linear_activation_scheme=linear_activation_scheme,
            moe_activation_scheme=moe_activation_scheme,
            weight_block_size=weight_block_size,
        )
```
**EN:** This block defines `W4AFp8Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `W4AFp8Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 93-105: W4AFp8Config.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if isinstance(layer, LinearBase):
            if is_layer_skipped(prefix, self.ignored_layers):
                return UnquantizedLinearMethod()
            return Fp8LinearMethod(self)
        elif isinstance(layer, FusedMoE):
            return W4AFp8MoEMethod(self)
        return None
```
**EN:** This block defines `W4AFp8Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W4AFp8Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 107-108: W4AFp8Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `W4AFp8Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W4AFp8Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 111-125: interleave_scales()
```python
def interleave_scales(scales: torch.Tensor) -> torch.Tensor:
    """Interleave scales in groups of 4 similar to TRT-LLM implementation."""
    s_shape = scales.shape
    # Reshape to separate groups of 4
    alignment = 4 if s_shape[2] % 4 == 0 else 1
    scales_interleaved = scales.reshape(
        s_shape[0], s_shape[1], (s_shape[2] // alignment), alignment
    )
    # Permute dimensions to interleave
    scales_interleaved = scales_interleaved.permute(0, 2, 1, 3)
    # Reshape back to original dimensions but with interleaved values
    scales_interleaved = scales_interleaved.reshape(
        s_shape[0], s_shape[2] // alignment, s_shape[1] * alignment
    )
    return scales_interleaved.contiguous()
```
**EN:** This block defines `interleave_scales()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `interleave_scales()`，用于实现量化栈中的可复用模块逻辑。

### Lines 128-128: class W4AFp8MoEMethod: definition
```python
class W4AFp8MoEMethod(FusedMoEMethodBase):
```
**EN:** This block declares `W4AFp8MoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `W4AFp8MoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 129-130: W4AFp8MoEMethod.__init__()
```python
    def __init__(self, quant_config: W4AFp8Config):
        self.quant_config = quant_config
```
**EN:** This block defines `W4AFp8MoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `W4AFp8MoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 132-176: W4AFp8MoEMethod.create_weights() (part 1/3)
```python
    def create_weights(
        self,
        layer: Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        assert "weight_loader" in extra_weight_attrs

        # Fused gate_up_proj (column parallel)
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition * 2,
                hidden_size // 2,
                dtype=torch.int8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        # down_proj (row parallel)
        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // 2,
                dtype=torch.int8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.GROUP.value}
        )
        w13_weight_scale = torch.nn.Parameter(
            torch.zeros(
                num_experts,
```
**EN:** This segment of `W4AFp8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `W4AFp8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 177-221: W4AFp8MoEMethod.create_weights() (part 2/3)
```python
                2 * intermediate_size_per_partition,
                hidden_size // self.quant_config.group_size,
                dtype=torch.float32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale_inv", w13_weight_scale)
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)

        w2_weight_scale = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // self.quant_config.group_size,
                dtype=torch.float32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_scale_inv", w2_weight_scale)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        # Input scales
        w13_input_scale = torch.nn.Parameter(
            torch.ones((num_experts, 2), dtype=torch.bfloat16),
            requires_grad=False,
        )
        layer.register_parameter("w13_input_scale", w13_input_scale)
        set_weight_attrs(w13_input_scale, extra_weight_attrs)

        w2_input_scale = torch.nn.Parameter(
            torch.ones(num_experts, dtype=torch.bfloat16),
            requires_grad=False,
        )
        layer.register_parameter("w2_input_scale", w2_input_scale)
        set_weight_attrs(w2_input_scale, extra_weight_attrs)

        # Pre-populate the strides
        device = layer.w13_weight.device

        self.a_strides1 = torch.full(
            (num_experts, 3),
            hidden_size,
            device=device,
            dtype=torch.int64,
        )
```
**EN:** This segment of `W4AFp8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `W4AFp8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 222-255: W4AFp8MoEMethod.create_weights() (part 3/3)
```python
        self.c_strides1 = torch.full(
            (num_experts, 3),
            2 * intermediate_size_per_partition,
            device=device,
            dtype=torch.int64,
        )
        self.a_strides2 = torch.full(
            (num_experts, 3),
            intermediate_size_per_partition,
            device=device,
            dtype=torch.int64,
        )
        self.c_strides2 = torch.full(
            (num_experts, 3),
            hidden_size,
            device=device,
            dtype=torch.int64,
        )
        self.b_strides1 = self.a_strides1
        self.s_strides13 = self.c_strides1
        self.b_strides2 = self.a_strides2
        self.s_strides2 = self.c_strides2

        self.expert_offsets = torch.empty(
            (num_experts + 1), dtype=torch.int32, device=device
        )
        self.problem_sizes1 = torch.empty(
            (num_experts, 3), dtype=torch.int32, device=device
        )
        self.problem_sizes2 = torch.empty(
            (num_experts, 3), dtype=torch.int32, device=device
        )

        return
```
**EN:** This segment of `W4AFp8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `W4AFp8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 257-284: W4AFp8MoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        dtype = torch.bfloat16
        device = layer.w2_weight.device

        # Interleave w13_weight_scale (gate_up_proj)
        w13_weight_scale = layer.w13_weight_scale_inv.to(dtype)
        w13_weight_scale = interleave_scales(w13_weight_scale)
        layer.w13_weight_scale_inv = Parameter(w13_weight_scale, requires_grad=False)

        # Interleave w2_weight_scale (down_proj)
        w2_weight_scale = layer.w2_weight_scale_inv.to(dtype)
        w2_weight_scale = interleave_scales(w2_weight_scale)
        layer.w2_weight_scale_inv = Parameter(w2_weight_scale, requires_grad=False)

        # Process input scales
        w13_input_scale_max = layer.w13_input_scale.max().to(torch.float32).item()
        new_w13_input_scale = torch.tensor(
            [w13_input_scale_max],
            dtype=torch.float32,
            device=device,
        )
        layer.w13_input_scale = Parameter(new_w13_input_scale, requires_grad=False)

        w2_input_scale_max = layer.w2_input_scale.max().to(torch.float32).item()
        new_w2_input_scale = torch.tensor(
            [w2_input_scale_max], dtype=torch.float32, device=device
        )
        layer.w2_input_scale = Parameter(new_w2_input_scale, requires_grad=False)
```
**EN:** This block defines `W4AFp8MoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `W4AFp8MoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 286-289: W4AFp8MoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `W4AFp8MoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `W4AFp8MoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 291-327: W4AFp8MoEMethod.apply()
```python
    def apply(
        self,
        layer: Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        from sglang.srt.layers.moe.cutlass_w4a8_moe import cutlass_w4a8_moe
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output
        topk_weights, topk_ids, _ = topk_output

        output = cutlass_w4a8_moe(
            x,
            layer.w13_weight,
            layer.w2_weight,
            layer.w13_weight_scale_inv,
            layer.w2_weight_scale_inv,
            topk_weights,
            topk_ids,
            self.a_strides1,
            self.b_strides1,
            self.c_strides1,
            self.a_strides2,
            self.b_strides2,
            self.c_strides2,
            self.s_strides13,
            self.s_strides2,
            self.expert_offsets,
            self.problem_sizes1,
            self.problem_sizes2,
            layer.w13_input_scale,
            layer.w2_input_scale,
            routed_scaling_factor=self.moe_runner_config.routed_scaling_factor or 1.0,
        )
        return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `W4AFp8MoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `W4AFp8MoEMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 329-363: W4AFp8MoEMethod.apply_deepep_ll()
```python
    def apply_deepep_ll(
        self,
        layer: DeepEPMoE,
        dispatch_output: DeepEPLLDispatchOutput,
    ) -> torch.Tensor:

        from sglang.srt.layers.moe.cutlass_w4a8_moe import cutlass_w4a8_moe_deepep_ll

        hidden_states, hidden_scales, topk_ids, _, masked_m, _ = dispatch_output

        output = cutlass_w4a8_moe_deepep_ll(
            hidden_states,
            hidden_scales,
            layer.w13_weight,
            layer.w2_weight,
            layer.w13_weight_scale_inv,
            layer.w2_weight_scale_inv,
            topk_ids,
            masked_m,
            layer.quant_method.a_strides1,
            layer.quant_method.b_strides1,
            layer.quant_method.c_strides1,
            layer.quant_method.a_strides2,
            layer.quant_method.b_strides2,
            layer.quant_method.c_strides2,
            layer.quant_method.s_strides13,
            layer.quant_method.s_strides2,
            layer.quant_method.expert_offsets,
            layer.quant_method.problem_sizes1,
            layer.quant_method.problem_sizes2,
            layer.w13_input_scale,
            layer.w2_input_scale,
        )

        return output
```
**EN:** This block defines `W4AFp8MoEMethod.apply_deepep_ll()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `W4AFp8MoEMethod.apply_deepep_ll()`，用于将量化计算应用到运行时输入上。

### Lines 365-407: W4AFp8MoEMethod.apply_deepep_normal()
```python
    def apply_deepep_normal(
        self,
        layer: DeepEPMoE,
        dispatch_output: DeepEPNormalDispatchOutput,
    ) -> torch.Tensor:
        from sglang.srt.layers.moe.cutlass_w4a8_moe import (
            cutlass_w4a8_moe_deepep_normal,
        )

        hidden_states, topk_idx, topk_weights = (
            dispatch_output.hidden_states,
            dispatch_output.topk_ids,
            dispatch_output.topk_weights,
        )
        if isinstance(hidden_states, tuple):
            hidden_states = hidden_states[0]

        num_tokens = hidden_states.shape[0]
        if num_tokens > 0:
            return cutlass_w4a8_moe_deepep_normal(
                hidden_states,
                layer.w13_weight,
                layer.w2_weight,
                layer.w13_weight_scale_inv,
                layer.w2_weight_scale_inv,
                topk_weights,
                topk_idx,
                self.a_strides1,
                self.b_strides1,
                self.c_strides1,
                self.a_strides2,
                self.b_strides2,
                self.c_strides2,
                self.s_strides13,
                self.s_strides2,
                self.expert_offsets,
                self.problem_sizes1,
                self.problem_sizes2,
                layer.w13_input_scale,
                layer.w2_input_scale,
            )
        else:
            return hidden_states
```
**EN:** This block defines `W4AFp8MoEMethod.apply_deepep_normal()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `W4AFp8MoEMethod.apply_deepep_normal()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `ACTIVATION_SCHEMES`: A module-level constant or registry. / `ACTIVATION_SCHEMES`：模块级常量或注册表。
- `W4AFp8Config`: A configuration class that structures file-level quantization behavior. / `W4AFp8Config` 是一个配置类，用于组织该文件中的量化行为。
- `interleave_scales()` : A public function that implements reusable module logic for the quantization stack. / `interleave_scales()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `W4AFp8MoEMethod`: A runtime method class that structures file-level quantization behavior. / `W4AFp8MoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `torch`, `torch.nn`, `torch.nn.parameter`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.cutlass_w4a8_moe`, `sglang.srt.layers.moe.ep_moe.layer`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.fp8`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
