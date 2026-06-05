# nightly.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/nightly.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements a PyTorch developer-tool script that transforms metadata, validates state, or orchestrates workflow steps.
- **Purpose (CN)**: 实现一个 PyTorch 开发工具脚本，用于转换元数据、校验状态或编排工作流步骤。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
#!/usr/bin/env python3
# Much of the logging code here was forked from https://github.com/ezyang/ghstack
# Copyright (c) Edward Z. Yang <ezyang@mit.edu>
r"""Checks out the nightly development version of PyTorch and installs pre-built
binaries into the repo.

You can use this script to check out a new nightly branch with the following::

    $ ./tools/nightly.py checkout -b my-nightly-branch
    $ source venv/bin/activate  # or `. .\venv\Scripts\activate` on Windows

Or if you would like to check out the nightly commit in detached HEAD mode::

    $ ./tools/nightly.py checkout
    $ source venv/bin/activate  # or `. .\venv\Scripts\activate` on Windows
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 17-32
```python
Or if you would like to reuse an existing virtual environment, you can pass in
the prefix argument (--prefix)::

    $ ./tools/nightly.py checkout -b my-nightly-branch -p my-env
    $ source my-env/bin/activate  # or `& .\my-env\Scripts\Activate.ps1` on Windows

To install the nightly binaries built with CUDA, you can pass in the flag --cuda::

    $ ./tools/nightly.py checkout -b my-nightly-branch --cuda
    $ source venv/bin/activate  # or `. .\venv\Scripts\activate` on Windows

To install the nightly binaries built with ROCm, you can pass in the flag --rocm::

    $ ./tools/nightly.py checkout -b my-nightly-branch --rocm
    $ source venv/bin/activate  # or `. .\venv\Scripts\activate` on Windows
```
- **EN**: Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 33-48
```python
You can also use this tool to pull the nightly commits into the current branch as
well. This can be done with::

    $ ./tools/nightly.py pull
    $ source venv/bin/activate  # or `. .\venv\Scripts\activate` on Windows

To create the virtual environment with a specific Python interpreter, you can
pass in the --python argument::

    $ ./tools/nightly.py --python /path/to/python3.12
    $ source venv/bin/activate  # or `. .\venv\Scripts\activate` on Windows

Pulling will recreate a fresh virtual environment and reinstall the development
dependencies as well as the nightly binaries into the repo directory.
"""
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。

### Lines 49-74
```python
from __future__ import annotations

import argparse
import atexit
import contextlib
import functools
import itertools
import json
import logging
import os
import re
import shlex
import shutil
import subprocess
import sys
import tempfile
import textwrap
import time
import uuid
from ast import literal_eval
from collections.abc import Callable
from datetime import datetime
from pathlib import Path
from platform import system as platform_system
from typing import Any, cast, NamedTuple, TYPE_CHECKING, TypeVar
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, argparse, atexit, and 21 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、argparse、atexit 等共 24 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 75-105
```python

if TYPE_CHECKING:
    from collections.abc import Generator, Iterable, Iterator


try:
    from packaging.version import Version
except ImportError:
    Version = None  # type: ignore[assignment,misc]


REPO_ROOT = Path(__file__).absolute().parent.parent
GITHUB_REMOTE_URL = "https://github.com/pytorch/pytorch.git"
PACKAGES_TO_INSTALL = (
    "torch",
    "numpy",
    "cmake",
    "ninja",
    "packaging",
    "ruff",
    "mypy",
    "pytest",
    "hypothesis",
    "ipython",
    "rich",
    "clang-format",
    "clang-tidy",
    "sphinx",
)
DEFAULT_VENV_DIR = REPO_ROOT / "venv"
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as collections.abc; external packages such as packaging.version. Configuration constants such as REPO_ROOT, GITHUB_REMOTE_URL, PACKAGES_TO_INSTALL, and 1 more centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 collections.abc；外部依赖包，如 packaging.version。 REPO_ROOT、GITHUB_REMOTE_URL、PACKAGES_TO_INSTALL 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 106-123
```python

LOGGER: logging.Logger | None = None
VERBOSE: bool = False
DATETIME_FORMAT = "%Y-%m-%d_%Hh%Mm%Ss"
SHA1_RE = re.compile(r"(?P<sha1>[0-9a-fA-F]{40})")
USERNAME_PASSWORD_RE = re.compile(r":\/\/(.*?)\@")
LOG_DIRNAME_RE = re.compile(
    r"(?P<datetime>\d{4}-\d\d-\d\d_\d\dh\d\dm\d\ds)_"
    r"(?P<uuid>[0-9a-f]{8}-(?:[0-9a-f]{4}-){3}[0-9a-f]{12})",
)


PLATFORM = platform_system().replace("Darwin", "macOS")
LINUX = PLATFORM == "Linux"
MACOS = PLATFORM == "macOS"
WINDOWS = PLATFORM == "Windows"
POSIX = LINUX or MACOS
```
- **EN**: Configuration constants such as DATETIME_FORMAT, SHA1_RE, USERNAME_PASSWORD_RE, and 6 more centralize defaults so later functions share the same policy knobs. Pattern-matching logic extracts structured facts from loosely formatted text inputs.
- **CN**: DATETIME_FORMAT、SHA1_RE、USERNAME_PASSWORD_RE 等共 9 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。

### Lines 124-140
```python

class PipSource(NamedTuple):
    name: str
    index_url: str
    supported_platforms: set[str]
    accelerator: str


# Generate: .github/scripts/nightly_source_matrix.json
GENERATE_MATRIX_SCRIPT = (
    REPO_ROOT / ".github" / "scripts" / "generate_binary_build_matrix.py"
)
subprocess.check_call(
    [sys.executable, str(GENERATE_MATRIX_SCRIPT)],
    cwd=GENERATE_MATRIX_SCRIPT.parent,
)
```
- **EN**: It introduces classes such as PipSource, which package state and behavior for this tooling task. This chunk continues `PipSource` and expands its internal control flow or data movement. Configuration constants such as GENERATE_MATRIX_SCRIPT centralize defaults so later functions share the same policy knobs. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 它引入了 PipSource 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `PipSource`，进一步展开其内部控制流或数据流转。 GENERATE_MATRIX_SCRIPT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 141-157
```python
# See: .github/scripts/nightly_source_matrix.json
NIGHTLY_SOURCE_FILE = GENERATE_MATRIX_SCRIPT.with_name("nightly_source_matrix.json")
NIGHTLY_SOURCE_MATRIX = json.loads(NIGHTLY_SOURCE_FILE.read_text(encoding="utf-8"))
PIP_SOURCES = {name: PipSource(**data) for name, data in NIGHTLY_SOURCE_MATRIX.items()}


class Formatter(logging.Formatter):
    redactions: dict[str, str]

    def __init__(self, fmt: str | None = None, datefmt: str | None = None) -> None:
        super().__init__(fmt, datefmt)
        self.redactions = {}

    # Remove sensitive information from URLs
    def _filter(self, s: str) -> str:
        s = USERNAME_PASSWORD_RE.sub(r"://<USERNAME>:<PASSWORD>@", s)
        for needle, replace in self.redactions.items():
```
- **EN**: It introduces classes such as Formatter, which package state and behavior for this tooling task. This chunk defines `_filter`, which narrows a larger candidate set down to the items relevant for the current workflow. Configuration constants such as NIGHTLY_SOURCE_FILE, NIGHTLY_SOURCE_MATRIX, PIP_SOURCES centralize defaults so later functions share the same policy knobs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 它引入了 Formatter 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `_filter`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 NIGHTLY_SOURCE_FILE、NIGHTLY_SOURCE_MATRIX、PIP_SOURCES 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 158-173
```python
            s = s.replace(needle, replace)
        return s

    def formatMessage(self, record: logging.LogRecord) -> str:
        if record.levelno == logging.INFO or record.levelno == logging.DEBUG:
            # Log INFO/DEBUG without any adornment
            return record.getMessage()
        else:
            # I'm not sure why, but formatMessage doesn't show up
            # even though it's in the typeshed for Python >3
            return super().formatMessage(record)

    def format(self, record: logging.LogRecord) -> str:
        return self._filter(super().format(record))

    def redact(self, needle: str, replace: str = "<REDACTED>") -> None:
```
- **EN**: This chunk defines `redact`, which implements a focused step inside the pytorch tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `redact`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 174-191
```python
        """Redact specific strings; e.g., authorization tokens.  This won't
        retroactively redact stuff you've already leaked, so make sure
        you redact things as soon as possible.
        """
        # Don't redact empty strings; this will lead to something
        # that looks like s<REDACTED>t<REDACTED>r<REDACTED>...
        if needle == "":
            return
        self.redactions[needle] = replace


@contextlib.contextmanager
def timer(logger: logging.Logger, prefix: str) -> Iterator[None]:
    """Timed context manager"""
    start_time = time.perf_counter()
    yield
    logger.info("%s took %.3f [s]", prefix, time.perf_counter() - start_time)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `timer`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `timer`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 192-208
```python

F = TypeVar("F", bound=Callable[..., Any])


def timed(prefix: str) -> Callable[[F], F]:
    """Decorator for timing functions"""

    def decorator(f: F) -> F:
        @functools.wraps(f)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            logger = cast(logging.Logger, LOGGER)
            logger.info(prefix)
            with timer(logger, prefix):
                return f(*args, **kwargs)

        return cast(F, wrapper)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `wrapper`, which implements a focused step inside the pytorch tooling pipeline. Configuration constants such as F centralize defaults so later functions share the same policy knobs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `wrapper`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 F 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 209-224
```python
    return decorator


class Venv:
    """Virtual environment manager"""

    AGGRESSIVE_UPDATE_PACKAGES = (
        "uv",
        "pip",
        "setuptools<82",
        "packaging",
        "wheel",
        "build[uv]",
    )

    def __init__(
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as Venv, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the pytorch tooling pipeline. Configuration constants such as AGGRESSIVE_UPDATE_PACKAGES centralize defaults so later functions share the same policy knobs. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 Venv 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 AGGRESSIVE_UPDATE_PACKAGES 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 225-252
```python
        self,
        prefix: Path | str,
        pip_source: PipSource,
        *,
        base_executable: Path | str | None = None,
    ) -> None:
        base_executable = Path(base_executable or sys.executable)
        if not base_executable.is_absolute():
            base_exec = shutil.which(str(base_executable))
            if base_exec is None:
                raise RuntimeError(
                    f"Could not find Python executable {base_executable}",
                )
            base_executable = Path(base_exec)

        self.prefix = Path(prefix).absolute()
        self.pip_source = pip_source
        self.base_executable = base_executable.absolute()
        self._executable: Path | None = None
        self._bindir: Path | None = None
        self._env = {
            "PIP_EXTRA_INDEX_URL": self.pip_source.index_url,
            "UV_INDEX": self.pip_source.index_url,
            "UV_PYTHON_DOWNLOADS": "never",
            "FORCE_COLOR": "1",
            "CLICOLOR_FORCE": "1",
        }
```
- **EN**: This chunk continues `__init__` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `__init__`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 253-268
```python
    def is_venv(self) -> bool:
        """Check if the prefix is a virtual environment."""
        return self.prefix.is_dir() and (self.prefix / "pyvenv.cfg").is_file()

    @property
    def bindir(self) -> Path:
        """Get the bin directory for the virtual environment."""
        if not self.is_venv():
            raise AssertionError("Not a virtual environment")
        if self._bindir is None:
            if WINDOWS:
                self._bindir = self.prefix / "Scripts"
            else:
                self._bindir = self.prefix / "bin"
        return self._bindir
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `bindir`, which implements a focused step inside the pytorch tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `bindir`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 269-294
```python
    @property
    def executable(self) -> Path:
        """Get the Python executable for the virtual environment."""
        if not self.is_venv():
            raise AssertionError("Not a virtual environment")
        if self._executable is None:
            executable = self.bindir / ("python.exe" if WINDOWS else "python")
            if not (executable.is_file() or executable.is_symlink()):
                raise AssertionError(f"{executable} is not a file or symlink")
            if not os.access(executable, os.X_OK):
                raise AssertionError(f"{executable} is not executable")
            self._executable = executable
        return self._executable

    def site_packages(self, python: Path | str | None = None) -> Path:
        """Get the site-packages directory for the virtual environment."""
        output = self.python(
            "-c",
            "import site; [print(p) for p in site.getsitepackages()]",
            python=python,
            capture_output=True,
        ).stdout
        # pyrefly: ignore [bad-argument-type]
        candidates = list(map(Path, filter(None, map(str.strip, output.splitlines()))))
        candidates = [p for p in candidates if p.is_dir() and p.name == "site-packages"]
        if not candidates:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `site_packages`, which implements a focused step inside the pytorch tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `site_packages`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 295-310
```python
            raise RuntimeError(
                f"No site-packages directory found for executable {python}"
            )
        return candidates[0]

    @property
    def activate_script(self) -> Path:
        """Get the activation script for the virtual environment."""
        if WINDOWS:
            # Assume PowerShell
            return self.prefix / "Scripts" / "activate"
        # Assume POSIX-compliant shell: Bash, Zsh, etc.
        return self.prefix / "bin" / "activate"

    @property
    def activate_command(self) -> str:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `activate_command`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `activate_command`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 311-327
```python
        """Get the command to activate the virtual environment."""
        if WINDOWS:
            # Assume PowerShell
            return f'. "{self.activate_script}"'
        # Assume Bash, Zsh, etc.
        # POSIX standard should use dot `. venv/bin/activate` rather than `source`
        return f"source {shlex.quote(str(self.activate_script))}"

    @timed("Creating virtual environment")
    def create(
        self,
        *,
        remove_if_exists: bool = False,
        assume_yes: bool = False,
    ) -> Path:
        """Create a virtual environment."""
        if self.prefix.exists():
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `create`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `create`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 328-349
```python
            if remove_if_exists:
                # If the venv directory already exists, remove it first
                if not self.is_venv():
                    raise RuntimeError(
                        f"The path {self.prefix} already exists and is not a virtual environment. "
                        "Please remove it manually or choose a different prefix."
                    )
                if any(
                    Path(p).absolute().samefile(self.prefix)
                    for p in [
                        sys.prefix,
                        sys.exec_prefix,
                        sys.base_prefix,
                        sys.base_exec_prefix,
                    ]
                ):
                    raise RuntimeError(
                        f"The path {self.prefix} trying to remove is the same as the interpreter "
                        "to run this script. Please choose a different prefix or deactivate the "
                        "current virtual environment."
                    )
                if any(
```
- **EN**: This chunk continues `create` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `create`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 350-371
```python
                    Path(
                        self.base_python(
                            "-c",
                            f"import os, sys; print(os.path.abspath({p}))",
                            capture_output=True,
                        ).stdout.strip()
                    )
                    .absolute()
                    .samefile(self.prefix)
                    for p in [
                        "sys.prefix",
                        "sys.exec_prefix",
                        "sys.base_prefix",
                        "sys.base_exec_prefix",
                    ]
                ):
                    raise RuntimeError(
                        f"The Python executable {self.base_executable} trying to remove is the "
                        "same as the interpreter to create the virtual environment. Please choose "
                        "a different prefix or a different Python interpreter."
                    )
                if not assume_yes:
```
- **EN**: This chunk continues `create` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `create`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 372-387
```python
                    answer = input(
                        f"The virtual environment {self.prefix} already exists. "
                        "Do you want to remove it and recreate it? [y/N] "
                    )
                    if answer.lower() not in ("y", "yes"):
                        if answer.lower() not in ("n", "no", ""):
                            print(f"Invalid answer: {answer!r}")
                        else:
                            print(f"Aborting due to existing prefix: {self.prefix}")
                        sys.exit(1)

                print(f"Removing existing venv: {self.prefix}")
                _remove_existing(self.prefix)
            else:
                raise RuntimeError(f"Path {self.prefix} already exists.")
```
- **EN**: This chunk continues `create` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `create`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 388-411
```python
        print(f"Creating venv (Python {self.base_python_version()}): {self.prefix}")
        self.base_python("-m", "venv", str(self.prefix))
        if not self.is_venv():
            raise AssertionError("Failed to create virtual environment.")
        (self.prefix / ".gitignore").write_text("*\n", encoding="utf-8")

        if LINUX:
            activate_script = self.activate_script
            st_mode = activate_script.stat().st_mode
            # The activate script may be read-only and we need to add write permissions
            activate_script.chmod(st_mode | 0o200)
            with activate_script.open(mode="a", encoding="utf-8") as f:
                f.write(
                    "\n"
                    + textwrap.dedent(
                        f"""
                        # Add NVIDIA PyPI packages to LD_LIBRARY_PATH
                        export LD_LIBRARY_PATH="$(
                            {self.executable.name} - <<EOS
                        import glob
                        import itertools
                        import os
                        import site
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as glob, itertools, os, and 1 more. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 glob、itertools、os 等共 4 项。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 412-430
```python
                        nvidia_libs = [
                            p.rstrip("/")
                            for p in itertools.chain.from_iterable(
                                glob.iglob(f"{{site_dir}}/{{pattern}}/", recursive=True)
                                for site_dir in site.getsitepackages()
                                for pattern in ("nvidia/**/lib", "cu*/**/lib")
                            )
                        ]
                        ld_library_path = os.getenv("LD_LIBRARY_PATH", "").split(os.pathsep)
                        print(os.pathsep.join(dict.fromkeys(nvidia_libs + ld_library_path)))
                        EOS
                        )"
                        """
                    ).strip()
                    + "\n"
                )
            # Change the file mode back
            activate_script.chmod(st_mode)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `create` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `create`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 431-446
```python
        return self.ensure()

    def ensure(self) -> Path:
        """Ensure the virtual environment exists."""
        if not self.is_venv():
            return self.create(remove_if_exists=True)
        if (
            self.python_version().split(".")[:2]
            != self.base_python_version().split(".")[:2]
        ):
            raise RuntimeError(
                f"Python version mismatch: venv has Python {self.python_version()} "
                f"but base Python is {self.base_python_version()}. "
                "Please recreate the virtual environment with the correct Python version."
            )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `ensure`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `ensure`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 447-471
```python
        self.pip_install(*self.AGGRESSIVE_UPDATE_PACKAGES, upgrade=True)
        return self.prefix

    def python(
        self,
        *args: str,
        python: Path | str | None = None,
        **popen_kwargs: Any,
    ) -> subprocess.CompletedProcess[str]:
        """Run a Python command in the virtual environment."""
        if python is None:
            python = self.executable
        cmd = [str(python), *args]
        env = popen_kwargs.pop("env", None) or {}
        check = popen_kwargs.pop("check", True)
        # pyrefly: ignore [no-matching-overload]
        return subprocess.run(
            cmd,
            check=check,
            text=True,
            encoding="utf-8",
            env={**os.environ, **self._env, **env},
            **popen_kwargs,
        )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `python`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `python`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 472-494
```python
    def base_python(
        self,
        *args: str,
        **popen_kwargs: Any,
    ) -> subprocess.CompletedProcess[str]:
        """Run a Python command in the base environment."""
        return self.python(*args, python=self.base_executable, **popen_kwargs)

    def python_version(self, *, python: Path | str | None = None) -> str:
        """Get the Python version for the virtual environment.

        Return a string like "3.13.7", "3.13.7t", "3.13.7d", "3.13.7td", etc.
        """
        return self.python(
            "-c",
            (
                "import sys; print('{0.major}.{0.minor}.{0.micro}{1}'."
                "format(sys.version_info, getattr(sys, 'abiflags', '')))"
            ),
            python=python,
            capture_output=True,
        ).stdout.strip()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `python_version`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `python_version`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 495-523
```python
    def base_python_version(self) -> str:
        """Get the Python version for the base environment.

        Return a string like "3.13.7", "3.13.7t", "3.13.7d", "3.13.7td", etc.
        """
        return self.python_version(python=self.base_executable)

    def uv(
        self,
        *args: str,
        python: Path | str | None = None,
        **popen_kwargs: Any,
    ) -> subprocess.CompletedProcess[str]:
        """Run a uv command in the virtual environment."""
        if python is None:
            python = self.executable
        cmd = [str(self.bindir / "uv"), *args]
        env = popen_kwargs.pop("env", None) or {}
        check = popen_kwargs.pop("check", True)
        # pyrefly: ignore [no-matching-overload]
        return subprocess.run(
            cmd,
            check=check,
            text=True,
            encoding="utf-8",
            env={**os.environ, **self._env, **env, "UV_PYTHON": str(python)},
            **popen_kwargs,
        )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `uv`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `uv`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 524-539
```python
    @timed("Installing packages")
    def uv_pip_install(
        self,
        *packages: str,
        prerelease: bool = False,
        upgrade: bool = False,
        no_deps: bool = False,
        **popen_kwargs: Any,
    ) -> subprocess.CompletedProcess[str]:
        """Run a pip install command in the virtual environment."""
        uv_pip_args = []
        if VERBOSE:
            uv_pip_args.append("-v")
        if prerelease:
            uv_pip_args.append("--prerelease")
        if upgrade:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `uv_pip_install`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `uv_pip_install`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 540-555
```python
            uv_pip_args.append("--upgrade")
            verb = "Upgrading"
        else:
            verb = "Installing"
        if no_deps:
            uv_pip_args.append("--no-deps")
        print(f"{verb} package(s) ({self.pip_source.index_url}):")
        for package in packages:
            print(f"  - {os.path.basename(package)}")
        return self.uv("pip", "install", *uv_pip_args, *packages, **popen_kwargs)

    def pip(self, *args: str, **popen_kwargs: Any) -> subprocess.CompletedProcess[str]:
        """Run a pip command in the virtual environment."""
        return self.python("-m", "pip", *args, **popen_kwargs)

    @timed("Installing packages")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `pip`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `pip`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 556-575
```python
    def pip_install(
        self,
        *packages: str,
        prerelease: bool = False,
        upgrade: bool = False,
        no_deps: bool = False,
        **popen_kwargs: Any,
    ) -> subprocess.CompletedProcess[str]:
        """Run a pip install command in the virtual environment."""
        pip_args = []
        if VERBOSE:
            pip_args.append("-v")
        if prerelease:
            pip_args.append("--pre")
        if upgrade:
            pip_args.append("--upgrade")
            verb = "Upgrading"
        else:
            verb = "Installing"
        if no_deps:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `pip_install`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `pip_install`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 576-599
```python
            pip_args.append("--no-deps")
        print(f"{verb} package(s) ({self.pip_source.index_url}):")
        for package in packages:
            print(f"  - {os.path.basename(package)}")
        return self.pip("install", *pip_args, *packages, **popen_kwargs)

    @timed("Downloading packages")
    def pip_download(
        self,
        *packages: str,
        prerelease: bool = False,
        no_deps: bool = False,
        **popen_kwargs: Any,
    ) -> list[Path]:
        """Download a package in the virtual environment."""
        tmpdir = tempfile.TemporaryDirectory(prefix="pip-download-")
        atexit.register(tmpdir.cleanup)
        tempdir = Path(tmpdir.name).absolute()
        print(
            f"Downloading package(s) ({self.pip_source.index_url}): "
            f"{', '.join(packages)}"
        )
        pip_args = []
        if VERBOSE:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `pip_download`, which parses or loads structured input into tool-friendly data structures. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `pip_download`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 600-619
```python
            pip_args.append("-v")
        if prerelease:
            pip_args.append("--pre")
        if no_deps:
            pip_args.append("--no-deps")
        self.pip("download", f"--dest={tempdir}", *pip_args, *packages, **popen_kwargs)
        files = list(tempdir.iterdir())
        print(f"Downloaded {len(files)} file(s) to {tempdir}:")
        for file in files:
            print(f"  - {file.name}")
        return files

    def wheel(
        self,
        *args: str,
        **popen_kwargs: Any,
    ) -> subprocess.CompletedProcess[str]:
        """Run a wheel command in the virtual environment."""
        return self.python("-m", "wheel", *args, **popen_kwargs)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `wheel`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `wheel`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 620-635
```python
    @timed("Unpacking wheel file")
    def wheel_unpack(
        self,
        wheel: Path | str,
        dest: Path | str,
        **popen_kwargs: Any,
    ) -> subprocess.CompletedProcess[str]:
        """Unpack a wheel into a directory."""
        wheel = Path(wheel).absolute()
        dest = Path(dest).absolute()
        if not (wheel.is_file() and wheel.suffix.lower() == ".whl"):
            raise AssertionError(f"{wheel} is not a valid wheel file")
        return self.wheel("unpack", f"--dest={dest}", str(wheel), **popen_kwargs)

    @contextlib.contextmanager
    def extracted_wheel(self, wheel: Path | str) -> Generator[Path]:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `extracted_wheel`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `extracted_wheel`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 636-651
```python
        """Download and extract a wheel into a temporary directory."""
        with tempfile.TemporaryDirectory(prefix="wheel-") as tempdir:
            self.wheel_unpack(wheel, tempdir)
            subdirs = [p for p in Path(tempdir).absolute().iterdir() if p.is_dir()]
            if len(subdirs) != 1:
                raise RuntimeError(
                    f"Expected exactly one directory in {tempdir}, "
                    f"got {[str(d) for d in subdirs]}."
                )
            yield subdirs[0]


def git(*args: str) -> list[str]:
    return ["git", "-C", str(REPO_ROOT), *args]
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `git`, which implements a focused step inside the pytorch tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `git`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 652-667
```python
@functools.lru_cache
def logging_base_dir() -> Path:
    base_dir = REPO_ROOT / "nightly" / "log"
    base_dir.mkdir(parents=True, exist_ok=True)
    return base_dir


@functools.lru_cache
def logging_run_dir() -> Path:
    base_dir = logging_base_dir()
    cur_dir = base_dir / f"{datetime.now().strftime(DATETIME_FORMAT)}_{uuid.uuid1()}"
    cur_dir.mkdir(parents=True, exist_ok=True)
    return cur_dir


@functools.lru_cache
```
- **EN**: This chunk defines `logging_run_dir`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `logging_run_dir`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 668-683
```python
def logging_record_argv() -> None:
    s = subprocess.list2cmdline(sys.argv)
    (logging_run_dir() / "argv").write_text(s, encoding="utf-8")


def logging_record_exception(e: BaseException) -> None:
    text = f"{type(e).__name__}: {e}"
    if isinstance(e, subprocess.CalledProcessError):
        text += f"\n\nstdout: {e.stdout}\n\nstderr: {e.stderr}"
    (logging_run_dir() / "exception").write_text(text, encoding="utf-8")


def logging_rotate() -> None:
    log_base = logging_base_dir()
    old_logs = sorted(log_base.iterdir(), reverse=True)
    for stale_log in old_logs[1000:]:
```
- **EN**: This chunk defines `logging_rotate`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `logging_rotate`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 684-701
```python
        # Sanity check that it looks like a log
        if LOG_DIRNAME_RE.fullmatch(stale_log.name) is not None:
            shutil.rmtree(stale_log)


@contextlib.contextmanager
def logging_manager(*, debug: bool = False) -> Generator[logging.Logger, None, None]:
    """Setup logging. If a failure starts here we won't
    be able to save the user in a reasonable way.

    Logging structure: there is one logger (the root logger)
    and in processes all events.  There are two handlers:
    stderr (INFO) and file handler (DEBUG).
    """
    formatter = Formatter(fmt="%(levelname)s: %(message)s", datefmt="")
    root_logger = logging.getLogger("pytorch-nightly")
    root_logger.setLevel(logging.DEBUG)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `logging_manager`, which implements a focused step inside the pytorch tooling pipeline. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `logging_manager`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 702-717
```python
    console_handler = logging.StreamHandler()
    if debug:
        console_handler.setLevel(logging.DEBUG)
    else:
        console_handler.setLevel(logging.INFO)
    console_handler.setFormatter(formatter)
    root_logger.addHandler(console_handler)

    log_file = logging_run_dir() / "nightly.log"

    file_handler = logging.FileHandler(log_file)
    file_handler.setFormatter(formatter)
    root_logger.addHandler(file_handler)
    logging_record_argv()

    try:
```
- **EN**: This chunk continues `logging_manager` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `logging_manager`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 718-734
```python
        logging_rotate()
        print(f"log file: {log_file}")
        yield root_logger
    except Exception as e:
        logging.exception("Fatal exception")  # noqa: LOG015
        logging_record_exception(e)
        print(f"log file: {log_file}")
        sys.exit(1)
    except BaseException as e:
        # You could logging.debug here to suppress the backtrace
        # entirely, but there is no reason to hide it from technically
        # savvy users.
        logging.info("", exc_info=True)  # noqa: LOG015
        logging_record_exception(e)
        print(f"log file: {log_file}")
        sys.exit(1)
```
- **EN**: This chunk continues `logging_manager` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `logging_manager`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 735-752
```python

def check_branch(subcommand: str, branch: str | None) -> str | None:
    """Checks that the branch name can be checked out."""
    if subcommand != "checkout":
        return None
    # next check that the local repo is clean
    cmd = git("status", "--untracked-files=no", "--porcelain")
    stdout = subprocess.check_output(cmd, text=True, encoding="utf-8")
    if stdout.strip():
        return "Need to have clean working tree to checkout!\n\n" + stdout
    # next check that the branch name doesn't already exist (if a branch name is provided)
    if branch is not None:
        cmd = git("show-ref", "--verify", "--quiet", f"refs/heads/{branch}")
        p = subprocess.run(cmd, capture_output=True, check=False)  # type: ignore[assignment]
        if not p.returncode:
            return f"Branch {branch!r} already exists"
    return None
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `check_branch`, which validates invariants and reports policy violations early. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `check_branch`，其作用是校验不变量，并尽早报告策略违规。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 753-773
```python

@timed("Installing dependencies")
def install_packages(venv: Venv, packages: Iterable[str]) -> None:
    """Install dependencies to deps environment"""
    # install packages
    packages = list(dict.fromkeys(packages))
    if packages:
        venv.uv_pip_install(*packages)


def _ensure_commit(git_sha1: str) -> None:
    """Make sure that we actually have the commit locally"""
    cmd = git("cat-file", "-e", git_sha1 + r"^{commit}")
    p = subprocess.run(cmd, capture_output=True, check=False)
    if p.returncode == 0:
        # we have the commit locally
        return
    # we don't have the commit, must fetch
    cmd = git("fetch", GITHUB_REMOTE_URL, git_sha1)
    subprocess.check_call(cmd)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_ensure_commit`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_ensure_commit`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 774-793
```python

def _nightly_version(site_dir: Path) -> str:
    # first get the git version from the installed module
    version_file = site_dir / "torch" / "version.py"
    with version_file.open(encoding="utf-8") as f:
        for line in f:
            if not line.startswith("git_version"):
                continue
            git_version = literal_eval(line.partition("=")[2].strip())
            break
        else:
            raise RuntimeError(f"Could not find git_version in {version_file}")

    print(f"Found released git version {git_version}")
    # now cross reference with nightly version
    _ensure_commit(git_version)
    cmd = git("show", "--no-patch", "--format=%s", git_version)
    stdout = subprocess.check_output(cmd, text=True, encoding="utf-8")
    m = SHA1_RE.search(stdout)
    if m is None:
```
- **EN**: This chunk defines `_nightly_version`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段定义了 `_nightly_version`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 794-815
```python
        raise RuntimeError(
            f"Could not find nightly release in git history:\n  {stdout}"
        )
    nightly_version = m.group("sha1")
    print(f"Found nightly release version {nightly_version}")
    # now checkout nightly version
    _ensure_commit(nightly_version)
    return nightly_version


@timed("Checking out nightly PyTorch")
def checkout_nightly_version(branch: str | None, site_dir: Path) -> None:
    """Gets the nightly version and then checks it out."""
    nightly_version = _nightly_version(site_dir)
    if branch is None:
        # Detached mode - explicitly use --detach flag
        cmd = git("checkout", "--detach", nightly_version)
    else:
        # Branch mode
        cmd = git("checkout", "-b", branch, nightly_version)
    subprocess.check_call(cmd)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `checkout_nightly_version`, which validates invariants and reports policy violations early. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `checkout_nightly_version`，其作用是校验不变量，并尽早报告策略违规。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 816-833
```python

@timed("Pulling nightly PyTorch")
def pull_nightly_version(site_dir: Path) -> None:
    """Fetches the nightly version and then merges it ."""
    nightly_version = _nightly_version(site_dir)
    cmd = git("merge", nightly_version)
    subprocess.check_call(cmd)


def _get_listing_linux(source_dir: Path) -> list[Path]:
    return list(
        itertools.chain(
            source_dir.glob("*.so"),
            (source_dir / "lib").glob("*.so"),
            (source_dir / "lib").glob("*.so.*"),
        )
    )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_get_listing_linux`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_get_listing_linux`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 834-853
```python

def _get_listing_macos(source_dir: Path) -> list[Path]:
    # oddly, these are .so files even on Mac
    return list(
        itertools.chain(
            source_dir.glob("*.so"),
            (source_dir / "lib").glob("*.dylib"),
        )
    )


def _get_listing_windows(source_dir: Path) -> list[Path]:
    return list(
        itertools.chain(
            source_dir.glob("*.pyd"),
            (source_dir / "lib").glob("*.lib"),
            (source_dir / "lib").glob(".dll"),
        )
    )
```
- **EN**: This chunk defines `_get_listing_windows`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_get_listing_windows`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 854-881
```python

def _glob_pyis(d: Path) -> set[str]:
    return {p.relative_to(d).as_posix() for p in d.rglob("*.pyi")}


def _find_missing_pyi(source_dir: Path, target_dir: Path) -> list[Path]:
    source_pyis = _glob_pyis(source_dir)
    target_pyis = _glob_pyis(target_dir)
    missing_pyis = sorted(source_dir / p for p in (source_pyis - target_pyis))
    return missing_pyis


def _get_listing(source_dir: Path, target_dir: Path) -> list[Path]:
    if LINUX:
        listing = _get_listing_linux(source_dir)
    elif MACOS:
        listing = _get_listing_macos(source_dir)
    elif WINDOWS:
        listing = _get_listing_windows(source_dir)
    else:
        raise RuntimeError(f"Platform {platform_system()!r} not recognized")
    listing.extend(_find_missing_pyi(source_dir, target_dir))
    listing.append(source_dir / "version.py")
    listing.append(source_dir / "testing" / "_internal" / "generated")
    listing.append(source_dir / "bin")
    listing.append(source_dir / "include")
    return listing
```
- **EN**: This chunk defines `_get_listing`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_get_listing`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 882-902
```python

def _remove_existing(path: Path) -> None:
    if path.exists():
        if path.is_dir():
            shutil.rmtree(path)
        else:
            path.unlink()


def _move_single(
    src: Path,
    source_dir: Path,
    target_dir: Path,
    mover: Callable[[Path, Path], None],
    verb: str,
) -> None:
    relpath = src.relative_to(source_dir)
    trg = target_dir / relpath
    _remove_existing(trg)
    # move over new files
    if src.is_dir():
```
- **EN**: This chunk defines `_move_single`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `_move_single`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 903-919
```python
        trg.mkdir(parents=True, exist_ok=True)
        for root, dirs, files in os.walk(src):
            relroot = Path(root).relative_to(src)
            for name in files:
                relname = relroot / name
                s = src / relname
                t = trg / relname
                if VERBOSE:
                    print(f"{verb} {s} -> {t}")
                mover(s, t)
            for name in dirs:
                (trg / relroot / name).mkdir(parents=True, exist_ok=True)
    else:
        if VERBOSE:
            print(f"{verb} {src} -> {trg}")
        mover(src, trg)
```
- **EN**: This chunk continues `_move_single` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_move_single`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 920-940
```python

def _copy_files(listing: list[Path], source_dir: Path, target_dir: Path) -> None:
    for src in listing:
        # pyrefly: ignore [bad-argument-type]
        _move_single(src, source_dir, target_dir, shutil.copy2, "Copying")


def _link_files(listing: list[Path], source_dir: Path, target_dir: Path) -> None:
    for src in listing:
        _move_single(src, source_dir, target_dir, os.link, "Linking")


@timed("Moving nightly files into repo")
def move_nightly_files(site_dir: Path) -> None:
    """Moves PyTorch files from temporary installed location to repo."""
    # get file listing
    source_dir = site_dir / "torch"
    target_dir = REPO_ROOT / "torch"
    listing = _get_listing(source_dir, target_dir)
    # copy / link files
    if WINDOWS:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `move_nightly_files`, which implements a focused step inside the pytorch tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `move_nightly_files`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 941-959
```python
        _copy_files(listing, source_dir, target_dir)
    else:
        try:
            _link_files(listing, source_dir, target_dir)
        except Exception:
            _copy_files(listing, source_dir, target_dir)


@timed("Writing pytorch-nightly.pth")
def write_pth(venv: Venv) -> None:
    """Writes Python path file for this dir."""
    (venv.site_packages() / "pytorch-nightly.pth").write_text(
        "# This file was autogenerated by PyTorch's tools/nightly.py\n"
        "# Please delete this file if you no longer need the following development\n"
        "# version of PyTorch to be importable\n"
        f"{REPO_ROOT}\n",
        encoding="utf-8",
    )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `write_pth`, which generates derived source text, templates, or metadata outputs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `write_pth`，其作用是生成派生源码文本、模板或元数据输出。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 960-977
```python

def parse_dependencies(
    venv: Venv,
    wheel_site_dir: Path,
) -> list[str]:
    """Parse dependencies from the torch wheel's metadata."""
    dist_info_dirs = list(wheel_site_dir.glob("*.dist-info"))
    if len(dist_info_dirs) != 1:
        raise RuntimeError(
            f"Expected exactly one .dist-info directory in {wheel_site_dir}, "
            f"got {dist_info_dirs}"
        )
    dist_info_dir = dist_info_dirs[0]
    if not (dist_info_dir / "METADATA").is_file():
        raise RuntimeError(
            f"Expected METADATA file in {dist_info_dir}, but it does not exist."
        )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `parse_dependencies`, which parses or loads structured input into tool-friendly data structures. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `parse_dependencies`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 978-1001
```python
    # Use the Python interpreter in the virtual environment instead of the interpreter
    # running this script, so that we can evaluate markers correctly.
    dependencies = (
        venv.python(
            "-c",
            textwrap.dedent(
                """
                from packaging.metadata import Metadata

                with open("METADATA", encoding="utf-8") as f:
                    metadata = Metadata.from_email(f.read())
                for req in metadata.requires_dist:
                    if req.marker is None or req.marker.evaluate():
                        print(req)
                """
            ).strip(),
            cwd=dist_info_dir,
            capture_output=True,
        )
        .stdout.strip()
        .splitlines()
    )
    return [dep.strip() for dep in dependencies]
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as packaging.metadata. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 packaging.metadata。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 1002-1018
```python

def install(
    *,
    venv: Venv,
    packages: Iterable[str],
    subcommand: str = "checkout",
    branch: str | None = None,
    fresh_venv: bool = False,
    assume_yes: bool = False,
) -> None:
    """Development install of PyTorch"""
    if not fresh_venv:
        print(f"Using existing venv: {venv.prefix}")
        venv.ensure()
    else:
        venv.create(remove_if_exists=True, assume_yes=assume_yes)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `install`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `install`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 1019-1034
```python
    packages = [p for p in packages if p != "torch"]

    downloaded_files = venv.pip_download("torch", prerelease=True, no_deps=True)
    if len(downloaded_files) != 1:
        raise RuntimeError(f"Expected exactly one torch wheel, got {downloaded_files}")
    torch_wheel = downloaded_files[0]
    if not (
        torch_wheel.name.startswith("torch-") and torch_wheel.name.endswith(".whl")
    ):
        raise RuntimeError(f"Expected exactly one torch wheel, got {torch_wheel}")

    with venv.extracted_wheel(torch_wheel) as wheel_site_dir:
        dependencies = parse_dependencies(venv, wheel_site_dir)
        install_packages(venv, [*dependencies, *packages])

        if subcommand == "checkout":
```
- **EN**: This chunk continues `install` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `install`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 1035-1050
```python
            checkout_nightly_version(branch, wheel_site_dir)
        elif subcommand == "pull":
            pull_nightly_version(wheel_site_dir)
        else:
            raise ValueError(f"Subcommand {subcommand} must be one of: checkout, pull.")
        move_nightly_files(wheel_site_dir)

    write_pth(venv)
    cast(logging.Logger, LOGGER).info(
        "-------\n"
        "PyTorch Development Environment set up!\n"
        "Please activate to enable this environment:\n\n"
        "  $ %s\n",
        venv.activate_command,
    )
```
- **EN**: This chunk continues `install` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `install`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 1051-1078
```python

def make_parser() -> argparse.ArgumentParser:
    def find_executable(name: str) -> Path:
        executable = shutil.which(name)
        if executable is None:
            raise argparse.ArgumentTypeError(
                f"Could not find executable {name} in PATH."
            )
        return Path(executable).absolute()

    parser = argparse.ArgumentParser()
    # subcommands
    subcmd = parser.add_subparsers(dest="subcmd", help="subcommand to execute")
    checkout = subcmd.add_parser("checkout", help="checkout a new branch")
    checkout.add_argument(
        "-b",
        "--branch",
        help="Branch name to checkout (if omitted, checks out in detached HEAD mode)",
        dest="branch",
        default=None,
        metavar="NAME",
    )
    pull = subcmd.add_parser(
        "pull", help="pulls the nightly commits into the current branch"
    )
    # general arguments
    subparsers = [checkout, pull]
    for subparser in subparsers:
```
- **EN**: This chunk defines `find_executable`, which implements a focused step inside the pytorch tooling pipeline. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `find_executable`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 1079-1110
```python
        subparser.add_argument(
            "--python",
            "--base-executable",
            type=find_executable,
            help=(
                "Path to Python interpreter to use for creating the virtual environment. "
                "Defaults to the interpreter running this script."
            ),
            dest="base_executable",
            default=None,
            metavar="PYTHON",
        )
        subparser.add_argument(
            "--prefix",
            "-p",
            type=lambda p: Path(p).absolute(),
            help='Path to virtual environment directory (e.g. "./venv")',
            dest="prefix",
            default=str(DEFAULT_VENV_DIR),
            metavar="PATH",
        )
        subparser.add_argument(
            "--fresh",
            help="Remove existing virtual environment if it exists",
            dest="fresh",
            default=False,
            action="store_true",
        )
        subparser.add_argument(
            "--yes",
            "-y",
            help=(
```
- **EN**: This chunk continues `find_executable` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `find_executable`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 1111-1142
```python
                "Automatic yes to prompts; assume 'yes' as answer to all prompts "
                "(e.g., removing existing venv)"
            ),
            dest="yes",
            default=False,
            action="store_true",
        )
        subparser.add_argument(
            "--verbose",
            "-v",
            help="Provide debugging info",
            dest="verbose",
            default=False,
            action="store_true",
        )
        subparser.add_argument(
            "--cuda",
            help=(
                "CUDA version to install "
                "(defaults to the latest version available on the platform)"
            ),
            dest="cuda",
            nargs="?",
            default=argparse.SUPPRESS,
            metavar="VERSION",
        )
        subparser.add_argument(
            "--rocm",
            help=(
                "ROCm version to install "
                "(defaults to the latest version available on the platform)"
            ),
```
- **EN**: This chunk continues `find_executable` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `find_executable`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 1143-1158
```python
            dest="rocm",
            nargs="?",
            default=argparse.SUPPRESS,
            metavar="VERSION",
        )
    return parser


def parse_arguments() -> argparse.Namespace:
    parser = make_parser()
    args = parser.parse_args()
    args.branch = getattr(args, "branch", None)
    if hasattr(args, "cuda") and hasattr(args, "rocm"):
        parser.error("Cannot specify both CUDA and ROCm versions.")
    return args
```
- **EN**: This chunk defines `parse_arguments`, which parses or loads structured input into tool-friendly data structures. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `parse_arguments`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 1159-1177
```python

def main() -> None:
    """Main entry point"""
    global LOGGER, VERBOSE
    args = parse_arguments()
    VERBOSE = args.verbose

    status = check_branch(args.subcmd, args.branch)
    if status:
        sys.exit(status)

    pip_source = None
    for toolkit in ("CUDA", "ROCm"):
        accel = toolkit.lower()
        if hasattr(args, accel):
            requested = getattr(args, accel)
            available_sources = {
                src.name[len(f"{accel}-") :]: src
                for src in PIP_SOURCES.values()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Configuration constants such as VERBOSE centralize defaults so later functions share the same policy knobs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 VERBOSE 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 1178-1194
```python
                if src.name.startswith(f"{accel}-")
                and PLATFORM in src.supported_platforms
            }
            if not available_sources:
                print(f"No {toolkit} versions available on platform {PLATFORM}.")
                sys.exit(1)
            if requested is not None:
                pip_source = available_sources.get(requested)
                if pip_source is None:
                    print(
                        f"{toolkit} {requested} is not available on platform {PLATFORM}. "
                        f"Available version(s): {', '.join(sorted(available_sources, key=Version))}"
                    )
                    sys.exit(1)
            else:
                pip_source = available_sources[max(available_sources, key=Version)]
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 1195-1213
```python
    if pip_source is None:
        pip_source = PIP_SOURCES["cpu"]  # always available

    with logging_manager(debug=args.verbose) as logger:
        LOGGER = logger
        venv = Venv(
            prefix=args.prefix,
            pip_source=pip_source,
            base_executable=args.base_executable,
        )
        install(
            venv=venv,
            packages=PACKAGES_TO_INSTALL,
            subcommand=args.subcmd,
            branch=args.branch,
            fresh_venv=args.fresh,
            assume_yes=args.yes,
        )
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Configuration constants such as LOGGER centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 LOGGER 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 1214-1216
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
- **Build configuration**
  - EN: The file helps discover tools, cache build settings, or translate Python-side configuration into native build arguments.
  - CN: 该文件帮助发现工具、缓存构建设置，或把 Python 侧配置转换为原生构建参数。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Pytest integration**
  - EN: The file plugs into pytest collection, execution, or reporting behavior.
  - CN: 该文件接入 pytest 的收集、执行或报告行为。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `argparse`, `atexit`, `contextlib`, `functools`, `itertools`, `json`, `logging`, `os`, `re`, `shlex`, `shutil`, `subprocess`, `sys`, `tempfile`
- **External packages / 外部依赖包**: `packaging.version`, `packaging.metadata`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `GITHUB_REMOTE_URL`, `PACKAGES_TO_INSTALL`, `DEFAULT_VENV_DIR`, `DATETIME_FORMAT`, `SHA1_RE`, `USERNAME_PASSWORD_RE`, `LOG_DIRNAME_RE`, `PLATFORM`, `LINUX`, `MACOS`, `WINDOWS`
- **Note / 说明**: 28 imported modules were detected; only the first few are listed above for readability. / 检测到 28 个导入模块，为便于阅读这里只列出前若干项。
