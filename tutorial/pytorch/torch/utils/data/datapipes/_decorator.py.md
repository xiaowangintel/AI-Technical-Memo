# _decorator.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/data/datapipes/_decorator.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities. This specific file centers on `_decorator.py`. Dataset iteration, batching, and worker orchestration are central concerns here.
- **Purpose (CN) / 用途（中文）**: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。 该文件具体围绕 `_decorator.py` 展开。 数据集迭代、批处理以及 worker 协调是这里的核心关注点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-19 / 第 1-19 行
```python
# mypy: allow-untyped-defs
import inspect
from collections.abc import Callable
from functools import wraps
from typing import Any, get_type_hints

from torch.utils.data.datapipes._typing import _DataPipeMeta
from torch.utils.data.datapipes.datapipe import IterDataPipe, MapDataPipe


######################################################
# Functional API
######################################################
class functional_datapipe:
    name: str

    def __init__(self, name: str, enable_df_api_tracing=False) -> None:
        """
        Define a functional datapipe.
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch.utils.data.datapipes._typing:_DataPipeMeta, torch.utils.data.datapipes.datapipe:IterDataPipe, torch.utils.data.datapipes.datapipe:MapDataPipe; standard-library helpers such as inspect, collections.abc:Callable, functools:wraps, typing:Any. It introduces or extends class-level abstractions such as `functional_datapipe`, which organize state and behavior for this subsystem. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch.utils.data.datapipes._typing:_DataPipeMeta, torch.utils.data.datapipes.datapipe:IterDataPipe, torch.utils.data.datapipes.datapipe:MapDataPipe；标准库辅助模块，如 inspect, collections.abc:Callable, functools:wraps, typing:Any。 它引入或扩展了 `functional_datapipe` 等类级抽象，用于组织该子系统的状态与行为。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 21-44 / 第 21-44 行
```python
        Args:
            enable_df_api_tracing - if set, any returned DataPipe would accept
            DataFrames API in tracing mode.
        """
        self.name = name
        self.enable_df_api_tracing = enable_df_api_tracing

    def __call__(self, cls):
        if issubclass(cls, IterDataPipe):
            if isinstance(cls, type):  # type: ignore[arg-type]
                if not isinstance(cls, _DataPipeMeta):
                    raise TypeError(
                        "`functional_datapipe` can only decorate IterDataPipe"
                    )
            # with non_deterministic decorator
            else:
                if not isinstance(cls, non_deterministic) and not (
                    hasattr(cls, "__self__")
                    and isinstance(cls.__self__, non_deterministic)
                ):
                    raise TypeError(
                        "`functional_datapipe` can only decorate IterDataPipe"
                    )
            IterDataPipe.register_datapipe_as_function(
```
- **EN**: It introduces or extends class-level abstractions such as `functional_datapipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `functional_datapipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 45-65 / 第 45-65 行
```python
                self.name, cls, enable_df_api_tracing=self.enable_df_api_tracing
            )
        elif issubclass(cls, MapDataPipe):
            MapDataPipe.register_datapipe_as_function(self.name, cls)

        return cls


######################################################
# Determinism
######################################################
_determinism: bool = False


class guaranteed_datapipes_determinism:
    prev: bool

    def __init__(self) -> None:
        global _determinism
        self.prev = _determinism
        _determinism = True
```
- **EN**: It introduces or extends class-level abstractions such as `functional_datapipe`, `guaranteed_datapipes_determinism`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `functional_datapipe`, `guaranteed_datapipes_determinism` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 67-90 / 第 67-90 行
```python
    def __enter__(self) -> None:
        pass

    def __exit__(self, exc_type: Any, exc_value: Any, traceback: Any) -> None:
        global _determinism
        _determinism = self.prev


class non_deterministic:
    cls: type[IterDataPipe] | None = None
    # TODO: Lambda for picking
    deterministic_fn: Callable[..., bool]

    def __init__(self, arg: type[IterDataPipe] | Callable[..., bool]) -> None:
        # 1. Decorator doesn't have any argument
        if isinstance(arg, type):  # type: ignore[arg-type]
            if not issubclass(arg, IterDataPipe):  # type: ignore[arg-type]
                raise TypeError(
                    "Only `IterDataPipe` can be decorated with `non_deterministic`"
                    f", but {arg.__name__} is found"
                )
            self.cls = arg  # type: ignore[assignment]
        # 2. Decorator has an argument of a function
        #    This class should behave differently given different inputs. Use this
```
- **EN**: It introduces or extends class-level abstractions such as `guaranteed_datapipes_determinism`, `non_deterministic`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `guaranteed_datapipes_determinism`, `non_deterministic` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 91-109 / 第 91-109 行
```python
        #    function to verify the determinism for each instance.
        #    When the function returns True, the instance is non-deterministic. Otherwise,
        #    the instance is a deterministic DataPipe.
        elif isinstance(arg, Callable):  # type:ignore[arg-type]
            self.deterministic_fn = arg
        else:
            raise TypeError(f"{arg} can not be decorated by non_deterministic")

    def __call__(self, *args, **kwargs):
        global _determinism
        #  Decorate IterDataPipe
        if self.cls is not None:
            if _determinism:
                raise TypeError(
                    f"{self.cls.__name__} is non-deterministic, but you set 'guaranteed_datapipes_determinism'. "
                    "You can turn off determinism for this DataPipe if that is acceptable "
                    "for your application"
                )
            return self.cls(*args, **kwargs)  # type: ignore[call-arg]
```
- **EN**: It introduces or extends class-level abstractions such as `non_deterministic`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `non_deterministic` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 111-134 / 第 111-134 行
```python
        # Decorate with a functional argument
        if not (
            isinstance(args[0], type) and issubclass(args[0], IterDataPipe)  # type: ignore[arg-type]
        ):
            raise TypeError(
                f"Only `IterDataPipe` can be decorated, but {args[0].__name__} is found"
            )
        self.cls = args[0]
        return self.deterministic_wrapper_fn

    def deterministic_wrapper_fn(self, *args, **kwargs) -> IterDataPipe:
        res = self.deterministic_fn(*args, **kwargs)
        if not isinstance(res, bool):
            raise TypeError(
                "deterministic_fn of `non_deterministic` decorator is required "
                f"to return a boolean value, but {type(res)} is found"
            )
        global _determinism
        if _determinism and res:
            raise TypeError(
                f"{self.cls.__name__} is non-deterministic with the inputs, but you set "  # type: ignore[union-attr]
                "'guaranteed_datapipes_determinism'. You can turn off determinism "
                "for this DataPipe if that is acceptable for your application"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `non_deterministic`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `non_deterministic` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 135-158 / 第 135-158 行
```python
        return self.cls(*args, **kwargs)  # type: ignore[call-arg, misc]


######################################################
# Type validation
######################################################
# Validate each argument of DataPipe with hint as a subtype of the hint.
def argument_validation(f):
    signature = inspect.signature(f)
    hints = get_type_hints(f)

    @wraps(f)
    def wrapper(*args, **kwargs):
        bound = signature.bind(*args, **kwargs)
        for argument_name, value in bound.arguments.items():
            if argument_name in hints and isinstance(
                hints[argument_name], _DataPipeMeta
            ):
                hint = hints[argument_name]
                if not isinstance(value, IterDataPipe):
                    raise TypeError(
                        f"Expected argument '{argument_name}' as a IterDataPipe, but found {type(value)}"
                    )
                if not value.type.issubtype(hint.type):
```
- **EN**: It introduces or extends class-level abstractions such as `non_deterministic`, which organize state and behavior for this subsystem. Key callable entry points in this range include `argument_validation`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `non_deterministic` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `argument_validation`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 159-179 / 第 159-179 行
```python
                    raise TypeError(
                        f"Expected type of argument '{argument_name}' as a subtype of "
                        f"hint {hint.type}, but found {value.type}"
                    )

        return f(*args, **kwargs)

    return wrapper


# Default value is True
_runtime_validation_enabled: bool = True


class runtime_validation_disabled:
    prev: bool

    def __init__(self) -> None:
        global _runtime_validation_enabled
        self.prev = _runtime_validation_enabled
        _runtime_validation_enabled = False
```
- **EN**: It introduces or extends class-level abstractions such as `runtime_validation_disabled`, which organize state and behavior for this subsystem. Key callable entry points in this range include `argument_validation`. They package a focused unit of behavior behind named helpers or APIs. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `runtime_validation_disabled` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `argument_validation`，它们把聚焦的行为封装成具名辅助函数或 API。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 181-197 / 第 181-197 行
```python
    def __enter__(self) -> None:
        pass

    def __exit__(self, exc_type: Any, exc_value: Any, traceback: Any) -> None:
        global _runtime_validation_enabled
        _runtime_validation_enabled = self.prev


# Runtime checking
# Validate output data is subtype of return hint
def runtime_validation(f):
    # TODO:
    # Can be extended to validate '__getitem__' and nonblocking
    if f.__name__ != "__iter__":
        raise TypeError(
            f"Can not decorate function {f.__name__} with 'runtime_validation'"
        )
```
- **EN**: It introduces or extends class-level abstractions such as `runtime_validation_disabled`, which organize state and behavior for this subsystem. Key callable entry points in this range include `runtime_validation`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `runtime_validation_disabled` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `runtime_validation`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 199-213 / 第 199-213 行
```python
    @wraps(f)
    def wrapper(self):
        global _runtime_validation_enabled
        if not _runtime_validation_enabled:
            yield from f(self)
        else:
            it = f(self)
            for d in it:
                if not self.type.issubtype_of_instance(d):
                    raise RuntimeError(
                        f"Expected an instance as subtype of {self.type}, but found {d}({type(d)})"
                    )
                yield d

    return wrapper
```
- **EN**: Key callable entry points in this range include `runtime_validation`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `runtime_validation`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities.
  - CN: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。
- **DataPipes**
  - EN: Builds composable input pipelines that lazily transform or route dataset elements.
  - CN: 构建可组合的输入流水线，以惰性方式变换或路由数据集元素。
- **functional_datapipe**
  - EN: `functional_datapipe` is one of the main classes that structures the file's behavior.
  - CN: `functional_datapipe` 是组织该文件行为的核心类之一。
- **guaranteed_datapipes_determinism**
  - EN: `guaranteed_datapipes_determinism` is one of the main classes that structures the file's behavior.
  - CN: `guaranteed_datapipes_determinism` 是组织该文件行为的核心类之一。
- **argument_validation**
  - EN: `argument_validation` is a representative function that exposes or coordinates an important action in this module.
  - CN: `argument_validation` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
- **runtime_validation**
  - EN: `runtime_validation` is a representative function that exposes or coordinates an important action in this module.
  - CN: `runtime_validation` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch.utils.data.datapipes._typing:_DataPipeMeta`, `torch.utils.data.datapipes.datapipe:IterDataPipe`, `torch.utils.data.datapipes.datapipe:MapDataPipe`
- **Python standard library / Python 标准库**: `inspect`, `collections.abc:Callable`, `functools:wraps`, `typing:Any`, `typing:get_type_hints`
- **Primary symbols / 核心符号**: `functional_datapipe`, `guaranteed_datapipes_determinism`, `non_deterministic`, `runtime_validation_disabled`, `argument_validation`, `runtime_validation`
