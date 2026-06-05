# scatter_gather.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/parallel/scatter_gather.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Provides wrappers and helpers for running neural-network modules across devices or processes.
- **Purpose (CN)**: 提供跨设备或进程运行神经网络模块的包装器与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
# mypy: allow-untyped-defs
from collections.abc import Sequence
from typing import Any, overload, TypeVar
from typing_extensions import deprecated

import torch
from torch.nn.parallel._functions import Gather, Scatter


__all__ = ["scatter", "scatter_kwargs", "gather"]


@deprecated(
    "`is_namedtuple` is deprecated, please use the python checks instead",
    category=FutureWarning,
)
```
- **EN**: These decorators register or transform the following definition so it can prepare neural-network operators or module behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够准备神经网络算子或模块行为。

### Lines 17-31
```python
def is_namedtuple(obj: Any) -> bool:
    # Check if type was created from collections.namedtuple or a typing.NamedTuple.
    return _is_namedtuple(obj)


def _is_namedtuple(obj: Any) -> bool:
    # Check if type was created from collections.namedtuple or a typing.NamedTuple.
    return (
        isinstance(obj, tuple) and hasattr(obj, "_asdict") and hasattr(obj, "_fields")
    )


T = TypeVar("T", dict, list, tuple)
```
- **EN**: This range initializes module-level constants or registries that later code reuses to prepare neural-network operators or module behavior.
- **CN**: 这一段初始化模块级常量或注册表，供后续代码重复使用以准备神经网络算子或模块行为。

### Lines 32-48
```python
# For some reason, 'scatter' returns a tuple when given a single Tensor input but a list otherwise.
@overload
def scatter(
    inputs: torch.Tensor,
    target_gpus: Sequence[int | torch.device],
    dim: int = ...,
) -> tuple[torch.Tensor, ...]: ...


@overload
def scatter(
    inputs: T,
    target_gpus: Sequence[int | torch.device],
    dim: int = ...,
) -> list[T]: ...
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 49-60
```python
def scatter(inputs, target_gpus, dim=0):
    r"""Slice tensors into approximately equal chunks and distributes them across given GPUs.

    Duplicates references to objects that are not tensors.
    """

    def scatter_map(obj):
        if isinstance(obj, torch.Tensor):
            return Scatter.apply(target_gpus, None, dim, obj)
        if _is_namedtuple(obj):
            return [
                type(obj)(*args)
```
- **EN**: Defines the `scatter` function; this block introduces logic that prepare neural-network operators or module behavior.
- **CN**: 定义`scatter` 函数；该代码块引入了用于准备神经网络算子或模块行为的逻辑。

### Lines 61-77
```python
                # pyrefly: ignore [bad-argument-type, no-matching-overload]
                for args in zip(*map(scatter_map, obj), strict=False)
            ]
        if isinstance(obj, tuple) and len(obj) > 0:
            # pyrefly: ignore [bad-argument-type, no-matching-overload]
            return list(zip(*map(scatter_map, obj), strict=False))
        if isinstance(obj, list) and len(obj) > 0:
            # pyrefly: ignore [bad-argument-type, no-matching-overload]
            return [list(i) for i in zip(*map(scatter_map, obj), strict=False)]
        if isinstance(obj, dict) and len(obj) > 0:
            return [
                type(obj)(i)
                # pyrefly: ignore [bad-argument-type, no-matching-overload]
                for i in zip(*map(scatter_map, obj.items()), strict=False)
            ]
        return [obj for _ in target_gpus]
```
- **EN**: This block continues `scatter` and works to prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `scatter`，用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 78-89
```python
    # After scatter_map is called, a scatter_map cell will exist. This cell
    # has a reference to the actual function scatter_map, which has references
    # to a closure that has a reference to the scatter_map cell (because the
    # fn is recursive). To avoid this reference cycle, we set the function to
    # None, clearing the cell
    try:
        res = scatter_map(inputs)
    finally:
        scatter_map = None  # type: ignore[assignment]
    return res
```
- **EN**: This module-level block helps prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果。

### Lines 90-101
```python
def scatter_kwargs(
    inputs: tuple[Any, ...],
    kwargs: dict[str, Any] | None,
    target_gpus: Sequence[int | torch.device],
    dim: int = 0,
) -> tuple[tuple[Any, ...], tuple[dict[str, Any], ...]]:
    r"""Scatter with support for kwargs dictionary."""
    scattered_inputs = scatter(inputs, target_gpus, dim) if inputs else []
    scattered_kwargs = scatter(kwargs, target_gpus, dim) if kwargs else []
    if len(scattered_inputs) < len(scattered_kwargs):
        scattered_inputs.extend(
            () for _ in range(len(scattered_kwargs) - len(scattered_inputs))
```
- **EN**: Defines the `scatter_kwargs` function; this block introduces logic that normalize dtype/device related arguments and behavior.
- **CN**: 定义`scatter_kwargs` 函数；该代码块引入了用于规范化 dtype/device 相关参数与行为的逻辑。

### Lines 102-116
```python
        )
    elif len(scattered_kwargs) < len(inputs):
        scattered_kwargs.extend(
            {} for _ in range(len(scattered_inputs) - len(scattered_kwargs))
        )
    return tuple(scattered_inputs), tuple(scattered_kwargs)


def gather(outputs: Any, target_device: int | torch.device, dim: int = 0) -> Any:
    r"""Gather tensors from different GPUs on a specified device.

    This function is useful for gathering the results of a distributed computation.
    It takes a sequence of objects, one for each GPU, and returns a single object
    on the specified device.
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果。

### Lines 117-128
```python
    Args:
        outputs (Any): A sequence of objects (potentially tensors) to gather.
        target_device (Union[int, torch.device]): The device to gather the tensors to.
            Use 'cpu' for CPU to avoid a deprecation warning.
        dim (int, optional): The dimension along which to gather. Default: 0.

    Returns:
        Any: A gathered object (potentially tensor) on the specified device.
    """

    def gather_map(outputs):
        out = outputs[0]
```
- **EN**: Defines the `gather` function; this block introduces logic that normalize dtype/device related arguments and behavior.
- **CN**: 定义`gather` 函数；该代码块引入了用于规范化 dtype/device 相关参数与行为的逻辑。

### Lines 129-143
```python
        if isinstance(out, torch.Tensor):
            return Gather.apply(target_device, dim, *outputs)
        if out is None:
            return None
        if isinstance(out, dict):
            if not all(len(out) == len(d) for d in outputs):
                raise ValueError("All dicts must have the same number of keys")
            # pyrefly: ignore [not-callable]
            return type(out)((k, gather_map([d[k] for d in outputs])) for k in out)
        if _is_namedtuple(out):
            # pyrefly: ignore [bad-argument-type]
            return type(out)._make(map(gather_map, zip(*outputs, strict=True)))
        # pyrefly: ignore [bad-argument-type]
        return type(out)(map(gather_map, zip(*outputs, strict=True)))
```
- **EN**: This block continues `gather` and works to normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `gather`，用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 144-150
```python
    # Recursive function calls like this create reference cycles.
    # Setting the function to None clears the refcycle.
    try:
        res = gather_map(outputs)
    finally:
        gather_map = None  # type: ignore[assignment]
    return res
```
- **EN**: This block continues `gather` and works to prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `gather`，用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn.parallel._functions`
- **Standard library / 标准库**: `collections.abc`, `typing`, `typing_extensions`
- **Primary symbols / 核心符号**: `__all__`, `is_namedtuple`, `_is_namedtuple`, `T`, `scatter`, `scatter_kwargs`, `gather`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
