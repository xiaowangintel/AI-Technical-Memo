# bench_ir_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/ir/bench_ir_ops.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, centered around `_REPO_ROOT`, `BenchConfig`, `_pkg_version`, `_METADATA_LABELS`. / 实现与基准测试编排相关的逻辑，核心符号包括 `_REPO_ROOT`, `BenchConfig`, `_pkg_version`, `_METADATA_LABELS`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-13)
```python
"""
Generic benchmark harness for vLLM IR ops.

Usage:
    python benchmarks/kernels/ir/bench_ir_ops.py
    python benchmarks/kernels/ir/bench_ir_ops.py --ops rms_norm
    python benchmarks/kernels/ir/bench_ir_ops.py --ops rms_norm,silu_mul
    python benchmarks/kernels/ir/bench_ir_ops.py --no-cuda-graph
    python benchmarks/kernels/ir/bench_ir_ops.py --ops rms_norm --save-path ./results/
"""
```
**EN:** The module docstring introduces Generic benchmark harness for vLLM IR ops. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Generic benchmark harness for vLLM IR ops 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 15-24)
```python
import argparse
import contextlib
import csv
import dataclasses
import datetime
import math
import os
import subprocess
import sys
import tempfile
```
**EN:** This block gathers standard-library helpers such as `argparse`, `contextlib`, `csv`, `dataclasses`, `datetime`, `math`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `contextlib`, `csv`, `dataclasses`, `datetime`, `math`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 27-43)
```python
_REPO_ROOT = os.path.abspath(os.path.join(os.path.dirname(__file__), "../../.."))
if _REPO_ROOT not in sys.path:
    sys.path.insert(0, _REPO_ROOT)

# Suppress noisy C++ warnings from vllm kernel registration (written to fd 2
# directly by the dynamic linker, so Python-level sys.stderr redirect won't
# catch them).
_saved_fd = os.dup(2)
try:
    with open(os.devnull, "w") as _devnull:
        os.dup2(_devnull.fileno(), 2)
        import torch

        import vllm.kernels  # noqa: E402, F401
finally:
    os.dup2(_saved_fd, 2)
    os.close(_saved_fd)
```
**EN:** This top-level block prepares shared state such as `_REPO_ROOT`, `_saved_fd`. It uses `os.path.abspath`, `os.path.join`, `os.path.dirname`, `sys.path.insert`, `os.dup`, `open` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `_REPO_ROOT`, `_saved_fd`。它借助 `os.path.abspath`, `os.path.join`, `os.path.dirname`, `sys.path.insert`, `os.dup`, `open` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Additional imports (lines 45-45)
```python
from tqdm import tqdm  # noqa: E402
```
**EN:** This block gathers third-party packages such as `tqdm`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `tqdm`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 47-47)
```python
from benchmarks.kernels.ir.shapes import SHAPE_CONFIGS  # noqa: E402  # isort: skip
```
**EN:** This block gathers project-local modules such as `benchmarks.kernels.ir.shapes`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `benchmarks.kernels.ir.shapes`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 48-48)
```python
from vllm.ir.op import IrOp  # noqa: E402
```
**EN:** This block gathers project-local modules such as `vllm.ir.op`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.ir.op`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 49-49)
```python
from vllm.platforms import current_platform  # noqa: E402
```
**EN:** This block gathers project-local modules such as `vllm.platforms`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.platforms`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 50-50)
```python
from vllm.triton_utils import triton  # noqa: E402
```
**EN:** This block gathers project-local modules such as `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Class `BenchConfig` (lines 54-57)
```python
class BenchConfig:
    use_cuda_graph: bool = True
    warmup: int = 25
    rep: int = 100
```
**EN:** Class `BenchConfig` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as its methods.
**CN:** 类 `BenchConfig` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 its methods 等方法。

### Function `_pkg_version` (lines 60-65)
```python
def _pkg_version(name: str) -> str:
    from importlib.metadata import PackageNotFoundError, version

    with contextlib.suppress(PackageNotFoundError):
        return version(name)
    return "not installed"
```
**EN:** `_pkg_version` implements a helper used by `bench_ir_ops.py`. It mainly works with `name` and relies on `contextlib.suppress`, `version` plus context management to move data through this part of the benchmark pipeline.
**CN:** `_pkg_version` 负责实现 `bench_ir_ops.py` 使用的辅助逻辑。 它主要处理 `name`，并结合 `contextlib.suppress`, `version` 以及 上下文管理 来完成这一段基准测试流程。

### Top-level setup (lines 68-81)
```python
_METADATA_LABELS = {
    "timestamp": "Timestamp",
    "git_commit": "Git commit",
    "vllm": "vLLM",
    "pytorch": "PyTorch",
    "cuda_runtime": "CUDA runtime",
    "triton": "Triton",
    "cutlass": "CUTLASS",
    "helion": "Helion",
    "device": "Device",
    "bench_mode": "Bench mode",
    "warmup": "Warmup",
    "rep": "Repetitions",
}
```
**EN:** This top-level block prepares shared state such as `_METADATA_LABELS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `_METADATA_LABELS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `collect_env_metadata` (lines 84-117)
```python
def collect_env_metadata(cfg: BenchConfig) -> dict[str, str]:
    from vllm.collect_env import get_env_info

    env = get_env_info()

    git_sha = "unknown"
    with contextlib.suppress(subprocess.CalledProcessError, FileNotFoundError):
        git_sha = (
            subprocess.check_output(
                ["git", "rev-parse", "--short", "HEAD"], stderr=subprocess.DEVNULL
            )
            .decode()
            .strip()
        )

    device_name = current_platform.get_device_name()

    warmup_note = " ms" if not cfg.use_cuda_graph else " ms (ignored)"
    rep_note = " replays" if cfg.use_cuda_graph else " ms"

    return {
        "timestamp": datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "git_commit": git_sha,
        "vllm": str(env.vllm_version),
        "pytorch": str(env.torch_version),
        "cuda_runtime": str(env.cuda_runtime_version),
        "triton": triton.__version__,
        "cutlass": _pkg_version("nvidia-cutlass-dsl"),
        "helion": _pkg_version("helion"),
        "device": device_name,
        "bench_mode": "cuda_graph" if cfg.use_cuda_graph else "eager",
        "warmup": f"{cfg.warmup}{warmup_note}",
        "rep": f"{cfg.rep}{rep_note}",
    }
```
**EN:** `collect_env_metadata` implements a helper used by `bench_ir_ops.py`. It mainly works with `cfg` and relies on `get_env_info`, `contextlib.suppress`, `subprocess.check_output.decode.strip`, `subprocess.check_output.decode`, `subprocess.check_output`, `current_platform.get_device_name` plus context management to move data through this part of the benchmark pipeline.
**CN:** `collect_env_metadata` 负责实现 `bench_ir_ops.py` 使用的辅助逻辑。 它主要处理 `cfg`，并结合 `get_env_info`, `contextlib.suppress`, `subprocess.check_output.decode.strip`, `subprocess.check_output.decode`, `subprocess.check_output`, `current_platform.get_device_name` 以及 上下文管理 来完成这一段基准测试流程。

### Function `print_metadata` (lines 120-124)
```python
def print_metadata(metadata: dict[str, str]):
    print("=" * 60)
    for key, val in metadata.items():
        print(f"{_METADATA_LABELS.get(key, key) + ':':<16}{val}")
    print("=" * 60)
```
**EN:** `print_metadata` formats results for display or export. It mainly works with `metadata` and relies on `print`, `metadata.items`, `_METADATA_LABELS.get` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `print_metadata` 负责格式化结果以便展示或导出。 它主要处理 `metadata`，并结合 `print`, `metadata.items`, `_METADATA_LABELS.get` 以及 循环迭代 来完成这一段基准测试流程。

### Function `_clone_args` (lines 127-128)
```python
def _clone_args(args: tuple) -> tuple:
    return tuple(a.clone() if isinstance(a, torch.Tensor) else a for a in args)
```
**EN:** `_clone_args` implements a helper used by `bench_ir_ops.py`. It mainly works with `args` and relies on `tuple`, `isinstance`, `a.clone` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_clone_args` 负责实现 `bench_ir_ops.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `tuple`, `isinstance`, `a.clone` 以及 结果返回 来完成这一段基准测试流程。

### Function `_bench_one` (lines 135-145)
```python
def _bench_one(fn, args, cfg: BenchConfig) -> float:
    bench_args = _clone_args(args)
    bench_fn = lambda: fn(*bench_args)

    if cfg.use_cuda_graph:
        ms = triton.testing.do_bench_cudagraph(bench_fn, rep=cfg.rep, quantiles=[0.5])
    else:
        ms = triton.testing.do_bench(
            bench_fn, warmup=cfg.warmup, rep=cfg.rep, quantiles=[0.5]
        )
    return ms * 1000
```
**EN:** `_bench_one` implements a helper used by `bench_ir_ops.py`. It mainly works with `fn`, `args`, `cfg` and relies on `_clone_args`, `fn`, `triton.testing.do_bench_cudagraph`, `triton.testing.do_bench` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_bench_one` 负责实现 `bench_ir_ops.py` 使用的辅助逻辑。 它主要处理 `fn`, `args`, `cfg`，并结合 `_clone_args`, `fn`, `triton.testing.do_bench_cudagraph`, `triton.testing.do_bench` 以及 条件分支 来完成这一段基准测试流程。

### Function `collect_timings` (lines 151-178)
```python
def collect_timings(
    op: IrOp, shape_configs: list[dict], cfg: BenchConfig
) -> tuple[list[str], list[str], dict[str, dict[str, float]]]:
    def fmt(v) -> str:
        return str(v).split(".")[-1] if isinstance(v, torch.dtype) else str(v)

    case_names = [
        "_".join(f"{k}={fmt(v)}" for k, v in kwargs.items()) for kwargs in shape_configs
    ]
    providers = [n for n, impl in op.impls.items() if impl.supported]

    results: dict[str, dict[str, float]] = {c: {} for c in case_names}
    for provider in providers:
        impl = op.impls[provider]
        desc = f"{op.name} / {provider}"
        for case_name, kwargs in tqdm(
            zip(case_names, shape_configs),
            desc=desc,
            total=len(case_names),
            unit=" cases",
        ):
            args = op.generate_inputs(**kwargs)
            if impl.supports_args(*args):
                results[case_name][provider] = _bench_one(impl.impl_fn, args, cfg)
            else:
                results[case_name][provider] = float("nan")

    return case_names, providers, results
```
**EN:** `collect_timings` implements a helper used by `bench_ir_ops.py`. It mainly works with `op`, `shape_configs`, `cfg` and relies on `isinstance`, `str.split`, `str`, `join`, `fmt`, `kwargs.items` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `collect_timings` 负责实现 `bench_ir_ops.py` 使用的辅助逻辑。 它主要处理 `op`, `shape_configs`, `cfg`，并结合 `isinstance`, `str.split`, `str`, `join`, `fmt`, `kwargs.items` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `analyze_results` (lines 181-252)
```python
def analyze_results(
    op_name: str,
    case_names: list[str],
    providers: list[str],
    results: dict[str, dict[str, float]],
) -> tuple[list[dict[str, str]], list[dict[str, str]], list[str]]:
    native_col = "native"
    non_native = [p for p in providers if p != native_col]

    header_cols = ["case"]
    for p in providers:
        header_cols.append(f"{p} (us)")
    for p in non_native:
        header_cols.append(f"{p} speedup")

    detail_rows: list[dict[str, str]] = []
    speedup_data: dict[str, list[tuple[float, str]]] = {p: [] for p in non_native}

    for case_name in case_names:
        timings = results[case_name]
        row: dict[str, str] = {"case": case_name}

        for p in providers:
            val = timings.get(p, float("nan"))
            row[f"{p} (us)"] = f"{val:.2f}" if not math.isnan(val) else "n/a"

        native_us = timings.get(native_col, float("nan"))
        for p in non_native:
            p_us = timings.get(p, float("nan"))
            if not math.isnan(native_us) and not math.isnan(p_us) and p_us > 0:
                speedup = native_us / p_us
                row[f"{p} speedup"] = f"{speedup:.2f}x"
                speedup_data[p].append((speedup, case_name))
            else:
    # ... omitted for brevity ...
                "worst_case": worst_case,
                "wins": str(wins),
                "losses": str(losses),
                "total": str(total),
            }
        )

    return detail_rows, summary_rows, header_cols
```
**EN:** `analyze_results` implements a helper used by `bench_ir_ops.py`. It mainly works with `op_name`, `case_names`, `providers`, `results` and relies on `header_cols.append`, `timings.get`, `float`, `math.isnan`, `speedup_data.append`, `detail_rows.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `analyze_results` 负责实现 `bench_ir_ops.py` 使用的辅助逻辑。 它主要处理 `op_name`, `case_names`, `providers`, `results`，并结合 `header_cols.append`, `timings.get`, `float`, `math.isnan`, `speedup_data.append`, `detail_rows.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `write_csv` (lines 255-259)
```python
def write_csv(path: str, rows: list[dict[str, str]], fieldnames: list[str]):
    with open(path, "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(rows)
```
**EN:** `write_csv` writes benchmark outputs to persistent storage. It mainly works with `path`, `rows`, `fieldnames` and relies on `open`, `csv.DictWriter`, `writer.writeheader`, `writer.writerows` plus context management to move data through this part of the benchmark pipeline.
**CN:** `write_csv` 负责将基准结果写入持久化存储。 它主要处理 `path`, `rows`, `fieldnames`，并结合 `open`, `csv.DictWriter`, `writer.writeheader`, `writer.writerows` 以及 上下文管理 来完成这一段基准测试流程。

### Function `save_results` (lines 262-285)
```python
def save_results(
    save_dir: str,
    op_name: str,
    detail_rows: list[dict[str, str]],
    header_cols: list[str],
    all_summary_rows: list[dict[str, str]],
    metadata: dict[str, str],
):
    write_csv(
        os.path.join(save_dir, f"{op_name}_detail.csv"),
        detail_rows,
        header_cols,
    )
    if all_summary_rows:
        write_csv(
            os.path.join(save_dir, "summary.csv"),
            all_summary_rows,
            list(all_summary_rows[0].keys()),
        )
    write_csv(
        os.path.join(save_dir, "metadata.csv"),
        [metadata],
        list(metadata.keys()),
    )
```
**EN:** `save_results` writes benchmark outputs to persistent storage. It mainly works with `save_dir`, `op_name`, `detail_rows`, `header_cols`, `all_summary_rows`, ... and relies on `write_csv`, `os.path.join`, `list`, `all_summary_rows.keys`, `metadata.keys` plus branching to move data through this part of the benchmark pipeline.
**CN:** `save_results` 负责将基准结果写入持久化存储。 它主要处理 `save_dir`, `op_name`, `detail_rows`, `header_cols`, `all_summary_rows`, ...，并结合 `write_csv`, `os.path.join`, `list`, `all_summary_rows.keys`, `metadata.keys` 以及 条件分支 来完成这一段基准测试流程。

### Function `parse_args` (lines 288-320)
```python
def parse_args():
    parser = argparse.ArgumentParser(description="Benchmark vLLM IR ops")
    parser.add_argument(
        "--ops",
        type=str,
        default=None,
        help="Comma-separated list of op names to benchmark (substring match)",
    )
    parser.add_argument(
        "--no-cuda-graph",
        action="store_true",
        help="Disable CUDA graph; use do_bench with L2 cache flushing instead",
    )
    parser.add_argument(
        "--warmup",
        type=int,
        default=25,
        help="Warmup time in ms (do_bench) or ignored with CUDA graph (default: 25)",
    )
    parser.add_argument(
        "--rep",
        type=int,
        default=100,
        help="Repetition time in ms (do_bench) or number of graph replays "
        "(do_bench_cudagraph) (default: 100)",
    )
    parser.add_argument(
        "--save-path",
        type=str,
        default=None,
        help="Directory to save results (default: auto-created temp dir)",
    )
    return parser.parse_args()
```
**EN:** `parse_args` parses external input into structured benchmark settings. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args` plus value production to move data through this part of the benchmark pipeline.
**CN:** `parse_args` 负责把外部输入解析为结构化的基准配置。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args` 以及 结果返回 来完成这一段基准测试流程。

### Function `main` (lines 323-374)
```python
def main():
    args = parse_args()
    cfg = BenchConfig(
        use_cuda_graph=not args.no_cuda_graph,
        warmup=args.warmup,
        rep=args.rep,
    )

    torch.set_default_device(current_platform.device_type)

    metadata = collect_env_metadata(cfg)
    print_metadata(metadata)

    timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    save_dir = args.save_path or os.path.join(
        tempfile.gettempdir(), f"vllm_ir_bench_{timestamp}"
    )
    os.makedirs(save_dir, exist_ok=True)

    op_filters = [f.strip() for f in args.ops.split(",")] if args.ops else None
    all_summary_rows: list[dict[str, str]] = []

    for op in IrOp.registry.values():
        if op_filters and not any(f in op.name for f in op_filters):
            continue
        if not op.has_input_generator:
            print(f"Skipping op '{op.name}': no input generator registered")
            continue
        if op.name not in SHAPE_CONFIGS:
            raise RuntimeError(
                f"No benchmark shape config for op '{op.name}'. "
                f"Add it to benchmarks/kernels/ir/shapes.py"
            )

    # ... omitted for brevity ...
            op.name,
            detail_rows,
            header_cols,
            all_summary_rows,
            metadata,
        )

    print(f"\nResults saved to: {save_dir}")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `parse_args`, `BenchConfig`, `torch.set_default_device`, `collect_env_metadata`, `print_metadata`, `datetime.datetime.now.strftime` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `parse_args`, `BenchConfig`, `torch.set_default_device`, `collect_env_metadata`, `print_metadata`, `datetime.datetime.now.strftime` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 377-378)
```python
if __name__ == "__main__":
    main()
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `contextlib`, `csv`, `dataclasses`, `datetime`, `math`.
- **CN:** 标准库依赖：`argparse`, `contextlib`, `csv`, `dataclasses`, `datetime`, `math`。
- **EN:** Third-party packages: `tqdm`.
- **CN:** 第三方依赖：`tqdm`。
- **EN:** Internal modules: `benchmarks.kernels.ir.shapes`, `vllm.ir.op`, `vllm.platforms`, `vllm.triton_utils`.
- **CN:** 内部模块：`benchmarks.kernels.ir.shapes`, `vllm.ir.op`, `vllm.platforms`, `vllm.triton_utils`。
