# ci_coverage_report.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/ci_coverage_report.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `ci_coverage_report` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `ci_coverage_report` 流程，主要负责CI 编排、测试执行。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-10: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
CI Coverage Report Generator

Collects all CI test registrations from test/registered/ and generates
a coverage report organized by folder, backend, and suite.

Usage:
    python scripts/ci/utils/ci_coverage_report.py [--output-format markdown|json]
"""
```
**EN:** CI Coverage Report Generator Collects all CI test registrations from test/registered/ and generates a coverage report organized by folder, backend, and suite.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 12-18: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import glob
import json
import os
import sys
from collections import defaultdict
from pathlib import Path
```
**EN:** This block loads argparse, glob, json, os, sys, collections, pathlib. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, glob, json, os, sys, collections, pathlib。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 21-26: Top-level expr logic / 顶层 expr 逻辑
```python
sys.path.insert(
    0,
    str(
        Path(__file__).parent.parent.parent.parent / "python" / "sglang" / "test" / "ci"
    ),
)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 28-28: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
from ci_register import CIRegistry, HWBackend, ut_parse_one_file
```
**EN:** This block loads ci_register. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 ci_register。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 31-43: Defines the `collect_all_tests` routine / 定义 `collect_all_tests` 例程
```python
def collect_all_tests(registered_dir: str) -> list[CIRegistry]:
    """Collect all CI registrations from registered directory."""
    files = glob.glob(f"{registered_dir}/**/*.py", recursive=True)
    all_tests = []

    for file in sorted(files):
        try:
            registries, _ = ut_parse_one_file(file)
            all_tests.extend(registries)
        except Exception as e:
            print(f"Warning: Failed to parse {file}: {e}", file=sys.stderr)

    return all_tests
```
**EN:** This block defines `collect_all_tests`. Collect all CI registrations from registered directory. It accepts 1 parameter(s): registered_dir. Internally it emits status messages.
**CN:** 该代码块定义了 `collect_all_tests`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：registered_dir。内部会输出状态信息。

### Lines 46-54: Defines the `get_folder_name` routine / 定义 `get_folder_name` 例程
```python
def get_folder_name(filename: str) -> str:
    """Extract folder name from test filename."""
    # e.g., "registered/models/test_foo.py" -> "models"
    parts = Path(filename).parts
    if "registered" in parts:
        idx = parts.index("registered")
        if idx + 1 < len(parts) - 1:  # Has subfolder
            return parts[idx + 1]
    return "root"
```
**EN:** This block defines `get_folder_name`. Extract folder name from test filename. It accepts 1 parameter(s): filename. Internally it touches filesystem paths.
**CN:** 该代码块定义了 `get_folder_name`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：filename。内部会处理文件系统路径。

### Lines 57-59: Defines the `get_test_basename` routine / 定义 `get_test_basename` 例程
```python
def get_test_basename(filename: str) -> str:
    """Extract just the test file name from the path."""
    return Path(filename).name
```
**EN:** This block defines `get_test_basename`. Extract just the test file name from the path. It accepts 1 parameter(s): filename. Internally it touches filesystem paths.
**CN:** 该代码块定义了 `get_test_basename`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：filename。内部会处理文件系统路径。

### Lines 62-89: Defines the `organize_test_data` routine / 定义 `organize_test_data` 例程
```python
def organize_test_data(tests: list[CIRegistry]) -> dict:
    """Organize tests into various groupings."""
    by_backend = defaultdict(list)
    by_folder = defaultdict(list)
    disabled_tests = []

    for t in tests:
        by_backend[t.backend.name].append(t)
        by_folder[get_folder_name(t.filename)].append(t)
        if t.disabled:
            disabled_tests.append(t)

    # Count unique test files (a file may be registered for multiple backends)
    unique_files = set(t.filename for t in tests)
    unique_enabled_files = set(t.filename for t in tests if not t.disabled)
    unique_disabled_files = set(t.filename for t in tests if t.disabled)

    return {
        "total": len(tests),
        "total_unique_files": len(unique_files),
        "enabled": len(tests) - len(disabled_tests),
        "enabled_unique_files": len(unique_enabled_files),
        "disabled_count": len(disabled_tests),
        "disabled_unique_files": len(unique_disabled_files),
        "by_backend": by_backend,
        "by_folder": by_folder,
        "disabled_tests": disabled_tests,
    }
```
**EN:** This block defines `organize_test_data`. Organize tests into various groupings. It accepts 1 parameter(s): tests. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `organize_test_data`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：tests。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 92-163: Defines the `generate_summary_section` routine / 定义 `generate_summary_section` 例程
```python
def generate_summary_section(data: dict) -> str:
    """Generate the summary/overview section."""
    lines = []
    lines.append("# CI Coverage Overview\n")
    lines.append(
        f"**Unique Test Files:** {data['total_unique_files']} ({data['enabled_unique_files']} enabled, {data['disabled_unique_files']} disabled)\n"
    )
    lines.append(
        f"**Total Registrations:** {data['total']} ({data['enabled']} enabled, {data['disabled_count']} disabled)\n"
    )
    lines.append(
        "*Note: A test file may be registered for multiple backends (e.g., CUDA + AMD), so total registrations > unique files.*\n"
    )

    by_backend = data["by_backend"]
    by_folder = data["by_folder"]
    disabled_tests = data["disabled_tests"]

    # Backend summary (collapsible)
    lines.append("<details>")
    lines.append("<summary><h2>Backend Summary</h2></summary>\n")
    lines.append("| Backend | Total | Enabled | Disabled | Per-Commit | Nightly |")
    lines.append("|---------|-------|---------|----------|------------|---------|")

    for backend in ["CUDA", "AMD", "NPU", "CPU"]:
        backend_tests = by_backend.get(backend, [])
        if not backend_tests:
            continue
        b_total = len(backend_tests)
        b_disabled = sum(1 for t in backend_tests if t.disabled)
        b_enabled = b_total - b_disabled
        b_per_commit = sum(1 for t in backend_tests if not t.nightly and not t.disabled)
        b_nightly = sum(1 for t in backend_tests if t.nightly and not t.disabled)
        lines.append(
            f"| {backend} | {b_total} | {b_enabled} | {b_disabled} | {b_per_commit} | {b_nightly} |"
        )

    lines.append("\n</details>\n")

    # Folder summary (collapsible)
    lines.append("<details>")
    lines.append("<summary><h2>Folder Summary</h2></summary>\n")
    lines.append("| Folder | CUDA | AMD | NPU | CPU | Total |")
    lines.append("|--------|------|-----|-----|-----|-------|")

    for folder in sorted(by_folder.keys()):
        folder_tests = by_folder[folder]
        cuda = sum(1 for t in folder_tests if t.backend == HWBackend.CUDA)
        amd = sum(1 for t in folder_tests if t.backend == HWBackend.AMD)
        npu = sum(1 for t in folder_tests if t.backend == HWBackend.NPU)
        cpu = sum(1 for t in folder_tests if t.backend == HWBackend.CPU)
        lines.append(
            f"| {folder} | {cuda} | {amd} | {npu} | {cpu} | {len(folder_tests)} |"
        )

    lines.append("\n</details>\n")

    # Disabled tests section (collapsible)
    if disabled_tests:
        lines.append("<details>")
        lines.append("<summary><h2>Disabled Tests</h2></summary>\n")
        lines.append("| File | Backend | Suite | Reason |")
        lines.append("|------|---------|-------|--------|")
        for t in sorted(disabled_tests, key=lambda x: (x.backend.name, x.filename)):
            test_name = get_test_basename(t.filename)
            reason = t.disabled[:50] + "..." if len(t.disabled) > 50 else t.disabled
            lines.append(
                f"| `{test_name}` | {t.backend.name} | {t.effective_suite} | {reason} |"
            )
        lines.append("\n</details>\n")

    return "\n".join(lines)
```
**EN:** This block defines `generate_summary_section`. Generate the summary/overview section. It accepts 1 parameter(s): data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_summary_section`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 166-209: Defines the `generate_by_folder_section` routine / 定义 `generate_by_folder_section` 例程
```python
def generate_by_folder_section(data: dict) -> str:
    """Generate the 'All Tests by Folder' section."""
    lines = []
    by_folder = data["by_folder"]

    lines.append("# All Tests by Folder\n")

    for folder in sorted(by_folder.keys()):
        folder_tests = by_folder[folder]
        lines.append("<details>")
        lines.append(
            f"<summary><h2>{folder}/ ({len(folder_tests)} tests)</h2></summary>\n"
        )

        # Group by backend within folder
        folder_by_backend = defaultdict(list)
        for t in folder_tests:
            folder_by_backend[t.backend.name].append(t)

        for backend in ["CUDA", "AMD", "NPU", "CPU"]:
            backend_tests = folder_by_backend.get(backend, [])
            if not backend_tests:
                continue

            lines.append(f"### {backend} ({len(backend_tests)} tests)\n")
            lines.append("| Test File | Suite | Est. Time | Status |")
            lines.append("|-----------|-------|-----------|--------|")

            for t in sorted(backend_tests, key=lambda x: x.filename):
                test_name = get_test_basename(t.filename)
                status = (
                    "Disabled"
                    if t.disabled
                    else ("Nightly" if t.nightly else "Per-Commit")
                )
                lines.append(
                    f"| `{test_name}` | {t.effective_suite} | {t.est_time:.0f}s | {status} |"
                )

            lines.append("")

        lines.append("</details>\n")

    return "\n".join(lines)
```
**EN:** This block defines `generate_by_folder_section`. Generate the 'All Tests by Folder' section. It accepts 1 parameter(s): data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_by_folder_section`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 212-274: Defines the `generate_by_suite_section` routine / 定义 `generate_by_suite_section` 例程
```python
def generate_by_suite_section(data: dict) -> str:
    """Generate the 'All Tests by Test Suite' section."""
    lines = []
    by_backend = data["by_backend"]

    lines.append("# All Tests by Test Suite\n")

    for backend in ["CUDA", "AMD", "NPU", "CPU"]:
        backend_tests = by_backend.get(backend, [])
        if not backend_tests:
            continue

        b_total = len(backend_tests)
        b_disabled = sum(1 for t in backend_tests if t.disabled)
        b_enabled = b_total - b_disabled

        lines.append("<details>")
        lines.append(
            f"<summary><h2>{backend} Backend ({b_enabled} enabled, {b_disabled} disabled)</h2></summary>\n"
        )

        # Group by suite within backend
        backend_suites = defaultdict(list)
        for t in backend_tests:
            backend_suites[t.effective_suite].append(t)

        for suite in sorted(backend_suites.keys()):
            suite_tests = backend_suites[suite]
            s_enabled = sum(1 for t in suite_tests if not t.disabled)
            s_disabled = sum(1 for t in suite_tests if t.disabled)
            s_est_time = sum(t.est_time for t in suite_tests if not t.disabled)
            is_nightly = any(t.nightly for t in suite_tests if not t.disabled)

            suite_type = "Nightly" if is_nightly else "Per-Commit"
            lines.append("<details>")
            lines.append(
                f"<summary><h3>{suite} ({s_enabled} enabled, {s_disabled} disabled) - {suite_type}</h3></summary>\n"
            )
            lines.append(f"*Estimated total time: {s_est_time:.0f}s*\n")

            lines.append("| Test File | Folder | Est. Time | Status |")
            lines.append("|-----------|--------|-----------|--------|")

            for t in sorted(suite_tests, key=lambda x: x.filename):
                test_name = get_test_basename(t.filename)
                folder = get_folder_name(t.filename)
                if t.disabled:
                    status = (
                        f"Disabled: {t.disabled[:30]}..."
                        if len(t.disabled) > 30
                        else f"Disabled: {t.disabled}"
                    )
                else:
                    status = "Nightly" if t.nightly else "Per-Commit"
                lines.append(
                    f"| `{test_name}` | {folder} | {t.est_time:.0f}s | {status} |"
                )

            lines.append("\n</details>\n")

        lines.append("</details>\n")

    return "\n".join(lines)
```
**EN:** This block defines `generate_by_suite_section`. Generate the 'All Tests by Test Suite' section. It accepts 1 parameter(s): data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_by_suite_section`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 277-295: Defines the `generate_markdown_report` routine / 定义 `generate_markdown_report` 例程
```python
def generate_markdown_report(tests: list[CIRegistry], section: str = "all") -> str:
    """Generate markdown report for GitHub step summary."""
    data = organize_test_data(tests)

    if section == "summary":
        return generate_summary_section(data)
    elif section == "by-folder":
        return generate_by_folder_section(data)
    elif section == "by-suite":
        return generate_by_suite_section(data)
    else:  # "all"
        parts = [
            generate_summary_section(data),
            "---",
            generate_by_folder_section(data),
            "---",
            generate_by_suite_section(data),
        ]
        return "\n".join(parts)
```
**EN:** This block defines `generate_markdown_report`. Generate markdown report for GitHub step summary. It accepts 2 parameter(s): tests, section. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_markdown_report`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：tests, section。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 298-433: Defines the `generate_json_report` routine / 定义 `generate_json_report` 例程
```python
def generate_json_report(tests: list[CIRegistry]) -> str:
    """Generate JSON report with detailed test listings."""
    by_backend = defaultdict(list)
    by_folder = defaultdict(list)

    for t in tests:
        by_backend[t.backend.name].append(t)
        by_folder[get_folder_name(t.filename)].append(t)

    disabled_tests = [t for t in tests if t.disabled]

    # Build structured data
    data = {
        "summary": {
            "total": len(tests),
            "enabled": len(tests) - len(disabled_tests),
            "disabled": len(disabled_tests),
        },
        "tests_by_folder": {},
        "tests_by_suite": {},
        "backend_summary": {},
        "folder_summary": {},
        "disabled_tests": [],
    }

    # Section 1: Tests by Folder
    for folder in sorted(by_folder.keys()):
        folder_tests = by_folder[folder]
        folder_by_backend = defaultdict(list)
        for t in folder_tests:
            folder_by_backend[t.backend.name].append(t)

        data["tests_by_folder"][folder] = {
            "total": len(folder_tests),
            "backends": {},
        }

        for backend in ["CUDA", "AMD", "NPU", "CPU"]:
            backend_tests = folder_by_backend.get(backend, [])
            if backend_tests:
                data["tests_by_folder"][folder]["backends"][backend] = [
                    {
                        "filename": get_test_basename(t.filename),
                        "suite": t.effective_suite,
                        "est_time": t.est_time,
                        "status": (
                            "disabled"
                            if t.disabled
                            else ("nightly" if t.nightly else "per-commit")
                        ),
                    }
                    for t in sorted(backend_tests, key=lambda x: x.filename)
                ]

    # Section 2: Tests by Suite (Backend -> Suite)
    for backend in ["CUDA", "AMD", "NPU", "CPU"]:
        backend_tests = by_backend.get(backend, [])
        if not backend_tests:
            continue

        backend_suites = defaultdict(list)
        for t in backend_tests:
            backend_suites[t.effective_suite].append(t)

        data["tests_by_suite"][backend] = {
            "total": len(backend_tests),
            "enabled": sum(1 for t in backend_tests if not t.disabled),
            "disabled": sum(1 for t in backend_tests if t.disabled),
            "suites": {},
        }

        for suite in sorted(backend_suites.keys()):
            suite_tests = backend_suites[suite]
            is_nightly = any(t.nightly for t in suite_tests if not t.disabled)

            data["tests_by_suite"][backend]["suites"][suite] = {
                "total": len(suite_tests),
                "enabled": sum(1 for t in suite_tests if not t.disabled),
                "disabled": sum(1 for t in suite_tests if t.disabled),
                "est_time": sum(t.est_time for t in suite_tests if not t.disabled),
                "type": "nightly" if is_nightly else "per-commit",
                "tests": [
                    {
                        "filename": get_test_basename(t.filename),
                        "folder": get_folder_name(t.filename),
                        "est_time": t.est_time,
                        "status": (
                            "disabled"
                            if t.disabled
                            else ("nightly" if t.nightly else "per-commit")
                        ),
                        "disabled_reason": t.disabled if t.disabled else None,
                    }
                    for t in sorted(suite_tests, key=lambda x: x.filename)
                ],
            }

    # Backend summary
    for backend in ["CUDA", "AMD", "NPU", "CPU"]:
        backend_tests = by_backend.get(backend, [])
        if backend_tests:
            data["backend_summary"][backend] = {
                "total": len(backend_tests),
                "enabled": sum(1 for t in backend_tests if not t.disabled),
                "disabled": sum(1 for t in backend_tests if t.disabled),
                "per_commit": sum(
                    1 for t in backend_tests if not t.nightly and not t.disabled
                ),
                "nightly": sum(
                    1 for t in backend_tests if t.nightly and not t.disabled
                ),
            }

    # Folder summary
    for folder in sorted(by_folder.keys()):
        folder_tests = by_folder[folder]
        data["folder_summary"][folder] = {
            "CUDA": sum(1 for t in folder_tests if t.backend == HWBackend.CUDA),
            "AMD": sum(1 for t in folder_tests if t.backend == HWBackend.AMD),
            "NPU": sum(1 for t in folder_tests if t.backend == HWBackend.NPU),
            "CPU": sum(1 for t in folder_tests if t.backend == HWBackend.CPU),
            "total": len(folder_tests),
        }

    # Disabled tests
    for t in sorted(disabled_tests, key=lambda x: (x.backend.name, x.filename)):
        data["disabled_tests"].append(
            {
                "filename": get_test_basename(t.filename),
                "backend": t.backend.name,
                "suite": t.effective_suite,
                "reason": t.disabled,
            }
        )

    return json.dumps(data, indent=2)
```
**EN:** This block defines `generate_json_report`. Generate JSON report with detailed test listings. It accepts 1 parameter(s): tests. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_json_report`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：tests。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 436-475: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(description="Generate CI coverage report")
    parser.add_argument(
        "--output-format",
        choices=["markdown", "json"],
        default="markdown",
        help="Output format (default: markdown)",
    )
    parser.add_argument(
        "--section",
        choices=["all", "summary", "by-folder", "by-suite"],
        default="all",
        help="Which section to output (default: all). Only applies to markdown format.",
    )
    parser.add_argument(
        "--registered-dir",
        default="test/registered",
        help="Path to registered test directory",
    )
    args = parser.parse_args()

    # Change to repo root if needed
    script_dir = Path(__file__).parent.parent
    repo_root = script_dir.parent.parent
    os.chdir(repo_root)

    tests = collect_all_tests(args.registered_dir)

    if args.output_format == "markdown":
        report = generate_markdown_report(tests, section=args.section)
    else:
        report = generate_json_report(tests)

    print(report)

    # Write to GITHUB_STEP_SUMMARY if available
    summary_file = os.environ.get("GITHUB_STEP_SUMMARY")
    if summary_file and args.output_format == "markdown":
        with open(summary_file, "a") as f:
            f.write(report)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it touches filesystem paths, reads or writes files, parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会处理文件系统路径、读写文件、解析命令行参数、输出状态信息。

### Lines 478-479: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
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
- **Standard library / 标准库**: `argparse`, `collections`, `glob`, `json`, `os`, `pathlib`, `sys`
- **Third-party modules / 第三方模块**: `ci_register`
- **External commands inferred from code / 从代码推断的外部命令**: `python`
