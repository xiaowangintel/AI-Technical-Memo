# _core.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_subclasses/complex_tensor/_core.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements tensor-subclass helpers, fake tensor support, and subclass-aware dispatch utilities.
- **Purpose (CN)**: 实现张量子类辅助逻辑、fake tensor 支持以及面向子类的 dispatch 工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行
````python
from __future__ import annotations

from typing import Any, TYPE_CHECKING
from typing_extensions import Self

import torch
from torch import Tensor
from torch.autograd import Function
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.autograd; standard-library helpers such as __future__, typing; other helper packages such as typing_extensions. The future import postpones annotation evaluation, keeping type hints lightweight at import time.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.autograd；标准库辅助模块，如 __future__、typing；其他辅助包，如 typing_extensions。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。

### Lines 11-21 / 第 11-21 行
````python
if TYPE_CHECKING:
    from torch._ops import OpOverload
    from torch._prims_common import DeviceLikeType
    from torch.autograd.function import FunctionCtx


class ComplexTensor(Tensor):
    """A class that decomposes all ops on complex Tensors into their real and imaginary parts."""

    _re: Tensor
    _im: Tensor
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._ops, torch._prims_common, torch.autograd.function. It introduces or extends `ComplexTensor`, which hold the main object-oriented state for this portion of the file. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._ops、torch._prims_common、torch.autograd.function。 它引入或扩展了 `ComplexTensor`，这些类承载了本段涉及的主要面向对象状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 23-33 / 第 23-33 行
````python
    def __new__(cls, real: Tensor, imag: Tensor) -> Self:
        """Initialize a ComplexTensor from its real and imaginary parts."""
        from ._ops.common import REAL_TO_COMPLEX

        shape = real.shape
        device = real.device

        # TODO (hameerabbasi): `torch.compile` sometimes fails here without making these
        # contiguous. Why?
        real = real.contiguous()
        imag = imag.contiguous()
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ops.common. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `__new__`, which initializes object state and binds construction-time configuration.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ops.common。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `__new__`，其作用是初始化对象状态并绑定构造期配置。

### Lines 35-48 / 第 35-48 行
````python
        # TODO (hameerabbasi):
        # What should we do with dtype?
        # We could convert to the complex type (float32 -> complex64), but we
        # can't use that model for say `bfloat16` which does not have a
        # corresponding complex dtype.
        # If we want to support this complex rep using any float type (see
        # https://github.com/pytorch/pytorch/issues/95100)
        # We either need to:
        # 1) add the complex types for say `complexbf32`, knowing they can't really be used anywhere
        #    else.
        # 2) We use the real float dtype here, and it is up to the user to know
        #    that dtype=float<size> here really means complex<2xSize> with dtype
        #    matching that of re/im parts alone
        # I'm going with 1 for now, so that I can make gradcheck and some complex
````
- **EN**: This chunk continues `__new__` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `__new__`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 49-59 / 第 49-59 行
````python
        # ops work properly, but might want to discuss this in the RFP.
        dtype = REAL_TO_COMPLEX.get(real.dtype)
        if dtype is None:
            raise TypeError(
                "Unsupported dtype for constituent tensors. Supported dtypes are: "
                f"{set(REAL_TO_COMPLEX.keys())!r}."
            )
        storage_offset = real.storage_offset()
        strides = real.stride()
        layout = real.layout
        pin_memory = real.is_pinned()
````
- **EN**: This chunk continues `__new__` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `__new__`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 61-70 / 第 61-70 行
````python
        if shape != imag.shape:
            raise AssertionError(f"Expected imag shape {shape}, got {imag.shape}")
        if device != imag.device:
            raise AssertionError(f"Expected imag device {device}, got {imag.device}")
        if real.dtype != imag.dtype:
            raise AssertionError(f"Expected imag dtype {real.dtype}, got {imag.dtype}")
        if pin_memory != imag.is_pinned():
            raise AssertionError(
                f"Expected imag pinning {pin_memory}, got {imag.is_pinned()}"
            )
````
- **EN**: This chunk continues `__new__` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `__new__`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 72-84 / 第 72-84 行
````python
        res = Tensor._make_wrapper_subclass(  # type: ignore[attr-defined]
            cls,
            shape,
            device=device,
            dtype=dtype,
            storage_offset=storage_offset,
            strides=strides,
            pin_memory=pin_memory,
            layout=layout,
            requires_grad=False,
        )
        res._re = real.clone().detach()
        res._im = imag.clone().detach()
````
- **EN**: This chunk continues `__new__` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `__new__`，进一步展开其内部控制流或状态更新。

### Lines 86-94 / 第 86-94 行
````python
        return res

    @property
    def re(self) -> Tensor:
        return self._re

    @property
    def im(self) -> Tensor:
        return self._im
````
- **EN**: This chunk defines `im`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `im`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 96-107 / 第 96-107 行
````python
    @classmethod
    def __torch_dispatch__(  # type: ignore[bad-override]
        cls,
        func: OpOverload,
        types: tuple[type, ...],
        # pyrefly: ignore [implicit-any]
        args: tuple = (),
        kwargs: dict[str, Any] | None = None,
    ) -> Any:
        from ._ops.common import lookup_complex

        kwargs = {} if kwargs is None else kwargs
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ops.common. This chunk defines `__torch_dispatch__`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ops.common。 这一段定义了 `__torch_dispatch__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 109-119 / 第 109-119 行
````python
        impl = lookup_complex(func, *args, **kwargs)
        if impl is None:
            return NotImplemented

        return impl(*args, **kwargs)

    @staticmethod
    def from_interleaved(t: Tensor) -> ComplexTensor:
        t_real = torch.real(t)
        t_imag = torch.imag(t) if t.dtype.is_complex else torch.zeros_like(t_real)
        return Complex.apply(t_real, t_imag)
````
- **EN**: This chunk defines `from_interleaved`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `from_interleaved`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 121-134 / 第 121-134 行
````python
    def as_interleaved(self) -> Tensor:
        return torch.complex(self.real, self.imag)

    @staticmethod
    def __tensor_unflatten__(
        inner_tensors: dict[str, Tensor],
        meta: Any,
        outer_size: tuple[int, ...],
        outer_stride: tuple[int, ...],
    ) -> ComplexTensor:
        if meta is not None:
            raise AssertionError(f"meta must be None, got {meta}")
        re, im = inner_tensors["re"], inner_tensors["im"]
        return ComplexTensor(re, im)
````
- **EN**: This chunk defines `__tensor_unflatten__`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__tensor_unflatten__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 136-143 / 第 136-143 行
````python
    def __tensor_flatten__(self) -> tuple[list[str], Any]:
        return ["re", "im"], None

    def __repr__(self, *, tensor_contents: object | None = None) -> str:
        return f"ComplexTensor(real={self.re!r}, imag={self.im!r})"

    def is_pinned(self, device: DeviceLikeType | None = None) -> bool:
        return self.re.is_pinned(device)
````
- **EN**: This chunk defines `is_pinned`, which checks a capability or invariant before later code relies on it. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_pinned`，其作用是检查某项能力或不变量，供后续逻辑依赖。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 146-153 / 第 146-153 行
````python
class Complex(Function):
    @staticmethod
    def forward(ctx: FunctionCtx, real: Tensor, imag: Tensor) -> ComplexTensor:  # type: ignore[bad-override]
        return ComplexTensor(real, imag)

    @staticmethod
    def backward(ctx: FunctionCtx, grad_output: ComplexTensor) -> tuple[Tensor, Tensor]:  # type: ignore[bad-override]
        return grad_output.real, grad_output.imag
````
- **EN**: It introduces or extends `Complex`, which hold the main object-oriented state for this portion of the file. This chunk defines `backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `Complex`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `backward`，其作用是实现与前向路径配套的求导期行为。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor subclassing**
  - EN: Handles tensor subclasses and fake/symbolic tensor behavior without breaking dispatch.
  - CN: 在不破坏 dispatch 的前提下处理张量子类与 fake/符号张量行为。
- **ComplexTensor**
  - EN: `ComplexTensor` is one of the main symbols declared or implemented in this file.
  - CN: `ComplexTensor` 是本文件声明或实现的主要符号之一。
- **Complex**
  - EN: `Complex` is one of the main symbols declared or implemented in this file.
  - CN: `Complex` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.autograd`, `torch._ops`, `torch._prims_common`, `torch.autograd.function`, `._ops.common`
- **Standard library / 标准库**: `__future__`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `ComplexTensor`, `Complex`
