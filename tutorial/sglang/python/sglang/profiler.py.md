# profiler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/profiler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Run live profiling. / 该文件围绕上述模块职责展开实现，组织核心流程、辅助逻辑以及与相邻组件的协作。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6: Module-level supporting statements
```python
"""
Run live profiling.

Usage:
python3 -m sglang.profiler
"""
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 8-8: Module-level supporting statements
```python
import argparse
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: Module-level supporting statements
```python
import json
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-10: Module-level supporting statements
```python
import os
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Module-level supporting statements
```python
import time
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python
from argparse import ArgumentParser
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
from pathlib import Path
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-14: Module-level supporting statements
```python
from typing import List, Optional
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-15: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-16: Module-level supporting statements
```python
import requests
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 17-17: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 18-18: Module-level supporting statements
```python
PROFILER_DIR = os.getenv("SGLANG_TORCH_PROFILER_DIR", "/tmp")
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 19-20: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 21-68: run profile function
```python
def run_profile(
    url: Optional[str],
    num_steps: int,
    activities: List[str],
    output_dir: Optional[str] = None,
    profile_by_stage: bool = False,
    merge_profiles: bool = False,
    profile_prefix: Optional[str] = None,
    start_step: Optional[int] = None,
) -> str:
    if output_dir is None:
        output_dir = PROFILER_DIR

    output_dir = Path(os.path.abspath(os.path.normpath(output_dir))) / str(time.time())
    output_dir.mkdir(exist_ok=True, parents=True)

    print(f"Dump profiling traces to {output_dir}")
    print(
        f"Waiting for {num_steps} steps and the trace to be flushed.... ({profile_by_stage=})"
    )

    # Dump server args.
    file_path = Path(output_dir) / "server_args.json"
    if not file_path.exists():
        response = requests.get(url + "/server_info")
        response.raise_for_status()
        server_args_data = response.json()
        with open(file_path, "w") as file:
            file.write(json.dumps(server_args_data))

    # Start profiler. The API replies when all steps are processed
    # and files are generated.
    json_data = {
        "output_dir": str(output_dir),
        "num_steps": str(num_steps),
        "activities": activities,
        "profile_by_stage": profile_by_stage,
        "merge_profiles": merge_profiles,
        "profile_prefix": profile_prefix,
    }
    if start_step is not None:
        json_data["start_step"] = str(start_step)

    response = requests.post(url=url + "/start_profile", json=json_data)
    response.raise_for_status()

    trace_link = str(output_dir)
    return trace_link
```
**EN:** This block uses `run_profile` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `run_profile` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 69-70: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 71-130: Module-level supporting statements (part 1/2)
```python
if __name__ == "__main__":
    parser = ArgumentParser(description="Benchmark the online serving throughput.")
    parser.add_argument(
        "--url",
        type=str,
        default="http://localhost:30000",
        help="Server or API base url if not using http host and port.",
    )
    parser.add_argument(
        "--output-dir",
        type=str,
        default=None,
        help="Profile directory to dump profile traces.",
    )
    parser.add_argument(
        "--num-steps",
        type=int,
        default=5,
        help="The number of forward steps to profile.",
    )
    parser.add_argument(
        "--profile-by-stage",
        action=argparse.BooleanOptionalAction,
        type=bool,
        default=False,
        help="Whether to profile prefill and decode separately",
    )
    parser.add_argument(
        "--profile-prefix",
        type=str,
        help="The prefix of this profiler file.",
    )
    parser.add_argument(
        "--cpu",
        action=argparse.BooleanOptionalAction,
        type=bool,
        default=True,
        help="Whether to profile CPU activity",
    )
    parser.add_argument(
        "--gpu",
        action=argparse.BooleanOptionalAction,
        type=bool,
        default=True,
        help="Whether to profile GPU activity",
    )
    parser.add_argument(
        "--mem",
        action=argparse.BooleanOptionalAction,
        type=bool,
        default=False,
        help="Whether to profile memory usage (https://pytorch.org/memory_viz)",
    )
    parser.add_argument(
        "--rpd",
        action=argparse.BooleanOptionalAction,
        type=bool,
        default=False,
        help="Whether to use ROCM rpd profiler (https://github.com/ROCm/rocmProfileData)",
    )
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 131-158: Module-level supporting statements (part 2/2)
```python
    parser.add_argument(
        "--merge-profiles",
        action=argparse.BooleanOptionalAction,
        type=bool,
        default=False,
        help="Whether to merge profiles from all ranks into a single trace file",
    )

    args = parser.parse_args()
    activities = []
    if args.cpu:
        activities.append("CPU")
    if args.gpu:
        activities.append("GPU")
    if args.mem:
        activities.append("MEM")
    if args.rpd:
        activities.append("RPD")

    run_profile(
        url=args.url,
        num_steps=args.num_steps,
        activities=activities,
        output_dir=args.output_dir,
        profile_by_stage=args.profile_by_stage,
        profile_prefix=args.profile_prefix,
        merge_profiles=args.merge_profiles,
    )
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

## Key Concepts / 关键概念
- Profiling / 性能剖析
- External API integration / 外部 API 集成

## Dependencies / 依赖关系
### Internal / 内部
- None / 无
### External / 外部
- `requests`
- `argparse` (stdlib)
- `json` (stdlib)
- `os` (stdlib)
- `pathlib` (stdlib)
- `time` (stdlib)
- `typing` (stdlib)
