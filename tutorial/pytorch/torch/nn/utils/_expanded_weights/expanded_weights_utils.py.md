# expanded_weights_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/utils/_expanded_weights/expanded_weights_utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects helper routines that reshape, inspect, or adapt nn.Module behavior.
- **Purpose (CN)**: 汇集用于重塑、检查或适配 nn.Module 行为的辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```python
# mypy: allow-untyped-defs

import torch

from .expanded_weights_impl import ExpandedWeight


def is_batch_first(expanded_args_and_kwargs):
    batch_first = None
    # pyrefly: ignore [bad-assignment]
    for arg in expanded_args_and_kwargs:
        if not isinstance(arg, ExpandedWeight):
            continue
```
- **EN**: This module-level block helps prepare neural-network operators or module behavior. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于准备神经网络算子或模块行为。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 15-26
```python
        if not batch_first:
            batch_first = arg.batch_first
        elif arg.batch_first != batch_first:
            raise RuntimeError(
                "Got conflicting batch_first arguments in the same layer"
            )
    return batch_first


def standard_kwargs(kwarg_names, expanded_args):
    r"""Separate args and kwargs from `__torch_function__`s that standardize kwargs.
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 27-43
```python
    Most `__torch_function__`s standardize the kwargs that they give, so this will separate
    the args and kwargs they pass. Functions that don't are linear and convAND.
    """
    kwarg_values = expanded_args[len(expanded_args) - len(kwarg_names) :]
    expanded_args_without_kwargs = expanded_args[
        : len(expanded_args) - len(kwarg_names)
    ]
    expanded_kwargs = dict(zip(kwarg_names, kwarg_values, strict=True))
    return expanded_args_without_kwargs, expanded_kwargs


def forward_helper(func, expanded_args, expanded_kwargs):
    r"""Compute the forward pass for a function that has expanded weight(s) passed to it.

    It will run the forward pass where all ExpandedWeights are their original
    weight. It runs checks on the given arguments and detaches the outputs.
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 44-61
```python
    .. note:: First argument in :attr:`expanded_args` must be the input with the batch
    dimension as the first element of the shape

    .. note:: :attr:`func` must return a Tensor or tuple of Tensors

    Args:
        func: The function to be called
        expanded_args: Arguments to be passed to :attr:`func`. Will include arguments
          that need to be unpacked because they are ExpandedWeights
        expanded_kwargs: Keyword arguments to be passed to :attr:`func`.
          Similar to :attr:`expanded_args`.
    """
    unexpanded_args, unexpanded_kwargs = _check_and_unexpand_args(
        func, expanded_args, expanded_kwargs
    )
    return func(*unexpanded_args, **unexpanded_kwargs)
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 62-73
```python
def _check_and_unexpand_args(func, expanded_args, expanded_kwargs):
    # input must be the first argument passed
    input = expanded_args[0]
    if isinstance(input, ExpandedWeight):
        raise RuntimeError(
            "Expanded Weights do not support inputs that are also ExpandedWeights. "
            f"Input must be a Tensor, got {type(input).__name__} in function {func.__name__}"
        )
    if not isinstance(input, torch.Tensor):
        raise RuntimeError(
            "Expanded Weights requires a Tensor as the first input to get the batch dimension, "
            f"got {type(input).__name__} in function {func.__name__}"
```
- **EN**: Defines the `_check_and_unexpand_args` function; this block introduces logic that validate invariants and surface meaningful failures.
- **CN**: 定义`_check_and_unexpand_args` 函数；该代码块引入了用于校验不变量并给出有意义的失败信息的逻辑。

### Lines 74-85
```python
        )
    if len(input.shape) == 0:
        raise RuntimeError(
            f"Expanded Weights requires a batch dimension but got an input of size 0 in function {func.__name__}"
        )
    if input.shape[0] == 0:
        raise RuntimeError(
            "0 is not a valid batch size for Expanded Weights but got input tensor of "
            f"{input} in function {func.__name__}"
        )
    for arg in expanded_args + tuple(expanded_kwargs.values()):
        if not isinstance(arg, ExpandedWeight):
```
- **EN**: This block continues `_check_and_unexpand_args` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_check_and_unexpand_args`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 86-97
```python
            continue
        batch_size = input.shape[0] if arg.batch_first else input.shape[1]
        if (arg.allow_smaller_batches and batch_size > arg.batch_size) or (
            not arg.allow_smaller_batches and arg.batch_size != batch_size
        ):
            raise RuntimeError(
                "Expected ExpandedWeights to have batch size matching input but got "
                f"input batch size of {batch_size} with ExpandedWeight of batch size {arg.batch_size}"
            )

    loss_reduction: str | None = None
    for arg in expanded_args + tuple(expanded_kwargs.values()):
```
- **EN**: This block continues `_check_and_unexpand_args` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_check_and_unexpand_args`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 98-109
```python
        if isinstance(arg, ExpandedWeight):
            if loss_reduction is None:
                loss_reduction = arg.loss_reduction
            elif loss_reduction != arg.loss_reduction:
                raise RuntimeError(
                    "Expected ExpandedWeights to all have the same loss_reduction argument but got one"
                    f"with {loss_reduction} and one with {arg.loss_reduction}"
                )

    unexpanded_args = tuple(
        arg.orig_weight if isinstance(arg, ExpandedWeight) else arg
        for arg in expanded_args
```
- **EN**: This block continues `_check_and_unexpand_args` and works to validate invariants and surface meaningful failures. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_check_and_unexpand_args`，用于校验不变量并给出有意义的失败信息。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 110-124
```python
    )
    unexpanded_kwargs = {
        name: arg.orig_weight if isinstance(arg, ExpandedWeight) else arg
        for (name, arg) in expanded_kwargs.items()
    }
    return unexpanded_args, unexpanded_kwargs


def maybe_scale_by_batch_size(grad_sample, expanded_weight):
    if expanded_weight.loss_reduction == "mean":
        return grad_sample * expanded_weight.batch_size
    else:
        return grad_sample
```
- **EN**: This module-level block helps connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 125-142
```python
def set_grad_sample_if_exists(maybe_expanded_weight, per_sample_grad_fn) -> None:
    unpacked = unpack_expanded_weight_or_tensor(maybe_expanded_weight)
    if isinstance(maybe_expanded_weight, ExpandedWeight):
        grad_sample_contribution = maybe_scale_by_batch_size(
            per_sample_grad_fn(unpacked), maybe_expanded_weight
        )

        if maybe_expanded_weight.batch_size > grad_sample_contribution.shape[0]:
            # this only passes the other checks if the arg allows smaller batch sizes
            intermediate = torch.zeros(
                maybe_expanded_weight.batch_size,
                *grad_sample_contribution.shape[1:],
                dtype=grad_sample_contribution.dtype,
                device=grad_sample_contribution.device,
            )
            intermediate[: grad_sample_contribution.shape[0]] = grad_sample_contribution
            grad_sample_contribution = intermediate
```
- **EN**: Defines the `set_grad_sample_if_exists` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`set_grad_sample_if_exists` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 143-154
```python
        if hasattr(unpacked, "grad_sample") and unpacked.grad_sample is not None:
            unpacked.grad_sample = unpacked.grad_sample + grad_sample_contribution
        else:
            unpacked.grad_sample = grad_sample_contribution


def unpack_expanded_weight_or_tensor(maybe_expanded_weight, func=lambda x: x):
    if isinstance(maybe_expanded_weight, ExpandedWeight):
        orig_weight = maybe_expanded_weight.orig_weight
        return func(orig_weight)
    elif (
        isinstance(maybe_expanded_weight, torch.Tensor)
```
- **EN**: This module-level block helps connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 155-171
```python
        and not maybe_expanded_weight.requires_grad
    ):
        return func(maybe_expanded_weight)
    elif isinstance(maybe_expanded_weight, torch.Tensor):
        raise RuntimeError(
            "ExpandedWeights currently does not support a mixture of ExpandedWeight parameters "
            "and normal Parameters. Please file and issue with pytorch/pytorch"
        )


def sum_over_all_but_batch_and_last_n(
    tensor: torch.Tensor,
    n_dims: int,
) -> torch.Tensor:
    r"""
    Calculate the sum over all dimensions, except the first (batch dimension), and excluding the last n_dims.
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设。

### Lines 172-183
```python
    This function will ignore the first dimension and it will
    not aggregate over the last n_dims dimensions.
    Args:
        tensor: An input tensor of shape ``(B, ..., X[n_dims-1])``.
        n_dims: Number of dimensions to keep.
    Example:
        >>> tensor = torch.ones(1, 2, 3, 4, 5)
        >>> sum_over_all_but_batch_and_last_n(tensor, n_dims=2).shape
        torch.Size([1, 4, 5])
    Returns:
        A tensor of shape ``(B, ..., X[n_dims-1])``
    """
```
- **EN**: This block continues `sum_over_all_but_batch_and_last_n` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `sum_over_all_but_batch_and_last_n`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 184-188
```python
    if tensor.dim() == n_dims + 1:
        return tensor
    else:
        dims = list(range(1, tensor.dim() - n_dims))
        return tensor.sum(dim=dims)
```
- **EN**: This block continues `sum_over_all_but_batch_and_last_n` and works to prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `sum_over_all_but_batch_and_last_n`，用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

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
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `.expanded_weights_impl`
- **Primary symbols / 核心符号**: `is_batch_first`, `standard_kwargs`, `forward_helper`, `_check_and_unexpand_args`, `maybe_scale_by_batch_size`, `set_grad_sample_if_exists`, `unpack_expanded_weight_or_tensor`, `sum_over_all_but_batch_and_last_n`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
