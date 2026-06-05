# copy_from_oss.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/code_sync/copy_from_oss.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `copy_from_oss` workflow in SGLang. It mainly handles CI orchestration, dependency installation. / 该Python 模块用于支撑 SGLang 中的 `copy_from_oss` 流程，主要负责CI 编排、依赖安装。它属于 `code_sync` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Sync code from OSS repo to the local repo and open a PR if changes exist.

NOTE:
1. You need to execute this script in the git root folder.
2. A GH_TOKEN environment variable is required to create the pull request.
  - see also https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

This script will:
1. Clone the sgl-project/sglang repository (or use a local copy).
2. Sync specified files and directories using rsync.
3. Check if the sync operation resulted in any changes.
4. If there are changes:
   a. Create a new branch.
   b. Commit and push the changes.
   c. Open a pull request using the GitHub CLI (gh).

Usage:
# Run the full sync and PR creation process
python3 scripts/copy_from_oss.py

# Perform a dry run without making any actual changes
python3 scripts/copy_from_oss.py --dry-run

# Use a local directory as the source instead of cloning
python3 scripts/copy_from_oss.py --local-dir ~/projects/sglang
"""
```
**EN:** Sync code from OSS repo to the local repo and open a PR if changes exist.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 29-35: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import datetime
import os
import shutil
import subprocess
import sys
import tempfile
```
**EN:** This block loads argparse, datetime, os, shutil, subprocess, sys, tempfile. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, datetime, os, shutil, subprocess, sys, tempfile。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 38-38: Top-level expr logic / 顶层 expr 逻辑
```python
sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 40-40: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
from utils import FOLDER_NAMES, write_github_step_summary  # noqa: E402
```
**EN:** This block loads utils. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 utils。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 43-43: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
private_repo = "your-org/sglang-private-repo"
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 47-53: Defines the `check_dependencies` routine / 定义 `check_dependencies` 例程
```python
def check_dependencies():
    """Check for required command-line tools."""
    if not shutil.which("git"):
        raise EnvironmentError("git is not installed or not in PATH.")
    if not shutil.which("gh"):
        raise EnvironmentError("GitHub CLI (gh) is not installed or not in PATH.")
    print("✅ All dependencies (git, gh) are available.")
```
**EN:** This block defines `check_dependencies`. Check for required command-line tools. It takes no explicit parameters at the top level. Internally it emits status messages.
**CN:** 该代码块定义了 `check_dependencies`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会输出状态信息。

### Lines 56-70: Defines the `checkout_main` routine / 定义 `checkout_main` 例程
```python
def checkout_main(dry_run):
    """Checkout to the main branch."""
    commands = [
        "git checkout main",
        "git reset --hard",
    ]
    for cmd in commands:
        print(f"Run: {cmd}")
        if not dry_run:
            try:
                subprocess.run(cmd, shell=True, check=True, capture_output=True)
            except subprocess.CalledProcessError as e:
                print(f"Git command failed: {e.stderr.decode()}")
                raise
    print("✅ Checkout the main branch.")
```
**EN:** This block defines `checkout_main`. Checkout to the main branch. It accepts 1 parameter(s): dry_run. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `checkout_main`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：dry_run。内部会输出状态信息、执行外部命令。

### Lines 73-119: Defines the `get_source_folder` routine / 定义 `get_source_folder` 例程
```python
def get_source_folder(args):
    """
    Prepare the source repository, either by cloning from GitHub or using a local directory.
    Returns the path to the source repo root, a temporary directory path (if created),
    and the short commit hash.
    """
    temp_dir = None
    if args.local_dir:
        oss_root = os.path.expanduser(args.local_dir)
        if not os.path.exists(oss_root):
            raise FileNotFoundError(
                f"Specified local directory {oss_root} does not exist."
            )
        print(f"Using local directory as the source: {oss_root}")
    else:
        temp_dir = tempfile.mkdtemp()
        oss_root = temp_dir
        print(f"Created temporary directory: {oss_root}")

        repo_url = "https://github.com/sgl-project/sglang.git"
        try:
            subprocess.run(
                [
                    "git",
                    "clone",
                    "--single-branch",
                    "--branch",
                    "main",
                    repo_url,
                    temp_dir,
                ],
                check=True,
                capture_output=True,
            )
            print(f"Successfully cloned repository to {temp_dir}")
        except subprocess.CalledProcessError as e:
            print(f"Error cloning repository: {e.stderr.decode()}")
            raise

    commit_hash = subprocess.run(
        ["git", "-C", oss_root, "rev-parse", "HEAD"],
        capture_output=True,
        text=True,
        check=True,
    ).stdout.strip()[:8]
    print(f"✅ Get source OSS code at commit: {commit_hash}")
    return oss_root, temp_dir, commit_hash
```
**EN:** This block defines `get_source_folder`. Prepare the source repository, either by cloning from GitHub or using a local directory. It accepts 1 parameter(s): args. Internally it checks file existence, emits status messages, executes external commands.
**CN:** 该代码块定义了 `get_source_folder`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：args。内部会检查文件是否存在、输出状态信息、执行外部命令。

### Lines 122-139: Defines the `sync_directories` routine / 定义 `sync_directories` 例程
```python
def sync_directories(oss_root, sync_paths, dry_run):
    """Sync specified directories from oss_root to current working directory."""
    rsync_commands = []
    for folder_name in sync_paths:
        target_name = f"{oss_root}/{folder_name}"
        src_name = "./" + "/".join(folder_name.split("/")[:-1])
        cmd = f"rsync -r --delete {target_name} {src_name}"
        rsync_commands.append(cmd)

    for cmd in rsync_commands:
        try:
            print(f"Run: {cmd}")
            if not dry_run:
                subprocess.run(cmd, shell=True, check=True)
        except subprocess.CalledProcessError as e:
            print(f"Error executing command '{cmd}': {e}")
            raise
    print(f"✅ Sync all folders.")
```
**EN:** This block defines `sync_directories`. Sync specified directories from oss_root to current working directory. It accepts 3 parameter(s): oss_root, sync_paths, dry_run. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `sync_directories`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：oss_root, sync_paths, dry_run。内部会输出状态信息、执行外部命令。

### Lines 142-146: Defines the `check_for_changes` routine / 定义 `check_for_changes` 例程
```python
def check_for_changes():
    """Check if there are any uncommitted git changes."""
    # This command exits with 1 if there are changes, 0 otherwise.
    result = subprocess.run(["git", "diff", "--quiet"])
    return result.returncode != 0
```
**EN:** This block defines `check_for_changes`. Check if there are any uncommitted git changes. It takes no explicit parameters at the top level. Internally it executes external commands.
**CN:** 该代码块定义了 `check_for_changes`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会执行外部命令。

### Lines 149-167: Defines the `create_and_push_branch` routine / 定义 `create_and_push_branch` 例程
```python
def create_and_push_branch(branch_name, commit_message, dry_run):
    """Create a new branch, commit all changes, and push to origin."""
    commands = [
        f"git checkout -b {branch_name}",
        "git config user.name 'github-actions[bot]'",
        "git config user.email 'github-actions[bot]@users.noreply.github.com'",
        "git add .",
        f"git commit -m '{commit_message}'",
        f"git push origin {branch_name} --force",
    ]
    print("\nCreating and pushing git branch...")
    for cmd in commands:
        print(f"Run: {cmd}")
        if not dry_run:
            try:
                subprocess.run(cmd, shell=True, check=True, capture_output=True)
            except subprocess.CalledProcessError as e:
                print(f"Git command failed: {e.stderr.decode()}")
                raise
```
**EN:** This block defines `create_and_push_branch`. Create a new branch, commit all changes, and push to origin. It accepts 3 parameter(s): branch_name, commit_message, dry_run. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `create_and_push_branch`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：branch_name, commit_message, dry_run。内部会输出状态信息、执行外部命令。

### Lines 170-210: Defines the `create_pull_request` routine / 定义 `create_pull_request` 例程
```python
def create_pull_request(branch_name, title, body, dry_run):
    """Create a pull request using the GitHub CLI."""
    gh_token = os.getenv("GH_TOKEN")
    if not gh_token:
        print(
            "\n⚠️ Warning: GH_TOKEN environment variable not set. Skipping PR creation."
        )
        if not dry_run:
            return

    print("\nCreating pull request...")
    command = [
        "gh",
        "pr",
        "create",
        "--base",
        "main",
        "--head",
        branch_name,
        "--repo",
        private_repo,
        "--title",
        title,
        "--body",
        body,
    ]
    print(f"Run: {' '.join(command)}")
    if not dry_run:
        env = os.environ.copy()
        env["GH_TOKEN"] = gh_token
        try:
            result = subprocess.run(
                command, check=True, capture_output=True, text=True, env=env
            )
            pr_url = result.stdout.strip()
            msg = f"✅ Successfully created pull request: {pr_url}"
            print(msg)
            write_github_step_summary(msg)
        except subprocess.CalledProcessError as e:
            print(f"Error creating pull request: {e.stderr}")
            raise
```
**EN:** This block defines `create_pull_request`. Create a pull request using the GitHub CLI. It accepts 4 parameter(s): branch_name, title, body, dry_run. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `create_pull_request`。其文档字符串说明了该逻辑的预期职责。它接收 4 个参数：branch_name, title, body, dry_run。内部会输出状态信息、执行外部命令。

### Lines 213-269: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Copy code from OSS and open a PR if changes are detected."
    )
    parser.add_argument(
        "--local-dir",
        type=str,
        help="Path to local SGLang directory to use instead of cloning from GitHub.",
    )
    parser.add_argument(
        "--dry-run",
        action="store_true",
        help="Dry run the script without executing git, rsync, or gh commands.",
    )
    args = parser.parse_args()

    check_dependencies()
    checkout_main(args.dry_run)

    oss_root, temp_dir, oss_commit = get_source_folder(args)

    try:
        # Sync directories
        sync_directories(oss_root, FOLDER_NAMES, args.dry_run)

        # Check for changes and create PR if necessary
        if not check_for_changes():
            msg = "😴 No changes detected. The code is already in sync."
            print(msg)
            write_github_step_summary(msg)
            return

        print("✅ Changes detected. Proceeding to create a PR.")

        current_date = datetime.datetime.now().strftime("%Y%m%d")
        branch_name = f"copy-from-oss-{oss_commit}-{current_date}"
        commit_message = f"Copy OSS code from {oss_commit} on {current_date}"
        pr_title = (
            f"[Automated PR] Copy OSS code from commit {oss_commit} on {current_date}"
        )
        pr_body = (
            f"Copy OSS code from https://github.com/sgl-project/sglang/commit/{oss_commit} on {current_date}."
            "\n\n---\n\n"
            "*This is an automated PR created by scripts/copy_from_oss.py.*"
        )

        create_and_push_branch(branch_name, commit_message, args.dry_run)
        create_pull_request(branch_name, pr_title, pr_body, args.dry_run)

    finally:
        # Remove temporary directory if it was created
        if temp_dir:
            try:
                shutil.rmtree(temp_dir)
                print(f"\nRemoved temporary directory: {temp_dir}")
            except OSError as e:
                print(f"Error removing temporary directory {temp_dir}: {e}")
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、输出状态信息。

### Lines 272-273: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Subprocess orchestration** / 子进程编排
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `datetime`, `os`, `shutil`, `subprocess`, `sys`, `tempfile`
- **Third-party modules / 第三方模块**: `utils`
- **External commands inferred from code / 从代码推断的外部命令**: `git`
