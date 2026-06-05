# utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/unification/multipledispatch/utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```python
from __future__ import annotations

from collections import OrderedDict
from typing import TYPE_CHECKING, TypeVar


if TYPE_CHECKING:
    from collections.abc import Callable, Iterable, Mapping, Sequence

_T = TypeVar("_T")


__all__ = ["raises", "expand_tuples", "reverse_dict", "groupby", "typename"]
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 16-27
```python
def raises(
    err: type[BaseException],
    lamda: Callable[[], object],  # codespell:ignore lamda
) -> bool:
    try:
        lamda()  # codespell:ignore lamda
        return False
    except err:
        return True


def expand_tuples(L: Sequence[type | tuple[type, ...]]) -> list[tuple[type, ...]]:
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果。

### Lines 28-43
```python
    """
    >>> expand_tuples([1, (2, 3)])
    [(1, 2), (1, 3)]
    >>> expand_tuples([1, 2])
    [(1, 2)]
    """
    if not L:
        return [()]
    elif not isinstance(L[0], tuple):
        rest = expand_tuples(L[1:])
        return [(L[0],) + t for t in rest]
    else:
        rest = expand_tuples(L[1:])
        return [(item,) + t for t in rest for item in L[0]]
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 44-55
```python
# Taken from theano/theano/gof/sched.py
# Avoids licensing issues because this was written by Matthew Rocklin
def _toposort(edges: Mapping[_T, Iterable[_T]]) -> list[_T]:
    """Topological sort algorithm by Kahn [1] - O(nodes + vertices)
    inputs:
        edges - a dict of the form {a: {b, c}} where b and c depend on a
    outputs:
        L - an ordered list of nodes that satisfy the dependencies of edges
    >>> _toposort({1: (2, 3), 2: (3,)})
    [1, 2, 3]
    >>> # Closely follows the wikipedia page [2]
    >>> # [1] Kahn, Arthur B. (1962), "Topological sorting of large networks",
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 56-67
```python
    >>> # Communications of the ACM
    >>> # [2] http://en.wikipedia.org/wiki/Toposort#Algorithms
    """
    incoming_edges = reverse_dict(edges)
    incoming_edges = OrderedDict((k, set(val)) for k, val in incoming_edges.items())
    S = OrderedDict.fromkeys(v for v in edges if v not in incoming_edges)
    L = []

    while S:
        n, _ = S.popitem()
        L.append(n)
        for m in edges.get(n, ()):
```
- **EN**: This block continues `_toposort` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `_toposort`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会遍历输入、节点或簿记结构。

### Lines 68-82
```python
            if n not in incoming_edges[m]:
                raise AssertionError(f"Expected {n} in incoming_edges[{m}]")
            incoming_edges[m].remove(n)
            if not incoming_edges[m]:
                S[m] = None
    if any(incoming_edges.get(v, None) for v in edges):
        raise ValueError("Input has cycles")
    return L


def reverse_dict(
    d: Mapping[_T, Iterable[_T]],
) -> OrderedDict[_T, tuple[_T, ...]]:
    """Reverses direction of dependence dict.
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 83-100
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
    result = OrderedDict()  # type: ignore[var-annotated]
    for key in d:
        for val in d[key]:
            # pyrefly: ignore [unsupported-operation]
            result[val] = result.get(val, ()) + (key,)
    return result  # pyrefly: ignore[bad-return]
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 101-116
```python
# Taken from toolz
# Avoids licensing issues because this version was authored by Matthew Rocklin
def groupby(
    func: Callable[[_T], object], seq: Iterable[_T]
) -> OrderedDict[object, list[_T]]:
    """Group a collection by a key function
    >>> names = ["Alice", "Bob", "Charlie", "Dan", "Edith", "Frank"]
    >>> groupby(len, names)  # doctest: +SKIP
    {3: ['Bob', 'Dan'], 5: ['Alice', 'Edith', 'Frank'], 7: ['Charlie']}
    >>> iseven = lambda x: x % 2 == 0
    >>> groupby(iseven, [1, 2, 3, 4, 5, 6, 7, 8])  # doctest: +SKIP
    {False: [1, 3, 5, 7], True: [2, 4, 6, 8]}
    See Also:
        ``countby``
    """
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。

### Lines 117-128
```python
    d = OrderedDict()  # type: ignore[var-annotated]
    for item in seq:
        key = func(item)
        if key not in d:
            d[key] = []
        d[key].append(item)
    return d


def typename(type: type | tuple[type, ...]) -> str:
    """Get the name of `type`.
    Parameters
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 129-140
```python
    ----------
    type : Union[Type, Tuple[Type]]
    Returns
    -------
    str
        The name of `type` or a tuple of the names of the types in `type`.
    Examples
    --------
    >>> typename(int)
    'int'
    >>> typename((int, float))
    '(int, float)'
```
- **EN**: This block continues `typename` and works to manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 该代码块继续实现 `typename`，用于操作 FX 图、跟踪辅助逻辑或变换工具。

### Lines 141-147
```python
    """
    try:
        return type.__name__  # pyrefly: ignore[missing-attribute]
    except AttributeError:
        if len(type) == 1:
            return typename(*type)
        return f"({', '.join(map(typename, type))})"
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Standard library / 标准库**: `__future__`, `collections`, `typing`, `collections.abc`
- **Primary symbols / 核心符号**: `_T`, `__all__`, `raises`, `expand_tuples`, `_toposort`, `reverse_dict`, `groupby`, `typename`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
