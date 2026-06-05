# publish_diffusion_gt.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/diffusion/publish_diffusion_gt.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `publish_diffusion_gt` workflow in SGLang. It mainly handles CI orchestration, nightly workflow scheduling. / 该Python 模块用于支撑 SGLang 中的 `publish_diffusion_gt` 流程，主要负责CI 编排、夜间任务调度。它属于 `diffusion` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Publish diffusion CI ground-truth images to sgl-project/ci-data
via the GitHub API (same pattern as publish_traces.py).
"""
```
**EN:** Publish diffusion CI ground-truth images to sgl-project/ci-data via the GitHub API (same pattern as publish_traces.py).
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 6-12: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import hashlib
import json
import os
import sys
from pathlib import Path
from urllib.error import HTTPError
```
**EN:** This block loads argparse, hashlib, json, os, sys, pathlib, urllib.error. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, hashlib, json, os, sys, pathlib, urllib.error。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 16-42: Top-level if logic / 顶层 if 逻辑
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
        make_github_request,
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
        make_github_request,
        update_branch_ref,
        verify_token_permissions,
    )
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 44-49: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
REPO_OWNER = "sgl-project"
REPO_NAME = "ci-data"
BRANCH = "main"
DEFAULT_TARGET_DIR = "diffusion-ci/consistency_gt/sglang_generated"

IMAGE_EXTENSIONS = {".png", ".jpg", ".jpeg", ".webp"}
```
**EN:** This section defines REPO_OWNER, REPO_NAME, BRANCH, DEFAULT_TARGET_DIR, IMAGE_EXTENSIONS, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 REPO_OWNER, REPO_NAME, BRANCH, DEFAULT_TARGET_DIR, IMAGE_EXTENSIONS，用于把可复用的默认值集中在模块顶部。

### Lines 52-66: Defines the `collect_images` routine / 定义 `collect_images` 例程
```python
def collect_images(source_dir, target_dir):
    """Collect image files from source_dir and return list of (repo_path, content) tuples."""
    files = []
    for entry in sorted(os.listdir(source_dir)):
        ext = os.path.splitext(entry)[1].lower()
        if ext not in IMAGE_EXTENSIONS:
            continue
        full_path = os.path.join(source_dir, entry)
        if not os.path.isfile(full_path):
            continue
        with open(full_path, "rb") as f:
            content = f.read()
        repo_path = f"{target_dir}/{entry}"
        files.append((repo_path, content))
    return files
```
**EN:** This block defines `collect_images`. Collect image files from source_dir and return list of (repo_path, content) tuples. It accepts 2 parameter(s): source_dir, target_dir. Internally it reads or writes files.
**CN:** 该代码块定义了 `collect_images`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：source_dir, target_dir。内部会读写文件。

### Lines 69-71: Defines the `git_blob_sha` routine / 定义 `git_blob_sha` 例程
```python
def git_blob_sha(content):
    header = f"blob {len(content)}\0".encode()
    return hashlib.sha1(header + content).hexdigest()
```
**EN:** This block defines `git_blob_sha`. It accepts 1 parameter(s): content. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `git_blob_sha`。它接收 1 个参数：content。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 74-90: Defines the `get_remote_blob_shas` routine / 定义 `get_remote_blob_shas` 例程
```python
def get_remote_blob_shas(repo_owner, repo_name, target_dir, token):
    url = (
        f"https://api.github.com/repos/{repo_owner}/{repo_name}/contents/"
        f"{target_dir}?ref={BRANCH}"
    )
    try:
        response = make_github_request(url, token)
    except HTTPError as e:
        if e.code == 404:
            return {}
        raise
    entries = json.loads(response)
    return {
        item["path"]: item["sha"]
        for item in entries
        if item.get("type") == "file" and "sha" in item
    }
```
**EN:** This block defines `get_remote_blob_shas`. It accepts 4 parameter(s): repo_owner, repo_name, target_dir, token. Internally it loads structured data.
**CN:** 该代码块定义了 `get_remote_blob_shas`。它接收 4 个参数：repo_owner, repo_name, target_dir, token。内部会加载结构化数据。

### Lines 93-98: Defines the `filter_changed_files` routine / 定义 `filter_changed_files` 例程
```python
def filter_changed_files(files, remote_blob_shas):
    return [
        (path, content)
        for path, content in files
        if remote_blob_shas.get(path) != git_blob_sha(content)
    ]
```
**EN:** This block defines `filter_changed_files`. It accepts 2 parameter(s): files, remote_blob_shas. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `filter_changed_files`。它接收 2 个参数：files, remote_blob_shas。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 101-198: Defines the `publish` routine / 定义 `publish` 例程
```python
def publish(source_dir, target_dir=None):
    target_dir = target_dir or DEFAULT_TARGET_DIR
    token = os.getenv("GITHUB_TOKEN")
    if not token:
        print("Error: GITHUB_TOKEN environment variable not set")
        sys.exit(1)

    files_to_upload = collect_images(source_dir, target_dir)
    if not files_to_upload:
        print(f"No image files found in {source_dir}")
        return

    print(
        f"Found {len(files_to_upload)} image(s) to upload to {REPO_OWNER}/{REPO_NAME}/{target_dir}"
    )

    # Verify token
    perm = verify_token_permissions(REPO_OWNER, REPO_NAME, token)
    if perm == "rate_limited":
        print("GitHub API rate-limited, skipping upload.")
        return
    if not perm:
        print("Token permission verification failed.")
        sys.exit(1)

    # Commit with retry (handle concurrent pushes)
    max_retries = 5
    for attempt in range(max_retries):
        try:
            branch_sha = get_branch_sha(REPO_OWNER, REPO_NAME, BRANCH, token)
            tree_sha = get_tree_sha(REPO_OWNER, REPO_NAME, branch_sha, token)
            remote_blob_shas = get_remote_blob_shas(
                REPO_OWNER, REPO_NAME, target_dir, token
            )
            changed_files = filter_changed_files(files_to_upload, remote_blob_shas)
            if not changed_files:
                print("No image changes to publish.")
                return

            try:
                tree_items = create_blobs(REPO_OWNER, REPO_NAME, changed_files, token)
            except Exception as e:
                if is_rate_limit_error(e):
                    print("Rate-limited during blob creation, skipping.")
                    return
                if is_permission_error(e):
                    print(
                        f"ERROR: Token lacks write permission to {REPO_OWNER}/{REPO_NAME}. "
                        "Update GH_PAT_FOR_NIGHTLY_CI_DATA with a token that has contents:write."
                    )
                    sys.exit(1)
                raise

            new_tree_sha = create_tree(
                REPO_OWNER, REPO_NAME, tree_sha, tree_items, token
            )
            if new_tree_sha == tree_sha:
                print("No tree changes to publish.")
                return

            commit_msg = f"diffusion-ci: update images in {target_dir} ({len(changed_files)} files) [automated]"
            commit_sha = create_commit(
                REPO_OWNER, REPO_NAME, new_tree_sha, branch_sha, commit_msg, token
            )
            update_branch_ref(REPO_OWNER, REPO_NAME, BRANCH, commit_sha, token)
            print(
                f"Successfully pushed {len(changed_files)} changed images (commit {commit_sha[:10]})"
            )
            return
        except Exception as e:
            if is_rate_limit_error(e):
                print("Rate-limited, skipping.")
                return
            if is_permission_error(e):
                print(f"ERROR: permission denied to {REPO_OWNER}/{REPO_NAME}")
                sys.exit(1)

            retryable = False
            if hasattr(e, "error_body"):
                if "Update is not a fast forward" in e.error_body:
                    retryable = True
                elif "Object does not exist" in e.error_body:
                    retryable = True

            if isinstance(e, HTTPError) and e.code in [422, 500, 502, 503, 504]:
                retryable = True

            if retryable and attempt < max_retries - 1:
                import time

                wait = 2**attempt
                print(
                    f"Attempt {attempt + 1}/{max_retries} failed, retrying in {wait}s..."
                )
                time.sleep(wait)
            else:
                print(f"Failed after {attempt + 1} attempts: {e}")
                raise
```
**EN:** This block defines `publish`. It accepts 2 parameter(s): source_dir, target_dir. Internally it emits status messages, controls process exit status.
**CN:** 该代码块定义了 `publish`。它接收 2 个参数：source_dir, target_dir。内部会输出状态信息、控制进程退出状态。

### Lines 201-215: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Publish diffusion GT images to GitHub"
    )
    parser.add_argument(
        "--source-dir", required=True, help="Directory containing GT images"
    )
    parser.add_argument(
        "--target-dir",
        required=False,
        default=None,
        help=f"Target directory in the remote repo (default: {DEFAULT_TARGET_DIR})",
    )
    args = parser.parse_args()
    publish(args.source_dir, args.target_dir)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数。

### Lines 218-219: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
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
- **Standard library / 标准库**: `argparse`, `hashlib`, `json`, `os`, `pathlib`, `sys`, `time`, `urllib`
- **Third-party modules / 第三方模块**: `publish_traces`
- **Repository-local imports / 仓库内导入**: `publish_traces`
