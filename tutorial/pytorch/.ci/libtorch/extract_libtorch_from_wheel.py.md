# extract_libtorch_from_wheel.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `.ci/libtorch/extract_libtorch_from_wheel.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation. The file header summarizes the intent as: "!/usr/bin/env python3."
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。 文件头部将其意图概括为：“!/usr/bin/env python3”。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```python
#!/usr/bin/env python3
"""Extract libtorch package from a PyTorch wheel.

Creates a libtorch zip from a pre-built wheel by copying the C++ libraries,
headers, and CMake files. On Linux, optionally splits debug symbols from
libtorch_cpu.so into a separate debug zip.

Usage:
    python extract_libtorch_from_wheel.py \
        --wheel-dir DIR --output-dir DIR --platform linux|macos|windows
"""

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 13-22 / 第 13-22 行

```python
import argparse
import glob
import os
import re
import shutil
import subprocess
import sys
import zipfile
from pathlib import Path

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 23-30 / 第 23-30 行

```python

def find_wheel(wheel_dir: str) -> Path:
    wheels = glob.glob(os.path.join(wheel_dir, "*.whl"))
    if not wheels:
        raise FileNotFoundError(f"No .whl files found in {wheel_dir}")
    if len(wheels) > 1:
        raise RuntimeError(f"Multiple .whl files found in {wheel_dir}: {wheels}")
    return Path(wheels[0])
```

- **EN:** Important local symbols in this block include find_wheel.
- **CN:** 该代码块中的重要局部符号包括 find_wheel。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 31-39 / 第 31-39 行

```python


def parse_version_from_wheel(wheel_path: Path) -> str:
    # Wheel filename format: {name}-{version}(-{build})?-{python}-{abi}-{platform}.whl
    name = wheel_path.stem
    parts = name.split("-")
    if len(parts) < 3:
        raise ValueError(f"Cannot parse version from wheel filename: {wheel_path.name}")
    return parts[1]
```

- **EN:** Important local symbols in this block include parse_version_from_wheel.
- **CN:** 该代码块中的重要局部符号包括 parse_version_from_wheel。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 40-47 / 第 40-47 行

```python


def extract_wheel(wheel_path: Path, extract_dir: Path) -> Path:
    with zipfile.ZipFile(wheel_path, "r") as zf:
        zf.extractall(extract_dir)
    # Find the torch directory
    torch_dir = extract_dir / "torch"
    if not torch_dir.is_dir():
```

- **EN:** Important local symbols in this block include extract_wheel.
- **CN:** 该代码块中的重要局部符号包括 extract_wheel。

### Lines 48-56 / 第 48-56 行

```python
        raise FileNotFoundError(
            f"No 'torch' directory found in extracted wheel at {extract_dir}"
        )
    return torch_dir


def should_exclude_lib(filename: str) -> bool:
    """Return True for files that should not go into the libtorch package."""
    if filename.startswith("libtorch_python"):
```

- **EN:** Important local symbols in this block include should_exclude_lib.
- **CN:** 该代码块中的重要局部符号包括 should_exclude_lib。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 57-64 / 第 57-64 行

```python
        return True
    if re.match(r"_C\.cpython.*", filename):
        return True
    if filename.endswith((".py", ".pyc")):
        return True
    if filename == "__init__.py":
        return True
    return False
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 65-72 / 第 65-72 行

```python


def _is_lib_file(name: str, platform: str) -> bool:
    """Return True if the file looks like a library or header to include."""
    if platform == "linux":
        return ".so" in name or name.endswith(".a")
    elif platform == "macos":
        return name.endswith((".dylib", ".a"))
```

- **EN:** Important local symbols in this block include _is_lib_file.
- **CN:** 该代码块中的重要局部符号包括 _is_lib_file。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 73-81 / 第 73-81 行

```python
    elif platform == "windows":
        return name.endswith((".dll", ".lib", ".pdb"))
    return False


def copy_libraries(torch_dir: Path, libtorch_lib: Path, platform: str) -> None:
    """Copy libraries from torch/lib/ to libtorch/lib/."""
    torch_lib = torch_dir / "lib"
    if not torch_lib.is_dir():
```

- **EN:** Important local symbols in this block include copy_libraries.
- **CN:** 该代码块中的重要局部符号包括 copy_libraries。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 82-89 / 第 82-89 行

```python
        raise FileNotFoundError(f"torch/lib/ not found at {torch_lib}")

    for item in torch_lib.iterdir():
        if item.is_dir():
            # Copy subdirectories (e.g. libshm/) as-is
            shutil.copytree(item, libtorch_lib / item.name, dirs_exist_ok=True)
            continue
        if should_exclude_lib(item.name):
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 90-97 / 第 90-97 行

```python
            continue
        if _is_lib_file(item.name, platform):
            shutil.copy2(item, libtorch_lib / item.name)

    # On macOS, also copy delocated dylibs from torch/.dylibs/ if present
    if platform == "macos":
        dylibs_dir = torch_dir / ".dylibs"
        if dylibs_dir.is_dir():
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 98-105 / 第 98-105 行

```python
            for item in dylibs_dir.iterdir():
                if item.suffix == ".dylib" and not should_exclude_lib(item.name):
                    shutil.copy2(item, libtorch_lib / item.name)


def copy_includes(torch_dir: Path, libtorch_include: Path) -> None:
    torch_include = torch_dir / "include"
    if not torch_include.is_dir():
```

- **EN:** Important local symbols in this block include copy_includes.
- **CN:** 该代码块中的重要局部符号包括 copy_includes。

### Lines 106-113 / 第 106-113 行

```python
        # Some older wheels might have include under torch/lib/include
        torch_include = torch_dir / "lib" / "include"
    if not torch_include.is_dir():
        raise FileNotFoundError("include/ not found in torch directory")
    shutil.copytree(torch_include, libtorch_include, dirs_exist_ok=True)


def copy_cmake(torch_dir: Path, libtorch_share: Path) -> None:
```

- **EN:** Important local symbols in this block include copy_cmake.
- **CN:** 该代码块中的重要局部符号包括 copy_cmake。

### Lines 114-121 / 第 114-121 行

```python
    torch_cmake = torch_dir / "share" / "cmake"
    if not torch_cmake.is_dir():
        print(f"Warning: share/cmake/ not found at {torch_cmake}", file=sys.stderr)
        return
    cmake_dest = libtorch_share / "cmake"
    shutil.copytree(torch_cmake, cmake_dest, dirs_exist_ok=True)


```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 122-130 / 第 122-130 行

```python
def copy_bin(torch_dir: Path, libtorch_bin: Path, platform: str) -> None:
    """Copy binary executables (mainly relevant for Windows)."""
    if platform == "windows":
        torch_lib = torch_dir / "lib"
        if torch_lib.is_dir():
            for item in torch_lib.iterdir():
                if item.suffix == ".dll" and not should_exclude_lib(item.name):
                    shutil.copy2(item, libtorch_bin / item.name)

```

- **EN:** Important local symbols in this block include copy_bin.
- **CN:** 该代码块中的重要局部符号包括 copy_bin。

### Lines 131-140 / 第 131-140 行

```python

def write_metadata(libtorch_dir: Path, version: str, git_hash: str) -> None:
    (libtorch_dir / "build-version").write_text(version + "\n")
    (libtorch_dir / "build-hash").write_text(git_hash + "\n")


def get_git_hash(torch_dir: Path) -> str:
    """Read git_version from the wheel's torch/version.py."""
    version_file = torch_dir / "version.py"
    if not version_file.exists():
```

- **EN:** Important local symbols in this block include write_metadata, get_git_hash.
- **CN:** 该代码块中的重要局部符号包括 write_metadata、get_git_hash。

### Lines 141-150 / 第 141-150 行

```python
        return "unknown"
    from ast import literal_eval

    for line in version_file.read_text().splitlines():
        if line.strip().startswith("git_version"):
            try:
                return literal_eval(line.partition("=")[2].strip())
            except Exception:
                pass
    return "unknown"
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 151-158 / 第 151-158 行

```python


def split_debug_symbols(
    libtorch_dir: Path, output_dir: Path, zip_prefix: str, version: str
) -> None:
    """Split debug symbols from libtorch_cpu.so (Linux only)."""
    libtorch_cpu = libtorch_dir / "lib" / "libtorch_cpu.so"
    if not libtorch_cpu.exists():
```

- **EN:** Important local symbols in this block include split_debug_symbols.
- **CN:** 该代码块中的重要局部符号包括 split_debug_symbols。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 159-168 / 第 159-168 行

```python
        print(
            "Warning: libtorch_cpu.so not found, skipping debug symbol split",
            file=sys.stderr,
        )
        return

    debug_dir = libtorch_dir.parent / "debug"
    debug_dir.mkdir(exist_ok=True)
    dbg_file = debug_dir / "libtorch_cpu.so.dbg"

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 169-177 / 第 169-177 行

```python
    # Copy to create debug file
    shutil.copy2(libtorch_cpu, dbg_file)

    # Keep only debug symbols
    subprocess.run(
        ["strip", "--only-keep-debug", str(dbg_file)],
        check=True,
    )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 178-190 / 第 178-190 行

```python
    # Strip debug info from release lib
    subprocess.run(
        ["strip", "--strip-debug", str(libtorch_cpu)],
        check=True,
    )

    # Add debug link
    subprocess.run(
        ["objcopy", str(libtorch_cpu), f"--add-gnu-debuglink={dbg_file}"],
        check=True,
        cwd=str(libtorch_dir / "lib"),
    )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 191-206 / 第 191-206 行

```python
    # Extract CRC32 from the debug link section
    try:
        result = subprocess.run(
            [
                "bash",
                "-c",
                f"objcopy --dump-section .gnu_debuglink=>(tail -c4 | od -t x4 -An | xargs echo) {libtorch_cpu}",
            ],
            capture_output=True,
            text=True,
            check=True,
        )
        crc32 = result.stdout.strip()
    except subprocess.CalledProcessError:
        crc32 = "unknown"

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 207-214 / 第 207-214 行

```python
    # Create debug zip
    debug_zip = output_dir / f"debug-{zip_prefix}-{version}-{crc32}.zip"
    with zipfile.ZipFile(debug_zip, "w", zipfile.ZIP_DEFLATED) as zf:
        zf.write(dbg_file, "debug/libtorch_cpu.so.dbg")

    print(f"Debug symbols zip: {debug_zip}")


```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 215-223 / 第 215-223 行

```python
def create_libtorch_zip(
    libtorch_dir: Path,
    output_dir: Path,
    zip_prefix: str,
    version: str,
) -> Path:
    zip_path = output_dir / f"{zip_prefix}-{version}.zip"
    with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as zf:
        for root, dirs, files in os.walk(libtorch_dir):
```

- **EN:** Important local symbols in this block include create_libtorch_zip.
- **CN:** 该代码块中的重要局部符号包括 create_libtorch_zip。

### Lines 224-231 / 第 224-231 行

```python
            for f in files:
                filepath = Path(root) / f
                arcname = filepath.relative_to(libtorch_dir.parent)
                zf.write(filepath, arcname)
    # Create latest symlink
    latest_zip = output_dir / f"{zip_prefix}-latest.zip"
    latest_zip.symlink_to(zip_path.name)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 232-239 / 第 232-239 行

```python
    print(f"Libtorch zip: {zip_path}")
    print(f"Libtorch latest zip: {latest_zip}")
    return zip_path


def compute_zip_prefix(platform: str, desired_cuda: str, libtorch_variant: str) -> str:
    """Compute the zip filename prefix matching existing naming conventions.

```

- **EN:** Important local symbols in this block include compute_zip_prefix.
- **CN:** 该代码块中的重要局部符号包括 compute_zip_prefix。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 240-247 / 第 240-247 行

```python
    Linux:  libtorch-shared-with-deps
    macOS:  libtorch-macos-arm64
    Windows: libtorch-win-shared-with-deps (or libtorch-win-arm64-shared-with-deps)
    """
    if platform == "macos":
        return "libtorch-macos-arm64"
    elif platform == "windows":
        return f"libtorch-win-{libtorch_variant}"
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 248-263 / 第 248-263 行

```python
    else:
        return f"libtorch-{libtorch_variant}"


def main() -> None:
    parser = argparse.ArgumentParser(description="Extract libtorch from PyTorch wheel")
    parser.add_argument(
        "--wheel-dir", required=True, help="Directory containing the .whl file"
    )
    parser.add_argument(
        "--output-dir", required=True, help="Directory for output zip files"
    )
    parser.add_argument(
        "--platform",
        required=True,
        choices=["linux", "macos", "windows"],
```

- **EN:** Important local symbols in this block include main.
- **CN:** 该代码块中的重要局部符号包括 main。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 264-279 / 第 264-279 行

```python
        help="Target platform",
    )
    parser.add_argument(
        "--desired-cuda",
        default="cpu",
        help="CUDA variant (cpu, cu126, cu128, rocm7.1, etc.)",
    )
    parser.add_argument(
        "--libtorch-variant",
        default="shared-with-deps",
        help="Libtorch variant (shared-with-deps, etc.)",
    )
    parser.add_argument(
        "--git-hash",
        default="",
        help="Git hash to use for build-hash (auto-detected if not set)",
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 280-292 / 第 280-292 行

```python
    )
    args = parser.parse_args()

    wheel_dir = Path(args.wheel_dir)
    output_dir = Path(args.output_dir)
    output_dir.mkdir(parents=True, exist_ok=True)

    # Find and extract wheel
    wheel_path = find_wheel(str(wheel_dir))
    version = parse_version_from_wheel(wheel_path)
    print(f"Found wheel: {wheel_path}")
    print(f"Version: {version}")

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 293-300 / 第 293-300 行

```python
    extract_dir = wheel_dir / "_extract_tmp"
    if extract_dir.exists():
        shutil.rmtree(extract_dir)
    extract_dir.mkdir()

    try:
        torch_dir = extract_wheel(wheel_path, extract_dir)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 301-312 / 第 301-312 行

```python
        # Create libtorch directory structure
        libtorch_dir = extract_dir / "libtorch"
        libtorch_dir.mkdir()
        for subdir in ["lib", "bin", "include", "share"]:
            (libtorch_dir / subdir).mkdir()

        # Copy components
        copy_libraries(torch_dir, libtorch_dir / "lib", args.platform)
        copy_includes(torch_dir, libtorch_dir / "include")
        copy_cmake(torch_dir, libtorch_dir / "share")
        copy_bin(torch_dir, libtorch_dir / "bin", args.platform)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 313-321 / 第 313-321 行

```python
        # Write metadata
        git_hash = args.git_hash or get_git_hash(torch_dir)
        write_metadata(libtorch_dir, version, git_hash)

        # Compute zip prefix
        zip_prefix = compute_zip_prefix(
            args.platform, args.desired_cuda, args.libtorch_variant
        )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 322-331 / 第 322-331 行

```python
        # Split debug symbols on Linux
        if args.platform == "linux":
            split_debug_symbols(libtorch_dir, output_dir, zip_prefix, version)

        # Create the zip
        create_libtorch_zip(libtorch_dir, output_dir, zip_prefix, version)

    finally:
        shutil.rmtree(extract_dir)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 332-334 / 第 332-334 行

```python

if __name__ == "__main__":
    main()
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。
- **Representative symbols: find_wheel, parse_version_from_wheel, extract_wheel, should_exclude_lib, _is_lib_file, copy_libraries, copy_includes, copy_cmake** — 代表性符号：find_wheel、parse_version_from_wheel、extract_wheel、should_exclude_lib、_is_lib_file、copy_libraries、copy_includes、copy_cmake

## Dependencies / 依赖关系

- `argparse`
- `glob`
- `os`
- `re`
- `shutil`
- `subprocess`
- `sys`
- `zipfile`
- `pathlib`
- `ast`
