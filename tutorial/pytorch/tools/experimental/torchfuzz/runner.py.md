# runner.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/experimental/torchfuzz/runner.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements experimental fuzzing helpers used to stress PyTorch operators or APIs.
- **Purpose (CN)**: 实现实验性模糊测试辅助逻辑，用于压测 PyTorch 算子或 API。
## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```python
"""
Program runner utilities for PyTorch fuzzer.
This module handles running and testing generated PyTorch programs.
"""

import os
import random
import subprocess
import sys
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as os, random, subprocess, and 1 more. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 os、random、subprocess 等共 4 项。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 11-17
```python

class ProgramRunner:
    """Runs generated PyTorch programs and handles output/error reporting."""

    def __init__(self):
        pass
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as ProgramRunner, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the torch fuzzing pipeline.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 ProgramRunner 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。

### Lines 18-24
```python
    def run_program(self, program_path):
        """
        Run a generated Python program and handle output/errors.

        Args:
            program_path: Path to the Python program to run
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `run_program`, which orchestrates command-line execution and forwards parsed arguments into the core logic.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `run_program`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。

### Lines 25-30
```python
        Returns:
            bool: True if program ran successfully, False otherwise
        """
        abs_path = os.path.abspath(program_path)
        print(f"Running: {abs_path}")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `run_program` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `run_program`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 31-37
```python
        # Select a random CUDA device if available
        cuda_visible_devices = os.environ.get("CUDA_VISIBLE_DEVICES")
        if cuda_visible_devices:
            devices = [d.strip() for d in cuda_visible_devices.split(",") if d.strip()]
        else:
            # Default to all GPUs if not set
            try:
```
- **EN**: This chunk continues `run_program` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `run_program`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 38-44
```python
                import torch

                num_gpus = torch.cuda.device_count()
                devices = [str(i) for i in range(num_gpus)]
            except ImportError:
                devices = []
        if devices:
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as torch. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 torch。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 45-51
```python
            selected_device = random.choice(devices)
            env = os.environ.copy()
            env["CUDA_VISIBLE_DEVICES"] = selected_device
            print(f"Selected CUDA_VISIBLE_DEVICES={selected_device}")
        else:
            env = None  # No GPU available or torch not installed
```
- **EN**: This chunk continues `run_program` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `run_program`，进一步展开其内部控制流或数据流转。

### Lines 52-63
```python
        try:
            result = subprocess.run(
                [sys.executable, abs_path],
                capture_output=True,
                text=True,
                check=True,
                env=env,
            )
            print("=== Program Output ===")
            print(result.stdout)
            print(result.stderr)
            return True
```
- **EN**: This chunk continues `run_program` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `run_program`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 64-75
```python

        except subprocess.CalledProcessError as e:
            print("=== Program Output (Failure) ===")
            print(e.stdout)
            print(e.stderr)
            print("===============================")
            print("=== Program Source ===")
            with open(abs_path) as f:
                print(f.read())
            print("======================")
            print(f"Program exited with code: {e.returncode}")
            sys.exit(1)
```
- **EN**: This chunk continues `run_program` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `run_program`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 76-83
```python

    def run_and_validate(self, program_path):
        """
        Run a program and return detailed results for validation.

        Args:
            program_path: Path to the Python program to run
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `run_and_validate`, which validates invariants and reports policy violations early. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `run_and_validate`，其作用是校验不变量，并尽早报告策略违规。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 84-91
```python
        Returns:
            dict: Dictionary with 'success', 'stdout', 'stderr', 'returncode'
        """
        abs_path = os.path.abspath(program_path)

        # Select a random CUDA device if available
        cuda_visible_devices = os.environ.get("CUDA_VISIBLE_DEVICES")
        if cuda_visible_devices:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `run_and_validate` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `run_and_validate`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 92-98
```python
            devices = [d.strip() for d in cuda_visible_devices.split(",") if d.strip()]
        else:
            try:
                import torch

                num_gpus = torch.cuda.device_count()
                if num_gpus > 1:
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as torch. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 torch。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 99-104
```python
                    devices = [str(i) for i in range(1, num_gpus)]
                else:
                    devices = [str(i) for i in range(num_gpus)]
            except ImportError:
                devices = []
        if devices:
```
- **EN**: This chunk continues `run_and_validate` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `run_and_validate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 105-111
```python
            selected_device = random.choice(devices)
            env = os.environ.copy()
            env["CUDA_VISIBLE_DEVICES"] = selected_device
            print(f"Selected CUDA_VISIBLE_DEVICES={selected_device}")
        else:
            env = None
```
- **EN**: This chunk continues `run_and_validate` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `run_and_validate`，进一步展开其内部控制流或数据流转。

### Lines 112-123
```python
        try:
            result = subprocess.run(
                [sys.executable, abs_path],
                capture_output=True,
                text=True,
                check=True,
                env=env,
            )
            return {
                "success": True,
                "stdout": result.stdout,
                "stderr": result.stderr,
```
- **EN**: This chunk continues `run_and_validate` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `run_and_validate`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 124-133
```python
                "returncode": result.returncode,
            }

        except subprocess.CalledProcessError as e:
            return {
                "success": False,
                "stdout": e.stdout,
                "stderr": e.stderr,
                "returncode": e.returncode,
            }
```
- **EN**: This chunk continues `run_and_validate` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `run_and_validate`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **Torch fuzzing**
  - EN: This file belongs to the torch fuzzing layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Torch 模糊测试层，应结合同一子目录中的相邻脚本一起理解。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Fuzz testing**
  - EN: The implementation creates randomized or adversarial inputs to probe API robustness.
  - CN: 该实现构造随机或对抗性输入，以探测 API 的健壮性。
- **ProgramRunner**
  - EN: `ProgramRunner` is one of the main local symbols exposed or implemented here.
  - CN: `ProgramRunner` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `torch`
- **Python standard library / Python 标准库**: `os`, `random`, `subprocess`, `sys`
- **Primary symbols in this file / 本文件核心符号**: `ProgramRunner`
