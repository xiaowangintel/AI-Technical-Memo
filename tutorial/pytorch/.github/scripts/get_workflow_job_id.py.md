# get_workflow_job_id.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/get_workflow_job_id.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行

````python
# Helper to get the id of the currently running job in a GitHub Actions
# workflow. GitHub does not provide this information to workflow runs, so we
# need to figure it out based on what they *do* provide.
from __future__ import annotations

import argparse
import json
import operator
import os
import re
import sys
import time
import urllib
import urllib.parse
````

- EN: This block imports dependencies such as `__future__`, `argparse`, `json`, `operator`; parses command-line arguments for script entry points; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 15-28 / 第 15-28 行

````python
from typing import Any, TYPE_CHECKING
from urllib.request import Request, urlopen


if TYPE_CHECKING:
    from collections.abc import Callable


def parse_json_and_links(conn: Any) -> tuple[Any, dict[str, dict[str, str]]]:
    links = {}
    # Extract links which GH uses for pagination
    # see https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Link
    if "Link" in conn.headers:
        for elem in re.split(", *<", conn.headers["Link"]):
````

- EN: This block imports dependencies such as `typing`, `urllib.request`, `collections.abc`; defines callable units such as `parse_json_and_links`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 29-42 / 第 29-42 行

````python
            try:
                url, params_ = elem.split(";", 1)
            except ValueError:
                continue
            url = urllib.parse.unquote(url.strip("<> "))
            qparams = urllib.parse.parse_qs(params_.strip(), separator=";")
            params = {
                k: v[0].strip('"')
                for k, v in qparams.items()
                if type(v) is list and len(v) > 0
            }
            params["url"] = url
            if "rel" in params:
                links[params["rel"]] = params
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 44-57 / 第 44-57 行

````python
    return json.load(conn), links


def fetch_url(
    url: str,
    *,
    headers: dict[str, str] | None = None,
    reader: Callable[[Any], Any] = lambda x: x.read(),
    retries: int | None = 3,
    backoff_timeout: float = 0.5,
) -> Any:
    if headers is None:
        headers = {}
    try:
````

- EN: This block defines callable units such as `fetch_url`; branches on runtime conditions or configuration choices; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 58-71 / 第 58-71 行

````python
        with urlopen(Request(url, headers=headers)) as conn:
            return reader(conn)
    except urllib.error.HTTPError as err:
        if isinstance(retries, (int, float)) and retries > 0:
            time.sleep(backoff_timeout)
            return fetch_url(
                url,
                headers=headers,
                reader=reader,
                retries=retries - 1,
                backoff_timeout=backoff_timeout,
            )
        exception_message = (
            "Is github alright?",
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 72-85 / 第 72-85 行

````python
            f"Received status code '{err.code}' when attempting to retrieve {url}:\n",
            f"{err.reason}\n\nheaders={err.headers}",
        )
        raise RuntimeError(exception_message) from err


def parse_args() -> Any:
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "workflow_run_id", help="The id of the workflow run, should be GITHUB_RUN_ID"
    )
    parser.add_argument(
        "runner_name",
        help="The name of the runner to retrieve the job id, should be RUNNER_NAME",
````

- EN: This block defines callable units such as `parse_args`; parses command-line arguments for script entry points; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 86-99 / 第 86-99 行

````python
    )

    return parser.parse_args()


def fetch_jobs(url: str, headers: dict[str, str]) -> list[dict[str, str]]:
    response, links = fetch_url(url, headers=headers, reader=parse_json_and_links)
    jobs = response["jobs"]
    if type(jobs) is not list:
        raise AssertionError(f"Expected jobs to be a list, got {type(jobs).__name__}")
    while "next" in links:
        response, links = fetch_url(
            links["next"]["url"], headers=headers, reader=parse_json_and_links
        )
````

- EN: This block defines callable units such as `fetch_jobs`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败；序列化或解析结构化数据文件。

### Lines 100-113 / 第 100-113 行

````python
        jobs.extend(response["jobs"])

    return jobs


# Our strategy is to retrieve the parent workflow run, then filter its jobs on
# RUNNER_NAME to figure out which job we're currently running.
#
# Why RUNNER_NAME? Because it's the only thing that uniquely identifies a job within a workflow.
# GITHUB_JOB doesn't work, as it corresponds to the job yaml id
# (https://bit.ly/37e78oI), which has two problems:
# 1. It's not present in the workflow job JSON object, so we can't use it as a filter.
# 2. It isn't unique; for matrix jobs the job yaml id is the same for all jobs in the matrix.
#
````

- EN: This block iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 114-127 / 第 114-127 行

````python
# RUNNER_NAME on the other hand is unique across the pool of runners. Also,
# since only one job can be scheduled on a runner at a time, we know that
# looking for RUNNER_NAME will uniquely identify the job we're currently
# running.


def find_job_id_name(args: Any) -> tuple[str, str]:
    # From https://docs.github.com/en/actions/learn-github-actions/environment-variables
    PYTORCH_REPO = os.environ.get("GITHUB_REPOSITORY", "pytorch/pytorch")
    PYTORCH_GITHUB_API = f"https://api.github.com/repos/{PYTORCH_REPO}"
    GITHUB_TOKEN = os.environ["GITHUB_TOKEN"]
    REQUEST_HEADERS = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": "token " + GITHUB_TOKEN,
````

- EN: This block defines callable units such as `find_job_id_name`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 128-139 / 第 128-139 行

````python
    }

    url = f"{PYTORCH_GITHUB_API}/actions/runs/{args.workflow_run_id}/jobs?per_page=100"
    jobs = fetch_jobs(url, REQUEST_HEADERS)

    # Sort the jobs list by start time, in descending order. We want to get the most
    # recently scheduled job on the runner.
    jobs.sort(key=operator.itemgetter("started_at"), reverse=True)

    for job in jobs:
        if job["runner_name"] == args.runner_name:
            return (job["id"], job["name"])
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 141-150 / 第 141-150 行

````python
    raise RuntimeError(f"Can't find job id for runner {args.runner_name}")


def set_output(name: str, val: Any) -> None:
    print(f"Setting output {name}={val}")
    if os.getenv("GITHUB_OUTPUT"):
        with open(str(os.getenv("GITHUB_OUTPUT")), "a") as env:
            print(f"{name}={val}", file=env)
    else:
        print(f"::set-output name={name}::{val}")
````

- EN: This block defines callable units such as `set_output`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 153-163 / 第 153-163 行

````python
def main() -> None:
    args = parse_args()
    try:
        # Get both the job ID and job name because we have already spent a request
        # here to get the job info
        job_id, job_name = find_job_id_name(args)
        set_output("job-id", job_id)
        set_output("job-name", job_name)
    except Exception as e:
        print(repr(e), file=sys.stderr)
        print(f"workflow-{args.workflow_run_id}")
````

- EN: This block defines callable units such as `main`.
- CN: 该代码块定义可调用函数来执行具体自动化步骤。

### Lines 166-167 / 第 166-167 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。
- EN: CLI entry points — the module exposes command-line arguments so developers or CI jobs can drive automation predictably.
  CN: 命令行入口——该模块暴露命令行参数，使开发者或 CI 作业能够以可预测方式驱动自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `__future__`, `argparse`, `json`, `operator`, `os`, `re`, `sys`, `time`, ...
- Internal imports / 内部导入: none
- External imports / 外部导入: none
- Classes / 类: none
- Functions / 函数: `parse_json_and_links`, `fetch_url`, `parse_args`, `fetch_jobs`, `find_job_id_name`, `set_output`, `main`
- Test entry points / 测试入口: none
