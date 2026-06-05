# close_nonexistent_disable_issues.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/close_nonexistent_disable_issues.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17 / 第 1-17 行

````python
import argparse
import json
import multiprocessing as mp
import os
import re
import sys
import tempfile
from pathlib import Path
from typing import Any

import requests
from gitutils import retries_decorator


REPO_ROOT = Path(__file__).resolve().parent.parent.parent
sys.path.insert(0, str(REPO_ROOT))
from tools.testing.clickhouse import query_clickhouse
````

- EN: This block imports dependencies such as `argparse`, `json`, `multiprocessing`, `os`; parses command-line arguments for script entry points; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 20-37 / 第 20-37 行

````python
sys.path.pop(0)


LOGS_QUERY = """
with
    shas as (
        SELECT
            distinct
            push.head_commit.id as sha
        FROM
            -- Not bothering with final here
            default.push
        WHERE
            push.ref = 'refs/heads/viable/strict'
            AND push.repository.'full_name' = 'pytorch/pytorch'
        ORDER BY
            push.head_commit.'timestamp' desc
        LIMIT
````

- EN: This block implements local helper logic for close nonexistent disable issues.
- CN: 该代码块实现与 close nonexistent disable issues 相关的局部辅助逻辑。

### Lines 38-51 / 第 38-51 行

````python
            5
    )
select
    id,
    name
from
    default.workflow_job j final
    join shas on shas.sha = j.head_sha
where
    j.id in (select id from materialized_views.workflow_job_by_head_sha where head_sha in (select sha from shas))
    and j.name like '% / test%'
    and j.name not like '%rerun_disabled_tests%'
    and j.name not like '%mem_leak_check%'
"""
````

- EN: This block implements local helper logic for close nonexistent disable issues.
- CN: 该代码块实现与 close nonexistent disable issues 相关的局部辅助逻辑。

### Lines 53-63 / 第 53-63 行

````python
TEST_EXISTS_QUERY = """
select
    name
from
    default.test_run_s3
where
    name::String like {name: String}
    and classname like {classname: String}
    and time_inserted > CURRENT_TIMESTAMP() - INTERVAL 7 DAY
limit 1
"""
````

- EN: This block implements local helper logic for close nonexistent disable issues.
- CN: 该代码块实现与 close nonexistent disable issues 相关的局部辅助逻辑。

### Lines 65-80 / 第 65-80 行

````python
CLOSING_COMMENT = (
    "I cannot find any mention of this test in the database for the past 7 days "
    "or in the logs for the past 5 commits on viable/strict.  Closing this "
    "issue as it is highly likely that this test has either been renamed or "
    "removed.  If you think this is a false positive, please feel free to "
    "re-open this issue."
)

DISABLED_TESTS_JSON = (
    "https://ossci-metrics.s3.amazonaws.com/disabled-tests-condensed.json"
)


@retries_decorator()
def query_db(query: str, params: dict[str, Any]) -> list[dict[str, Any]]:
    return query_clickhouse(query, params)
````

- EN: This block defines callable units such as `query_db`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 83-97 / 第 83-97 行

````python
def parse_args() -> Any:
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--dry-run",
        action="store_true",
        help="Only list the tests.",
    )
    return parser.parse_args()


def download_log_worker(temp_dir: str, id: int, name: str) -> None:
    url = f"https://ossci-raw-job-status.s3.amazonaws.com/log/{id}"
    data = requests.get(url).text
    with open(f"{temp_dir}/{name.replace('/', '_')} {id}.txt", "x") as f:
        f.write(data)
````

- EN: This block defines callable units such as `parse_args`, `download_log_worker`; parses command-line arguments for script entry points; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 100-117 / 第 100-117 行

````python
def printer(item: tuple[str, tuple[int, str, list[Any]]], extra: str) -> None:
    test, (_, link, _) = item
    print(f"{link:<55} {test:<120} {extra}")


def close_issue(num: int) -> None:
    headers = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": f"token {os.environ['GITHUB_TOKEN']}",
    }
    response = requests.post(
        f"https://api.github.com/repos/pytorch/pytorch/issues/{num}/comments",
        data=json.dumps({"body": CLOSING_COMMENT}),
        headers=headers,
    )
    if response.status_code != 201:
        raise RuntimeError(f"Failed to comment on issue {num}: {response.text}")
    response = requests.patch(
````

- EN: This block defines callable units such as `printer`, `close_issue`; branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 118-133 / 第 118-133 行

````python
        f"https://api.github.com/repos/pytorch/pytorch/issues/{num}",
        data=json.dumps({"state": "closed"}),
        headers=headers,
    )
    if response.status_code != 200:
        raise RuntimeError(f"Failed to close issue {num}: {response.text}")


def check_if_exists(
    item: tuple[str, tuple[int, str, list[str]]], all_logs: list[str]
) -> tuple[bool, str]:
    test, (_, link, _) = item
    # Test names should look like `test_a (module.path.classname)`
    reg = re.match(r"(\S+) \((\S*)\)", test)
    if reg is None:
        return False, "poorly formed"
````

- EN: This block defines callable units such as `check_if_exists`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 135-149 / 第 135-149 行

````python
    name = reg[1]
    classname = reg[2].split(".")[-1]

    # Check if there is any mention of the link or the test name in the logs.
    # The link usually shows up in the skip reason.
    present = False
    for log in all_logs:
        if link in log:
            present = True
            break
        if f"{classname}::{name}" in log:
            present = True
            break
    if present:
        return True, "found in logs"
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 151-162 / 第 151-162 行

````python
    # Query DB to see if the test is there
    count = query_db(
        TEST_EXISTS_QUERY, {"name": f"{name}%", "classname": f"{classname}%"}
    )
    if len(count) == 0:
        return False, "not found"
    return True, "found in DB"


if __name__ == "__main__":
    args = parse_args()
    disabled_tests_json = json.loads(requests.get(DISABLED_TESTS_JSON).text)
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 164-177 / 第 164-177 行

````python
    all_logs = []
    jobs = query_db(LOGS_QUERY, {})
    with tempfile.TemporaryDirectory() as temp_dir:
        pool = mp.Pool(20)
        for job in jobs:
            id = job["id"]
            name = job["name"]
            pool.apply_async(download_log_worker, args=(temp_dir, id, name))
        pool.close()
        pool.join()

        for filename in os.listdir(temp_dir):
            with open(f"{temp_dir}/{filename}") as f:
                all_logs.append(f.read())
````

- EN: This block iterates through collections, records, or generated items.
- CN: 该代码块遍历集合、记录或生成项。

### Lines 179-194 / 第 179-194 行

````python
    # If its less than 200 something definitely went wrong.
    if len(all_logs) <= 200:
        raise AssertionError(
            f"Expected more than 200 logs, got {len(all_logs)}. Something went wrong."
        )
    if len(all_logs) != len(jobs):
        raise AssertionError(
            f"Number of logs ({len(all_logs)}) does not match number of jobs ({len(jobs)})"
        )

    to_be_closed = []
    for item in disabled_tests_json.items():
        exists, reason = check_if_exists(item, all_logs)
        printer(item, reason)
        if not exists:
            to_be_closed.append(item)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; validates assumptions and surfaces failures explicitly; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；校验前提条件并显式暴露失败；序列化或解析结构化数据文件。

### Lines 196-212 / 第 196-212 行

````python
    print(f"There are {len(to_be_closed)} issues that will be closed:")
    for item in to_be_closed:
        printer(item, "")

    if args.dry_run:
        print("dry run, not actually closing")
    else:
        failed = False
        for item in to_be_closed:
            _, (num, _, _) = item
            try:
                close_issue(num)
            except RuntimeError as e:
                print(e)
                failed = True
        if failed:
            sys.exit(1)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。


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
- Standard-library imports / 标准库导入: `argparse`, `json`, `multiprocessing`, `os`, `re`, `sys`, `tempfile`, `pathlib`, ...
- Internal imports / 内部导入: none
- External imports / 外部导入: `requests`, `gitutils`, `tools.testing.clickhouse`
- Classes / 类: none
- Functions / 函数: `query_db`, `parse_args`, `download_log_worker`, `printer`, `close_issue`, `check_if_exists`
- Test entry points / 测试入口: none
