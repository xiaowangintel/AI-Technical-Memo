# update_nightly_whl_index.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/update_nightly_whl_index.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `update_nightly_whl_index` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `update_nightly_whl_index` 流程，主要负责CI 编排、测试执行。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-11: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Update the wheel index for nightly SGLang releases.

This script generates a PyPI-compatible index.html file at cu{version}/sglang/index.html
containing all historical nightly builds, ordered by commit count (newest first).

The CUDA version is specified via the --cuda-version argument.

Reference: https://github.com/flashinfer-ai/flashinfer/blob/v0.2.0/scripts/update_whl_index.py
"""
```
**EN:** Update the wheel index for nightly SGLang releases.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 13-16: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import hashlib
import pathlib
import re
```
**EN:** This block loads argparse, hashlib, pathlib, re. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, hashlib, pathlib, re。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 19-25: Defines the `compute_sha256` routine / 定义 `compute_sha256` 例程
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

### Lines 28-156: Defines the `update_wheel_index` routine / 定义 `update_wheel_index` 例程
```python
def update_wheel_index(
    commit_hash: str, nightly_version: str, cuda_version: str, build_date: str = None
):
    """Update the wheel index for nightly releases.

    Creates an index at cu{version}/sglang/index.html containing all historical nightlies.

    Args:
        commit_hash: Short git commit hash (e.g., 'c5f1e86')
        nightly_version: Full nightly version string (e.g., '0.5.6.post1.dev7716+gc5f1e86')
        cuda_version: CUDA version string (e.g., '129' or '130')
        build_date: Build date in YYYY-MM-DD format (e.g., '2025-12-13')
    """
    dist_dir = pathlib.Path("dist")
    whl_repo_dir = pathlib.Path("sgl-whl")

    if not dist_dir.exists():
        raise FileNotFoundError(
            f"{dist_dir} does not exist — the download-artifact step did not "
            f"populate it; refusing to silently no-op the index update"
        )

    # Format CUDA version with 'cu' prefix if not already present
    if not cuda_version.startswith("cu"):
        cuda_version = f"cu{cuda_version}"
    print(f"Using CUDA version: {cuda_version}")

    # Base URL for wheels stored in GitHub Releases
    base_url = "https://github.com/sgl-project/whl/releases/download"
    # Use date-based tag if build_date is provided, otherwise fall back to commit-only
    if build_date:
        release_tag = f"nightly-{build_date}-{commit_hash}"
    else:
        release_tag = f"nightly-{commit_hash}"

    # Create directory structure following PEP 503
    # /cu{version}/index.html -> links to sglang/ and sgl-kernel/
    # /cu{version}/sglang/index.html -> contains wheel links
    cuda_dir = whl_repo_dir / cuda_version
    cuda_dir.mkdir(parents=True, exist_ok=True)

    sglang_dir = cuda_dir / "sglang"
    sglang_dir.mkdir(parents=True, exist_ok=True)

    root_index = cuda_dir / "index.html"
    package_index = sglang_dir / "index.html"

    print(f"\nUpdating nightly wheel index")
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
        filename = wheel_path.name
        sha256 = compute_sha256(wheel_path)

        # URL format: {base_url}/{release_tag}/{filename}#sha256={hash}
        wheel_url = f"{base_url}/{release_tag}/{filename}#sha256={sha256}"
        link = f'<a href="{wheel_url}">{filename}</a><br>'

        new_links.append(link)
        print(f"  Added: {filename}")

    if not new_links:
        raise RuntimeError(
            f"No wheels found in {dist_dir} — index update for {cuda_version} "
            f"would be a no-op; failing loudly instead of pushing an empty change"
        )

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

    # Update root index to include both sgl-kernel and sglang
    # Read existing packages from root index if it exists
    existing_packages = set()
    if root_index.exists():
        with open(root_index, "r") as f:
            content = f.read()
            # Extract existing package links
            for match in re.finditer(r'<a href="([^"]+)/">', content):
                existing_packages.add(match.group(1))

    # Add sglang to the package list
    existing_packages.add("sglang")

    # Write root index with all packages (sorted for consistency)
    with open(root_index, "w") as f:
        f.write("<!DOCTYPE html>\n")
        for pkg in sorted(existing_packages):
            f.write(f'<a href="{pkg}/">{pkg}</a>\n')

    print(f"  Written root index: {root_index} (packages: {sorted(existing_packages)})")

    # Write package index in minimal format (matching production sgl-kernel index)
    with open(package_index, "w") as f:
        f.write("<!DOCTYPE html>\n")
        f.write(f"<h1>SGLang Nightly Wheels ({cuda_version})</h1>\n")
        # Write links only
        f.write("\n".join(unique_links))
        f.write("\n")

    print(f"  Written {len(unique_links)} total wheels to {package_index}")
    print(f"\nDone! Users can install with:")
    print(
        f"  pip install sglang --pre --extra-index-url https://sgl-project.github.io/whl/{cuda_version}/"
    )
```
**EN:** This block defines `update_wheel_index`. Update the wheel index for nightly releases. It accepts 4 parameter(s): commit_hash, nightly_version, cuda_version, build_date. Internally it creates directories, checks file existence, reads or writes files, touches filesystem paths.
**CN:** 该代码块定义了 `update_wheel_index`。其文档字符串说明了该逻辑的预期职责。它接收 4 个参数：commit_hash, nightly_version, cuda_version, build_date。内部会创建目录、检查文件是否存在、读写文件、处理文件系统路径。

### Lines 159-199: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Update wheel index for nightly SGLang releases"
    )
    parser.add_argument(
        "--commit-hash",
        type=str,
        required=True,
        help="Short git commit hash (e.g., 'c5f1e86')",
    )
    parser.add_argument(
        "--nightly-version",
        type=str,
        required=True,
        help="Full nightly version string (e.g., '0.5.6.post1.dev7716+gc5f1e86')",
    )
    parser.add_argument(
        "--cuda-version",
        type=str,
        default="130",
        help="CUDA version (e.g., '129' or '130'). Defaults to '130'.",
    )
    parser.add_argument(
        "--build-date",
        type=str,
        required=False,
        help="Build date in YYYY-MM-DD format (e.g., '2025-12-13')",
    )

    args = parser.parse_args()

    print(f"Updating nightly wheel index")
    print(f"  Commit: {args.commit_hash}")
    print(f"  Version: {args.nightly_version}")
    print(f"  CUDA version: {args.cuda_version}")
    if args.build_date:
        print(f"  Build date: {args.build_date}")

    update_wheel_index(
        args.commit_hash, args.nightly_version, args.cuda_version, args.build_date
    )
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、输出状态信息。

### Lines 202-203: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
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
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `hashlib`, `pathlib`, `re`
