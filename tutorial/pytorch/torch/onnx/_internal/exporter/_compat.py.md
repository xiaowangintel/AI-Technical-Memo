# _compat.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_compat.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for compat, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 compat 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
"""Compatibility functions for the torch.onnx.export API."""

# mypy: allow-untyped-defs
# mypy: disable-error-code=attr-defined
from __future__ import annotations

import io
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 8-14
```python
import logging
import warnings
from collections.abc import Callable, Mapping, Sequence
from typing import Any, TYPE_CHECKING

import torch
from torch.onnx import _constants as onnx_constants
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.onnx`; external imports: `logging`, `warnings`, `collections.abc`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.onnx`；外部导入：`logging`, `warnings`, `collections.abc`, `typing`。

### Lines 15-24
```python
from torch.onnx._internal._lazy_import import onnx
from torch.onnx._internal.exporter import (
    _constants,
    _core,
    _dynamic_shapes,
    _exportable_module,
    _onnx_program,
    _registration,
)

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 25-31
```python

if TYPE_CHECKING:
    import os

logger = logging.getLogger(__name__)


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `os`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`os`。

### Lines 32-40
```python
def _get_torch_export_args(
    args: tuple[Any, ...],
    kwargs: dict[str, Any] | None,
) -> tuple[tuple[Any, ...], dict[str, Any] | None]:
    """Obtain the arguments for torch.onnx.export from the model and the input arguments."""
    if not kwargs and args and isinstance(args[-1], dict):
        kwargs = args[-1]
        args = args[:-1]
    return args, kwargs
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_torch_export_args`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_torch_export_args`。

### Lines 41-54
```python


def export_compat(
    model: torch.nn.Module
    | torch.export.ExportedProgram
    | torch.jit.ScriptModule
    | torch.jit.ScriptFunction,
    args: tuple[Any, ...],
    f: str | os.PathLike | None = None,
    *,
    kwargs: dict[str, Any] | None = None,
    export_params: bool = True,
    verbose: bool | None = None,
    input_names: Sequence[str] | None = None,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `export_compat`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`export_compat`。

### Lines 55-68
```python
    output_names: Sequence[str] | None = None,
    opset_version: int | None = onnx_constants.ONNX_DEFAULT_OPSET,
    custom_translation_table: dict[Callable, Callable] | None = None,
    dynamic_axes: Mapping[str, Mapping[int, str]]
    | Mapping[str, Sequence[int]]
    | None = None,
    dynamic_shapes: dict[str, Any] | tuple[Any, ...] | list[Any] | None = None,
    keep_initializers_as_inputs: bool = False,
    external_data: bool = True,
    report: bool = False,
    optimize: bool = True,
    verify: bool = False,
    profile: bool = False,
    dump_exported_program: bool = False,
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 69-75
```python
    artifacts_dir: str | os.PathLike = ".",
) -> _onnx_program.ONNXProgram:
    if opset_version is None:
        opset_version = onnx_constants.ONNX_DEFAULT_OPSET

    if isinstance(model, torch.nn.Module):
        if model.training:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 76-84
```python
            warnings.warn(
                "Exporting a model while it is in training mode. "
                "Please ensure that this is intended, as it may lead to "
                "different behavior during inference. "
                "Calling model.eval() before export is recommended.",
                UserWarning,
                stacklevel=3,
            )

```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 85-91
```python
    if isinstance(model, _exportable_module.ExportableModule):
        # Skip argument extraction if args or kwargs are provided
        if not args and not kwargs:
            args, kwargs = model.example_arguments()
            if input_names is None:
                input_names = model.input_names()
            if output_names is None:
```
- EN: This block advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 92-102
```python
                output_names = model.output_names()
            if dynamic_shapes is None:
                dynamic_shapes = model.dynamic_shapes()

    if isinstance(model, torch.export.ExportedProgram):
        # We know the model is already exported program, so the args, kwargs, and dynamic_shapes
        # are not used
        dynamic_shapes = dynamic_shapes or {}
    else:
        args, kwargs = _get_torch_export_args(args, kwargs)
        if dynamic_shapes is None and dynamic_axes is not None:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 103-116
```python
            warnings.warn(
                "# 'dynamic_axes' is not recommended when dynamo=True, "
                "and may lead to 'torch._dynamo.exc.UserError: Constraints violated.' "
                "Supply the 'dynamic_shapes' argument instead if export is unsuccessful.",
                UserWarning,
                stacklevel=3,
            )
            try:
                dynamic_shapes, args, kwargs = (
                    _dynamic_shapes.from_dynamic_axes_to_dynamic_shapes(
                        model,
                        args,
                        kwargs,
                        dynamic_axes=dynamic_axes,
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 117-127
```python
                        input_names=input_names,
                        output_names=set(output_names or ()),
                    )
                )
            except Exception as e:
                raise RuntimeError(
                    "# Failed to convert 'dynamic_axes' to 'dynamic_shapes'. "
                    "Please provide 'dynamic_shapes' directly. "
                    "Refer to the documentation for 'torch.export.export' for more information on dynamic shapes."
                ) from e

```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 128-141
```python
    dynamic_shapes_with_export_dim, need_axis_mapping = (
        _dynamic_shapes.convert_str_to_export_dim(dynamic_shapes)
    )

    if opset_version < _constants.TORCHLIB_OPSET:
        logger.warning(
            "Setting ONNX exporter to use operator set version %s because "
            "the requested opset_version %s is a lower version than we have implementations for. "
            "Automatic version conversion will be performed, which may not be successful "
            "at converting to the requested version. If version conversion is unsuccessful, "
            "the opset version of the exported model will be kept at %s. "
            "Please consider setting opset_version >=%s to leverage latest ONNX features",
            _constants.TORCHLIB_OPSET,
            opset_version,
```
- EN: This block advances ONNX export translation; bridges to backend-specific execution artifacts; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；桥接到特定后端的执行产物；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 142-148
```python
            _constants.TORCHLIB_OPSET,
            _constants.TORCHLIB_OPSET,
        )
        registry_opset_version = _constants.TORCHLIB_OPSET
    else:
        registry_opset_version = opset_version

```
- EN: This block advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 149-155
```python
    registry = _registration.ONNXRegistry().from_torchlib(
        opset_version=registry_opset_version
    )
    if custom_translation_table is not None:
        for torch_op, onnx_op in custom_translation_table.items():
            # TODO(justinchuby): Support complex inputs with annotations
            if isinstance(onnx_op, Sequence):
```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 156-169
```python
                raise TypeError(
                    "The value in custom_translation_table should be a single callable, not a sequence"
                )
            registry.register_op(torch_op, onnx_op, is_complex=False)

    onnx_program = _core.export(
        model,
        args,
        kwargs,
        registry=registry,
        dynamic_shapes=dynamic_shapes_with_export_dim,
        input_names=input_names,
        output_names=output_names,
        profile=profile,
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 170-178
```python
        report=report,
        verify=verify,
        dump_exported_program=dump_exported_program,
        artifacts_dir=artifacts_dir,
        verbose=verbose,
        optimize=optimize,
        opset_version=opset_version,
    )

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 179-192
```python
    if need_axis_mapping and dynamic_shapes is not None:
        onnx_program._rename_dynamic_axes(dynamic_shapes)

    if f is not None:
        if isinstance(f, io.BytesIO):
            # For legacy export compatibility, we allow f to be a BytesIO object.
            # This is not explicitly supported but we may need to maintain the
            # behavior indefinitely.
            warnings.warn(
                "Saving ONNX model to a BytesIO object is deprecated. "
                "Please use a file path instead.",
                DeprecationWarning,
                stacklevel=2,
            )
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 193-201
```python
            onnx.save(onnx_program.model_proto, f)
        else:
            onnx_program.save(
                f,
                include_initializers=export_params,
                keep_initializers_as_inputs=keep_initializers_as_inputs,
                external_data=external_data,
            )

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 202-202
```python
    return onnx_program
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.onnx`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`
- External imports / 外部导入: `__future__`, `io`, `logging`, `warnings`, `collections.abc`, `typing`, `os`
- Representative symbols / 代表性符号: `_get_torch_export_args`, `export_compat`
