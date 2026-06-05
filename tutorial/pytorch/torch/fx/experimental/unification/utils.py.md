# utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/unification/utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
from __future__ import annotations

from typing import TYPE_CHECKING, TypeVar


if TYPE_CHECKING:
    from collections.abc import Callable, Iterable

    from .variable import Var

_T = TypeVar("_T")


__all__ = ["hashable", "transitive_get", "raises", "reverse_dict", "xfail", "freeze"]
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 17-28
```python
def hashable(x: object) -> bool:
    try:
        hash(x)
        return True
    except TypeError:
        return False


def transitive_get(key: object, d: dict[Var, object]) -> object:
    """Transitive dict.get
    >>> d = {1: 2, 2: 3, 3: 4}
    >>> d.get(1)
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果。

### Lines 29-40
```python
    2
    >>> transitive_get(1, d)
    4
    """
    while hashable(key) and key in d:
        key = d[key]
    return key


def raises(
    err: type[BaseException],
    lamda: Callable[[], object],  # codespell:ignore lamda
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 41-52
```python
) -> bool:
    try:
        lamda()  # codespell:ignore lamda
        return False
    except err:
        return True


# Taken from theano/theano/gof/sched.py
# Avoids licensing issues because this was written by Matthew Rocklin
def _toposort(edges: dict[_T, Iterable[_T]]) -> list[_T]:
    """Topological sort algorithm by Kahn [1] - O(nodes + vertices)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 53-69
```python
    inputs:
        edges - a dict of the form {a: {b, c}} where b and c depend on a
    outputs:
        L - an ordered list of nodes that satisfy the dependencies of edges
    >>> # xdoctest: +SKIP
    >>> _toposort({1: (2, 3), 2: (3,)})
    [1, 2, 3]
    Closely follows the wikipedia page [2]
    [1] Kahn, Arthur B. (1962), "Topological sorting of large networks",
    Communications of the ACM
    [2] http://en.wikipedia.org/wiki/Toposort#Algorithms
    """
    incoming_edges = reverse_dict(edges)
    incoming_edges = {k: set(val) for k, val in incoming_edges.items()}
    S = {v for v in edges if v not in incoming_edges}
    L = []
```
- **EN**: This block continues `_toposort` and works to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 该代码块继续实现 `_toposort`，用于构建、遍历或改写图结构及其元数据。

### Lines 70-86
```python
    while S:
        n = S.pop()
        L.append(n)
        for m in edges.get(n, ()):
            if n not in incoming_edges[m]:
                raise AssertionError(f"Expected {n} in incoming_edges[{m}]")
            incoming_edges[m].remove(n)
            if not incoming_edges[m]:
                S.add(m)
    if any(incoming_edges.get(v) for v in edges):
        raise ValueError("Input has cycles")
    return L


def reverse_dict(d: dict[_T, Iterable[_T]]) -> dict[_T, tuple[_T, ...]]:
    """Reverses direction of dependence dict.
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 87-104
```python
    >>> d = {"a": (1, 2), "b": (2, 3), "c": ()}
    >>> reverse_dict(d)  # doctest: +SKIP
    {1: ('a',), 2: ('a', 'b'), 3: ('b',)}

    .. note::
        dict order are not deterministic. As we iterate on the
        input dict, it make the output of this function depend on the
        dict order. So this function output order should be considered
        as undeterministic.
    """
    result = {}  # type: ignore[var-annotated]
    for key in d:
        for val in d[key]:
            # pyrefly: ignore [unsupported-operation]
            result[val] = result.get(val, ()) + (key,)
    return result  # pyrefly: ignore[bad-return]
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 105-116
```python
def xfail(func: Callable[[], object]) -> None:
    try:
        func()
        raise Exception("XFailed test passed")  # pragma:nocover  # noqa: TRY002
    except Exception:
        pass


def freeze(d: object) -> object:
    """Freeze container to hashable form
    >>> freeze(1)
    1
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also validates assumptions before proceeding.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会在继续前校验关键假设。

### Lines 117-128
```python
    >>> freeze([1, 2])
    (1, 2)
    >>> freeze({1: 2})  # doctest: +SKIP
    frozenset([(1, 2)])
    """
    if isinstance(d, dict):
        return frozenset(map(freeze, d.items()))
    if isinstance(d, set):
        return frozenset(map(freeze, d))
    if isinstance(d, (tuple, list)):
        return tuple(map(freeze, d))
    return d
```
- **EN**: This block continues `freeze` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `freeze`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `.variable`
- **Standard library / 标准库**: `__future__`, `typing`, `collections.abc`
- **Primary symbols / 核心符号**: `_T`, `__all__`, `hashable`, `transitive_get`, `raises`, `_toposort`, `reverse_dict`, `xfail`, `freeze`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
