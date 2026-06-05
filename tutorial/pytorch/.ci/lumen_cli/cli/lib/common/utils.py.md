# utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `.ci/lumen_cli/cli/lib/common/utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6 / 第 1-6 行

```python
"""
General Utility helpers for CLI tasks.
"""

from __future__ import annotations

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 7-14 / 第 7-14 行

```python
import logging
import os
import shlex
import subprocess
import sys
from contextlib import contextmanager
from pathlib import Path

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 15-26 / 第 15-26 行

```python

logger = logging.getLogger(__name__)


def run_command(
    cmd: str,
    use_shell: bool = False,
    log_cmd: bool = True,
    cwd: str | None = None,
    env: dict | None = None,
    check: bool = True,
) -> int:
```

- **EN:** Important local symbols in this block include run_command.
- **CN:** 该代码块中的重要局部符号包括 run_command。

### Lines 27-36 / 第 27-36 行

```python
    """Run a command with optional shell execution."""
    if use_shell:
        args = cmd
        log_prefix = "[shell]"
        executable = "/bin/bash"
    else:
        args = shlex.split(cmd)
        log_prefix = "[cmd]"
        executable = None

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 37-42 / 第 37-42 行

```python
    if log_cmd:
        display_cmd = cmd if use_shell else " ".join(args)
        logger.info("%s %s", log_prefix, display_cmd)

    run_env = {**os.environ, **(env or {})}

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 43-53 / 第 43-53 行

```python
    proc = subprocess.run(
        args,
        shell=use_shell,
        executable=executable,
        stdout=sys.stdout,
        stderr=sys.stderr,
        cwd=cwd,
        env=run_env,
        check=False,
    )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 54-61 / 第 54-61 行

```python
    if check and proc.returncode != 0:
        logger.error(
            "%s Command failed (exit %s): %s", log_prefix, proc.returncode, cmd
        )
        raise subprocess.CalledProcessError(
            proc.returncode, args if not use_shell else cmd
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 62-67 / 第 62-67 行

```python
    return proc.returncode


def str2bool(value: str | None) -> bool:
    """Convert environment variables to boolean values."""
    if not value:
```

- **EN:** Important local symbols in this block include str2bool.
- **CN:** 该代码块中的重要局部符号包括 str2bool。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 68-74 / 第 68-74 行

```python
        return False
    if not isinstance(value, str):
        raise ValueError(
            f"Expected a string value for boolean conversion, got {type(value)}"
        )
    value = value.strip().lower()

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 75-80 / 第 75-80 行

```python
    true_value_set = {"1", "true", "t", "yes", "y", "on", "enable", "enabled", "found"}
    false_value_set = {"0", "false", "f", "no", "n", "off", "disable"}

    if value in true_value_set:
        return True
    if value in false_value_set:
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 81-86 / 第 81-86 行

```python
        return False
    raise ValueError(f"Invalid string value for boolean conversion: {value}")


@contextmanager
def temp_environ(updates: dict[str, str]):
```

- **EN:** Important local symbols in this block include temp_environ.
- **CN:** 该代码块中的重要局部符号包括 temp_environ。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 87-98 / 第 87-98 行

```python
    """
    Temporarily set environment variables and restore them after the block.
    Args:
        updates: Dict of environment variables to set.
    """
    missing = object()
    old: dict[str, str | object] = {k: os.environ.get(k, missing) for k in updates}
    try:
        os.environ.update(updates)
        yield
    finally:
        for k, v in old.items():
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 99-104 / 第 99-104 行

```python
            if v is missing:
                os.environ.pop(k, None)
            else:
                os.environ[k] = v  # type: ignore[arg-type]


```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 105-110 / 第 105-110 行

```python
@contextmanager
def working_directory(path: str):
    """
    Temporarily change the working directory inside a context.
    """
    if not path:
```

- **EN:** Important local symbols in this block include working_directory.
- **CN:** 该代码块中的重要局部符号包括 working_directory。

### Lines 111-120 / 第 111-120 行

```python
        # No-op context
        yield
        return
    prev_cwd = os.getcwd()
    try:
        os.chdir(path)
        yield
    finally:
        os.chdir(prev_cwd)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 121-128 / 第 121-128 行

```python

def get_wheels(
    output_dir: Path,
    max_depth: int | None = None,
) -> list[str]:
    """Return a list of wheels found in the given output directory."""
    root = Path(output_dir)
    if not root.exists():
```

- **EN:** Important local symbols in this block include get_wheels.
- **CN:** 该代码块中的重要局部符号包括 get_wheels。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 129-135 / 第 129-135 行

```python
        return []
    items = []
    for dirpath, _, filenames in os.walk(root):
        depth = Path(dirpath).relative_to(root).parts
        if max_depth is not None and len(depth) > max_depth:
            continue
        for fname in sorted(filenames):
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 136-140 / 第 136-140 行

```python
            if fname.endswith(".whl"):
                pkg = fname.split("-")[0]
                relpath = str((Path(dirpath) / fname).relative_to(root))
                items.append({"pkg": pkg, "relpath": relpath})
    return items
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Shell automation** — 通过 shell 命令和环境变量编排仓库任务。
- **Representative symbols: run_command, str2bool, temp_environ, working_directory, get_wheels** — 代表性符号：run_command、str2bool、temp_environ、working_directory、get_wheels

## Dependencies / 依赖关系

- `__future__`
- `logging`
- `os`
- `shlex`
- `subprocess`
- `sys`
- `contextlib`
- `pathlib`
