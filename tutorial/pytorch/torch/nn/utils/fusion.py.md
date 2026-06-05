# fusion.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/utils/fusion.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects helper routines that reshape, inspect, or adapt nn.Module behavior.
- **Purpose (CN)**: 汇集用于重塑、检查或适配 nn.Module 行为的辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
from __future__ import annotations

import copy
from typing import TypeVar

import torch


__all__ = [
    "fuse_conv_bn_eval",
    "fuse_conv_bn_weights",
    "fuse_linear_bn_eval",
    "fuse_linear_bn_weights",
]

ConvT = TypeVar("ConvT", bound="torch.nn.modules.conv._ConvNd")
LinearT = TypeVar("LinearT", bound="torch.nn.Linear")
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 19-34
```python

def fuse_conv_bn_eval(
    conv: ConvT,
    bn: torch.nn.modules.batchnorm._BatchNorm,
    transpose: bool = False,
) -> ConvT:
    r"""Fuse a convolutional module and a BatchNorm module into a single, new convolutional module.

    Args:
        conv (torch.nn.modules.conv._ConvNd): A convolutional module.
        bn (torch.nn.modules.batchnorm._BatchNorm): A BatchNorm module.
        transpose (bool, optional): If True, transpose the convolutional weight. Defaults to False.

    Returns:
        torch.nn.modules.conv._ConvNd: The fused convolutional module.
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 35-46
```python
    .. note::
        Both ``conv`` and ``bn`` must be in eval mode, and ``bn`` must have its running buffers computed.
    """
    if conv.training or bn.training:
        raise AssertionError("Fusion only for eval!")
    fused_conv = copy.deepcopy(conv)

    if bn.running_mean is None or bn.running_var is None:
        raise AssertionError("bn.running_mean and bn.running_var must not be None")
    fused_conv.weight, fused_conv.bias = fuse_conv_bn_weights(
        fused_conv.weight,
        fused_conv.bias,
```
- **EN**: This block continues `fuse_conv_bn_eval` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `fuse_conv_bn_eval`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 47-58
```python
        bn.running_mean,
        bn.running_var,
        bn.eps,
        bn.weight,
        bn.bias,
        transpose,
    )

    return fused_conv


def fuse_conv_bn_weights(
```
- **EN**: This module-level block helps prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果。

### Lines 59-70
```python
    conv_w: torch.Tensor,
    conv_b: torch.Tensor | None,
    bn_rm: torch.Tensor,
    bn_rv: torch.Tensor,
    bn_eps: float,
    bn_w: torch.Tensor | None,
    bn_b: torch.Tensor | None,
    transpose: bool = False,
) -> tuple[torch.nn.Parameter, torch.nn.Parameter]:
    r"""Fuse convolutional module parameters and BatchNorm module parameters into new convolutional module parameters.

    Args:
```
- **EN**: This block continues `fuse_conv_bn_weights` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `fuse_conv_bn_weights`，用于组织可复用的模块行为与状态。

### Lines 71-82
```python
        conv_w (torch.Tensor): Convolutional weight.
        conv_b (Optional[torch.Tensor]): Convolutional bias.
        bn_rm (torch.Tensor): BatchNorm running mean.
        bn_rv (torch.Tensor): BatchNorm running variance.
        bn_eps (float): BatchNorm epsilon.
        bn_w (Optional[torch.Tensor]): BatchNorm weight.
        bn_b (Optional[torch.Tensor]): BatchNorm bias.
        transpose (bool, optional): If True, transpose the conv weight. Defaults to False.

    Returns:
        Tuple[torch.nn.Parameter, torch.nn.Parameter]: Fused convolutional weight and bias.
    """
```
- **EN**: This block continues `fuse_conv_bn_weights` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `fuse_conv_bn_weights`，用于组织可复用的模块行为与状态。

### Lines 83-97
```python
    conv_weight_dtype = conv_w.dtype
    conv_bias_dtype = conv_b.dtype if conv_b is not None else conv_weight_dtype
    if conv_b is None:
        conv_b = torch.zeros_like(bn_rm)
    if bn_w is None:
        bn_w = torch.ones_like(bn_rm)
    if bn_b is None:
        bn_b = torch.zeros_like(bn_rm)
    bn_var_rsqrt = torch.rsqrt(bn_rv + bn_eps)

    if transpose:
        shape = [1, -1] + [1] * (len(conv_w.shape) - 2)
    else:
        shape = [-1, 1] + [1] * (len(conv_w.shape) - 2)
```
- **EN**: This block continues `fuse_conv_bn_weights` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `fuse_conv_bn_weights`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 98-110
```python
    fused_conv_w = (conv_w * (bn_w * bn_var_rsqrt).reshape(shape)).to(
        dtype=conv_weight_dtype
    )
    fused_conv_b = ((conv_b - bn_rm) * bn_var_rsqrt * bn_w + bn_b).to(
        dtype=conv_bias_dtype
    )

    return (
        torch.nn.Parameter(fused_conv_w, conv_w.requires_grad),
        torch.nn.Parameter(fused_conv_b, conv_b.requires_grad),
    )
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 111-123
```python
def fuse_linear_bn_eval(
    linear: LinearT,
    bn: torch.nn.modules.batchnorm._BatchNorm,
) -> LinearT:
    r"""Fuse a linear module and a BatchNorm module into a single, new linear module.

    Args:
        linear (torch.nn.Linear): A Linear module.
        bn (torch.nn.modules.batchnorm._BatchNorm): A BatchNorm module.

    Returns:
        torch.nn.Linear: The fused linear module.
```
- **EN**: Defines the `fuse_linear_bn_eval` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`fuse_linear_bn_eval` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 124-135
```python
    .. note::
        Both ``linear`` and ``bn`` must be in eval mode, and ``bn`` must have its running buffers computed.
    """
    if linear.training or bn.training:
        raise AssertionError("Fusion only for eval!")
    fused_linear = copy.deepcopy(linear)

    """
    Linear-BN needs to be fused while preserving the shapes of linear weight/bias.
    To preserve the shapes of linear weight/bias, the channel dim of bn needs to be broadcastable with the last dim of linear,
    because bn operates over the channel dim, (N, C_in, H, W) while linear operates over the last dim, (*, H_in).
    To be broadcastable, the number of features in bn and
```
- **EN**: This block continues `fuse_linear_bn_eval` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `fuse_linear_bn_eval`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 136-147
```python
    the number of output features from linear must satisfy the following condition:
    1. they are equal, or
    2. the number of features in bn is 1
    Otherwise, skip the folding path
    """
    if linear.out_features != bn.num_features and bn.num_features != 1:
        raise AssertionError(
            f"To fuse, linear.out_features == bn.num_features or bn.num_features == 1, "
            f"got linear.out_features={linear.out_features} and bn.num_features={bn.num_features}"
        )

    if bn.running_mean is None or bn.running_var is None:
```
- **EN**: This block continues `fuse_linear_bn_eval` and works to validate invariants and surface meaningful failures. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `fuse_linear_bn_eval`，用于校验不变量并给出有意义的失败信息。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 148-161
```python
        raise AssertionError("bn.running_mean and bn.running_var must not be None")
    fused_linear.weight, fused_linear.bias = fuse_linear_bn_weights(
        fused_linear.weight,
        fused_linear.bias,
        bn.running_mean,
        bn.running_var,
        bn.eps,
        bn.weight,
        bn.bias,
    )

    return fused_linear
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设。

### Lines 162-173
```python
def fuse_linear_bn_weights(
    linear_w: torch.Tensor,
    linear_b: torch.Tensor | None,
    bn_rm: torch.Tensor,
    bn_rv: torch.Tensor,
    bn_eps: float,
    bn_w: torch.Tensor,
    bn_b: torch.Tensor,
) -> tuple[torch.nn.Parameter, torch.nn.Parameter]:
    r"""Fuse linear module parameters and BatchNorm module parameters into new linear module parameters.

    Args:
```
- **EN**: Defines the `fuse_linear_bn_weights` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`fuse_linear_bn_weights` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 174-190
```python
        linear_w (torch.Tensor): Linear weight.
        linear_b (Optional[torch.Tensor]): Linear bias.
        bn_rm (torch.Tensor): BatchNorm running mean.
        bn_rv (torch.Tensor): BatchNorm running variance.
        bn_eps (float): BatchNorm epsilon.
        bn_w (torch.Tensor): BatchNorm weight.
        bn_b (torch.Tensor): BatchNorm bias.

    Returns:
        Tuple[torch.nn.Parameter, torch.nn.Parameter]: Fused linear weight and bias.
    """
    linear_weight_dtype = linear_w.dtype
    linear_bias_dtype = linear_b.dtype if linear_b is not None else linear_weight_dtype
    if linear_b is None:
        linear_b = torch.zeros_like(bn_rm)
    bn_scale = bn_w * torch.rsqrt(bn_rv + bn_eps)
```
- **EN**: This block continues `fuse_linear_bn_weights` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `fuse_linear_bn_weights`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 191-196
```python
    fused_w = linear_w * bn_scale.unsqueeze(-1).to(dtype=linear_weight_dtype)
    fused_b = ((linear_b - bn_rm) * bn_scale + bn_b).to(dtype=linear_bias_dtype)

    return torch.nn.Parameter(fused_w, linear_w.requires_grad), torch.nn.Parameter(
        fused_b, linear_b.requires_grad
    )
```
- **EN**: This block continues `fuse_linear_bn_weights` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `fuse_linear_bn_weights`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`
- **Standard library / 标准库**: `__future__`, `copy`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `fuse_conv_bn_eval`, `fuse_conv_bn_weights`, `fuse_linear_bn_eval`, `fuse_linear_bn_weights`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
