# check_kernel_version_to_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/release/check_kernel_version_to_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `check_kernel_version_to_sglang` workflow in SGLang. It mainly handles container build steps, policy validation. / 该Python 模块用于支撑 SGLang 中的 `check_kernel_version_to_sglang` 流程，主要负责容器构建步骤、策略校验。它属于 `release` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-6: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Check if sglang-kernel version from sgl-kernel/pyproject.toml matches the versions
used in SGLang files (python/pyproject.toml, engine.py, and Dockerfile).
Sets GitHub Actions output variables to indicate if sync is needed.
"""
```
**EN:** Check if sglang-kernel version from sgl-kernel/pyproject.toml matches the versions used in SGLang files (python/pyproject.toml, engine.py, and Dockerfile).
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 8-11: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import os
import re
import sys
from pathlib import Path
```
**EN:** This block loads os, re, sys, pathlib. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 os, re, sys, pathlib。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 13-16: Top-level try logic / 顶层 try 逻辑
```python
try:
    import tomllib  # Python 3.11+
except ImportError:
    import tomli as tomllib  # Fallback for older Python versions
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 19-35: Defines the `get_kernel_version_from_source` routine / 定义 `get_kernel_version_from_source` 例程
```python
def get_kernel_version_from_source() -> str:
    """Extract version from sgl-kernel/pyproject.toml (line 11)"""
    pyproject_path = Path("sgl-kernel/pyproject.toml")

    if not pyproject_path.exists():
        print(f"Error: {pyproject_path} not found")
        sys.exit(1)

    with open(pyproject_path, "rb") as f:
        data = tomllib.load(f)

    version = data.get("project", {}).get("version")
    if not version:
        print("Error: Could not find version in sgl-kernel/pyproject.toml")
        sys.exit(1)

    return version
```
**EN:** This block defines `get_kernel_version_from_source`. Extract version from sgl-kernel/pyproject.toml (line 11) It takes no explicit parameters at the top level. Internally it touches filesystem paths, reads or writes files, emits status messages, checks file existence.
**CN:** 该代码块定义了 `get_kernel_version_from_source`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会处理文件系统路径、读写文件、输出状态信息、检查文件是否存在。

### Lines 38-54: Defines the `get_kernel_version_from_python_pyproject` routine / 定义 `get_kernel_version_from_python_pyproject` 例程
```python
def get_kernel_version_from_python_pyproject() -> str:
    """Extract sglang-kernel version from python/pyproject.toml"""
    pyproject_path = Path("python/pyproject.toml")

    if not pyproject_path.exists():
        print(f"Error: {pyproject_path} not found")
        sys.exit(1)

    content = pyproject_path.read_text()

    # Match "sglang-kernel==x.x.x"
    match = re.search(r'"sglang-kernel==([^"]+)"', content)
    if not match:
        print("Error: Could not find sglang-kernel version in python/pyproject.toml")
        sys.exit(1)

    return match.group(1)
```
**EN:** This block defines `get_kernel_version_from_python_pyproject`. Extract sglang-kernel version from python/pyproject.toml It takes no explicit parameters at the top level. Internally it touches filesystem paths, emits status messages, checks file existence, controls process exit status.
**CN:** 该代码块定义了 `get_kernel_version_from_python_pyproject`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会处理文件系统路径、输出状态信息、检查文件是否存在、控制进程退出状态。

### Lines 57-76: Defines the `get_kernel_version_from_engine` routine / 定义 `get_kernel_version_from_engine` 例程
```python
def get_kernel_version_from_engine() -> str:
    """Extract sglang-kernel version from python/sglang/srt/entrypoints/engine.py"""
    engine_path = Path("python/sglang/srt/entrypoints/engine.py")

    if not engine_path.exists():
        print(f"Error: {engine_path} not found")
        sys.exit(1)

    content = engine_path.read_text()

    # Find the assert_pkg_version call for sglang-kernel
    # Look for the pattern: assert_pkg_version("sglang-kernel", "version", ...)
    match = re.search(
        r'assert_pkg_version\s*\(\s*"sglang-kernel"\s*,\s*"([^"]+)"', content
    )
    if not match:
        print("Error: Could not find sglang-kernel version in engine.py")
        sys.exit(1)

    return match.group(1)
```
**EN:** This block defines `get_kernel_version_from_engine`. Extract sglang-kernel version from python/sglang/srt/entrypoints/engine.py It takes no explicit parameters at the top level. Internally it touches filesystem paths, checks file existence, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `get_kernel_version_from_engine`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会处理文件系统路径、检查文件是否存在、输出状态信息、控制进程退出状态。

### Lines 79-95: Defines the `get_kernel_version_from_dockerfile` routine / 定义 `get_kernel_version_from_dockerfile` 例程
```python
def get_kernel_version_from_dockerfile() -> str:
    """Extract SGL_KERNEL_VERSION from docker/Dockerfile"""
    dockerfile_path = Path("docker/Dockerfile")

    if not dockerfile_path.exists():
        print(f"Error: {dockerfile_path} not found")
        sys.exit(1)

    content = dockerfile_path.read_text()

    # Match ARG SGL_KERNEL_VERSION=x.x.x
    match = re.search(r"^ARG\s+SGL_KERNEL_VERSION=(.+)$", content, re.MULTILINE)
    if not match:
        print("Error: Could not find SGL_KERNEL_VERSION in Dockerfile")
        sys.exit(1)

    return match.group(1).strip()
```
**EN:** This block defines `get_kernel_version_from_dockerfile`. Extract SGL_KERNEL_VERSION from docker/Dockerfile It takes no explicit parameters at the top level. Internally it touches filesystem paths, checks file existence, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `get_kernel_version_from_dockerfile`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会处理文件系统路径、检查文件是否存在、输出状态信息、控制进程退出状态。

### Lines 98-146: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    kernel_version = get_kernel_version_from_source()
    pyproject_version = get_kernel_version_from_python_pyproject()
    engine_version = get_kernel_version_from_engine()
    dockerfile_version = get_kernel_version_from_dockerfile()

    print(f"Kernel version in sgl-kernel/pyproject.toml: {kernel_version}")
    print(
        f"SGLang kernel dependency version in python/pyproject.toml: {pyproject_version}"
    )
    print(f"SGLang kernel dependency version in engine.py: {engine_version}")
    print(f"Kernel version in Dockerfile: {dockerfile_version}")

    # Check if any version differs from the source
    needs_sync = (
        kernel_version != pyproject_version
        or kernel_version != engine_version
        or kernel_version != dockerfile_version
    )

    # Set GitHub Actions output
    github_output = os.getenv("GITHUB_OUTPUT")
    if github_output:
        with open(github_output, "a") as f:
            f.write(f"needs_sync={'true' if needs_sync else 'false'}\n")
            f.write(f"kernel_version={kernel_version}\n")

    if needs_sync:
        print(f"\n✓ Sync needed to version: {kernel_version}")
        mismatches = []
        if kernel_version != pyproject_version:
            mismatches.append(
                f"  - python/pyproject.toml: {pyproject_version} → {kernel_version}"
            )
        if kernel_version != engine_version:
            mismatches.append(f"  - engine.py: {engine_version} → {kernel_version}")
        if kernel_version != dockerfile_version:
            mismatches.append(
                f"  - Dockerfile: {dockerfile_version} → {kernel_version}"
            )

        print("Changes needed:")
        for mismatch in mismatches:
            print(mismatch)

        sys.exit(0)
    else:
        print("\n✓ All versions are in sync, no action needed")
        sys.exit(0)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it reads or writes files, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会读写文件、输出状态信息、控制进程退出状态。

### Lines 149-150: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Git state inspection** / Git 状态检查
- **Container execution** / 容器执行

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `pathlib`, `re`, `sys`, `tomllib`
- **Third-party modules / 第三方模块**: `tomli`
