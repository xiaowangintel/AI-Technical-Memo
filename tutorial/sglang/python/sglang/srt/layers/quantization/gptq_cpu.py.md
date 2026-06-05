# gptq_cpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/gptq_cpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for gptq cpu quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 GPTQ CPU 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: module imports and setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING, List, Optional

import torch

from sglang.srt.layers.moe import (
    MoeRunnerConfig,
)
from sglang.srt.layers.parameter import (
    ChannelQuantScaleParameter,
    GroupQuantScaleParameter,
    PackedColumnParameter,
    PackedvLLMParameter,
    RowvLLMParameter,
)
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
)
```
**EN:** This block imports __future__, torch, typing, .gptq, sglang.srt.layers.amx_utils, sglang.srt.layers.linear, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, typing, .gptq, sglang.srt.layers.amx_utils, sglang.srt.layers.linear, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 22-25: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 27-32: module imports and setup
```python
from sglang.srt.layers.amx_utils import (
    CPUQuantMethod,
    _amx_process_weight_after_loading,
)

from .gptq import GPTQConfig
```
**EN:** This block imports __future__, torch, typing, .gptq, sglang.srt.layers.amx_utils, sglang.srt.layers.linear, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, typing, .gptq, sglang.srt.layers.amx_utils, sglang.srt.layers.linear, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 35-37: class CPUGPTQConfig: definition
```python
class CPUGPTQConfig(GPTQConfig):
    """CPU Config class for AWQ, inherit from AWQConfig"""
```
**EN:** This block declares `CPUGPTQConfig`, a configuration class for the quantization stack. It organizes behaviors such as get_supported_act_dtypes, get_quant_method.
**CN:** 该代码块声明 `CPUGPTQConfig`，它是量化栈中的配置类，组织了 get_supported_act_dtypes, get_quant_method 等行为。

### Lines 38-40: CPUGPTQConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.half, torch.bfloat16]
```
**EN:** This block defines `CPUGPTQConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CPUGPTQConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 42-53: CPUGPTQConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[LinearMethodBase]:
        # Delay the import to avoid circular dependency
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if isinstance(layer, FusedMoE):
            return GPTQMoEIntelAMXMethod(self)

        if isinstance(layer, LinearBase):
            return GPTQLinearIntelAMXMethod(self)
```
**EN:** This block defines `CPUGPTQConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CPUGPTQConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 56-58: class GPTQLinearIntelAMXMethod: definition
```python
class GPTQLinearIntelAMXMethod(LinearMethodBase):
    """Linear method for GPTQ on Intel CPU with AMX."""
```
**EN:** This block declares `GPTQLinearIntelAMXMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `GPTQLinearIntelAMXMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 59-62: GPTQLinearIntelAMXMethod.__init__()
```python
    def __init__(self, quant_config: GPTQConfig):
        self.quant_config = quant_config
        # GPTQ v1 and v2 format deals with zero points differently
        self.use_v2_format = quant_config.checkpoint_format == "gptq_v2"
```
**EN:** This block defines `GPTQLinearIntelAMXMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GPTQLinearIntelAMXMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 64-108: GPTQLinearIntelAMXMethod.create_weights() (part 1/3)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        del output_size  # Unused.
        weight_loader = extra_weight_attrs.get("weight_loader")
        if input_size_per_partition % self.quant_config.group_size != 0:
            raise ValueError(
                "The input size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size."
            )
        output_size_per_partition = sum(output_partition_sizes)
        if output_size_per_partition % self.quant_config.pack_factor.numerator != 0:
            raise ValueError(
                "The output size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size."
            )

        if self.quant_config.desc_act and not (
            self.quant_config.true_sequential and self.quant_config.static_groups
        ):
            raise ValueError(
                "Currently, desc_act (True) is only supported with sequential and static group on CPU with AMX."
            )
        if self.quant_config.weight_bits != 4:
            raise ValueError("Currently, only 4bits is supported on CPU with AMX.")
        if self.use_v2_format:
            raise ValueError("Currently, gptq_v2 is not supported on CPU with AMX.")

        if self.quant_config.group_size != -1:
            group_size = self.quant_config.group_size
        else:
            group_size = input_size

        scale_and_zero_size = input_size_per_partition // group_size
        scale_and_zero_input_dim = 0
```
**EN:** This segment of `GPTQLinearIntelAMXMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQLinearIntelAMXMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 109-153: GPTQLinearIntelAMXMethod.create_weights() (part 2/3)
```python
        qweight = PackedvLLMParameter(
            data=torch.empty(
                input_size_per_partition // self.quant_config.pack_factor,
                output_size_per_partition,
                dtype=torch.int32,
            ),
            input_dim=0,
            output_dim=1,
            packed_dim=0,
            packed_factor=self.quant_config.pack_factor,
            weight_loader=weight_loader,
        )

        g_idx = RowvLLMParameter(
            data=torch.tensor(
                [
                    i // self.quant_config.group_size
                    for i in range(input_size_per_partition)
                ],
                dtype=torch.int32,
            ),
            input_dim=0,
            weight_loader=weight_loader,
        )
        qzeros_args = {
            "data": torch.empty(
                scale_and_zero_size,
                output_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            "weight_loader": weight_loader,
        }
        weight_scale_args = {
            "data": torch.empty(
                scale_and_zero_size,
                output_size_per_partition,
                dtype=params_dtype,
            ),
            "weight_loader": weight_loader,
        }
        if scale_and_zero_input_dim is None:
            scales = ChannelQuantScaleParameter(output_dim=1, **weight_scale_args)
            qzeros = PackedColumnParameter(
                output_dim=1,
                packed_dim=1,
```
**EN:** This segment of `GPTQLinearIntelAMXMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQLinearIntelAMXMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 154-173: GPTQLinearIntelAMXMethod.create_weights() (part 3/3)
```python
                packed_factor=self.quant_config.pack_factor,
                **qzeros_args,
            )

        else:
            scales = GroupQuantScaleParameter(
                output_dim=1, input_dim=0, **weight_scale_args
            )
            qzeros = PackedvLLMParameter(
                input_dim=0,
                output_dim=1,
                packed_dim=1,
                packed_factor=self.quant_config.pack_factor,
                **qzeros_args,
            )

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("g_idx", g_idx)
        layer.register_parameter("qzeros", qzeros)
        layer.register_parameter("scales", scales)
```
**EN:** This segment of `GPTQLinearIntelAMXMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQLinearIntelAMXMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 175-178: GPTQLinearIntelAMXMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        _amx_process_weight_after_loading(
            layer, ["qweight", "qzeros", "scales"], None, "gptq"
        )
```
**EN:** This block defines `GPTQLinearIntelAMXMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `GPTQLinearIntelAMXMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 180-192: GPTQLinearIntelAMXMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return torch.ops.sgl_kernel.int4_scaled_mm_cpu(
            x,
            layer.qweight,
            layer.qzeros,
            layer.scales,
            bias,
        )
```
**EN:** This block defines `GPTQLinearIntelAMXMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GPTQLinearIntelAMXMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 195-197: class GPTQMoEIntelAMXMethod: definition
```python
class GPTQMoEIntelAMXMethod(FusedMoEMethodBase):
    """MoE method for GPTQ on Intel CPU with AMX."""
```
**EN:** This block declares `GPTQMoEIntelAMXMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, create_moe_runner, process_weights_after_loading.
**CN:** 该代码块声明 `GPTQMoEIntelAMXMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, create_moe_runner, process_weights_after_loading 等行为。

### Lines 198-202: GPTQMoEIntelAMXMethod.__init__()
```python
    def __init__(self, quant_config: GPTQConfig):
        super().__init__()
        self.quant_config = quant_config
        self.use_v2_format = quant_config.checkpoint_format == "gptq_v2"
        self.moe_runner_config: Optional[MoeRunnerConfig] = None
```
**EN:** This block defines `GPTQMoEIntelAMXMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GPTQMoEIntelAMXMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 204-248: GPTQMoEIntelAMXMethod.create_weights() (part 1/3)
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
        if self.quant_config.desc_act and not (
            self.quant_config.true_sequential and self.quant_config.static_groups
        ):
            raise ValueError(
                "Currently, desc_act (True) is only supported with sequential and static group on CPU with AMX."
            )
        if self.quant_config.weight_bits != 4:
            raise ValueError("Currently, only 4bits is supported on CPU with AMX.")
        if self.use_v2_format:
            raise ValueError("Currently, gptq_v2 is not supported on CPU with AMX.")
        # Delay the import to avoid circular dependency
        from sglang.srt.layers.linear import set_weight_attrs
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        if self.quant_config.group_size != -1:
            scales_size13 = hidden_size // self.quant_config.group_size
            w2_scales_size = intermediate_size_per_partition
            scales_size2 = w2_scales_size // self.quant_config.group_size
            strategy = FusedMoeWeightScaleSupported.GROUP.value
        else:
            scales_size13 = 1
            scales_size2 = 1
            strategy = FusedMoeWeightScaleSupported.CHANNEL.value

        extra_weight_attrs.update({"quant_method": strategy, "is_transposed": True})
        # Fused gate_up_proj (column parallel)
        w13_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size // self.quant_config.pack_factor,
                2 * intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qweight", w13_qweight)
```
**EN:** This segment of `GPTQMoEIntelAMXMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMoEIntelAMXMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 249-293: GPTQMoEIntelAMXMethod.create_weights() (part 2/3)
```python
        set_weight_attrs(w13_qweight, extra_weight_attrs)
        # down_proj (row parallel)
        w2_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition // self.quant_config.pack_factor,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qweight", w2_qweight)
        set_weight_attrs(w2_qweight, extra_weight_attrs)
        # up_proj scales
        w13_scales = torch.nn.Parameter(
            torch.empty(
                num_experts,
                scales_size13,
                2 * intermediate_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_scales", w13_scales)
        set_weight_attrs(w13_scales, extra_weight_attrs)
        # down_proj scales
        w2_scales = torch.nn.Parameter(
            torch.empty(num_experts, scales_size2, hidden_size, dtype=params_dtype),
            requires_grad=False,
        )
        layer.register_parameter("w2_scales", w2_scales)
        set_weight_attrs(w2_scales, extra_weight_attrs)
        # dont shard the w2 scales when running act order
        set_weight_attrs(w2_scales, {"load_full_w2": self.quant_config.desc_act})
        # up_proj scales
        w13_qzeros = torch.nn.Parameter(
            torch.empty(
                num_experts,
                scales_size13,
                2 * intermediate_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qzeros", w13_qzeros)
```
**EN:** This segment of `GPTQMoEIntelAMXMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMoEIntelAMXMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 294-328: GPTQMoEIntelAMXMethod.create_weights() (part 3/3)
```python
        set_weight_attrs(w13_qzeros, extra_weight_attrs)
        # down_proj scales
        w2_qzeros = torch.nn.Parameter(
            torch.empty(
                num_experts,
                scales_size2,
                hidden_size // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qzeros", w2_qzeros)
        set_weight_attrs(w2_qzeros, extra_weight_attrs)
        # dont shard the w2 scales when running act order
        set_weight_attrs(w2_qzeros, {"load_full_w2": self.quant_config.desc_act})
        w13_g_idx = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_g_idx", w13_g_idx)
        set_weight_attrs(w13_g_idx, extra_weight_attrs)
        w2_g_idx = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_g_idx", w2_g_idx)
        set_weight_attrs(w2_g_idx, extra_weight_attrs)
```
**EN:** This segment of `GPTQMoEIntelAMXMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMoEIntelAMXMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 330-336: GPTQMoEIntelAMXMethod.create_moe_runner()
```python
    def create_moe_runner(
        self,
        layer: torch.nn.Module,
        moe_runner_config: MoeRunnerConfig,
        **extra_weight_attrs,
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `GPTQMoEIntelAMXMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GPTQMoEIntelAMXMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 338-344: GPTQMoEIntelAMXMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        _amx_process_weight_after_loading(
            layer, ["w13_qweight", "w13_qzeros", "w13_scales"], None, "gptq"
        )
        _amx_process_weight_after_loading(
            layer, ["w2_qweight", "w2_qzeros", "w2_scales"], None, "gptq"
        )
```
**EN:** This block defines `GPTQMoEIntelAMXMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `GPTQMoEIntelAMXMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 346-375: GPTQMoEIntelAMXMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> torch.Tensor:
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        assert (
            self.moe_runner_config.activation == "silu"
        ), "Only SiLU activation is supported."

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output
        topk_weights, topk_ids, _ = topk_output
        output = torch.ops.sgl_kernel.fused_experts_cpu(
            x,
            layer.w13_qweight,
            layer.w2_qweight,
            topk_weights,
            topk_ids,
            False,  # inplace See [Note] inplace should be False in fused_experts.
            CPUQuantMethod.INT4_W4A8,
            layer.w13_scales,  # w1_scale
            layer.w2_scales,  # w2_scale
            layer.w13_qzeros,
            layer.w2_qzeros,
            None,  # block_size
            True,  # is_vnni
        )
        return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `GPTQMoEIntelAMXMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GPTQMoEIntelAMXMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `CPUGPTQConfig`: A configuration class that structures file-level quantization behavior. / `CPUGPTQConfig` 是一个配置类，用于组织该文件中的量化行为。
- `GPTQLinearIntelAMXMethod`: A runtime method class that structures file-level quantization behavior. / `GPTQLinearIntelAMXMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GPTQMoEIntelAMXMethod`: A runtime method class that structures file-level quantization behavior. / `GPTQMoEIntelAMXMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`, `typing`
- **Internal / 内部**: `.gptq`, `sglang.srt.layers.amx_utils`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`
