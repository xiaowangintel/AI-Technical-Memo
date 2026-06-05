# autograd.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_library/autograd.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python helpers for torch.library registration, library fragments, and operator definitions.
- **Purpose (CN)**: 实现 torch.library 注册、库片段以及算子定义相关的 Python 辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
````python
# mypy: allow-untyped-defs
import dataclasses
from collections.abc import Callable
from dataclasses import dataclass
from typing import Any, Protocol

from torch import _C, _ops, autograd, Tensor
from torch.utils import _pytree

from . import utils


class InfoProtocol(Protocol):
    _backward_fn: Callable | None
    _setup_context_fn: Callable | None
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.utils, .; standard-library helpers such as dataclasses, collections.abc, typing. It introduces or extends `InfoProtocol`, which hold the main object-oriented state for this portion of the file.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.utils、.；标准库辅助模块，如 dataclasses、collections.abc、typing。 它引入或扩展了 `InfoProtocol`，这些类承载了本段涉及的主要面向对象状态。

### Lines 18-36 / 第 18-36 行
````python
@dataclasses.dataclass
class Info:
    _backward_fn: Callable | None
    _setup_context_fn: Callable | None


def make_autograd_impl(op: _ops.OpOverload, info: InfoProtocol) -> Callable:
    name: str = f"GeneratedBackwardFor_{op._namespace}_{op._opname}_{op._overloadname}"

    has_kwarg_only_args = utils.has_kwarg_only_args(op._schema)

    @dataclass
    class Metadata:
        keyset: _C.DispatchKeySet
        keyword_only_args: dict[str, Any]

    def forward_no_grad(*args):
        metadata = args[-1]
        args = args[:-1]
````
- **EN**: It introduces or extends `Info`, `Metadata`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward_no_grad`, which implements a focused helper used by the surrounding module. Decorators such as `dataclasses.dataclass`, `dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 它引入或扩展了 `Info`、`Metadata`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward_no_grad`，其作用是实现周边模块使用的关键辅助逻辑。 像 `dataclasses.dataclass`、`dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 38-57 / 第 38-57 行
````python
        with _C._AutoDispatchBelowAutograd():
            keyset = metadata.keyset
            kwargs = metadata.keyword_only_args
            result = op.redispatch(keyset & _C._after_autograd_keyset, *args, **kwargs)
            return result

    def forward(ctx, *args):
        metadata = args[-1]
        args = args[:-1]

        with _C._AutoDispatchBelowAutograd():
            keyset = metadata.keyset
            kwargs = metadata.keyword_only_args
            result = op.redispatch(keyset & _C._after_autograd_keyset, *args, **kwargs)
            if info._setup_context_fn:
                # The Dispatcher will remove args that are equal to their default
                # values from (args, kwargs). We're going to add it back so that
                # the user can access them.
                #
                # This is OK to do: The Dispatcher removed the args for serialization
````
- **EN**: This chunk defines `forward`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 58-71 / 第 58-71 行
````python
                # FC/BC reasons (that is, a graph will not store args that are equal
                # to their default values), but that doesn't matter here. If the user
                # adds a new default arg, then they must update
                # their setup_context (along with the rest of their operator
                # registrations)
                args, kwargs = utils.fill_defaults(op._schema, args, kwargs)

                if has_kwarg_only_args:
                    info._setup_context_fn(
                        ctx=ctx, inputs=args, keyword_only_inputs=kwargs, output=result
                    )
                else:
                    info._setup_context_fn(ctx=ctx, inputs=args, output=result)
            return result
````
- **EN**: This chunk continues `forward` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `forward`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 73-88 / 第 73-88 行
````python
    def backward(ctx, *grads):
        if info._backward_fn:
            try:
                prev_needs_input_grad = ctx.needs_input_grad
                ctx.needs_input_grad = ctx.needs_input_grad[:-1]
                result = info._backward_fn(ctx, *grads)
            finally:
                ctx.needs_input_grad = prev_needs_input_grad
            if isinstance(result, tuple):
                return (*result, None)
            return result, None
        raise RuntimeError(
            f"Trying to backward through {op} but no autograd "
            f"formula was registered. "
            f"Please use register_autograd to add one."
        )
````
- **EN**: This chunk defines `backward`, which implements differentiation-time behavior that complements the forward path. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `backward`，其作用是实现与前向路径配套的求导期行为。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 90-104 / 第 90-104 行
````python
    Generated = type(
        name,
        (autograd.Function,),
        {
            "forward": staticmethod(forward),
            "backward": staticmethod(backward),
        },
    )

    schema = op._schema
    if any(
        utils.is_tensorlist_like_type(a.type)
        for a in (*schema.arguments, *schema.returns)
    ):
        Generated = supports_tensorlist(Generated)
````
- **EN**: This chunk continues `backward` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `backward`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 106-119 / 第 106-119 行
````python
    # The dispatcher passes any keyword-only-args as kwargs and the
    # rest of the args (even if specified as kwargs) as args.
    def autograd_impl(keyset, *args, **keyword_only_args):
        if _C.is_grad_enabled() and _C._any_requires_grad(*args):
            result = Generated.apply(*args, Metadata(keyset, keyword_only_args))  # type: ignore[attr-defined]
        else:
            result = forward_no_grad(*args, Metadata(keyset, keyword_only_args))
        return result

    return autograd_impl


def supports_tensorlist(cls: Any) -> Any:
    """Allows a given autograd.Function class to support List[Tensor] inputs/outputs.
````
- **EN**: This chunk defines `supports_tensorlist`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `supports_tensorlist`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 121-133 / 第 121-133 行
````python
    Regular autograd.Function has a constraint that it only directly supports autograd for
    Tensors. Applying @supports_tensorlist enables an autograd.Function to support
    autograd for List[Tensor] inputs and outputs.
    """
    orig_forward = cls.forward
    orig_backward = cls.backward
    orig_apply = cls.apply

    @dataclass
    class Metadata:
        input_spec: _pytree.TreeSpec
        output_spec: _pytree.TreeSpec | None = None
        result_is_tuple: bool | None = None
````
- **EN**: It introduces or extends `Metadata`, which hold the main object-oriented state for this portion of the file. This chunk continues `Metadata` and expands its internal control flow or state updates. Decorators such as `dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `Metadata`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `Metadata`，进一步展开其内部控制流或状态更新。 像 `dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 135-150 / 第 135-150 行
````python
    def new_forward(ctx, *args):
        metadata = args[-1]
        args = args[:-1]
        if not isinstance(metadata, Metadata):
            raise NotImplementedError(
                "NYI: calling supports_tensorlist autograd.Function.forward directly. "
                "You should probably be calling .apply instead. "
                "Please file an issue if not."
            )
        args = _pytree.tree_unflatten(list(args), metadata.input_spec)
        result = orig_forward(ctx, *args)
        metadata.result_is_tuple = isinstance(result, tuple)
        if not metadata.result_is_tuple:
            result = (result,)
        flat_result, output_spec = _pytree.tree_flatten(result, not_list_of_tensor)
        metadata.output_spec = output_spec
````
- **EN**: This chunk defines `new_forward`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `new_forward`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 152-169 / 第 152-169 行
````python
        if hasattr(ctx, "_pt_metadata"):
            raise RuntimeError(
                "Please don't set ctx._pt_metadata; PyTorch uses it to store info"
            )
        ctx._pt_metadata = metadata

        return tuple(flat_result)

    def new_backward(ctx, *grads):
        if not hasattr(ctx, "_pt_metadata"):
            raise NotImplementedError(
                "NYI: calling supports_tensorlist autograd.Function.backward directly. "
                "This will automatically get called by PyTorch autograd. "
                "Please file an issue if you need this."
            )

        metadata = ctx._pt_metadata
        grads = _pytree.tree_unflatten(list(grads), metadata.output_spec)
````
- **EN**: This chunk defines `new_backward`, which implements differentiation-time behavior that complements the forward path. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `new_backward`，其作用是实现与前向路径配套的求导期行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 171-184 / 第 171-184 行
````python
        # If the user's input is ([x, y, z], w),
        # then needs_input_grad is (bool, bool, bool, bool, bool).
        # We need to
        # 1. get rid of the additional bool (which comes from the extra
        # `metadata input`)
        # 2. _pytree.tree_unflatten to get the right structure.
        prev_needs_input_grad = ctx.needs_input_grad
        try:
            ctx.needs_input_grad = _pytree.tree_unflatten(
                list(ctx.needs_input_grad[:-1]), metadata.input_spec
            )
            grad_inputs = orig_backward(ctx, *grads)
        finally:
            ctx.needs_input_grad = prev_needs_input_grad
````
- **EN**: This chunk continues `new_backward` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior.
- **CN**: 这一段延续了 `new_backward`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。

### Lines 186-202 / 第 186-202 行
````python
        if not isinstance(grad_inputs, tuple):
            grad_inputs = (grad_inputs,)
        # Assume that any Nones in the backward are Tensors.
        # If the forward has an arg that is [1, 2, 3], the backward should
        # return None as the grad.
        # If the forward has an arg that is [tensor, tensor], the backward
        # may return [None, None], [grad, None], [None, grad], or [grad, grad].
        flat_grad_inputs, grad_inputs_spec = _pytree.tree_flatten(
            grad_inputs, not_list_of_optional_tensor
        )
        if grad_inputs_spec != metadata.input_spec:
            raise RuntimeError(
                f"Expected the return from backward to be of the same structure "
                f"as the inputs. Got: {grad_inputs_spec} (return from backward), "
                f"{metadata.input_spec} (inputs)"
            )
        return tuple(flat_grad_inputs + [None])
````
- **EN**: This chunk continues `new_backward` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `new_backward`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 204-219 / 第 204-219 行
````python
    def new_apply(*args):
        flat_args, input_spec = _pytree.tree_flatten(args, is_leaf=not_list_of_tensor)
        metadata = Metadata(input_spec)
        result = orig_apply(*flat_args, metadata)  # type: ignore[misc]
        if metadata.output_spec is None:
            raise AssertionError("metadata.output_spec must not be None")
        result = _pytree.tree_unflatten(list(result), metadata.output_spec)
        if not metadata.result_is_tuple:
            if not isinstance(result, tuple):
                raise AssertionError(f"result must be tuple, got {type(result)}")
            if len(result) != 1:
                raise AssertionError(
                    f"result tuple must have length 1, got {len(result)}"
                )
            return result[0]
        return result
````
- **EN**: This chunk defines `new_apply`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `new_apply`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 221-240 / 第 221-240 行
````python
    cls.forward = new_forward
    cls.backward = new_backward
    cls.apply = new_apply
    return cls


def not_list_of_tensor(tree):
    if isinstance(tree, tuple):
        return False
    if isinstance(tree, list):
        return any(not isinstance(l, Tensor) for l in tree)
    return True


def not_list_of_optional_tensor(tree):
    if isinstance(tree, tuple):
        return False
    if isinstance(tree, list):
        return any(l is not None and not isinstance(l, Tensor) for l in tree)
    return True
````
- **EN**: This chunk defines `not_list_of_optional_tensor`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `not_list_of_optional_tensor`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Library registration**
  - EN: Builds Python-side operator/library registrations that feed the dispatcher.
  - CN: 构建 Python 侧的算子/库注册，并将其接入 dispatcher。
- **InfoProtocol**
  - EN: `InfoProtocol` is one of the main symbols declared or implemented in this file.
  - CN: `InfoProtocol` 是本文件声明或实现的主要符号之一。
- **Info**
  - EN: `Info` is one of the main symbols declared or implemented in this file.
  - CN: `Info` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.utils`, `.`
- **Standard library / 标准库**: `dataclasses`, `collections.abc`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `InfoProtocol`, `Info`, `make_autograd_impl`, `supports_tensorlist`, `not_list_of_tensor`, `not_list_of_optional_tensor`
