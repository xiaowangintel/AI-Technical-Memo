# jsontree.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/jsontree.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Helper functions to work with nested JSON structures / 该模块围绕 `jsontree` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-33)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Helper functions to work with nested JSON structures."""

from collections.abc import Callable, Iterable
from functools import reduce
from typing import TYPE_CHECKING, Any, TypeAlias, TypeVar, overload

if TYPE_CHECKING:
    import torch

    from vllm.multimodal.inputs import BatchedTensorInputs

_T = TypeVar("_T")
_U = TypeVar("_U")

JSONTree: TypeAlias = (
    dict[str, "JSONTree[_T]"] | list["JSONTree[_T]"] | tuple["JSONTree[_T]", ...] | _T
)
"""A nested JSON structure where the leaves need not be JSON-serializable."""

_JSONTree: TypeAlias = (
    dict[str, "JSONTree[_T]"]
    | list["JSONTree[_T]"]
    # ...
    | tuple[_T, ...]
    | _T
)
"""
Same as `JSONTree` but with additional `Union` members to satisfy overloads.
"""
```
**EN:** Sets up the module with standard-library support such as `collections.abc`, `functools`, `typing`, external packages such as `torch`, vLLM modules such as `vllm.multimodal.inputs`. It prepares the symbols later used by `json_iter_leaves`, `json_map_leaves`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.multimodal.inputs` 等 vLLM 内部依赖。 这些准备工作为后续的 `json_iter_leaves`, `json_map_leaves` 提供上下文。

### json_iter_leaves (lines 36-45)
```python
def json_iter_leaves(value: JSONTree[_T]) -> Iterable[_T]:
    """Iterate through each leaf in a nested JSON structure."""
    if isinstance(value, dict):
        for v in value.values():
            yield from json_iter_leaves(v)
    elif isinstance(value, (list, tuple)):
        for v in value:
            yield from json_iter_leaves(v)
    else:
        yield value
```
**EN:** `json_iter_leaves`: Iterate through each leaf in a nested JSON structure. It mainly works with `value`. Inside the body, it relies on `value.values`, `json_iter_leaves` to complete the main steps.
**CN:** `json_iter_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `value` 等参数。 实现过程中会调用 `value.values`, `json_iter_leaves` 等函数完成关键步骤。

### json_map_leaves (lines 49-52)
```python
def json_map_leaves(
    func: Callable[["torch.Tensor"], "torch.Tensor"],
    value: "BatchedTensorInputs",
) -> "BatchedTensorInputs": ...
```
**EN:** `json_map_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_map_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_map_leaves (lines 56-59)
```python
def json_map_leaves(
    func: Callable[[_T], _U],
    value: _T | dict[str, _T],
) -> _U | dict[str, _U]: ...
```
**EN:** `json_map_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_map_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_map_leaves (lines 63-66)
```python
def json_map_leaves(
    func: Callable[[_T], _U],
    value: _T | list[_T],
) -> _U | list[_U]: ...
```
**EN:** `json_map_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_map_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_map_leaves (lines 70-73)
```python
def json_map_leaves(
    func: Callable[[_T], _U],
    value: _T | tuple[_T, ...],
) -> _U | tuple[_U, ...]: ...
```
**EN:** `json_map_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_map_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_map_leaves (lines 77-80)
```python
def json_map_leaves(
    func: Callable[[_T], _U],
    value: JSONTree[_T],
) -> JSONTree[_U]: ...
```
**EN:** `json_map_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_map_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_map_leaves (lines 83-95)
```python
def json_map_leaves(
    func: Callable[[_T], _U],
    value: Any,
) -> "BatchedTensorInputs" | _JSONTree[_U]:
    """Apply a function to each leaf in a nested JSON structure."""
    if isinstance(value, dict):
        return {k: json_map_leaves(func, v) for k, v in value.items()}  # type: ignore
    elif isinstance(value, list):
        return [json_map_leaves(func, v) for v in value]  # type: ignore
    elif isinstance(value, tuple):
        return tuple(json_map_leaves(func, v) for v in value)
    else:
        return func(value)
```
**EN:** `json_map_leaves`: Apply a function to each leaf in a nested JSON structure. It mainly works with `func`, `value`. Inside the body, it relies on `json_map_leaves`, `value.items`, `func` to complete the main steps.
**CN:** `json_map_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。 实现过程中会调用 `json_map_leaves`, `value.items`, `func` 等函数完成关键步骤。

### json_reduce_leaves (lines 99-103)
```python
def json_reduce_leaves(
    func: Callable[[_T, _T], _T],
    value: _T | dict[str, _T],
    /,
) -> _T: ...
```
**EN:** `json_reduce_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_reduce_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_reduce_leaves (lines 107-111)
```python
def json_reduce_leaves(
    func: Callable[[_T, _T], _T],
    value: _T | list[_T],
    /,
) -> _T: ...
```
**EN:** `json_reduce_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_reduce_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_reduce_leaves (lines 115-119)
```python
def json_reduce_leaves(
    func: Callable[[_T, _T], _T],
    value: _T | tuple[_T, ...],
    /,
) -> _T: ...
```
**EN:** `json_reduce_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_reduce_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_reduce_leaves (lines 123-127)
```python
def json_reduce_leaves(
    func: Callable[[_T, _T], _T],
    value: JSONTree[_T],
    /,
) -> _T: ...
```
**EN:** `json_reduce_leaves` implements helper logic used by this module. It mainly works with `func`, `value`.
**CN:** `json_reduce_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value` 等参数。

### json_reduce_leaves (lines 131-136)
```python
def json_reduce_leaves(
    func: Callable[[_U, _T], _U],
    value: JSONTree[_T],
    initial: _U,
    /,
) -> _U: ...
```
**EN:** `json_reduce_leaves` implements helper logic used by this module. It mainly works with `func`, `value`, `initial`.
**CN:** `json_reduce_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value`, `initial` 等参数。

### json_reduce_leaves (lines 139-153)
```python
def json_reduce_leaves(
    func: Callable[[_T, _T], _T] | Callable[[_U, _T], _U],
    value: _JSONTree[_T],
    initial: _U = ...,  # type: ignore[assignment]
    /,
) -> _T | _U:
    """
    Apply a function of two arguments cumulatively to each leaf in a
    nested JSON structure, from left to right, so as to reduce the
    sequence to a single value.
    """
    if initial is ...:
        return reduce(func, json_iter_leaves(value))  # type: ignore

    return reduce(func, json_iter_leaves(value), initial)  # type: ignore
```
**EN:** `json_reduce_leaves`: Apply a function of two arguments cumulatively to each leaf in a nested JSON structure, from left to right, so as to reduce the sequence to a single value. It mainly works with `func`, `value`, `initial`. Inside the body, it relies on `reduce`, `json_iter_leaves` to complete the main steps.
**CN:** `json_reduce_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `value`, `initial` 等参数。 实现过程中会调用 `reduce`, `json_iter_leaves` 等函数完成关键步骤。

### json_count_leaves (lines 156-158)
```python
def json_count_leaves(value: JSONTree[_T]) -> int:
    """Count the number of leaves in a nested JSON structure."""
    return sum(1 for _ in json_iter_leaves(value))
```
**EN:** `json_count_leaves`: Count the number of leaves in a nested JSON structure. It mainly works with `value`. Inside the body, it relies on `sum`, `json_iter_leaves` to complete the main steps.
**CN:** `json_count_leaves` 负责实现本模块使用的辅助逻辑。 它主要处理 `value` 等参数。 实现过程中会调用 `sum`, `json_iter_leaves` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`json_iter_leaves`**: Key helper or entry point in this file. / **`json_iter_leaves`**：本文件中的关键辅助函数或入口。
- **`json_map_leaves`**: Key helper or entry point in this file. / **`json_map_leaves`**：本文件中的关键辅助函数或入口。
- **`json_map_leaves`**: Key helper or entry point in this file. / **`json_map_leaves`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: collections.abc, functools, typing
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.multimodal.inputs
