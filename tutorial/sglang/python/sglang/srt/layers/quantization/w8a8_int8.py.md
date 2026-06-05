# w8a8_int8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/w8a8_int8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for w8a8 int8 quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 W8A8 INT8 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: module imports and setup
```python
from __future__ import annotations

import logging
from types import MappingProxyType
from typing import TYPE_CHECKING, Any, Dict, List, Mapping, Optional, cast

import torch
from torch.nn.parameter import Parameter

from sglang.srt.distributed import get_tensor_model_parallel_world_size
from sglang.srt.layers.amx_utils import (
    CPUQuantMethod,
    _amx_process_weight_after_loading,
)
from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
from sglang.srt.layers.parameter import ChannelQuantScaleParameter, ModelWeightParameter
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.compressed_tensors.utils import should_ignore_layer
from sglang.srt.layers.quantization.int8_kernel import per_token_quant_int8
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.utils import (
    cpu_has_amx_support,
    is_cpu,
    is_cuda,
    is_host_cpu_arm64,
    set_weight_attrs,
    use_intel_amx_backend,
)
from sglang.srt.utils.patch_torch import register_fake_if_exists
```
**EN:** This block imports __future__, logging, sgl_kernel, torch, sglang.srt.distributed, sglang.srt.layers.amx_utils, sglang.srt.layers.linear, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, sgl_kernel, torch, sglang.srt.distributed, sglang.srt.layers.amx_utils, sglang.srt.layers.linear, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 37-38: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import StandardDispatchOutput
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 40-40: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 41-41: initialize _is_cpu_amx_available
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cpu_amx_available.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cpu_amx_available。

### Lines 42-42: initialize _is_cpu
```python
_is_cpu = is_cpu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cpu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cpu。

### Lines 43-43: initialize _is_cpu_arm64
```python
_is_cpu_arm64 = is_host_cpu_arm64()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cpu_arm64.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cpu_arm64。

### Lines 45-59: conditional logic for _is_cuda
```python
if _is_cuda:
    from sgl_kernel import int8_scaled_mm

    @register_fake_if_exists("sgl_kernel::int8_scaled_mm")
    def _int8_scaled_mm_abstract(
        mat_a,
        mat_b,
        scales_a,
        scales_b,
        out_dtype,
        bias=None,
    ):
        M = mat_a.shape[-2]
        N = mat_b.shape[-1]
        return mat_a.new_empty((M, N), dtype=out_dtype)
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 62-62: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 65-71: class W8A8Int8Config: definition
```python
class W8A8Int8Config(QuantizationConfig):
    """Config class for W8A8 Quantization.

    - Weight: static, per-channel, symmetric
    - Activation: dynamic, per-token, symmetric
    """
```
**EN:** This block declares `W8A8Int8Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_supported_act_dtypes, get_min_capability, get_name.
**CN:** 该代码块声明 `W8A8Int8Config`，它是量化栈中的配置类，组织了 __init__, get_supported_act_dtypes, get_min_capability, get_name 等行为。

### Lines 72-81: W8A8Int8Config.__init__()
```python
    def __init__(self, quant_config: Dict[str, Any] = {}):
        super().__init__()
        self.quant_description = quant_config
        self.is_dynamic = quant_config.get("is_dynamic", False)
        ignore = cast(List[str], quant_config.get("ignore", []))
        self.ignore = ignore if ignore is not None else []
        packed_modules_mapping = quant_config.get("packed_modules_mapping", {})
        self.packed_modules_mapping = (
            packed_modules_mapping if packed_modules_mapping is not None else {}
        )
```
**EN:** This block defines `W8A8Int8Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `W8A8Int8Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 83-85: W8A8Int8Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.float16, torch.bfloat16]
```
**EN:** This block defines `W8A8Int8Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Int8Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 87-89: W8A8Int8Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 75
```
**EN:** This block defines `W8A8Int8Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Int8Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 91-93: W8A8Int8Config.get_name()
```python
    @classmethod
    def get_name(self) -> str:
        return "w8a8_int8"
```
**EN:** This block defines `W8A8Int8Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Int8Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 95-98: W8A8Int8Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        filenames = []
        return filenames
```
**EN:** This block defines `W8A8Int8Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Int8Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 100-102: W8A8Int8Config.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> W8A8Int8Config:
        return cls(config)
```
**EN:** This block defines `W8A8Int8Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `W8A8Int8Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 104-120: W8A8Int8Config.get_quant_method()
```python
    def get_quant_method(
        self,
        layer: torch.nn.Module,
        prefix: str,
    ) -> Optional[QuantizeMethodBase]:
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if should_ignore_layer(
            prefix, ignore=self.ignore, fused_mapping=self.packed_modules_mapping
        ):
            return UnquantizedLinearMethod()
        if isinstance(layer, LinearBase):
            return W8A8Int8LinearMethod(self)
        elif isinstance(layer, FusedMoE):
            return W8A8Int8MoEMethod(self)
        return None
```
**EN:** This block defines `W8A8Int8Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Int8Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 122-151: W8A8Int8Config.is_layer_skipped()
```python
    def is_layer_skipped(
        self, prefix: str, fused_mapping: Mapping[str, List[str]] = MappingProxyType({})
    ):
        # adapted from vllm.model_executor.layers.quantization.utils.quant_utils.is_layer_skipped
        proj_name = prefix.split(".")[-1]
        if proj_name in fused_mapping:
            shard_prefixes = [
                prefix.replace(proj_name, shard_proj_name)
                for shard_proj_name in fused_mapping[proj_name]
            ]

            is_skipped = None
            for shard_prefix in shard_prefixes:
                is_shard_skipped = (
                    self.quant_description[shard_prefix + ".weight"] == "FLOAT"
                )

                if is_skipped is None:
                    is_skipped = is_shard_skipped
                elif is_shard_skipped != is_skipped:
                    raise ValueError(
                        f"Detected some but not all shards of {prefix} "
                        "are quantized. All shards of fused layers "
                        "to have the same precision."
                    )
        else:
            is_skipped = self.quant_description[prefix + ".weight"] == "FLOAT"

        assert is_skipped is not None
        return is_skipped
```
**EN:** This block defines `W8A8Int8Config.is_layer_skipped()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `W8A8Int8Config.is_layer_skipped()`，用于检查供上层控制流使用的条件。

### Lines 153-154: W8A8Int8Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `W8A8Int8Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Int8Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 157-158: class W8A8Int8LinearMethod: definition
```python
class W8A8Int8LinearMethod(LinearMethodBase):
```
**EN:** This block declares `W8A8Int8LinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, process_weights_after_loading, create_weights, apply.
**CN:** 该代码块声明 `W8A8Int8LinearMethod`，它是量化栈中的运行方法类，组织了 __init__, process_weights_after_loading, create_weights, apply 等行为。

### Lines 159-160: W8A8Int8LinearMethod.__init__()
```python
    def __init__(self, quantization_config: W8A8Int8Config):
        self.quantization_config = quantization_config
```
**EN:** This block defines `W8A8Int8LinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `W8A8Int8LinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 162-172: W8A8Int8LinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if _is_cpu:
            if _is_cpu_amx_available:
                _amx_process_weight_after_loading(layer, ["weight"])
            elif _is_cpu_arm64:
                layer.weight = Parameter(layer.weight.data, requires_grad=False)
            else:
                assert False, "W8A8Int8LinearMethod on CPU only works on AMX or Arm64"
        else:
            layer.weight = Parameter(layer.weight.t(), requires_grad=False)
        layer.weight_scale = Parameter(layer.weight_scale.data, requires_grad=False)
```
**EN:** This block defines `W8A8Int8LinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `W8A8Int8LinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 174-203: W8A8Int8LinearMethod.create_weights()
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

        weight_loader = extra_weight_attrs.get("weight_loader")
        self.logical_widths = output_partition_sizes

        weight = ModelWeightParameter(
            data=torch.empty(
                sum(output_partition_sizes), input_size_per_partition, dtype=torch.int8
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        weight_scale = ChannelQuantScaleParameter(
            data=torch.empty((sum(output_partition_sizes), 1), dtype=torch.float32),
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight_scale", weight_scale)
```
**EN:** This block defines `W8A8Int8LinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `W8A8Int8LinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 205-235: W8A8Int8LinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ):
        if use_intel_amx_backend(layer) or _is_cpu_arm64:
            return torch.ops.sgl_kernel.int8_scaled_mm_with_quant(
                x,
                layer.weight,
                layer.weight_scale,
                bias,
                x.dtype,
                True,  # is_vnni
            )
        x_q, x_scale = per_token_quant_int8(x)

        x_q_2d = x_q.view(-1, x_q.shape[-1])
        x_scale_2d = x_scale.view(-1, x_scale.shape[-1])
        output_shape = [*x_q.shape[:-1], layer.weight.shape[1]]

        output = int8_scaled_mm(
            x_q_2d,
            layer.weight,
            x_scale_2d,
            layer.weight_scale,
            out_dtype=x.dtype,
            bias=bias,
        )

        return output.view(output_shape)
```
**EN:** This block defines `W8A8Int8LinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `W8A8Int8LinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 238-248: class W8A8Int8MoEMethod: definition
```python
class W8A8Int8MoEMethod(FusedMoEMethodBase):
    """MoE method for INT8.
    Supports loading INT8 checkpoints with static weight scale and
    dynamic/static activation scale.
    Also supports loading quantized FP16/BF16 model checkpoints with dynamic
    activation scaling. The weight scaling factor will be initialized after
    the model weights are loaded.
    Args:
        quant_config: The quantization config.
    """
```
**EN:** This block declares `W8A8Int8MoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `W8A8Int8MoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 249-250: W8A8Int8MoEMethod.__init__()
```python
    def __init__(self, quant_config: W8A8Int8Config):
        self.quant_config = quant_config
```
**EN:** This block defines `W8A8Int8MoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `W8A8Int8MoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 252-314: W8A8Int8MoEMethod.create_weights()
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

        tp_size = get_tensor_model_parallel_world_size()

        # WEIGHTS
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
**EN:** This block defines `W8A8Int8MoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `W8A8Int8MoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 316-327: W8A8Int8MoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if _is_cpu_amx_available:
            _amx_process_weight_after_loading(layer, ["w13_weight", "w2_weight"])
        else:
            layer.w13_weight = Parameter(layer.w13_weight, requires_grad=False)
            layer.w2_weight = Parameter(layer.w2_weight, requires_grad=False)
        layer.w13_weight_scale = Parameter(
            layer.w13_weight_scale.data, requires_grad=False
        )
        layer.w2_weight_scale = Parameter(
            layer.w2_weight_scale.data, requires_grad=False
        )
```
**EN:** This block defines `W8A8Int8MoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `W8A8Int8MoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 329-333: W8A8Int8MoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.TRITON, moe_runner_config)
```
**EN:** This block defines `W8A8Int8MoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `W8A8Int8MoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 335-345: W8A8Int8MoEMethod.get_triton_quant_info()
```python
    def get_triton_quant_info(self, layer: torch.nn.Module) -> TritonMoeQuantInfo:
        return TritonMoeQuantInfo(
            w13_weight=layer.w13_weight,
            w2_weight=layer.w2_weight,
            use_int8_w8a8=True,
            per_channel_quant=True,
            w13_scale=layer.w13_weight_scale,
            w2_scale=layer.w2_weight_scale,
            a13_scale=layer.w13_input_scale,
            a2_scale=layer.w2_input_scale,
        )
```
**EN:** This block defines `W8A8Int8MoEMethod.get_triton_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `W8A8Int8MoEMethod.get_triton_quant_info()`，用于为调用方获取或计算派生值。

### Lines 347-383: W8A8Int8MoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> torch.Tensor:
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        if use_intel_amx_backend(layer) or _is_cpu_arm64:
            from sglang.srt.layers.moe.topk import apply_topk_weights_cpu

            topk_weights, topk_ids, _ = topk_output
            topk_ids = topk_ids.int()
            x, topk_weights = apply_topk_weights_cpu(
                self.moe_runner_config.apply_router_weight_on_input, topk_weights, x
            )
            output = torch.ops.sgl_kernel.fused_experts_cpu(
                x,
                layer.w13_weight,
                layer.w2_weight,
                topk_weights,
                topk_ids,
                False,  # inplace See [Note] inplace should be False in fused_experts.
                CPUQuantMethod.INT8_W8A8,
                layer.w13_weight_scale,  # w1_scale
                layer.w2_weight_scale,  # w2_scale
                None,  # w1_zp
                None,  # w2_zp
                None,  # block_size
                True,  # is_vnni
            )
            return StandardCombineInput(hidden_states=output)

        quant_info = self.get_triton_quant_info(layer)
        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `W8A8Int8MoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `W8A8Int8MoEMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `W8A8Int8Config`: A configuration class that structures file-level quantization behavior. / `W8A8Int8Config` 是一个配置类，用于组织该文件中的量化行为。
- `W8A8Int8LinearMethod`: A runtime method class that structures file-level quantization behavior. / `W8A8Int8LinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `W8A8Int8MoEMethod`: A runtime method class that structures file-level quantization behavior. / `W8A8Int8MoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `sgl_kernel`, `torch`, `torch.nn.parameter`, `types`, `typing`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.layers.amx_utils`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.topk`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.compressed_tensors.utils`, `sglang.srt.layers.quantization.int8_kernel`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.utils`, `sglang.srt.utils.patch_torch`
