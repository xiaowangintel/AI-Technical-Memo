# auto_gptq.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/auto_gptq.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AutoGPTQConfig`, `AutoGPTQLinearMethod`, `AutoGPTQMoEMethod` for quantization backends, schemes, and utilities. / 实现 `AutoGPTQConfig`, `AutoGPTQLinearMethod`, `AutoGPTQMoEMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-64)
```python
from copy import deepcopy
from typing import Any

import torch
from safetensors.torch import _TYPES as _SAFETENSORS_TO_TORCH_DTYPE
from transformers import PretrainedConfig

import vllm.model_executor.layers.fused_moe  # noqa
from vllm.logger import init_logger
from vllm.model_executor.kernels.linear import (
    MPLinearLayerConfig,
    choose_mp_linear_kernel,
)
from vllm.model_executor.layers.fused_moe import (
    FusedMoEConfig,
    FusedMoEMethodBase,
    FusedMoEQuantConfig,
    FusedMoeWeightScaleSupported,
    RoutedExperts,
    SharedExperts,
    UnquantizedFusedMoEMethod,
)
from vllm.model_executor.layers.fused_moe.oracle.int_wna16 import (
    convert_to_wna16_moe_kernel_format,
    make_wna16_moe_kernel,
    select_wna16_moe_backend,
)
from vllm.model_executor.layers.linear import LinearMethodBase, set_weight_attrs
from vllm.model_executor.layers.quantization import QuantizationMethods
from vllm.model_executor.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from vllm.model_executor.layers.quantization.utils import replace_parameter
from vllm.model_executor.layers.quantization.utils.gptq_utils import (
    get_dynamic_override,
    get_linear_quant_method,
    override_config,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    check_moe_marlin_supports_layer,
    get_marlin_input_dtype,
    marlin_make_workspace_new,
    marlin_repeat_scales_on_all_ranks,
    verify_marlin_supported,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kInt4StaticGroupScale,
    kInt8StaticGroupScale,
)
from vllm.model_executor.parameter import (
    ChannelQuantScaleParameter,
    GroupQuantScaleParameter,
    PackedColumnParameter,
    PackedvLLMParameter,
    RowvLLMParameter,
)
from vllm.scalar_type import scalar_types
from vllm.transformers_utils.config import get_safetensors_params_metadata
from vllm.utils.collection_utils import is_list_of
```
**EN:** This opening block pulls in external dependencies such as `copy`, `typing`, `torch`, `safetensors`, `transformers` and internal modules such as `vllm.model_executor.layers.fused_moe`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.oracle.int_wna16`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `copy`, `typing`, `torch`, `safetensors`, `transformers`）以及内部模块（如 `vllm.model_executor.layers.fused_moe`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.oracle.int_wna16`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 66-66)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `get_moe_quant_method` (lines 69-92)
```python
def get_moe_quant_method(
    config: "AutoGPTQConfig",
    layer: RoutedExperts,
    prefix: str,
    moe_method_cls: type,
):
    cloned_config = deepcopy(config)

    assert isinstance(layer, RoutedExperts)
    # False = skip module, None = no override, else = Positive match
    if (
        get_dynamic_override(  # noqa: E712
            cloned_config,  # noqa: E712
            layer_name=prefix,
        )
        == False
    ):  # noqa: E712
        return UnquantizedFusedMoEMethod(layer.moe_config)

    if prefix:
        # Dynamic per module/layer rules may override base config
        override_config(cloned_config, prefix=prefix)

    return moe_method_cls(cloned_config, layer.moe_config)
```
**EN:** Defines function `get_moe_quant_method` with signature `get_moe_quant_method(config: 'AutoGPTQConfig', layer: RoutedExperts, prefix: str, moe_method_cls: type)`. It mainly works with `config`, `layer`, `prefix`, `moe_method_cls`; handles quantization-related transformation logic. The body uses branching, validation/error handling. Key calls include `deepcopy`, `isinstance`, `moe_method_cls`, `get_dynamic_override`, `UnquantizedFusedMoEMethod`, `override_config`.
**CN:** 定义函数 `get_moe_quant_method`，其签名为 `get_moe_quant_method(config: 'AutoGPTQConfig', layer: RoutedExperts, prefix: str, moe_method_cls: type)`。它主要围绕 `config`, `layer`, `prefix`, `moe_method_cls` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑。关键调用包括 `deepcopy`, `isinstance`, `moe_method_cls`, `get_dynamic_override`, `UnquantizedFusedMoEMethod`, `override_config`。

### Class `AutoGPTQConfig` overview (lines 95-299)
```python
class AutoGPTQConfig(QuantizationConfig):
    """Config class for AutoGPTQ quantization using Marlin kernels."""

    # (num_bits, is_sym) -> quant_type
    TYPE_MAP = {
        (4, True): scalar_types.uint4b8,
        (8, True): scalar_types.uint8b128,
    }

    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        desc_act: bool,
        is_sym: bool,
        lm_head_quantized: bool,
        dynamic: dict[str, dict[str, int | bool]],
        full_config: dict[str, Any],
        modules_in_block_to_quantize: list[str] | None = None,
    ) -> None:
        super().__init__()
        if desc_act and group_size == -1:
            # In this case, act_order == True is the same as act_order == False
            # (since we have only one group per output channel)
            desc_act = False
```
**EN:** Defines class `AutoGPTQConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 11 direct methods, with notable entries `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`. Its docstring says: Config class for AutoGPTQ quantization using Marlin kernels.
**CN:** 定义类 `AutoGPTQConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 11 个方法，较重要的包括 `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`。 文档字符串进一步说明了该类的定位。

### Method `AutoGPTQConfig.__init__` (lines 104-164)
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        desc_act: bool,
        is_sym: bool,
        lm_head_quantized: bool,
        dynamic: dict[str, dict[str, int | bool]],
        full_config: dict[str, Any],
        modules_in_block_to_quantize: list[str] | None = None,
    ) -> None:
        super().__init__()
        if desc_act and group_size == -1:
            # In this case, act_order == True is the same as act_order == False
            # (since we have only one group per output channel)
            desc_act = False

        # GPTQModel use `dynamic` config property to allow per module
        # quantization config so each module can be individually optimized.
        # Format is dict[str, dict] where key is a regex string that can
        # perform both positive ("+:" prefixed) or negative ("-:" prefixed)
        # matching of a module.
        # Default to positive match, override base quant config mode, if no
        # prefix is used. Value is in dict format of field key and override
        # value.
        # Negative matching will skip quantization init for this module
        # entirely:
        # non-quantized inference. More details and quantization examples can be
        # found at: https://github.com/ModelCloud/GPTQModel
        # Example:
        #  # last 1/2 of the layers 10-21 has 8bit vs 4bit for 0-9
        #  # last 1/4 of the layers 16-21 has 8bit and group_size 64
        # dynamic = {
        #  #`.*\.` matches the layers_node prefix
        #  # positive match layer 10-15
        #  r"+:.*\.(?:1[0-5])\..*": {"bits": 8,},
        #  # positive match layer 16-21
        #  r"+:.*\.(?:1[6-9]|20|21)\..*": {"bits": 8, "group_size": 64,},
# ... truncated for analysis ...
        self.desc_act = desc_act
        self.lm_head_quantized = lm_head_quantized
        self.full_config = full_config

        if (weight_bits, is_sym) not in self.TYPE_MAP:
            raise ValueError(
                f"Unsupported quantization config: bits={weight_bits}, sym={is_sym}"
            )

        self.quant_type = self.TYPE_MAP[(weight_bits, is_sym)]

        self.modules_in_block_to_quantize = modules_in_block_to_quantize or []
        # used to identify GPTQ model quantized by autoround
        self.autoround_version = full_config.get("autoround_version", "")
```
**EN:** Defines function `AutoGPTQConfig.__init__` with signature `__init__(self, weight_bits: int, group_size: int, desc_act: bool, is_sym: bool, lm_head_quantized: bool, dynamic: dict[str, dict[str, int | bool]], full_config: dict[str, Any], modules_in_block_to_quantize: list[str] | None=None) -> None`. It mainly works with `weight_bits`, `group_size`, `desc_act`, `is_sym`, `lm_head_quantized`, `dynamic`, `full_config`, `modules_in_block_to_quantize`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `full_config.get`, `ValueError`, `super`.
**CN:** 定义函数 `AutoGPTQConfig.__init__`，其签名为 `__init__(self, weight_bits: int, group_size: int, desc_act: bool, is_sym: bool, lm_head_quantized: bool, dynamic: dict[str, dict[str, int | bool]], full_config: dict[str, Any], modules_in_block_to_quantize: list[str] | None=None) -> None`。它主要围绕 `weight_bits`, `group_size`, `desc_act`, `is_sym`, `lm_head_quantized`, `dynamic`, `full_config`, `modules_in_block_to_quantize` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `full_config.get`, `ValueError`, `super`。

### Method `AutoGPTQConfig.from_config` (lines 193-214)
```python
    def from_config(cls, config: dict[str, Any]) -> "AutoGPTQConfig":
        dynamic = cls.get_from_keys_or(config, ["dynamic"], default={})
        dynamic = {} if dynamic is None else dynamic

        weight_bits = cls.get_from_keys(config, ["bits"])
        group_size = cls.get_from_keys(config, ["group_size"])
        desc_act = cls.get_from_keys(config, ["desc_act"])
        is_sym = cls.get_from_keys(config, ["sym"])
        lm_head_quantized = cls.get_from_keys_or(config, ["lm_head"], default=False)
        modules_in_block_to_quantize = cls.get_from_keys_or(
            config, ["modules_in_block_to_quantize"], default=None
        )
        return cls(
            weight_bits,
            group_size,
            desc_act,
            is_sym,
            lm_head_quantized,
            dynamic,
            config,
            modules_in_block_to_quantize,
        )
```
**EN:** Defines function `AutoGPTQConfig.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'AutoGPTQConfig'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `cls.get_from_keys_or`, `cls.get_from_keys`, `cls`.
**CN:** 定义函数 `AutoGPTQConfig.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'AutoGPTQConfig'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `cls.get_from_keys_or`, `cls.get_from_keys`, `cls`。

### Method `AutoGPTQConfig.get_quant_method` (lines 238-266)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> "QuantizeMethodBase | None":
        if isinstance(layer, RoutedExperts):
            from vllm.model_executor.layers.quantization.moe_wna16 import MoeWNA16Config

            if not check_moe_marlin_supports_layer(layer, self.group_size):
                logger.warning_once(
                    f"Layer '{prefix}' is not supported by GPTQMoeMarlin. "
                    "Falling back to Moe WNA16 kernels."
                )
                return MoeWNA16Config.from_config(self.full_config).get_quant_method(
                    layer, prefix
                )
            moe_quant_method = get_moe_quant_method(
                self, layer, prefix, AutoGPTQMoEMethod
            )
            if moe_quant_method is None:
                return None
            moe_quant_method.input_dtype = get_marlin_input_dtype(prefix)
            return moe_quant_method

        quant_method = get_linear_quant_method(
            self, layer, prefix, AutoGPTQLinearMethod
        )
        if quant_method is None:
            return None
        quant_method.input_dtype = get_marlin_input_dtype(prefix)
        return quant_method
```
**EN:** Defines function `AutoGPTQConfig.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching. Key calls include `isinstance`, `get_linear_quant_method`, `get_marlin_input_dtype`, `get_moe_quant_method`, `check_moe_marlin_supports_layer`, `logger.warning_once`.
**CN:** 定义函数 `AutoGPTQConfig.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `isinstance`, `get_linear_quant_method`, `get_marlin_input_dtype`, `get_moe_quant_method`, `check_moe_marlin_supports_layer`, `logger.warning_once`。

### Method `AutoGPTQConfig.apply_vllm_mapper` (lines 268-272)
```python
    def apply_vllm_mapper(self, hf_to_vllm_mapper):
        if self.modules_in_block_to_quantize is not None:
            self.modules_in_block_to_quantize = hf_to_vllm_mapper.apply_list(
                self.modules_in_block_to_quantize
            )
```
**EN:** Defines function `AutoGPTQConfig.apply_vllm_mapper` with signature `apply_vllm_mapper(self, hf_to_vllm_mapper)`. It mainly works with `hf_to_vllm_mapper`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `hf_to_vllm_mapper.apply_list`.
**CN:** 定义函数 `AutoGPTQConfig.apply_vllm_mapper`，其签名为 `apply_vllm_mapper(self, hf_to_vllm_mapper)`。它主要围绕 `hf_to_vllm_mapper` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `hf_to_vllm_mapper.apply_list`。

### Class `AutoGPTQLinearMethod` overview (lines 302-460)
```python
class AutoGPTQLinearMethod(LinearMethodBase):
    """Linear method for AutoGPTQ using Marlin kernels.

    Args:
        quant_config: The AutoGPTQ quantization config.
    """

    _kernel_backends_being_used: set[str] = set()

    def __init__(self, quant_config: AutoGPTQConfig) -> None:
        self.quant_config = quant_config
        self.input_dtype = None
        self.quant_type = self.quant_config.quant_type

        # Verify supported on platform.
        verify_marlin_supported(
            quant_type=self.quant_config.quant_type,
            group_size=self.quant_config.group_size,
        )

    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
```
**EN:** Defines class `AutoGPTQLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for AutoGPTQ using Marlin kernels.
**CN:** 定义类 `AutoGPTQLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `AutoGPTQLinearMethod.__init__` (lines 311-320)
```python
    def __init__(self, quant_config: AutoGPTQConfig) -> None:
        self.quant_config = quant_config
        self.input_dtype = None
        self.quant_type = self.quant_config.quant_type

        # Verify supported on platform.
        verify_marlin_supported(
            quant_type=self.quant_config.quant_type,
            group_size=self.quant_config.group_size,
        )
```
**EN:** Defines function `AutoGPTQLinearMethod.__init__` with signature `__init__(self, quant_config: AutoGPTQConfig) -> None`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `verify_marlin_supported`.
**CN:** 定义函数 `AutoGPTQLinearMethod.__init__`，其签名为 `__init__(self, quant_config: AutoGPTQConfig) -> None`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `verify_marlin_supported`。

### Method `AutoGPTQLinearMethod.create_weights` (lines 322-449)
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
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)
        is_row_parallel = input_size != input_size_per_partition
        weight_loader = extra_weight_attrs.get("weight_loader")
        input_dtype = self.input_dtype

        mp_linear_kernel_config = MPLinearLayerConfig(
            full_weight_shape=(input_size, output_size),
            partition_weight_shape=(
                input_size_per_partition,
                output_size_per_partition,
            ),
            weight_type=self.quant_config.quant_type,
            act_type=params_dtype if input_dtype is None else input_dtype,
            group_size=self.quant_config.group_size,
            zero_points=False,
            has_g_idx=self.quant_config.desc_act,
        )

        kernel_type = choose_mp_linear_kernel(mp_linear_kernel_config)

        if kernel_type.__name__ not in self._kernel_backends_being_used:
            logger.info("Using %s for AutoGPTQLinearMethod", kernel_type.__name__)
            self._kernel_backends_being_used.add(kernel_type.__name__)

        # Normalize group_size
        if self.quant_config.group_size != -1:
            group_size = self.quant_config.group_size
        else:
# ... truncated for analysis ...
            )

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("g_idx", g_idx)
        layer.register_parameter("scales", scales)
        layer.register_parameter("qzeros", qzeros)

        self.kernel = kernel_type(
            mp_linear_kernel_config,
            w_q_param_name="qweight",
            w_s_param_name="scales",
            w_zp_param_name="qzeros",
            w_gidx_param_name="g_idx",
        )
```
**EN:** Defines function `AutoGPTQLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs) -> None`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `MPLinearLayerConfig`, `choose_mp_linear_kernel`, `marlin_repeat_scales_on_all_ranks`, `PackedvLLMParameter`.
**CN:** 定义函数 `AutoGPTQLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs) -> None`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `MPLinearLayerConfig`, `choose_mp_linear_kernel`, `marlin_repeat_scales_on_all_ranks`, `PackedvLLMParameter`。

### Method `AutoGPTQLinearMethod.process_weights_after_loading` (lines 451-452)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self.kernel.process_weights_after_loading(layer)
```
**EN:** Defines function `AutoGPTQLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses mostly straightforward data movement and object wiring. Key calls include `self.kernel.process_weights_after_loading`.
**CN:** 定义函数 `AutoGPTQLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.kernel.process_weights_after_loading`。

### Method `AutoGPTQLinearMethod.apply` (lines 454-460)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.kernel.apply_weights(layer, x, bias)
```
**EN:** Defines function `AutoGPTQLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.kernel.apply_weights`.
**CN:** 定义函数 `AutoGPTQLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.kernel.apply_weights`。

### Class `AutoGPTQMoEMethod` overview (lines 463-794)
```python
class AutoGPTQMoEMethod(FusedMoEMethodBase):
    """MoE Marlin method with quantization."""

    def __init__(
        self,
        quant_config: AutoGPTQConfig,
        moe: FusedMoEConfig,
    ) -> None:
        super().__init__(moe)
        self.quant_config = quant_config
        if self.quant_config.quant_type.size_bits == 4:
            quant_type = scalar_types.uint4b8
            scale = kInt4StaticGroupScale
        elif self.quant_config.quant_type.size_bits == 8:
            quant_type = scalar_types.uint8b128
            scale = kInt8StaticGroupScale
        else:
            raise ValueError("AutoGPTQMoEMethod only supports int4 and int8 now.")
        self.input_dtype = None
        self.use_marlin = True
        weight_key = QuantKey(quant_type, scale)

        self.wna16_moe_backend, self.experts_cls = select_wna16_moe_backend(
            moe, weight_key, quant_config.weight_bits
        )
```
**EN:** Defines class `AutoGPTQMoEMethod` with base classes `FusedMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 7 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `_setup_kernel`, `get_fused_moe_quant_config`, `select_gemm_impl`. Its docstring says: MoE Marlin method with quantization.
**CN:** 定义类 `AutoGPTQMoEMethod`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 7 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `_setup_kernel`, `get_fused_moe_quant_config`, `select_gemm_impl`。 文档字符串进一步说明了该类的定位。

### Method `AutoGPTQMoEMethod.__init__` (lines 466-487)
```python
    def __init__(
        self,
        quant_config: AutoGPTQConfig,
        moe: FusedMoEConfig,
    ) -> None:
        super().__init__(moe)
        self.quant_config = quant_config
        if self.quant_config.quant_type.size_bits == 4:
            quant_type = scalar_types.uint4b8
            scale = kInt4StaticGroupScale
        elif self.quant_config.quant_type.size_bits == 8:
            quant_type = scalar_types.uint8b128
            scale = kInt8StaticGroupScale
        else:
            raise ValueError("AutoGPTQMoEMethod only supports int4 and int8 now.")
        self.input_dtype = None
        self.use_marlin = True
        weight_key = QuantKey(quant_type, scale)

        self.wna16_moe_backend, self.experts_cls = select_wna16_moe_backend(
            moe, weight_key, quant_config.weight_bits
        )
```
**EN:** Defines function `AutoGPTQMoEMethod.__init__` with signature `__init__(self, quant_config: AutoGPTQConfig, moe: FusedMoEConfig) -> None`. It mainly works with `quant_config`, `moe`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `QuantKey`, `select_wna16_moe_backend`, `super`, `ValueError`.
**CN:** 定义函数 `AutoGPTQMoEMethod.__init__`，其签名为 `__init__(self, quant_config: AutoGPTQConfig, moe: FusedMoEConfig) -> None`。它主要围绕 `quant_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `QuantKey`, `select_wna16_moe_backend`, `super`, `ValueError`。

### Method `AutoGPTQMoEMethod.create_weights` (lines 489-643)
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
        layer.input_dtype = self.input_dtype
        is_a_8bit = self.input_dtype is not None and self.input_dtype.itemsize == 1

        if is_a_8bit:
            assert self.quant_config.quant_type.size_bits == 8, (
                "W8A8-INT8 is not supported by marlin kernel."
            )

        intermediate_size_full = extra_weight_attrs.pop("intermediate_size_full")

        self.is_k_full = (not self.quant_config.desc_act) or (
            intermediate_size_per_partition == intermediate_size_full
        )

        if self.quant_config.group_size != -1:
            scales_size13 = hidden_size // self.quant_config.group_size
            w2_scales_size = (
                intermediate_size_full
                if self.quant_config.desc_act
                else intermediate_size_per_partition
            )
            scales_size2 = w2_scales_size // self.quant_config.group_size
            strategy = FusedMoeWeightScaleSupported.GROUP.value
        else:
            scales_size13 = 1
            scales_size2 = 1
            strategy = FusedMoeWeightScaleSupported.CHANNEL.value

        layer.num_groups_w13 = scales_size13
# ... truncated for analysis ...
        set_weight_attrs(w13_g_idx_sort_indices, extra_weight_attrs)
        w2_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_g_idx_sort_indices", w2_g_idx_sort_indices)
        set_weight_attrs(w2_g_idx_sort_indices, extra_weight_attrs)

        device = layer.w13_qweight.device
        layer.workspace = marlin_make_workspace_new(device, 4)
```
**EN:** Defines function `AutoGPTQMoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `extra_weight_attrs.pop`, `extra_weight_attrs.update`, `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `marlin_make_workspace_new`.
**CN:** 定义函数 `AutoGPTQMoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `extra_weight_attrs.pop`, `extra_weight_attrs.update`, `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `marlin_make_workspace_new`。

### Method `AutoGPTQMoEMethod.process_weights_after_loading` (lines 645-722)
```python
    def process_weights_after_loading(self, layer: RoutedExperts) -> None:
        is_a_8bit = self.input_dtype is not None and self.input_dtype.itemsize == 1

        if is_a_8bit:
            assert self.quant_config.quant_type.size_bits == 8, (
                "W8A8-INT8 is not supported by marlin kernel."
            )

        (
            w13,
            w2,
            w13_scale,
            w2_scale,
            w13_g_idx,
            w2_g_idx,
            w13_g_idx_sort_indices,
            w2_g_idx_sort_indices,
            w13_input_global_scale,
            w2_input_global_scale,
            w13_bias,
            w2_bias,
        ) = convert_to_wna16_moe_kernel_format(
            backend=self.wna16_moe_backend,
            layer=layer,
            quant_config=self.quant_config,
            input_dtype=self.input_dtype,
            w13=layer.w13_qweight,
            w2=layer.w2_qweight,
            w13_scale=layer.w13_scales,
            w2_scale=layer.w2_scales,
            w13_g_idx=layer.w13_g_idx,
            w2_g_idx=layer.w2_g_idx,
            w13_bias=getattr(layer, "w13_bias", None),
            w2_bias=getattr(layer, "w2_bias", None),
        )

        replace_parameter(layer, "w13_qweight", w13)
        replace_parameter(layer, "w2_qweight", w2)
# ... truncated for analysis ...
                replace_parameter(layer, "w13_bias", w13_bias)
            else:
                layer.register_parameter(
                    "w13_bias", torch.nn.Parameter(w13_bias, requires_grad=False)
                )
        if w2_bias is not None:
            if hasattr(layer, "w2_bias"):
                replace_parameter(layer, "w2_bias", w2_bias)
            else:
                layer.register_parameter(
                    "w2_bias", torch.nn.Parameter(w2_bias, requires_grad=False)
                )

        self._setup_kernel(layer)
```
**EN:** Defines function `AutoGPTQMoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `convert_to_wna16_moe_kernel_format`, `replace_parameter`, `self._setup_kernel`, `hasattr`, `getattr`, `layer.register_parameter`.
**CN:** 定义函数 `AutoGPTQMoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `convert_to_wna16_moe_kernel_format`, `replace_parameter`, `self._setup_kernel`, `hasattr`, `getattr`, `layer.register_parameter`。

### Method `AutoGPTQMoEMethod.apply` (lines 771-794)
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
        assert not self.is_monolithic
        assert self.moe_kernel is not None
        return self.moe_kernel.apply(
            hidden_states=x,
            w1=layer.w13_qweight,
            w2=layer.w2_qweight,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            activation=layer.activation,
            global_num_experts=layer.global_num_experts,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            expert_map=layer.expert_map,
            shared_experts=shared_experts,
            shared_experts_input=shared_experts_input,
        )
```
**EN:** Defines function `AutoGPTQMoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply`.
**CN:** 定义函数 `AutoGPTQMoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `AutoGPTQConfig`, `AutoGPTQLinearMethod`, `AutoGPTQMoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `AutoGPTQConfig`, `AutoGPTQLinearMethod`, `AutoGPTQMoEMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_moe_quant_method` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_moe_quant_method` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `copy`, `typing`, `torch`, `safetensors`, `transformers`
- **Internal / 内部**: `vllm.model_executor.layers.fused_moe`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.oracle.int_wna16`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.gptq_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`, `vllm.scalar_type`, `vllm.transformers_utils.config`, `vllm.utils.collection_utils`
