# _ir_passes.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_ir_passes.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for ir passes, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 ir passes 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
# mypy: allow-untyped-defs
from __future__ import annotations

import logging
import re
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `__future__`, `logging`, `re`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`__future__`, `logging`, `re`。

### Lines 6-10
```python
from typing import TYPE_CHECKING

from torch.onnx._internal._lazy_import import onnx_ir as ir
from torch.onnx._internal.exporter import _constants

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`; external imports: `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`；外部导入：`typing`。

### Lines 11-15
```python

if TYPE_CHECKING:
    from collections.abc import Sequence


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `collections.abc`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`collections.abc`。

### Lines 16-20
```python
# The opset domain for ONNX operators
_ONNX_DOMAIN = ""

logger = logging.getLogger(__name__)

```
- EN: This block advances ONNX export translation; iterates over collections or execution units. Key symbols: `_ONNX_DOMAIN`.
- CN: 该代码块推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：`_ONNX_DOMAIN`。

### Lines 21-27
```python

def rename_inputs(model: ir.Model, new_names: Sequence[str]) -> None:
    unique_names = frozenset(new_names)
    if len(unique_names) != len(new_names):
        seen = set()
        duplicates = []
        for name in new_names:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rename_inputs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rename_inputs`。

### Lines 28-32
```python
            if name in seen:
                duplicates.append(name)
            seen.add(name)
        raise ValueError(f"Input names cannot be duplicated: {duplicates}")

```
- EN: This block reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 33-37
```python
    for input, new_name in zip(model.graph.inputs, new_names):
        input.metadata_props["pkg.torch.onnx.original_node_name"] = str(input.name)
        input.name = new_name


```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 38-43
```python
def rename_outputs(model: ir.Model, new_names: Sequence[str]) -> None:
    unique_names = frozenset(new_names)
    if len(unique_names) != len(new_names):
        seen = set()
        duplicates = []
        for name in new_names:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rename_outputs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rename_outputs`。

### Lines 44-48
```python
            if name in seen:
                duplicates.append(name)
            seen.add(name)
        raise ValueError(f"Output names cannot be duplicated: {duplicates}")

```
- EN: This block reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 49-53
```python
    for output, new_name in zip(model.graph.outputs, new_names):
        output.metadata_props["pkg.torch.onnx.original_node_name"] = str(output.name)
        output.name = new_name


```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 54-59
```python
def _all_values(model: ir.Model):
    """Yield all values in a model."""
    # Yield all values in the model
    yield from model.graph.inputs
    yield from model.graph.initializers.values()
    for node in ir.traversal.RecursiveGraphIterator(model.graph):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_all_values`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_all_values`。

### Lines 60-64
```python
        yield from node.outputs
    # Yield all values in functions
    for function in model.functions.values():
        yield from function.inputs
        for node in ir.traversal.RecursiveGraphIterator(function):
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 65-70
```python
            yield from node.outputs


def _replace_names(shape_expr: str, rename_mapping: dict[str, str]) -> str:
    """Replace all known names in a shape expression with new names."""
    for old_name, new_name in rename_mapping.items():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_replace_names`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_replace_names`。

### Lines 71-75
```python
        shape_expr = re.sub(
            rf"(?<!\w){re.escape(old_name)}(?!\w)", new_name, shape_expr
        )
    return shape_expr

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 76-81
```python

def rename_axis(
    model: ir.Model, rename_mapping: dict[str | ir.SymbolicDim, str]
) -> None:
    """Rename dynamic axes in a model according to the specified dynamic_axes names."""

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rename_axis`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rename_axis`。

### Lines 82-86
```python
    # Create a mapping from string to string for easier replacement
    string_mapping: dict[str, str] = {}
    for key, value in tuple(rename_mapping.items()):
        if isinstance(key, ir.SymbolicDim):
            if isinstance(key.value, str):
```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 87-96
```python
                string_mapping[key.value] = value
            else:
                raise ValueError(
                    f"Invalid SymbolicDim value in rename_mapping: {key.value!r}. "
                    "Expected str."
                )
        elif isinstance(key, str):
            string_mapping[key] = value
        else:
            raise ValueError(
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 97-106
```python
                f"Invalid key type in rename_mapping: {type(key)}({key!r}). Expected "
                "str or ir.SymbolicDim."
            )

    # NOTE: Mapping needs to be sorted by length because the shape expression
    # could have multiple ways to be expressed, for example,
    # {"s1": sequence_length, "s11": "past_sequence_length", "s1 + s11": "masked_sequence_length"}
    # We prefer the replacement starts from the longest match.
    sorted_rename_mapping = dict(
        sorted(string_mapping.items(), key=lambda item: len(item[0]), reverse=True)
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 107-113
```python
    )
    for value in _all_values(model):
        if value.shape is None:
            continue
        new_shape = []
        changed = False
        for dim in value.shape:
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 114-118
```python
            if not isinstance(dim, ir.SymbolicDim):
                new_shape.append(dim)
                continue
            dim_name = dim.value
            if dim_name in sorted_rename_mapping:
```
- EN: This block advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 119-125
```python
                new_shape.append(sorted_rename_mapping[dim_name])
                changed = True
            elif dim_name is not None:
                # For example: "2*s1", "s1+1", "s1-1", "s1*s2", "s1/s2"
                new_name = _replace_names(dim_name, sorted_rename_mapping)
                new_shape.append(new_name)
                if new_name != dim_name:
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 126-131
```python
                    changed = True
            else:
                new_shape.append(None)
        if changed:
            value.shape = ir.Shape(new_shape)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 132-137
```python

def _maybe_set_opset_version(
    opset_imports: dict[str, int], domain: str, version: int | None
) -> None:
    """Set the opset version for the domain."""
    if domain in opset_imports and opset_imports[domain] != 1:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_maybe_set_opset_version`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_maybe_set_opset_version`。

### Lines 138-143
```python
        # Already set
        return
    if domain == _ONNX_DOMAIN:
        opset_imports[domain] = _constants.TORCHLIB_OPSET
        return
    if version is None:
```
- EN: This block advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 144-150
```python
        # We don't know the opset version, so set it to 1
        # This is valid for the custom function domains like "pkg.torch.__subgraph__"
        opset_imports[domain] = 1
        return
    # Set the known opset version for the domain
    opset_imports[domain] = version

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 151-157
```python

def add_opset_imports(model: ir.Model) -> None:
    """Collect all opsets used and add opset imports to the model and functions."""
    for node in ir.traversal.RecursiveGraphIterator(model.graph):
        domain = node.domain
        _maybe_set_opset_version(model.opset_imports, domain, node.version)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `add_opset_imports`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`add_opset_imports`。

### Lines 158-162
```python
    for function in model.functions.values():
        for node in ir.traversal.RecursiveGraphIterator(function):
            domain = node.domain
            _maybe_set_opset_version(function.opset_imports, domain, node.version)
        for domain, version in function.opset_imports.items():
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 163-166
```python
            # Add all opsets used in the function to the model, because ONNX Runtime
            # does not handle adding the opset imports to the model after inlining during inference.
            # This should happen after all opsets are collected for the function from its nodes.
            _maybe_set_opset_version(model.opset_imports, domain, version)
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`
- External imports / 外部导入: `__future__`, `logging`, `re`, `typing`, `collections.abc`
- Representative symbols / 代表性符号: `_ONNX_DOMAIN`, `rename_inputs`, `rename_outputs`, `_all_values`, `_replace_names`, `rename_axis`, `_maybe_set_opset_version`, `add_opset_imports`
