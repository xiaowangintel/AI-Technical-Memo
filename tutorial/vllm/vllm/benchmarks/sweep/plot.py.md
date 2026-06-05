# plot.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/sweep/plot.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements plot support for the `sweep` portion of vLLM. / 为 vLLM 的 `sweep` 子目录实现与 plot 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-18)
```python
import argparse

import json

from abc import ABC, abstractmethod

from concurrent.futures import ProcessPoolExecutor

from dataclasses import dataclass

from functools import partial

from pathlib import Path

from types import TracebackType

from typing import TYPE_CHECKING, ClassVar

from typing_extensions import Self, override

from vllm.utils.collection_utils import full_groupby

from vllm.utils.import_utils import PlaceholderModule

from .utils import sanitize_filename
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 103-154)
```python
PLOT_FILTERS: dict[str, type[PlotFilterBase]] = {
    "==": PlotEqualTo,
    "!=": PlotNotEqualTo,
    "<=": PlotLessThanOrEqualTo,
    ">=": PlotGreaterThanOrEqualTo,
    "<": PlotLessThan,
    ">": PlotGreaterThan,
}

PLOT_BINNERS: dict[str, type[PlotBinner]] = {
    "%": PlotBinner,
}
```
**EN:** This constant/configuration block defines `PLOT_FILTERS`, `PLOT_BINNERS`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `PLOT_FILTERS`, `PLOT_BINNERS`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Large-file note
```python
# Focused on representative top-level definitions for a large module.
# The full file contains additional helpers following the same patterns.
```
**EN:** Because the file is large, the analysis below focuses on the most important top-level definitions and leaves repetitive helpers summarized at a higher level.
**CN:** 由于文件较大，下面的分析聚焦最重要的顶层定义；其余重复性辅助逻辑将以更高层次方式概括。

### Class `PlotFilterBase` (lines 25-47)
```python
class PlotFilterBase(ABC):
    var: str
    target: str

    @classmethod
    def parse_str(cls, s: str):
        for op_key in PLOT_FILTERS:
            if op_key in s:
                key, value = s.split(op_key)
                return PLOT_FILTERS[op_key](
                    key,
                    value.removeprefix(op_key).strip("'").strip('"'),
                )
        else:
            raise ValueError(
                f"Invalid operator for plot filter '{s}'. "
                f"Valid operators are: {sorted(PLOT_FILTERS)}",
            )

    @abstractmethod
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        """Applies this filter to a DataFrame."""
        raise NotImplementedError
```
**EN:** Class `PlotFilterBase` is a structured building block in this module. It inherits from `ABC` and uses a dataclass-style declaration to store explicit state. Key methods include `parse_str`, `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotFilterBase` 是该模块中的结构化构件，继承自 `ABC`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `parse_str`, `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotFilterBase.parse_str` (lines 30-42)
```python
    def parse_str(cls, s: str):
        for op_key in PLOT_FILTERS:
            if op_key in s:
                key, value = s.split(op_key)
                return PLOT_FILTERS[op_key](
                    key,
                    value.removeprefix(op_key).strip("'").strip('"'),
                )
        else:
            raise ValueError(
                f"Invalid operator for plot filter '{s}'. "
                f"Valid operators are: {sorted(PLOT_FILTERS)}",
            )
```
**EN:** Method `PlotFilterBase.parse_str` parses configuration, arguments, or structured metadata. Key calls such as `s.split`, `value.removeprefix(op_key).strip("'").strip`, `value.removeprefix(op_key).strip`, `value.removeprefix`, `ValueError` show the concrete execution path.
**CN:** Method `PlotFilterBase.parse_str` 负责解析配置、参数或结构化元数据。 像 `s.split`, `value.removeprefix(op_key).strip("'").strip`, `value.removeprefix(op_key).strip`, `value.removeprefix`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PlotFilterBase.apply` (lines 45-47)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        """Applies this filter to a DataFrame."""
        raise NotImplementedError
```
**EN:** Method `PlotFilterBase.apply` provides a reusable helper around the module's main workflow. The docstring highlights: Applies this filter to a DataFrame.
**CN:** Method `PlotFilterBase.apply` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Applies this filter to a DataFrame.

### Class `PlotEqualTo` (lines 51-59)
```python
class PlotEqualTo(PlotFilterBase):
    @override
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        try:
            target = float(self.target)
        except ValueError:
            target = self.target

        return df[df[self.var] == target]
```
**EN:** Class `PlotEqualTo` is a structured building block in this module. It inherits from `PlotFilterBase` and uses a dataclass-style declaration to store explicit state. Key methods include `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotEqualTo` 是该模块中的结构化构件，继承自 `PlotFilterBase`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotEqualTo.apply` (lines 53-59)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        try:
            target = float(self.target)
        except ValueError:
            target = self.target

        return df[df[self.var] == target]
```
**EN:** Method `PlotEqualTo.apply` provides a reusable helper around the module's main workflow. Key calls such as `float` show the concrete execution path.
**CN:** Method `PlotEqualTo.apply` 为模块主流程提供可复用的辅助逻辑。 像 `float` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PlotNotEqualTo` (lines 63-71)
```python
class PlotNotEqualTo(PlotFilterBase):
    @override
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        try:
            target = float(self.target)
        except ValueError:
            target = self.target

        return df[df[self.var] != target]
```
**EN:** Class `PlotNotEqualTo` is a structured building block in this module. It inherits from `PlotFilterBase` and uses a dataclass-style declaration to store explicit state. Key methods include `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotNotEqualTo` 是该模块中的结构化构件，继承自 `PlotFilterBase`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotNotEqualTo.apply` (lines 65-71)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        try:
            target = float(self.target)
        except ValueError:
            target = self.target

        return df[df[self.var] != target]
```
**EN:** Method `PlotNotEqualTo.apply` provides a reusable helper around the module's main workflow. Key calls such as `float` show the concrete execution path.
**CN:** Method `PlotNotEqualTo.apply` 为模块主流程提供可复用的辅助逻辑。 像 `float` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PlotLessThan` (lines 75-78)
```python
class PlotLessThan(PlotFilterBase):
    @override
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        return df[df[self.var] < float(self.target)]
```
**EN:** Class `PlotLessThan` is a structured building block in this module. It inherits from `PlotFilterBase` and uses a dataclass-style declaration to store explicit state. Key methods include `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotLessThan` 是该模块中的结构化构件，继承自 `PlotFilterBase`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotLessThan.apply` (lines 77-78)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        return df[df[self.var] < float(self.target)]
```
**EN:** Method `PlotLessThan.apply` provides a reusable helper around the module's main workflow. Key calls such as `float` show the concrete execution path.
**CN:** Method `PlotLessThan.apply` 为模块主流程提供可复用的辅助逻辑。 像 `float` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PlotLessThanOrEqualTo` (lines 82-85)
```python
class PlotLessThanOrEqualTo(PlotFilterBase):
    @override
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        return df[df[self.var] <= float(self.target)]
```
**EN:** Class `PlotLessThanOrEqualTo` is a structured building block in this module. It inherits from `PlotFilterBase` and uses a dataclass-style declaration to store explicit state. Key methods include `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotLessThanOrEqualTo` 是该模块中的结构化构件，继承自 `PlotFilterBase`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotLessThanOrEqualTo.apply` (lines 84-85)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        return df[df[self.var] <= float(self.target)]
```
**EN:** Method `PlotLessThanOrEqualTo.apply` provides a reusable helper around the module's main workflow. Key calls such as `float` show the concrete execution path.
**CN:** Method `PlotLessThanOrEqualTo.apply` 为模块主流程提供可复用的辅助逻辑。 像 `float` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PlotGreaterThan` (lines 89-92)
```python
class PlotGreaterThan(PlotFilterBase):
    @override
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        return df[df[self.var] > float(self.target)]
```
**EN:** Class `PlotGreaterThan` is a structured building block in this module. It inherits from `PlotFilterBase` and uses a dataclass-style declaration to store explicit state. Key methods include `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotGreaterThan` 是该模块中的结构化构件，继承自 `PlotFilterBase`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotGreaterThan.apply` (lines 91-92)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        return df[df[self.var] > float(self.target)]
```
**EN:** Method `PlotGreaterThan.apply` provides a reusable helper around the module's main workflow. Key calls such as `float` show the concrete execution path.
**CN:** Method `PlotGreaterThan.apply` 为模块主流程提供可复用的辅助逻辑。 像 `float` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PlotGreaterThanOrEqualTo` (lines 96-99)
```python
class PlotGreaterThanOrEqualTo(PlotFilterBase):
    @override
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        return df[df[self.var] >= float(self.target)]
```
**EN:** Class `PlotGreaterThanOrEqualTo` is a structured building block in this module. It inherits from `PlotFilterBase` and uses a dataclass-style declaration to store explicit state. Key methods include `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotGreaterThanOrEqualTo` 是该模块中的结构化构件，继承自 `PlotFilterBase`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotGreaterThanOrEqualTo.apply` (lines 98-99)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        return df[df[self.var] >= float(self.target)]
```
**EN:** Method `PlotGreaterThanOrEqualTo.apply` provides a reusable helper around the module's main workflow. Key calls such as `float` show the concrete execution path.
**CN:** Method `PlotGreaterThanOrEqualTo.apply` 为模块主流程提供可复用的辅助逻辑。 像 `float` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PlotFilters` (lines 113-125)
```python
class PlotFilters(list[PlotFilterBase]):
    @classmethod
    def parse_str(cls, s: str):
        if not s:
            return cls()

        return cls(PlotFilterBase.parse_str(e) for e in s.split(","))

    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        for item in self:
            df = item.apply(df)

        return df
```
**EN:** Class `PlotFilters` is a structured building block in this module. It inherits from `list[PlotFilterBase]`. Key methods include `parse_str`, `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotFilters` 是该模块中的结构化构件，继承自 `list[PlotFilterBase]`。 关键方法包括 `parse_str`, `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotFilters.parse_str` (lines 115-119)
```python
    def parse_str(cls, s: str):
        if not s:
            return cls()

        return cls(PlotFilterBase.parse_str(e) for e in s.split(","))
```
**EN:** Method `PlotFilters.parse_str` parses configuration, arguments, or structured metadata. Key calls such as `cls`, `PlotFilterBase.parse_str`, `s.split` show the concrete execution path.
**CN:** Method `PlotFilters.parse_str` 负责解析配置、参数或结构化元数据。 像 `cls`, `PlotFilterBase.parse_str`, `s.split` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PlotFilters.apply` (lines 121-125)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        for item in self:
            df = item.apply(df)

        return df
```
**EN:** Method `PlotFilters.apply` provides a reusable helper around the module's main workflow. Key calls such as `item.apply` show the concrete execution path.
**CN:** Method `PlotFilters.apply` 为模块主流程提供可复用的辅助逻辑。 像 `item.apply` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PlotBinner` (lines 129-149)
```python
class PlotBinner:
    var: str
    bin_size: float

    @classmethod
    def parse_str(cls, s: str):
        for op_key in PLOT_BINNERS:
            if op_key in s:
                key, value = s.split(op_key)
                return PLOT_BINNERS[op_key](key, float(value.removeprefix(op_key)))
        else:
            raise ValueError(
                f"Invalid operator for plot binner '{s}'. "
                f"Valid operators are: {sorted(PLOT_BINNERS)}",
            )

    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        """Applies this binner to a DataFrame."""
        df = df.copy()
        df[self.var] = df[self.var] // self.bin_size * self.bin_size
        return df
```
**EN:** Class `PlotBinner` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `parse_str`, `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotBinner` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `parse_str`, `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotBinner.parse_str` (lines 134-143)
```python
    def parse_str(cls, s: str):
        for op_key in PLOT_BINNERS:
            if op_key in s:
                key, value = s.split(op_key)
                return PLOT_BINNERS[op_key](key, float(value.removeprefix(op_key)))
        else:
            raise ValueError(
                f"Invalid operator for plot binner '{s}'. "
                f"Valid operators are: {sorted(PLOT_BINNERS)}",
            )
```
**EN:** Method `PlotBinner.parse_str` parses configuration, arguments, or structured metadata. Key calls such as `s.split`, `float`, `value.removeprefix`, `ValueError`, `sorted` show the concrete execution path.
**CN:** Method `PlotBinner.parse_str` 负责解析配置、参数或结构化元数据。 像 `s.split`, `float`, `value.removeprefix`, `ValueError`, `sorted` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PlotBinner.apply` (lines 145-149)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        """Applies this binner to a DataFrame."""
        df = df.copy()
        df[self.var] = df[self.var] // self.bin_size * self.bin_size
        return df
```
**EN:** Method `PlotBinner.apply` provides a reusable helper around the module's main workflow. The docstring highlights: Applies this binner to a DataFrame. Key calls such as `df.copy` show the concrete execution path.
**CN:** Method `PlotBinner.apply` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Applies this binner to a DataFrame. 像 `df.copy` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PlotBinners` (lines 157-169)
```python
class PlotBinners(list[PlotBinner]):
    @classmethod
    def parse_str(cls, s: str):
        if not s:
            return cls()

        return cls(PlotBinner.parse_str(e) for e in s.split(","))

    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        for item in self:
            df = item.apply(df)

        return df
```
**EN:** Class `PlotBinners` is a structured building block in this module. It inherits from `list[PlotBinner]`. Key methods include `parse_str`, `apply`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PlotBinners` 是该模块中的结构化构件，继承自 `list[PlotBinner]`。 关键方法包括 `parse_str`, `apply`，它们共同定义初始化、校验、变换或访问模式。

### Method `PlotBinners.parse_str` (lines 159-163)
```python
    def parse_str(cls, s: str):
        if not s:
            return cls()

        return cls(PlotBinner.parse_str(e) for e in s.split(","))
```
**EN:** Method `PlotBinners.parse_str` parses configuration, arguments, or structured metadata. Key calls such as `cls`, `PlotBinner.parse_str`, `s.split` show the concrete execution path.
**CN:** Method `PlotBinners.parse_str` 负责解析配置、参数或结构化元数据。 像 `cls`, `PlotBinner.parse_str`, `s.split` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PlotBinners.apply` (lines 165-169)
```python
    def apply(self, df: "pd.DataFrame") -> "pd.DataFrame":
        for item in self:
            df = item.apply(df)

        return df
```
**EN:** Method `PlotBinners.apply` provides a reusable helper around the module's main workflow. Key calls such as `item.apply` show the concrete execution path.
**CN:** Method `PlotBinners.apply` 为模块主流程提供可复用的辅助逻辑。 像 `item.apply` 这样的关键调用展示了该代码块的具体执行路径。

### Class `DummyExecutor` (lines 222-234)
```python
class DummyExecutor:
    map = map

    def __enter__(self) -> Self:
        return self

    def __exit__(
        self,
        exc_type: type[BaseException] | None,
        exc_value: BaseException | None,
        exc_traceback: TracebackType | None,
    ) -> None:
        return None
```
**EN:** Class `DummyExecutor` is a structured building block in this module. Key methods include `__enter__`, `__exit__`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `DummyExecutor` 是该模块中的结构化构件。 关键方法包括 `__enter__`, `__exit__`，它们共同定义初始化、校验、变换或访问模式。

### Method `DummyExecutor.__enter__` (lines 225-226)
```python
    def __enter__(self) -> Self:
        return self
```
**EN:** Method `DummyExecutor.__enter__` provides a reusable helper around the module's main workflow.
**CN:** Method `DummyExecutor.__enter__` 为模块主流程提供可复用的辅助逻辑。

### Method `DummyExecutor.__exit__` (lines 228-234)
```python
    def __exit__(
        self,
        exc_type: type[BaseException] | None,
        exc_value: BaseException | None,
        exc_traceback: TracebackType | None,
    ) -> None:
        return None
```
**EN:** Method `DummyExecutor.__exit__` provides a reusable helper around the module's main workflow.
**CN:** Method `DummyExecutor.__exit__` 为模块主流程提供可复用的辅助逻辑。

### Class `SweepPlotArgs` (lines 480-655)
```python
class SweepPlotArgs:
    output_dir: Path
    fig_dir: Path
    fig_by: list[str]
    row_by: list[str]
    col_by: list[str]
    curve_by: list[str]
    var_x: str
    var_y: str
    filter_by: PlotFilters
    bin_by: PlotBinners
    scale_x: str | None
    scale_y: str | None
    dry_run: bool
    fig_name: str = "FIGURE"
    error_bars: bool = True
    fig_height: float = 6.4
    fig_dpi: int = 300

    parser_name: ClassVar[str] = "plot"
    parser_help: ClassVar[str] = "Plot performance curves from parameter sweep results."

    @classmethod
    # ... omitted for brevity ...

        return parser
```
**EN:** Class `SweepPlotArgs` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `from_cli_args`, `add_cli_args`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `SweepPlotArgs` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `from_cli_args`, `add_cli_args`，它们共同定义初始化、校验、变换或访问模式。

### Method `SweepPlotArgs.from_cli_args` (lines 503-531)
```python
    def from_cli_args(cls, args: argparse.Namespace):
        output_dir = Path(args.EXPERIMENT_DIR)
        if not output_dir.exists():
            raise ValueError(f"No parameter sweep results under {output_dir}")

        curve_by = [] if not args.curve_by else args.curve_by.split(",")
        row_by = [] if not args.row_by else args.row_by.split(",")
        col_by = [] if not args.col_by else args.col_by.split(",")
        fig_by = [] if not args.fig_by else args.fig_by.split(",")

        return cls(
            output_dir=output_dir,
            fig_dir=output_dir / args.fig_dir,
            fig_by=fig_by,
            row_by=row_by,
            col_by=col_by,
            curve_by=curve_by,
            var_x=args.var_x,
            var_y=args.var_y,
    # ... omitted for brevity ...
            fig_dpi=args.fig_dpi,
        )
```
**EN:** Method `SweepPlotArgs.from_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `Path`, `output_dir.exists`, `ValueError`, `args.curve_by.split`, `args.row_by.split` show the concrete execution path.
**CN:** Method `SweepPlotArgs.from_cli_args` 负责解析配置、参数或结构化元数据。 像 `Path`, `output_dir.exists`, `ValueError`, `args.curve_by.split`, `args.row_by.split` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepPlotArgs.add_cli_args` (lines 534-655)
```python
    def add_cli_args(cls, parser: argparse.ArgumentParser) -> argparse.ArgumentParser:
        parser.add_argument(
            "EXPERIMENT_DIR",
            type=str,
            help="The directory containing the sweep results to plot.",
        )
        parser.add_argument(
            "--fig-dir",
            type=str,
            default="",
            help="The directory to save the figures, relative to `OUTPUT_DIR`. "
            "By default, the same directory is used.",
        )
        parser.add_argument(
            "--fig-by",
            type=str,
            default="",
            help="A comma-separated list of variables, such that a separate figure "
            "is created for each combination of these variables.",
    # ... omitted for brevity ...

        return parser
```
**EN:** Method `SweepPlotArgs.add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `parser.add_argument` show the concrete execution path.
**CN:** Method `SweepPlotArgs.add_cli_args` 负责解析配置、参数或结构化元数据。 像 `parser.add_argument` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 680-681)
```python
def main(args: argparse.Namespace):
    run_main(SweepPlotArgs.from_cli_args(args))
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `run_main`, `SweepPlotArgs.from_cli_args` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `run_main`, `SweepPlotArgs.from_cli_args` 这样的关键调用展示了该代码块的具体执行路径。

### Entrypoint guard (lines 684-688)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description=SweepPlotArgs.parser_help)
    SweepPlotArgs.add_cli_args(parser)

    main(parser.parse_args())
```
**EN:** This standard `__main__` guard turns the module into an executable script and forwards control to the top-level CLI or main workflow.
**CN:** 标准的 `__main__` 守卫让该模块可以作为脚本执行，并把控制流转交给顶层 CLI 或主流程。

## Key Concepts / 关键概念
- **CLI workflow / 命令行流程**
  - **EN:** The module exposes a command-line entrypoint so it can be run as a standalone tool.
  - **CN:** 该模块暴露了命令行入口，因此可以作为独立工具运行。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。
- **Serialization / 序列化**
  - **EN:** The module converts in-memory structures into stable textual or binary representations.
  - **CN:** 该模块会把内存中的结构转换为稳定的文本或二进制表示。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import json`, `from abc import ABC, abstractmethod`, `from concurrent.futures import ProcessPoolExecutor`, `from dataclasses import dataclass`, `from functools import partial`, `from pathlib import Path`, `from types import TracebackType`, `from typing import TYPE_CHECKING, ClassVar`
- **Third-party / 第三方**: `from typing_extensions import Self, override`
- **vLLM internal / vLLM 内部依赖**: `from vllm.utils.collection_utils import full_groupby`, `from vllm.utils.import_utils import PlaceholderModule`, `from .utils import sanitize_filename`
