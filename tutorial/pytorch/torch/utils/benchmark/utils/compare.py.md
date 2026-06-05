# compare.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/benchmark/utils/compare.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements benchmarking helpers that measure performance, compare runs, and format timing-oriented reports. This specific file centers on `compare.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. The implementation is performance-facing and focuses on measurement fidelity or result reporting. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现基准测试辅助逻辑，用于测量性能、比较运行结果并格式化计时报告。 该文件具体围绕 `compare.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 该实现面向性能分析，重点在于测量可信度或结果报告。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
```python
# mypy: allow-untyped-defs
"""Display class to aggregate and print the results of many measurements."""
import collections
import enum
import itertools as it

from torch.utils.benchmark.utils import common
from torch import tensor as _tensor
import operator

__all__ = ["Colorize", "Compare"]

BEST = "\033[92m"
GOOD = "\033[34m"
BAD = "\033[2m\033[91m"
VERY_BAD = "\033[31m"
BOLD = "\033[1m"
TERMINATE = "\033[0m"
```
- **EN**: The opening docstring frames the module contract, telling readers what subsystem responsibilities the rest of the file implements. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. Named constants such as `BEST`, `GOOD`, `BAD`, `VERY_BAD` centralize shared configuration or sentinel values. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 开头的模块文档字符串给出了整体契约，说明后续代码要实现的子系统职责。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 `BEST, GOOD, BAD, VERY_BAD` 等具名常量把共享配置或哨兵值集中定义在一起。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 21-44 / 第 21-44 行
```python
class Colorize(enum.Enum):
    NONE = "none"
    COLUMNWISE = "columnwise"
    ROWWISE = "rowwise"


# Classes to separate internal bookkeeping from what is rendered.
class _Column:
    def __init__(
        self,
        grouped_results: list[tuple[common.Measurement | None, ...]],
        time_scale: float,
        time_unit: str,
        trim_significant_figures: bool,
        highlight_warnings: bool,
    ) -> None:
        self._grouped_results = grouped_results
        self._flat_results = [*it.chain.from_iterable(grouped_results)]
        self._time_scale = time_scale
        self._time_unit = time_unit
        self._trim_significant_figures = trim_significant_figures
        self._highlight_warnings = (
            highlight_warnings
            and any(r.has_warnings for r in self._flat_results if r)
```
- **EN**: It introduces or extends class-level abstractions such as `Colorize`, `_Column`, which organize state and behavior for this subsystem. Named constants such as `NONE`, `COLUMNWISE`, `ROWWISE` centralize shared configuration or sentinel values. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Colorize`, `_Column` 等类级抽象，用于组织该子系统的状态与行为。 `NONE, COLUMNWISE, ROWWISE` 等具名常量把共享配置或哨兵值集中定义在一起。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 45-67 / 第 45-67 行
```python
        )
        leading_digits = [
            int(_tensor(r.median / self._time_scale).log10().ceil()) if r else None
            for r in self._flat_results
        ]
        unit_digits = max(d for d in leading_digits if d is not None)
        decimal_digits = min(
            max(m.significant_figures - digits, 0)
            for digits, m in zip(leading_digits, self._flat_results, strict=True)
            if (m is not None) and (digits is not None)
        ) if self._trim_significant_figures else 1
        length = unit_digits + decimal_digits + (1 if decimal_digits else 0)
        self._template = f"{{:>{length}.{decimal_digits}f}}{{:>{7 if self._highlight_warnings else 0}}}"

    def get_results_for(self, group):
        return self._grouped_results[group]

    def num_to_str(self, value: float | None, estimated_sigfigs: int, spread: float | None):
        if value is None:
            return " " * len(self.num_to_str(1, estimated_sigfigs, None))

        if self._trim_significant_figures:
            value = common.trim_sigfig(value, estimated_sigfigs)
```
- **EN**: It introduces or extends class-level abstractions such as `_Column`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `_Column` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 69-91 / 第 69-91 行
```python
        return self._template.format(
            value,
            f" (! {spread * 100:.0f}%)" if self._highlight_warnings and spread is not None else "")


def optional_min(seq):
    l = list(seq)
    return None if len(l) == 0 else min(l)


class _Row:
    def __init__(self, results, row_group, render_env, env_str_len,
                 row_name_str_len, time_scale, colorize, num_threads=None) -> None:
        super().__init__()
        self._results = results
        self._row_group = row_group
        self._render_env = render_env
        self._env_str_len = env_str_len
        self._row_name_str_len = row_name_str_len
        self._time_scale = time_scale
        self._colorize = colorize
        self._columns: tuple[_Column, ...] = ()
        self._num_threads = num_threads
```
- **EN**: It introduces or extends class-level abstractions such as `_Column`, `_Row`, which organize state and behavior for this subsystem. Key callable entry points in this range include `optional_min`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `_Column`, `_Row` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `optional_min`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 93-110 / 第 93-110 行
```python
    def register_columns(self, columns: tuple[_Column, ...]) -> None:
        self._columns = columns

    def as_column_strings(self):
        concrete_results = [r for r in self._results if r is not None]
        env = f"({concrete_results[0].env})" if self._render_env else ""
        env = env.ljust(self._env_str_len + 4)
        output = ["  " + env + concrete_results[0].as_row_name]
        for m, col in zip(self._results, self._columns or (), strict=False):
            if m is None:
                output.append(col.num_to_str(None, 1, None))
            else:
                output.append(col.num_to_str(
                    m.median / self._time_scale,
                    m.significant_figures,
                    m.iqr / m.median if m.has_warnings else None
                ))
        return output
```
- **EN**: It introduces or extends class-level abstractions such as `_Row`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `_Row` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 112-129 / 第 112-129 行
```python
    @staticmethod
    def color_segment(segment, value, best_value):
        if value <= best_value * 1.01 or value <= best_value + 100e-9:
            return BEST + BOLD + segment + TERMINATE * 2
        if value <= best_value * 1.1:
            return GOOD + BOLD + segment + TERMINATE * 2
        if value >= best_value * 5:
            return VERY_BAD + BOLD + segment + TERMINATE * 2
        if value >= best_value * 2:
            return BAD + segment + TERMINATE * 2

        return segment

    def row_separator(self, overall_width):
        return (
            [f"{self._num_threads} threads: ".ljust(overall_width, "-")]
            if self._num_threads is not None else []
        )
```
- **EN**: It introduces or extends class-level abstractions such as `_Row`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `_Row` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 131-148 / 第 131-148 行
```python
    def finalize_column_strings(self, column_strings, col_widths):
        best_values = [-1 for _ in column_strings]
        if self._colorize == Colorize.ROWWISE:
            row_min = min(r.median for r in self._results if r is not None)
            best_values = [row_min for _ in column_strings]
        elif self._colorize == Colorize.COLUMNWISE:
            best_values = [
                optional_min(r.median for r in column.get_results_for(self._row_group) if r is not None)
                for column in (self._columns or ())
            ]

        row_contents = [column_strings[0].ljust(col_widths[0])]
        for col_str, width, result, best_value in zip(column_strings[1:], col_widths[1:], self._results, best_values, strict=False):
            col_str = col_str.center(width)
            if self._colorize != Colorize.NONE and result is not None and best_value is not None:
                col_str = self.color_segment(col_str, result.median, best_value)
            row_contents.append(col_str)
        return row_contents
```
- **EN**: It introduces or extends class-level abstractions such as `_Row`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `_Row` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 151-169 / 第 151-169 行
```python
class Table:
    def __init__(
            self,
            results: list[common.Measurement],
            colorize: Colorize,
            trim_significant_figures: bool,
            highlight_warnings: bool
    ) -> None:
        if len({r.label for r in results}) != 1:
            raise AssertionError("All results must share the same label")

        self.results = results
        self._colorize = colorize
        self._trim_significant_figures = trim_significant_figures
        self._highlight_warnings = highlight_warnings
        self.label = results[0].label
        self.time_unit, self.time_scale = common.select_unit(
            min(r.median for r in results)
        )
```
- **EN**: It introduces or extends class-level abstractions such as `Table`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Table` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 171-194 / 第 171-194 行
```python
        self.row_keys = common.ordered_unique([self.row_fn(i) for i in results])
        self.row_keys.sort(key=operator.itemgetter(slice(2)))  # preserve stmt order
        self.column_keys = common.ordered_unique([self.col_fn(i) for i in results])
        self.rows, self.columns = self.populate_rows_and_columns()

    @staticmethod
    def row_fn(m: common.Measurement) -> tuple[int, str | None, str]:
        return m.num_threads, m.env, m.as_row_name

    @staticmethod
    def col_fn(m: common.Measurement) -> str | None:
        return m.description

    def populate_rows_and_columns(self) -> tuple[tuple[_Row, ...], tuple[_Column, ...]]:
        rows: list[_Row] = []
        columns: list[_Column] = []
        ordered_results: list[list[common.Measurement | None]] = [
            [None for _ in self.column_keys]
            for _ in self.row_keys
        ]
        row_position = {key: i for i, key in enumerate(self.row_keys)}
        col_position = {key: i for i, key in enumerate(self.column_keys)}
        for r in self.results:
            i = row_position[self.row_fn(r)]
```
- **EN**: It introduces or extends class-level abstractions such as `Table`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Table` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 195-218 / 第 195-218 行
```python
            j = col_position[self.col_fn(r)]
            ordered_results[i][j] = r

        unique_envs = {r.env for r in self.results}
        render_env = len(unique_envs) > 1
        env_str_len = max(len(i) for i in unique_envs) if render_env else 0

        row_name_str_len = max(len(r.as_row_name) for r in self.results)

        prior_num_threads = -1
        prior_env = ""
        row_group = -1
        rows_by_group: list[list[list[common.Measurement | None]]] = []
        for (num_threads, env, _), row in zip(self.row_keys, ordered_results, strict=True):
            thread_transition = (num_threads != prior_num_threads)
            if thread_transition:
                prior_num_threads = num_threads
                prior_env = ""
                row_group += 1
                rows_by_group.append([])
            rows.append(
                _Row(
                    results=row,
                    row_group=row_group,
```
- **EN**: It introduces or extends class-level abstractions such as `Table`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Table` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 219-238 / 第 219-238 行
```python
                    render_env=(render_env and env != prior_env),
                    env_str_len=env_str_len,
                    row_name_str_len=row_name_str_len,
                    time_scale=self.time_scale,
                    colorize=self._colorize,
                    num_threads=num_threads if thread_transition else None,
                )
            )
            rows_by_group[-1].append(row)
            prior_env = env

        for i in range(len(self.column_keys)):
            grouped_results = [tuple(row[i] for row in g) for g in rows_by_group]
            column = _Column(
                grouped_results=grouped_results,
                time_scale=self.time_scale,
                time_unit=self.time_unit,
                trim_significant_figures=self._trim_significant_figures,
                highlight_warnings=self._highlight_warnings,)
            columns.append(column)
```
- **EN**: It introduces or extends class-level abstractions such as `Table`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Table` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 240-257 / 第 240-257 行
```python
        rows_tuple, columns_tuple = tuple(rows), tuple(columns)
        for ri in rows_tuple:
            ri.register_columns(columns_tuple)
        return rows_tuple, columns_tuple

    def render(self) -> str:
        string_rows = [[""] + self.column_keys]
        string_rows.extend(r.as_column_strings() for r in self.rows)
        num_cols = max(len(i) for i in string_rows)
        for sr in string_rows:
            sr.extend(["" for _ in range(num_cols - len(sr))])

        col_widths = [max(len(j) for j in i) for i in zip(*string_rows, strict=True)]
        finalized_columns = ["  |  ".join(i.center(w) for i, w in zip(string_rows[0], col_widths, strict=True))]
        overall_width = len(finalized_columns[0])
        for string_row, row in zip(string_rows[1:], self.rows, strict=True):
            finalized_columns.extend(row.row_separator(overall_width))
            finalized_columns.append("  |  ".join(row.finalize_column_strings(string_row, col_widths)))
```
- **EN**: It introduces or extends class-level abstractions such as `Table`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Table` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 259-280 / 第 259-280 行
```python
        newline = "\n"
        has_warnings = self._highlight_warnings and any(ri.has_warnings for ri in self.results)
        return f"""
[{(' ' + (self.label or '') + ' ').center(overall_width - 2, '-')}]
{newline.join(finalized_columns)}

Times are in {common.unit_to_english(self.time_unit)}s ({self.time_unit}).
{'(! XX%) Measurement has high variance, where XX is the IQR / median * 100.' + newline if has_warnings else ""}"""[1:]


class Compare:
    """Helper class for displaying the results of many measurements in a
    formatted table.

    The table format is based on the information fields provided in
    :class:`torch.utils.benchmark.Timer` (`description`, `label`, `sub_label`,
    `num_threads`, etc).

    The table can be directly printed using :meth:`print` or casted as a `str`.

    For a full tutorial on how to use this class, see:
    https://pytorch.org/tutorials/recipes/recipes/benchmark.html
```
- **EN**: It introduces or extends class-level abstractions such as `Table`, `Compare`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Table`, `Compare` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 282-296 / 第 282-296 行
```python
    Args:
        results: List of Measurement to display.
    """
    def __init__(self, results: list[common.Measurement]) -> None:
        self._results: list[common.Measurement] = []
        self.extend_results(results)
        self._trim_significant_figures = False
        self._colorize = Colorize.NONE
        self._highlight_warnings = False

    def __str__(self) -> str:
        return "\n".join(self._render())

    def extend_results(self, results) -> None:
        """Append results to already stored ones.
```
- **EN**: It introduces or extends class-level abstractions such as `Compare`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Compare` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 298-320 / 第 298-320 行
```python
        All added results must be instances of ``Measurement``.
        """
        for r in results:
            if not isinstance(r, common.Measurement):
                raise ValueError(
                    "Expected an instance of `Measurement`, " f"got {type(r)} instead."
                )
        self._results.extend(results)

    def trim_significant_figures(self) -> None:
        """Enables trimming of significant figures when building the formatted table."""
        self._trim_significant_figures = True

    def colorize(self, rowwise=False) -> None:
        """Colorize formatted table.

        Colorize columnwise by default.
        """
        self._colorize = Colorize.ROWWISE if rowwise else Colorize.COLUMNWISE

    def highlight_warnings(self) -> None:
        """Enables warning highlighting when building formatted table."""
        self._highlight_warnings = True
```
- **EN**: It introduces or extends class-level abstractions such as `Compare`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Compare` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 322-345 / 第 322-345 行
```python
    def print(self) -> None:
        """Print formatted table"""
        print(str(self))

    def _render(self):
        results = common.Measurement.merge(self._results)
        grouped_results = self._group_by_label(results)
        output = [self._layout(group) for group in grouped_results.values()]
        return output

    def _group_by_label(self, results: list[common.Measurement]):
        grouped_results: collections.defaultdict[str, list[common.Measurement]] = collections.defaultdict(list)
        for r in results:
            grouped_results[r.label].append(r)
        return grouped_results

    def _layout(self, results: list[common.Measurement]):
        table = Table(
            results,
            self._colorize,
            self._trim_significant_figures,
            self._highlight_warnings
        )
        return table.render()
```
- **EN**: It introduces or extends class-level abstractions such as `Compare`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `Compare` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements benchmarking helpers that measure performance, compare runs, and format timing-oriented reports.
  - CN: 实现基准测试辅助逻辑，用于测量性能、比较运行结果并格式化计时报告。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Numeric comparison**
  - EN: Helps compare reference and transformed executions to diagnose quantization effects.
  - CN: 帮助比较参考执行与转换后执行，从而诊断量化带来的影响。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch.utils.benchmark.utils:common`, `torch:tensor`
- **Python standard library / Python 标准库**: `collections`, `enum`, `itertools`, `operator`
- **Explicit exports / 显式导出**: `Colorize`, `Compare`
- **Primary symbols / 核心符号**: `Colorize`, `_Column`, `_Row`, `Table`, `Compare`, `optional_min`
