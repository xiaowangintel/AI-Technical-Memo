# fake_impls.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_subclasses/fake_impls.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements tensor-subclass helpers, fake tensor support, and subclass-aware dispatch utilities.
- **Purpose (CN)**: 实现张量子类辅助逻辑、fake tensor 支持以及面向子类的 dispatch 工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-36 / 第 1-36 行
````python
from __future__ import annotations

import functools
import itertools
import math
import operator
import sys
from functools import reduce
from typing import Any, cast as typing_cast, TYPE_CHECKING, TypeVar
from typing_extensions import ParamSpec

import torch
import torch._custom_op
import torch._logging
import torch._prims_common as utils
from torch._dispatch.python import no_python_dispatcher
from torch._ops import OpOverload
from torch._prims_common import (
    canonicalize_dim,
    elementwise_dtypes,
    ELEMENTWISE_TYPE_PROMOTION_KIND,
    is_boolean_dtype,
    is_contiguous,
    is_contiguous_for_memory_format_or_false,
    is_contiguous_or_false,
    is_float_dtype,
    is_integer_dtype,
    make_contiguous_strides_for,
    ShapeType,
)
from torch._subclasses.fake_tensor import (
    DataDependentOutputException,
    DynamicOutputShapeException,
    FakeTensor,
    in_kernel_invocation_manager,
    run_fallback_kernel,
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._custom_op, torch._logging, ...; standard-library helpers such as __future__, functools, itertools, ...; other helper packages such as typing_extensions. The future import postpones annotation evaluation, keeping type hints lightweight at import time.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._custom_op、torch._logging、...；标准库辅助模块，如 __future__、functools、itertools、...；其他辅助包，如 typing_extensions。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。

### Lines 37-70 / 第 37-70 行
````python
    UnsupportedOperatorException,
)
from torch.fx.operator_schemas import _normalize_function_or_error
from torch.utils._stats import count_label


if TYPE_CHECKING:
    from collections.abc import Callable, Sequence

    from torch._subclasses.fake_tensor import FakeTensorMode
    from torch.types import IntLikeType


FakeTensorLike = FakeTensor | torch.Tensor
_P = ParamSpec("_P")
_R = TypeVar("_R")
_T = TypeVar("_T")

pytree = torch.utils._pytree

__all__ = [
    "op_implementations_checks",
    "get_fast_op_impls",
    "stride_incorrect_op",
    "has_meta",
]

# pyrefly: ignore [implicit-any]
op_implementations_dict = {}
# pyrefly: ignore [implicit-any]
op_implementations_checks = []


aten = torch._ops.ops.aten
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.operator_schemas, torch.utils._stats, torch._subclasses.fake_tensor, ...; standard-library helpers such as collections.abc. `__all__` defines the public symbols that this module chooses to export. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.operator_schemas、torch.utils._stats、torch._subclasses.fake_tensor、...；标准库辅助模块，如 collections.abc。 `__all__` 定义了本模块选择导出的公共符号。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 73-108 / 第 73-108 行
````python
def ordered_set(*items: _T) -> dict[_T, bool]:
    return dict.fromkeys(items, True)


# This function indicates if the backend device
# supports non-contiguous tensors
def is_noncontiguous_supported(device: torch.device) -> bool:
    return device.type != "hpu"


_like_tensor_constructors = ordered_set(
    aten.empty_like.default,
    aten.empty_like.out,
    aten.full_like.default,
    aten.full_like.out,
    aten.ones_like.default,
    aten.ones_like.out,
    aten.rand_like.default,
    aten.rand_like.generator,
    aten.rand_like.out,
    aten.rand_like.generator_out,
    aten.randn_like.default,
    aten.randn_like.generator,
    aten.randn_like.out,
    aten.randn_like.generator_out,
    aten.randint_like.default,
    aten.randint_like.generator,
    aten.randint_like.Tensor,
    aten.randint_like.Tensor_generator,
    aten.randint_like.Tensor_out,
    aten.randint_like.Tensor_generator_out,
    aten.randint_like.out,
    aten.randint_like.generator_out,
    aten.randint_like.low_dtype,
    aten.randint_like.low_generator_dtype,
    aten.randint_like.low_dtype_out,
````
- **EN**: This chunk defines `is_noncontiguous_supported`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_noncontiguous_supported`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 109-140 / 第 109-140 行
````python
    aten.randint_like.low_generator_dtype_out,
    aten.zeros_like.default,
    aten.zeros_like.out,
    aten.new_empty.default,
    aten.new_empty.out,
    aten.new_empty_strided.default,
    aten.new_empty_strided.out,
    aten.new_full.default,
    aten.new_full.out,
    aten.new_zeros.default,
    aten.new_zeros.out,
    aten.new_ones.default,
    aten.new_ones.out,
)


_device_not_kwarg_ops = ordered_set(
    aten._resize_output_.default,
    aten._nested_tensor_from_tensor_list.default,
    aten._nested_tensor_from_tensor_list.out,
    aten.pin_memory.default,
    aten.to.device,
    aten.to.prim_Device,
    aten.is_pinned.default,
    aten._pin_memory.default,
    aten._pin_memory.out,
    aten._resize_output.default,
    aten._resize_output.out,
)

# this op is never actually used
_non_kwarg_device_constructors = (aten._list_to_tensor,)
````
- **EN**: This chunk continues `is_noncontiguous_supported` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `is_noncontiguous_supported`，进一步展开其内部控制流或状态更新。

### Lines 143-178 / 第 143-178 行
````python
def contains_tensor_types(type_: Any) -> bool:
    tensor_type = torch._C.TensorType.get()
    return type_.isSubtypeOf(tensor_type) or any(
        contains_tensor_types(e) for e in type_.containedTypes()
    )


@functools.cache
def _is_tensor_constructor(func: OpOverload) -> bool:
    if not isinstance(func, OpOverload):
        raise AssertionError(f"func must be an OpOverload, got {type(func)}")
    schema = func._schema
    if any(contains_tensor_types(arg.type) for arg in schema.arguments):
        return False
    # TODO: no real reason to restrict multiple outputs
    return (
        len(schema.returns) == 1 and schema.returns[0].type is torch._C.TensorType.get()
    )


def register_op_impl(
    run_impl_check: Callable[[OpOverload], bool]
    | OpOverload
    | list[OpOverload]
    | tuple[OpOverload, ...],
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]]:
    def impl_decorator(op_impl: Callable[_P, _R]) -> Callable[_P, _R]:
        if isinstance(run_impl_check, OpOverload):
            if run_impl_check in op_implementations_dict:
                raise AssertionError(f"duplicate registration: {run_impl_check}")
            op_implementations_dict[run_impl_check] = op_impl
        elif isinstance(run_impl_check, (list, tuple)):
            for op in run_impl_check:
                register_op_impl(op)(op_impl)
        else:
            if not callable(run_impl_check):
````
- **EN**: This chunk defines `impl_decorator`, which implements a focused helper used by the surrounding module. Decorators such as `functools.cache` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_decorator`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.cache` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 179-205 / 第 179-205 行
````python
                raise AssertionError(
                    f"run_impl_check must be callable, got {type(run_impl_check)}"
                )
            op_implementations_checks.append((run_impl_check, op_impl))

        return op_impl

    return impl_decorator


def _is_op_registered_to_fake_rule(op: OpOverload) -> bool:
    return op in op_implementations_dict


def _deregister_op_impl(op: OpOverload) -> None:
    op_implementations_dict.pop(op, None)
    for check, impl in op_implementations_checks:
        if check is op:
            op_implementations_checks.remove((check, impl))
            break


@register_op_impl(op_implementations_dict.__contains__)
def dispatch_to_op_implementations_dict(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> Any:
    return op_implementations_dict[func](fake_mode, func, *args, **kwargs)
````
- **EN**: This chunk defines `dispatch_to_op_implementations_dict`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `dispatch_to_op_implementations_dict`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 208-239 / 第 208-239 行
````python
@register_op_impl(_is_tensor_constructor)
@register_op_impl([*_like_tensor_constructors])
def constructors(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    if func in _non_kwarg_device_constructors:
        raise AssertionError(
            f"func must not be in _non_kwarg_device_constructors, got {func}"
        )
    _, new_kwargs = _normalize_function_or_error(
        func, args=args, kwargs=kwargs, normalize_to_only_use_kwargs=True
    )
    if "names" in kwargs:
        # REASON: "torch.compile doesn't support named tensors"
        raise UnsupportedOperatorException(func)

    if func in _like_tensor_constructors:
        default_device = new_kwargs["input"].device
        # TODO: file issue
        args = (new_kwargs.pop("input"),)
    else:
        # cpu is default device if none is specified
        default_device = torch.device("cpu")
        args = ()
    out_device = new_kwargs.pop("device", None)
    out_device = out_device if out_device is not None else default_device
    new_kwargs["device"] = torch.device("meta")
    # _like constructors have fake tensor inputs (maybe this causes the non-like
    # to fail? hmmm)
    with in_kernel_invocation_manager(fake_mode):
        r = func(*args, **new_kwargs)
    return FakeTensor(fake_mode, r, out_device)
````
- **EN**: This chunk defines `constructors`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `constructors`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 242-269 / 第 242-269 行
````python
@register_op_impl(aten.is_pinned.default)
def non_kwarg_is_pinned(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> bool:
    _, new_kwargs = _normalize_function_or_error(
        func, args, kwargs, normalize_to_only_use_kwargs=True
    )
    inp = new_kwargs.pop("input")
    # we'll ignore device argument because it is deprecated and not
    # actually used by is_pinned.
    with in_kernel_invocation_manager(fake_mode):
        r = func(inp)
    return r


# Legacy profiler ops return Tensors but don't follow tensor constructor patterns
# They take string arguments and should not have device/dtype parameters added
@register_op_impl(torch.ops.profiler._record_function_enter.default)
def _record_function_enter(
    fake_mode: FakeTensorMode, func: OpOverload, name: str, args: object | None = None
) -> FakeTensor:
    # Call the real implementation to get a real handle tensor
    with in_kernel_invocation_manager(fake_mode):
        real_handle = func(name, args)
    # Create a meta tensor with the same properties as the real handle
    meta_handle = torch.empty_like(real_handle, device="meta")
    # Wrap it as a FakeTensor
    return FakeTensor(fake_mode, meta_handle, torch.device("cpu"))
````
- **EN**: This chunk defines `_record_function_enter`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_record_function_enter`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 272-307 / 第 272-307 行
````python
@register_op_impl(torch.ops.profiler._record_function_exit.default)
def _record_function_exit(
    fake_mode: FakeTensorMode, func: OpOverload, handle: Any
) -> None:
    # Exit doesn't return anything and doesn't need to do anything for fake tensors
    # Just return None (the actual return type is void)
    pass


@register_op_impl(torch.ops.profiler._record_function_enter_new.default)
def _record_function_enter_new(
    fake_mode: FakeTensorMode, func: OpOverload, name: str, args: object | None = None
) -> Any:
    # Call the real implementation - returns a custom class, not a tensor
    # Just pass through without wrapping
    with in_kernel_invocation_manager(fake_mode):
        return func(name, args)


@register_op_impl(aten.to.prim_Device)
@register_op_impl(aten.to.device)
def non_kwarg_to(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    _, new_kwargs = _normalize_function_or_error(
        func, args, kwargs, normalize_to_only_use_kwargs=True
    )
    input_device = new_kwargs["device"]
    out_device = input_device if input_device else new_kwargs["input"].device
    new_kwargs["device"] = torch.device("meta")
    inp = new_kwargs.pop("input")
    with in_kernel_invocation_manager(fake_mode):
        r = func(inp, **new_kwargs)
    # TODO: I think this does the wrong thing if r is inp
    return fake_mode.fake_tensor_converter.from_meta_and_device(
        fake_mode, r, out_device
````
- **EN**: This chunk defines `non_kwarg_to`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `non_kwarg_to`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 308-343 / 第 308-343 行
````python
    )


def stride_incorrect_op(op: OpOverload) -> bool:
    return False


# These operators have meta implementations with incorrect strides
@register_op_impl(stride_incorrect_op)
def workaround_stride_incorrect_op(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    # This is a workaround for meta implementations with incorrect strides

    def is_symbolic(x: object) -> bool:
        if isinstance(x, FakeTensor):
            return x._has_symbolic_sizes_strides
        if isinstance(x, (torch.SymInt, torch.SymFloat, torch.SymBool)):
            return True
        return False

    # For static shapes, we can fall back to eager for the real strides
    if fake_mode.allow_fallback_kernels:
        require_dynamic = any(
            is_symbolic(x) for x in itertools.chain(args, kwargs.values())
        )
        if not require_dynamic:
            flat_args, args_spec = pytree.tree_flatten((args, kwargs))
            return run_fallback_kernel(
                fake_mode,
                func,
                flat_args,
                args_spec,
                # TODO: refactor to lambda so we don't instantiate extra errors before
                # calling
                RuntimeError("Cannot run fallback kernel for stride_incorrect_op"),
````
- **EN**: This chunk defines `is_symbolic`, which checks a capability or invariant before later code relies on it. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_symbolic`，其作用是检查某项能力或不变量，供后续逻辑依赖。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 344-375 / 第 344-375 行
````python
            )

    raise UnsupportedOperatorException(func)


# Dont default to default device handling,
# since the device of `the_template` is ignored
@register_op_impl(aten.resize_as_.default)
def resize_as_(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    with in_kernel_invocation_manager(fake_mode):
        return func(*args, **kwargs)


@register_op_impl(aten._sparse_coo_tensor_with_dims_and_tensors.default)
def _sparse_coo_tensor_with_dims_and_tensors(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    return constructors(fake_mode, func, *args, **kwargs)


# index.Tensor data-dependent in only some conditions
@register_op_impl(
    lambda func: torch.Tag.dynamic_output_shape in func.tags
    and func
    not in [aten.index.Tensor, aten.nonzero.default, aten.repeat_interleave.Tensor]
)
def dyn_shape(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> None:
    raise DynamicOutputShapeException(func)
````
- **EN**: This chunk defines `dyn_shape`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `dyn_shape`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 378-404 / 第 378-404 行
````python
def _unique(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    arg: FakeTensor,
    dim: int | None,
    sorted: bool = True,
    return_inverse: bool = False,
    return_counts: bool = False,
    *,
    unique_consecutive: bool = False,
) -> tuple[FakeTensor, FakeTensor, FakeTensor]:
    if (
        fake_mode.shape_env is None
        or not fake_mode.shape_env.allow_dynamic_output_shape_ops
    ):
        # Without symints/symfloats, cannot handle this
        raise DynamicOutputShapeException(func)

    nnz = arg.unique_consecutive_memo if unique_consecutive else arg.unique_memo

    # Do not use a memo for unique_dim
    if dim is not None or nnz is None:
        # Avoid importing sympy at a module level
        from torch.fx.experimental.symbolic_shapes import (
            _constrain_range_for_size,
            has_free_symbols,
        )
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `_unique`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `_unique`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 406-438 / 第 406-438 行
````python
        if not has_free_symbols(arg.numel()) and arg.numel() == 0:
            # If numel is zero, then the output size must be zero.
            # In this case, we must not allocate an unbacked SymInt,
            # because if we do, it will immediately get refined to
            # zero, but this will be inconsistent with size oblivious
            # tests (which will continue to claim that the unbacked
            # symint cannot equal zero).  We could also unconditionally
            # allocate an unbacked SymInt and not refine its range,
            # but this seems more precise.
            nnz = 0
        else:
            nnz = fake_mode.shape_env.create_unbacked_symint()

            maxval = sys.maxsize - 1

            numel = arg.numel() if dim is None else arg.size(dim)
            if not has_free_symbols(numel):
                maxval = int(numel)

            _constrain_range_for_size(nnz, max=maxval)

        if dim is None:
            if unique_consecutive:
                arg.unique_consecutive_memo = nnz  # pyrefly: ignore[bad-assignment]
            else:
                arg.unique_memo = nnz  # pyrefly: ignore[bad-assignment]

    if dim is None:
        # pyrefly: ignore[no-matching-overload]
        ret = [arg.new_empty((nnz,))]
    else:
        # pyrefly: ignore[no-matching-overload]
        ret = [arg.new_empty(*arg.shape[:dim], nnz, *arg.shape[dim + 1 :])]
````
- **EN**: This chunk continues `_unique` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_unique`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 440-469 / 第 440-469 行
````python
    return_if_dim_and_cpu = dim is not None and arg.fake_device == torch.device("cpu")
    if return_inverse or return_if_dim_and_cpu:
        inverse = arg.new_empty(
            arg.shape if dim is None else (arg.shape[dim],), dtype=torch.int64
        )
    else:
        inverse = arg.new_empty(0, dtype=torch.int64)
    ret.append(inverse)

    if return_counts or return_if_dim_and_cpu:
        counts = arg.new_empty(
            ret[0].shape if dim is None else (ret[0].shape[dim],), dtype=torch.int64
        )
    else:
        counts = arg.new_empty(0, dtype=torch.int64)
    ret.append(counts)

    return tuple(ret)


@register_op_impl(aten._unique2.default)
def unique2(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    arg: FakeTensor,
    sorted: bool = True,
    return_inverse: bool = False,
    return_counts: bool = False,
) -> tuple[FakeTensor, FakeTensor, FakeTensor]:
    return _unique(fake_mode, func, arg, None, sorted, return_inverse, return_counts)
````
- **EN**: This chunk defines `unique2`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `unique2`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 472-501 / 第 472-501 行
````python
@register_op_impl(aten.select.int)
def meta_select(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    self: FakeTensor,
    dim: int,
    index: IntLikeType,
) -> FakeTensor:
    from torch.fx.experimental.symbolic_shapes import guard_or_false

    if self.is_sparse:
        return NotImplemented

    ndim = self.dim()
    torch._check_index(
        ndim != 0,
        lambda: "select() cannot be applied to a 0-dim tensor.",
    )

    dim = dim if dim >= 0 else dim + ndim
    size = self.size(dim)

    new_size = list(self.size())
    new_stride = list(self.stride())

    new_storage_offset = None
    if guard_or_false(index >= 0):
        new_storage_offset = self.storage_offset() + index * new_stride[dim]
    elif guard_or_false(index < 0):
        new_storage_offset = self.storage_offset() + (index + size) * new_stride[dim]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `meta_select`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `meta_select`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 503-538 / 第 503-538 行
````python
    if new_storage_offset is None:
        if fake_mode.shape_env is None or (
            not fake_mode.shape_env.allow_scalar_outputs
            and not fake_mode.allow_scalar_outputs
        ):
            raise DataDependentOutputException(func)

        # index is data-dependent, we do not know which index we are accessing it could be index or index+size!
        # we assign a new data-dependent symbol for the storage offset.
        new_storage_offset = fake_mode.shape_env.create_unbacked_symint()

    del new_size[dim]
    del new_stride[dim]
    if new_storage_offset is None:
        raise AssertionError("new_storage_offset must not be None")
    # pyrefly: ignore[bad-return]
    return self.as_strided(new_size, new_stride, new_storage_offset)


@register_op_impl(aten.unique_dim.default)
def unique_dim(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    arg: FakeTensor,
    dim: int,
    sorted: bool = True,
    return_inverse: bool = False,
    return_counts: bool = False,
) -> tuple[FakeTensor, FakeTensor, FakeTensor]:
    return _unique(
        fake_mode,
        func,
        arg,
        # normalize dim to be non-negative
        dim if dim >= 0 else dim % max(arg.ndim, 1),
        sorted,
````
- **EN**: This chunk defines `unique_dim`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `unique_dim`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 539-562 / 第 539-562 行
````python
        return_inverse,
        return_counts,
    )


@register_op_impl(aten.unique_consecutive.default)
def unique_consecutive(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    arg: FakeTensor,
    return_inverse: bool = False,
    return_counts: bool = False,
    dim: int | None = None,
) -> tuple[FakeTensor, FakeTensor, FakeTensor]:
    return _unique(
        fake_mode,
        func,
        arg,
        dim,
        False,
        return_inverse,
        return_counts,
        unique_consecutive=True,
    )
````
- **EN**: This chunk defines `unique_consecutive`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `unique_consecutive`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 565-598 / 第 565-598 行
````python
# This function is python match of computeStride_impl in TensorUtils.cpp
def _compute_stride(
    old_shape: Sequence[IntLikeType],
    old_stride: Sequence[IntLikeType],
    new_shape: Sequence[IntLikeType],
    size_oblivious: bool = False,
) -> list[IntLikeType] | None:
    from torch.fx.experimental.symbolic_shapes import (
        guard_or_false,
        guard_or_true,
        sym_eq,
    )

    def maybe_guard_or_false(x: Any) -> Any:
        if size_oblivious:
            return guard_or_false(x)

        return x

    def maybe_guard_or_true(x: Any) -> Any:
        if size_oblivious:
            return guard_or_true(x)

        return x

    if len(old_shape) == 0:
        return [1] * len(new_shape)

    numel = reduce(operator.mul, old_shape, 1)
    zero_numel = maybe_guard_or_false(numel == 0)
    if zero_numel and maybe_guard_or_false(sym_eq(old_shape, new_shape)):
        return list(old_stride)

    new_stride: list[IntLikeType] = [0] * len(new_shape)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `maybe_guard_or_true`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `maybe_guard_or_true`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 600-634 / 第 600-634 行
````python
    if zero_numel:
        for view_d in range(len(new_shape) - 1, -1, -1):
            if view_d == len(new_shape) - 1:
                new_stride[view_d] = 1
            else:
                new_stride[view_d] = (
                    max(new_shape[view_d + 1], 1) * new_stride[view_d + 1]
                )
        return new_stride

    view_d = len(new_shape) - 1
    # Annotate type here to support type checking
    chunk_base_stride: IntLikeType = old_stride[-1]
    tensor_numel: IntLikeType = 1
    view_numel: IntLikeType = 1

    for tensor_d in range(len(old_shape) - 1, -1, -1):
        tensor_numel *= old_shape[tensor_d]

        if tensor_d == 0 or (
            maybe_guard_or_true(old_shape[tensor_d - 1] != 1)
            and maybe_guard_or_true(
                old_stride[tensor_d - 1] != tensor_numel * chunk_base_stride
            )
        ):
            while view_d >= 0 and (
                maybe_guard_or_true(view_numel < tensor_numel)
                or maybe_guard_or_false(new_shape[view_d] == 1)
            ):
                new_stride[view_d] = view_numel * chunk_base_stride
                view_numel *= new_shape[view_d]
                view_d -= 1

            if maybe_guard_or_true(view_numel != tensor_numel):
                return None
````
- **EN**: This chunk continues `maybe_guard_or_true` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `maybe_guard_or_true`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 636-669 / 第 636-669 行
````python
            if tensor_d > 0:
                chunk_base_stride = old_stride[tensor_d - 1]
                tensor_numel = 1
                view_numel = 1
    if view_d != -1:
        return None
    return new_stride


def _view_has_unbacked_input(
    a: torch.Tensor, shape: ShapeType | tuple[ShapeType]
) -> bool:
    from torch.fx.experimental.symbolic_shapes import has_guarding_hint

    shape = utils.extract_shape_from_varargs(shape, validate=False)

    return (
        any(not has_guarding_hint(s) for s in a.size())
        or any(not has_guarding_hint(s) for s in a.stride())
        or any(not has_guarding_hint(s) for s in shape)
    )


def _view_unbacked_meta(
    a: torch.Tensor,
    shape: ShapeType | tuple[ShapeType],
    size_oblivious_enabled: bool = True,
    allow_copy: bool = False,
) -> torch.Tensor:
    from torch._prims import view_of
    from torch.fx.experimental.symbolic_shapes import guard_or_false, sym_eq

    # Creates a valid shape
    shape = utils.extract_shape_from_varargs(shape, validate=False)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes, torch._prims. This chunk defines `_view_unbacked_meta`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes、torch._prims。 这一段定义了 `_view_unbacked_meta`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 671-705 / 第 671-705 行
````python
    # Reshape may be given a shape with a -1 length
    # This indicates that the dimension's length should be inferred
    shape = utils.infer_size(shape, a.numel())

    # Special-cases reshaping zero dim tensors
    if a.ndim == 0:
        _a = a
        for length in shape:
            torch._check(length == 1)
            _a = torch._refs.unsqueeze(_a, -1)
        if _a is a:
            return view_of(a)
        else:
            return _a  # type: ignore[return-value]

    # Special-cases reshaping to zero dim tensors
    if len(shape) == 0:
        _a = a
        for length in a.shape:
            torch._check(length == 1)
            _a = torch._refs.squeeze(_a, -1)
        if _a is a:
            return view_of(a)
        else:
            return _a  # type: ignore[return-value]

    shape_numel = reduce(operator.mul, shape, 1)

    torch._check(
        a.numel() == shape_numel,
        lambda: f"Could not reshape a tensor with shape {a.shape} as a tensor with shape {shape}!",
    )

    if len(shape) == len(a.shape) and guard_or_false(sym_eq(shape, a.shape)):
        return view_of(a)
````
- **EN**: This chunk continues `_view_unbacked_meta` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_view_unbacked_meta`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 707-739 / 第 707-739 行
````python
    if is_contiguous_or_false(a) if size_oblivious_enabled else is_contiguous(a):
        strides = make_contiguous_strides_for(shape)
        return a.as_strided(shape, strides)  # type: ignore[return-value]

    new_strides = _compute_stride(
        a.size(), a.stride(), shape, size_oblivious=size_oblivious_enabled
    )

    if new_strides is not None:
        return a.as_strided(shape, new_strides)  # type: ignore[return-value]

    # If we fail to do size oblivious view, and backed_size_oblivious was on,
    # then we redo everything by looking at hints and guarding instead of failing.
    # Also if the expression has unbacked symbols, then we run again with size_oblivious_enabled=False
    # to throw a data dependent error.

    if size_oblivious_enabled and (
        torch.fx.experimental._config.backed_size_oblivious
        or _view_has_unbacked_input(a, shape)
    ):
        return _view_unbacked_meta(
            a, shape, size_oblivious_enabled=False, allow_copy=allow_copy
        )

    # When allow_copy=True (i.e., view_copy), define unbacked semantics
    # as "materialize": clone the input to break aliasing, then reshape.
    if allow_copy:
        strides = make_contiguous_strides_for(shape)
        # pyrefly: ignore[bad-return]
        return a.clone(memory_format=torch.contiguous_format).as_strided(shape, strides)

    msg = f"Cannot view a tensor with shape {a.shape} and strides {a.stride()} as a tensor with shape {shape}!"
    raise ValueError(msg)
````
- **EN**: This chunk continues `_view_unbacked_meta` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_view_unbacked_meta`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 742-777 / 第 742-777 行
````python
@register_op_impl(aten._reshape_copy.default)
def _reshape_copy(
    fake_mode: FakeTensorMode, func: OpOverload, a: FakeTensor, *shape: Any
) -> FakeTensor | Exception:
    if a.is_sparse or a.is_mkldnn:
        return NotImplemented

    # pyrefly: ignore[bad-argument-count]
    shape = utils.infer_size(*shape, a.numel())
    if is_contiguous_or_false(a):
        view = _view_meta(fake_mode, func, a, *shape)
        return typing_cast(
            FakeTensor, view.clone(memory_format=torch.contiguous_format)
        )
    else:
        return _view_meta(
            fake_mode,
            func,
            typing_cast(FakeTensor, a.clone(memory_format=torch.contiguous_format)),
            *shape,
        )


@register_op_impl(aten.view.default)
@register_op_impl(aten._unsafe_view.default)
def _view_meta(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    a: FakeTensor,
    *shape: Any,
    allow_copy: bool = False,
) -> FakeTensor:
    if torch.fx.experimental._config.backed_size_oblivious or _view_has_unbacked_input(
        a, shape
    ):
        return typing_cast(
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk defines `_view_meta`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段定义了 `_view_meta`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 778-806 / 第 778-806 行
````python
            FakeTensor, _view_unbacked_meta(a, shape, allow_copy=allow_copy)
        )
    else:
        return typing_cast(
            FakeTensor,
            torch._refs._reshape_view_helper(a, *shape, allow_copy=allow_copy),
        )


@register_op_impl(aten.view_copy.default)
def _view_meta_copy(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    a: FakeTensor,
    *shape: IntLikeType,
    out: FakeTensor | None = None,
) -> FakeTensor:
    # view_copy is the non-aliasing counterpart of view. Eager may succeed on
    # cases where a pure view is impossible (e.g. expand -> flatten) by
    # materializing the result. Match eager by allowing copy-if-needed in meta.
    result = _view_meta(fake_mode, func, a, *shape, allow_copy=True)

    if out is not None:
        return result

    return pytree.tree_map(
        lambda x: x.clone(memory_format=torch.contiguous_format),
        result,
    )
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk defines `_view_meta_copy`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段定义了 `_view_meta_copy`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 809-830 / 第 809-830 行
````python
@register_op_impl(aten.repeat_interleave.Tensor)
def repeat_interleave_tensor(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    repeats: FakeTensor,
    output_size: IntLikeType | None = None,
) -> FakeTensor:
    if output_size is None:
        if (
            fake_mode.shape_env is None
            or not fake_mode.shape_env.allow_dynamic_output_shape_ops
        ):
            raise DynamicOutputShapeException(func)

        output_size = fake_mode.shape_env.create_unbacked_symint()

        # Avoid importing sympy at a module level
        from torch.fx.experimental.symbolic_shapes import _constrain_range_for_size

        _constrain_range_for_size(output_size)
        # TODO: consider a memo
    return repeats.new_empty(output_size)  # type: ignore[return-value]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `repeat_interleave_tensor`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `repeat_interleave_tensor`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 833-862 / 第 833-862 行
````python
@register_op_impl(torch.ops.aten.item.default)
@register_op_impl(torch.ops.aten._local_scalar_dense.default)
def local_scalar_dense(
    fake_mode: FakeTensorMode, func: OpOverload, arg: FakeTensor
) -> int | float | bool | torch.SymInt | torch.SymFloat | torch.SymBool:
    if (r := arg.item_memo) is not None:
        return r
    if fake_mode.shape_env is None or (
        not fake_mode.shape_env.allow_scalar_outputs
        and not fake_mode.allow_scalar_outputs
    ):
        # Without symints/symfloats, cannot handle this
        raise DataDependentOutputException(func)
    if is_float_dtype(arg.dtype):
        r = fake_mode.shape_env.create_unbacked_symfloat()
    elif is_integer_dtype(arg.dtype):
        r = fake_mode.shape_env.create_unbacked_symint()
    elif is_boolean_dtype(arg.dtype):
        r = fake_mode.shape_env.create_unbacked_symbool()
    else:
        raise NotImplementedError(f"local_scalar_dense/item NYI for {arg.dtype}")
    arg.item_memo = r
    return r


@register_op_impl(torch.ops.aten.nonzero_numpy.default)
def nonzero_numpy(
    fake_mode: FakeTensorMode, func: OpOverload, arg: FakeTensor
) -> list[FakeTensor]:
    return torch.ops.aten.nonzero.default(arg).unbind(1)
````
- **EN**: This chunk defines `nonzero_numpy`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `nonzero_numpy`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 865-896 / 第 865-896 行
````python
@register_op_impl(torch.ops.aten.nonzero.default)
def nonzero(fake_mode: FakeTensorMode, func: OpOverload, arg: FakeTensor) -> FakeTensor:
    if (
        fake_mode.shape_env is None
        or not fake_mode.shape_env.allow_dynamic_output_shape_ops
    ):
        # Without symints/symfloats, cannot handle this
        raise DynamicOutputShapeException(func)

    if (nnz := arg.nonzero_memo) is None:
        # Avoid importing sympy at a module level
        from torch.fx.experimental.symbolic_shapes import (
            _constrain_range_for_size,
            has_free_symbols,
        )
        from torch.utils._sympy.numbers import IntInfinity
        from torch.utils._sympy.value_ranges import bound_sympy

        if not has_free_symbols(arg.numel()) and arg.numel() == 0:
            # If numel is zero, then the output size must be zero.
            # In this case, we must not allocate an unbacked SymInt,
            # because if we do, it will immediately get refined to
            # zero, but this will be inconsistent with size oblivious
            # tests (which will continue to claim that the unbacked
            # symint cannot equal zero).  We could also unconditionally
            # allocate an unbacked SymInt and not refine its range,
            # but this seems more precise.
            nnz = 0
        else:
            nnz = fake_mode.shape_env.create_unbacked_symint()

            maxval = sys.maxsize - 1
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes, torch.utils._sympy.numbers, torch.utils._sympy.value_ranges. This chunk defines `nonzero`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes、torch.utils._sympy.numbers、torch.utils._sympy.value_ranges。 这一段定义了 `nonzero`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 898-928 / 第 898-928 行
````python
            if not has_free_symbols(arg.numel()):
                maxval = int(arg.numel())
            else:
                prod_node = math.prod(arg.shape).node  # type: ignore[union-attr]
                prod_range = bound_sympy(
                    prod_node.expr, prod_node.shape_env.var_to_range
                )
                if isinstance(prod_range.upper, IntInfinity):
                    maxval = sys.maxsize - 1
                else:
                    maxval = prod_range.upper

            _constrain_range_for_size(nnz, max=maxval)

        arg.nonzero_memo = nnz  # pyrefly: ignore[bad-assignment]
    return arg.new_empty_strided((nnz, arg.dim()), (1, nnz), dtype=torch.int64)  # type: ignore[return]


@register_op_impl(torch.ops.aten._padded_dense_to_jagged_forward.default)
def _padded_dense_to_jagged_forward(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    padded: FakeTensor,
    offsets: list[FakeTensor],
    total_L: IntLikeType | None = None,
) -> FakeTensor:
    # only one jagged dim is supported for now
    if len(offsets) != 1:
        raise AssertionError(
            f"Only one jagged dim is supported, got {len(offsets)} offsets"
        )
````
- **EN**: This chunk defines `_padded_dense_to_jagged_forward`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_padded_dense_to_jagged_forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 930-958 / 第 930-958 行
````python
    if not total_L:
        if (
            fake_mode.shape_env is None
            or not fake_mode.shape_env.allow_dynamic_output_shape_ops
        ):
            # Without symints/symfloats, cannot handle this
            raise DynamicOutputShapeException(func)

        total_L = fake_mode.shape_env.create_unbacked_symint()

        maxval = sys.maxsize - 1

        # Avoid importing sympy at a module level
        from torch.fx.experimental.symbolic_shapes import (
            _constrain_range_for_size,
            has_free_symbols,
        )

        if not has_free_symbols(padded.numel()):
            maxval = int(padded.numel())

        _constrain_range_for_size(total_L, min=0, max=maxval)

    output_shape = (total_L, *padded.shape[2:])
    return padded.new_empty(output_shape)  # type: ignore[return]


def _compute_slice_index(size: IntLikeType, index: IntLikeType) -> IntLikeType | None:
    from torch.fx.experimental.symbolic_shapes import guard_or_false, sym_and
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `_compute_slice_index`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `_compute_slice_index`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 960-989 / 第 960-989 行
````python
    if guard_or_false(sym_and(index >= 0, index <= size)):
        return index
    elif guard_or_false(sym_and(index < 0, index >= -size)):
        return index + size
    elif guard_or_false(index < -size):
        return 0
    elif guard_or_false(index > size):
        return size
    elif guard_or_false(index >= 0):
        return torch.sym_min(index, size)
    elif guard_or_false(index < 0):
        return torch.sym_max(index + size, 0)

    return None


@register_op_impl(torch.ops.aten.slice.Tensor)
def slice_forward(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    self: FakeTensor,
    dim: int = 0,
    start: int | None = None,
    end: int | None = None,
    step: int = 1,
) -> FakeTensor:
    from torch.fx.experimental.symbolic_shapes import (
        guard_or_false,
        statically_known_true,
    )
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `slice_forward`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `slice_forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 991-1022 / 第 991-1022 行
````python
    shape_env = fake_mode.shape_env
    ndim = self.dim()
    if ndim == 0:
        raise RuntimeError("slice() cannot be applied to a 0-dim tensor.")
    dim = canonicalize_dim(self.dim(), dim)
    sizes = list(self.size())
    strides = list(self.stride())

    if step <= 0:
        raise RuntimeError("slice step must be positive")

    # start, end
    start_index = 0 if start is None else _compute_slice_index(sizes[dim], start)
    end_index = (
        sizes[dim]
        if statically_known_true(end == sys.maxsize) or end is None
        else _compute_slice_index(sizes[dim], end)
    )

    # size
    new_size: IntLikeType | None = None
    if start_index is not None and end_index is not None:
        if guard_or_false(end_index >= start_index):
            new_size = (end_index - start_index + step - 1) // step
        elif guard_or_false(start_index >= end_index):
            new_size = 0
        else:
            # Both indices are resolved but we can't statically determine their
            # ordering (e.g., when they involve Min/Max). Compute the size via
            # max(end - start, 0) to avoid creating an unbacked symint.
            diff = torch.sym_max(end_index - start_index, 0)
            new_size = (diff + step - 1) // step
````
- **EN**: This chunk continues `slice_forward` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `slice_forward`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 1024-1051 / 第 1024-1051 行
````python
    # create unbacked if case unknown
    if new_size is None:
        if shape_env is None:
            raise AssertionError("Must have shape_env to create symint")
        new_size = shape_env.create_unbacked_symint()
        torch._check(new_size >= 0)
        torch._check(new_size <= sizes[dim])

    # stride
    new_stride = strides[dim] * step

    # storage offset
    if start_index is not None:
        storage_offset = self.storage_offset() + start_index * strides[dim]
    else:
        if shape_env is None:
            raise AssertionError("Must have shape_env to create symint")
        storage_offset = shape_env.create_unbacked_symint()
        torch._check(storage_offset >= 0)

    sizes[dim] = new_size  # type: ignore[unsupported-operation]
    strides[dim] = new_stride
    if self.is_quantized:
        raise NotImplementedError(
            "Slice decomposition for quantized tensors aren't implemented"
        )
    else:
        return self.as_strided(sizes, strides, storage_offset)  # type: ignore[return-value]
````
- **EN**: This chunk continues `slice_forward` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `slice_forward`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1054-1076 / 第 1054-1076 行
````python
@register_op_impl(torch.ops.aten.masked_select.default)
def masked_select(
    fake_mode: FakeTensorMode, func: OpOverload, self: FakeTensor, mask: FakeTensor
) -> FakeTensor:
    if (
        fake_mode.shape_env is None
        or not fake_mode.shape_env.allow_dynamic_output_shape_ops
    ):
        # Without symints/symfloats, cannot handle this
        raise DynamicOutputShapeException(func)

    nnz = fake_mode.shape_env.create_unbacked_symint()

    # see nonzero for commentary
    maxval = sys.maxsize - 1

    # Avoid importing sympy at a module level
    from torch.fx.experimental.symbolic_shapes import (
        _constrain_range_for_size,
        has_free_symbols,
    )
    from torch.utils._sympy.numbers import IntInfinity
    from torch.utils._sympy.value_ranges import bound_sympy
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes, torch.utils._sympy.numbers, torch.utils._sympy.value_ranges. This chunk defines `masked_select`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes、torch.utils._sympy.numbers、torch.utils._sympy.value_ranges。 这一段定义了 `masked_select`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 1078-1113 / 第 1078-1113 行
````python
    # If num elements is expressed symbolically, calculate
    # the concrete value based on upper bounds. Otherwise,
    # we can set max val directly.
    if not has_free_symbols(self.numel()):
        num_elements = int(self.numel())
    else:
        prod_node = math.prod(self.shape).node  # type: ignore[union-attr]
        prod_range = bound_sympy(prod_node.expr, prod_node.shape_env.var_to_range)
        if isinstance(prod_range.upper, IntInfinity):
            num_elements = sys.maxsize - 1
        else:
            num_elements = prod_range.upper
    if num_elements > 2:
        maxval = num_elements

    _constrain_range_for_size(nnz, max=maxval)

    return self.new_empty((nnz,))  # type: ignore[return]


@register_op_impl(torch.ops.aten._assert_tensor_metadata.default)
def assert_tensor_metadata(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    t: FakeTensor,
    sizes: torch.Size | None = None,
    strides: tuple[int, ...] | None = None,
    dtype: torch.dtype | None = None,
    *,
    device: torch.device | None = None,
    layout: torch.layout | None = None,
) -> None:
    if sizes is not None:
        if t.size() != sizes:
            raise AssertionError(
                f"Tensor sizes mismatch! Expected: {sizes}, Got: {t.size()}"
````
- **EN**: This chunk defines `assert_tensor_metadata`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `assert_tensor_metadata`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1114-1142 / 第 1114-1142 行
````python
            )
    if strides is not None:
        if t.stride() != strides:
            raise AssertionError(
                f"Tensor strides mismatch! Expected: {strides}, Got: {t.stride()}"
            )
    if dtype is not None:
        if t.dtype != dtype:
            raise AssertionError(
                f"Tensor dtype mismatch! Expected: {dtype}, Got: {t.dtype}"
            )
    if layout is not None:
        if t.layout != layout:
            raise AssertionError(
                f"Tensor layout mismatch! Expected: {layout}, Got: {t.layout}"
            )
    if device is not None:
        if t.device != device:
            raise AssertionError(
                f"Tensor device mismatch! Expected: {device}, Got: {t.device}"
            )


# NB: this must be ordered after local_scalar_dense
@register_op_impl(lambda func: torch.Tag.data_dependent_output in func.tags)
def data_dep(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> None:
    raise DataDependentOutputException(func)
````
- **EN**: This chunk defines `data_dep`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `data_dep`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 1145-1176 / 第 1145-1176 行
````python
# Bool Indices get Expanded as Masks
# See: IndexingUtils.h:expandTensors
def check_no_bool_index_tensors(
    func: OpOverload, self: FakeTensor, indices: list[FakeTensor | None]
) -> None:
    for index in indices:
        if index is not None and index.dtype in (torch.bool, torch.uint8):
            raise DynamicOutputShapeException(func)


def run_and_return_new_tensor_of_input_device(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    args: tuple[Any, ...],
    kwargs: dict[str, Any],
) -> FakeTensor:
    # TODO: ref
    _, new_kwargs = _normalize_function_or_error(
        func, args=args, kwargs=kwargs, normalize_to_only_use_kwargs=True
    )
    out_device = new_kwargs["input"].device
    with in_kernel_invocation_manager(fake_mode):
        out = func(*args, **kwargs)
        if not is_noncontiguous_supported(out_device):
            out = out.new_empty(out.shape)

    if out is new_kwargs["input"]:
        return out  # copy_
    return FakeTensor(fake_mode, out, out_device)


_is_builtin_namespaces = ordered_set("aten", "prims", "prim")
````
- **EN**: This chunk defines `run_and_return_new_tensor_of_input_device`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `run_and_return_new_tensor_of_input_device`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1179-1211 / 第 1179-1211 行
````python
def is_builtin(op: OpOverload) -> bool:
    return op.namespace in _is_builtin_namespaces


def has_meta(func: OpOverload) -> bool:
    return torch._C._dispatch_has_computed_kernel_for_dispatch_key(func.name(), "Meta")


# These are for the `torch._foreach_...` ops like `torch._foreach_add`.
@register_op_impl(
    lambda func: is_builtin(func)
    and func.name().startswith("aten::_foreach_")
    and has_meta(func)
)
def foreach_run_and_map_input_device(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> list[FakeTensor] | None:
    tensor_lists = [
        arg
        for arg in itertools.chain(args, kwargs.values())
        if isinstance(arg, (list, tuple))
        and len(arg)
        and isinstance(arg[0], torch.Tensor)
    ]

    try:
        with in_kernel_invocation_manager(fake_mode):
            out_meta = func(*args, **kwargs)
    except NotImplementedError:
        return NotImplemented

    if not out_meta:
        return out_meta
````
- **EN**: This chunk defines `foreach_run_and_map_input_device`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `foreach_run_and_map_input_device`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1213-1245 / 第 1213-1245 行
````python
    if not tensor_lists:
        raise AssertionError("tensor_lists must not be empty")
    out_fake = []

    for i, meta_t in enumerate(out_meta):
        device, _ = FakeTensor._find_common_device(func, [tl[i] for tl in tensor_lists])
        out_fake.append(
            fake_mode.fake_tensor_converter.from_meta_and_device(
                fake_mode, meta_t, device
            )
        )

    return out_fake


# Dont default to default device handling,
# Since op can take in non-zero sized cpu
# index tensors with cuda self
@register_op_impl(aten.index.Tensor)
def index_tensor(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    from torch._meta_registrations import meta_index_Tensor

    _, new_kwargs = _normalize_function_or_error(
        func, args=args, kwargs=kwargs, normalize_to_only_use_kwargs=True
    )

    out_device = new_kwargs["input"].device
    # ensure nonzero call goes to fake tensor
    with fake_mode:
        out = meta_index_Tensor(*args, **kwargs)
        return out.to(out_device)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._meta_registrations. This chunk defines `index_tensor`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._meta_registrations。 这一段定义了 `index_tensor`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1248-1279 / 第 1248-1279 行
````python
# Can take mixed meta/non-meta arguments; the meta registration
# will roughly do the right thing even when given real devices
@register_op_impl(aten._embedding_bag.default)
def embedding_bag(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> tuple[FakeTensor, FakeTensor, FakeTensor, FakeTensor]:
    from torch._meta_registrations import meta_embedding_bag

    with fake_mode:
        return meta_embedding_bag(*args, **kwargs)


# takes in multiple-devices, dont default to default device handling
@register_op_impl(aten._unsafe_index_put.default)
@register_op_impl(aten.copy.default)
@register_op_impl(aten.copy_.default)
@register_op_impl(aten.slice_scatter.default)
@register_op_impl(aten.diagonal_scatter.default)
def multi_device_op_default(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    return run_and_return_new_tensor_of_input_device(fake_mode, func, args, kwargs)


# same with multi_device_op_default, but return the input
@register_op_impl(aten.copy.out)
@register_op_impl(aten.slice_scatter.out)
def multi_device_op_out(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    with in_kernel_invocation_manager(fake_mode):
        func(*args, **kwargs)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._meta_registrations. This chunk defines `multi_device_op_out`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._meta_registrations。 这一段定义了 `multi_device_op_out`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1281-1307 / 第 1281-1307 行
````python
    _, new_kwargs = _normalize_function_or_error(
        func, args=args, kwargs=kwargs, normalize_to_only_use_kwargs=True
    )

    return new_kwargs["input"]


@register_op_impl(aten.index_put.default)
@register_op_impl(aten.index_put_.default)
def index_put_impl(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor:
    _, new_kwargs = _normalize_function_or_error(
        func, args=args, kwargs=kwargs, normalize_to_only_use_kwargs=True
    )
    values = new_kwargs["values"]
    self_device = new_kwargs["input"].fake_device
    torch._check(
        self_device == values.fake_device or (values.ndim == 0 and values.numel() == 1),
        lambda: f"Mismatching {func} device between self ({self_device}) and values ({values.device})",
    )

    out = run_and_return_new_tensor_of_input_device(fake_mode, func, args, kwargs)
    if func is aten.index_put_.default:
        return new_kwargs["input"]
    else:
        return out
````
- **EN**: This chunk defines `index_put_impl`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `index_put_impl`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1310-1337 / 第 1310-1337 行
````python
@register_op_impl(aten._nested_tensor_from_tensor_list.default)
@register_op_impl(aten._nested_tensor_from_tensor_list.out)
@register_op_impl(aten._nested_view_from_buffer.default)
@register_op_impl(aten._nested_view_from_buffer_copy.default)
def nested_tensors_unsupported(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> None:
    raise UnsupportedOperatorException(func)


@register_op_impl(
    [
        x
        for x in _device_not_kwarg_ops
        if x
        not in (
            # these are already registered elsewhere
            aten.is_pinned.default,
            aten.to.device,
            aten.to.prim_Device,
            aten._nested_tensor_from_tensor_list.default,
            aten._nested_tensor_from_tensor_list.out,
        )
    ]
)
def nyi(fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any) -> None:
    if func in _device_not_kwarg_ops:
        raise AssertionError(f"NYI: {func}")
````
- **EN**: This chunk defines `nyi`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `nyi`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 1340-1375 / 第 1340-1375 行
````python
@register_op_impl([aten.convolution.default, aten.convolution_backward.default])
def conv(
    fake_mode: FakeTensorMode, func: OpOverload, *args: Any, **kwargs: Any
) -> FakeTensor | tuple[FakeTensor | None, FakeTensor | None, FakeTensor | None]:
    _, new_kwargs = _normalize_function_or_error(
        func, args=args, kwargs=kwargs, normalize_to_only_use_kwargs=True
    )
    input_ = new_kwargs["input"]
    weight = new_kwargs["weight"]
    device = input_.fake_device
    # need to re-enable mode so the tensors report fake device
    with fake_mode:
        # if the input is unsqueezed in Convolution.cpp we get segfault
        k = weight.ndim

        # Avoid importing sympy at a module level
        from torch.fx.experimental.symbolic_shapes import has_guarding_hint

        all_hinted = all(has_guarding_hint(s) for s in input_.shape) and all(
            has_guarding_hint(s) for s in weight.shape
        )

        if not all_hinted:
            # TODO: We can make this a little more faithful with best effort
            # channels last detection (but only if it's statically obvious!)
            mem_fmt = None
        else:
            # convolution has "bias" but not "bias_sizes"; convolution_backward
            # has "bias_sizes" but not "bias". .get() handles both with one call.
            bias = new_kwargs.get("bias")
            select_kwargs: dict[str, object] = dict(
                stride=new_kwargs["stride"],
                padding=new_kwargs["padding"],
                dilation=new_kwargs["dilation"],
                transposed=new_kwargs["transposed"],
                output_padding=new_kwargs["output_padding"],
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `conv`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `conv`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 1376-1409 / 第 1376-1409 行
````python
                groups=new_kwargs["groups"],
                bias=bias,
            )
            if bias is None:
                select_kwargs["bias_sizes"] = new_kwargs.get("bias_sizes")
            conv_backend = torch._C._select_conv_backend(
                input_, weight, **select_kwargs
            )
            # Expand 1d -> 2d.
            # Note: Avoid expanding before calling _select_conv_backend,
            # as the function handles 2D expansion internally.
            if k == 3 and not input_.is_mkldnn and not input_.is_xpu:
                # Note: Using input.to(memory_format=contiguous) does not work.
                input_ = input_.contiguous().unsqueeze(2)
                weight = weight.unsqueeze(2)
            mem_fmt = torch._C._conv_determine_backend_memory_format(
                input_, weight, conv_backend
            )

    def convert(
        t: torch.Tensor | None, mem_fmt: torch.memory_format | None
    ) -> FakeTensor | None:
        if t is None:
            return t
        if mem_fmt is not None:
            # channels last only support 4d, try to expand dim then convert it back later.
            if t.dim() == 3 and mem_fmt == torch.channels_last:
                t = t.unsqueeze(2).to(memory_format=mem_fmt).squeeze(2)
            else:
                t = t.to(memory_format=mem_fmt)
        return FakeTensor(fake_mode, t, device)

    with in_kernel_invocation_manager(fake_mode):
        out = func(**new_kwargs)
````
- **EN**: This chunk defines `convert`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `convert`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1411-1442 / 第 1411-1442 行
````python
        if func is aten.convolution.default:
            return convert(out, mem_fmt)  # type: ignore[return]
        else:
            return (
                convert(out[0], mem_fmt),
                convert(out[1], mem_fmt),
                convert(out[2], None),
            )


@register_op_impl(torch.ops.aten.bincount.default)
def bincount(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    inputs: FakeTensor,
    weights: FakeTensor | None = None,
    minlength: IntLikeType = 0,
) -> FakeTensor:
    if (
        fake_mode.shape_env is None
        or not fake_mode.shape_env.allow_dynamic_output_shape_ops
    ):
        # Without symints/symfloats, cannot handle this
        raise DynamicOutputShapeException(func)

    new_size = fake_mode.shape_env.create_unbacked_symint()

    from torch.fx.experimental.symbolic_shapes import _constrain_range_for_size

    _constrain_range_for_size(new_size)
    torch._check(new_size >= minlength)
    return inputs.new_empty(new_size)  # type: ignore[return]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `bincount`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `bincount`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1445-1477 / 第 1445-1477 行
````python
@register_op_impl(torch.ops.aten._pack_padded_sequence.default)
def _pack_padded_sequence(
    fake_mode: FakeTensorMode,
    func: OpOverload,
    inputs: FakeTensor,
    lengths: FakeTensor,
    batch_first: bool,
) -> tuple[FakeTensor, FakeTensor]:
    if (
        fake_mode.shape_env is None
        or not fake_mode.shape_env.allow_dynamic_output_shape_ops
    ):
        # Without symints/symfloats, cannot handle this
        raise DynamicOutputShapeException(func)

    new_batch_size = fake_mode.shape_env.create_unbacked_symint()

    from torch.fx.experimental.symbolic_shapes import _constrain_range_for_size

    _constrain_range_for_size(new_batch_size)

    if not batch_first:
        # Inputs should have shape (batch_size, seq_len, *)
        inputs = inputs.transpose(0, 1)  # type: ignore[assignment]

    res_size = inputs.shape[1:]
    packed_data = inputs.new_empty(res_size)
    batch_size = inputs.new_empty((new_batch_size,))
    return (packed_data, batch_size)  # type: ignore[return]


# pyrefly: ignore [implicit-any]
FAST_OP_IMPLEMENTATIONS = {}
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `_pack_padded_sequence`, which implements a focused helper used by the surrounding module. Decorators such as `register_op_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `_pack_padded_sequence`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_op_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1480-1507 / 第 1480-1507 行
````python
# Unlike register_op_impl, these don't do the slow iteration for
# run_impl_check, and these run BEFORE decompositions
def register_fast_op_impl(
    func: OpOverload,
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]]:
    def impl_decorator(op_impl: Callable[_P, _R]) -> Callable[_P, _R]:
        FAST_OP_IMPLEMENTATIONS[func] = op_impl
        return op_impl

    return impl_decorator


# infer_size_impl in ExpandUtils
def infer_size(
    a: Sequence[IntLikeType], b: Sequence[IntLikeType]
) -> tuple[IntLikeType, ...]:
    from torch.fx.experimental.symbolic_shapes import guard_or_false

    dimsA = len(a)
    dimsB = len(b)
    ndim = max(dimsA, dimsB)
    expandedSizes: list[IntLikeType] = [0] * ndim
    for i in range(ndim - 1, -1, -1):
        offset = ndim - 1 - i
        dimA = dimsA - 1 - offset
        dimB = dimsB - 1 - offset
        sizeA = a[dimA] if dimA >= 0 else 1
        sizeB = b[dimB] if dimB >= 0 else 1
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk defines `infer_size`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段定义了 `infer_size`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1509-1540 / 第 1509-1540 行
````python
        # NB: It is very important to test for broadcasting, before testing
        # sizeA == sizeB.  This is because the broadcasting tests are likely
        # to be statically known (in particular, if sizeA/sizeB is unbacked
        # but size-like, we will unsoundly assume they never equal 1), but
        # the sizeA == sizeB test may not be statically known.  However, once
        # we have established that no broadcasting is happening, the
        # sizeA == sizeB is now expect_true and we can defer it as a runtime
        # assert (this works because Python will return the terminal
        # expression of an or statement as-is, without bool()'ing it; if this
        # were not the case, we'd need to write this using torch.sym_or() or
        # something like that).
        torch._check(
            guard_or_false(sizeA == 1) or guard_or_false(sizeB == 1) or sizeA == sizeB,
            lambda: f"The size of tensor a ({sizeA}) "
            f"must match the size of tensor b ({sizeB}) "
            f"at non-singleton dimension {i})",
        )
        expandedSizes[i] = sizeB if guard_or_false(sizeA == 1) else sizeA
    return tuple(expandedSizes)


def make_fast_binary_impl(
    slow_ref: Callable[..., Any],
    type_promotion_kind: ELEMENTWISE_TYPE_PROMOTION_KIND = ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
) -> Callable[..., FakeTensor]:
    def fast_binary_impl(mode: FakeTensorMode, *args: Any, **kwargs: Any) -> FakeTensor:
        def slow(msg: str) -> FakeTensor:
            count_label(f"slow {msg}")
            with mode:
                return slow_ref(*args, **kwargs)

        count_label("attempt fast")
````
- **EN**: This chunk defines `slow`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `slow`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1542-1566 / 第 1542-1566 行
````python
        # Fast path (based off of TensorIterator fast path).
        # Unfortunately, there is no way to easily deduplicate
        # this with either the TensorIterator C++ implementation
        # (which we don't want to SymIntify, and also the algorithm
        # here is slightly different from TensorIterator to allow
        # for broadcasting), nor the PrimTorch implementation
        # (which does not actually implement a fast path.)

        operands = args

        # compute_shape
        final_shape: ShapeType | None = None
        for op in operands:
            shape: ShapeType = op.shape if isinstance(op, torch.Tensor) else ()
            if final_shape is None:
                final_shape = shape
            # TODO: Minor optimization: track if the shapes
            # were equal so you can skip the equality check
            # below if unnecessary
            # pyrefly: ignore[bad-assignment]
            final_shape = infer_size(final_shape, shape)
        if final_shape is None:
            raise AssertionError("final_shape must not be None")

        from torch.fx.experimental.symbolic_shapes import guard_or_false, sym_eq
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes. This chunk continues `slow` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes。 这一段延续了 `slow`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 1568-1600 / 第 1568-1600 行
````python
        # Do some extra safety checks to see if the output
        # stride is obvious
        for op in operands:
            if (
                isinstance(op, torch.Tensor)
                and len(op.shape) == len(final_shape)
                # take the slow path if result is not determined.
                and guard_or_false(sym_eq(op.shape, final_shape))  # type: ignore[arg-type]
            ):
                break
        else:
            # if we never break in the for loop above we take the slow path.
            return slow("both tensors nontrivially broadcast")

        # compute_types
        cpu = torch.device("cpu")
        common_device: torch.device = cpu
        common_dtype: torch.dtype | None = None
        has_different_input_dtypes = False
        for op in operands:
            if not isinstance(op, torch.Tensor):
                # Use elementwise_dtypes for the tricky case
                has_different_input_dtypes = True
                continue
            if common_device == cpu and op.device.type != "cpu":
                common_device = op.device
            if common_dtype is None:
                if type_promotion_kind != ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT:
                    has_different_input_dtypes = True
                else:
                    common_dtype = op.dtype
            elif common_dtype != op.dtype:
                has_different_input_dtypes = True
````
- **EN**: This chunk continues `slow` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `slow`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1602-1628 / 第 1602-1628 行
````python
        if has_different_input_dtypes:
            # compute promotion
            # TODO: we don't need the compute type
            _, common_dtype = elementwise_dtypes(
                *operands, type_promotion_kind=type_promotion_kind
            )

        # check all tensors on same device
        # cpu scalars are assumed allow
        current_cpu_scalars_on_non_cpu = 0
        max_cpu_scalars_on_non_cpu = 1  # hard coded atm
        for op in operands:
            if not isinstance(op, torch.Tensor):
                continue
            if common_device != cpu and op.dim() == 0 and op.device == cpu:
                if current_cpu_scalars_on_non_cpu >= max_cpu_scalars_on_non_cpu:
                    return slow("error")
                current_cpu_scalars_on_non_cpu += 1
            elif op.device != common_device:
                return slow("error")

        # compute_fast_setup_type
        definitely_contiguous = True
        definitely_channels_last = True

        # TODO: is_non-overlapping_and_dense not bound from Python
        # no inplace, no out, everything defined
````
- **EN**: This chunk continues `slow` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `slow`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1630-1665 / 第 1630-1665 行
````python
        if is_noncontiguous_supported(common_device):
            for op in operands:
                if not isinstance(op, torch.Tensor):
                    continue
                definitely_contiguous = (
                    definitely_contiguous
                    and is_contiguous_for_memory_format_or_false(
                        op, memory_format=torch.contiguous_format
                    )
                )
                definitely_channels_last = (
                    definitely_channels_last
                    and is_contiguous_for_memory_format_or_false(
                        op, memory_format=torch.channels_last
                    )
                )
        if definitely_contiguous:
            # do contiguous
            count_label("fast is_contiguous")
            return FakeTensor(
                mode,
                torch.empty(
                    final_shape,
                    dtype=common_dtype,
                    device="meta",
                    memory_format=torch.contiguous_format,
                ),
                device=common_device,
            )
        if definitely_channels_last:
            count_label("fast channels_last")
            # do channels last
            return FakeTensor(
                mode,
                torch.empty(
                    final_shape,
````
- **EN**: This chunk continues `slow` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `slow`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1666-1692 / 第 1666-1692 行
````python
                    dtype=common_dtype,
                    device="meta",
                    memory_format=torch.channels_last,
                ),
                device=common_device,
            )

        return slow("no contiguity match")

    return fast_binary_impl


# disable the python dispatcher to avoid decomposing detach() further
# (proxy_mode should still decompose detach() though)
def fast_detach(
    fake_mode: FakeTensorMode, x: FakeTensor, include_real: bool = False
) -> FakeTensor:
    with no_python_dispatcher(), in_kernel_invocation_manager(fake_mode):
        out = torch.ops.aten.detach.default(x)
    if include_real:
        return FakeTensor(fake_mode, out, x.device, real_tensor=x.real_tensor)
    return FakeTensor(fake_mode, out, x.device)


@functools.cache
def get_fast_op_impls() -> dict[OpOverload, Callable[..., Any]]:
    import torch._refs
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._refs. This chunk defines `get_fast_op_impls`, which retrieves runtime state and exposes it through a Python-friendly accessor. Decorators such as `functools.cache` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._refs。 这一段定义了 `get_fast_op_impls`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 像 `functools.cache` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1694-1710 / 第 1694-1710 行
````python
    register_fast_op_impl(torch.ops.aten.add.Tensor)(
        make_fast_binary_impl(torch._refs.add)
    )
    register_fast_op_impl(torch.ops.aten.sub.Tensor)(
        make_fast_binary_impl(torch._refs.sub)
    )
    register_fast_op_impl(torch.ops.aten.mul.Tensor)(
        make_fast_binary_impl(torch._refs.mul)
    )  # type: ignore[has-type]
    register_fast_op_impl(torch.ops.aten.div.Tensor)(
        make_fast_binary_impl(
            torch._refs.div,
            type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
        )
    )
    register_fast_op_impl(torch.ops.aten.detach.default)(fast_detach)
    return FAST_OP_IMPLEMENTATIONS
````
- **EN**: This chunk continues `get_fast_op_impls` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_fast_op_impls`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor subclassing**
  - EN: Handles tensor subclasses and fake/symbolic tensor behavior without breaking dispatch.
  - CN: 在不破坏 dispatch 的前提下处理张量子类与 fake/符号张量行为。
- **_P**
  - EN: `_P` is one of the main symbols declared or implemented in this file.
  - CN: `_P` 是本文件声明或实现的主要符号之一。
- **_R**
  - EN: `_R` is one of the main symbols declared or implemented in this file.
  - CN: `_R` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._custom_op`, `torch._logging`, `torch._prims_common`, `torch._dispatch.python`, `torch._ops`, `torch._subclasses.fake_tensor`, `torch.fx.operator_schemas`, `torch.utils._stats`, `torch.types`, `torch.fx.experimental.symbolic_shapes`, `torch._prims`
- **Standard library / 标准库**: `__future__`, `functools`, `itertools`, `math`, `operator`, `sys`, `typing`, `collections.abc`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `_P`, `_R`, `_T`, `__all__`, `ordered_set`, `is_noncontiguous_supported`, `contains_tensor_types`, `_is_tensor_constructor`, `register_op_impl`, `_is_op_registered_to_fake_rule`
