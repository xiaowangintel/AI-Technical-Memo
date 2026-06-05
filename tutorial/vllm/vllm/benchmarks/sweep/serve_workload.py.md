# serve_workload.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/sweep/serve_workload.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements serve workload support for the `sweep` portion of vLLM. / 为 vLLM 的 `sweep` 子目录实现与 服务workload 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-22)
```python
import argparse

import math

from dataclasses import dataclass, fields

from pathlib import Path

from typing import ClassVar, Literal, get_args

import numpy as np

from typing_extensions import assert_never

from vllm.benchmarks.datasets import DEFAULT_NUM_PROMPTS

from vllm.utils.import_utils import PlaceholderModule

from .param_sweep import ParameterSweep, ParameterSweepItem

from .serve import (
    SweepServeArgs,
    _get_comb_base_path,
    run_comb,
    server_ctx,
)

from .server import ServerProcess
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 30-30)
```python
WorkloadVariable = Literal["request_rate", "max_concurrency"]
```
**EN:** This constant/configuration block defines `WorkloadVariable`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `WorkloadVariable`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `_estimate_workload_value` (lines 33-44)
```python
def _estimate_workload_value(
    run_data: dict[str, object],
    workload_var: WorkloadVariable,
):
    request_throughput = float(run_data["request_throughput"])  # type: ignore
    if workload_var == "request_rate":
        return request_throughput
    if workload_var == "max_concurrency":
        mean_latency_ms = float(run_data["mean_e2el_ms"])  # type: ignore
        return request_throughput * mean_latency_ms / 1000

    assert_never(workload_var)
```
**EN:** Function `_estimate_workload_value` handles loading or retrieval of external/internal data. Key calls such as `float`, `assert_never` show the concrete execution path.
**CN:** Function `_estimate_workload_value` 负责加载或获取外部/内部数据。 像 `float`, `assert_never` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_estimate_workload_avg` (lines 47-52)
```python
def _estimate_workload_avg(
    runs: list[dict[str, object]],
    workload_var: WorkloadVariable,
):
    total = sum(_estimate_workload_value(run, workload_var) for run in runs)
    return total / len(runs)
```
**EN:** Function `_estimate_workload_avg` handles loading or retrieval of external/internal data. Key calls such as `sum`, `_estimate_workload_value`, `len` show the concrete execution path.
**CN:** Function `_estimate_workload_avg` 负责加载或获取外部/内部数据。 像 `sum`, `_estimate_workload_value`, `len` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_comb_workload` (lines 55-84)
```python
def run_comb_workload(
    server: ServerProcess | None,
    bench_cmd: list[str],
    *,
    serve_comb: ParameterSweepItem,
    bench_comb: ParameterSweepItem,
    link_vars: list[tuple[str, str]],
    experiment_dir: Path,
    num_runs: int,
    dry_run: bool,
    workload_var: WorkloadVariable,
    workload_value: int,
) -> list[dict[str, object]] | None:
    bench_comb_workload = bench_comb | {workload_var: workload_value}

    return run_comb(
        server,
        bench_cmd,
        serve_comb=serve_comb,
        bench_comb=bench_comb_workload,
        link_vars=link_vars,
        base_path=_get_comb_base_path(
            experiment_dir,
            serve_comb,
            bench_comb,
    # ... omitted for brevity ...
        dry_run=dry_run,
    )
```
**EN:** Function `run_comb_workload` handles loading or retrieval of external/internal data. Key calls such as `run_comb`, `_get_comb_base_path` show the concrete execution path.
**CN:** Function `run_comb_workload` 负责加载或获取外部/内部数据。 像 `run_comb`, `_get_comb_base_path` 这样的关键调用展示了该代码块的具体执行路径。

### Function `explore_comb_workloads` (lines 87-191)
```python
def explore_comb_workloads(
    server: ServerProcess | None,
    bench_cmd: list[str],
    *,
    serve_comb: ParameterSweepItem,
    bench_comb: ParameterSweepItem,
    link_vars: list[tuple[str, str]],
    workload_var: WorkloadVariable,
    workload_iters: int,
    experiment_dir: Path,
    num_runs: int,
    dry_run: bool,
):
    print("[WL START]")
    print(f"Serve parameters: {serve_comb.as_text() or '(None)'}")
    print(f"Bench parameters: {bench_comb.as_text() or '(None)'}")
    print(f"Number of workload iterations: {workload_iters}")

    if workload_iters < 2:
        raise ValueError("`workload_iters` should be at least 2")

    dataset_size = DEFAULT_NUM_PROMPTS
    if "num_prompts" in bench_comb:
        dataset_size = int(bench_comb["num_prompts"])  # type: ignore
    else:
    # ... omitted for brevity ...

    return serial_workload_data + inter_workloads_data + batch_workload_data
```
**EN:** Function `explore_comb_workloads` handles loading or retrieval of external/internal data. Key calls such as `print`, `serve_comb.as_text`, `bench_comb.as_text`, `ValueError`, `int` show the concrete execution path.
**CN:** Function `explore_comb_workloads` 负责加载或获取外部/内部数据。 像 `print`, `serve_comb.as_text`, `bench_comb.as_text`, `ValueError`, `int` 这样的关键调用展示了该代码块的具体执行路径。

### Function `explore_combs_workloads` (lines 194-251)
```python
def explore_combs_workloads(
    serve_cmd: list[str],
    bench_cmd: list[str],
    after_bench_cmd: list[str],
    *,
    show_stdout: bool,
    server_ready_timeout: int,
    serve_params: ParameterSweep,
    bench_params: ParameterSweep,
    link_vars: list[tuple[str, str]],
    workload_var: WorkloadVariable,
    workload_iters: int,
    experiment_dir: Path,
    num_runs: int,
    dry_run: bool,
):
    if any(bench_comb.has_param(workload_var) for bench_comb in bench_params):
        raise ValueError(
            f"You should not override `{workload_var}` in `bench_params` "
            "since it is supposed to be explored automatically."
        )

    all_data = list[dict[str, object]]()
    for serve_comb in serve_params:
        with server_ctx(
    # ... omitted for brevity ...

    return combined_df
```
**EN:** Function `explore_combs_workloads` handles loading or retrieval of external/internal data. Key calls such as `any`, `bench_comb.has_param`, `ValueError`, `server_ctx`, `explore_comb_workloads` show the concrete execution path.
**CN:** Function `explore_combs_workloads` 负责加载或获取外部/内部数据。 像 `any`, `bench_comb.has_param`, `ValueError`, `server_ctx`, `explore_comb_workloads` 这样的关键调用展示了该代码块的具体执行路径。

### Class `SweepServeWorkloadArgs` (lines 255-296)
```python
class SweepServeWorkloadArgs(SweepServeArgs):
    workload_var: WorkloadVariable
    workload_iters: int

    parser_name: ClassVar[str] = "serve_workload"
    parser_help: ClassVar[str] = (
        "Explore the latency-throughput tradeoff for different workload levels."
    )

    @classmethod
    def from_cli_args(cls, args: argparse.Namespace):
        # NOTE: Don't use super() as `from_cli_args` calls `cls()`
        base_args = SweepServeArgs.from_cli_args(args)

        return cls(
            **{f.name: getattr(base_args, f.name) for f in fields(base_args)},
            workload_var=args.workload_var,
            workload_iters=args.workload_iters,
        )

    @classmethod
    def add_cli_args(cls, parser: argparse.ArgumentParser) -> argparse.ArgumentParser:
        parser = super().add_cli_args(parser)
    # ... omitted for brevity ...

        return parser
```
**EN:** Class `SweepServeWorkloadArgs` is a structured building block in this module. It inherits from `SweepServeArgs` and uses a dataclass-style declaration to store explicit state. Key methods include `from_cli_args`, `add_cli_args`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `SweepServeWorkloadArgs` 是该模块中的结构化构件，继承自 `SweepServeArgs`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `from_cli_args`, `add_cli_args`，它们共同定义初始化、校验、变换或访问模式。

### Method `SweepServeWorkloadArgs.from_cli_args` (lines 265-273)
```python
    def from_cli_args(cls, args: argparse.Namespace):
        # NOTE: Don't use super() as `from_cli_args` calls `cls()`
        base_args = SweepServeArgs.from_cli_args(args)

        return cls(
            **{f.name: getattr(base_args, f.name) for f in fields(base_args)},
            workload_var=args.workload_var,
            workload_iters=args.workload_iters,
        )
```
**EN:** Method `SweepServeWorkloadArgs.from_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `SweepServeArgs.from_cli_args`, `cls`, `getattr`, `fields` show the concrete execution path.
**CN:** Method `SweepServeWorkloadArgs.from_cli_args` 负责解析配置、参数或结构化元数据。 像 `SweepServeArgs.from_cli_args`, `cls`, `getattr`, `fields` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepServeWorkloadArgs.add_cli_args` (lines 276-296)
```python
    def add_cli_args(cls, parser: argparse.ArgumentParser) -> argparse.ArgumentParser:
        parser = super().add_cli_args(parser)

        workload_group = parser.add_argument_group("workload options")
        workload_group.add_argument(
            "--workload-var",
            type=str,
            choices=get_args(WorkloadVariable),
            default="request_rate",
            help="The variable to adjust in each iteration.",
        )
        workload_group.add_argument(
            "--workload-iters",
            type=int,
            default=10,
            help="Number of workload levels to explore. "
            "This includes the first two iterations used to interpolate the value of "
            "`workload_var` for remaining iterations.",
        )

        return parser
```
**EN:** Method `SweepServeWorkloadArgs.add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `super().add_cli_args`, `super`, `parser.add_argument_group`, `workload_group.add_argument`, `get_args` show the concrete execution path.
**CN:** Method `SweepServeWorkloadArgs.add_cli_args` 负责解析配置、参数或结构化元数据。 像 `super().add_cli_args`, `super`, `parser.add_argument_group`, `workload_group.add_argument`, `get_args` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_main` (lines 299-317)
```python
def run_main(args: SweepServeWorkloadArgs):
    experiment_dir = args.resolve_experiment_dir()

    with args.run_ctx(experiment_dir):
        return explore_combs_workloads(
            serve_cmd=args.serve_cmd,
            bench_cmd=args.bench_cmd,
            after_bench_cmd=args.after_bench_cmd,
            show_stdout=args.show_stdout,
            server_ready_timeout=args.server_ready_timeout,
            serve_params=args.serve_params,
            bench_params=args.bench_params,
            link_vars=args.link_vars,
            workload_var=args.workload_var,
            workload_iters=args.workload_iters,
            experiment_dir=experiment_dir,
            num_runs=args.num_runs,
            dry_run=args.dry_run,
        )
```
**EN:** Function `run_main` provides a reusable helper around the module's main workflow. Key calls such as `args.resolve_experiment_dir`, `args.run_ctx`, `explore_combs_workloads` show the concrete execution path.
**CN:** Function `run_main` 为模块主流程提供可复用的辅助逻辑。 像 `args.resolve_experiment_dir`, `args.run_ctx`, `explore_combs_workloads` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 320-321)
```python
def main(args: argparse.Namespace):
    run_main(SweepServeWorkloadArgs.from_cli_args(args))
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `run_main`, `SweepServeWorkloadArgs.from_cli_args` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `run_main`, `SweepServeWorkloadArgs.from_cli_args` 这样的关键调用展示了该代码块的具体执行路径。

### Entrypoint guard (lines 324-328)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description=SweepServeWorkloadArgs.parser_help)
    SweepServeWorkloadArgs.add_cli_args(parser)

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
- **Standard library / 标准库**: `import argparse`, `import math`, `from dataclasses import dataclass, fields`, `from pathlib import Path`, `from typing import ClassVar, Literal, get_args`
- **Third-party / 第三方**: `import numpy as np`, `from typing_extensions import assert_never`
- **vLLM internal / vLLM 内部依赖**: `from vllm.benchmarks.datasets import DEFAULT_NUM_PROMPTS`, `from vllm.utils.import_utils import PlaceholderModule`, `from .param_sweep import ParameterSweep, ParameterSweepItem`, `from .serve import SweepServeArgs, _get_comb_base_path, run_comb, server_ctx`, `from .server import ServerProcess`
