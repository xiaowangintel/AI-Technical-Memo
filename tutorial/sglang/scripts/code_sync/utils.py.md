# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/code_sync/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `utils` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `utils` 流程，主要负责CI 编排、测试执行。它属于 `code_sync` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Shared constants and helpers for code-sync scripts.
"""
```
**EN:** Shared constants and helpers for code-sync scripts.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 5-8: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import os
import re
import subprocess
from typing import Optional
```
**EN:** This block loads os, re, subprocess, typing. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 os, re, subprocess, typing。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 13-35: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
FOLDER_NAMES = [
    "3rdparty",
    "assets",
    "benchmark",
    "docker",
    "docs",
    "examples",
    "python/sglang/lang",
    "python/sglang/jit_kernel",
    "python/sglang/srt",
    "python/sglang/test",
    "python/sglang/utils.py",
    "python/sglang/README.md",
    "sgl-kernel",
    "test/manual",
    "test/registered",
    "test/srt",
    "test/README.md",
    "test/run_suite.py",
    "README.md",
]

SYNC_COMMIT_PREFIX = r"\[Automated PR\] Copy OSS code from commit"
```
**EN:** This section defines FOLDER_NAMES, SYNC_COMMIT_PREFIX, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 FOLDER_NAMES, SYNC_COMMIT_PREFIX，用于把可复用的默认值集中在模块顶部。

### Lines 39-45: Defines the `write_github_step_summary` routine / 定义 `write_github_step_summary` 例程
```python
def write_github_step_summary(content: str) -> None:
    """Append *content* to the GitHub Actions step summary (no-op outside CI)."""
    summary_path = os.environ.get("GITHUB_STEP_SUMMARY")
    if not summary_path:
        return
    with open(summary_path, "a") as f:
        f.write(content)
```
**EN:** This block defines `write_github_step_summary`. Append *content* to the GitHub Actions step summary (no-op outside CI). It accepts 1 parameter(s): content. Internally it reads or writes files.
**CN:** 该代码块定义了 `write_github_step_summary`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：content。内部会读写文件。

### Lines 48-88: Defines the `get_last_sync_commit` routine / 定义 `get_last_sync_commit` 例程
```python
def get_last_sync_commit(repo_root: Optional[str] = None) -> Optional[str]:
    """
    Find the most recent sync commit that copied from OSS.

    Returns the full private-repo commit hash, or None if not found.
    The match is restricted to commits whose **subject** starts with the
    sync prefix so that unrelated commits mentioning the phrase in their
    body are ignored.
    """
    subject_pattern = re.compile("^" + SYNC_COMMIT_PREFIX)

    try:
        cmd = [
            "git",
            "log",
            "--all",
            "--grep",
            SYNC_COMMIT_PREFIX,
            "--format=%H %s",
        ]
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            check=True,
            cwd=repo_root,
        ).stdout.strip()

        for line in result.splitlines():
            # Format: "<full_hash> <subject>"
            parts = line.split(" ", 1)
            if len(parts) != 2:
                continue
            commit_hash, subject = parts
            if subject_pattern.search(subject):
                return commit_hash

        return None
    except subprocess.CalledProcessError as e:
        print(f"Error finding last sync commit: {e.stderr}")
        return None
```
**EN:** This block defines `get_last_sync_commit`. Find the most recent sync commit that copied from OSS. It accepts 1 parameter(s): repo_root. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `get_last_sync_commit`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：repo_root。内部会输出状态信息、执行外部命令。

### Lines 91-136: Defines the `find_latest_oss_sync_commit` routine / 定义 `find_latest_oss_sync_commit` 例程
```python
def find_latest_oss_sync_commit(repo_root: Optional[str] = None) -> Optional[str]:
    """
    Search the private repo history for the latest commit whose **subject**
    matches "[Automated PR] Copy OSS code from commit {commit_id} on {date}"
    and return the embedded **OSS** commit hash.

    Returns the short OSS commit hash string, or None if not found.
    """
    oss_hash_pattern = re.compile("^" + SYNC_COMMIT_PREFIX + r" ([0-9a-f]+)")

    try:
        # --grep filters on the full message body, so we request subject-only
        # output and validate the pattern against the subject ourselves.
        result = subprocess.run(
            [
                "git",
                "log",
                "--all",
                "--grep",
                SYNC_COMMIT_PREFIX,
                "--pretty=%s",
            ],
            capture_output=True,
            text=True,
            check=True,
            cwd=repo_root,
        )

        for subject in result.stdout.strip().splitlines():
            m = oss_hash_pattern.search(subject)
            if m:
                oss_commit = m.group(1)
                print(
                    f"✅ Latest OSS sync commit found: {oss_commit} "
                    f"(from: {subject})"
                )
                return oss_commit

        print(
            "⚠️  No '[Automated PR] Copy OSS code from commit ...' " "found in history."
        )
        return None

    except subprocess.CalledProcessError as e:
        print(f"Error searching for OSS sync commits: {e.stderr.strip()}")
        return None
```
**EN:** This block defines `find_latest_oss_sync_commit`. Search the private repo history for the latest commit whose **subject** matches "[Automated PR] Copy OSS code from commit {commit_id} on {date}" and return the embedded **OSS** commit hash. It accepts 1 parameter(s): repo_root. Internally it emits status messages, executes external commands.
**CN:** 该代码块定义了 `find_latest_oss_sync_commit`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：repo_root。内部会输出状态信息、执行外部命令。

## Key Concepts / 关键概念
- **Subprocess orchestration** / 子进程编排
- **Environment management** / 环境管理
- **Git state inspection** / Git 状态检查
- **Container execution** / 容器执行
- **Performance benchmarking** / 性能基准测试

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `re`, `subprocess`, `typing`
- **External commands inferred from code / 从代码推断的外部命令**: `docker`, `git`
