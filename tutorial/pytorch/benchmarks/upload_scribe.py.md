# upload_scribe.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/upload_scribe.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```python
"""Scribe Uploader for Pytorch Benchmark Data

Currently supports data in pytest-benchmark format but can be extended.

New fields can be added just by modifying the schema in this file, schema
checking is only here to encourage reusing existing fields and avoiding typos.
"""

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 9-15 / 第 9-15 行

```python
import argparse
import json
import os
import subprocess
import time
from collections import defaultdict

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 16-22 / 第 16-22 行

```python
import requests


class ScribeUploader:
    def __init__(self, category):
        self.category = category

```

- **EN:** Important local symbols in this block include ScribeUploader, __init__.
- **CN:** 该代码块中的重要局部符号包括 ScribeUploader、__init__。

### Lines 23-28 / 第 23-28 行

```python
    def format_message(self, field_dict):
        if "time" not in field_dict:
            raise AssertionError("Missing required Scribe field 'time'")
        message = defaultdict(dict)
        for field, value in field_dict.items():
            if field in self.schema["normal"]:
```

- **EN:** Important local symbols in this block include format_message.
- **CN:** 该代码块中的重要局部符号包括 format_message。

### Lines 29-38 / 第 29-38 行

```python
                message["normal"][field] = str(value)
            elif field in self.schema["int"]:
                message["int"][field] = int(value)
            elif field in self.schema["float"]:
                message["float"][field] = float(value)
            else:
                raise ValueError(
                    f"Field {field} is not currently used, be intentional about adding new fields"
                )
        return message
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 39-45 / 第 39-45 行

```python

    def _upload_intern(self, messages):
        for m in messages:
            json_str = json.dumps(m)
            cmd = ["scribe_cat", self.category, json_str]
            subprocess.run(cmd)

```

- **EN:** Important local symbols in this block include _upload_intern.
- **CN:** 该代码块中的重要局部符号包括 _upload_intern。

### Lines 46-57 / 第 46-57 行

```python
    def upload(self, messages):
        if os.environ.get("SCRIBE_INTERN"):
            return self._upload_intern(messages)
        access_token = os.environ.get("SCRIBE_GRAPHQL_ACCESS_TOKEN")
        if not access_token:
            raise ValueError("Can't find access token from environment variable")
        url = "https://graph.facebook.com/scribe_logs"  # @lint-ignore
        r = requests.post(
            url,
            data={
                "access_token": access_token,
                "logs": json.dumps(
```

- **EN:** Important local symbols in this block include upload.
- **CN:** 该代码块中的重要局部符号包括 upload。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 58-64 / 第 58-64 行

```python
                    [
                        {
                            "category": self.category,
                            "message": json.dumps(message),
                            "line_escape": False,
                        }
                        for message in messages
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 65-71 / 第 65-71 行

```python
                    ]
                ),
            },
        )
        print(r.text)
        r.raise_for_status()

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 72-83 / 第 72-83 行

```python

class PytorchBenchmarkUploader(ScribeUploader):
    def __init__(self):
        super().__init__("perfpipe_pytorch_benchmarks")
        self.schema = {
            "int": [
                "time",
                "rounds",
            ],
            "normal": [
                "benchmark_group",
                "benchmark_name",
```

- **EN:** Important local symbols in this block include PytorchBenchmarkUploader, __init__.
- **CN:** 该代码块中的重要局部符号包括 PytorchBenchmarkUploader、__init__。

### Lines 84-95 / 第 84-95 行

```python
                "benchmark_executor",
                "benchmark_fuser",
                "benchmark_class",
                "benchmark_time",
                "pytorch_commit_id",
                "pytorch_branch",
                "pytorch_commit_time",
                "pytorch_version",
                "pytorch_git_dirty",
                "machine_kernel",
                "machine_processor",
                "machine_hostname",
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 96-107 / 第 96-107 行

```python
                "circle_build_num",
                "circle_project_reponame",
            ],
            "float": [
                "stddev",
                "min",
                "median",
                "max",
                "mean",
            ],
        }

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 108-113 / 第 108-113 行

```python
    def post_pytest_benchmarks(self, pytest_json):
        machine_info = pytest_json["machine_info"]
        commit_info = pytest_json["commit_info"]
        upload_time = int(time.time())
        messages = []
        for b in pytest_json["benchmarks"]:
```

- **EN:** Important local symbols in this block include post_pytest_benchmarks.
- **CN:** 该代码块中的重要局部符号包括 post_pytest_benchmarks。

### Lines 114-125 / 第 114-125 行

```python
            test = b["name"].split("[")[0]
            net_name = b["params"]["net_name"]
            benchmark_name = f"{test}[{net_name}]"
            executor = b["params"]["executor"]
            fuser = b["params"]["fuser"]
            m = self.format_message(
                {
                    "time": upload_time,
                    "benchmark_group": b["group"],
                    "benchmark_name": benchmark_name,
                    "benchmark_executor": executor,
                    "benchmark_fuser": fuser,
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 126-137 / 第 126-137 行

```python
                    "benchmark_class": b["fullname"],
                    "benchmark_time": pytest_json["datetime"],
                    "pytorch_commit_id": commit_info["id"],
                    "pytorch_branch": commit_info["branch"],
                    "pytorch_commit_time": commit_info["time"],
                    "pytorch_version": None,
                    "pytorch_git_dirty": commit_info["dirty"],
                    "machine_kernel": machine_info["release"],
                    "machine_processor": machine_info["processor"],
                    "machine_hostname": machine_info["node"],
                    "circle_build_num": os.environ.get("CIRCLE_BUILD_NUM"),
                    "circle_project_reponame": os.environ.get(
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 138-149 / 第 138-149 行

```python
                        "CIRCLE_PROJECT_REPONAME"
                    ),
                    "stddev": b["stats"]["stddev"],
                    "rounds": b["stats"]["rounds"],
                    "min": b["stats"]["min"],
                    "median": b["stats"]["median"],
                    "max": b["stats"]["max"],
                    "mean": b["stats"]["mean"],
                }
            )
            messages.append(m)
        self.upload(messages)
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 150-161 / 第 150-161 行

```python


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument(
        "--pytest-bench-json",
        "--pytest_bench_json",
        type=argparse.FileType("r"),
        help="Upload json data formatted by pytest-benchmark module",
    )
    args = parser.parse_args()
    if args.pytest_bench_json:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 162-164 / 第 162-164 行

```python
        benchmark_uploader = PytorchBenchmarkUploader()
        json_data = json.load(args.pytest_bench_json)
        benchmark_uploader.post_pytest_benchmarks(json_data)
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: ScribeUploader, PytorchBenchmarkUploader, __init__, format_message, _upload_intern, upload, post_pytest_benchmarks** — 代表性符号：ScribeUploader、PytorchBenchmarkUploader、__init__、format_message、_upload_intern、upload、post_pytest_benchmarks

## Dependencies / 依赖关系

- `argparse`
- `json`
- `os`
- `subprocess`
- `time`
- `collections`
- `requests`
