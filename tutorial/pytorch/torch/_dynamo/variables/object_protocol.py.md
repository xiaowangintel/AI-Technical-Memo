# object_protocol.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/variables/object_protocol.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines abstract variable trackers that model Python values during graph capture.
- **Purpose (CN)**: 定义抽象的变量跟踪器，用于在图捕获过程中建模 Python 值。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```python
"""
Dynamo implementations of CPython's PyObject_* default slot algorithms.

Analogous to CPython's Objects/object.c, this module holds the general
dispatch machinery that is independent of any specific type.
Per-type hook implementations (bool_impl, richcompare_impl, etc.)
live in their respective VT files.
"""

from functools import lru_cache
from typing import TYPE_CHECKING

from torch._C._dynamo import (
    get_type_slots,
    has_slot,
    PyMappingSlots,
    PyNumberSlots,
    PySequenceSlots,
)
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 21-43
```python
from .. import graph_break_hints
from ..exc import (
    handle_observed_exception,
    ObservedTypeError,
    raise_observed_exception,
    raise_type_error,
    unimplemented,
)
from ..utils import istype
from .base import NO_SUCH_SUBOBJ, VariableTracker
from .constant import ConstantVariable


if TYPE_CHECKING:
    from ..symbolic_convert import InstructionTranslator


def vt_identity_compare(
    left: VariableTracker,
    right: VariableTracker,
) -> "VariableTracker | None":
    """Try to determine Python identity (left is right) at trace time.
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 44-66
```python
    Returns ConstantVariable(True/False) if determinable, else None.
    Mirrors the logic in BuiltinVariable's handle_is handler.
    """
    if left is right:
        return ConstantVariable.create(True)

    left_val = left.get_real_python_backed_value()
    right_val = right.get_real_python_backed_value()
    left_known = left_val is not NO_SUCH_SUBOBJ
    right_known = right_val is not NO_SUCH_SUBOBJ

    if left_known and right_known:
        return (
            ConstantVariable.create(True)
            if left_val is right_val
            else ConstantVariable.create(False)
        )

    # One side has a concrete backing object, the other doesn't — they can't
    # be the same object.
    if left_known != right_known:
        return ConstantVariable.create(False)
```
- **EN**: This block continues `vt_identity_compare` and works to capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `vt_identity_compare`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 67-93
```python
    # Mutable containers created during tracing: VT identity = Python identity.
    from .dicts import ConstDictVariable
    from .lists import ListVariable
    from .sets import SetVariable

    if isinstance(left, (ConstDictVariable, ListVariable, SetVariable)):
        return ConstantVariable.create(False)

    # Different Python types can never be the same object.
    try:
        if left.python_type() is not right.python_type():
            return ConstantVariable.create(False)
    except NotImplementedError:
        pass

    # Different exception types are never identical.
    from .. import variables

    if (
        istype(left, variables.ExceptionVariable)
        and istype(right, variables.ExceptionVariable)
        and left.exc_type is not right.exc_type  # type: ignore[attr-defined]
    ):
        return ConstantVariable.create(False)

    return None
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 94-118
```python

@lru_cache(maxsize=256)
def _get_cached_slots(obj_type: type) -> tuple[int, int, int, int]:
    """Get all type slots for a type (cached)."""
    return get_type_slots(obj_type)


def type_implements_sq_length(obj_type: type) -> bool:
    """Check whether obj_type implements __len__ as sequence protocol"""
    seq_slots, _, _, _ = _get_cached_slots(obj_type)
    return has_slot(seq_slots, PySequenceSlots.SQ_LENGTH)


def type_implements_mp_length(obj_type: type) -> bool:
    """Check whether obj_type implements __len__ as mapping protocol"""
    _, map_slots, _, _ = _get_cached_slots(obj_type)
    return has_slot(map_slots, PyMappingSlots.MP_LENGTH)


def type_implements_nb_bool(obj_type: type) -> bool:
    """Check whether obj_type implements the nb_bool slot (i.e. has __bool__ or __len__)."""
    _, _, number_slots, _ = _get_cached_slots(obj_type)
    return has_slot(number_slots, PyNumberSlots.NB_BOOL)
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 119-136
```python
def type_implements_nb_int(obj_type: type) -> bool:
    """Check whether obj_type implements the nb_int slot."""
    _, _, number_slots, _ = _get_cached_slots(obj_type)
    return has_slot(number_slots, PyNumberSlots.NB_INT)


def type_implements_nb_index(obj_type: type) -> bool:
    """Check whether obj_type implements the nb_index slot."""
    _, _, number_slots, _ = _get_cached_slots(obj_type)
    return has_slot(number_slots, PyNumberSlots.NB_INDEX)


def type_implements_nb_float(obj_type: type) -> bool:
    """Check whether obj_type implements the nb_float slot."""
    _, _, number_slots, _ = _get_cached_slots(obj_type)
    return has_slot(number_slots, PyNumberSlots.NB_FLOAT)
```
- **EN**: This module-level block helps capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果。

### Lines 137-162
```python
def maybe_get_python_type(obj: VariableTracker) -> type:
    try:
        return obj.python_type()
    except NotImplementedError:
        unimplemented(
            gb_type="Unsupported python_type() call",
            context=f"{obj} does not implement python_type()",
            explanation="This VariableTracker does not implement python_type(), "
            "which is required for object protocol operations.",
            hints=[
                *graph_break_hints.DYNAMO_BUG,
            ],
        )


def vt_mapping_size(
    tx: "InstructionTranslator", obj: "VariableTracker"
) -> "VariableTracker":
    # ref: https://github.com/python/cpython/blob/v3.13.3/Objects/abstract.c#L2308-L2330
    T = maybe_get_python_type(obj)
    if type_implements_mp_length(T):
        return obj.mp_length(tx)

    if type_implements_sq_length(T):
        raise_type_error(tx, f"{obj.python_type_name()} is not a mapping")
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 163-189
```python
    raise_type_error(tx, f"object of type {obj.python_type_name()} has no len()")


def generic_len(
    tx: "InstructionTranslator", obj: "VariableTracker"
) -> "VariableTracker":
    # ref: https://github.com/python/cpython/blob/v3.13.3/Objects/abstract.c#L53-L69
    """
    Implements PyObject_Size/PyObject_Length semantics for VariableTracker objects.
    Dispatches to sq_length (sequences) or mp_length (mappings) depending on the VT type.
    """

    T = maybe_get_python_type(obj)
    if type_implements_sq_length(T):
        return obj.sq_length(tx)
    return vt_mapping_size(tx, obj)


def generic_bool(tx: "InstructionTranslator", obj: VariableTracker) -> VariableTracker:
    """Mirrors PyObject_IsTrue.

    https://github.com/python/cpython/blob/c09ccd9c429/Objects/object.c#L2135-L2158

    Resolution order: constants → nb_bool → mp_length/sq_length → truthy.
    """
    from .constant import ConstantVariable
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 190-212
```python
    if obj.is_python_constant():
        return ConstantVariable.create(bool(obj.as_python_constant()))

    obj_type = maybe_get_python_type(obj)

    if type_implements_nb_bool(obj_type):
        result = obj.bool_impl(tx)
        if result is not None:
            return result

    try:
        length = generic_len(tx, obj)
        from .tensor import SymNodeVariable

        if isinstance(length, SymNodeVariable):
            return SymNodeVariable.create(tx, length.as_proxy() > 0)
        return ConstantVariable.create(length.as_python_constant() > 0)
    except ObservedTypeError:
        handle_observed_exception(tx)

    return ConstantVariable.create(True)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 213-235
```python
def vt_getitem(
    tx: "InstructionTranslator",
    obj: VariableTracker,
    key: VariableTracker,
) -> VariableTracker:
    """CPython's PyObject_GetItem — dispatch to the type's mp_subscript/sq_item.

    PyObject_GetItem: https://github.com/python/cpython/blob/62a6e898e01/Objects/abstract.c#L155-L206

    CPython checks three branches in order:
      1. tp_as_mapping->mp_subscript  (L161-166)
      2. tp_as_sequence->sq_item      (L168-181) — only if key passes _PyIndex_Check
      3. PyType_Check(o)              (L183-203) — type[int] → GenericAlias/__class_getitem__

    Branch 1 is the common path (list, tuple, dict, range all have mp_subscript).
    TODO(follow-up): use has_slot(map_slots, PyMappingSlots.MP_SUBSCRIPT) to gate
    Branch 1 and has_slot(seq_slots, PySequenceSlots.SQ_ITEM) to gate Branch 2,
    matching CPython's dispatch order.
    TODO(follow-up): Branch 2 (sq_item) for C extension types that only have
    tp_as_sequence (e.g. deque — Modules/_collectionsmodule.c:1888).
    Branch 3 is handled by TypingVariable.mp_subscript_impl for typing module types
    and by BuiltinVariable for builtin types like list[int].
```
- **EN**: Defines the `vt_getitem` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`vt_getitem` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 236-262
```python
    Types that work via constant fold fallback (no dedicated mp_subscript_impl):
    TODO(follow-up): str (unicode_subscript, Objects/unicodeobject.c:13809)
    TODO(follow-up): bytes (bytes_subscript, Objects/bytesobject.c)
    """
    return obj.mp_subscript_impl(tx, key)


def generic_int(tx: "InstructionTranslator", obj: VariableTracker) -> VariableTracker:
    """Mirrors PyNumber_Long (int(x) dispatch).

    https://github.com/python/cpython/blob/v3.13.0/Objects/abstract.c#L1520-L1632

    Resolution: nb_int → nb_index → str/bytes/bytearray parsing → TypeError.
    """
    from .constant import ConstantVariable

    # Fast path for int (sub)class instances — mirrors PyLong_Check at the
    # top of PyNumber_Long (abstract.c:1531). Avoids infinite recursion for
    # int subclasses like IntEnum whose __int__ calls int() again.
    if obj.is_python_constant() and isinstance(obj.as_python_constant(), int):
        return ConstantVariable.create(int(obj.as_python_constant()))

    obj_type = maybe_get_python_type(obj)

    if type_implements_nb_int(obj_type):
        return obj.nb_int_impl(tx)
```
- **EN**: This module-level block helps initialize learnable tensors and related state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于初始化可学习张量及相关状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 263-287
```python
    if type_implements_nb_index(obj_type):
        return obj.nb_index_impl(tx)

    # String/bytes/bytearray parsing fallback.
    # https://github.com/python/cpython/blob/v3.13.0/Objects/abstract.c#L1598-L1612
    if obj.is_python_constant() and isinstance(
        obj.as_python_constant(), (str, bytes, bytearray)
    ):
        try:
            return ConstantVariable.create(int(obj.as_python_constant()))
        except ValueError as e:
            raise_observed_exception(ValueError, tx, args=[str(e)])

    raise_type_error(
        tx,
        f"int() argument must be a string, a bytes-like object "
        f"or a real number, not '{obj.python_type_name()}'",
    )


def generic_float(tx: "InstructionTranslator", obj: VariableTracker) -> VariableTracker:
    """Mirrors PyNumber_Float (float(x) dispatch).

    https://github.com/python/cpython/blob/v3.13.0/Objects/abstract.c#L1635-L1692
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 288-314
```python
    Resolution: nb_float → nb_index → str parsing → TypeError.
    """
    from .constant import ConstantVariable

    # Fast path: if the value is already a float constant, return it directly.
    # Mirrors PyFloat_CheckExact fast path at the top of PyNumber_Float
    # (abstract.c:1641-1643).
    if obj.is_python_constant() and isinstance(obj.as_python_constant(), float):
        return ConstantVariable.create(float(obj.as_python_constant()))

    obj_type = maybe_get_python_type(obj)

    if type_implements_nb_float(obj_type):
        return obj.nb_float_impl(tx)

    # https://github.com/python/cpython/blob/v3.13.0/Objects/abstract.c#L1674-L1685
    if type_implements_nb_index(obj_type):
        return obj.nb_index_impl(tx)

    # PyFloat_FromString fallback — handles str and bytes.
    # https://github.com/python/cpython/blob/v3.13.0/Objects/abstract.c#L1691
    if obj.is_python_constant() and isinstance(obj.as_python_constant(), (str, bytes)):
        try:
            return ConstantVariable.create(float(obj.as_python_constant()))
        except ValueError as e:
            raise_observed_exception(ValueError, tx, args=[str(e)])
```
- **EN**: This block continues `generic_float` and works to validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `generic_float`，用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 315-319
```python
    raise_type_error(
        tx,
        f"float() argument must be a string or a real number, "
        f"not '{obj.python_type_name()}'",
    )
```
- **EN**: This block continues `generic_float` and works to validate invariants and surface meaningful failures.
- **CN**: 该代码块继续实现 `generic_float`，用于校验不变量并给出有意义的失败信息。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
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
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch._C._dynamo`, `..`, `..exc`, `..utils`, `.base`, `.constant`, `..symbolic_convert`, `.dicts`, `.lists`, `.sets`, `.tensor`
- **Standard library / 标准库**: `functools`, `typing`
- **Primary symbols / 核心符号**: `vt_identity_compare`, `_get_cached_slots`, `type_implements_sq_length`, `type_implements_mp_length`, `type_implements_nb_bool`, `type_implements_nb_int`, `type_implements_nb_index`, `type_implements_nb_float`, `maybe_get_python_type`, `vt_mapping_size`, `generic_len`, `generic_bool`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
