# analyze_whl_kernel_sizes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/analyze_whl_kernel_sizes.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module organizes Python logic for `analyze whl kernel sizes`, including reusable helpers, configuration, or integration code. / 该模块组织与 `analyze whl kernel sizes` 相关的 Python 逻辑，包括可复用辅助函数、配置或集成代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and module setup
````python
import argparse
import json
import os
import shutil
import subprocess
import sys
import tempfile
import zipfile
from pathlib import Path
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 12-14: `extract_whl` definition
````python
def extract_whl(whl_file, extract_dir):
    with zipfile.ZipFile(whl_file, "r") as zip_ref:
        zip_ref.extractall(extract_dir)
````
**EN:** This section defines `extract_whl` and implements the core logic associated with extract whl.
**CN:** 该部分定义 `extract_whl`，并实现与 extract whl 相关的核心逻辑。

### Lines 17-27: `find_binary_files` definition
````python
def find_binary_files(extract_dir):
    binary_files = []
    extract_path = Path(extract_dir)

    for so_file in extract_path.rglob("*.so"):
        binary_files.append(str(so_file))

    for cubin_file in extract_path.rglob("*.cubin"):
        binary_files.append(str(cubin_file))

    return sorted(binary_files)
````
**EN:** This section defines `find_binary_files` and implements the core logic associated with find binary files.
**CN:** 该部分定义 `find_binary_files`，并实现与 find binary files 相关的核心逻辑。

### Lines 30-48: `run_cubloaty` definition
````python
def run_cubloaty(binary_file):
    result = subprocess.run(
        ["cubloaty", binary_file, "--format", "json"],
        capture_output=True,
        text=True,
        timeout=60,
    )

    if result.returncode != 0:
        if (
            "No CUDA binary sections found" in result.stderr
            or "does not contain device code" in result.stderr
        ):
            return {}
        raise subprocess.CalledProcessError(
            result.returncode, result.args, result.stdout, result.stderr
        )

    return json.loads(result.stdout)
````
**EN:** This section defines `run_cubloaty` and implements the core logic associated with run cubloaty.
**CN:** 该部分定义 `run_cubloaty`，并实现与 run cubloaty 相关的核心逻辑。

### Lines 51-83: `analyze_whl` definition
````python
def analyze_whl(whl_file):
    temp_dir = tempfile.mkdtemp(prefix="sgl_kernel_analysis_")

    try:
        extract_whl(whl_file, temp_dir)

        binary_files = find_binary_files(temp_dir)
        if not binary_files:
            print(f"No .so or .cubin files found in {whl_file}")
            return []

        all_kernels = []

        for binary_file in binary_files:
            file_name = os.path.basename(binary_file)
            data = run_cubloaty(binary_file)

            if not data or "kernels" not in data:
                continue
            for kernel in data["kernels"]:
                all_kernels.append(
                    {
                        "file": file_name,
                        "name": kernel.get("name", "unknown"),
                        "size": kernel.get("size", 0),
                        "size_kb": kernel.get("size", 0) / 1024,
                        "size_mb": kernel.get("size", 0) / 1024 / 1024,
                    }
                )
        return all_kernels

    finally:
        shutil.rmtree(temp_dir, ignore_errors=True)
````
**EN:** This section defines `analyze_whl` and implements the core logic associated with analyze whl.
**CN:** 该部分定义 `analyze_whl`，并实现与 analyze whl 相关的核心逻辑。

### Lines 86-89: `extract_kernel_prefix` definition
````python
def extract_kernel_prefix(kernel_name):
    if "<" in kernel_name:
        return kernel_name.split("<")[0]
    return kernel_name
````
**EN:** This section defines `extract_kernel_prefix` and implements the core logic associated with extract kernel prefix.
**CN:** 该部分定义 `extract_kernel_prefix`，并实现与 extract kernel prefix 相关的核心逻辑。

### Lines 92-195: `generate_report` definition
````python
def generate_report(all_kernels, output_file):
    if not all_kernels:
        print("No kernels found")
        return

    sorted_kernels = sorted(all_kernels, key=lambda x: x["size"], reverse=True)
    total_size = sum(k["size"] for k in all_kernels)
    total_size_mb = total_size / 1024 / 1024

    from collections import defaultdict

    kernel_groups = defaultdict(lambda: {"size": 0, "count": 0})
    for kernel in all_kernels:
        prefix = extract_kernel_prefix(kernel["name"])
        kernel_groups[prefix]["size"] += kernel["size"]
        kernel_groups[prefix]["count"] += 1

    sorted_groups = sorted(
        kernel_groups.items(), key=lambda x: x[1]["size"], reverse=True
    )

    lines = []
    lines.append("=" * 140)
    lines.append("CUDA Kernel Size Analysis")
    lines.append("=" * 140)
    lines.append("")
    lines.append(f"Total kernels: {len(all_kernels)}")
    lines.append(f"Total size: {total_size_mb:.2f} MB ({total_size:,} bytes)")
    lines.append(f"Average kernel size: {total_size / len(all_kernels) / 1024:.2f} KB")
    lines.append("")

    lines.append("=" * 140)
    lines.append("Kernel Groups (by name prefix) - Top 20")
    lines.append("=" * 140)
    lines.append(
        f"{'Rank':<6} {'Kernel Prefix':<80} {'Count':<8} {'Total (MB)':<12} {'%':<8}"
    )
    lines.append("-" * 140)

    TOP_N = 20
    for i, (prefix, stats) in enumerate(sorted_groups[:TOP_N], 1):
        percentage = (stats["size"] / total_size * 100) if total_size > 0 else 0
        size_mb = stats["size"] / 1024 / 1024

        display_prefix = prefix
        if len(display_prefix) > 77:
            display_prefix = display_prefix[:74] + "..."

        lines.append(
            f"{i:<6} {display_prefix:<80} {stats['count']:<8} {size_mb:<12.2f} {percentage:<8.2f}"
        )

    if len(sorted_groups) > TOP_N:
        other_size = sum(stats["size"] for _, stats in sorted_groups[TOP_N:])
        other_count = sum(stats["count"] for _, stats in sorted_groups[TOP_N:])
        other_percentage = (other_size / total_size * 100) if total_size > 0 else 0
        other_size_mb = other_size / 1024 / 1024

        lines.append(
            f"{'Other':<6} {'(remaining ' + str(len(sorted_groups) - TOP_N) + ' kernel groups)':<80} "
            f"{other_count:<8} {other_size_mb:<12.2f} {other_percentage:<8.2f}"
        )

    lines.append("")
    lines.append("=" * 140)
    lines.append("Individual Kernels (sorted by size) - Top 20")
    lines.append("=" * 140)
    lines.append(
        f"{'Rank':<6} {'File':<40} {'Kernel Name':<70} {'Size (KB)':<12} {'Size (MB)':<12} {'%':<8}"
    )
    lines.append("-" * 140)

    for i, kernel in enumerate(sorted_kernels[:TOP_N], 1):
        percentage = (kernel["size"] / total_size * 100) if total_size > 0 else 0
        kernel_name = kernel["name"]
        if len(kernel_name) > 67:
            kernel_name = kernel_name[:64] + "..."

        file_name = kernel["file"]
        if len(file_name) > 37:
            file_name = file_name[:34] + "..."

        lines.append(
            f"{i:<6} {file_name:<40} {kernel_name:<70} "
            f"{kernel['size_kb']:<12.2f} {kernel['size_mb']:<12.4f} {percentage:<8.2f}"
        )

    if len(sorted_kernels) > TOP_N:
        other_size = sum(k["size"] for k in sorted_kernels[TOP_N:])
        other_count = len(sorted_kernels) - TOP_N
        other_percentage = (other_size / total_size * 100) if total_size > 0 else 0
        other_size_kb = other_size / 1024
        other_size_mb = other_size / 1024 / 1024

        lines.append(
            f"{'Other':<6} {'(remaining ' + str(other_count) + ' kernels)':<40} "
            f"{'':<70} {other_size_kb:<12.2f} {other_size_mb:<12.4f} {other_percentage:<8.2f}"
        )

    report_text = "\n".join(lines)

    with open(output_file, "w") as f:
        f.write(report_text)
    print(f"Report saved to: {output_file}")
````
**EN:** This section defines `generate_report` and implements the core logic associated with generate report.
**CN:** 该部分定义 `generate_report`，并实现与 generate report 相关的核心逻辑。

### Lines 198-217: `main` definition
````python
def main():
    parser = argparse.ArgumentParser(
        description="Analyze CUDA kernel sizes in sglang-kernel wheel files"
    )
    parser.add_argument("whl", type=str, help="Path to whl file")
    parser.add_argument(
        "--output", type=str, default="kernel_analysis.txt", help="Output report file"
    )
    args = parser.parse_args()

    if not os.path.exists(args.whl):
        print(f"Error: {args.whl} not found")
        sys.exit(1)

    all_kernels = analyze_whl(args.whl)

    if all_kernels:
        generate_report(all_kernels, args.output)
    else:
        print("No kernel information extracted")
````
**EN:** This section defines `main` and implements the core logic associated with main.
**CN:** 该部分定义 `main`，并实现与 main 相关的核心逻辑。

### Lines 220-221: Command-line entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Reusable Python module / 可复用 Python 模块
- **Primary symbols / 主要符号**: `extract_whl`, `find_binary_files`, `run_cubloaty`, `analyze_whl`, `extract_kernel_prefix`, `generate_report`, `main`
- **Core technologies / 核心技术**: CLI argument parsing / 命令行参数解析, toolchain subprocess control / 工具链子进程控制
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `argparse`, `collections`, `json`, `os`, `pathlib`, `shutil`, `subprocess`, `sys`, `tempfile`, `zipfile`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
