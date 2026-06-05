# check_doc_redirects.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/check_doc_redirects.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16 / 第 1-16 行

````python
#!/usr/bin/env python3
"""
Check for moved/deleted doc files and optionally auto-update redirects.py.

This script detects when documentation files in docs/source/ are moved or deleted
and verifies that corresponding redirects exist in docs/source/redirects.py.

Usage:
    # Check only (CI mode) - reports missing redirects
    python check_doc_redirects.py --base-ref origin/main

    # Auto-update redirects.py with missing entries
    python check_doc_redirects.py --base-ref origin/main --auto-fix
"""

from __future__ import annotations
````

- EN: This block imports dependencies such as `__future__`; iterates through collections, records, or generated items.
- CN: 该代码块导入当前模块运行所需的依赖；遍历集合、记录或生成项。

### Lines 18-34 / 第 18-34 行

````python
import argparse
import ast
import re
import subprocess
import sys
from pathlib import Path


def run_git(args: list[str]) -> str:
    """Run a git command and return stdout."""
    result = subprocess.run(["git"] + args, capture_output=True, text=True)
    return result.stdout.strip()


def get_doc_changes(base_ref: str) -> list[tuple[str, str, str | None]]:
    """
    Get moved/deleted doc files between base_ref and HEAD.
````

- EN: This block imports dependencies such as `argparse`, `ast`, `re`, `subprocess`; defines callable units such as `run_git`, `get_doc_changes`; parses command-line arguments for script entry points; returns computed values to callers.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；向调用方返回计算结果。

### Lines 36-50 / 第 36-50 行

````python
    Returns:
        List of (status, old_path, new_path) tuples.
        For deletions, new_path is None.
    """
    diff = run_git(
        [
            "diff",
            "--name-status",
            "-M",  # Enable rename detection
            f"{base_ref}...HEAD",
            "--",
            "docs/source/*.rst",
            "docs/source/*.md",
        ]
    )
````

- EN: This block iterates through collections, records, or generated items.
- CN: 该代码块遍历集合、记录或生成项。

### Lines 52-66 / 第 52-66 行

````python
    changes = []
    for line in diff.split("\n"):
        if not line:
            continue
        parts = line.split("\t")
        status = parts[0]

        # Renames: R100 (100% similar), R095 (95% similar), etc.
        if status.startswith("R") and len(parts) >= 3:
            changes.append((status, parts[1], parts[2]))
        # Deletions
        elif status == "D" and len(parts) >= 2:
            changes.append((status, parts[1], None))

    return changes
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 69-85 / 第 69-85 行

````python
def path_to_key(path: str) -> str:
    """
    Convert a file path to a redirect key.

    Example: docs/source/torch.compiler.rst -> torch.compiler
    Example: docs/source/user_guide/foo.rst -> user_guide/foo
    """
    return re.sub(r"\.(rst|md)$", "", path.replace("docs/source/", ""))


def path_to_url(path: str) -> str:
    """
    Convert a file path to an HTML URL for redirects.

    Example: docs/source/user_guide/foo.rst -> user_guide/foo.html
    """
    return re.sub(r"\.(rst|md)$", ".html", path.replace("docs/source/", ""))
````

- EN: This block defines callable units such as `path_to_key`, `path_to_url`; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 88-105 / 第 88-105 行

````python
def parse_existing_redirects(redirects_file: Path) -> dict[str, str]:
    """
    Parse redirects.py and return the existing redirects dictionary.

    Uses AST parsing for robustness.
    """
    content = redirects_file.read_text()
    tree = ast.parse(content)

    for node in ast.walk(tree):
        if isinstance(node, ast.Assign):
            for target in node.targets:
                if isinstance(target, ast.Name) and target.id == "redirects":
                    if isinstance(node.value, ast.Dict):
                        return {
                            k.value: v.value
                            for k, v in zip(node.value.keys, node.value.values)
                            if isinstance(k, ast.Constant)
````

- EN: This block defines callable units such as `parse_existing_redirects`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 106-116 / 第 106-116 行

````python
                            and isinstance(v, ast.Constant)
                        }
    return {}


def find_missing_redirects(
    changes: list[tuple[str, str, str | None]],
    existing: dict[str, str],
) -> list[tuple[str, str | None]]:
    """
    Find file changes that don't have corresponding redirects.
````

- EN: This block defines callable units such as `find_missing_redirects`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 118-134 / 第 118-134 行

````python
    Returns:
        List of (old_key, new_url) tuples. new_url is None for deletions.
    """
    missing = []
    for status, old_path, new_path in changes:
        old_key = path_to_key(old_path)
        if old_key not in existing:
            new_url = path_to_url(new_path) if new_path else None
            missing.append((old_key, new_url))
    return missing


def update_redirects_file(
    redirects_file: Path, new_entries: list[tuple[str, str]]
) -> None:
    """
    Add new redirect entries to redirects.py.
````

- EN: This block defines callable units such as `update_redirects_file`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 136-150 / 第 136-150 行

````python
    Inserts entries just before the closing brace of the redirects dict.
    """
    content = redirects_file.read_text()
    lines = content.split("\n")

    # Find the line with the closing brace
    insert_idx = None
    for i in range(len(lines) - 1, -1, -1):
        if lines[i].strip() == "}":
            insert_idx = i
            break

    if insert_idx is None:
        print("Error: Could not find closing brace in redirects.py", file=sys.stderr)
        sys.exit(1)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 152-161 / 第 152-161 行

````python
    # Generate new entries
    new_lines = ["    # Auto-generated redirects for moved files"]
    for old_key, new_url in new_entries:
        new_lines.append(f'    "{old_key}": "{new_url}",')

    # Insert the new entries before the closing brace
    lines = lines[:insert_idx] + new_lines + lines[insert_idx:]

    redirects_file.write_text("\n".join(lines))
    print(f"✅ Added {len(new_entries)} redirect(s) to {redirects_file}")
````

- EN: This block iterates through collections, records, or generated items.
- CN: 该代码块遍历集合、记录或生成项。

### Lines 164-181 / 第 164-181 行

````python
def main() -> None:
    parser = argparse.ArgumentParser(
        description="Check for missing doc redirects and optionally auto-fix"
    )
    parser.add_argument(
        "--base-ref",
        default="origin/main",
        help="Base git ref to compare against (default: origin/main)",
    )
    parser.add_argument(
        "--redirects-file",
        default="docs/source/redirects.py",
        help="Path to redirects.py (default: docs/source/redirects.py)",
    )
    parser.add_argument(
        "--auto-fix",
        action="store_true",
        help="Automatically add missing redirects for moved files",
````

- EN: This block defines callable units such as `main`; parses command-line arguments for script entry points; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；遍历集合、记录或生成项。

### Lines 182-196 / 第 182-196 行

````python
    )
    args = parser.parse_args()

    redirects_file = Path(args.redirects_file)
    if not redirects_file.exists():
        print(f"Error: {redirects_file} not found", file=sys.stderr)
        sys.exit(1)

    # Get file changes
    changes = get_doc_changes(args.base_ref)
    if not changes:
        print("✅ No doc files were moved or deleted")
        return

    print(f"Found {len(changes)} moved/deleted doc file(s)")
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 198-211 / 第 198-211 行

````python
    # Parse existing redirects
    existing = parse_existing_redirects(redirects_file)
    print(f"Found {len(existing)} existing redirect(s)")

    # Find missing redirects
    missing = find_missing_redirects(changes, existing)

    if not missing:
        print("✅ All moved/deleted doc files have corresponding redirects")
        return

    # Separate auto-fixable (moves with known destination) from manual (deletes)
    auto_fixable = [(k, v) for k, v in missing if v is not None]
    manual_needed = [(k, v) for k, v in missing if v is None]
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 213-226 / 第 213-226 行

````python
    # Auto-fix mode
    if args.auto_fix and auto_fixable:
        update_redirects_file(redirects_file, auto_fixable)

        if manual_needed:
            print(f"\n⚠️  {len(manual_needed)} deleted file(s) need manual redirects:")
            for old_key, _ in manual_needed:
                print(f"  • {old_key}")
            print("\nPlease add redirects for deleted files manually.")
            sys.exit(1)
        return

    # Report mode - show what's missing
    print("\n❌ Missing redirects detected!\n")
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 228-242 / 第 228-242 行

````python
    for old_key, new_url in missing:
        if new_url:
            print(f"  • MOVED: {old_key} → {new_url}")
        else:
            print(f"  • DELETED: {old_key} (needs manual redirect target)")

    if auto_fixable:
        print("\n📝 Suggested additions to docs/source/redirects.py:\n")
        for old_key, new_url in auto_fixable:
            print(f'    "{old_key}": "{new_url}",')
        print("\n💡 To auto-fix, run:")
        print(
            f"    python3 .github/scripts/check_doc_redirects.py "
            f"--base-ref {args.base_ref} --auto-fix"
        )
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 244-248 / 第 244-248 行

````python
    if manual_needed:
        print(f"\n⚠️  {len(manual_needed)} deleted file(s) need manual redirects.")
        print("Please determine appropriate redirect targets for deleted files.")

    sys.exit(1)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 251-252 / 第 251-252 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。
- EN: CLI entry points — the module exposes command-line arguments so developers or CI jobs can drive automation predictably.
  CN: 命令行入口——该模块暴露命令行参数，使开发者或 CI 作业能够以可预测方式驱动自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `__future__`, `argparse`, `ast`, `re`, `subprocess`, `sys`, `pathlib`
- Internal imports / 内部导入: none
- External imports / 外部导入: none
- Classes / 类: none
- Functions / 函数: `run_git`, `get_doc_changes`, `path_to_key`, `path_to_url`, `parse_existing_redirects`, `find_missing_redirects`, `update_redirects_file`, `main`
- Test entry points / 测试入口: none
