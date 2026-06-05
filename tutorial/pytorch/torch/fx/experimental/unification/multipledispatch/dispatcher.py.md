# dispatcher.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/unification/multipledispatch/dispatcher.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```python
from __future__ import annotations

import inspect
import itertools as itl
from typing import Any, TYPE_CHECKING, TypeVar
from typing_extensions import deprecated
from warnings import warn


if TYPE_CHECKING:
    from collections.abc import Callable, Generator, Iterable, Iterator

from .conflict import ambiguities, AmbiguityWarning, ordering, super_signature
from .utils import expand_tuples
from .variadic import isvariadic, Variadic


_T = TypeVar("_T")
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 21-44
```python
__all__ = [
    "MDNotImplementedError",
    "ambiguity_warn",
    "halt_ordering",
    "restart_ordering",
    "variadic_signature_matches_iter",
    "variadic_signature_matches",
    "Dispatcher",
    "source",
    "MethodDispatcher",
    "str_signature",
    "warning_text",
]


class MDNotImplementedError(NotImplementedError):
    """A NotImplementedError for multiple dispatch"""


def ambiguity_warn(
    dispatcher: Dispatcher, ambiguities: set[tuple[tuple[type, ...], tuple[type, ...]]]
) -> None:
    """Raise warning when ambiguity is detected.
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 45-67
```python
    Parameters
    ----------
    dispatcher : Dispatcher
        The dispatcher on which the ambiguity was detected
    ambiguities : set
        Set of type signature pairs that are ambiguous within this dispatcher

    See Also
    --------
    Dispatcher.add
    warning_text
    """
    warn(warning_text(dispatcher.name, ambiguities), AmbiguityWarning)


@deprecated(
    "`halt_ordering` is deprecated, you can safely remove this call.",
    category=FutureWarning,
)
def halt_ordering() -> None:
    """Deprecated interface to temporarily disable ordering."""
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 68-90
```python
@deprecated(
    "`restart_ordering` is deprecated, if you would like to eagerly order the dispatchers, "
    "you should call the `reorder()` method on each dispatcher.",
    category=FutureWarning,
)
def restart_ordering(on_ambiguity: Callable[..., None] = ambiguity_warn) -> None:
    """Deprecated interface to temporarily resume ordering."""


def variadic_signature_matches_iter(
    types: tuple[type, ...], full_signature: tuple[type, ...]
) -> Generator[bool, None, None]:
    """Check if a set of input types matches a variadic signature.

    Notes
    -----
    The algorithm is as follows:

    Initialize the current signature to the first in the sequence.
    For each type in ``types``:

    - If the current signature is variadic
```
- **EN**: These decorators register or transform the following definition so it can initialize learnable tensors and related state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够初始化可学习张量及相关状态。

### Lines 91-108
```python
      - If the type matches the signature, yield True
      - Else, try to get the next signature.
        If no signatures are left we can't possibly have a match,
        so yield False.

    - Else, yield True if the type matches the current signature.
      Get the next signature.
    """
    sigiter = iter(full_signature)
    sig = next(sigiter)
    for typ in types:
        matches = issubclass(typ, sig)
        yield matches
        if not isvariadic(sig):
            # we're not matching a variadic argument, so move to the next
            # element in the signature
            sig = next(sigiter)
    else:
```
- **EN**: This block continues `variadic_signature_matches_iter` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also streams values incrementally; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `variadic_signature_matches_iter`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会以增量方式产生值；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 109-129
```python
        try:
            sig = next(sigiter)
        except StopIteration:
            if not isvariadic(sig):
                raise AssertionError("Expected variadic signature") from None
            yield True
        else:
            # We have signature items left over, so all of our arguments
            # haven't matched
            yield False


def variadic_signature_matches(
    types: tuple[type, ...], full_signature: tuple[type, ...]
) -> bool:
    # No arguments always matches a variadic signature
    if not full_signature:
        raise AssertionError("full_signature is empty")
    return all(variadic_signature_matches_iter(types, full_signature))
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding; streams values incrementally.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；以增量方式产生值。

### Lines 130-155
```python
class Dispatcher:
    """Dispatch methods based on type signature
    Use ``dispatch`` to add implementations
    Examples
    --------
    >>> # xdoctest: +SKIP("bad import name")
    >>> from multipledispatch import dispatch
    >>> @dispatch(int)
    ... def f(x):
    ...     return x + 1
    >>> @dispatch(float)
    ... def f(x):
    ...     return x - 1
    >>> f(3)
    4
    >>> f(3.0)
    2.0
    """

    __slots__ = "__name__", "name", "funcs", "_ordering", "_cache", "doc"

    def __init__(self, name: str, doc: str | None = None) -> None:
        self.name = self.__name__ = name
        self.funcs: dict[tuple[type, ...], Callable[..., object]] = {}
        self.doc = doc
```
- **EN**: Declares `Dispatcher`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `Dispatcher`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 156-181
```python
        self._cache: dict[tuple[type, ...], Callable[..., object]] = {}

    def register(
        self, *types: type, **kwargs: object
    ) -> Callable[[Callable[..., _T]], Callable[..., _T]]:
        """register dispatcher with new implementation
        >>> # xdoctest: +SKIP
        >>> f = Dispatcher("f")
        >>> @f.register(int)
        ... def inc(x):
        ...     return x + 1
        >>> @f.register(float)
        ... def dec(x):
        ...     return x - 1
        >>> @f.register(list)
        ... @f.register(tuple)
        ... def reverse(x):
        ...     return x[::-1]
        >>> f(1)
        2
        >>> f(1.0)
        0.0
        >>> f([1, 2, 3])
        [3, 2, 1]
        """
```
- **EN**: Declares `Dispatcher`; this class packages state and methods that route operators through dispatch and decomposition helpers.
- **CN**: 声明 `Dispatcher`；该类封装了状态与方法，用于通过分发与分解辅助逻辑路由算子。

### Lines 182-204
```python
        def _df(func: Callable[..., _T]) -> Callable[..., _T]:
            self.add(types, func, **kwargs)  # type: ignore[call-arg]
            return func

        return _df

    @classmethod
    def get_func_params(
        cls, func: Callable[..., object]
    ) -> Iterable[inspect.Parameter] | None:
        if hasattr(inspect, "signature"):
            sig = inspect.signature(func)
            return sig.parameters.values()

    @classmethod
    def get_func_annotations(
        cls, func: Callable[..., object]
    ) -> tuple[type, ...] | None:
        """get annotations of function positional parameters"""
        params = cls.get_func_params(func)
        if params:
            Parameter = inspect.Parameter
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 205-222
```python
            params = (
                param
                for param in params
                if param.kind
                in (Parameter.POSITIONAL_ONLY, Parameter.POSITIONAL_OR_KEYWORD)
            )

            annotations = tuple(param.annotation for param in params)

            if all(ann is not Parameter.empty for ann in annotations):
                return annotations

    def add(self, signature: tuple[type, ...], func: Callable[..., object]) -> None:
        """Add new types/method pair to dispatcher
        >>> # xdoctest: +SKIP
        >>> D = Dispatcher("add")
        >>> D.add((int, int), lambda x, y: x + y)
        >>> D.add((float, float), lambda x, y: x + y)
```
- **EN**: Declares `Dispatcher`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `Dispatcher`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 223-246
```python
        >>> D(1, 2)
        3
        >>> D(1, 2.0)
        Traceback (most recent call last):
        ...
        NotImplementedError: Could not find signature for add: <int, float>
        >>> # When ``add`` detects a warning it calls the ``on_ambiguity`` callback
        >>> # with a dispatcher/itself, and a set of ambiguous type signature pairs
        >>> # as inputs.  See ``ambiguity_warn`` for an example.
        """
        # Handle annotations
        if not signature:
            annotations = self.get_func_annotations(func)
            if annotations:
                signature = annotations

        # Handle union types
        if any(isinstance(typ, tuple) for typ in signature):
            for typs in expand_tuples(signature):
                self.add(typs, func)
            return

        new_signature = []
```
- **EN**: This block continues `Dispatcher.add` and works to trace Python execution into an intermediate graph representation. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `Dispatcher.add`，用于将 Python 执行过程跟踪为中间图表示。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 247-273
```python
        for index, typ in enumerate(signature, start=1):
            if not isinstance(typ, (type, list)):
                str_sig = ", ".join(
                    c.__name__ if isinstance(c, type) else str(c) for c in signature
                )
                raise TypeError(
                    f"Tried to dispatch on non-type: {typ}\n"
                    f"In signature: <{str_sig}>\n"
                    f"In function: {self.name}"
                )

            # handle variadic signatures
            if isinstance(typ, list):
                if index != len(signature):
                    raise TypeError("Variadic signature must be the last element")

                if len(typ) != 1:
                    raise TypeError(
                        "Variadic signature must contain exactly one element. "
                        "To use a variadic union type place the desired types "
                        "inside of a tuple, e.g., [(int, str)]"
                    )
                # pyrefly: ignore [bad-specialization]
                new_signature.append(Variadic[typ[0]])
            else:
                new_signature.append(typ)  # pyrefly: ignore[bad-argument-type]
```
- **EN**: This block continues `Dispatcher.add` and works to route operators through dispatch and decomposition helpers. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `Dispatcher.add`，用于通过分发与分解辅助逻辑路由算子。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 274-297
```python
        self.funcs[tuple(new_signature)] = func
        self._cache.clear()

        try:
            del self._ordering
        except AttributeError:
            pass

    @property
    def ordering(self) -> list[tuple[type, ...]]:
        try:
            return self._ordering
        except AttributeError:
            return self.reorder()

    def reorder(
        self, on_ambiguity: Callable[..., None] = ambiguity_warn
    ) -> list[tuple[type, ...]]:
        self._ordering = od = ordering(self.funcs)
        amb = ambiguities(self.funcs)
        if amb:
            on_ambiguity(self, amb)
        return od
```
- **EN**: These decorators register or transform the following definition so it can validate invariants and surface meaningful failures.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够校验不变量并给出有意义的失败信息。

### Lines 298-320
```python
    def __call__(self, *args: object, **kwargs: object) -> object:
        types = tuple(type(arg) for arg in args)
        try:
            func = self._cache[types]
        except KeyError as e:
            func = self.dispatch(*types)
            if not func:
                raise NotImplementedError(
                    f"Could not find signature for {self.name}: <{str_signature(types)}>"
                ) from e
            self._cache[types] = func
        try:
            return func(*args, **kwargs)

        except MDNotImplementedError as e:
            funcs = self.dispatch_iter(*types)
            next(funcs)  # burn first
            for func in funcs:
                try:
                    return func(*args, **kwargs)
                except MDNotImplementedError:
                    pass
```
- **EN**: Defines the `Dispatcher.__call__` method; this block introduces logic that route operators through dispatch and decomposition helpers.
- **CN**: 定义`Dispatcher.__call__` 方法；该代码块引入了用于通过分发与分解辅助逻辑路由算子的逻辑。

### Lines 321-338
```python
            raise NotImplementedError(
                "Matching functions for "
                f"{self.name}: <{str_signature(types)}> found, but none completed successfully",
            ) from e

    def __str__(self) -> str:
        return f"<dispatched {self.name}>"

    __repr__ = __str__

    def dispatch(self, *types: type) -> Callable[..., object] | None:
        """Determine appropriate implementation for this type signature
        This method is internal.  Users should call this object as a function.
        Implementation resolution occurs within the ``__call__`` method.
        >>> # xdoctest: +SKIP
        >>> from multipledispatch import dispatch
        >>> @dispatch(int)
        ... def inc(x):
```
- **EN**: Declares `Dispatcher`; this class packages state and methods that route operators through dispatch and decomposition helpers.
- **CN**: 声明 `Dispatcher`；该类封装了状态与方法，用于通过分发与分解辅助逻辑路由算子。

### Lines 339-356
```python
        ...     return x + 1
        >>> implementation = inc.dispatch(int)
        >>> implementation(3)
        4
        >>> print(inc.dispatch(float))
        None
        See Also:
          ``multipledispatch.conflict`` - module to determine resolution order
        """

        if types in self.funcs:
            return self.funcs[types]

        try:
            return next(self.dispatch_iter(*types))
        except StopIteration:
            return None
```
- **EN**: This block continues `Dispatcher` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `Dispatcher`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 357-382
```python
    def dispatch_iter(
        self, *types: type
    ) -> Generator[Callable[..., object], None, None]:
        n = len(types)
        for signature in self.ordering:
            if len(signature) == n and all(map(issubclass, types, signature)):
                result = self.funcs[signature]
                yield result
            elif len(signature) and isvariadic(signature[-1]):
                if variadic_signature_matches(types, signature):
                    result = self.funcs[signature]
                    yield result

    @deprecated(
        "`resolve()` is deprecated, use `dispatch(*types)`", category=FutureWarning
    )
    def resolve(self, types: tuple[type, ...]) -> Callable[..., object] | None:
        """Determine appropriate implementation for this type signature
        .. deprecated:: 0.4.4
            Use ``dispatch(*types)`` instead
        """
        return self.dispatch(*types)

    def __getstate__(self) -> dict[str, Any]:
        return {"name": self.name, "funcs": self.funcs}
```
- **EN**: These decorators register or transform the following definition so it can route operators through dispatch and decomposition helpers.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够通过分发与分解辅助逻辑路由算子。

### Lines 383-409
```python
    def __setstate__(self, d: dict[str, Any]) -> None:
        self.name = d["name"]
        self.funcs = d["funcs"]
        self._ordering = ordering(self.funcs)
        self._cache = {}

    @property
    def __doc__(self):  # type: ignore[override]
        docs = [f"Multiply dispatched method: {self.name}"]

        if self.doc:
            docs.append(self.doc)

        other: list[str] = []
        for sig in self.ordering[::-1]:
            func = self.funcs[sig]
            if func.__doc__:
                s = f"Inputs: <{str_signature(sig)}>\n"
                s += "-" * len(s) + "\n"
                s += func.__doc__.strip()
                docs.append(s)
            else:
                other.append(str_signature(sig))

        if other:
            docs.append("Other signatures:\n    " + "\n    ".join(other))
```
- **EN**: This block attaches or refines API documentation so public operators expose richer help text.
- **CN**: 该代码块补充或细化 API 文档，使公开算子能够暴露更完整的帮助信息。

### Lines 410-435
```python
        return "\n\n".join(docs)

    def _help(self, *args: object) -> str | None:
        return self.dispatch(*map(type, args)).__doc__

    def help(self, *args: object, **kwargs: object) -> None:
        """Print docstring for the function corresponding to inputs"""
        print(self._help(*args))

    def _source(self, *args: object) -> str:
        func = self.dispatch(*map(type, args))
        if not func:
            raise TypeError("No function found")
        return source(func)

    def source(self, *args: object, **kwargs: object) -> None:
        """Print source code for the function corresponding to inputs"""
        print(self._source(*args))


def source(func: Callable[..., object]) -> str:
    s = f"File: {inspect.getsourcefile(func)}\n\n"
    s = s + inspect.getsource(func)
    return s
```
- **EN**: This block attaches or refines API documentation so public operators expose richer help text.
- **CN**: 该代码块补充或细化 API 文档，使公开算子能够暴露更完整的帮助信息。

### Lines 436-456
```python
class MethodDispatcher(Dispatcher):
    """Dispatch methods based on type signature
    See Also:
        Dispatcher
    """

    __slots__ = ("obj", "cls")

    @classmethod
    def get_func_params(
        cls, func: Callable[..., object]
    ) -> Iterator[inspect.Parameter] | None:
        if hasattr(inspect, "signature"):
            sig = inspect.signature(func)
            return itl.islice(sig.parameters.values(), 1, None)

    def __get__(self, instance: object | None, owner: type) -> MethodDispatcher:
        self.obj = instance
        self.cls = owner
        return self
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 457-474
```python
    def __call__(self, *args: object, **kwargs: object) -> object:
        types = tuple(type(arg) for arg in args)
        func = self.dispatch(*types)
        if not func:
            raise NotImplementedError(
                f"Could not find signature for {self.name}: <{str_signature(types)}>"
            )
        return func(self.obj, *args, **kwargs)


def str_signature(sig: Iterable[type]) -> str:
    """String representation of type signature
    >>> str_signature((int, float))
    'int, float'
    """
    return ", ".join(cls.__name__ for cls in sig)
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 475-488
```python
def warning_text(name: str, amb: set[tuple[tuple[type, ...], tuple[type, ...]]]) -> str:
    """The text for ambiguity warnings"""
    text = f"\nAmbiguities exist in dispatched function {name}\n\n"
    text += "The following signatures may result in ambiguous behavior:\n"
    for pair in amb:
        text += "\t" + ", ".join("[" + str_signature(s) + "]" for s in pair) + "\n"
    text += "\n\nConsider making the following additions:\n\n"
    text += "\n\n".join(
        [
            "@dispatch(" + str_signature(super_signature(s)) + f")\ndef {name}(...)"
            for s in amb
        ]
    )
    return text
```
- **EN**: Defines the `warning_text` function; this block introduces logic that route operators through dispatch and decomposition helpers.
- **CN**: 定义`warning_text` 函数；该代码块引入了用于通过分发与分解辅助逻辑路由算子的逻辑。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `.conflict`, `.utils`, `.variadic`
- **Standard library / 标准库**: `__future__`, `inspect`, `itertools`, `typing`, `typing_extensions`, `warnings`, `collections.abc`
- **Primary symbols / 核心符号**: `_T`, `__all__`, `MDNotImplementedError`, `ambiguity_warn`, `halt_ordering`, `restart_ordering`, `variadic_signature_matches_iter`, `variadic_signature_matches`, `Dispatcher`, `source`, `MethodDispatcher`, `str_signature`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
