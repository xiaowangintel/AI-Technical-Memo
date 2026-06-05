# collection_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/collection_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Contains helpers that are applied to collections / 该模块围绕 `collection_utils` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-18)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
Contains helpers that are applied to collections.

This is similar in concept to the `collections` module.
"""

from collections import defaultdict
from collections.abc import Callable, Generator, Hashable, Iterable, Mapping, Sequence
from typing import Generic, Literal, TypeVar

from typing_extensions import TypeIs, assert_never, overload

T = TypeVar("T")

_K = TypeVar("_K", bound=Hashable)
_V = TypeVar("_V")
```
**EN:** Sets up the module with standard-library support such as `collections`, `collections.abc`, `typing`, external packages such as `typing_extensions`. It prepares the symbols later used by `LazyDict`, `as_list`, `is_list_of`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `LazyDict`, `as_list`, `is_list_of` 提供上下文。

### LazyDict (lines 21-46)
```python
class LazyDict(Mapping[str, _V], Generic[_V]):
    """
    Evaluates dictionary items only when they are accessed.

    Adapted from: https://stackoverflow.com/a/47212782/5082708
    """

    def __init__(self, factory: dict[str, Callable[[], _V]]):
        self._factory = factory
        self._dict: dict[str, _V] = {}

    def __getitem__(self, key: str) -> _V:
        if key not in self._dict:
            if key not in self._factory:
                raise KeyError(key)
            self._dict[key] = self._factory[key]()
        return self._dict[key]

    def __setitem__(self, key: str, value: Callable[[], _V]):
        self._factory[key] = value

    def __iter__(self):
        return iter(self._factory)

    def __len__(self):
        return len(self._factory)
```
**EN:** `LazyDict`: Evaluates dictionary items only when they are accessed. It extends `Mapping`, `Generic`. Key methods include `__init__`.
**CN:** `LazyDict` 是该文件中的核心类，用于封装与 `LazyDict` 相关的状态和行为。 它继承自 `Mapping`, `Generic`。 关键方法包括 `__init__`。

### as_list (lines 49-51)
```python
def as_list(maybe_list: Iterable[T]) -> list[T]:
    """Convert iterable to list, unless it's already a list."""
    return maybe_list if isinstance(maybe_list, list) else list(maybe_list)
```
**EN:** `as_list`: Convert iterable to list, unless it's already a list. It mainly works with `maybe_list`.
**CN:** `as_list` 负责实现本模块使用的辅助逻辑。 它主要处理 `maybe_list` 等参数。

### is_list_of (lines 54-68)
```python
def is_list_of(
    value: object,
    typ: type[T] | tuple[type[T], ...],
    *,
    check: Literal["first", "all"] = "first",
) -> TypeIs[list[T]]:
    if not isinstance(value, list):
        return False

    if check == "first":
        return len(value) == 0 or isinstance(value[0], typ)
    elif check == "all":
        return all(isinstance(v, typ) for v in value)

    assert_never(check)
```
**EN:** `is_list_of` checks a condition and returns a boolean-style result. It mainly works with `value`, `typ`, `check`. Inside the body, it relies on `assert_never`, `all` to complete the main steps.
**CN:** `is_list_of` 负责检查条件并返回布尔结果。 它主要处理 `value`, `typ`, `check` 等参数。 实现过程中会调用 `assert_never`, `all` 等函数完成关键步骤。

### common_prefix (lines 72-72)
```python
def common_prefix(items: Sequence[str]) -> str: ...
```
**EN:** `common_prefix` implements helper logic used by this module. It mainly works with `items`.
**CN:** `common_prefix` 负责实现本模块使用的辅助逻辑。 它主要处理 `items` 等参数。

### common_prefix (lines 76-76)
```python
def common_prefix(items: Sequence[Sequence[T]]) -> Sequence[T]: ...
```
**EN:** `common_prefix` implements helper logic used by this module. It mainly works with `items`.
**CN:** `common_prefix` 负责实现本模块使用的辅助逻辑。 它主要处理 `items` 等参数。

### common_prefix (lines 79-96)
```python
def common_prefix(items: Sequence[Sequence[T] | str]) -> Sequence[T] | str:
    """Find the longest prefix common to all items."""
    if len(items) == 0:
        return []
    if len(items) == 1:
        return items[0]

    shortest = min(items, key=len)
    if not shortest:
        return shortest[:0]

    for match_len in range(1, len(shortest) + 1):
        match = shortest[:match_len]
        for item in items:
            if item[:match_len] != match:
                return shortest[: match_len - 1]

    return shortest
```
**EN:** `common_prefix`: Find the longest prefix common to all items. It mainly works with `items`. Inside the body, it relies on `min` to complete the main steps.
**CN:** `common_prefix` 负责实现本模块使用的辅助逻辑。 它主要处理 `items` 等参数。 实现过程中会调用 `min` 等函数完成关键步骤。

### chunk_list (lines 99-102)
```python
def chunk_list(lst: list[T], chunk_size: int) -> Generator[list[T]]:
    """Yield successive chunk_size chunks from lst."""
    for i in range(0, len(lst), chunk_size):
        yield lst[i : i + chunk_size]
```
**EN:** `chunk_list`: Yield successive chunk_size chunks from lst. It mainly works with `lst`, `chunk_size`.
**CN:** `chunk_list` 负责实现本模块使用的辅助逻辑。 它主要处理 `lst`, `chunk_size` 等参数。

### flatten_2d_lists (lines 105-107)
```python
def flatten_2d_lists(lists: Iterable[Iterable[T]]) -> list[T]:
    """Flatten a list of lists to a single list."""
    return [item for sublist in lists for item in sublist]
```
**EN:** `flatten_2d_lists`: Flatten a list of lists to a single list. It mainly works with `lists`.
**CN:** `flatten_2d_lists` 负责实现本模块使用的辅助逻辑。 它主要处理 `lists` 等参数。

### full_groupby (lines 110-120)
```python
def full_groupby(values: Iterable[_V], *, key: Callable[[_V], _K]):
    """
    Unlike [`itertools.groupby`][], groups are not broken by
    non-contiguous data.
    """
    groups = defaultdict[_K, list[_V]](list)

    for value in values:
        groups[key(value)].append(value)

    return groups.items()
```
**EN:** `full_groupby`: Unlike [`itertools.groupby`][], groups are not broken by non-contiguous data. It mainly works with `values`, `key`. Inside the body, it relies on `defaultdict`, `groups.items`, `groups.append` to complete the main steps.
**CN:** `full_groupby` 负责实现本模块使用的辅助逻辑。 它主要处理 `values`, `key` 等参数。 实现过程中会调用 `defaultdict`, `groups.items`, `groups.append` 等函数完成关键步骤。

### swap_dict_values (lines 123-134)
```python
def swap_dict_values(obj: dict[_K, _V], key1: _K, key2: _K) -> None:
    """Swap values between two keys."""
    v1 = obj.get(key1)
    v2 = obj.get(key2)
    if v1 is not None:
        obj[key2] = v1
    else:
        obj.pop(key2, None)
    if v2 is not None:
        obj[key1] = v2
    else:
        obj.pop(key1, None)
```
**EN:** `swap_dict_values`: Swap values between two keys. It mainly works with `obj`, `key1`, `key2`. Inside the body, it relies on `obj.get`, `obj.pop` to complete the main steps.
**CN:** `swap_dict_values` 负责实现本模块使用的辅助逻辑。 它主要处理 `obj`, `key1`, `key2` 等参数。 实现过程中会调用 `obj.get`, `obj.pop` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`LazyDict`**: Core class that organizes module behavior. / **`LazyDict`**：组织模块行为的核心类。
- **`as_list`**: Key helper or entry point in this file. / **`as_list`**：本文件中的关键辅助函数或入口。
- **`is_list_of`**: Key helper or entry point in this file. / **`is_list_of`**：本文件中的关键辅助函数或入口。
- **`common_prefix`**: Key helper or entry point in this file. / **`common_prefix`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: collections, collections.abc, typing
- **Third-party / 第三方**: typing_extensions
- **Internal vLLM / vLLM 内部依赖**: None / 无
