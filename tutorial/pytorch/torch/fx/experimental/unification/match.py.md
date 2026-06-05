# match.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/unification/match.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```python
from __future__ import annotations

from typing import TYPE_CHECKING

from .core import reify, unify  # type: ignore[attr-defined]


if TYPE_CHECKING:
    from collections.abc import Callable, Iterable

    from .variable import Var

from .unification_tools import first, groupby  # type: ignore[import]
from .utils import _toposort, freeze
from .variable import isvar
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 18-31
```python
class Dispatcher:
    def __init__(self, name: str) -> None:
        self.name = name
        self.funcs: dict[object, Callable[..., object]] = {}
        self.ordering: list[object] = []

    def add(self, signature: tuple[object, ...], func: Callable[..., object]) -> None:
        self.funcs[freeze(signature)] = func
        self.ordering = ordering(self.funcs)

    def __call__(self, *args: object, **kwargs: object) -> object:
        func, _ = self.resolve(args)
        return func(*args, **kwargs)
```
- **EN**: Declares `Dispatcher`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `Dispatcher`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 32-49
```python
    def resolve(
        self, args: tuple[object, ...]
    ) -> tuple[Callable[..., object], dict[Var, object]]:
        n = len(args)
        for signature in self.ordering:
            if len(signature) != n:  # pyrefly: ignore[bad-argument-type]
                continue
            s = unify(freeze(args), signature)
            if s is not False:
                result = self.funcs[signature]
                return result, s  # pyrefly: ignore[bad-return]
        raise NotImplementedError(
            "No match found. \nKnown matches: "
            + str(self.ordering)
            + "\nInput: "
            + str(args)
        )
```
- **EN**: Declares `Dispatcher`; this class packages state and methods that validate invariants and surface meaningful failures.
- **CN**: 声明 `Dispatcher`；该类封装了状态与方法，用于校验不变量并给出有意义的失败信息。

### Lines 50-61
```python
    def register(self, *signature: object) -> Callable[..., object]:
        def _(func: Callable[..., object]) -> Dispatcher:
            self.add(signature, func)
            return self

        return _


class VarDispatcher(Dispatcher):
    """A dispatcher that calls functions with variable names
    >>> # xdoctest: +SKIP
    >>> d = VarDispatcher("d")
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。 同时它还会计算并返回中间值或结果。

### Lines 62-79
```python
    >>> x = var("x")
    >>> @d.register("inc", x)
    ... def f(x):
    ...     return x + 1
    >>> @d.register("double", x)
    ... def f(x):
    ...     return x * 2
    >>> d("inc", 10)
    11
    >>> d("double", 10)
    20
    """

    def __call__(self, *args: object, **kwargs: object) -> object:
        func, s = self.resolve(args)
        d = {k.token: v for k, v in s.items()}  # pyrefly: ignore[missing-attribute]
        return func(**d)
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

### Lines 80-94
```python

global_namespace: dict[str, Dispatcher] = {}


def match(*signature: object, **kwargs: object) -> Callable[..., object]:
    namespace: dict[str, Dispatcher] = kwargs.get(  # type: ignore[assignment]
        "namespace", global_namespace
    )
    dispatcher_cls: type[Dispatcher] = kwargs.get(  # type: ignore[assignment]
        "Dispatcher", Dispatcher
    )

    def _(func: Callable[..., object]) -> Dispatcher:
        name = func.__name__
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。

### Lines 95-106
```python
        if name not in namespace:
            namespace[name] = dispatcher_cls(name)
        d = namespace[name]

        d.add(signature, func)

        return d

    return _


def supercedes(a: object, b: object) -> bool:
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 107-124
```python
    """``a`` is a more specific match than ``b``"""
    if isvar(b) and not isvar(a):
        return True
    s = unify(a, b)
    if s is False:
        return False
    s = {
        k: v
        for k, v in s.items()  # pyrefly: ignore[missing-attribute]
        if not isvar(k) or not isvar(v)
    }
    if reify(a, s) == a:
        return True
    if reify(b, s) == b:
        return False
    return False
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 125-137
```python
# Taken from multipledispatch
def edge(a: object, b: object, tie_breaker: Callable[[object], int] = hash) -> bool:
    """A should be checked before B
    Tie broken by tie_breaker, defaults to ``hash``
    """
    if supercedes(a, b):
        if supercedes(b, a):
            return tie_breaker(a) > tie_breaker(b)
        else:
            return True
    return False
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 138-149
```python
# Taken from multipledispatch
def ordering(signatures: Iterable[object]) -> list[object]:
    """A sane ordering of signatures to check, first to last
    Topological sort of edges as given by ``edge`` and ``supercedes``
    """
    signatures = list(map(tuple, signatures))  # pyrefly: ignore[bad-argument-type]
    edges = [(a, b) for a in signatures for b in signatures if edge(a, b)]
    edges = groupby(first, edges)
    for s in signatures:
        if s not in edges:
            edges[s] = []
    edges = {k: [b for a, b in v] for k, v in edges.items()}  # type: ignore[attr-defined, assignment]
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 150-150
```python
    return _toposort(edges)  # pyrefly: ignore[bad-argument-type]
```
- **EN**: This block continues `ordering` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `ordering`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `.core`, `.unification_tools`, `.utils`, `.variable`
- **Standard library / 标准库**: `__future__`, `typing`, `collections.abc`
- **Primary symbols / 核心符号**: `Dispatcher`, `VarDispatcher`, `match`, `supercedes`, `edge`, `ordering`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
