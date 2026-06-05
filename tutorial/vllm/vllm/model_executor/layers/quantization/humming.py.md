# humming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/humming.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `HummingConfig`, `HummingLayerQuantizationConfig`, `HummingLinearMethod` for quantization backends, schemes, and utilities. / 实现 `HummingConfig`, `HummingLayerQuantizationConfig`, `HummingLinearMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-45)
```python
import json
import math
from collections.abc import Callable
from typing import TYPE_CHECKING, Any

import regex as re
import torch

from vllm import envs
from vllm.model_executor.layers.fused_moe import (
    FusedMoEMethodBase,
    RoutedExperts,
    SharedExperts,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method import (
    UnquantizedFusedMoEMethod,
)
from vllm.model_executor.layers.linear import (
    LinearBase,
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from vllm.model_executor.layers.quantization import QuantizationMethods
from vllm.model_executor.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.model_executor.parameter import (
    BasevLLMParameter,
    BlockQuantScaleParameter,
    ChannelQuantScaleParameter,
    GroupQuantScaleParameter,
    ModelWeightParameter,
    PackedvLLMParameter,
    PerTensorScaleParameter,
    RowvLLMParameter,
)
from vllm.model_executor.utils import set_weight_attrs
```
**EN:** This opening block pulls in external dependencies such as `json`, `math`, `collections`, `typing`, `regex`, `torch` and internal modules such as `vllm`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `json`, `math`, `collections`, `typing`, `regex`, `torch`）以及内部模块（如 `vllm`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `assert_humming_available` (lines 72-76)
```python
def assert_humming_available():
    assert HummingMethod is not None, (
        "humming is not available, please run "
        "'pip install git+https://github.com/inclusionAI/humming' to install it."
    )
```
**EN:** Defines function `assert_humming_available` with signature `assert_humming_available()`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `assert_humming_available`，其签名为 `assert_humming_available()`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Function `prepare_padded_shape` (lines 79-81)
```python
def prepare_padded_shape(shape, x):
    padded_shape = math.ceil(shape / x) * x
    return padded_shape, padded_shape - shape
```
**EN:** Defines function `prepare_padded_shape` with signature `prepare_padded_shape(shape, x)`. It mainly works with `shape`, `x`; constructs runtime helpers from the current configuration. The body uses mostly straightforward data movement and object wiring. Key calls include `math.ceil`.
**CN:** 定义函数 `prepare_padded_shape`，其签名为 `prepare_padded_shape(shape, x)`。它主要围绕 `shape`, `x` 展开；根据当前配置构建运行时辅助对象。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `math.ceil`。

### Function `prepare_param` (lines 84-129)
```python
def prepare_param(tensor, name, extra_attrs):
    extra_attrs = extra_attrs.copy()
    scale_type = extra_attrs.pop("scale_type", None)
    param_cls_name_map = {
        "block": BlockQuantScaleParameter,
        "tensor": PerTensorScaleParameter,
        "group": GroupQuantScaleParameter,
        "channel": ChannelQuantScaleParameter,
        "input_scale": PerTensorScaleParameter,
    }

    param_cls: type[BasevLLMParameter]
    if "packed_dim" in extra_attrs:
        param_cls = PackedvLLMParameter
    elif scale_type in param_cls_name_map:
        param_cls = param_cls_name_map[scale_type]
    elif "output_dim" in extra_attrs and "input_dim" in extra_attrs:
        param_cls = ModelWeightParameter
    elif "input_dim" in extra_attrs:
        param_cls = RowvLLMParameter
    elif "output_dim" in extra_attrs:
        param_cls = ChannelQuantScaleParameter
    else:
        param_cls = BasevLLMParameter

    kwargs_keys = [
        "input_dim",
        "output_dim",
        "packed_dim",
        "packed_factor",
        "weight_loader",
    ]
    cls_kwargs = {}
    for key in extra_attrs.copy():
        if key in kwargs_keys:
            cls_kwargs[key] = extra_attrs.pop(key)

    param = param_cls(data=tensor, **cls_kwargs)
    set_weight_attrs(param, extra_attrs)

    param.param_name = name
    param.ignore_warning = True
    if scale_type in ["tensor", "input_scale"]:
        param.needs_scalar_to_array = True

    return param
```
**EN:** Defines function `prepare_param` with signature `prepare_param(tensor, name, extra_attrs)`. It mainly works with `tensor`, `name`, `extra_attrs`; constructs runtime helpers from the current configuration. The body uses branching, iteration. Key calls include `extra_attrs.copy`, `extra_attrs.pop`, `param_cls`, `set_weight_attrs`.
**CN:** 定义函数 `prepare_param`，其签名为 `prepare_param(tensor, name, extra_attrs)`。它主要围绕 `tensor`, `name`, `extra_attrs` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、循环处理。关键调用包括 `extra_attrs.copy`, `extra_attrs.pop`, `param_cls`, `set_weight_attrs`。

### Function `prepare_moe_param` (lines 132-144)
```python
def prepare_moe_param(tensor, name, extra_attrs):
    param = torch.nn.Parameter(tensor, requires_grad=False)
    if "scale_type" in extra_attrs:
        extra_attrs["quant_method"] = extra_attrs["scale_type"]

    if "input_dim" in extra_attrs and "output_dim" in extra_attrs:
        input_dim = extra_attrs["input_dim"]
        output_dim = extra_attrs["output_dim"]
        extra_attrs["is_transposed"] = input_dim < output_dim

    set_weight_attrs(param, extra_attrs)
    param.param_name = name
    return param
```
**EN:** Defines function `prepare_moe_param` with signature `prepare_moe_param(tensor, name, extra_attrs)`. It mainly works with `tensor`, `name`, `extra_attrs`; constructs runtime helpers from the current configuration. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `set_weight_attrs`.
**CN:** 定义函数 `prepare_moe_param`，其签名为 `prepare_moe_param(tensor, name, extra_attrs)`。它主要围绕 `tensor`, `name`, `extra_attrs` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `set_weight_attrs`。

### Function `may_pad_loaded_weight` (lines 147-160)
```python
def may_pad_loaded_weight(param, loaded_weight):
    pad_shape = getattr(param, "pad_shape", None)
    if pad_shape is None:
        return loaded_weight
    value = 1 if loaded_weight.dtype == torch.float8_e8m0fnu else 0
    padding = []
    for x in pad_shape[::-1][: loaded_weight.ndim]:
        padding += [0, x]
    loaded_weight = torch.nn.functional.pad(
        input=loaded_weight,
        pad=padding,
        value=value,
    )
    return loaded_weight
```
**EN:** Defines function `may_pad_loaded_weight` with signature `may_pad_loaded_weight(param, loaded_weight)`. It mainly works with `param`, `loaded_weight`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, tensor/kernel operations. Key calls include `getattr`, `torch.nn.functional.pad`.
**CN:** 定义函数 `may_pad_loaded_weight`，其签名为 `may_pad_loaded_weight(param, loaded_weight)`。它主要围绕 `param`, `loaded_weight` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `getattr`, `torch.nn.functional.pad`。

### Function `compressed_tensors_get_config` (lines 163-182)
```python
def compressed_tensors_get_config(config: dict[str, Any], key: str):
    assert key in ["weights", "input_activations"]
    target_group_config = None
    for group_config in config["config_groups"].values():
        if "Linear" in group_config["targets"]:
            if "weights" not in group_config:
                return None
            if key not in group_config or group_config[key] is None:
                return None
            target_group_config = group_config[key].copy()
            break

    if target_group_config is None:
        return None
    target_group_config["quant_method"] = config["quant_method"]
    if config["quant_method"] == "compressed-tensors":
        target_group_config["format"] = config["format"]
    elif config["quant_method"] == "modelopt":
        target_group_config["quant_algo"] = config["quant_algo"]
    return target_group_config
```
**EN:** Defines function `compressed_tensors_get_config` with signature `compressed_tensors_get_config(config: dict[str, Any], key: str)`. It mainly works with `config`, `key`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, validation/error handling. Key calls include `config.values`, `group_config.copy`.
**CN:** 定义函数 `compressed_tensors_get_config`，其签名为 `compressed_tensors_get_config(config: dict[str, Any], key: str)`。它主要围绕 `config`, `key` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `config.values`, `group_config.copy`。

### Class `HummingConfig` overview (lines 185-353)
```python
class HummingConfig(QuantizationConfig):
    packed_modules_mapping: dict[str, list[str]] = {}

    def __init__(self, full_config: dict[str, Any] | None = None):
        assert_humming_available()
        self.full_config: dict[str, Any] = full_config or {}

    @classmethod
    def get_name(cls) -> QuantizationMethods:
        return "humming"

    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.bfloat16, torch.half]

    @classmethod
    def get_min_capability(cls) -> int:
        return 75

    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return []

    @classmethod
    def from_config(cls, config: dict[str, Any]) -> "HummingConfig":
```
**EN:** Defines class `HummingConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 13 direct methods, with notable entries `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`, `from_config`.
**CN:** 定义类 `HummingConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 13 个方法，较重要的包括 `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`, `from_config`。

### Method `HummingConfig.__init__` (lines 188-190)
```python
    def __init__(self, full_config: dict[str, Any] | None = None):
        assert_humming_available()
        self.full_config: dict[str, Any] = full_config or {}
```
**EN:** Defines function `HummingConfig.__init__` with signature `__init__(self, full_config: dict[str, Any] | None=None)`. It mainly works with `full_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `assert_humming_available`.
**CN:** 定义函数 `HummingConfig.__init__`，其签名为 `__init__(self, full_config: dict[str, Any] | None=None)`。它主要围绕 `full_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `assert_humming_available`。

### Method `HummingConfig.from_config` (lines 209-210)
```python
    def from_config(cls, config: dict[str, Any]) -> "HummingConfig":
        return cls(full_config=config)
```
**EN:** Defines function `HummingConfig.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'HummingConfig'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `cls`.
**CN:** 定义函数 `HummingConfig.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'HummingConfig'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls`。

### Method `HummingConfig.apply_vllm_mapper` (lines 227-228)
```python
    def apply_vllm_mapper(self, hf_to_vllm_mapper: "WeightsMapper"):
        self.hf_to_vllm_mapper = hf_to_vllm_mapper
```
**EN:** Defines function `HummingConfig.apply_vllm_mapper` with signature `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`. It mainly works with `hf_to_vllm_mapper`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `HummingConfig.apply_vllm_mapper`，其签名为 `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`。它主要围绕 `hf_to_vllm_mapper` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `HummingConfig.get_quant_config_for_layer` (lines 288-332)
```python
    def get_quant_config_for_layer(
        self, prefix: str, layer_type: str
    ) -> "HummingLayerQuantizationConfig | None":
        weight_schema: BaseWeightSchema | None = None
        force_weight_schema: HummingWeightSchema | None = None

        if self.full_config:
            weight_schema = self.get_layer_weight_schema(self.full_config, prefix)

        is_online_quant = False
        online_quant_config = envs.VLLM_HUMMING_ONLINE_QUANT_CONFIG or {}
        if not self.full_config or online_quant_config.get("force_requant", False):
            online_quant_config["quant_method"] = "humming"
            schema = self.get_layer_weight_schema(online_quant_config, prefix)
            if not self.full_config:
                weight_schema = schema
                is_online_quant = True
            else:
                force_weight_schema = schema

        if weight_schema is not None:
            input_schema = None
            force_input_schema = None

            if self.full_config:
                input_schema = self.get_layer_input_schema(self.full_config, prefix)

            if envs.VLLM_HUMMING_INPUT_QUANT_CONFIG:
                quant_config = envs.VLLM_HUMMING_INPUT_QUANT_CONFIG.copy()
                quant_config["quant_method"] = "humming"
                force_input_schema = self.get_layer_input_schema(quant_config, prefix)
                if input_schema is None:
                    input_schema = force_input_schema

            if force_weight_schema is not None and force_input_schema is None:
                force_input_schema = HummingInputSchema()

            return HummingLayerQuantizationConfig(
                weight_schema=weight_schema,
                input_schema=input_schema,
                force_weight_schema=force_weight_schema,
                force_input_schema=force_input_schema,
                is_online_quant=is_online_quant,
            )
        return None
```
**EN:** Defines function `HummingConfig.get_quant_config_for_layer` with signature `get_quant_config_for_layer(self, prefix: str, layer_type: str) -> 'HummingLayerQuantizationConfig | None'`. It mainly works with `prefix`, `layer_type`; handles quantization-related transformation logic. The body uses branching. Key calls include `self.get_layer_weight_schema`, `online_quant_config.get`, `HummingLayerQuantizationConfig`, `self.get_layer_input_schema`, `envs.VLLM_HUMMING_INPUT_QUANT_CONFIG.copy`, `HummingInputSchema`.
**CN:** 定义函数 `HummingConfig.get_quant_config_for_layer`，其签名为 `get_quant_config_for_layer(self, prefix: str, layer_type: str) -> 'HummingLayerQuantizationConfig | None'`。它主要围绕 `prefix`, `layer_type` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `self.get_layer_weight_schema`, `online_quant_config.get`, `HummingLayerQuantizationConfig`, `self.get_layer_input_schema`, `envs.VLLM_HUMMING_INPUT_QUANT_CONFIG.copy`, `HummingInputSchema`。

### Class `HummingLayerQuantizationConfig` overview (lines 356-381)
```python
class HummingLayerQuantizationConfig(HummingConfig):
    def __init__(
        self,
        weight_schema: "BaseWeightSchema",
        input_schema: "BaseInputSchema | None" = None,
        force_weight_schema: "HummingWeightSchema | None" = None,
        force_input_schema: "HummingInputSchema | None" = None,
        is_online_quant: bool = False,
    ):
        self.weight_schema = weight_schema
        if input_schema is None:
            input_schema = HummingInputSchema()
        self.input_schema = input_schema
        self.force_weight_schema = force_weight_schema
        self.force_input_schema = force_input_schema
        self.is_online_quant = is_online_quant

    @classmethod
    def from_config(cls, config):
        weight_schema = BaseWeightSchema.from_config(config)
        return cls(weight_schema)

    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> QuantizeMethodBase | None:
```
**EN:** Defines class `HummingLayerQuantizationConfig` with base classes `HummingConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 3 direct methods, with notable entries `__init__`, `from_config`, `get_quant_method`.
**CN:** 定义类 `HummingLayerQuantizationConfig`，其基类为 `HummingConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 3 个方法，较重要的包括 `__init__`, `from_config`, `get_quant_method`。

### Method `HummingLayerQuantizationConfig.__init__` (lines 357-371)
```python
    def __init__(
        self,
        weight_schema: "BaseWeightSchema",
        input_schema: "BaseInputSchema | None" = None,
        force_weight_schema: "HummingWeightSchema | None" = None,
        force_input_schema: "HummingInputSchema | None" = None,
        is_online_quant: bool = False,
    ):
        self.weight_schema = weight_schema
        if input_schema is None:
            input_schema = HummingInputSchema()
        self.input_schema = input_schema
        self.force_weight_schema = force_weight_schema
        self.force_input_schema = force_input_schema
        self.is_online_quant = is_online_quant
```
**EN:** Defines function `HummingLayerQuantizationConfig.__init__` with signature `__init__(self, weight_schema: 'BaseWeightSchema', input_schema: 'BaseInputSchema | None'=None, force_weight_schema: 'HummingWeightSchema | None'=None, force_input_schema: 'HummingInputSchema | None'=None, is_online_quant: bool=False)`. It mainly works with `weight_schema`, `input_schema`, `force_weight_schema`, `force_input_schema`, `is_online_quant`; initializes the object state and cached resources. The body uses branching. Key calls include `HummingInputSchema`.
**CN:** 定义函数 `HummingLayerQuantizationConfig.__init__`，其签名为 `__init__(self, weight_schema: 'BaseWeightSchema', input_schema: 'BaseInputSchema | None'=None, force_weight_schema: 'HummingWeightSchema | None'=None, force_input_schema: 'HummingInputSchema | None'=None, is_online_quant: bool=False)`。它主要围绕 `weight_schema`, `input_schema`, `force_weight_schema`, `force_input_schema`, `is_online_quant` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `HummingInputSchema`。

### Method `HummingLayerQuantizationConfig.from_config` (lines 374-376)
```python
    def from_config(cls, config):
        weight_schema = BaseWeightSchema.from_config(config)
        return cls(weight_schema)
```
**EN:** Defines function `HummingLayerQuantizationConfig.from_config` with signature `from_config(cls, config)`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `BaseWeightSchema.from_config`, `cls`.
**CN:** 定义函数 `HummingLayerQuantizationConfig.from_config`，其签名为 `from_config(cls, config)`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `BaseWeightSchema.from_config`, `cls`。

### Method `HummingLayerQuantizationConfig.get_quant_method` (lines 378-381)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> QuantizeMethodBase | None:
        raise NotImplementedError
```
**EN:** Defines function `HummingLayerQuantizationConfig.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> QuantizeMethodBase | None`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `HummingLayerQuantizationConfig.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> QuantizeMethodBase | None`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Class `HummingLinearMethod` overview (lines 384-623)
```python
class HummingLinearMethod(LinearMethodBase):
    def __init__(self, quant_config: HummingLayerQuantizationConfig):
        self.quant_config = quant_config
        self.weight_schema = quant_config.weight_schema
        self.input_schema = quant_config.input_schema
        self.force_weight_schema = quant_config.force_weight_schema
        self.force_input_schema = quant_config.force_input_schema
        self.is_online_quant = self.quant_config.is_online_quant

    def prepare_weight_loader(self, layer: torch.nn.Module, weight_loader: Callable):
        def new_weight_loader(
            param: torch.nn.Parameter,
            loaded_weight: torch.Tensor,
            shard_id: str | int | None = None,
        ):
            name = param.param_name
            float_dtypes = [torch.float16, torch.bfloat16, torch.float32]
            is_unquantized = name == "weight" and loaded_weight.dtype in float_dtypes
            if is_unquantized and self.is_online_quant:
                # online quant (fp16/bf16 -> quant_type)
                assert isinstance(self.weight_schema, HummingWeightSchema)
                f16_dtype = DataType.from_torch_dtype(layer.param_dtype)
                has_global_scale = "TENSOR" in str(self.weight_schema.weight_scale_type)
                tensor_list = quantize_weight(
                    weight=loaded_weight,
```
**EN:** Defines class `HummingLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 5 direct methods, with notable entries `__init__`, `prepare_weight_loader`, `create_weights`, `process_weights_after_loading`, `apply`.
**CN:** 定义类 `HummingLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 5 个方法，较重要的包括 `__init__`, `prepare_weight_loader`, `create_weights`, `process_weights_after_loading`, `apply`。

### Method `HummingLinearMethod.__init__` (lines 385-391)
```python
    def __init__(self, quant_config: HummingLayerQuantizationConfig):
        self.quant_config = quant_config
        self.weight_schema = quant_config.weight_schema
        self.input_schema = quant_config.input_schema
        self.force_weight_schema = quant_config.force_weight_schema
        self.force_input_schema = quant_config.force_input_schema
        self.is_online_quant = self.quant_config.is_online_quant
```
**EN:** Defines function `HummingLinearMethod.__init__` with signature `__init__(self, quant_config: HummingLayerQuantizationConfig)`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `HummingLinearMethod.__init__`，其签名为 `__init__(self, quant_config: HummingLayerQuantizationConfig)`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `HummingLinearMethod.create_weights` (lines 473-531)
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
        layer.is_fallback = False
        layer.param_dtype = params_dtype
        layer.input_size = input_size
        layer.output_size = output_size
        layer.input_size_per_partition = input_size_per_partition
        layer.output_partition_sizes_sum = sum(output_partition_sizes)
        layer.output_partition_sizes = output_partition_sizes
        layer.extra_weight_attrs = extra_weight_attrs.copy()

        weight_loader = extra_weight_attrs.get("weight_loader", default_weight_loader)
        new_weight_loader = self.prepare_weight_loader(layer, weight_loader)
        extra_weight_attrs["weight_loader"] = new_weight_loader

        for key in ["weight_block_size", "block_structure"]:
            block_size = getattr(self.weight_schema, key, None)
            if block_size is not None:
                layer.weight_block_size = block_size

        weight_tensor_attrs = self.weight_schema.get_tensors_attrs(
            shape_n=layer.output_partition_sizes_sum,
            shape_k=layer.input_size_per_partition,
            param_dtype=params_dtype,
            stack_size=len(layer.output_partition_sizes),
        )

        input_tensor_attrs = self.input_schema.get_tensors_attrs(
            shape_k=layer.input_size_per_partition,
            param_dtype=params_dtype,
            stack_size=len(layer.output_partition_sizes),
        )

        tensors_attrs = weight_tensor_attrs | input_tensor_attrs

        for name, attrs in tensors_attrs.items():
            tensor = torch.empty(attrs["shape"], dtype=attrs["dtype"])
            extra_attrs = attrs.get("extra_attrs", {}).copy()
            extra_attrs.update(extra_weight_attrs)
            param = prepare_param(tensor, name, extra_attrs)
            setattr(layer, name, param)

        locks = torch.zeros(1024, dtype=torch.int32)
        layer.register_buffer("locks", locks)

        if self.force_input_schema is not None:
            self.input_schema = self.force_input_schema

        if not hasattr(layer, "weight"):
            param = prepare_param(torch.tensor(0), "weight", extra_weight_attrs)
            layer.weight = param
```
**EN:** Defines function `HummingLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, iteration, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.copy`, `extra_weight_attrs.get`, `self.prepare_weight_loader`, `self.weight_schema.get_tensors_attrs`, `self.input_schema.get_tensors_attrs`.
**CN:** 定义函数 `HummingLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.copy`, `extra_weight_attrs.get`, `self.prepare_weight_loader`, `self.weight_schema.get_tensors_attrs`, `self.input_schema.get_tensors_attrs`。

### Method `HummingLinearMethod.process_weights_after_loading` (lines 533-608)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if layer.is_fallback:
            return None

        # convert from checkpoint format to humming format
        if not isinstance(self.weight_schema, HummingWeightSchema):
            self.weight_schema, tensors = self.weight_schema.convert_humming(
                tensors=layer.state_dict(),
                shape_n_stacks=layer.output_partition_sizes,
                shape_k_stacks=[layer.input_size_per_partition],
                param_dtype=layer.param_dtype,
            )

            self.input_schema, _ = self.input_schema.convert_humming(
                tensors=layer.state_dict(),
                shape_n_stacks=layer.output_partition_sizes,
                shape_k_stacks=[layer.input_size_per_partition],
                param_dtype=layer.param_dtype,
            )

            for name, _ in list(layer.named_parameters()):
                delattr(layer, name)

            for name, tensor in tensors.items():
                param = torch.nn.Parameter(tensor, requires_grad=False)
                setattr(layer, name, param)

            del tensors

        # force requant (origin quant setting -> fp16/bf16 -> new_quant setting)
        assert isinstance(self.weight_schema, HummingWeightSchema)
        force_requant = self.force_weight_schema is not None
        if force_requant and self.weight_schema != self.force_weight_schema:
            tensors = self.weight_schema.requant_tensors(
                tensors=layer.state_dict(),
                target_weight_schema=self.force_weight_schema,
                param_dtype=layer.param_dtype,
            )
# ... truncated for analysis ...
        )

        # preprocess weight for inference
        HummingMethod.transform_humming_layer(layer)

        # compute_config: kernel configs that do not directly affect weights
        # but significantly impact kernel behavior or computation precision.
        # see https://github.com/inclusionAI/humming/blob/main/docs/config.md
        compute_config = {
            "use_batch_invariant": envs.VLLM_BATCH_INVARIANT,
            "use_f16_accum": envs.VLLM_HUMMING_USE_F16_ACCUM,
            "gemm_type": "dense",
        }
        self.compute_config = json.dumps(compute_config)
```
**EN:** Defines function `HummingLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `isinstance`, `HummingMethod.prepare_layer_meta`, `HummingMethod.transform_humming_layer`, `json.dumps`, `self.weight_schema.convert_humming`, `self.input_schema.convert_humming`.
**CN:** 定义函数 `HummingLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `isinstance`, `HummingMethod.prepare_layer_meta`, `HummingMethod.transform_humming_layer`, `json.dumps`, `self.weight_schema.convert_humming`, `self.input_schema.convert_humming`。

### Method `HummingLinearMethod.apply` (lines 610-623)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        flatten_inputs = x.view(-1, x.size(-1))
        output = HummingMethod.forward_layer(
            layer=layer,
            inputs=flatten_inputs,
            compute_config=self.compute_config,
        )
        output = output.view(*x.shape[:-1], output.size(-1))
        return output
```
**EN:** Defines function `HummingLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `x.view`, `HummingMethod.forward_layer`, `output.view`, `x.size`, `output.size`.
**CN:** 定义函数 `HummingLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `x.view`, `HummingMethod.forward_layer`, `output.view`, `x.size`, `output.size`。

### Class `HummingMoEMethod` overview (lines 626-926)
```python
class HummingMoEMethod(FusedMoEMethodBase):
    def __init__(
        self, quant_config: HummingLayerQuantizationConfig, moe: "FusedMoEConfig"
    ) -> None:
        super().__init__(moe)
        self.quant_config = quant_config
        self.moe = moe
        self.weight_schema = quant_config.weight_schema
        self.input_schema = quant_config.input_schema
        self.force_weight_schema = quant_config.force_weight_schema
        self.force_input_schema = quant_config.force_input_schema

    def prepare_weight_loader(self, layer, weight_loader):
        def new_weight_loader(
            param: torch.nn.Parameter,
            loaded_weight: torch.Tensor,
            weight_name: str,
            shard_id: str,
            expert_id: int | None = None,
            return_success: bool = False,
        ):
            name = param.param_name
            float_dtypes = [torch.float16, torch.bfloat16, torch.float32]
            is_unquantized = name == "weight" and loaded_weight.dtype in float_dtypes
            # online quant (fp16/bf16 -> quant_type)
```
**EN:** Defines class `HummingMoEMethod` with base classes `FusedMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 7 direct methods, with notable entries `__init__`, `prepare_weight_loader`, `create_weights`, `get_fused_moe_quant_config`, `process_weights_after_loading`, `select_gemm_impl`.
**CN:** 定义类 `HummingMoEMethod`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 7 个方法，较重要的包括 `__init__`, `prepare_weight_loader`, `create_weights`, `get_fused_moe_quant_config`, `process_weights_after_loading`, `select_gemm_impl`。

### Method `HummingMoEMethod.__init__` (lines 627-636)
```python
    def __init__(
        self, quant_config: HummingLayerQuantizationConfig, moe: "FusedMoEConfig"
    ) -> None:
        super().__init__(moe)
        self.quant_config = quant_config
        self.moe = moe
        self.weight_schema = quant_config.weight_schema
        self.input_schema = quant_config.input_schema
        self.force_weight_schema = quant_config.force_weight_schema
        self.force_input_schema = quant_config.force_input_schema
```
**EN:** Defines function `HummingMoEMethod.__init__` with signature `__init__(self, quant_config: HummingLayerQuantizationConfig, moe: 'FusedMoEConfig') -> None`. It mainly works with `quant_config`, `moe`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `HummingMoEMethod.__init__`，其签名为 `__init__(self, quant_config: HummingLayerQuantizationConfig, moe: 'FusedMoEConfig') -> None`。它主要围绕 `quant_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `HummingMoEMethod.create_weights` (lines 702-759)
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
        layer.num_experts = num_experts
        layer.param_dtype = params_dtype
        layer.intermediate_size = intermediate_size_per_partition
        weight_loader = extra_weight_attrs.get("weight_loader", default_weight_loader)
        weight_loader = self.prepare_weight_loader(layer, weight_loader)
        extra_weight_attrs["weight_loader"] = weight_loader

        # sublayer: a layer contains multiple sets of weights for quantized GEMM
        # (e.g., weight, weight_scale, etc.).
        # The weight names of sublayer start with the prefix "{sublayer_name}_"
        layer.sublayer_configs = {
            "w13": {
                "shape_n": intermediate_size_per_partition * 2,
                "shape_k": hidden_size,
                "tensors_attrs": self.weight_schema.get_padded_tensors_attrs(
                    shape_n=intermediate_size_per_partition * 2,
                    shape_k=hidden_size,
                    num_experts=num_experts,
                    param_dtype=params_dtype,
                    has_bias=self.moe.has_bias,
                ),
            },
            "w2": {
                "shape_n": hidden_size,
                "shape_k": intermediate_size_per_partition,
                "tensors_attrs": self.weight_schema.get_padded_tensors_attrs(
                    shape_n=hidden_size,
                    shape_k=intermediate_size_per_partition,
                    num_experts=num_experts,
                    param_dtype=params_dtype,
                    has_bias=self.moe.has_bias,
                ),
            },
        }

        for sublayer_name, configs in layer.sublayer_configs.items():
            for name, attrs in configs["tensors_attrs"].items():
                tensor = torch.empty(attrs["shape"], dtype=attrs["dtype"])
                param = torch.nn.Parameter(tensor, requires_grad=False)
                extra_attrs = attrs.get("extra_attrs", {}).copy()
                extra_attrs.update(extra_weight_attrs)
                param = prepare_moe_param(tensor, name, extra_attrs)
                setattr(layer, f"{sublayer_name}_{name}", param)

        if self.force_input_schema is not None:
            self.input_schema = self.force_input_schema

        locks = torch.zeros(1024, dtype=torch.int32)
        layer.register_buffer("locks", locks)
```
**EN:** Defines function `HummingMoEMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, iteration, tensor/kernel operations. Key calls include `extra_weight_attrs.get`, `self.prepare_weight_loader`, `layer.sublayer_configs.items`, `torch.zeros`, `layer.register_buffer`, `configs.items`.
**CN:** 定义函数 `HummingMoEMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `extra_weight_attrs.get`, `self.prepare_weight_loader`, `layer.sublayer_configs.items`, `torch.zeros`, `layer.register_buffer`, `configs.items`。

### Method `HummingMoEMethod.process_weights_after_loading` (lines 768-876)
```python
    def process_weights_after_loading(self, layer: RoutedExperts) -> None:
        if getattr(self, "processed", False):
            return
        self.processed = True
        layer.weight_schemas = {}
        layer.input_schemas = {}
        for sublayer_name, configs in layer.sublayer_configs.items():
            input_schema = self.input_schema
            weight_schema = self.weight_schema
            # convert from checkpoint format to humming format
            if not isinstance(weight_schema, HummingWeightSchema):
                tensors: dict[str, torch.Tensor] = dict(
                    (key.removeprefix(sublayer_name + "_"), value)
                    for key, value in layer.state_dict().items()
                    if key.startswith(sublayer_name + "_")
                )

                shape_k_stacks = [configs["shape_k"]]
                shape_n_stacks = [configs["shape_n"]]
                if sublayer_name == "w13":
                    shape_n_stacks = [configs["shape_n"] // 2] * 2

                weight_schema, tensors = weight_schema.convert_humming(
                    tensors=tensors,
                    shape_n_stacks=shape_n_stacks,
                    shape_k_stacks=shape_k_stacks,
                    param_dtype=layer.param_dtype,
                    num_experts=layer.num_experts,
                )

                input_schema, _ = input_schema.convert_humming(
                    tensors=tensors,
                    shape_n_stacks=shape_n_stacks,
                    shape_k_stacks=shape_k_stacks,
                    param_dtype=layer.param_dtype,
                    num_experts=layer.num_experts,
                )

# ... truncated for analysis ...
                sublayer_name=sublayer_name,
            )

            # preprocess weight for inference
            HummingMethod.transform_humming_layer(layer, sublayer_name=sublayer_name)

        # use moe modular
        experts: HummingIndexedExperts | HummingGroupedExperts
        assert self.moe_quant_config is not None
        if get_humming_moe_gemm_type() == "indexed":
            experts = HummingIndexedExperts(layer, self.moe, self.moe_quant_config)
        else:
            experts = HummingGroupedExperts(layer, self.moe, self.moe_quant_config)
        self.experts = experts
```
**EN:** Defines function `HummingMoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `getattr`, `layer.sublayer_configs.items`, `isinstance`, `HummingMethod.prepare_layer_meta`, `HummingMethod.transform_humming_layer`, `get_humming_moe_gemm_type`.
**CN:** 定义函数 `HummingMoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `getattr`, `layer.sublayer_configs.items`, `isinstance`, `HummingMethod.prepare_layer_meta`, `HummingMethod.transform_humming_layer`, `get_humming_moe_gemm_type`。

### Method `HummingMoEMethod.apply` (lines 900-926)
```python
    def apply(
        self,
        layer: RoutedExperts,
        x: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        shared_experts: SharedExperts | None,
        shared_experts_input: torch.Tensor | None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        workspace1, workspace2, output = self.experts.make_workspaces(
            M=topk_ids.size(0),
            topk=topk_ids.size(1),
            activation=layer.activation,
        )

        assert workspace1.data_ptr() == output.data_ptr()

        self.experts.main_apply(
            hidden_states=x,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            workspace1=workspace1,
            workspace2=workspace2,
            expert_tokens_meta=None,
        )

        return output
```
**EN:** Defines function `HummingMoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.experts.make_workspaces`, `self.experts.main_apply`, `workspace1.data_ptr`, `output.data_ptr`, `topk_ids.size`.
**CN:** 定义函数 `HummingMoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.experts.make_workspaces`, `self.experts.main_apply`, `workspace1.data_ptr`, `output.data_ptr`, `topk_ids.size`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `HummingConfig`, `HummingLayerQuantizationConfig`, `HummingLinearMethod`, `HummingMoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `HummingConfig`, `HummingLayerQuantizationConfig`, `HummingLinearMethod`, `HummingMoEMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `assert_humming_available`, `prepare_padded_shape`, `prepare_param`, `prepare_moe_param`, `may_pad_loaded_weight` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `assert_humming_available`, `prepare_padded_shape`, `prepare_param`, `prepare_moe_param`, `may_pad_loaded_weight` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `json`, `math`, `collections`, `typing`, `regex`, `torch`
- **Internal / 内部**: `vllm`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.model_loader.weight_utils`, `vllm.model_executor.parameter`, `vllm.model_executor.utils`
