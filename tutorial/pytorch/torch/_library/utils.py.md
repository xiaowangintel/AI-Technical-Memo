# utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_library/utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python helpers for torch.library registration, library fragments, and operator definitions.
- **Purpose (CN)**: 实现 torch.library 注册、库片段以及算子定义相关的 Python 辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-27 / 第 1-27 行
````python
# mypy: allow-untyped-defs
import dataclasses
import inspect
import sys
from collections.abc import Callable, Iterable, Iterator
from typing import Any, Literal, overload

import torch
import torch.utils._pytree as pytree
import torchgen
from torch import _C, _utils_internal
from torch._ops import OpOverload


@dataclasses.dataclass
class Kernel:
    """Models a (function, source location)"""

    func: Callable
    source: str

    def __call__(self, *args, **kwargs):
        return self.func(*args, **kwargs)


class RegistrationHandle:
    """Does something when someone calls .destroy() on it"""
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.utils._pytree, torch._ops; standard-library helpers such as dataclasses, inspect, sys, ...; other helper packages such as torchgen. It introduces or extends `Kernel`, `RegistrationHandle`, which hold the main object-oriented state for this portion of the file. This chunk defines `__call__`, which implements a focused helper used by the surrounding module. Decorators such as `dataclasses.dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.utils._pytree、torch._ops；标准库辅助模块，如 dataclasses、inspect、sys、...；其他辅助包，如 torchgen。 它引入或扩展了 `Kernel`、`RegistrationHandle`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__call__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `dataclasses.dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 29-47 / 第 29-47 行
````python
    def __init__(self, on_destroy: Callable):
        self._on_destroy = on_destroy

    def destroy(self) -> None:
        self._on_destroy()


def get_source(stacklevel: int) -> str:
    """Get a string that represents the caller.

    Example: "/path/to/foo.py:42"

    Use stacklevel=1 to get the caller's source
    Use stacklevel=2 to get the caller's caller's source
    etc.
    """
    frame = inspect.getframeinfo(sys._getframe(stacklevel))
    source = f"{frame.filename}:{frame.lineno}"
    return source
````
- **EN**: This chunk defines `get_source`, which retrieves runtime state and exposes it through a Python-friendly accessor. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_source`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 50-76 / 第 50-76 行
````python
def parse_namespace(qualname: str) -> tuple[str, str]:
    splits = qualname.split("::")
    if len(splits) != 2:
        raise ValueError(
            f"Expected `qualname` to be of the form "
            f'"namespace::name", but got {qualname}. '
            f"The qualname passed to the torch.library APIs must consist "
            f"of a namespace and a name, e.g. aten::sin"
        )
    return splits[0], splits[1]


def lookup_op(qualname: str) -> OpOverload:
    namespace, name = parse_namespace(qualname)
    if "." in name:
        name, overload = name.split(".")
    else:
        overload = "default"
    ns = getattr(torch.ops, namespace)
    packet = getattr(ns, name)
    return getattr(packet, overload)


def is_builtin(op: OpOverload) -> bool:
    if not isinstance(op, OpOverload):
        raise AssertionError(f"op must be OpOverload, got {type(op)}")
    return op.namespace in {"aten", "prim", "prims"}
````
- **EN**: This chunk defines `is_builtin`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_builtin`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 79-104 / 第 79-104 行
````python
def is_functional_schema(schema: Any, *, allow_valid_view: bool = False) -> bool:
    """Check if the schema is functional.

    An operator is functional if:
    - it does not mutate any of its inputs
    - If no view are allowed
        - it does not return a view on any of its inputs
    - If valid views are allowed
        - it is not a view or a view with a single input Tensor and single output Tensor
    - it has at least one return
    """

    def is_functional(schema):
        if schema.is_mutable:
            return False
        rets = schema.returns
        is_non_mutating_view = len(rets) > 0 and any(
            r.alias_info is not None and not r.alias_info.is_write for r in rets
        )
        num_tensor_inputs = 0
        num_tensor_outputs = 0

        if isinstance(schema, torch.FunctionSchema):
            for arg in schema.arguments:
                if isinstance(arg.type, torch.TensorType):
                    num_tensor_inputs += 1
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `is_functional`, which checks a capability or invariant before later code relies on it. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `is_functional`，其作用是检查某项能力或不变量，供后续逻辑依赖。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 106-131 / 第 106-131 行
````python
            for ret in schema.returns:
                if isinstance(ret.type, torch.TensorType):
                    num_tensor_outputs += 1

        elif isinstance(schema, torchgen.model.FunctionSchema):
            for argument in schema.arguments.flat_non_out:
                if argument.type.is_tensor_like():
                    num_tensor_inputs += 1

            for ret_arg in schema.returns:
                if ret_arg.type.is_tensor_like():
                    num_tensor_outputs += 1

        if is_non_mutating_view:
            return allow_valid_view and (
                num_tensor_inputs == 1 and num_tensor_outputs == 1
            )
        if not schema.returns:
            return False
        return True

    if isinstance(schema, torch._C.FunctionSchema):
        return is_functional(schema)

    # Lazy import because not all PyTorch builds have torchgen
    from torchgen.model import FunctionSchema
````
- **EN**: This block assembles module dependencies, pulling in other helper packages such as torchgen.model. This chunk continues `is_functional` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了其他辅助包，如 torchgen.model。 这一段延续了 `is_functional`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 133-156 / 第 133-156 行
````python
    if isinstance(schema, str):
        schema = FunctionSchema.parse(schema)
    if not isinstance(schema, FunctionSchema):
        raise AssertionError(f"schema must be FunctionSchema, got {type(schema)}")
    return is_functional(schema)


# should be torch._C.JitType but that annotation is busted
def is_tensorlist_like_type(typ: Any) -> bool:
    return (
        typ == _C.ListType(_C.TensorType.get())
        or typ == _C.ListType(_C.OptionalType(_C.TensorType.get()))
        or typ == _C.OptionalType(_C.ListType(_C.TensorType.get()))
        or typ == _C.OptionalType(_C.ListType(_C.OptionalType(_C.TensorType.get())))
    )


# should be torch._C.JitType but that annotation is busted
def is_tensor_like_type(typ: Any) -> bool:
    return typ == _C.TensorType.get() or typ == _C.OptionalType(_C.TensorType.get())


def mutates_and_returns_first_arg(op: OpOverload):
    """Check if an op is an inplace aten op, i.e. it mutates and returns the first arg.
````
- **EN**: This chunk defines `mutates_and_returns_first_arg`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `mutates_and_returns_first_arg`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 158-185 / 第 158-185 行
````python
    TODO: torchgen/model.py's FunctionSchema.parse is the source of truth for this,
    but not all PyTorch builds have torchgen (due to the yaml dependency being weird).
    Figure this out.

    Example: add_(Tensor(a!) x, Tensor y) -> Tensor(a)
    """
    if op.namespace != "aten":
        return False
    schema = op._schema
    if len(schema.returns) != 1:
        return False
    if schema.returns[0].alias_info is None:
        return False
    alias_set = schema.returns[0].alias_info.after_set
    if len(alias_set) != 1:
        return False
    loc = next(iter(alias_set))
    if len(schema.arguments) < 1:
        return False
    first_arg = schema.arguments[0]
    if first_arg.alias_info is None:
        return False
    if not first_arg.alias_info.is_write:
        return False
    alias_set = first_arg.alias_info.after_set
    if len(alias_set) != 1:
        return False
    if loc != next(iter(alias_set)):
````
- **EN**: This chunk continues `mutates_and_returns_first_arg` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `mutates_and_returns_first_arg`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 186-208 / 第 186-208 行
````python
        return False
    for arg in schema.arguments[1:]:
        if arg.alias_info is not None:
            return False
    return True


def fill_defaults(schema, args, kwargs):
    new_args = []
    new_kwargs = {}
    for i in range(len(schema.arguments)):
        info = schema.arguments[i]
        if info.kwarg_only:
            if info.name in kwargs:
                new_kwargs[info.name] = kwargs[info.name]
            else:
                new_kwargs[info.name] = info.default_value
        else:
            if i < len(args):
                new_args.append(args[i])
            else:
                new_args.append(info.default_value)
    return tuple(new_args), new_kwargs
````
- **EN**: This chunk defines `fill_defaults`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `fill_defaults`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 211-237 / 第 211-237 行
````python
def zip_schema(
    schema: _C.FunctionSchema, args: tuple[Any, ...], kwargs: dict[str, Any]
) -> Iterable[tuple[_C.Argument, Any]]:
    """zips schema.arguments and (args, kwargs) together.

    Assumes that (args, kwargs) were the inputs to some torch._ops.OpOverload:
    that is, (args, kwargs) must be bindable to the schema (args, kwargs).
    """
    if len(schema.arguments) < len(args) + len(kwargs):
        raise AssertionError(
            f"schema has {len(schema.arguments)} arguments but got {len(args)} args and {len(kwargs)} kwargs"
        )
    for i in range(len(schema.arguments)):
        info = schema.arguments[i]
        if info.kwarg_only:
            if info.name in kwargs:
                yield info, kwargs[info.name]
            continue
        if i >= len(args):
            if not info.kwarg_only and info.name in kwargs:
                yield info, kwargs[info.name]
            # args that are equal to their default values are not populated
            # if they are followed by args that are equal to their defaults.
            # Skip these.
            continue
        yield info, args[i]
    return
````
- **EN**: This chunk defines `zip_schema`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `zip_schema`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 240-266 / 第 240-266 行
````python
def hop_schema_from_fx_node(node):
    from torchgen.gen_schema_utils import FunctionSchemaGen

    hop = node.target
    if not isinstance(hop, torch._ops.HigherOrderOperator):
        raise RuntimeError("fx_node's target must be a hop.")

    def _collect_example_val(node):
        meta_val = node.meta.get("val", None)
        if meta_val is None:
            if node.op != "get_attr":
                raise AssertionError(
                    f"node.op must be 'get_attr' when val is None, got {node.op!r}"
                )
            meta_val = getattr(node.graph.owning_module, node.target)
        return meta_val

    example_inputs = []
    for arg in node.args:
        if isinstance(arg, (torch.fx.Node, torch.fx.node.Node)):
            example_inputs.append(_collect_example_val(arg))
        elif isinstance(
            arg, (torch.fx.immutable_collections.immutable_list, list, tuple)
        ):
            example_inputs.append([_collect_example_val(x) for x in arg])
        else:
            raise RuntimeError(f"Unsupported arg type {type(arg)}")
````
- **EN**: This block assembles module dependencies, pulling in other helper packages such as torchgen.gen_schema_utils. This chunk defines `_collect_example_val`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了其他辅助包，如 torchgen.gen_schema_utils。 这一段定义了 `_collect_example_val`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 268-295 / 第 268-295 行
````python
    # Bound the arguments to make sure number of inputs are correct
    bound_args: inspect.BoundArguments = inspect.signature(hop.__call__).bind(
        *example_inputs
    )

    # We treat example_output as a single value in return. This is to differentiate 1. return a single val
    # vs 2. return a tuple with one element.
    example_output = _collect_example_val(node)
    return FunctionSchemaGen.from_example(
        hop._name, tuple(bound_args.arguments.items()), (list(example_output),)
    )


def can_generate_trivial_fake_impl(op: OpOverload) -> bool:
    if not isinstance(op, OpOverload):
        raise AssertionError(f"op must be OpOverload, got {type(op)}")
    if is_builtin(op):
        # We control the built-ins. These may (in rare cases)
        # do input metadata mutation (which we have banned on custom ops)
        return False
    schema = op._schema
    # It's suspicious if the op is not mutable but returns nothing, so we return False out of an abundance of caution
    if not schema.is_mutable:
        return False
    if len(schema.returns) > 0:
        return False
    # If the op returns nothing, then it has a trivial fake impl.
    return True
````
- **EN**: This chunk defines `can_generate_trivial_fake_impl`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `can_generate_trivial_fake_impl`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 298-323 / 第 298-323 行
````python
def requires_set_python_module() -> bool:
    """If an op was defined in C++ and extended from Python using the
    torch.library APIs, returns if we require that there have been a
    m.set_python_module("mylib.ops") call from C++ that associates
    the C++ op with a python module.
    """
    return getattr(_utils_internal, "REQUIRES_SET_PYTHON_MODULE", True)


def handle_dispatch_mode(curr_mode, op_overload, *args, **kwargs):
    if not isinstance(curr_mode, torch.utils._python_dispatch.TorchDispatchMode):
        raise AssertionError(
            f"curr_mode must be TorchDispatchMode, got {type(curr_mode)}"
        )
    args_flattened, _ = torch.utils._pytree.tree_flatten((args, kwargs.values()))
    # TODO: need to double check the semantics of the "types" argument to torch_dispatch.
    # It's generated in PyInterpreter.cpp, but seems to be generated in two places,
    # where in one case we only include tensors with the python key, and in another
    # we include **all** tensors.
    overload_types = [
        type(a)
        for a in args_flattened
        if isinstance(a, torch.Tensor)
        and torch._C._dispatch_keys(a).has(torch._C.DispatchKey.Python)
    ]
    # TODO: check that I got these args correct (in C++, we pass in "0000"??)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `handle_dispatch_mode`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `handle_dispatch_mode`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 325-350 / 第 325-350 行
````python
    return curr_mode.__torch_dispatch__(op_overload, overload_types, args, kwargs)


def has_kwarg_only_args(schema: _C.FunctionSchema):
    return any(a.kwarg_only for a in schema.arguments)


def has_kwarg_only_tensors(schema: _C.FunctionSchema):
    for a in schema.arguments:
        if not (is_tensor_like_type(a.type) or is_tensorlist_like_type(a.type)):
            continue
        if not a.kwarg_only:
            continue
        return True
    return False


def has_tensor_arg(schema: _C.FunctionSchema) -> bool:
    """
    Given a schema, returns True if the schema has a Tensor arg.
    A Tensor arg is any arg with a type annotation that might involve Tensor.
    """
    return any(
        (is_tensor_like_type(a.type) or is_tensorlist_like_type(a.type))
        for a in schema.arguments
    )
````
- **EN**: This chunk defines `has_tensor_arg`, which checks a capability or invariant before later code relies on it. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `has_tensor_arg`，其作用是检查某项能力或不变量，供后续逻辑依赖。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 353-376 / 第 353-376 行
````python
def get_device_arg_index(schema: _C.FunctionSchema) -> int | None:
    """
    Given a schema, returns the id of the `device: torch.device` argument.
    If it does not exist, returns None.
    """
    for index, arg in enumerate(schema.arguments):
        if arg.type is _C.DeviceObjType.get() and arg.name == "device":
            return index
    return None


def iter_tensors(
    args: tuple[Any], kwargs: dict[str, Any], allowed_nesting: int = 1
) -> Iterator[torch.Tensor]:
    def check(arg):
        if isinstance(arg, torch.Tensor):
            yield arg
        elif allowed_nesting > 0 and isinstance(arg, (tuple, list)):
            yield from iter_tensors(tuple(arg), {}, allowed_nesting - 1)

    for arg in args:
        yield from check(arg)
    for kwarg in kwargs.values():
        yield from check(kwarg)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `check`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `check`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 379-402 / 第 379-402 行
````python
def check_aliasing_constraint(name, prev, result, get_module=lambda: "???"):
    """
    custom operators' outputs must not alias any inputs or other outputs.
    """
    storages = {t.untyped_storage()._cdata for t in prev if isinstance(t, torch.Tensor)}
    tuple_result = result
    if not isinstance(result, tuple):
        tuple_result = (result,)
    for tensor in iter_tensors(tuple_result, {}):
        key = tensor.untyped_storage()._cdata
        if tensor.untyped_storage()._cdata in storages:
            raise RuntimeError(
                f"{name} (with implementation in {get_module()}): "
                f"The output of this custom operator (1) must not "
                f"also be an input to this custom operator and "
                f"(2) may not alias any inputs to this custom operator "
                f"or other returns. "
                f"The most common way to trigger this error is if "
                f"we have y = custom_op(x) and y and x are the same Tensor. "
                f"Please instead return a clone of the offending output "
                f"tensor(s) (e.g. return x.clone()) or refactor the custom "
                f"operator to not return y."
            )
        storages.add(key)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `check_aliasing_constraint`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `check_aliasing_constraint`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 405-427 / 第 405-427 行
````python
def _c_check_aliasing_constraint(name, args, kwargs, result, get_module=lambda: "???"):
    """
    custom operators' outputs must not have any aliases
    This version uses C++ implementation for perf.
    Only List container is supported.
    Tensors in Lists with not only Tensors are checked.
    """
    tuple_result = result
    if not isinstance(result, tuple):
        tuple_result = (result,)
    if _C._any_output_is_alias_to_input_or_output(args, kwargs, tuple_result):
        raise RuntimeError(
            f"{name} (with implementation in {get_module()}): "
            f"The output of this custom operator (1) must not "
            f"also be an input to this custom operator and "
            f"(2) may not alias any inputs to this custom operator "
            f"or other returns. "
            f"The most common way to trigger this error is if "
            f"we have y = custom_op(x) and y and x are the same Tensor. "
            f"Please instead return a clone of the offending output "
            f"tensor(s) (e.g. return x.clone()) or refactor the custom "
            f"operator to not return y."
        )
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_c_check_aliasing_constraint`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_c_check_aliasing_constraint`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 430-457 / 第 430-457 行
````python
class MutationChecker:
    """
    Check if an operator mutated its arguments.
    Usage:

    checker = MutationChecker(op, flat_args, args_spec)
    op(*args, **kwargs)
    checker.check()
    """

    def __init__(self, op, flat_args, args_spec):
        self.op = op
        self.args_spec = args_spec
        self.flat_args = flat_args
        self.real_pre_hashes = [
            hash_tensor(a) if isinstance(a, torch.Tensor) else None for a in flat_args
        ]

    def check(self):
        real_post_hashes = [
            hash_tensor(a) if isinstance(a, torch.Tensor) else None
            for a in self.flat_args
        ]
        was_mutated = [
            not torch.equal(pre, post)
            and not (pre.isnan().all() and post.isnan().all())
            if isinstance(pre, torch.Tensor) and isinstance(post, torch.Tensor)
            else None
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `MutationChecker`, which hold the main object-oriented state for this portion of the file. This chunk defines `check`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `MutationChecker`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `check`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 458-479 / 第 458-479 行
````python
            for pre, post in zip(self.real_pre_hashes, real_post_hashes)
        ]
        was_mutated_args, was_mutated_kwargs = pytree.tree_unflatten(
            was_mutated, self.args_spec
        )
        for info, was_mutated in zip_schema(
            self.op._schema, was_mutated_args, was_mutated_kwargs
        ):

            def check_one(info, was_mutated):
                if info.is_write == was_mutated:
                    return
                raise RuntimeError(
                    f"{self.op._name}: for argument '{info.name}': the operator's schema "
                    f"{self.op._schema} specified that "
                    f"the operator {'mutates' if info.is_write else 'does not mutate'} "
                    f"the argument, but this seems to be empirically wrong. "
                    f"Please make the schema and operator behavior consistent. "
                    f"You can specify that an operator mutates a Tensor by "
                    f"e.g. changing its schema type from 'Tensor name' to 'Tensor(a!) name'"
                    f"(use different identifiers (a, b, c, ...) for different Tensors)"
                )
````
- **EN**: This chunk defines `check_one`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `check_one`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 481-508 / 第 481-508 行
````python
            if is_tensor_like_type(info.type):
                check_one(info, was_mutated)
            elif is_tensorlist_like_type(info.type):
                was_any_mutated = False if was_mutated is None else any(was_mutated)
                check_one(info, was_any_mutated)


def hash_tensor(t: torch.Tensor) -> torch.Tensor:
    """Some inexpensive hash. Used as a quick and dirty indicator for tensor mutation"""
    return t.detach().float().mean()


def has_fake_kernel(op: torch._ops.OpOverload) -> bool:
    """If an operator (that stays alive until FakeTensorMode) has a Fake kernel.
    Don't use this if the operator decomposes before FakeTensorMode.
    """
    if can_generate_trivial_fake_impl(op):
        return True
    name = op._name
    if torch._C._dispatch_has_kernel_for_dispatch_key(
        name, "CompositeImplicitAutograd"
    ):
        return True
    opdef = torch._library.custom_ops._maybe_get_opdef(name)
    if opdef is None:
        # the non-torch.library.custom_op path
        if torch._C._dispatch_has_kernel_for_dispatch_key(
            name, "CompositeExplicitAutograd"
````
- **EN**: This chunk defines `has_fake_kernel`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `has_fake_kernel`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 509-532 / 第 509-532 行
````python
        ):
            return True
        entry = torch._library.simple_registry.singleton.find(name)
        if entry.fake_impl.kernel is not None:
            return True
        if torch._C._dispatch_has_kernel_for_dispatch_key(name, "Meta"):
            return True
    else:
        # the torch.library.custom_op path
        if opdef._abstract_fn is not None:
            return True
    return False


def mutated_args_kwargs(schema: _C.FunctionSchema) -> tuple[list[int], list[str]]:
    idxs = []
    keys = []
    for i, info in enumerate(schema.arguments):
        if info.alias_info is not None and info.alias_info.is_write:
            if info.kwarg_only:
                keys.append(info.name)
            else:
                idxs.append(i)
    return idxs, keys
````
- **EN**: This chunk defines `mutated_args_kwargs`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `mutated_args_kwargs`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 535-554 / 第 535-554 行
````python
tags_by_priority = [
    _C.Tag.needs_exact_strides,
    _C.Tag.needs_contiguous_strides,
    _C.Tag.needs_fixed_stride_order,
    _C.Tag.flexible_layout,
]


# Case 1: with_default=True (or omitted). Return type is guaranteed to be a Tag.
@overload
def get_layout_constraint_tag(
    fn: Any, *, with_default: Literal[True] = True
) -> _C.Tag: ...


# Case 2: with_default=False. Return type can be a Tag or None.
@overload
def get_layout_constraint_tag(
    fn: Any, *, with_default: Literal[False]
) -> _C.Tag | None: ...
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk defines `get_layout_constraint_tag`, which retrieves runtime state and exposes it through a Python-friendly accessor. Decorators such as `overload` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段定义了 `get_layout_constraint_tag`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 像 `overload` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 557-584 / 第 557-584 行
````python
def get_layout_constraint_tag(fn, *, with_default=True):
    for tag in tags_by_priority:
        if tag in fn.tags:
            return tag
    if with_default:
        if is_builtin(fn):
            return _C.Tag.flexible_layout
        import torch._functorch
        from torch._functorch import config

        return getattr(torch._C.Tag, config.custom_op_default_layout_constraint)
    return None


# List of random functions that should be treated as impure
_RANDOM_FUNCTIONS = {
    torch.rand,
    torch.randn,
    torch.randint,
    torch.randperm,
    torch.rand_like,
    torch.randn_like,
    torch.randint_like,
    torch.normal,
    torch.poisson,
    torch.bernoulli,
    torch.multinomial,
}
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._functorch. This chunk defines `get_layout_constraint_tag`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._functorch。 这一段定义了 `get_layout_constraint_tag`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 587-611 / 第 587-611 行
````python
def is_impure(
    op: Callable,
    *,
    args: tuple[Any, ...] | None = None,
    kwargs: dict[str, Any] | None = None,
    impure_random: bool = True,
) -> bool:
    """
    An operator is impure if it:
    - Mutates its inputs (has a mutable schema)
    - Has nondeterministic/random behavior that mutates RNG state
    - Is explicitly marked as effectful via torch.library._register_effectful_op

    Args:
        op: The operator to check (function, OpOverload, HigherOrderOperator, etc.)
        args: Optional arguments that would be passed to the callable
        kwargs: Optional keyword arguments that would be passed to the callable
        impure_random: Whether to treat random operations as impure (default: True)

    Returns:
        bool: True if the callable has side effects, False otherwise
    """
    # Import here to avoid circular dependencies
    from torch._higher_order_ops.effects import _get_effect
    from torch.fx.node import _side_effectful_functions
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._higher_order_ops.effects, torch.fx.node. This chunk defines `is_impure`, which checks a capability or invariant before later code relies on it.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._higher_order_ops.effects、torch.fx.node。 这一段定义了 `is_impure`，其作用是检查某项能力或不变量，供后续逻辑依赖。

### Lines 613-638 / 第 613-638 行
````python
    if isinstance(op, torch._ops.OpOverload):
        schema = getattr(op, "_schema", None)
        if schema is not None and schema.is_mutable:
            return True

        if op in _side_effectful_functions:
            return True

        if _get_effect(op) is not None:
            return True

    if isinstance(op, torch._ops.HigherOrderOperator):
        if op in (
            torch.ops.higher_order.auto_functionalized,
            torch.ops.higher_order.auto_functionalized_v2,
        ):
            # Check if the auto-functionalized operator (the first argument) is
            # side-effectful
            if args and len(args) > 0:
                return args[0] in _side_effectful_functions

        if _get_effect(op) is not None:
            return True

        if op in _side_effectful_functions:
            return True
````
- **EN**: This chunk continues `is_impure` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `is_impure`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 640-662 / 第 640-662 行
````python
        return False

    # Impure since it mutates RNG state
    if impure_random and getattr(op, "_nondeterministic_seeded", False):
        return True

    # Handle Python random functions that don't have _nondeterministic_seeded
    # but still affect global RNG state (issue #151524)
    # These should be impure regardless of impure_random setting to maintain
    # consistency between eager and compiled execution
    # All random operations are impure to ensure consistent behavior
    # between eager and compiled execution, regardless of generator usage
    if op in _RANDOM_FUNCTIONS:
        return True

    schema = getattr(op, "_schema", None)
    if schema is not None and schema.is_mutable:
        return True

    if op in _side_effectful_functions:
        return True

    return False
````
- **EN**: This chunk continues `is_impure` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `is_impure`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Library registration**
  - EN: Builds Python-side operator/library registrations that feed the dispatcher.
  - CN: 构建 Python 侧的算子/库注册，并将其接入 dispatcher。
- **Kernel**
  - EN: `Kernel` is one of the main symbols declared or implemented in this file.
  - CN: `Kernel` 是本文件声明或实现的主要符号之一。
- **RegistrationHandle**
  - EN: `RegistrationHandle` is one of the main symbols declared or implemented in this file.
  - CN: `RegistrationHandle` 是本文件声明或实现的主要符号之一。
- **Deferred execution**
  - EN: The file records intent or metadata now so execution can be materialized later.
  - CN: 该文件先记录意图或元数据，等待后续阶段再真正执行。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.utils._pytree`, `torch._ops`, `torch._functorch`, `torch._higher_order_ops.effects`, `torch.fx.node`
- **Standard library / 标准库**: `dataclasses`, `inspect`, `sys`, `collections.abc`, `typing`
- **Other helper packages / 其他辅助包**: `torchgen`, `torchgen.model`, `torchgen.gen_schema_utils`
- **Primary symbols in this file / 本文件核心符号**: `Kernel`, `RegistrationHandle`, `get_source`, `parse_namespace`, `lookup_op`, `is_builtin`, `is_functional_schema`, `is_tensorlist_like_type`, `is_tensor_like_type`, `mutates_and_returns_first_arg`
