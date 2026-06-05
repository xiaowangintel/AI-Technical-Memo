# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/profiler/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `utils`-related logic centered around `TablePrinter`, `trim_string_front`, `trim_string_back`. / 实现与 `utils` 相关的逻辑，核心符号包括 `TablePrinter`, `trim_string_front`, `trim_string_back`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-8)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import dataclasses
from collections.abc import Callable

from _typeshed import DataclassInstance
from torch._C._profiler import _EventType, _ProfilerEvent, _TensorMetadata
```
**EN:** Sets up the module with standard-library support such as `dataclasses`, `collections.abc`, external packages such as `_typeshed`, `torch._C._profiler`. It prepares the symbols later used by `TablePrinter`, `trim_string_front`, `trim_string_back`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `TablePrinter`, `trim_string_front`, `trim_string_back` 提供上下文。

### trim_string_front (lines 15-21)
```python
def trim_string_front(string: str, width: int) -> str:
    if len(string) > width:
        offset = len(string) - width + 3
        string = string[offset:]
        if len(string) > 3:
            string = "..." + string[3:]
    return string
```
**EN:** `trim_string_front` implements helper logic used by this module. It mainly works with `string`, `width`.
**CN:** `trim_string_front` 负责实现本模块使用的辅助逻辑。 它主要处理 `string`, `width` 等参数。

### trim_string_back (lines 24-30)
```python
def trim_string_back(string: str, width: int) -> str:
    if len(string) > width:
        offset = len(string) - width + 3
        string = string[:-offset]
        if len(string) > 3:
            string = string + "..."
    return string
```
**EN:** `trim_string_back` implements helper logic used by this module. It mainly works with `string`, `width`.
**CN:** `trim_string_back` 负责实现本模块使用的辅助逻辑。 它主要处理 `string`, `width` 等参数。

### TablePrinter (lines 33-76)
```python
class TablePrinter:
    def __init__(self, row_cls: type[DataclassInstance], column_widths: dict[str, int]):
        self.row_cls = row_cls
        self.fieldnames = [x.name for x in dataclasses.fields(row_cls)]
        self.column_widths = column_widths
        assert set(self.column_widths.keys()) == set(self.fieldnames)

    def print_table(self, rows: list[DataclassInstance]):
        self._print_header()
        self._print_line()
        for row in rows:
            self._print_row(row)

    def _print_header(self):
        for i, f in enumerate(self.fieldnames):
            last = i == len(self.fieldnames) - 1
            col_width = self.column_widths[f]
            print(
                trim_string_back(f, col_width).ljust(col_width),
                end=" | " if not last else "\n",
            )

    def _print_row(self, row):
        assert isinstance(row, self.row_cls)

        for i, f in enumerate(self.fieldnames):
            last = i == len(self.fieldnames) - 1
            col_width = self.column_widths[f]
    # ...

    def _print_line(self):
        total_col_width = 0
        for column_width in self.column_widths.values():
            total_col_width += column_width
        print("=" * (total_col_width + 3 * (len(self.column_widths) - 1)))
```
**EN:** Defines the `TablePrinter` class used by this module. Key methods include `__init__`, `print_table`.
**CN:** `TablePrinter` 是该文件中的核心类，用于封装与 `TablePrinter` 相关的状态和行为。 关键方法包括 `__init__`, `print_table`。

### indent_string (lines 79-88)
```python
def indent_string(
    string: str, indent: int, indent_style: Callable[[int], str] | str = " "
) -> str:
    if indent:
        if isinstance(indent_style, str):
            return indent_style * indent + string
        else:
            return indent_style(indent) + string
    else:
        return string
```
**EN:** `indent_string` implements helper logic used by this module. It mainly works with `string`, `indent`, `indent_style`. Inside the body, it relies on `indent_style` to complete the main steps.
**CN:** `indent_string` 负责实现本模块使用的辅助逻辑。 它主要处理 `string`, `indent`, `indent_style` 等参数。 实现过程中会调用 `indent_style` 等函数完成关键步骤。

### event_has_module (lines 96-100)
```python
def event_has_module(event: _ProfilerEvent) -> bool:
    event_type, typed_event = event.typed
    if event_type == _EventType.PyCall:
        return typed_event.module is not None
    return False
```
**EN:** `event_has_module` implements helper logic used by this module. It mainly works with `event`.
**CN:** `event_has_module` 负责实现本模块使用的辅助逻辑。 它主要处理 `event` 等参数。

### event_is_torch_op (lines 103-104)
```python
def event_is_torch_op(event: _ProfilerEvent) -> bool:
    return event.tag == _EventType.TorchOp
```
**EN:** `event_is_torch_op` implements helper logic used by this module. It mainly works with `event`.
**CN:** `event_is_torch_op` 负责实现本模块使用的辅助逻辑。 它主要处理 `event` 等参数。

### event_arg_repr (lines 107-117)
```python
def event_arg_repr(arg) -> str:
    if arg is None or type(arg) in [float, int, bool, str]:
        return f"{arg}"
    elif isinstance(arg, list):
        return f"[{', '.join([event_arg_repr(x) for x in arg])}]"
    elif isinstance(arg, tuple):
        return f"({', '.join([event_arg_repr(x) for x in arg])})"
    else:
        assert isinstance(arg, _TensorMetadata), f"Unsupported type: {type(arg)}"
        sizes_str = ", ".join([str(x) for x in arg.sizes])
        return f"{str(arg.dtype).replace('torch.', '')}[{sizes_str}]"
```
**EN:** `event_arg_repr` implements helper logic used by this module. It mainly works with `arg`. Inside the body, it relies on `type`, `join`, `str.replace` to complete the main steps.
**CN:** `event_arg_repr` 负责实现本模块使用的辅助逻辑。 它主要处理 `arg` 等参数。 实现过程中会调用 `type`, `join`, `str.replace` 等函数完成关键步骤。

### event_torch_op_repr (lines 120-123)
```python
def event_torch_op_repr(event: _ProfilerEvent) -> str:
    assert event.tag == _EventType.TorchOp
    args_str = ", ".join([event_arg_repr(x) for x in event.typed[1].inputs])
    return f"{event.name}({args_str})".replace("aten::", "")
```
**EN:** `event_torch_op_repr` implements helper logic used by this module. It mainly works with `event`. Inside the body, it relies on `join`, `replace`, `event_arg_repr` to complete the main steps.
**CN:** `event_torch_op_repr` 负责实现本模块使用的辅助逻辑。 它主要处理 `event` 等参数。 实现过程中会调用 `join`, `replace`, `event_arg_repr` 等函数完成关键步骤。

### event_module_repr (lines 126-135)
```python
def event_module_repr(event: _ProfilerEvent) -> str:
    assert event_has_module(event)
    module = event.typed[1].module
    if module.parameters and len(module.parameters) > 0:
        args_str = ", ".join(
            [f"{x[0]}={event_arg_repr(x[1])}" for x in module.parameters]
        )
        return f"{module.cls_name}({args_str})"
    else:
        return module.cls_name
```
**EN:** `event_module_repr` implements helper logic used by this module. It mainly works with `event`. Inside the body, it relies on `event_has_module`, `join`, `event_arg_repr` to complete the main steps.
**CN:** `event_module_repr` 负责实现本模块使用的辅助逻辑。 它主要处理 `event` 等参数。 实现过程中会调用 `event_has_module`, `join`, `event_arg_repr` 等函数完成关键步骤。

### event_torch_op_stack_trace (lines 138-150)
```python
def event_torch_op_stack_trace(
    curr_event: _ProfilerEvent, until: Callable[[_ProfilerEvent], bool]
) -> str:
    trace = ""
    curr_event = curr_event.parent
    while curr_event and not until(curr_event):
        if event_is_torch_op(curr_event):
            if len(trace) > 0:
                trace += " <- "
            trace += event_torch_op_repr(curr_event)
        curr_event = curr_event.parent

    return trace
```
**EN:** `event_torch_op_stack_trace` implements helper logic used by this module. It mainly works with `curr_event`, `until`. Inside the body, it relies on `event_is_torch_op`, `until`, `event_torch_op_repr` to complete the main steps.
**CN:** `event_torch_op_stack_trace` 负责实现本模块使用的辅助逻辑。 它主要处理 `curr_event`, `until` 等参数。 实现过程中会调用 `event_is_torch_op`, `until`, `event_torch_op_repr` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`TablePrinter`**: Core class that organizes module behavior. / **`TablePrinter`**：组织模块行为的核心类。
- **`trim_string_front`**: Key helper or entry point in this file. / **`trim_string_front`**：本文件中的关键辅助函数或入口。
- **`trim_string_back`**: Key helper or entry point in this file. / **`trim_string_back`**：本文件中的关键辅助函数或入口。
- **`indent_string`**: Key helper or entry point in this file. / **`indent_string`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: dataclasses, collections.abc
- **Third-party / 第三方**: _typeshed, torch._C._profiler
- **Internal vLLM / vLLM 内部依赖**: None / 无
