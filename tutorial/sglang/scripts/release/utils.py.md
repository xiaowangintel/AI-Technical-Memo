# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/release/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `utils` workflow in SGLang. It mainly handles CI orchestration, policy validation. / 该Python 模块用于支撑 SGLang 中的 `utils` 流程，主要负责CI 编排、策略校验。它属于 `release` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import re
import sys
from pathlib import Path
from typing import List, Tuple
```
**EN:** This block loads re, sys, pathlib, typing. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 re, sys, pathlib, typing。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 6-9: Top-level try logic / 顶层 try 逻辑
```python
try:
    import tomllib  # Python 3.11+
except ImportError:
    import tomli as tomllib  # Fallback for older Python versions
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 12-14: Defines the `normalize_version` routine / 定义 `normalize_version` 例程
```python
def normalize_version(version: str) -> str:
    """Remove 'v' prefix from version string if present."""
    return version.lstrip("v")
```
**EN:** This block defines `normalize_version`. Remove 'v' prefix from version string if present. It accepts 1 parameter(s): version. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `normalize_version`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：version。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 17-20: Defines the `validate_version` routine / 定义 `validate_version` 例程
```python
def validate_version(version: str) -> bool:
    """Validate version format: X.Y.Z, X.Y.Zrc0, or X.Y.Z.post1"""
    pattern = r"^\d+\.\d+\.\d+(rc\d+|\.post\d+)?$"
    return bool(re.match(pattern, version))
```
**EN:** This block defines `validate_version`. Validate version format: X.Y.Z, X.Y.Zrc0, or X.Y.Z.post1 It accepts 1 parameter(s): version. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `validate_version`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：version。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 23-57: Defines the `parse_version` routine / 定义 `parse_version` 例程
```python
def parse_version(version: str) -> Tuple[int, int, int, int, int]:
    """
    Parse version string into comparable components.

    Returns: (major, minor, patch, pre_release, post_release)
    - pre_release: -1000 + rc_number for rcN, 0 for stable (rc0 < rc1 < stable)
    - post_release: N for .postN, 0 otherwise

    The pre_release field uses negative numbers to ensure RC versions come before
    stable versions when tuples are compared. Python compares tuples element by
    element, so (0, 5, 3, -1000, 0) < (0, 5, 3, 0, 0) ensures rc0 < stable.

    Examples:
    - "0.5.3rc0" → (0, 5, 3, -1000, 0)  # rc0 comes before stable
    - "0.5.3rc1" → (0, 5, 3, -999, 0)   # rc1 comes after rc0
    - "0.5.3"    → (0, 5, 3, 0, 0)      # stable version
    - "0.5.3.post1" → (0, 5, 3, 0, 1)   # post comes after stable
    """
    # Match version components
    match = re.match(r"^(\d+)\.(\d+)\.(\d+)(?:rc(\d+)|\.post(\d+))?$", version)
    if not match:
        raise ValueError(f"Invalid version format: {version}")

    major, minor, patch, rc, post = match.groups()
    major, minor, patch = int(major), int(minor), int(patch)

    if rc is not None:
        # RC version: pre_release = -1000 + rc_number (ensures rc0 < rc1 < ... < stable)
        return (major, minor, patch, -1000 + int(rc), 0)
    elif post is not None:
        # Post version: post_release = N
        return (major, minor, patch, 0, int(post))
    else:
        # Stable version
        return (major, minor, patch, 0, 0)
```
**EN:** This block defines `parse_version`. Parse version string into comparable components. It accepts 1 parameter(s): version. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `parse_version`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：version。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 60-79: Defines the `compare_versions` routine / 定义 `compare_versions` 例程
```python
def compare_versions(v1: str, v2: str) -> int:
    """
    Compare two version strings following PEP 440 ordering.

    Returns:
    - -1 if v1 < v2
    -  0 if v1 == v2
    -  1 if v1 > v2

    Version ordering: X.Y.ZrcN < X.Y.Z < X.Y.Z.postN < X.Y.(Z+1)
    """
    parsed_v1 = parse_version(v1)
    parsed_v2 = parse_version(v2)

    if parsed_v1 < parsed_v2:
        return -1
    elif parsed_v1 > parsed_v2:
        return 1
    else:
        return 0
```
**EN:** This block defines `compare_versions`. Compare two version strings following PEP 440 ordering. It accepts 2 parameter(s): v1, v2. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `compare_versions`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：v1, v2。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 82-83: Defines the `get_repo_root` routine / 定义 `get_repo_root` 例程
```python
def get_repo_root() -> Path:
    return Path(__file__).parent.parent.parent
```
**EN:** This block defines `get_repo_root`. It takes no explicit parameters at the top level. Internally it touches filesystem paths.
**CN:** 该代码块定义了 `get_repo_root`。它在顶层定义中不接收显式参数。内部会处理文件系统路径。

### Lines 86-91: Defines the `read_current_version` routine / 定义 `read_current_version` 例程
```python
def read_current_version(version_file: Path) -> str:
    content = version_file.read_text()
    match = re.search(r'__version__\s*=\s*["\']([^"\']+)["\']', content)
    if not match:
        raise ValueError(f"Could not find version in {version_file}")
    return match.group(1)
```
**EN:** This block defines `read_current_version`. It accepts 1 parameter(s): version_file. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `read_current_version`。它接收 1 个参数：version_file。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 94-135: Defines the `replace_in_file` routine / 定义 `replace_in_file` 例程
```python
def replace_in_file(file_path: Path, old_version: str, new_version: str) -> bool:
    if not file_path.exists():
        print(f"Warning: {file_path} does not exist, skipping")
        return False

    content = file_path.read_text()

    # For TOML files, parse and update only the [project] version field
    if file_path.suffix == ".toml":
        try:
            # Parse TOML to verify structure
            toml_data = tomllib.loads(content)

            # Check if [project] section exists and has version field
            if "project" not in toml_data or "version" not in toml_data["project"]:
                print(
                    f"Warning: {file_path} does not have [project] version field, skipping"
                )
                return False

            # Use regex to replace only the version field in [project] section
            # This pattern matches the version field that comes after [project]
            # and before any other section marker
            pattern = r'(\[project\].*?version\s*=\s*)["\']([^"\']+)["\']'
            new_content = re.sub(
                pattern, rf'\g<1>"{new_version}"', content, flags=re.DOTALL
            )
        except Exception as e:
            print(f"Warning: Failed to parse {file_path} as TOML: {e}")
            print("Falling back to simple string replacement")
            new_content = content.replace(old_version, new_version)
    else:
        # For non-TOML files, use simple string replacement
        new_content = content.replace(old_version, new_version)

    if content == new_content:
        print(f"No changes needed in {file_path}")
        return False

    file_path.write_text(new_content)
    print(f"✓ Updated {file_path}")
    return True
```
**EN:** This block defines `replace_in_file`. It accepts 3 parameter(s): file_path, old_version, new_version. Internally it checks file existence, emits status messages.
**CN:** 该代码块定义了 `replace_in_file`。它接收 3 个参数：file_path, old_version, new_version。内部会检查文件是否存在、输出状态信息。

### Lines 138-220: Defines the `bump_version` routine / 定义 `bump_version` 例程
```python
def bump_version(
    new_version: str,
    version_file: Path,
    files_to_update: List[Path],
) -> None:
    # Normalize version (remove 'v' prefix if present)
    new_version = normalize_version(new_version)

    if not validate_version(new_version):
        print(f"Error: Invalid version format: {new_version}")
        print("Expected format: X.Y.Z, X.Y.ZrcN, or X.Y.Z.postN")
        print("Examples: 0.5.4, 0.5.3rc0, 0.5.3.post1")
        sys.exit(1)

    repo_root = get_repo_root()
    version_file_abs = repo_root / version_file

    if not version_file_abs.exists():
        print(f"Error: Version file {version_file_abs} does not exist")
        sys.exit(1)

    old_version = read_current_version(version_file_abs)
    print(f"Current version: {old_version}")
    print(f"New version: {new_version}")
    print()

    # Compare versions
    comparison = compare_versions(new_version, old_version)
    if comparison == 0:
        print("Error: New version is the same as current version")
        sys.exit(1)
    elif comparison < 0:
        print(
            f"Error: New version ({new_version}) is older than current version ({old_version})"
        )
        print("Version must be greater than the current version")
        sys.exit(1)

    updated_count = 0
    for file_rel in files_to_update:
        file_abs = repo_root / file_rel
        if replace_in_file(file_abs, old_version, new_version):
            updated_count += 1

    print()
    print(f"Successfully updated {updated_count} file(s)")
    print(f"Version bumped from {old_version} to {new_version}")

    # Validate that all files now contain the new version
    print("\nValidating version updates...")
    failed_files = []
    for file_rel in files_to_update:
        file_abs = repo_root / file_rel
        if not file_abs.exists():
            print(f"Warning: File {file_rel} does not exist, skipping validation.")
            continue

        content = file_abs.read_text()

        # For TOML files, use regex to specifically check the version field
        if file_abs.suffix == ".toml":
            # Match version field with optional quotes
            pattern = r'version\s*=\s*["\']?' + re.escape(new_version) + r'["\']?'
            if not re.search(pattern, content):
                failed_files.append(file_rel)
                print(f"✗ {file_rel} does not contain version {new_version}")
            else:
                print(f"✓ {file_rel} validated")
        else:
            # For non-TOML files, use simple string search
            if new_version not in content:
                failed_files.append(file_rel)
                print(f"✗ {file_rel} does not contain version {new_version}")
            else:
                print(f"✓ {file_rel} validated")

    if failed_files:
        print(f"\nError: {len(failed_files)} file(s) were not updated correctly:")
        for file_rel in failed_files:
            print(f"  - {file_rel}")
        sys.exit(1)

    print("\nAll files validated successfully!")
```
**EN:** This block defines `bump_version`. It accepts 3 parameter(s): new_version, version_file, files_to_update. Internally it checks file existence, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `bump_version`。它接收 3 个参数：new_version, version_file, files_to_update。内部会检查文件是否存在、输出状态信息、控制进程退出状态。

## Key Concepts / 关键概念
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制

## Dependencies / 依赖关系
- **Standard library / 标准库**: `pathlib`, `re`, `sys`, `tomllib`, `typing`
- **Third-party modules / 第三方模块**: `tomli`
