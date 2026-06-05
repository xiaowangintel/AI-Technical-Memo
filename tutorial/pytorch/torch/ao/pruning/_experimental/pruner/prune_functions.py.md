# prune_functions.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/pruning/_experimental/pruner/prune_functions.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure. This specific file centers on `prune_functions.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。 该文件具体围绕 `prune_functions.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
```python
# mypy: allow-untyped-defs
"""
Collection of conversion functions for linear / conv2d structured pruning
Also contains utilities for bias propagation
"""

from collections.abc import Callable
from typing import cast

import torch
from torch import nn, Tensor
from torch.nn.utils import parametrize
from torch.nn.utils.parametrize import ParametrizationList

from .parametrization import BiasHook, FakeStructuredSparsity
```
- **EN**: The opening docstring frames the module contract, telling readers what subsystem responsibilities the rest of the file implements. This block establishes the module dependencies, pulling in internal torch modules such as torch, torch:nn, torch:Tensor, torch.nn.utils:parametrize; standard-library helpers such as collections.abc:Callable, typing:cast. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Hook registration/order is important because the value comes from when callbacks fire, not just what they compute.
- **CN**: 开头的模块文档字符串给出了整体契约，说明后续代码要实现的子系统职责。 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch:nn, torch:Tensor, torch.nn.utils:parametrize；标准库辅助模块，如 collections.abc:Callable, typing:cast。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 钩子的价值不仅在于计算什么，更在于何时触发以及触发顺序。

### Lines 18-41 / 第 18-41 行
```python
# BIAS PROPAGATION
def _remove_bias_handles(module: nn.Module) -> None:
    if hasattr(module, "_forward_hooks"):
        bias_hooks: list[int] = []
        for key, hook in module._forward_hooks.items():
            if isinstance(hook, BiasHook):
                bias_hooks.append(key)

        for key in bias_hooks:
            del module._forward_hooks[key]


def _get_adjusted_next_layer_bias(
    next_layer: nn.Module, pruned_biases: Tensor, mask: Tensor
) -> nn.Parameter:
    r"""Returns new adjusted bias for the second supported module"""
    if parametrize.is_parametrized(next_layer):
        # need to access original weight
        parametrization_dict = cast(nn.ModuleDict, next_layer.parametrizations)
        weight_parameterizations = cast(
            ParametrizationList, parametrization_dict.weight
        )
        next_weight = weight_parameterizations.original
    else:
```
- **EN**: Key callable entry points in this range include `_remove_bias_handles`, `_get_adjusted_next_layer_bias`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Hook registration/order is important because the value comes from when callbacks fire, not just what they compute.
- **CN**: 这一段的重要可调用入口包括 `_remove_bias_handles`, `_get_adjusted_next_layer_bias`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 钩子的价值不仅在于计算什么，更在于何时触发以及触发顺序。

### Lines 42-64 / 第 42-64 行
```python
        next_weight = cast(Tensor, next_layer.weight)

    scaling_weight = next_weight[:, ~mask]
    if isinstance(next_layer, nn.Conv2d):  # checking for Conv2d
        # Propagating first layer pruned biases and calculating the new second layer bias
        # involves more steps since the Conv2d scaling weight has extra dimensions,
        # so adding bias involves broadcasting, logically:
        # for each channel k in range(oC):
        #     scaled_biases = sum(first_bias[pruned_idx] @ next_weight[k, pruned_idx, :, :].T)
        #     new_next_bias[k] = old_next_bias[k] + scaled_biases
        scaling_product = torch.matmul(
            pruned_biases.reshape(1, -1), torch.transpose(scaling_weight, 1, 2)
        )
        sum_range = list(range(len(scaling_product.shape)))[
            1:
        ]  # all but the first dimension
        scaled_biases = torch.sum(scaling_product, sum_range)
    elif isinstance(next_layer, nn.Linear):  # Linear
        scaled_biases = torch.matmul(
            pruned_biases, torch.transpose(scaling_weight, 0, 1)
        )  # recall b2_new = b1 @ w2.T + b2
    else:
        raise NotImplementedError(f"Type {type(next_layer)} not supported yet.")
```
- **EN**: Key callable entry points in this range include `_get_adjusted_next_layer_bias`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `_get_adjusted_next_layer_bias`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 66-85 / 第 66-85 行
```python
    if (
        parametrize.is_parametrized(next_layer)
        and getattr(next_layer, "_bias", None) is not None
    ):  # next_layer is parametrized & has original bias ._bias
        adjusted_bias = nn.Parameter(scaled_biases + next_layer._bias)  # type: ignore[operator]
    elif (
        not parametrize.is_parametrized(next_layer) and next_layer.bias is not None
    ):  # next_layer not parametrized & has .bias
        adjusted_bias = nn.Parameter(scaled_biases + next_layer.bias)  # type: ignore[operator]
    else:  # next_layer has no bias
        adjusted_bias = nn.Parameter(scaled_biases)
    return adjusted_bias


def _prune_module_bias(module: nn.Module, mask: Tensor) -> None:
    r"""Applies mask to given modules bias"""
    # prune bias along with weights, discard pruned indices of bias
    original_bias = cast(Tensor, getattr(module, "_bias", module.bias))
    if original_bias is not None:
        module.bias = nn.Parameter(original_bias[mask])
```
- **EN**: Key callable entry points in this range include `_get_adjusted_next_layer_bias`, `_prune_module_bias`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_get_adjusted_next_layer_bias`, `_prune_module_bias`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 87-107 / 第 87-107 行
```python
    #  remove _bias parameter
    if hasattr(module, "_bias"):
        delattr(module, "_bias")


def _propagate_module_bias(module: nn.Module, mask: Tensor) -> Tensor | None:
    r"""
    In the case that we need to propagate biases, this function will return the biases we need
    """
    # set current module bias
    if module.bias is not None:
        module.bias = nn.Parameter(cast(Tensor, module.bias)[mask])
    elif getattr(module, "_bias", None) is not None:
        # pyrefly: ignore [bad-assignment]
        module.bias = nn.Parameter(cast(Tensor, module._bias)[mask])

    # get pruned biases to propagate to subsequent layer
    if getattr(module, "_bias", None) is not None:
        pruned_biases = cast(Tensor, module._bias)[~mask]
    else:
        pruned_biases = None
```
- **EN**: Key callable entry points in this range include `_prune_module_bias`, `_propagate_module_bias`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_prune_module_bias`, `_propagate_module_bias`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 109-131 / 第 109-131 行
```python
    if hasattr(module, "_bias"):
        delattr(module, "_bias")

    return pruned_biases


# LINEAR
def _prune_linear_helper(linear: nn.Linear) -> Tensor:
    # expects linear to be a parameterized linear module
    parametrization_dict = cast(nn.ModuleDict, linear.parametrizations)
    weight_parameterizations = cast(ParametrizationList, parametrization_dict.weight)
    for p in weight_parameterizations:
        if isinstance(p, FakeStructuredSparsity):
            mask = cast(Tensor, p.mask)

    with torch.no_grad():
        parametrize.remove_parametrizations(linear, "weight", leave_parametrized=True)
        linear.weight = nn.Parameter(linear.weight[mask])  # type: ignore[possibly-undefined]
    linear.out_features = linear.weight.shape[0]
    _remove_bias_handles(linear)

    # pyrefly: ignore [unbound-name]
    return mask
```
- **EN**: Key callable entry points in this range include `_propagate_module_bias`, `_prune_linear_helper`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_propagate_module_bias`, `_prune_linear_helper`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 134-157 / 第 134-157 行
```python
def prune_linear(linear: nn.Linear) -> None:
    mask = _prune_linear_helper(linear)
    if getattr(linear, "prune_bias", False):
        _prune_module_bias(linear, mask)


def prune_linear_linear(linear1: nn.Linear, linear2: nn.Linear) -> None:
    prune_linear_activation_linear(linear1, None, linear2)


def prune_linear_activation_linear(
    linear1: nn.Linear,
    activation: Callable[[Tensor], Tensor] | None,
    linear2: nn.Linear,
):
    mask = _prune_linear_helper(linear1)
    if getattr(linear1, "prune_bias", False):
        _prune_module_bias(linear1, mask)
    else:
        pruned_biases = _propagate_module_bias(linear1, mask)
        if pruned_biases is not None:
            if activation:
                pruned_biases = activation(pruned_biases)
            linear2.bias = _get_adjusted_next_layer_bias(linear2, pruned_biases, mask)
```
- **EN**: Key callable entry points in this range include `prune_linear`, `prune_linear_linear`, `prune_linear_activation_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 这一段的重要可调用入口包括 `prune_linear`, `prune_linear_linear`, `prune_linear_activation_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

### Lines 159-181 / 第 159-181 行
```python
    with torch.no_grad():
        if parametrize.is_parametrized(linear2):
            parametrization_dict = cast(nn.ModuleDict, linear2.parametrizations)
            weight_parameterizations = cast(
                ParametrizationList, parametrization_dict.weight
            )

            weight_parameterizations.original = nn.Parameter(
                weight_parameterizations.original[:, mask]
            )
            linear2.in_features = weight_parameterizations.original.shape[1]
        else:
            linear2.weight = nn.Parameter(linear2.weight[:, mask])
            linear2.in_features = linear2.weight.shape[1]


# CONV2D
def _prune_conv2d_helper(conv2d: nn.Conv2d) -> Tensor:
    parametrization_dict = cast(nn.ModuleDict, conv2d.parametrizations)
    weight_parameterizations = cast(ParametrizationList, parametrization_dict.weight)
    for p in weight_parameterizations:
        if isinstance(p, FakeStructuredSparsity):
            mask = cast(Tensor, p.mask)
```
- **EN**: Key callable entry points in this range include `prune_linear_activation_linear`, `_prune_conv2d_helper`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit.
- **CN**: 这一段的重要可调用入口包括 `prune_linear_activation_linear`, `_prune_conv2d_helper`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。

### Lines 183-201 / 第 183-201 行
```python
    with torch.no_grad():
        parametrize.remove_parametrizations(conv2d, "weight", leave_parametrized=True)
        conv2d.weight = nn.Parameter(conv2d.weight[mask])  # type: ignore[possibly-undefined]
    conv2d.out_channels = conv2d.weight.shape[0]

    _remove_bias_handles(conv2d)
    # pyrefly: ignore [unbound-name]
    return mask


def prune_conv2d_padded(conv2d_1: nn.Conv2d) -> None:
    parametrization_dict = cast(nn.ModuleDict, conv2d_1.parametrizations)
    weight_parameterizations = cast(ParametrizationList, parametrization_dict.weight)
    for p in weight_parameterizations:
        if isinstance(p, FakeStructuredSparsity):
            mask = cast(Tensor, p.mask)

    with torch.no_grad():
        parametrize.remove_parametrizations(conv2d_1, "weight", leave_parametrized=True)
```
- **EN**: Key callable entry points in this range include `_prune_conv2d_helper`, `prune_conv2d_padded`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_prune_conv2d_helper`, `prune_conv2d_padded`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 203-224 / 第 203-224 行
```python
    if getattr(conv2d_1, "_bias", None) is not None:
        if (
            conv2d_1.bias is not None
        ):  # conv2d_1 has original bias and bias propagated from previous layer
            new_bias = torch.zeros(conv2d_1.bias.shape)
            new_bias[mask] = conv2d_1.bias[mask]  # type: ignore[possibly-undefined]
            # adjusted bias that to keep in conv2d_1
            # pyrefly: ignore [unbound-name]
            new_bias[~mask] = cast(Tensor, conv2d_1._bias)[~mask]
            # pruned biases that are kept instead of propagated
            conv2d_1.bias = nn.Parameter(new_bias)
        else:  # conv2d_1 has only original bias
            conv2d_1.bias = nn.Parameter(cast(Tensor, conv2d_1._bias))
    else:
        # no original bias, only propagated bias
        if (
            conv2d_1.bias is not None
        ):  # conv2d_1 has bias propagated from previous layer
            conv2d_1.bias.data[~mask] = 0  # type: ignore[possibly-undefined]

    if hasattr(conv2d_1, "_bias"):
        delattr(conv2d_1, "_bias")
```
- **EN**: Key callable entry points in this range include `prune_conv2d_padded`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 这一段的重要可调用入口包括 `prune_conv2d_padded`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

### Lines 227-249 / 第 227-249 行
```python
def prune_conv2d(conv2d: nn.Conv2d) -> None:
    mask = _prune_conv2d_helper(conv2d)
    if getattr(conv2d, "prune_bias", False):
        _prune_module_bias(conv2d, mask)


def prune_conv2d_conv2d(conv2d_1: nn.Conv2d, conv2d_2: nn.Conv2d) -> None:
    prune_conv2d_activation_conv2d(conv2d_1, None, conv2d_2)


def prune_conv2d_activation_conv2d(
    conv2d_1: nn.Conv2d,
    activation: Callable[[Tensor], Tensor] | None,
    conv2d_2: nn.Conv2d,
):
    r"""
    Fusion Pattern for conv2d -> some activation module / function -> conv2d layers
    """
    parametrization_dict = cast(nn.ModuleDict, conv2d_1.parametrizations)
    weight_parameterizations = cast(ParametrizationList, parametrization_dict.weight)
    for p in weight_parameterizations:
        if isinstance(p, FakeStructuredSparsity):
            mask = cast(Tensor, p.mask)
```
- **EN**: Key callable entry points in this range include `prune_conv2d`, `prune_conv2d_conv2d`, `prune_conv2d_activation_conv2d`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 这一段的重要可调用入口包括 `prune_conv2d`, `prune_conv2d_conv2d`, `prune_conv2d_activation_conv2d`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 251-269 / 第 251-269 行
```python
    prune_bias = getattr(conv2d_1, "prune_bias", False)
    if (
        hasattr(conv2d_2, "padding")
        and cast(tuple[int], conv2d_2.padding) > (0, 0)
        and (conv2d_1.bias is not None or getattr(conv2d_1, "_bias", None) is not None)
    ):
        prune_conv2d_padded(conv2d_1)
    else:
        mask = _prune_conv2d_helper(conv2d_1)
        if prune_bias:
            _prune_module_bias(conv2d_1, mask)
        else:
            pruned_biases = _propagate_module_bias(conv2d_1, mask)
            if pruned_biases is not None:
                if activation:
                    pruned_biases = activation(pruned_biases)
                conv2d_2.bias = _get_adjusted_next_layer_bias(
                    conv2d_2, pruned_biases, mask
                )
```
- **EN**: Key callable entry points in this range include `prune_conv2d_activation_conv2d`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 这一段的重要可调用入口包括 `prune_conv2d_activation_conv2d`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

### Lines 271-292 / 第 271-292 行
```python
        if (
            not (
                hasattr(conv2d_2, "padding")
                and cast(tuple[int], conv2d_2.padding) > (0, 0)
            )
            or conv2d_1.bias is None
        ):
            with torch.no_grad():
                if parametrize.is_parametrized(conv2d_2):
                    parametrization_dict = cast(
                        nn.ModuleDict, conv2d_2.parametrizations
                    )
                    weight_parameterizations = cast(
                        ParametrizationList, parametrization_dict.weight
                    )
                    weight_parameterizations.original = nn.Parameter(
                        weight_parameterizations.original[:, mask]
                    )
                    conv2d_2.in_channels = weight_parameterizations.original.shape[1]
                else:
                    conv2d_2.weight = nn.Parameter(conv2d_2.weight[:, mask])
                    conv2d_2.in_channels = conv2d_2.weight.shape[1]
```
- **EN**: Key callable entry points in this range include `prune_conv2d_activation_conv2d`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit.
- **CN**: 这一段的重要可调用入口包括 `prune_conv2d_activation_conv2d`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。

### Lines 295-310 / 第 295-310 行
```python
def prune_conv2d_pool_activation_conv2d(
    c1: nn.Conv2d,
    pool: nn.Module,
    activation: Callable[[Tensor], Tensor] | None,
    c2: nn.Conv2d,
) -> None:
    prune_conv2d_activation_conv2d(c1, activation, c2)


def prune_conv2d_activation_pool_conv2d(
    c1: nn.Conv2d,
    activation: Callable[[Tensor], Tensor] | None,
    pool: nn.Module,
    c2: nn.Conv2d,
) -> None:
    prune_conv2d_activation_conv2d(c1, activation, c2)
```
- **EN**: Key callable entry points in this range include `prune_conv2d_pool_activation_conv2d`, `prune_conv2d_activation_pool_conv2d`. They package a focused unit of behavior behind named helpers or APIs.
- **CN**: 这一段的重要可调用入口包括 `prune_conv2d_pool_activation_conv2d`, `prune_conv2d_activation_pool_conv2d`，它们把聚焦的行为封装成具名辅助函数或 API。

### Lines 313-332 / 第 313-332 行
```python
def prune_conv2d_pool_flatten_linear(
    conv2d: nn.Conv2d,
    pool: nn.Module,
    flatten: Callable[[Tensor], Tensor] | None,
    linear: nn.Linear,
) -> None:
    mask = _prune_conv2d_helper(conv2d)

    # We map the pruned indices of the Conv2d output to the flattened indices of the Linear following the Flatten layer.
    # we determine the flattening scale (h * w), and readjust `first_pruned_indices`
    # (each idx maps to range idx * h * w to (idx+1) * h * w), `first_valid_indices`,
    # and `pruned_biases` (repeat each bias by h * w).
    if parametrize.is_parametrized(linear):
        parametrization_dict = cast(nn.ModuleDict, linear.parametrizations)
        weight_parameterizations = cast(
            ParametrizationList, parametrization_dict.weight
        )
        linear_ic = weight_parameterizations.original.shape[1]
    else:
        linear_ic = linear.weight.shape[1]
```
- **EN**: Key callable entry points in this range include `prune_conv2d_pool_flatten_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 这一段的重要可调用入口包括 `prune_conv2d_pool_flatten_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

### Lines 334-354 / 第 334-354 行
```python
    conv2d_oc = len(mask)
    if linear_ic % conv2d_oc != 0:
        raise AssertionError(
            f"Flattening from dimensions {conv2d_oc} to {linear_ic} not supported"
        )

    flatten_scale = linear_ic // conv2d_oc
    flattened_mask = torch.tensor(
        [[val] * flatten_scale for val in mask], dtype=torch.bool, device=mask.device
    ).flatten()

    if getattr(conv2d, "prune_bias", False):
        _prune_module_bias(conv2d, mask)
    else:
        pruned_biases = cast(Tensor, _propagate_module_bias(conv2d, mask))
        flattened_pruned_biases = torch.tensor(
            [[bias] * flatten_scale for bias in pruned_biases], device=mask.device
        ).flatten()
        linear.bias = _get_adjusted_next_layer_bias(
            linear, flattened_pruned_biases, flattened_mask
        )
```
- **EN**: Key callable entry points in this range include `prune_conv2d_pool_flatten_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `prune_conv2d_pool_flatten_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 356-374 / 第 356-374 行
```python
    with torch.no_grad():
        if parametrize.is_parametrized(linear):
            parametrization_dict = cast(nn.ModuleDict, linear.parametrizations)
            weight_parameterizations = cast(
                ParametrizationList, parametrization_dict.weight
            )
            weight_parameterizations.original = nn.Parameter(
                weight_parameterizations.original[:, flattened_mask]
            )
            linear.in_features = weight_parameterizations.original.shape[1]
        else:
            linear.weight = nn.Parameter(linear.weight[:, flattened_mask])
            linear.in_features = linear.weight.shape[1]


def prune_lstm_output_linear(
    lstm: nn.LSTM, getitem: Callable, linear: nn.Linear
) -> None:
    prune_lstm_output_layernorm_linear(lstm, getitem, None, linear)
```
- **EN**: Key callable entry points in this range include `prune_conv2d_pool_flatten_linear`, `prune_lstm_output_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit.
- **CN**: 这一段的重要可调用入口包括 `prune_conv2d_pool_flatten_linear`, `prune_lstm_output_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。

### Lines 377-400 / 第 377-400 行
```python
def prune_lstm_output_layernorm_linear(
    lstm: nn.LSTM,
    getitem: Callable,
    layernorm: nn.LayerNorm | None,
    linear: nn.Linear,
) -> None:
    for i in range(lstm.num_layers):
        if parametrize.is_parametrized(lstm, f"weight_ih_l{i}"):
            parametrization_dict = cast(nn.ModuleDict, lstm.parametrizations)
            weight_parameterizations = cast(
                ParametrizationList, parametrization_dict[f"weight_ih_l{i}"]
            )
            mask = weight_parameterizations[0].mask

            with torch.no_grad():
                parametrize.remove_parametrizations(
                    lstm, f"weight_ih_l{i}", leave_parametrized=True
                )
                setattr(
                    lstm,
                    f"weight_ih_l{i}",
                    nn.Parameter(getattr(lstm, f"weight_ih_l{i}")[mask]),
                )
                setattr(
```
- **EN**: Key callable entry points in this range include `prune_lstm_output_layernorm_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit.
- **CN**: 这一段的重要可调用入口包括 `prune_lstm_output_layernorm_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。

### Lines 401-421 / 第 401-421 行
```python
                    lstm,
                    f"bias_ih_l{i}",
                    nn.Parameter(getattr(lstm, f"bias_ih_l{i}")[mask]),
                )

        if parametrize.is_parametrized(lstm, f"weight_hh_l{i}"):
            parametrization_dict = cast(nn.ModuleDict, lstm.parametrizations)
            weight_parameterizations = cast(
                ParametrizationList, parametrization_dict[f"weight_hh_l{i}"]
            )
            mask = weight_parameterizations[0].mask

            with torch.no_grad():
                parametrize.remove_parametrizations(
                    lstm, f"weight_hh_l{i}", leave_parametrized=True
                )
                # splitting out hidden-hidden masks
                W_hi, W_hf, W_hg, W_ho = torch.split(
                    getattr(lstm, f"weight_hh_l{i}"), lstm.hidden_size
                )
                M_hi, M_hf, M_hg, M_ho = torch.split(mask, lstm.hidden_size)  # type: ignore[arg-type]
```
- **EN**: Key callable entry points in this range include `prune_lstm_output_layernorm_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit.
- **CN**: 这一段的重要可调用入口包括 `prune_lstm_output_layernorm_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。

### Lines 423-446 / 第 423-446 行
```python
                # resize each individual weight separately
                W_hi = W_hi[M_hi][:, M_hi]
                W_hf = W_hf[M_hf][:, M_hf]
                W_hg = W_hg[M_hg][:, M_hg]
                W_ho = W_ho[M_ho][:, M_ho]

                # concat, use this as new weight
                new_weight = torch.cat((W_hi, W_hf, W_hg, W_ho))
                setattr(lstm, f"weight_hh_l{i}", nn.Parameter(new_weight))
                setattr(
                    lstm,
                    f"bias_hh_l{i}",
                    nn.Parameter(getattr(lstm, f"bias_hh_l{i}")[mask]),
                )

            # If this is the final layer, then we need to prune linear layer columns
            if i + 1 == lstm.num_layers:
                lstm.hidden_size = int(M_hi.sum())
                with torch.no_grad():
                    if parametrize.is_parametrized(linear):
                        parametrization_dict = cast(
                            nn.ModuleDict, linear.parametrizations
                        )
                        weight_parameterizations = cast(
```
- **EN**: Key callable entry points in this range include `prune_lstm_output_layernorm_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit.
- **CN**: 这一段的重要可调用入口包括 `prune_lstm_output_layernorm_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。

### Lines 447-462 / 第 447-462 行
```python
                            ParametrizationList, parametrization_dict.weight
                        )

                        weight_parameterizations.original = nn.Parameter(
                            weight_parameterizations.original[:, M_ho]
                        )
                        linear.in_features = weight_parameterizations.original.shape[1]
                    else:
                        linear.weight = nn.Parameter(linear.weight[:, M_ho])
                        linear.in_features = linear.weight.shape[1]

                    # if layernorm module, prune weight and bias
                    if layernorm is not None:
                        layernorm.normalized_shape = (linear.in_features,)
                        layernorm.weight = nn.Parameter(layernorm.weight[M_ho])
                        layernorm.bias = nn.Parameter(layernorm.bias[M_ho])
```
- **EN**: Key callable entry points in this range include `prune_lstm_output_layernorm_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 这一段的重要可调用入口包括 `prune_lstm_output_layernorm_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

### Lines 464-485 / 第 464-485 行
```python
            # otherwise need to prune the columns of the input of the next LSTM layer
            else:
                with torch.no_grad():
                    if parametrize.is_parametrized(lstm, f"weight_ih_l{i + 1}"):
                        parametrization_dict = cast(
                            nn.ModuleDict, lstm.parametrizations
                        )
                        weight_parameterizations = cast(
                            ParametrizationList,
                            getattr(parametrization_dict, f"weight_ih_l{i + 1}"),
                        )

                        weight_parameterizations.original = nn.Parameter(
                            weight_parameterizations.original[:, M_ho]
                        )
                    else:
                        next_layer_weight = getattr(lstm, f"weight_ih_l{i + 1}")
                        setattr(
                            lstm,
                            f"weight_ih_l{i + 1}",
                            nn.Parameter(next_layer_weight[:, M_ho]),
                        )
```
- **EN**: Key callable entry points in this range include `prune_lstm_output_layernorm_linear`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit.
- **CN**: 这一段的重要可调用入口包括 `prune_lstm_output_layernorm_linear`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure.
  - CN: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
- **Hooks and callbacks**
  - EN: Coordinates callback registration and ordered execution around framework events.
  - CN: 协调回调注册以及围绕框架事件的有序执行。
- **_remove_bias_handles**
  - EN: `_remove_bias_handles` is a representative function that exposes or coordinates an important action in this module.
  - CN: `_remove_bias_handles` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
- **_get_adjusted_next_layer_bias**
  - EN: `_get_adjusted_next_layer_bias` is a representative function that exposes or coordinates an important action in this module.
  - CN: `_get_adjusted_next_layer_bias` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch:nn`, `torch:Tensor`, `torch.nn.utils:parametrize`, `torch.nn.utils.parametrize:ParametrizationList`, `.parametrization:BiasHook`, `.parametrization:FakeStructuredSparsity`
- **Python standard library / Python 标准库**: `collections.abc:Callable`, `typing:cast`
- **Primary symbols / 核心符号**: `_remove_bias_handles`, `_get_adjusted_next_layer_bias`, `_prune_module_bias`, `_propagate_module_bias`, `_prune_linear_helper`, `prune_linear`, `prune_linear_linear`, `prune_linear_activation_linear`, `_prune_conv2d_helper`, `prune_conv2d_padded`, `prune_conv2d`, `prune_conv2d_conv2d`, `prune_conv2d_activation_conv2d`, `prune_conv2d_pool_activation_conv2d`, `prune_conv2d_activation_pool_conv2d`
