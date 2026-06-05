# build_wheel.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/packaging/build_wheel.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements a PyTorch developer-tool script that transforms metadata, validates state, or orchestrates workflow steps.
- **Purpose (CN)**: 实现一个 PyTorch 开发工具脚本，用于转换元数据、校验状态或编排工作流步骤。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```python
#!/usr/bin/env python3

import argparse
import contextlib
import logging
import os
import re
import subprocess
import sys
import tempfile
import time
from collections.abc import Iterator
from pathlib import Path
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, contextlib, logging, and 8 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、contextlib、logging 等共 11 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 15-25
```python

logging.basicConfig(
    level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s"
)
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

ROOT_PATH = Path(__file__).absolute().parent.parent.parent
REQUIREMENTS_PATH = ROOT_PATH / "requirements.txt"
PYPROJECT_TOML_PATH = ROOT_PATH / "pyproject.toml"
```
- **EN**: Configuration constants such as ROOT_PATH, REQUIREMENTS_PATH, PYPROJECT_TOML_PATH centralize defaults so later functions share the same policy knobs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: ROOT_PATH、REQUIREMENTS_PATH、PYPROJECT_TOML_PATH 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 26-38
```python

def run_cmd(
    cmd: list[str], capture_output: bool = False
) -> subprocess.CompletedProcess[bytes]:
    logger.debug("Running command: %s", " ".join(cmd))
    return subprocess.run(
        cmd,
        # Give the parent environment to the subprocess
        env={**os.environ},
        capture_output=capture_output,
        check=True,
    )
```
- **EN**: This chunk defines `run_cmd`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Subprocess orchestration bridges this script to external build tools, linters, or system commands. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `run_cmd`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 39-47
```python

def interpreter_version(interpreter: str) -> str:
    version_string = (
        run_cmd([interpreter, "--version"], capture_output=True)
        .stdout.decode("utf-8")
        .strip()
    )
    return str(version_string.split(" ")[1])
```
- **EN**: This chunk defines `interpreter_version`, which implements a focused step inside the pytorch tooling pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `interpreter_version`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 48-57
```python

def get_supported_python_versions() -> list[str]:
    """Extract supported Python versions from pyproject.toml classifiers."""
    with open(PYPROJECT_TOML_PATH) as f:
        content = f.read()

    # Find Python version classifiers
    pattern = r'"Programming Language :: Python :: (\d+\.\d+)"'
    matches = re.findall(pattern, content)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_supported_python_versions`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_supported_python_versions`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 58-68
```python
    # Sort versions and return them
    return sorted(matches, key=lambda x: tuple(map(int, x.split("."))))


def find_python_interpreters(mode: str) -> list[str]:
    """Find Python interpreters based on the specified mode."""
    if mode == "manylinux":
        return _find_manylinux_interpreters()
    else:
        raise ValueError(f"Unsupported mode: {mode}")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `find_python_interpreters`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `find_python_interpreters`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 69-79
```python

def _find_manylinux_interpreters() -> list[str]:
    """Find Python interpreters in manylinux format (/opt/python/)."""
    supported_versions = get_supported_python_versions()
    interpreters = []

    python_root = Path("/opt/python")
    if not python_root.exists():
        logger.warning("Path /opt/python does not exist, no interpreters found")
        return []
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_find_manylinux_interpreters`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_find_manylinux_interpreters`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 80-90
```python
    # Find all python3 binaries in /opt/python/
    python_binaries = list(python_root.glob("*/bin/python3"))

    for python_path in python_binaries:
        try:
            # Check if it's PyPy (skip it)
            version_output = run_cmd(
                [str(python_path), "--version"], capture_output=True
            )
            version_string = version_output.stdout.decode("utf-8").strip()
```
- **EN**: This chunk continues `_find_manylinux_interpreters` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `_find_manylinux_interpreters`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 91-100
```python
            if "PyPy" in version_string:
                logger.debug("Skipping PyPy interpreter: %s", python_path)
                continue

            # Extract Python version (e.g., "Python 3.9.1" -> "3.9")
            match = re.search(r"Python (\d+\.\d+)", version_string)
            if not match:
                logger.debug("Could not parse version from: %s", version_string)
                continue
```
- **EN**: This chunk continues `_find_manylinux_interpreters` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_find_manylinux_interpreters`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 101-115
```python
            python_version = match.group(1)

            # Check if this version is supported
            if python_version in supported_versions:
                interpreters.append(str(python_path))
                logger.debug(
                    "Found supported Python %s at %s", python_version, python_path
                )
            else:
                logger.debug(
                    "Python %s not in supported versions: %s",
                    python_version,
                    supported_versions,
                )
```
- **EN**: This chunk continues `_find_manylinux_interpreters` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_find_manylinux_interpreters`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 116-133
```python
        except subprocess.CalledProcessError as e:
            logger.debug("Failed to get version for %s: %s", python_path, e)
            continue
    return interpreters


@contextlib.contextmanager
def venv(interpreter: str) -> Iterator[str]:
    # Should this use EnvBuilder? Probably, maybe a good todo in the future
    python_version = interpreter_version(interpreter)
    with tempfile.TemporaryDirectory(
        suffix=f"_pytorch_builder_{python_version}"
    ) as tmp_dir:
        logger.info(
            "Creating virtual environment (Python %s) at %s",
            python_version,
            tmp_dir,
        )
```
- **EN**: This chunk defines `venv`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `venv`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 134-142
```python
        run_cmd([interpreter, "-m", "venv", tmp_dir])
        yield str(Path(tmp_dir) / "bin" / "python3")


class Builder:
    # The python interpreter that we should be using
    interpreter: str

    def __init__(self, interpreter: str) -> None:
```
- **EN**: It introduces classes such as Builder, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the pytorch tooling pipeline.
- **CN**: 它引入了 Builder 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。

### Lines 143-160
```python
        self.interpreter = interpreter

    def build_wheel(self, destination: str) -> bool:
        logger.info("Running bdist_wheel -d %s", destination)
        return (
            run_cmd(
                [
                    self.interpreter,
                    "-m",
                    "build",
                    "--wheel",
                    "--no-isolation",
                    "--outdir",
                    destination,
                    str(ROOT_PATH),
                ]
            ).returncode
            == 0
```
- **EN**: This chunk defines `build_wheel`, which prepares build-system state and translates configuration into downstream tool invocations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `build_wheel`，其作用是准备构建系统状态，并把配置转换为后续工具调用。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 161-172
```python
        )

    def clean(self) -> bool:
        logger.info("Running clean")
        return run_cmd([self.interpreter, "setup.py", "clean"]).returncode == 0

    def install_requirements(self) -> None:
        logger.info("Installing requirements")
        run_cmd(
            [self.interpreter, "-m", "pip", "install", "-r", str(REQUIREMENTS_PATH)]
        )
```
- **EN**: This chunk defines `install_requirements`, which implements a focused step inside the pytorch tooling pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `install_requirements`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 173-190
```python

def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "-p",
        "--python",
        action="append",
        type=str,
        help=(
            "Python interpreters to build packages for, can be set multiple times,"
            " should ideally be full paths, (default: %(default)s)"
        ),
    )
    parser.add_argument(
        "--find-python",
        type=str,
        choices=["manylinux"],
        help=(
```
- **EN**: This chunk defines `parse_args`, which parses or loads structured input into tool-friendly data structures. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `parse_args`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 191-204
```python
            "Automatically find Python interpreters based on the specified mode. "
            "Available modes: 'manylinux' (searches /opt/python/ for interpreters "
            "matching supported versions in pyproject.toml)"
        ),
    )
    parser.add_argument(
        "-d",
        "--destination",
        default="dist/",
        type=str,
        help="Destination to put the compiled binaries",
    )
    return parser.parse_args()
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 205-215
```python

def main() -> None:
    args = parse_args()

    if args.find_python:
        if args.python:
            logger.warning(
                "Both --python and --find-python specified. Using --find-python and ignoring --python."
            )
        pythons = find_python_interpreters(args.find_python)
        if not pythons:
```
- **EN**: This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 216-227
```python
            logger.error(
                "No Python interpreters found with --find-python %s", args.find_python
            )
            sys.exit(1)
        logger.info(
            "Found %d supported Python interpreters: %s",
            len(pythons),
            ", ".join(pythons),
        )
    else:
        pythons = args.python or [sys.executable]
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。

### Lines 228-242
```python
    build_times: dict[str, float] = dict()

    if len(pythons) > 1 and args.destination == "dist/":
        logger.warning(
            "dest is 'dist/' while multiple python versions specified, output will be overwritten"
        )

    for interpreter in pythons:
        with venv(interpreter) as venv_interpreter:
            builder = Builder(venv_interpreter)
            # clean actually requires setuptools so we need to ensure we
            # install requirements before
            builder.install_requirements()
            builder.clean()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 243-252
```python
            start_time = time.time()

            builder.build_wheel(args.destination)

            end_time = time.time()

            build_times[interpreter_version(venv_interpreter)] = end_time - start_time
    for version, build_time in build_times.items():
        logger.info("Build time (%s): %fs", version, build_time)
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 253-255
```python

if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **PyTorch tooling**
  - EN: This file belongs to the pytorch tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于PyTorch 工具链层，应结合同一子目录中的相邻脚本一起理解。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **ROOT_PATH**
  - EN: `ROOT_PATH` is one of the main local symbols exposed or implemented here.
  - CN: `ROOT_PATH` 是此处暴露或实现的主要局部符号之一。
- **REQUIREMENTS_PATH**
  - EN: `REQUIREMENTS_PATH` is one of the main local symbols exposed or implemented here.
  - CN: `REQUIREMENTS_PATH` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `argparse`, `contextlib`, `logging`, `os`, `re`, `subprocess`, `sys`, `tempfile`, `time`, `collections.abc`, `pathlib`
- **Primary symbols in this file / 本文件核心符号**: `ROOT_PATH`, `REQUIREMENTS_PATH`, `PYPROJECT_TOML_PATH`, `run_cmd`, `interpreter_version`, `get_supported_python_versions`, `find_python_interpreters`, `_find_manylinux_interpreters`, `venv`, `Builder`, `parse_args`, `main`
