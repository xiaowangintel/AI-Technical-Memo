# moe_wna16.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/moe_wna16.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `MoeWNA16Config`, `MoeWNA16Method` for quantization backends, schemes, and utilities. / 实现 `MoeWNA16Config`, `MoeWNA16Method`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-35)
```python
from typing import Any

import torch

from vllm.distributed import get_tensor_model_parallel_rank, get_tp_group
from vllm.model_executor.layers.fused_moe import (
    FusedMoEConfig,
    FusedMoEMethodBase,
    FusedMoeWeightScaleSupported,
    RoutedExperts,
    SharedExperts,
)
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEQuantConfig,
    int4_w4a16_moe_quant_config,
    int8_w8a16_moe_quant_config,
)
from vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method import (
    UnquantizedFusedMoEMethod,
)
from vllm.model_executor.layers.linear import LinearBase, UnquantizedLinearMethod
from vllm.model_executor.layers.quantization import QuantizationMethods
from vllm.model_executor.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    check_marlin_supports_layer,
)
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch` and internal modules such as `vllm.distributed`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method`, `vllm.model_executor.layers.linear`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`）以及内部模块（如 `vllm.distributed`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method`, `vllm.model_executor.layers.linear`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `is_layer_skipped_quant` (lines 204-205)
```python
def is_layer_skipped_quant(prefix: str, modules_to_not_convert: list[str]):
    return any(module_name in prefix for module_name in modules_to_not_convert)
```
**EN:** Defines function `is_layer_skipped_quant` with signature `is_layer_skipped_quant(prefix: str, modules_to_not_convert: list[str])`. It mainly works with `prefix`, `modules_to_not_convert`; handles quantization-related transformation logic. The body uses comprehensions. Key calls include `any`.
**CN:** 定义函数 `is_layer_skipped_quant`，其签名为 `is_layer_skipped_quant(prefix: str, modules_to_not_convert: list[str])`。它主要围绕 `prefix`, `modules_to_not_convert` 展开；处理量化相关的变换逻辑。函数体包含推导式。关键调用包括 `any`。

### Class `MoeWNA16Config` overview (lines 38-201)
```python
class MoeWNA16Config(QuantizationConfig):
    """Config class for MOE WNA16 (W8A16/W4A16) quantization."""

    def __init__(
        self,
        linear_quant_method: str,
        weight_bits: int,
        group_size: int,
        has_zp: bool,
        lm_head_quantized: bool,
        modules_to_not_convert: list[str] | None,
        full_config: dict[str, Any],
    ) -> None:
        super().__init__()
        self.weight_bits = weight_bits
        self.group_size = group_size
        self.has_zp = has_zp
        self.bit8_pack_factor = 8 // self.weight_bits
        self.lm_head_quantized = lm_head_quantized
        self.linear_quant_method = linear_quant_method
        self.full_config = full_config
        self.use_marlin = False
        # Avoid circular import
        from vllm.model_executor.layers.quantization.awq import AWQConfig
        from vllm.model_executor.layers.quantization.awq_marlin import AWQMarlinConfig
```
**EN:** Defines class `MoeWNA16Config` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 9 direct methods, with notable entries `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`, `from_config`. Its docstring says: Config class for MOE WNA16 (W8A16/W4A16) quantization.
**CN:** 定义类 `MoeWNA16Config`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 9 个方法，较重要的包括 `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`, `from_config`。 文档字符串进一步说明了该类的定位。

### Method `MoeWNA16Config.__init__` (lines 41-86)
```python
    def __init__(
        self,
        linear_quant_method: str,
        weight_bits: int,
        group_size: int,
        has_zp: bool,
        lm_head_quantized: bool,
        modules_to_not_convert: list[str] | None,
        full_config: dict[str, Any],
    ) -> None:
        super().__init__()
        self.weight_bits = weight_bits
        self.group_size = group_size
        self.has_zp = has_zp
        self.bit8_pack_factor = 8 // self.weight_bits
        self.lm_head_quantized = lm_head_quantized
        self.linear_quant_method = linear_quant_method
        self.full_config = full_config
        self.use_marlin = False
        # Avoid circular import
        from vllm.model_executor.layers.quantization.awq import AWQConfig
        from vllm.model_executor.layers.quantization.awq_marlin import AWQMarlinConfig

        if self.linear_quant_method == "gptq":
            pass
        elif self.linear_quant_method in ("awq", "awq_marlin"):
            capability_tuple = current_platform.get_device_capability()
            device_capability = (
                -1 if capability_tuple is None else capability_tuple.to_int()
            )
            awq_min_capability = AWQConfig.get_min_capability()
            if device_capability < awq_min_capability:
                raise ValueError(
                    "The quantization method moe_wna16 + awq is not supported "
                    "for the current GPU. "
                    f"Minimum capability: {awq_min_capability}. "
                    f"Current capability: {device_capability}."
                )
            self.use_marlin = AWQMarlinConfig.is_awq_marlin_compatible(full_config)
        else:
            raise ValueError("moe_wna16 only support gptq and awq.")

        if modules_to_not_convert is None:
            self.modules_to_not_convert = []
        else:
            self.modules_to_not_convert = modules_to_not_convert
```
**EN:** Defines function `MoeWNA16Config.__init__` with signature `__init__(self, linear_quant_method: str, weight_bits: int, group_size: int, has_zp: bool, lm_head_quantized: bool, modules_to_not_convert: list[str] | None, full_config: dict[str, Any]) -> None`. It mainly works with `linear_quant_method`, `weight_bits`, `group_size`, `has_zp`, `lm_head_quantized`, `modules_to_not_convert`, `full_config`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `super`, `current_platform.get_device_capability`, `AWQConfig.get_min_capability`, `AWQMarlinConfig.is_awq_marlin_compatible`, `ValueError`.
**CN:** 定义函数 `MoeWNA16Config.__init__`，其签名为 `__init__(self, linear_quant_method: str, weight_bits: int, group_size: int, has_zp: bool, lm_head_quantized: bool, modules_to_not_convert: list[str] | None, full_config: dict[str, Any]) -> None`。它主要围绕 `linear_quant_method`, `weight_bits`, `group_size`, `has_zp`, `lm_head_quantized`, `modules_to_not_convert`, `full_config` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `super`, `current_platform.get_device_capability`, `AWQConfig.get_min_capability`, `AWQMarlinConfig.is_awq_marlin_compatible`, `ValueError`。

### Method `MoeWNA16Config.get_name` (lines 89-90)
```python
    def get_name(cls) -> QuantizationMethods:
        return "moe_wna16"
```
**EN:** Defines function `MoeWNA16Config.get_name` with signature `get_name(cls) -> QuantizationMethods`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MoeWNA16Config.get_name`，其签名为 `get_name(cls) -> QuantizationMethods`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MoeWNA16Config.from_config` (lines 105-129)
```python
    def from_config(cls, config: dict[str, Any]) -> "MoeWNA16Config":
        linear_quant_method = cls.get_from_keys(config, ["quant_method"])
        weight_bits = cls.get_from_keys(config, ["bits"])
        group_size = cls.get_from_keys(config, ["group_size"])
        lm_head_quantized = cls.get_from_keys_or(config, ["lm_head"], default=False)
        if linear_quant_method == "gptq":
            has_zp = not cls.get_from_keys(config, ["sym"])
            modules_to_not_convert = []
        elif linear_quant_method in ("awq", "awq_marlin"):
            has_zp = cls.get_from_keys(config, ["zero_point"])
            modules_to_not_convert = cls.get_from_keys_or(
                config, ["modules_to_not_convert"], None
            )
        else:
            raise ValueError("moe_wna16 only support gptq and awq.")

        return cls(
            linear_quant_method,
            weight_bits,
            group_size,
            has_zp,
            lm_head_quantized,
            modules_to_not_convert,
            config,
        )
```
**EN:** Defines function `MoeWNA16Config.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'MoeWNA16Config'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`, `ValueError`.
**CN:** 定义函数 `MoeWNA16Config.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'MoeWNA16Config'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`, `ValueError`。

### Method `MoeWNA16Config.get_quant_method` (lines 165-201)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> "QuantizeMethodBase | None":
        if is_layer_skipped_quant(prefix, self.modules_to_not_convert):
            if isinstance(layer, RoutedExperts):
                return UnquantizedFusedMoEMethod(layer.moe_config)
            return UnquantizedLinearMethod()
        elif isinstance(layer, LinearBase):
            # Avoid circular import
            from vllm.model_executor.layers.quantization.auto_gptq import (
                AutoGPTQConfig,
            )
            from vllm.model_executor.layers.quantization.awq import AWQConfig
            from vllm.model_executor.layers.quantization.awq_marlin import (
                AWQMarlinConfig,
            )

            if self.linear_quant_method == "gptq":
                return AutoGPTQConfig.from_config(self.full_config).get_quant_method(
                    layer, prefix
                )
            elif self.linear_quant_method in ("awq", "awq_marlin"):
                if self.use_marlin and check_marlin_supports_layer(
                    layer, self.group_size
                ):
                    return AWQMarlinConfig.from_config(
                        self.full_config
                    ).get_quant_method(layer, prefix)
                else:
                    return AWQConfig.from_config(self.full_config).get_quant_method(
                        layer, prefix
                    )
            else:
                raise ValueError("moe_wna16 only support gptq and awq.")
        elif isinstance(layer, RoutedExperts):
            return MoeWNA16Method(self, layer.moe_config)
        return None
```
**EN:** Defines function `MoeWNA16Config.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching, validation/error handling. Key calls include `is_layer_skipped_quant`, `isinstance`, `UnquantizedLinearMethod`, `UnquantizedFusedMoEMethod`, `AutoGPTQConfig.from_config.get_quant_method`, `MoeWNA16Method`.
**CN:** 定义函数 `MoeWNA16Config.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑。关键调用包括 `is_layer_skipped_quant`, `isinstance`, `UnquantizedLinearMethod`, `UnquantizedFusedMoEMethod`, `AutoGPTQConfig.from_config.get_quant_method`, `MoeWNA16Method`。

### Class `MoeWNA16Method` overview (lines 208-512)
```python
class MoeWNA16Method(FusedMoEMethodBase):
    """Linear method for MOE WNA16 (W8A16/W4A16) quantization.

    Args:
        quant_config: The MOE WNA16 (W8A16/W4A16) quantization config.
    """

    def __init__(self, quant_config: MoeWNA16Config, moe: "FusedMoEConfig") -> None:
        super().__init__(moe)
        self.quant_config = quant_config

    def create_weights(
        self,
        layer: RoutedExperts,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        layer.quant_config = self.quant_config
        bit8_pack_factor = self.quant_config.bit8_pack_factor
        group_size = self.quant_config.group_size
        group_size_div_factor = 1
```
**EN:** Defines class `MoeWNA16Method` with base classes `FusedMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 5 direct methods, with notable entries `__init__`, `create_weights`, `get_fused_moe_quant_config`, `apply`, `get_weight_loader`. Its docstring says: Linear method for MOE WNA16 (W8A16/W4A16) quantization.
**CN:** 定义类 `MoeWNA16Method`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 5 个方法，较重要的包括 `__init__`, `create_weights`, `get_fused_moe_quant_config`, `apply`, `get_weight_loader`。 文档字符串进一步说明了该类的定位。

### Method `MoeWNA16Method.__init__` (lines 215-217)
```python
    def __init__(self, quant_config: MoeWNA16Config, moe: "FusedMoEConfig") -> None:
        super().__init__(moe)
        self.quant_config = quant_config
```
**EN:** Defines function `MoeWNA16Method.__init__` with signature `__init__(self, quant_config: MoeWNA16Config, moe: 'FusedMoEConfig') -> None`. It mainly works with `quant_config`, `moe`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `MoeWNA16Method.__init__`，其签名为 `__init__(self, quant_config: MoeWNA16Config, moe: 'FusedMoEConfig') -> None`。它主要围绕 `quant_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `MoeWNA16Method.create_weights` (lines 219-337)
```python
    def create_weights(
        self,
        layer: RoutedExperts,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        layer.quant_config = self.quant_config
        bit8_pack_factor = self.quant_config.bit8_pack_factor
        group_size = self.quant_config.group_size
        group_size_div_factor = 1

        # make intermediate_size and hidden_size divisible by group_size
        # we reduce the group size to ensure that
        # and we would repeat the loaded_weight later
        while intermediate_size_per_partition % group_size or hidden_size % group_size:
            group_size = group_size // 2
            group_size_div_factor *= 2
            assert group_size >= 32
        layer.group_size = group_size
        layer.group_size_div_factor = group_size_div_factor

        strategy = FusedMoeWeightScaleSupported.GROUP.value
        extra_weight_attrs.update({"quant_method": strategy, "is_transposed": False})

        assert "weight_loader" in extra_weight_attrs
        weight_loader = extra_weight_attrs["weight_loader"]
        wrapped_weight_loader = MoeWNA16Method.get_weight_loader(layer, weight_loader)
        extra_weight_attrs["weight_loader"] = wrapped_weight_loader

        # Fused gate_up_proj (column parallel)
        w13_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // bit8_pack_factor,
# ... truncated for analysis ...
            set_weight_attrs(w2_qzeros, extra_weight_attrs)

        if self.quant_config.linear_quant_method == "gptq":
            # some param are unused, but we need to init them in order to
            # load weights
            invalid_param_keys = ["w13_g_idx", "w2_g_idx"]
            if not self.quant_config.has_zp:
                invalid_param_keys += ["w13_qzeros", "w2_qzeros"]
            for key in invalid_param_keys:
                param = torch.nn.Parameter(
                    torch.empty((0,), dtype=torch.int32), requires_grad=False
                )
                layer.register_parameter(key, param)
                set_weight_attrs(param, extra_weight_attrs)
```
**EN:** Defines function `MoeWNA16Method.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `extra_weight_attrs.update`, `MoeWNA16Method.get_weight_loader`, `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.empty`.
**CN:** 定义函数 `MoeWNA16Method.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `extra_weight_attrs.update`, `MoeWNA16Method.get_weight_loader`, `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.empty`。

### Method `MoeWNA16Method.apply` (lines 359-385)
```python
    def apply(
        self,
        layer: RoutedExperts,
        x: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        shared_experts: SharedExperts | None,
        shared_experts_input: torch.Tensor | None,
    ) -> torch.Tensor:
        from vllm.model_executor.layers.fused_moe import fused_experts

        assert layer.activation == MoEActivation.SILU, (
            f"Only SiLU activation is supported, not {layer.activation}."
        )

        return fused_experts(
            x,
            layer.w13_qweight,
            layer.w2_qweight,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            inplace=not self.moe.disable_inplace,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            global_num_experts=layer.global_num_experts,
            expert_map=layer.expert_map,
            quant_config=self.moe_quant_config,
        )
```
**EN:** Defines function `MoeWNA16Method.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `fused_experts`.
**CN:** 定义函数 `MoeWNA16Method.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `fused_experts`。

### Method `MoeWNA16Method.get_weight_loader` (lines 388-512)
```python
    def get_weight_loader(layer, weight_loader):
        def convert_awq_tensor(tensor, tensor_type):
            # convert awq qweight/qzeros to a standard format (assume int4)
            # qweight: (k, n // pack_factor_bit32) -> (n, k // pack_factor_bit8)
            # qzeros: (k // group_size, n // pack_factor_bit32) ->
            #         (n // pack_factor_bit8, k // group_size)
            # pack_factor_bit32 = 32 // weight_bits
            # pack_factor_bit8 = 8 // weight_bits

            # 0. suppose origin shape (a, b), dtype int32
            # 1. convert to uint8, shape (a, b) -> (a, 4 * b)
            size0 = tensor.size(0)
            tensor = tensor.view(torch.uint8)

            # 2. unpack to uint4 (only when weight_bits == 4)
            #    shape (a, 4 * b) -> (a, 4 * b, 2)
            shifter = torch.tensor([0, 4], dtype=torch.uint8, device=tensor.device)
            tensor = (tensor[:, :, None] >> shifter) & 0xF

            # 3. change order, see
            # https://github.com/casper-hansen/AutoAWQ/blob/v0.2.8/awq/utils/quant_utils.py
            # shape -> (a, 4 * b * pack_factor_bit8)
            reverse_awq_pack_order = [0, 4, 1, 5, 2, 6, 3, 7]
            tensor = tensor.view(-1, 8)[:, reverse_awq_pack_order]
            tensor = tensor.view(size0, -1)

            # 4. transpose, shape -> (4 * b * pack_factor_bit8, a)
            tensor = tensor.T.contiguous()

            # 5. repack (only when weight_bits == 4)
            # qweight shape -> (4 * b * pack_factor_bit8, a // pack_factor_bit8)
            # qzeros shape -> (4 * b, a)

            if tensor_type == "qweight":
                tensor = tensor[:, 1::2] * 16 + tensor[:, ::2]
            elif tensor_type == "qzeros":
                tensor = tensor[1::2, :] * 16 + tensor[::2, :]
            return tensor
# ... truncated for analysis ...
                )[:, tp_rank]
                return True if return_success else None
            else:
                # Delegate to the original loader, passing return_success
                return weight_loader(
                    param,
                    loaded_weight,
                    weight_name,
                    shard_id,
                    expert_id,
                    return_success=return_success,
                )

        return moe_wna16_weight_loader
```
**EN:** Defines function `MoeWNA16Method.get_weight_loader` with signature `get_weight_loader(layer, weight_loader)`. It mainly works with `layer`, `weight_loader`; returns a derived property or capability check. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `tensor.size`, `tensor.view`, `torch.tensor`, `tensor.T.contiguous`, `get_tensor_model_parallel_rank`, `loaded_weight.to`.
**CN:** 定义函数 `MoeWNA16Method.get_weight_loader`，其签名为 `get_weight_loader(layer, weight_loader)`。它主要围绕 `layer`, `weight_loader` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `tensor.size`, `tensor.view`, `torch.tensor`, `tensor.T.contiguous`, `get_tensor_model_parallel_rank`, `loaded_weight.to`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `MoeWNA16Config`, `MoeWNA16Method`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `MoeWNA16Config`, `MoeWNA16Method`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `is_layer_skipped_quant` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `is_layer_skipped_quant` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`
- **Internal / 内部**: `vllm.distributed`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.utils`, `vllm.platforms`
