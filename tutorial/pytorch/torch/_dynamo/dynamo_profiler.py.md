# dynamo_profiler.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/dynamo_profiler.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: Dynamo Profiler - tracks where Dynamo spends time during compilation.
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```python
"""
Dynamo Profiler - tracks where Dynamo spends time during compilation.

This module provides profiling functionality for Dynamo tracing, showing per-function
cumtime (inclusive) and tottime (exclusive) in a cProfile-compatible format.
The output can be visualized with tools like snakeviz.

Usage:
    # Enable via config (prints pstats output):
    torch._dynamo.config.dynamo_profiler = True

    # Or save to file for snakeviz:
    torch._dynamo.config.dynamo_profiler = "/tmp/dynamo.prof"
    # Then: snakeviz /tmp/dynamo.prof
"""

from __future__ import annotations

from dataclasses import dataclass
from typing import Any, TYPE_CHECKING


if TYPE_CHECKING:
    import pstats
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 27-44
```python
@dataclass
class FunctionTraceTiming:
    """
    Timing data for a single inlined function trace.

    Follows cProfile conventions:
    - cumtime: total time in function including all subcalls (inclusive)
    - tottime: time in function excluding subcalls (exclusive)
    - caller info: who called this function (for building call graph)
    """

    # Function identification
    func_name: str
    filename: str
    firstlineno: int
    # Timing data (in nanoseconds) - cProfile-style
    cumtime_ns: int  # Inclusive time (includes subcalls)
    tottime_ns: int  # Exclusive time (excludes subcalls)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 45-68
```python
    # Code stats (for comparing tracing overhead vs function complexity)
    bytecode_count: int
    # Nesting depth when this function was traced
    inline_depth: int
    # Caller information (for building call graph edges)
    caller_func_name: str | None = None
    caller_filename: str | None = None
    caller_firstlineno: int | None = None
    # Whether this is a primitive (non-recursive) call
    # A call is primitive if the function doesn't appear anywhere in the call stack
    is_primitive_call: bool = True
    # Full call stack at the time of this call (for proper snakeviz drill-down)
    # Each entry is (func_name, filename, firstlineno)
    call_stack: tuple[tuple[str, str, int], ...] = ()

    # Backwards compatibility alias
    @property
    def trace_time_ns(self) -> int:
        return self.cumtime_ns

    @property
    def trace_time_ms(self) -> float:
        return self.cumtime_ns / 1e6
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 69-92
```python
    @property
    def cumtime_ms(self) -> float:
        return self.cumtime_ns / 1e6

    @property
    def tottime_ms(self) -> float:
        return self.tottime_ns / 1e6

    @property
    def caller_key(self) -> tuple[str, int, str] | None:
        """Return caller as a pstats-compatible key tuple."""
        if self.caller_func_name is not None:
            return (
                self.caller_filename or "",
                self.caller_firstlineno or 0,
                self.caller_func_name,
            )
        return None

    @property
    def func_key(self) -> tuple[str, int, str]:
        """Return this function as a pstats-compatible key tuple."""
        return (self.filename, self.firstlineno, self.func_name)
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 93-119
```python
    def __repr__(self) -> str:
        return (
            f"FunctionTraceTiming({self.func_name} at {self.filename}:{self.firstlineno}, "
            f"cumtime={self.cumtime_ms:.2f}ms, tottime={self.tottime_ms:.2f}ms, "
            f"bytecode={self.bytecode_count}, depth={self.inline_depth})"
        )


@dataclass
class ProfilerStackEntry:
    """Stack entry for tracking function timing in the Dynamo profiler."""

    func_name: str
    filename: str
    firstlineno: int
    start_time_ns: int
    child_time_ns: int  # Accumulated time spent in traced children
    is_primitive_call: bool = True  # Whether this is a non-recursive call


class DynamoProfilerState:
    """State for Dynamo profiler tracking function trace timings."""

    def __init__(self) -> None:
        self.timings: list[FunctionTraceTiming] = []
        self.stack: list[ProfilerStackEntry] = []
```
- **EN**: These decorators register or transform the following definition so it can trace Python execution into an intermediate graph representation.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将 Python 执行过程跟踪为中间图表示。

### Lines 120-137
```python
    def record_timing(self, timing: FunctionTraceTiming) -> None:
        """Record timing data for a traced function."""
        self.timings.append(timing)

    def get_timings(self) -> list[FunctionTraceTiming]:
        """Get all recorded timings."""
        return self.timings

    def push(
        self, func_name: str, filename: str, firstlineno: int, start_time_ns: int
    ) -> None:
        """Push a new entry onto the timing stack."""
        # Check if this function already exists in the stack (indirect recursion)
        is_primitive = not any(
            entry.func_name == func_name
            and entry.filename == filename
            and entry.firstlineno == firstlineno
            for entry in self.stack
```
- **EN**: Declares `DynamoProfilerState`; this class packages state and methods that trace Python execution into an intermediate graph representation.
- **CN**: 声明 `DynamoProfilerState`；该类封装了状态与方法，用于将 Python 执行过程跟踪为中间图表示。

### Lines 138-160
```python
        )
        self.stack.append(
            ProfilerStackEntry(
                func_name=func_name,
                filename=filename,
                firstlineno=firstlineno,
                start_time_ns=start_time_ns,
                child_time_ns=0,
                is_primitive_call=is_primitive,
            )
        )

    def pop(self) -> ProfilerStackEntry | None:
        """Pop the top entry from the timing stack."""
        if self.stack:
            return self.stack.pop()
        return None

    def add_child_time(self, child_cumtime_ns: int) -> None:
        """Add the child's cumulative time to the parent's child_time accumulator."""
        if self.stack:
            self.stack[-1].child_time_ns += child_cumtime_ns
```
- **EN**: Declares `DynamoProfilerState`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `DynamoProfilerState`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 161-187
```python
    def get_current_caller(self) -> tuple[str, str, int] | None:
        """Get the current caller (top of stack) as (func_name, filename, firstlineno)."""
        if self.stack:
            entry = self.stack[-1]
            return (entry.func_name, entry.filename, entry.firstlineno)
        return None

    def get_call_stack(self) -> tuple[tuple[str, str, int], ...]:
        """Get the full current call stack as tuple of (func_name, filename, firstlineno)."""
        return tuple(
            (entry.func_name, entry.filename, entry.firstlineno) for entry in self.stack
        )

    def generate_pstats(
        self, output_file: str | None = None, print_raw: bool = False
    ) -> pstats.Stats:
        """Generate pstats.Stats object from recorded timings.

        Args:
            output_file: Optional file path to save the stats.
            print_raw: If True, print raw aggregate timings before returning.
        """
        import cProfile
        import io
        import logging
        import pstats
```
- **EN**: Declares `DynamoProfilerState`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `DynamoProfilerState`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 188-212
```python
        log = logging.getLogger(__name__)

        # Aggregate by (filename, lineno, func_name)
        aggregated: dict[tuple[str, int, str], dict[str, Any]] = {}
        # caller_edges[callee_key][caller_key] -> edge stats
        caller_edges: dict[
            tuple[str, int, str], dict[tuple[str, int, str], dict[str, Any]]
        ] = {}

        for t in self.timings:
            key = (t.filename, t.firstlineno, t.func_name)

            if key not in aggregated:
                aggregated[key] = {
                    "ncalls": 0,
                    "pcalls": 0,
                    "tottime": 0.0,
                    "cumtime": 0.0,
                }
                caller_edges[key] = {}

            agg = aggregated[key]
            agg["ncalls"] += 1
            agg["tottime"] += t.tottime_ns / 1e9
```
- **EN**: This block continues `DynamoProfilerState.generate_pstats` and works to capture Python execution for torch.compile and maintain compiler state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `DynamoProfilerState.generate_pstats`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 213-239
```python
            if t.is_primitive_call:
                agg["pcalls"] += 1
                agg["cumtime"] += t.cumtime_ns / 1e9

            # Build caller edge
            if t.caller_filename is not None:
                caller_key = (
                    t.caller_filename,
                    t.caller_firstlineno or 0,
                    t.caller_func_name or "",
                )
                if caller_key not in caller_edges[key]:
                    caller_edges[key][caller_key] = {
                        "ncalls": 0,
                        "pcalls": 0,
                        "tottime": 0.0,
                        "cumtime": 0.0,
                    }
                edge = caller_edges[key][caller_key]
                edge["ncalls"] += 1
                edge["tottime"] += t.tottime_ns / 1e9
                # Always add cumtime to edges for visualization (gprof2dot)
                # Function-level cumtime is already correct (only primitive calls)
                edge["cumtime"] += t.cumtime_ns / 1e9
                if t.is_primitive_call:
                    edge["pcalls"] += 1
```
- **EN**: This block continues `DynamoProfilerState.generate_pstats` and works to capture Python execution for torch.compile and maintain compiler state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `DynamoProfilerState.generate_pstats`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会根据运行时条件分支处理。

### Lines 240-257
```python
        if print_raw:
            sorted_items = sorted(
                aggregated.items(), key=lambda x: x[1]["cumtime"], reverse=True
            )
            print("\n=== Aggregate Timings (raw) ===")
            print(
                f"{'ncalls':>8} {'pcalls':>8} {'tottime':>12} {'cumtime':>12}  function"
            )
            print("-" * 80)
            total_cumtime = 0.0
            total_tottime = 0.0
            for (filename, lineno, func_name), agg in sorted_items:
                ncalls = agg["ncalls"]
                pcalls = agg["pcalls"]
                tottime = agg["tottime"] * 1000  # convert to ms
                cumtime = agg["cumtime"] * 1000
                total_cumtime += cumtime
                total_tottime += tottime
```
- **EN**: This block continues `DynamoProfilerState.generate_pstats` and works to capture Python execution for torch.compile and maintain compiler state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `DynamoProfilerState.generate_pstats`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 258-284
```python
                short_file = filename.split("/")[-1] if "/" in filename else filename
                print(
                    f"{ncalls:>8} {pcalls:>8} {tottime:>10.2f}ms {cumtime:>10.2f}ms  "
                    f"{func_name} ({short_file}:{lineno})"
                )
            print("-" * 80)
            print(
                f"Total timings: {len(self.timings)}, unique functions: {len(aggregated)}"
            )
            print(
                f"Sum tottime: {total_tottime:.2f}ms, Sum cumtime: {total_cumtime:.2f}ms"
            )

        # Ensure caller-only functions have a top-level entry.
        # gprof2dot expects every function referenced as a caller to also
        # exist as a top-level entry in the stats dict with timing data.
        for key in list(caller_edges.keys()):
            for caller_key in caller_edges[key]:
                if caller_key not in aggregated:
                    aggregated[caller_key] = {
                        "ncalls": 0,
                        "pcalls": 0,
                        "tottime": 0.0,
                        "cumtime": 0.0,
                    }
                    caller_edges[caller_key] = {}
```
- **EN**: This block continues `DynamoProfilerState.generate_pstats` and works to capture Python execution for torch.compile and maintain compiler state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `DynamoProfilerState.generate_pstats`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 285-307
```python
        # Build the stats dict in pstats format
        stats_dict: dict[
            tuple[str, int, str], tuple[int, int, float, float, dict[Any, Any]]
        ] = {}

        for key, agg in aggregated.items():
            callers: dict[tuple[str, int, str], tuple[int, int, float, float]] = {}
            for caller_key, edge in caller_edges[key].items():
                callers[caller_key] = (
                    edge["ncalls"],
                    edge["pcalls"],
                    edge["tottime"],
                    edge["cumtime"],
                )

            stats_dict[key] = (
                agg["pcalls"],
                agg["ncalls"],
                agg["tottime"],
                agg["cumtime"],
                callers,
            )
```
- **EN**: This block continues `DynamoProfilerState.generate_pstats` and works to capture Python execution for torch.compile and maintain compiler state. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `DynamoProfilerState.generate_pstats`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会遍历输入、节点或簿记结构。

### Lines 308-333
```python
        # Create a pstats.Stats object
        dummy_profile = cProfile.Profile()
        dummy_profile.enable()
        dummy_profile.disable()
        stats = pstats.Stats(dummy_profile, stream=io.StringIO())

        stats.stats = stats_dict  # type: ignore[attr-defined]
        stats.total_calls = sum(s[1] for s in stats_dict.values())  # type: ignore[attr-defined]
        stats.prim_calls = sum(s[0] for s in stats_dict.values())  # type: ignore[attr-defined]
        stats.total_tt = sum(s[2] for s in stats_dict.values())  # type: ignore[attr-defined]

        if output_file:
            stats.dump_stats(output_file)
            log.info(
                "Saved pstats to %s. Visualize with: snakeviz %s",
                output_file,
                output_file,
            )

        return stats

    def generate_svg(
        self, profile_file: str, svg_file: str | None = None
    ) -> str | None:
        """Generate an SVG call graph from a profile file using gprof2dot and graphviz.
```
- **EN**: Declares `DynamoProfilerState`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `DynamoProfilerState`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 334-356
```python
        Args:
            profile_file: Path to the pstats profile file.
            svg_file: Optional path for the output SVG. If not provided, uses
                profile_file with .svg extension.

        Returns:
            Path to the generated SVG file, or None if generation failed.
        """
        import os
        import shutil
        import subprocess

        if not shutil.which("gprof2dot"):
            print("gprof2dot not found. Install with: pip install gprof2dot")
            return None

        if not shutil.which("dot"):
            print("graphviz 'dot' not found. Install graphviz package.")
            return None

        if svg_file is None:
            svg_file = profile_file.rsplit(".", 1)[0] + ".svg"
```
- **EN**: This block continues `DynamoProfilerState.generate_svg` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `DynamoProfilerState.generate_svg`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 357-381
```python
        try:
            # gprof2dot -f pstats profile.prof | dot -Tsvg -o profile.svg
            gprof2dot = subprocess.Popen(
                [
                    "gprof2dot",
                    "-f",
                    "pstats",
                    "--node-label=total-time-percentage",
                    "--node-label=self-time-percentage",
                    "--node-label=total-time",
                    profile_file,
                ],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
            )
            dot = subprocess.Popen(
                ["dot", "-Tsvg", "-o", svg_file],
                stdin=gprof2dot.stdout,
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
            )
            gprof2dot.stdout.close()  # type: ignore[union-attr]
            _, dot_err = dot.communicate()
            _, gprof2dot_err = gprof2dot.communicate()
```
- **EN**: This block continues `DynamoProfilerState.generate_svg` and works to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 该代码块继续实现 `DynamoProfilerState.generate_svg`，用于构建、遍历或改写图结构及其元数据。

### Lines 382-405
```python
            if gprof2dot.returncode != 0:
                print(f"gprof2dot failed: {gprof2dot_err.decode()}")
                return None

            if dot.returncode != 0:
                print(f"graphviz dot failed: {dot_err.decode()}")
                return None

            if not os.path.isfile(svg_file):
                print(f"SVG file was not created: {svg_file}")
                return None

            print(f"SVG call graph saved to: {svg_file}")
            return svg_file

        except Exception as e:
            print(f"Failed to generate SVG: {e}")
            return None

    def dump_stats(
        self, output_file: str | None = None, generate_svg: bool = True
    ) -> None:
        """Print profiler stats to stdout and optionally save to file.
```
- **EN**: Declares `DynamoProfilerState`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `DynamoProfilerState`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 406-424
```python
        Args:
            output_file: Optional path to save the pstats profile.
            generate_svg: If True and output_file is provided, also generate an SVG
                call graph using gprof2dot and graphviz.
        """
        import sys

        if not self.timings:
            return

        stats = self.generate_pstats(output_file, print_raw=True)
        print("\n=== Dynamo Profiler (pstats) ===")
        stats.stream = sys.stdout  # type: ignore[attr-defined]
        stats.sort_stats("cumulative").print_stats()

        if output_file:
            print(f"\nProfile saved to: {output_file}")
            print(f"Visualize with: snakeviz {output_file}")
```
- **EN**: This block continues `DynamoProfilerState.dump_stats` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `DynamoProfilerState.dump_stats`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 425-426
```python
            if generate_svg:
                self.generate_svg(output_file)
```
- **EN**: This block continues `DynamoProfilerState.dump_stats` and works to capture Python execution for torch.compile and maintain compiler state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `DynamoProfilerState.dump_stats`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会根据运行时条件分支处理。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Python execution capture / Python 执行捕获**
  - EN: The file hooks or rewrites Python execution machinery to observe user programs.
  - CN: 该文件会挂接或改写 Python 执行机制，以观察用户程序。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`, `pstats`, `cProfile`, `io`, `logging`, `os`, `shutil`, `subprocess`, `sys`
- **Primary symbols / 核心符号**: `FunctionTraceTiming`, `ProfilerStackEntry`, `DynamoProfilerState`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
