# autocast_mode.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/amp/autocast_mode.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements automatic mixed-precision contexts, scaling logic, and precision-management helpers.
- **Purpose (CN)**: 实现自动混合精度上下文、梯度缩放逻辑以及精度管理辅助功能。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
````python
# mypy: allow-untyped-defs
import collections
import functools
import warnings
from typing import Any

import torch
from torch.types import _dtype


try:
    import numpy as np

    HAS_NUMPY = True
except ModuleNotFoundError:
    HAS_NUMPY = False
    np = None  # type: ignore[assignment]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.types; standard-library helpers such as collections, functools, warnings, ...; other helper packages such as numpy. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.types；标准库辅助模块，如 collections、functools、warnings、...；其他辅助包，如 numpy。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。

### Lines 19-37 / 第 19-37 行
````python
__all__ = [
    "autocast_decorator",
    "autocast",
    "is_autocast_available",
    "custom_fwd",
    "custom_bwd",
]


def is_autocast_available(device_type: str) -> bool:
    r"""
    Return a bool indicating if autocast is available on :attr:`device_type`.

    Args:
        device_type(str):  Device type to use. Possible values are: 'cuda', 'cpu', 'mtia', 'maia', 'xpu', and so on.
            The type is the same as the `type` attribute of a :class:`torch.device`.
            Thus, you may obtain the device type of a tensor using `Tensor.device.type`.
    """
    return torch._C._is_autocast_available(device_type)
````
- **EN**: `__all__` defines the public symbols that this module chooses to export. This chunk defines `is_autocast_available`, which checks a capability or invariant before later code relies on it. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: `__all__` 定义了本模块选择导出的公共符号。 这一段定义了 `is_autocast_available`，其作用是检查某项能力或不变量，供后续逻辑依赖。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 40-55 / 第 40-55 行
````python
def autocast_decorator(autocast_instance, func):
    @functools.wraps(func)
    def decorate_autocast(*args, **kwargs):
        with autocast_instance:
            return func(*args, **kwargs)

    decorate_autocast.__script_unsupported = (  # type: ignore[attr-defined]
        "@autocast() decorator is not supported in script mode"
    )
    return decorate_autocast


class autocast:
    r"""
    Instances of :class:`autocast` serve as context managers or decorators that
    allow regions of your script to run in mixed precision.
````
- **EN**: It introduces or extends `autocast`, which hold the main object-oriented state for this portion of the file. This chunk defines `decorate_autocast`, which implements a focused helper used by the surrounding module. Decorators such as `functools.wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `autocast`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `decorate_autocast`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 57-75 / 第 57-75 行
````python
    In these regions, ops run in an op-specific dtype chosen by autocast
    to improve performance while maintaining accuracy.
    See the :ref:`Autocast Op Reference<autocast-op-reference>` for details.

    When entering an autocast-enabled region, Tensors may be any type.
    You should not call ``half()`` or ``bfloat16()`` on your model(s) or inputs when using autocasting.

    :class:`autocast` should wrap only the forward pass(es) of your network, including the loss
    computation(s).  Backward passes under autocast are not recommended.
    Backward ops run in the same type that autocast used for corresponding forward ops.

    Example for CUDA Devices::

        # Creates model and optimizer in default precision
        model = Net().cuda()
        optimizer = optim.SGD(model.parameters(), ...)

        for input, target in data:
            optimizer.zero_grad()
````
- **EN**: This chunk continues `autocast` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `autocast`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 77-95 / 第 77-95 行
````python
            # Enables autocasting for the forward pass (model + loss)
            with torch.autocast(device_type="cuda"):
                output = model(input)
                loss = loss_fn(output, target)

            # Exits the context manager before backward()
            loss.backward()
            optimizer.step()

    See the :ref:`Automatic Mixed Precision examples<amp-examples>` for usage (along with gradient scaling)
    in more complex scenarios (e.g., gradient penalty, multiple models/losses, custom autograd functions).

    :class:`autocast` can also be used as a decorator, e.g., on the ``forward`` method of your model::

        class AutocastModel(nn.Module):
            ...

            @torch.autocast(device_type="cuda")
            def forward(self, input): ...
````
- **EN**: It introduces or extends `AutocastModel`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Decorators such as `torch.autocast` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `AutocastModel`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `torch.autocast` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 97-109 / 第 97-109 行
````python
    Floating-point Tensors produced in an autocast-enabled region may be ``float16``.
    After returning to an autocast-disabled region, using them with floating-point
    Tensors of different dtypes may cause type mismatch errors.  If so, cast the Tensor(s)
    produced in the autocast region back to ``float32`` (or other dtype if desired).
    If a Tensor from the autocast region is already ``float32``, the cast is a no-op,
    and incurs no additional overhead.
    CUDA Example::

        # Creates some tensors in default dtype (here assumed to be float32)
        a_float32 = torch.rand((8, 8), device="cuda")
        b_float32 = torch.rand((8, 8), device="cuda")
        c_float32 = torch.rand((8, 8), device="cuda")
        d_float32 = torch.rand((8, 8), device="cuda")
````
- **EN**: This chunk continues `AutocastModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `AutocastModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 111-126 / 第 111-126 行
````python
        with torch.autocast(device_type="cuda"):
            # torch.mm is on autocast's list of ops that should run in float16.
            # Inputs are float32, but the op runs in float16 and produces float16 output.
            # No manual casts are required.
            e_float16 = torch.mm(a_float32, b_float32)
            # Also handles mixed input types
            f_float16 = torch.mm(d_float32, e_float16)

        # After exiting autocast, calls f_float16.float() to use with d_float32
        g_float32 = torch.mm(d_float32, f_float16.float())

    CPU Training Example::

        # Creates model and optimizer in default precision
        model = Net()
        optimizer = optim.SGD(model.parameters(), ...)
````
- **EN**: This chunk continues `AutocastModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段延续了 `AutocastModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 128-144 / 第 128-144 行
````python
        for epoch in epochs:
            for input, target in data:
                optimizer.zero_grad()

                # Runs the forward pass with autocasting.
                with torch.autocast(device_type="cpu", dtype=torch.bfloat16):
                    output = model(input)
                    loss = loss_fn(output, target)

                loss.backward()
                optimizer.step()


    CPU Inference Example::

        # Creates model in default precision
        model = Net().eval()
````
- **EN**: This chunk continues `AutocastModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `AutocastModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 146-164 / 第 146-164 行
````python
        with torch.autocast(device_type="cpu", dtype=torch.bfloat16):
            for input in data:
                # Runs the forward pass with autocasting.
                output = model(input)

    CPU Inference Example with Jit Trace::

        class TestModel(nn.Module):
            def __init__(self, input_size, num_classes):
                super().__init__()
                self.fc1 = nn.Linear(input_size, num_classes)

            def forward(self, x):
                return self.fc1(x)


        input_size = 2
        num_classes = 2
        model = TestModel(input_size, num_classes).eval()
````
- **EN**: It introduces or extends `TestModel`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `TestModel`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 166-184 / 第 166-184 行
````python
        # For now, we suggest to disable the Jit Autocast Pass,
        # As the issue: https://github.com/pytorch/pytorch/issues/75956
        torch._C._jit_set_autocast_mode(False)

        with torch.cpu.amp.autocast(cache_enabled=False):
            model = torch.jit.trace(model, torch.randn(1, input_size))
        model = torch.jit.freeze(model)
        # Models Run
        for _ in range(3):
            model(torch.randn(1, input_size))

    Type mismatch errors *in* an autocast-enabled region are a bug; if this is what you observe,
    please file an issue.

    ``autocast(enabled=False)`` subregions can be nested in autocast-enabled regions.
    Locally disabling autocast can be useful, for example, if you want to force a subregion
    to run in a particular ``dtype``.  Disabling autocast gives you explicit control over
    the execution type.  In the subregion, inputs from the surrounding region
    should be cast to ``dtype`` before use::
````
- **EN**: This chunk continues `TestModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `TestModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 186-201 / 第 186-201 行
````python
        # Creates some tensors in default dtype (here assumed to be float32)
        a_float32 = torch.rand((8, 8), device="cuda")
        b_float32 = torch.rand((8, 8), device="cuda")
        c_float32 = torch.rand((8, 8), device="cuda")
        d_float32 = torch.rand((8, 8), device="cuda")

        with torch.autocast(device_type="cuda"):
            e_float16 = torch.mm(a_float32, b_float32)
            with torch.autocast(device_type="cuda", enabled=False):
                # Calls e_float16.float() to ensure float32 execution
                # (necessary because e_float16 was created in an autocasted region)
                f_float32 = torch.mm(c_float32, e_float16.float())

            # No manual casts are required when re-entering the autocast-enabled region.
            # torch.mm again runs in float16 and produces float16 output, regardless of input types.
            g_float16 = torch.mm(d_float32, f_float32)
````
- **EN**: This chunk continues `TestModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段延续了 `TestModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 203-220 / 第 203-220 行
````python
    The autocast state is thread-local.  If you want it enabled in a new thread, the context manager or decorator
    must be invoked in that thread.  This affects :class:`torch.nn.DataParallel` and
    :class:`torch.nn.parallel.DistributedDataParallel` when used with more than one GPU per process
    (see :ref:`Working with Multiple GPUs<amp-multigpu>`).

    Args:
        device_type(str, required):  Device type to use. Possible values are: 'cuda', 'cpu', 'mtia', 'maia', 'xpu', and 'hpu'.
                                     The type is the same as the `type` attribute of a :class:`torch.device`.
                                     Thus, you may obtain the device type of a tensor using `Tensor.device.type`.
        enabled(bool, optional):  Whether autocasting should be enabled in the region.
            Default: ``True``
        dtype(torch_dtype, optional):  Data type for ops run in autocast. It uses the default value
            (``torch.float16`` for CUDA and ``torch.bfloat16`` for CPU), given by
            :func:`~torch.get_autocast_dtype`, if :attr:`dtype` is ``None``.
            Default: ``None``
        cache_enabled(bool, optional):  Whether the weight cache inside autocast should be enabled.
            Default: ``True``
    """
````
- **EN**: This chunk continues `TestModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `TestModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 222-241 / 第 222-241 行
````python
    def __init__(
        self,
        device_type: str,
        dtype: _dtype | None = None,
        enabled: bool = True,
        cache_enabled: bool | None = None,
    ):
        if not isinstance(device_type, str):
            raise ValueError(
                f"Expected `device_type` of type `str`, got: `{type(device_type)}`"
            )
        self.fast_dtype = (
            torch.get_autocast_dtype(device_type) if dtype is None else dtype
        )
        if torch._jit_internal.is_scripting():
            self._enabled = enabled
            self.device = device_type
            if self.fast_dtype is None:
                raise AssertionError("fast_dtype must not be None in scripting mode")
            return
````
- **EN**: This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 242-258 / 第 242-258 行
````python
        self.device = device_type
        if not is_autocast_available(self.device):
            raise RuntimeError(
                f"User specified an unsupported autocast device_type '{self.device}'"
            )

        device_supported_dtypes = [torch.bfloat16, torch.float16]

        self.custom_backend_name = torch._C._get_privateuse1_backend_name()
        if self.device == self.custom_backend_name:
            necessary_funcs = [
                "get_amp_supported_dtype",
            ]
            message = f"Tried to use AMP with the `{self.custom_backend_name}` backend, but the backend has not "
            message += "registered a module or  the module miss some necessary funcs. The backend should register "
            message += "a module by `torch._register_device_module`, and the module must have these funcs: \n"
            message += "`get_amp_supported_dtype() -> List[torch.dtype]`. \n"
````
- **EN**: This chunk continues `__init__` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `__init__`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 260-274 / 第 260-274 行
````python
            if not hasattr(torch, self.custom_backend_name):
                raise AssertionError(message)
            self.custom_device_mod = getattr(torch, self.custom_backend_name)
            for func in necessary_funcs:
                if not hasattr(self.custom_device_mod, func):
                    raise AssertionError(
                        message + f"But the func `{func}` is missing. \n"
                    )
            device_supported_dtypes = self.custom_device_mod.get_amp_supported_dtype()

        self._cache_enabled = (
            torch.is_autocast_cache_enabled()
            if cache_enabled is None
            else cache_enabled
        )
````
- **EN**: This chunk continues `__init__` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `__init__`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 276-295 / 第 276-295 行
````python
        device_name = (
            self.device
            if self.device == self.custom_backend_name
            else self.device.upper()
        )
        if enabled:
            # Special case for CUDA AMP and bfloat16 support
            if self.device == "cuda":
                if torch.cuda.amp.common.amp_definitely_not_available():
                    warnings.warn(
                        "CUDA is not available or torch_xla is imported. Disabling autocast.",
                        stacklevel=2,
                    )
                    enabled = False
                elif (
                    self.fast_dtype == torch.bfloat16
                    and not torch.cuda.is_bf16_supported()
                ):
                    raise RuntimeError(
                        "Current CUDA Device does not support bfloat16. Please switch dtype to float16."
````
- **EN**: This chunk continues `__init__` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `__init__`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 296-315 / 第 296-315 行
````python
                    )
            elif self.fast_dtype not in device_supported_dtypes:
                error_message = (
                    f"In {device_name} autocast, but the target dtype is not supported. Disabling autocast.\n"
                    f"{device_name} Autocast only supports dtypes of "
                    + ", ".join(map(str, device_supported_dtypes))
                    + " currently."
                )
                warnings.warn(error_message, stacklevel=2)
                enabled = False
                # Special case for MPS bfloat16 support on macOS < 14
                if (
                    self.device == "mps"
                    and self.fast_dtype == torch.bfloat16
                    and not torch.backends.mps.is_macos_or_newer(14, 0)
                ):
                    error_message = (
                        "In MPS autocast, but the target dtype torch.bfloat16 is not supported "
                        "on macOS versions below 14. Disabling autocast."
                    )
````
- **EN**: This chunk continues `__init__` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `__init__`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 316-332 / 第 316-332 行
````python
                    warnings.warn(error_message, stacklevel=2)
                    enabled = False
        self._enabled = enabled

    def __enter__(self):
        if torch._jit_internal.is_scripting():
            if self.fast_dtype is None:
                raise AssertionError("fast_dtype must not be None in scripting mode")
            return self

        self.prev_cache_enabled = torch.is_autocast_cache_enabled()
        self.prev = torch.is_autocast_enabled(self.device)
        self.prev_fastdtype = torch.get_autocast_dtype(self.device)
        torch.set_autocast_enabled(self.device, self._enabled)
        torch.set_autocast_dtype(self.device, self.fast_dtype)  # type: ignore[arg-type]
        torch.autocast_increment_nesting()
        torch.set_autocast_cache_enabled(self._cache_enabled)
````
- **EN**: This chunk defines `__enter__`, which implements context-manager boundaries around temporary runtime state. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__enter__`，其作用是实现围绕临时运行时状态的上下文管理器边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 334-351 / 第 334-351 行
````python
        # only dispatch to PreDispatchTorchFunctionMode to avoid exposing this
        # API to other functional modes. We only expose to PreDispatchTorchFunctionMode
        # for preserving autocast in torch.export.export.
        if torch._C._is_torch_function_mode_enabled():
            stacks = torch.overrides._get_current_function_mode_stack()
            for mode in stacks:
                if isinstance(
                    mode,
                    torch.fx.experimental.proxy_tensor.PreDispatchTorchFunctionMode,
                ):
                    args = (
                        self.device,
                        self.fast_dtype,
                        self._enabled,
                        self._cache_enabled,
                    )
                    mode.__torch_function__(torch.amp._enter_autocast, (), args)
                    return self
````
- **EN**: This chunk continues `__enter__` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `__enter__`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 353-372 / 第 353-372 行
````python
        return self

    def __exit__(self, exc_type: Any, exc_val: Any, exc_tb: Any):  # type: ignore[override]
        if torch._jit_internal.is_scripting():
            return

        # Drop the cache when we exit to a nesting level that's outside any instance of autocast.
        if torch.autocast_decrement_nesting() == 0:
            torch.clear_autocast_cache()
        torch.set_autocast_enabled(self.device, self.prev)
        torch.set_autocast_dtype(self.device, self.prev_fastdtype)
        torch.set_autocast_cache_enabled(self.prev_cache_enabled)

        # only dispatch to PreDispatchTorchFunctionMode to avoid exposing this
        # API to other functional modes. We only expose to PreDispatchTorchFunctionMode
        # for preserving autocast in torch.export.export.
        if torch._C._is_torch_function_mode_enabled():
            stacks = torch.overrides._get_current_function_mode_stack()
            for mode in stacks:
                if isinstance(
````
- **EN**: This chunk defines `__exit__`, which implements context-manager boundaries around temporary runtime state. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__exit__`，其作用是实现围绕临时运行时状态的上下文管理器边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 373-392 / 第 373-392 行
````python
                    mode,
                    torch.fx.experimental.proxy_tensor.PreDispatchTorchFunctionMode,
                ):
                    mode.__torch_function__(torch.amp._exit_autocast, (), ())
                    # This is very important because the above line actually doesn't
                    # run exit code so it end up swallowing exceptions.
                    return False
        return False

    def __call__(self, func):
        if torch._jit_internal.is_scripting():
            return func
        if not callable(func):
            raise TypeError(
                f"autocast()(func) requires a callable, but got {type(func).__name__}. "
                f"Did you mean to use autocast as a context manager? For example:\n"
                f"    with torch.autocast(device_type=...):\n"
                f"        output = model(input)"
            )
        return autocast_decorator(self, func)
````
- **EN**: This chunk defines `__call__`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__call__`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 395-411 / 第 395-411 行
````python
# Subclass to distinguish autocast variables created by _enter_autocast (and not managed by a with statement)
class _UnmanagedAutocast(autocast):
    pass


# These functions aren't meant for public usage.
# They are what we trace into a graph during pre_dispatch tracing
# when we encounter an autocast context manager.
def _enter_autocast(*vals):
    # For pre-dispatch tracing, if a TorchFunction mode is active, we'll want to trace this into a graph.
    if torch._C._is_torch_function_mode_enabled():
        return torch.overrides.handle_torch_function(
            torch.amp._enter_autocast, [], *vals
        )
    mode = _UnmanagedAutocast(*vals)
    mode.__enter__()
    return mode
````
- **EN**: It introduces or extends `_UnmanagedAutocast`, which hold the main object-oriented state for this portion of the file. This chunk defines `_enter_autocast`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_UnmanagedAutocast`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_enter_autocast`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 414-433 / 第 414-433 行
````python
def _exit_autocast(mode):
    if torch._C._is_torch_function_mode_enabled():
        return torch.overrides.handle_torch_function(torch.amp._exit_autocast, [], mode)
    mode.__exit__(None, None, None)


# Casts Tensors and containers of Tensors.  Special-cases passthroughs for strings and np.ndarrays, which
# may be falsely detected as "Iterables."
def _cast(value, device_type: str, dtype: _dtype):
    if isinstance(value, torch.Tensor):
        is_eligible = (
            value.is_floating_point()
            and value.device.type == device_type
            and (value.dtype is not torch.float64)
        )
        return value.to(dtype) if is_eligible else value
    elif isinstance(value, (str, bytes)):
        return value
    elif HAS_NUMPY and isinstance(
        value,
````
- **EN**: This chunk defines `_cast`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_cast`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 434-450 / 第 434-450 行
````python
        # pyrefly: ignore [missing-attribute]
        np.ndarray,
    ):
        return value
    elif isinstance(value, collections.abc.Mapping):
        return {
            _cast(k, device_type, dtype): _cast(v, device_type, dtype)
            for k, v in value.items()
        }
    elif isinstance(value, collections.abc.Iterable):
        iterable = (_cast(v, device_type, dtype) for v in value)
        if isinstance(value, (list, tuple)):
            return type(value)(iterable)
        else:
            return iterable
    else:
        return value
````
- **EN**: This chunk continues `_cast` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_cast`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 453-472 / 第 453-472 行
````python
def custom_fwd(
    fwd=None,
    *,
    device_type: str,
    cast_inputs: _dtype | None = None,
):
    """
    Create a helper decorator for ``forward`` methods of custom autograd functions.

    Autograd functions are subclasses of :class:`torch.autograd.Function`.
    See the :ref:`example page<amp-custom-examples>` for more detail.

    Args:
        device_type(str):  Device type to use. 'cuda', 'cpu', 'mtia', 'maia', 'xpu' and so on.
            The type is the same as the `type` attribute of a :class:`torch.device`.
            Thus, you may obtain the device type of a tensor using `Tensor.device.type`.
        cast_inputs (:class:`torch.dtype` or None, optional, default=None):  If not ``None``,
            when ``forward`` runs in an autocast-enabled region, casts incoming
            floating-point Tensors to the target dtype (non-floating-point Tensors are not affected),
            then executes ``forward`` with autocast disabled.
````
- **EN**: This chunk defines `custom_fwd`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `custom_fwd`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 473-486 / 第 473-486 行
````python
            If ``None``, ``forward``'s internal ops execute with the current autocast state.

    .. note::
        If the decorated ``forward`` is called outside an autocast-enabled region,
        :func:`custom_fwd<custom_fwd>` is a no-op and ``cast_inputs`` has no effect.
    """
    if not isinstance(device_type, str):
        raise ValueError(
            f"Expected `device_type` of type `str`, got: `{type(device_type)}`"
        )
    if fwd is None:
        return functools.partial(
            custom_fwd, device_type=device_type, cast_inputs=cast_inputs
        )
````
- **EN**: This chunk continues `custom_fwd` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `custom_fwd`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 488-506 / 第 488-506 行
````python
    @functools.wraps(fwd)
    def decorate_fwd(*args, **kwargs):
        args[0]._dtype = torch.get_autocast_dtype(device_type)
        if cast_inputs is None:
            args[0]._fwd_used_autocast = torch.is_autocast_enabled(device_type)
            return fwd(*args, **kwargs)  # pyrefly: ignore [not-callable]
        else:
            autocast_context = torch.is_autocast_enabled(device_type)
            args[0]._fwd_used_autocast = False
            if autocast_context:
                with autocast(device_type=device_type, enabled=False):
                    return fwd(  # pyrefly: ignore  # not-callable
                        *_cast(args, device_type, cast_inputs),
                        **_cast(kwargs, device_type, cast_inputs),
                    )
            else:
                return fwd(*args, **kwargs)  # pyrefly: ignore [not-callable]

    return decorate_fwd
````
- **EN**: This chunk defines `decorate_fwd`, which implements a focused helper used by the surrounding module. Decorators such as `functools.wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `decorate_fwd`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 509-523 / 第 509-523 行
````python
# Autograd ensures incoming gradients are the same type as forward outputs.  Allowing a separate
# cast_inputs argument on custom_bwd is unnecessary and could cause errors if it doesn't match
# cast_inputs supplied to custom_fwd.
def custom_bwd(bwd=None, *, device_type: str):
    """Create a helper decorator for backward methods of custom autograd functions.

    Autograd functions are subclasses of :class:`torch.autograd.Function`.
    Ensures that ``backward`` executes with the same autocast state as ``forward``.
    See the :ref:`example page<amp-custom-examples>` for more detail.

    Args:
        device_type(str):  Device type to use. 'cuda', 'cpu', 'mtia', 'maia', 'xpu' and so on.
            The type is the same as the `type` attribute of a :class:`torch.device`.
            Thus, you may obtain the device type of a tensor using `Tensor.device.type`.
    """
````
- **EN**: This chunk defines `custom_bwd`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `custom_bwd`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 525-541 / 第 525-541 行
````python
    if not isinstance(device_type, str):
        raise ValueError(
            f"Expected `device_type` of type `str`, got: `{type(device_type)}`"
        )
    if bwd is None:
        return functools.partial(custom_bwd, device_type=device_type)

    @functools.wraps(bwd)
    def decorate_bwd(*args, **kwargs):
        with autocast(
            device_type=device_type,
            enabled=args[0]._fwd_used_autocast,
            dtype=args[0]._dtype,
        ):
            return bwd(*args, **kwargs)  # pyrefly: ignore [not-callable]

    return decorate_bwd
````
- **EN**: This chunk defines `decorate_bwd`, which implements a focused helper used by the surrounding module. Decorators such as `functools.wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `decorate_bwd`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Mixed precision**
  - EN: Controls autocast behavior, dtype selection, and gradient scaling around lower-precision execution.
  - CN: 围绕低精度执行控制 autocast 行为、dtype 选择以及梯度缩放。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **is_autocast_available**
  - EN: `is_autocast_available` is one of the main symbols declared or implemented in this file.
  - CN: `is_autocast_available` 是本文件声明或实现的主要符号之一。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **NumPy compatibility**
  - EN: The implementation mirrors NumPy-style semantics to keep array/tensor behavior familiar.
  - CN: 实现对齐 NumPy 风格语义，使数组/张量行为更易理解。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.types`
- **Standard library / 标准库**: `collections`, `functools`, `warnings`, `typing`
- **Other helper packages / 其他辅助包**: `numpy`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `is_autocast_available`, `autocast_decorator`, `autocast`, `_UnmanagedAutocast`, `_enter_autocast`, `_exit_autocast`, `_cast`, `custom_fwd`, `custom_bwd`
