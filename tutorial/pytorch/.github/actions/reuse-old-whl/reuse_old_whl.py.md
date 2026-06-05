# reuse_old_whl.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/actions/reuse-old-whl/reuse_old_whl.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16 / 第 1-16 行

````python
from __future__ import annotations

import argparse
import os
import subprocess
import sys
from functools import lru_cache
from pathlib import Path
from typing import Any, cast

import requests


REPO_ROOT = Path(__file__).resolve().parent.parent.parent.parent
sys.path.insert(0, str(REPO_ROOT))
from tools.stats.upload_metrics import emit_metric
````

- EN: This block imports dependencies such as `__future__`, `argparse`, `os`, `subprocess`; parses command-line arguments for script entry points; touches GitHub-related data or remote service interactions.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数；处理 GitHub 相关数据或远程服务交互。

### Lines 19-36 / 第 19-36 行

````python
sys.path.remove(str(REPO_ROOT))  # Clean up sys.path after import


FORCE_REBUILD_LABEL = "ci-force-rebuild"


@lru_cache
def get_merge_base() -> str:
    merge_base = subprocess.check_output(
        ["git", "merge-base", "HEAD", "origin/main"],
        text=True,
        stderr=subprocess.DEVNULL,
    ).strip()
    # Remove this when we turn this off for the main branch
    if merge_base == get_head_sha():
        print("Merge base is the same as HEAD, using HEAD^")
        merge_base = subprocess.check_output(
            ["git", "rev-parse", "HEAD^"],
````

- EN: This block defines callable units such as `get_merge_base`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 37-51 / 第 37-51 行

````python
            text=True,
            stderr=subprocess.DEVNULL,
        ).strip()
    print(f"Merge base: {merge_base}")
    return merge_base


@lru_cache
def get_head_sha() -> str:
    sha = subprocess.check_output(
        ["git", "rev-parse", "HEAD"],
        text=True,
        stderr=subprocess.DEVNULL,
    ).strip()
    return sha
````

- EN: This block defines callable units such as `get_head_sha`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 54-69 / 第 54-69 行

````python
def is_main_branch() -> bool:
    return False
    # Testing on main branch for now
    # print(
    #     f"Checking if we are on main branch: merge base {get_merge_base()}, head {get_head_sha()}"
    # )
    # return get_merge_base() == get_head_sha()


def query_github_api(url: str) -> Any:
    headers = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}",
    }
    response = requests.get(url, headers=headers)
    return response.json()
````

- EN: This block defines callable units such as `is_main_branch`, `query_github_api`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 72-89 / 第 72-89 行

````python
@lru_cache
def check_labels_for_pr() -> bool:
    # Check if the current commit is part of a PR and if it has the
    # FORCE_REBUILD_LABEL
    head_sha = get_head_sha()
    url = f"https://api.github.com/repos/pytorch/pytorch/commits/{head_sha}/pulls"
    response = query_github_api(url)

    print(
        f"Found {len(response)} PRs for commit {head_sha}: {[pr['number'] for pr in response]}"
    )
    for pr in response:
        labels = pr.get("labels", [])
        for label in labels:
            if label["name"] == FORCE_REBUILD_LABEL:
                print(f"Found label {FORCE_REBUILD_LABEL} in PR {pr['number']}.")
                return True
    return False
````

- EN: This block defines callable units such as `check_labels_for_pr`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 92-102 / 第 92-102 行

````python
def check_issue_open() -> bool:
    # Check if issue #153759 is open.  This is the config issue for quickly
    # forcing everyone to build
    url = "https://api.github.com/repos/pytorch/pytorch/issues/153759"
    response = query_github_api(url)
    if response.get("state") == "open":
        print("Issue #153759 is open.")
        return True
    else:
        print("Issue #153759 is not open.")
        return False
````

- EN: This block defines callable units such as `check_issue_open`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 105-114 / 第 105-114 行

````python
def get_workflow_id(run_id: str) -> str | None:
    # Get the workflow ID that corresponds to the file for the run ID
    url = f"https://api.github.com/repos/pytorch/pytorch/actions/runs/{run_id}"
    response = query_github_api(url)
    if "workflow_id" in response:
        print(f"Found workflow ID for run ID {run_id}: {response['workflow_id']}")
        return cast(str, response["workflow_id"])
    else:
        print("No workflow ID found.")
        return None
````

- EN: This block defines callable units such as `get_workflow_id`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 117-130 / 第 117-130 行

````python
def ok_changed_file(file: str) -> bool:
    # Return true if the file is in the list of allowed files to be changed to
    # reuse the old whl
    if (
        file.startswith("torch/")
        and file.endswith(".py")
        and not file.startswith("torch/csrc/")
    ):
        return True
    if file.startswith("test/") and file.endswith(".py"):
        return True
    if file.startswith("docs/") and file.endswith((".md", ".rst")):
        return True
    return False
````

- EN: This block defines callable units such as `ok_changed_file`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 133-150 / 第 133-150 行

````python
def check_changed_files(sha: str) -> bool:
    # Return true if all the changed files are in the list of allowed files to
    # be changed to reuse the old whl

    # Removing files in the torch folder is not allowed since rsync will not
    # remove files
    removed_files = (
        subprocess.check_output(
            [
                "git",
                "diff",
                "--name-only",
                sha,
                "HEAD",
                "--diff-filter=D",
                "--no-renames",
            ],
            text=True,
````

- EN: This block defines callable units such as `check_changed_files`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 151-168 / 第 151-168 行

````python
            stderr=subprocess.DEVNULL,
        )
        .strip()
        .split()
    )
    if any(file.startswith("torch/") for file in removed_files):
        print(
            f"Removed files between {sha} and HEAD: {removed_files}, cannot reuse old whl"
        )
        return False

    changed_files = (
        subprocess.check_output(
            ["git", "diff", "--name-only", sha, "HEAD", "--no-renames"],
            text=True,
            stderr=subprocess.DEVNULL,
        )
        .strip()
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 169-178 / 第 169-178 行

````python
        .split()
    )
    print(f"Checking changed files between {sha} and HEAD:")
    for file in changed_files:
        if not ok_changed_file(file):
            print(f"  File {file} is not allowed to be changed.")
            return False
        else:
            print(f"  File {file} is allowed to be changed.")
    return True
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 181-198 / 第 181-198 行

````python
def find_old_whl(workflow_id: str, build_environment: str, sha: str) -> bool:
    # Find the old whl on s3 and download it to artifacts.zip
    if build_environment is None:
        print("BUILD_ENVIRONMENT is not set.")
        return False
    print(f"SHA: {sha}, workflow_id: {workflow_id}")

    workflow_runs = query_github_api(
        f"https://api.github.com/repos/pytorch/pytorch/actions/workflows/{workflow_id}/runs?head_sha={sha}&branch=main&per_page=100"
    )
    if workflow_runs.get("total_count", 0) == 0:
        print("No workflow runs found.")
        return False
    for run in workflow_runs.get("workflow_runs", []):
        # Look in s3 for the old whl
        run_id = run["id"]
        try:
            url = f"https://gha-artifacts.s3.amazonaws.com/pytorch/pytorch/{run_id}/{build_environment}/artifacts.zip"
````

- EN: This block defines callable units such as `find_old_whl`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 199-211 / 第 199-211 行

````python
            print(f"Checking for old whl at {url}")
            response = requests.get(
                url,
            )
            if response.status_code == 200:
                with open("artifacts.zip", "wb") as f:
                    f.write(response.content)
                    print(f"Found old whl file from s3: {url}")
                    return True
        except requests.RequestException as e:
            print(f"Error checking for old whl: {e}")
            continue
    return False
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 214-231 / 第 214-231 行

````python
def unzip_artifact_and_replace_files() -> None:
    # Unzip the artifact and replace files
    subprocess.check_output(
        ["unzip", "-o", "artifacts.zip", "-d", "artifacts"],
    )
    os.remove("artifacts.zip")

    head_sha = get_head_sha()

    # Rename wheel into zip
    wheel_path = Path("artifacts/dist").glob("*.whl")
    for path in wheel_path:
        # Should be of the form torch-2.0.0+git1234567-cp37-etc.whl
        # Should usually be the merge base sha but for the ones that didn't do
        # the replacement, it won't be.  Can probably change it to just be merge
        # base later
        old_version = f"+git{path.stem.split('+')[1].split('-')[0][3:]}"
        new_version = f"+git{head_sha[:7]}"
````

- EN: This block defines callable units such as `unzip_artifact_and_replace_files`; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项。

### Lines 233-248 / 第 233-248 行

````python
        def rename_to_new_version(file: str | Path) -> None:
            # Rename file with old_version to new_version
            subprocess.check_output(
                ["mv", file, str(file).replace(old_version, new_version)]
            )

        def change_content_to_new_version(file: str | Path) -> None:
            # Check if is a file
            if os.path.isdir(file):
                return
            # Replace the old version in the file with the new version
            with open(file) as f:
                content = f.read()
                content = content.replace(old_version, new_version)
            with open(file, "w") as f:
                f.write(content)
````

- EN: This block defines callable units such as `rename_to_new_version`, `change_content_to_new_version`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 250-266 / 第 250-266 行

````python
        zip_path = path.with_suffix(".zip")
        os.rename(path, zip_path)
        old_stem = zip_path.stem
        # Unzip the wheel
        subprocess.check_output(
            ["unzip", "-o", zip_path, "-d", f"artifacts/dist/{old_stem}"],
        )

        # Remove the old wheel (which is now a zip file)
        os.remove(zip_path)

        # Copy python files into the artifact
        subprocess.check_output(
            ["rsync", "-avz", "torch", f"artifacts/dist/{old_stem}"],
        )

        change_content_to_new_version(f"artifacts/dist/{old_stem}/torch/version.py")
````

- EN: This block implements local helper logic for reuse old whl.
- CN: 该代码块实现与 reuse old whl 相关的局部辅助逻辑。

### Lines 268-279 / 第 268-279 行

````python
        for file in Path(f"artifacts/dist/{old_stem}").glob(
            "*.dist-info/*",
        ):
            change_content_to_new_version(file)

        rename_to_new_version(f"artifacts/dist/{old_stem}")
        new_stem = old_stem.replace(old_version, new_version)

        for file in Path(f"artifacts/dist/{new_stem}").glob(
            "*.dist-info",
        ):
            rename_to_new_version(file)
````

- EN: This block iterates through collections, records, or generated items.
- CN: 该代码块遍历集合、记录或生成项。

### Lines 281-293 / 第 281-293 行

````python
        # Zip the wheel back
        subprocess.check_output(
            ["zip", "-r", f"{new_stem}.zip", "."],
            cwd=f"artifacts/dist/{new_stem}",
        )

        subprocess.check_output(
            [
                "mv",
                f"artifacts/dist/{new_stem}/{new_stem}.zip",
                f"artifacts/dist/{new_stem}.whl",
            ],
        )
````

- EN: This block implements local helper logic for reuse old whl.
- CN: 该代码块实现与 reuse old whl 相关的局部辅助逻辑。

### Lines 295-305 / 第 295-305 行

````python
        # Remove the extracted folder
        subprocess.check_output(
            ["rm", "-rf", f"artifacts/dist/{new_stem}"],
        )

    # Rezip the artifact
    subprocess.check_output(["zip", "-r", "artifacts.zip", "."], cwd="artifacts")
    subprocess.check_output(
        ["mv", "artifacts/artifacts.zip", "."],
    )
    return None
````

- EN: This block returns computed values to callers.
- CN: 该代码块向调用方返回计算结果。

### Lines 308-325 / 第 308-325 行

````python
def set_output() -> None:
    print("Setting output reuse=true")
    if os.getenv("GITHUB_OUTPUT"):
        with open(str(os.getenv("GITHUB_OUTPUT")), "a") as env:
            print("reuse=true", file=env)
    else:
        print("::set-output name=reuse::true")


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(description="Check for old whl files.")
    parser.add_argument("--run-id", type=str, required=True, help="Workflow ID")
    parser.add_argument(
        "--build-environment", type=str, required=True, help="Build environment"
    )
    parser.add_argument(
        "--github-ref",
        type=str,
````

- EN: This block defines callable units such as `set_output`, `parse_args`; parses command-line arguments for script entry points; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 326-340 / 第 326-340 行

````python
    )
    return parser.parse_args()


def can_reuse_whl(args: argparse.Namespace) -> tuple[bool, str]:
    if args.github_ref and any(
        args.github_ref.startswith(x)
        for x in [
            "refs/heads/release",
            "refs/tags/v",
            "refs/heads/nightly",
        ]
    ):
        print("Release branch, rebuild whl")
        return (False, "Release branch")
````

- EN: This block defines callable units such as `can_reuse_whl`; parses command-line arguments for script entry points; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 342-357 / 第 342-357 行

````python
    if not check_changed_files(get_merge_base()):
        print("Cannot use old whl due to the changed files, rebuild whl")
        return (False, "Changed files not allowed")

    if check_labels_for_pr():
        print(f"Found {FORCE_REBUILD_LABEL} label on PR, rebuild whl")
        return (False, "Found FORCE_REBUILD_LABEL on PR")

    if check_issue_open():
        print("Issue #153759 is open, rebuild whl")
        return (False, "Issue #153759 is open")

    workflow_id = get_workflow_id(args.run_id)
    if workflow_id is None:
        print("No workflow ID found, rebuild whl")
        return (False, "No workflow ID found")
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 359-375 / 第 359-375 行

````python
    if not find_old_whl(workflow_id, args.build_environment, get_merge_base()):
        print("No old whl found, rebuild whl")
        return (False, "No old whl found")
        # TODO: go backwards from merge base to find more runs

    return (True, "Found old whl")


if __name__ == "__main__":
    args = parse_args()

    reuse_whl, reason = can_reuse_whl(args)

    if reuse_whl:
        print("Reusing old whl")
        unzip_artifact_and_replace_files()
        set_output()
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 377-386 / 第 377-386 行

````python
    emit_metric(
        "reuse_old_whl",
        {
            "reuse_whl": reuse_whl,
            "reason": reason,
            "build_environment": args.build_environment,
            "merge_base": get_merge_base(),
            "head_sha": get_head_sha(),
        },
    )
````

- EN: This block implements local helper logic for reuse old whl.
- CN: 该代码块实现与 reuse old whl 相关的局部辅助逻辑。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `__future__`, `argparse`, `os`, `subprocess`, `sys`, `functools`, `pathlib`, `typing`
- Internal imports / 内部导入: none
- External imports / 外部导入: `requests`, `tools.stats.upload_metrics`
- Classes / 类: none
- Functions / 函数: `get_merge_base`, `get_head_sha`, `is_main_branch`, `query_github_api`, `check_labels_for_pr`, `check_issue_open`, `get_workflow_id`, `ok_changed_file`, `check_changed_files`, `find_old_whl`, ...
- Test entry points / 测试入口: none
