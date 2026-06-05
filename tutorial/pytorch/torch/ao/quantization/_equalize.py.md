# _equalize.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/quantization/_equalize.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers. This specific file centers on `_equalize.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。 该文件具体围绕 `_equalize.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行
```python
# mypy: allow-untyped-defs
import copy
from itertools import chain
from typing import Any

import torch


__all__ = [
    "set_module_weight",
    "set_module_bias",
    "has_bias",
    "get_module_weight",
    "get_module_bias",
    "max_over_ndim",
    "min_over_ndim",
    "channel_range",
    "get_name_by_module",
    "cross_layer_equalization",
    "process_paired_modules_list_to_name",
    "expand_groups_in_paired_modules_list",
    "equalize",
    "converged",
]
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 26-46 / 第 26-46 行
```python
_supported_types = {torch.nn.Conv2d, torch.nn.Linear, torch.nn.Conv1d}
_supported_intrinsic_types = {
    torch.ao.nn.intrinsic.ConvReLU2d,
    torch.ao.nn.intrinsic.LinearReLU,
    torch.ao.nn.intrinsic.ConvReLU1d,
}
_all_supported_types = _supported_types.union(_supported_intrinsic_types)


def set_module_weight(module, weight) -> None:
    if type(module) in _supported_types:
        module.weight = torch.nn.Parameter(weight)
    else:
        module[0].weight = torch.nn.Parameter(weight)


def set_module_bias(module, bias) -> None:
    if type(module) in _supported_types:
        module.bias = torch.nn.Parameter(bias)
    else:
        module[0].bias = torch.nn.Parameter(bias)
```
- **EN**: Key callable entry points in this range include `set_module_weight`, `set_module_bias`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `set_module_weight`, `set_module_bias`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 49-67 / 第 49-67 行
```python
def has_bias(module) -> bool:
    if type(module) in _supported_types:
        return module.bias is not None
    else:
        return module[0].bias is not None


def get_module_weight(module):
    if type(module) in _supported_types:
        return module.weight
    else:
        return module[0].weight


def get_module_bias(module):
    if type(module) in _supported_types:
        return module.bias
    else:
        return module[0].bias
```
- **EN**: Key callable entry points in this range include `has_bias`, `get_module_weight`, `get_module_bias`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `has_bias`, `get_module_weight`, `get_module_bias`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 70-90 / 第 70-90 行
```python
def max_over_ndim(input, axis_list, keepdim=False):
    """Apply 'torch.max' over the given axes."""
    axis_list.sort(reverse=True)
    for axis in axis_list:
        input, _ = input.max(axis, keepdim)
    return input


def min_over_ndim(input, axis_list, keepdim=False):
    """Apply 'torch.min' over the given axes."""
    axis_list.sort(reverse=True)
    for axis in axis_list:
        input, _ = input.min(axis, keepdim)
    return input


def channel_range(input, axis=0):
    """Find the range of weights associated with a specific channel."""
    size_of_tensor_dim = input.ndim
    axis_list = list(range(size_of_tensor_dim))
    axis_list.remove(axis)
```
- **EN**: Key callable entry points in this range include `max_over_ndim`, `min_over_ndim`, `channel_range`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `max_over_ndim`, `min_over_ndim`, `channel_range`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 92-107 / 第 92-107 行
```python
    mins = min_over_ndim(input, axis_list)
    maxs = max_over_ndim(input, axis_list)

    if mins.size(0) != input.size(axis):
        raise AssertionError(
            "Dimensions of resultant channel range does not match size of requested axis"
        )
    return maxs - mins


def get_name_by_module(model, module):
    """Get the name of a module within a model.

    Args:
        model: a model (nn.module) that equalization is to be applied on
        module: a module within the model
```
- **EN**: Key callable entry points in this range include `channel_range`, `get_name_by_module`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `channel_range`, `get_name_by_module`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 109-131 / 第 109-131 行
```python
    Returns:
        name: the name of the module within the model
    """
    for name, m in model.named_modules():
        if m is module:
            return name
    raise ValueError("module is not in the model")


def cross_layer_equalization(module1, module2, output_axis=0, input_axis=1):
    """Scale the range of Tensor1.output to equal Tensor2.input.

    Given two adjacent tensors', the weights are scaled such that
    the ranges of the first tensors' output channel are equal to the
    ranges of the second tensors' input channel
    """
    if (
        type(module1) not in _all_supported_types
        or type(module2) not in _all_supported_types
    ):
        raise ValueError(
            "module type not supported:", type(module1), " ", type(module2)
        )
```
- **EN**: Key callable entry points in this range include `get_name_by_module`, `cross_layer_equalization`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_name_by_module`, `cross_layer_equalization`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 133-153 / 第 133-153 行
```python
    bias = get_module_bias(module1) if has_bias(module1) else None

    weight1 = get_module_weight(module1)
    weight2 = get_module_weight(module2)

    if weight1.size(output_axis) != weight2.size(input_axis):
        raise TypeError(
            "Number of output channels of first arg do not match \
        number input channels of second arg"
        )

    weight1_range = channel_range(weight1, output_axis)
    weight2_range = channel_range(weight2, input_axis)

    # producing scaling factors to applied
    weight2_range += 1e-9
    scaling_factors = torch.sqrt(weight1_range / weight2_range)
    inverse_scaling_factors = torch.reciprocal(scaling_factors)

    if bias is not None:
        bias = bias * inverse_scaling_factors
```
- **EN**: Key callable entry points in this range include `cross_layer_equalization`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `cross_layer_equalization`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 155-175 / 第 155-175 行
```python
    # formatting the scaling (1D) tensors to be applied on the given argument tensors
    # pads axis to (1D) tensors to then be broadcasted
    size1 = [1] * weight1.ndim
    size1[output_axis] = weight1.size(output_axis)
    size2 = [1] * weight2.ndim
    size2[input_axis] = weight2.size(input_axis)

    scaling_factors = torch.reshape(scaling_factors, size2)
    inverse_scaling_factors = torch.reshape(inverse_scaling_factors, size1)

    weight1 = weight1 * inverse_scaling_factors
    weight2 = weight2 * scaling_factors

    set_module_weight(module1, weight1)
    if bias is not None:
        set_module_bias(module1, bias)
    set_module_weight(module2, weight2)


def process_paired_modules_list_to_name(model, paired_modules_list):
    """Processes a list of paired modules to a list of names of paired modules."""
```
- **EN**: Key callable entry points in this range include `cross_layer_equalization`, `process_paired_modules_list_to_name`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `cross_layer_equalization`, `process_paired_modules_list_to_name`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 177-198 / 第 177-198 行
```python
    for group in paired_modules_list:
        for i, item in enumerate(group):
            if isinstance(item, torch.nn.Module):
                group[i] = get_name_by_module(model, item)
            elif not isinstance(item, str):
                raise TypeError("item must be a nn.Module or a string")
    return paired_modules_list


def expand_groups_in_paired_modules_list(paired_modules_list):
    """Expands module pair groups larger than two into groups of two modules."""
    new_list = []

    for group in paired_modules_list:
        if len(group) == 1:
            raise ValueError("Group must have at least two modules")
        elif len(group) == 2:
            new_list.append(group)
        elif len(group) > 2:
            new_list.extend([group[i], group[i + 1]] for i in range(len(group) - 1))

    return new_list
```
- **EN**: Key callable entry points in this range include `process_paired_modules_list_to_name`, `expand_groups_in_paired_modules_list`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `process_paired_modules_list_to_name`, `expand_groups_in_paired_modules_list`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 201-224 / 第 201-224 行
```python
def equalize(model, paired_modules_list, threshold=1e-4, inplace=True):
    """Equalize modules until convergence is achieved.

    Given a list of adjacent modules within a model, equalization will
    be applied between each pair, this will repeated until convergence is achieved

    Keeps a copy of the changing modules from the previous iteration, if the copies
    are not that different than the current modules (determined by converged_test),
    then the modules have converged enough that further equalizing is not necessary

    Reference is section 4.1 of this paper https://arxiv.org/pdf/1906.04721.pdf

    Args:
        model: a model (nn.Module) that equalization is to be applied on
            paired_modules_list (List(List[nn.module || str])): a list of lists
            where each sublist is a pair of two submodules found in the model,
            for each pair the two modules have to be adjacent in the model,
            with only piece-wise-linear functions like a (P)ReLU or LeakyReLU in between
            to get expected results.
            The list can contain either modules, or names of modules in the model.
            If you pass multiple modules in the same list, they will all be equalized together.
            threshold (float): a number used by the converged function to determine what degree
            of similarity between models is necessary for them to be called equivalent
        inplace (bool): determines if function is inplace or not
```
- **EN**: Key callable entry points in this range include `equalize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `equalize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 225-247 / 第 225-247 行
```python
    """

    paired_modules_list = process_paired_modules_list_to_name(
        model, paired_modules_list
    )

    if not inplace:
        model = copy.deepcopy(model)

    paired_modules_list = expand_groups_in_paired_modules_list(paired_modules_list)

    name_to_module: dict[str, torch.nn.Module] = {}
    previous_name_to_module: dict[str, Any] = {}
    name_set = set(chain.from_iterable(paired_modules_list))

    for name, module in model.named_modules():
        if name in name_set:
            name_to_module[name] = module
            previous_name_to_module[name] = None
    while not converged(name_to_module, previous_name_to_module, threshold):
        for pair in paired_modules_list:
            previous_name_to_module[pair[0]] = copy.deepcopy(name_to_module[pair[0]])
            previous_name_to_module[pair[1]] = copy.deepcopy(name_to_module[pair[1]])
```
- **EN**: Key callable entry points in this range include `equalize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `equalize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 249-268 / 第 249-268 行
```python
            cross_layer_equalization(name_to_module[pair[0]], name_to_module[pair[1]])

    return model


def converged(curr_modules, prev_modules, threshold=1e-4):
    """Test whether modules are converged to a specified threshold.

    Tests for the summed norm of the differences between each set of modules
    being less than the given threshold

    Takes two dictionaries mapping names to modules, the set of names for each dictionary
    should be the same, looping over the set of names, for each name take the difference
    between the associated modules in each dictionary

    """
    if curr_modules.keys() != prev_modules.keys():
        raise ValueError(
            "The keys to the given mappings must have the same set of names of modules"
        )
```
- **EN**: Key callable entry points in this range include `equalize`, `converged`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `equalize`, `converged`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 270-279 / 第 270-279 行
```python
    summed_norms = torch.tensor(0.0)
    if None in prev_modules.values():
        return False
    for name in curr_modules:
        curr_weight = get_module_weight(curr_modules[name])
        prev_weight = get_module_weight(prev_modules[name])

        difference = curr_weight.sub(prev_weight)
        summed_norms += torch.norm(difference)
    return bool(summed_norms < threshold)
```
- **EN**: Key callable entry points in this range include `converged`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `converged`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers.
  - CN: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **set_module_weight**
  - EN: `set_module_weight` is a representative function that exposes or coordinates an important action in this module.
  - CN: `set_module_weight` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
- **set_module_bias**
  - EN: `set_module_bias` is a representative function that exposes or coordinates an important action in this module.
  - CN: `set_module_bias` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
- **Public exports**
  - EN: The module explicitly curates part of its public API through `__all__`.
  - CN: 该模块通过 `__all__` 显式整理了部分公开 API。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`
- **Python standard library / Python 标准库**: `copy`, `itertools:chain`, `typing:Any`
- **Explicit exports / 显式导出**: `set_module_weight`, `set_module_bias`, `has_bias`, `get_module_weight`, `get_module_bias`, `max_over_ndim`, `min_over_ndim`, `channel_range`, `get_name_by_module`, `cross_layer_equalization`, `process_paired_modules_list_to_name`, `expand_groups_in_paired_modules_list`, `equalize`, `converged`
- **Primary symbols / 核心符号**: `set_module_weight`, `set_module_bias`, `has_bias`, `get_module_weight`, `get_module_bias`, `max_over_ndim`, `min_over_ndim`, `channel_range`, `get_name_by_module`, `cross_layer_equalization`, `process_paired_modules_list_to_name`, `expand_groups_in_paired_modules_list`, `equalize`, `converged`
