# lintrunner.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `scripts/lintrunner.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides repository maintenance scripts used for setup, validation, packaging, and developer automation.
- **用途 (CN)**: 提供仓库维护脚本，用于环境准备、校验、打包以及开发者自动化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行

````python
#!/usr/bin/env python3
"""
Wrapper script to run the isolated hook version of lintrunner.

This allows developers to easily run lintrunner (including with -a for auto-fixes)
using the same isolated environment that the pre-push hook uses, without having
to manually activate/deactivate virtual environments.

Usage:
    python scripts/lintrunner.py          # Check mode (same as git push)
    python scripts/lintrunner.py -a       # Auto-fix mode
    python scripts/lintrunner.py --help   # Show lintrunner help
````

- EN: This block iterates through collections, records, or generated items.
- CN: 该代码块遍历集合、记录或生成项。

### Lines 14-23 / 第 14-23 行

````python
This module also provides shared functionality for lintrunner hash management.
"""

from __future__ import annotations

import hashlib
import os
import subprocess
import sys
from pathlib import Path
````

- EN: This block imports dependencies such as `__future__`, `hashlib`, `os`, `subprocess`; iterates through collections, records, or generated items.
- CN: 该代码块导入当前模块运行所需的依赖；遍历集合、记录或生成项。

### Lines 26-37 / 第 26-37 行

````python
def find_repo_root() -> Path:
    """Find repository root using git."""
    try:
        result = subprocess.run(
            ["git", "rev-parse", "--show-toplevel"],
            capture_output=True,
            text=True,
            check=True,
        )
        return Path(result.stdout.strip())
    except subprocess.CalledProcessError:
        sys.exit("❌ Not in a git repository")
````

- EN: This block defines callable units such as `find_repo_root`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 40-46 / 第 40-46 行

````python
def compute_file_hash(path: Path) -> str:
    """Returns SHA256 hash of a file's contents."""
    hasher = hashlib.sha256()
    with path.open("rb") as f:
        while chunk := f.read(8192):
            hasher.update(chunk)
    return hasher.hexdigest()
````

- EN: This block defines callable units such as `compute_file_hash`; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 49-59 / 第 49-59 行

````python
def read_stored_hash(path: Path) -> str | None:
    if not path.exists():
        return None
    try:
        return path.read_text().strip()
    except Exception:
        return None


# Venv location - change this if the path changes
HOOK_VENV_PATH = ".git/hooks/linter/.venv"
````

- EN: This block defines callable units such as `read_stored_hash`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 62-70 / 第 62-70 行

````python
def get_hook_venv_path() -> Path:
    """Get the path to the hook virtual environment."""
    repo_root = find_repo_root()
    return repo_root / HOOK_VENV_PATH


def find_hook_venv() -> Path:
    """Locate the isolated hook virtual environment."""
    venv_dir = get_hook_venv_path()
````

- EN: This block defines callable units such as `get_hook_venv_path`, `find_hook_venv`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 72-78 / 第 72-78 行

````python
    if not venv_dir.exists():
        sys.exit(
            f"❌ Hook virtual environment not found at {venv_dir}\n"
            "   Please set this up by running: python scripts/setup_hooks.py"
        )

    return venv_dir
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 81-94 / 第 81-94 行

````python
def check_lintrunner_installed(venv_dir: Path) -> None:
    """Check if lintrunner is installed in the given venv, exit if not."""
    result = subprocess.run(
        [
            "uv",
            "pip",
            "show",
            "--python",
            str(venv_dir / "bin" / "python"),
            "lintrunner",
        ],
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL,
    )
````

- EN: This block defines callable units such as `check_lintrunner_installed`; branches on runtime conditions or configuration choices.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支。

### Lines 95-108 / 第 95-108 行

````python
    if result.returncode != 0:
        sys.exit(
            "❌ lintrunner is required but was not found in the hook environment. "
            "Please run `python scripts/setup_hooks.py` to reinstall."
        )
    print("✅ lintrunner is already installed")


def run_lintrunner(venv_dir: Path, args: list[str]) -> int:
    """Run lintrunner command in the specified venv and return exit code."""
    # Run lintrunner directly from the venv's bin directory with environment setup
    lintrunner_exe = venv_dir / "bin" / "lintrunner"
    cmd = [str(lintrunner_exe)] + args
    env = os.environ.copy()
````

- EN: This block defines callable units such as `run_lintrunner`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 110-117 / 第 110-117 行

````python
    # PATH: Ensures lintrunner can find other tools in the venv (like python, pip, etc.)
    env["PATH"] = str(venv_dir / "bin") + os.pathsep + env.get("PATH", "")
    # VIRTUAL_ENV: Tells tools like pip_init.py that we're in a venv (prevents --user flag issues)
    env["VIRTUAL_ENV"] = str(venv_dir)

    # Note: Progress tends to be slightly garbled due to terminal control sequences,
    # but functionality and final results will be correct
    return subprocess.call(cmd, env=env)
````

- EN: This block returns computed values to callers.
- CN: 该代码块向调用方返回计算结果。

### Lines 120-131 / 第 120-131 行

````python
def initialize_lintrunner_if_needed(venv_dir: Path) -> None:
    """Check if lintrunner needs initialization and run init if needed."""
    repo_root = find_repo_root()
    lintrunner_toml_path = repo_root / ".lintrunner.toml"
    initialized_hash_path = venv_dir / ".lintrunner_plugins_hash"

    if not lintrunner_toml_path.exists():
        print("⚠️ No .lintrunner.toml found. Skipping init.")
        return

    current_hash = compute_file_hash(lintrunner_toml_path)
    stored_hash = read_stored_hash(initialized_hash_path)
````

- EN: This block defines callable units such as `initialize_lintrunner_if_needed`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 133-140 / 第 133-140 行

````python
    if current_hash != stored_hash:
        print("🔁 Running `lintrunner init` …", file=sys.stderr)
        result = run_lintrunner(venv_dir, ["init"])
        if result != 0:
            sys.exit(f"❌ lintrunner init failed")
        initialized_hash_path.write_text(current_hash)
    else:
        print("✅ Lintrunner plugins already initialized and up to date.")
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。

### Lines 143-155 / 第 143-155 行

````python
def main() -> None:
    """Run lintrunner in the isolated hook environment."""
    venv_dir = find_hook_venv()
    python_exe = venv_dir / "bin" / "python"

    if not python_exe.exists():
        sys.exit(f"❌ Python executable not found at {python_exe}")

    try:
        print(f"🐍 Virtual env being used: {venv_dir}", file=sys.stderr)

        # 1. Ensure lintrunner binary is available in the venv
        check_lintrunner_installed(venv_dir)
````

- EN: This block defines callable units such as `main`; branches on runtime conditions or configuration choices.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支。

### Lines 157-169 / 第 157-169 行

````python
        # 2. Check for plugin updates and re-init if needed
        initialize_lintrunner_if_needed(venv_dir)

        # 3. Run lintrunner with any passed arguments and propagate its exit code
        args = sys.argv[1:]
        result = run_lintrunner(venv_dir, args)

        # If lintrunner failed and we're not already in auto-fix mode, suggest the wrapper
        if result != 0 and "-a" not in args:
            print(
                "\n💡 To auto-fix these issues, run: python scripts/lintrunner.py -a",
                file=sys.stderr,
            )
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 171-175 / 第 171-175 行

````python
        sys.exit(result)

    except KeyboardInterrupt:
        print("\n  Lintrunner interrupted by user (KeyboardInterrupt)", file=sys.stderr)
        sys.exit(1)  # Tell git push to fail
````

- EN: This block implements local helper logic for lintrunner.
- CN: 该代码块实现与 lintrunner 相关的局部辅助逻辑。

### Lines 178-179 / 第 178-179 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `__future__`, `hashlib`, `os`, `subprocess`, `sys`, `pathlib`
- Internal imports / 内部导入: none
- External imports / 外部导入: none
- Classes / 类: none
- Functions / 函数: `find_repo_root`, `compute_file_hash`, `read_stored_hash`, `get_hook_venv_path`, `find_hook_venv`, `check_lintrunner_installed`, `run_lintrunner`, `initialize_lintrunner_if_needed`, `main`
- Test entry points / 测试入口: none
