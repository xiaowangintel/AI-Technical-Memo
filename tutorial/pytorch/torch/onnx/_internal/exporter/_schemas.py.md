# _schemas.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_schemas.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for schemas, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 schemas 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: allow-untyped-defs
"""Helpers for constructing ONNX operator signatures from Python functions."""

from __future__ import annotations

import collections.abc
import inspect
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `__future__`, `collections.abc`, `inspect`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`__future__`, `collections.abc`, `inspect`。

### Lines 8-14
```python
import logging
import types
import typing
from collections.abc import Sequence
from typing import Any, Optional, TypeVar, Union

from torch.onnx._internal._lazy_import import onnx_ir as ir, onnxscript
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal._lazy_import`; external imports: `logging`, `types`, `typing`, `collections.abc`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal._lazy_import`；外部导入：`logging`, `types`, `typing`, `collections.abc`。

### Lines 15-28
```python


logger = logging.getLogger(__name__)


# Map from python type to corresponding ONNX AttributeProto type
_PY_TYPE_TO_ATTR_TYPE = {
    float: ir.AttributeType.FLOAT,
    int: ir.AttributeType.INT,
    str: ir.AttributeType.STRING,
    bool: ir.AttributeType.INT,
    ir.Tensor: ir.AttributeType.TENSOR,
    ir.TensorProtocol: ir.AttributeType.TENSOR,
    ir.Graph: ir.AttributeType.GRAPH,
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 29-42
```python
    ir.GraphProtocol: ir.AttributeType.GRAPH,
}

# Map from python type to corresponding ONNX AttributeProto type,
# for repeated (i.e., list of) values
_LIST_TYPE_TO_ATTR_TYPE = {
    float: ir.AttributeType.FLOATS,
    int: ir.AttributeType.INTS,
    str: ir.AttributeType.STRINGS,
    bool: ir.AttributeType.INTS,
    ir.Tensor: ir.AttributeType.TENSORS,
    ir.TensorProtocol: ir.AttributeType.TENSORS,
    ir.Graph: ir.AttributeType.GRAPHS,
    ir.GraphProtocol: ir.AttributeType.GRAPHS,
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 43-50
```python
}

_ALL_VALUE_TYPES = (
    {ir.TensorType(dtype) for dtype in ir.DataType}
    | {ir.SequenceType(ir.TensorType(dtype)) for dtype in ir.DataType}
    | {ir.OptionalType(ir.TensorType(dtype)) for dtype in ir.DataType}
)

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 51-61
```python
# TypeAnnotationValue represents the (value of) valid type-annotations recognized
# by ONNX Script. Currently, it supports
# - float, int, str (primitive attribute types)
# - Sequence[float], Sequence[int], Sequence[str] (attribute types)
# - Tensor types
# - Sequence[Tensor] types
# - Union of above 2
# - TypeVars with above bounds
# - Above types with annotation attached
TypeAnnotationValue = Any

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 62-68
```python

def _is_optional(type_: type) -> bool:
    """Returns whether a type_ is an Optional."""
    origin_type = typing.get_origin(type_)
    if origin_type is Union and type(None) in typing.get_args(type_):
        # Python < 3.10
        return True
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_is_optional`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_is_optional`。

### Lines 69-78
```python
    if origin_type is Optional:
        # Python >= 3.10
        return True
    if (
        hasattr(types, "UnionType")
        and origin_type is types.UnionType
        and type(None) in typing.get_args(type_)
    ):
        # Python >= 3.10
        return True
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 79-85
```python
    return False


def _get_attr_type(type_: type) -> ir.AttributeType:
    """Obtain the type of the attribute from a Python class."""
    try:
        if type_ in _PY_TYPE_TO_ATTR_TYPE:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_attr_type`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_attr_type`。

### Lines 86-99
```python
            return _PY_TYPE_TO_ATTR_TYPE[type_]
        origin_type = typing.get_origin(type_)
        if origin_type is None:
            return ir.AttributeType.UNDEFINED
        if origin_type in (
            collections.abc.Sequence,
            Sequence,
            list,
            list,
            tuple,
            tuple,
        ):
            inner_type = typing.get_args(type_)[0]
            if inner_type in _LIST_TYPE_TO_ATTR_TYPE:
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 100-106
```python
                return _LIST_TYPE_TO_ATTR_TYPE[inner_type]
    except TypeError:
        logger.warning("TypeError when checking %s.", type_, exc_info=True)
    return ir.AttributeType.UNDEFINED


def _get_type_constraint_name(type_: TypeAnnotationValue) -> str | None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_type_constraint_name`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_type_constraint_name`。

### Lines 107-116
```python
    """Returns the name of the type constraint for a given type annotation.

    Args:
        type_: A Python type.

    Returns:
        The name of the type constraint if it is a TypeVar.
        - Prefixes the name with "Sequence_" if the type annotation is a Sequence[].
    """
    if isinstance(type_, TypeVar):
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 117-124
```python
        return type_.__name__
    if _is_optional(type_):
        subtypes = typing.get_args(type_)
        for subtype in subtypes:
            if subtype is type(None):
                continue
            type_param_name = _get_type_constraint_name(subtype)
            return type_param_name if type_param_name else None
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 125-131
```python
    origin_type = typing.get_origin(type_)
    if isinstance(origin_type, type) and issubclass(origin_type, Sequence):
        subtypes = typing.get_args(type_)
        type_param_name = _get_type_constraint_name(subtypes[0])
        return f"Sequence_{type_param_name}" if type_param_name else None
    return None

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 132-139
```python

def _get_allowed_types_from_type_annotation(
    type_: TypeAnnotationValue,
) -> set[ir.TypeProtocol]:
    """Obtain the allowed types from a type annotation."""
    if type_ is onnxscript.onnx_types.TensorType:
        # Any tensor type
        return {ir.TensorType(dtype) for dtype in ir.DataType}
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_allowed_types_from_type_annotation`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_allowed_types_from_type_annotation`。

### Lines 140-146
```python

    allowed_types: set[ir.TypeProtocol]

    if isinstance(type_, TypeVar):
        allowed_types = set()
        if constraints := type_.__constraints__:
            for constraint in constraints:
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 147-156
```python
                allowed_types.update(
                    _get_allowed_types_from_type_annotation(constraint)
                )
        else:
            bound = type_.__bound__
            if bound is None:
                allowed_types = _ALL_VALUE_TYPES  # type: ignore[assignment]
            else:
                allowed_types.update(_get_allowed_types_from_type_annotation(bound))
        return allowed_types
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 157-163
```python
    if hasattr(type_, "dtype"):
        # A single tensor type like INT64, FLOAT, etc.
        return {ir.TensorType(ir.DataType(type_.dtype))}
    if _is_optional(type_):
        allowed_types = set()
        subtypes = typing.get_args(type_)
        for subtype in subtypes:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 164-171
```python
            if subtype is type(None):
                continue
            allowed_types.update(_get_allowed_types_from_type_annotation(subtype))
        # NOTE: We do not consider dynamic optional types like optional(float) because they are not very useful.
        return allowed_types

    origin_type = typing.get_origin(type_)
    if origin_type is Union:
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 172-180
```python
        allowed_types = set()
        subtypes = typing.get_args(type_)
        for subtype in subtypes:
            if subtype is type(None):
                raise AssertionError(
                    "Union should not contain None type because it is handled by _is_optional."
                )
            allowed_types.update(_get_allowed_types_from_type_annotation(subtype))
        return allowed_types
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 181-188
```python

    if isinstance(origin_type, type) and issubclass(origin_type, Sequence):
        subtypes = typing.get_args(type_)
        return {
            ir.SequenceType(t)
            for t in _get_allowed_types_from_type_annotation(subtypes[0])
        }

```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 189-202
```python
    # Allow everything by default
    return _ALL_VALUE_TYPES  # type: ignore[return-value]


def op_signature_from_function(
    func,
    domain: str,
    name: str | None = None,
    overload: str = "",
    *,
    since_version: int = 1,
) -> ir.schemas.OpSignature:
    """Produce an OpSignature from a function using type annotation."""

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `op_signature_from_function`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`op_signature_from_function`。

### Lines 203-211
```python
    py_signature = inspect.signature(func)
    # Not using inspect.get_annotations because typing.get_type_hints seems to handle more cases
    # https://github.com/python/cpython/issues/102405
    type_hints = typing.get_type_hints(func)

    params: list[ir.schemas.Parameter | ir.schemas.AttributeParameter] = []
    # Create a mapping from type to a unique name
    type_constraints: dict[str, ir.schemas.TypeConstraintParam] = {}

```
- EN: This block implements local helper logic for schemas. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 schemas 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 212-224
```python
    for param in py_signature.parameters.values():
        if param.name not in type_hints:
            logger.debug(
                "Missing annotation for parameter '%s' from %s. Treating as an Input.",
                param.name,
                py_signature,
            )
            type_constraint = ir.schemas.TypeConstraintParam.any_value(
                f"T_{param.name}"
            )
            type_constraints[param.name] = type_constraint
            kwargs: dict[str, Any] = {}
            if param.default is not inspect.Parameter.empty:
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 225-238
```python
                kwargs["default"] = param.default
            params.append(
                ir.schemas.Parameter(
                    name=param.name,
                    type_constraint=type_constraint,
                    required=param.default is inspect.Parameter.empty,
                    # TODO: Handle variadic
                    variadic=False,
                    **kwargs,
                )
            )
        else:
            type_ = type_hints[param.name]
            if (attr_type := _get_attr_type(type_)) != ir.AttributeType.UNDEFINED:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 239-252
```python
                # Construct the default attribute
                if param.default is not inspect.Parameter.empty:
                    # TODO: Use ir_convenience instead to handle int as float
                    default = ir.Attr(param.name, attr_type, param.default)
                else:
                    default = None
                params.append(
                    ir.schemas.AttributeParameter(
                        name=param.name,
                        type=attr_type,
                        required=param.default is inspect.Parameter.empty,
                        default=default,
                    )
                )
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 253-260
```python
            else:
                # Obtain the type constraint from the type annotation

                # 1. Get a type constraint name from the type annotation
                # If the type annotation is a TypeVar or Optional[TypeVar], get its name
                # Otherwise, name it T_{param.name}
                type_constraint_name = _get_type_constraint_name(type_)
                if type_constraint_name is None:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 261-274
```python
                    type_constraint_name = f"T_{param.name}"

                # 2. If the type constraint param is already initialized, use it
                if type_constraint_name in type_constraints:
                    type_constraint = type_constraints[type_constraint_name]
                else:
                    # 3. Otherwise, create a new TypeConstraintParam
                    type_constraint = ir.schemas.TypeConstraintParam(
                        name=type_constraint_name,
                        allowed_types=_get_allowed_types_from_type_annotation(type_),
                    )
                    type_constraints[type_constraint_name] = type_constraint
                # 4. Create Parameter
                kwargs: dict[str, Any] = {}
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 275-287
```python
                if param.default is not inspect.Parameter.empty:
                    kwargs["default"] = param.default
                params.append(
                    ir.schemas.Parameter(
                        name=param.name,
                        type_constraint=type_constraint,
                        required=param.default is inspect.Parameter.empty,
                        # TODO: Handle variadic
                        variadic=False,
                        **kwargs,
                    )
                )

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 288-295
```python
    return_type = type_hints.get("return")

    outputs = []
    if return_type is None:
        # No returns
        pass
    else:
        if typing.get_origin(return_type) is tuple:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 296-302
```python
            # Multiple returns
            return_types = typing.get_args(return_type)
        else:
            return_types = [return_type]  # type: ignore[assignment]

        for i, return_type_i in enumerate(return_types):
            if (
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 303-316
```python
                return_param_name := _get_type_constraint_name(return_type_i)
            ) in type_constraints:
                # pyrefly: ignore [bad-index]
                type_constraint = type_constraints[return_param_name]
            else:
                return_param_name = f"TReturn{i}"
                type_constraint = ir.schemas.TypeConstraintParam(
                    name=return_param_name,
                    allowed_types=_get_allowed_types_from_type_annotation(
                        return_type_i
                    ),
                )
                type_constraints[return_param_name] = type_constraint
            outputs.append(
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 317-324
```python
                ir.schemas.Parameter(
                    name=return_param_name,
                    type_constraint=type_constraint,
                    required=True,
                    variadic=False,
                )
            )

```
- EN: This block implements local helper logic for schemas. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 schemas 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 325-332
```python
    return ir.schemas.OpSignature(
        domain=domain,
        name=name or func.__name__,
        overload=overload,
        params=params,
        outputs=outputs,
        since_version=since_version,
    )
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch.onnx._internal._lazy_import`
- External imports / 外部导入: `__future__`, `collections.abc`, `inspect`, `logging`, `types`, `typing`
- Representative symbols / 代表性符号: `_PY_TYPE_TO_ATTR_TYPE`, `_LIST_TYPE_TO_ATTR_TYPE`, `_ALL_VALUE_TYPES`, `_is_optional`, `_get_attr_type`, `_get_type_constraint_name`, `_get_allowed_types_from_type_annotation`, `op_signature_from_function`
