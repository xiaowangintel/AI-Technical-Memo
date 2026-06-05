# check_wheel_tags.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `.ci/pytorch/smoke_test/check_wheel_tags.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```python
"""Validate wheel platform tags and macOS dylib minos.
Supports two modes:
1. Pre-install: reads .whl files from PYTORCH_FINAL_PACKAGE_DIR
2. Post-install: reads metadata from installed torch package (soft warnings)
- (macOS only) dylib minos matches the wheel platform tag
"""

import os
import platform
import re
import subprocess
import sys
import tempfile
import zipfile
from pathlib import Path

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 17-26 / 第 17-26 行

```python

EXPECTED_PLATFORM_TAGS: dict[str, str] = {
    "linux": r"_x86_64$",
    "linux-aarch64": r"_aarch64$",
    "windows": r"^win_amd64$",
    "win32": r"^win_amd64$",
    "macos-arm64": r"^macosx_\d+_\d+_arm64$",
    "darwin": r"^macosx_\d+_\d+_(arm64|x86_64)$",
}

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 27-34 / 第 27-34 行

```python

def _extract_wheel_tags(whl_path: Path) -> list[str]:
    """Extract Tag values from the WHEEL metadata file inside a .whl archive."""
    tags = []
    with zipfile.ZipFile(whl_path, "r") as zf:
        wheel_files = [n for n in zf.namelist() if n.endswith("/WHEEL")]
        if not wheel_files:
            return tags
```

- **EN:** Important local symbols in this block include _extract_wheel_tags.
- **CN:** 该代码块中的重要局部符号包括 _extract_wheel_tags。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 35-42 / 第 35-42 行

```python
        content = zf.read(wheel_files[0]).decode("utf-8")
        for line in content.splitlines():
            if line.startswith("Tag:"):
                tags.append(line.split(":", 1)[1].strip())
    return tags


def _extract_installed_wheel_tags(package: str = "torch") -> list[str]:
```

- **EN:** Important local symbols in this block include _extract_installed_wheel_tags.
- **CN:** 该代码块中的重要局部符号包括 _extract_installed_wheel_tags。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 43-51 / 第 43-51 行

```python
    """Extract Tag values from an installed package's WHEEL metadata."""
    from importlib.metadata import distribution

    dist = distribution(package)
    wheel_text = dist.read_text("WHEEL")
    if not wheel_text:
        return []
    tags = []
    for line in wheel_text.splitlines():
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 52-59 / 第 52-59 行

```python
        if line.startswith("Tag:"):
            tags.append(line.split(":", 1)[1].strip())
    return tags


def check_wheel_platform_tag() -> None:
    """Validate that wheel Tags in WHEEL metadata match the expected platform.

```

- **EN:** Important local symbols in this block include check_wheel_platform_tag.
- **CN:** 该代码块中的重要局部符号包括 check_wheel_platform_tag。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 60-70 / 第 60-70 行

```python
    Mode 1: PYTORCH_FINAL_PACKAGE_DIR set → read .whl file (strict, raises on mismatch)
    Mode 2: No wheel dir → read from installed torch package (soft, prints warnings)
    """
    wheel_dir = os.getenv("PYTORCH_FINAL_PACKAGE_DIR", "")

    target_os = os.getenv("TARGET_OS", sys.platform)
    if target_os == "linux" and platform.machine() == "aarch64":
        target_os = "linux-aarch64"
    expected_python = f"cp{sys.version_info.major}{sys.version_info.minor}"
    import sysconfig

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 71-80 / 第 71-80 行

```python
    abiflags = getattr(sys, "abiflags", "")
    if not abiflags and (
        os.getenv("MATRIX_PYTHON_VERSION", "").endswith("t")
        or bool(sysconfig.get_config_var("Py_GIL_DISABLED"))
        or not getattr(sys, "_is_gil_enabled", lambda: True)()
    ):
        abiflags = "t"
    expected_abi = f"cp{sys.version_info.major}{sys.version_info.minor}{abiflags}"
    print(f"Expected ABI tag: {expected_abi}")

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 81-88 / 第 81-88 行

```python
    platform_pattern = EXPECTED_PLATFORM_TAGS.get(target_os)
    if not platform_pattern:
        print(
            f"No expected platform pattern for TARGET_OS={target_os}, "
            "skipping wheel tag check"
        )
        return

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 89-104 / 第 89-104 行

```python
    # Mode 1: Read from .whl file
    if wheel_dir and os.path.isdir(wheel_dir):
        whls = list(Path(wheel_dir).glob("torch-*.whl"))
        if not whls:
            print(f"No torch wheel found in {wheel_dir}, skipping wheel tag check")
            return
        if len(whls) > 1:
            raise RuntimeError(
                f"Expected exactly one torch wheel in {wheel_dir}, "
                f"found {len(whls)}: {[w.name for w in whls]}"
            )
        whl = whls[0]
        print(f"Checking wheel platform tag for: {whl.name}")
        tags = _extract_wheel_tags(whl)
        source = whl.name
    else:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 105-113 / 第 105-113 行

```python
        # Mode 2: Read from installed package (soft)
        print("PYTORCH_FINAL_PACKAGE_DIR not set, reading from installed torch package")
        try:
            tags = _extract_installed_wheel_tags("torch")
            source = "installed torch"
        except Exception as e:
            print(f"Could not read installed torch metadata: {e}, skipping")
            return

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 114-125 / 第 114-125 行

```python
    if not tags:
        raise RuntimeError(f"No Tag found in WHEEL metadata of {source}")

    for tag_str in tags:
        parts = tag_str.split("-")
        if len(parts) != 3:
            msg = (
                f"Malformed wheel tag '{tag_str}' in {source}, "
                f"expected format: <python>-<abi>-<platform>"
            )
            raise RuntimeError(msg)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 126-135 / 第 126-135 行

```python
        python_tag, abi_tag, platform_tag = parts

        print(f"Checking tag: {tag_str} (from {source})")
        if python_tag != expected_python:
            msg: str = (
                f"Python tag mismatch in {source}: "
                f"got '{python_tag}', expected '{expected_python}'"
            )
            raise RuntimeError(msg)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 136-143 / 第 136-143 行

```python
        if abi_tag != expected_abi:
            msg = (
                f"ABI tag mismatch in {source}: "
                f"got '{abi_tag}', expected '{expected_abi}'"
            )
            raise RuntimeError(msg)

        if not re.search(platform_pattern, platform_tag):
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 144-152 / 第 144-152 行

```python
            msg = (
                f"Platform tag mismatch in {source}: "
                f"got '{platform_tag}', expected pattern matching "
                f"'{platform_pattern}' for TARGET_OS={target_os}"
            )
            raise RuntimeError(msg)

    print(f"OK: Wheel tag(s) valid for {source}: {', '.join(tags)}")

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 153-166 / 第 153-166 行

```python

def _check_dylibs_minos(dylibs: list, expected_minos: str, source: str) -> None:
    mismatches = []
    for dylib in dylibs:
        try:
            result = subprocess.run(
                ["otool", "-l", str(dylib)],
                capture_output=True,
                text=True,
                timeout=30,
            )
        except Exception:
            continue

```

- **EN:** Important local symbols in this block include _check_dylibs_minos.
- **CN:** 该代码块中的重要局部符号包括 _check_dylibs_minos。

### Lines 167-177 / 第 167-177 行

```python
        minos = None
        lines = result.stdout.splitlines()
        for i, line in enumerate(lines):
            s = line.strip()
            if "LC_BUILD_VERSION" in s:
                for j in range(i + 1, min(i + 6, len(lines))):
                    if lines[j].strip().startswith("minos"):
                        minos = lines[j].strip().split()[1]
                        break
                break
            if "LC_VERSION_MIN_MACOSX" in s:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 178-186 / 第 178-186 行

```python
                for j in range(i + 1, min(i + 4, len(lines))):
                    if lines[j].strip().startswith("version"):
                        minos = lines[j].strip().split()[1]
                        break
                break

        # A dylib with a lower minos than the wheel tag is safe (forward compatible).
        # Only flag dylibs that require a *higher* macOS than the wheel claims to support.
        if minos and tuple(int(x) for x in minos.split(".")) > tuple(
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 187-202 / 第 187-202 行

```python
            int(x) for x in expected_minos.split(".")
        ):
            mismatches.append(
                f"{dylib.name}: minos={minos}, expected<={expected_minos}"
            )

    if mismatches:
        raise RuntimeError(
            f"minos/platform tag mismatch in {len(mismatches)} dylib(s):\n"
            + "\n".join(f"  {m}" for m in mismatches)
        )
    print(
        f"OK: All {len(dylibs)} dylib(s) have minos matching "
        f"platform tag ({expected_minos}) for {source}"
    )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 203-210 / 第 203-210 行

```python

def check_mac_wheel_minos() -> None:
    if sys.platform != "darwin":
        return

    wheel_dir = os.getenv("PYTORCH_FINAL_PACKAGE_DIR", "")

    if wheel_dir and os.path.isdir(wheel_dir):
```

- **EN:** Important local symbols in this block include check_mac_wheel_minos.
- **CN:** 该代码块中的重要局部符号包括 check_mac_wheel_minos。

### Lines 211-218 / 第 211-218 行

```python
        # Mode 1: extract dylibs from .whl file
        whls = list(Path(wheel_dir).glob("*.whl"))
        if not whls:
            print(f"No .whl files in {wheel_dir}, skipping wheel minos check")
            return

        macos_whl_re = re.compile(r"macosx_(\d+)_(\d+)_(\w+)\.whl$")
        for whl in whls:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 219-229 / 第 219-229 行

```python
            print(f"Checking wheel tag minos for: {whl.name}")
            m = macos_whl_re.search(whl.name)
            if not m:
                print(f"No macOS platform tag in {whl.name}, skipping")
                continue
            expected_minos = f"{m.group(1)}.{m.group(2)}"

            with tempfile.TemporaryDirectory() as tmpdir:
                with zipfile.ZipFile(whl, "r") as zf:
                    dylib_names = [n for n in zf.namelist() if n.endswith(".dylib")]
                    if not dylib_names:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 230-244 / 第 230-244 行

```python
                        print("No .dylib files in wheel, skipping minos check")
                        continue
                    for name in dylib_names:
                        zf.extract(name, tmpdir)
                dylibs = list(Path(tmpdir).rglob("*.dylib"))
                _check_dylibs_minos(dylibs, expected_minos, whl.name)
    else:
        # Mode 2: read from installed torch package
        print("PYTORCH_FINAL_PACKAGE_DIR not set, checking installed torch dylibs")
        try:
            tags = _extract_installed_wheel_tags("torch")
        except Exception as e:
            print(f"Could not read installed torch metadata: {e}, skipping")
            return

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 245-252 / 第 245-252 行

```python
        expected_minos = None
        for tag_str in tags:
            m = re.search(r"macosx_(\d+)_(\d+)_\w+", tag_str)
            if m:
                expected_minos = f"{m.group(1)}.{m.group(2)}"
                break

        if not expected_minos:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 253-262 / 第 253-262 行

```python
            print("No macOS platform tag found in installed torch metadata, skipping")
            return

        print(f"Expected minos from installed wheel tag: {expected_minos}")

        import torch

        torch_dir = Path(torch.__file__).parent
        dylibs = list(torch_dir.rglob("*.dylib"))
        if not dylibs:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 263-269 / 第 263-269 行

```python
            raise RuntimeError("No .dylib files found in installed torch")
        _check_dylibs_minos(dylibs, expected_minos, "installed torch")


if __name__ == "__main__":
    check_wheel_platform_tag()
    check_mac_wheel_minos()
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Representative symbols: _extract_wheel_tags, _extract_installed_wheel_tags, check_wheel_platform_tag, _check_dylibs_minos, check_mac_wheel_minos** — 代表性符号：_extract_wheel_tags、_extract_installed_wheel_tags、check_wheel_platform_tag、_check_dylibs_minos、check_mac_wheel_minos

## Dependencies / 依赖关系

- `os`
- `platform`
- `re`
- `subprocess`
- `sys`
- `tempfile`
- `zipfile`
- `pathlib`
- `importlib.metadata`
- `sysconfig`
- `torch`
