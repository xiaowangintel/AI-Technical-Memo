# _numeric_suite.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/ns/_numeric_suite.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements numeric-suite tooling used to compare floating-point and quantized model behavior. This specific file centers on `_numeric_suite.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现数值套件工具，用于比较浮点模型与量化模型的行为。 该文件具体围绕 `_numeric_suite.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
```python
# mypy: allow-untyped-defs
from collections.abc import Callable
from typing import Any

import torch
import torch.ao.nn.quantized as nnq
import torch.ao.nn.quantized.dynamic as nnqd
import torch.nn as nn
from torch.ao.quantization import prepare
from torch.ao.quantization.quantization_mappings import (
    get_default_compare_output_module_list,
)


NON_LEAF_MODULE_TO_ADD_OBSERVER_ALLOW_LIST = {
    nnqd.Linear,
    nnq.Linear,
    nnqd.LSTM,
    nn.LSTM,
}
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.ao.nn.quantized, torch.ao.nn.quantized.dynamic, torch.nn; standard-library helpers such as collections.abc:Callable, typing:Any. Named constants such as `NON_LEAF_MODULE_TO_ADD_OBSERVER_ALLOW_LIST` centralize shared configuration or sentinel values. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.ao.nn.quantized, torch.ao.nn.quantized.dynamic, torch.nn；标准库辅助模块，如 collections.abc:Callable, typing:Any。 `NON_LEAF_MODULE_TO_ADD_OBSERVER_ALLOW_LIST` 等具名常量把共享配置或哨兵值集中定义在一起。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 23-53 / 第 23-53 行
```python
def _find_match(
    str_list: dict[str, Any] | list[str],
    key_str: str,
    postfix: str,
) -> str | None:
    split_str = key_str.split(".")
    if split_str[-1] == postfix:
        match_string = "".join(key_str.split(".")[0:-1])
        for s2 in str_list:
            pattern1 = "".join(s2.split(".")[0:-1])
            pattern2 = "".join(s2.split(".")[0:-2])
            if match_string == pattern1:
                return s2
            if match_string == pattern2:
                return s2

        # For matching "fc.weight" and "fc._packed_params._packed_params"
        if postfix == "_packed_params":
            match_string = "".join(key_str.split(".")[0:-2])
            if len(match_string) == 0:
                return None
            for s2 in str_list:
                pattern1 = "".join(s2.split(".")[0:-1])
                pattern2 = "".join(s2.split(".")[0:-2])
                if match_string == pattern1:
                    return s2
                if match_string == pattern2:
                    return s2
        return None
    else:
        return None
```
- **EN**: Key callable entry points in this range include `_find_match`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_find_match`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 56-79 / 第 56-79 行
```python
def compare_weights(
    float_dict: dict[str, Any], quantized_dict: dict[str, Any]
) -> dict[str, dict[str, torch.Tensor]]:
    r"""Compare the weights of the float module with its corresponding quantized
    module. Return a dict with key corresponding to module names and each entry being
    a dictionary with two keys 'float' and 'quantized', containing the float and
    quantized weights. This dict can be used to compare and compute the quantization
    error of the weights of float and quantized models.

    Example usage::

        wt_compare_dict = compare_weights(float_model.state_dict(), qmodel.state_dict())
        for key in wt_compare_dict:
            print(
                key,
                compute_error(
                    wt_compare_dict[key]["float"],
                    wt_compare_dict[key]["quantized"].dequantize(),
                ),
            )

    Args:
        float_dict: state dict of the float model
        quantized_dict: state dict of the quantized model
```
- **EN**: Key callable entry points in this range include `compare_weights`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `compare_weights`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 81-101 / 第 81-101 行
```python
    Return:
        weight_dict: dict with key corresponding to module names and each entry being
        a dictionary with two keys 'float' and 'quantized', containing the float and
        quantized weights
    """
    torch._C._log_api_usage_once("quantization_api._numeric_suite.compare_weights")
    weight_dict: dict[str, dict] = {}
    for key in quantized_dict:
        match_key = _find_match(float_dict, key, "weight")
        if match_key is not None:
            weight_dict[key] = {}
            weight_dict[key]["float"] = float_dict[match_key]
            weight_dict[key]["quantized"] = quantized_dict[key]
            continue

        # For matching "fc.weight" and "fc._packed_params._packed_params"
        match_key = _find_match(float_dict, key, "_packed_params")
        if match_key is not None:
            weight_dict[key] = {}
            weight_dict[key]["float"] = float_dict[match_key]
            weight_dict[key]["quantized"] = quantized_dict[key][0]
```
- **EN**: Key callable entry points in this range include `compare_weights`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `compare_weights`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 103-129 / 第 103-129 行
```python
        # For LSTM
        split_str = key.split(".")
        if split_str[-1] == "param" and split_str[-3] == "_all_weight_values":
            layer = split_str[-2]
            module_name = ".".join(split_str[:-3])
            float_weight_ih_key = module_name + ".weight_ih_l" + layer
            float_weight_hh_key = module_name + ".weight_hh_l" + layer
            if float_weight_ih_key in float_dict and float_weight_hh_key in float_dict:
                weight_dict[key] = {}
                weight_dict[key]["float"] = float_dict[float_weight_ih_key]
                weight_dict[key]["quantized"] = (
                    quantized_dict[key].__getstate__()[0][4][0].__getstate__()[0][0]
                )
                weight_dict[key]["float"] = float_dict[float_weight_hh_key]
                weight_dict[key]["quantized"] = (
                    quantized_dict[key].__getstate__()[0][4][1].__getstate__()[0][0]
                )

    return weight_dict


def _get_logger_dict_helper(
    mod: nn.Module,
    target_dict: dict[str, Any],
    prefix: str = "",
) -> None:
    r"""This is the helper function for get_logger_dict
```
- **EN**: Key callable entry points in this range include `compare_weights`, `_get_logger_dict_helper`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `compare_weights`, `_get_logger_dict_helper`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 131-160 / 第 131-160 行
```python
    Args:
        mod: module we want to save all logger stats
        prefix: prefix for the current module
        target_dict: the dictionary used to save all logger stats
    """

    def get_prefix(prefix):
        return prefix if prefix == "" else prefix + "."

    for child in mod.children():
        if isinstance(child, Logger):
            target_dict[get_prefix(prefix) + "stats"] = child.stats
            break

    for name, child in mod.named_children():
        module_prefix = get_prefix(prefix) + name if prefix else name
        _get_logger_dict_helper(child, target_dict, module_prefix)


def get_logger_dict(mod: nn.Module, prefix: str = "") -> dict[str, dict]:
    r"""Traverse the modules and save all logger stats into target dict.
    This is mainly used for quantization accuracy debug.

    Type of loggers supported:
        ShadowLogger: used to log the outputs of the quantized module and its matching float shadow module,
        OutputLogger: used to log the outputs of the modules

    Args:
        mod: module we want to save all logger stats
        prefix: prefix for the current module
```
- **EN**: Key callable entry points in this range include `_get_logger_dict_helper`, `get_logger_dict`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_get_logger_dict_helper`, `get_logger_dict`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 162-188 / 第 162-188 行
```python
    Return:
        target_dict: the dictionary used to save all logger stats

    """
    torch._C._log_api_usage_once("quantization_api._numeric_suite.get_logger_dict")

    target_dict: dict[str, dict] = {}
    _get_logger_dict_helper(mod, target_dict, prefix)
    return target_dict


class Logger(nn.Module):
    r"""Base class for stats logging"""

    def __init__(self):
        super().__init__()
        self.stats = {}
        # We only insert observer if the op is quantized with static quantization,
        # which is identified by activation_observer.dtype == quint8.  This is needed
        # when attaching Logger as observer for FX mode
        self.dtype = torch.quint8

    def forward(self, x):
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
```
- **EN**: It introduces or extends class-level abstractions such as `Logger`, which organize state and behavior for this subsystem. Key callable entry points in this range include `get_logger_dict`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Logger` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `get_logger_dict`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 191-219 / 第 191-219 行
```python
class ShadowLogger(Logger):
    r"""Class used in Shadow module to record the outputs of the original and
    shadow modules.
    """

    def __init__(self):
        super().__init__()
        self.stats["float"] = []
        self.stats["quantized"] = []

    def forward(self, x, y):  # type: ignore[override]
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        if len(x) > 1:
            x = x[0]
        if len(y) > 1:
            y = y[0]
        self.stats["quantized"].append(x.detach())
        self.stats["float"].append(y.detach())


class OutputLogger(Logger):
    r"""Class used to log the outputs of the module"""

    def __init__(self):
        super().__init__()
        self.stats["tensor_val"] = []
```
- **EN**: It introduces or extends class-level abstractions such as `ShadowLogger`, `OutputLogger`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ShadowLogger`, `OutputLogger` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 221-247 / 第 221-247 行
```python
    def forward(self, x):
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        self.stats["tensor_val"].append(x)
        return x


def _convert_tuple_to_list(t: Any) -> Any:
    return [_convert_tuple_to_list(x) for x in t] if type(t) is tuple else t


def _dequantize_tensor_list(t: Any) -> Any:
    return (
        [_dequantize_tensor_list(x) for x in t]
        if type(t) is list
        else t.dequantize()
        if t.is_quantized
        else t
    )


class Shadow(nn.Module):
    r"""Shadow module attaches the float module to its matching quantized module
    as the shadow. Then it uses Logger module to process the outputs of both
    modules.
```
- **EN**: It introduces or extends class-level abstractions such as `OutputLogger`, `Shadow`, which organize state and behavior for this subsystem. Key callable entry points in this range include `_convert_tuple_to_list`, `_dequantize_tensor_list`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `OutputLogger`, `Shadow` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `_convert_tuple_to_list`, `_dequantize_tensor_list`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 249-273 / 第 249-273 行
```python
    Args:
        q_module: module quantized from float_module that we want to shadow
        float_module: float module used to shadow q_module
        logger_cls: type of logger used to process the outputs of q_module and
            float_module. ShadowLogger or custom loggers can be used.
    """

    def __init__(self, q_module, float_module, logger_cls):
        super().__init__()
        self.orig_module = q_module
        self.shadow_module = float_module
        self.dequant = nnq.DeQuantize()
        self.logger = logger_cls()

    def forward(self, *x) -> torch.Tensor:
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        xl = _convert_tuple_to_list(x)
        output = self.orig_module(*xl)
        xl_float = _dequantize_tensor_list(xl)
        shadow_output = self.shadow_module(*xl_float)
        self.logger(output, shadow_output)
        return output
```
- **EN**: It introduces or extends class-level abstractions such as `Shadow`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Shadow` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 275-296 / 第 275-296 行
```python
    def add(self, x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        output = self.orig_module.add(x, y)
        x = x.dequantize()
        y = y.dequantize()
        shadow_output = self.shadow_module.add(x, y)
        self.logger(output, shadow_output)
        return output

    def add_scalar(self, x: torch.Tensor, y: float) -> torch.Tensor:
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        output = self.orig_module.add_scalar(x, y)
        x = x.dequantize()
        shadow_output = self.shadow_module.add_scalar(x, y)
        self.logger(output, shadow_output)
        return output
```
- **EN**: It introduces or extends class-level abstractions such as `Shadow`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Shadow` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 298-319 / 第 298-319 行
```python
    def mul(self, x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        output = self.orig_module.mul(x, y)
        x = x.dequantize()
        y = y.dequantize()
        shadow_output = self.shadow_module.mul(x, y)
        self.logger(output, shadow_output)
        return output

    def mul_scalar(self, x: torch.Tensor, y: float) -> torch.Tensor:
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        output = self.orig_module.mul_scalar(x, y)
        x = x.dequantize()
        shadow_output = self.shadow_module.mul_scalar(x, y)
        self.logger(output, shadow_output)
        return output
```
- **EN**: It introduces or extends class-level abstractions such as `Shadow`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Shadow` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 321-342 / 第 321-342 行
```python
    def cat(self, x: list[torch.Tensor], dim: int = 0) -> torch.Tensor:
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        output = self.orig_module.cat(x, dim)
        x = [y.dequantize() for y in x]
        shadow_output = self.shadow_module.cat(x, dim)
        self.logger(output, shadow_output)
        return output

    def add_relu(self, x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
        # fmt: off
        """
        """  # blank docblock to make autodoc happy
        # fmt: on
        output = self.orig_module.add_relu(x, y)
        x = x.dequantize()
        y = y.dequantize()
        shadow_output = self.shadow_module.add_relu(x, y)
        self.logger(output, shadow_output)
        return output
```
- **EN**: It introduces or extends class-level abstractions such as `Shadow`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Shadow` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 345-371 / 第 345-371 行
```python
def prepare_model_with_stubs(
    float_module: nn.Module,
    q_module: nn.Module,
    module_swap_list: set[type],
    logger_cls: Callable,
) -> None:
    r"""Prepare the model by attaching the float module to its matching quantized
    module as the shadow if the float module type is in module_swap_list.

    Example usage::

        prepare_model_with_stubs(float_model, q_model, module_swap_list, Logger)
        q_model(data)
        ob_dict = get_logger_dict(q_model)

    Args:
        float_module: float module used to generate the q_module
        q_module: module quantized from float_module
        module_swap_list: list of float module types to attach the shadow
        logger_cls: type of logger to be used in shadow module to process the outputs of
            quantized module and its float shadow module
    """
    torch._C._log_api_usage_once(
        "quantization_api._numeric_suite.prepare_model_with_stubs"
    )

    float_module_children = dict(float_module.named_children())
```
- **EN**: Key callable entry points in this range include `prepare_model_with_stubs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `prepare_model_with_stubs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 373-398 / 第 373-398 行
```python
    reassign = {}
    for name, mod in q_module.named_children():
        if name not in float_module_children:
            continue

        float_mod = float_module_children[name]

        if type(float_mod) not in module_swap_list:
            prepare_model_with_stubs(float_mod, mod, module_swap_list, logger_cls)

        # Insert shadow module only if the module is not of the same type as
        # the floating point module
        if type(float_mod) in module_swap_list and not _is_identical_module_type(
            mod, float_mod
        ):
            reassign[name] = Shadow(mod, float_mod, logger_cls)

    for key, value in reassign.items():
        q_module._modules[key] = value


def _is_identical_module_type(mod1, mod2):
    # Compare if two modules have the same dtype
    mod1_module_types = [type(mod) for mod in mod1.modules()]
    mod2_module_types = [type(mod) for mod in mod2.modules()]
    return mod1_module_types == mod2_module_types
```
- **EN**: Key callable entry points in this range include `prepare_model_with_stubs`, `_is_identical_module_type`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `prepare_model_with_stubs`, `_is_identical_module_type`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 401-423 / 第 401-423 行
```python
def compare_model_stub(
    float_model: nn.Module,
    q_model: nn.Module,
    module_swap_list: set[type],
    *data,
    logger_cls=ShadowLogger,
) -> dict[str, dict]:
    r"""Compare quantized module in a model with its floating point counterpart,
    feeding both of them the same input. Return a dict with key corresponding to
    module names and each entry being a dictionary with two keys 'float' and
    'quantized', containing the output tensors of quantized and its matching
    float shadow module. This dict can be used to compare and compute the module
    level quantization error.

    This function first call prepare_model_with_stubs() to swap the quantized
    module that we want to compare with the Shadow module, which takes quantized
    module, corresponding float module and logger as input, and creates a forward
    path inside to make the float module to shadow quantized module sharing the
    same input. The logger can be customizable, default logger is ShadowLogger
    and it will save the outputs of the quantized module and float module that
    can be used to compute the module level quantization error.

    Example usage::
```
- **EN**: Key callable entry points in this range include `compare_model_stub`. They package a focused unit of behavior behind named helpers or APIs. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `compare_model_stub`，它们把聚焦的行为封装成具名辅助函数或 API。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 425-450 / 第 425-450 行
```python
        module_swap_list = [
            torchvision.models.quantization.resnet.QuantizableBasicBlock
        ]
        ob_dict = compare_model_stub(float_model, qmodel, module_swap_list, data)
        for key in ob_dict:
            print(
                key,
                compute_error(
                    ob_dict[key]["float"], ob_dict[key]["quantized"].dequantize()
                ),
            )

    Args:
        float_model: float model used to generate the q_model
        q_model: model quantized from float_model
        module_swap_list: list of float module types at which shadow modules will
            be attached.
        data: input data used to run the prepared q_model
        logger_cls: type of logger to be used in shadow module to process the outputs of
            quantized module and its float shadow module
    """
    torch._C._log_api_usage_once("quantization_api._numeric_suite.compare_model_stub")
    prepare_model_with_stubs(float_model, q_model, module_swap_list, logger_cls)
    q_model(*data)
    ob_dict = get_logger_dict(q_model)
    return ob_dict
```
- **EN**: Key callable entry points in this range include `compare_model_stub`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `compare_model_stub`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 453-482 / 第 453-482 行
```python
def get_matching_activations(
    float_module: nn.Module,
    q_module: nn.Module,
) -> dict[str, dict[str, torch.Tensor]]:
    r"""Find the matching activation between float and quantized modules.

    Args:
        float_module: float module used to generate the q_module
        q_module: module quantized from float_module

    Return:
        act_dict: dict with key corresponding to quantized module names and each
        entry being a dictionary with two keys 'float' and 'quantized', containing
        the matching float and quantized activations
    """
    torch._C._log_api_usage_once(
        "quantization_api._numeric_suite.get_matching_activations"
    )
    float_dict = get_logger_dict(float_module)
    quantized_dict = get_logger_dict(q_module)
    act_dict: dict[str, dict] = {}
    for key in quantized_dict:
        if len(quantized_dict[key]["tensor_val"]) == 0:
            continue
        match_key = _find_match(sorted(float_dict, reverse=True), key, "stats")
        if match_key is not None:
            act_dict[key] = {}
            act_dict[key]["float"] = float_dict[match_key]["tensor_val"]
            act_dict[key]["quantized"] = quantized_dict[key]["tensor_val"]
    return act_dict
```
- **EN**: Key callable entry points in this range include `get_matching_activations`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_matching_activations`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 485-516 / 第 485-516 行
```python
def prepare_model_outputs(
    float_module: nn.Module,
    q_module: nn.Module,
    logger_cls=OutputLogger,
    allow_list=None,
) -> None:
    r"""Prepare the model by attaching the logger to both float module
    and quantized module if they are in the allow_list.

    Args:
        float_module: float module used to generate the q_module
        q_module: module quantized from float_module
        logger_cls: type of logger to be attached to float_module and q_module
        allow_list: list of module types to attach logger
    """
    torch._C._log_api_usage_once(
        "quantization_api._numeric_suite.prepare_model_outputs"
    )
    if allow_list is None:
        allow_list = get_default_compare_output_module_list()

    qconfig_debug = torch.ao.quantization.QConfig(activation=logger_cls, weight=None)
    float_module.qconfig = qconfig_debug  # type: ignore[assignment]
    prepare(
        float_module, inplace=True, allow_list=allow_list, prepare_custom_config_dict={}
    )
    q_module.qconfig = qconfig_debug  # type: ignore[assignment]
    prepare(
        q_module,
        inplace=True,
        allow_list=allow_list,
        observer_non_leaf_module_list=NON_LEAF_MODULE_TO_ADD_OBSERVER_ALLOW_LIST,
```
- **EN**: Key callable entry points in this range include `prepare_model_outputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `prepare_model_outputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 517-545 / 第 517-545 行
```python
        prepare_custom_config_dict={},
    )


def compare_model_outputs(
    float_model: nn.Module,
    q_model: nn.Module,
    *data,
    logger_cls=OutputLogger,
    allow_list=None,
) -> dict[str, dict[str, torch.Tensor]]:
    r"""Compare output activations between float and quantized models at
    corresponding locations for the same input. Return a dict with key corresponding
    to quantized module names and each entry being a dictionary with two keys
    'float' and 'quantized', containing the activations of quantized model and
    float model at matching locations. This dict can be used to compare and
    compute the propagation quantization error.

    Example usage::

        act_compare_dict = compare_model_outputs(float_model, qmodel, data)
        for key in act_compare_dict:
            print(
                key,
                compute_error(
                    act_compare_dict[key]["float"],
                    act_compare_dict[key]["quantized"].dequantize(),
                ),
            )
```
- **EN**: Key callable entry points in this range include `prepare_model_outputs`, `compare_model_outputs`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `prepare_model_outputs`, `compare_model_outputs`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 547-568 / 第 547-568 行
```python
    Args:
        float_model: float model used to generate the q_model
        q_model: model quantized from float_model
        data: input data used to run the prepared float_model and q_model
        logger_cls: type of logger to be attached to float_module and q_module
        allow_list: list of module types to attach logger

    Return:
        act_compare_dict: dict with key corresponding to quantized module names
        and each entry being a dictionary with two keys 'float' and 'quantized',
        containing the matching float and quantized activations
    """
    torch._C._log_api_usage_once(
        "quantization_api._numeric_suite.compare_model_outputs"
    )
    if allow_list is None:
        allow_list = get_default_compare_output_module_list()
    prepare_model_outputs(float_model, q_model, logger_cls, allow_list)
    float_model(*data)
    q_model(*data)
    act_compare_dict = get_matching_activations(float_model, q_model)
    return act_compare_dict
```
- **EN**: Key callable entry points in this range include `compare_model_outputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `compare_model_outputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements numeric-suite tooling used to compare floating-point and quantized model behavior.
  - CN: 实现数值套件工具，用于比较浮点模型与量化模型的行为。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Observer statistics**
  - EN: Collects runtime ranges or histograms so later code can derive quantization settings.
  - CN: 收集运行时范围或直方图，以便后续逻辑推导量化设置。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Numeric comparison**
  - EN: Helps compare reference and transformed executions to diagnose quantization effects.
  - CN: 帮助比较参考执行与转换后执行，从而诊断量化带来的影响。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.quantized`, `torch.ao.nn.quantized.dynamic`, `torch.nn`, `torch.ao.quantization:prepare`, `torch.ao.quantization.quantization_mappings:get_default_compare_output_module_list`
- **Python standard library / Python 标准库**: `collections.abc:Callable`, `typing:Any`
- **Primary symbols / 核心符号**: `Logger`, `ShadowLogger`, `OutputLogger`, `Shadow`, `_find_match`, `compare_weights`, `_get_logger_dict_helper`, `get_logger_dict`, `_convert_tuple_to_list`, `_dequantize_tensor_list`, `prepare_model_with_stubs`, `_is_identical_module_type`, `compare_model_stub`, `get_matching_activations`, `prepare_model_outputs`
