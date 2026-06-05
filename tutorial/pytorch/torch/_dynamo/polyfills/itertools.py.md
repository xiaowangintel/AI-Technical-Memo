# itertools.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/polyfills/itertools.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: Python polyfills for itertools
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
"""
Python polyfills for itertools
"""

from __future__ import annotations

import itertools
import operator
from collections.abc import Callable
from typing import overload, TYPE_CHECKING, TypeAlias, TypeVar

from ..decorators import substitute_in_graph


if TYPE_CHECKING:
    from collections.abc import Iterable, Iterator
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 19-42
```python
__all__ = [
    "accumulate",
    "chain",
    "chain_from_iterable",
    "compress",
    "cycle",
    "dropwhile",
    "filterfalse",
    "islice",
    "pairwise",
    "starmap",
    "takewhile",
    "tee",
    "zip_longest",
]


_T = TypeVar("_T")
_U = TypeVar("_U")
_Predicate: TypeAlias = Callable[[_T], object]
_T1 = TypeVar("_T1")
_T2 = TypeVar("_T2")
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 43-62
```python
# Reference: https://docs.python.org/3/library/itertools.html#itertools.chain
@substitute_in_graph(itertools.chain, is_embedded_type=True)  # type: ignore[arg-type]
def chain(*iterables: Iterable[_T]) -> Iterator[_T]:
    for iterable in iterables:
        yield from iterable


# Reference: https://docs.python.org/3/library/itertools.html#itertools.accumulate
@substitute_in_graph(itertools.accumulate, is_embedded_type=True)  # type: ignore[arg-type]
def accumulate(
    iterable: Iterable[_T],
    func: Callable[[_T, _T], _T] | None = None,
    *,
    initial: _T | None = None,
) -> Iterator[_T]:
    # call iter outside of the generator to match cypthon behavior
    iterator = iter(iterable)
    if func is None:
        func = operator.add
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 63-89
```python
    def _accumulate(iterator: Iterator[_T]) -> Iterator[_T]:
        total = initial
        if total is None:
            try:
                total = next(iterator)
            except StopIteration:
                return

        yield total
        for element in iterator:
            total = func(total, element)
            yield total

    return _accumulate(iterator)


@substitute_in_graph(itertools.chain.from_iterable)  # type: ignore[arg-type]
def chain_from_iterable(iterable: Iterable[Iterable[_T]], /) -> Iterator[_T]:
    # previous version of this code was:
    #   return itertools.chain(*iterable)
    # If iterable is an infinite generator, this will lead to infinite recursion
    for it in iterable:
        yield from it


chain.from_iterable = chain_from_iterable  # type: ignore[attr-defined]
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 90-115
```python

# Reference: https://docs.python.org/3/library/itertools.html#itertools.compress
@substitute_in_graph(itertools.compress, is_embedded_type=True)  # type: ignore[arg-type]
def compress(data: Iterable[_T], selectors: Iterable[_U], /) -> Iterator[_T]:
    return (datum for datum, selector in zip(data, selectors) if selector)


# Reference: https://docs.python.org/3/library/itertools.html#itertools.cycle
@substitute_in_graph(itertools.cycle, is_embedded_type=True)  # type: ignore[arg-type]
def cycle(iterable: Iterable[_T]) -> Iterator[_T]:
    iterator = iter(iterable)

    def _cycle(iterator: Iterator[_T]) -> Iterator[_T]:
        # pyrefly: ignore [implicit-any]
        saved = []
        for element in iterable:
            yield element
            saved.append(element)

        while saved:
            for element in saved:
                yield element

    return _cycle(iterator)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 116-138
```python
# Reference: https://docs.python.org/3/library/itertools.html#itertools.dropwhile
@substitute_in_graph(itertools.dropwhile, is_embedded_type=True)  # type: ignore[arg-type]
def dropwhile(predicate: _Predicate[_T], iterable: Iterable[_T], /) -> Iterator[_T]:
    # dropwhile(lambda x: x < 5, [1, 4, 6, 3, 8]) -> 6 3 8
    if not callable(predicate):
        raise TypeError(f"'{type(predicate).__name__}' object is not callable")

    iterator = iter(iterable)
    for x in iterator:
        if not predicate(x):
            yield x
            break

    yield from iterator


# Reference: https://docs.python.org/3/library/itertools.html#itertools.takewhile
@substitute_in_graph(itertools.takewhile, is_embedded_type=True)  # type: ignore[arg-type]
def takewhile(predicate: _Predicate[_T], iterable: Iterable[_T], /) -> Iterator[_T]:
    # takewhile(lambda x: x<5, [1,4,6,3,8]) → 1 4
    if not callable(predicate):
        raise TypeError(f"'{type(predicate).__name__}' object is not callable")
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 139-160
```python
    for x in iterable:
        if not predicate(x):
            break
        yield x


@overload
def starmap(
    function: Callable[[], _U],
    iterable: Iterable[tuple[()]],
    /,
) -> itertools.starmap[_U]: ...


@overload
def starmap(
    function: Callable[[_T], _U],
    iterable: Iterable[tuple[_T]],
    /,
) -> itertools.starmap[_U]: ...
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 161-187
```python
@overload
def starmap(
    function: Callable[[_T, _T1], _U],
    iterable: Iterable[tuple[_T, _T1]],
    /,
) -> itertools.starmap[_U]: ...


@overload
def starmap(
    function: Callable[[_T, _T1, _T2], _U],
    iterable: Iterable[tuple[_T, _T1, _T2]],
    /,
) -> itertools.starmap[_U]: ...


# Reference: https://docs.python.org/3/library/itertools.html#itertools.starmap
@substitute_in_graph(itertools.starmap, is_embedded_type=True)  # type: ignore[arg-type]
# pyrefly: ignore [implicit-any]
def starmap(function: Callable[..., _T], iterable: Iterable, /) -> Iterable[_T]:
    # starmap(pow, [(2,5), (3,2), (10,3)]) → 32 9 1000
    if not callable(function):
        raise TypeError(f"'{type(function).__name__}' object is not callable")

    for args in iterable:
        yield function(*args)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 188-209
```python

@substitute_in_graph(itertools.filterfalse, is_embedded_type=True)  # type: ignore[arg-type]
def filterfalse(function: _Predicate[_T], iterable: Iterable[_T], /) -> Iterator[_T]:
    it = iter(iterable)
    if function is None:
        return filter(operator.not_, it)
    else:
        return filter(lambda x: not function(x), it)


# Reference: https://docs.python.org/3/library/itertools.html#itertools.islice
@substitute_in_graph(itertools.islice, is_embedded_type=True)  # type: ignore[arg-type]
def islice(iterable: Iterable[_T], /, *args: int | None) -> Iterator[_T]:
    s = slice(*args)
    start = 0 if s.start is None else s.start
    stop = s.stop
    step = 1 if s.step is None else s.step
    if start < 0 or (stop is not None and stop < 0) or step <= 0:
        raise ValueError(
            "Indices for islice() must be None or an integer: 0 <= x <= sys.maxsize.",
        )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 210-228
```python
    if stop is None:
        # TODO: use indices = itertools.count() and merge implementation with the else branch
        #       when we support infinite iterators
        next_i = start
        for i, element in enumerate(iterable):
            if i == next_i:
                yield element
                next_i += step
    else:
        indices = range(max(start, stop))
        next_i = start
        for i, element in zip(indices, iterable):
            if i == next_i:
                yield element
                next_i += step


# Reference: https://docs.python.org/3/library/itertools.html#itertools.pairwise
@substitute_in_graph(itertools.pairwise, is_embedded_type=True)  # type: ignore[arg-type]
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 229-246
```python
def pairwise(iterable: Iterable[_T], /) -> Iterator[tuple[_T, _T]]:
    a = None
    first = True
    for b in iterable:
        if first:
            first = False
        else:
            yield a, b  # type: ignore[misc]
        a = b


# Reference: https://docs.python.org/3/library/itertools.html#itertools.tee
@substitute_in_graph(itertools.tee)
def tee(iterable: Iterable[_T], n: int = 2, /) -> tuple[Iterator[_T], ...]:
    iterator = iter(iterable)
    shared_link = [None, None]

    def _tee(link) -> Iterator[_T]:  # type: ignore[no-untyped-def]
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 247-269
```python
        try:
            while True:
                if link[1] is None:
                    link[0] = next(iterator)
                    link[1] = [None, None]
                value, link = link
                yield value
        except StopIteration:
            return

    return tuple(_tee(shared_link) for _ in range(n))


@overload
# pyrefly: ignore [inconsistent-overload]
def zip_longest(
    iter1: Iterable[_T1],
    /,
    *,
    fillvalue: _U = ...,
) -> Iterator[tuple[_T1]]: ...
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 270-289
```python
@overload
# pyrefly: ignore [inconsistent-overload]
def zip_longest(
    iter1: Iterable[_T1],
    iter2: Iterable[_T2],
    /,
) -> Iterator[tuple[_T1 | None, _T2 | None]]: ...


@overload
# pyrefly: ignore [inconsistent-overload]
def zip_longest(
    iter1: Iterable[_T1],
    iter2: Iterable[_T2],
    /,
    *,
    fillvalue: _U = ...,
) -> Iterator[tuple[_T1 | _U, _T2 | _U]]: ...
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 290-312
```python
@overload
# pyrefly: ignore [inconsistent-overload]
def zip_longest(
    iter1: Iterable[_T],
    iter2: Iterable[_T],
    iter3: Iterable[_T],
    /,
    *iterables: Iterable[_T],
) -> Iterator[tuple[_T | None, ...]]: ...


@overload
# pyrefly: ignore [inconsistent-overload]
def zip_longest(
    iter1: Iterable[_T],
    iter2: Iterable[_T],
    iter3: Iterable[_T],
    /,
    *iterables: Iterable[_T],
    fillvalue: _U = ...,
) -> Iterator[tuple[_T | _U, ...]]: ...
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 313-330
```python
# Reference: https://docs.python.org/3/library/itertools.html#itertools.zip_longest
@substitute_in_graph(itertools.zip_longest, is_embedded_type=True)  # type: ignore[arg-type,misc]
def zip_longest(
    *iterables: Iterable[_T],
    fillvalue: _U = None,  # type: ignore[assignment]
) -> Iterator[tuple[_T | _U, ...]]:
    # zip_longest('ABCD', 'xy', fillvalue='-') -> Ax By C- D-

    iterators = list(map(iter, iterables))
    num_active = len(iterators)
    if not num_active:
        return

    while True:
        values = []
        for i, iterator in enumerate(iterators):
            try:
                value = next(iterator)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 331-338
```python
            except StopIteration:
                num_active -= 1
                if not num_active:
                    return
                iterators[i] = itertools.repeat(fillvalue)  # type: ignore[arg-type]
                value = fillvalue  # type: ignore[assignment]
            values.append(value)
        yield tuple(values)
```
- **EN**: This block continues `zip_longest` and works to capture Python execution for torch.compile and maintain compiler state. It also streams values incrementally; branches on runtime conditions.
- **CN**: 该代码块继续实现 `zip_longest`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会以增量方式产生值；根据运行时条件分支处理。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `..decorators`
- **Standard library / 标准库**: `__future__`, `itertools`, `operator`, `collections.abc`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `_T`, `_U`, `_T1`, `_T2`, `chain`, `accumulate`, `chain_from_iterable`, `compress`, `cycle`, `dropwhile`, `takewhile`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
