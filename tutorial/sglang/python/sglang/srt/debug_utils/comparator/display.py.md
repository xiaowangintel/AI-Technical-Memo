# display.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/display.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on comparison result visualization. It mainly turns raw comparison state into readable debug output. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于比较结果可视化。它主要用于把原始比较状态转换为可读的调试输出。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from collections import defaultdict
from io import StringIO
from pathlib import Path
from typing import Any, Optional

import polars as pl

from sglang.srt.debug_utils.comparator.output_types import (
    InputIdsRecord,
    RankInfoRecord,
)
from sglang.srt.debug_utils.comparator.report_sink import report_sink
from sglang.srt.debug_utils.dump_loader import LOAD_FAILED, ValueWithMeta
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 17-17: Declare module-level symbols such as `PARALLEL_INFO_KEYS` / 声明模块级符号，例如 `PARALLEL_INFO_KEYS`
```python
PARALLEL_INFO_KEYS: list[str] = ["sglang_parallel_info", "megatron_parallel_info"]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 20-37: Implement function `emit_display_records` / 实现函数 `emit_display_records`
```python
def emit_display_records(
    *,
    df: pl.DataFrame,
    dump_dir: Path,
    label: str,
    tokenizer: Any,
) -> None:
    rank_rows: Optional[list[dict[str, Any]]] = _collect_rank_info(
        df, dump_dir=dump_dir
    )
    if rank_rows is not None:
        report_sink.add(RankInfoRecord(label=label, rows=rank_rows))

    input_ids_rows: Optional[list[dict[str, Any]]] = _collect_input_ids_and_positions(
        df, dump_dir=dump_dir, tokenizer=tokenizer
    )
    if input_ids_rows is not None:
        report_sink.add(InputIdsRecord(label=label, rows=input_ids_rows))
```
**EN:** Function `emit_display_records` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `emit_display_records` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 40-52: Implement helper `_render_polars_as_text` / 实现辅助函数 `_render_polars_as_text`
```python
def _render_polars_as_text(df: pl.DataFrame, *, title: Optional[str] = None) -> str:
    from rich.console import Console
    from rich.table import Table

    table = Table(title=title)
    for col in df.columns:
        table.add_column(col)
    for row in df.iter_rows():
        table.add_row(*[str(v) for v in row])

    buf = StringIO()
    Console(file=buf, force_terminal=False, width=200).print(table)
    return buf.getvalue().rstrip("\n")
```
**EN:** Function `_render_polars_as_text` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_render_polars_as_text` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 55-65: Implement helper `_render_polars_as_rich_table` / 实现辅助函数 `_render_polars_as_rich_table`
```python
def _render_polars_as_rich_table(
    df: pl.DataFrame, *, title: Optional[str] = None
) -> Any:
    from rich.table import Table

    table = Table(title=title)
    for col in df.columns:
        table.add_column(col)
    for row in df.iter_rows():
        table.add_row(*[str(v) for v in row])
    return table
```
**EN:** Function `_render_polars_as_rich_table` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_render_polars_as_rich_table` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 68-88: Implement helper `_collect_rank_info` / 实现辅助函数 `_collect_rank_info`
```python
def _collect_rank_info(
    df: pl.DataFrame, dump_dir: Path
) -> Optional[list[dict[str, Any]]]:
    unique_rows: pl.DataFrame = (
        df.filter(pl.col("name") == "input_ids")
        .sort("rank")
        .unique(subset=["rank"], keep="first")
    )
    if unique_rows.is_empty():
        return None

    table_rows: list[dict[str, Any]] = []
    for row in unique_rows.to_dicts():
        meta: dict[str, Any] = ValueWithMeta.load(dump_dir / row["filename"]).meta

        row_data: dict[str, Any] = {"rank": row["rank"]}
        for key in PARALLEL_INFO_KEYS:
            _extract_parallel_info(row_data=row_data, info=meta.get(key, {}))
        table_rows.append(row_data)

    return table_rows or None
```
**EN:** Function `_collect_rank_info` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_collect_rank_info` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 91-132: Implement helper `_collect_input_ids_and_positions` / 实现辅助函数 `_collect_input_ids_and_positions`
```python
def _collect_input_ids_and_positions(
    df: pl.DataFrame,
    dump_dir: Path,
    *,
    tokenizer: Any = None,
) -> Optional[list[dict[str, Any]]]:
    filtered: pl.DataFrame = df.filter(pl.col("name").is_in(["input_ids", "positions"]))
    if filtered.is_empty():
        return None

    data_by_step_rank: dict[tuple[int, int], dict[str, Any]] = defaultdict(dict)
    for row in filtered.to_dicts():
        key: tuple[int, int] = (row["step"], row["rank"])
        item: ValueWithMeta = ValueWithMeta.load(dump_dir / row["filename"])
        if item.value is not LOAD_FAILED:
            data_by_step_rank[key][row["name"]] = item.value

    table_rows: list[dict[str, Any]] = []
    for (step, rank), data in sorted(data_by_step_rank.items()):
        ids = data.get("input_ids")
        pos = data.get("positions")

        ids_list: Optional[list[int]] = (
            ids.flatten().tolist() if ids is not None else None
        )

        row_data: dict[str, Any] = {
            "step": step,
            "rank": rank,
            "num_tokens": len(ids_list) if ids_list is not None else None,
            "input_ids": str(ids_list) if ids_list is not None else "N/A",
            "positions": str(pos.flatten().tolist()) if pos is not None else "N/A",
        }

        if tokenizer is not None and ids_list is not None:
            row_data["decoded_text"] = repr(
                tokenizer.decode(ids_list, skip_special_tokens=False)
            )

        table_rows.append(row_data)

    return table_rows or None
```
**EN:** Function `_collect_input_ids_and_positions` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_collect_input_ids_and_positions` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 135-144: Implement helper `_extract_parallel_info` / 实现辅助函数 `_extract_parallel_info`
```python
def _extract_parallel_info(row_data: dict[str, Any], info: dict[str, Any]) -> None:
    if not info or info.get("error"):
        return

    for key in sorted(info.keys()):
        if key.endswith("_rank"):
            base: str = key[:-5]
            size_key: str = f"{base}_size"
            if size_key in info:
                row_data[base] = f"{info[key]}/{info[size_key]}"
```
**EN:** Function `_extract_parallel_info` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_extract_parallel_info` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `emit_display_records`, `_render_polars_as_text`, `_render_polars_as_rich_table`, `_collect_rank_info`, `_collect_input_ids_and_positions`, `_extract_parallel_info`
- **Module role / 模块角色**: Comparison result visualization / 比较结果可视化
- **Implementation focus / 实现重点**: Turns raw comparison state into readable debug output / 把原始比较状态转换为可读的调试输出

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `collections`, `io`, `pathlib`, `typing`
- **Third-party / 第三方**: `polars`, `rich`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.report_sink`, `sglang.srt.debug_utils.dump_loader`
