# random.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_numpy/random.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements NumPy-compatibility helpers, wrappers, and interoperability utilities.
- **Purpose (CN)**: 实现 NumPy 兼容辅助逻辑、包装层以及互操作工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11 / 第 1-11 行
````python
# mypy: ignore-errors

"""Wrapper to mimic (parts of) np.random API surface.

NumPy has strict guarantees on reproducibility etc; here we don't give any.

Q: default dtype is float64 in numpy

"""

from __future__ import annotations
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as __future__. The future import postpones annotation evaluation, keeping type hints lightweight at import time. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 __future__。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 13-26 / 第 13-26 行
````python
import functools
from math import sqrt

import torch

from . import _dtypes_impl, _util
from ._normalizations import array_or_scalar, ArrayLike, normalizer


__all__ = [
    "seed",
    "random_sample",
    "sample",
    "random",
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, ., ._normalizations; standard-library helpers such as functools, math. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、.、._normalizations；标准库辅助模块，如 functools、math。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 27-39 / 第 27-39 行
````python
    "rand",
    "randn",
    "normal",
    "choice",
    "randint",
    "shuffle",
    "uniform",
]


def use_numpy_random():
    # local import to avoid ref cycles
    import torch._dynamo.config as config
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._dynamo.config. This chunk defines `use_numpy_random`, which implements a focused helper used by the surrounding module.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._dynamo.config。 这一段定义了 `use_numpy_random`，其作用是实现周边模块使用的关键辅助逻辑。

### Lines 41-52 / 第 41-52 行
````python
    return config.use_numpy_random_stream


def deco_stream(func):
    @functools.wraps(func)
    def inner(*args, **kwds):
        if not use_numpy_random():
            return func(*args, **kwds)
        else:
            import numpy

            from ._ndarray import ndarray
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ndarray; other helper packages such as numpy. This chunk defines `inner`, which implements a focused helper used by the surrounding module. Decorators such as `functools.wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ndarray；其他辅助包，如 numpy。 这一段定义了 `inner`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 54-65 / 第 54-65 行
````python
            f = getattr(numpy.random, func.__name__)

            # numpy funcs accept numpy ndarrays, unwrap
            args = tuple(
                arg.tensor.numpy() if isinstance(arg, ndarray) else arg for arg in args
            )
            kwds = {
                key: val.tensor.numpy() if isinstance(val, ndarray) else val
                for key, val in kwds.items()
            }

            value = f(*args, **kwds)
````
- **EN**: This chunk continues `inner` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `inner`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 67-79 / 第 67-79 行
````python
            # `value` can be either numpy.ndarray or python scalar (or None)
            if isinstance(value, numpy.ndarray):
                value = ndarray(torch.as_tensor(value))

            return value

    return inner


@deco_stream
def seed(seed=None):
    if seed is not None:
        torch.random.manual_seed(seed)
````
- **EN**: This chunk defines `seed`, which implements a focused helper used by the surrounding module. Decorators such as `deco_stream` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `seed`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deco_stream` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 82-94 / 第 82-94 行
````python
@deco_stream
def random_sample(size=None):
    if size is None:
        size = ()
    dtype = _dtypes_impl.default_dtypes().float_dtype
    values = torch.empty(size, dtype=dtype).uniform_()
    return array_or_scalar(values, return_scalar=size == ())


def rand(*size):
    if size == ():
        size = None
    return random_sample(size)
````
- **EN**: This chunk defines `rand`, which implements a focused helper used by the surrounding module. Decorators such as `deco_stream` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `rand`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deco_stream` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 97-107 / 第 97-107 行
````python
sample = random_sample
random = random_sample


@deco_stream
def uniform(low=0.0, high=1.0, size=None):
    if size is None:
        size = ()
    dtype = _dtypes_impl.default_dtypes().float_dtype
    values = torch.empty(size, dtype=dtype).uniform_(low, high)
    return array_or_scalar(values, return_scalar=size == ())
````
- **EN**: This chunk defines `uniform`, which implements a focused helper used by the surrounding module. Decorators such as `deco_stream` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `uniform`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deco_stream` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 110-123 / 第 110-123 行
````python
@deco_stream
def randn(*size):
    dtype = _dtypes_impl.default_dtypes().float_dtype
    values = torch.randn(size, dtype=dtype)
    return array_or_scalar(values, return_scalar=size == ())


@deco_stream
def normal(loc=0.0, scale=1.0, size=None):
    if size is None:
        size = ()
    dtype = _dtypes_impl.default_dtypes().float_dtype
    values = torch.empty(size, dtype=dtype).normal_(loc, scale)
    return array_or_scalar(values, return_scalar=size == ())
````
- **EN**: This chunk defines `normal`, which implements a focused helper used by the surrounding module. Decorators such as `deco_stream` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `normal`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deco_stream` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 126-136 / 第 126-136 行
````python
@deco_stream
def shuffle(x):
    # no @normalizer because we do not cast e.g. lists to tensors
    from ._ndarray import ndarray

    if isinstance(x, torch.Tensor):
        tensor = x
    elif isinstance(x, ndarray):
        tensor = x.tensor
    else:
        raise NotImplementedError("We do not random.shuffle lists in-place")
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ndarray. This chunk defines `shuffle`, which implements a focused helper used by the surrounding module. Decorators such as `deco_stream` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ndarray。 这一段定义了 `shuffle`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deco_stream` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 138-151 / 第 138-151 行
````python
    perm = torch.randperm(tensor.shape[0])
    xp = tensor[perm]
    tensor.copy_(xp)


@deco_stream
def randint(low, high=None, size=None):
    if size is None:
        size = ()
    if not isinstance(size, (tuple, list)):
        size = (size,)
    if high is None:
        low, high = 0, low
    values = torch.randint(low, high, size=size)
````
- **EN**: This chunk defines `randint`, which implements a focused helper used by the surrounding module. Decorators such as `deco_stream` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `randint`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deco_stream` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 152-162 / 第 152-162 行
````python
    return array_or_scalar(values, int, return_scalar=size == ())


@deco_stream
@normalizer
def choice(a: ArrayLike, size=None, replace=True, p: ArrayLike | None = None):
    # https://stackoverflow.com/questions/59461811/random-choice-with-pytorch
    if a.numel() == 1:
        a = torch.arange(a)

    # TODO: check a.dtype is integer -- cf np.random.choice(3.4) which raises
````
- **EN**: This chunk defines `choice`, which implements a focused helper used by the surrounding module. Decorators such as `deco_stream`, `normalizer` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `choice`，其作用是实现周边模块使用的关键辅助逻辑。 像 `deco_stream`、`normalizer` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 164-176 / 第 164-176 行
````python
    # number of draws
    if size is None:
        num_el = 1
    elif _util.is_sequence(size):
        num_el = 1
        for el in size:
            num_el *= el
    else:
        num_el = size

    # prepare the probabilities
    if p is None:
        p = torch.ones_like(a) / a.shape[0]
````
- **EN**: This chunk continues `choice` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `choice`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 178-191 / 第 178-191 行
````python
    # cf https://github.com/numpy/numpy/blob/main/numpy/random/mtrand.pyx#L973
    atol = sqrt(torch.finfo(p.dtype).eps)
    if abs(p.sum() - 1.0) > atol:
        raise ValueError("probabilities do not sum to 1.")

    # actually sample
    indices = torch.multinomial(p, num_el, replacement=replace)

    if _util.is_sequence(size):
        indices = indices.reshape(size)

    samples = a[indices]

    return samples
````
- **EN**: This chunk continues `choice` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `choice`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **NumPy interop**
  - EN: Keeps PyTorch behavior compatible with NumPy expectations where practical.
  - CN: 在可行范围内让 PyTorch 行为与 NumPy 预期保持兼容。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **use_numpy_random**
  - EN: `use_numpy_random` is one of the main symbols declared or implemented in this file.
  - CN: `use_numpy_random` 是本文件声明或实现的主要符号之一。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **NumPy compatibility**
  - EN: The implementation mirrors NumPy-style semantics to keep array/tensor behavior familiar.
  - CN: 实现对齐 NumPy 风格语义，使数组/张量行为更易理解。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `.`, `._normalizations`, `torch._dynamo.config`, `._ndarray`
- **Standard library / 标准库**: `__future__`, `functools`, `math`
- **Other helper packages / 其他辅助包**: `numpy`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `use_numpy_random`, `deco_stream`, `seed`, `random_sample`, `rand`, `uniform`, `randn`, `normal`, `shuffle`
