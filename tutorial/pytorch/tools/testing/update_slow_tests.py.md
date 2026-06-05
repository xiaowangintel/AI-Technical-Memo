# update_slow_tests.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/update_slow_tests.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides shared testing infrastructure, test-selection helpers, or execution utilities for PyTorch CI.
- **Purpose (CN)**: 提供 PyTorch CI 使用的共享测试基础设施、测试选择辅助逻辑或执行工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```python
import json
import os
import subprocess
import time
from pathlib import Path
from typing import Any, cast

import requests
from clickhouse import query_clickhouse  # type: ignore[import]
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as json, os, subprocess, and 3 more; external packages such as requests, clickhouse. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 json、os、subprocess 等共 6 项；外部依赖包，如 requests、clickhouse。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 11-28
```python

REPO_ROOT = Path(__file__).resolve().parent.parent.parent
QUERY = """
WITH most_recent_strict_commits AS (
    SELECT
        distinct push.head_commit.'id' as sha
    FROM
        -- not bothering with final
        default.push
    WHERE
        push.ref = 'refs/heads/viable/strict'
        AND push.repository.'full_name' = 'pytorch/pytorch'
    ORDER BY
        push.head_commit.'timestamp' desc
    LIMIT
        3
), workflows AS (
    SELECT
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Configuration constants such as REPO_ROOT, QUERY centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 REPO_ROOT、QUERY 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 29-46
```python
        id
    FROM
        default.workflow_run w final
    WHERE
        w.id in (select id from materialized_views.workflow_run_by_head_sha
            where head_sha in (select sha from most_recent_strict_commits)
        )
        and w.name != 'periodic'
),
job AS (
    SELECT
        j.id as id
    FROM
        default.workflow_job j final
    WHERE
        j.run_id in (select id from workflows)
        and j.name NOT LIKE '%asan%'
),
```
- **EN**: This chunk contributes a small but necessary piece of the testing infrastructure implementation, wiring local data and helper logic together.
- **CN**: 这一段补上了测试基础设施实现中的一小块但必要的基础逻辑，用于衔接局部数据与辅助实现。

### Lines 47-64
```python
duration_per_job AS (
    SELECT
        test_run.classname as classname,
        test_run.name as name,
        job.id as id,
        SUM(test_run.time) as time
    FROM
        default.test_run_s3 test_run
        INNER JOIN job ON test_run.job_id = job.id
    WHERE
        /* cpp tests do not populate `file` for some reason. */
        /* Exclude them as we don't include them in our slow test infra */
        test_run.file != ''
        /* do some more filtering to cut down on the test_run size */
        AND empty(test_run.skipped)
        AND empty(test_run.failure)
        AND empty(test_run.error)
        and test_run.job_id in (select id from job)
```
- **EN**: Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 65-82
```python
    GROUP BY
        test_run.classname,
        test_run.name,
        job.id
)
SELECT
    CONCAT(
        name,
        ' (__main__.',
        classname,
        ')'
    ) as test_name,
    AVG(time) as avg_duration_sec
FROM
    duration_per_job
GROUP BY
    CONCAT(
        name,
```
- **EN**: This chunk contributes a small but necessary piece of the testing infrastructure implementation, wiring local data and helper logic together.
- **CN**: 这一段补上了测试基础设施实现中的一小块但必要的基础逻辑，用于衔接局部数据与辅助实现。

### Lines 83-92
```python
        ' (__main__.',
        classname,
        ')'
    )
HAVING
    AVG(time) > 60.0
ORDER BY
    test_name
"""
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。

### Lines 93-105
```python

UPDATEBOT_TOKEN = os.environ["UPDATEBOT_TOKEN"]
PYTORCHBOT_TOKEN = os.environ["PYTORCHBOT_TOKEN"]


def git_api(
    url: str, params: dict[str, Any], type: str = "get", token: str = UPDATEBOT_TOKEN
) -> Any:
    headers = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": f"token {token}",
    }
    if type == "post":
```
- **EN**: This chunk defines `git_api`, which implements a focused step inside the testing infrastructure pipeline. Configuration constants such as UPDATEBOT_TOKEN, PYTORCHBOT_TOKEN centralize defaults so later functions share the same policy knobs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `git_api`，其作用是实现测试基础设施流水线中的一个关键步骤。 UPDATEBOT_TOKEN、PYTORCHBOT_TOKEN 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 106-123
```python
        return requests.post(
            f"https://api.github.com{url}",
            data=json.dumps(params),
            headers=headers,
        ).json()
    elif type == "patch":
        return requests.patch(
            f"https://api.github.com{url}",
            data=json.dumps(params),
            headers=headers,
        ).json()
    else:
        return requests.get(
            f"https://api.github.com{url}",
            params=params,
            headers=headers,
        ).json()
```
- **EN**: This chunk continues `git_api` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `git_api`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 124-140
```python

def make_pr(source_repo: str, params: dict[str, Any]) -> int:
    response = git_api(f"/repos/{source_repo}/pulls", params, type="post")
    print(f"made pr {response['html_url']}")
    return cast(int, response["number"])


def approve_pr(source_repo: str, pr_number: int) -> None:
    params = {"event": "APPROVE"}
    # use pytorchbot to approve the pr
    git_api(
        f"/repos/{source_repo}/pulls/{pr_number}/reviews",
        params,
        type="post",
        token=PYTORCHBOT_TOKEN,
    )
```
- **EN**: This chunk defines `approve_pr`, which implements a focused step inside the testing infrastructure pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `approve_pr`，其作用是实现测试基础设施流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 141-151
```python

def make_comment(source_repo: str, pr_number: int, msg: str) -> None:
    params = {"body": msg}
    # comment with pytorchbot because pytorchmergebot gets ignored
    git_api(
        f"/repos/{source_repo}/issues/{pr_number}/comments",
        params,
        type="post",
        token=PYTORCHBOT_TOKEN,
    )
```
- **EN**: This chunk defines `make_comment`, which implements a focused step inside the testing infrastructure pipeline.
- **CN**: 这一段定义了 `make_comment`，其作用是实现测试基础设施流水线中的一个关键步骤。

### Lines 152-160
```python

def add_labels(source_repo: str, pr_number: int, labels: list[str]) -> None:
    params = {"labels": labels}
    git_api(
        f"/repos/{source_repo}/issues/{pr_number}/labels",
        params,
        type="post",
    )
```
- **EN**: This chunk defines `add_labels`, which implements a focused step inside the testing infrastructure pipeline.
- **CN**: 这一段定义了 `add_labels`，其作用是实现测试基础设施流水线中的一个关键步骤。

### Lines 161-178
```python

def search_for_open_pr(source_repo: str, search_string: str) -> tuple[int, str] | None:
    params = {
        "q": f"is:pr is:open in:title author:pytorchupdatebot repo:{source_repo} {search_string}",
        "sort": "created",
    }
    response = git_api("/search/issues", params)
    if response["total_count"] != 0:
        # pr does exist
        pr_num = response["items"][0]["number"]
        link = response["items"][0]["html_url"]
        response = git_api(f"/repos/{source_repo}/pulls/{pr_num}", {})
        branch_name = response["head"]["ref"]
        print(
            f"pr does exist, number is {pr_num}, branch name is {branch_name}, link is {link}"
        )
        return pr_num, branch_name
    return None
```
- **EN**: This chunk defines `search_for_open_pr`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `search_for_open_pr`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 179-187
```python


if __name__ == "__main__":
    results = query_clickhouse(QUERY, {})
    slow_tests = {row["test_name"]: row["avg_duration_sec"] for row in results}

    with open(REPO_ROOT / "test" / "slow_tests.json", "w") as f:
        json.dump(slow_tests, f, indent=2)
```
- **EN**: This chunk continues `search_for_open_pr` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `search_for_open_pr`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 188-201
```python
    branch_name = f"update_slow_tests_{int(time.time())}"
    pr_num = None

    open_pr = search_for_open_pr("pytorch/pytorch", "Update slow tests")
    if open_pr is not None:
        pr_num, branch_name = open_pr

    subprocess.run(["git", "checkout", "-b", branch_name], cwd=REPO_ROOT)
    subprocess.run(["git", "add", "test/slow_tests.json"], cwd=REPO_ROOT)
    subprocess.run(["git", "commit", "-m", "Update slow tests"], cwd=REPO_ROOT)
    subprocess.run(
        f"git push --set-upstream origin {branch_name} -f".split(), cwd=REPO_ROOT
    )
```
- **EN**: This chunk continues `search_for_open_pr` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `search_for_open_pr`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 202-215
```python
    params = {
        "title": "Update slow tests",
        "head": branch_name,
        "base": "main",
        "body": "This PR is auto-generated weekly by [this action](https://github.com/pytorch/pytorch/blob/main/"
        + ".github/workflows/weekly.yml).\nUpdate the list of slow tests.",
    }
    if pr_num is None:
        # no existing pr, so make a new one and approve it
        pr_num = make_pr("pytorch/pytorch", params)
        time.sleep(5)
        add_labels("pytorch/pytorch", pr_num, ["ciflow/slow", "ci-no-td"])
        approve_pr("pytorch/pytorch", pr_num)
    make_comment("pytorch/pytorch", pr_num, "@pytorchbot merge")
```
- **EN**: This chunk continues `search_for_open_pr` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `search_for_open_pr`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **REPO_ROOT**
  - EN: `REPO_ROOT` is one of the main local symbols exposed or implemented here.
  - CN: `REPO_ROOT` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `json`, `os`, `subprocess`, `time`, `pathlib`, `typing`
- **External packages / 外部依赖包**: `requests`, `clickhouse`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `QUERY`, `UPDATEBOT_TOKEN`, `PYTORCHBOT_TOKEN`, `git_api`, `make_pr`, `approve_pr`, `make_comment`, `add_labels`, `search_for_open_pr`
