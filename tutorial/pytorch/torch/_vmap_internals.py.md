# _vmap_internals.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_vmap_internals.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements internal helpers that support vmap-style batching transforms.
- **Purpose (CN)**: 实现支撑 vmap 类批处理变换的内部辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13 / 第 1-13 行
````python
# mypy: allow-untyped-defs
import functools
from collections.abc import Callable
from typing import Any
from typing_extensions import deprecated

import torch
from torch import Tensor
from torch.utils._pytree import _broadcast_to_and_flatten, tree_flatten, tree_unflatten


in_dims_t = int | tuple
out_dims_t = int | tuple[int, ...]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.utils._pytree; standard-library helpers such as functools, collections.abc, typing; other helper packages such as typing_extensions.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.utils._pytree；标准库辅助模块，如 functools、collections.abc、typing；其他辅助包，如 typing_extensions。

### Lines 16-31 / 第 16-31 行
````python
# Checks that all args-to-be-batched have the same batch dim size
def _validate_and_get_batch_size(
    flat_in_dims: list[int | None],
    flat_args: list,
) -> int:
    batch_sizes = [
        arg.size(in_dim)
        for in_dim, arg in zip(flat_in_dims, flat_args)
        if in_dim is not None
    ]
    if batch_sizes and any(size != batch_sizes[0] for size in batch_sizes):
        raise ValueError(
            f"vmap: Expected all tensors to have the same size in the mapped "
            f"dimension, got sizes {batch_sizes} for the mapped dimension"
        )
    return batch_sizes[0]
````
- **EN**: This chunk defines `_validate_and_get_batch_size`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_validate_and_get_batch_size`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 34-51 / 第 34-51 行
````python
def _num_outputs(batched_outputs: Tensor | tuple[Tensor, ...]) -> int:
    if isinstance(batched_outputs, tuple):
        return len(batched_outputs)
    return 1


# If value is a tuple, check it has length `num_elements`.
# If value is not a tuple, make a tuple with `value` repeated `num_elements` times
def _as_tuple(
    value: Any,
    num_elements: int,
    error_message_lambda: Callable[[], str],
) -> tuple:
    if not isinstance(value, tuple):
        return (value,) * num_elements
    if len(value) != num_elements:
        raise ValueError(error_message_lambda())
    return value
````
- **EN**: This chunk defines `_as_tuple`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_as_tuple`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 54-73 / 第 54-73 行
````python
# Creates BatchedTensors for every Tensor in arg that should be batched.
# Returns the (potentially) batched arguments and the batch_size.
def _create_batched_inputs(
    in_dims: in_dims_t,
    args: tuple,
    vmap_level: int,
    func: Callable,
) -> tuple[tuple, int]:
    if not isinstance(in_dims, int) and not isinstance(in_dims, tuple):
        raise ValueError(
            f"vmap({_get_name(func)}, in_dims={in_dims}, ...)(<inputs>): "
            f"expected `in_dims` to be int or a (potentially nested) tuple "
            f"matching the structure of inputs, got: {type(in_dims)}."
        )
    if len(args) == 0:
        raise ValueError(
            f"vmap({_get_name(func)})(<inputs>): got no inputs. Maybe you forgot to add "
            f"inputs, or you are trying to vmap over a function with no inputs. "
            f"The latter is unsupported."
        )
````
- **EN**: This chunk defines `_create_batched_inputs`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_create_batched_inputs`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 75-94 / 第 75-94 行
````python
    flat_args, args_spec = tree_flatten(args)
    flat_in_dims = _broadcast_to_and_flatten(in_dims, args_spec)
    if flat_in_dims is None:
        raise ValueError(
            f"vmap({_get_name(func)}, in_dims={in_dims}, ...)(<inputs>): "
            f"in_dims is not compatible with the structure of `inputs`. "
            f"in_dims has structure {tree_flatten(in_dims)[1]} but inputs "
            f"has structure {args_spec}."
        )

    for arg, in_dim in zip(flat_args, flat_in_dims):
        if not isinstance(in_dim, int) and in_dim is not None:
            raise ValueError(
                f"vmap({_get_name(func)}, in_dims={in_dims}, ...)(<inputs>): "
                f"Got in_dim={in_dim} for an input but in_dim must be either "
                f"an integer dimension or None."
            )
        if isinstance(in_dim, int) and not isinstance(arg, Tensor):
            raise ValueError(
                f"vmap({_get_name(func)}, in_dims={in_dims}, ...)(<inputs>): "
````
- **EN**: This chunk continues `_create_batched_inputs` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_create_batched_inputs`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 95-113 / 第 95-113 行
````python
                f"Got in_dim={in_dim} for an input but the input is of type "
                f"{type(arg)}. We cannot vmap over non-Tensor arguments, "
                f"please use None as the respective in_dim"
            )
        if in_dim is not None and (in_dim < 0 or in_dim >= arg.dim()):
            raise ValueError(
                f"vmap({_get_name(func)}, in_dims={in_dims}, ...)(<inputs>): "
                f"Got in_dim={in_dim} for some input, but that input is a Tensor "
                f"of dimensionality {arg.dim()} so expected in_dim to satisfy "
                f"0 <= in_dim < {arg.dim()}."
            )

    batch_size = _validate_and_get_batch_size(flat_in_dims, flat_args)
    # See NOTE [Ignored _remove_batch_dim, _add_batch_dim]
    batched_inputs = [
        arg if in_dim is None else torch._add_batch_dim(arg, in_dim, vmap_level)
        for in_dim, arg in zip(flat_in_dims, flat_args)
    ]
    return tree_unflatten(batched_inputs, args_spec), batch_size
````
- **EN**: This chunk continues `_create_batched_inputs` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_create_batched_inputs`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 116-131 / 第 116-131 行
````python
# Undos the batching (and any batch dimensions) associated with the `vmap_level`.
def _unwrap_batched(
    batched_outputs: Tensor | tuple[Tensor, ...],
    out_dims: out_dims_t,
    vmap_level: int,
    batch_size: int,
    func: Callable,
    allow_none_pass_through: bool = False,
) -> tuple:
    num_outputs = _num_outputs(batched_outputs)
    out_dims_as_tuple = _as_tuple(
        out_dims,
        num_outputs,
        lambda: f"vmap({_get_name(func)}, ..., out_dims={out_dims}): `out_dims` must "
        f"have one dim per output (got {num_outputs} outputs) of {_get_name(func)}.",
    )
````
- **EN**: This chunk defines `_unwrap_batched`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段定义了 `_unwrap_batched`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 133-152 / 第 133-152 行
````python
    # NOTE [Ignored _remove_batch_dim, _add_batch_dim]
    # There is something wrong with our type bindings for functions that begin
    # with '_', see #40397.
    if isinstance(batched_outputs, Tensor):
        out_dim = out_dims_as_tuple[0]
        return torch._remove_batch_dim(batched_outputs, vmap_level, batch_size, out_dim)  # type: ignore[return-value]
    if allow_none_pass_through:
        return tuple(
            (
                torch._remove_batch_dim(out, vmap_level, batch_size, out_dim)
                if out is not None
                else None
            )
            for out, out_dim in zip(batched_outputs, out_dims_as_tuple)
        )
    else:
        return tuple(
            torch._remove_batch_dim(out, vmap_level, batch_size, out_dim)
            for out, out_dim in zip(batched_outputs, out_dims_as_tuple)
        )
````
- **EN**: This chunk continues `_unwrap_batched` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_unwrap_batched`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 155-173 / 第 155-173 行
````python
# Checks that `fn` returned one or more Tensors and nothing else.
# NB: A python function that return multiple arguments returns a single tuple,
# so we are effectively checking that `outputs` is a single Tensor or a tuple of
# Tensors.
def _validate_outputs(outputs: Any, func: Callable) -> None:
    if isinstance(outputs, Tensor):
        return
    if not isinstance(outputs, tuple):
        raise ValueError(
            f"vmap({_get_name(func)}, ...): `{_get_name(func)}` must only return "
            f"Tensors, got type {type(outputs)} as the return."
        )
    for idx, output in enumerate(outputs):
        if isinstance(output, Tensor):
            continue
        raise ValueError(
            f"vmap({_get_name(func)}, ...): `{_get_name(func)}` must only return "
            f"Tensors, got type {type(output)} for return {idx}."
        )
````
- **EN**: This chunk defines `_validate_outputs`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_validate_outputs`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 176-191 / 第 176-191 行
````python
def _check_out_dims_is_int_or_int_tuple(out_dims: out_dims_t, func: Callable) -> None:
    if isinstance(out_dims, int):
        return
    if not isinstance(out_dims, tuple) or not all(
        isinstance(out_dim, int) for out_dim in out_dims
    ):
        raise ValueError(
            f"vmap({_get_name(func)}, ..., out_dims={out_dims}): `out_dims` must be "
            f"an int or a tuple of int representing where in the outputs the "
            f"vmapped dimension should appear."
        )


def _get_name(func: Callable):
    if hasattr(func, "__name__"):
        return func.__name__
````
- **EN**: This chunk defines `_get_name`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_name`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 193-210 / 第 193-210 行
````python
    # Not all callables have __name__, in fact, only static functions/methods do.
    # A callable created via functools.partial or an nn.Module, to name some
    # examples, don't have a __name__.
    return repr(func)


# vmap(func)(inputs) wraps all Tensor inputs to be batched in BatchedTensors,
# sends those into func, and then unwraps the output BatchedTensors. Operations
# on BatchedTensors perform the batched operations that the user is asking for.
@deprecated(
    "Please use `torch.vmap` instead of `torch._vmap_internals.vmap`.",
    category=FutureWarning,
)
def vmap(func: Callable, in_dims: in_dims_t = 0, out_dims: out_dims_t = 0) -> Callable:
    """
    Please use torch.vmap instead of this API.
    """
    return _vmap(func, in_dims, out_dims)
````
- **EN**: This chunk defines `vmap`, which implements a focused helper used by the surrounding module. Decorators such as `deprecated` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `vmap`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deprecated` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 213-232 / 第 213-232 行
````python
# A version of vmap but without the initial "experimental prototype" warning
def _vmap(
    func: Callable,
    in_dims: in_dims_t = 0,
    out_dims: out_dims_t = 0,
    allow_none_pass_through: bool = False,
) -> Callable:
    # The `allow_none_pass_through` argument is a temporary workaround may be removed.
    # Currently it enables us to wrap the call in `autograd.grad` to the autograd engine,
    # which may return None if any of the inputs are unused. See the issue discussing this:
    # https://github.com/pytorch/functorch/issues/159.
    @functools.wraps(func)
    def wrapped(*args):
        _check_out_dims_is_int_or_int_tuple(out_dims, func)
        vmap_level = torch._C._vmapmode_increment_nesting()
        try:
            batched_inputs, batch_size = _create_batched_inputs(
                in_dims, args, vmap_level, func
            )
            batched_outputs = func(*batched_inputs)
````
- **EN**: This chunk defines `wrapped`, which implements a focused helper used by the surrounding module. Decorators such as `functools.wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `wrapped`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 233-246 / 第 233-246 行
````python
            if not allow_none_pass_through:
                _validate_outputs(batched_outputs, func)
            return _unwrap_batched(
                batched_outputs,
                out_dims,
                vmap_level,
                batch_size,
                func,
                allow_none_pass_through=allow_none_pass_through,
            )
        finally:
            torch._C._vmapmode_decrement_nesting()

    return wrapped
````
- **EN**: This chunk continues `wrapped` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `wrapped`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **_vmap_internals**
  - EN: Implements internal helpers that support vmap-style batching transforms.
  - CN: 实现支撑 vmap 类批处理变换的内部辅助逻辑。
- **_validate_and_get_batch_size**
  - EN: `_validate_and_get_batch_size` is one of the main symbols declared or implemented in this file.
  - CN: `_validate_and_get_batch_size` 是本文件声明或实现的主要符号之一。
- **_num_outputs**
  - EN: `_num_outputs` is one of the main symbols declared or implemented in this file.
  - CN: `_num_outputs` 是本文件声明或实现的主要符号之一。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.utils._pytree`
- **Standard library / 标准库**: `functools`, `collections.abc`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `_validate_and_get_batch_size`, `_num_outputs`, `_as_tuple`, `_create_batched_inputs`, `_unwrap_batched`, `_validate_outputs`, `_check_out_dims_is_int_or_int_tuple`, `_get_name`, `vmap`, `_vmap`
