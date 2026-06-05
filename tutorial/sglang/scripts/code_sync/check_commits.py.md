# check_commits.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/code_sync/check_commits.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `check_commits` workflow in SGLang. It mainly handles CI orchestration, dependency installation. / 该Python 模块用于支撑 SGLang 中的 `check_commits` 流程，主要负责CI 编排、依赖安装。它属于 `code_sync` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
List commits in the private repo that need to be synced to the OSS repo.

NOTE:
1. This script resolves the git root automatically and can be run anywhere
   inside the repo.

This script will:
1. Find the most recent sync commit (message starts with
   "[Automated PR] Copy OSS code from commit").
2. Scan commits after that point and keep those that touch the configured paths.
3. Compare added diff lines in relevant files against OSS main.
4. Print a markdown summary with commit links and write it to GitHub Step Summary.

Usage:
python3 scripts/code_sync/check_commits.py
"""
```
**EN:** List commits in the private repo that need to be synced to the OSS repo.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 19-25: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import os
import shutil
import subprocess
import sys
from dataclasses import dataclass
from typing import Dict, List, Optional, Set, Tuple
```
**EN:** This block loads argparse, os, shutil, subprocess, sys, dataclasses, typing. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, os, shutil, subprocess, sys, dataclasses, typing。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 28-28: Top-level expr logic / 顶层 expr 逻辑
```python
sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 30-34: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
from utils import (  # noqa: E402
    FOLDER_NAMES,
    get_last_sync_commit,
    write_github_step_summary,
)
```
**EN:** This block loads utils. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 utils。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 37-40: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
private_repo = "your-org/sglang-private-repo"
oss_repo_url = "https://github.com/sgl-project/sglang.git"
oss_repo_branch = "main"
default_oss_repo_dir = ".oss_repo"
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 45-51: Declares the `CommitInfo` class / 声明 `CommitInfo` 类
```python
class CommitInfo:
    commit_hash: str
    subject: str
    commit_date: str
    relevant_files: List[str]
    synced_lines: int
    total_added_lines: int
```
**EN:** This block declares the `CommitInfo` class. Its body mainly serves as a namespace or lightweight data container.
**CN:** 该代码块声明了 `CommitInfo` 类。它的主体更像命名空间或轻量数据容器。

### Lines 54-57: Defines the `check_dependencies` routine / 定义 `check_dependencies` 例程
```python
def check_dependencies() -> None:
    """Check for required command-line tools."""
    if not shutil.which("git"):
        raise EnvironmentError("git is not installed or not in PATH.")
```
**EN:** This block defines `check_dependencies`. Check for required command-line tools. It takes no explicit parameters at the top level. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `check_dependencies`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 60-73: Defines the `get_repo_root` routine / 定义 `get_repo_root` 例程
```python
def get_repo_root() -> str:
    try:
        output = subprocess.run(
            ["git", "rev-parse", "--show-toplevel"],
            capture_output=True,
            text=True,
            check=True,
        ).stdout.strip()
    except subprocess.CalledProcessError as e:
        raise RuntimeError(f"Unable to determine git repo root: {e.stderr or e}") from e

    if not output:
        raise RuntimeError("Unable to determine git repo root.")
    return os.path.abspath(output)
```
**EN:** This block defines `get_repo_root`. It takes no explicit parameters at the top level. Internally it executes external commands.
**CN:** 该代码块定义了 `get_repo_root`。它在顶层定义中不接收显式参数。内部会执行外部命令。

### Lines 76-98: Defines the `get_repo_from_origin` routine / 定义 `get_repo_from_origin` 例程
```python
def get_repo_from_origin(repo_root: str) -> str:
    """Try to infer the repo slug (owner/name) from git remote.origin.url."""
    try:
        url = subprocess.run(
            ["git", "config", "--get", "remote.origin.url"],
            capture_output=True,
            text=True,
            check=True,
            cwd=repo_root,
        ).stdout.strip()
    except subprocess.CalledProcessError:
        return private_repo

    if url.startswith("git@github.com:"):
        repo = url.split("git@github.com:", 1)[1]
    elif url.startswith("https://github.com/"):
        repo = url.split("https://github.com/", 1)[1]
    else:
        return private_repo

    if repo.endswith(".git"):
        repo = repo[: -len(".git")]
    return repo or private_repo
```
**EN:** This block defines `get_repo_from_origin`. Try to infer the repo slug (owner/name) from git remote.origin.url. It accepts 1 parameter(s): repo_root. Internally it executes external commands.
**CN:** 该代码块定义了 `get_repo_from_origin`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：repo_root。内部会执行外部命令。

### Lines 101-105: Defines the `get_default_oss_repo_path` routine / 定义 `get_default_oss_repo_path` 例程
```python
def get_default_oss_repo_path(repo_root: str) -> str:
    env_path = os.environ.get("OSS_REPO_PATH")
    if env_path:
        return os.path.abspath(env_path)
    return os.path.abspath(os.path.join(repo_root, default_oss_repo_dir))
```
**EN:** This block defines `get_default_oss_repo_path`. It accepts 1 parameter(s): repo_root. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_default_oss_repo_path`。它接收 1 个参数：repo_root。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 108-139: Defines the `ensure_oss_repo` routine / 定义 `ensure_oss_repo` 例程
```python
def ensure_oss_repo(oss_repo_path: str, repo_url: str, branch: str) -> str:
    oss_repo_path = os.path.abspath(oss_repo_path)
    if os.path.exists(oss_repo_path) and not os.path.isdir(oss_repo_path):
        raise RuntimeError(f"OSS repo path is not a directory: {oss_repo_path}")

    if os.path.isdir(os.path.join(oss_repo_path, ".git")):
        try:
            subprocess.run(
                ["git", "-C", oss_repo_path, "rev-parse", "--is-inside-work-tree"],
                capture_output=True,
                text=True,
                check=True,
            )
        except subprocess.CalledProcessError as e:
            raise RuntimeError(
                f"OSS repo path exists but is not a git repo: {oss_repo_path}"
            ) from e

        subprocess.run(
            ["git", "-C", oss_repo_path, "fetch", "origin", branch, "--depth", "1"],
            check=True,
        )
        return oss_repo_path

    parent_dir = os.path.dirname(oss_repo_path)
    if parent_dir and not os.path.isdir(parent_dir):
        os.makedirs(parent_dir, exist_ok=True)
    subprocess.run(
        ["git", "clone", "--depth", "1", "--branch", branch, repo_url, oss_repo_path],
        check=True,
    )
    return oss_repo_path
```
**EN:** This block defines `ensure_oss_repo`. It accepts 3 parameter(s): oss_repo_path, repo_url, branch. Internally it checks file existence, executes external commands.
**CN:** 该代码块定义了 `ensure_oss_repo`。它接收 3 个参数：oss_repo_path, repo_url, branch。内部会检查文件是否存在、执行外部命令。

### Lines 142-155: Defines the `get_commits_since` routine / 定义 `get_commits_since` 例程
```python
def get_commits_since(repo_root: str, last_sync_hash: Optional[str]) -> List[str]:
    """Get commit hashes from last sync commit (exclusive) to HEAD."""
    try:
        if last_sync_hash:
            command = ["git", "rev-list", f"{last_sync_hash}..HEAD"]
        else:
            command = ["git", "rev-list", "HEAD"]
        result = subprocess.run(
            command, capture_output=True, text=True, check=True, cwd=repo_root
        ).stdout.strip()
        return [line for line in result.split("\n") if line]
    except subprocess.CalledProcessError as e:
        print(f"Error getting commit list: {e.stderr}")
        return []
```
**EN:** This block defines `get_commits_since`. Get commit hashes from last sync commit (exclusive) to HEAD. It accepts 2 parameter(s): repo_root, last_sync_hash. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `get_commits_since`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：repo_root, last_sync_hash。内部会输出状态信息、执行外部命令。

### Lines 158-170: Defines the `get_changed_files` routine / 定义 `get_changed_files` 例程
```python
def get_changed_files(repo_root: str, commit_hash: str) -> List[str]:
    try:
        output = subprocess.run(
            ["git", "diff-tree", "--no-commit-id", "--name-only", "-r", commit_hash],
            capture_output=True,
            text=True,
            check=True,
            cwd=repo_root,
        ).stdout.strip()
        return [line for line in output.split("\n") if line]
    except subprocess.CalledProcessError as e:
        print(f"Error getting changed files for {commit_hash}: {e.stderr}")
        return []
```
**EN:** This block defines `get_changed_files`. It accepts 2 parameter(s): repo_root, commit_hash. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `get_changed_files`。它接收 2 个参数：repo_root, commit_hash。内部会输出状态信息、执行外部命令。

### Lines 173-176: Defines the `is_relevant_path` routine / 定义 `is_relevant_path` 例程
```python
def is_relevant_path(changed_file: str, path_prefix: str) -> bool:
    if changed_file == path_prefix:
        return True
    return changed_file.startswith(f"{path_prefix}/")
```
**EN:** This block defines `is_relevant_path`. It accepts 2 parameter(s): changed_file, path_prefix. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `is_relevant_path`。它接收 2 个参数：changed_file, path_prefix。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 179-184: Defines the `get_relevant_files` routine / 定义 `get_relevant_files` 例程
```python
def get_relevant_files(changed_files: List[str]) -> List[str]:
    return [
        changed_file
        for changed_file in changed_files
        if any(is_relevant_path(changed_file, path) for path in FOLDER_NAMES)
    ]
```
**EN:** This block defines `get_relevant_files`. It accepts 1 parameter(s): changed_files. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_relevant_files`。它接收 1 个参数：changed_files。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 187-239: Defines the `get_added_lines_by_file` routine / 定义 `get_added_lines_by_file` 例程
```python
def get_added_lines_by_file(
    repo_root: str, commit_hash: str, relevant_files: List[str]
) -> Dict[str, List[str]]:
    if not relevant_files:
        return {}

    command = [
        "git",
        "show",
        "--no-color",
        "--unified=0",
        "--format=",
        commit_hash,
        "--",
    ] + relevant_files
    try:
        output = subprocess.run(
            command, capture_output=True, text=True, check=True, cwd=repo_root
        ).stdout
    except subprocess.CalledProcessError as e:
        print(f"Error getting diff for {commit_hash}: {e.stderr}")
        return {}

    added_lines: Dict[str, List[str]] = {path: [] for path in relevant_files}
    relevant_set = set(relevant_files)
    current_file: Optional[str] = None
    for line in output.splitlines():
        if line.startswith("diff --git "):
            current_file = None
            continue
        if line.startswith("+++ "):
            file_path = None
            if line.startswith("+++ b/"):
                file_path = line[6:]
            else:
                candidate = line[4:]
                if candidate == "/dev/null":
                    file_path = None
                elif candidate.startswith("b/") or candidate.startswith("a/"):
                    file_path = candidate[2:]
                else:
                    file_path = candidate

            if file_path in relevant_set:
                current_file = file_path
            else:
                current_file = None
            continue

        if current_file and line.startswith("+") and not line.startswith("+++ "):
            added_lines[current_file].append(line[1:])

    return added_lines
```
**EN:** This block defines `get_added_lines_by_file`. It accepts 3 parameter(s): repo_root, commit_hash, relevant_files. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `get_added_lines_by_file`。它接收 3 个参数：repo_root, commit_hash, relevant_files。内部会输出状态信息、执行外部命令。

### Lines 242-265: Defines the `get_oss_file_lines` routine / 定义 `get_oss_file_lines` 例程
```python
def get_oss_file_lines(
    oss_repo_path: str,
    oss_ref: str,
    file_path: str,
    cache: Dict[str, Optional[Set[str]]],
) -> Optional[Set[str]]:
    if file_path in cache:
        return cache[file_path]
    try:
        output = subprocess.run(
            ["git", "-C", oss_repo_path, "show", f"{oss_ref}:{file_path}"],
            capture_output=True,
            text=True,
            errors="replace",
            check=True,
        ).stdout
    except subprocess.CalledProcessError:
        cache[file_path] = None
        return None

    lines = output.splitlines()
    line_set = set(lines)
    cache[file_path] = line_set
    return line_set
```
**EN:** This block defines `get_oss_file_lines`. It accepts 4 parameter(s): oss_repo_path, oss_ref, file_path, cache. Internally it executes external commands.
**CN:** 该代码块定义了 `get_oss_file_lines`。它接收 4 个参数：oss_repo_path, oss_ref, file_path, cache。内部会执行外部命令。

### Lines 268-288: Defines the `count_synced_lines` routine / 定义 `count_synced_lines` 例程
```python
def count_synced_lines(
    added_lines_by_file: Dict[str, List[str]],
    oss_repo_path: str,
    oss_ref: str,
    oss_file_cache: Dict[str, Optional[Set[str]]],
) -> Tuple[int, int]:
    total_added_lines = 0
    synced_lines = 0
    for file_path, lines in added_lines_by_file.items():
        total_added_lines += len(lines)
        if not lines:
            continue
        oss_lines = get_oss_file_lines(
            oss_repo_path, oss_ref, file_path, oss_file_cache
        )
        if not oss_lines:
            continue
        for line in lines:
            if line in oss_lines:
                synced_lines += 1
    return synced_lines, total_added_lines
```
**EN:** This block defines `count_synced_lines`. It accepts 4 parameter(s): added_lines_by_file, oss_repo_path, oss_ref, oss_file_cache. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `count_synced_lines`。它接收 4 个参数：added_lines_by_file, oss_repo_path, oss_ref, oss_file_cache。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 291-306: Defines the `get_commit_summary` routine / 定义 `get_commit_summary` 例程
```python
def get_commit_summary(repo_root: str, commit_hash: str) -> Tuple[str, str]:
    """Return (subject, date) for a commit."""
    try:
        output = subprocess.run(
            ["git", "show", "-s", "--format=%s%x00%ad", "--date=short", commit_hash],
            capture_output=True,
            text=True,
            check=True,
            cwd=repo_root,
        ).stdout.strip()
        subject, commit_date = output.split("\x00", 1)
    except subprocess.CalledProcessError as e:
        print(f"Error getting commit subject for {commit_hash}: {e.stderr}")
        subject = "(unknown subject)"
        commit_date = "(unknown date)"
    return subject, commit_date
```
**EN:** This block defines `get_commit_summary`. Return (subject, date) for a commit. It accepts 2 parameter(s): repo_root, commit_hash. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `get_commit_summary`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：repo_root, commit_hash。内部会输出状态信息、执行外部命令。

### Lines 309-310: Defines the `format_files_list` routine / 定义 `format_files_list` 例程
```python
def format_files_list(relevant_files: List[str]) -> str:
    return "\n".join([f"- {file_path}" for file_path in relevant_files])
```
**EN:** This block defines `format_files_list`. It accepts 1 parameter(s): relevant_files. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `format_files_list`。它接收 1 个参数：relevant_files。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 313-327: Defines the `format_last_sync_block` routine / 定义 `format_last_sync_block` 例程
```python
def format_last_sync_block(
    repo: str, subject: str, commit_hash: str, commit_date: str
) -> str:
    short_hash = commit_hash[:9]
    commit_url = f"https://github.com/{repo}/commit/{commit_hash}"
    return "\n".join(
        [
            "## Last sync",
            "",
            f"#### {subject}",
            f"date: {commit_date}",
            f"commit: [{short_hash}]({commit_url})",
            "",
        ]
    )
```
**EN:** This block defines `format_last_sync_block`. It accepts 4 parameter(s): repo, subject, commit_hash, commit_date. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `format_last_sync_block`。它接收 4 个参数：repo, subject, commit_hash, commit_date。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 330-357: Defines the `format_commit_block` routine / 定义 `format_commit_block` 例程
```python
def format_commit_block(
    repo: str,
    subject: str,
    commit_hash: str,
    commit_date: str,
    relevant_files: List[str],
    synced_lines: int,
    total_added_lines: int,
) -> str:
    short_hash = commit_hash[:9]
    commit_url = f"https://github.com/{repo}/commit/{commit_hash}"
    files_str = format_files_list(relevant_files) if relevant_files else "- None"
    status_icon = "✅" if synced_lines == total_added_lines else "❌"
    status_line = (
        f"status: {status_icon} {synced_lines}/{total_added_lines} lines synced"
    )
    return "\n".join(
        [
            f"#### {subject}",
            status_line,
            f"date: {commit_date}",
            "files to sync:",
            files_str,
            "",
            f"commit: [{short_hash}]({commit_url})",
            "",
        ]
    )
```
**EN:** This block defines `format_commit_block`. It accepts 7 parameter(s): repo, subject, commit_hash, commit_date, relevant_files, synced_lines. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `format_commit_block`。它接收 7 个参数：repo, subject, commit_hash, commit_date, relevant_files, synced_lines。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 360-390: Defines the `format_output` routine / 定义 `format_output` 例程
```python
def format_output(
    repo: str,
    last_sync: Optional[Tuple[str, str, str]],
    commits: List[CommitInfo],
) -> str:
    lines: List[str] = []
    if last_sync:
        subject, commit_hash, commit_date = last_sync
        lines.append(format_last_sync_block(repo, subject, commit_hash, commit_date))
    else:
        lines.extend(["## Last sync", "", "No sync commit found.", ""])

    lines.extend(["## Commits to sync", ""])
    if not commits:
        lines.append("No commits need to be synced.")
        return "\n".join(lines) + "\n"

    for commit in commits:
        lines.append(
            format_commit_block(
                repo,
                commit.subject,
                commit.commit_hash,
                commit.commit_date,
                commit.relevant_files,
                commit.synced_lines,
                commit.total_added_lines,
            )
        )

    return "\n".join(lines)
```
**EN:** This block defines `format_output`. It accepts 3 parameter(s): repo, last_sync, commits. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `format_output`。它接收 3 个参数：repo, last_sync, commits。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 393-479: Defines the `main` routine / 定义 `main` 例程
```python
def main() -> None:
    parser = argparse.ArgumentParser(
        description="List commits in the private repo that need to be synced to OSS."
    )
    parser.add_argument(
        "--limit",
        type=int,
        default=0,
        help="Limit number of commits printed (0 means no limit).",
    )
    parser.add_argument(
        "--oss-repo-path",
        default=None,
        help="Path to OSS repo clone (default: $OSS_REPO_PATH or .oss_repo).",
    )
    parser.add_argument(
        "--oss-repo-url",
        default=oss_repo_url,
        help="OSS repo URL (default: https://github.com/sgl-project/sglang.git).",
    )
    parser.add_argument(
        "--oss-branch",
        default=oss_repo_branch,
        help="OSS repo branch to check (default: main).",
    )
    args = parser.parse_args()

    check_dependencies()
    repo_root = get_repo_root()
    oss_repo_path = (
        os.path.abspath(args.oss_repo_path)
        if args.oss_repo_path
        else get_default_oss_repo_path(repo_root)
    )

    repo = get_repo_from_origin(repo_root)
    last_sync_hash = get_last_sync_commit(repo_root)
    last_sync_block = None
    if last_sync_hash:
        last_sync_subject, last_sync_date = get_commit_summary(
            repo_root, last_sync_hash
        )
        last_sync_block = (last_sync_subject, last_sync_hash, last_sync_date)

    commits = get_commits_since(repo_root, last_sync_hash)
    if args.limit > 0:
        commits = commits[: args.limit]

    relevant_commit_inputs: List[Tuple[str, List[str]]] = []
    for commit_hash in commits:
        changed_files = get_changed_files(repo_root, commit_hash)
        if not changed_files:
            continue
        relevant_files = get_relevant_files(changed_files)
        if relevant_files:
            relevant_commit_inputs.append((commit_hash, relevant_files))

    relevant_commits: List[CommitInfo] = []
    if relevant_commit_inputs:
        oss_repo_path = ensure_oss_repo(
            oss_repo_path, args.oss_repo_url, args.oss_branch
        )
        oss_ref = f"origin/{args.oss_branch}"
        oss_file_cache: Dict[str, Optional[Set[str]]] = {}
        for commit_hash, relevant_files in relevant_commit_inputs:
            subject, commit_date = get_commit_summary(repo_root, commit_hash)
            added_lines_by_file = get_added_lines_by_file(
                repo_root, commit_hash, relevant_files
            )
            synced_lines, total_added_lines = count_synced_lines(
                added_lines_by_file, oss_repo_path, oss_ref, oss_file_cache
            )
            relevant_commits.append(
                CommitInfo(
                    commit_hash=commit_hash,
                    subject=subject,
                    commit_date=commit_date,
                    relevant_files=relevant_files,
                    synced_lines=synced_lines,
                    total_added_lines=total_added_lines,
                )
            )

    output = format_output(repo, last_sync_block, relevant_commits)
    print(output)
    if os.environ.get("GITHUB_STEP_SUMMARY"):
        write_github_step_summary(output)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、输出状态信息。

### Lines 482-483: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
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
- **Standard library / 标准库**: `argparse`, `dataclasses`, `os`, `shutil`, `subprocess`, `sys`, `typing`
- **Third-party modules / 第三方模块**: `utils`
- **External commands inferred from code / 从代码推断的外部命令**: `git`
