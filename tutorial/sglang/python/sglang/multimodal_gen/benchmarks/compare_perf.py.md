# compare_perf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/benchmarks/compare_perf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `calculate_diff`, `calculate_upper_bound`, and `calculate_lower_bound`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于多模态生成模块。它围绕 `calculate_diff`、`calculate_upper_bound` 和 `calculate_lower_bound` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: module setup and imports / 模块初始化与导入
```python
import argparse
import json
import os
import re
from datetime import datetime
from typing import Any, Dict, List, Tuple
```
**EN:** This block establishes the module context and imports `argparse`, `json`, `os`, `re`, `datetime`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `argparse`、`json`、`os`、`re`、`datetime` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 9-16: `calculate_diff` implementation / `calculate_diff` 实现
```python
def calculate_diff(base: float, new: float) -> Tuple[float, float]:
    """Returns (diff, diff_percent)."""
    diff = new - base
    if base == 0:
        percent = 0.0
    else:
        percent = (diff / base) * 100
    return diff, percent
```
**EN:** This block defines function `calculate_diff`. Returns (diff, diff_percent). The implementation branches on conditions. Parameters such as `base`, and `new` drive the behavior in this section.
**CN:** 该代码块定义了函数 `calculate_diff`。 它用于处理 calculate diff 相关逻辑。 实现中包含条件分支。 本段逻辑主要由 `base` 和 `new` 等参数驱动。

### Lines 19-23: `calculate_upper_bound` implementation / `calculate_upper_bound` 实现
```python
def calculate_upper_bound(baseline: float, rel_tol: float, min_abs_tol: float) -> float:
    """Calculates the upper bound for performance regression check."""
    rel_limit = baseline * (1 + rel_tol)
    abs_limit = baseline + min_abs_tol
    return max(rel_limit, abs_limit)
```
**EN:** This block defines function `calculate_upper_bound`. Calculates the upper bound for performance regression check. Key calls include `max`. Parameters such as `baseline`, `rel_tol`, and `min_abs_tol` drive the behavior in this section.
**CN:** 该代码块定义了函数 `calculate_upper_bound`。 它用于处理 calculate upper bound 相关逻辑。 关键调用包括 `max`。 本段逻辑主要由 `baseline`、`rel_tol` 和 `min_abs_tol` 等参数驱动。

### Lines 26-30: `calculate_lower_bound` implementation / `calculate_lower_bound` 实现
```python
def calculate_lower_bound(baseline: float, rel_tol: float, min_abs_tol: float) -> float:
    """Calculates the lower bound for performance improvement check."""
    rel_lower = baseline * (1 - rel_tol)
    abs_lower = baseline - min_abs_tol
    return min(rel_lower, abs_lower)
```
**EN:** This block defines function `calculate_lower_bound`. Calculates the lower bound for performance improvement check. Key calls include `min`. Parameters such as `baseline`, `rel_tol`, and `min_abs_tol` drive the behavior in this section.
**CN:** 该代码块定义了函数 `calculate_lower_bound`。 它用于处理 calculate lower bound 相关逻辑。 关键调用包括 `min`。 本段逻辑主要由 `baseline`、`rel_tol` 和 `min_abs_tol` 等参数驱动。

### Lines 33-54: `get_perf_status_emoji` implementation / `get_perf_status_emoji` 实现
```python
def get_perf_status_emoji(
    baseline: float,
    new: float,
    rel_tol: float = 0.1,
    min_abs_tol: float = 120.0,
) -> str:
    """
    Determines the status emoji based on performance difference.

    Logic:
      Upper bound (Slower): max(baseline * (1 + rel_tol), baseline + min_abs_tol)
      Lower bound (Faster): min(baseline * (1 - rel_tol), baseline - min_abs_tol)
    """
    upper_bound = calculate_upper_bound(baseline, rel_tol, min_abs_tol)
    lower_bound = calculate_lower_bound(baseline, rel_tol, min_abs_tol)

    if new > upper_bound:
        return "🔴"
    elif new < lower_bound:
        return "🟢"
    else:
        return "⚪️"
```
**EN:** This block defines function `get_perf_status_emoji`. Determines the status emoji based on performance difference. Logic: Upper bound (Slower): max(baseline * (1 + rel_tol), baseline + min_abs_tol) Lower bound (Faster): min(baseline * (1 - rel_tol), baseline - min_abs_tol) Key calls include `calculate_upper_bound`, and `calculate_lower_bound`. The implementation branches on conditions. Parameters such as `baseline`, `new`, `rel_tol`, and `min_abs_tol` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_perf_status_emoji`。 它用于获取perf status emoji。 关键调用包括 `calculate_upper_bound` 和 `calculate_lower_bound`。 实现中包含条件分支。 本段逻辑主要由 `baseline`、`new`、`rel_tol` 和 `min_abs_tol` 等参数驱动。

### Lines 57-103: `consolidate_steps` implementation / `consolidate_steps` 实现
```python
def consolidate_steps(
    steps_list: List[Dict[str, Any]],
) -> Tuple[Dict[str, float], List[str], Dict[str, int]]:
    """
    Aggregates specific repeating steps (like denoising_step_*) into groups.
    Returns:
        - aggregated_durations: {name: duration_ms}
        - ordered_names: list of names in execution order
        - counts: {name: count_of_steps_aggregated}
    """
    durations = {}
    counts = {}
    ordered_names = []
    seen_names = set()

    # Regex for steps to group
    # Group "denoising_step_0", "denoising_step_1" -> "Denoising Loop"
    denoise_pattern = re.compile(r"^denoising_step_(\d+)$")
    denoising_group_name = "Denoising Loop"

    for step in steps_list:
        name = step.get("name", "unknown")
        dur = step.get("duration_ms", 0.0)

        match = denoise_pattern.match(name)
        if match:
            key = denoising_group_name
            if key not in durations:
                durations[key] = 0.0
                counts[key] = 0
                if key not in seen_names:
                    ordered_names.append(key)
                    seen_names.add(key)
            durations[key] += dur
            counts[key] += 1
        else:
            # Standard stage (preserve order)
            if name not in durations:
                durations[name] = 0.0
                counts[name] = 0
                if name not in seen_names:
                    ordered_names.append(name)
                    seen_names.add(name)
            durations[name] += dur
            counts[name] += 1

    return durations, ordered_names, counts
```
**EN:** This block defines function `consolidate_steps`. Aggregates specific repeating steps (like denoising_step_*) into groups. Returns: - aggregated_durations: {name: duration_ms} - ordered_names: list of names in execution order - counts: {name: count_of_steps_aggregated} Key calls include `set`, `re.compile`, `step.get`, `denoise_pattern.match`, and `ordered_names.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `steps_list` drive the behavior in this section.
**CN:** 该代码块定义了函数 `consolidate_steps`。 它用于处理 consolidate steps 相关逻辑。 关键调用包括 `set`、`re.compile`、`step.get`、`denoise_pattern.match` 和 `ordered_names.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `steps_list` 等参数驱动。

### Lines 106-109: `_load_benchmark_file` implementation / `_load_benchmark_file` 实现
```python
def _load_benchmark_file(file_path: str) -> Dict[str, Any]:
    """Loads a benchmark JSON file."""
    with open(file_path, "r", encoding="utf-8") as f:
        return json.load(f)
```
**EN:** This block defines function `_load_benchmark_file`. Loads a benchmark JSON file. Key calls include `open`, and `json.load`. The implementation uses context-managed resources. Parameters such as `file_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_load_benchmark_file`。 它用于加载benchmark file。 关键调用包括 `open` 和 `json.load`。 实现中使用上下文管理资源。 本段逻辑主要由 `file_path` 等参数驱动。

### Lines 112-118: `_get_status_emoji_from_diff_percent` implementation / `_get_status_emoji_from_diff_percent` 实现
```python
def _get_status_emoji_from_diff_percent(diff_pct):
    if diff_pct < -2.0:
        return "✅"
    elif diff_pct > 2.0:
        return "❌"
    else:
        return "⚪️"
```
**EN:** This block defines function `_get_status_emoji_from_diff_percent`. It retrieves status emoji from diff percent. The implementation branches on conditions. Parameters such as `diff_pct` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_get_status_emoji_from_diff_percent`。 它用于获取status emoji from diff percent。 实现中包含条件分支。 本段逻辑主要由 `diff_pct` 等参数驱动。

### Lines 121-165: `_print_single_comparison_report` implementation / `_print_single_comparison_report` 实现
```python
def _print_single_comparison_report(
    others_data, base_e2e, combined_order, base_durations, others_processed, base_counts
):
    new_data = others_data[0]
    new_e2e = new_data.get("total_duration_ms", 0)
    diff_ms, diff_pct = calculate_diff(base_e2e, new_e2e)
    status = _get_status_emoji_from_diff_percent(diff_pct)

    print("#### 1. High-level Summary")
    print("| Metric | Baseline | New | Diff | Status |")
    print("| :--- | :--- | :--- | :--- | :--- |")
    print(
        f"| **E2E Latency** | {base_e2e:.2f} ms | {new_e2e:.2f} ms | **{diff_ms:+.2f} ms ({diff_pct:+.1f}%)** | {status} |"
    )
    print(
        f"| **Throughput** | {1000 / base_e2e if base_e2e else 0:.2f} req/s | {1000 / new_e2e if new_e2e else 0:.2f} req/s | - | - |"
    )
    print("\n")

    print("#### 2. Stage Breakdown")
    print("| Stage Name | Baseline (ms) | New (ms) | Diff (ms) | Diff (%) | Status |")
    print("| :--- | :--- | :--- | :--- | :--- | :--- |")

    new_durations, _, new_counts = others_processed[0]

    for stage in combined_order:
        b_val = base_durations.get(stage, 0.0)
        n_val = new_durations.get(stage, 0.0)
        b_count = base_counts.get(stage, 1)
        n_count = new_counts.get(stage, 1)

        s_diff, s_pct = calculate_diff(b_val, n_val)

        count_str = ""
        if stage == "Denoising Loop":
            count_str = (
                f" ({n_count} steps)"
                if n_count == b_count
                else f" ({b_count}->{n_count} steps)"
            )

        status_emoji = get_perf_status_emoji(b_val, n_val)
        print(
            f"| {stage}{count_str} | {b_val:.2f} | {n_val:.2f} | {s_diff:+.2f} | {s_pct:+.1f}% | {status_emoji} |"
        )
```
**EN:** This block defines function `_print_single_comparison_report`. It handles print single comparison report logic. Key calls include `new_data.get`, `calculate_diff`, `_get_status_emoji_from_diff_percent`, `print`, and `base_durations.get`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `others_data`, `base_e2e`, `combined_order`, `base_durations`, and `others_processed` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_print_single_comparison_report`。 它用于处理 print single comparison report 相关逻辑。 关键调用包括 `new_data.get`、`calculate_diff`、`_get_status_emoji_from_diff_percent`、`print` 和 `base_durations.get`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `others_data`、`base_e2e`、`combined_order`、`base_durations` 和 `others_processed` 等参数驱动。

### Lines 168-211: `_print_multi_comparison_report` implementation / `_print_multi_comparison_report` 实现
```python
def _print_multi_comparison_report(
    base_e2e,
    others_data,
    other_labels,
    combined_order,
    base_durations,
    others_processed,
):
    print("#### 1. High-level Summary")
    header = "| Metric | Baseline | " + " | ".join(other_labels) + " |"
    sep = "| :--- | :--- | " + " | ".join([":---"] * len(other_labels)) + " |"
    print(header)
    print(sep)

    # E2E Row
    row_e2e = f"| **E2E Latency** | {base_e2e:.2f} ms |"
    for i, d in enumerate(others_data):
        val = d.get("total_duration_ms", 0)
        diff_ms, diff_pct = calculate_diff(base_e2e, val)

        status = _get_status_emoji_from_diff_percent(diff_pct)

        row_e2e += f" {val:.2f} ms ({diff_pct:+.1f}%) {status} |"
    print(row_e2e)
    print("\n")

    print("#### 2. Stage Breakdown")
    # Header: Stage | Baseline | Label1 | Label2 ...
    header = "| Stage Name | Baseline | " + " | ".join(other_labels) + " |"
    sep = "| :--- | :--- | " + " | ".join([":---"] * len(other_labels)) + " |"
    print(header)
    print(sep)

    for stage in combined_order:
        b_val = base_durations.get(stage, 0.0)
        row_str = f"| {stage} | {b_val:.2f} |"

        for i, (n_durations, _, n_counts) in enumerate(others_processed):
            n_val = n_durations.get(stage, 0.0)
            _, s_pct = calculate_diff(b_val, n_val)
            status_emoji = get_perf_status_emoji(b_val, n_val)

            row_str += f" {n_val:.2f} ({s_pct:+.1f}%) {status_emoji} |"
        print(row_str)
```
**EN:** This block defines function `_print_multi_comparison_report`. It handles print multi comparison report logic. Key calls include `print`, `enumerate`, `d.get`, `calculate_diff`, and `_get_status_emoji_from_diff_percent`. The implementation iterates over collections or steps. Parameters such as `base_e2e`, `others_data`, `other_labels`, `combined_order`, and `base_durations` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_print_multi_comparison_report`。 它用于处理 print multi comparison report 相关逻辑。 关键调用包括 `print`、`enumerate`、`d.get`、`calculate_diff` 和 `_get_status_emoji_from_diff_percent`。 实现中会遍历集合或步骤。 本段逻辑主要由 `base_e2e`、`others_data`、`other_labels`、`combined_order` 和 `base_durations` 等参数驱动。

### Lines 214-287: `compare_benchmarks` implementation / `compare_benchmarks` 实现
```python
def compare_benchmarks(file_paths: List[str], output_format: str = "markdown"):
    """
    Compares benchmark JSON files and prints a report.
    First file is baseline, others will be compared against it.
    """
    if len(file_paths) < 2:
        print("Error: Need at least 2 files to compare.")
        return

    try:
        data_list = [_load_benchmark_file(f) for f in file_paths]
    except Exception as e:
        print(f"Error loading benchmark files: {e}")
        return

    base_data = data_list[0]
    others_data = data_list[1:]

    # Use filenames as labels if multiple comparisons, else just "New"
    other_labels = [os.path.basename(p) for p in file_paths[1:]]

    base_e2e = base_data.get("total_duration_ms", 0)

    base_durations, base_order, base_counts = consolidate_steps(
        base_data.get("steps", [])
    )

    others_processed = []
    for d in others_data:
        dur, order, counts = consolidate_steps(d.get("steps", []))
        others_processed.append((dur, order, counts))

    combined_order = []
    # Collect all unique stages maintaining order from newest to baseline
    for _, order, _ in reversed(others_processed):
        for name in order:
            if name not in combined_order:
                combined_order.append(name)
    for name in base_order:
        if name not in combined_order:
            combined_order.append(name)

    if output_format == "markdown":
        print("### Performance Comparison Report\n")

        if len(others_data) == 1:
            _print_single_comparison_report(
                others_data,
                base_e2e,
                combined_order,
                base_durations,
                others_processed,
                base_counts,
            )
        else:
            _print_multi_comparison_report(
                base_e2e,
                others_data,
                other_labels,
                combined_order,
                base_durations,
                others_processed,
            )

        print("\n")
        # Metadata
        print("<details>")
        print("<summary>Metadata</summary>\n")
        print(f"- Baseline Commit: `{base_data.get('commit_hash', 'N/A')}`")
        for i, d in enumerate(others_data):
            label = "New" if len(others_data) == 1 else other_labels[i]
            print(f"- {label} Commit: `{d.get('commit_hash', 'N/A')}`")
        print(f"- Timestamp: {datetime.now().isoformat()}")
        print("</details>")
```
**EN:** This block defines function `compare_benchmarks`. Compares benchmark JSON files and prints a report. First file is baseline, others will be compared against it. Key calls include `base_data.get`, `consolidate_steps`, `reversed`, `len`, and `print`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `file_paths`, and `output_format` drive the behavior in this section.
**CN:** 该代码块定义了函数 `compare_benchmarks`。 它用于处理 compare benchmarks 相关逻辑。 关键调用包括 `base_data.get`、`consolidate_steps`、`reversed`、`len` 和 `print`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `file_paths` 和 `output_format` 等参数驱动。

### Lines 290-301: supporting statements / 辅助语句
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Compare sglang-diffusion performance JSON files."
    )
    parser.add_argument(
        "files",
        nargs="+",
        help="List of JSON files. First is baseline, others are compared against it.",
    )
    args = parser.parse_args()

    compare_benchmarks(args.files)
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, and `compare_benchmarks`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `argparse.ArgumentParser`、`parser.add_argument`、`parser.parse_args` 和 `compare_benchmarks` 协同工作。

## Key Concepts / 关键概念
- `calculate_diff`: Returns (diff, diff_percent). / 顶层函数，用于处理 calculate diff 相关逻辑。
- `calculate_upper_bound`: Calculates the upper bound for performance regression check. / 顶层函数，用于处理 calculate upper bound 相关逻辑。
- `calculate_lower_bound`: Calculates the lower bound for performance improvement check. / 顶层函数，用于处理 calculate lower bound 相关逻辑。
- `get_perf_status_emoji`: Determines the status emoji based on performance difference. / 顶层函数，用于获取perf status emoji。
- `consolidate_steps`: Aggregates specific repeating steps (like denoising_step_*) into groups. / 顶层函数，用于处理 consolidate steps 相关逻辑。
- `_load_benchmark_file`: Loads a benchmark JSON file. / 顶层函数，用于加载benchmark file。
- `_get_status_emoji_from_diff_percent`: Top-level function that retrieves status emoji from diff percent. / 顶层函数，用于获取status emoji from diff percent。
- `_print_single_comparison_report`: Top-level function that handles print single comparison report logic. / 顶层函数，用于处理 print single comparison report 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `os`, `re`, `datetime`, `typing`

- **Total lines / 总行数**: 301
