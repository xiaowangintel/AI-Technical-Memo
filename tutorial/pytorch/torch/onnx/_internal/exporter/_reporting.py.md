# _reporting.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_reporting.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for reporting, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 reporting 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: allow-untyped-defs
from __future__ import annotations

import dataclasses
import re
from typing import TYPE_CHECKING

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `__future__`, `dataclasses`, `re`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`__future__`, `dataclasses`, `re`, `typing`。

### Lines 8-14
```python
from torch.onnx._internal.exporter import _analysis, _registration, _verification


if TYPE_CHECKING:
    import os

    import onnx_ir as ir  # type: ignore[import-untyped]
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal.exporter`; external imports: `os`, `onnx_ir`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal.exporter`；外部导入：`os`, `onnx_ir`。

### Lines 15-28
```python

    import torch


@dataclasses.dataclass
class ExportStatus:
    # Whether torch.export.export(..., strict=True) succeeds
    torch_export_strict: bool | None = None
    # Whether torch.export.export(..., strict=False) succeeds
    torch_export_non_strict: bool | None = None
    # Whether torch.export.draft_export() succeeds
    torch_export_draft_export: bool | None = None
    # Whether decomposition succeeds
    decomposition: bool | None = None
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ExportStatus`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ExportStatus`。

### Lines 29-37
```python
    # Whether ONNX translation succeeds
    onnx_translation: bool | None = None
    # Whether ONNX model passes onnx.checker.check_model
    onnx_checker: bool | None = None
    # Whether ONNX model runs successfully with ONNX Runtime
    onnx_runtime: bool | None = None
    # Whether the output of the ONNX model is accurate
    output_accuracy: bool | None = None

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 38-44
```python

def _status_emoji(status: bool | None) -> str:
    if status is None:
        return "⚪"
    return "✅" if status else "❌"


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_status_emoji`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_status_emoji`。

### Lines 45-58
```python
def _format_export_status(status: ExportStatus) -> str:
    return (
        f"```\n"
        f"{_status_emoji(status.torch_export_non_strict)} Obtain model graph with `torch.export.export(..., strict=False)`\n"
        f"{_status_emoji(status.torch_export_strict)} Obtain model graph with `torch.export.export(..., strict=True)`\n"
        f"{_status_emoji(status.torch_export_draft_export)} Obtain model graph with `torch.export.draft_export`\n"
        f"{_status_emoji(status.decomposition)} Decompose operators for ONNX compatibility\n"
        f"{_status_emoji(status.onnx_translation)} Translate the graph into ONNX\n"
        f"{_status_emoji(status.onnx_checker)} Run `onnx.checker` on the ONNX model\n"
        f"{_status_emoji(status.onnx_runtime)} Execute the model with ONNX Runtime\n"
        f"{_status_emoji(status.output_accuracy)} Validate model output accuracy\n"
        f"```\n\n"
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_format_export_status`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_format_export_status`。

### Lines 59-65
```python

def _strip_color_from_string(text: str) -> str:
    # This regular expression matches ANSI escape codes
    # https://github.com/pytorch/pytorch/blob/9554a9af8788c57e1c5222c39076a5afcf0998ae/torch/_dynamo/utils.py#L2785-L2788
    ansi_escape = re.compile(r"\x1B[@-_][0-?]*[ -/]*[@-~]")
    return ansi_escape.sub("", text)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_strip_color_from_string`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_strip_color_from_string`。

### Lines 66-75
```python

def _format_exported_program(exported_program: torch.export.ExportedProgram) -> str:
    # Adapted from https://github.com/pytorch/pytorch/pull/128476
    # to remove colors
    # Even though we can call graph_module.print_readable directly, since the
    # colored option was added only recently, we can't guarantee that the
    # version of PyTorch used by the user has this option. Therefore, we
    # still call str(ExportedProgram)
    text = f"```python\n{_strip_color_from_string(str(exported_program))}\n```\n\n"
    return text
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_format_exported_program`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_format_exported_program`。

### Lines 76-89
```python


def construct_report_file_name(timestamp: str, status: ExportStatus) -> str:
    # Status could be None. So we need to check for False explicitly.
    if not (
        status.torch_export_non_strict
        or status.torch_export_strict
        or status.torch_export_draft_export
    ):
        # All strategies failed
        postfix = "pt_export"
    elif status.decomposition is False:
        postfix = "decomp"
    elif status.onnx_translation is False:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `construct_report_file_name`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`construct_report_file_name`。

### Lines 90-103
```python
        postfix = "conversion"
    elif status.onnx_checker is False:
        postfix = "checker"
    elif status.onnx_runtime is False:
        postfix = "runtime"
    elif status.output_accuracy is False:
        postfix = "accuracy"
    elif (
        status.torch_export_strict is False
        or status.torch_export_non_strict is False
        or status.torch_export_draft_export is False
    ):
        # Some strategies failed
        postfix = "strategies"
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 104-114
```python
    else:
        postfix = "success"
    return f"onnx_export_{timestamp}_{postfix}.md"


def format_decomp_comparison(
    pre_decomp_unique_ops: set[str],
    post_decomp_unique_ops: set[str],
) -> str:
    """Format the decomposition comparison result.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `format_decomp_comparison`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`format_decomp_comparison`。

### Lines 115-122
```python
    Args:
        unique_ops_in_a: The unique ops in the first program.
        unique_ops_in_b: The unique ops in the second program.

    Returns:
        The formatted comparison result.
    """
    return (
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 123-132
```python
        f"Ops exist only in the ExportedProgram before decomposition: `{sorted(pre_decomp_unique_ops)}`\n\n"
        f"Ops exist only in the ExportedProgram after decomposition: `{sorted(post_decomp_unique_ops)}`\n"
    )


def format_verification_infos(
    verification_infos: list[_verification.VerificationInfo],
) -> str:
    """Format the verification result.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `format_verification_infos`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`format_verification_infos`。

### Lines 133-139
```python
    Args:
        verification_infos: The verification result.

    Returns:
        The formatted verification result.
    """
    return "\n".join(
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 140-146
```python
        f"`{info.name}`: `max_abs_diff={info.max_abs_diff:e}`, `max_rel_diff={info.max_rel_diff:e}`, "
        f"`abs_diff_hist={info.abs_diff_hist}`, `rel_diff_hist={info.rel_diff_hist}`"
        for info in verification_infos
    )


def create_torch_export_error_report(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `create_torch_export_error_report`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`create_torch_export_error_report`。

### Lines 147-160
```python
    filename: str | os.PathLike,
    formatted_traceback: str,
    *,
    export_status: ExportStatus,
    profile_result: str | None,
) -> None:
    with open(filename, "w", encoding="utf-8") as f:
        f.write("# PyTorch ONNX Conversion Error Report\n\n")
        f.write(_format_export_status(export_status))
        f.write("Error message:\n\n")
        f.write("```pytb\n")
        f.write(formatted_traceback)
        f.write("```\n\n")
        if profile_result is not None:
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 161-167
```python
            f.write("## Profiling result\n\n")
            f.write("```\n")
            f.write(profile_result)
            f.write("```\n")


def create_onnx_export_report(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `create_onnx_export_report`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`create_onnx_export_report`。

### Lines 168-181
```python
    filename: str | os.PathLike,
    formatted_traceback: str,
    program: torch.export.ExportedProgram,
    *,
    decomp_comparison: str | None = None,
    export_status: ExportStatus,
    profile_result: str | None,
    model: ir.Model | None = None,
    registry: _registration.ONNXRegistry | None = None,
    verification_result: str | None = None,
) -> None:
    with open(filename, "w", encoding="utf-8") as f:
        f.write("# PyTorch ONNX Conversion Report\n\n")
        f.write(_format_export_status(export_status))
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 182-188
```python
        f.write("## Error messages\n\n")
        f.write("```pytb\n")
        f.write(formatted_traceback)
        f.write("\n```\n\n")
        f.write("## Exported program\n\n")
        f.write(_format_exported_program(program))
        if model is not None:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 189-195
```python
            f.write("## ONNX model\n\n")
            f.write("```python\n")
            f.write(str(model))
            f.write("\n```\n\n")
        f.write("## Analysis\n\n")
        _analysis.analyze(program, file=f, registry=registry)
        if decomp_comparison is not None:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 196-203
```python
            f.write("\n## Decomposition comparison\n\n")
            f.write(decomp_comparison)
            f.write("\n")
        if verification_result is not None:
            f.write("\n## Verification results\n\n")
            f.write(verification_result)
            f.write("\n")
        if profile_result is not None:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 204-207
```python
            f.write("\n## Profiling result\n\n")
            f.write("```\n")
            f.write(profile_result)
            f.write("```\n")
```
- EN: This block implements local helper logic for reporting. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 reporting 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch.onnx._internal.exporter`, `torch`
- External imports / 外部导入: `__future__`, `dataclasses`, `re`, `typing`, `os`, `onnx_ir`
- Representative symbols / 代表性符号: `ExportStatus`, `_status_emoji`, `_format_export_status`, `_strip_color_from_string`, `_format_exported_program`, `construct_report_file_name`, `format_decomp_comparison`, `format_verification_infos`, `create_torch_export_error_report`, `create_onnx_export_report`
