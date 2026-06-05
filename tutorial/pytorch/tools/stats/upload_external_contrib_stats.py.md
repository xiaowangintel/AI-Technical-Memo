# upload_external_contrib_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/upload_external_contrib_stats.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
from __future__ import annotations

import argparse
import datetime
import json
import os
import time
import urllib.parse
from typing import Any, cast, TYPE_CHECKING
from urllib.error import HTTPError
from urllib.request import Request, urlopen
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, argparse, datetime, and 7 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、argparse、datetime 等共 10 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 13-18
```python
from tools.stats.upload_stats_lib import upload_to_s3


if TYPE_CHECKING:
    from collections.abc import Callable
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.upload_stats_lib; Python standard-library modules such as collections.abc. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.upload_stats_lib；Python 标准库模块，如 collections.abc。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 19-28
```python

FILTER_OUT_USERS = {
    "pytorchmergebot",
    "facebook-github-bot",
    "pytorch-bot[bot]",
    "pytorchbot",
    "pytorchupdatebot",
    "dependabot[bot]",
}
```
- **EN**: Configuration constants such as FILTER_OUT_USERS centralize defaults so later functions share the same policy knobs.
- **CN**: FILTER_OUT_USERS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 29-38
```python

def _fetch_url(
    url: str,
    headers: dict[str, str],
    data: dict[str, Any] | None = None,
    method: str | None = None,
    reader: Callable[[Any], Any] = lambda x: x.read(),
) -> Any:
    token = os.environ.get("GITHUB_TOKEN")
    if token is not None and url.startswith("https://api.github.com/"):
```
- **EN**: This chunk defines `_fetch_url`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `_fetch_url`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 39-47
```python
        headers["Authorization"] = f"token {token}"
    data_ = json.dumps(data).encode() if data is not None else None
    try:
        with urlopen(Request(url, headers=headers, data=data_, method=method)) as conn:
            return reader(conn)
    except HTTPError as err:
        print(err.reason)
        print(err.headers)
        if err.code == 403 and all(
```
- **EN**: This chunk continues `_fetch_url` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_fetch_url`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 48-54
```python
            key in err.headers for key in ["X-RateLimit-Limit", "X-RateLimit-Used"]
        ):
            print(
                f"Rate limit exceeded: {err.headers['X-RateLimit-Used']}/{err.headers['X-RateLimit-Limit']}"
            )
        raise
```
- **EN**: This chunk continues `_fetch_url` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `_fetch_url`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 55-62
```python

def fetch_json(
    url: str,
    params: dict[str, Any] | None = None,
    data: dict[str, Any] | None = None,
) -> list[dict[str, Any]]:
    headers = {"Accept": "application/vnd.github.v3+json"}
    if params is not None and len(params) > 0:
```
- **EN**: This chunk defines `fetch_json`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `fetch_json`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 63-70
```python
        url += "?" + "&".join(
            f"{name}={urllib.parse.quote(str(val))}" for name, val in params.items()
        )
    return cast(
        list[dict[str, Any]],
        _fetch_url(url, headers=headers, data=data, reader=json.load),
    )
```
- **EN**: This chunk continues `fetch_json` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `fetch_json`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 71-77
```python

def get_external_pr_data(
    start_date: datetime.date, end_date: datetime.date, period_length: int = 1
) -> list[dict[str, Any]]:
    pr_info = []
    period_begin_date = start_date
```
- **EN**: This chunk defines `get_external_pr_data`, which implements a focused step inside the ci statistics pipeline.
- **CN**: 这一段定义了 `get_external_pr_data`，其作用是实现CI 统计流水线中的一个关键步骤。

### Lines 78-84
```python
    pr_count = 0
    users: set[str] = set()
    while period_begin_date < end_date:
        period_end_date = period_begin_date + datetime.timedelta(days=period_length - 1)
        page = 1
        responses: list[dict[str, Any]] = []
        while len(responses) > 0 or page == 1:
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 85-96
```python
            response = cast(
                dict[str, Any],
                fetch_json(
                    "https://api.github.com/search/issues",  # @lint-ignore
                    params={
                        "q": f'repo:pytorch/pytorch is:pr is:closed \
                            label:"open source" label:Merged -label:Reverted closed:{period_begin_date}..{period_end_date}',
                        "per_page": "100",
                        "page": str(page),
                    },
                ),
            )
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 97-104
```python
            items = response["items"]
            for item in items:
                u = item["user"]["login"]
                if u not in FILTER_OUT_USERS:
                    pr_count += 1
                    users.add(u)
            page += 1
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 105-115
```python
        pr_info.append(
            {
                "date": str(period_begin_date),
                "pr_count": pr_count,
                "user_count": len(users),
                "users": list(users),
            }
        )
        period_begin_date = period_end_date + datetime.timedelta(days=1)
    return pr_info
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 116-127
```python

if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Upload external contribution stats to s3"
    )
    parser.add_argument(
        "--startDate",
        type=datetime.date.fromisoformat,
        required=True,
        help="the first date to upload data for in any valid ISO 8601 format format (eg. YYYY-MM-DD).",
    )
    parser.add_argument(
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 128-139
```python
        "--length",
        type=int,
        required=False,
        help="the number of days to upload data for. Default is 1.",
        default=1,
    )
    parser.add_argument(
        "--period-length",
        type=int,
        required=False,
        help="the number of days to group data for. Default is 1.",
        default=1,
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 140-150
```python
    )
    args = parser.parse_args()
    for i in range(args.length):
        tries = 0
        startdate = args.startDate + datetime.timedelta(days=i)
        data = get_external_pr_data(
            startdate,
            startdate + datetime.timedelta(days=args.period_length),
            period_length=args.period_length,
        )
        for pr_info in data:
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 151-162
```python
            # sometimes users does not get added, so we check it got uploaded
            if "users" not in pr_info:
                raise AssertionError("'users' key not found in pr_info")
            if not isinstance(pr_info["users"], list):
                raise AssertionError(
                    f"'users' must be a list, got {type(pr_info['users'])}"
                )
        print(f"uploading the following data: \n {data}")
        upload_to_s3(
            bucket_name="torchci-contribution-data",
            key=f"external_contribution_counts/{str(startdate)}",
            docs=data,
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 163-165
```python
        )
        # get around rate limiting
        time.sleep(10)
```
- **EN**: This chunk continues `get_external_pr_data` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_external_pr_data`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **CI statistics**
  - EN: This file belongs to the ci statistics layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于CI 统计层，应结合同一子目录中的相邻脚本一起理解。
- **Lint enforcement**
  - EN: The code applies policy checks to source files and reports actionable diagnostics.
  - CN: 代码对源码施加策略检查，并产出可执行的诊断信息。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.upload_stats_lib`
- **Python standard library / Python 标准库**: `__future__`, `argparse`, `datetime`, `json`, `os`, `time`, `urllib.parse`, `typing`, `urllib.error`, `urllib.request`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `FILTER_OUT_USERS`, `_fetch_url`, `fetch_json`, `get_external_pr_data`
