# bump_kernel_version_to_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/release/bump_kernel_version_to_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `bump_kernel_version_to_sglang` workflow in SGLang. It mainly handles container build steps, release automation. / 该Python 模块用于支撑 SGLang 中的 `bump_kernel_version_to_sglang` 流程，主要负责容器构建步骤、发布自动化。它属于 `release` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-8: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Bump sglang-kernel version in SGLang files to match the version in sgl-kernel/pyproject.toml.
Updates:
  - python/pyproject.toml
  - python/sglang/srt/entrypoints/engine.py
  - docker/Dockerfile
"""
```
**EN:** Bump sglang-kernel version in SGLang files to match the version in sgl-kernel/pyproject.toml.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 10-12: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import re
import sys
from pathlib import Path
```
**EN:** This block loads re, sys, pathlib. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 re, sys, pathlib。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 14-17: Top-level try logic / 顶层 try 逻辑
```python
try:
    import tomllib  # Python 3.11+
except ImportError:
    import tomli as tomllib  # Fallback for older Python versions
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 20-36: Defines the `get_kernel_version_from_source` routine / 定义 `get_kernel_version_from_source` 例程
```python
def get_kernel_version_from_source() -> str:
    """Extract version from sgl-kernel/pyproject.toml"""
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
**EN:** This block defines `get_kernel_version_from_source`. Extract version from sgl-kernel/pyproject.toml It takes no explicit parameters at the top level. Internally it touches filesystem paths, reads or writes files, emits status messages, checks file existence.
**CN:** 该代码块定义了 `get_kernel_version_from_source`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会处理文件系统路径、读写文件、输出状态信息、检查文件是否存在。

### Lines 39-62: Defines the `update_python_pyproject` routine / 定义 `update_python_pyproject` 例程
```python
def update_python_pyproject(new_version: str) -> bool:
    """Update sglang-kernel version in python/pyproject.toml"""
    pyproject_path = Path("python/pyproject.toml")

    if not pyproject_path.exists():
        print(f"Error: {pyproject_path} not found")
        sys.exit(1)

    content = pyproject_path.read_text()

    # Replace "sglang-kernel==x.x.x" with new version
    new_content = re.sub(
        r'"sglang-kernel==[^"]+"',
        f'"sglang-kernel=={new_version}"',
        content,
    )

    if content == new_content:
        print("No changes needed in python/pyproject.toml")
        return False

    pyproject_path.write_text(new_content)
    print(f"✓ Updated python/pyproject.toml to version {new_version}")
    return True
```
**EN:** This block defines `update_python_pyproject`. Update sglang-kernel version in python/pyproject.toml It accepts 1 parameter(s): new_version. Internally it touches filesystem paths, emits status messages, checks file existence, controls process exit status.
**CN:** 该代码块定义了 `update_python_pyproject`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：new_version。内部会处理文件系统路径、输出状态信息、检查文件是否存在、控制进程退出状态。

### Lines 65-88: Defines the `update_engine_py` routine / 定义 `update_engine_py` 例程
```python
def update_engine_py(new_version: str) -> bool:
    """Update sglang-kernel version in python/sglang/srt/entrypoints/engine.py"""
    engine_path = Path("python/sglang/srt/entrypoints/engine.py")

    if not engine_path.exists():
        print(f"Error: {engine_path} not found")
        sys.exit(1)

    content = engine_path.read_text()

    # Replace version in assert_pkg_version("sglang-kernel", "version", ...)
    new_content = re.sub(
        r'(assert_pkg_version\s*\(\s*"sglang-kernel"\s*,\s*)"[^"]+"',
        rf'\1"{new_version}"',
        content,
    )

    if content == new_content:
        print("No changes needed in engine.py")
        return False

    engine_path.write_text(new_content)
    print(f"✓ Updated engine.py to version {new_version}")
    return True
```
**EN:** This block defines `update_engine_py`. Update sglang-kernel version in python/sglang/srt/entrypoints/engine.py It accepts 1 parameter(s): new_version. Internally it touches filesystem paths, checks file existence, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `update_engine_py`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：new_version。内部会处理文件系统路径、检查文件是否存在、输出状态信息、控制进程退出状态。

### Lines 91-115: Defines the `update_dockerfile` routine / 定义 `update_dockerfile` 例程
```python
def update_dockerfile(new_version: str) -> bool:
    """Update SGL_KERNEL_VERSION in docker/Dockerfile"""
    dockerfile_path = Path("docker/Dockerfile")

    if not dockerfile_path.exists():
        print(f"Error: {dockerfile_path} not found")
        sys.exit(1)

    content = dockerfile_path.read_text()

    # Replace ARG SGL_KERNEL_VERSION=x.x.x with new version
    new_content = re.sub(
        r"^(ARG\s+SGL_KERNEL_VERSION=)(.+)$",
        rf"\g<1>{new_version}",
        content,
        flags=re.MULTILINE,
    )

    if content == new_content:
        print("No changes needed in Dockerfile")
        return False

    dockerfile_path.write_text(new_content)
    print(f"✓ Updated Dockerfile to version {new_version}")
    return True
```
**EN:** This block defines `update_dockerfile`. Update SGL_KERNEL_VERSION in docker/Dockerfile It accepts 1 parameter(s): new_version. Internally it touches filesystem paths, checks file existence, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `update_dockerfile`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：new_version。内部会处理文件系统路径、检查文件是否存在、输出状态信息、控制进程退出状态。

### Lines 118-139: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    kernel_version = get_kernel_version_from_source()
    print(f"Bumping sglang-kernel version to: {kernel_version}\n")

    updated_files = []

    if update_python_pyproject(kernel_version):
        updated_files.append("python/pyproject.toml")

    if update_engine_py(kernel_version):
        updated_files.append("python/sglang/srt/entrypoints/engine.py")

    if update_dockerfile(kernel_version):
        updated_files.append("docker/Dockerfile")

    print()
    if updated_files:
        print(f"✓ Successfully updated {len(updated_files)} file(s):")
        for file in updated_files:
            print(f"  - {file}")
    else:
        print("✓ All files already have the correct version")
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会输出状态信息。

### Lines 142-143: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Filesystem coordination** / 文件系统协同
- **Process control** / 进程控制
- **Container execution** / 容器执行

## Dependencies / 依赖关系
- **Standard library / 标准库**: `pathlib`, `re`, `sys`, `tomllib`
- **Third-party modules / 第三方模块**: `tomli`
