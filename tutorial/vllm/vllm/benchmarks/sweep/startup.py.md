# startup.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/sweep/startup.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements startup support for the `sweep` portion of vLLM. / 为 vLLM 的 `sweep` 子目录实现与 startup 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-19)
```python
import argparse

import json

import shlex

import subprocess

from contextlib import contextmanager

from dataclasses import dataclass

from datetime import datetime

from functools import lru_cache

from pathlib import Path

from typing import ClassVar

from vllm.benchmarks.startup import add_cli_args as add_startup_cli_args

from vllm.utils.argparse_utils import FlexibleArgumentParser

from vllm.utils.import_utils import PlaceholderModule

from .param_sweep import ParameterSweep, ParameterSweepItem

from .utils import sanitize_filename
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Function `_get_supported_startup_keys` (lines 28-40)
```python
def _get_supported_startup_keys() -> set[str]:
    parser = FlexibleArgumentParser(add_help=False)
    add_startup_cli_args(parser)

    supported: set[str] = {"config"}
    for action in parser._actions:
        if action.dest and action.dest is not argparse.SUPPRESS:
            supported.add(action.dest)
        for option in action.option_strings:
            if option.startswith("--"):
                supported.add(option.lstrip("-").replace("-", "_"))

    return supported
```
**EN:** Function `_get_supported_startup_keys` provides a reusable helper around the module's main workflow. Key calls such as `FlexibleArgumentParser`, `add_startup_cli_args`, `supported.add`, `option.startswith`, `option.lstrip('-').replace` show the concrete execution path.
**CN:** Function `_get_supported_startup_keys` 为模块主流程提供可复用的辅助逻辑。 像 `FlexibleArgumentParser`, `add_startup_cli_args`, `supported.add`, `option.startswith`, `option.lstrip('-').replace` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_is_supported_param` (lines 43-48)
```python
def _is_supported_param(param_key: str, supported: set[str]) -> bool:
    if param_key == "_benchmark_name":
        return True
    prefix = param_key.split(".", 1)[0]
    normalized = prefix.replace("-", "_")
    return normalized in supported
```
**EN:** Function `_is_supported_param` provides a reusable helper around the module's main workflow. Key calls such as `param_key.split`, `prefix.replace` show the concrete execution path.
**CN:** Function `_is_supported_param` 为模块主流程提供可复用的辅助逻辑。 像 `param_key.split`, `prefix.replace` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_filter_params` (lines 51-77)
```python
def _filter_params(
    params: ParameterSweep, *, supported: set[str], strict: bool
) -> ParameterSweep:
    filtered = []
    for item in params:
        kept: dict[str, object] = {}
        dropped: list[str] = []
        for key, value in item.items():
            if _is_supported_param(key, supported):
                kept[key] = value
            else:
                dropped.append(key)

        if dropped:
            label = item.get("_benchmark_name") or item.as_text()
            message = (
                "Ignoring unsupported startup params"
                f"{' for ' + str(label) if label else ''}: "
                f"{', '.join(sorted(dropped))}"
            )
            if strict:
                raise ValueError(message)
            print(message)

        filtered.append(ParameterSweepItem.from_record(kept))

    return ParameterSweep(filtered)
```
**EN:** Function `_filter_params` provides a reusable helper around the module's main workflow. Key calls such as `item.items`, `_is_supported_param`, `dropped.append`, `item.get`, `item.as_text` show the concrete execution path.
**CN:** Function `_filter_params` 为模块主流程提供可复用的辅助逻辑。 像 `item.items`, `_is_supported_param`, `dropped.append`, `item.get`, `item.as_text` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_update_run_data` (lines 80-89)
```python
def _update_run_data(
    run_data: dict[str, object],
    serve_overrides: ParameterSweepItem,
    startup_overrides: ParameterSweepItem,
    run_number: int,
) -> dict[str, object]:
    run_data["run_number"] = run_number
    run_data.update(serve_overrides)
    run_data.update(startup_overrides)
    return run_data
```
**EN:** Function `_update_run_data` provides a reusable helper around the module's main workflow. Key calls such as `run_data.update` show the concrete execution path.
**CN:** Function `_update_run_data` 为模块主流程提供可复用的辅助逻辑。 像 `run_data.update` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_strip_arg` (lines 92-105)
```python
def _strip_arg(cmd: list[str], keys: tuple[str, ...]) -> list[str]:
    stripped: list[str] = []
    skip_next = False
    for arg in cmd:
        if skip_next:
            skip_next = False
            continue
        if arg in keys:
            skip_next = True
            continue
        if any(arg.startswith(f"{key}=") for key in keys):
            continue
        stripped.append(arg)
    return stripped
```
**EN:** Function `_strip_arg` parses configuration, arguments, or structured metadata. Key calls such as `any`, `arg.startswith`, `stripped.append` show the concrete execution path.
**CN:** Function `_strip_arg` 负责解析配置、参数或结构化元数据。 像 `any`, `arg.startswith`, `stripped.append` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_apply_output_json` (lines 108-111)
```python
def _apply_output_json(cmd: list[str], output_path: Path) -> list[str]:
    keys = ("--output-json", "--output_json")
    cmd = _strip_arg(cmd, keys)
    return [*cmd, keys[0], str(output_path)]
```
**EN:** Function `_apply_output_json` provides a reusable helper around the module's main workflow. Key calls such as `_strip_arg`, `str` show the concrete execution path.
**CN:** Function `_apply_output_json` 为模块主流程提供可复用的辅助逻辑。 像 `_strip_arg`, `str` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_comb_base_path` (lines 114-125)
```python
def _get_comb_base_path(
    experiment_dir: Path,
    serve_comb: ParameterSweepItem,
    startup_comb: ParameterSweepItem,
) -> Path:
    parts = list[str]()
    if serve_comb:
        parts.extend(("SERVE-", serve_comb.name))
    if startup_comb:
        parts.extend(("STARTUP-", startup_comb.name))

    return experiment_dir / sanitize_filename("-".join(parts))
```
**EN:** Function `_get_comb_base_path` provides a reusable helper around the module's main workflow. Key calls such as `parts.extend`, `sanitize_filename`, `'-'.join` show the concrete execution path.
**CN:** Function `_get_comb_base_path` 为模块主流程提供可复用的辅助逻辑。 像 `parts.extend`, `sanitize_filename`, `'-'.join` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_comb_run_path` (lines 128-131)
```python
def _get_comb_run_path(base_path: Path, run_number: int | None) -> Path:
    if run_number is None:
        return base_path / "summary.json"
    return base_path / f"run={run_number}.json"
```
**EN:** Function `_get_comb_run_path` provides a reusable helper around the module's main workflow.
**CN:** Function `_get_comb_run_path` 为模块主流程提供可复用的辅助逻辑。

### Function `run_benchmark` (lines 134-187)
```python
def run_benchmark(
    startup_cmd: list[str],
    *,
    serve_overrides: ParameterSweepItem,
    startup_overrides: ParameterSweepItem,
    run_number: int,
    output_path: Path,
    show_stdout: bool,
    dry_run: bool,
) -> dict[str, object] | None:
    cmd = serve_overrides.apply_to_cmd(startup_cmd)
    cmd = startup_overrides.apply_to_cmd(cmd)
    cmd = _apply_output_json(cmd, output_path)

    print("[BEGIN BENCHMARK]")
    print(f"Serve overrides: {serve_overrides}")
    print(f"Startup overrides: {startup_overrides}")
    print(f"Run Number: {run_number}")
    print(f"Benchmark command: {cmd}")
    print(f"Output file: {output_path}")

    if output_path.exists():
        print("Found existing results.")
        print("[SKIPPED BENCHMARK]")

    # ... omitted for brevity ...
    print("[END BENCHMARK]")
    return run_data
```
**EN:** Function `run_benchmark` coordinates benchmarking or serving-oriented control flow. Key calls such as `serve_overrides.apply_to_cmd`, `startup_overrides.apply_to_cmd`, `_apply_output_json`, `print`, `output_path.exists` show the concrete execution path.
**CN:** Function `run_benchmark` 负责协调基准测试或服务侧控制流程。 像 `serve_overrides.apply_to_cmd`, `startup_overrides.apply_to_cmd`, `_apply_output_json`, `print`, `output_path.exists` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_comb` (lines 190-222)
```python
def run_comb(
    startup_cmd: list[str],
    *,
    serve_comb: ParameterSweepItem,
    startup_comb: ParameterSweepItem,
    base_path: Path,
    num_runs: int,
    show_stdout: bool,
    dry_run: bool,
) -> list[dict[str, object]] | None:
    comb_data = list[dict[str, object]]()
    for run_number in range(num_runs):
        run_data = run_benchmark(
            startup_cmd,
            serve_overrides=serve_comb,
            startup_overrides=startup_comb,
            run_number=run_number,
            output_path=_get_comb_run_path(base_path, run_number),
            show_stdout=show_stdout,
            dry_run=dry_run,
        )
        if run_data is not None:
            comb_data.append(run_data)

    if dry_run:
    # ... omitted for brevity ...

    return comb_data
```
**EN:** Function `run_comb` provides a reusable helper around the module's main workflow. Key calls such as `range`, `run_benchmark`, `_get_comb_run_path`, `comb_data.append`, `_get_comb_run_path(base_path, run_number=None).open` show the concrete execution path.
**CN:** Function `run_comb` 为模块主流程提供可复用的辅助逻辑。 像 `range`, `run_benchmark`, `_get_comb_run_path`, `comb_data.append`, `_get_comb_run_path(base_path, run_number=None).open` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_combs` (lines 225-256)
```python
def run_combs(
    startup_cmd: list[str],
    *,
    serve_params: ParameterSweep,
    startup_params: ParameterSweep,
    experiment_dir: Path,
    num_runs: int,
    show_stdout: bool,
    dry_run: bool,
) -> "pd.DataFrame | None":
    all_data = list[dict[str, object]]()
    for serve_comb in serve_params:
        for startup_comb in startup_params:
            base_path = _get_comb_base_path(experiment_dir, serve_comb, startup_comb)
            comb_data = run_comb(
                startup_cmd,
                serve_comb=serve_comb,
                startup_comb=startup_comb,
                base_path=base_path,
                num_runs=num_runs,
                show_stdout=show_stdout,
                dry_run=dry_run,
            )
            if comb_data is not None:
                all_data.extend(comb_data)
    # ... omitted for brevity ...
    combined_df.to_csv(experiment_dir / "summary.csv")
    return combined_df
```
**EN:** Function `run_combs` provides a reusable helper around the module's main workflow. Key calls such as `_get_comb_base_path`, `run_comb`, `all_data.extend`, `pd.DataFrame.from_records`, `combined_df.to_csv` show the concrete execution path.
**CN:** Function `run_combs` 为模块主流程提供可复用的辅助逻辑。 像 `_get_comb_base_path`, `run_comb`, `all_data.extend`, `pd.DataFrame.from_records`, `combined_df.to_csv` 这样的关键调用展示了该代码块的具体执行路径。

### Class `SweepStartupArgs` (lines 260-418)
```python
class SweepStartupArgs:
    startup_cmd: list[str]
    serve_params: ParameterSweep
    startup_params: ParameterSweep
    output_dir: Path
    experiment_name: str
    num_runs: int
    show_stdout: bool
    dry_run: bool
    resume: bool

    parser_name: ClassVar[str] = "startup"
    parser_help: ClassVar[str] = (
        "Benchmark vLLM startup time over parameter combinations."
    )

    @classmethod
    def from_cli_args(cls, args: argparse.Namespace):
        startup_cmd = shlex.split(args.startup_cmd)

        if args.serve_params:
            serve_params = ParameterSweep.read_json(args.serve_params)
        else:
    # ... omitted for brevity ...
                "to continue the script from its last checkpoint."
            ) from exc
```
**EN:** Class `SweepStartupArgs` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `from_cli_args`, `add_cli_args`, `resolve_experiment_dir`, `run_ctx`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `SweepStartupArgs` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `from_cli_args`, `add_cli_args`, `resolve_experiment_dir`, `run_ctx`，它们共同定义初始化、校验、变换或访问模式。

### Method `SweepStartupArgs.from_cli_args` (lines 277-317)
```python
    def from_cli_args(cls, args: argparse.Namespace):
        startup_cmd = shlex.split(args.startup_cmd)

        if args.serve_params:
            serve_params = ParameterSweep.read_json(args.serve_params)
        else:
            serve_params = ParameterSweep.from_records([{}])

        if args.startup_params:
            startup_params = ParameterSweep.read_json(args.startup_params)
        else:
            startup_params = ParameterSweep.from_records([{}])

        supported = _get_supported_startup_keys()
        strict_params = args.strict_params
        serve_params = _filter_params(
            serve_params, supported=supported, strict=strict_params
        )
        startup_params = _filter_params(
    # ... omitted for brevity ...
            resume=args.resume,
        )
```
**EN:** Method `SweepStartupArgs.from_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `shlex.split`, `ParameterSweep.read_json`, `ParameterSweep.from_records`, `_get_supported_startup_keys`, `_filter_params` show the concrete execution path.
**CN:** Method `SweepStartupArgs.from_cli_args` 负责解析配置、参数或结构化元数据。 像 `shlex.split`, `ParameterSweep.read_json`, `ParameterSweep.from_records`, `_get_supported_startup_keys`, `_filter_params` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepStartupArgs.add_cli_args` (lines 320-390)
```python
    def add_cli_args(cls, parser: argparse.ArgumentParser) -> argparse.ArgumentParser:
        parser.add_argument(
            "--startup-cmd",
            type=str,
            default="vllm bench startup",
            help="The command used to run the startup benchmark.",
        )

        parser.add_argument(
            "--serve-params",
            type=str,
            default=None,
            help="Path to JSON file containing parameter combinations "
            "for the `vllm serve` command. Only parameters supported by "
            "`vllm bench startup` will be applied.",
        )
        parser.add_argument(
            "--startup-params",
            type=str,
    # ... omitted for brevity ...

        return parser
```
**EN:** Method `SweepStartupArgs.add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `parser.add_argument` show the concrete execution path.
**CN:** Method `SweepStartupArgs.add_cli_args` 负责解析配置、参数或结构化元数据。 像 `parser.add_argument` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepStartupArgs.resolve_experiment_dir` (lines 392-402)
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
**EN:** Method `SweepStartupArgs.resolve_experiment_dir` provides a reusable helper around the module's main workflow. Key calls such as `experiment_dir.exists`, `ValueError` show the concrete execution path.
**CN:** Method `SweepStartupArgs.resolve_experiment_dir` 为模块主流程提供可复用的辅助逻辑。 像 `experiment_dir.exists`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SweepStartupArgs.run_ctx` (lines 405-418)
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
**EN:** Method `SweepStartupArgs.run_ctx` provides a reusable helper around the module's main workflow. Key calls such as `print`, `RuntimeError` show the concrete execution path.
**CN:** Method `SweepStartupArgs.run_ctx` 为模块主流程提供可复用的辅助逻辑。 像 `print`, `RuntimeError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_main` (lines 421-433)
```python
def run_main(args: SweepStartupArgs):
    experiment_dir = args.resolve_experiment_dir()

    with args.run_ctx(experiment_dir):
        return run_combs(
            startup_cmd=args.startup_cmd,
            serve_params=args.serve_params,
            startup_params=args.startup_params,
            experiment_dir=experiment_dir,
            num_runs=args.num_runs,
            show_stdout=args.show_stdout,
            dry_run=args.dry_run,
        )
```
**EN:** Function `run_main` provides a reusable helper around the module's main workflow. Key calls such as `args.resolve_experiment_dir`, `args.run_ctx`, `run_combs` show the concrete execution path.
**CN:** Function `run_main` 为模块主流程提供可复用的辅助逻辑。 像 `args.resolve_experiment_dir`, `args.run_ctx`, `run_combs` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 436-437)
```python
def main(args: argparse.Namespace):
    run_main(SweepStartupArgs.from_cli_args(args))
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `run_main`, `SweepStartupArgs.from_cli_args` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `run_main`, `SweepStartupArgs.from_cli_args` 这样的关键调用展示了该代码块的具体执行路径。

### Entrypoint guard (lines 440-443)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description=SweepStartupArgs.parser_help)
    SweepStartupArgs.add_cli_args(parser)
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
- **Standard library / 标准库**: `import argparse`, `import json`, `import shlex`, `import subprocess`, `from contextlib import contextmanager`, `from dataclasses import dataclass`, `from datetime import datetime`, `from functools import lru_cache`, `from pathlib import Path`, `from typing import ClassVar`
- **vLLM internal / vLLM 内部依赖**: `from vllm.benchmarks.startup import add_cli_args as add_startup_cli_args`, `from vllm.utils.argparse_utils import FlexibleArgumentParser`, `from vllm.utils.import_utils import PlaceholderModule`, `from .param_sweep import ParameterSweep, ParameterSweepItem`, `from .utils import sanitize_filename`
