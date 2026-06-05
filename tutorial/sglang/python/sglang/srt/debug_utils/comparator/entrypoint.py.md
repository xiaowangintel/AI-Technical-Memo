# entrypoint.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/entrypoint.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on top-level debug workflow entrypoints. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于顶层调试工作流入口。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-47: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

import argparse
import sys
import traceback as _traceback_module
from pathlib import Path
from typing import Any, Iterator, Optional, Union

import polars as pl

from sglang.srt.debug_utils.comparator.aligner.token_aligner.entrypoint import (
    TokenAlignerResult,
    compute_maybe_token_aligner_result,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader import (
    AUX_NAMES,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerPlan,
)
from sglang.srt.debug_utils.comparator.bundle_comparator import compare_bundle_pair
from sglang.srt.debug_utils.comparator.bundle_matcher import (
    TensorBundleInfo,
    match_bundles,
)
from sglang.srt.debug_utils.comparator.display import emit_display_records
from sglang.srt.debug_utils.comparator.meta_overrider import MetaOverrider
from sglang.srt.debug_utils.comparator.output_types import (
    ComparisonErrorRecord,
    ComparisonNonTensorRecord,
    ComparisonSkipRecord,
    ComparisonTensorRecord,
    ConfigRecord,
    RecordLocation,
    SummaryRecord,
)
from sglang.srt.debug_utils.comparator.per_token_visualizer import (
    generate_per_token_heatmap,
)
from sglang.srt.debug_utils.comparator.preset import PRESETS, expand_preset
from sglang.srt.debug_utils.comparator.report_sink import report_sink
from sglang.srt.debug_utils.comparator.utils import (
    Pair,
    auto_descend_dir,
    compute_exit_code,
)
from sglang.srt.debug_utils.dump_loader import read_meta, read_tokenizer_path
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 49-57: Declare module-level symbols such as `_DEFAULT_SKIP_KEYS`, `_DIMS_DEBUG_HINT` / 声明模块级符号，例如 `_DEFAULT_SKIP_KEYS`, `_DIMS_DEBUG_HINT`
```python
_DEFAULT_SKIP_KEYS: set[str] = {"dump_index", "filename"}

_DIMS_DEBUG_HINT: str = (
    "\nHint: If this is a dims annotation issue, do NOT re-run expensive dumps.\n"
    "Use --override-dims at comparison time, e.g.:\n"
    '  python -m sglang.srt.debug_utils.comparator --override-dims "tensor_name:b s h[tp] d"\n'
    "(Use --override-baseline-dims / --override-target-dims for per-side overrides.\n"
    " Use --override-config for bulk overrides via YAML file.)"
)
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 60-62: Implement function `main` / 实现函数 `main`
```python
def main() -> None:
    args = parse_args(sys.argv[1:])
    sys.exit(run(args))
```
**EN:** Function `main` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `main` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 65-166: Implement function `run` / 实现函数 `run`
```python
def run(args: argparse.Namespace) -> int:
    report_sink.configure(
        output_format=args.output_format,
        report_path=None,
        verbosity=args.verbosity,
    )

    dir_pair: Pair[Path] = Pair(
        x=auto_descend_dir(Path(args.baseline_path), label="baseline_path"),
        y=auto_descend_dir(Path(args.target_path), label="target_path"),
    )
    viz_output_dir: Optional[Path] = (
        Path(args.viz_output_dir) if args.viz_bundle_details else None
    )
    visualize_per_token: Optional[Path] = (
        Path(args.visualize_per_token) if args.visualize_per_token else None
    )
    override_config: Optional[Path] = (
        Path(args.override_config) if args.override_config else None
    )

    report_path: Optional[Path] = _resolve_report_path(
        target_path=dir_pair.y,
        report_path_arg=args.report_path,
    )
    report_sink.configure(
        output_format=args.output_format,
        report_path=report_path,
        verbosity=args.verbosity,
    )

    try:
        report_sink.add(ConfigRecord(config=vars(args)))

        dfs: Pair[pl.DataFrame] = _read_df(
            dir_pair=dir_pair,
            start_step=args.start_step,
            end_step=args.end_step,
            filter_pattern=args.filter,
        )

        tokenizer: Any = _maybe_load_tokenizer(
            tokenizer_arg=args.tokenizer, dir_pair=dir_pair
        )
        for label, df, dump_dir in [
            ("baseline", dfs.x, dir_pair.x),
            ("target", dfs.y, dir_pair.y),
        ]:
            emit_display_records(
                df=df, dump_dir=dump_dir, label=label, tokenizer=tokenizer
# ... truncated for brevity ...
            meta_overrider=meta_overrider,
        )
        summary, skipped_names, failed_names, errored_names = (
            _consume_comparison_records(
                comparison_records=comparison_records,
                visualize_per_token=visualize_per_token,
            )
        )
        return compute_exit_code(
            summary,
            allow_skipped_pattern=args.allow_skipped_pattern,
            skipped_names=skipped_names,
            allow_failed_pattern=args.allow_failed_pattern,
            failed_names=failed_names,
            errored_names=errored_names,
        )
    finally:
        report_sink.close()
        if report_path is not None:
            print(f"Report: {report_path}", file=sys.stderr)
```
**EN:** Function `run` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps. The snippet is abbreviated to keep the analysis readable.
**CN:** 函数 `run` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。 为了保持分析可读性，这里的代码片段做了节选。

### Lines 169-174: Implement helper `_resolve_report_path` / 实现辅助函数 `_resolve_report_path`
```python
def _resolve_report_path(
    *, target_path: Path, report_path_arg: Optional[str]
) -> Optional[Path]:
    if report_path_arg is not None:
        return Path(report_path_arg) if report_path_arg else None
    return target_path / "comparator_report.jsonl"
```
**EN:** Function `_resolve_report_path` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_resolve_report_path` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 177-194: Implement helper `_maybe_load_tokenizer` / 实现辅助函数 `_maybe_load_tokenizer`
```python
def _maybe_load_tokenizer(*, tokenizer_arg: Optional[str], dir_pair: Pair[Path]) -> Any:
    tokenizer_path: Optional[str] = tokenizer_arg

    if tokenizer_path is None:
        for directory in [dir_pair.x, dir_pair.y]:
            tokenizer_path = read_tokenizer_path(directory)
            if tokenizer_path is not None:
                break

    if tokenizer_path is None:
        return None

    try:
        from transformers import AutoTokenizer

        return AutoTokenizer.from_pretrained(tokenizer_path)
    except Exception:
        return None
```
**EN:** Function `_maybe_load_tokenizer` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_maybe_load_tokenizer` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 197-214: Implement helper `_read_df` / 实现辅助函数 `_read_df`
```python
def _read_df(
    *,
    dir_pair: Pair[Path],
    start_step: int,
    end_step: int,
    filter_pattern: Optional[str],
) -> Pair[pl.DataFrame]:
    df_baseline = read_meta(dir_pair.x)

    df_target = read_meta(dir_pair.y)
    df_target = df_target.filter(
        (pl.col("step") >= start_step) & (pl.col("step") <= end_step)
    )
    if filter_pattern:
        df_target = df_target.filter(pl.col("filename").str.contains(filter_pattern))
    assert all(c in df_target.columns for c in ["rank", "step", "dump_index", "name"])

    return Pair(x=df_baseline, y=df_target)
```
**EN:** Function `_read_df` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_read_df` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 217-278: Implement helper `_compare_bundle_pairs` / 实现辅助函数 `_compare_bundle_pairs`
```python
def _compare_bundle_pairs(
    *,
    bundle_info_pairs: list[Pair[TensorBundleInfo]],
    dir_pair: Pair[Path],
    token_aligner_mode: Optional[str],
    token_aligner_plan: Optional[TokenAlignerPlan],
    diff_threshold: float,
    thd_seq_lens_by_step_pair: Pair[Optional[dict[int, list[int]]]],
    viz_output_dir: Optional[Path] = None,
    compute_per_token: bool = False,
    meta_overrider: Optional[MetaOverrider] = None,
) -> Iterator[
    Union[
        ComparisonTensorRecord,
        ComparisonSkipRecord,
        ComparisonNonTensorRecord,
        ComparisonErrorRecord,
    ]
]:
    for bundle_info_pair in bundle_info_pairs:
        if not bundle_info_pair.y:
            continue

        name: str = bundle_info_pair.y[0].name
        filenames_pair: Pair[list[str]] = bundle_info_pair.map(
            lambda infos: [info.filename for info in infos]
        )

        record: Union[
            ComparisonTensorRecord,
            ComparisonSkipRecord,
            ComparisonNonTensorRecord,
            ComparisonErrorRecord,
        ]
        try:
            record = compare_bundle_pair(
                name=name,
                filenames_pair=filenames_pair,
                dir_pair=dir_pair,
                token_aligner_mode=token_aligner_mode,
                token_aligner_plan=token_aligner_plan,
                diff_threshold=diff_threshold,
                thd_seq_lens_by_step_pair=thd_seq_lens_by_step_pair,
                viz_output_dir=viz_output_dir,
                compute_per_token=compute_per_token,
                meta_overrider=meta_overrider,
            )
        except Exception as exc:
            tb = _traceback_module.format_exc()
            record = ComparisonErrorRecord(
                name=name,
                exception_type=type(exc).__name__,
                exception_message=str(exc),
                traceback_str=f"{_DIMS_DEBUG_HINT}\n\n{tb}",
            )

        target_steps: set[int] = {info.step for info in bundle_info_pair.y}
        step: Optional[int] = target_steps.pop() if len(target_steps) == 1 else None
        if step is not None:
            record = record.model_copy(update={"location": RecordLocation(step=step)})

        yield record
```
**EN:** Function `_compare_bundle_pairs` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compare_bundle_pairs` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 281-322: Implement helper `_consume_comparison_records` / 实现辅助函数 `_consume_comparison_records`
```python
def _consume_comparison_records(
    *,
    comparison_records: Iterator[
        Union[
            ComparisonTensorRecord,
            ComparisonSkipRecord,
            ComparisonNonTensorRecord,
            ComparisonErrorRecord,
        ]
    ],
    visualize_per_token: Optional[Path] = None,
) -> tuple[SummaryRecord, list[str], list[str], list[str]]:
    counts: dict[str, int] = {"passed": 0, "failed": 0, "skipped": 0, "errored": 0}
    collected_comparisons: list[ComparisonTensorRecord] = []
    skipped_names: list[str] = []
    failed_names: list[str] = []
    errored_names: list[str] = []

    for record in comparison_records:
        counts[record.category] += 1
        report_sink.add(record)
        if isinstance(record, ComparisonSkipRecord) and record.category == "skipped":
            skipped_names.append(record.name)
        if record.category == "failed":
            failed_names.append(record.name)
        if isinstance(record, ComparisonErrorRecord):
            errored_names.append(record.name)
        if visualize_per_token is not None and isinstance(
            record, ComparisonTensorRecord
        ):
            collected_comparisons.append(record)

    summary: SummaryRecord = SummaryRecord(total=sum(counts.values()), **counts)
    report_sink.add(summary)

    if visualize_per_token is not None and collected_comparisons:
        generate_per_token_heatmap(
            records=collected_comparisons,
            output_path=visualize_per_token,
        )

    return summary, skipped_names, failed_names, errored_names
```
**EN:** Function `_consume_comparison_records` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_consume_comparison_records` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 325-451: Implement function `parse_args` / 实现函数 `parse_args`
```python
def parse_args(argv: list[str]) -> argparse.Namespace:
    """Parse CLI arguments from an argv list. Applies preset expansion."""
    argv = expand_preset(argv, presets=PRESETS)

    parser = argparse.ArgumentParser()
    parser.add_argument("--baseline-path", type=str)
    parser.add_argument("--target-path", type=str)
    parser.add_argument("--start-step", type=int, default=0)
    parser.add_argument("--end-step", type=int, default=1000000)
    parser.add_argument("--diff-threshold", type=float, default=1e-3)
    parser.add_argument(
        "--filter", type=str, default=None, help="Regex to filter filenames (include)"
    )
    parser.add_argument(
        "--output-format",
        type=str,
        choices=["text", "json"],
        default="text",
        help="Output format: text (default) or json (JSONL, one JSON object per line)",
    )
    parser.add_argument(
        "--verbosity",
        type=str,
        choices=["minimal", "normal", "verbose"],
        default="normal",
        help="Output verbosity: minimal (1 line per tensor), normal (compact lifecycle), "
        "verbose (full detail). Default: normal",
    )
    parser.add_argument(
        "--preset",
        type=str,
        choices=list(PRESETS.keys()),
        default=None,
        help="Preset configuration (expanded before parsing). "
        f"Available: {list(PRESETS.keys())}",
    )
    parser.add_argument(
        "--grouping-skip-keys",
        nargs="*",
        default=None,
        help="Metadata keys to skip when grouping bundles (additive on top of "
        "always-skipped dump_index and filename). "
        "E.g. '--grouping-skip-keys rank step' skips rank and step.",
    )
    parser.add_argument(
        "--token-aligner",
        type=str,
        choices=["smart", "concat_steps"],
        default=None,
        help="Token aligner mode: concat_steps (BS=1, no aux needed) or smart (BS>1, sequence matching). "
# ... truncated for brevity ...
        "Default '.*' allows all skips. Use '^$' to forbid all skips.",
    )
    parser.add_argument(
        "--allow-failed-pattern",
        type=str,
        default=None,
        help="Regex pattern for tensor names allowed to fail without affecting exit code. "
        "Default None (all failures affect exit code).",
    )

    # Report output
    parser.add_argument(
        "--report-path",
        type=str,
        default=None,
        help="Path for JSONL report (default: <target-path>/comparator_report.jsonl). "
        "Pass empty string '' to disable.",
    )

    return parser.parse_args(argv)
```
**EN:** Function `parse_args` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps. The snippet is abbreviated to keep the analysis readable.
**CN:** 函数 `parse_args` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。 为了保持分析可读性，这里的代码片段做了节选。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `main`, `run`, `_resolve_report_path`, `_maybe_load_tokenizer`, `_read_df`, `_compare_bundle_pairs`, `_consume_comparison_records`, `parse_args`
- **Module role / 模块角色**: Top-level debug workflow entrypoints / 顶层调试工作流入口
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `argparse`, `sys`, `traceback`, `pathlib`, `typing`
- **Third-party / 第三方**: `polars`, `transformers`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.token_aligner.entrypoint`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.bundle_comparator`, `sglang.srt.debug_utils.comparator.bundle_matcher`, `sglang.srt.debug_utils.comparator.display`, `sglang.srt.debug_utils.comparator.meta_overrider`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.per_token_visualizer`, `sglang.srt.debug_utils.comparator.preset`, `sglang.srt.debug_utils.comparator.report_sink`, `sglang.srt.debug_utils.comparator.utils`, `sglang.srt.debug_utils.dump_loader`
