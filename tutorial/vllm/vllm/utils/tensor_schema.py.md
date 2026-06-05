# tensor_schema.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/tensor_schema.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-10)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from types import UnionType
from typing import Annotated, Any, Union, get_args, get_origin, get_type_hints

import torch

from vllm.logger import init_logger

logger = init_logger(__name__)
```
**EN:** Sets up the module with standard-library support such as `types`, `typing`, external packages such as `torch`, vLLM modules such as `vllm.logger`. It prepares the symbols later used by `TensorShape`, `TensorSchema`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `TensorShape`, `TensorSchema` 提供上下文。

### TensorShape (lines 13-44)
```python
class TensorShape:
    def __init__(
        self,
        *dims: int | str,
        dynamic_dims: set[str] | None = None,
    ) -> None:
        super().__init__()

        self.dims = dims
        self.dynamic_dims = dynamic_dims if dynamic_dims else set()

    def resolve(self, **bindings: int) -> tuple[int | str, ...]:
        resolved = list[int | str]()
        for dim in self.dims:
            if isinstance(dim, str) and dim in bindings:
                resolved.append(bindings[dim])
            else:
                resolved.append(dim)
        return tuple(resolved)

    def __str__(self) -> str:
        """Return a string representation of the tensor shape."""
        dim_strs = []
        for dim in self.dims:
            if isinstance(dim, str):
                if dim in self.dynamic_dims:
                    dim_strs.append(f"{dim}*")  # Mark dynamic dimensions with *
                else:
                    dim_strs.append(dim)
            else:
                dim_strs.append(str(dim))
        return f"({', '.join(dim_strs)})"
```
**EN:** Defines the `TensorShape` class used by this module. Key methods include `__init__`, `resolve`.
**CN:** `TensorShape` 是该文件中的核心类，用于封装与 `TensorShape` 相关的状态和行为。 关键方法包括 `__init__`, `resolve`。

### TensorSchema overview (lines 47-255)
```python
class TensorSchema:
    def __init__(
        self,
        *,
        validate: bool = True,
        resolve_bindings: dict[str, int] | None = None,
        **kwargs: Any,
    ) -> None:
        super().__init__()

        self._resolve_bindings = resolve_bindings if resolve_bindings else {}

        for key, value in kwargs.items():
            setattr(self, key, value)

        if validate:
            self.validate()

    def __getitem__(self, key: str) -> Any:
        return getattr(self, key)

    def get(self, key: str, default: Any = None) -> Any:
        return getattr(self, key, default)

    def _match_shape_with_dynamic(
        self,
        actual: tuple[int, ...],
    # ...
```
**EN:** Defines the `TensorSchema` class used by this module. Key methods include `__init__`, `get`, `validate`, `print_shapes`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `TensorSchema` 是该文件中的核心类，用于封装与 `TensorSchema` 相关的状态和行为。 关键方法包括 `__init__`, `get`, `validate`, `print_shapes`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### TensorSchema.__init__ (lines 48-63)
```python
    def __init__(
        self,
        *,
        validate: bool = True,
        resolve_bindings: dict[str, int] | None = None,
        **kwargs: Any,
    ) -> None:
        super().__init__()

        self._resolve_bindings = resolve_bindings if resolve_bindings else {}

        for key, value in kwargs.items():
            setattr(self, key, value)

        if validate:
            self.validate()
```
**EN:** `__init__` initializes state required by the module. It mainly works with `validate`, `resolve_bindings`, `**kwargs`. Inside the body, it relies on `super.__init__`, `kwargs.items`, `self.validate` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 它主要处理 `validate`, `resolve_bindings`, `**kwargs` 等参数。 实现过程中会调用 `super.__init__`, `kwargs.items`, `self.validate` 等函数完成关键步骤。

### TensorSchema.validate (lines 199-243)
```python
    def validate(self) -> None:
        type_hints = get_type_hints(self.__class__, include_extras=True)
        shape_env = dict[str, int]()

        for field_name, field_type in type_hints.items():
            # Check if field is missing
            if not hasattr(self, field_name) or getattr(self, field_name) is None:
                # Check if field is marked as optional
                actual_type = field_type
                if get_origin(field_type) is Annotated:
                    args = get_args(field_type)
                    actual_type = args[0]

                # Check arg was provided as Union
                if get_origin(actual_type) in {Union, UnionType}:
                    # Union for Union[X, Y] and UnionType for X | Y
                    args = get_args(actual_type)
                    # Skip validation when Union contains None
                    if type(None) in args:
                        continue
    # ...
                            actual_shape,
                            expected_shape,
                            field_name,
                            shape_env,
                            arg.dynamic_dims,
                        )
```
**EN:** `validate` validates values and raises clear errors when constraints are violated. Inside the body, it relies on `get_type_hints`, `type_hints.items`, `ValueError` to complete the main steps.
**CN:** `validate` 负责校验取值并在约束不满足时抛出清晰错误。 实现过程中会调用 `get_type_hints`, `type_hints.items`, `ValueError` 等函数完成关键步骤。

### TensorSchema._validate_field (lines 100-153)
```python
    def _validate_field(
        self,
        value: object,
        field_name: str,
        expected_shape: tuple[int | str, ...],
        dynamic_dims: set[str],
        leading_idxs: tuple[int, ...] = (),
    ) -> tuple[int, ...]:
        """Validate a field and return the actual shape."""
        if isinstance(value, (int, float)):
            return ()  # Scalar
        if isinstance(value, torch.Tensor):
            return value.shape

        if not isinstance(value, (list, tuple)):
            raise TypeError(
                f"{field_name}{self._fmt_indexer(leading_idxs)} is not "
                f"one of the expected types: int, float, Tensor, list, tuple. "
                f"Got: {type(value)}"
            )
    # ...
                    f"(index 0) vs {shape} (index {i})"
                )

        # Treat the list as a stacked tensor:
        # shape = (len(list), *tensor.shape)
        return (len(value),) + first_shape
```
**EN:** `_validate_field`: Validate a field and return the actual shape. It mainly works with `value`, `field_name`, `expected_shape`, `dynamic_dims`. Inside the body, it relies on `TypeError`, `ValueError`, `self._validate_field` to complete the main steps.
**CN:** `_validate_field` 负责校验取值并在约束不满足时抛出清晰错误。 它主要处理 `value`, `field_name`, `expected_shape`, `dynamic_dims` 等参数。 实现过程中会调用 `TypeError`, `ValueError`, `self._validate_field` 等函数完成关键步骤。

### TensorSchema._validate_tensor_shape_expected (lines 155-197)
```python
    def _validate_tensor_shape_expected(
        self,
        actual_shape: tuple[int, ...],
        expected_shape: tuple[int | str, ...],
        field_name: str,
        shape_env: dict[str, int],
        dynamic_dims: set[str],
    ) -> None:
        """Validate that the actual tensor shape matches the expected shape."""

        if len(actual_shape) != len(expected_shape):
            raise ValueError(
                f"{field_name} has rank {len(actual_shape)} "
                f"but expected {len(expected_shape)}. "
                f"Expected shape: {expected_shape}, "
                f"but got {actual_shape}"
            )

        for i, dim in enumerate(expected_shape):
            if dim in dynamic_dims:
    # ...
                else:
                    shape_env[dim] = actual_shape[i]
            else:
                raise TypeError(
                    f"{field_name} dim[{i}] has unsupported type: {type(dim)}"
                )
```
**EN:** `_validate_tensor_shape_expected`: Validate that the actual tensor shape matches the expected shape. It mainly works with `actual_shape`, `expected_shape`, `field_name`, `shape_env`. Inside the body, it relies on `ValueError`, `TypeError`, `type` to complete the main steps.
**CN:** `_validate_tensor_shape_expected` 负责校验取值并在约束不满足时抛出清晰错误。 它主要处理 `actual_shape`, `expected_shape`, `field_name`, `shape_env` 等参数。 实现过程中会调用 `ValueError`, `TypeError`, `type` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`TensorShape`**: Core class that organizes module behavior. / **`TensorShape`**：组织模块行为的核心类。
- **`TensorSchema`**: Core class that organizes module behavior. / **`TensorSchema`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: types, typing
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.logger
