# __init__.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/__init__.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Provides public ONNX export APIs and compatibility helpers for init.
- 用途 (CN): 为 init 提供公开的 ONNX 导出 API 与兼容性辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
```python
# mypy: allow-untyped-defs
from __future__ import annotations


__all__ = [
    # Modules
    "errors",
    "ops",
    # Public functions
    "export",
    "is_in_onnx_export",
    # Base error
    "OnnxExporterError",
    "ONNXProgram",
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 15-21
```python
    "ExportableModule",
    "InputObserver",
]

from typing import Any, TYPE_CHECKING

import torch
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 22-28
```python
from torch._C import _onnx as _C_onnx
from torch._C._onnx import (  # Deprecated members that are excluded from __all__
    OperatorExportTypes as OperatorExportTypes,
    TensorProtoDataType as TensorProtoDataType,
    TrainingMode as TrainingMode,
)

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 29-39
```python
from . import errors, ops
from ._internal.exporter._exportable_module import ExportableModule
from ._internal.exporter._input_observer import InputObserver
from ._internal.exporter._onnx_program import ONNXProgram
from ._internal.torchscript_exporter import (  # Deprecated members that are excluded from __all__
    symbolic_helper,
    symbolic_opset10,
    symbolic_opset9,
    utils,
)
from ._internal.torchscript_exporter._type_utils import (
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 40-47
```python
    JitScalarType,  # Deprecated members that are excluded from __all__
)
from ._internal.torchscript_exporter.utils import (  # Deprecated members that are excluded from __all__
    register_custom_op_symbolic,
    select_model_mode_for_export,  # pyrefly: ignore  # deprecated
    unregister_custom_op_symbolic,
)
from .errors import OnnxExporterError
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 48-59
```python


if TYPE_CHECKING:
    import os
    from collections.abc import Callable, Collection, Mapping, Sequence

# Set namespace for exposed private names
ONNXProgram.__module__ = "torch.onnx"
ExportableModule.__module__ = "torch.onnx"
OnnxExporterError.__module__ = "torch.onnx"
InputObserver.__module__ = "torch.onnx"

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 60-73
```python
# TODO(justinchuby): Remove these two properties
producer_name = "pytorch"
producer_version = _C_onnx.PRODUCER_VERSION


def export(
    model: torch.nn.Module
    | torch.export.ExportedProgram
    | torch.jit.ScriptModule
    | torch.jit.ScriptFunction,
    args: tuple[Any, ...] = (),
    f: str | os.PathLike | None = None,
    *,
    kwargs: dict[str, Any] | None = None,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `export`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`export`。

### Lines 74-87
```python
    verbose: bool | None = None,
    input_names: Sequence[str] | None = None,
    output_names: Sequence[str] | None = None,
    opset_version: int | None = None,
    dynamo: bool = True,
    # Dynamo only options
    external_data: bool = True,
    dynamic_shapes: dict[str, Any] | tuple[Any, ...] | list[Any] | None = None,
    custom_translation_table: dict[Callable, Callable] | None = None,
    report: bool = False,
    optimize: bool = True,
    verify: bool = False,
    profile: bool = False,
    dump_exported_program: bool = False,
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 88-101
```python
    artifacts_dir: str | os.PathLike = ".",
    # BC options
    export_params: bool = True,
    keep_initializers_as_inputs: bool = False,
    dynamic_axes: Mapping[str, Mapping[int, str]]
    | Mapping[str, Sequence[int]]
    | None = None,
    # Deprecated options
    training: _C_onnx.TrainingMode = _C_onnx.TrainingMode.EVAL,
    operator_export_type: _C_onnx.OperatorExportTypes = _C_onnx.OperatorExportTypes.ONNX,
    do_constant_folding: bool = True,
    custom_opsets: Mapping[str, int] | None = None,
    export_modules_as_functions: bool | Collection[type[torch.nn.Module]] = False,
    autograd_inlining: bool = True,
```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 102-109
```python
) -> ONNXProgram | None:
    r"""Exports a model into ONNX format.

    Setting ``dynamo=True`` enables the new ONNX export logic
    which is based on :class:`torch.export.ExportedProgram` and a more modern
    set of translation logic. This is the recommended and default way to export models
    to ONNX.

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 110-117
```python
    When ``dynamo=True``:

    The exporter tries the following strategies to get an ExportedProgram for conversion to ONNX.

    #. If the model is already an ExportedProgram, it will be used as-is.
    #. Use :func:`torch.export.export` and set ``strict=False``.
    #. Use :func:`torch.export.export` and set ``strict=True``.

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 118-131
```python
    Args:
        model: The model to be exported.
        args: Example positional inputs. Any non-Tensor arguments will be hard-coded into the
            exported model; any Tensor arguments will become inputs of the exported model,
            in the order they occur in the tuple.
        f: Path to the output ONNX model file. E.g. "model.onnx". This argument is kept for
            backward compatibility. It is recommended to leave unspecified (None)
            and use the returned :class:`torch.onnx.ONNXProgram` to serialize the model
            to a file instead.
        kwargs: Optional example keyword inputs.
        verbose: Whether to enable verbose logging.
        input_names: names to assign to the input nodes of the graph, in order.
        output_names: names to assign to the output nodes of the graph, in order.
        opset_version: The version of the
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 132-145
```python
            `default (ai.onnx) opset <https://github.com/onnx/onnx/blob/master/docs/Operators.md>`_
            to target. You should set ``opset_version`` according to the supported opset versions
            of the runtime backend or compiler you want to run the exported model with.
            Leave as default (``None``) to use the recommended version, or refer to
            the ONNX operators documentation for more information.
        dynamo: Whether to export the model with ``torch.export`` ExportedProgram instead of TorchScript.
        external_data: Whether to save the model weights as an external data file.
            This is required for models with large weights that exceed the ONNX file size limit (2GB).
            When False, the weights are saved in the ONNX file with the model architecture.
        dynamic_shapes: A dictionary or a tuple of dynamic shapes for the model inputs. Refer to
            :func:`torch.export.export` for more details. This is only used (and preferred) when dynamo is True.
            Note that dynamic_shapes is designed to be used when the model is exported with dynamo=True, while
            dynamic_axes is used when dynamo=False.
        custom_translation_table: A dictionary of custom decompositions for operators in the model.
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; bridges to backend-specific execution artifacts; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；桥接到特定后端的执行产物；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 146-159
```python
            The dictionary should have the callable target in the fx Node as the key (e.g. ``torch.ops.aten.stft.default``),
            and the value should be a function that builds that graph using ONNX Script. This option
            is only valid when dynamo is True.
        report: Whether to generate a markdown report for the export process. This option
            is only valid when dynamo is True.
        optimize: Whether to optimize the exported model. This option
            is only valid when dynamo is True. Default is True.
        verify: Whether to verify the exported model using ONNX Runtime. This option
            is only valid when dynamo is True.
        profile: Whether to profile the export process. This option
            is only valid when dynamo is True.
        dump_exported_program: Whether to dump the :class:`torch.export.ExportedProgram` to a file.
            This is useful for debugging the exporter. This option is only valid when dynamo is True.
        artifacts_dir: The directory to save the debugging artifacts like the report and the serialized
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 160-170
```python
            exported program. This option is only valid when dynamo is True.
        export_params: **When ``f`` is specified**: If false, parameters (weights) will not be exported.

            You can also leave it unspecified and use the returned :class:`torch.onnx.ONNXProgram`
            to control how initializers are treated when serializing the model.
        keep_initializers_as_inputs: **When ``f`` is specified**: If True, all the
            initializers (typically corresponding to model weights) in the
            exported graph will also be added as inputs to the graph. If False,
            then initializers are not added as inputs to the graph, and only
            the user inputs are added as inputs.

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 171-179
```python
            Set this to True if you intend to supply model weights at runtime.
            Set it to False if the weights are static to allow for better optimizations
            (e.g. constant folding) by backends/runtimes.

            You can also leave it unspecified and use the returned :class:`torch.onnx.ONNXProgram`
            to control how initializers are treated when serializing the model.
        dynamic_axes:
            Deprecated: Prefer specifying ``dynamic_shapes`` when ``dynamo=True``.

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 180-188
```python
            By default the exported model will have the shapes of all input and output tensors
            set to exactly match those given in ``args``. To specify axes of tensors as
            dynamic (i.e. known only at run-time), set ``dynamic_axes`` to a dict with schema:

            * KEY (str): an input or output name. Each name must also be provided in ``input_names`` or
                ``output_names``.
            * VALUE (dict or list): If a dict, keys are axis indices and values are axis names. If a
                list, each element is an axis index.

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 189-195
```python
            For example::

                class SumModule(torch.nn.Module):
                    def forward(self, x):
                        return torch.sum(x, dim=1)


```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `SumModule`, `forward`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`SumModule`, `forward`。

### Lines 196-203
```python
                torch.onnx.export(
                    SumModule(),
                    (torch.ones(2, 2),),
                    "onnx.pb",
                    input_names=["x"],
                    output_names=["sum"],
                )

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 204-217
```python
            Produces::

                input {
                  name: "x"
                  ...
                      shape {
                        dim {
                          dim_value: 2  # axis 0
                        }
                        dim {
                          dim_value: 2  # axis 1
                ...
                output {
                  name: "sum"
```
- EN: This block implements local helper logic for init. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 init 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 218-225
```python
                  ...
                      shape {
                        dim {
                          dim_value: 2  # axis 0
                ...

            While::

```
- EN: This block implements local helper logic for init. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 init 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 226-239
```python
                torch.onnx.export(
                    SumModule(),
                    (torch.ones(2, 2),),
                    "onnx.pb",
                    input_names=["x"],
                    output_names=["sum"],
                    dynamic_axes={
                        # dict value: manually named axes
                        "x": {0: "my_custom_axis_name"},
                        # list value: automatic names
                        "sum": [0],
                    },
                )

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 240-253
```python
            Produces::

                input {
                  name: "x"
                  ...
                      shape {
                        dim {
                          dim_param: "my_custom_axis_name"  # axis 0
                        }
                        dim {
                          dim_value: 2  # axis 1
                ...
                output {
                  name: "sum"
```
- EN: This block implements local helper logic for init. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 init 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 254-266
```python
                  ...
                      shape {
                        dim {
                          dim_param: "sum_dynamic_axes_1"  # axis 0
                ...

        training: Deprecated option. Instead, set the training mode of the model before exporting.
        operator_export_type: Deprecated option. Only ONNX is supported.
        do_constant_folding: Deprecated option.
        custom_opsets: Deprecated option.
        export_modules_as_functions: Deprecated option.
        autograd_inlining: Deprecated option.

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 267-280
```python
    Returns:
        :class:`torch.onnx.ONNXProgram` if dynamo is True, otherwise None.

    .. versionchanged:: 2.6
        ``training`` is now deprecated. Instead, set the training mode of the model before exporting.
        ``operator_export_type`` is now deprecated. Only ONNX is supported.
        ``do_constant_folding`` is now deprecated. It is always enabled.
        ``export_modules_as_functions`` is now deprecated.
        ``autograd_inlining`` is now deprecated.
    .. versionchanged:: 2.7
        ``optimize`` is now True by default.
    .. versionchanged:: 2.9
        ``dynamo`` is now True by default.
    .. versionchanged:: 2.11
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 281-287
```python
        ``fallback`` option has been removed.
    """
    if dynamo is True or isinstance(
        model, (torch.export.ExportedProgram, ExportableModule)
    ):
        from torch.onnx._internal.exporter import _compat

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 288-301
```python
        if isinstance(args, torch.Tensor):
            args = (args,)

        return _compat.export_compat(
            model,
            args,
            f,
            kwargs=kwargs,
            export_params=export_params,
            verbose=verbose,
            input_names=input_names,
            output_names=output_names,
            opset_version=opset_version,
            custom_translation_table=custom_translation_table,
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 302-314
```python
            dynamic_axes=dynamic_axes,
            keep_initializers_as_inputs=keep_initializers_as_inputs,
            external_data=external_data,
            dynamic_shapes=dynamic_shapes,
            report=report,
            optimize=optimize,
            verify=verify,
            profile=profile,
            dump_exported_program=dump_exported_program,
            artifacts_dir=artifacts_dir,
        )
    else:
        import warnings
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 315-327
```python

        from ._internal.torchscript_exporter.utils import export

        warnings.warn(
            "You are using the legacy TorchScript-based ONNX export. Starting in PyTorch 2.9, "
            "the new torch.export-based ONNX exporter has become the default. "
            "Learn more about the new export logic: https://docs.pytorch.org/docs/stable/onnx_export.html. "
            "For exporting control flow: "
            "https://pytorch.org/tutorials/beginner/onnx/export_control_flow_model_to_onnx_tutorial.html",
            category=DeprecationWarning,
            stacklevel=2,
        )

```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 328-341
```python
        if dynamic_shapes:
            raise ValueError(
                "The exporter only supports dynamic shapes "
                "through parameter dynamic_axes when dynamo=False."
            )

        export(
            model,
            args,
            f,  # type: ignore[arg-type]
            kwargs=kwargs,
            export_params=export_params,
            verbose=verbose is True,
            input_names=input_names,
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 342-353
```python
            output_names=output_names,
            opset_version=opset_version,
            dynamic_axes=dynamic_axes,
            keep_initializers_as_inputs=keep_initializers_as_inputs,
            training=training,
            operator_export_type=operator_export_type,
            do_constant_folding=do_constant_folding,
            custom_opsets=custom_opsets,
            export_modules_as_functions=export_modules_as_functions,
            autograd_inlining=autograd_inlining,
        )
        return None
```
- EN: This block advances ONNX export translation; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 354-360
```python


def is_in_onnx_export() -> bool:
    """Returns whether it is in the middle of ONNX export."""
    from torch.onnx._internal.exporter import _flags
    from torch.onnx._internal.torchscript_exporter._globals import GLOBALS

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal.exporter`, `torch.onnx._internal.torchscript_exporter._globals`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal.exporter`, `torch.onnx._internal.torchscript_exporter._globals`；外部导入：无。

### Lines 361-361
```python
    return GLOBALS.in_onnx_export or _flags._is_onnx_exporting
```
- EN: This block advances ONNX export translation; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
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

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch._C`, `torch._C._onnx`, `.`, `._internal.exporter._exportable_module`, `._internal.exporter._input_observer`, `._internal.exporter._onnx_program`, `._internal.torchscript_exporter`, `._internal.torchscript_exporter._type_utils`, `._internal.torchscript_exporter.utils`, `...`
- External imports / 外部导入: `__future__`, `typing`, `os`, `collections.abc`, `warnings`
- Representative symbols / 代表性符号: `export`, `is_in_onnx_export`
