# create_alerts.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/alerts/create_alerts.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements alerting logic that turns CI or repository signals into actionable notifications.
- **Purpose (CN)**: 实现告警逻辑，把 CI 或仓库信号转换为可执行的通知。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
#!/usr/bin/env python3

from __future__ import annotations

import argparse
import json
import os
import re
from collections import defaultdict
from difflib import SequenceMatcher
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, argparse, json, and 5 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、argparse、json 等共 8 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 13-24
```python
import requests
from setuptools import distutils  # type: ignore[import,attr-defined]


ALL_SKIPPED_THRESHOLD = 100
SIMILARITY_THRESHOLD = 0.75
FAILURE_CHAIN_THRESHOLD = 2
MAX_CONCURRENT_ALERTS = 1
FAILED_JOB_PATTERN = (
    r"^- \[(.*)\]\(.*\) failed consecutively starting with commit \[.*\]\(.*\)$"
)
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as requests, setuptools. Configuration constants such as ALL_SKIPPED_THRESHOLD, SIMILARITY_THRESHOLD, FAILURE_CHAIN_THRESHOLD, and 2 more centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 requests、setuptools。 ALL_SKIPPED_THRESHOLD、SIMILARITY_THRESHOLD、FAILURE_CHAIN_THRESHOLD 等共 5 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 25-42
```python
PENDING = "pending"
NEUTRAL = "neutral"
SKIPPED = "skipped"
SUCCESS = "success"
FAILURE = "failure"
CANCELED = "canceled"

ISSUES_WITH_LABEL_QUERY = """
query ($owner: String!, $name: String!, $labels: [String!]) {
  repository(owner: $owner, name: $name, followRenames: false) {
    issues(last: 10, labels: $labels, states: [OPEN]) {
      nodes {
        id
        title
        closed
        number
        body
        createdAt
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Configuration constants such as PENDING, NEUTRAL, SKIPPED, and 4 more centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 PENDING、NEUTRAL、SKIPPED 等共 7 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 43-54
```python
        comments(first: 100) {
          nodes {
            bodyText
            databaseId
          }
        }
      }
    }
  }
}
"""
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。

### Lines 55-67
```python
NUM_ISSUES_QUERY = """
query ($query: String!) {
  search(type: ISSUE, query: $query) {
    issueCount
  }
}
"""

DISABLED_ALERTS = [
    "rerun_disabled_tests",
    "unstable",
]
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Configuration constants such as NUM_ISSUES_QUERY, DISABLED_ALERTS centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 NUM_ISSUES_QUERY、DISABLED_ALERTS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 68-77
```python

class JobStatus:
    job_name: str = ""
    jobs: list[Any] = []
    current_status: Any = None
    job_statuses: list[Any] = []
    filtered_statuses: list[Any] = []
    failure_chain: list[Any] = []
    flaky_jobs: list[Any] = []
```
- **EN**: It introduces classes such as JobStatus, which package state and behavior for this tooling task. This chunk continues `JobStatus` and expands its internal control flow or data movement.
- **CN**: 它引入了 JobStatus 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `JobStatus`，进一步展开其内部控制流或数据流转。

### Lines 78-88
```python
    def __init__(self, job_name: str, job_statuses: list[Any]) -> None:
        self.job_name = job_name
        self.job_statuses = job_statuses

        self.filtered_statuses = list(
            filter(lambda j: not is_job_skipped(j), job_statuses)
        )
        self.current_status = self.get_current_status()
        self.failure_chain = self.get_most_recent_failure_chain()
        self.flaky_jobs = self.get_flaky_jobs()
```
- **EN**: This chunk defines `__init__`, which implements a focused step inside the alert automation pipeline.
- **CN**: 这一段定义了 `__init__`，其作用是实现告警自动化流水线中的一个关键步骤。

### Lines 89-98
```python
    def get_current_status(self) -> Any:
        """
        When getting the current status, we want the latest status which is not pending,
        be it success or failure
        """
        for status in self.filtered_statuses:
            if status["conclusion"] != PENDING:
                return status
        return None
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_current_status`, which implements a focused step inside the alert automation pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_current_status`，其作用是实现告警自动化流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 99-107
```python
    def get_unique_failures(self, jobs: list[Any]) -> dict[str, list[Any]]:
        """
        Returns list of jobs grouped by failureCaptures from the input list
        """
        failures = defaultdict(list)
        for job in jobs:
            if job["conclusion"] == "failure":
                found_similar_failure = False
                if "failureCaptures" not in job:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_unique_failures`, which implements a focused step inside the alert automation pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_unique_failures`，其作用是实现告警自动化流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 108-116
```python
                    failures["unclassified"] = [job]
                    continue

                # This is now a list returned by HUD API, not a string
                failureCaptures = " ".join(job["failureCaptures"])

                for failure in failures:
                    seq = SequenceMatcher(None, failureCaptures, failure)
                    if seq.ratio() > SIMILARITY_THRESHOLD:
```
- **EN**: This chunk continues `get_unique_failures` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_unique_failures`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 117-126
```python
                        failures[failure].append(job)
                        found_similar_failure = True
                        break
                if not found_similar_failure:
                    failures[failureCaptures] = [job]

        return failures

    # A flaky job is if it's the only job that has that failureCapture and is not the most recent job
    def get_flaky_jobs(self) -> list[Any]:
```
- **EN**: This chunk defines `get_flaky_jobs`, which implements a focused step inside the alert automation pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_flaky_jobs`，其作用是实现告警自动化流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 127-137
```python
        unique_failures = self.get_unique_failures(self.filtered_statuses)
        flaky_jobs = []
        for failure in unique_failures:
            failure_list = unique_failures[failure]
            if (
                len(failure_list) == 1
                and failure_list[0]["sha"] != self.current_status["sha"]
            ):
                flaky_jobs.append(failure_list[0])
        return flaky_jobs
```
- **EN**: This chunk continues `get_flaky_jobs` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_flaky_jobs`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 138-148
```python
    # The most recent failure chain is an array of jobs that have the same-ish failures.
    # A success in the middle of the chain will terminate the chain.
    def get_most_recent_failure_chain(self) -> list[Any]:
        failures = []
        found_most_recent_failure = False

        for job in self.filtered_statuses:
            if is_job_failed(job):
                failures.append(job)
                found_most_recent_failure = True
            if found_most_recent_failure and not is_job_failed(job):
```
- **EN**: This chunk defines `get_most_recent_failure_chain`, which implements a focused step inside the alert automation pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `get_most_recent_failure_chain`，其作用是实现告警自动化流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 149-158
```python
                break

        return failures

    def should_alert(self) -> bool:
        # Group jobs by their failures. The length of the failure chain is used
        # to raise the alert, so we can do a simple tweak here to use the length
        # of the longest unique chain
        unique_failures = self.get_unique_failures(self.failure_chain)
```
- **EN**: This chunk defines `should_alert`, which aggregates signals and turns them into summaries, metrics, or alerts. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `should_alert`，其作用是聚合信号，并将其转化为摘要、指标或告警。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 159-168
```python
        return (
            self.current_status is not None
            and self.current_status["conclusion"] != SUCCESS
            and any(
                len(failure_chain) >= FAILURE_CHAIN_THRESHOLD
                for failure_chain in unique_failures.values()
            )
            and all(
                disabled_alert not in self.job_name
                for disabled_alert in DISABLED_ALERTS
```
- **EN**: This chunk continues `should_alert` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `should_alert`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 169-181
```python
            )
        )

    def __repr__(self) -> str:
        return f"jobName: {self.job_name}"


def fetch_hud_data(repo: str, branch: str) -> Any:
    response = requests.get(f"https://hud.pytorch.org/api/hud/{repo}/{branch}/0")
    response.raise_for_status()
    hud_data = json.loads(response.text)
    return (hud_data["jobNames"], hud_data["shaGrid"])
```
- **EN**: This chunk defines `fetch_hud_data`, which implements a focused step inside the alert automation pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `fetch_hud_data`，其作用是实现告警自动化流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 182-190
```python

# Creates a Dict of Job Name -> [JobData]. Essentially a Column in HUD
def map_job_data(jobNames: Any, shaGrid: Any) -> dict[str, Any]:
    jobData = defaultdict(list)
    for sha in shaGrid:
        for ind, job in enumerate(sha["jobs"]):
            jobData[jobNames[ind]].append(job)
    return jobData
```
- **EN**: This chunk defines `map_job_data`, which implements a focused step inside the alert automation pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `map_job_data`，其作用是实现告警自动化流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 191-200
```python

def is_job_failed(job: Any) -> bool:
    conclusion = job.get("conclusion", None)
    return conclusion is not None and conclusion != SUCCESS and conclusion != PENDING


def is_job_skipped(job: Any) -> bool:
    conclusion = job.get("conclusion", None)
    return conclusion in (NEUTRAL, SKIPPED) or conclusion is None
```
- **EN**: This chunk defines `is_job_skipped`, which implements a focused step inside the alert automation pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `is_job_skipped`，其作用是实现告警自动化流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 201-218
```python

def get_failed_jobs(job_data: list[Any]) -> list[Any]:
    return [job for job in job_data if job["conclusion"] == "failure"]


def classify_jobs(
    all_job_names: list[str], sha_grid: Any, filtered_jobs_names: set[str]
) -> tuple[list[JobStatus], list[Any]]:
    """
    Creates Job Statuses which has the logic for if need to alert or if there's flaky jobs.
    Classifies jobs into jobs to alert on and flaky jobs.
    :param all_job_names: list of all job names as returned by the HUD
    :param sha_grid: list of all job data as returned by the HUD (parallel index to all_job_names)
    :param filtered_jobs_names: set of job names to actually consider
    :return:
    """
    job_data = map_job_data(all_job_names, sha_grid)
    job_statuses: list[JobStatus] = []
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `classify_jobs`, which implements a focused step inside the alert automation pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `classify_jobs`，其作用是实现告警自动化流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 219-228
```python
    for job in job_data:
        job_statuses.append(JobStatus(job, job_data[job]))

    jobs_to_alert_on = []
    flaky_jobs = []

    for job_status in job_statuses:
        if job_status.job_name not in filtered_jobs_names:
            continue
        if job_status.should_alert():
```
- **EN**: This chunk continues `classify_jobs` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `classify_jobs`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 229-237
```python
            jobs_to_alert_on.append(job_status)
        flaky_jobs.extend(job_status.flaky_jobs)

    return jobs_to_alert_on, flaky_jobs


# filter job names that don't match the regex
def filter_job_names(job_names: list[str], job_name_regex: str) -> list[str]:
    if job_name_regex:
```
- **EN**: This chunk defines `filter_job_names`, which narrows a larger candidate set down to the items relevant for the current workflow. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `filter_job_names`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 238-248
```python
        return [
            job_name for job_name in job_names if re.match(job_name_regex, job_name)
        ]
    return job_names


def get_recurrently_failing_jobs_alerts(
    repo: str, branch: str, job_name_regex: str
) -> list[dict[str, Any]]:
    job_names, sha_grid = fetch_hud_data(repo=repo, branch=branch)
```
- **EN**: This chunk defines `get_recurrently_failing_jobs_alerts`, which aggregates signals and turns them into summaries, metrics, or alerts. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_recurrently_failing_jobs_alerts`，其作用是聚合信号，并将其转化为摘要、指标或告警。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 249-259
```python
    filtered_job_names = set(filter_job_names(job_names, job_name_regex))
    if job_name_regex:
        print()
        print(f"Filtered to {len(filtered_job_names)} jobs:")
        if len(filtered_job_names) == 0:
            print("No jobs matched the regex")
        elif len(filtered_job_names) == len(job_names):
            print("All jobs matched the regex")
        else:
            print("\n".join(filtered_job_names))
```
- **EN**: This chunk continues `get_recurrently_failing_jobs_alerts` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_recurrently_failing_jobs_alerts`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 260-277
```python
    (recurrently_failing_jobs, flaky_jobs) = classify_jobs(
        job_names, sha_grid, filtered_job_names
    )

    alerts = []
    for job in recurrently_failing_jobs:
        entry = {
            "AlertType": "Recurrently Failing Job",
            "AlertObject": job.job_name,
            "OncallTeams": [],
            "OncallIndividuals": [],
            "Flags": [],
            "sha": job.failure_chain[-1]["sha"],
            "branch": branch,
        }
        alerts.append(entry)
    return alerts
```
- **EN**: This chunk continues `get_recurrently_failing_jobs_alerts` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_recurrently_failing_jobs_alerts`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 278-295
```python

def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--repo",
        help="Repository to do checks for",
        type=str,
        default=os.getenv("REPO_TO_CHECK", "pytorch/pytorch"),
    )
    parser.add_argument(
        "--branch",
        help="Branch to do checks for",
        type=str,
        default=os.getenv("BRANCH_TO_CHECK", "main"),
    )
    parser.add_argument(
        "--job-name-regex",
        help="Consider only job names matching given regex (if omitted, all jobs are matched)",
```
- **EN**: This chunk defines `parse_args`, which parses or loads structured input into tool-friendly data structures. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `parse_args`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 296-312
```python
        type=str,
        default=os.getenv("JOB_NAME_REGEX", ""),
    )
    parser.add_argument(
        "--with-flaky-test-alert",
        help="Run this script with the flaky test alerting",
        type=distutils.util.strtobool,
        default=os.getenv("WITH_FLAKY_TEST_ALERT", "YES"),
    )
    parser.add_argument(
        "--dry-run",
        help="Whether or not to actually post issues",
        type=distutils.util.strtobool,
        default=os.getenv("DRY_RUN", "YES"),
    )
    return parser.parse_args()
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Pattern-matching logic extracts structured facts from loosely formatted text inputs. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 313-320
```python

if __name__ == "__main__":
    args = parse_args()
    data = json.dumps(
        get_recurrently_failing_jobs_alerts(args.repo, args.branch, args.job_name_regex)
    )

    print(data)
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Alert automation**
  - EN: This file belongs to the alert automation layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于告警自动化层，应结合同一子目录中的相邻脚本一起理解。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **ALL_SKIPPED_THRESHOLD**
  - EN: `ALL_SKIPPED_THRESHOLD` is one of the main local symbols exposed or implemented here.
  - CN: `ALL_SKIPPED_THRESHOLD` 是此处暴露或实现的主要局部符号之一。
- **SIMILARITY_THRESHOLD**
  - EN: `SIMILARITY_THRESHOLD` is one of the main local symbols exposed or implemented here.
  - CN: `SIMILARITY_THRESHOLD` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `argparse`, `json`, `os`, `re`, `collections`, `difflib`, `typing`
- **External packages / 外部依赖包**: `requests`, `setuptools`
- **Primary symbols in this file / 本文件核心符号**: `ALL_SKIPPED_THRESHOLD`, `SIMILARITY_THRESHOLD`, `FAILURE_CHAIN_THRESHOLD`, `MAX_CONCURRENT_ALERTS`, `FAILED_JOB_PATTERN`, `PENDING`, `NEUTRAL`, `SKIPPED`, `SUCCESS`, `FAILURE`, `CANCELED`, `ISSUES_WITH_LABEL_QUERY`
