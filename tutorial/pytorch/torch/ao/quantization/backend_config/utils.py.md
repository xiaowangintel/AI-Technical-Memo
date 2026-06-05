# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/quantization/backend_config/utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers. This specific file centers on `utils.py`. It also participates in graph/module transformation before or after quantized execution. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。 该文件具体围绕 `utils.py` 展开。 它也参与量化执行前后的图/模块转换。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行
```python
# mypy: allow-untyped-defs
from collections.abc import Callable
from typing import Any

import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.ao.quantization.fuser_method_mappings import _reverse2, _reverse3
from torch.ao.quantization.utils import Pattern

from .backend_config import BackendConfig, BackendPatternConfig, DTypeConfig


__all__ = [
    "get_pattern_to_dtype_configs",
    "get_qat_module_classes",
    "get_fused_module_classes",
    "get_pattern_to_input_type_to_index",
    "get_root_module_to_quantized_reference_module",
    "get_fuser_method_mapping",
    "get_module_to_qat_module",
    "get_fusion_pattern_to_root_node_getter",
    "get_fusion_pattern_to_extra_inputs_getter",
    "remove_boolean_dispatch_from_name",
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.nn, torch.nn.functional, torch.ao.quantization.fuser_method_mappings:_reverse2; standard-library helpers such as collections.abc:Callable, typing:Any. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.nn, torch.nn.functional, torch.ao.quantization.fuser_method_mappings:_reverse2；标准库辅助模块，如 collections.abc:Callable, typing:Any。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 25-45 / 第 25-45 行
```python
    "pattern_to_human_readable",
    "entry_to_pretty_str",
]


def get_pattern_to_dtype_configs(
    backend_config: BackendConfig,
) -> dict[Pattern, list[DTypeConfig]]:
    pattern_to_dtype_configs: dict[Pattern, list[DTypeConfig]] = {}
    for pattern, config in backend_config._pattern_complex_format_to_config.items():
        pattern_to_dtype_configs[pattern] = config.dtype_configs
    return pattern_to_dtype_configs


def get_qat_module_classes(backend_config: BackendConfig) -> tuple[type, ...]:
    qat_module_classes = [
        config.qat_module
        for config in backend_config.configs
        if config.qat_module is not None
    ]
    return tuple(set(qat_module_classes))
```
- **EN**: Key callable entry points in this range include `get_pattern_to_dtype_configs`, `get_qat_module_classes`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_pattern_to_dtype_configs`, `get_qat_module_classes`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 48-63 / 第 48-63 行
```python
def get_fused_module_classes(backend_config: BackendConfig) -> tuple[type, ...]:
    fused_module_classes = [
        config.fused_module
        for config in backend_config.configs
        if config.fused_module is not None
    ]
    return tuple(set(fused_module_classes))


def get_pattern_to_input_type_to_index(
    backend_config: BackendConfig,
) -> dict[Pattern, dict[str, int]]:
    pattern_to_input_type_to_index: dict[Pattern, dict[str, int]] = {}
    for pattern, config in backend_config._pattern_complex_format_to_config.items():
        pattern_to_input_type_to_index[pattern] = config._input_type_to_index
    return pattern_to_input_type_to_index
```
- **EN**: Key callable entry points in this range include `get_fused_module_classes`, `get_pattern_to_input_type_to_index`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_fused_module_classes`, `get_pattern_to_input_type_to_index`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 66-89 / 第 66-89 行
```python
def get_root_module_to_quantized_reference_module(
    backend_config: BackendConfig,
) -> dict[type[torch.nn.Module], type[torch.nn.Module]]:
    mapping: dict[type[torch.nn.Module], type[torch.nn.Module]] = {}
    for config in backend_config.configs:
        if (
            config.root_module is not None
            and config.reference_quantized_module is not None
        ):
            mapping[config.root_module] = config.reference_quantized_module
    return mapping


def get_fuser_method_mapping(
    backend_config: BackendConfig,
) -> dict[Pattern, nn.Sequential | Callable]:
    fuser_method_mapping: dict[Pattern, nn.Sequential | Callable] = {}
    for pattern, config in backend_config._pattern_complex_format_to_config.items():
        if config.fuser_method is not None:
            # Note: both the fuser method and the pattern are specified in forward order in the
            # BackendConfig, but the internal pattern matching code uses the reversed nested tuple
            # format, so we need to convert both to the internal format
            fuser_method = _get_fuser_method_in_reversed_nested_tuple_format(config)
            fuser_method_mapping[pattern] = fuser_method
```
- **EN**: Key callable entry points in this range include `get_root_module_to_quantized_reference_module`, `get_fuser_method_mapping`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_root_module_to_quantized_reference_module`, `get_fuser_method_mapping`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 90-113 / 第 90-113 行
```python
    return fuser_method_mapping


def get_module_to_qat_module(
    backend_config: BackendConfig,
) -> dict[Pattern, type[torch.nn.Module]]:
    module_to_qat_module: dict[Pattern, type[torch.nn.Module]] = {}
    for pattern, config in backend_config._pattern_complex_format_to_config.items():
        if config.qat_module is not None:
            module_to_qat_module[pattern] = config.qat_module
    return module_to_qat_module


def get_fusion_pattern_to_root_node_getter(
    backend_config: BackendConfig,
) -> dict[Pattern, Callable]:
    """Get a map from fusion pattern to a function that returns the root node
    from the fusion pattern, e.g. the most common one is:
    def get_root_node(node_pattern):
        while not isinstance(node_pattern[-1], Node):
            node_pattern = node_pattern[-1]
        return node_pattern[-1]
    This can work for all patterns whose root node is the "last node" in the pattern,
    e.g. (torch.add, MatchAllNode, (torch.ReLU, torch.Conv2d))
```
- **EN**: Key callable entry points in this range include `get_fuser_method_mapping`, `get_module_to_qat_module`, `get_fusion_pattern_to_root_node_getter`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_fuser_method_mapping`, `get_module_to_qat_module`, `get_fusion_pattern_to_root_node_getter`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 114-137 / 第 114-137 行
```python
    """
    root_node_getter_mapping: dict[Pattern, Callable] = {}
    for pattern, config in backend_config._pattern_complex_format_to_config.items():
        if config._root_node_getter is not None:
            root_node_getter_mapping[pattern] = config._root_node_getter
    return root_node_getter_mapping


def get_fusion_pattern_to_extra_inputs_getter(
    backend_config: BackendConfig,
) -> dict[Pattern, Callable]:
    """Get a map from fusion pattern to a function that returns extra input nodes
    from the fusion pattern, in the order required by the root node. This is optional,
    if not specified, we will not copy over any extra inputs for the root node.
    Example:
    # Let's say we have the pattern (torch.add, MatchAllNode, (torch.nn.BatchNorm2d, torch.nn.Conv2d))
    # and root node is torch.nn.Conv2d, and the node in MatchAllNode would be an extra
    # argument to the fused module, we can unpack the pattern and return the node at
    # MatchAllNode here
    # we can implement extra_inputs_getter as follows:
    def extra_inputs_getter(pattern) -> List[Any]:
        add, extra_input, conv_pattern = pattern
        return [extra_input]
    """
```
- **EN**: Key callable entry points in this range include `get_fusion_pattern_to_root_node_getter`, `get_fusion_pattern_to_extra_inputs_getter`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_fusion_pattern_to_root_node_getter`, `get_fusion_pattern_to_extra_inputs_getter`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 138-161 / 第 138-161 行
```python
    extra_inputs_getter_mapping: dict[Pattern, Callable] = {}
    for pattern, config in backend_config._pattern_complex_format_to_config.items():
        if config._extra_inputs_getter is not None:
            extra_inputs_getter_mapping[pattern] = config._extra_inputs_getter
    return extra_inputs_getter_mapping


def remove_boolean_dispatch_from_name(p) -> Any:
    """
    Some ops have a default string representation such as
    '<function boolean_dispatch.<locals>.fn at 0x7ff1106bf280>',
    this function replaces them with the hardcoded function names.
    """
    if p is F.fractional_max_pool2d:
        return "torch.nn.functional.fractional_max_pool2d"
    elif p is F.fractional_max_pool3d:
        return "torch.nn.functional.fractional_max_pool3d"
    elif p is F.max_pool1d:
        return "torch.nn.functional.max_pool1d"
    elif p is F.max_pool2d:
        return "torch.nn.functional.max_pool2d"
    elif p is F.max_pool3d:
        return "torch.nn.functional.max_pool3d"
    elif p is F.adaptive_max_pool1d:
```
- **EN**: Key callable entry points in this range include `get_fusion_pattern_to_extra_inputs_getter`, `remove_boolean_dispatch_from_name`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_fusion_pattern_to_extra_inputs_getter`, `remove_boolean_dispatch_from_name`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 162-184 / 第 162-184 行
```python
        return "torch.nn.functional.adaptive_max_pool1d"
    elif p is F.adaptive_max_pool2d:
        return "torch.nn.functional.adaptive_max_pool2d"
    elif p is F.adaptive_max_pool3d:
        return "torch.nn.functional.adaptive_max_pool3d"
    if "boolean_dispatch" in str(p):
        raise AssertionError(
            f"{p} does not have a human readable representation in "
            + "quantization documentation"
        )
    return p


def pattern_to_human_readable(p) -> Any:
    if isinstance(p, tuple):
        # nested patterns, recurse
        return tuple(pattern_to_human_readable(inner_p) for inner_p in p)
    elif isinstance(p, str):
        # method names are already human readable
        return p
    else:
        p = remove_boolean_dispatch_from_name(p)
        return p
```
- **EN**: Key callable entry points in this range include `remove_boolean_dispatch_from_name`, `pattern_to_human_readable`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `remove_boolean_dispatch_from_name`, `pattern_to_human_readable`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 187-210 / 第 187-210 行
```python
# TODO(future PR): move backend_config_dict to use dataclass and move this logic to
# the corresponding __str__ function
def entry_to_pretty_str(entry) -> str:
    """
    Given a backend_config_dict entry, returns a string with the human readable
    representation of it.
    """
    s = "{\n"

    # always output the pattern first
    if "pattern" in entry:
        pattern_str = pattern_to_human_readable(entry["pattern"])

        s += f"  'pattern': {pattern_str},\n"

    # custom output for dtype_configs to make it look nice
    if "dtype_configs" in entry:
        s += "  'dtype_configs': [\n"
        for dtype_config in entry["dtype_configs"]:
            s += "    {\n"
            for k, v in dtype_config.items():
                s += f"      '{k}': {v},\n"
            s += "    },\n"
        s += "  ],\n"
```
- **EN**: Key callable entry points in this range include `entry_to_pretty_str`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `entry_to_pretty_str`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 212-231 / 第 212-231 行
```python
    # custom output for num_tensor_args_to_observation_type to make it look nice
    if "num_tensor_args_to_observation_type" in entry:
        s += "  'num_tensor_args_to_observation_type': {\n"
        for k, v in entry["num_tensor_args_to_observation_type"].items():
            s += f"    {k}: {v},\n"
        s += "  },\n"

    # output all the other fields
    custom_handled_fields = [
        "pattern",
        "dtype_configs",
        "num_tensor_args_to_observation_type",
    ]
    for field_name in entry:
        if field_name in custom_handled_fields:
            continue
        s += f"  '{field_name}': {entry[field_name]},\n"

    s += "}"
    return s
```
- **EN**: Key callable entry points in this range include `entry_to_pretty_str`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `entry_to_pretty_str`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 234-251 / 第 234-251 行
```python
def _get_pattern_in_reversed_nested_tuple_format(
    config: BackendPatternConfig,
) -> Pattern:
    """
    Return the pattern specified in the given config in the reversed nested tuple format
    used internally in the quantization pattern matching code.

    If the pattern is not a tuple, or the pattern is already specified in the reversed
    nested tuple format, return the pattern as is. Otherwise:

    For 2-tuples (a, b), return (b, a).
    For 3-tuples (a, b, c), return (c, (b, a)).

    For example:
        * Given nn.Linear, return nn.Linear
        * Given (nn.Linear, nn.ReLU), return (nn.ReLU, nn.Linear)
        * Given (nn.Conv2d, nn.BatchNorm2d, nn.ReLU), return
          (nn.ReLU, (nn.BatchNorm2d, nn.Conv2d))
```
- **EN**: Key callable entry points in this range include `_get_pattern_in_reversed_nested_tuple_format`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_pattern_in_reversed_nested_tuple_format`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 253-273 / 第 253-273 行
```python
    For context, the reason why this is needed is the user-facing BackendConfig
    API accepts the flat 2-or-3-tuple format in forward order. While this simple
    format handles the vast majority of use cases, it does not handle the more
    complex ones, and so the internal pattern matching code for quantization uses
    the following, more general reversed nested tuple format instead:

        operator = module_type | functional | torch op | native op | MatchAllNode
        Pattern = (operator, Pattern, Pattern, ...) | operator

    In the future, we expect to replace the above complex format with the one used
    by the subgraph rewriter in torch.fx, so we don't have to maintain our own
    complex pattern matching code. Then we won't need this helper function anymore.
    """
    if config._pattern_complex_format is not None:
        return config._pattern_complex_format
    if config.pattern is None:
        raise ValueError(
            "Either 'pattern' or 'pattern_complex_format' must be specified"
        )
    if not isinstance(config.pattern, tuple):
        return config.pattern
```
- **EN**: Key callable entry points in this range include `_get_pattern_in_reversed_nested_tuple_format`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_pattern_in_reversed_nested_tuple_format`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 275-297 / 第 275-297 行
```python
    # Pattern is specified in the simple tuple format, need to convert
    if len(config.pattern) == 2:
        (a, b) = config.pattern
        return (b, a)
    elif len(config.pattern) == 3:
        (a, b, c) = config.pattern
        return (c, (b, a))
    else:
        raise ValueError(
            f"Expected a tuple with 2 or 3 elements, got: {config.pattern}"
        )


def _get_fuser_method_in_reversed_nested_tuple_format(
    config: BackendPatternConfig,
) -> Callable:
    """
    Return the fuser method specified in the given config in the reversed nested
    tuple format used internally in the quantization pattern matching code.

    If pattern is specified in the reversed nested tuple format, we assume the
    fuser method is also specified in this format and simply return it as is.
    Otherwise, we convert the fuser method as follows:
```
- **EN**: Key callable entry points in this range include `_get_pattern_in_reversed_nested_tuple_format`, `_get_fuser_method_in_reversed_nested_tuple_format`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_pattern_in_reversed_nested_tuple_format`, `_get_fuser_method_in_reversed_nested_tuple_format`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 299-321 / 第 299-321 行
```python
        * Given f(is_qat, conv, relu), return f'(is_qat, relu, conv)
        * Given f(is_qat, conv, bn, relu), return f'(is_qat, relu, bn_conv),
          where bn_conv is a 2-tuple (bn, conv)

    The first argument of a fuser method is always `is_qat` and is not affected
    in the conversion. We currently only support functions with 3 or 4 arguments.
    """
    if config.fuser_method is None:
        raise AssertionError("config.fuser_method must be provided")
    if config._pattern_complex_format is not None:
        return config.fuser_method
    if not isinstance(config.pattern, tuple):
        raise ValueError(f"Expected pattern to be a tuple, got: {config.pattern}")

    # Pattern is specified in the simple tuple format, need to convert
    if len(config.pattern) == 2:
        return _reverse2(config.fuser_method)
    elif len(config.pattern) == 3:
        return _reverse3(config.fuser_method)
    else:
        raise ValueError(
            f"Expected a tuple with 2 or 3 elements, got: {config.pattern}"
        )
```
- **EN**: Key callable entry points in this range include `_get_fuser_method_in_reversed_nested_tuple_format`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_fuser_method_in_reversed_nested_tuple_format`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

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
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.nn`, `torch.nn.functional`, `torch.ao.quantization.fuser_method_mappings:_reverse2`, `torch.ao.quantization.fuser_method_mappings:_reverse3`, `torch.ao.quantization.utils:Pattern`, `.backend_config:BackendConfig`, `.backend_config:BackendPatternConfig`, `.backend_config:DTypeConfig`
- **Python standard library / Python 标准库**: `collections.abc:Callable`, `typing:Any`
- **Explicit exports / 显式导出**: `get_pattern_to_dtype_configs`, `get_qat_module_classes`, `get_fused_module_classes`, `get_pattern_to_input_type_to_index`, `get_root_module_to_quantized_reference_module`, `get_fuser_method_mapping`, `get_module_to_qat_module`, `get_fusion_pattern_to_root_node_getter`, `get_fusion_pattern_to_extra_inputs_getter`, `remove_boolean_dispatch_from_name`, `pattern_to_human_readable`, `entry_to_pretty_str`
- **Primary symbols / 核心符号**: `get_pattern_to_dtype_configs`, `get_qat_module_classes`, `get_fused_module_classes`, `get_pattern_to_input_type_to_index`, `get_root_module_to_quantized_reference_module`, `get_fuser_method_mapping`, `get_module_to_qat_module`, `get_fusion_pattern_to_root_node_getter`, `get_fusion_pattern_to_extra_inputs_getter`, `remove_boolean_dispatch_from_name`, `pattern_to_human_readable`, `entry_to_pretty_str`, `_get_pattern_in_reversed_nested_tuple_format`, `_get_fuser_method_in_reversed_nested_tuple_format`
