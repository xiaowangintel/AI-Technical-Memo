# custom_config.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/quantization/fx/custom_config.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers. This specific file centers on `custom_config.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. It also participates in graph/module transformation before or after quantized execution. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。 该文件具体围绕 `custom_config.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 它也参与量化执行前后的图/模块转换。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行
```python
# mypy: allow-untyped-defs
from __future__ import annotations

from dataclasses import dataclass
from typing import Any

from torch.ao.quantization import QConfigMapping
from torch.ao.quantization.backend_config import BackendConfig
from torch.ao.quantization.quant_type import (
    _get_quant_type_to_str,
    _quant_type_from_str,
    QuantType,
)


__all__ = [
    "ConvertCustomConfig",
    "FuseCustomConfig",
    "PrepareCustomConfig",
    "StandaloneModuleConfigEntry",
]
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch.ao.quantization:QConfigMapping, torch.ao.quantization.backend_config:BackendConfig, torch.ao.quantization.quant_type:_get_quant_type_to_str, torch.ao.quantization.quant_type:_quant_type_from_str; standard-library helpers such as __future__:annotations, dataclasses:dataclass, typing:Any. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch.ao.quantization:QConfigMapping, torch.ao.quantization.backend_config:BackendConfig, torch.ao.quantization.quant_type:_get_quant_type_to_str, torch.ao.quantization.quant_type:_quant_type_from_str；标准库辅助模块，如 __future__:annotations, dataclasses:dataclass, typing:Any。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 24-51 / 第 24-51 行
```python
# TODO: replace all usages with these constants
STANDALONE_MODULE_NAME_DICT_KEY = "standalone_module_name"
STANDALONE_MODULE_CLASS_DICT_KEY = "standalone_module_class"
FLOAT_TO_OBSERVED_DICT_KEY = "float_to_observed_custom_module_class"
OBSERVED_TO_QUANTIZED_DICT_KEY = "observed_to_quantized_custom_module_class"
NON_TRACEABLE_MODULE_NAME_DICT_KEY = "non_traceable_module_name"
NON_TRACEABLE_MODULE_CLASS_DICT_KEY = "non_traceable_module_class"
INPUT_QUANTIZED_INDEXES_DICT_KEY = "input_quantized_idxs"
OUTPUT_QUANTIZED_INDEXES_DICT_KEY = "output_quantized_idxs"
PRESERVED_ATTRIBUTES_DICT_KEY = "preserved_attributes"


@dataclass
class StandaloneModuleConfigEntry:
    # qconfig_mapping for the prepare function called in the submodule,
    # None means use qconfig from parent qconfig_mapping
    qconfig_mapping: QConfigMapping | None
    example_inputs: tuple[Any, ...]
    prepare_custom_config: PrepareCustomConfig | None
    backend_config: BackendConfig | None


class PrepareCustomConfig:
    """
    Custom configuration for :func:`~torch.ao.quantization.quantize_fx.prepare_fx` and
    :func:`~torch.ao.quantization.quantize_fx.prepare_qat_fx`.

    Example usage::
```
- **EN**: It introduces or extends class-level abstractions such as `StandaloneModuleConfigEntry`, `PrepareCustomConfig`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Named constants such as `STANDALONE_MODULE_NAME_DICT_KEY`, `STANDALONE_MODULE_CLASS_DICT_KEY`, `FLOAT_TO_OBSERVED_DICT_KEY`, `OBSERVED_TO_QUANTIZED_DICT_KEY` centralize shared configuration or sentinel values. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `StandaloneModuleConfigEntry`, `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 `STANDALONE_MODULE_NAME_DICT_KEY, STANDALONE_MODULE_CLASS_DICT_KEY, FLOAT_TO_OBSERVED_DICT_KEY, OBSERVED_TO_QUANTIZED_DICT_KEY` 等具名常量把共享配置或哨兵值集中定义在一起。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 53-78 / 第 53-78 行
```python
        prepare_custom_config = PrepareCustomConfig() \
            .set_standalone_module_name("module1", qconfig_mapping, example_inputs, \
                child_prepare_custom_config, backend_config) \
            .set_standalone_module_class(MyStandaloneModule, qconfig_mapping, example_inputs, \
                child_prepare_custom_config, backend_config) \
            .set_float_to_observed_mapping(FloatCustomModule, ObservedCustomModule) \
            .set_non_traceable_module_names(["module2", "module3"]) \
            .set_non_traceable_module_classes([NonTraceableModule1, NonTraceableModule2]) \
            .set_input_quantized_indexes([0]) \
            .set_output_quantized_indexes([0]) \
            .set_preserved_attributes(["attr1", "attr2"])
    """

    def __init__(self) -> None:
        self.standalone_module_names: dict[str, StandaloneModuleConfigEntry] = {}
        self.standalone_module_classes: dict[type, StandaloneModuleConfigEntry] = {}
        self.float_to_observed_mapping: dict[QuantType, dict[type, type]] = {}
        self.non_traceable_module_names: list[str] = []
        self.non_traceable_module_classes: list[type] = []
        self.input_quantized_indexes: list[int] = []
        self.output_quantized_indexes: list[int] = []
        self.preserved_attributes: list[str] = []

    def __repr__(self):
        dict_nonempty = {k: v for k, v in self.__dict__.items() if len(v) > 0}
        return f"PrepareCustomConfig({dict_nonempty})"
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 80-109 / 第 80-109 行
```python
    def set_standalone_module_name(
        self,
        module_name: str,
        qconfig_mapping: QConfigMapping | None,
        example_inputs: tuple[Any, ...],
        prepare_custom_config: PrepareCustomConfig | None,
        backend_config: BackendConfig | None,
    ) -> PrepareCustomConfig:
        """
        Set the configuration for running a standalone module identified by ``module_name``.

        If ``qconfig_mapping`` is None, the parent ``qconfig_mapping`` will be used instead.
        If ``prepare_custom_config`` is None, an empty ``PrepareCustomConfig`` will be used.
        If ``backend_config`` is None, the parent ``backend_config`` will be used instead.
        """
        self.standalone_module_names[module_name] = StandaloneModuleConfigEntry(
            qconfig_mapping, example_inputs, prepare_custom_config, backend_config
        )
        return self

    def set_standalone_module_class(
        self,
        module_class: type,
        qconfig_mapping: QConfigMapping | None,
        example_inputs: tuple[Any, ...],
        prepare_custom_config: PrepareCustomConfig | None,
        backend_config: BackendConfig | None,
    ) -> PrepareCustomConfig:
        """
        Set the configuration for running a standalone module identified by ``module_class``.
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 111-139 / 第 111-139 行
```python
        If ``qconfig_mapping`` is None, the parent ``qconfig_mapping`` will be used instead.
        If ``prepare_custom_config`` is None, an empty ``PrepareCustomConfig`` will be used.
        If ``backend_config`` is None, the parent ``backend_config`` will be used instead.
        """
        self.standalone_module_classes[module_class] = StandaloneModuleConfigEntry(
            qconfig_mapping, example_inputs, prepare_custom_config, backend_config
        )
        return self

    def set_float_to_observed_mapping(
        self,
        float_class: type,
        observed_class: type,
        quant_type: QuantType = QuantType.STATIC,
    ) -> PrepareCustomConfig:
        """
        Set the mapping from a custom float module class to a custom observed module class.

        The observed module class must have a ``from_float`` class method that converts the float module class
        to the observed module class. This is currently only supported for static quantization.
        """
        if quant_type != QuantType.STATIC:
            raise ValueError(
                "set_float_to_observed_mapping is currently only supported for static quantization"
            )
        if quant_type not in self.float_to_observed_mapping:
            self.float_to_observed_mapping[quant_type] = {}
        self.float_to_observed_mapping[quant_type][float_class] = observed_class
        return self
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 141-165 / 第 141-165 行
```python
    def set_non_traceable_module_names(
        self, module_names: list[str]
    ) -> PrepareCustomConfig:
        """
        Set the modules that are not symbolically traceable, identified by name.
        """
        self.non_traceable_module_names = module_names
        return self

    def set_non_traceable_module_classes(
        self, module_classes: list[type]
    ) -> PrepareCustomConfig:
        """
        Set the modules that are not symbolically traceable, identified by class.
        """
        self.non_traceable_module_classes = module_classes
        return self

    def set_input_quantized_indexes(self, indexes: list[int]) -> PrepareCustomConfig:
        """
        Set the indexes of the inputs of the graph that should be quantized.
        Inputs are otherwise assumed to be in fp32 by default instead.
        """
        self.input_quantized_indexes = indexes
        return self
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 167-195 / 第 167-195 行
```python
    def set_output_quantized_indexes(self, indexes: list[int]) -> PrepareCustomConfig:
        """
        Set the indexes of the outputs of the graph that should be quantized.
        Outputs are otherwise assumed to be in fp32 by default instead.
        """
        self.output_quantized_indexes = indexes
        return self

    def set_preserved_attributes(self, attributes: list[str]) -> PrepareCustomConfig:
        """
        Set the names of the attributes that will persist in the graph module even if they are not used in
        the model's ``forward`` method.
        """
        self.preserved_attributes = attributes
        return self

    # TODO: remove this
    @classmethod
    def from_dict(
        cls, prepare_custom_config_dict: dict[str, Any]
    ) -> PrepareCustomConfig:
        """
        Create a ``PrepareCustomConfig`` from a dictionary with the following items:

            "standalone_module_name": a list of (module_name, qconfig_mapping, example_inputs,
            child_prepare_custom_config, backend_config) tuples

            "standalone_module_class" a list of (module_class, qconfig_mapping, example_inputs,
            child_prepare_custom_config, backend_config) tuples
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 197-220 / 第 197-220 行
```python
            "float_to_observed_custom_module_class": a nested dictionary mapping from quantization
            mode to an inner mapping from float module classes to observed module classes, e.g.
            {"static": {FloatCustomModule: ObservedCustomModule}}

            "non_traceable_module_name": a list of modules names that are not symbolically traceable
            "non_traceable_module_class": a list of module classes that are not symbolically traceable
            "input_quantized_idxs": a list of indexes of graph inputs that should be quantized
            "output_quantized_idxs": a list of indexes of graph outputs that should be quantized
            "preserved_attributes": a list of attributes that persist even if they are not used in ``forward``

        This function is primarily for backward compatibility and may be removed in the future.
        """

        def _get_qconfig_mapping(obj: Any, dict_key: str) -> QConfigMapping | None:
            """
            Convert the given object into a QConfigMapping if possible, else throw an exception.
            """
            if isinstance(obj, QConfigMapping) or obj is None:
                return obj
            if isinstance(obj, dict):
                return QConfigMapping.from_dict(obj)
            raise ValueError(
                f"Expected QConfigMapping in prepare_custom_config_dict[\"{dict_key}\"], got '{type(obj)}'"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 222-246 / 第 222-246 行
```python
        def _get_prepare_custom_config(
            obj: Any, dict_key: str
        ) -> PrepareCustomConfig | None:
            """
            Convert the given object into a PrepareCustomConfig if possible, else throw an exception.
            """
            if isinstance(obj, PrepareCustomConfig) or obj is None:
                return obj
            if isinstance(obj, dict):
                return PrepareCustomConfig.from_dict(obj)
            raise ValueError(
                f"Expected PrepareCustomConfig in prepare_custom_config_dict[\"{dict_key}\"], got '{type(obj)}'"
            )

        def _get_backend_config(obj: Any, dict_key: str) -> BackendConfig | None:
            """
            Convert the given object into a BackendConfig if possible, else throw an exception.
            """
            if isinstance(obj, BackendConfig) or obj is None:
                return obj
            if isinstance(obj, dict):
                return BackendConfig.from_dict(obj)
            raise ValueError(
                f"Expected BackendConfig in prepare_custom_config_dict[\"{dict_key}\"], got '{type(obj)}'"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 248-279 / 第 248-279 行
```python
        conf = cls()
        for (
            module_name,
            qconfig_dict,
            example_inputs,
            _prepare_custom_config_dict,
            backend_config_dict,
        ) in prepare_custom_config_dict.get(STANDALONE_MODULE_NAME_DICT_KEY, []):
            qconfig_mapping = _get_qconfig_mapping(
                qconfig_dict, STANDALONE_MODULE_NAME_DICT_KEY
            )
            prepare_custom_config = _get_prepare_custom_config(
                _prepare_custom_config_dict, STANDALONE_MODULE_NAME_DICT_KEY
            )
            backend_config = _get_backend_config(
                backend_config_dict, STANDALONE_MODULE_NAME_DICT_KEY
            )
            conf.set_standalone_module_name(
                module_name,
                qconfig_mapping,
                example_inputs,
                prepare_custom_config,
                backend_config,
            )
        for (
            module_class,
            qconfig_dict,
            example_inputs,
            _prepare_custom_config_dict,
            backend_config_dict,
        ) in prepare_custom_config_dict.get(STANDALONE_MODULE_CLASS_DICT_KEY, []):
            qconfig_mapping = _get_qconfig_mapping(
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 280-311 / 第 280-311 行
```python
                qconfig_dict, STANDALONE_MODULE_CLASS_DICT_KEY
            )
            prepare_custom_config = _get_prepare_custom_config(
                _prepare_custom_config_dict, STANDALONE_MODULE_CLASS_DICT_KEY
            )
            backend_config = _get_backend_config(
                backend_config_dict, STANDALONE_MODULE_CLASS_DICT_KEY
            )
            conf.set_standalone_module_class(
                module_class,
                qconfig_mapping,
                example_inputs,
                prepare_custom_config,
                backend_config,
            )
        for quant_type_name, custom_module_mapping in prepare_custom_config_dict.get(
            FLOAT_TO_OBSERVED_DICT_KEY, {}
        ).items():
            quant_type = _quant_type_from_str(quant_type_name)
            for float_class, observed_class in custom_module_mapping.items():
                conf.set_float_to_observed_mapping(
                    float_class, observed_class, quant_type
                )
        conf.set_non_traceable_module_names(
            prepare_custom_config_dict.get(NON_TRACEABLE_MODULE_NAME_DICT_KEY, [])
        )
        conf.set_non_traceable_module_classes(
            prepare_custom_config_dict.get(NON_TRACEABLE_MODULE_CLASS_DICT_KEY, [])
        )
        conf.set_input_quantized_indexes(
            prepare_custom_config_dict.get(INPUT_QUANTIZED_INDEXES_DICT_KEY, [])
        )
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 312-337 / 第 312-337 行
```python
        conf.set_output_quantized_indexes(
            prepare_custom_config_dict.get(OUTPUT_QUANTIZED_INDEXES_DICT_KEY, [])
        )
        conf.set_preserved_attributes(
            prepare_custom_config_dict.get(PRESERVED_ATTRIBUTES_DICT_KEY, [])
        )
        return conf

    def to_dict(self) -> dict[str, Any]:
        """
        Convert this ``PrepareCustomConfig`` to a dictionary with the items described in
        :func:`~torch.ao.quantization.fx.custom_config.PrepareCustomConfig.from_dict`.
        """

        def _make_tuple(key: Any, e: StandaloneModuleConfigEntry):
            qconfig_dict = e.qconfig_mapping.to_dict() if e.qconfig_mapping else None
            prepare_custom_config_dict = (
                e.prepare_custom_config.to_dict() if e.prepare_custom_config else None
            )
            return (
                key,
                qconfig_dict,
                e.example_inputs,
                prepare_custom_config_dict,
                e.backend_config,
            )
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 339-370 / 第 339-370 行
```python
        d: dict[str, Any] = {}
        for module_name, sm_config_entry in self.standalone_module_names.items():
            if STANDALONE_MODULE_NAME_DICT_KEY not in d:
                d[STANDALONE_MODULE_NAME_DICT_KEY] = []
            d[STANDALONE_MODULE_NAME_DICT_KEY].append(
                _make_tuple(module_name, sm_config_entry)
            )
        for module_class, sm_config_entry in self.standalone_module_classes.items():
            if STANDALONE_MODULE_CLASS_DICT_KEY not in d:
                d[STANDALONE_MODULE_CLASS_DICT_KEY] = []
            d[STANDALONE_MODULE_CLASS_DICT_KEY].append(
                _make_tuple(module_class, sm_config_entry)
            )
        for (
            quant_type,
            float_to_observed_mapping,
        ) in self.float_to_observed_mapping.items():
            if FLOAT_TO_OBSERVED_DICT_KEY not in d:
                d[FLOAT_TO_OBSERVED_DICT_KEY] = {}
            d[FLOAT_TO_OBSERVED_DICT_KEY][_get_quant_type_to_str(quant_type)] = (
                float_to_observed_mapping
            )
        if len(self.non_traceable_module_names) > 0:
            d[NON_TRACEABLE_MODULE_NAME_DICT_KEY] = self.non_traceable_module_names
        if len(self.non_traceable_module_classes) > 0:
            d[NON_TRACEABLE_MODULE_CLASS_DICT_KEY] = self.non_traceable_module_classes
        if len(self.input_quantized_indexes) > 0:
            d[INPUT_QUANTIZED_INDEXES_DICT_KEY] = self.input_quantized_indexes
        if len(self.output_quantized_indexes) > 0:
            d[OUTPUT_QUANTIZED_INDEXES_DICT_KEY] = self.output_quantized_indexes
        if len(self.preserved_attributes) > 0:
            d[PRESERVED_ATTRIBUTES_DICT_KEY] = self.preserved_attributes
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 371-400 / 第 371-400 行
```python
        return d


class ConvertCustomConfig:
    """
    Custom configuration for :func:`~torch.ao.quantization.quantize_fx.convert_fx`.

    Example usage::

        convert_custom_config = ConvertCustomConfig() \
            .set_observed_to_quantized_mapping(ObservedCustomModule, QuantizedCustomModule) \
            .set_preserved_attributes(["attr1", "attr2"])
    """

    def __init__(self) -> None:
        self.observed_to_quantized_mapping: dict[QuantType, dict[type, type]] = {}
        self.preserved_attributes: list[str] = []

    def __repr__(self):
        dict_nonempty = {k: v for k, v in self.__dict__.items() if len(v) > 0}
        return f"ConvertCustomConfig({dict_nonempty})"

    def set_observed_to_quantized_mapping(
        self,
        observed_class: type,
        quantized_class: type,
        quant_type: QuantType = QuantType.STATIC,
    ) -> ConvertCustomConfig:
        """
        Set the mapping from a custom observed module class to a custom quantized module class.
```
- **EN**: It introduces or extends class-level abstractions such as `PrepareCustomConfig`, `ConvertCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PrepareCustomConfig`, `ConvertCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 402-424 / 第 402-424 行
```python
        The quantized module class must have a ``from_observed`` class method that converts the observed module class
        to the quantized module class.
        """
        if quant_type not in self.observed_to_quantized_mapping:
            self.observed_to_quantized_mapping[quant_type] = {}
        self.observed_to_quantized_mapping[quant_type][observed_class] = quantized_class
        return self

    def set_preserved_attributes(self, attributes: list[str]) -> ConvertCustomConfig:
        """
        Set the names of the attributes that will persist in the graph module even if they are not used in
        the model's ``forward`` method.
        """
        self.preserved_attributes = attributes
        return self

    # TODO: remove this
    @classmethod
    def from_dict(
        cls, convert_custom_config_dict: dict[str, Any]
    ) -> ConvertCustomConfig:
        """
        Create a ``ConvertCustomConfig`` from a dictionary with the following items:
```
- **EN**: It introduces or extends class-level abstractions such as `ConvertCustomConfig`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvertCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 426-449 / 第 426-449 行
```python
            "observed_to_quantized_custom_module_class": a nested dictionary mapping from quantization
            mode to an inner mapping from observed module classes to quantized module classes, e.g.::
            {
            "static": {FloatCustomModule: ObservedCustomModule},
            "dynamic": {FloatCustomModule: ObservedCustomModule},
            "weight_only": {FloatCustomModule: ObservedCustomModule}
            }
            "preserved_attributes": a list of attributes that persist even if they are not used in ``forward``

        This function is primarily for backward compatibility and may be removed in the future.
        """
        conf = cls()
        for quant_type_name, custom_module_mapping in convert_custom_config_dict.get(
            OBSERVED_TO_QUANTIZED_DICT_KEY, {}
        ).items():
            quant_type = _quant_type_from_str(quant_type_name)
            for observed_class, quantized_class in custom_module_mapping.items():
                conf.set_observed_to_quantized_mapping(
                    observed_class, quantized_class, quant_type
                )
        conf.set_preserved_attributes(
            convert_custom_config_dict.get(PRESERVED_ATTRIBUTES_DICT_KEY, [])
        )
        return conf
```
- **EN**: It introduces or extends class-level abstractions such as `ConvertCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvertCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 451-480 / 第 451-480 行
```python
    def to_dict(self) -> dict[str, Any]:
        """
        Convert this ``ConvertCustomConfig`` to a dictionary with the items described in
        :func:`~torch.ao.quantization.fx.custom_config.ConvertCustomConfig.from_dict`.
        """
        d: dict[str, Any] = {}
        for (
            quant_type,
            observed_to_quantized_mapping,
        ) in self.observed_to_quantized_mapping.items():
            if OBSERVED_TO_QUANTIZED_DICT_KEY not in d:
                d[OBSERVED_TO_QUANTIZED_DICT_KEY] = {}
            d[OBSERVED_TO_QUANTIZED_DICT_KEY][_get_quant_type_to_str(quant_type)] = (
                observed_to_quantized_mapping
            )
        if len(self.preserved_attributes) > 0:
            d[PRESERVED_ATTRIBUTES_DICT_KEY] = self.preserved_attributes
        return d


class FuseCustomConfig:
    """
    Custom configuration for :func:`~torch.ao.quantization.quantize_fx.fuse_fx`.

    Example usage::

        fuse_custom_config = FuseCustomConfig().set_preserved_attributes(
            ["attr1", "attr2"]
        )
    """
```
- **EN**: It introduces or extends class-level abstractions such as `ConvertCustomConfig`, `FuseCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvertCustomConfig`, `FuseCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 482-511 / 第 482-511 行
```python
    def __init__(self) -> None:
        self.preserved_attributes: list[str] = []

    def __repr__(self):
        dict_nonempty = {k: v for k, v in self.__dict__.items() if len(v) > 0}
        return f"FuseCustomConfig({dict_nonempty})"

    def set_preserved_attributes(self, attributes: list[str]) -> FuseCustomConfig:
        """
        Set the names of the attributes that will persist in the graph module even if they are not used in
        the model's ``forward`` method.
        """
        self.preserved_attributes = attributes
        return self

    # TODO: remove this
    @classmethod
    def from_dict(cls, fuse_custom_config_dict: dict[str, Any]) -> FuseCustomConfig:
        """
        Create a ``ConvertCustomConfig`` from a dictionary with the following items:

            "preserved_attributes": a list of attributes that persist even if they are not used in ``forward``

        This function is primarily for backward compatibility and may be removed in the future.
        """
        conf = cls()
        conf.set_preserved_attributes(
            fuse_custom_config_dict.get(PRESERVED_ATTRIBUTES_DICT_KEY, [])
        )
        return conf
```
- **EN**: It introduces or extends class-level abstractions such as `FuseCustomConfig`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `FuseCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 513-521 / 第 513-521 行
```python
    def to_dict(self) -> dict[str, Any]:
        """
        Convert this ``FuseCustomConfig`` to a dictionary with the items described in
        :func:`~torch.ao.quantization.fx.custom_config.ConvertCustomConfig.from_dict`.
        """
        d: dict[str, Any] = {}
        if len(self.preserved_attributes) > 0:
            d[PRESERVED_ATTRIBUTES_DICT_KEY] = self.preserved_attributes
        return d
```
- **EN**: It introduces or extends class-level abstractions such as `FuseCustomConfig`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `FuseCustomConfig` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers.
  - CN: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **StandaloneModuleConfigEntry**
  - EN: `StandaloneModuleConfigEntry` is one of the main classes that structures the file's behavior.
  - CN: `StandaloneModuleConfigEntry` 是组织该文件行为的核心类之一。
- **PrepareCustomConfig**
  - EN: `PrepareCustomConfig` is one of the main classes that structures the file's behavior.
  - CN: `PrepareCustomConfig` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch.ao.quantization:QConfigMapping`, `torch.ao.quantization.backend_config:BackendConfig`, `torch.ao.quantization.quant_type:_get_quant_type_to_str`, `torch.ao.quantization.quant_type:_quant_type_from_str`, `torch.ao.quantization.quant_type:QuantType`
- **Python standard library / Python 标准库**: `__future__:annotations`, `dataclasses:dataclass`, `typing:Any`
- **Explicit exports / 显式导出**: `ConvertCustomConfig`, `FuseCustomConfig`, `PrepareCustomConfig`, `StandaloneModuleConfigEntry`
- **Primary symbols / 核心符号**: `StandaloneModuleConfigEntry`, `PrepareCustomConfig`, `ConvertCustomConfig`, `FuseCustomConfig`
