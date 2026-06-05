# plot_pareto.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/sweep/plot_pareto.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements plot pareto support for the `sweep` portion of vLLM. / 为 vLLM 的 `sweep` 子目录实现与 plotpareto 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-15)
```python
import argparse

import math

from concurrent.futures import ProcessPoolExecutor

from dataclasses import dataclass

from functools import partial

from pathlib import Path

from typing import TYPE_CHECKING, ClassVar

from vllm.utils.collection_utils import full_groupby

from vllm.utils.import_utils import PlaceholderModule

from .plot import DummyExecutor, _json_load_bytes

from .utils import sanitize_filename
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Function `_first_present` (lines 21-26)
```python
def _first_present(run_data: dict[str, object], keys: list[str]):
    for key in keys:
        for candidate in {key, key.replace("_", "-"), key.replace("-", "_")}:
            if candidate in run_data:
                return run_data[candidate]
    return None
```
**EN:** Function `_first_present` provides a reusable helper around the module's main workflow. Key calls such as `key.replace` show the concrete execution path.
**CN:** Function `_first_present` 为模块主流程提供可复用的辅助逻辑。 像 `key.replace` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_numeric` (lines 29-50)
```python
def _get_numeric(
    run_data: dict[str, object],
    keys: list[str],
    *,
    allow_zero: bool = True,
) -> float | None:
    value = _first_present(run_data, keys)
    if value is None:
        return None

    try:
        numeric = float(value)
    except (TypeError, ValueError) as exc:
        raise ValueError(
            f"Expected numeric value for one of {keys}, "
            f"but found {value!r} in {run_data=}"
        ) from exc

    if not allow_zero and numeric == 0:
        return None

    return numeric
```
**EN:** Function `_get_numeric` provides a reusable helper around the module's main workflow. Key calls such as `_first_present`, `float`, `ValueError` show the concrete execution path.
**CN:** Function `_get_numeric` 为模块主流程提供可复用的辅助逻辑。 像 `_first_present`, `float`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_infer_user_count` (lines 53-64)
```python
def _infer_user_count(
    run_data: dict[str, object],
    user_count_var: str | None,
) -> float | None:
    candidates = [user_count_var] if user_count_var else []
    candidates.extend(["request_rate"])
    user_count = _get_numeric(run_data, candidates, allow_zero=False)
    if user_count is not None:
        return user_count

    # Fallback to the observed peak if configured value is missing.
    return _get_numeric(run_data, ["max_concurrent_requests"], allow_zero=False)
```
**EN:** Function `_infer_user_count` provides a reusable helper around the module's main workflow. Key calls such as `candidates.extend`, `_get_numeric` show the concrete execution path.
**CN:** Function `_infer_user_count` 为模块主流程提供可复用的辅助逻辑。 像 `candidates.extend`, `_get_numeric` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_infer_gpu_count` (lines 67-87)
```python
def _infer_gpu_count(
    run_data: dict[str, object],
    gpu_count_var: str | None,
) -> float:
    direct_candidates = [gpu_count_var] if gpu_count_var else []
    direct_gpu_count = _get_numeric(run_data, direct_candidates, allow_zero=False)
    if direct_gpu_count:
        return direct_gpu_count

    tp_size = _get_numeric(run_data, ["tensor_parallel_size", "tp"])
    pp_size = _get_numeric(run_data, ["pipeline_parallel_size", "pp"])
    dp_size = _get_numeric(run_data, ["data_parallel_size", "dp"])
    world_size = 1.0
    if tp_size:
        world_size *= tp_size
    if pp_size:
        world_size *= pp_size
    if dp_size:
        world_size *= dp_size

    return world_size
```
**EN:** Function `_infer_gpu_count` provides a reusable helper around the module's main workflow. Key calls such as `_get_numeric` show the concrete execution path.
**CN:** Function `_infer_gpu_count` 为模块主流程提供可复用的辅助逻辑。 像 `_get_numeric` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_throughput` (lines 90-101)
```python
def _get_throughput(
    run_data: dict[str, object],
    throughput_var: str,
) -> float:
    throughput = _get_numeric(run_data, [throughput_var])
    if throughput is None:
        raise ValueError(
            f"Cannot find throughput metric {throughput_var!r} in run data. "
            f"Available keys: {sorted(run_data)}"
        )

    return throughput
```
**EN:** Function `_get_throughput` coordinates benchmarking or serving-oriented control flow. Key calls such as `_get_numeric`, `ValueError`, `sorted` show the concrete execution path.
**CN:** Function `_get_throughput` 负责协调基准测试或服务侧控制流程。 像 `_get_numeric`, `ValueError`, `sorted` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_prepare_records` (lines 104-134)
```python
def _prepare_records(
    all_data: list[dict[str, object]],
    *,
    user_count_var: str | None,
    gpu_count_var: str | None,
) -> tuple[list[dict[str, object]], int]:
    prepared = []
    skipped_missing_users = 0

    for record in all_data:
        throughput = _get_throughput(record, "output_throughput")
        user_count = _infer_user_count(record, user_count_var)
        if user_count is None:
            skipped_missing_users += 1
            continue

        gpu_count = _infer_gpu_count(record, gpu_count_var)
        tokens_per_user = throughput / user_count
        tokens_per_gpu = throughput / gpu_count

        prepared.append(
            {
                **record,
                "tokens_per_user": tokens_per_user,
                "tokens_per_gpu": tokens_per_gpu,
    # ... omitted for brevity ...

    return prepared, skipped_missing_users
```
**EN:** Function `_prepare_records` provides a reusable helper around the module's main workflow. Key calls such as `_get_throughput`, `_infer_user_count`, `_infer_gpu_count`, `prepared.append` show the concrete execution path.
**CN:** Function `_prepare_records` 为模块主流程提供可复用的辅助逻辑。 像 `_get_throughput`, `_infer_user_count`, `_infer_gpu_count`, `prepared.append` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_pareto_frontier` (lines 137-154)
```python
def _pareto_frontier(
    df: "pd.DataFrame",
    x_col: str,
    y_col: str,
    *,
    epsilon: float = 1e-9,
) -> "pd.DataFrame":
    sorted_df = df.sort_values([x_col, y_col], ascending=[False, False])
    frontier_indices = []
    best_y = -math.inf

    for idx, row in sorted_df.iterrows():
        y_val = row[y_col]
        if y_val >= best_y - epsilon:
            frontier_indices.append(idx)
            best_y = max(best_y, y_val)

    return df.loc[frontier_indices]
```
**EN:** Function `_pareto_frontier` provides a reusable helper around the module's main workflow. Key calls such as `df.sort_values`, `sorted_df.iterrows`, `frontier_indices.append`, `max` show the concrete execution path.
**CN:** Function `_pareto_frontier` 为模块主流程提供可复用的辅助逻辑。 像 `df.sort_values`, `sorted_df.iterrows`, `frontier_indices.append`, `max` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_fig_path` (lines 157-165)
```python
def _get_fig_path(
    fig_dir: Path,
    fig_group: tuple[tuple[str, str], ...],
) -> Path:
    parts = ["PARETO"]
    if fig_group:
        parts.extend(f"{k}={v}" for k, v in fig_group)
    filename = sanitize_filename("-".join(parts) + ".png")
    return fig_dir / filename
```
**EN:** Function `_get_fig_path` provides a reusable helper around the module's main workflow. Key calls such as `parts.extend`, `sanitize_filename`, `'-'.join` show the concrete execution path.
**CN:** Function `_get_fig_path` 为模块主流程提供可复用的辅助逻辑。 像 `parts.extend`, `sanitize_filename`, `'-'.join` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_plot_fig` (lines 168-254)
```python
def _plot_fig(
    fig_dir: Path,
    fig_group_data: tuple[tuple[tuple[str, str], ...], list[dict[str, object]]],
    label_by: list[str],
    *,
    dry_run: bool,
):
    fig_group, fig_data = fig_group_data
    fig_path = _get_fig_path(fig_dir, fig_group)

    print("[BEGIN FIGURE]")
    print(f"Group: {dict(fig_group)}")
    print(f"Output file: {fig_path}")

    if dry_run:
        print("[END FIGURE]")
        return

    # Lazy-import matplotlib/pandas/seaborn
    try:
        import matplotlib.pyplot as plt
    except ImportError:
        plt = PlaceholderModule("matplotlib").placeholder_attr("pyplot")
    try:
        import pandas as pd
    # ... omitted for brevity ...
    )
    print("[END FIGURE]")
```
**EN:** Function `_plot_fig` provides a reusable helper around the module's main workflow. Key calls such as `_get_fig_path`, `print`, `dict`, `PlaceholderModule('matplotlib').placeholder_attr`, `PlaceholderModule` show the concrete execution path.
**CN:** Function `_plot_fig` 为模块主流程提供可复用的辅助逻辑。 像 `_get_fig_path`, `print`, `dict`, `PlaceholderModule('matplotlib').placeholder_attr`, `PlaceholderModule` 这样的关键调用展示了该代码块的具体执行路径。

### Function `plot_pareto` (lines 257-311)
```python
def plot_pareto(
    output_dir: Path,
    user_count_var: str | None,
    gpu_count_var: str | None,
    label_by: list[str],
    *,
    dry_run: bool,
):
    fig_dir = output_dir / "pareto"
    raw_data = [
        run_data
        for path in output_dir.rglob("**/summary.json")
        for run_data in _json_load_bytes(path)
    ]

    if not raw_data:
        raise ValueError(f"Did not find any parameter sweep results under {output_dir}")

    fig_dir.mkdir(parents=True, exist_ok=True)

    prepared_data, skipped_missing_users = _prepare_records(
        raw_data,
        user_count_var=user_count_var,
        gpu_count_var=gpu_count_var,
    )
    # ... omitted for brevity ...
            )
        )
```
**EN:** Function `plot_pareto` provides a reusable helper around the module's main workflow. Key calls such as `output_dir.rglob`, `_json_load_bytes`, `ValueError`, `fig_dir.mkdir`, `_prepare_records` show the concrete execution path.
**CN:** Function `plot_pareto` 为模块主流程提供可复用的辅助逻辑。 像 `output_dir.rglob`, `_json_load_bytes`, `ValueError`, `fig_dir.mkdir`, `_prepare_records` 这样的关键调用展示了该代码块的具体执行路径。

### Class `SweepPlotParetoArgs` (lines 315-379)
```python
class SweepPlotParetoArgs:
    output_dir: Path
    user_count_var: str | None
    gpu_count_var: str | None
    label_by: list[str]
    dry_run: bool

    parser_name: ClassVar[str] = "plot_pareto"
    parser_help: ClassVar[str] = (
        "Plot Pareto frontier between tokens/s/user and tokens/s/GPU "
        "from parameter sweep results."
    )

    @classmethod
    def from_cli_args(cls, args: argparse.Namespace):
        output_dir = Path(args.EXPERIMENT_DIR)
        if not output_dir.exists():
            raise ValueError(f"No parameter sweep results under {output_dir}")

        label_by = [] if not args.label_by else args.label_by.split(",")

        return cls(
            output_dir=output_dir,
    # ... omitted for brevity ...

        return parser
```
**EN:** Class `SweepPlotParetoArgs` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `from_cli_args`, `add_cli_args`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `SweepPlotParetoArgs` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `from_cli_args`, `add_cli_args`，它们共同定义初始化、校验、变换或访问模式。

### Method `SweepPlotParetoArgs.from_cli_args` (lines 329-342)
```python
    def from_cli_args(cls, args: argparse.Namespace):
        output_dir = Path(args.EXPERIMENT_DIR)
        if not output_dir.exists():
            raise ValueError(f"No parameter sweep results under {output_dir}")

        label_by = [] if not args.label_by else args.label_by.split(",")

        return cls(
            output_dir=output_dir,
            user_count_var=args.user_count_var,
            gpu_count_var=args.gpu_count_var,
            label_by=label_by,
            dry_run=args.dry_run,
        )
```
**EN:** Method `SweepPlotParetoArgs.from_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `Path`, `output_dir.exists`, `ValueError`, `args.label_by.split`, `cls` show the concrete execution path.
**CN:** Method `SweepPlotParetoArgs.from_cli_args` 负责解析配置、参数或结构化元数据。 像 `Path`, `output_dir.exists`, `ValueError`, `args.label_by.split`, `cls` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepPlotParetoArgs.add_cli_args` (lines 345-379)
```python
    def add_cli_args(cls, parser: argparse.ArgumentParser):
        parser.add_argument(
            "EXPERIMENT_DIR",
            type=str,
            help="The directory containing the sweep results to plot.",
        )
        parser.add_argument(
            "--user-count-var",
            type=str,
            default="max_concurrency",
            help="Result key that stores concurrent user count. "
            "Falls back to max_concurrent_requests if missing.",
        )
        parser.add_argument(
            "--gpu-count-var",
            type=str,
            default=None,
            help="Result key that stores GPU count. "
            "If not provided, falls back to num_gpus/gpu_count "
    # ... omitted for brevity ...

        return parser
```
**EN:** Method `SweepPlotParetoArgs.add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `parser.add_argument` show the concrete execution path.
**CN:** Method `SweepPlotParetoArgs.add_cli_args` 负责解析配置、参数或结构化元数据。 像 `parser.add_argument` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_main` (lines 382-389)
```python
def run_main(args: SweepPlotParetoArgs):
    return plot_pareto(
        output_dir=args.output_dir,
        user_count_var=args.user_count_var,
        gpu_count_var=args.gpu_count_var,
        label_by=args.label_by,
        dry_run=args.dry_run,
    )
```
**EN:** Function `run_main` provides a reusable helper around the module's main workflow. Key calls such as `plot_pareto` show the concrete execution path.
**CN:** Function `run_main` 为模块主流程提供可复用的辅助逻辑。 像 `plot_pareto` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 392-393)
```python
def main(args: argparse.Namespace):
    run_main(SweepPlotParetoArgs.from_cli_args(args))
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `run_main`, `SweepPlotParetoArgs.from_cli_args` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `run_main`, `SweepPlotParetoArgs.from_cli_args` 这样的关键调用展示了该代码块的具体执行路径。

### Entrypoint guard (lines 396-400)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description=SweepPlotParetoArgs.parser_help)
    SweepPlotParetoArgs.add_cli_args(parser)

    main(parser.parse_args())
```
**EN:** This standard `__main__` guard turns the module into an executable script and forwards control to the top-level CLI or main workflow.
**CN:** 标准的 `__main__` 守卫让该模块可以作为脚本执行，并把控制流转交给顶层 CLI 或主流程。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **CLI workflow / 命令行流程**
  - **EN:** The module exposes a command-line entrypoint so it can be run as a standalone tool.
  - **CN:** 该模块暴露了命令行入口，因此可以作为独立工具运行。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import math`, `from concurrent.futures import ProcessPoolExecutor`, `from dataclasses import dataclass`, `from functools import partial`, `from pathlib import Path`, `from typing import TYPE_CHECKING, ClassVar`
- **vLLM internal / vLLM 内部依赖**: `from vllm.utils.collection_utils import full_groupby`, `from vllm.utils.import_utils import PlaceholderModule`, `from .plot import DummyExecutor, _json_load_bytes`, `from .utils import sanitize_filename`
