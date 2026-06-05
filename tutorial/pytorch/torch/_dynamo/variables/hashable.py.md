# hashable.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/variables/hashable.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines abstract variable trackers that model Python values during graph capture.
- **Purpose (CN)**: 定义抽象的变量跟踪器，用于在图捕获过程中建模 Python 值。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
"""
Hashability utilities for PyTorch Dynamo variable tracking.

This module provides the HashableTracker wrapper class and associated utilities
for making VariableTracker instances usable as dictionary keys and set elements
during symbolic execution. Used by both ConstDictVariable and SetVariable.
"""

from typing import TYPE_CHECKING

import torch

from .. import variables
from ..exc import raise_observed_exception
from ..utils import specialize_symnode
from .base import VariableTracker
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 19-34
```python
if TYPE_CHECKING:
    from torch._dynamo.symbolic_convert import InstructionTranslator


def raise_unhashable(
    arg: VariableTracker, tx: "InstructionTranslator | None" = None
) -> None:
    if tx is None:
        from torch._dynamo.symbolic_convert import InstructionTranslator

        tx = InstructionTranslator.current_tx()
    try:
        arg_type = arg.python_type()
    except Exception:
        arg_type = type(arg)
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 35-46
```python
    raise_observed_exception(
        TypeError,
        tx,
        args=[
            f"unhashable type: {arg_type!r} and variable tracker = {type(arg.realize())}",
        ],
    )


def is_hashable(x: VariableTracker) -> bool:
    # NB - performing isinstance check on a LazVT realizes the VT, accidentally
    # inserting the guard. To avoid this, lazyVT `is_hashable` methods looks at
```
- **EN**: This module-level block helps enforce guards that validate whether cached compiled code can be reused.
- **CN**: 这个模块级代码块用于实施守卫检查以判断缓存的编译代码能否复用。

### Lines 47-64
```python
    # the underlying value without realizing the VT. Consider updating the
    # lazyVT `is_hashable` method if you see unnecessary guarding for a key VT.
    if (
        isinstance(x, variables.LazyVariableTracker)
        and not x.is_realized()
        and x.is_hashable()
    ):
        return True
    return x.is_python_hashable()


class HashableTracker:
    """
    Class that wraps a VariableTracker and makes it hashable.
    Note that it's fine to put VTs into dictionaries and sets, but doing so
    does not take into account aliasing.
    """
```
- **EN**: This module-level block helps enforce guards that validate whether cached compiled code can be reused. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于实施守卫检查以判断缓存的编译代码能否复用。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 65-82
```python
    _MISSING = object()

    def __init__(self, vt: VariableTracker) -> None:
        # We specialize SymNodes
        vt = specialize_symnode(vt)

        # If Dynamo does not know the hashability of the vt, it will raise unsupported here
        # TODO(follow-up): check tp_hash via C-level slot detection — unhashable keys
        # (e.g. list) should raise TypeError, not graph break via is_python_hashable/unimplemented.
        if not is_hashable(vt):
            raise_unhashable(vt)
        self.vt = vt

    @classmethod
    def _maybe_constant_torch_size(cls, vt: VariableTracker) -> object:
        from .lists import SizeVariable
        from .tensor import TensorVariable
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 83-98
```python
        if (
            isinstance(vt, variables.LazyVariableTracker)
            and not vt.is_realized()
            and isinstance(vt.original_value(), torch.Size)
        ):
            return vt.original_value()

        if not isinstance(vt, SizeVariable):
            return cls._MISSING

        items = []
        for item in vt.items:
            if item.is_python_constant():
                items.append(item.as_python_constant())
                continue
```
- **EN**: This block continues `HashableTracker._maybe_constant_torch_size` and works to capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `HashableTracker._maybe_constant_torch_size`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 99-115
```python
            if isinstance(item, TensorVariable):
                proxy = getattr(item, "proxy", None)
                node = getattr(proxy, "node", None)
                meta = getattr(node, "meta", None) if node is not None else None
                example_value = (
                    meta.get("example_value") if isinstance(meta, dict) else None
                )
                constant = getattr(example_value, "constant", None)

                if isinstance(constant, torch.Tensor) and constant.numel() == 1:
                    items.append(constant.item())
                    continue

            return cls._MISSING

        return torch.Size(items)
```
- **EN**: This block continues `HashableTracker` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `HashableTracker`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 116-133
```python
    def __hash__(self) -> int:
        """
        Computes the hash value for the wrapped VariableTracker.

        For unrealized LazyVariableTrackers, uses the hash of the original value
        to avoid realizing the tracker and inserting unnecessary guards.
        For all other cases, delegates to the VariableTracker's get_python_hash method.

        Returns:
            The hash value of the underlying variable tracker
        """
        if (
            isinstance(self.vt, variables.LazyVariableTracker)
            and not self.vt.is_realized()
            and self.vt.is_hashable()
        ):
            return hash(self.vt.original_value())
```
- **EN**: Defines the `HashableTracker.__hash__` method; this block introduces logic that enforce guards that validate whether cached compiled code can be reused.
- **CN**: 定义`HashableTracker.__hash__` 方法；该代码块引入了用于实施守卫检查以判断缓存的编译代码能否复用的逻辑。

### Lines 134-149
```python
        maybe_constant = self._maybe_constant_torch_size(self.vt)
        if maybe_constant is not self._MISSING:
            return hash(maybe_constant)

        return self.vt.get_python_hash()

    def __eq__(self, other: object) -> bool:
        """
        Checks equality between two HashableTracker instances.

        Delegates to the VariableTracker's is_python_equal method to compare
        the underlying variable trackers for Python-level equality.

        Args:
            other: Another HashableTracker instance to compare with
```
- **EN**: Declares `HashableTracker`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `HashableTracker`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 150-162
```python
        Returns:
            True if the underlying variable trackers are Python-equal, False otherwise
        """
        if not isinstance(other, HashableTracker):
            return False
        if self.vt is other.vt:
            return True

        self_constant = self._maybe_constant_torch_size(self.vt)
        other_constant = self._maybe_constant_torch_size(other.vt)
        if self_constant is not self._MISSING and other_constant is not self._MISSING:
            return self_constant == other_constant
```
- **EN**: This block continues `HashableTracker.__eq__` and works to capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `HashableTracker.__eq__`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 163-163
```python
        return self.vt.is_python_equal(other.vt)
```
- **EN**: This block continues `HashableTracker.__eq__` and works to capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `HashableTracker.__eq__`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `..`, `..exc`, `..utils`, `.base`, `torch._dynamo.symbolic_convert`, `.lists`, `.tensor`
- **Standard library / 标准库**: `typing`
- **Primary symbols / 核心符号**: `raise_unhashable`, `is_hashable`, `HashableTracker`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
