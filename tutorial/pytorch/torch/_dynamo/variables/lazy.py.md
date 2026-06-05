# lazy.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/variables/lazy.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines abstract variable trackers that model Python values during graph capture.
- **Purpose (CN)**: 定义抽象的变量跟踪器，用于在图捕获过程中建模 Python 值。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
```python
from __future__ import annotations

import collections
import functools
import inspect
from typing import Any, TYPE_CHECKING

from ..utils import is_function_or_wrapper
from .base import SourceLocation, VariableTracker, VariableTrackerMeta


if TYPE_CHECKING:
    from collections.abc import Callable
    from typing_extensions import Self

    from .tensor import SymNodeVariable


class LazyCache:
    """Container to cache the real VariableTracker"""
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 22-47
```python
    def __init__(self, value: Any, source: Any) -> None:
        if not isinstance(value, LazySymNodeFormatString):
            assert source
        self.value = value
        self.source = source
        self.name_hint: str | None = None
        self.source_location: SourceLocation | None = None
        self.vt: VariableTracker | None = None

    def realize(self) -> None:
        assert self.vt is None
        from ..symbolic_convert import InstructionTranslator
        from . import builder

        tx = InstructionTranslator.current_tx()

        if isinstance(self.value, LazySymNodeFormatString):
            self.vt = builder.SourcelessBuilder.create(tx, self.value)
        else:
            # Pass allow_lazy_constant=False to prevent VariableBuilder from
            # returning LazyConstantVariable, which would cause infinite recursion
            # when LazyVariableTracker.realize() returns LazyConstantVariable.
            self.vt = builder.VariableBuilder(
                tx, self.source, allow_lazy_constant=False
            )(self.value)
```
- **EN**: Declares `LazyCache`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `LazyCache`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 48-73
```python
        if self.name_hint is not None:
            self.vt.set_name_hint(self.name_hint)

        if self.source_location is not None and self.vt.source_location is None:
            self.vt.set_source_location(self.source_location)

        del self.value
        del self.source
        del self.name_hint
        del self.source_location


class LazyVariableTracker(VariableTracker, metaclass=VariableTrackerMeta):
    """
    A structure that defers the creation of the actual VariableTracker
    for a given underlying value until it is accessed.

    The `realize` function invokes VariableTracker.build() to produce the real object.
    Once a LazyVariableTracker has been realized, internal bookkeeping will
    prevent double realization.

    This object should be utilized for processing containers, or objects that
    reference other objects where we may not want to take on creating all the
    VariableTrackers right away.
    """
```
- **EN**: This module-level block helps capture Python execution for torch.compile and maintain compiler state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 74-98
```python
    # Flag to prevent implicit realization in isinstance checks (inherited by subclasses)
    _no_implicit_realize = True
    _nonvar_fields = {"_cache", *VariableTracker._nonvar_fields}

    @staticmethod
    def create(value: Any, source: Any, **options: Any) -> VariableTracker:
        if type(value) in LazyConstantVariable.supported_types:
            return LazyConstantVariable.create(value, source, **options)

        # Cache based on source when no extra options are passed
        if source is not None and not options:
            from ..symbolic_convert import InstructionTranslator

            tx = InstructionTranslator.current_tx()
            if tx is not None:
                cache = tx.output.variable_tracker_cache
                cached = cache.get(source)
                if cached is not None:
                    return cached
                vt = LazyVariableTracker(LazyCache(value, source), source=source)
                cache[source] = vt
                return vt

        return LazyVariableTracker(LazyCache(value, source), source=source, **options)
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 99-124
```python
    def __init__(self, _cache: LazyCache, **kwargs: Any) -> None:
        assert isinstance(_cache, LazyCache)
        super().__init__(**kwargs)
        self._cache = _cache

    def realize(self) -> VariableTracker:
        """Force construction of the real VariableTracker"""
        if self._cache.vt is None:
            self._cache.realize()
            assert self._cache.vt is not None
        return self._cache.vt

    def lazy_isinstance(self, cls: type) -> bool:
        """Check isinstance after realizing, used by ImplicitRealizingVariableTrackerMeta"""
        return type.__instancecheck__(cls, self.realize())

    def unwrap(self) -> VariableTracker | Self:
        """Return the real VariableTracker if it already exists"""
        if self.is_realized():
            assert self._cache.vt is not None
            return self._cache.vt
        return self

    def is_realized(self) -> bool:
        return self._cache.vt is not None
```
- **EN**: Declares `LazyVariableTracker(VariableTracker)`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `LazyVariableTracker(VariableTracker)`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 125-151
```python
    def clone(self, **kwargs: Any) -> VariableTracker:
        assert kwargs.get("_cache", self._cache) is self._cache
        if kwargs.get("source", self.source) is not self.source:
            self.realize()
        return VariableTracker.clone(self.unwrap(), **kwargs)

    def peek_type(self) -> type[Any]:
        assert not self.is_realized()
        return type(self._cache.value)

    def peek_value(self) -> Any:
        assert not self.is_realized()
        return self._cache.value

    def set_name_hint(self, name: str) -> None:
        if self.is_realized():
            self._cache.vt.set_name_hint(name)  # type: ignore[union-attr]
        else:
            self._cache.name_hint = name

    def set_source_location(self, source_location: SourceLocation) -> None:
        self.source_location = source_location
        if self.is_realized():
            self._cache.vt.set_source_location(source_location)  # type: ignore[union-attr]
        else:
            self._cache.source_location = source_location
```
- **EN**: Declares `LazyVariableTracker(VariableTracker)`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `LazyVariableTracker(VariableTracker)`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 152-169
```python
    def __str__(self) -> str:
        variable_info = "LazyVariableTracker("
        if self.is_realized():
            variable_info += f"realized: {repr(self.unwrap())})"
        else:
            variable_info += f"unrealized: {self.peek_type()})"

        return variable_info

    def __getattr__(self, item: str) -> Any:
        return getattr(self.realize(), item)

    # most methods are auto-generated below, these are the ones we want to exclude
    visit = VariableTracker.visit  # type: ignore[assignment]
    __repr__ = __str__

    @classmethod
    def realize_all(
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 170-187
```python
        cls,
        value: Any,
        cache: dict[int, tuple[Any, Any]] | None = None,
        *,
        allow_lazy_constant: bool = False,
    ) -> Any:
        """
        Walk an object and realize all LazyVariableTrackers inside it.
        """
        if cache is None:
            cache = {}

        idx = id(value)
        if idx in cache:
            return cache[idx][0]

        value_cls = type(value)
        if issubclass(value_cls, LazyVariableTracker):
```
- **EN**: This block continues `LazyVariableTracker.realize_all` and works to capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `LazyVariableTracker.realize_all`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 188-205
```python
            # Allow LazyConstantVariable to stay lazy when returning from a frame
            keep_lazy = allow_lazy_constant and isinstance(value, LazyConstantVariable)
            if keep_lazy:
                result = value
            else:
                result = cls.realize_all(
                    value.realize(), cache, allow_lazy_constant=allow_lazy_constant
                )
        elif issubclass(value_cls, VariableTracker):
            # update value in-place
            result = value
            # update cache now to prevent infinite recursion
            cache[idx] = (result, value)
            value_dict = value.__dict__
            nonvars = value._nonvar_fields
            for key in value_dict:
                if key not in nonvars:
                    value_dict[key] = cls.realize_all(
```
- **EN**: This block continues `LazyVariableTracker.realize_all` and works to initialize learnable tensors and related state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `LazyVariableTracker.realize_all`，用于初始化可学习张量及相关状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 206-229
```python
                        value_dict[key], cache, allow_lazy_constant=allow_lazy_constant
                    )
        elif value_cls is list:
            result = [
                cls.realize_all(v, cache, allow_lazy_constant=allow_lazy_constant)
                for v in value
            ]
        elif value_cls is tuple:
            result = tuple(
                cls.realize_all(v, cache, allow_lazy_constant=allow_lazy_constant)
                for v in value
            )
        elif value_cls in (dict, collections.OrderedDict):
            result = {
                k: cls.realize_all(v, cache, allow_lazy_constant=allow_lazy_constant)
                for k, v in list(value.items())
            }
        else:
            result = value

        # save `value` to keep it alive and ensure id() isn't reused
        cache[idx] = (result, value)
        return result
```
- **EN**: This block continues `LazyVariableTracker` and works to capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `LazyVariableTracker`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 230-252
```python
    def is_hashable(self) -> bool:
        # Checks that the underlying value is hashable without realizing the VT.
        # This is used by ConstDictVariable tracker to find if the key LazyVT
        # can be hashed.
        def _helper(value: Any) -> bool:
            # TODO: Add support for more types
            return (
                inspect.isbuiltin(value)
                or issubclass(type(value), type)
                or is_function_or_wrapper(value)
            )

        assert not self.is_realized()
        value = self._cache.value
        if isinstance(value, tuple):
            return all(_helper(v) for v in value)
        return _helper(value)

    def original_value(self) -> Any:
        # Returns the value without realizing the VT.
        assert not self.is_realized()
        return self._cache.value
```
- **EN**: Declares `LazyVariableTracker(VariableTracker)`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `LazyVariableTracker(VariableTracker)`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 253-279
```python
    def original_source(self) -> Any:
        # Returns the source without realizing the VT.
        assert not self.is_realized()
        return self._cache.source


class LazyConstantVariable(LazyVariableTracker):
    """
    A lazy variable tracker for constants (int, float, bool, str) that defers
    guarding until the value is actually used in a way that requires it.

    This allows constants that are just passed through (e.g., returned without
    being used in control flow or math) to avoid unnecessary recompilation when
    their values change.
    """

    supported_types = (int, float, bool, str)

    @staticmethod
    def create(
        value: Any,
        source: Any,
        **options: Any,
    ) -> LazyConstantVariable:
        assert type(value) in LazyConstantVariable.supported_types
        return LazyConstantVariable(LazyCache(value, source), source=source, **options)
```
- **EN**: These decorators register or transform the following definition so it can enforce guards that validate whether cached compiled code can be reused.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够实施守卫检查以判断缓存的编译代码能否复用。

### Lines 280-298
```python

class LazySymNodeFormatString:
    def __init__(
        self, sym_node_variable: SymNodeVariable, fmt_spec_var: VariableTracker
    ) -> None:
        from .constant import ConstantVariable

        self.sym_node_var = sym_node_variable
        self.fmt_var = ConstantVariable.create(
            "{:" + fmt_spec_var.as_python_constant() + "}"
        )

    def __repr__(self) -> str:
        return str.format(
            self.fmt_var.as_python_constant(),
            str(self.sym_node_var.evaluate_expr()),
        )
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 299-317
```python
def _create_realize_and_forward(
    name: str,
) -> Callable[[LazyVariableTracker, Any, Any], Any]:
    @functools.wraps(getattr(VariableTracker, name))
    def realize_and_forward(
        self: LazyVariableTracker, *args: Any, **kwargs: Any
    ) -> Any:
        return getattr(self.realize(), name)(*args, **kwargs)

    return realize_and_forward


def _populate() -> None:
    for name, value in VariableTracker.__dict__.items():
        if name not in LazyVariableTracker.__dict__:
            if callable(value):
                setattr(LazyVariableTracker, name, _create_realize_and_forward(name))
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 318-318
```python
_populate()
```
- **EN**: This module-level block helps capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这个模块级代码块用于为 torch.compile 捕获 Python 执行并维护编译器状态。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Python execution capture / Python 执行捕获**
  - EN: The file hooks or rewrites Python execution machinery to observe user programs.
  - CN: 该文件会挂接或改写 Python 执行机制，以观察用户程序。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `..utils`, `.base`, `.tensor`, `..symbolic_convert`, `.`, `.constant`
- **Standard library / 标准库**: `__future__`, `collections`, `functools`, `inspect`, `typing`, `collections.abc`, `typing_extensions`
- **Primary symbols / 核心符号**: `LazyCache`, `LazyVariableTracker`, `LazyConstantVariable`, `LazySymNodeFormatString`, `_create_realize_and_forward`, `_populate`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
