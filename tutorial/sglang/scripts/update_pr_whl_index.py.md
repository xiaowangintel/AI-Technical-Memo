# update_pr_whl_index.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/update_pr_whl_index.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `update_pr_whl_index` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `update_pr_whl_index` 流程，主要负责CI 编排、测试执行。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-9: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Update the wheel index for PR SGLang releases.

This script generates a single PyPI-compatible index.html file at pr/index.html
containing all PR builds, ordered by PR number and commit count (newest first).

Similar to update_nightly_whl_index.py but for PR builds.
"""
```
**EN:** Update the wheel index for PR SGLang releases.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 11-14: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import hashlib
import pathlib
import re
```
**EN:** This block loads argparse, hashlib, pathlib, re. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, hashlib, pathlib, re。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 17-23: Defines the `compute_sha256` routine / 定义 `compute_sha256` 例程
```python
def compute_sha256(file_path: pathlib.Path) -> str:
    """Compute SHA256 hash of a file."""
    sha256_hash = hashlib.sha256()
    with open(file_path, "rb") as f:
        for byte_block in iter(lambda: f.read(4096), b""):
            sha256_hash.update(byte_block)
    return sha256_hash.hexdigest()
```
**EN:** This block defines `compute_sha256`. Compute SHA256 hash of a file. It accepts 1 parameter(s): file_path. Internally it reads or writes files.
**CN:** 该代码块定义了 `compute_sha256`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：file_path。内部会读写文件。

### Lines 26-137: Defines the `update_wheel_index` routine / 定义 `update_wheel_index` 例程
```python
def update_wheel_index(
    pr_number: str, commit_hash: str, wheel_version: str, build_date: str
):
    """Update the wheel index for PR releases.

    Creates a single index at pr/index.html containing all PR builds.

    Args:
        pr_number: PR number (e.g., '123')
        commit_hash: Short git commit hash (e.g., 'c5f1e86')
        wheel_version: Full wheel version string (e.g., '0.5.6.dev7716+pr-123.gc5f1e86')
        build_date: Build date in YYYY-MM-DD format (e.g., '2025-12-13')
    """
    dist_dir = pathlib.Path("dist")
    whl_repo_dir = pathlib.Path("sgl-whl")

    if not dist_dir.exists():
        print(f"Warning: {dist_dir} does not exist, skipping index update")
        return

    # Base URL for wheels stored in GitHub Releases
    base_url = "https://github.com/sgl-project/whl/releases/download"
    release_tag = f"pr-{pr_number}-{build_date}-{commit_hash}"

    # Create pr directory structure following PEP 503
    # /pr/index.html -> links to sglang/
    # /pr/sglang/index.html -> contains wheel links
    pr_dir = whl_repo_dir / "pr"
    pr_dir.mkdir(parents=True, exist_ok=True)

    sglang_dir = pr_dir / "sglang"
    sglang_dir.mkdir(parents=True, exist_ok=True)

    root_index = pr_dir / "index.html"
    package_index = sglang_dir / "index.html"

    print(f"\nUpdating PR wheel index")
    print(f"  Root index: {root_index}")
    print(f"  Package index: {package_index}")

    # Read existing package index if it exists
    existing_links = []
    if package_index.exists():
        with open(package_index, "r") as f:
            content = f.read()
            # Extract existing links (skip header and HTML boilerplate)
            existing_links = [
                line for line in content.split("\n") if line.startswith("<a href=")
            ]

    # Generate new links for current wheels
    new_links = []
    for wheel_path in sorted(dist_dir.glob("*.whl")):
        try:
            filename = wheel_path.name
            sha256 = compute_sha256(wheel_path)

            # URL format: {base_url}/{release_tag}/{filename}#sha256={hash}
            wheel_url = f"{base_url}/{release_tag}/{filename}#sha256={sha256}"
            link = f'<a href="{wheel_url}">{filename}</a><br>'

            new_links.append(link)
            print(f"  Added: {filename}")
        except Exception as e:
            print(f"  Error processing {wheel_path.name}: {e}")
            continue

    if not new_links:
        print("  No new wheels to add")
        return

    # Combine existing and new links (new links first for latest)
    all_links = new_links + existing_links

    # Remove duplicates while preserving order (newer first)
    seen = set()
    unique_links = []
    for link in all_links:
        # Extract filename from link to check for duplicates
        filename_match = re.search(r">([^<]+\.whl)</a>", link)
        if filename_match:
            filename = filename_match.group(1)
            if filename not in seen:
                seen.add(filename)
                unique_links.append(link)

    # Write root index (links to sglang package directory)
    with open(root_index, "w") as f:
        f.write("<!DOCTYPE html>\n")
        f.write('<a href="sglang/">sglang</a>\n')

    print(f"  Written root index: {root_index}")

    # Write package index in minimal format
    with open(package_index, "w") as f:
        f.write("<!DOCTYPE html>\n")
        f.write("<h1>SGLang PR Wheels</h1>\n")
        # Write links only
        f.write("\n".join(unique_links))
        f.write("\n")

    print(f"  Written {len(unique_links)} total wheels to {package_index}")
    print(f"\nDone! Users can install with:")
    print(
        f"  pip install sglang --pre --extra-index-url https://sgl-project.github.io/whl/pr/"
    )
    print(f"\nOr install specific PR #{pr_number} wheel directly:")
    if new_links:
        first_wheel_match = re.search(r'href="([^"]+)"', new_links[0])
        if first_wheel_match:
            wheel_url = first_wheel_match.group(1).split("#")[0]  # Remove sha256 hash
            print(f"  pip install {wheel_url}")
```
**EN:** This block defines `update_wheel_index`. Update the wheel index for PR releases. It accepts 4 parameter(s): pr_number, commit_hash, wheel_version, build_date. Internally it checks file existence, reads or writes files, touches filesystem paths, creates directories.
**CN:** 该代码块定义了 `update_wheel_index`。其文档字符串说明了该逻辑的预期职责。它接收 4 个参数：pr_number, commit_hash, wheel_version, build_date。内部会检查文件是否存在、读写文件、处理文件系统路径、创建目录。

### Lines 140-179: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Update wheel index for PR SGLang releases"
    )
    parser.add_argument(
        "--pr-number",
        type=str,
        required=True,
        help="PR number (e.g., '123')",
    )
    parser.add_argument(
        "--commit-hash",
        type=str,
        required=True,
        help="Short git commit hash (e.g., 'c5f1e86')",
    )
    parser.add_argument(
        "--wheel-version",
        type=str,
        required=True,
        help="Full wheel version string (e.g., '0.5.6.dev7716+pr-123.gc5f1e86')",
    )
    parser.add_argument(
        "--build-date",
        type=str,
        required=True,
        help="Build date in YYYY-MM-DD format (e.g., '2025-12-13')",
    )

    args = parser.parse_args()

    print(f"Updating PR wheel index")
    print(f"  PR: #{args.pr_number}")
    print(f"  Commit: {args.commit_hash}")
    print(f"  Version: {args.wheel_version}")
    print(f"  Build date: {args.build_date}")

    update_wheel_index(
        args.pr_number, args.commit_hash, args.wheel_version, args.build_date
    )
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、输出状态信息。

### Lines 182-183: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `hashlib`, `pathlib`, `re`
