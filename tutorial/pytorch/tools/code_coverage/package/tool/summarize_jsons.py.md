# summarize_jsons.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/code_coverage/package/tool/summarize_jsons.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements code-coverage parsing, aggregation, or reporting helpers for CI analysis.
- **Purpose (CN)**: 实现代码覆盖率解析、聚合或报告辅助逻辑，用于 CI 分析。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
from __future__ import annotations

import json
import os
import time
from typing import Any, TYPE_CHECKING

from ..util.setting import (
    CompilerType,
    JSON_FOLDER_BASE_DIR,
    TestList,
    TestPlatform,
    TestStatusType,
)
from ..util.utils import (
    detect_compiler_type,
    print_error,
    print_time,
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as ..util.setting, ..util.utils; Python standard-library modules such as __future__, json, os, and 2 more. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 ..util.setting、..util.utils；Python 标准库模块，如 __future__、json、os 等共 5 项。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 19-28
```python
    related_to_test_list,
)
from .parser.gcov_coverage_parser import GcovCoverageParser
from .parser.llvm_coverage_parser import LlvmCoverageParser
from .print_report import (
    file_oriented_report,
    html_oriented_report,
    line_oriented_report,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as .parser.gcov_coverage_parser, .parser.llvm_coverage_parser, .print_report.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 .parser.gcov_coverage_parser、.parser.llvm_coverage_parser、.print_report。

### Lines 29-38
```python

if TYPE_CHECKING:
    from .parser.coverage_record import CoverageRecord


# coverage_records: dict[str, LineInfo] = {}
covered_lines: dict[str, set[int]] = {}
uncovered_lines: dict[str, set[int]] = {}
tests_type: TestStatusType = {"success": set(), "partial": set(), "fail": set()}
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as .parser.coverage_record. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 .parser.coverage_record。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 39-47
```python

def transform_file_name(
    file_path: str, interested_folders: list[str], platform: TestPlatform
) -> str:
    remove_patterns: set[str] = {".DEFAULT.cpp", ".AVX.cpp", ".AVX2.cpp"}
    for pattern in remove_patterns:
        file_path = file_path.replace(pattern, "")
    # if user has specified interested folder
    if interested_folders:
```
- **EN**: This chunk defines `transform_file_name`, which implements a focused step inside the coverage tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `transform_file_name`，其作用是实现覆盖率工具流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 48-56
```python
        for folder in interested_folders:
            if folder in file_path:
                return file_path[file_path.find(folder) :]
    # remove pytorch base folder path
    if platform == TestPlatform.OSS:
        from package.oss.utils import get_pytorch_folder  # type: ignore[import]

        pytorch_foler = get_pytorch_folder()
        if not file_path.startswith(pytorch_foler):
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as package.oss.utils. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 package.oss.utils。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 57-66
```python
            raise AssertionError(f"file_path must start with {pytorch_foler}")
        file_path = file_path[len(pytorch_foler) + 1 :]
    return file_path


def is_intrested_file(
    file_path: str, interested_folders: list[str], platform: TestPlatform
) -> bool:
    ignored_patterns = ["cuda", "aten/gen_aten", "aten/aten_", "build/"]
    if any(pattern in file_path for pattern in ignored_patterns):
```
- **EN**: This chunk defines `is_intrested_file`, which implements a focused step inside the coverage tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `is_intrested_file`，其作用是实现覆盖率工具流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 67-77
```python
        return False

    # ignore files that are not belong to pytorch
    if platform == TestPlatform.OSS:
        # pyrefly: ignore [missing-import]
        from package.oss.utils import get_pytorch_folder

        if not file_path.startswith(get_pytorch_folder()):
            return False
    # if user has specified interested folder
    if interested_folders:
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as package.oss.utils. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 package.oss.utils。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 78-86
```python
        for folder in interested_folders:
            intersted_folder_path = folder if folder.endswith("/") else f"{folder}/"
            if intersted_folder_path in file_path:
                return True
        return False
    else:
        return True
```
- **EN**: This chunk continues `is_intrested_file` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `is_intrested_file`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 87-98
```python
def get_json_obj(json_file: str) -> tuple[Any, int]:
    """
    Sometimes at the start of file llvm/gcov will complains "fail to find coverage data",
    then we need to skip these lines
      -- success read: 0      -  this json file have the full json coverage information
      -- partial success: 1   -  this json file starts with some error prompt, but still have the coverage information
      -- fail to read: 2      -  this json file doesn't have any coverage information
    """
    read_status = -1
    with open(json_file) as f:
        lines = f.readlines()
        for line in lines:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_json_obj`, which implements a focused step inside the coverage tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_json_obj`，其作用是实现覆盖率工具流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 99-110
```python
            try:
                json_obj = json.loads(line)
            except json.JSONDecodeError:
                read_status = 1
                continue
            else:
                if read_status == -1:
                    # not meet jsonDecoderError before, return success
                    read_status = 0
                return (json_obj, read_status)
    return None, 2
```
- **EN**: This chunk continues `get_json_obj` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_json_obj`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 111-124
```python

def parse_json(json_file: str, platform: TestPlatform) -> list[CoverageRecord]:
    print("start parse:", json_file)
    json_obj, read_status = get_json_obj(json_file)
    if read_status == 0:
        tests_type["success"].add(json_file)
    elif read_status == 1:
        tests_type["partial"].add(json_file)
    else:
        tests_type["fail"].add(json_file)
        raise RuntimeError(
            "Fail to do code coverage! Fail to load json file: ", json_file
        )
```
- **EN**: This chunk defines `parse_json`, which parses or loads structured input into tool-friendly data structures. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段定义了 `parse_json`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 125-133
```python
    cov_type = detect_compiler_type(platform)

    coverage_records: list[CoverageRecord] = []
    if cov_type == CompilerType.CLANG:
        coverage_records = LlvmCoverageParser(json_obj).parse("fbcode")
        # print(coverage_records)
    elif cov_type == CompilerType.GCC:
        coverage_records = GcovCoverageParser(json_obj).parse()
```
- **EN**: This chunk continues `parse_json` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `parse_json`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 134-142
```python
    return coverage_records


def parse_jsons(
    test_list: TestList, interested_folders: list[str], platform: TestPlatform
) -> None:
    g = os.walk(JSON_FOLDER_BASE_DIR)

    for path, _, file_list in g:
```
- **EN**: This chunk defines `parse_jsons`, which parses or loads structured input into tool-friendly data structures. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `parse_jsons`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 143-152
```python
        for file_name in file_list:
            if file_name.endswith(".json"):
                # if compiler is clang, we only analyze related json / when compiler is gcc, we analyze all jsons
                cov_type = detect_compiler_type(platform)
                if cov_type == CompilerType.CLANG and not related_to_test_list(
                    file_name, test_list
                ):
                    continue
                json_file = os.path.join(path, file_name)
                try:
```
- **EN**: This chunk continues `parse_jsons` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `parse_jsons`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 153-161
```python
                    coverage_records = parse_json(json_file, platform)
                except RuntimeError:
                    print_error("Fail to load json file: ", json_file)
                    continue
                # collect information from each target's export file and merge them together:
                update_coverage(coverage_records, interested_folders, platform)


def update_coverage(
```
- **EN**: This chunk defines `update_coverage`, which implements a focused step inside the coverage tooling pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `update_coverage`，其作用是实现覆盖率工具流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 162-170
```python
    coverage_records: list[CoverageRecord],
    interested_folders: list[str],
    platform: TestPlatform,
) -> None:
    for item in coverage_records:
        # extract information for the record
        record = item.to_dict()
        file_path = record["filepath"]
        if not is_intrested_file(file_path, interested_folders, platform):
```
- **EN**: This chunk continues `update_coverage` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `update_coverage`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 171-180
```python
            continue
        covered_range = record["covered_lines"]
        uncovered_range = record["uncovered_lines"]
        # transform file name: remote/13223/caffe2/aten -> caffe2/aten
        file_path = transform_file_name(file_path, interested_folders, platform)

        # if file not exists, add it into dictionary
        if file_path not in covered_lines:
            covered_lines[file_path] = set()
        if file_path not in uncovered_lines:
```
- **EN**: This chunk continues `update_coverage` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `update_coverage`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 181-189
```python
            uncovered_lines[file_path] = set()
        # update this file's covered and uncovered lines
        if covered_range is not None:
            covered_lines[file_path].update(covered_range)
        if uncovered_range is not None:
            uncovered_lines[file_path].update(uncovered_range)


def update_set() -> None:
```
- **EN**: This chunk defines `update_set`, which implements a focused step inside the coverage tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `update_set`，其作用是实现覆盖率工具流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 190-202
```python
    for file_name in covered_lines:
        # difference_update
        uncovered_lines[file_name].difference_update(covered_lines[file_name])


def summarize_jsons(
    test_list: TestList,
    interested_folders: list[str],
    coverage_only: list[str],
    platform: TestPlatform,
) -> None:
    start_time = time.time()
    if detect_compiler_type(platform) == CompilerType.GCC:
```
- **EN**: This chunk defines `summarize_jsons`, which implements a focused step inside the coverage tooling pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `summarize_jsons`，其作用是实现覆盖率工具流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 203-220
```python
        html_oriented_report()
    else:
        parse_jsons(test_list, interested_folders, platform)
        update_set()
        line_oriented_report(
            test_list,
            tests_type,
            interested_folders,
            coverage_only,
            covered_lines,
            uncovered_lines,
        )
        file_oriented_report(
            test_list,
            tests_type,
            interested_folders,
            coverage_only,
            covered_lines,
```
- **EN**: This chunk continues `summarize_jsons` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `summarize_jsons`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 221-223
```python
            uncovered_lines,
        )
    print_time("summary jsons take time: ", start_time)
```
- **EN**: This chunk continues `summarize_jsons` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `summarize_jsons`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

## Key Concepts / 关键概念

- **Coverage tooling**
  - EN: This file belongs to the coverage tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于覆盖率工具层，应结合同一子目录中的相邻脚本一起理解。
- **Coverage analysis**
  - EN: The implementation extracts, merges, or summarizes code-coverage information.
  - CN: 该实现负责提取、合并或汇总代码覆盖率信息。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **transform_file_name**
  - EN: `transform_file_name` is one of the main local symbols exposed or implemented here.
  - CN: `transform_file_name` 是此处暴露或实现的主要局部符号之一。
- **is_intrested_file**
  - EN: `is_intrested_file` is one of the main local symbols exposed or implemented here.
  - CN: `is_intrested_file` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `..util.setting`, `..util.utils`, `.parser.gcov_coverage_parser`, `.parser.llvm_coverage_parser`, `.print_report`, `.parser.coverage_record`
- **Python standard library / Python 标准库**: `__future__`, `json`, `os`, `time`, `typing`
- **External packages / 外部依赖包**: `package.oss.utils`
- **Primary symbols in this file / 本文件核心符号**: `transform_file_name`, `is_intrested_file`, `get_json_obj`, `parse_json`, `parse_jsons`, `update_coverage`, `update_set`, `summarize_jsons`
