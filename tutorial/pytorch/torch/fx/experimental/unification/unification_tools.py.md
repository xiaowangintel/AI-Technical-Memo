# unification_tools.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/unification/unification_tools.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```python
from __future__ import annotations

import collections
import operator
from collections.abc import Mapping
from functools import reduce
from typing import TYPE_CHECKING, TypeVar


if TYPE_CHECKING:
    from collections.abc import Callable, Iterable

_K = TypeVar("_K")
_V = TypeVar("_V")
_K2 = TypeVar("_K2")
_V2 = TypeVar("_V2")
_T = TypeVar("_T")
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 20-45
```python
__all__ = [
    "merge",
    "merge_with",
    "valmap",
    "keymap",
    "itemmap",
    "valfilter",
    "keyfilter",
    "itemfilter",
    "assoc",
    "dissoc",
    "assoc_in",
    "update_in",
    "get_in",
]


def _get_factory(f: Callable[..., object], kwargs: dict[str, object]) -> type:
    factory: type = kwargs.pop("factory", dict)  # type: ignore[assignment]
    if kwargs:
        raise TypeError(
            f"{f.__name__}() got an unexpected keyword argument '{kwargs.popitem()[0]}'"
        )
    return factory
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 46-69
```python
def merge(*dicts: Mapping[object, object], **kwargs: object) -> object:
    """Merge a collection of dictionaries

    >>> merge({1: "one"}, {2: "two"})
    {1: 'one', 2: 'two'}

    Later dictionaries have precedence

    >>> merge({1: 2, 3: 4}, {3: 3, 4: 4})
    {1: 2, 3: 3, 4: 4}

    See Also:
        merge_with
    """
    if len(dicts) == 1 and not isinstance(dicts[0], Mapping):
        dicts = dicts[0]
    factory = _get_factory(merge, kwargs)

    rv = factory()
    for d in dicts:
        rv.update(d)
    return rv
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 70-90
```python
def merge_with(
    func: Callable[..., object], *dicts: Mapping[object, object], **kwargs: object
) -> object:
    """Merge dictionaries and apply function to combined values

    A key may occur in more than one dict, and all values mapped from the key
    will be passed to the function as a list, such as func([val1, val2, ...]).

    >>> merge_with(sum, {1: 1, 2: 2}, {1: 10, 2: 20})
    {1: 11, 2: 22}

    >>> merge_with(first, {1: 1, 2: 2}, {2: 20, 3: 30})  # doctest: +SKIP
    {1: 1, 2: 2, 3: 30}

    See Also:
        merge
    """
    if len(dicts) == 1 and not isinstance(dicts[0], Mapping):
        dicts = dicts[0]
    factory = _get_factory(merge_with, kwargs)
```
- **EN**: Defines the `merge_with` function; this block introduces logic that manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 定义`merge_with` 函数；该代码块引入了用于操作 FX 图、跟踪辅助逻辑或变换工具的逻辑。

### Lines 91-117
```python
    result = factory()
    for d in dicts:
        for k, v in d.items():
            if k not in result:
                result[k] = [v]
            else:
                result[k].append(v)
    return valmap(func, result, factory)


def valmap(
    func: Callable[[_V], _V2], d: Mapping[_K, _V], factory: type = dict
) -> dict[_K, _V2]:
    """Apply function to values of dictionary

    >>> bills = {"Alice": [20, 15, 30], "Bob": [10, 35]}
    >>> valmap(sum, bills)  # doctest: +SKIP
    {'Alice': 65, 'Bob': 45}

    See Also:
        keymap
        itemmap
    """
    rv = factory()
    rv.update(zip(d.keys(), map(func, d.values())))
    return rv
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 118-141
```python

def keymap(
    func: Callable[[_K], _K2], d: Mapping[_K, _V], factory: type = dict
) -> dict[_K2, _V]:
    """Apply function to keys of dictionary

    >>> bills = {"Alice": [20, 15, 30], "Bob": [10, 35]}
    >>> keymap(str.lower, bills)  # doctest: +SKIP
    {'alice': [20, 15, 30], 'bob': [10, 35]}

    See Also:
        valmap
        itemmap
    """
    rv = factory()
    rv.update(zip(map(func, d.keys()), d.values()))
    return rv


def itemmap(
    func: Callable[[tuple[_K, _V]], object], d: Mapping[_K, _V], factory: type = dict
) -> dict[object, object]:
    """Apply function to items of dictionary
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果。

### Lines 142-164
```python
    >>> accountids = {"Alice": 10, "Bob": 20}
    >>> itemmap(reversed, accountids)  # doctest: +SKIP
    {10: "Alice", 20: "Bob"}

    See Also:
        keymap
        valmap
    """
    rv = factory()
    rv.update(map(func, d.items()))
    return rv


def valfilter(
    predicate: Callable[[_V], bool], d: Mapping[_K, _V], factory: type = dict
) -> dict[_K, _V]:
    """Filter items in dictionary by value

    >>> iseven = lambda x: x % 2 == 0
    >>> d = {1: 2, 2: 3, 3: 4, 4: 5}
    >>> valfilter(iseven, d)
    {1: 2, 3: 4}
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

### Lines 165-186
```python
    See Also:
        keyfilter
        itemfilter
        valmap
    """
    rv = factory()
    for k, v in d.items():
        if predicate(v):
            rv[k] = v
    return rv


def keyfilter(
    predicate: Callable[[_K], bool], d: Mapping[_K, _V], factory: type = dict
) -> dict[_K, _V]:
    """Filter items in dictionary by key

    >>> iseven = lambda x: x % 2 == 0
    >>> d = {1: 2, 2: 3, 3: 4, 4: 5}
    >>> keyfilter(iseven, d)
    {2: 3, 4: 5}
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 187-211
```python
    See Also:
        valfilter
        itemfilter
        keymap
    """
    rv = factory()
    for k, v in d.items():
        if predicate(k):
            rv[k] = v
    return rv


def itemfilter(
    predicate: Callable[[tuple[_K, _V]], bool], d: Mapping[_K, _V], factory: type = dict
) -> dict[_K, _V]:
    """Filter items in dictionary by item

    >>> def isvalid(item):
    ...     k, v = item
    ...     return k % 2 == 0 and v < 4

    >>> d = {1: 2, 2: 3, 3: 4, 4: 5}
    >>> itemfilter(isvalid, d)
    {2: 3}
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 212-231
```python
    See Also:
        keyfilter
        valfilter
        itemmap
    """
    rv = factory()
    for item in d.items():
        if predicate(item):
            k, v = item
            rv[k] = v
    return rv


def assoc(
    d: Mapping[_K, _V], key: object, value: object, factory: type = dict
) -> dict[_K, _V]:
    """Return a new dict with new key value pair

    New dict has d[key] set to value. Does not modify the initial dictionary.
```
- **EN**: This module-level block helps initialize learnable tensors and related state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于初始化可学习张量及相关状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 232-258
```python
    >>> assoc({"x": 1}, "x", 2)
    {'x': 2}
    >>> assoc({"x": 1}, "y", 3)  # doctest: +SKIP
    {'x': 1, 'y': 3}
    """
    d2 = factory()
    d2.update(d)
    d2[key] = value
    return d2


def dissoc(d: Mapping[object, object], *keys: object, **kwargs: object) -> object:
    """Return a new dict with the given key(s) removed.

    New dict has d[key] deleted for each supplied key.
    Does not modify the initial dictionary.

    >>> dissoc({"x": 1, "y": 2}, "y")
    {'x': 1}
    >>> dissoc({"x": 1, "y": 2}, "y", "x")
    {}
    >>> dissoc({"x": 1}, "y")  # Ignores missing keys
    {'x': 1}
    """
    factory = _get_factory(dissoc, kwargs)
    d2 = factory()
```
- **EN**: This module-level block helps initialize learnable tensors and related state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于初始化可学习张量及相关状态。 同时它还会计算并返回中间值或结果。

### Lines 259-279
```python
    if len(keys) < len(d) * 0.6:
        d2.update(d)
        for key in keys:
            if key in d2:
                del d2[key]
    else:
        remaining = set(d)
        remaining.difference_update(keys)
        for k in remaining:
            d2[k] = d[k]
    return d2


def assoc_in(
    d: Mapping[object, object],
    keys: Iterable[object],
    value: object,
    factory: type = dict,
) -> object:
    """Return a new dict with new, potentially nested, key value pair
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 280-306
```python
    >>> purchase = {
    ...     "name": "Alice",
    ...     "order": {"items": ["Apple", "Orange"], "costs": [0.50, 1.25]},
    ...     "credit card": "5555-1234-1234-1234",
    ... }
    >>> assoc_in(purchase, ["order", "costs"], [0.25, 1.00])  # doctest: +SKIP
    {'credit card': '5555-1234-1234-1234',
     'name': 'Alice',
     'order': {'costs': [0.25, 1.00], 'items': ['Apple', 'Orange']}}
    """
    return update_in(d, keys, lambda x: value, value, factory)


def update_in(
    d: Mapping[object, object],
    keys: Iterable[object],
    func: Callable[..., object],
    default: object = None,
    factory: type = dict,
) -> object:
    """Update value in a (potentially) nested dictionary

    inputs:
    d - dictionary on which to operate
    keys - list or tuple giving the location of the value to be changed in d
    func - function to operate on that value
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

### Lines 307-327
```python
    If keys == [k0,..,kX] and d[k0]..[kX] == v, update_in returns a copy of the
    original dictionary with v replaced by func(v), but does not mutate the
    original dictionary.

    If k0 is not a key in d, update_in creates nested dictionaries to the depth
    specified by the keys, with the innermost value set to func(default).

    >>> inc = lambda x: x + 1
    >>> update_in({"a": 0}, ["a"], inc)
    {'a': 1}

    >>> transaction = {
    ...     "name": "Alice",
    ...     "purchase": {"items": ["Apple", "Orange"], "costs": [0.50, 1.25]},
    ...     "credit card": "5555-1234-1234-1234",
    ... }
    >>> update_in(transaction, ["purchase", "costs"], sum)  # doctest: +SKIP
    {'credit card': '5555-1234-1234-1234',
     'name': 'Alice',
     'purchase': {'costs': 1.75, 'items': ['Apple', 'Orange']}}
```
- **EN**: This block continues `update_in` and works to manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 该代码块继续实现 `update_in`，用于操作 FX 图、跟踪辅助逻辑或变换工具。

### Lines 328-350
```python
    >>> # updating a value when k0 is not in d
    >>> update_in({}, [1, 2, 3], str, default="bar")
    {1: {2: {3: 'bar'}}}
    >>> update_in({1: "foo"}, [2, 3, 4], inc, 0)
    {1: 'foo', 2: {3: {4: 1}}}
    """
    ks = iter(keys)
    k = next(ks)

    rv = inner = factory()
    rv.update(d)

    for key in ks:
        if k in d:
            d = d[k]  # pyrefly: ignore[bad-assignment]
            dtemp = factory()
            dtemp.update(d)
        else:
            d = dtemp = factory()

        inner[k] = inner = dtemp
        k = key
```
- **EN**: This block continues `update_in` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `update_in`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 351-371
```python
    if k in d:
        inner[k] = func(d[k])
    else:
        inner[k] = func(default)
    return rv


def get_in(
    keys: Iterable[object],
    coll: object,
    default: object = None,
    no_default: bool = False,
) -> object:
    """Returns coll[i0][i1]...[iX] where [i0, i1, ..., iX]==keys.

    If coll[i0][i1]...[iX] cannot be found, returns ``default``, unless
    ``no_default`` is specified, then it raises KeyError or IndexError.

    ``get_in`` is a generalization of ``operator.getitem`` for nested data
    structures such as dictionaries and lists.
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 372-390
```python
    >>> transaction = {
    ...     "name": "Alice",
    ...     "purchase": {"items": ["Apple", "Orange"], "costs": [0.50, 1.25]},
    ...     "credit card": "5555-1234-1234-1234",
    ... }
    >>> get_in(["purchase", "items", 0], transaction)
    'Apple'
    >>> get_in(["name"], transaction)
    'Alice'
    >>> get_in(["purchase", "total"], transaction)
    >>> get_in(["purchase", "items", "apple"], transaction)
    >>> get_in(["purchase", "items", 10], transaction)
    >>> get_in(["purchase", "total"], transaction, 0)
    0
    >>> get_in(["y"], {}, no_default=True)
    Traceback (most recent call last):
        ...
    KeyError: 'y'
```
- **EN**: This block continues `get_in` and works to trace Python execution into an intermediate graph representation.
- **CN**: 该代码块继续实现 `get_in`，用于将 Python 执行过程跟踪为中间图表示。

### Lines 391-408
```python
    See Also:
        itertoolz.get
        operator.getitem
    """
    try:
        return reduce(
            operator.getitem,
            keys,  # pyrefly: ignore[bad-argument-type]
            coll,  # pyrefly: ignore[bad-argument-type]
        )
    except (KeyError, IndexError, TypeError):
        if no_default:
            raise
        return default


def getter(index: object) -> Callable[..., object]:
    if isinstance(index, list):
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 409-432
```python
        if len(index) == 1:
            index = index[0]
            return lambda x: (x[index],)
        elif index:
            return operator.itemgetter(*index)
        else:
            return lambda x: ()
    else:
        return operator.itemgetter(index)


def groupby(key: object, seq: Iterable[object]) -> dict[object, list[object]]:
    """Group a collection by a key function

    >>> names = ["Alice", "Bob", "Charlie", "Dan", "Edith", "Frank"]
    >>> groupby(len, names)  # doctest: +SKIP
    {3: ['Bob', 'Dan'], 5: ['Alice', 'Edith', 'Frank'], 7: ['Charlie']}

    >>> iseven = lambda x: x % 2 == 0
    >>> groupby(iseven, [1, 2, 3, 4, 5, 6, 7, 8])  # doctest: +SKIP
    {False: [1, 3, 5, 7], True: [2, 4, 6, 8]}

    Non-callable keys imply grouping on a member.
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 433-459
```python
    >>> groupby(
    ...     "gender",
    ...     [
    ...         {"name": "Alice", "gender": "F"},
    ...         {"name": "Bob", "gender": "M"},
    ...         {"name": "Charlie", "gender": "M"},
    ...     ],
    ... )  # doctest:+SKIP
    {'F': [{'gender': 'F', 'name': 'Alice'}],
     'M': [{'gender': 'M', 'name': 'Bob'},
           {'gender': 'M', 'name': 'Charlie'}]}

    Not to be confused with ``itertools.groupby``

    See Also:
        countby
    """
    if not callable(key):
        key = getter(key)
    d = collections.defaultdict(lambda: [].append)  # type: ignore[var-annotated]
    for item in seq:
        d[key(item)](item)
    rv: dict[object, list[object]] = {}
    for k, v in d.items():
        rv[k] = v.__self__  # type: ignore[attr-defined]
    return rv
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 460-467
```python

def first(seq: Iterable[_T]) -> _T:
    """The first element in a sequence

    >>> first("ABC")
    'A'
    """
    return next(iter(seq))
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
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

- **Standard library / 标准库**: `__future__`, `collections`, `operator`, `collections.abc`, `functools`, `typing`
- **Primary symbols / 核心符号**: `_K`, `_V`, `_K2`, `_V2`, `_T`, `__all__`, `_get_factory`, `merge`, `merge_with`, `valmap`, `keymap`, `itemmap`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
