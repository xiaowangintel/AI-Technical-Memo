# clang_coverage.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/code_coverage/package/tool/clang_coverage.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements code-coverage parsing, aggregation, or reporting helpers for CI analysis.
- **Purpose (CN)**: 实现代码覆盖率解析、聚合或报告辅助逻辑，用于 CI 分析。
## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```python
from __future__ import annotations

import os
import subprocess
import time
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, os, subprocess, and 1 more. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、os、subprocess 等共 4 项。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 7-18
```python
from ..util.setting import (
    JSON_FOLDER_BASE_DIR,
    MERGED_FOLDER_BASE_DIR,
    TestList,
    TestPlatform,
    TestType,
)
from ..util.utils import (
    check_platform_type,
    convert_to_relative_path,
    create_folder,
    get_raw_profiles_folder,
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as ..util.setting, ..util.utils. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 ..util.setting、..util.utils。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 19-26
```python
    get_test_name_from_whole_path,
    print_log,
    print_time,
    related_to_test_list,
    replace_extension,
)
from .utils import get_tool_path_by_platform, run_cpp_test
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as .utils.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 .utils。

### Lines 27-37
```python

def create_corresponding_folder(
    cur_path: str, prefix_cur_path: str, dir_list: list[str], new_base_folder: str
) -> None:
    for dir_name in dir_list:
        relative_path = convert_to_relative_path(
            cur_path, prefix_cur_path
        )  # get folder name like 'aten'
        new_folder_path = os.path.join(new_base_folder, relative_path, dir_name)
        create_folder(new_folder_path)
```
- **EN**: This chunk defines `create_corresponding_folder`, which implements a focused step inside the coverage tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `create_corresponding_folder`，其作用是实现覆盖率工具流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 38-46
```python

def run_target(
    binary_file: str, raw_file: str, test_type: TestType, platform_type: TestPlatform
) -> None:
    print_log("start run: ", binary_file)
    # set environment variable -- raw profile output path of the binary run
    os.environ["LLVM_PROFILE_FILE"] = raw_file
    # run binary
    if test_type == TestType.PY and platform_type == TestPlatform.OSS:
```
- **EN**: This chunk defines `run_target`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `run_target`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 47-52
```python
        from ..oss.utils import run_oss_python_test

        run_oss_python_test(binary_file)
    else:
        run_cpp_test(binary_file)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as ..oss.utils.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 ..oss.utils。

### Lines 53-64
```python

def merge_target(raw_file: str, merged_file: str, platform_type: TestPlatform) -> None:
    print_log("start to merge target: ", raw_file)
    # run command
    llvm_tool_path = get_tool_path_by_platform(platform_type)
    subprocess.check_call(
        [
            f"{llvm_tool_path}/llvm-profdata",
            "merge",
            "-sparse",
            raw_file,
            "-o",
```
- **EN**: This chunk defines `merge_target`, which implements a focused step inside the coverage tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段定义了 `merge_target`，其作用是实现覆盖率工具流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 65-70
```python
            merged_file,
        ]
    )


def export_target(
```
- **EN**: This chunk defines `export_target`, which implements a focused step inside the coverage tooling pipeline.
- **CN**: 这一段定义了 `export_target`，其作用是实现覆盖率工具流水线中的一个关键步骤。

### Lines 71-77
```python
    merged_file: str,
    json_file: str,
    binary_file: str,
    shared_library_list: list[str],
    platform_type: TestPlatform,
) -> None:
    if binary_file is None:
```
- **EN**: This chunk continues `export_target` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `export_target`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 78-85
```python
        raise Exception(  # noqa: TRY002
            f"{merged_file} doesn't have corresponding binary!"
        )
    print_log("start to export: ", merged_file)
    # run export
    cmd_shared_library = (
        ""
        if not shared_library_list
```
- **EN**: This chunk continues `export_target` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `export_target`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 86-91
```python
        else f" -object  {' -object '.join(shared_library_list)}"
    )
    # if binary_file = "", then no need to add it (python test)
    cmd_binary = "" if not binary_file else f" -object {binary_file} "
    llvm_tool_path = get_tool_path_by_platform(platform_type)
```
- **EN**: This chunk continues `export_target` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `export_target`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 92-102
```python
    cmd = f"{llvm_tool_path}/llvm-cov export {cmd_binary} {cmd_shared_library}  -instr-profile={merged_file} > {json_file}"
    os.system(cmd)


def merge(test_list: TestList, platform_type: TestPlatform) -> None:
    print("start merge")
    start_time = time.time()
    # find all raw profile under raw_folder and sub-folders
    raw_folder_path = get_raw_profiles_folder()
    g = os.walk(raw_folder_path)
    for path, dir_list, file_list in g:
```
- **EN**: This chunk defines `merge`, which implements a focused step inside the coverage tooling pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `merge`，其作用是实现覆盖率工具流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 103-108
```python
        # if there is a folder raw/aten/, create corresponding merged folder profile/merged/aten/ if not exists yet
        create_corresponding_folder(
            path, raw_folder_path, dir_list, MERGED_FOLDER_BASE_DIR
        )
        # check if we can find raw profile under this path's folder
        for file_name in file_list:
```
- **EN**: This chunk continues `merge` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `merge`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 109-120
```python
            if file_name.endswith(".profraw"):
                if not related_to_test_list(file_name, test_list):
                    continue
                print(f"start merge {file_name}")
                raw_file = os.path.join(path, file_name)
                merged_file_name = replace_extension(file_name, ".merged")
                merged_file = os.path.join(
                    MERGED_FOLDER_BASE_DIR,
                    convert_to_relative_path(path, raw_folder_path),
                    merged_file_name,
                )
                merge_target(raw_file, merged_file, platform_type)
```
- **EN**: This chunk continues `merge` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `merge`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 121-129
```python
    print_time("merge take time: ", start_time, summary_time=True)


def export(test_list: TestList, platform_type: TestPlatform) -> None:
    print("start export")
    start_time = time.time()
    # find all merged profile under merged_folder and sub-folders
    g = os.walk(MERGED_FOLDER_BASE_DIR)
    for path, dir_list, file_list in g:
```
- **EN**: This chunk defines `export`, which implements a focused step inside the coverage tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `export`，其作用是实现覆盖率工具流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 130-135
```python
        # create corresponding merged folder in [json folder] if not exists yet
        create_corresponding_folder(
            path, MERGED_FOLDER_BASE_DIR, dir_list, JSON_FOLDER_BASE_DIR
        )
        # check if we can find merged profile under this path's folder
        for file_name in file_list:
```
- **EN**: This chunk continues `export` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `export`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 136-147
```python
            if file_name.endswith(".merged"):
                if not related_to_test_list(file_name, test_list):
                    continue
                print(f"start export {file_name}")
                # merged file
                merged_file = os.path.join(path, file_name)
                # json file
                json_file_name = replace_extension(file_name, ".json")
                json_file = os.path.join(
                    JSON_FOLDER_BASE_DIR,
                    convert_to_relative_path(path, MERGED_FOLDER_BASE_DIR),
                    json_file_name,
```
- **EN**: This chunk continues `export` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `export`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 148-153
```python
                )
                check_platform_type(platform_type)
                # binary file and shared library
                binary_file = ""
                shared_library_list = []
                if platform_type == TestPlatform.FBCODE:
```
- **EN**: This chunk continues `export` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `export`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 154-164
```python
                    from caffe2.fb.code_coverage.tool.package.fbcode.utils import (  # type: ignore[import]
                        get_fbcode_binary_folder,
                    )

                    binary_file = os.path.join(
                        get_fbcode_binary_folder(path),
                        get_test_name_from_whole_path(merged_file),
                    )
                elif platform_type == TestPlatform.OSS:
                    from ..oss.utils import get_oss_binary_file, get_oss_shared_library
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as ..oss.utils; external packages such as caffe2.fb.code_coverage.tool.package.fbcode.utils. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 ..oss.utils；外部依赖包，如 caffe2.fb.code_coverage.tool.package.fbcode.utils。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 165-176
```python
                    test_name = get_test_name_from_whole_path(merged_file)
                    # if it is python test, no need to provide binary, shared library is enough
                    binary_file = (
                        ""
                        if test_name.endswith(".py")
                        else get_oss_binary_file(test_name, TestType.CPP)
                    )
                    shared_library_list = get_oss_shared_library()
                export_target(
                    merged_file,
                    json_file,
                    binary_file,
```
- **EN**: This chunk continues `export` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `export`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 177-180
```python
                    shared_library_list,
                    platform_type,
                )
    print_time("export take time: ", start_time, summary_time=True)
```
- **EN**: This chunk continues `export` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `export`，进一步展开其内部控制流或数据流转。

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
- **create_corresponding_folder**
  - EN: `create_corresponding_folder` is one of the main local symbols exposed or implemented here.
  - CN: `create_corresponding_folder` 是此处暴露或实现的主要局部符号之一。
- **run_target**
  - EN: `run_target` is one of the main local symbols exposed or implemented here.
  - CN: `run_target` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `..util.setting`, `..util.utils`, `.utils`, `..oss.utils`
- **Python standard library / Python 标准库**: `__future__`, `os`, `subprocess`, `time`
- **External packages / 外部依赖包**: `caffe2.fb.code_coverage.tool.package.fbcode.utils`
- **Primary symbols in this file / 本文件核心符号**: `create_corresponding_folder`, `run_target`, `merge_target`, `export_target`, `merge`, `export`
