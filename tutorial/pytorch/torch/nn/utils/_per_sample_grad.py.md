# _per_sample_grad.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/utils/_per_sample_grad.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects helper routines that reshape, inspect, or adapt nn.Module behavior.
- **Purpose (CN)**: 汇集用于重塑、检查或适配 nn.Module 行为的辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
# mypy: allow-untyped-defs
import functools

import torch
from torch.nn.utils._expanded_weights.expanded_weights_impl import ExpandedWeight
from torch.utils import _pytree as pytree


# dependency on `functional_call` means that this can't be exposed in utils
# without creating circular dependency
def call_for_per_sample_grads(
    module,
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 13-24
```python
    *,
    batch_size=None,
    loss_reduction="sum",
    batch_first=True,
):
    r"""
    Return a forward function for a module, populating grad_sample with per sample gradients on backward invocation.

    Args:
        module: The ``nn.Module`` to get per sample gradients with respect to. All trainable
          parameters will compute per sample gradients, located in a ``grad_sample``
          field when ``backward`` is invoked
```
- **EN**: This block continues `call_for_per_sample_grads` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `call_for_per_sample_grads`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 25-36
```python
        batch_size: The batch size of the input. If None is passed, all tensor arguments in args and kwargs must have
          the same batch size, which is the size of the first dimension. Otherwise, it must be passed manually.
          Default: None
        loss_reduction: Indicates if the loss reduction (for aggregating the gradients) is a sum or a mean operation. If
          "mean", per sample gradients will be scaled by the batch size to offset the crossbatch interaction from
          running mean across a batch. Must be "mean" or "sum". Default: "sum"
        batch_first: Indicates if the batch dimension is the first dimension. If True, the batch dimension is the first
          dimension. If False, it's the second dimension. Default: True.

    Examples::
        >>> # xdoctest: +SKIP
        >>> model = nn.Linear(4, 3)
```
- **EN**: This block continues `call_for_per_sample_grads` and works to connect execution with differentiation-aware logic.
- **CN**: 该代码块继续实现 `call_for_per_sample_grads`，用于将执行过程与可微分逻辑连接起来。

### Lines 37-48
```python
        >>> batched_input = torch.randn(5, 4)  # batch size of 5
        >>> res = call_for_per_sample_grads(model)(batched_input).sum()
        >>> res.backward()
        >>> assert model.weight.shape == (3, 4)
        >>> assert model.weight.grad_sample.shape == (5, 3, 4)
        >>> assert model.weight.grad is None
        >>> assert model.bias.shape == (3,)
        >>> assert model.bias.grad_sample.shape == (5, 3)
        >>> assert model.bias.grad is None

    An example using "mean" loss reduction. The grad_sample fields will be scaled by batch_size from what they would be
    if we ran the same code with loss_reduction="sum". This is because the mean at the end will scale all
```
- **EN**: This block continues `call_for_per_sample_grads` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `call_for_per_sample_grads`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 49-61
```python
    grad_outputs by 1 / batch_size from cross batch interaction.
        >>> model = nn.Linear(4, 3)
        >>> batched_input = torch.randn(5, 4)  # batch size of 5
        >>> res = call_for_per_sample_grads(model, 5, loss_reduction="mean")(
        ...     batched_input
        ... ).mean()
        >>> res.backward()

    Note::
        Does not work with any `nn.RNN`, including `nn.GRU` or `nn.LSTM`. Please use custom
        rewrites that wrap an `nn.Linear` module. See Opacus for an example
    """
```
- **EN**: This block continues `call_for_per_sample_grads` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `call_for_per_sample_grads`，用于组织可复用的模块行为与状态。

### Lines 62-74
```python
    def maybe_build_expanded_weight(og_tensor, batch_size):
        if og_tensor.requires_grad:
            return ExpandedWeight(og_tensor, batch_size, loss_reduction)
        else:
            return og_tensor

    def compute_batch_size(*args, **kwargs):
        args_and_kwargs = pytree.arg_tree_leaves(*args, **kwargs)
        batch_size = None
        for arg in args_and_kwargs:
            if not isinstance(arg, torch.Tensor):
                continue
```
- **EN**: Defines the `call_for_per_sample_grads` function; this block introduces logic that connect execution with differentiation-aware logic.
- **CN**: 定义`call_for_per_sample_grads` 函数；该代码块引入了用于将执行过程与可微分逻辑连接起来的逻辑。

### Lines 75-90
```python
            arg_batch_size = arg.shape[0] if batch_first else arg.shape[1]
            if batch_size is not None and batch_size != arg_batch_size:
                raise RuntimeError(
                    "When computing batch size, found at least one input with batch size "
                    f"{batch_size} and one with batch size {arg_batch_size}. Please specify it "
                    "explicitly using the batch size kwarg in call_for_per_sample_grads"
                )
            batch_size = arg_batch_size
        if batch_size is None:
            raise RuntimeError(
                "Unable to find a tensor in the passed args and kwargs. They may not be pytree-able "
                "and so ExpandedWeights cannot compute the batch size from the inputs. Please specify "
                "it explicitly"
            )
        return batch_size
```
- **EN**: This block continues `call_for_per_sample_grads` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `call_for_per_sample_grads`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 91-102
```python
    if loss_reduction not in ["sum", "mean"]:
        raise RuntimeError(
            f"Expected loss_reduction argument to be sum or mean, got {loss_reduction}"
        )

    if not isinstance(module, torch.nn.Module):
        raise RuntimeError(
            f"Module passed must be nn.Module, got {type(module).__name__}"
        )
    if not (batch_size is None or isinstance(batch_size, int)):
        raise RuntimeError(
            f"Batch size passed must be None or an integer, got {type(batch_size).__name__}"
```
- **EN**: This block continues `call_for_per_sample_grads` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `call_for_per_sample_grads`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 103-119
```python
        )
    if batch_size is not None and batch_size < 1:
        raise RuntimeError(f"Batch size must be positive, got {batch_size}")
    for weight in module.parameters():
        if hasattr(weight, "grad_sample") and weight.grad_sample is not None:  # type: ignore[attr-defined]
            raise RuntimeError(
                "Current Expanded Weights accumulates the gradients, which will be incorrect for multiple "
                f"calls without clearing gradients. Please clear out the grad_sample parameter of {weight} or "
                "post an issue to pytorch/pytorch to prioritize correct behavior"
            )

    @functools.wraps(module.forward)
    def wrapper(*args, **kwargs):
        wrapper_batch_size = batch_size
        if wrapper_batch_size is None:
            wrapper_batch_size = compute_batch_size(*args, **kwargs)
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 120-126
```python
        params = {
            name: maybe_build_expanded_weight(value, wrapper_batch_size)
            for (name, value) in module.named_parameters()
        }
        return torch.func.functional_call(module, params, args, kwargs)

    return wrapper
```
- **EN**: This block continues `call_for_per_sample_grads` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `call_for_per_sample_grads`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn.utils._expanded_weights.expanded_weights_impl`, `torch.utils`
- **Standard library / 标准库**: `functools`
- **Primary symbols / 核心符号**: `call_for_per_sample_grads`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
