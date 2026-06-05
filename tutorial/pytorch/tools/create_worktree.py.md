# create_worktree.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/create_worktree.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements a PyTorch developer-tool script that transforms metadata, validates state, or orchestrates workflow steps.
- **Purpose (CN)**: 实现一个 PyTorch 开发工具脚本，用于转换元数据、校验状态或编排工作流步骤。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```python
#!/usr/bin/env python3
"""Create or remove git worktrees with submodules cloned locally.

This avoids fetching submodules from remote, which is slow for large repos
like PyTorch. Instead, each submodule is cloned directly from the local
checkout's git object store, so no network access is needed.

Usage:
    python tools/create_worktree.py                  # pytorch-worktree-1
    python tools/create_worktree.py my-worktree      # custom name
    python tools/create_worktree.py remove my-worktree  # force-remove
"""
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 14-29
```python
import argparse
import configparser
import subprocess
import sys
from pathlib import Path


def get_repo_root() -> Path:
    result = subprocess.run(
        ["git", "rev-parse", "--show-toplevel"],
        capture_output=True,
        text=True,
        check=True,
    )
    return Path(result.stdout.strip())
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, configparser, subprocess, and 2 more. This chunk defines `get_repo_root`, which implements a focused step inside the pytorch tooling pipeline. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、configparser、subprocess 等共 5 项。 这一段定义了 `get_repo_root`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 30-40
```python

def get_existing_worktrees(repo_root: Path) -> list[str]:
    result = subprocess.run(
        ["git", "worktree", "list", "--porcelain"],
        capture_output=True,
        text=True,
        check=True,
        cwd=repo_root,
    )
    names = []
    for line in result.stdout.splitlines():
```
- **EN**: This chunk defines `get_existing_worktrees`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `get_existing_worktrees`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 41-49
```python
        if line.startswith("worktree "):
            names.append(Path(line.split(" ", 1)[1]).name)
    return names


def next_worktree_name(repo_root: Path) -> str:
    existing = get_existing_worktrees(repo_root)
    n = 1
    while True:
```
- **EN**: This chunk defines `next_worktree_name`, which implements a focused step inside the pytorch tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `next_worktree_name`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 50-58
```python
        name = f"pytorch-worktree-{n}"
        if name not in existing:
            return name
        n += 1


def parse_gitmodules(root: Path) -> list[dict[str, str]]:
    gitmodules = root / ".gitmodules"
    if not gitmodules.exists():
```
- **EN**: This chunk defines `parse_gitmodules`, which parses or loads structured input into tool-friendly data structures. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `parse_gitmodules`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 59-69
```python
        return []
    config = configparser.ConfigParser()
    config.read(gitmodules)
    modules = []
    for section in config.sections():
        if section.startswith('submodule "'):
            path = config.get(section, "path")
            url = config.get(section, "url")
            modules.append({"path": path, "url": url})
    return modules
```
- **EN**: This chunk continues `parse_gitmodules` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `parse_gitmodules`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 70-79
```python

def get_submodule_commit(parent_repo: Path, submodule_path: str) -> str | None:
    """Get the commit hash a parent repo expects for a submodule."""
    result = subprocess.run(
        ["git", "ls-tree", "HEAD", submodule_path],
        capture_output=True,
        text=True,
        cwd=parent_repo,
    )
    if result.returncode != 0 or not result.stdout.strip():
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_submodule_commit`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_submodule_commit`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 80-88
```python
        return None
    # format: <mode> <type> <hash>\t<path>
    parts = result.stdout.strip().split()
    return parts[2] if len(parts) >= 3 else None


def resolve_git_dir(submodule_worktree: Path) -> Path | None:
    """Resolve the actual git object directory for a submodule.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `resolve_git_dir`, which implements a focused step inside the pytorch tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `resolve_git_dir`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 89-100
```python
    Submodules can have .git as either a directory (standalone clone) or a
    file containing 'gitdir: <path>' (gitlink). This resolves to the actual
    git directory in both cases.
    """
    git_path = submodule_worktree / ".git"
    if not git_path.exists():
        return None
    if git_path.is_dir():
        return git_path
    # gitlink: read the target
    content = git_path.read_text().strip()
    if content.startswith("gitdir: "):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `resolve_git_dir` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `resolve_git_dir`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 101-118
```python
        target = content[len("gitdir: ") :]
        resolved = (submodule_worktree / target).resolve()
        if resolved.exists():
            return resolved
    return None


def clone_submodule_recursive(
    worktree_root: Path,
    source_root: Path,
    submodule_path: str,
    url: str,
    depth: int = 0,
) -> None:
    prefix = "  " * depth
    source_sub = source_root / submodule_path
    worktree_sub = worktree_root / submodule_path
```
- **EN**: This chunk defines `clone_submodule_recursive`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `clone_submodule_recursive`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 119-127
```python
    git_dir = resolve_git_dir(source_sub)
    if git_dir is None:
        print(f"{prefix}  skipping {submodule_path} (not initialized in source)")
        return

    commit = get_submodule_commit(source_root, submodule_path)

    print(f"{prefix}  cloning {submodule_path}...")
```
- **EN**: This chunk continues `clone_submodule_recursive` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `clone_submodule_recursive`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 128-144
```python
    # The worktree checkout creates empty dirs for submodule paths; remove them
    # so git clone can create the directory itself.
    if worktree_sub.exists():
        try:
            worktree_sub.rmdir()
        except OSError:
            pass

    # Clone from the resolved git directory — works for both standalone .git
    # dirs and gitlinks pointing into .git/modules/. No network needed.
    subprocess.run(
        ["git", "clone", str(git_dir), str(worktree_sub)],
        check=True,
        capture_output=True,
        text=True,
    )
```
- **EN**: This chunk continues `clone_submodule_recursive` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `clone_submodule_recursive`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 145-154
```python
    # Checkout the exact commit the parent repo expects.
    if commit:
        subprocess.run(
            ["git", "checkout", "--quiet", commit],
            cwd=worktree_sub,
            check=True,
            capture_output=True,
            text=True,
        )
```
- **EN**: This chunk continues `clone_submodule_recursive` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `clone_submodule_recursive`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 155-163
```python
    # Point the remote back to the real URL so future fetches work.
    subprocess.run(
        ["git", "remote", "set-url", "origin", url],
        cwd=worktree_sub,
        check=True,
        capture_output=True,
        text=True,
    )
```
- **EN**: This chunk continues `clone_submodule_recursive` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段延续了 `clone_submodule_recursive`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 164-172
```python
    # Register nested submodules in this repo's config so that
    # `git submodule status --recursive` recognizes them.
    subprocess.run(
        ["git", "submodule", "init"],
        cwd=worktree_sub,
        capture_output=True,
        text=True,
    )
```
- **EN**: This chunk continues `clone_submodule_recursive` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段延续了 `clone_submodule_recursive`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 173-185
```python
    # Recurse into nested submodules
    nested = parse_gitmodules(worktree_sub)
    for mod in nested:
        try:
            clone_submodule_recursive(
                worktree_sub, source_sub, mod["path"], mod["url"], depth + 1
            )
        except subprocess.CalledProcessError as e:
            print(
                f"{prefix}    WARNING: nested {mod['path']}: {e}",
                file=sys.stderr,
            )
```
- **EN**: This chunk continues `clone_submodule_recursive` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `clone_submodule_recursive`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 186-194
```python

def cmd_create(args: argparse.Namespace) -> None:
    repo_root = get_repo_root()
    parent_dir = Path(args.parent_dir) if args.parent_dir else repo_root.parent
    name = args.name or next_worktree_name(repo_root)
    worktree_path = parent_dir / name

    print(f"Creating worktree at {worktree_path}")
```
- **EN**: This chunk defines `cmd_create`, which implements a focused step inside the pytorch tooling pipeline. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `cmd_create`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 195-211
```python
    cmd = ["git", "worktree", "add", str(worktree_path), args.commit]
    subprocess.run(cmd, check=True, cwd=repo_root)

    modules = parse_gitmodules(repo_root)
    if modules:
        print(f"Cloning {len(modules)} submodules from local checkout...")
        for mod in modules:
            try:
                clone_submodule_recursive(
                    worktree_path, repo_root, mod["path"], mod["url"]
                )
            except subprocess.CalledProcessError as e:
                print(
                    f"  WARNING: failed to clone {mod['path']}: {e}",
                    file=sys.stderr,
                )
                if e.stderr:
```
- **EN**: This chunk continues `cmd_create` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `cmd_create`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 212-221
```python
                    print(f"    {e.stderr.strip()}", file=sys.stderr)

    print(f"\nWorktree ready at {worktree_path}")


def cmd_remove(args: argparse.Namespace) -> None:
    repo_root = get_repo_root()
    parent_dir = Path(args.parent_dir) if args.parent_dir else repo_root.parent
    worktree_path = parent_dir / args.name
```
- **EN**: This chunk defines `cmd_remove`, which implements a focused step inside the pytorch tooling pipeline. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `cmd_remove`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 222-230
```python
    print(f"Removing worktree at {worktree_path}")
    subprocess.run(
        ["git", "worktree", "remove", "--force", str(worktree_path)],
        check=True,
        cwd=repo_root,
    )
    print("Done.")
```
- **EN**: This chunk continues `cmd_remove` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段延续了 `cmd_remove`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 231-248
```python
def main() -> None:
    parser = argparse.ArgumentParser(
        description="Create or remove git worktrees with locally-cloned submodules."
    )
    subparsers = parser.add_subparsers(dest="command")

    # "create" subcommand (also the default when no subcommand given)
    create_parser = subparsers.add_parser(
        "create", help="Create a new worktree (default)"
    )
    create_parser.add_argument(
        "name",
        nargs="?",
        default=None,
        help="Worktree directory name (default: pytorch-worktree-$n)",
    )
    create_parser.add_argument(
        "--commit",
```
- **EN**: This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 249-257
```python
        default="HEAD",
        help="Commit/branch to check out in the worktree (default: HEAD)",
    )
    create_parser.add_argument(
        "--parent-dir",
        default=None,
        help="Parent directory for the worktree (default: parent of repo root)",
    )
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 258-269
```python
    # "remove" subcommand
    remove_parser = subparsers.add_parser("remove", help="Force-remove a worktree")
    remove_parser.add_argument(
        "name",
        help="Worktree directory name to remove",
    )
    remove_parser.add_argument(
        "--parent-dir",
        default=None,
        help="Parent directory of the worktree (default: parent of repo root)",
    )
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 270-278
```python
    args = parser.parse_args()

    # Default to "create" when no subcommand is given
    if args.command is None:
        args.command = "create"
        args.name = None
        args.commit = "HEAD"
        args.parent_dir = None
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 279-286
```python
    if args.command == "create":
        cmd_create(args)
    elif args.command == "remove":
        cmd_remove(args)


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **PyTorch tooling**
  - EN: This file belongs to the pytorch tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于PyTorch 工具链层，应结合同一子目录中的相邻脚本一起理解。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **get_repo_root**
  - EN: `get_repo_root` is one of the main local symbols exposed or implemented here.
  - CN: `get_repo_root` 是此处暴露或实现的主要局部符号之一。
- **get_existing_worktrees**
  - EN: `get_existing_worktrees` is one of the main local symbols exposed or implemented here.
  - CN: `get_existing_worktrees` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `argparse`, `configparser`, `subprocess`, `sys`, `pathlib`
- **Primary symbols in this file / 本文件核心符号**: `get_repo_root`, `get_existing_worktrees`, `next_worktree_name`, `parse_gitmodules`, `get_submodule_commit`, `resolve_git_dir`, `clone_submodule_recursive`, `cmd_create`, `cmd_remove`, `main`
