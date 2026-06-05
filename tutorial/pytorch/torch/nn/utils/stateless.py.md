# stateless.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/utils/stateless.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects helper routines that reshape, inspect, or adapt nn.Module behavior.
- **Purpose (CN)**: 汇集用于重塑、检查或适配 nn.Module 行为的辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```python
# mypy: allow-untyped-defs
import contextlib
from typing import Any
from typing_extensions import deprecated

import torch
from torch import Tensor
from torch.nn.utils._named_member_accessor import NamedMemberAccessor


__all__ = ["functional_call"]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 14-27
```python
def _untie_named_tensors_map(
    module: "torch.nn.Module",
    parameters_and_buffers: dict[str, Tensor],
) -> dict[str, Tensor]:
    """
    Unties all tied tensors in the module to parameters_and_buffers.

    This function returns a new untied_parameters_and_buffers dictionary and leave the original
    untied_parameters_and_buffers dictionary unchanged. It adds new (missing) keys for tied tensors
    in the module to untied_parameters_and_buffers. The value of the new key is the user-given value
    in the original parameters_and_buffers dictionary.

    If there are more than one user-given values for the same tied tensor, it will raise an error.
```
- **EN**: Defines the `_untie_named_tensors_map` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`_untie_named_tensors_map` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 28-39
```python
    For example, if the module has two tied weights self.foo and self.tied_foo and the user passes
    {'foo': foo_value, ...}, this will return {'foo': foo_value, 'tied_foo': foo_value, ...}. If the
    user passes {'foo': foo_value, 'tied_foo': tied_foo_value, ...}, it will raise an error. If the
    user passes {'foo': foo_value, 'tied_foo': foo_value, ...}, it will not raise an error.

    Args:
        module (torch.nn.Module): the module to determine which tensors are tied.
        parameters_and_buffers (Dict[str, Tensor]): a map of {name: tensor} for reparamaterizing the module.

    Returns:
        A new untied version of the parameters_and_buffers dictionary.
```
- **EN**: This block continues `_untie_named_tensors_map` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding.
- **CN**: 该代码块继续实现 `_untie_named_tensors_map`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设。

### Lines 40-54
```python
    Raises:
        ValueError: if there are more than one user-given values for the same tied tensor.
    """
    # A map of {name: tensor} for all tensors (including tied ones) in the module.
    all_named_tensors: dict[str, Tensor] = {}
    all_named_tensors.update(module.named_parameters(remove_duplicate=False))
    all_named_tensors.update(module.named_buffers(remove_duplicate=False))

    # A map of {tensor: set(all_tied_names)} for all tensor names in the module.
    tensor_to_tied_names_map: dict[Tensor, set[str]] = {}
    for name, tensor in all_named_tensors.items():
        if tensor not in tensor_to_tied_names_map:
            tensor_to_tied_names_map[tensor] = set()
        tensor_to_tied_names_map[tensor].add(name)
```
- **EN**: This block continues `_untie_named_tensors_map` and works to organize reusable module behavior and state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_untie_named_tensors_map`，用于组织可复用的模块行为与状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 55-71
```python
    # A map of {tied_name: set(all_tied_names)} for all tensor names in the module.
    # If a name is not tied, it will not be in this map.
    tied_names_map: dict[str, set[str]] = {}
    for tied_names in tensor_to_tied_names_map.values():
        if len(tied_names) > 1:
            for tied_name in tied_names:
                tied_names_map[tied_name] = tied_names

    # Make sure the user didn't pass multiple values for the same tied tensor.
    given_names = set(parameters_and_buffers.keys())
    # same as given_names.intersection(tied_names_map.keys()) but dynamo can't
    # handle that
    given_names_for_tied_tensors: set[str] = set()
    for name in given_names:
        if name in tied_names_map:
            given_names_for_tied_tensors.add(name)
```
- **EN**: This block continues `_untie_named_tensors_map` and works to organize reusable module behavior and state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_untie_named_tensors_map`，用于组织可复用的模块行为与状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 72-86
```python
    for given_name in given_names_for_tied_tensors:
        tied_names = tied_names_map[given_name]
        if (
            # Detect if there are multiple keys present for the same tied tensor.
            len(tied_names.intersection(given_names_for_tied_tensors)) > 1
            # Only raise an error if the user passed multiple values for the same tied tensor.
            # If all given values are the same, don't raise.
            and len({parameters_and_buffers[tied_name] for tied_name in tied_names})
            != 1
        ):
            raise ValueError(
                f"functional_call got multiple values for keys {sorted(tied_names)}, "
                f"which are tied. Consider using tie_weights=False"
            )
```
- **EN**: This block continues `_untie_named_tensors_map` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_untie_named_tensors_map`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 87-98
```python
    # Untie the given named tensor map
    # Make a copy for not modifying the original dict
    untied_parameters_and_buffers = parameters_and_buffers.copy()
    for given_name in given_names_for_tied_tensors:
        for tied_name in tied_names_map[given_name]:
            untied_parameters_and_buffers[tied_name] = parameters_and_buffers[
                given_name
            ]
    return untied_parameters_and_buffers


@contextlib.contextmanager
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 99-112
```python
def _reparametrize_module(
    module: "torch.nn.Module",
    parameters_and_buffers: dict[str, Tensor],
    tie_weights: bool = False,
    strict: bool = False,
    stack_weights: bool = False,
):
    if tie_weights:
        untied_parameters_and_buffers = _untie_named_tensors_map(
            module, parameters_and_buffers
        )
    else:
        untied_parameters_and_buffers = parameters_and_buffers
```
- **EN**: Defines the `_reparametrize_module` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`_reparametrize_module` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 113-124
```python
    accessor = NamedMemberAccessor(module)
    if strict:
        missing_keys, unexpected_keys = accessor.check_keys(
            untied_parameters_and_buffers
        )
        error_msgs = []
        if len(unexpected_keys) > 0:
            error_msgs.append(
                f"Unexpected key(s): {', '.join(map(repr, unexpected_keys))}."
            )
        if len(missing_keys) > 0:
            error_msgs.append(f"Missing key(s): {', '.join(map(repr, missing_keys))}.")
```
- **EN**: This block continues `_reparametrize_module` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `_reparametrize_module`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 125-136
```python
        if len(error_msgs) > 0:
            raise RuntimeError(
                "Error(s) in reparametrizing for {}:\n\t{}".format(
                    module._get_name(), "\n\t".join(error_msgs)
                )
            )

    orig_parameters_and_buffers: dict[str, Tensor] = {}
    try:
        orig_parameters_and_buffers, _ = accessor.swap_tensors_dict(
            untied_parameters_and_buffers, allow_missing=True
        )
```
- **EN**: This block continues `_reparametrize_module` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_reparametrize_module`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 137-148
```python
        yield
    finally:
        if stack_weights:
            # When stacking is enabled, we will restore the weights in LIFO order.
            orig_parameters_and_buffers = dict(
                reversed(orig_parameters_and_buffers.items())
            )
        new_parameters_and_buffers, _ = accessor.swap_tensors_dict(
            orig_parameters_and_buffers, allow_missing=True
        )
        # Sometimes the module is not completely stateless and has some in-place modifications on
        # the _parameters and _buffers dictionaries.
```
- **EN**: This block continues `_reparametrize_module` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `_reparametrize_module`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 149-164
```python
        # Write the changed parameters and buffers back to the original dict.
        parameters_and_buffers.update(
            {
                k: new_parameters_and_buffers[k]
                for k in parameters_and_buffers
                if k in new_parameters_and_buffers
            }
        )


@deprecated(
    "`torch.nn.utils.stateless.functional_call` is deprecated as of PyTorch 2.0 "
    "and will be removed in a future version of PyTorch. "
    "Please use `torch.func.functional_call` instead which is a drop-in replacement.",
    category=FutureWarning,
)
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 165-181
```python
def functional_call(
    module: "torch.nn.Module",
    parameters_and_buffers: dict[str, Tensor],
    args: Any | tuple | None = None,
    kwargs: dict[str, Any] | None = None,
    *,
    tie_weights: bool = True,
    strict: bool = False,
):
    r"""Perform a functional call on the module by replacing the module parameters and buffers with the provided ones.

    .. warning::

        This API is deprecated as of PyTorch 2.0 and will be removed in a future
        version of PyTorch. Please use :func:`torch.func.functional_call` instead,
        which is a drop-in replacement for this API.
```
- **EN**: Defines the `functional_call` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`functional_call` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 182-193
```python
    .. note:: If the module has active parametrizations, passing a value in the
        :attr:`parameters_and_buffers` argument with the name set to the regular parameter
        name will completely disable the parametrization.
        If you want to apply the parametrization function to the value passed
        please set the key as ``{submodule_name}.parametrizations.{parameter_name}.original``.

    .. note:: If the module performs in-place operations on parameters/buffers, these will be reflected
        in the `parameters_and_buffers` input.

        Example::

            >>> a = {'foo': torch.zeros(())}
```
- **EN**: This block continues `functional_call` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `functional_call`，用于组织可复用的模块行为与状态。

### Lines 194-205
```python
            >>> # xdoctest: +SKIP
            >>> mod = Foo()  # does self.foo = self.foo + 1
            >>> print(mod.foo)  # tensor(0.)
            >>> functional_call(mod, a, torch.ones(()))
            >>> print(mod.foo)  # tensor(0.)
            >>> print(a['foo'])  # tensor(1.)

    .. note:: If the module has tied weights, whether or not functional_call respects the tying is determined by the
        tie_weights flag.

        Example::
```
- **EN**: This block continues `functional_call` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `functional_call`，用于组织可复用的模块行为与状态。

### Lines 206-217
```python
            >>> a = {'foo': torch.zeros(())}
            >>> # xdoctest: +SKIP
            >>> mod = Foo()  # has both self.foo and self.foo_tied which are tied. Returns x + self.foo + self.foo_tied
            >>> print(mod.foo)  # tensor(1.)
            >>> mod(torch.zeros(()))  # tensor(2.)
            >>> functional_call(mod, a, torch.zeros(()))  # tensor(0.) since it will change self.foo_tied too
            >>> functional_call(mod, a, torch.zeros(()), tie_weights=False)  # tensor(1.)--self.foo_tied is not updated
            >>> new_a = {'foo': torch.zeros(()), 'foo_tied': torch.zeros(())}
            >>> functional_call(mod, new_a, torch.zeros()) # tensor(0.)

    Args:
        module (torch.nn.Module): the module to call
```
- **EN**: This block continues `functional_call` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `functional_call`，用于组织可复用的模块行为与状态。

### Lines 218-229
```python
        parameters_and_buffers (dict of str and Tensor): the parameters that will be used in
            the module call.
        args (Any or tuple): arguments to be passed to the module call. If not a tuple, considered a single argument.
        kwargs (dict): keyword arguments to be passed to the module call
        tie_weights (bool, optional): If True, then parameters and buffers tied in the original model will be treated as
            tied in the reparamaterized version. Therefore, if True and different values are passed for the tied
            parameters and buffers, it will error. If False, it will not respect the originally tied parameters and
            buffers unless the values passed for both weights are the same. Default: True.
        strict (bool, optional): If True, then the parameters and buffers passed in must match the parameters and
            buffers in the original module. Therefore, if True and there are any missing or unexpected keys, it will
            error. Default: False.
```
- **EN**: This block continues `functional_call` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `functional_call`，用于组织可复用的模块行为与状态。

### Lines 230-242
```python
    Returns:
        Any: the result of calling ``module``.
    """
    return _functional_call(
        module,
        parameters_and_buffers,
        args,
        kwargs,
        tie_weights=tie_weights,
        strict=strict,
    )
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 243-254
```python
def _functional_call(
    module: "torch.nn.Module",
    parameters_and_buffers: dict[str, Tensor],
    args: Any | tuple | None = None,
    kwargs: dict[str, Any] | None = None,
    *,
    tie_weights: bool = True,
    strict: bool = False,
):
    # TODO allow kwargs such as unsafe and others for parametrization
    if (
        torch.jit.is_tracing()
```
- **EN**: Defines the `_functional_call` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`_functional_call` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 255-266
```python
        or torch.jit.is_scripting()
        or isinstance(
            module,
            (
                torch.jit.RecursiveScriptModule,
                torch.jit.ScriptModule,
                torch.jit.ScriptFunction,
            ),
        )
    ):
        raise RuntimeError("The stateless API can't be used with Jitted modules")
    if isinstance(module, torch.nn.DataParallel):
```
- **EN**: This block continues `_functional_call` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_functional_call`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 267-278
```python
        raise RuntimeError(
            "The stateless API can't be used with nn.DataParallel module"
        )
    if kwargs is None:
        kwargs = {}
    if args is None:
        args = ()
    elif not isinstance(args, tuple):
        args = (args,)
    with _reparametrize_module(
        module, parameters_and_buffers, tie_weights=tie_weights, strict=strict
    ):
```
- **EN**: This block continues `_functional_call` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `_functional_call`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 279-279
```python
        return module(*args, **kwargs)
```
- **EN**: This block continues `_functional_call` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `_functional_call`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn.utils._named_member_accessor`
- **Standard library / 标准库**: `contextlib`, `typing`, `typing_extensions`
- **Primary symbols / 核心符号**: `__all__`, `_untie_named_tensors_map`, `_reparametrize_module`, `functional_call`, `_functional_call`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
