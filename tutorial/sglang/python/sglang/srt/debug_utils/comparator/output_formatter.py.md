# output_formatter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/output_formatter.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on comparison result visualization. It mainly turns raw comparison state into readable debug output. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于比较结果可视化。它主要用于把原始比较状态转换为可读的调试输出。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Document the module intent / 说明模块意图
```python
"""Formatting functions for comparator output records.

Extracted from output_types.py to separate data-structure definitions
from rendering / formatting logic.
"""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 7-18: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Literal

from rich.console import Group
from rich.markup import escape
from rich.panel import Panel

from sglang.srt.debug_utils.comparator.tensor_comparator.formatter import (
    format_comparison,
    format_replicated_checks,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 20-40: Handle conditional module logic / 处理条件模块逻辑
```python
if TYPE_CHECKING:
    from rich.console import RenderableType

    from sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types import (
        TracedAlignerPlan,
        TracedSubPlan,
    )
    from sglang.srt.debug_utils.comparator.aligner.entrypoint.types import AlignerPlan
    from sglang.srt.debug_utils.comparator.output_types import (
        ComparisonErrorRecord,
        ComparisonNonTensorRecord,
        ComparisonSkipRecord,
        ComparisonTensorRecord,
        ConfigRecord,
        ErrorLog,
        InfoLog,
        LogRecord,
        SummaryRecord,
        _OutputRecord,
        _TableRecord,
    )
```
**EN:** This conditional branch selects behavior based on runtime state, optional inputs, or developer-facing entry conditions.
**CN:** 该条件分支会根据运行时状态、可选输入或面向开发者的入口条件来选择不同的行为。

### Lines 42-42: Declare module-level symbols such as `Verbosity` / 声明模块级符号，例如 `Verbosity`
```python
Verbosity = Literal["minimal", "normal", "verbose"]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 48-63: Implement helper `_render_record_rich` / 实现辅助函数 `_render_record_rich`
```python
def _render_record_rich(
    record: _OutputRecord, *, verbosity: Verbosity = "normal"
) -> RenderableType:
    body: RenderableType = record._format_rich_body(verbosity=verbosity)

    log_lines: list[str] = _format_log_lines_rich(
        errors=record.errors, infos=record.infos
    )

    if not log_lines:
        return body

    log_block: str = "\n".join(log_lines)
    if isinstance(body, str):
        return body + "\n" + log_block
    return Group(body, log_block)
```
**EN:** Function `_render_record_rich` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_render_record_rich` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 66-74: Implement helper `_render_record_text` / 实现辅助函数 `_render_record_text`
```python
def _render_record_text(record: _OutputRecord) -> str:
    body: str = record._format_body()

    log_suffix: str = _format_log_lines_text(errors=record.errors, infos=record.infos)

    if log_suffix:
        body += "\n" + log_suffix

    return body
```
**EN:** Function `_render_record_text` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_render_record_text` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 77-87: Implement helper `_format_log_lines_rich` / 实现辅助函数 `_format_log_lines_rich`
```python
def _format_log_lines_rich(
    *, errors: list[ErrorLog], infos: list[InfoLog]
) -> list[str]:
    lines: list[str] = []

    if errors:
        lines.extend(f"  [red]✗ {e.to_text()}[/]" for e in errors)
    if infos:
        lines.extend(f"  [dim]ℹ {i.to_text()}[/]" for i in infos)

    return lines
```
**EN:** Function `_format_log_lines_rich` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_log_lines_rich` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 90-98: Implement helper `_format_log_lines_text` / 实现辅助函数 `_format_log_lines_text`
```python
def _format_log_lines_text(*, errors: list[ErrorLog], infos: list[InfoLog]) -> str:
    lines: list[str] = []

    if errors:
        lines.extend(f"  ✗ {e.to_text()}" for e in errors)
    if infos:
        lines.extend(f"  ℹ {i.to_text()}" for i in infos)

    return "\n".join(lines)
```
**EN:** Function `_format_log_lines_text` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_log_lines_text` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 104-105: Implement helper `_format_config_body` / 实现辅助函数 `_format_config_body`
```python
def _format_config_body(record: ConfigRecord) -> str:
    return f"Config: {record.config}"
```
**EN:** Function `_format_config_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_config_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 108-112: Implement helper `_format_config_rich_body` / 实现辅助函数 `_format_config_rich_body`
```python
def _format_config_rich_body(
    record: ConfigRecord, verbosity: Verbosity = "normal"
) -> RenderableType:
    lines: list[str] = [f"  [bold]{k}[/] : {v}" for k, v in record.config.items()]
    return Panel("\n".join(lines), title="Comparator Config", border_style="cyan")
```
**EN:** Function `_format_config_rich_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_config_rich_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 118-131: Implement helper `_format_skip_body` / 实现辅助函数 `_format_skip_body`
```python
def _format_skip_body(record: ComparisonSkipRecord) -> str:
    text: str = (
        f"Skip: {record.name}{record._format_location_suffix()} ({record.reason})"
    )
    if record.available_side is not None and record.available_tensor_info is not None:
        info = record.available_tensor_info
        text += f"\n  {record.available_side}: shape={info.shape} dtype={info.dtype}"
        text += (
            f" mean={info.stats.mean:.4f} std={info.stats.std:.4f}"
            f" range=[{info.stats.min:.4f}, {info.stats.max:.4f}]"
        )
        if info.sample is not None:
            text += f"\n  sample: {info.sample}"
    return text
```
**EN:** Function `_format_skip_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_skip_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 134-183: Implement helper `_format_skip_rich_body` / 实现辅助函数 `_format_skip_rich_body`
```python
def _format_skip_rich_body(
    record: ComparisonSkipRecord, verbosity: Verbosity = "normal"
) -> RenderableType:
    suffix: str = record._format_location_suffix()
    header: str = (
        f"[dim]⊘ {escape(record.name)}{suffix} ── skipped ({escape(record.reason)})[/]"
    )

    if (
        verbosity == "minimal"
        or record.available_side is None
        or record.available_tensor_info is None
    ):
        return header

    info = record.available_tensor_info
    side: str = record.available_side
    dtype_str: str = info.dtype.replace("torch.", "")

    lines: list[str] = [header]

    # Bundle info line
    if record.available_bundle_info is not None:
        bi = record.available_bundle_info
        shapes: list[list[int]] = [f.shape for f in bi.files]
        unique_shapes: set[str] = {str(s) for s in shapes}
        shape_desc: str = (
            escape(str(shapes[0])) if len(unique_shapes) == 1 else "mixed shapes"
        )
        dims_part: str = f"  [dim]dims: {bi.dims}[/]" if bi.dims else ""
        lines.append(
            f"   {side:8s}  [cyan]{bi.num_files} files[/]"
            f" × {shape_desc} {dtype_str}{dims_part}"
        )
    else:
        lines.append(f"   {side:8s}  {escape(str(info.shape))} {dtype_str}")

    # Stats line (compact single-side)
    stats = info.stats
    range_str: str = escape(f"[{stats.min:.4f}, {stats.max:.4f}]")
    lines.append(
        f"   [dim]stats[/]     mean={stats.mean:.4f}  std={stats.std:.4f}"
        f"  range={range_str}"
    )

    # Sample
    if info.sample is not None:
        lines.append(f"   [dim]sample[/]    {escape(info.sample)}")

    return "\n".join(lines)
```
**EN:** Function `_format_skip_rich_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_skip_rich_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 189-195: Implement helper `_format_error_body` / 实现辅助函数 `_format_error_body`
```python
def _format_error_body(record: ComparisonErrorRecord) -> str:
    prefix: str = record._format_location_prefix()
    return (
        f"{prefix}Error: {record.name} ({record.exception_type})\n"
        f"{record.exception_message}\n"
        f"{record.traceback_str}"
    )
```
**EN:** Function `_format_error_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_error_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 198-209: Implement helper `_format_error_rich_body` / 实现辅助函数 `_format_error_rich_body`
```python
def _format_error_rich_body(
    record: ComparisonErrorRecord, verbosity: Verbosity = "normal"
) -> RenderableType:
    prefix: str = record._format_location_prefix_rich()
    name: str = escape(record.name)
    header: str = (
        f"{prefix}[bold red]{name} ── errored ({escape(record.exception_type)}): "
        f"{escape(record.exception_message)}[/]"
    )
    if verbosity == "minimal":
        return header
    return header + f"\n[dim]{escape(record.traceback_str)}[/]"
```
**EN:** Function `_format_error_rich_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_error_rich_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 215-222: Implement helper `_format_table_body` / 实现辅助函数 `_format_table_body`
```python
def _format_table_body(record: _TableRecord) -> str:
    import polars as pl

    from sglang.srt.debug_utils.comparator.display import _render_polars_as_text

    return _render_polars_as_text(
        pl.DataFrame(record.rows), title=record._table_title()
    )
```
**EN:** Function `_format_table_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_table_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 225-236: Implement helper `_format_table_rich_body` / 实现辅助函数 `_format_table_rich_body`
```python
def _format_table_rich_body(
    record: _TableRecord, verbosity: Verbosity = "normal"
) -> RenderableType:
    import polars as pl

    from sglang.srt.debug_utils.comparator.display import (
        _render_polars_as_rich_table,
    )

    return _render_polars_as_rich_table(
        pl.DataFrame(record.rows), title=record._table_title()
    )
```
**EN:** Function `_format_table_rich_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_table_rich_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 242-248: Implement helper `_format_tensor_comparison_body` / 实现辅助函数 `_format_tensor_comparison_body`
```python
def _format_tensor_comparison_body(record: ComparisonTensorRecord) -> str:
    body: str = record._format_location_prefix() + format_comparison(record)
    if record.replicated_checks:
        body += "\n" + format_replicated_checks(record.replicated_checks)
    if record.traced_plan is not None:
        body += "\n" + _format_aligner_plan(record.traced_plan)
    return body
```
**EN:** Function `_format_tensor_comparison_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_tensor_comparison_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 251-260: Implement helper `_format_tensor_comparison_rich_body` / 实现辅助函数 `_format_tensor_comparison_rich_body`
```python
def _format_tensor_comparison_rich_body(
    record: ComparisonTensorRecord, verbosity: Verbosity = "normal"
) -> RenderableType:
    from sglang.srt.debug_utils.comparator.tensor_comparator.formatter import (
        format_comparison_rich,
    )

    return record._format_location_prefix_rich() + format_comparison_rich(
        record=record, verbosity=verbosity
    )
```
**EN:** Function `_format_tensor_comparison_rich_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_tensor_comparison_rich_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 266-274: Implement helper `_format_non_tensor_body` / 实现辅助函数 `_format_non_tensor_body`
```python
def _format_non_tensor_body(record: ComparisonNonTensorRecord) -> str:
    suffix: str = record._format_location_suffix()
    if record.values_equal:
        return f"NonTensor: {record.name}{suffix} = {record.baseline_value} ({record.baseline_type}) [equal]"
    return (
        f"NonTensor: {record.name}{suffix}\n"
        f"  baseline = {record.baseline_value} ({record.baseline_type})\n"
        f"  target   = {record.target_value} ({record.target_type})"
    )
```
**EN:** Function `_format_non_tensor_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_non_tensor_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 277-294: Implement helper `_format_non_tensor_rich_body` / 实现辅助函数 `_format_non_tensor_rich_body`
```python
def _format_non_tensor_rich_body(
    record: ComparisonNonTensorRecord, verbosity: Verbosity = "normal"
) -> RenderableType:
    suffix: str = record._format_location_suffix()
    name: str = escape(record.name)
    baseline_val: str = escape(record.baseline_value)
    target_val: str = escape(record.target_value)

    if record.values_equal:
        return (
            f"═ {name}{suffix} = {baseline_val} "
            f"({record.baseline_type}) [green]✓[/]"
        )
    return (
        f"═ [bold red]{name}{suffix}[/]\n"
        f"  baseline = {baseline_val} ({record.baseline_type})\n"
        f"  target   = {target_val} ({record.target_type})"
    )
```
**EN:** Function `_format_non_tensor_rich_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_non_tensor_rich_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 300-307: Implement helper `_format_summary_body` / 实现辅助函数 `_format_summary_body`
```python
def _format_summary_body(record: SummaryRecord) -> str:
    text: str = (
        f"Summary: {record.passed} passed, {record.failed} failed, "
        f"{record.skipped} skipped (total {record.total})"
    )
    if record.errored > 0:
        text += f", {record.errored} errored"
    return text
```
**EN:** Function `_format_summary_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_summary_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 310-321: Implement helper `_format_summary_rich_body` / 实现辅助函数 `_format_summary_rich_body`
```python
def _format_summary_rich_body(
    record: SummaryRecord, verbosity: Verbosity = "normal"
) -> RenderableType:
    text: str = (
        f"[bold green]{record.passed} passed[/] │ "
        f"[bold red]{record.failed} failed[/] │ "
        f"[yellow]{record.skipped} skipped[/] │ "
        f"{record.total} total"
    )
    if record.errored > 0:
        text += f" │ [bold red]{record.errored} errored[/]"
    return Panel(text, title="SUMMARY", border_style="bold")
```
**EN:** Function `_format_summary_rich_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_summary_rich_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 327-328: Implement helper `_format_log_body` / 实现辅助函数 `_format_log_body`
```python
def _format_log_body(record: LogRecord) -> str:
    return ""
```
**EN:** Function `_format_log_body` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_log_body` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 334-356: Implement helper `_format_aligner_plan` / 实现辅助函数 `_format_aligner_plan`
```python
def _format_aligner_plan(traced_plan: TracedAlignerPlan) -> str:
    lines: list[str] = ["Aligner Plan:"]

    for side_label, traced_side in [
        ("baseline", traced_plan.per_side.x),
        ("target", traced_plan.per_side.y),
    ]:
        if not traced_side.step_plans:
            lines.append(f"  {side_label}: (no steps)")
            continue

        step_summaries: list[str] = []
        for traced_step in traced_side.step_plans:
            sub_strs: list[str] = [
                _format_sub_plan_text(traced_sub)
                for traced_sub in traced_step.sub_plans
            ]
            summary: str = ", ".join(sub_strs) if sub_strs else "passthrough"
            step_summaries.append(f"step={traced_step.step}: {summary}")
        lines.append(f"  {side_label}: [{'; '.join(step_summaries)}]")

    lines.extend(_format_cross_side_plan_text(traced_plan.plan))
    return "\n".join(lines)
```
**EN:** Function `_format_aligner_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_aligner_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 359-380: Implement helper `_format_sub_plan_text` / 实现辅助函数 `_format_sub_plan_text`
```python
def _format_sub_plan_text(traced_sub: TracedSubPlan) -> str:
    from sglang.srt.debug_utils.comparator.aligner.reorderer.types import ReordererPlan
    from sglang.srt.debug_utils.comparator.aligner.unsharder.types import UnsharderPlan

    sub = traced_sub.plan
    qualifier: str = ""
    if isinstance(sub, UnsharderPlan):
        qualifier = f"({sub.axis.value})"
    elif isinstance(sub, ReordererPlan):
        qualifier = f"({sub.params.op})"

    sub_desc: str = f"{sub.type}{qualifier}"

    if traced_sub.snapshot is not None:
        snap = traced_sub.snapshot
        in_count: int = len(snap.input_shapes)
        out_count: int = len(snap.output_shapes)
        in_shape: str = str(snap.input_shapes[0]) if snap.input_shapes else "?"
        out_shape: str = str(snap.output_shapes[0]) if snap.output_shapes else "?"
        sub_desc += f" {in_count}x{in_shape} -> {out_count}x{out_shape}"

    return sub_desc
```
**EN:** Function `_format_sub_plan_text` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_sub_plan_text` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 383-398: Implement helper `_format_cross_side_plan_text` / 实现辅助函数 `_format_cross_side_plan_text`
```python
def _format_cross_side_plan_text(plan: AlignerPlan) -> list[str]:
    lines: list[str] = []

    if plan.token_aligner_plan is not None:
        num_tokens: int = len(plan.token_aligner_plan.locators.x.steps)
        lines.append(f"  token_aligner: {num_tokens} tokens aligned")

    if plan.axis_aligner_plan is not None:
        parts: list[str] = []
        if plan.axis_aligner_plan.pattern.x:
            parts.append(f"x: {plan.axis_aligner_plan.pattern.x}")
        if plan.axis_aligner_plan.pattern.y:
            parts.append(f"y: {plan.axis_aligner_plan.pattern.y}")
        lines.append(f"  axis_aligner: {', '.join(parts)}")

    return lines
```
**EN:** Function `_format_cross_side_plan_text` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_cross_side_plan_text` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_render_record_rich`, `_render_record_text`, `_format_log_lines_rich`, `_format_log_lines_text`, `_format_config_body`, `_format_config_rich_body`, `_format_skip_body`, `_format_skip_rich_body`, `_format_error_body`, `_format_error_rich_body`, `_format_table_body`, `_format_table_rich_body`
- **Module role / 模块角色**: Comparison result visualization / 比较结果可视化
- **Implementation focus / 实现重点**: Turns raw comparison state into readable debug output / 把原始比较状态转换为可读的调试输出

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方**: `rich`, `polars`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.tensor_comparator.formatter`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.display`, `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`
