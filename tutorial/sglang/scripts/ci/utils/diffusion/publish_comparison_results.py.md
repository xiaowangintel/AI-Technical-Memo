# publish_comparison_results.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/diffusion/publish_comparison_results.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `publish_comparison_results` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `publish_comparison_results` 流程，主要负责CI 编排、测试执行。它属于 `diffusion` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Publish diffusion comparison results to sgl-project/ci-data repo.

Pushes comparison-results.json, dashboard.md, and chart PNG files to the
ci-data repository for historical tracking. Chart PNGs are stored under
diffusion-comparisons/charts/ so they can be referenced via
raw.githubusercontent URLs in the dashboard markdown (GitHub Step Summary
blocks data: URIs).

Usage:
    python3 scripts/ci/utils/diffusion/publish_comparison_results.py \
        --results comparison-results.json \
        --dashboard dashboard.md \
        --charts-dir comparison-charts/
"""
```
**EN:** Publish diffusion comparison results to sgl-project/ci-data repo.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 16-21: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import os
import sys
import time
from datetime import datetime, timezone
from pathlib import Path
```
**EN:** This block loads argparse, os, sys, time, datetime, pathlib. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, os, sys, time, datetime, pathlib。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 25-49: Top-level if logic / 顶层 if 逻辑
```python
if __package__:
    from ..publish_traces import (
        create_blobs,
        create_commit,
        create_tree,
        get_branch_sha,
        get_tree_sha,
        is_permission_error,
        is_rate_limit_error,
        update_branch_ref,
        verify_token_permissions,
    )
else:
    sys.path.insert(0, str(Path(__file__).resolve().parents[1]))
    from publish_traces import (
        create_blobs,
        create_commit,
        create_tree,
        get_branch_sha,
        get_tree_sha,
        is_permission_error,
        is_rate_limit_error,
        update_branch_ref,
        verify_token_permissions,
    )
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 52-55: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
REPO_OWNER = "sgl-project"
REPO_NAME = "ci-data"
BRANCH = "main"
STORAGE_PREFIX = "diffusion-comparisons"
```
**EN:** This section defines REPO_OWNER, REPO_NAME, BRANCH, STORAGE_PREFIX, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 REPO_OWNER, REPO_NAME, BRANCH, STORAGE_PREFIX，用于把可复用的默认值集中在模块顶部。

### Lines 58-76: Defines the `_collect_chart_files` routine / 定义 `_collect_chart_files` 例程
```python
def _collect_chart_files(charts_dir: str) -> list[tuple[str, bytes]]:
    """Collect PNG chart files from directory for upload."""
    files: list[tuple[str, bytes]] = []
    if not charts_dir or not os.path.isdir(charts_dir):
        return files

    for entry in sorted(os.listdir(charts_dir)):
        if not entry.lower().endswith(".png"):
            continue
        full_path = os.path.join(charts_dir, entry)
        if not os.path.isfile(full_path):
            continue
        with open(full_path, "rb") as f:
            content = f.read()
        # Store charts under diffusion-comparisons/charts/
        repo_path = f"{STORAGE_PREFIX}/charts/{entry}"
        files.append((repo_path, content))

    return files
```
**EN:** This block defines `_collect_chart_files`. Collect PNG chart files from directory for upload. It accepts 1 parameter(s): charts_dir. Internally it reads or writes files.
**CN:** 该代码块定义了 `_collect_chart_files`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：charts_dir。内部会读写文件。

### Lines 79-194: Defines the `publish_comparison` routine / 定义 `publish_comparison` 例程
```python
def publish_comparison(
    results_path: str,
    dashboard_path: str | None = None,
    charts_dir: str | None = None,
) -> None:
    """Publish comparison results, dashboard, and charts to ci-data repo."""
    token = os.environ.get("GH_PAT_FOR_NIGHTLY_CI_DATA") or os.environ.get(
        "GITHUB_TOKEN"
    )
    if not token:
        print("Error: GH_PAT_FOR_NIGHTLY_CI_DATA or GITHUB_TOKEN not set")
        sys.exit(1)

    run_id = os.environ.get("GITHUB_RUN_ID", "local")
    run_number = os.environ.get("GITHUB_RUN_NUMBER", "0")

    # Verify permissions
    perm = verify_token_permissions(REPO_OWNER, REPO_NAME, token)
    if perm == "rate_limited":
        print("Warning: Rate limited, skipping publish")
        return
    elif not perm:
        print("Error: Token permission verification failed")
        sys.exit(1)

    # Prepare files to upload
    files_to_upload: list[tuple[str, bytes]] = []

    # Results JSON: stored with date prefix for chronological ordering
    date_prefix = datetime.now(timezone.utc).strftime("%Y-%m-%d")
    results_target = f"{STORAGE_PREFIX}/{date_prefix}_{run_id}.json"
    with open(results_path, "rb") as f:
        files_to_upload.append((results_target, f.read()))

    # Dashboard markdown: always overwrite latest
    if dashboard_path and os.path.exists(dashboard_path):
        dashboard_target = f"{STORAGE_PREFIX}/dashboard.md"
        with open(dashboard_path, "rb") as f:
            files_to_upload.append((dashboard_target, f.read()))

    # Chart PNG files
    chart_files = _collect_chart_files(charts_dir)
    if chart_files:
        print(f"Found {len(chart_files)} chart PNG(s) to upload")
        files_to_upload.extend(chart_files)

    print(f"Publishing {len(files_to_upload)} file(s) to {REPO_OWNER}/{REPO_NAME}")

    # Create blobs
    try:
        tree_items = create_blobs(REPO_OWNER, REPO_NAME, files_to_upload, token)
    except Exception as e:
        if is_rate_limit_error(e):
            print("Warning: Rate limited during blob creation, skipping")
            return
        if is_permission_error(e):
            print(f"Error: No write permission to {REPO_OWNER}/{REPO_NAME}")
            sys.exit(1)
        raise

    # Commit with retry (handle concurrent writes)
    max_retries = 5
    retry_delay = 5

    for attempt in range(max_retries):
        try:
            branch_sha = get_branch_sha(REPO_OWNER, REPO_NAME, BRANCH, token)
            tree_sha = get_tree_sha(REPO_OWNER, REPO_NAME, branch_sha, token)

            new_tree_sha = create_tree(
                REPO_OWNER, REPO_NAME, tree_sha, tree_items, token
            )

            commit_msg = (
                f"Diffusion comparison results for run {run_id} (#{run_number})"
            )
            commit_sha = create_commit(
                REPO_OWNER, REPO_NAME, new_tree_sha, branch_sha, commit_msg, token
            )

            update_branch_ref(REPO_OWNER, REPO_NAME, BRANCH, commit_sha, token)
            print(
                f"Successfully published comparison results (commit {commit_sha[:7]})"
            )
            return

        except Exception as e:
            is_retryable = False
            if hasattr(e, "error_body"):
                body = getattr(e, "error_body", "")
                if "Update is not a fast forward" in body:
                    is_retryable = True
                elif "Object does not exist" in body:
                    is_retryable = True

            from urllib.error import HTTPError

            if isinstance(e, HTTPError) and e.code in [422, 500, 502, 503, 504]:
                is_retryable = True

            if is_rate_limit_error(e):
                print("Warning: Rate limited, skipping publish")
                return

            if is_permission_error(e):
                print(f"Error: No write permission to {REPO_OWNER}/{REPO_NAME}")
                sys.exit(1)

            if is_retryable and attempt < max_retries - 1:
                print(
                    f"Attempt {attempt + 1}/{max_retries} failed, retrying in {retry_delay}s..."
                )
                time.sleep(retry_delay)
            else:
                print(f"Failed to publish after {attempt + 1} attempts: {e}")
                raise
```
**EN:** This block defines `publish_comparison`. Publish comparison results, dashboard, and charts to ci-data repo. It accepts 3 parameter(s): results_path, dashboard_path, charts_dir. Internally it reads or writes files, checks file existence, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `publish_comparison`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：results_path, dashboard_path, charts_dir。内部会读写文件、检查文件是否存在、输出状态信息、控制进程退出状态。

### Lines 197-227: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Publish diffusion comparison results to ci-data"
    )
    parser.add_argument(
        "--results",
        required=True,
        help="Path to comparison-results.json",
    )
    parser.add_argument(
        "--dashboard",
        default=None,
        help="Path to dashboard.md (optional)",
    )
    parser.add_argument(
        "--charts-dir",
        default=None,
        help="Directory containing chart PNG files to upload (optional)",
    )

    args = parser.parse_args()

    if not os.path.exists(args.results):
        print(f"Error: Results file not found: {args.results}")
        sys.exit(1)

    publish_comparison(
        results_path=args.results,
        dashboard_path=args.dashboard,
        charts_dir=args.charts_dir,
    )
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it checks file existence, parses CLI arguments, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会检查文件是否存在、解析命令行参数、输出状态信息、控制进程退出状态。

### Lines 230-231: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `datetime`, `os`, `pathlib`, `sys`, `time`, `urllib`
- **Third-party modules / 第三方模块**: `publish_traces`
- **Repository-local imports / 仓库内导入**: `publish_traces`
