# _capture_strategies.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_capture_strategies.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for capture strategies, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 capture strategies 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
"""Strategies for capturing ExportedPrograms."""

# mypy: allow-untyped-defs
from __future__ import annotations

import abc
import contextlib
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `__future__`, `abc`, `contextlib`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`__future__`, `abc`, `contextlib`。

### Lines 8-14
```python
import dataclasses
import datetime
import logging
import pathlib
from typing import Any, TYPE_CHECKING

import torch
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`; external imports: `dataclasses`, `datetime`, `logging`, `pathlib`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`；外部导入：`dataclasses`, `datetime`, `logging`, `pathlib`, `typing`。

### Lines 15-21
```python
from torch.onnx import _flags


if TYPE_CHECKING:
    import os
    from collections.abc import Callable

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx`; external imports: `os`, `collections.abc`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx`；外部导入：`os`, `collections.abc`。

### Lines 22-28
```python

logger = logging.getLogger(__name__)


def _verbose_printer(verbose: bool | None) -> Callable[..., None]:
    """Prints messages based on `verbose`."""
    if verbose is False:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_verbose_printer`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_verbose_printer`。

### Lines 29-38
```python
        return lambda *_, **__: None

    return lambda *args, **kwargs: print("[torch.onnx]", *args, **kwargs)


def _take_first_line(text: str) -> str:
    """Take the first line of a text."""
    lines = text.split("\n", maxsplit=1)
    first_line = lines[0]
    if len(lines) > 1:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_take_first_line`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_take_first_line`。

### Lines 39-47
```python
        first_line += "[...]"
    return first_line


@contextlib.contextmanager
def _patch_dynamo_unsupported_functions():
    """Patch PyTorch to bypass some functions torch.export.export does not support."""
    # TODO: Remove the patches once dynamo supports these functions.
    import torch.jit
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_patch_dynamo_unsupported_functions`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_patch_dynamo_unsupported_functions`。

### Lines 48-58
```python

    # Replace torch.jit.isinstance with isinstance
    jit_isinstance = torch.jit.isinstance
    # pyrefly: ignore [bad-assignment]
    torch.jit.isinstance = isinstance
    logger.info("Replaced torch.jit.isinstance with isinstance to allow dynamo tracing")
    try:
        yield
    finally:
        torch.jit.isinstance = jit_isinstance

```
- EN: This block implements local helper logic for capture strategies. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 capture strategies 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 59-65
```python

@dataclasses.dataclass
class Result:
    exported_program: torch.export.ExportedProgram | None
    strategy: str
    exception: Exception | None = None

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Result`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Result`。

### Lines 66-75
```python
    @property
    def success(self) -> bool:
        """Whether the capture was successful.

        An exception can still be recorded even if the capture was successful. In
        this case the exception is informational only. For example, draft_export
        can record an exception if there are warnings during the export. The exceptions
        will go into the onnx export report when report=True.
        """
        return self.exported_program is not None
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `success`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`success`。

### Lines 76-83
```python


class CaptureStrategy(abc.ABC):
    """Strategy for capturing a module as ExportedProgram.

    To use a strategy, create an instance and call it with the model, args, kwargs, and dynamic_shapes.
    Example::

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `CaptureStrategy`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`CaptureStrategy`。

### Lines 84-97
```python
        strategy = TorchExportNonStrictStrategy(verbose=True)
        result = strategy(model, args, kwargs, dynamic_shapes)
    """

    def __init__(
        self,
        *,
        verbose: bool = False,
        dump: bool = False,
        artifacts_dir: str | os.PathLike = ".",
        timestamp: str | None = None,
    ) -> None:
        """Initialize the strategy.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 98-109
```python
        Args:
            verbose: Whether to print verbose messages.
            dump: Whether to dump the intermediate artifacts to a file.
        """
        self._verbose_print = _verbose_printer(verbose)
        self._dump = dump
        self._artifacts_dir = pathlib.Path(artifacts_dir)
        self._timestamp = timestamp or datetime.datetime.now().strftime(
            "%Y-%m-%d_%H-%M-%S-%f"
        )
        self._exception: Exception | None = None

```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 110-118
```python
    def __call__(
        self,
        model: torch.nn.Module | torch.jit.ScriptFunction,
        args: tuple[Any, ...],
        kwargs: dict[str, Any] | None,
        dynamic_shapes,
    ) -> Result:
        self._enter(model)
        if kwargs is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__call__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__call__`。

### Lines 119-130
```python
            kwargs = {}
        try:
            exported_program = self._capture(model, args, kwargs, dynamic_shapes)
        except Exception as e:
            self._failure(model, e)
            return Result(
                exported_program=None,
                strategy=self.__class__.__name__,
                exception=e,
            )
        self._success(model)
        return Result(
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 131-137
```python
            exported_program,
            strategy=self.__class__.__name__,
            exception=self._exception,
        )

    @abc.abstractmethod
    def _capture(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_capture`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_capture`。

### Lines 138-144
```python
        self, model, args, kwargs, dynamic_shapes
    ) -> torch.export.ExportedProgram:
        raise NotImplementedError

    def _enter(self, model: torch.nn.Module | torch.jit.ScriptFunction) -> None:
        return

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_enter`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_enter`。

### Lines 145-152
```python
    def _success(self, model: torch.nn.Module | torch.jit.ScriptFunction) -> None:
        return

    def _failure(
        self, model: torch.nn.Module | torch.jit.ScriptFunction, e: Exception
    ) -> None:
        return

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_success`, `_failure`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_success`, `_failure`。

### Lines 153-164
```python

class TorchExportStrictStrategy(CaptureStrategy):
    def _capture(
        self, model, args, kwargs, dynamic_shapes
    ) -> torch.export.ExportedProgram:
        with (
            _patch_dynamo_unsupported_functions(),
            # Support the dynamism with 0/1 input dim
            torch.fx.experimental._config.patch(backed_size_oblivious=True),  # type: ignore[attr-defined]
        ):
            try:
                return torch.export.export(
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `TorchExportStrictStrategy`, `_capture`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`TorchExportStrictStrategy`, `_capture`。

### Lines 165-178
```python
                    model,
                    args,
                    kwargs=kwargs,
                    dynamic_shapes=dynamic_shapes,
                    strict=True,
                    prefer_deferred_runtime_asserts_over_guards=_flags.PREFER_DEFERRED_RUNTIME_ASSERTS_OVER_GUARDS,
                )
            except torch._dynamo.exc.UserError as exc:
                # Refine the dynamic shapes based on the suggested fixes.
                try:
                    new_shapes = torch.export.dynamic_shapes.refine_dynamic_shapes_from_suggested_fixes(
                        exc.msg, dynamic_shapes
                    )
                except Exception:
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; reports or normalizes error conditions; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；报告或规范化错误情况；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 179-189
```python
                    # If the dynamic shapes cannot be refined, re-raise the exception.
                    raise exc from None
                return torch.export.export(
                    model,
                    args,
                    kwargs=kwargs,
                    dynamic_shapes=new_shapes,
                    strict=True,
                    prefer_deferred_runtime_asserts_over_guards=_flags.PREFER_DEFERRED_RUNTIME_ASSERTS_OVER_GUARDS,
                )

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 190-196
```python
    def _enter(self, model) -> None:
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.export(..., strict=True)`..."
        )

    def _success(self, model) -> None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_enter`, `_success`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_enter`, `_success`。

### Lines 197-208
```python
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.export(..., strict=True)`... ✅"
        )

    def _failure(self, model, e) -> None:
        del e  # Unused
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.export(..., strict=True)`... ❌"
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_failure`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_failure`。

### Lines 209-219
```python

class TorchExportNonStrictStrategy(CaptureStrategy):
    def _capture(
        self, model, args, kwargs, dynamic_shapes
    ) -> torch.export.ExportedProgram:
        with (
            # Support the dynamism with 0/1 input dim
            torch.fx.experimental._config.patch(backed_size_oblivious=True),  # type: ignore[attr-defined]
        ):
            try:
                return torch.export.export(
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `TorchExportNonStrictStrategy`, `_capture`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`TorchExportNonStrictStrategy`, `_capture`。

### Lines 220-233
```python
                    model,
                    args,
                    kwargs=kwargs,
                    dynamic_shapes=dynamic_shapes,
                    strict=False,
                    prefer_deferred_runtime_asserts_over_guards=_flags.PREFER_DEFERRED_RUNTIME_ASSERTS_OVER_GUARDS,
                )
            except torch._dynamo.exc.UserError as exc:
                # Refine the dynamic shapes based on the suggested fixes.
                try:
                    new_shapes = torch.export.dynamic_shapes.refine_dynamic_shapes_from_suggested_fixes(
                        exc.msg, dynamic_shapes
                    )
                except Exception:
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; reports or normalizes error conditions; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；报告或规范化错误情况；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 234-244
```python
                    # If the dynamic shapes cannot be refined, re-raise the exception.
                    raise exc from None
                return torch.export.export(
                    model,
                    args,
                    kwargs=kwargs,
                    dynamic_shapes=new_shapes,
                    strict=False,
                    prefer_deferred_runtime_asserts_over_guards=_flags.PREFER_DEFERRED_RUNTIME_ASSERTS_OVER_GUARDS,
                )

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 245-251
```python
    def _enter(self, model) -> None:
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.export(..., strict=False)`..."
        )

    def _success(self, model) -> None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_enter`, `_success`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_enter`, `_success`。

### Lines 252-263
```python
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.export(..., strict=False)`... ✅"
        )

    def _failure(self, model, e) -> None:
        del e  # Unused
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.export(..., strict=False)`... ❌"
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_failure`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_failure`。

### Lines 264-273
```python

class TorchExportDraftExportStrategy(CaptureStrategy):
    def _capture(
        self, model, args, kwargs, dynamic_shapes
    ) -> torch.export.ExportedProgram:
        ep = torch.export.draft_export(
            model, args, kwargs=kwargs, dynamic_shapes=dynamic_shapes
        )
        report = ep._report  # type: ignore[attr-defined]
        if not report.successful():
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `TorchExportDraftExportStrategy`, `_capture`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`TorchExportDraftExportStrategy`, `_capture`。

### Lines 274-283
```python
            self._exception = RuntimeError(str(report))
            self._verbose_print(f"Draft Export report:\n{report}")
        return ep

    def _enter(self, model) -> None:
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.draft_export`..."
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_enter`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_enter`。

### Lines 284-290
```python
    def _success(self, model) -> None:
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.draft_export`... ✅"
        )

    def _failure(self, model, e) -> None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_success`, `_failure`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_success`, `_failure`。

### Lines 291-297
```python
        del e  # Unused
        model_repr = _take_first_line(repr(model))
        self._verbose_print(
            f"Obtain model graph for `{model_repr}` with `torch.export.draft_export`... ❌"
        )


```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 298-304
```python
CAPTURE_STRATEGIES: tuple[type[CaptureStrategy], ...] = (
    TorchExportNonStrictStrategy,  # strict=False is preferred over strict=True because it does not have dynamo issues
    TorchExportStrictStrategy,
)

if _flags.ENABLE_DRAFT_EXPORT:
    CAPTURE_STRATEGIES = (*CAPTURE_STRATEGIES, TorchExportDraftExportStrategy)
```
- EN: This block advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
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
- Internal imports / 内部导入: `torch`, `torch.onnx`, `torch.jit`
- External imports / 外部导入: `__future__`, `abc`, `contextlib`, `dataclasses`, `datetime`, `logging`, `pathlib`, `typing`, `os`, `collections.abc`
- Representative symbols / 代表性符号: `_verbose_printer`, `_take_first_line`, `_patch_dynamo_unsupported_functions`, `Result`, `CaptureStrategy`, `TorchExportStrictStrategy`, `TorchExportNonStrictStrategy`, `TorchExportDraftExportStrategy`
