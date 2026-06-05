# download_reports.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `scripts/compile_tests/download_reports.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides Python helpers that download, aggregate, and summarize compile-test reporting artifacts.
- **用途 (CN)**: 提供 Python 辅助工具，用于下载、聚合并汇总编译测试报告产物。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行

````python
import json
import os
import pprint
import re
import subprocess

import requests


CONFIGS = {
    "dynamo39": {
        "linux-jammy-py3.10-clang12 / test (dynamo_wrapped, 1, 3, linux.2xlarge)",
        "linux-jammy-py3.10-clang12 / test (dynamo_wrapped, 2, 3, linux.2xlarge)",
        "linux-jammy-py3.10-clang12 / test (dynamo_wrapped, 3, 3, linux.2xlarge)",
````

- EN: This block imports dependencies such as `json`, `os`, `pprint`, `re`; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 15-28 / 第 15-28 行

````python
    },
    "dynamo313": {
        "linux-jammy-py3.13-clang12 / test (dynamo_wrapped, 1, 3, linux.2xlarge)",
        "linux-jammy-py3.13-clang12 / test (dynamo_wrapped, 2, 3, linux.2xlarge)",
        "linux-jammy-py3.13-clang12 / test (dynamo_wrapped, 3, 3, linux.2xlarge)",
    },
    "eager313": {
        "linux-jammy-py3.13-clang12 / test (default, 1, 5, linux.4xlarge)",
        "linux-jammy-py3.13-clang12 / test (default, 2, 5, linux.4xlarge)",
        "linux-jammy-py3.13-clang12 / test (default, 3, 5, linux.4xlarge)",
        "linux-jammy-py3.13-clang12 / test (default, 4, 5, linux.4xlarge)",
        "linux-jammy-py3.13-clang12 / test (default, 5, 5, linux.4xlarge)",
    },
}
````

- EN: This block implements local helper logic for download reports.
- CN: 该代码块实现与 download reports 相关的局部辅助逻辑。

### Lines 31-39 / 第 31-39 行

````python
def download_reports(commit_sha, configs=("dynamo39", "dynamo313", "eager313")):
    log_dir = "tmp_test_reports_" + commit_sha

    def subdir_path(config):
        return f"{log_dir}/{config}"

    for config in configs:
        assert config in CONFIGS.keys(), config
    subdir_paths = [subdir_path(config) for config in configs]
````

- EN: This block defines callable units such as `download_reports`, `subdir_path`; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 41-51 / 第 41-51 行

````python
    # See which configs we haven't downloaded logs for yet
    missing_configs = []
    for config, path in zip(configs, subdir_paths):
        if os.path.exists(path):
            continue
        missing_configs.append(config)
    if len(missing_configs) == 0:
        print(
            f"All required logs appear to exist, not downloading again. Run `rm -rf {log_dir}` if this is not the case"
        )
        return subdir_paths
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 53-66 / 第 53-66 行

````python
    output = subprocess.check_output(
        ["gh", "run", "list", "-c", commit_sha, "-w", "pull", "--json", "databaseId"]
    ).decode()
    workflow_run_id = str(json.loads(output)[0]["databaseId"])
    output = subprocess.check_output(["gh", "run", "view", workflow_run_id])
    workflow_jobs = parse_workflow_jobs(output)
    print("found the following workflow jobs:")
    pprint.pprint(workflow_jobs)

    # Figure out which jobs we need to download logs for
    required_jobs = []
    for config in configs:
        required_jobs.extend(list(CONFIGS[config]))
    for job in required_jobs:
````

- EN: This block iterates through collections, records, or generated items; serializes or parses structured data files.
- CN: 该代码块遍历集合、记录或生成项；序列化或解析结构化数据文件。

### Lines 67-74 / 第 67-74 行

````python
        assert job in workflow_jobs, (
            f"{job} not found, is the commit_sha correct? has the job finished running? The GitHub API may take a couple minutes to update."
        )

    # This page lists all artifacts.
    listings = requests.get(
        f"https://hud.pytorch.org/api/artifacts/s3/{workflow_run_id}"
    ).json()
````

- EN: This block validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 76-89 / 第 76-89 行

````python
    def download_report(job_name, subdir):
        job_id = workflow_jobs[job_name]
        for listing in listings:
            name = listing["name"]
            if not name.startswith("test-reports-"):
                continue
            if name.endswith(f"_{job_id}.zip"):
                url = listing["url"]
                subprocess.run(["wget", "-P", subdir, url], check=True)
                path_to_zip = f"{subdir}/{name}"
                dir_name = path_to_zip[:-4]
                subprocess.run(["unzip", path_to_zip, "-d", dir_name], check=True)
                return
        raise AssertionError("should not be hit")
````

- EN: This block defines callable units such as `download_report`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 91-103 / 第 91-103 行

````python
    if not os.path.exists(log_dir):
        os.mkdir(log_dir)

    for config in set(configs) - set(missing_configs):
        print(
            f"Logs for {config} already exist, not downloading again. Run `rm -rf {subdir_path(config)}` if this is not the case."
        )
    for config in missing_configs:
        subdir = subdir_path(config)
        os.mkdir(subdir)
        job_names = CONFIGS[config]
        for job_name in job_names:
            download_report(job_name, subdir)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 105-116 / 第 105-116 行

````python
    return subdir_paths


def parse_workflow_jobs(output):
    result = {}
    lines = output.decode().split("\n")
    for line in lines:
        match = re.search(r"(\S+ / .*) in .* \(ID (\d+)\)", line)
        if match is None:
            continue
        result[match.group(1)] = match.group(2)
    return result
````

- EN: This block defines callable units such as `parse_workflow_jobs`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `json`, `os`, `pprint`, `re`, `subprocess`
- Internal imports / 内部导入: none
- External imports / 外部导入: `requests`
- Classes / 类: none
- Functions / 函数: `download_reports`, `parse_workflow_jobs`
- Test entry points / 测试入口: none
