# diffusion_skill_env.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/.claude/skills/sglang-diffusion-benchmark-profile/scripts/diffusion_skill_env.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `get_repo_root`, `get_assets_dir`, and `get_output_dir`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于多模态生成模块。它围绕 `get_repo_root`、`get_assets_dir` 和 `get_output_dir` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

import argparse
import csv
import os
import subprocess
from pathlib import Path
```
**EN:** This block establishes the module context and imports `__future__`, `argparse`, `csv`, `os`, `subprocess`, and `pathlib`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`argparse`、`csv`、`os`、`subprocess` 和 `pathlib`。这些依赖为后续实现提供所需符号。

### Lines 9-12: supporting statements / 辅助语句
```python
OUTPUT_DIR_NAMES = {
    "benchmarks": Path("outputs/diffusion_benchmarks"),
    "profiles": Path("outputs/diffusion_profiles"),
}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `OUTPUT_DIR_NAMES`. The code collaborates with `Path`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `OUTPUT_DIR_NAMES` 等名称。 代码会与 `Path` 协同工作。

### Lines 15-18: `get_repo_root` implementation / `get_repo_root` 实现
```python
def get_repo_root() -> Path:
    import sglang

    return Path(sglang.__file__).resolve().parents[2]
```
**EN:** This block defines function `get_repo_root`. It retrieves repo root. Key calls include `Path.resolve`, and `Path`.
**CN:** 该代码块定义了函数 `get_repo_root`。 它用于获取repo root。 关键调用包括 `Path.resolve` 和 `Path`。

### Lines 21-23: `get_assets_dir` implementation / `get_assets_dir` 实现
```python
def get_assets_dir(repo_root: Path | None = None) -> Path:
    root = repo_root or get_repo_root()
    return root / "inputs" / "diffusion_benchmark" / "figs"
```
**EN:** This block defines function `get_assets_dir`. It retrieves assets dir. Key calls include `get_repo_root`. Parameters such as `repo_root` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_assets_dir`。 它用于获取assets dir。 关键调用包括 `get_repo_root`。 本段逻辑主要由 `repo_root` 等参数驱动。

### Lines 26-30: `get_output_dir` implementation / `get_output_dir` 实现
```python
def get_output_dir(name: str, repo_root: Path | None = None) -> Path:
    if name not in OUTPUT_DIR_NAMES:
        raise KeyError(f"Unknown output dir name: {name}")
    root = repo_root or get_repo_root()
    return root / OUTPUT_DIR_NAMES[name]
```
**EN:** This block defines function `get_output_dir`. It retrieves output dir. Key calls include `KeyError`, and `get_repo_root`. The implementation branches on conditions. Parameters such as `name`, and `repo_root` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_output_dir`。 它用于获取output dir。 关键调用包括 `KeyError` 和 `get_repo_root`。 实现中包含条件分支。 本段逻辑主要由 `name` 和 `repo_root` 等参数驱动。

### Lines 33-35: `ensure_dir` implementation / `ensure_dir` 实现
```python
def ensure_dir(path: Path) -> Path:
    path.mkdir(parents=True, exist_ok=True)
    return path
```
**EN:** This block defines function `ensure_dir`. It handles ensure dir logic. Key calls include `path.mkdir`. Parameters such as `path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `ensure_dir`。 它用于处理 ensure dir 相关逻辑。 关键调用包括 `path.mkdir`。 本段逻辑主要由 `path` 等参数驱动。

### Lines 38-43: `check_write_access` implementation / `check_write_access` 实现
```python
def check_write_access(repo_root: Path | None = None) -> Path:
    root = repo_root or get_repo_root()
    probe_dir = ensure_dir(root / ".cache" / "diffusion_skill_write_test")
    probe_file = probe_dir / "probe.txt"
    probe_file.write_text("ok", encoding="utf-8")
    return probe_file
```
**EN:** This block defines function `check_write_access`. It checks write access. Key calls include `ensure_dir`, `probe_file.write_text`, and `get_repo_root`. Parameters such as `repo_root` drive the behavior in this section.
**CN:** 该代码块定义了函数 `check_write_access`。 它用于检查write access。 关键调用包括 `ensure_dir`、`probe_file.write_text` 和 `get_repo_root`。 本段逻辑主要由 `repo_root` 等参数驱动。

### Lines 46-59: `_run_nvidia_smi` implementation / `_run_nvidia_smi` 实现
```python
def _run_nvidia_smi(query: str) -> list[list[str]]:
    command = [
        "nvidia-smi",
        f"--query-{query}",
        "--format=csv,noheader,nounits",
    ]
    result = subprocess.run(command, check=True, capture_output=True, text=True)
    rows: list[list[str]] = []
    for raw_line in result.stdout.splitlines():
        line = raw_line.strip()
        if not line:
            continue
        rows.append([field.strip() for field in csv.reader([line]).__next__()])
    return rows
```
**EN:** This block defines function `_run_nvidia_smi`. It runs nvidia smi. Key calls include `subprocess.run`, `result.stdout.splitlines`, `raw_line.strip`, `rows.append`, and `field.strip`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `query` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_run_nvidia_smi`。 它用于运行nvidia smi。 关键调用包括 `subprocess.run`、`result.stdout.splitlines`、`raw_line.strip`、`rows.append` 和 `field.strip`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `query` 等参数驱动。

### Lines 62-75: `get_gpu_inventory` implementation / `get_gpu_inventory` 实现
```python
def get_gpu_inventory() -> list[dict[str, int | str]]:
    rows = _run_nvidia_smi("gpu=index,uuid,memory.used,memory.total,utilization.gpu")
    inventory = []
    for index, uuid, memory_used, memory_total, utilization_gpu in rows:
        inventory.append(
            {
                "index": int(index),
                "uuid": uuid,
                "memory_used_mib": int(memory_used),
                "memory_total_mib": int(memory_total),
                "utilization_gpu_pct": int(utilization_gpu),
            }
        )
    return inventory
```
**EN:** This block defines function `get_gpu_inventory`. It retrieves gpu inventory. Key calls include `_run_nvidia_smi`, `inventory.append`, and `int`. The implementation iterates over collections or steps.
**CN:** 该代码块定义了函数 `get_gpu_inventory`。 它用于获取gpu inventory。 关键调用包括 `_run_nvidia_smi`、`inventory.append` 和 `int`。 实现中会遍历集合或步骤。

### Lines 78-80: `get_busy_gpu_uuids` implementation / `get_busy_gpu_uuids` 实现
```python
def get_busy_gpu_uuids() -> set[str]:
    rows = _run_nvidia_smi("compute-apps=gpu_uuid,pid,process_name,used_gpu_memory")
    return {gpu_uuid for gpu_uuid, *_ in rows}
```
**EN:** This block defines function `get_busy_gpu_uuids`. It retrieves busy gpu uuids. Key calls include `_run_nvidia_smi`.
**CN:** 该代码块定义了函数 `get_busy_gpu_uuids`。 它用于获取busy gpu uuids。 关键调用包括 `_run_nvidia_smi`。

### Lines 83-103: `pick_idle_gpus` implementation / `pick_idle_gpus` 实现
```python
def pick_idle_gpus(
    required_gpus: int,
    max_memory_used_mib: int = 32,
    max_utilization_gpu_pct: int = 5,
) -> list[int]:
    inventory = get_gpu_inventory()
    busy_uuids = get_busy_gpu_uuids()

    idle = [
        int(gpu["index"])
        for gpu in inventory
        if gpu["uuid"] not in busy_uuids
        and int(gpu["memory_used_mib"]) <= max_memory_used_mib
        and int(gpu["utilization_gpu_pct"]) <= max_utilization_gpu_pct
    ]
    if len(idle) < required_gpus:
        raise RuntimeError(
            "Not enough idle GPUs. "
            f"required={required_gpus}, idle={idle}, inventory={inventory}, busy={sorted(busy_uuids)}"
        )
    return idle[:required_gpus]
```
**EN:** This block defines function `pick_idle_gpus`. It handles pick idle gpus logic. Key calls include `get_gpu_inventory`, `get_busy_gpu_uuids`, `int`, `len`, and `RuntimeError`. The implementation branches on conditions. Parameters such as `required_gpus`, `max_memory_used_mib`, and `max_utilization_gpu_pct` drive the behavior in this section.
**CN:** 该代码块定义了函数 `pick_idle_gpus`。 它用于处理 pick idle gpus 相关逻辑。 关键调用包括 `get_gpu_inventory`、`get_busy_gpu_uuids`、`int`、`len` 和 `RuntimeError`。 实现中包含条件分支。 本段逻辑主要由 `required_gpus`、`max_memory_used_mib` 和 `max_utilization_gpu_pct` 等参数驱动。

### Lines 106-112: `configure_runtime_env` implementation / `configure_runtime_env` 实现
```python
def configure_runtime_env(required_gpus: int = 1) -> str | None:
    os.environ.setdefault("FLASHINFER_DISABLE_VERSION_CHECK", "1")
    if os.environ.get("CUDA_VISIBLE_DEVICES"):
        return None
    selected = ",".join(str(index) for index in pick_idle_gpus(required_gpus))
    os.environ["CUDA_VISIBLE_DEVICES"] = selected
    return selected
```
**EN:** This block defines function `configure_runtime_env`. It handles configure runtime env logic. Key calls include `os.environ.setdefault`, `os.environ.get`, `join`, `str`, and `pick_idle_gpus`. The implementation branches on conditions. Parameters such as `required_gpus` drive the behavior in this section.
**CN:** 该代码块定义了函数 `configure_runtime_env`。 它用于处理 configure runtime env 相关逻辑。 关键调用包括 `os.environ.setdefault`、`os.environ.get`、`join`、`str` 和 `pick_idle_gpus`。 实现中包含条件分支。 本段逻辑主要由 `required_gpus` 等参数驱动。

### Lines 115-170: `main` implementation / `main` 实现
```python
def main() -> None:
    parser = argparse.ArgumentParser(
        description="Resolve SGLang diffusion skill paths and idle GPUs."
    )
    parser.add_argument(
        "command",
        choices=[
            "print-root",
            "print-assets-dir",
            "print-output-dir",
            "print-idle-gpus",
            "check-write-access",
        ],
    )
    parser.add_argument(
        "--kind",
        choices=sorted(OUTPUT_DIR_NAMES),
        help="Output directory kind for print-output-dir.",
    )
    parser.add_argument(
        "--count",
        type=int,
        default=1,
        help="Number of idle GPUs to print.",
    )
    parser.add_argument(
        "--mkdir",
        action="store_true",
        help="Create the requested directory before printing it.",
    )
    args = parser.parse_args()

    if args.command == "print-root":
        print(get_repo_root())
        return
    if args.command == "print-assets-dir":
        path = get_assets_dir()
        if args.mkdir:
            ensure_dir(path)
        print(path)
        return
    if args.command == "print-output-dir":
        if not args.kind:
            raise SystemExit("--kind is required for print-output-dir")
        path = get_output_dir(args.kind)
        if args.mkdir:
            ensure_dir(path)
        print(path)
        return
    if args.command == "print-idle-gpus":
        print(",".join(str(index) for index in pick_idle_gpus(args.count)))
        return
    if args.command == "check-write-access":
        print(check_write_access())
        return
    raise SystemExit(f"Unhandled command: {args.command}")
```
**EN:** This block defines function `main`. It handles main logic. Key calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `SystemExit`, and `print`. The implementation branches on conditions.
**CN:** 该代码块定义了函数 `main`。 它用于处理 main 相关逻辑。 关键调用包括 `argparse.ArgumentParser`、`parser.add_argument`、`parser.parse_args`、`SystemExit` 和 `print`。 实现中包含条件分支。

### Lines 173-174: supporting statements / 辅助语句
```python
if __name__ == "__main__":
    main()
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `main`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `main` 协同工作。

## Key Concepts / 关键概念
- `get_repo_root`: Top-level function that retrieves repo root. / 顶层函数，用于获取repo root。
- `get_assets_dir`: Top-level function that retrieves assets dir. / 顶层函数，用于获取assets dir。
- `get_output_dir`: Top-level function that retrieves output dir. / 顶层函数，用于获取output dir。
- `ensure_dir`: Top-level function that handles ensure dir logic. / 顶层函数，用于处理 ensure dir 相关逻辑。
- `check_write_access`: Top-level function that checks write access. / 顶层函数，用于检查write access。
- `_run_nvidia_smi`: Top-level function that runs nvidia smi. / 顶层函数，用于运行nvidia smi。
- `get_gpu_inventory`: Top-level function that retrieves gpu inventory. / 顶层函数，用于获取gpu inventory。
- `get_busy_gpu_uuids`: Top-level function that retrieves busy gpu uuids. / 顶层函数，用于获取busy gpu uuids。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `argparse`, `csv`, `os`, `subprocess`, `pathlib`
- **Internal modules / 内部模块**: `sglang`

- **Total lines / 总行数**: 174
