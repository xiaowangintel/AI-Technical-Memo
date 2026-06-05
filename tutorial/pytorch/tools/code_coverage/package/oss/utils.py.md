# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/code_coverage/package/oss/utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements code-coverage parsing, aggregation, or reporting helpers for CI analysis.
- **Purpose (CN)**: 实现代码覆盖率解析、聚合或报告辅助逻辑，用于 CI 分析。
## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```python
from __future__ import annotations

import os
import subprocess

from ..util.setting import CompilerType, TestType, TOOLS_FOLDER
from ..util.utils import print_error, remove_file
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as ..util.setting, ..util.utils; Python standard-library modules such as __future__, os, subprocess. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 ..util.setting、..util.utils；Python 标准库模块，如 __future__、os、subprocess。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 9-17
```python

def get_oss_binary_folder(test_type: TestType) -> str:
    if test_type not in {TestType.CPP, TestType.PY}:
        raise AssertionError(f"Invalid test_type: {test_type}")
    # TODO: change the way we get binary file -- binary may not in build/bin ?
    return os.path.join(
        get_pytorch_folder(), "build/bin" if test_type == TestType.CPP else "test"
    )
```
- **EN**: This chunk defines `get_oss_binary_folder`, which implements a focused step inside the coverage tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_oss_binary_folder`，其作用是实现覆盖率工具流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 18-23
```python

def get_oss_shared_library() -> list[str]:
    lib_dir = os.path.join(get_pytorch_folder(), "build", "lib")
    return [
        os.path.join(lib_dir, lib)
        for lib in os.listdir(lib_dir)
```
- **EN**: This chunk defines `get_oss_shared_library`, which implements a focused step inside the coverage tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_oss_shared_library`，其作用是实现覆盖率工具流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 24-29
```python
        if lib.endswith(".dylib")
    ]


def get_oss_binary_file(test_name: str, test_type: TestType) -> str:
    if test_type not in {TestType.CPP, TestType.PY}:
```
- **EN**: This chunk defines `get_oss_binary_file`, which implements a focused step inside the coverage tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `get_oss_binary_file`，其作用是实现覆盖率工具流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 30-37
```python
        raise AssertionError(f"Invalid test_type: {test_type}")
    binary_folder = get_oss_binary_folder(test_type)
    binary_file = os.path.join(binary_folder, test_name)
    if test_type == TestType.PY:
        # add python to the command so we can directly run the script by using binary_file variable
        binary_file = "python " + binary_file
    return binary_file
```
- **EN**: This chunk continues `get_oss_binary_file` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_oss_binary_file`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 38-43
```python

def get_llvm_tool_path() -> str:
    return os.environ.get(
        "LLVM_TOOL_PATH", "/usr/local/opt/llvm/bin"
    )  # set default as llvm path in dev server, on mac the default may be /usr/local/opt/llvm/bin
```
- **EN**: This chunk defines `get_llvm_tool_path`, which implements a focused step inside the coverage tooling pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_llvm_tool_path`，其作用是实现覆盖率工具流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 44-50
```python

def get_pytorch_folder() -> str:
    # TOOLS_FOLDER in oss: pytorch/tools/code_coverage
    return os.path.abspath(
        os.environ.get("PYTORCH_FOLDER", os.path.dirname(os.path.dirname(TOOLS_FOLDER)))
    )
```
- **EN**: This chunk defines `get_pytorch_folder`, which implements a focused step inside the coverage tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_pytorch_folder`，其作用是实现覆盖率工具流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 51-56
```python

def detect_compiler_type() -> CompilerType | None:
    # check if user specifies the compiler type
    user_specify = os.environ.get("CXX", None)
    if user_specify:
        if user_specify in ["clang", "clang++"]:
```
- **EN**: This chunk defines `detect_compiler_type`, which implements a focused step inside the coverage tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `detect_compiler_type`，其作用是实现覆盖率工具流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 57-62
```python
            return CompilerType.CLANG
        elif user_specify in ["gcc", "g++"]:
            return CompilerType.GCC

        raise RuntimeError(f"User specified compiler is not valid {user_specify}")
```
- **EN**: This chunk continues `detect_compiler_type` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `detect_compiler_type`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 63-72
```python
    # auto detect
    auto_detect_result = subprocess.check_output(
        ["cc", "-v"], stderr=subprocess.STDOUT
    ).decode("utf-8")
    if "clang" in auto_detect_result:
        return CompilerType.CLANG
    elif "gcc" in auto_detect_result:
        return CompilerType.GCC
    raise RuntimeError(f"Auto detected compiler is not valid {auto_detect_result}")
```
- **EN**: This chunk continues `detect_compiler_type` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `detect_compiler_type`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 73-78
```python

def clean_up_gcda() -> None:
    gcda_files = get_gcda_files()
    for item in gcda_files:
        remove_file(item)
```
- **EN**: This chunk defines `clean_up_gcda`, which implements a focused step inside the coverage tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `clean_up_gcda`，其作用是实现覆盖率工具流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 79-89
```python

def get_gcda_files() -> list[str]:
    folder_has_gcda = os.path.join(get_pytorch_folder(), "build")
    if os.path.isdir(folder_has_gcda):
        # TODO use glob
        # output = glob.glob(f"{folder_has_gcda}/**/*.gcda")
        output = subprocess.check_output(["find", folder_has_gcda, "-iname", "*.gcda"])
        return output.decode("utf-8").split("\n")
    else:
        return []
```
- **EN**: This chunk defines `get_gcda_files`, which implements a focused step inside the coverage tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_gcda_files`，其作用是实现覆盖率工具流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 90-98
```python

def run_oss_python_test(binary_file: str) -> None:
    # python test script
    try:
        subprocess.check_call(
            binary_file, shell=True, cwd=get_oss_binary_folder(TestType.PY)
        )
    except subprocess.CalledProcessError:
        print_error(f"Binary failed to run: {binary_file}")
```
- **EN**: This chunk defines `run_oss_python_test`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `run_oss_python_test`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

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
- **get_oss_binary_folder**
  - EN: `get_oss_binary_folder` is one of the main local symbols exposed or implemented here.
  - CN: `get_oss_binary_folder` 是此处暴露或实现的主要局部符号之一。
- **get_oss_shared_library**
  - EN: `get_oss_shared_library` is one of the main local symbols exposed or implemented here.
  - CN: `get_oss_shared_library` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `..util.setting`, `..util.utils`
- **Python standard library / Python 标准库**: `__future__`, `os`, `subprocess`
- **Primary symbols in this file / 本文件核心符号**: `get_oss_binary_folder`, `get_oss_shared_library`, `get_oss_binary_file`, `get_llvm_tool_path`, `get_pytorch_folder`, `detect_compiler_type`, `clean_up_gcda`, `get_gcda_files`, `run_oss_python_test`
