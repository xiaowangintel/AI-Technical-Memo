# serve.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/sweep/serve.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements serve support for the `sweep` portion of vLLM. / 为 vLLM 的 `sweep` 子目录实现与 服务 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-17)
```python
import argparse

import contextlib

import json

import shlex

from contextlib import contextmanager

from dataclasses import dataclass

from datetime import datetime

from pathlib import Path

from typing import ClassVar

from vllm.utils.import_utils import PlaceholderModule

from .param_sweep import ParameterSweep, ParameterSweepItem

from .server import ServerProcess

from .utils import sanitize_filename
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Function `run_server` (lines 26-50)
```python
def run_server(
    serve_cmd: list[str],
    after_bench_cmd: list[str],
    *,
    show_stdout: bool,
    serve_overrides: ParameterSweepItem,
    dry_run: bool,
    server_ready_timeout: int = 300,
):
    server_cmd = serve_overrides.apply_to_cmd(serve_cmd)

    print("[BEGIN SERVER]")
    print(f"Server overrides: {serve_overrides}")
    print(f"Server command: {server_cmd}")

    if dry_run:
        yield None
        print("[END SERVER]")
        return

    with ServerProcess(server_cmd, after_bench_cmd, show_stdout=show_stdout) as server:
        server.wait_until_ready(timeout=server_ready_timeout)
        yield server

    print("[END SERVER]")
```
**EN:** Function `run_server` coordinates benchmarking or serving-oriented control flow. Key calls such as `serve_overrides.apply_to_cmd`, `print`, `ServerProcess`, `server.wait_until_ready` show the concrete execution path.
**CN:** Function `run_server` 负责协调基准测试或服务侧控制流程。 像 `serve_overrides.apply_to_cmd`, `print`, `ServerProcess`, `server.wait_until_ready` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_update_run_data` (lines 53-63)
```python
def _update_run_data(
    run_data: dict[str, object],
    serve_overrides: ParameterSweepItem,
    bench_overrides: ParameterSweepItem,
    run_number: int,
):
    run_data["run_number"] = run_number
    run_data.update(serve_overrides)
    run_data.update(bench_overrides)

    return run_data
```
**EN:** Function `_update_run_data` provides a reusable helper around the module's main workflow. Key calls such as `run_data.update` show the concrete execution path.
**CN:** Function `_update_run_data` 为模块主流程提供可复用的辅助逻辑。 像 `run_data.update` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_benchmark` (lines 66-135)
```python
def run_benchmark(
    server: ServerProcess | None,
    bench_cmd: list[str],
    *,
    serve_overrides: ParameterSweepItem,
    bench_overrides: ParameterSweepItem,
    run_number: int,
    output_path: Path,
    dry_run: bool,
):
    benchmark_cmd = [
        *bench_overrides.apply_to_cmd(bench_cmd),
        "--percentile-metrics",
        "ttft,tpot,itl,e2el",
        "--save-result",
        "--result-dir",
        str(output_path.parent),
        "--result-filename",
        output_path.name,
    ]

    print("[BEGIN BENCHMARK]")
    print(f"Benchmark overrides: {bench_overrides}")
    print(f"Run Number: {run_number}")
    print(f"Benchmark command: {benchmark_cmd}")
    # ... omitted for brevity ...

    return run_data
```
**EN:** Function `run_benchmark` coordinates benchmarking or serving-oriented control flow. Key calls such as `bench_overrides.apply_to_cmd`, `str`, `print`, `output_path.exists`, `output_path.open` show the concrete execution path.
**CN:** Function `run_benchmark` 负责协调基准测试或服务侧控制流程。 像 `bench_overrides.apply_to_cmd`, `str`, `print`, `output_path.exists`, `output_path.open` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_comb_base_path` (lines 138-153)
```python
def _get_comb_base_path(
    experiment_dir: Path,
    serve_comb: ParameterSweepItem,
    bench_comb: ParameterSweepItem,
    *,
    extra_parts: tuple[str, ...] = (),
):
    parts = list[str]()
    if serve_comb:
        parts.extend(("SERVE-", serve_comb.name))
    if bench_comb:
        parts.extend(("BENCH-", bench_comb.name))
    if extra_parts:
        parts.extend(extra_parts)

    return experiment_dir / sanitize_filename("-".join(parts))
```
**EN:** Function `_get_comb_base_path` provides a reusable helper around the module's main workflow. Key calls such as `parts.extend`, `sanitize_filename`, `'-'.join` show the concrete execution path.
**CN:** Function `_get_comb_base_path` 为模块主流程提供可复用的辅助逻辑。 像 `parts.extend`, `sanitize_filename`, `'-'.join` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_comb_run_path` (lines 156-160)
```python
def _get_comb_run_path(base_path: Path, run_number: int | None):
    if run_number is None:
        return base_path / "summary.json"

    return base_path / f"run={run_number}.json"
```
**EN:** Function `_get_comb_run_path` provides a reusable helper around the module's main workflow.
**CN:** Function `_get_comb_run_path` 为模块主流程提供可复用的辅助逻辑。

### Function `_comb_needs_server` (lines 163-173)
```python
def _comb_needs_server(
    serve_comb: ParameterSweepItem,
    bench_combs: ParameterSweep,
    experiment_dir: Path,
):
    for bench_comb in bench_combs:
        base_path = _get_comb_base_path(experiment_dir, serve_comb, bench_comb)
        if not _get_comb_run_path(base_path, run_number=None).exists():
            return True

    return False
```
**EN:** Function `_comb_needs_server` coordinates benchmarking or serving-oriented control flow. Key calls such as `_get_comb_base_path`, `_get_comb_run_path(base_path, run_number=None).exists`, `_get_comb_run_path` show the concrete execution path.
**CN:** Function `_comb_needs_server` 负责协调基准测试或服务侧控制流程。 像 `_get_comb_base_path`, `_get_comb_run_path(base_path, run_number=None).exists`, `_get_comb_run_path` 这样的关键调用展示了该代码块的具体执行路径。

### Function `server_ctx` (lines 176-197)
```python
def server_ctx(
    serve_cmd: list[str],
    after_bench_cmd: list[str],
    *,
    show_stdout: bool,
    serve_comb: ParameterSweepItem,
    bench_params: ParameterSweep,
    experiment_dir: Path,
    dry_run: bool,
    server_ready_timeout: int = 300,
):
    if not _comb_needs_server(serve_comb, bench_params, experiment_dir):
        return contextlib.nullcontext()

    return run_server(
        serve_cmd,
        after_bench_cmd,
        show_stdout=show_stdout,
        serve_overrides=serve_comb,
        dry_run=dry_run,
        server_ready_timeout=server_ready_timeout,
    )
```
**EN:** Function `server_ctx` coordinates benchmarking or serving-oriented control flow. Key calls such as `_comb_needs_server`, `contextlib.nullcontext`, `run_server` show the concrete execution path.
**CN:** Function `server_ctx` 负责协调基准测试或服务侧控制流程。 像 `_comb_needs_server`, `contextlib.nullcontext`, `run_server` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_comb_is_valid` (lines 200-210)
```python
def _comb_is_valid(
    serve_comb: ParameterSweepItem,
    bench_comb: ParameterSweepItem,
    link_vars: list[tuple[str, str]],
) -> bool:
    return all(
        serve_key in serve_comb
        and bench_key in bench_comb
        and serve_comb[serve_key] == bench_comb[bench_key]
        for serve_key, bench_key in link_vars
    )
```
**EN:** Function `_comb_is_valid` provides a reusable helper around the module's main workflow. Key calls such as `all` show the concrete execution path.
**CN:** Function `_comb_is_valid` 为模块主流程提供可复用的辅助逻辑。 像 `all` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_comb` (lines 213-249)
```python
def run_comb(
    server: ServerProcess | None,
    bench_cmd: list[str],
    *,
    serve_comb: ParameterSweepItem,
    bench_comb: ParameterSweepItem,
    link_vars: list[tuple[str, str]],
    base_path: Path,
    num_runs: int,
    dry_run: bool,
):
    if not _comb_is_valid(serve_comb, bench_comb, link_vars):
        return None

    comb_data = list[dict[str, object]]()

    for run_number in range(num_runs):
        run_data = run_benchmark(
            server,
            bench_cmd,
            serve_overrides=serve_comb,
            bench_overrides=bench_comb,
            run_number=run_number,
            output_path=_get_comb_run_path(base_path, run_number),
            dry_run=dry_run,
    # ... omitted for brevity ...

    return comb_data
```
**EN:** Function `run_comb` provides a reusable helper around the module's main workflow. Key calls such as `_comb_is_valid`, `range`, `run_benchmark`, `_get_comb_run_path`, `comb_data.append` show the concrete execution path.
**CN:** Function `run_comb` 为模块主流程提供可复用的辅助逻辑。 像 `_comb_is_valid`, `range`, `run_benchmark`, `_get_comb_run_path`, `comb_data.append` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_combs` (lines 252-301)
```python
def run_combs(
    serve_cmd: list[str],
    bench_cmd: list[str],
    after_bench_cmd: list[str],
    *,
    show_stdout: bool,
    server_ready_timeout: int,
    serve_params: ParameterSweep,
    bench_params: ParameterSweep,
    link_vars: list[tuple[str, str]],
    experiment_dir: Path,
    num_runs: int,
    dry_run: bool,
):
    all_data = list[dict[str, object]]()
    for serve_comb in serve_params:
        with server_ctx(
            serve_cmd,
            after_bench_cmd,
            show_stdout=show_stdout,
            serve_comb=serve_comb,
            bench_params=bench_params,
            experiment_dir=experiment_dir,
            dry_run=dry_run,
            server_ready_timeout=server_ready_timeout,
    # ... omitted for brevity ...

    return combined_df
```
**EN:** Function `run_combs` provides a reusable helper around the module's main workflow. Key calls such as `server_ctx`, `_get_comb_base_path`, `run_comb`, `all_data.extend`, `pd.DataFrame.from_records` show the concrete execution path.
**CN:** Function `run_combs` 为模块主流程提供可复用的辅助逻辑。 像 `server_ctx`, `_get_comb_base_path`, `run_comb`, `all_data.extend`, `pd.DataFrame.from_records` 这样的关键调用展示了该代码块的具体执行路径。

### Class `SweepServeArgs` (lines 305-507)
```python
class SweepServeArgs:
    serve_cmd: list[str]
    bench_cmd: list[str]
    after_bench_cmd: list[str]
    show_stdout: bool
    server_ready_timeout: int
    serve_params: ParameterSweep
    bench_params: ParameterSweep
    link_vars: list[tuple[str, str]]
    output_dir: Path
    experiment_name: str
    num_runs: int
    dry_run: bool
    resume: bool

    parser_name: ClassVar[str] = "serve"
    parser_help: ClassVar[str] = "Run vLLM server benchmark under multiple settings."

    @classmethod
    def from_cli_args(cls, args: argparse.Namespace):
        serve_cmd = shlex.split(args.serve_cmd)
        bench_cmd = shlex.split(args.bench_cmd)
        after_bench_cmd = (
    # ... omitted for brevity ...
                "to continue the script from its last checkpoint."
            ) from exc
```
**EN:** Class `SweepServeArgs` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `from_cli_args`, `add_cli_args`, `parse_link_vars`, `resolve_experiment_dir`, `run_ctx`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `SweepServeArgs` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `from_cli_args`, `add_cli_args`, `parse_link_vars`, `resolve_experiment_dir`, `run_ctx`，它们共同定义初始化、校验、变换或访问模式。

### Method `SweepServeArgs.from_cli_args` (lines 324-368)
```python
    def from_cli_args(cls, args: argparse.Namespace):
        serve_cmd = shlex.split(args.serve_cmd)
        bench_cmd = shlex.split(args.bench_cmd)
        after_bench_cmd = (
            [] if args.after_bench_cmd is None else shlex.split(args.after_bench_cmd)
        )

        if args.serve_params:
            serve_params = ParameterSweep.read_json(args.serve_params)
        else:
            # i.e.: run serve_cmd without any modification
            serve_params = ParameterSweep.from_records([{}])

        if args.bench_params:
            bench_params = ParameterSweep.read_json(args.bench_params)
        else:
            # i.e.: run bench_cmd without any modification
            bench_params = ParameterSweep.from_records([{}])

    # ... omitted for brevity ...
            server_ready_timeout=args.server_ready_timeout,
        )
```
**EN:** Method `SweepServeArgs.from_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `shlex.split`, `ParameterSweep.read_json`, `ParameterSweep.from_records`, `cls.parse_link_vars`, `datetime.now().strftime` show the concrete execution path.
**CN:** Method `SweepServeArgs.from_cli_args` 负责解析配置、参数或结构化元数据。 像 `shlex.split`, `ParameterSweep.read_json`, `ParameterSweep.from_records`, `cls.parse_link_vars`, `datetime.now().strftime` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepServeArgs.add_cli_args` (lines 371-469)
```python
    def add_cli_args(cls, parser: argparse.ArgumentParser) -> argparse.ArgumentParser:
        parser.add_argument(
            "--serve-cmd",
            type=str,
            required=True,
            help="The command used to run the server: `vllm serve ...`",
        )
        parser.add_argument(
            "--bench-cmd",
            type=str,
            required=True,
            help="The command used to run the benchmark: `vllm bench serve ...`",
        )
        parser.add_argument(
            "--after-bench-cmd",
            type=str,
            default=None,
            help="After a benchmark run is complete, invoke this command instead of "
            "the default `ServerWrapper.clear_cache()`.",
    # ... omitted for brevity ...

        return parser
```
**EN:** Method `SweepServeArgs.add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `parser.add_argument` show the concrete execution path.
**CN:** Method `SweepServeArgs.add_cli_args` 负责解析配置、参数或结构化元数据。 像 `parser.add_argument` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepServeArgs.parse_link_vars` (lines 472-479)
```python
    def parse_link_vars(s: str) -> list[tuple[str, str]]:
        if not s:
            return []
        pairs = []
        for item in s.split(","):
            a, b = item.split("=")
            pairs.append((a.strip(), b.strip()))
        return pairs
```
**EN:** Method `SweepServeArgs.parse_link_vars` parses configuration, arguments, or structured metadata. Key calls such as `s.split`, `item.split`, `pairs.append`, `a.strip`, `b.strip` show the concrete execution path.
**CN:** Method `SweepServeArgs.parse_link_vars` 负责解析配置、参数或结构化元数据。 像 `s.split`, `item.split`, `pairs.append`, `a.strip`, `b.strip` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepServeArgs.resolve_experiment_dir` (lines 481-491)
```python
    def resolve_experiment_dir(self) -> Path:
        experiment_dir = self.output_dir / self.experiment_name

        if self.resume:
            if not experiment_dir.exists():
                raise ValueError(f"Cannot resume from non-existent {experiment_dir=}")
        else:
            if experiment_dir.exists():
                raise ValueError(f"Cannot overwrite existing {experiment_dir=}")

        return experiment_dir
```
**EN:** Method `SweepServeArgs.resolve_experiment_dir` provides a reusable helper around the module's main workflow. Key calls such as `experiment_dir.exists`, `ValueError` show the concrete execution path.
**CN:** Method `SweepServeArgs.resolve_experiment_dir` 为模块主流程提供可复用的辅助逻辑。 像 `experiment_dir.exists`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepServeArgs.run_ctx` (lines 494-507)
```python
    def run_ctx(self, experiment_dir: Path):
        if self.dry_run:
            yield
            print(f"Experiment will be saved at: {experiment_dir}")
            return

        try:
            yield
            print(f"Experiment has been saved at: {experiment_dir}")
        except BaseException as exc:
            raise RuntimeError(
                "The script was terminated early. Use `--resume` "
                "to continue the script from its last checkpoint."
            ) from exc
```
**EN:** Method `SweepServeArgs.run_ctx` provides a reusable helper around the module's main workflow. Key calls such as `print`, `RuntimeError` show the concrete execution path.
**CN:** Method `SweepServeArgs.run_ctx` 为模块主流程提供可复用的辅助逻辑。 像 `print`, `RuntimeError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_main` (lines 510-526)
```python
def run_main(args: SweepServeArgs):
    experiment_dir = args.resolve_experiment_dir()

    with args.run_ctx(experiment_dir):
        return run_combs(
            serve_cmd=args.serve_cmd,
            bench_cmd=args.bench_cmd,
            link_vars=args.link_vars,
            after_bench_cmd=args.after_bench_cmd,
            show_stdout=args.show_stdout,
            server_ready_timeout=args.server_ready_timeout,
            serve_params=args.serve_params,
            bench_params=args.bench_params,
            experiment_dir=experiment_dir,
            num_runs=args.num_runs,
            dry_run=args.dry_run,
        )
```
**EN:** Function `run_main` provides a reusable helper around the module's main workflow. Key calls such as `args.resolve_experiment_dir`, `args.run_ctx`, `run_combs` show the concrete execution path.
**CN:** Function `run_main` 为模块主流程提供可复用的辅助逻辑。 像 `args.resolve_experiment_dir`, `args.run_ctx`, `run_combs` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 529-530)
```python
def main(args: argparse.Namespace):
    run_main(SweepServeArgs.from_cli_args(args))
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `run_main`, `SweepServeArgs.from_cli_args` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `run_main`, `SweepServeArgs.from_cli_args` 这样的关键调用展示了该代码块的具体执行路径。

### Entrypoint guard (lines 533-537)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description=SweepServeArgs.parser_help)
    SweepServeArgs.add_cli_args(parser)

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
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import contextlib`, `import json`, `import shlex`, `from contextlib import contextmanager`, `from dataclasses import dataclass`, `from datetime import datetime`, `from pathlib import Path`, `from typing import ClassVar`
- **vLLM internal / vLLM 内部依赖**: `from vllm.utils.import_utils import PlaceholderModule`, `from .param_sweep import ParameterSweep, ParameterSweepItem`, `from .server import ServerProcess`, `from .utils import sanitize_filename`
