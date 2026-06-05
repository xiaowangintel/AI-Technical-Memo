# fake_impl.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_library/fake_impl.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python helpers for torch.library registration, library fragments, and operator definitions.
- **Purpose (CN)**: 实现 torch.library 注册、库片段以及算子定义相关的 Python 辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import contextlib
import functools
from collections.abc import Callable
from typing_extensions import deprecated

import torch
from torch._library.utils import Kernel, RegistrationHandle


class FakeImplHolder:
    """A holder where one can register an fake impl to."""

    def __init__(self, qualname: str):
        self.qualname: str = qualname
        # kernels stores all registered fake kernels, ordered by registration
        # time ascendingly (newer registration after older registration). If an
        # operator library gets loaded that overrides an existing fake kernel,
        # both kernels will be in the list, but the newest one will be the one
        # that is run. If the library is unloaded, we will remove the kernel
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._library.utils; standard-library helpers such as contextlib, functools, collections.abc; other helper packages such as typing_extensions. It introduces or extends `FakeImplHolder`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._library.utils；标准库辅助模块，如 contextlib、functools、collections.abc；其他辅助包，如 typing_extensions。 它引入或扩展了 `FakeImplHolder`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。

### Lines 21-37 / 第 21-37 行
````python
        # from this list.
        self.kernels: list[Kernel] = []

    @property
    def kernel(self):
        if len(self.kernels) == 0:
            return None
        return self.kernels[-1]

    @kernel.setter
    def kernel(self, value):
        raise RuntimeError("Unable to directly set kernel.")

    def register(
        self, func: Callable, source: str, lib, *, allow_override=False
    ) -> RegistrationHandle:
        """Register an fake impl.
````
- **EN**: This chunk defines `register`, which registers a hook, schema, operator, or callback with surrounding infrastructure. Decorators such as `property`, `kernel.setter` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `register`，其作用是向周边基础设施注册钩子、schema、算子或回调。 像 `property`、`kernel.setter` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 39-57 / 第 39-57 行
````python
        Returns a RegistrationHandle that one can use to de-register this
        fake impl.
        """

        if not allow_override:
            if self.kernel is not None:
                raise RuntimeError(
                    f"register_fake(...): the operator {self.qualname} "
                    f"already has an fake impl registered at "
                    f"{self.kernel.source}."
                )
            if torch._C._dispatch_has_kernel_for_dispatch_key(self.qualname, "Meta"):
                raise RuntimeError(
                    f"register_fake(...): the operator {self.qualname} "
                    f"already has an DispatchKey::Meta implementation via a "
                    f"pre-existing torch.library or TORCH_LIBRARY registration. "
                    f"Please either remove that registration or don't call "
                    f"register_fake."
                )
````
- **EN**: This chunk continues `register` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `register`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 59-76 / 第 59-76 行
````python
            if torch._C._dispatch_has_kernel_for_dispatch_key(
                self.qualname, "CompositeImplicitAutograd"
            ):
                raise RuntimeError(
                    f"register_fake(...): the operator {self.qualname} "
                    f"already has an implementation for this device type via a "
                    f"pre-existing registration to "
                    f"DispatchKey::CompositeImplicitAutograd."
                    f"CompositeImplicitAutograd operators do not need an fake "
                    f"impl; "
                    f"instead, the operator will decompose into its constituents "
                    f"and those "
                    f"can have fake impls defined on them."
                )

        # Store the kernel in this holder
        kernel = Kernel(func, source)
        self.kernels.append(kernel)
````
- **EN**: This chunk continues `register` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `register`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 78-96 / 第 78-96 行
````python
        def deregister_fake_kernel():
            self.kernels.remove(kernel)

        meta_kernel = construct_meta_kernel(self.qualname, self)
        lib.impl(self.qualname, meta_kernel, "Meta", allow_override=allow_override)

        handle = RegistrationHandle(deregister_fake_kernel)
        return handle


def construct_meta_kernel(qualname: str, fake_impl_holder: FakeImplHolder) -> Callable:
    if fake_impl_holder.kernel is None:
        raise AssertionError("fake_impl_holder.kernel must not be None")

    @functools.wraps(fake_impl_holder.kernel.func)
    def meta_kernel(*args, **kwargs):
        if fake_impl_holder.kernel is None:
            raise AssertionError("fake_impl_holder.kernel must not be None")
        source = fake_impl_holder.kernel.source
````
- **EN**: This chunk defines `meta_kernel`, which implements a focused helper used by the surrounding module. Decorators such as `functools.wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `meta_kernel`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 98-113 / 第 98-113 行
````python
        def error_on_ctx():
            raise RuntimeError(
                f"{qualname} ({source}): You're trying to run this operator "
                f"with meta Tensors (as opposed to FakeTensors), but this "
                f"operator may return an output Tensor with data-dependent shape. Meta "
                f"Tensors don't support operators with outputs that have data-dependent shapes "
                f"but FakeTensors do. "
                f"If your operator does not return an output with data-dependent shape, "
                f"make sure the FakeTensor and/or meta kernel does not call "
                f"torch.library.get_ctx(). Otherwise, please use FakeTensors."
            )

        with set_ctx_getter(error_on_ctx):
            return fake_impl_holder.kernel(*args, **kwargs)

    return meta_kernel
````
- **EN**: This chunk defines `error_on_ctx`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `error_on_ctx`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 116-131 / 第 116-131 行
````python
def get_none():
    return None


global_ctx_getter: Callable = get_none


@contextlib.contextmanager
def set_ctx_getter(ctx_getter):
    global global_ctx_getter
    prev = global_ctx_getter
    try:
        global_ctx_getter = ctx_getter
        yield
    finally:
        global_ctx_getter = prev
````
- **EN**: This chunk defines `set_ctx_getter`, which mutates configuration or backend state that affects later execution. Decorators such as `contextlib.contextmanager` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `set_ctx_getter`，其作用是修改会影响后续执行的配置或后端状态。 像 `contextlib.contextmanager` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 134-152 / 第 134-152 行
````python
class FakeImplCtx:
    """
    Context object for writing fake implementations for custom operators.
    """

    def __init__(self, _fake_mode, _op):
        self._fake_mode = _fake_mode
        self._shape_env = _fake_mode.shape_env
        self._op = _op

    @deprecated(
        "`create_unbacked_symint` is deprecated, please use `new_dynamic_size` instead",
        category=FutureWarning,
    )
    def create_unbacked_symint(self, *, min=2, max=None) -> torch.SymInt:
        return self.new_dynamic_size(min=min, max=max)

    def new_dynamic_size(self, *, min=0, max=None) -> torch.SymInt:
        """Constructs a new symint (symbolic int) representing a data-dependent value.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `FakeImplCtx`, which hold the main object-oriented state for this portion of the file. This chunk defines `new_dynamic_size`, which implements a focused helper used by the surrounding module. Decorators such as `deprecated` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `FakeImplCtx`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `new_dynamic_size`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deprecated` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 154-168 / 第 154-168 行
````python
        This is useful for writing the fake implementation (which is necessary
        for torch.compile) for a CustomOp where an output Tensor has a size
        that depends on the data of the input Tensors.

        Args:
            min (int): A statically known inclusive lower bound for this symint. Default: 0
            max (Optional[int]): A statically known inclusive upper bound for this
                symint. Default: None

        .. warning:

            It is important that the ``min`` and ``max`` (if not None) values are set
            correctly, otherwise, there will be undefined behavior under
            torch.compile. The default value of ``min`` is 2 due to torch.compile
            specializing on 0/1 sizes.
````
- **EN**: This chunk continues `FakeImplCtx` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `FakeImplCtx`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 170-189 / 第 170-189 行
````python
            You must also verify that your implementation on concrete Tensors
            (e.g. CPU/CUDA) only returns Tensors where the size that corresponds
            to the symint also has respects these constraint.
            The easiest way to do this is to add an assertion in the CPU/CUDA/etc
            implementation that the size follows these bounds.

        Example::

            >>> # An operator with data-dependent output shape
            >>> lib = torch.library.Library("mymodule", "FRAGMENT")
            >>> lib.define("mymodule::custom_nonzero(Tensor x) -> Tensor")
            >>>
            >>> @torch.library.register_fake("mymodule::custom_nonzero")
            >>> def _(x):
            >>>     # Number of nonzero-elements is data-dependent.
            >>>     # Since we cannot peek at the data in an fake impl,
            >>>     # we use the ctx object to construct a new symint that
            >>>     # represents the data-dependent size.
            >>>     ctx = torch.library.get_ctx()
            >>>     nnz = ctx.new_dynamic_size()
````
- **EN**: This chunk continues `FakeImplCtx` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段延续了 `FakeImplCtx`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 190-205 / 第 190-205 行
````python
            >>>     shape = [nnz, x.dim()]
            >>>     result = x.new_empty(shape, dtype=torch.int64)
            >>>     return result
            >>>
            >>> @torch.library.impl(lib, "custom_nonzero", "CPU")
            >>> def _(x):
            >>>     x_np = x.numpy()
            >>>     res = np.stack(np.nonzero(x_np), axis=1)
            >>>     return torch.tensor(res, device=x.device)

        """
        if (
            self._shape_env is None
            or not self._shape_env.allow_dynamic_output_shape_ops
        ):
            raise torch._subclasses.fake_tensor.DynamicOutputShapeException(self._op)
````
- **EN**: This chunk continues `FakeImplCtx` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `FakeImplCtx`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 207-221 / 第 207-221 行
````python
        if isinstance(min, torch.SymInt) or isinstance(max, torch.SymInt):
            raise ValueError(
                f"ctx.new_dynamic_size(min={min}, max={max}): expected "
                f"min and max to be statically known ints but got SymInt. "
                f"This is not supported."
            )

        if min < 0:
            raise ValueError(
                f"ctx.new_dynamic_size(min={min}, ...): expected min to be "
                f"greater than or equal to 0: this API can only create "
                f"non-negative sizes."
            )

        return allocate_size(self._shape_env, min, max)
````
- **EN**: This chunk continues `FakeImplCtx` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `FakeImplCtx`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 224-229 / 第 224-229 行
````python
def allocate_size(shape_env, min_val=0, max_val=None):
    result = shape_env.create_unbacked_symint()
    torch.fx.experimental.symbolic_shapes._constrain_range_for_size(
        result, min=min_val, max=max_val
    )
    return result
````
- **EN**: This chunk defines `allocate_size`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `allocate_size`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Library registration**
  - EN: Builds Python-side operator/library registrations that feed the dispatcher.
  - CN: 构建 Python 侧的算子/库注册，并将其接入 dispatcher。
- **FakeImplHolder**
  - EN: `FakeImplHolder` is one of the main symbols declared or implemented in this file.
  - CN: `FakeImplHolder` 是本文件声明或实现的主要符号之一。
- **construct_meta_kernel**
  - EN: `construct_meta_kernel` is one of the main symbols declared or implemented in this file.
  - CN: `construct_meta_kernel` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **NumPy compatibility**
  - EN: The implementation mirrors NumPy-style semantics to keep array/tensor behavior familiar.
  - CN: 实现对齐 NumPy 风格语义，使数组/张量行为更易理解。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._library.utils`
- **Standard library / 标准库**: `contextlib`, `functools`, `collections.abc`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `FakeImplHolder`, `construct_meta_kernel`, `get_none`, `set_ctx_getter`, `FakeImplCtx`, `allocate_size`
