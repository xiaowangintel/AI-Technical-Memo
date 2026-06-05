# decompositions_for_jvp.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_decomp/decompositions_for_jvp.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines decomposition tables and helpers that rewrite higher-level operators into simpler building blocks.
- **Purpose (CN)**: 定义 decomposition 表和辅助逻辑，把高层算子重写为更基础的构件。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
````python
# mypy: allow-untyped-decorators
# mypy: allow-untyped-defs
import inspect
from collections.abc import Callable

import torch
import torch._decomp
from torch import Tensor
from torch._prims_common.wrappers import _maybe_remove_out_wrapper


decomposition_table = torch._decomp.decomposition_table
decomposition_table_for_jvp: dict[torch._ops.OperatorBase, Callable] = {}
register_decomposition = torch._decomp.register_decomposition
aten = torch.ops.aten
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._decomp, torch._prims_common.wrappers; standard-library helpers such as inspect, collections.abc.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._decomp、torch._prims_common.wrappers；标准库辅助模块，如 inspect、collections.abc。

### Lines 17-36 / 第 17-36 行
````python
# NOTE: [forward-mode AD decompositions mechanism]
#
# The mechanism is in VariableType,
#   IF any inputs have forward grad
#      AND there is no forward AD formula implemented
#      AND the functions are actually differentiable
#   run the decomposition
#      See run_jit_decomposition_with_args_for_jvp
#      We currently use python decompositions that we torchscript.
#
# Note that we would be building the backward graph at the decomposed level
# too, but that is OK, because we would've errored out otherwise anyway.
#
# TODO: The mechanism we are using to register decompositions doesn't
# seem to be exclusively used for jvp. So open question here is whether
# torch/csrc/jit/runtime/decomposition_registry.cpp is being used for other things.
# If that is the case, we may go down the decomposition path unexpectedly
# (and possibly produce an unintelligible error) vs erroring out earlier and
# printing that the forward AD formula is not implemented.
#
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 37-54 / 第 37-54 行
````python
# The solution to this may be to have an explicitly white list control when
# to enable the decomposition.


def maybe_register_decomposition(op):
    def decorator(f):
        try:
            return register_decomposition(op)(f)
        except Exception:
            return f

    return decorator


# Functions where we need a special decomposition for jvp but there's another version that
# should be used more generally (ex. for jvp we need to recompute the mean and variance for
# the backwards of a normalization function. Without jvp, it should use the saved value)
decomposition_table_for_jvp = {}
````
- **EN**: This chunk defines `decorator`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `decorator`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 57-73 / 第 57-73 行
````python
def register_decomposition_for_jvp(fn):
    return register_decomposition(fn, registry=decomposition_table_for_jvp)


def _register_jit_decomposition_for_jvp(decomp, use_python=False):
    if decomp in decomposition_table_for_jvp:
        decomposition_table_used = decomposition_table_for_jvp
    elif decomp in decomposition_table:
        decomposition_table_used = decomposition_table
    else:
        raise RuntimeError(f"could not find decomposition for {decomp}")
    decomp_fn = decomposition_table_used[decomp]

    # `out_wrapper` extends a decompositions signature with
    # an `out` parameter. However jit will use the unwrapped function's
    # signature instead so we need to unwrap here to prevent an error
    decomp_fn = _maybe_remove_out_wrapper(decomp_fn)
````
- **EN**: This chunk defines `_register_jit_decomposition_for_jvp`, which registers a hook, schema, operator, or callback with surrounding infrastructure. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_register_jit_decomposition_for_jvp`，其作用是向周边基础设施注册钩子、schema、算子或回调。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 75-88 / 第 75-88 行
````python
    if use_python:
        decomp_fn = torch.jit.ignore(decomp_fn)
        sig = inspect.signature(decomp_fn)

        # Create a string wrapping the function from the signature
        # example output:
        # def wrapped_decomp(x: torch.Tensor, y: int, z: int):
        #   return decomp_fn(x, y, z)
        # Thanks copilot!
        def get_function_def(sig):
            param_def = [f"{param_str}" for param_str in sig.parameters.values()]
            param_use = [f"{param_str}" for param_str in sig.parameters]

            return f"def wrapped_decomp({', '.join(param_def)}):\n  return decomp_fn({', '.join(param_use)})\n"
````
- **EN**: This chunk defines `get_function_def`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_function_def`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 90-103 / 第 90-103 行
````python
        f_str = get_function_def(sig)
        graph = torch.jit.CompilationUnit(f_str).wrapped_decomp.graph
    else:
        graph = torch.jit.script(decomp_fn).graph
    torch.jit._register_decomposition(decomp, graph)


# The only decompositions here are temporary or hacks for the purposes of jvp


# TODO: do these also belong here?
@maybe_register_decomposition(aten.trace.default)
def trace(self: Tensor) -> Tensor:
    return torch.sum(torch.diag(self))
````
- **EN**: This chunk defines `trace`, which coordinates graph-oriented state needed for tracing, capture, or compilation. Decorators such as `maybe_register_decomposition` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `trace`，其作用是协调 tracing、捕获或编译所需的图相关状态。 像 `maybe_register_decomposition` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 106-121 / 第 106-121 行
````python
@maybe_register_decomposition(aten.log_sigmoid_forward.default)
def log_sigmoid_forward(self: Tensor) -> tuple[Tensor, Tensor]:
    min = torch.minimum(self.new_zeros(()), self)
    z = torch.exp(-torch.abs(self))
    if self.is_cuda or self.is_xpu:
        buffer = self.new_zeros((0,))
    else:
        buffer = z
    return min - torch.log1p(z), buffer


def recompute_mean_var(
    input: Tensor, rstd: Tensor, inner_dim_indices: list[int], keepdim: bool
):
    # for most norm decompositions, it will be the same as the core version except for here.
    # We recompute the mean and variance so that they track gradients through input
````
- **EN**: This chunk defines `recompute_mean_var`, which implements a focused helper used by the surrounding module. Decorators such as `maybe_register_decomposition` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `recompute_mean_var`，其作用是实现周边模块使用的关键辅助逻辑。 像 `maybe_register_decomposition` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 123-142 / 第 123-142 行
````python
    mean = torch.mean(input, dim=inner_dim_indices, keepdim=keepdim)
    var = torch.var(input, dim=inner_dim_indices, unbiased=False, keepdim=keepdim)
    eps = torch.pow(1 / rstd, 2) - var  # this makes me so sad inside
    eps = eps.detach()
    rstd = 1 / torch.sqrt(var + eps)
    return mean, rstd


@register_decomposition_for_jvp(aten.native_layer_norm_backward)
def native_layer_norm_backward(
    grad_out: Tensor,
    input: Tensor,
    normalized_shape: list[int],
    mean: Tensor,
    rstd: Tensor,
    weight: Tensor | None,
    bias: Tensor | None,
    output_mask: list[bool],
) -> tuple[Tensor | None, Tensor | None, Tensor | None]:
    input_shape = input.shape
````
- **EN**: This chunk defines `native_layer_norm_backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `register_decomposition_for_jvp` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `native_layer_norm_backward`，其作用是实现与前向路径配套的求导期行为。 像 `register_decomposition_for_jvp` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 143-162 / 第 143-162 行
````python
    input_ndim = input.dim()

    axis = input_ndim - len(normalized_shape)
    inner_dims = input_shape[axis:]
    outer_dims = input_shape[:axis]
    inner_dim_indices = list(range(axis, input_ndim))
    outer_dim_indices = list(range(axis))

    N = 1
    for i in inner_dims:
        N *= i
    M = 1
    for i in outer_dims:
        M *= i
    if M <= 0 or N <= 0:
        return (
            input.new_zeros(input_shape),
            input.new_zeros(input_shape[axis:]),
            input.new_zeros(input_shape[axis:]),
        )
````
- **EN**: This chunk continues `native_layer_norm_backward` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `native_layer_norm_backward`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 164-181 / 第 164-181 行
````python
    mean_, rstd_ = recompute_mean_var(input, rstd, inner_dim_indices, keepdim=True)

    x_hat = (input - mean_) * rstd_
    if weight is not None:
        grad_x_hat = grad_out * weight
    else:
        grad_x_hat = grad_out
    a = grad_x_hat * N
    b = torch.sum(grad_x_hat, inner_dim_indices, True)
    c1 = torch.mul(grad_x_hat, x_hat)
    c2 = torch.sum(c1, inner_dim_indices, True)
    c3 = torch.mul(x_hat, c2)
    inner = a - b - c3

    if output_mask[0]:
        d_input: Tensor | None = (rstd_ / N) * inner
    else:
        d_input = torch.zeros_like(input)  # should be None but doesn't work with vjp
````
- **EN**: This chunk continues `native_layer_norm_backward` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `native_layer_norm_backward`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 183-202 / 第 183-202 行
````python
    if output_mask[1] and weight is not None:
        if len(outer_dim_indices) > 0:
            d_weight: Tensor | None = torch.sum(
                grad_out * x_hat, outer_dim_indices, False
            )
        else:
            d_weight = grad_out * x_hat
    elif weight is not None:
        d_weight = torch.zeros_like(weight)  # should be None but doesn't work with vjp
    else:
        d_weight = torch.zeros(())  # should be None but doesn't work with vjp

    if output_mask[2] and bias is not None:
        if len(outer_dim_indices) > 0:
            d_bias: Tensor | None = torch.sum(grad_out, outer_dim_indices, False)
        else:
            d_bias = grad_out.clone()
    elif bias is not None:
        d_bias = torch.zeros_like(bias)  # should be None but doesn't work with vjp
    else:
````
- **EN**: This chunk continues `native_layer_norm_backward` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `native_layer_norm_backward`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 203-222 / 第 203-222 行
````python
        d_bias = torch.zeros(())  # should be None but doesn't work with vjp

    return (d_input, d_weight, d_bias)


def prod(x: list[int]):
    r = 1
    for i in x:
        r *= i
    return r


@register_decomposition_for_jvp(aten.native_batch_norm_backward)
def native_batch_norm_backward(
    grad_out: Tensor,
    input: Tensor,
    weight: Tensor | None,
    running_mean: Tensor | None,
    running_var: Tensor | None,
    save_mean: Tensor | None,
````
- **EN**: This chunk defines `native_batch_norm_backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `register_decomposition_for_jvp` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `native_batch_norm_backward`，其作用是实现与前向路径配套的求导期行为。 像 `register_decomposition_for_jvp` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 223-241 / 第 223-241 行
````python
    save_invstd: Tensor | None,
    train: bool,
    eps: float,
    output_mask: list[bool],
) -> tuple[Tensor, Tensor | None, Tensor | None]:
    input_shape = input.shape
    input_rank = input.dim()
    if input_rank < 2:
        raise AssertionError(f"rank of the input must be at least 2, got {input_rank}")

    axis = 1
    num_features = prod(input_shape) / input_shape[axis]  # type: ignore[arg-type]
    mean = save_mean
    invstd = save_invstd
    if train:
        if save_mean is None or save_invstd is None:
            raise AssertionError(
                "when train=True, save_mean and save_invstd are required"
            )
````
- **EN**: This chunk continues `native_batch_norm_backward` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `native_batch_norm_backward`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 243-261 / 第 243-261 行
````python
        reduciton_dims = [0] + list(range(2, input.dim()))
        if invstd is None:
            raise AssertionError("invstd must not be None for typing")
        mean, invstd = recompute_mean_var(input, invstd, reduciton_dims, keepdim=False)
    else:
        if running_mean is None or running_var is None:
            raise AssertionError(
                "running_mean and running_var must not be None when train=False"
            )
        mean = running_mean
        invstd = torch.rsqrt(running_var + eps)

    if invstd is None or mean is None:
        raise AssertionError(
            f"invstd and mean must not be None, got invstd={invstd}, mean={mean}"
        )

    broadcast_mask = [1] * input_rank
    broadcast_mask[axis] = input_shape[axis]
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk continues `native_batch_norm_backward` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段延续了 `native_batch_norm_backward`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 263-279 / 第 263-279 行
````python
    reduction_axes: list[int] = []
    for i in range(input_rank):
        if i != axis:
            reduction_axes.append(i)

    mean = torch.reshape(mean, broadcast_mask)
    norm = 1.0 / num_features
    grad_output_sum = torch.sum(grad_out, reduction_axes)
    dot_p = torch.sum(grad_out * (input - mean), reduction_axes)

    grad_mean = torch.reshape(grad_output_sum * norm, broadcast_mask)
    proj_scale = torch.reshape(torch.mul(dot_p * norm, invstd * invstd), broadcast_mask)

    if weight is None:
        grad_scale = torch.reshape(invstd, broadcast_mask) * 1.0
    else:
        grad_scale = torch.reshape(invstd * weight, broadcast_mask)
````
- **EN**: This chunk continues `native_batch_norm_backward` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `native_batch_norm_backward`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 281-294 / 第 281-294 行
````python
    if train:
        proj = (input - mean) * proj_scale
        grad_input = ((grad_out - proj) - grad_mean) * grad_scale
    else:
        grad_input = grad_out * grad_scale

    if output_mask[1]:
        grad_weight = dot_p * invstd
    elif weight is not None:
        grad_weight = torch.zeros_like(
            weight
        )  # should be None but doesn't work with vjp
    else:
        grad_weight = torch.zeros(())  # should be None but doesn't work with vjp
````
- **EN**: This chunk continues `native_batch_norm_backward` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `native_batch_norm_backward`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 296-315 / 第 296-315 行
````python
    if output_mask[2]:
        grad_bias = grad_output_sum
    else:
        grad_bias = torch.zeros_like(
            grad_output_sum
        )  # should be None but doesn't work with vjp

    return (grad_input, grad_weight, grad_bias)


@register_decomposition_for_jvp(aten.batch_norm_backward)
def batch_norm_backward(
    grad_out: Tensor,
    input: Tensor,
    weight: Tensor,
    running_mean: Tensor | None,
    running_var: Tensor | None,
    save_mean: Tensor | None,
    save_var: Tensor | None,
    update: bool,
````
- **EN**: This chunk defines `batch_norm_backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `register_decomposition_for_jvp` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `batch_norm_backward`，其作用是实现与前向路径配套的求导期行为。 像 `register_decomposition_for_jvp` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 316-331 / 第 316-331 行
````python
    eps: float,
    output_mask: list[bool],
    reserve: Tensor,
) -> tuple[Tensor, Tensor | None, Tensor | None]:
    return native_batch_norm_backward(
        grad_out,
        input,
        weight,
        running_mean,
        running_var,
        save_mean,
        save_var,
        update,
        eps,
        output_mask,
    )
````
- **EN**: This chunk continues `batch_norm_backward` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `batch_norm_backward`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 334-344 / 第 334-344 行
````python
_register_jit_decomposition_for_jvp(torch.ops.aten.trace.default, use_python=True)
_register_jit_decomposition_for_jvp(torch.ops.aten.nll_loss_backward.default)
_register_jit_decomposition_for_jvp(torch.ops.aten.nll_loss2d_backward.default)
_register_jit_decomposition_for_jvp(torch.ops.aten._log_softmax_backward_data.default)
_register_jit_decomposition_for_jvp(torch.ops.aten._softmax_backward_data.default)
_register_jit_decomposition_for_jvp(torch.ops.aten.log_sigmoid_forward.default)
_register_jit_decomposition_for_jvp(torch.ops.aten.native_layer_norm_backward.default)
_register_jit_decomposition_for_jvp(torch.ops.aten.native_batch_norm_backward.default)
_register_jit_decomposition_for_jvp(torch.ops.aten.cudnn_batch_norm_backward.default)
_register_jit_decomposition_for_jvp(torch.ops.aten.batch_norm_backward.default)
_register_jit_decomposition_for_jvp(torch.ops.aten.miopen_batch_norm_backward.default)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

## Key Concepts / 关键概念

- **Operator decomposition**
  - EN: Rewrites composite operators into simpler primitives used by tracing and compilation flows.
  - CN: 把复合算子重写为 tracing 与编译流程可用的更基础 primitive。
- **maybe_register_decomposition**
  - EN: `maybe_register_decomposition` is one of the main symbols declared or implemented in this file.
  - CN: `maybe_register_decomposition` 是本文件声明或实现的主要符号之一。
- **register_decomposition_for_jvp**
  - EN: `register_decomposition_for_jvp` is one of the main symbols declared or implemented in this file.
  - CN: `register_decomposition_for_jvp` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Decomposition**
  - EN: The implementation rewrites larger operators into smaller primitives for reuse and lowering.
  - CN: 实现把较大的算子重写为更小的 primitive，以便复用和降级。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._decomp`, `torch._prims_common.wrappers`
- **Standard library / 标准库**: `inspect`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `maybe_register_decomposition`, `register_decomposition_for_jvp`, `_register_jit_decomposition_for_jvp`, `trace`, `log_sigmoid_forward`, `recompute_mean_var`, `native_layer_norm_backward`, `prod`, `native_batch_norm_backward`, `batch_norm_backward`
