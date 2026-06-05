# cmake.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/setup_helpers/cmake.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements build-configuration helpers that discover compilers, libraries, and platform-specific settings.
- **Purpose (CN)**: 实现构建配置辅助工具，用于发现编译器、库以及平台相关设置。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```python
"""Manages CMake."""

from __future__ import annotations

import functools
import json
import multiprocessing
import os
import platform
import shutil
import sys
import sysconfig
from pathlib import Path
from subprocess import CalledProcessError, check_call, check_output, DEVNULL
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, functools, json, and 8 more. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、functools、json 等共 11 项。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 16-25
```python
from .cmake_utils import CMakeValue, get_cmake_cache_variables_from_file
from .env import (
    BUILD_DIR,
    check_negative_env_flag,
    CMAKE_MINIMUM_VERSION_STRING,
    IS_64BIT,
    IS_DARWIN,
    IS_WINDOWS,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as .cmake_utils, .env.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 .cmake_utils、.env。

### Lines 26-36
```python

try:
    from packaging.version import Version
except ImportError:
    try:
        from setuptools.dist import Version  # type: ignore[attr-defined,no-redef]
    except ImportError:
        from distutils.version import (  # type: ignore[assignment,no-redef]
            LooseVersion as Version,
        )
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as packaging.version, setuptools.dist, distutils.version. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 packaging.version、setuptools.dist、distutils.version。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 37-45
```python

def _mkdir_p(d: str) -> None:
    try:
        os.makedirs(d, exist_ok=True)
    except OSError as e:
        raise RuntimeError(
            f"Failed to create folder {os.path.abspath(d)}: {e.strerror}"
        ) from e
```
- **EN**: This chunk defines `_mkdir_p`, which implements a focused step inside the build setup pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段定义了 `_mkdir_p`，其作用是实现构建配置流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 46-55
```python

# Print to stderr
eprint = functools.partial(print, file=sys.stderr, flush=True)


# Ninja
# Use ninja if it is on the PATH. Previous version of PyTorch required the
# ninja python package, but we no longer use it, so we do not have to import it
USE_NINJA = bool(not check_negative_env_flag("USE_NINJA") and shutil.which("ninja"))
if "CMAKE_GENERATOR" in os.environ:
```
- **EN**: This chunk continues `_mkdir_p` and expands its internal control flow or data movement. Configuration constants such as USE_NINJA centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_mkdir_p`，进一步展开其内部控制流或数据流转。 USE_NINJA 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 56-64
```python
    USE_NINJA = os.environ["CMAKE_GENERATOR"].lower() == "ninja"


CMAKE_MINIMUM_VERSION = Version(CMAKE_MINIMUM_VERSION_STRING)


class CMake:
    "Manages cmake."
```
- **EN**: It introduces classes such as CMake, which package state and behavior for this tooling task. This chunk continues `CMake` and expands its internal control flow or data movement. Configuration constants such as USE_NINJA, CMAKE_MINIMUM_VERSION centralize defaults so later functions share the same policy knobs.
- **CN**: 它引入了 CMake 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `CMake`，进一步展开其内部控制流或数据流转。 USE_NINJA、CMAKE_MINIMUM_VERSION 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 65-77
```python
    def __init__(self, build_dir: str = BUILD_DIR) -> None:
        self._cmake_command = CMake._get_cmake_command()
        self.build_dir = build_dir

    @property
    def _cmake_cache_file(self) -> str:
        r"""Returns the path to CMakeCache.txt.

        Returns:
          string: The path to CMakeCache.txt.
        """
        return os.path.join(self.build_dir, "CMakeCache.txt")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_cmake_cache_file`, which prepares build-system state and translates configuration into downstream tool invocations. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_cmake_cache_file`，其作用是准备构建系统状态，并把配置转换为后续工具调用。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 78-86
```python
    @property
    def _ninja_build_file(self) -> str:
        r"""Returns the path to build.ninja.

        Returns:
          string: The path to build.ninja.
        """
        return os.path.join(self.build_dir, "build.ninja")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_ninja_build_file`, which prepares build-system state and translates configuration into downstream tool invocations. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_ninja_build_file`，其作用是准备构建系统状态，并把配置转换为后续工具调用。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 87-96
```python
    @staticmethod
    def _get_cmake_command() -> str:
        """Returns cmake command."""

        if IS_WINDOWS:
            return "cmake"

        cmake_versions: list[str] = []
        valid_cmake_versions: dict[str, Version] = {}
        for cmd in ("cmake", "cmake3"):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_get_cmake_command`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_get_cmake_command`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 97-108
```python
            command = shutil.which(cmd)
            ver = CMake._get_version(command)
            if ver is not None:
                eprint(f"Found {cmd} ({command}) version: {ver}", end="")
                cmake_versions.append(f"{cmd}=={ver}")
                # pyrefly: ignore [unsupported-operation]
                if ver >= CMAKE_MINIMUM_VERSION:
                    eprint(f" (>={CMAKE_MINIMUM_VERSION})")
                    valid_cmake_versions[cmd] = ver
                else:
                    eprint(f" (<{CMAKE_MINIMUM_VERSION})")
```
- **EN**: This chunk continues `_get_cmake_command` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_get_cmake_command`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 109-117
```python
        if not valid_cmake_versions:
            raise RuntimeError(
                f"no cmake or cmake3 with version >= {CMAKE_MINIMUM_VERSION}, "
                f"found: {cmake_versions}"
            )
        return max(valid_cmake_versions, key=valid_cmake_versions.get)  # type: ignore[arg-type]

    @staticmethod
    def _get_version(cmd: str | None) -> Version | None:
```
- **EN**: This chunk defines `_get_version`, which implements a focused step inside the build setup pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_get_version`，其作用是实现构建配置流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 118-134
```python
        """Returns cmake version."""

        if cmd is None:
            return None

        try:
            cmake_capabilities = json.loads(
                check_output(
                    [cmd, "-E", "capabilities"],
                    stderr=DEVNULL,
                    text=True,
                ),
            )
        except (OSError, CalledProcessError, json.JSONDecodeError):
            cmake_capabilities = {}
        cmake_version = cmake_capabilities.get("version", {}).get("string")
        if cmake_version is not None:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `_get_version` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `_get_version`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 135-143
```python
            return Version(cmake_version)
        raise RuntimeError(f"Failed to get CMake version from command: {cmd}")

    def run(self, args: list[str], env: dict[str, str]) -> None:
        """Executes cmake with arguments and an environment."""

        command = [self._cmake_command] + args
        eprint(" ".join(command))
        try:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `run`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `run`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 144-152
```python
            check_call(command, cwd=self.build_dir, env=env)
        except (CalledProcessError, KeyboardInterrupt):
            # This error indicates that there was a problem with cmake, the
            # Python backtrace adds no signal here so skip over it by catching
            # the error and exiting manually
            sys.exit(1)

    @staticmethod
    def defines(args: list[str], **kwargs: CMakeValue) -> None:
```
- **EN**: This chunk defines `defines`, which implements a focused step inside the build setup pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `defines`，其作用是实现构建配置流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 153-165
```python
        """Adds definitions to a cmake argument list."""
        for key, value in sorted(kwargs.items()):
            if value is not None:
                args.append(f"-D{key}={value}")

    def get_cmake_cache_variables(self) -> dict[str, CMakeValue]:
        r"""Gets values in CMakeCache.txt into a dictionary.
        Returns:
          dict: A ``dict`` containing the value of cached CMake variables.
        """
        with open(self._cmake_cache_file) as f:
            return get_cmake_cache_variables_from_file(f)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_cmake_cache_variables`, which prepares build-system state and translates configuration into downstream tool invocations. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_cmake_cache_variables`，其作用是准备构建系统状态，并把配置转换为后续工具调用。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 166-176
```python
    def generate(
        self,
        version: str | None,
        cmake_python_library: str | None,
        build_python: bool,
        build_test: bool,
        my_env: dict[str, str],
        rerun: bool,
    ) -> None:
        """Runs cmake to generate native build files."""
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `generate`, which generates derived source text, templates, or metadata outputs.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `generate`，其作用是生成派生源码文本、模板或元数据输出。

### Lines 177-194
```python
        if rerun and os.path.isfile(self._cmake_cache_file):
            os.remove(self._cmake_cache_file)

        cmake_cache_file_available = os.path.exists(self._cmake_cache_file)
        if cmake_cache_file_available:
            cmake_cache_variables = self.get_cmake_cache_variables()
            make_program: str | None = cmake_cache_variables.get("CMAKE_MAKE_PROGRAM")  # type: ignore[assignment]
            if make_program and not shutil.which(make_program):
                # CMakeCache.txt exists, but the make program (e.g., ninja) does not.
                # See also: https://github.com/astral-sh/uv/issues/14269
                # This can happen if building with PEP-517 build isolation, where `ninja` was
                # installed in the isolated environment of the previous build run, but it has been
                # removed. The `ninja` executable with an old absolute path not available anymore.
                eprint(
                    "!!!WARNING!!!: CMakeCache.txt exists, "
                    f"but CMAKE_MAKE_PROGRAM ({make_program!r}) does not exist. "
                    "Clearing CMake cache."
                )
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 195-203
```python
                self.clear_cache()
                cmake_cache_file_available = False

        if cmake_cache_file_available and (
            not USE_NINJA or os.path.exists(self._ninja_build_file)
        ):
            # Everything's in place. Do not rerun.
            return
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 204-212
```python
        args = []
        if USE_NINJA:
            # Avoid conflicts in '-G' and the `CMAKE_GENERATOR`
            os.environ["CMAKE_GENERATOR"] = "Ninja"
            args.append("-GNinja")
        elif IS_WINDOWS:
            generator = os.getenv("CMAKE_GENERATOR", "Visual Studio 16 2019")
            supported = ["Visual Studio 16 2019", "Visual Studio 17 2022"]
            if generator not in supported:
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 213-223
```python
                eprint("Unsupported `CMAKE_GENERATOR`: " + generator)
                eprint("Please set it to one of the following values: ")
                eprint("\n".join(supported))
                sys.exit(1)
            args.append("-G" + generator)
            toolset_dict = {}
            toolset_version = os.getenv("CMAKE_GENERATOR_TOOLSET_VERSION")
            if toolset_version is not None:
                toolset_dict["version"] = toolset_version
                curr_toolset = os.getenv("VCToolsVersion")
                if curr_toolset is None:
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 224-236
```python
                    eprint(
                        "When you specify `CMAKE_GENERATOR_TOOLSET_VERSION`, you must also "
                        "activate the vs environment of this version. Please read the notes "
                        "in the build steps carefully."
                    )
                    sys.exit(1)
            if IS_64BIT:
                if platform.machine() == "ARM64":
                    args.append("-A ARM64")
                else:
                    args.append("-Ax64")
                    toolset_dict["host"] = "x64"
            if toolset_dict:
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 237-245
```python
                toolset_expr = ",".join([f"{k}={v}" for k, v in toolset_dict.items()])
                args.append("-T" + toolset_expr)

        # base_dir is used as cmake's source-dir arg and install prefix;
        # make it relative to build_dir so these are worktree-independent
        # (ccache/re-cc friendly).  cmake runs with cwd=build_dir so the
        # relative path resolves correctly.
        base_dir = str(Path(__file__).absolute().parents[2])
        if os.environ.get("USE_RELATIVE_PATHS"):
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 246-259
```python
            base_dir = os.path.relpath(
                str(Path(__file__).resolve().parents[2]), self.build_dir
            )
        install_dir = os.path.join(base_dir, "torch")

        _mkdir_p(install_dir)
        _mkdir_p(self.build_dir)

        # Environment variable forwarding (BUILD_*, USE_*, CMAKE_*, aliases,
        # passthrough vars, CMAKE_PREFIX_PATH, low-priority aliases) is now
        # handled by cmake/EnvVarForwarding.cmake, which is included early in
        # the top-level CMakeLists.txt. Only options that require Python-side
        # detection are passed here.
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 260-268
```python
        build_options: dict[str, CMakeValue] = {
            "CMAKE_INSTALL_PREFIX": install_dir,
            "BUILD_PYTHON": build_python,
            "BUILD_TEST": build_test,
        }

        use_numpy = not check_negative_env_flag("USE_NUMPY")
        build_options["USE_NUMPY"] = use_numpy
        if use_numpy:
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 269-279
```python
            try:
                import numpy

                build_options["Python_NumPy_INCLUDE_DIR"] = numpy.get_include()
            except ImportError:
                pass

        # NVSHMEM detection from Python lib path
        py_lib_path = sysconfig.get_path("purelib")
        nvshmem_py_dir = py_lib_path + "/nvidia/nvshmem"
        if os.path.exists(nvshmem_py_dir):
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as numpy. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 numpy。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 280-288
```python
            build_options["NVSHMEM_PY_DIR"] = nvshmem_py_dir

        CMake.defines(
            args,
            Python_EXECUTABLE=sys.executable,
            TORCH_BUILD_VERSION=version,
            **build_options,
        )
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Configuration constants such as TORCH_BUILD_VERSION centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 TORCH_BUILD_VERSION 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 289-297
```python
        expected_wrapper = "/usr/local/opt/ccache/libexec"
        if IS_DARWIN and os.path.exists(expected_wrapper):
            if "CMAKE_C_COMPILER" not in build_options and "CC" not in os.environ:
                CMake.defines(args, CMAKE_C_COMPILER=f"{expected_wrapper}/gcc")
            if "CMAKE_CXX_COMPILER" not in build_options and "CXX" not in os.environ:
                CMake.defines(args, CMAKE_CXX_COMPILER=f"{expected_wrapper}/g++")

        for env_var_name in my_env:
            if env_var_name.startswith("gh"):
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 298-314
```python
                # github env vars use utf-8, on windows, non-ascii code may
                # cause problem, so encode first
                try:
                    my_env[env_var_name] = str(my_env[env_var_name].encode("utf-8"))
                except UnicodeDecodeError as e:
                    shex = ":".join(f"{ord(c):02x}" for c in my_env[env_var_name])
                    eprint(f"Invalid ENV[{env_var_name}] = {shex}")
                    eprint(e)
        # According to the CMake manual, we should pass the arguments first,
        # and put the directory as the last element. Otherwise, these flags
        # may not be passed correctly.
        # Reference:
        # 1. https://cmake.org/cmake/help/latest/manual/cmake.1.html#synopsis
        # 2. https://stackoverflow.com/a/27169347
        args.append(base_dir)
        self.run(args, env=my_env)
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 315-328
```python
    def build(self, my_env: dict[str, str]) -> None:
        """Runs cmake to build binaries."""

        from .env import build_type

        build_args = [
            "--build",
            ".",
            "--target",
            "install",
            "--config",
            build_type.build_type_string,
        ]
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as .env. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `build`, which prepares build-system state and translates configuration into downstream tool invocations.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 .env。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `build`，其作用是准备构建系统状态，并把配置转换为后续工具调用。

### Lines 329-338
```python
        # Determine the parallelism according to the following
        # priorities:
        # 1) MAX_JOBS environment variable
        # 2) If using the Ninja build system, delegate decision to it.
        # 3) Otherwise, fall back to the number of processors.

        # Allow the user to set parallelism explicitly. If unset,
        # we'll try to figure it out.
        max_jobs = os.getenv("MAX_JOBS")
```
- **EN**: This chunk continues `build` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `build`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 339-350
```python
        if max_jobs is not None or not USE_NINJA:
            # Ninja is capable of figuring out the parallelism on its
            # own: only specify it explicitly if we are not using
            # Ninja.

            # This lists the number of processors available on the
            # machine. This may be an overestimate of the usable
            # processors if CPU scheduling affinity limits it
            # further. In the future, we should check for that with
            # os.sched_getaffinity(0) on platforms that support it.
            max_jobs = max_jobs or str(multiprocessing.cpu_count())
```
- **EN**: This chunk continues `build` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `build`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 351-359
```python
            # CMake 3.12 provides a '-j' option.
            build_args += ["-j", max_jobs]
        self.run(build_args, my_env)

    def clear_cache(self) -> None:
        """Clears the CMake cache."""
        if os.path.isfile(self._cmake_cache_file):
            os.remove(self._cmake_cache_file)
        if os.path.isfile(self._ninja_build_file):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `clear_cache`, which implements a focused step inside the build setup pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `clear_cache`，其作用是实现构建配置流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 360-360
```python
            os.remove(self._ninja_build_file)
```
- **EN**: This chunk continues `clear_cache` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `clear_cache`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Build setup**
  - EN: This file belongs to the build setup layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于构建配置层，应结合同一子目录中的相邻脚本一起理解。
- **Build configuration**
  - EN: The file helps discover tools, cache build settings, or translate Python-side configuration into native build arguments.
  - CN: 该文件帮助发现工具、缓存构建设置，或把 Python 侧配置转换为原生构建参数。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **_mkdir_p**
  - EN: `_mkdir_p` is one of the main local symbols exposed or implemented here.
  - CN: `_mkdir_p` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `.cmake_utils`, `.env`
- **Python standard library / Python 标准库**: `__future__`, `functools`, `json`, `multiprocessing`, `os`, `platform`, `shutil`, `sys`, `sysconfig`, `pathlib`, `subprocess`
- **External packages / 外部依赖包**: `packaging.version`, `setuptools.dist`, `distutils.version`, `numpy`
- **Primary symbols in this file / 本文件核心符号**: `_mkdir_p`, `USE_NINJA`, `CMAKE_MINIMUM_VERSION`, `CMake`
- **Note / 说明**: 17 imported modules were detected; only the first few are listed above for readability. / 检测到 17 个导入模块，为便于阅读这里只列出前若干项。
