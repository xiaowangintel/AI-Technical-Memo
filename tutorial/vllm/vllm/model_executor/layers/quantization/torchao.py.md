# torchao.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/torchao.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `TorchAOConfig`, `TorchAOLinearMethod` for quantization backends, schemes, and utilities. / 实现 `TorchAOConfig`, `TorchAOLinearMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-26)
```python
import importlib
import json
import types
from importlib.util import find_spec
from typing import Any

import regex as re
import torch
import torch.nn.functional as F
from packaging import version
from torch.nn.parameter import Parameter

from vllm.logger import init_logger
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
from vllm.model_executor.utils import set_weight_attrs
```
**EN:** This opening block pulls in external dependencies such as `importlib`, `json`, `types`, `typing`, `regex`, `torch` and internal modules such as `vllm.logger`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `importlib`, `json`, `types`, `typing`, `regex`, `torch`）以及内部模块（如 `vllm.logger`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 28-28)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `_bond_method_to_cls` (lines 31-36)
```python
def _bond_method_to_cls(func, obj):
    if hasattr(func, "__self__") or not callable(func):
        # If the function is already bound to an instance, return it as is
        return func
    else:
        return types.MethodType(func, obj)
```
**EN:** Defines function `_bond_method_to_cls` with signature `_bond_method_to_cls(func, obj)`. It mainly works with `func`, `obj`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `hasattr`, `types.MethodType`, `callable`.
**CN:** 定义函数 `_bond_method_to_cls`，其签名为 `_bond_method_to_cls(func, obj)`。它主要围绕 `func`, `obj` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `hasattr`, `types.MethodType`, `callable`。

### Function `_get_weight_attrs` (lines 39-55)
```python
def _get_weight_attrs(param):
    # record attributes attached to the weight, so we can
    # recover later
    recorded_weight_attr = {}
    for key in param.__dict__:
        if hasattr(param, key):
            attr = getattr(param, key)
            if not callable(attr):
                recorded_weight_attr[key] = attr
            elif hasattr(attr, "__self__") and param is attr.__self__:
                # if attr is a bonded method for an instance, and
                # attr.__self__ points to the instance (param)
                # we'll record the underlying function object
                recorded_weight_attr[key] = attr.__func__
            else:
                recorded_weight_attr[key] = attr
    return recorded_weight_attr
```
**EN:** Defines function `_get_weight_attrs` with signature `_get_weight_attrs(param)`. It mainly works with `param`; implements one step in the quantized-weight execution flow. The body uses branching, iteration. Key calls include `hasattr`, `getattr`, `callable`.
**CN:** 定义函数 `_get_weight_attrs`，其签名为 `_get_weight_attrs(param)`。它主要围绕 `param` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 `hasattr`, `getattr`, `callable`。

### Function `_restore_weight_attrs` (lines 58-61)
```python
def _restore_weight_attrs(param, recorded_weight_attr):
    for attr_name, attr in recorded_weight_attr.items():
        if not hasattr(param, attr_name):
            setattr(param, attr_name, _bond_method_to_cls(attr, param))
```
**EN:** Defines function `_restore_weight_attrs` with signature `_restore_weight_attrs(param, recorded_weight_attr)`. It mainly works with `param`, `recorded_weight_attr`; implements one step in the quantized-weight execution flow. The body uses branching, iteration. Key calls include `recorded_weight_attr.items`, `hasattr`, `setattr`, `_bond_method_to_cls`.
**CN:** 定义函数 `_restore_weight_attrs`，其签名为 `_restore_weight_attrs(param, recorded_weight_attr)`。它主要围绕 `param`, `recorded_weight_attr` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 `recorded_weight_attr.items`, `hasattr`, `setattr`, `_bond_method_to_cls`。

### Function `torchao_version_at_least` (lines 64-73)
```python
def torchao_version_at_least(torchao_version: str) -> bool:
    if find_spec("torchao"):
        try:
            if version.parse(importlib.metadata.version("torchao")) >= version.parse(
                torchao_version
            ):
                return True
        except (ImportError, version.InvalidVersion):
            return False
    return False
```
**EN:** Defines function `torchao_version_at_least` with signature `torchao_version_at_least(torchao_version: str) -> bool`. It mainly works with `torchao_version`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `find_spec`, `version.parse`, `importlib.metadata.version`.
**CN:** 定义函数 `torchao_version_at_least`，其签名为 `torchao_version_at_least(torchao_version: str) -> bool`。它主要围绕 `torchao_version` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `find_spec`, `version.parse`, `importlib.metadata.version`。

### Function `should_skip` (lines 76-91)
```python
def should_skip(prefix: str, skip_modules: list[str]) -> bool:
    """
    Robust skipping logic:
    should_skip("model.model.layers.1.q_proj",
                ["model.model.layers.1.q_proj"])  # True
    should_skip("model.model.layers.10.o_proj", ["o_proj"])  -> True
    should_skip("visual.model.layers.1.q_proj", ["visual"])   -> True
    should_skip("model.model.layers.1.q_proj", ["layers.1"])  -> True
    should_skip("model.model.layers.11.q_proj", ["layers.1"]) -> False
    """
    for s in skip_modules:
        if prefix == s:
            return True
        if f".{s}." in f".{prefix}.":
            return True
    return False
```
**EN:** Defines function `should_skip` with signature `should_skip(prefix: str, skip_modules: list[str]) -> bool`. It mainly works with `prefix`, `skip_modules`; implements one step in the quantized-weight execution flow. The body uses branching, iteration. Key calls include no notable helper calls.
**CN:** 定义函数 `should_skip`，其签名为 `should_skip(prefix: str, skip_modules: list[str]) -> bool`。它主要围绕 `prefix`, `skip_modules` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 无显著辅助调用。

### Function `torchao_quantize_param_data` (lines 258-286)
```python
def torchao_quantize_param_data(
    param: torch.Tensor, torchao_config: Any
) -> torch.nn.Parameter:
    """Quantize a Tensor with torchao quantization specified by torchao_config

    Args:
        param: weight parameter of the linear module
        torchao_config: type of quantization and their arguments we want to
            use to quantize the Tensor
    """
    from torchao.core.config import AOBaseConfig
    from torchao.quantization import quantize_

    assert isinstance(torchao_config, AOBaseConfig), f"{torchao_config}"
    """
    Avoid real weight allocation for faster load, since we will
    end up setting it to param.
    """
    with torch.device("meta"):
        # linear can't be top level module since quantize_ is inplace
        # while some of our configs need to do module swap, and only non-top
        # level modules support module swap
        dummy_linear = torch.nn.Sequential(
            torch.nn.Linear(param.shape[1], param.shape[0], bias=False)
        )

    dummy_linear[0].weight = param
    quantize_(dummy_linear, torchao_config)
    return dummy_linear[0].weight
```
**EN:** Defines function `torchao_quantize_param_data` with signature `torchao_quantize_param_data(param: torch.Tensor, torchao_config: Any) -> torch.nn.Parameter`. It mainly works with `param`, `torchao_config`; handles quantization-related transformation logic. The body uses context-managed execution, validation/error handling, tensor/kernel operations. Key calls include `isinstance`, `quantize_`, `torch.device`, `torch.nn.Sequential`, `torch.nn.Linear`.
**CN:** 定义函数 `torchao_quantize_param_data`，其签名为 `torchao_quantize_param_data(param: torch.Tensor, torchao_config: Any) -> torch.nn.Parameter`。它主要围绕 `param`, `torchao_config` 展开；处理量化相关的变换逻辑。函数体包含上下文管理、校验或报错逻辑、张量或内核操作。关键调用包括 `isinstance`, `quantize_`, `torch.device`, `torch.nn.Sequential`, `torch.nn.Linear`。

### Class `TorchAOConfig` overview (lines 102-255)
```python
class TorchAOConfig(QuantizationConfig):
    """Config class for torchao."""

    def __init__(
        self,
        torchao_config,
        skip_modules: list[str] | None = None,
        is_checkpoint_torchao_serialized: bool = False,
    ) -> None:
        super().__init__()
        self.torchao_config = torchao_config
        self.skip_modules = skip_modules or []
        self.is_checkpoint_torchao_serialized = is_checkpoint_torchao_serialized

    def __repr__(self) -> str:
        return (
            f"TorchAOConfig({self.torchao_config=}, {self.skip_modules=}, "
            f"{self.is_checkpoint_torchao_serialized=})"
        )

    def get_name(self) -> QuantizationMethods:
        return "torchao"

    def get_supported_act_dtypes(self) -> list[torch.dtype]:
        return [torch.float32, torch.float16, torch.bfloat16]
```
**EN:** Defines class `TorchAOConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 11 direct methods, with notable entries `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`. Its docstring says: Config class for torchao.
**CN:** 定义类 `TorchAOConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 11 个方法，较重要的包括 `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`。 文档字符串进一步说明了该类的定位。

### Method `TorchAOConfig.__init__` (lines 105-114)
```python
    def __init__(
        self,
        torchao_config,
        skip_modules: list[str] | None = None,
        is_checkpoint_torchao_serialized: bool = False,
    ) -> None:
        super().__init__()
        self.torchao_config = torchao_config
        self.skip_modules = skip_modules or []
        self.is_checkpoint_torchao_serialized = is_checkpoint_torchao_serialized
```
**EN:** Defines function `TorchAOConfig.__init__` with signature `__init__(self, torchao_config, skip_modules: list[str] | None=None, is_checkpoint_torchao_serialized: bool=False) -> None`. It mainly works with `torchao_config`, `skip_modules`, `is_checkpoint_torchao_serialized`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `TorchAOConfig.__init__`，其签名为 `__init__(self, torchao_config, skip_modules: list[str] | None=None, is_checkpoint_torchao_serialized: bool=False) -> None`。它主要围绕 `torchao_config`, `skip_modules`, `is_checkpoint_torchao_serialized` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `TorchAOConfig.get_config_filenames` (lines 133-137)
```python
    def get_config_filenames() -> list[str]:
        """torchao doesn't require additional config files, we use
        `config.json` from huggingface: `model_config.hf_config`
        """
        return []
```
**EN:** Defines function `TorchAOConfig.get_config_filenames` with signature `get_config_filenames() -> list[str]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `TorchAOConfig.get_config_filenames`，其签名为 `get_config_filenames() -> list[str]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `TorchAOConfig.from_config` (lines 140-179)
```python
    def from_config(cls, config: dict[str, Any]) -> "TorchAOConfig":
        """Create the quant config from an hf model config"""
        try:
            from torchao.core.config import config_from_dict
        except ImportError as err:
            raise ImportError(
                "Please install torchao>=0.10.0 via "
                "`pip install torchao>=0.10.0` to use torchao quantization."
            ) from err

        quant_method = cls.get_from_keys_or(config, ["quant_method"], None)
        is_checkpoint_torchao_serialized = (
            quant_method is not None and "torchao" in quant_method
        )

        hf_config = cls.get_from_keys_or(config, ["quant_type"], None)
        assert hf_config is not None, "quant_type must be specified"
        assert len(hf_config) == 1 and "default" in hf_config, (
            "Expected only one key 'default' in quant_type dictionary"
        )
        quant_type = hf_config["default"]
        ao_config = config_from_dict(quant_type)

        # Adds skipped modules defined in "modules_to_not_convert"
        skip_modules = config.get("modules_to_not_convert", []) or []

        # Adds skipped modules defined in "module_fqn_to_config"
        _data = quant_type.get("_data", {})
        if not isinstance(_data, dict):
            _data = {}

        module_fqn = _data.get("module_fqn_to_config", {})
        if not isinstance(module_fqn, dict):
            module_fqn = {}

        for layer, layer_cfg in module_fqn.items():
            if layer_cfg is None:
                skip_modules.append(layer)

        return cls(ao_config, skip_modules, is_checkpoint_torchao_serialized)
```
**EN:** Defines function `TorchAOConfig.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'TorchAOConfig'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, validation/error handling. Key calls include `cls.get_from_keys_or`, `config_from_dict`, `quant_type.get`, `_data.get`, `module_fqn.items`, `cls`.
**CN:** 定义函数 `TorchAOConfig.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'TorchAOConfig'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `cls.get_from_keys_or`, `config_from_dict`, `quant_type.get`, `_data.get`, `module_fqn.items`, `cls`。

### Method `TorchAOConfig.from_config_file` (lines 182-198)
```python
    def from_config_file(cls, config_file: str) -> "TorchAOConfig":
        """Initialize class from a config file. Example:
        ```
        config = Float8DynamicActivationFloat8WeightConfig(granularity=PerRow())
        fn = "torchao_config.json"

        with open(fn, "w") as f:
            f.write(json.dumps(config_to_dict(config)))
        ```
        """
        with open(config_file) as f:
            f.seek(0)
            f_read = f.read()
            config_dict = json.loads(f_read)

        hf_config = {"quant_type": {"default": config_dict}}
        return cls.from_config(hf_config)
```
**EN:** Defines function `TorchAOConfig.from_config_file` with signature `from_config_file(cls, config_file: str) -> 'TorchAOConfig'`. It mainly works with `config_file`; implements one step in the quantized-weight execution flow. The body uses context-managed execution. Key calls include `cls.from_config`, `open`, `f.seek`, `f.read`, `json.loads`.
**CN:** 定义函数 `TorchAOConfig.from_config_file`，其签名为 `from_config_file(cls, config_file: str) -> 'TorchAOConfig'`。它主要围绕 `config_file` 展开；实现量化权重执行流程中的一个步骤。函数体包含上下文管理。关键调用包括 `cls.from_config`, `open`, `f.seek`, `f.read`, `json.loads`。

### Method `TorchAOConfig.from_config_dict_json` (lines 201-208)
```python
    def from_config_dict_json(cls, config_dict_json: str) -> "TorchAOConfig":
        """Initialize class from a config_dict json string, got from
        torchao_config_object = some AOBaseConfig object
        json.dumps(config_to_dict(torchao_config_object))
        """
        config_dict = json.loads(config_dict_json)
        hf_config = {"quant_type": {"default": config_dict}}
        return cls.from_config(hf_config)
```
**EN:** Defines function `TorchAOConfig.from_config_dict_json` with signature `from_config_dict_json(cls, config_dict_json: str) -> 'TorchAOConfig'`. It mainly works with `config_dict_json`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `json.loads`, `cls.from_config`.
**CN:** 定义函数 `TorchAOConfig.from_config_dict_json`，其签名为 `from_config_dict_json(cls, config_dict_json: str) -> 'TorchAOConfig'`。它主要围绕 `config_dict_json` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `json.loads`, `cls.from_config`。

### Method `TorchAOConfig.get_quant_method` (lines 210-252)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> "QuantizeMethodBase | None":
        if not isinstance(layer, LinearBase):
            return None

        from torchao.quantization import ModuleFqnToConfig

        if should_skip(prefix, self.skip_modules):
            return UnquantizedLinearMethod()

        module_fqn = prefix
        if isinstance(self.torchao_config, ModuleFqnToConfig):
            module_fqn_to_config = self.torchao_config.module_fqn_to_config
            c = None
            if module_fqn in module_fqn_to_config:
                assert not module_fqn.startswith("re:"), (
                    "module fqn should not start with"
                    "`re:`, which is used for specifying regex"
                )
                c = module_fqn_to_config[module_fqn]
            else:
                for maybe_module_fqn_pattern in module_fqn_to_config:
                    if not maybe_module_fqn_pattern.startswith("re:"):
                        continue
                    elif re.fullmatch(maybe_module_fqn_pattern[3:], module_fqn):
                        # we'll apply the config for first fully matched pattern
                        c = module_fqn_to_config[maybe_module_fqn_pattern]
                        break
                else:
                    # fallback to use default if no module specific
                    # config is provided
                    c = module_fqn_to_config.get("_default", None)

            if c is not None:
                current_torchao_config = TorchAOConfig(
                    c, self.skip_modules, self.is_checkpoint_torchao_serialized
                )
                return TorchAOLinearMethod(current_torchao_config)
            else:
                return UnquantizedLinearMethod()

        return TorchAOLinearMethod(self)
```
**EN:** Defines function `TorchAOConfig.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching, iteration, validation/error handling. Key calls include `should_skip`, `isinstance`, `TorchAOLinearMethod`, `UnquantizedLinearMethod`, `TorchAOConfig`, `module_fqn.startswith`.
**CN:** 定义函数 `TorchAOConfig.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `should_skip`, `isinstance`, `TorchAOLinearMethod`, `UnquantizedLinearMethod`, `TorchAOConfig`, `module_fqn.startswith`。

### Class `TorchAOLinearMethod` overview (lines 289-366)
```python
class TorchAOLinearMethod(LinearMethodBase):
    """Linear method for torchao.

    Args:
        quant_config: The torchao quantization config, a string that encodes
            the type of quantization and all relevant arguments.
    """

    def __init__(self, quant_config: TorchAOConfig):
        self.quant_config = quant_config

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
        weight = Parameter(
            torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition,
```
**EN:** Defines class `TorchAOLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `apply`, `process_weights_after_loading`. Its docstring says: Linear method for torchao.
**CN:** 定义类 `TorchAOLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `apply`, `process_weights_after_loading`。 文档字符串进一步说明了该类的定位。

### Method `TorchAOLinearMethod.__init__` (lines 297-298)
```python
    def __init__(self, quant_config: TorchAOConfig):
        self.quant_config = quant_config
```
**EN:** Defines function `TorchAOLinearMethod.__init__` with signature `__init__(self, quant_config: TorchAOConfig)`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `TorchAOLinearMethod.__init__`，其签名为 `__init__(self, quant_config: TorchAOConfig)`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `TorchAOLinearMethod.create_weights` (lines 300-326)
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
        weight = Parameter(
            torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        if self.quant_config.is_checkpoint_torchao_serialized:
            weight = torchao_quantize_param_data(
                weight, self.quant_config.torchao_config
            )

        set_weight_attrs(weight, {"input_dim": 1, "output_dim": 0})

        layer.register_parameter("weight", weight)
        set_weight_attrs(weight, extra_weight_attrs)
```
**EN:** Defines function `TorchAOLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `Parameter`, `set_weight_attrs`, `layer.register_parameter`, `torch.empty`, `torchao_quantize_param_data`, `sum`.
**CN:** 定义函数 `TorchAOLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `Parameter`, `set_weight_attrs`, `layer.register_parameter`, `torch.empty`, `torchao_quantize_param_data`, `sum`。

### Method `TorchAOLinearMethod.apply` (lines 328-334)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return F.linear(x, layer.weight, bias)
```
**EN:** Defines function `TorchAOLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `F.linear`.
**CN:** 定义函数 `TorchAOLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `F.linear`。

### Method `TorchAOLinearMethod.process_weights_after_loading` (lines 336-366)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if self.quant_config.is_checkpoint_torchao_serialized:
            if not hasattr(layer, "weight"):
                return

            # record attributes attached to the weight, so we can
            # recover later
            recorded_weight_attr = _get_weight_attrs(layer.weight)

            layer.weight = Parameter(
                convert_to_packed_tensor_based_on_current_hardware(layer.weight),
                requires_grad=layer.weight.requires_grad,
            )

            _restore_weight_attrs(layer.weight, recorded_weight_attr)
            return

        # online quantize the weight if the checkpoint is not already
        # quantized by torchao
        recorded_weight_attr = _get_weight_attrs(layer.weight)

        weight = torchao_quantize_param_data(
            layer.weight, self.quant_config.torchao_config
        )
        weight = torch.nn.Parameter(
            convert_to_packed_tensor_based_on_current_hardware(weight),
            weight.requires_grad,
        )

        _restore_weight_attrs(weight, recorded_weight_attr)
        layer.register_parameter("weight", weight)
```
**EN:** Defines function `TorchAOLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, tensor/kernel operations. Key calls include `_get_weight_attrs`, `torchao_quantize_param_data`, `torch.nn.Parameter`, `_restore_weight_attrs`, `layer.register_parameter`, `Parameter`.
**CN:** 定义函数 `TorchAOLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、张量或内核操作。关键调用包括 `_get_weight_attrs`, `torchao_quantize_param_data`, `torch.nn.Parameter`, `_restore_weight_attrs`, `layer.register_parameter`, `Parameter`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `TorchAOConfig`, `TorchAOLinearMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `TorchAOConfig`, `TorchAOLinearMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `_bond_method_to_cls`, `_get_weight_attrs`, `_restore_weight_attrs`, `torchao_version_at_least`, `should_skip` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_bond_method_to_cls`, `_get_weight_attrs`, `_restore_weight_attrs`, `torchao_version_at_least`, `should_skip` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `importlib`, `json`, `types`, `typing`, `regex`, `torch`, `packaging`
- **Internal / 内部**: `vllm.logger`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.utils`
