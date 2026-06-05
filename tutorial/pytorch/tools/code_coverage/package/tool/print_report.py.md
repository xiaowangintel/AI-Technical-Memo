# print_report.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/code_coverage/package/tool/print_report.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements code-coverage parsing, aggregation, or reporting helpers for CI analysis.
- **Purpose (CN)**: 实现代码覆盖率解析、聚合或报告辅助逻辑，用于 CI 分析。
## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```python
from __future__ import annotations

import os
import subprocess
from typing import IO

from ..oss.utils import get_pytorch_folder
from ..util.setting import SUMMARY_FOLDER_DIR, TestList, TestStatusType
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as ..oss.utils, ..util.setting; Python standard-library modules such as __future__, os, subprocess, and 1 more. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 ..oss.utils、..util.setting；Python 标准库模块，如 __future__、os、subprocess 等共 4 项。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 10-18
```python

CoverageItem = tuple[str, float, int, int]


def key_by_percentage(x: CoverageItem) -> float:
    return x[1]


def key_by_name(x: CoverageItem) -> str:
```
- **EN**: This chunk defines `key_by_name`, which implements a focused step inside the coverage tooling pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `key_by_name`，其作用是实现覆盖率工具流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 19-27
```python
    return x[0]


def is_intrested_file(file_path: str, interested_folders: list[str]) -> bool:
    if "cuda" in file_path:
        return False
    if "aten/gen_aten" in file_path or "aten/aten_" in file_path:
        return False
    for folder in interested_folders:
```
- **EN**: This chunk defines `is_intrested_file`, which implements a focused step inside the coverage tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `is_intrested_file`，其作用是实现覆盖率工具流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 28-36
```python
        if folder in file_path:
            return True
    return False


def is_this_type_of_tests(target_name: str, test_set_by_type: set[str]) -> bool:
    # tests are divided into three types: success / partial success / fail to collect coverage
    for test in test_set_by_type:
        if target_name in test:
```
- **EN**: This chunk defines `is_this_type_of_tests`, which implements a focused step inside the coverage tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `is_this_type_of_tests`，其作用是实现覆盖率工具流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 37-45
```python
            return True
    return False


def print_test_by_type(
    tests: TestList, test_set_by_type: set[str], type_name: str, summary_file: IO[str]
) -> None:
    print("Tests " + type_name + " to collect coverage:", file=summary_file)
    for test in tests:
```
- **EN**: This chunk defines `print_test_by_type`, which implements a focused step inside the coverage tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `print_test_by_type`，其作用是实现覆盖率工具流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 46-63
```python
        if is_this_type_of_tests(test.name, test_set_by_type):
            print(test.target_pattern, file=summary_file)
    print(file=summary_file)


def print_test_condition(
    tests: TestList,
    tests_type: TestStatusType,
    interested_folders: list[str],
    coverage_only: list[str],
    summary_file: IO[str],
    summary_type: str,
) -> None:
    print_test_by_type(tests, tests_type["success"], "fully success", summary_file)
    print_test_by_type(tests, tests_type["partial"], "partially success", summary_file)
    print_test_by_type(tests, tests_type["fail"], "failed", summary_file)
    print(
        "\n\nCoverage Collected Over Interested Folders:\n",
```
- **EN**: This chunk defines `print_test_condition`, which implements a focused step inside the coverage tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `print_test_condition`，其作用是实现覆盖率工具流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 64-78
```python
        interested_folders,
        file=summary_file,
    )
    print(
        "\n\nCoverage Compilation Flags Only Apply To: \n",
        coverage_only,
        file=summary_file,
    )
    print(
        "\n\n---------------------------------- "
        + summary_type
        + " ----------------------------------",
        file=summary_file,
    )
```
- **EN**: This chunk continues `print_test_condition` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `print_test_condition`，进一步展开其内部控制流或数据流转。

### Lines 79-96
```python

def line_oriented_report(
    tests: TestList,
    tests_type: TestStatusType,
    interested_folders: list[str],
    coverage_only: list[str],
    covered_lines: dict[str, set[int]],
    uncovered_lines: dict[str, set[int]],
) -> None:
    with open(os.path.join(SUMMARY_FOLDER_DIR, "line_summary"), "w+") as report_file:
        print_test_condition(
            tests,
            tests_type,
            interested_folders,
            coverage_only,
            report_file,
            "LINE SUMMARY",
        )
```
- **EN**: This chunk defines `line_oriented_report`, which aggregates signals and turns them into summaries, metrics, or alerts. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段定义了 `line_oriented_report`，其作用是聚合信号，并将其转化为摘要、指标或告警。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 97-105
```python
        for file_name in covered_lines:
            covered = covered_lines[file_name]
            uncovered = uncovered_lines[file_name]
            print(
                f"{file_name}\n  covered lines: {sorted(covered)}\n  unconvered lines:{sorted(uncovered)}",
                file=report_file,
            )
```
- **EN**: This chunk continues `line_oriented_report` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `line_oriented_report`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 106-118
```python
def print_file_summary(
    covered_summary: int, total_summary: int, summary_file: IO[str]
) -> float:
    # print summary first
    try:
        coverage_percentage = 100.0 * covered_summary / total_summary
    except ZeroDivisionError:
        coverage_percentage = 0
    print(
        f"SUMMARY\ncovered: {covered_summary}\nuncovered: {total_summary}\npercentage: {coverage_percentage:.2f}%\n\n",
        file=summary_file,
    )
    if coverage_percentage == 0:
```
- **EN**: This chunk defines `print_file_summary`, which aggregates signals and turns them into summaries, metrics, or alerts. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `print_file_summary`，其作用是聚合信号，并将其转化为摘要、指标或告警。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 119-136
```python
        print("Coverage is 0, Please check if json profiles are valid")
    return coverage_percentage


def print_file_oriented_report(
    tests_type: TestStatusType,
    coverage: list[CoverageItem],
    covered_summary: int,
    total_summary: int,
    summary_file: IO[str],
    tests: TestList,
    interested_folders: list[str],
    coverage_only: list[str],
) -> None:
    coverage_percentage = print_file_summary(
        covered_summary, total_summary, summary_file
    )
    # print test condition (interested folder / tests that are successful or failed)
```
- **EN**: This chunk defines `print_file_oriented_report`, which aggregates signals and turns them into summaries, metrics, or alerts. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `print_file_oriented_report`，其作用是聚合信号，并将其转化为摘要、指标或告警。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 137-146
```python
    print_test_condition(
        tests,
        tests_type,
        interested_folders,
        coverage_only,
        summary_file,
        "FILE SUMMARY",
    )
    # print each file's information
    for item in coverage:
```
- **EN**: This chunk continues `print_file_oriented_report` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `print_file_oriented_report`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 147-156
```python
        print(
            item[0].ljust(75),
            (str(item[1]) + "%").rjust(10),
            str(item[2]).rjust(10),
            str(item[3]).rjust(10),
            file=summary_file,
        )

    print(f"summary percentage:{coverage_percentage:.2f}%")
```
- **EN**: This chunk continues `print_file_oriented_report` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `print_file_oriented_report`，进一步展开其内部控制流或数据流转。

### Lines 157-170
```python

def file_oriented_report(
    tests: TestList,
    tests_type: TestStatusType,
    interested_folders: list[str],
    coverage_only: list[str],
    covered_lines: dict[str, set[int]],
    uncovered_lines: dict[str, set[int]],
) -> None:
    with open(os.path.join(SUMMARY_FOLDER_DIR, "file_summary"), "w+") as summary_file:
        covered_summary = 0
        total_summary = 0
        coverage = []
        for file_name in covered_lines:
```
- **EN**: This chunk defines `file_oriented_report`, which aggregates signals and turns them into summaries, metrics, or alerts. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `file_oriented_report`，其作用是聚合信号，并将其转化为摘要、指标或告警。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 171-188
```python
            # get coverage number for this file
            covered_count = len(covered_lines[file_name])
            total_count = covered_count + len(uncovered_lines[file_name])
            try:
                percentage = round(covered_count / total_count * 100, 2)
            except ZeroDivisionError:
                percentage = 0
            # store information in a list to be sorted
            coverage.append((file_name, percentage, covered_count, total_count))
            # update summary
            covered_summary = covered_summary + covered_count
            total_summary = total_summary + total_count
        # sort
        coverage.sort(key=key_by_name)
        coverage.sort(key=key_by_percentage)
        # print
        print_file_oriented_report(
            tests_type,
```
- **EN**: This chunk continues `file_oriented_report` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `file_oriented_report`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 189-197
```python
            coverage,
            covered_summary,
            total_summary,
            summary_file,
            tests,
            interested_folders,
            coverage_only,
        )
```
- **EN**: This chunk continues `file_oriented_report` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `file_oriented_report`，进一步展开其内部控制流或数据流转。

### Lines 198-215
```python

def get_html_ignored_pattern() -> list[str]:
    return ["/usr/*", "*anaconda3/*", "*third_party/*"]


def html_oriented_report() -> None:
    # use lcov to generate the coverage report
    build_folder = os.path.join(get_pytorch_folder(), "build")
    coverage_info_file = os.path.join(SUMMARY_FOLDER_DIR, "coverage.info")
    # generate coverage report -- coverage.info in build folder
    subprocess.check_call(
        [
            "lcov",
            "--capture",
            "--directory",
            build_folder,
            "--output-file",
            coverage_info_file,
```
- **EN**: This chunk defines `html_oriented_report`, which aggregates signals and turns them into summaries, metrics, or alerts. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `html_oriented_report`，其作用是聚合信号，并将其转化为摘要、指标或告警。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 216-233
```python
        ]
    )
    # remove files that are unrelated
    cmd_array = (
        ["lcov", "--remove", coverage_info_file]
        + get_html_ignored_pattern()
        + ["--output-file", coverage_info_file]
    )
    subprocess.check_call(
        # ["lcov", "--remove", coverage_info_file, "--output-file", coverage_info_file]
        cmd_array
    )
    # generate beautiful html page
    subprocess.check_call(
        [
            "genhtml",
            coverage_info_file,
            "--output-directory",
```
- **EN**: This chunk continues `html_oriented_report` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段延续了 `html_oriented_report`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 234-236
```python
            os.path.join(SUMMARY_FOLDER_DIR, "html_report"),
        ]
    )
```
- **EN**: This chunk continues `html_oriented_report` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段延续了 `html_oriented_report`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

## Key Concepts / 关键概念

- **Coverage tooling**
  - EN: This file belongs to the coverage tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于覆盖率工具层，应结合同一子目录中的相邻脚本一起理解。
- **Coverage analysis**
  - EN: The implementation extracts, merges, or summarizes code-coverage information.
  - CN: 该实现负责提取、合并或汇总代码覆盖率信息。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **key_by_percentage**
  - EN: `key_by_percentage` is one of the main local symbols exposed or implemented here.
  - CN: `key_by_percentage` 是此处暴露或实现的主要局部符号之一。
- **key_by_name**
  - EN: `key_by_name` is one of the main local symbols exposed or implemented here.
  - CN: `key_by_name` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `..oss.utils`, `..util.setting`
- **Python standard library / Python 标准库**: `__future__`, `os`, `subprocess`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `key_by_percentage`, `key_by_name`, `is_intrested_file`, `is_this_type_of_tests`, `print_test_by_type`, `print_test_condition`, `line_oriented_report`, `print_file_summary`, `print_file_oriented_report`, `file_oriented_report`, `get_html_ignored_pattern`, `html_oriented_report`
