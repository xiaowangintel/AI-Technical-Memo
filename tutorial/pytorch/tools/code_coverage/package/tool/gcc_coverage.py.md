# gcc_coverage.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/code_coverage/package/tool/gcc_coverage.py`
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

### Lines 7-12
```python
# gcc is only used in oss
from ..oss.utils import get_gcda_files, run_oss_python_test
from ..util.setting import JSON_FOLDER_BASE_DIR, TestType
from ..util.utils import print_log, print_time
from .utils import run_cpp_test
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as ..oss.utils, ..util.setting, ..util.utils, and 1 more. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 ..oss.utils、..util.setting、..util.utils 等共 4 项。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 13-19
```python

def update_gzip_dict(gzip_dict: dict[str, int], file_name: str) -> str:
    file_name = file_name.lower()
    gzip_dict[file_name] = gzip_dict.get(file_name, 0) + 1
    num = gzip_dict[file_name]
    return str(num) + "_" + file_name
```
- **EN**: This chunk defines `update_gzip_dict`, which implements a focused step inside the coverage tooling pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `update_gzip_dict`，其作用是实现覆盖率工具流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 20-24
```python

def run_target(binary_file: str, test_type: TestType) -> None:
    print_log("start run", test_type.value, "test: ", binary_file)
    start_time = time.time()
    if test_type not in {TestType.CPP, TestType.PY}:
```
- **EN**: This chunk defines `run_target`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `run_target`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 25-30
```python
        raise AssertionError(f"Invalid test_type: {test_type}")
    if test_type == TestType.CPP:
        run_cpp_test(binary_file)
    else:
        run_oss_python_test(binary_file)
```
- **EN**: This chunk continues `run_target` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `run_target`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 31-34
```python
    print_time(" time: ", start_time)


def export() -> None:
```
- **EN**: This chunk defines `export`, which implements a focused step inside the coverage tooling pipeline.
- **CN**: 这一段定义了 `export`，其作用是实现覆盖率工具流水线中的一个关键步骤。

### Lines 35-40
```python
    start_time = time.time()
    # collect .gcda files
    gcda_files = get_gcda_files()
    # file name like utils.cpp may have same name in different folder
    gzip_dict: dict[str, int] = {}
    for gcda_item in gcda_files:
```
- **EN**: This chunk continues `export` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `export`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 41-48
```python
        # generate json.gz
        subprocess.check_call(["gcov", "-i", gcda_item])
        # cp json.gz to profile/json folder
        gz_file_name = os.path.basename(gcda_item) + ".gcov.json.gz"
        new_file_path = os.path.join(
            JSON_FOLDER_BASE_DIR, update_gzip_dict(gzip_dict, gz_file_name)
        )
        os.rename(gz_file_name, new_file_path)
```
- **EN**: This chunk continues `export` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `export`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 49-51
```python
        #  unzip json.gz to json
        subprocess.check_output(["gzip", "-d", new_file_path])
    print_time("export take time: ", start_time, summary_time=True)
```
- **EN**: This chunk continues `export` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `export`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

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
- **update_gzip_dict**
  - EN: `update_gzip_dict` is one of the main local symbols exposed or implemented here.
  - CN: `update_gzip_dict` 是此处暴露或实现的主要局部符号之一。
- **run_target**
  - EN: `run_target` is one of the main local symbols exposed or implemented here.
  - CN: `run_target` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `..oss.utils`, `..util.setting`, `..util.utils`, `.utils`
- **Python standard library / Python 标准库**: `__future__`, `os`, `subprocess`, `time`
- **Primary symbols in this file / 本文件核心符号**: `update_gzip_dict`, `run_target`, `export`
