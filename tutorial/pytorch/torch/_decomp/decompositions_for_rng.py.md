# decompositions_for_rng.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_decomp/decompositions_for_rng.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines decomposition tables and helpers that rewrite higher-level operators into simpler building blocks.
- **Purpose (CN)**: 定义 decomposition 表和辅助逻辑，把高层算子重写为更基础的构件。
## Line-by-Line Analysis / 逐行分析

### Lines 1-19 / 第 1-19 行
````python
# mypy: allow-untyped-decorators
# mypy: allow-untyped-defs
import functools
from collections import defaultdict
from collections.abc import Callable

import torch
import torch._decomp as decomp
from torch._decomp import get_decompositions
from torch._ops import OpOverload


aten = torch.ops.aten

rng_decompositions: dict[str, dict[OpOverload, Callable]] = defaultdict(dict)


def register_rng_decomposition(aten_op):
    return decomp.register_decomposition(aten_op, rng_decompositions)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._decomp, torch._ops; standard-library helpers such as functools, collections, collections.abc. This chunk defines `register_rng_decomposition`, which registers a hook, schema, operator, or callback with surrounding infrastructure. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._decomp、torch._ops；标准库辅助模块，如 functools、collections、collections.abc。 这一段定义了 `register_rng_decomposition`，其作用是向周边基础设施注册钩子、schema、算子或回调。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 22-41 / 第 22-41 行
````python
def throw_on_non_cuda(device):
    raise RuntimeError(
        f"You are trying to functionalize a {device.type} RNG operator but {device.type} does not "
        f"use Philox/counter-based RNG. Therefore, functionalizing a {device.type} RNG operator is "
        "not supported. We are discussing the possibility of a Philox-based RNG implementation for CPU."
    )


# TODO - We have to register many more distributions here, and also higher level
# ops like dropout which have fused implementation and can hide the rand inside.
@register_rng_decomposition(aten.rand)
def rand(shape, dtype=None, layout=torch.strided, device=None, pin_memory=False):
    if device and device.type != "cuda":
        throw_on_non_cuda(device)
    seed, offset = PhiloxStateTracker.get_state_as_tuple()
    dtype = dtype or torch.float32
    out, offset_jump = torch.ops.rngprims.philox_rand(
        shape, seed, offset, None, device, dtype
    )
    PhiloxStateTracker.advance_offset(offset_jump)
````
- **EN**: This chunk defines `rand`, which implements a focused helper used by the surrounding module. Decorators such as `register_rng_decomposition` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `rand`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_rng_decomposition` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 42-61 / 第 42-61 行
````python
    return out


@register_rng_decomposition(aten.rand_like)
def rand_like(
    x: torch.Tensor,
    dtype=None,
    layout=None,
    device=None,
    pin_memory=False,
    memory_format=torch.preserve_format,
):
    device = device or x.device
    if device.type != "cuda":
        throw_on_non_cuda(device)
    dtype = dtype or x.dtype
    seed, offset = PhiloxStateTracker.get_state_as_tuple()
    out, offset_jump = torch.ops.rngprims.philox_rand(
        x.shape, seed, offset, None, device, dtype
    )
````
- **EN**: This chunk defines `rand_like`, which implements a focused helper used by the surrounding module. Decorators such as `register_rng_decomposition` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `rand_like`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_rng_decomposition` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 62-75 / 第 62-75 行
````python
    PhiloxStateTracker.advance_offset(offset_jump)
    return out


class PhiloxState:
    """
    Represents a PhiloxRngState - (seed, offset) where offset = base_offset +
    relative_offset. seed and base_offset basically point to the rng state just
    before tracing starts. relative offset tracks the totally consumed offset at
    trace time.
    """

    def __init__(self) -> None:
        self.reset()
````
- **EN**: It introduces or extends `PhiloxState`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `PhiloxState`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 77-92 / 第 77-92 行
````python
    def reset(self):
        self.seed = torch.tensor(())
        self.base_offset = torch.tensor(())
        self.relative_offset = 0
        self.offset_advanced_at_least_once = False

    def validate_state(self):
        if self.seed.numel() == 0 or self.base_offset.numel() == 0:
            raise AssertionError(
                f"seed and base_offset must not be empty, got "
                f"seed.numel()={self.seed.numel()}, base_offset.numel()={self.base_offset.numel()}"
            )

    def advance_offset(self, consumed_offset):
        self.offset_advanced_at_least_once = True
        self.relative_offset = self.relative_offset + consumed_offset
````
- **EN**: This chunk defines `advance_offset`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `advance_offset`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 94-111 / 第 94-111 行
````python
    def set_state(self, seed, base_offset, relative_offset=0):
        self.seed = seed
        self.base_offset = base_offset
        self.relative_offset = relative_offset

    def get_state_as_tuple(self):
        self.validate_state()
        return (self.seed, self.base_offset + self.relative_offset)

    def get_state_as_tensor(self):
        # Only needed because we override get_rng_state.
        self.validate_state()
        return torch.stack([self.seed, self.base_offset + self.relative_offset])

    def set_state_from_tensor(self, state):
        # Only needed because we override set_rng_state.
        self.seed, self.base_offset = torch.unbind(state)
        self.relative_offset = 0
````
- **EN**: This chunk defines `set_state_from_tensor`, which mutates configuration or backend state that affects later execution. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `set_state_from_tensor`，其作用是修改会影响后续执行的配置或后端状态。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 114-131 / 第 114-131 行
````python
class PhiloxStateTracker:
    """
    Singleton class to track the philox rng state during AOT Autograd tracing.
    For each aot tracing instance, AOT Autograd resets this tracker and keeps
    track of both forward and backward offsets. At runtime, we only care about
    the total consumed forward and backward offsets. For dynamic shapes, these
    offsets are a function of input shapes. Therefore, the AOT generated graphs
    have additional outputs that compute total consumed forward and backward
    offsets.
    """

    running_state: PhiloxState
    fwd_state: PhiloxState
    bwd_state: PhiloxState

    def __enter__(self):
        PhiloxStateTracker.reset()
        return self
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `PhiloxStateTracker`, which hold the main object-oriented state for this portion of the file. This chunk defines `__enter__`, which implements context-manager boundaries around temporary runtime state. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `PhiloxStateTracker`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__enter__`，其作用是实现围绕临时运行时状态的上下文管理器边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 133-150 / 第 133-150 行
````python
    def __exit__(self, exc_type, exc_cal, exc_tb):
        PhiloxStateTracker.reset()

    @classmethod
    def reset(cls):
        cls.running_state = PhiloxState()
        cls.fwd_state = PhiloxState()
        cls.bwd_state = PhiloxState()

    @classmethod
    def mark_beginning_of_forward(cls):
        # Tells the tracker to use fwd_state as the running state
        cls.running_state = cls.fwd_state

    @classmethod
    def mark_beginning_of_backward(cls):
        # Tells the tracker to use bwd_state as the running state
        cls.running_state = cls.bwd_state
````
- **EN**: This chunk defines `mark_beginning_of_backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 这一段定义了 `mark_beginning_of_backward`，其作用是实现与前向路径配套的求导期行为。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 152-169 / 第 152-169 行
````python
    @classmethod
    def record_state(cls, seed, offset, mode):
        # Records the seed and offset tensors. These tensors are used to invoke
        # the philox_rand functional primitives.
        if mode == "forward":
            cls.fwd_state.set_state(seed, offset)
            cls.mark_beginning_of_forward()
        else:
            if mode != "backward":
                raise AssertionError(f"mode must be 'backward', got {mode}")
            cls.bwd_state.set_state(seed, offset)

    @classmethod
    def get_state_as_tensor(cls):
        # The only reason this exists is because we override get_rng_state and
        # set_rng_state during tracing. get_rng_state expects a tensor output,
        # so return (seed, offset) tuple upset other parts of the program like
        # ctx.saved_tensors.
````
- **EN**: This chunk defines `get_state_as_tensor`, which retrieves runtime state and exposes it through a Python-friendly accessor. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_state_as_tensor`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 171-188 / 第 171-188 行
````python
        # A bad consequence is that if user saves and restores rng state, we
        # have little bit of ugliness in the generated code, where we first
        # concat the (seed, offset) to create a tensor for get_rng_state, and
        # then split it back to get (seed, offset) tuple in set_rng_state.

        # TODO: Investigate if there is be a better way to wrap the tuple in a
        # false Tensor object, and then desugar it later on.
        return cls.running_state.get_state_as_tensor()

    @classmethod
    def get_state_as_tuple(cls):
        return cls.running_state.get_state_as_tuple()

    @classmethod
    def set_state_from_tensor(cls, x):
        # This is only needed because we override set_rng_state. Look at the
        # comment in get_state_from_tensor method.
        cls.running_state.set_state_from_tensor(x)
````
- **EN**: This chunk defines `set_state_from_tensor`, which mutates configuration or backend state that affects later execution. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `set_state_from_tensor`，其作用是修改会影响后续执行的配置或后端状态。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 190-203 / 第 190-203 行
````python
    @classmethod
    def advance_offset(cls, consumed_offset):
        cls.running_state.advance_offset(consumed_offset)

    @classmethod
    def get_current_relative_offset(cls):
        return cls.running_state.relative_offset

    @staticmethod
    def multiple_of_4(offset):
        # torch cuda rng state offset must be a multiple of 4. For inductor, as
        # we sum up all the numel, the result might not be a multiple of 4. This
        # method achieves that.
        return (offset + 3) // 4 * 4
````
- **EN**: This chunk defines `multiple_of_4`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod`, `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `multiple_of_4`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod`、`staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 205-221 / 第 205-221 行
````python
    @classmethod
    def get_updated_fwd_offset(cls):
        # Short circuit if no rand ops were observed
        if not cls.fwd_state.offset_advanced_at_least_once:
            return cls.fwd_state.base_offset
        return cls.multiple_of_4(
            cls.fwd_state.base_offset + cls.fwd_state.relative_offset
        )

    @classmethod
    def get_updated_bwd_offset(cls):
        # Short circuit if no rand ops were observed
        if not cls.bwd_state.offset_advanced_at_least_once:
            return cls.bwd_state.base_offset
        return cls.multiple_of_4(
            cls.bwd_state.base_offset + cls.bwd_state.relative_offset
        )
````
- **EN**: This chunk defines `get_updated_bwd_offset`, which retrieves runtime state and exposes it through a Python-friendly accessor. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_updated_bwd_offset`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 224-243 / 第 224-243 行
````python
# Adding more decompositions which eventually use rand_like inside decomps.
# Adding these in rng_decompositions ensures the functionalization of rand_like
# ops used in these decomps. The list is copied from inductor codebase, which
# uses it for similar purpose.
#
# Caution - These decomps do not have same accuracy as that of eager. However,
# we can't just disable them with a config flag like fallback_random, because
# for functionalization of rng ops, we have to decompose these ops.
extra_random_decomps = get_decompositions(
    [
        aten.cauchy,
        aten.cauchy_,
        aten.exponential,
        aten.exponential_,
        aten.geometric,
        aten.geometric_,
        aten.native_dropout,
        aten.normal,
        aten.normal_,
        aten.normal_functional,
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 244-260 / 第 244-260 行
````python
        aten.log_normal,
        aten.log_normal_,
        aten.rrelu_with_noise,
        aten.rrelu_with_noise_,
        aten.uniform_,
    ]
)
register_extra_random_decomp = functools.partial(
    decomp.register_decomposition, registry=extra_random_decomps
)


@register_extra_random_decomp([aten.bernoulli_])
def bernoulli_(self, p=0.5):
    if self.device == torch.device("cpu"):
        return NotImplemented
    return self.copy_(torch.rand_like(self, dtype=torch.float32) < p)
````
- **EN**: This chunk defines `bernoulli_`, which implements a focused helper used by the surrounding module. Decorators such as `register_extra_random_decomp` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `bernoulli_`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_extra_random_decomp` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 263-272 / 第 263-272 行
````python
@register_extra_random_decomp([aten.bernoulli.p])
def bernoulli_p(self, p=0.5, *, generator=None):
    if self.device == torch.device("cpu"):
        return NotImplemented
    if generator is not None:
        raise AssertionError(f"generator must be None, got {generator}")
    return torch.rand_like(self, dtype=torch.float32) < p


rng_decompositions.update(extra_random_decomps)  # type: ignore[arg-type]
````
- **EN**: This chunk defines `bernoulli_p`, which implements a focused helper used by the surrounding module. Decorators such as `register_extra_random_decomp` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `bernoulli_p`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_extra_random_decomp` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Operator decomposition**
  - EN: Rewrites composite operators into simpler primitives used by tracing and compilation flows.
  - CN: 把复合算子重写为 tracing 与编译流程可用的更基础 primitive。
- **register_rng_decomposition**
  - EN: `register_rng_decomposition` is one of the main symbols declared or implemented in this file.
  - CN: `register_rng_decomposition` 是本文件声明或实现的主要符号之一。
- **throw_on_non_cuda**
  - EN: `throw_on_non_cuda` is one of the main symbols declared or implemented in this file.
  - CN: `throw_on_non_cuda` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._decomp`, `torch._ops`
- **Standard library / 标准库**: `functools`, `collections`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `register_rng_decomposition`, `throw_on_non_cuda`, `rand`, `rand_like`, `PhiloxState`, `PhiloxStateTracker`, `bernoulli_`, `bernoulli_p`
