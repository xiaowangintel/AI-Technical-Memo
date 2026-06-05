# rng_prims.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_prims/rng_prims.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines primitive operators, context objects, and helper logic used by decomposition and tracing flows.
- **Purpose (CN)**: 定义 primitive 算子、上下文对象以及 decomposition/tracing 流程使用的辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
````python
# mypy: allow-untyped-defs
from typing import cast

import torch
import torch.utils._pytree as pytree
from torch import _prims
from torch._C import DispatchKey
from torch._higher_order_ops.utils import autograd_not_implemented
from torch._ops import HigherOrderOperator
from torch._prims_common import CUDARngStateHelper, make_contiguous_strides_for
from torch._subclasses.fake_tensor import FakeTensorMode
from torch.fx.experimental.proxy_tensor import (
    disable_proxy_modes_tracing,
    ProxyTorchDispatchMode,
    track_tensor_tree,
)
from torch.types import _device, _dtype
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.utils._pytree, torch._C, ...; standard-library helpers such as typing.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.utils._pytree、torch._C、...；标准库辅助模块，如 typing。

### Lines 20-38 / 第 20-38 行
````python
def throw_on_non_cuda(device):
    raise RuntimeError(
        f"You are trying to functionalize a {device.type} RNG operator but {device.type} does not "
        f"use Philox/counter-based RNG. Therefore, functionalizing a {device.type} RNG operator is "
        "not supported. We are discussing the possibility of a Philox-based RNG implementation for CPU."
    )


def register_rng_prim(name, schema, impl_aten, impl_meta, doc, tags=None):
    rngprim_def = torch.library.custom_op(
        "rngprims::" + name, impl_aten, mutates_args=(), schema=schema
    )

    rngprim_def.register_fake(impl_meta)

    prim_packet = getattr(torch._ops.ops.rngprims, name)
    prim = prim_packet.default
    if tags:
        prim._tags = tags
````
- **EN**: This chunk defines `register_rng_prim`, which registers a hook, schema, operator, or callback with surrounding infrastructure. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `register_rng_prim`，其作用是向周边基础设施注册钩子、schema、算子或回调。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 40-54 / 第 40-54 行
````python
    for p in (prim_packet, prim):
        p.__doc__ = doc
        p.return_type = torch._prims_common.RETURN_TYPE.NEW  # type: ignore[attr-defined]

        p.schema = name + schema
        p.impl_aten = impl_aten
        p.prim_meta_impl = impl_meta


# Philox rand offsets could be shared in future with other philox ops, so
# keeping these functions in global scope.
def philox_rand_offset_meta(
    shape: torch.Size,
):
    return _prims.TensorLike(torch.tensor(0, dtype=torch.int64))
````
- **EN**: This chunk defines `philox_rand_offset_meta`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `philox_rand_offset_meta`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 57-76 / 第 57-76 行
````python
def philox_rand_offset(
    shape: torch.Size,
):
    # For impl, look at the function calc_execution_policy in the file
    # aten/src/ATen/native/cuda/DistributionTemplates.h. The impl was copied at
    # commit hash 72aa0667bd16707d50eb8fa337092a1f5d11dfb6
    numel_scalar = 1
    for dim_size in shape:
        numel_scalar *= dim_size
    numel = torch.scalar_tensor(numel_scalar, dtype=torch.int64)

    block_size = 256
    unroll = 4
    curand4_engine_calls = 4
    device_property = torch.cuda.get_device_properties(torch.cuda.current_device())
    blocks_per_sm = device_property.max_threads_per_multi_processor // block_size
    num = cast(int, numel)
    grid_size = (num + block_size - 1) // block_size
    grid_size = min(grid_size, device_property.multi_processor_count * blocks_per_sm)
    return ((num - 1) // (block_size * grid_size * unroll) + 1) * curand4_engine_calls
````
- **EN**: This chunk defines `philox_rand_offset`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `philox_rand_offset`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 79-98 / 第 79-98 行
````python
def register_philox_rand():
    name = "philox_rand"
    schema = "(SymInt[] size, Tensor seed, Tensor offset, int[]? stride, Device? device=None, ScalarType? dtype=None) -> (Tensor, Tensor)"

    def _philox_rand_meta(
        shape: torch.Size,
        seed: torch.Tensor,
        offset: torch.Tensor,
        stride: tuple[int, ...] | None,
        device: _device,
        dtype: _dtype,
    ):
        # stride arg will be useful for distributed usecase. Currently, its unused.
        if stride is not None:
            raise AssertionError(f"stride must be None, got {stride}")
        stride = make_contiguous_strides_for(shape)
        random_values = _prims.TensorMeta(
            shape=shape, strides=stride, dtype=dtype, device=device
        )
        offset = philox_rand_offset_meta(shape)
````
- **EN**: This chunk defines `_philox_rand_meta`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_philox_rand_meta`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 99-115 / 第 99-115 行
````python
        return (random_values, offset)

    def _philox_rand(
        shape: torch.Size,
        seed: torch.Tensor,
        offset: torch.Tensor,
        stride: tuple[int, ...] | None,
        device: _device,
        dtype: _dtype,
    ):
        # stride arg will be useful for distributed usecase. Currently, its unused.
        if stride is not None:
            raise AssertionError(f"stride must be None, got {stride}")
        if device.type == "cpu":
            devices = []
        else:
            devices = [device]
````
- **EN**: This chunk defines `_philox_rand`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_philox_rand`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 117-133 / 第 117-133 行
````python
        if device.type != "cuda":
            raise throw_on_non_cuda(device)

        with torch.random.fork_rng(devices):
            CUDARngStateHelper.set_torch_state_tensor(seed, offset)
            random_values = torch.rand(shape, device=device, dtype=dtype)

        return random_values, philox_rand_offset(shape)

    register_rng_prim(
        name=name,
        schema=schema,
        impl_aten=_philox_rand,
        impl_meta=_philox_rand_meta,
        doc="Philox based stateless rand operator",
        tags=(torch.Tag.nondeterministic_seeded,),
    )
````
- **EN**: This chunk continues `_philox_rand` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_philox_rand`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 136-152 / 第 136-152 行
````python
def get_device(args, kwargs):
    if kwargs.get("device"):
        device = kwargs.get("device")
        if isinstance(device, str):
            device = torch.device(device)
        return device.type

    devices = {arg.device.type for arg in args if isinstance(arg, torch.Tensor)}
    if any(dev == "cuda" for dev in devices):
        return "cuda"
    elif any(dev == "xpu" for dev in devices):
        return "xpu"
    elif any(dev == "hpu" for dev in devices):
        return "hpu"
    elif any(dev == "cpu" for dev in devices):
        return "cpu"
    return None
````
- **EN**: This chunk defines `get_device`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_device`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 155-172 / 第 155-172 行
````python
def register_run_and_save_rng_state_op():
    class RunAndSaveRngState(HigherOrderOperator):
        def __init__(self):
            super().__init__("run_and_save_rng_state", cacheable=True)

        def __call__(self, op, *args, **kwargs):
            # pyrefly: ignore [missing-attribute]
            return super().__call__(op, *args, **kwargs)

    run_and_save_rng_state = RunAndSaveRngState()

    run_and_save_rng_state.py_impl(DispatchKey.Autograd)(
        autograd_not_implemented(run_and_save_rng_state, deferred_error=True)
    )

    @run_and_save_rng_state.py_impl(DispatchKey.CUDA)
    def impl_cuda(op, *args, **kwargs):
        return torch.cuda.get_rng_state(), op(*args, **kwargs)
````
- **EN**: It introduces or extends `RunAndSaveRngState`, which hold the main object-oriented state for this portion of the file. This chunk defines `impl_cuda`, which implements a focused helper used by the surrounding module. Decorators such as `run_and_save_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `RunAndSaveRngState`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `impl_cuda`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_and_save_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 174-186 / 第 174-186 行
````python
    @run_and_save_rng_state.py_impl(DispatchKey.CPU)
    def impl_cpu(op, *args, **kwargs):
        return torch.get_rng_state(), op(*args, **kwargs)

    @run_and_save_rng_state.py_impl(DispatchKey.HPU)
    def impl_hpu(op, *args, **kwargs):
        if hasattr(torch, "hpu"):
            return torch.hpu.get_rng_state(), op(*args, **kwargs)
        raise RuntimeError("functionalize a hpu RNG operator is not supported.")

    @run_and_save_rng_state.py_impl(DispatchKey.XPU)
    def impl_xpu(op, *args, **kwargs):
        return torch.xpu.get_rng_state(), op(*args, **kwargs)
````
- **EN**: This chunk defines `impl_xpu`, which implements a focused helper used by the surrounding module. Decorators such as `run_and_save_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_xpu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_and_save_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 188-206 / 第 188-206 行
````python
    @run_and_save_rng_state.py_impl(DispatchKey.BackendSelect)
    def impl_backend_select(op, *args, **kwargs):
        impl_map = {
            "cuda": impl_cuda,
            "cpu": impl_cpu,
            "hpu": impl_hpu,
            "xpu": impl_xpu,
        }
        device = get_device(args, kwargs)
        if device not in impl_map:
            raise AssertionError(f"Backend not supported for {device}")
        impl = impl_map[device]
        return impl(op, *args, **kwargs)

    @run_and_save_rng_state.py_impl(FakeTensorMode)
    def impl_fake_tensor_mode(mode, op, *args, **kwargs):
        # Check device to call the right impl
        with mode:
            return impl_backend_select(op, *args, **kwargs)
````
- **EN**: This chunk defines `impl_fake_tensor_mode`, which implements a focused helper used by the surrounding module. Decorators such as `run_and_save_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_fake_tensor_mode`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_and_save_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 208-224 / 第 208-224 行
````python
    @run_and_save_rng_state.py_impl(ProxyTorchDispatchMode)
    def impl_proxy_dispatch_mode(mode, op, *args, **kwargs):
        out = impl_backend_select(op, *args, **kwargs)
        proxy_args = pytree.tree_map(mode.tracer.unwrap_proxy, (op, *args))
        proxy_kwargs = pytree.tree_map(mode.tracer.unwrap_proxy, kwargs)
        out_proxy = mode.tracer.create_proxy(
            "call_function", run_and_save_rng_state, proxy_args, proxy_kwargs
        )
        return track_tensor_tree(out, out_proxy, constant=None, tracer=mode.tracer)

    return run_and_save_rng_state


def register_run_with_rng_state_op():
    class RunWithRngState(HigherOrderOperator):
        def __init__(self):
            super().__init__("run_with_rng_state", cacheable=True)
````
- **EN**: It introduces or extends `RunWithRngState`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Decorators such as `run_and_save_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `RunWithRngState`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 像 `run_and_save_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 226-242 / 第 226-242 行
````python
        def __call__(self, rng_state, op, *args, **kwargs):
            # pyrefly: ignore [missing-attribute]
            return super().__call__(rng_state, op, *args, **kwargs)

    run_with_rng_state = RunWithRngState()

    run_with_rng_state.py_impl(DispatchKey.Autograd)(
        autograd_not_implemented(run_with_rng_state, deferred_error=True)
    )

    @run_with_rng_state.py_impl(DispatchKey.CUDA)
    def impl_cuda(rng_state, op, *args, **kwargs):
        current_state = torch.cuda.get_rng_state()
        torch.cuda.set_rng_state(rng_state.cpu())
        out = op(*args, **kwargs)
        torch.cuda.set_rng_state(current_state)
        return out
````
- **EN**: This chunk defines `impl_cuda`, which implements a focused helper used by the surrounding module. Decorators such as `run_with_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_cuda`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_with_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 244-260 / 第 244-260 行
````python
    @run_with_rng_state.py_impl(DispatchKey.CPU)
    def impl_cpu(rng_state, op, *args, **kwargs):
        current_state = torch.get_rng_state()
        torch.set_rng_state(rng_state)
        out = op(*args, **kwargs)
        torch.set_rng_state(current_state)
        return out

    @run_with_rng_state.py_impl(DispatchKey.HPU)
    def impl_hpu(rng_state, op, *args, **kwargs):
        if hasattr(torch, "hpu"):
            current_state = torch.hpu.get_rng_state()
            torch.hpu.set_rng_state(rng_state)
            out = op(*args, **kwargs)
            torch.hpu.set_rng_state(current_state)
            return out
        raise RuntimeError("functionalize a hpu RNG operator is not supported.")
````
- **EN**: This chunk defines `impl_hpu`, which implements a focused helper used by the surrounding module. Decorators such as `run_with_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_hpu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_with_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 262-281 / 第 262-281 行
````python
    @run_with_rng_state.py_impl(DispatchKey.XPU)
    def impl_xpu(rng_state, op, *args, **kwargs):
        current_state = torch.xpu.get_rng_state()
        torch.xpu.set_rng_state(rng_state)
        out = op(*args, **kwargs)
        torch.xpu.set_rng_state(current_state)
        return out

    @run_with_rng_state.py_impl(ProxyTorchDispatchMode)
    def impl_proxy_dispatch_mode(mode, rng_state, op, *args, **kwargs):
        # TODO: you don't need to do this, the dispatch here already disabled
        # it
        with disable_proxy_modes_tracing():
            out = run_with_rng_state(rng_state, op, *args, **kwargs)
        proxy_args = pytree.tree_map(mode.tracer.unwrap_proxy, (rng_state, op, *args))
        proxy_kwargs = pytree.tree_map(mode.tracer.unwrap_proxy, kwargs)
        out_proxy = mode.tracer.create_proxy(
            "call_function", run_with_rng_state, proxy_args, proxy_kwargs
        )
        return track_tensor_tree(out, out_proxy, constant=None, tracer=mode.tracer)
````
- **EN**: This chunk defines `impl_proxy_dispatch_mode`, which implements a focused helper used by the surrounding module. Decorators such as `run_with_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_proxy_dispatch_mode`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_with_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 283-295 / 第 283-295 行
````python
    @run_with_rng_state.py_impl(DispatchKey.BackendSelect)
    def impl_backend_select(rng_state, op, *args, **kwargs):
        impl_map = {
            "cuda": impl_cuda,
            "cpu": impl_cpu,
            "hpu": impl_hpu,
            "xpu": impl_xpu,
        }
        device = get_device(args, kwargs)
        if device not in impl_map:
            raise AssertionError(f"Backend not supported for {device}")
        impl = impl_map[device]
        return impl(rng_state, op, *args, **kwargs)
````
- **EN**: This chunk defines `impl_backend_select`, which implements a focused helper used by the surrounding module. Decorators such as `run_with_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_backend_select`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_with_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 297-314 / 第 297-314 行
````python
    @run_with_rng_state.py_impl(FakeTensorMode)
    def impl_fake_tensor_mode(mode, rng_state, op, *args, **kwargs):
        # Skip setting the set_rng_state as it does not work well with fake tensors.
        # And it does not matter for the fake tensor mode.
        with mode:
            return op(*args, **kwargs)

    @run_with_rng_state.py_functionalize_impl
    def impl_functional(ctx, rng_state, op, *args, **kwargs):
        unwrapped_rng_state = ctx.unwrap_tensors(rng_state)
        unwrapped_args = ctx.unwrap_tensors(args)
        unwrapped_kwargs = ctx.unwrap_tensors(kwargs)

        with ctx.redispatch_to_next():
            out = run_with_rng_state(
                unwrapped_rng_state, op, *unwrapped_args, **unwrapped_kwargs
            )
            return ctx.wrap_tensors(out)
````
- **EN**: This chunk defines `impl_functional`, which implements a focused helper used by the surrounding module. Decorators such as `run_with_rng_state.py_impl`, `run_with_rng_state.py_functionalize_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_functional`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_with_rng_state.py_impl`、`run_with_rng_state.py_functionalize_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 316-332 / 第 316-332 行
````python
    return run_with_rng_state


run_and_save_rng_state = register_run_and_save_rng_state_op()
run_with_rng_state = register_run_with_rng_state_op()


def register_graphsafe_run_with_rng_state_op():
    class GraphSafeRunWithRngState(HigherOrderOperator):
        def __init__(self):
            super().__init__("graphsafe_run_with_rng_state")

        def __call__(self, op, *args, rng_state=None, **kwargs):
            # pyrefly: ignore [missing-attribute]
            return super().__call__(op, *args, rng_state=rng_state, **kwargs)

    graphsafe_run_with_rng_state = GraphSafeRunWithRngState()
````
- **EN**: It introduces or extends `GraphSafeRunWithRngState`, which hold the main object-oriented state for this portion of the file. This chunk defines `__call__`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `GraphSafeRunWithRngState`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__call__`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 334-348 / 第 334-348 行
````python
    graphsafe_run_with_rng_state.py_impl(DispatchKey.Autograd)(
        autograd_not_implemented(graphsafe_run_with_rng_state, deferred_error=True)
    )

    @graphsafe_run_with_rng_state.py_impl(DispatchKey.CUDA)
    def impl_cuda(op, *args, rng_state=None, **kwargs):
        # pyrefly: ignore [missing-attribute]
        device_idx = rng_state.device.index
        generator = torch.cuda.default_generators[device_idx]
        current_state = generator.graphsafe_get_state()

        generator.graphsafe_set_state(rng_state)
        out = op(*args, **kwargs)
        generator.graphsafe_set_state(current_state)
        return out
````
- **EN**: This chunk defines `impl_cuda`, which implements a focused helper used by the surrounding module. Decorators such as `graphsafe_run_with_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_cuda`，其作用是实现周边模块使用的关键辅助逻辑。 像 `graphsafe_run_with_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 350-362 / 第 350-362 行
````python
    @graphsafe_run_with_rng_state.py_impl(DispatchKey.BackendSelect)
    def impl_backend_select(op, *args, rng_state=None, **kwargs):
        device = get_device(args, kwargs)
        if device != "cuda":
            raise AssertionError(
                f"GraphSafe RNG operations only supported for CUDA, got {device}"
            )
        return impl_cuda(op, *args, rng_state=rng_state, **kwargs)

    @graphsafe_run_with_rng_state.py_impl(FakeTensorMode)
    def impl_fake_tensor_mode(mode, op, *args, rng_state=None, **kwargs):
        with mode:
            return op(*args, **kwargs)
````
- **EN**: This chunk defines `impl_fake_tensor_mode`, which implements a focused helper used by the surrounding module. Decorators such as `graphsafe_run_with_rng_state.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_fake_tensor_mode`，其作用是实现周边模块使用的关键辅助逻辑。 像 `graphsafe_run_with_rng_state.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 364-383 / 第 364-383 行
````python
    @graphsafe_run_with_rng_state.py_impl(ProxyTorchDispatchMode)
    def impl_proxy_dispatch_mode(mode, op, *args, rng_state=None, **kwargs):
        with disable_proxy_modes_tracing():
            out = graphsafe_run_with_rng_state(op, *args, rng_state=rng_state, **kwargs)
        proxy_args = pytree.tree_map(mode.tracer.unwrap_proxy, (op, *args))
        proxy_kwargs = pytree.tree_map(
            mode.tracer.unwrap_proxy, {"rng_state": rng_state, **kwargs}
        )
        out_proxy = mode.tracer.create_proxy(
            "call_function", graphsafe_run_with_rng_state, proxy_args, proxy_kwargs
        )
        return track_tensor_tree(out, out_proxy, constant=None, tracer=mode.tracer)

    @graphsafe_run_with_rng_state.py_functionalize_impl
    def impl_functional(ctx, op, *args, rng_state=None, **kwargs):
        unwrapped_rng_state = (
            ctx.unwrap_tensors(rng_state) if rng_state is not None else None
        )
        unwrapped_args = ctx.unwrap_tensors(args)
        unwrapped_kwargs = ctx.unwrap_tensors(kwargs)
````
- **EN**: This chunk defines `impl_functional`, which implements a focused helper used by the surrounding module. Decorators such as `graphsafe_run_with_rng_state.py_impl`, `graphsafe_run_with_rng_state.py_functionalize_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_functional`，其作用是实现周边模块使用的关键辅助逻辑。 像 `graphsafe_run_with_rng_state.py_impl`、`graphsafe_run_with_rng_state.py_functionalize_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 385-402 / 第 385-402 行
````python
        with ctx.redispatch_to_next():
            out = graphsafe_run_with_rng_state(
                op, *unwrapped_args, rng_state=unwrapped_rng_state, **unwrapped_kwargs
            )
            return ctx.wrap_tensors(out)

    return graphsafe_run_with_rng_state


graphsafe_run_with_rng_state = register_graphsafe_run_with_rng_state_op()


def register_run_dtensor_rng_op():
    """
    Register a higher-order operator for DTensor distributed random operations.
    Takes pre-computed integer offsets (start_offset_incr, end_offset_incr), an op,
    and args. Internally adjusts the RNG state using the offsets before/after
    running the op.
````
- **EN**: This chunk defines `register_run_dtensor_rng_op`, which registers a hook, schema, operator, or callback with surrounding infrastructure. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `register_run_dtensor_rng_op`，其作用是向周边基础设施注册钩子、schema、算子或回调。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 404-422 / 第 404-422 行
````python
    The offsets are computed at trace time from the DTensorSpec, so the compiled graph
    contains only plain integers — no DTensorSpec or DeviceMesh objects.
    """

    class RunDTensorRngOp(HigherOrderOperator):
        def __init__(self):
            super().__init__("run_dtensor_rng_op", cacheable=True)

        def __call__(self, start_offset_incr, end_offset_incr, op, *args, **kwargs):
            # pyrefly: ignore [missing-attribute]
            return super().__call__(
                start_offset_incr, end_offset_incr, op, *args, **kwargs
            )

    run_dtensor_rng_op = RunDTensorRngOp()

    run_dtensor_rng_op.py_impl(DispatchKey.Autograd)(
        autograd_not_implemented(run_dtensor_rng_op, deferred_error=True)
    )
````
- **EN**: It introduces or extends `RunDTensorRngOp`, which hold the main object-oriented state for this portion of the file. This chunk defines `__call__`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `RunDTensorRngOp`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__call__`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 424-442 / 第 424-442 行
````python
    @run_dtensor_rng_op.py_impl(DispatchKey.CUDA)
    def impl_cuda(start_offset_incr, end_offset_incr, op, *args, **kwargs):
        from torch.distributed.tensor._random import _PhiloxState

        state = _PhiloxState(torch.cuda.get_rng_state())
        old_offset = state.offset.clone()
        state.offset = old_offset + start_offset_incr

        device = (
            args[0].device
            if args and hasattr(args[0], "device")
            else torch.device(f"cuda:{torch.cuda.current_device()}")
        )
        with torch.random.fork_rng(devices=[device], device_type="cuda"):
            torch.cuda.set_rng_state(state.state)
            try:
                out = op(*args, **kwargs)
            finally:
                state.offset = old_offset + end_offset_incr
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.distributed.tensor._random. This chunk defines `impl_cuda`, which implements a focused helper used by the surrounding module. Decorators such as `run_dtensor_rng_op.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.distributed.tensor._random。 这一段定义了 `impl_cuda`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_dtensor_rng_op.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 444-462 / 第 444-462 行
````python
        torch.cuda.set_rng_state(state.state)
        return out

    @run_dtensor_rng_op.py_impl(DispatchKey.BackendSelect)
    def impl_backend_select(start_offset_incr, end_offset_incr, op, *args, **kwargs):
        device = get_device(args, kwargs)
        if device != "cuda":
            raise RuntimeError(
                f"run_dtensor_rng_op only supports CUDA device, got {device}. "
                f"This operator is designed for distributed random operations on CUDA."
            )
        return impl_cuda(start_offset_incr, end_offset_incr, op, *args, **kwargs)

    @run_dtensor_rng_op.py_impl(FakeTensorMode)
    def impl_fake_tensor_mode(
        mode, start_offset_incr, end_offset_incr, op, *args, **kwargs
    ):
        with mode:
            return op(*args, **kwargs)
````
- **EN**: This chunk defines `impl_fake_tensor_mode`, which implements a focused helper used by the surrounding module. Decorators such as `run_dtensor_rng_op.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_fake_tensor_mode`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_dtensor_rng_op.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 464-479 / 第 464-479 行
````python
    @run_dtensor_rng_op.py_impl(ProxyTorchDispatchMode)
    def impl_proxy_dispatch_mode(
        mode, start_offset_incr, end_offset_incr, op, *args, **kwargs
    ):
        with disable_proxy_modes_tracing():
            out = run_dtensor_rng_op(
                start_offset_incr, end_offset_incr, op, *args, **kwargs
            )
        proxy_args = pytree.tree_map(
            mode.tracer.unwrap_proxy, (start_offset_incr, end_offset_incr, op, *args)
        )
        proxy_kwargs = pytree.tree_map(mode.tracer.unwrap_proxy, kwargs)
        out_proxy = mode.tracer.create_proxy(
            "call_function", run_dtensor_rng_op, proxy_args, proxy_kwargs
        )
        return track_tensor_tree(out, out_proxy, constant=None, tracer=mode.tracer)
````
- **EN**: This chunk defines `impl_proxy_dispatch_mode`, which implements a focused helper used by the surrounding module. Decorators such as `run_dtensor_rng_op.py_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_proxy_dispatch_mode`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_dtensor_rng_op.py_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 481-499 / 第 481-499 行
````python
    @run_dtensor_rng_op.py_functionalize_impl
    def impl_functional(ctx, start_offset_incr, end_offset_incr, op, *args, **kwargs):
        unwrapped_args = ctx.unwrap_tensors(args)
        unwrapped_kwargs = ctx.unwrap_tensors(kwargs)

        with ctx.redispatch_to_next():
            out = run_dtensor_rng_op(
                start_offset_incr,
                end_offset_incr,
                op,
                *unwrapped_args,
                **unwrapped_kwargs,
            )
            return ctx.wrap_tensors(out)

    return run_dtensor_rng_op


run_dtensor_rng_op = register_run_dtensor_rng_op()
````
- **EN**: This chunk defines `impl_functional`, which implements a focused helper used by the surrounding module. Decorators such as `run_dtensor_rng_op.py_functionalize_impl` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl_functional`，其作用是实现周边模块使用的关键辅助逻辑。 像 `run_dtensor_rng_op.py_functionalize_impl` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 502-503 / 第 502-503 行
````python
def register_rng_prims():
    register_philox_rand()
````
- **EN**: This chunk defines `register_rng_prims`, which registers a hook, schema, operator, or callback with surrounding infrastructure.
- **CN**: 这一段定义了 `register_rng_prims`，其作用是向周边基础设施注册钩子、schema、算子或回调。

## Key Concepts / 关键概念

- **Primitive ops**
  - EN: Defines lower-level operators and contexts that higher layers can decompose into.
  - CN: 定义更底层的算子与上下文，供高层逻辑分解到这些 primitive。
- **throw_on_non_cuda**
  - EN: `throw_on_non_cuda` is one of the main symbols declared or implemented in this file.
  - CN: `throw_on_non_cuda` 是本文件声明或实现的主要符号之一。
- **register_rng_prim**
  - EN: `register_rng_prim` is one of the main symbols declared or implemented in this file.
  - CN: `register_rng_prim` 是本文件声明或实现的主要符号之一。
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

- **Internal torch modules / torch 内部模块**: `torch`, `torch.utils._pytree`, `torch._C`, `torch._higher_order_ops.utils`, `torch._ops`, `torch._prims_common`, `torch._subclasses.fake_tensor`, `torch.fx.experimental.proxy_tensor`, `torch.types`, `torch.distributed.tensor._random`
- **Standard library / 标准库**: `typing`
- **Primary symbols in this file / 本文件核心符号**: `throw_on_non_cuda`, `register_rng_prim`, `philox_rand_offset_meta`, `philox_rand_offset`, `register_philox_rand`, `get_device`, `register_run_and_save_rng_state_op`, `register_run_with_rng_state_op`, `register_graphsafe_run_with_rng_state_op`, `register_run_dtensor_rng_op`
