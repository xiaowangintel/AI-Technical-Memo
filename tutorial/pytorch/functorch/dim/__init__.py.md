# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/dim/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements dimension-aware tensor wrappers and tracing helpers for experimental named-dimension style transforms.
- **Purpose (CN)**: 实现带维度语义的张量包装器与跟踪辅助逻辑，用于实验性的命名维度变换。
## Line-by-Line Analysis / 逐行分析

### Lines 1-35
```python
from __future__ import annotations

import dis
import inspect
import sys
from typing import Any, Optional, TYPE_CHECKING, Union


if TYPE_CHECKING:
    from collections.abc import Callable, Sequence

import torch
from torch.utils._pytree import tree_flatten, tree_map, tree_unflatten

from ._dim_entry import _match_levels, DimEntry, ndim_of_levels
from ._enable_all_layers import EnableAllLayers
from ._py_inst_decoder import _PyInstDecoder
from ._tensor_info import TensorInfo


POINTWISE_OPTIMIZE = True
DOT_OPTIMIZED = True

# Global dimension level counter
_n_dims_created = 0


def _relevant_op(opcode: str | None) -> bool:
    """Check if opcode is relevant for variable assignment."""
    return bool(opcode and opcode.startswith("STORE_"))


def handle_from_tensor(tensor: torch.Tensor) -> torch.Tensor:
    """Handle tensor conversion for torch function integration."""
    return tensor
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.utils._pytree, ._dim_entry, ._enable_all_layers, and 2 more; standard-library modules such as __future__, dis, inspect, sys, and 2 more for the logic below. This chunk defines `handle_from_tensor`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.utils._pytree、._dim_entry、._enable_all_layers 等共 6 项；标准库模块，如 __future__、dis、inspect、sys 等共 6 项组织在一起，供下方逻辑使用。 这一段定义了 `handle_from_tensor`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 38-71
```python
def _create_dim(name: str, size: int | None = None) -> Dim:
    """Create a new Dim object."""
    return Dim(name, size if size is not None else -1)


def dims(
    n: int | None = None, sizes: list[int | None] | None = None
) -> Dim | tuple[Dim, ...]:
    """
    Create and return one or more Dim objects.

    Uses bytecode inspection to determine variable names when possible.

    Args:
        n (int, optional): The number of dimensions to create. Can be omitted if sizes is specified.
        sizes (List[Optional[int]], optional): A list the same size as the number of dimensions to be
          created, specifying each dimensions size, or None to leave the size unset.

    Returns:
        Union[Dim, Tuple[Dim, ...]]: Single Dim if n=1, tuple of Dims otherwise.

    Examples:
        >>> batch, channel, width, height = dims(4)
        >>> batch, channel, width, height = dims(sizes=[None, 3, 224, 224])
        >>> single_dim = dims(1)
    """
    specified_ndims = -1
    found_ndims = 0

    # Parse arguments
    if sizes is not None:
        specified_ndims = len(sizes)
    if n is not None:
        specified_ndims = n
```
- **EN**: This chunk defines `dims`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `dims`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 73-108
```python
    # Use bytecode inspection
    frame = inspect.currentframe()
    if frame is None:
        raise RuntimeError("Unable to get current frame")
    frame = frame.f_back
    try:
        if frame is None:
            raise RuntimeError("Unable to get caller frame")
        code = frame.f_code
        lasti = frame.f_lasti

        decoder = _PyInstDecoder(code, lasti)

        if sys.version_info >= (3, 11):
            if decoder.opcode() == "PRECALL":
                decoder.next()

        # Move to next instruction after the call
        decoder.next()

        # Determine number of dimensions from bytecode
        if _relevant_op(decoder.opcode()):
            found_ndims = 1
        elif decoder.opcode() == "UNPACK_SEQUENCE":
            found_ndims = decoder.oparg()
            decoder.next()  # Move past UNPACK_SEQUENCE

        if specified_ndims == -1:
            if found_ndims == 0:
                raise SyntaxError(
                    "dims() must be assigned to a sequence of variable names or have argument n specified"
                )
            specified_ndims = found_ndims

        if found_ndims != specified_ndims:
            found_ndims = 0
```
- **EN**: This chunk continues `dims` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `dims`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 110-144
```python
        def genobject(i: int) -> Dim:
            nonlocal found_ndims
            name = None
            if i < found_ndims:
                name = decoder.name()

            if not name:
                name = f"d{i}"
                found_ndims = 0
            else:
                decoder.next()  # Move to next STORE instruction

            size = sizes[i] if sizes is not None else None
            return _create_dim(name, size)

        # Validate sizes parameter
        if sizes is not None and len(sizes) != specified_ndims:
            raise ValueError(f"expected {specified_ndims} sizes but found {len(sizes)}")

        if specified_ndims == 1:
            return genobject(0)

        result = []
        for i in range(specified_ndims):
            result.append(genobject(i))

        return tuple(result)

    finally:
        del frame


class DimList:
    """
    A list of first-class dimensions that can be bound to tensor dimensions.
```
- **EN**: It introduces or extends DimList, dimensions, which hold the primary data model or public surface for this slice of the file. This chunk defines `genobject`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 它引入或扩展了 DimList、dimensions，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `genobject`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 146-182
```python
    A DimList can be in one of two states:
    1. Unbound: Created with just a name, no specific dimensions yet
    2. Bound: Either created with specific dimensions/sizes, or bound later via bind() or bind_len()
    """

    _name: str | None
    _dims: list[Dim]
    _bound: bool

    def __init__(
        self,
        len_or_dims: int | Sequence | None = None,
        name: str | None = None,
    ):
        """
        Initialize a new DimList object.

        Args:
            len_or_dims: Optional length (int) or sequence of dimensions/sizes
            name: Optional name for the dimension list
        """
        # Initialize attributes
        self._name = name
        self._dims: list = []
        self._bound = False

        if isinstance(len_or_dims, int):
            self.bind_len(len_or_dims)
        elif len_or_dims is not None:
            dims = []
            for i, item in enumerate(len_or_dims):
                if isinstance(item, int):
                    dim_name = f"{self._name}{i}" if self._name else f"dim{i}"
                    dims.append(Dim(dim_name, item))
                else:
                    dims.append(Dim(item))
            self._set_dims(dims)
```
- **EN**: This chunk defines `__init__`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `__init__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 184-222
```python
    def _set_dims(self, dims: list) -> None:
        """Set the dimensions and mark as bound."""
        self._bound = True
        self._dims = dims

    def bind_len(self, size: int) -> None:
        """
        Bind this DimList to a specific length.

        Args:
            size: Number of dimensions to bind to

        Raises:
            DimensionBindError: If already bound to a different size
        """
        if self._bound:
            if len(self._dims) != size:
                raise DimensionBindError(
                    f"Dimlist has size {len(self._dims)} but it is being bound to size {size}"
                )
        else:
            self._bound = True
            self._dims = []
            for i in range(size):
                dim_name = f"{self._name}{i}" if self._name else f"dim{i}"
                self._dims.append(Dim(dim_name))

    def bind(self, sizes: Sequence[int]) -> None:
        """
        Bind this DimList to specific sizes.

        Args:
            sizes: Sequence of sizes for each dimension

        Raises:
            ValueError: If sizes is not a sequence
        """
        if not hasattr(sizes, "__len__") or not hasattr(sizes, "__getitem__"):
            raise ValueError("expected a sequence")
```
- **EN**: This chunk defines `bind`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `bind`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 224-254
```python
        size = len(sizes)
        self.bind_len(size)

        for i, dim_size in enumerate(sizes):
            self._dims[i].size = int(dim_size)

    def _size(self) -> int:
        if not self._bound:
            raise DimensionBindError("DimList not bound")
        return len(self._dims)

    def size(self) -> int:
        """Return the size (number of dimensions) of this DimList."""
        return self._size()

    def _set_bound(self, b: bool) -> None:
        """Set the bound status (for internal use)."""
        self._bound = b

    @property
    def is_bound(self) -> bool:
        """Property to check if DimList is bound."""
        return self._bound

    def __len__(self) -> int:
        """Return the length of the DimList."""
        return self.size()

    def __getitem__(self, key: int | slice) -> Dim | tuple[Dim, ...]:
        if not self._bound:
            raise DimensionBindError("DimList not bound")
```
- **EN**: Decorators such as @property modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `__getitem__`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @property 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `__getitem__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 256-293
```python
        if isinstance(key, int):
            if key < 0 or key >= len(self._dims):
                raise IndexError("index out of bounds")
            return self._dims[key]
        elif isinstance(key, slice):
            start, stop, step = key.indices(len(self._dims))
            result = []
            for i in range(start, stop, step):
                result.append(self._dims[i])
            return tuple(result)
        else:
            raise ValueError("expected an int or a slice")

    def __repr__(self) -> str:
        """Return string representation of the DimList."""
        if self._bound:
            # Show as tuple representation
            return f"({', '.join(repr(dim) for dim in self._dims)})"
        elif self._name is not None:
            # Show as *name for unbound with name
            return f"*{self._name}"
        else:
            # Show as <unbound_dimlist> for unbound without name
            return "<unbound_dimlist>"

    def __str__(self) -> str:
        """Return string representation of the DimList."""
        return self.__repr__()

    @classmethod
    def __torch_function__(
        cls,
        func: Callable,
        types: tuple,
        args: tuple = (),
        kwargs: dict | None = None,
    ) -> Any:
        return _Tensor.__torch_function__(func, types, args, kwargs)
```
- **EN**: Decorators such as @classmethod modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `__torch_function__`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @classmethod 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `__torch_function__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 296-326
```python
def _create_dimlist(name: str, size: int | list[int | None] | None = None) -> DimList:
    """Create a DimList object with the given name and optional size."""
    dimlist = DimList(name=name)
    if size is not None:
        if isinstance(size, int):
            dimlist.bind_len(size)
        else:
            # size is a list of optional ints
            dimlist.bind_len(len(size))
            for i, s in enumerate(size):
                if s is not None:
                    dimlist._dims[i].size = s
    return dimlist


def dimlists(
    n: int | None = None, sizes: list[int | None] | None = None
) -> DimList | tuple[DimList, ...]:
    """
    Create and return one or more DimList objects.

    Similar to dims() but creates DimList objects instead.
    """
    specified_ndims = -1
    found_ndims = 0

    # Parse arguments
    if sizes is not None:
        specified_ndims = len(sizes)
    if n is not None:
        specified_ndims = n
```
- **EN**: This chunk defines `dimlists`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `dimlists`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 328-364
```python
    frame = inspect.currentframe()
    if frame is None:
        raise RuntimeError("Unable to get current frame")
    frame = frame.f_back
    try:
        if frame is None:
            raise RuntimeError("Unable to get caller frame")
        code = frame.f_code
        lasti = frame.f_lasti

        decoder = _PyInstDecoder(code, lasti)

        if sys.version_info >= (3, 11):
            if decoder.opcode() == "PRECALL":
                decoder.next()

        # Move to next instruction after the call
        decoder.next()

        # Determine number of dimensions from bytecode
        if _relevant_op(decoder.opcode()):
            found_ndims = 1
        elif decoder.opcode() == "UNPACK_SEQUENCE":
            found_ndims = decoder.oparg()
            decoder.next()  # Move past UNPACK_SEQUENCE

        if specified_ndims == -1:
            if found_ndims == 0:
                raise SyntaxError(
                    "dimlists() must be assigned to a sequence of variable names or have argument n specified"
                )
            specified_ndims = found_ndims

        if found_ndims != specified_ndims:
            found_ndims = 0

        # Generator function for dimlist names
```
- **EN**: This chunk continues `dimlists` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `dimlists`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 365-401
```python
        def genobject(i: int) -> str:
            nonlocal found_ndims
            name = None
            if i < found_ndims:
                name = decoder.name()

            if not name:
                name = f"d{i}"
                found_ndims = 0
            else:
                decoder.next()  # Move to next STORE instruction

            return name

        # Validate sizes
        if sizes is not None and len(sizes) != specified_ndims:
            raise ValueError(f"expected {specified_ndims} sizes but found {len(sizes)}")

        # Create dimlists
        if specified_ndims == 1:
            name = genobject(0)
            return _create_dimlist(name, sizes[0] if sizes is not None else None)

        result = []
        for i in range(specified_ndims):
            name = genobject(i)
            size = sizes[i] if sizes is not None else None
            result.append(_create_dimlist(name, size))

        return tuple(result)

    finally:
        del frame


class DimensionMismatchError(Exception):
    pass
```
- **EN**: It introduces or extends DimensionMismatchError, which hold the primary data model or public surface for this slice of the file. This chunk defines `genobject`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 它引入或扩展了 DimensionMismatchError，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `genobject`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 404-442
```python
class DimensionBindError(Exception):
    pass


from . import op_properties


def _safe_print(*args: Any, **kwargs: Any) -> None:
    """Safe print that avoids recursive torch function dispatches."""
    import sys

    # Convert any torch objects to basic representations
    safe_args = []
    for arg in args:
        if hasattr(arg, "__class__") and "torch" in str(type(arg)):
            safe_args.append(f"<{type(arg).__name__}>")
        else:
            safe_args.append(str(arg))

    print(*safe_args, **kwargs, file=sys.stderr)


class _Tensor:
    def _get_levels(self) -> list[Any]:
        raise NotImplementedError("_get_levels must be implemented by subclass")

    def _get_tensor(self) -> torch.Tensor | None:
        raise NotImplementedError("_get_tensor must be implemented by subclass")

    @property
    def ndim(self) -> int:
        raise NotImplementedError("ndim must be implemented by subclass")

    @property
    def dims(self) -> tuple[Any, ...]:
        return tuple(l.dim() for l in self._get_levels() if not l.is_positional())

    def dim(self) -> int:
        return self.ndim
```
- **EN**: The import section wires together PyTorch-local modules such as .; standard-library modules such as sys for the logic below. Decorators such as @property, @property modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends DimensionBindError, _Tensor, which hold the primary data model or public surface for this slice of the file. This chunk defines `dim`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 .；标准库模块，如 sys组织在一起，供下方逻辑使用。 像 @property、@property 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 DimensionBindError、_Tensor，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `dim`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 444-469
```python
    @classmethod
    def __torch_function__(
        cls,
        func: Callable,
        types: tuple,
        args: tuple = (),
        kwargs: dict | None = None,
    ) -> Any:
        if kwargs is None:
            kwargs = {}

        if DOT_OPTIMIZED and func is torch.Tensor.__mul__:
            # Check conditions: 2 args, both are tensor-like, both 0-dimensional
            if (
                len(args) == 2
                and not kwargs
                and isinstance(args[0], (_Tensor, torch.Tensor))
                and isinstance(args[1], (_Tensor, torch.Tensor))
            ):
                # Get tensor info for both operands
                lhs_info = TensorInfo.create(
                    args[0], ensure_batched=False, ensure_present=False
                )
                rhs_info = TensorInfo.create(
                    args[1], ensure_batched=False, ensure_present=False
                )
```
- **EN**: Decorators such as @classmethod modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `__torch_function__`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 像 @classmethod 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `__torch_function__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 471-506
```python
                if (
                    lhs_info
                    and rhs_info
                    and lhs_info.tensor is not None
                    and rhs_info.tensor is not None
                    and lhs_info.tensor.dim() == 0
                    and rhs_info.tensor.dim() == 0
                ):
                    if (
                        lhs_info.tensor.is_floating_point()
                        and rhs_info.tensor.is_floating_point()
                    ):
                        # Collect all unique levels and has_device
                        has_device = lhs_info.has_device or rhs_info.has_device
                        levels = []

                        for level in lhs_info.levels:
                            if level not in levels:
                                levels.append(level)
                        for level in rhs_info.levels:
                            if level not in levels:
                                levels.append(level)

                        # Debug print
                        # print(f"DEBUG: Creating delayed mul, levels: {levels}, has_device: {has_device}")

                        # Create delayed tensor
                        return Tensor.create_delayed(func, args, levels, has_device)

        if func is torch.Tensor.__getitem__:
            from functorch.dim._getsetitem import getitem

            return getitem(cls, func, types, args, kwargs)

        if func is torch.Tensor.__setitem__:
            from functorch.dim._getsetitem import setitem
```
- **EN**: The import section wires together PyTorch-local modules such as functorch.dim._getsetitem for the logic below. This chunk continues `__torch_function__` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 functorch.dim._getsetitem组织在一起，供下方逻辑使用。 这一段延续了 `__torch_function__`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 508-545
```python
            # args should be (tensor, index, value)
            if len(args) == 3:
                setitem(args[0], args[1], args[2])
                return None
            else:
                raise ValueError(f"Expected 3 args for __setitem__, got {len(args)}")

        # Fast-path for len; mostly to avoid infinite loop in TestMinFunctorchOnly.test_softmax_split
        if func is torch.Tensor.__len__:
            return args[0].size(0)

        # Special handling for torch.softmax - use the pre-wrapped version
        if func is torch.softmax:
            return softmax(*args, **kwargs)

        # Special handling for torch.stack - use the custom stack function
        if func is torch.stack:
            return stack(*args, **kwargs)

        if (
            func is torch.Tensor.split
            or func is torch._VF.split  # type: ignore[attr-defined]
            or func is torch._VF.split_with_sizes  # type: ignore[attr-defined]
            or func is torch.split
        ):
            return split(*args, **kwargs)

        return _Tensor._torch_function_fallback(func, types, args, kwargs)

    @staticmethod
    def _torch_function_fallback(
        func: Callable, types: tuple, args: tuple, kwargs: dict
    ) -> Any:
        """Fallback torch function implementation for non-special-cased functions."""
        is_pointwise = POINTWISE_OPTIMIZE and func in op_properties.pointwise
        # TODO: optimize pytree here
        flat_args, spec = tree_flatten((args, kwargs))
        device_holding_tensor = None
```
- **EN**: Decorators such as @staticmethod modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `_torch_function_fallback`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @staticmethod 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `_torch_function_fallback`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 547-580
```python
        infos: list[TensorInfo] = []
        result_levels: list[DimEntry] = []

        for f in flat_args:
            info = TensorInfo.create(f, not is_pointwise, False)
            infos.append(info)
            if info:
                if not (is_pointwise or info.batchedtensor is not None):
                    raise AssertionError(
                        "Expected pointwise or batchedtensor to be set"
                    )
                if device_holding_tensor is None and info.has_device:
                    device_holding_tensor = info.tensor
                # Collect all unique levels
                for level in info.levels:
                    if not isinstance(level, DimEntry):
                        raise AssertionError(f"Expected DimEntry, got {type(level)}")
                    if level not in result_levels:
                        result_levels.append(level)

        if is_pointwise:
            # Pointwise operation: match all tensors to common levels
            for i, info in enumerate(infos):
                if info and info.tensor is not None:
                    tensor = info.tensor
                    if device_holding_tensor is not None and not info.has_device:
                        tensor = tensor.to(device_holding_tensor.device)
                    ml = _match_levels(tensor, info.levels, result_levels)
                    flat_args[i] = handle_from_tensor(ml)

            unflat_args, unflat_kwargs = tree_unflatten(flat_args, spec)
            result = func(*unflat_args, **unflat_kwargs)

            # Wrap tensor results
```
- **EN**: This chunk continues `_torch_function_fallback` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `_torch_function_fallback`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 581-617
```python
            def wrap_tensor(obj: Any) -> Any:
                if isinstance(obj, torch.Tensor):
                    return Tensor.from_positional(
                        obj, result_levels, device_holding_tensor is not None
                    )
                return obj

            # Small fastpath
            if isinstance(result, torch.Tensor):
                return wrap_tensor(result)
            else:
                return tree_map(wrap_tensor, result)

        # Non-pointwise operation: use functorch vmap layers
        with EnableAllLayers(result_levels) as guard:
            # Update arguments with batched tensors
            for i, info in enumerate(infos):
                if info and info.batchedtensor is not None:
                    batched = info.batchedtensor
                    if device_holding_tensor is not None and not info.has_device:
                        batched = batched.to(device_holding_tensor.device)
                    guard.inplace_update_layers(batched, info.levels)
                    flat_args[i] = handle_from_tensor(batched)

            unflat_args, unflat_kwargs = tree_unflatten(flat_args, spec)
            result = func(*unflat_args, **unflat_kwargs)

            # Unwrap results from functorch layers
            def unwrap_tensor(obj: Any) -> Any:
                if isinstance(obj, torch.Tensor):
                    return guard.from_batched(obj, device_holding_tensor is not None)
                return obj

            if isinstance(result, torch.Tensor):
                return unwrap_tensor(result)
            else:
                return tree_map(unwrap_tensor, result)
```
- **EN**: This chunk defines `unwrap_tensor`, which wraps lower-level behavior in a transform-friendly interface. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `unwrap_tensor`，其作用是把底层行为包装为更适合变换组合的接口。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 619-657
```python
    def __setitem__(self, index: Any, value: Any) -> None:
        """Set values in tensor using first-class dimensions."""
        from functorch.dim._getsetitem import setitem

        return setitem(self, index, value)

    # expand and index are OK to be methods because they don't have torch.*
    # versions, but if they did they need the stack/cat treatment

    def expand(self, *args: Dim) -> _Tensor:
        """
        Expand tensor by adding new dimensions or expanding existing dimensions.

        If all arguments are Dim objects, adds new named dimensions.
        Otherwise, falls back to regular tensor expansion behavior.

        Args:
            args: Either Dim objects for new dimensions or sizes for regular expansion

        Returns:
            New tensor with expanded dimensions

        Example:
            >>> i, j = dims()
            >>> t = torch.randn(3, 4)
            >>> expanded = t[i].expand(j, k)  # Add j, k dimensions
            >>> expanded2 = t[i].expand(2, 4)  # Regular expand with sizes
        """
        info = TensorInfo.create(self, ensure_batched=False, ensure_present=False)

        for arg in args:
            if not isinstance(arg, Dim):
                # Not all args are Dims, fallback to regular expand
                if isinstance(self, torch.Tensor) and not isinstance(self, _Tensor):
                    return torch.Tensor.expand(self, *args)
                else:
                    return self.__torch_function__(
                        torch.Tensor.expand, (type(self),), (self,) + args
                    )
```
- **EN**: The import section wires together PyTorch-local modules such as functorch.dim._getsetitem for the logic below. It introduces or extends dimensions, which hold the primary data model or public surface for this slice of the file. This chunk defines `expand`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 functorch.dim._getsetitem组织在一起，供下方逻辑使用。 它引入或扩展了 dimensions，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `expand`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 659-695
```python
        # All args are Dim objects - proceed with first-class dimension expansion
        if not info:
            # No tensor info available, fallback
            return self.__torch_function__(
                torch.Tensor.expand, (type(self),), (self,) + args
            )

        # First-class dimension expansion - all args are Dim objects
        data = info.tensor
        if data is None:
            # No tensor data available, fallback
            return self.__torch_function__(
                torch.Tensor.expand, (type(self),), (self,) + args
            )

        levels = info.levels

        new_levels: list[DimEntry] = []
        new_sizes = []
        new_strides = []

        for d in args:
            # Check if dimension already exists in current levels or new_levels
            for level in levels:
                if not level.is_positional() and level.dim() is d:
                    raise DimensionBindError(
                        f"expanding dimension {d} already exists in tensor with dims"
                    )
            for new_level in new_levels:
                if not new_level.is_positional() and new_level.dim() is d:
                    raise DimensionBindError(
                        f"expanding dimension {d} already exists in tensor with dims"
                    )

            new_levels.append(DimEntry(d))
            new_sizes.append(d.size)
            new_strides.append(0)
```
- **EN**: It introduces or extends dimension, which hold the primary data model or public surface for this slice of the file. This chunk continues `dimension` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 它引入或扩展了 dimension，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `dimension`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 697-733
```python
        # Add existing levels
        new_levels.extend(levels)

        # Add existing sizes and strides
        orig_sizes = list(data.size())
        orig_strides = list(data.stride())
        new_sizes.extend(orig_sizes)
        new_strides.extend(orig_strides)

        # Create expanded tensor using as_strided
        expanded_data = data.as_strided(new_sizes, new_strides, data.storage_offset())

        # Return new tensor with expanded dimensions
        result = Tensor.from_positional(expanded_data, new_levels, info.has_device)
        return result  # type: ignore[return-value]  # Tensor and torch.Tensor are interchangeable

    def index(
        self,
        dims: int | Dim | tuple[int | Dim, ...] | list[int | Dim],
        indices: int
        | slice
        | torch.Tensor
        | tuple[int | slice | torch.Tensor, ...]
        | list[int | slice | torch.Tensor],
    ) -> _Tensor:
        """
        Index tensor using first-class dimensions.
        """
        from ._dim_entry import _match_levels
        from ._getsetitem import getsetitem_flat, invoke_getitem
        from ._wrap import _wrap_dim

        # Helper to check if obj is a dimpack (tuple/list) and extract items
        def maybe_dimpack(obj: Any, check_first: bool = False) -> tuple[Any, bool]:
            if isinstance(obj, (tuple, list)):
                return list(obj), True
            return None, False
```
- **EN**: The import section wires together PyTorch-local modules such as ._dim_entry, ._getsetitem, ._wrap for the logic below. It introduces or extends dimensions, which hold the primary data model or public surface for this slice of the file. This chunk defines `maybe_dimpack`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 ._dim_entry、._getsetitem、._wrap组织在一起，供下方逻辑使用。 它引入或扩展了 dimensions，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `maybe_dimpack`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 735-771
```python
        def parse_dim_entry(s: Any) -> Any:
            d = _wrap_dim(s, self.ndim, False)
            if d.is_none():
                raise TypeError(f"expected a dimension specifyer but found {repr(s)}")
            return d

        # Helper for dimension not present errors
        def dim_not_present(d: Any) -> None:
            if d.is_positional():
                raise TypeError(
                    f"dimension {d.position() + self.ndim} not in tensor of {self.ndim} dimensions"
                )
            else:
                raise TypeError(f"dimension {repr(d.dim())} not in tensor")

        dims_list: list[int | Dim] = []
        indices_list: list[int | slice | torch.Tensor] = []

        lhs_list = isinstance(dims, (tuple, list))
        rhs_list = isinstance(indices, (tuple, list))

        if lhs_list and rhs_list:
            # Type narrowing: we know dims and indices are sequences here
            dims_seq = dims  # type: ignore[assignment]
            indices_seq = indices  # type: ignore[assignment]
            if len(dims_seq) != len(indices_seq):  # type: ignore[arg-type]
                raise TypeError(
                    f"dims ({len(dims_seq)}) and indices ({len(indices_seq)}) must have the same length"  # type: ignore[arg-type]
                )
            dims_list.extend(dims_seq)  # type: ignore[arg-type]
            indices_list.extend(indices_seq)  # type: ignore[arg-type]
        else:
            dims_list.append(dims)  # type: ignore[arg-type]
            indices_list.append(indices)  # type: ignore[arg-type]

        # Create tensor info
        self_info = TensorInfo.create(self, False, False)
```
- **EN**: This chunk defines `dim_not_present`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `dim_not_present`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 773-806
```python
        new_levels: list[Any] = []
        to_flatten: list[Any] = []
        dims_list_flat = []

        # Process each dim specification
        for i in range(len(dims_list)):
            m, is_dimpack = maybe_dimpack(dims_list[i], check_first=False)
            if is_dimpack:
                if len(m) == 0:
                    dims_list_flat.append(DimEntry())  # Empty dimpack
                    continue

                first = parse_dim_entry(m[0])
                dims_list_flat.append(first)

                if len(m) == 1:
                    continue

                # Multi-element dimpack requires flattening
                if len(to_flatten) == 0:
                    new_levels.extend(self_info.levels)

                rest = []
                for j in range(1, len(m)):
                    d = parse_dim_entry(m[j])
                    removed = False
                    for k in range(len(new_levels)):
                        if new_levels[k] == d:
                            new_levels.pop(k)
                            removed = True
                            break
                    if not removed:
                        dim_not_present(d)
                    rest.append(d)
```
- **EN**: This chunk continues `dim_not_present` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `dim_not_present`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 808-846
```python
                # Find first in new_levels
                first_idx = None
                for k in range(len(new_levels)):
                    if new_levels[k] == first:
                        first_idx = k
                        break
                if first_idx is None:
                    dim_not_present(first)
                    continue  # Skip this iteration if dimension not found

                for j, r in enumerate(rest):
                    new_levels.insert(first_idx + 1 + j, r)
                to_flatten.extend(rest)
            else:
                dims_list_flat.append(parse_dim_entry(dims_list[i]))

        # Handle dimension flattening if needed
        if len(to_flatten) > 0:
            if self_info.tensor is None:
                raise AssertionError(
                    "Cannot perform dimension flattening on None tensor"
                )
            rearranged = _match_levels(self_info.tensor, self_info.levels, new_levels)
            sizes = rearranged.size()
            new_sizes: list[Any] = []
            reshape_levels = []

            for i in range(len(new_levels)):
                if new_levels[i] in to_flatten:
                    if len(new_sizes) == 0:
                        new_sizes.append(sizes[i])
                    else:
                        new_sizes[-1] *= sizes[i]
                else:
                    new_sizes.append(sizes[i])
                    reshape_levels.append(new_levels[i])

            self_info.tensor = rearranged.reshape(new_sizes)
            self_info.levels = reshape_levels
```
- **EN**: This chunk continues `dim_not_present` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `dim_not_present`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 848-882
```python
        # Check for dimpacks in indices
        has_dimpacks = False
        for idx in indices_list:
            if isinstance(idx, (tuple, list)):
                has_dimpacks = True
                break

        # Call getsetitem_flat with correct parameters
        info = getsetitem_flat(
            self_info,
            [],  # empty input_list
            dims_list_flat,  # keys
            indices_list,  # values
            has_dimpacks,
        )

        return invoke_getitem(info)

    def __repr__(self) -> str:
        tensor, levels, ndim = self._get_tensor(), self._get_levels(), self.ndim
        dims_repr = []
        for l in levels:
            if hasattr(l, "is_positional") and l.is_positional():
                # Convert negative positional to positive: -1 -> ndim-1, -2 -> ndim-2, etc.
                dims_repr.append(l.position() + ndim)
            elif hasattr(l, "dim"):
                dims_repr.append(l.dim())
            elif hasattr(l, "data"):
                dims_repr.append(l.data)
            else:
                dims_repr.append(l)
        return f"{tensor}\nwith dims={tuple(dims_repr)} sizes={tuple(tensor.size())}"  # type: ignore[union-attr]


TensorLike = (_Tensor, torch.Tensor)
```
- **EN**: This chunk defines `__repr__`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `__repr__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 885-923
```python
class Dim(_Tensor):
    _level: int
    _name: str
    _size: int
    _range: torch.Tensor | None
    _batchtensor: torch.Tensor | None

    def __init__(self, name: str, s: int = -1) -> None:
        global _n_dims_created
        self._name = name
        self._size = s
        self._level = _n_dims_created
        _n_dims_created += 1
        self._range = None
        self._batchtensor = None

    @property
    def ndim(self) -> int:
        return 1

    @classmethod
    def check_exact(cls, obj: Any) -> bool:
        return type(obj) is cls

    @property
    def size(self) -> int:
        if self._size == -1:
            raise ValueError(f"dimension {self._name} is unbound")
        return self._size

    @size.setter
    def size(self, v: int) -> None:
        if self._size == -1:
            self._size = v
        elif self._size != v:
            raise DimensionBindError(
                f"Dim '{repr(self)}' previously bound to a dimension of size {self._size} "
                f"cannot bind to a dimension of size {v}"
            )
```
- **EN**: Decorators such as @property, @classmethod, and 2 more modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends Dim, which hold the primary data model or public surface for this slice of the file. This chunk defines `size`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @property、@classmethod 等共 4 项 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 Dim，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `size`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 925-960
```python
    @property
    def is_bound(self) -> bool:
        """Return True if this dimension is bound to a size."""
        return self._size != -1

    def _get_range(self) -> torch.Tensor:
        """
        Get a tensor representing the range [0, size) for this dimension.

        Returns:
            A 1D tensor with values [0, 1, 2, ..., size-1]
        """
        if self._range is None:
            self._range = torch.arange(self.size)
        return self._range

    def _get_batchtensor(self) -> torch.Tensor:
        """
        Get a batched tensor representation of this dimension.

        Returns:
            A batched tensor created from the range tensor
        """
        if self._batchtensor is None:
            self._batchtensor = torch._C._functorch._add_batch_dim(
                self._get_range(), 0, self._level
            )
        return self._batchtensor

    def __repr__(self) -> str:
        """String representation of a Dim object."""
        return self._name

    # note that Dim comes before tensor because we want the Dim API for things like size to take precedence.
    # Tensor defines format, but we want to print Dims with special formatting
    __format__ = object.__format__
```
- **EN**: Decorators such as @property modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `__repr__`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @property 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `__repr__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 963-995
```python
# Somewhat confusingly, an FCD tensor is also called Tensor.  This confusion
# is somewhat intentional, as FCD tensors are intended to be substitutable
# with regular Tensor (just with some positional dims hidden).
class Tensor(_Tensor):
    _tensor: torch.Tensor | None
    _batchtensor: torch.Tensor | None
    _levels: list[DimEntry]
    _has_device: bool
    _delayed: Callable[[], torch.Tensor] | None
    _delayed_orig: Callable | None
    _delayed_args: tuple | None

    @property
    def ndim(self) -> int:
        return sum(1 if l.is_positional() else 0 for l in self._levels)

    @classmethod
    def check_exact(cls, other: Any) -> bool:
        return type(other) is cls

    @classmethod
    def from_positional(
        cls, tensor: torch.Tensor, levels: list[DimEntry], has_device: bool
    ) -> _Tensor | torch.Tensor:
        """
        Create a functorch Tensor from a regular PyTorch tensor with specified dimension levels.

        This is the primary way to create Tensor objects with first-class dimensions.

        Args:
            tensor: The underlying PyTorch tensor
            levels: List of DimEntry objects specifying the dimension structure
            has_device: Whether the tensor is on a device (not CPU)
```
- **EN**: Decorators such as @property, @classmethod, and 1 more modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends Tensor, dimensions, which hold the primary data model or public surface for this slice of the file. This chunk defines `from_positional`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @property、@classmethod 等共 3 项 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 Tensor、dimensions，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `from_positional`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 997-1033
```python
        Returns:
            A new Tensor instance with the specified dimensions, or a regular torch.Tensor
            if there are no named dimensions
        """
        seen_dims = 0
        last = 0

        for l in levels:
            if l.is_positional():
                # Validate consecutive positional dimensions
                if not (last == 0 or last + 1 == l.position()):
                    raise AssertionError(
                        f"Positional dimensions must be consecutive, got {last} then {l.position()}"
                    )
                last = l.position()
            else:
                # This is a named dimension
                seen_dims += 1

        # Validate final positional dimension
        if not (last == 0 or last == -1):
            raise AssertionError(
                f"Final positional dimension must be 0 or -1, got {last}"
            )

        if not seen_dims:
            return tensor

        # Create Tensor object with proper level management
        result = cls()
        result._tensor = tensor
        result._levels = levels
        result._has_device = has_device
        result._batchtensor = None  # Will be created lazily if needed
        result._delayed = None
        result._delayed_orig = None
        result._delayed_args = None
```
- **EN**: This chunk continues `from_positional` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `from_positional`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1035-1070
```python
        # Validate tensor dimensionality matches levels
        if tensor.dim() != len(levels):
            raise AssertionError(
                f"Tensor has {tensor.dim()} dimensions but {len(levels)} levels provided"
            )

        return result

    @classmethod
    def create_delayed(
        cls, orig: Callable, args: tuple, levels: list[DimEntry], has_device: bool
    ) -> _Tensor:
        """
        Create a delayed tensor that defers the operation until later.
        """
        result = cls()
        result._tensor = None  # Will be computed when needed
        result._levels = levels
        result._has_device = has_device
        result._batchtensor = None
        result._delayed_orig = orig
        result._delayed_args = args

        # Create delayed evaluation function that unwraps Tensor objects
        def evaluate_delayed() -> torch.Tensor:
            unwrapped_args = []
            for arg in args:
                if hasattr(arg, "_get_tensor"):
                    unwrapped_args.append(arg._get_tensor())
                else:
                    unwrapped_args.append(arg)
            return orig(*unwrapped_args)

        result._delayed = evaluate_delayed

        return result
```
- **EN**: Decorators such as @classmethod modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `evaluate_delayed`, which measures behavior so implementations or heuristics can be compared. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @classmethod 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `evaluate_delayed`，其作用是测量行为，以便比较不同实现或启发式规则。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1072-1106
```python
    def _get_tensor(self) -> torch.Tensor | None:
        """Get the underlying tensor, handling delayed operations if needed."""
        if (
            hasattr(self, "_delayed")
            and self._delayed is not None
            and self._tensor is None
        ):
            # Execute the delayed operation
            self._tensor = self._delayed()
            # Clear delayed operation to avoid re-execution
            self._delayed = None
            self._delayed_orig = None
            self._delayed_args = None
        return self._tensor

    def _get_levels(self) -> list[Any]:
        """Get the dimension levels."""
        return self._levels

    def _get_has_device(self) -> bool:
        """Get whether this tensor has device information."""
        return self._has_device

    def _get_batchtensor(self) -> torch.Tensor | None:
        """Get the batched tensor representation, creating it lazily if needed."""
        if self._batchtensor is None:
            self._batchtensor = self._add_batch_dims(
                self._get_tensor(), self._get_levels()
            )
        return self._batchtensor

    def _add_batch_dims(
        self, t: torch.Tensor | None, levels_: list[Any]
    ) -> torch.Tensor | None:
        levels = list(levels_)
```
- **EN**: This chunk defines `_add_batch_dims`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `_add_batch_dims`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1108-1143
```python
        while True:
            min_real_index = -1
            min_index = -1
            min_value = float("inf")  # INT_MAX equivalent
            i = 0
            r = 0

            for r, l in enumerate(levels):
                if not l.is_none():
                    if not l.is_positional() and l.dim()._level < min_value:
                        min_value = l.dim()._level
                        min_index = i
                        min_real_index = r
                    i += 1

            if min_index == -1:
                return t

            if t is None:
                raise AssertionError("Expected t to be non-None")
            t = torch._C._functorch._add_batch_dim(t, min_index, int(min_value))

            levels[min_real_index] = DimEntry()
        return None

    def order(self, *dims: Any) -> _Tensor:
        """Reorder the dimensions of this tensor."""
        from ._order import order

        result = order(self, *dims)
        return result  # type: ignore[return-value]  # Tensor and torch.Tensor are interchangeable


def stack(tensors: Any, new_dim: Any, dim: int = 0) -> _Tensor:
    """
    Stack tensors along a new dimension.
```
- **EN**: The import section wires together PyTorch-local modules such as ._order for the logic below. This chunk defines `stack`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 ._order组织在一起，供下方逻辑使用。 这一段定义了 `stack`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1145-1182
```python
    Args:
        tensors: Sequence of tensors to stack
        new_dim: The new Dim to create for stacking
        dim: The dimension position to insert the new dimension (default: 0)

    Returns:
        Stacked tensor with the new dimension
    """
    if not tensors:
        raise ValueError("stack expects a non-empty sequence of tensors")

    # Check if new_dim is a Dim object
    if not isinstance(new_dim, Dim):
        # Fall back to regular torch.stack
        result = torch.stack(tensors, dim=dim)
        return result  # type: ignore[return-value]

    # Collect all result_levels from input tensors
    result_levels = []
    infos = []

    for t in tensors:
        info = TensorInfo.create(t, ensure_batched=False, ensure_present=False)
        infos.append(info)
        for level in info.levels:
            if level not in result_levels:
                result_levels.append(level)

    # Set the new_dim size to match number of tensors
    new_dim.size = len(tensors)

    # Match all tensors to the common level structure using _match_levels
    inputs = []
    for info in infos:
        if info.tensor is None:
            raise AssertionError("Cannot stack tensors with None tensor data")
        matched_tensor = _match_levels(info.tensor, info.levels, result_levels)
        inputs.append(matched_tensor)
```
- **EN**: This chunk continues `stack` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `stack`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1184-1220
```python
    # Calculate ndim and resolve the dim parameter
    ndim = ndim_of_levels(result_levels)
    rawdim = 0
    if dim is not None and not (isinstance(dim, int) and dim == 0):
        from ._wrap import _wrap_dim

        d = _wrap_dim(dim, ndim, False)
        try:
            idx = result_levels.index(d)
        except ValueError:
            raise TypeError(f"Dimension {dim} does not exist in inputs") from None
        rawdim = idx

    # Stack tensors at the resolved dimension
    result = torch.stack(inputs, rawdim)

    # Insert new dimension entry at the correct position
    result_levels.insert(rawdim, DimEntry(new_dim))

    # Return as a first-class tensor
    tensor_result = Tensor.from_positional(
        result, result_levels, infos[0].has_device if infos else True
    )
    return tensor_result  # type: ignore[return-value]


def split(tensor: Any, split_size_or_sections: Any, dim: Any = None) -> tuple:
    """
    Split tensor along a dimension.

    Can handle both regular integer sizes and Dim objects for split sizes.
    When Dim objects are used, they get bound to the resulting tensor dimensions.
    """
    from ._wrap import _wrap_dim

    # Check if dim is a Dim object
    dim_is_object = isinstance(dim, Dim)
```
- **EN**: The import section wires together PyTorch-local modules such as ._wrap for the logic below. It introduces or extends tensor, which hold the primary data model or public surface for this slice of the file. This chunk defines `split`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 ._wrap组织在一起，供下方逻辑使用。 它引入或扩展了 tensor，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `split`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1222-1259
```python
    # Parse split_size_or_sections
    if isinstance(split_size_or_sections, int):
        # Single integer - use regular split
        if dim_is_object:
            raise TypeError(
                "when dim is specified as a Dim object, split sizes must also be dimensions."
            )
        return _Tensor._torch_function_fallback(
            torch.Tensor.split,
            (type(tensor),),
            (tensor, split_size_or_sections),
            {"dim": dim},
        )

    # Check if it's a sequence
    sizes = []
    all_dims = True
    all_ints = True

    for item in split_size_or_sections:
        sizes.append(item)
        if isinstance(item, Dim):
            all_ints = False
        else:
            all_dims = False

    if all_ints:
        # All integers - use regular split
        if dim_is_object:
            raise TypeError(
                "when dim is specified as a Dim object, split sizes must also be dimensions."
            )
        return _Tensor._torch_function_fallback(
            torch.Tensor.split,
            (type(tensor),),
            (tensor, split_size_or_sections),
            {"dim": dim},
        )
```
- **EN**: This chunk continues `split` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `split`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1261-1297
```python
    if not all_dims:
        raise TypeError("split list must be ints or dims but got a mix")

    # All are Dim objects - handle first-class dimension split
    self_info = TensorInfo.create(tensor, ensure_batched=False, ensure_present=False)
    ndim = self_info.ndim()

    if not dim_is_object and ndim == 0:
        raise TypeError("split expects at least a 1-dimension tensor")

    # Wrap the dimension
    dim_l = _wrap_dim(dim, ndim, False) if dim is not None else DimEntry(-ndim)

    # Find the index of the dimension in levels
    idx = None
    for i, level in enumerate(self_info.levels):
        if level == dim_l:
            idx = i
            break

    if idx is None:
        if dim is None:
            dim = 0
        raise TypeError(f"tensor does not contain dimension {dim}")

    # Calculate split indices
    indices = []
    total_size = 0
    unbound = []

    for i, size_dim in enumerate(sizes):
        if size_dim.is_bound:
            indices.append(size_dim.size)
            total_size += indices[-1]
        else:
            indices.append(0)
            unbound.append(i)
```
- **EN**: It introduces or extends dimension, which hold the primary data model or public surface for this slice of the file. This chunk continues `dimension` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 它引入或扩展了 dimension，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `dimension`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 1299-1336
```python
    if self_info.tensor is None:
        raise AssertionError("Cannot get tensor size on None tensor")
    tensor_size = self_info.tensor.size(idx)

    # Handle unbound dimensions
    if unbound:
        if total_size > tensor_size:
            raise TypeError(
                f"sizes of target dimensions add up to more ({total_size}) than source dim ({tensor_size})"
            )
        remaining_size = tensor_size - total_size
        chunk_size = (remaining_size + len(unbound) - 1) // len(unbound)
        for u in unbound:
            sz = min(chunk_size, remaining_size)
            sizes[u].size = sz
            indices[u] = sz
            remaining_size -= sz
    elif tensor_size != total_size:
        raise TypeError(
            f"sum of sizes of target dimensions ({total_size}) do not match the source dim ({tensor_size})"
        )

    # Perform the split
    result_tensors = self_info.tensor.split_with_sizes(indices, idx)

    # Create result with new levels
    result = []
    new_levels = list(self_info.levels)

    for i, (result_tensor, size_dim) in enumerate(zip(result_tensors, sizes)):
        new_levels[idx] = DimEntry(size_dim)
        result.append(
            Tensor.from_positional(
                result_tensor, list(new_levels), self_info.has_device
            )
        )

    return tuple(result)
```
- **EN**: This chunk continues `dimension` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Branch dispatch chooses specialized behavior for different cases, node kinds, or runtime situations. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `dimension`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 分支分发会为不同情况、节点类型或运行时场景选择专门行为。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1339-1377
```python
def cat(tensors: Any, dim: Any, new_dim: Any) -> _Tensor:
    n = dims(1)  # Get single Dim instead of tuple
    return stack(tensors, n, dim).index([n, dim], new_dim)  # type: ignore[list-item]


class DotPart:
    """
    Helper class for organizing dimensions in dot products.
    """

    def __init__(self) -> None:
        self.dims: list[DimEntry] = []
        self.total_size = 1

    def append(self, dim_entry: Any) -> None:
        """Add a dimension entry to this part."""
        self.dims.append(dim_entry)
        if not dim_entry.is_positional():
            self.total_size *= dim_entry.dim().size


def dot_prepare(parts: list[DotPart], tensor_info: TensorInfo) -> torch.Tensor:
    """
    Prepare tensor for dot product by matching levels and reshaping.
    """
    new_levels = []
    needs_reshape = False

    for part in parts:
        if len(part.dims) != 1:
            needs_reshape = True
        new_levels.extend(part.dims)

    if tensor_info.tensor is None:
        raise RuntimeError("Cannot perform dot product on None tensor")
    result = _match_levels(tensor_info.tensor, tensor_info.levels, new_levels)

    if not needs_reshape:
        return result
```
- **EN**: It introduces or extends DotPart, for, which hold the primary data model or public surface for this slice of the file. This chunk defines `dot_prepare`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 它引入或扩展了 DotPart、for，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `dot_prepare`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1379-1413
```python
    # Reshape for matrix operations
    view = [part.total_size for part in parts]
    return result.reshape(view)


def dot_finish(parts: list[DotPart], result_tensor: torch.Tensor) -> Tensor:
    """
    Finish dot product by reshaping result and creating Tensor.
    """
    result_levels = []
    needs_reshape = False

    for part in parts:
        if len(part.dims) != 1:
            needs_reshape = True
        result_levels.extend(part.dims)

    if needs_reshape:
        new_size = []
        for level in result_levels:
            new_size.append(level.dim().size)
        result_tensor = result_tensor.reshape(new_size)

    tensor_result = Tensor.from_positional(result_tensor, result_levels, True)
    return tensor_result  # type: ignore[return-value]


def dot(lhs: Any, rhs: Any, sum_dims: Any) -> _Tensor | torch.Tensor:
    """
    Perform dot product between two tensors along specified dimensions.

    Args:
        lhs: Left-hand side tensor
        rhs: Right-hand side tensor
        sum_dims: Dimensions to sum over (contract)
```
- **EN**: This chunk defines `dot`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `dot`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1415-1452
```python
    Returns:
        Result of dot product
    """
    # Get tensor info
    lhs_info = TensorInfo.create(lhs, ensure_batched=False, ensure_present=False)
    rhs_info = TensorInfo.create(rhs, ensure_batched=False, ensure_present=False)

    if not (lhs_info and rhs_info):
        # Fall back to regular operations
        return torch.matmul(lhs, rhs)

    if lhs_info.tensor is None or rhs_info.tensor is None:
        raise AssertionError("Cannot perform dot product on None tensors")

    lhs_strides = lhs_info.tensor.stride()
    rhs_strides = rhs_info.tensor.stride()

    # Create dot parts for different dimension categories
    lro_dims = DotPart()  # Left-right-output (batch dims)
    lo_dims = DotPart()  # Left-output only
    ro_dims = DotPart()  # Right-output only
    lr_dims = DotPart()  # Left-right (contracted dims)

    def insert_dim(d: Any, lhs_idx: Any, rhs_idx: Any) -> None:
        """Insert dimension into appropriate part based on stride pattern."""
        reduced = d in sum_dims
        lhs_stride = lhs_strides[lhs_idx] if lhs_idx is not None else 0
        rhs_stride = rhs_strides[rhs_idx] if rhs_idx is not None else 0

        if reduced:
            lr_dims.append(d)
        else:
            if (lhs_stride == 0) == (rhs_stride == 0):
                lro_dims.append(d)  # Both have or both lack this dim
            elif lhs_stride != 0:
                lo_dims.append(d)  # Only lhs has this dim
            else:
                ro_dims.append(d)  # Only rhs has this dim
```
- **EN**: This chunk defines `insert_dim`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `insert_dim`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1454-1491
```python
    # Track which rhs dimensions we've seen
    rhs_seen = [False] * len(rhs_info.levels)

    # Process lhs dimensions
    for i, lhs_level in enumerate(lhs_info.levels):
        rhs_idx = None
        for j, rhs_level in enumerate(rhs_info.levels):
            if lhs_level == rhs_level:
                rhs_idx = j
                rhs_seen[j] = True
                break

        insert_dim(lhs_level, i, rhs_idx)

    # Process remaining rhs dimensions
    for i, rhs_level in enumerate(rhs_info.levels):
        if not rhs_seen[i]:
            insert_dim(rhs_level, None, i)

    # Validate sum dimensions exist
    if len(lr_dims.dims) != len(sum_dims):
        for d in sum_dims:
            if d not in lhs_info.levels and d not in rhs_info.levels:
                raise ValueError(f"summing over non-existent dimension {d}")

    # Prepare tensors and perform matrix multiplication
    if len(lro_dims.dims) != 0:
        # Batched matrix multiply
        lhs_tensor = dot_prepare([lro_dims, lo_dims, lr_dims], lhs_info)
        rhs_tensor = dot_prepare([lro_dims, lr_dims, ro_dims], rhs_info)
        result = torch.bmm(lhs_tensor, rhs_tensor)
        return dot_finish([lro_dims, lo_dims, ro_dims], result)
    else:
        # Regular matrix multiply
        lhs_tensor = dot_prepare([lo_dims, lr_dims], lhs_info)
        rhs_tensor = dot_prepare([lr_dims, ro_dims], rhs_info)
        result = torch.mm(lhs_tensor, rhs_tensor)
        return dot_finish([lo_dims, ro_dims], result)
```
- **EN**: This chunk continues `insert_dim` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `insert_dim`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1494-1530
```python
from functorch.dim._wrap import _wrap
from functorch.dim.wrap_type import wrap_type


wrap_type(_Tensor, torch.Tensor, _Tensor.__torch_function__)
del _Tensor.ndim


def index(self: Any, positions: Any, dims: Any) -> _Tensor:
    """
    Index a regular tensor by binding specified positions to dims.

    This converts a regular tensor to a first-class tensor by binding
    the specified positional dimensions to Dim objects.

    Args:
        positions: Tuple of dimension positions to bind
        dims: Dim objects or tuple of Dim objects to bind to

    Returns:
        First-class tensor with specified dimensions bound
    """
    # If this is already a first-class tensor (_Tensor), call its index method directly
    if isinstance(self, _Tensor):
        return _Tensor.index(self, positions, dims)

    # Convert regular tensor to first-class tensor
    info = TensorInfo.create(self, ensure_batched=False, ensure_present=False)

    # Create the first-class tensor
    if info.tensor is None:
        raise AssertionError("Cannot index None tensor")
    result = Tensor.from_positional(info.tensor, info.levels, info.has_device)

    # Now call the index method on the first-class tensor
    # Cast result to _Tensor for the method call
    return _Tensor.index(result, positions, dims)  # type: ignore[arg-type]
```
- **EN**: The import section wires together PyTorch-local modules such as functorch.dim._wrap, functorch.dim.wrap_type for the logic below. It introduces or extends tensor, which hold the primary data model or public surface for this slice of the file. This chunk defines `index`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 functorch.dim._wrap、functorch.dim.wrap_type组织在一起，供下方逻辑使用。 它引入或扩展了 tensor，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `index`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 1533-1572
```python
def _def(name: str, *args: Any, **kwargs: Any) -> None:
    orig = getattr(torch.Tensor, name)
    setattr(_Tensor, name, _wrap(orig, *args, **kwargs))


_def("mean")
_def("sum")
_def("all")
_def("amax")
_def("amin")
_def("aminmax")
_def("any")
_def("count_nonzero")
_def("logsumexp")
_def("nanmean")
_def("nansum")
_def("prod")
_def("std", keepdim_offset=2)
_def("var", keepdim_offset=2)
_def("max", single_dim=True)
_def("min", single_dim=True)
_def("argmax", single_dim=True)
_def("argmin", single_dim=True)
_def("kthvalue", single_dim=True)
_def("median", single_dim=True)
_def("nanmedian", single_dim=True)
_def("mode", single_dim=True)
_def("sort", reduce=False)
_def("argsort", reduce=False)
_def("unbind", single_dim=True)
_def("chunk", dim_offset=1, reduce=False)
_def("cummax", single_dim=True, reduce=False)
_def("cummin", single_dim=True, reduce=False)
_def("cumprod", single_dim=True, reduce=False)
_def("cumprod_", single_dim=True, reduce=False)
_def("cumsum", single_dim=True, reduce=False)
_def("cumsum_", single_dim=True, reduce=False)
_def("logcumsumexp", single_dim=True, reduce=False)
_def("renorm", dim_offset=1, single_dim=True, reduce=False)
_def("softmax", single_dim=True, reduce=False)
```
- **EN**: This chunk defines `_def`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `_def`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 1573-1599
```python
softmax = _wrap(torch.nn.functional.softmax, single_dim=True, reduce=False)

# stuff to handle in the future, because they require special
# binding logic for dims
# cross
# diag_embed
# diagonal
# diagonal_scatter
# diff
# nanquantile
# quantile
# roll
# rot90
# topk (new dimes on output)
# should these all be subsumed by inplace indexing?
# index_add_
# index_add
# index_copy
# index_copy_
# index_fill
# index_fill_
# index_select
# scatter
# scatter_
# scatter_add
# scatter_add_
# scatter_reduce
```
- **EN**: This chunk continues `_def` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `_def`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

## Key Concepts / 关键概念

- **Functional tensor transforms**
  - EN: Applies composable transforms such as batching, gradients, and tracing.
  - CN: 应用可组合的变换，例如 batching、梯度和 tracing。
- **POINTWISE_OPTIMIZE**
  - EN: `POINTWISE_OPTIMIZE` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `POINTWISE_OPTIMIZE` 是本文件声明、导出或驱动的显著符号之一。
- **DOT_OPTIMIZED**
  - EN: `DOT_OPTIMIZED` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `DOT_OPTIMIZED` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.utils._pytree`, `._dim_entry`, `._enable_all_layers`, `._py_inst_decoder`, `._tensor_info`, `.`, `functorch.dim._getsetitem`, `._getsetitem`, `._wrap`, `._order`, `functorch.dim._wrap`
- **Standard library / 标准库**: `__future__`, `dis`, `inspect`, `sys`, `typing`, `collections.abc`
- **Primary symbols / 核心符号**: `POINTWISE_OPTIMIZE`, `DOT_OPTIMIZED`, `_relevant_op`, `handle_from_tensor`, `_create_dim`, `dims`, `genobject`, `DimList`, `__init__`, `_set_dims`
- **Note / 说明**: 25 imports were detected; only the first few are listed for readability. / 检测到 25 个导入项，为便于阅读这里只展示前若干项。
