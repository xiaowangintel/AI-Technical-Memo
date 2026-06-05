# autograd.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_custom_op/autograd.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides general-purpose torch Python helpers, package wiring, or small runtime-facing utilities.
- **Purpose (CN)**: 提供通用的 torch Python 辅助逻辑、包级连接代码或小型运行时工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import functools
from collections import namedtuple

import torch
import torch.utils._pytree as pytree


# NOTE [CustomOp autograd kernel indirection]
# We register `inner` as the autograd kernel for this custom_op.
# `inner` either calls the autograd formula registered by the user,
# or goes into an `autograd_not_implemented` kernel.
#
# The reason why this indirection exists is
# so that we can swap out the autograd kernel (the PyTorch dispatcher
# doesn't actually allow us to do this). By default, we want
# the `autograd_not_implemented` behavior, but then the user may come
# and register something that is actually a backward formula
def autograd_kernel_indirection(custom_op):
    autograd_fallback = autograd_not_implemented(custom_op)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.utils._pytree; standard-library helpers such as functools, collections. This chunk defines `autograd_kernel_indirection`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.utils._pytree；标准库辅助模块，如 functools、collections。 这一段定义了 `autograd_kernel_indirection`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 22-41 / 第 22-41 行
````python
    def inner(*args, **kwargs):
        if custom_op._has_impl("autograd"):
            kernel = custom_op._get_impl("autograd").func
            return kernel(*args, **kwargs)
        # As explained in NOTE ["backward", "save_for_backward", and "autograd"],
        # after the user gives us "backward" and "save_for_backward", we generate
        # the "autograd" impl. If the user only provided one, then we tell
        # the user they've done something wrong.
        if custom_op._has_impl("save_for_backward") or custom_op._has_impl("backward"):
            missing = (
                "save_for_backward" if custom_op._has_impl("backward") else "backward"
            )
            found = "save_for_backward" if missing == "backward" else "backward"
            loc = custom_op._get_impl(found).location
            raise RuntimeError(
                f"We found a '{found}' registration for {custom_op} at "
                f"{loc} but were unable to find a '{missing}' registration. "
                f"To use the CustomOp API to register a backward formula, "
                f"please provide us both a backward function and a "
                f"'save for backward' function via `impl_backward` and "
````
- **EN**: This chunk defines `inner`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `inner`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 42-60 / 第 42-60 行
````python
                f"`impl_save_for_backward` respectively."
            )
        return autograd_fallback(*args, **kwargs)

    return inner


# TODO(#101191): Use the actual C++ autograd not implemented fallback,
# or change the default autograd fallback to the autograd not implemented fallback.
def autograd_not_implemented(custom_op):
    def kernel(*args, **kwargs):
        if torch.is_grad_enabled() and pytree.tree_any(
            lambda x: isinstance(x, torch.Tensor) and x.requires_grad, (args, kwargs)
        ):
            raise RuntimeError("Autograd has not been implemented for operator")
        with torch._C._AutoDispatchBelowAutograd():
            return custom_op(*args, **kwargs)

    return kernel
````
- **EN**: This chunk defines `kernel`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `kernel`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 63-82 / 第 63-82 行
````python
def mark_non_differentiable(ctx, output, output_differentiability):
    # Output types are restricted to be:
    # - Tensor
    # - Tensor[]
    # - int, bool, Scalar, float
    # See _check_can_register_backward
    if output_differentiability is not None:
        if not isinstance(output, tuple):
            tuple_output = (output,)
        else:
            tuple_output = output  # type: ignore[assignment]
        if len(output_differentiability) != len(tuple_output):
            raise AssertionError(
                f"output_differentiability length {len(output_differentiability)} "
                f"!= output length {len(tuple_output)}"
            )
        non_differentiable_tensors = []
        for idx, (differentiable, out) in enumerate(
            zip(output_differentiability, tuple_output)
        ):
````
- **EN**: This chunk defines `mark_non_differentiable`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `mark_non_differentiable`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 83-99 / 第 83-99 行
````python
            if isinstance(out, torch.Tensor):
                if not differentiable:
                    non_differentiable_tensors.append(out)
                continue
            if isinstance(out, list):
                if not differentiable:
                    non_differentiable_tensors.extend(out)
                continue
            if differentiable:
                raise RuntimeError(
                    f"With output_differentiability={output_differentiability}. "
                    f"At idx {idx}, we received an object of type {type(out)} that "
                    f"is not a Tensor, so it cannot have be marked as differentiable in "
                    f"output_differentiability."
                )
        if non_differentiable_tensors:
            ctx.mark_non_differentiable(*non_differentiable_tensors)
````
- **EN**: This chunk continues `mark_non_differentiable` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `mark_non_differentiable`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 102-118 / 第 102-118 行
````python
def construct_autograd_kernel(
    schema,
    output_differentiability,
    custom_op,
    op_overload,
    save_for_backward_fn,
    backward_fn,
):
    def apply(*args):
        flat_args, spec = pytree.tree_flatten(args)
        out_spec = None

        def forward(ctx, *flat_args):
            ctx.set_materialize_grads(True)
            args = pytree.tree_unflatten(list(flat_args), spec)
            with torch._C._AutoDispatchBelowAutograd():
                output = op_overload(*args)
````
- **EN**: This chunk defines `forward`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 120-139 / 第 120-139 行
````python
            # We use the info about args to give better error messages in backward
            args_info = namedtuple_args(schema, pytree.tree_map(type, args))

            save_for_backward_fn_inputs = namedtuple_args(schema, args)
            to_save = save_for_backward_fn(save_for_backward_fn_inputs, output)

            save_pytree_for_backward(ctx, (to_save, args_info))
            mark_non_differentiable(ctx, output, output_differentiability)

            nonlocal out_spec
            flat_output, out_spec = pytree.tree_flatten(output)
            return tuple(flat_output)

        def backward(ctx, *flat_grad_output):
            if out_spec is None:
                raise AssertionError("out_spec is unexpectedly None")
            grads = pytree.tree_unflatten(list(flat_grad_output), out_spec)
            saved, args_info = unpack_saved(ctx)
            # There is nothing on the ctx object for now, it is just there so
            # that we can add additional things in the future.
````
- **EN**: This chunk defines `backward`, which implements differentiation-time behavior that complements the forward path. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `backward`，其作用是实现与前向路径配套的求导期行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 140-157 / 第 140-157 行
````python
            inner_ctx = object()
            if not isinstance(grads, tuple):
                grads = (grads,)
            grad_inputs_dict = backward_fn(inner_ctx, saved, *grads)

            # Massage the grad_inputs_dict to a form acceptable by
            # autograd.Function.
            validate_grad_inputs_dict(grad_inputs_dict, custom_op, args_info)
            return grad_inputs_dict_to_flat_tuple(grad_inputs_dict, args_info)

        generated_cls = gen_autograd_function(
            custom_op._opname + "_customop", forward, backward
        )

        flat_output = generated_cls.apply(*flat_args)
        if out_spec is None:
            raise AssertionError("out_spec is unexpectedly None")
        return pytree.tree_unflatten(list(flat_output), out_spec)
````
- **EN**: This chunk continues `backward` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `backward`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 159-171 / 第 159-171 行
````python
    return apply


def gen_autograd_function(name, forward, backward):
    generated_cls = type(
        name,
        (torch.autograd.Function,),
        {
            "forward": staticmethod(forward),
            "backward": staticmethod(backward),
        },
    )
    return generated_cls
````
- **EN**: This chunk defines `gen_autograd_function`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `gen_autograd_function`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 174-187 / 第 174-187 行
````python
@functools.lru_cache
def namedtuple_args_cls(schema):
    attribs = [arg.name for arg in schema.arguments.flat_all]
    name = str(schema.name) + "_args"
    # mypy doesn't support dynamic namedtuple name
    tuple_cls = namedtuple(name, attribs)  # type: ignore[misc]
    return tuple_cls


def namedtuple_args(schema, args):
    if not isinstance(args, tuple):
        raise AssertionError(f"expected tuple, got {type(args)}")
    tuple_cls = namedtuple_args_cls(schema)
    return tuple_cls(*args)
````
- **EN**: This chunk defines `namedtuple_args`, which implements a focused helper used by the surrounding module. Decorators such as `functools.lru_cache` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `namedtuple_args`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.lru_cache` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 190-202 / 第 190-202 行
````python
def validate_grad_inputs_dict(grad_inputs_dict, forward_op, args_info):
    def error(what):
        backward = forward_op._get_impl("backward")
        raise RuntimeError(
            f"In the backward function defined for {forward_op} at "
            f"{backward.location} using the CustomOp API, {what}"
        )

    if not isinstance(grad_inputs_dict, dict):
        error(
            f"expected the output of the backward function to be a dict but "
            f"got {type(grad_inputs_dict)}"
        )
````
- **EN**: This chunk defines `error`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `error`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 204-221 / 第 204-221 行
````python
    expected_keys = {
        arg.name
        for arg in forward_op._schema.arguments.flat_all
        if arg.type.is_tensor_like()
    }
    actual_keys = grad_inputs_dict.keys()
    if expected_keys != actual_keys:
        error(
            f"expected the returned grad_input dict to have keys "
            f"{expected_keys} but got {actual_keys}. The backward "
            f"function must return a gradient (can be None) for each arg "
            f"to the CustomOp that may be a Tensor or Sequence[Tensor]. "
            f"Args declared to be non-Tensor-like types should not appear "
            f"in the grad_input dict"
        )

    for name, grad in grad_inputs_dict.items():
        arg_info = getattr(args_info, name)
````
- **EN**: This chunk continues `error` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `error`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 223-242 / 第 223-242 行
````python
        if isinstance(arg_info, list):
            if not isinstance(grad, (tuple, list)):
                error(
                    f"for input '{name}' expected the grad_input dict to "
                    f"hold a list of gradients but got object of type "
                    f"{type(grad)}."
                )
            if len(grad) != len(arg_info):
                error(
                    f"for input '{name}' expected the grad_input dict to "
                    f"hold a list of {len(arg_info)} gradients but got "
                    f"{len(grad)}"
                )
            for idx, (g, info) in enumerate(zip(grad, arg_info)):
                if g is None:
                    continue
                if not isinstance(g, torch.Tensor):
                    error(
                        f"for input '{name}' expected the grad_input dict to "
                        f"hold a list of None or Tensor gradients but got "
````
- **EN**: This chunk continues `error` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `error`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 243-262 / 第 243-262 行
````python
                        f"object of {type(g)} at index {idx}"
                    )
                if not issubclass(info, torch.Tensor):
                    error(
                        f"for input '{name}', got a Tensor as the gradient "
                        f"for the {idx}-th value but expected None because "
                        f"the {idx}-th value was not a Tensor (it was "
                        f"type {arg_info}"
                    )
            continue

        if grad is None:
            continue
        if not isinstance(grad, torch.Tensor):
            error(
                f"got object of type {type(grad)} as the gradient for input "
                f"'{name}', "
                f"but expected the gradient to be either None or a Tensor"
            )
        if not issubclass(arg_info, torch.Tensor):
````
- **EN**: This chunk continues `error` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `error`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 263-277 / 第 263-277 行
````python
            error(
                f"got a Tensor as the gradient for input '{name}' but "
                f"expected None as the gradient because input '{name}' "
                f"was not a Tensor (it was type {arg_info})."
            )


def grad_inputs_dict_to_flat_tuple(grad_inputs_dict, args_info):
    result = []
    for name, arg_info in args_info._asdict().items():
        if name not in grad_inputs_dict:
            result.append(pytree.tree_map(lambda x: None, arg_info))
            continue
        result.append(grad_inputs_dict[name])
    return tuple(pytree.tree_leaves(result))
````
- **EN**: This chunk defines `grad_inputs_dict_to_flat_tuple`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `grad_inputs_dict_to_flat_tuple`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 280-296 / 第 280-296 行
````python
# Saves "stuff" (a pytree) onto the ctx object. Use unpack_saved to unpack it.
# autograd.Function prefers that users use ctx.save_for_backward to
# save Tensors (to avoid reference cycles) and for non-Tensors to go onto the
# ctx object.
def save_pytree_for_backward(ctx, stuff):
    flat_stuff, spec = pytree.tree_flatten(stuff)
    num_elts = len(flat_stuff)
    tensor_idxs = [
        idx for idx, thing in enumerate(flat_stuff) if isinstance(thing, torch.Tensor)
    ]
    non_tensor_idxs = [
        idx
        for idx, thing in enumerate(flat_stuff)
        if not isinstance(thing, torch.Tensor)
    ]
    tensors = [thing for thing in flat_stuff if isinstance(thing, torch.Tensor)]
    non_tensors = [thing for thing in flat_stuff if not isinstance(thing, torch.Tensor)]
````
- **EN**: This chunk defines `save_pytree_for_backward`, which serializes or reconstructs state across a Python-visible boundary. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `save_pytree_for_backward`，其作用是在 Python 可见边界上序列化或重建状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 298-314 / 第 298-314 行
````python
    ctx.spec = spec
    ctx.num_elts = num_elts
    ctx.save_for_backward(*tensors)
    ctx.tensor_idxs = tensor_idxs
    ctx.saved_non_tensors = non_tensors
    ctx.non_tensor_idxs = non_tensor_idxs


# Inverse operation to save_pytree_for_backward
def unpack_saved(ctx):
    flat_stuff = [None] * ctx.num_elts
    for tensor, idx in zip(ctx.saved_tensors, ctx.tensor_idxs):
        flat_stuff[idx] = tensor
    for non_tensor, idx in zip(ctx.saved_non_tensors, ctx.non_tensor_idxs):
        flat_stuff[idx] = non_tensor
    stuff = pytree.tree_unflatten(flat_stuff, ctx.spec)
    return stuff
````
- **EN**: This chunk defines `unpack_saved`, which serializes or reconstructs state across a Python-visible boundary. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `unpack_saved`，其作用是在 Python 可见边界上序列化或重建状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **autograd_kernel_indirection**
  - EN: `autograd_kernel_indirection` is one of the main symbols declared or implemented in this file.
  - CN: `autograd_kernel_indirection` 是本文件声明或实现的主要符号之一。
- **autograd_not_implemented**
  - EN: `autograd_not_implemented` is one of the main symbols declared or implemented in this file.
  - CN: `autograd_not_implemented` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.utils._pytree`
- **Standard library / 标准库**: `functools`, `collections`
- **Primary symbols in this file / 本文件核心符号**: `autograd_kernel_indirection`, `autograd_not_implemented`, `mark_non_differentiable`, `construct_autograd_kernel`, `gen_autograd_function`, `namedtuple_args_cls`, `namedtuple_args`, `validate_grad_inputs_dict`, `grad_inputs_dict_to_flat_tuple`, `save_pytree_for_backward`
