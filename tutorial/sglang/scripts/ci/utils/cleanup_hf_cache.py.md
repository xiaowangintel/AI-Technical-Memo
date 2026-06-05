# cleanup_hf_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/cleanup_hf_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `cleanup_hf_cache` workflow in SGLang. It mainly handles CI orchestration, cleanup tasks. / 该Python 模块用于支撑 SGLang 中的 `cleanup_hf_cache` 流程，主要负责CI 编排、清理任务。它属于 `utils` 自动化路径的一部分。

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
Clean up stale HuggingFace cache artifacts from previous failed downloads.

This script removes incomplete marker files, temporary files, and lock files
from the HuggingFace cache directory. These artifacts can accumulate from
interrupted or failed downloads and may interfere with future downloads.
"""
```
**EN:** Clean up stale HuggingFace cache artifacts from previous failed downloads.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 10-13: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import os
import sys
from pathlib import Path
from typing import List
```
**EN:** This block loads os, sys, pathlib, typing. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 os, sys, pathlib, typing。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 15-21: Top-level try logic / 顶层 try 逻辑
```python
try:
    from huggingface_hub import constants

    HF_HUB_AVAILABLE = True
except ImportError:
    print("Warning: huggingface_hub not available")
    HF_HUB_AVAILABLE = False
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 24-31: Defines the `get_hf_cache_dir` routine / 定义 `get_hf_cache_dir` 例程
```python
def get_hf_cache_dir() -> str:
    """Get the HuggingFace cache directory."""
    if HF_HUB_AVAILABLE:
        return constants.HF_HUB_CACHE

    # Fallback to environment variable or default
    hf_home = os.environ.get("HF_HOME", os.path.expanduser("~/.cache/huggingface"))
    return os.path.join(hf_home, "hub")
```
**EN:** This block defines `get_hf_cache_dir`. Get the HuggingFace cache directory. It takes no explicit parameters at the top level. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_hf_cache_dir`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 34-60: Defines the `find_stale_artifacts` routine / 定义 `find_stale_artifacts` 例程
```python
def find_stale_artifacts(cache_dir: str) -> List[Path]:
    """
    Find stale artifact files in the HuggingFace cache.

    Args:
        cache_dir: HuggingFace cache directory

    Returns:
        List of paths to stale artifact files
    """
    cache_path = Path(cache_dir)

    if not cache_path.exists():
        return []

    # Patterns for stale files to clean up
    patterns = [
        "**/*.incomplete",  # Incomplete download markers
        "**/*.tmp",  # Temporary files
        "**/*.lock",  # Lock files from interrupted downloads
    ]

    stale_files = []
    for pattern in patterns:
        stale_files.extend(cache_path.glob(pattern))

    return stale_files
```
**EN:** This block defines `find_stale_artifacts`. Find stale artifact files in the HuggingFace cache. It accepts 1 parameter(s): cache_dir. Internally it touches filesystem paths, checks file existence.
**CN:** 该代码块定义了 `find_stale_artifacts`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：cache_dir。内部会处理文件系统路径、检查文件是否存在。

### Lines 63-85: Defines the `cleanup_artifacts` routine / 定义 `cleanup_artifacts` 例程
```python
def cleanup_artifacts(artifacts: List[Path]) -> tuple[int, int]:
    """
    Remove stale artifact files.

    Args:
        artifacts: List of file paths to remove

    Returns:
        Tuple of (successful_removals, failed_removals)
    """
    successful = 0
    failed = 0

    for file_path in artifacts:
        try:
            file_path.unlink()
            print(f"  Removed: {file_path}")
            successful += 1
        except Exception as e:
            print(f"  Warning: Could not remove {file_path}: {e}")
            failed += 1

    return successful, failed
```
**EN:** This block defines `cleanup_artifacts`. Remove stale artifact files. It accepts 1 parameter(s): artifacts. Internally it emits status messages.
**CN:** 该代码块定义了 `cleanup_artifacts`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：artifacts。内部会输出状态信息。

### Lines 88-130: Defines the `main` routine / 定义 `main` 例程
```python
def main() -> int:
    """
    Main cleanup logic.

    Returns:
        Always returns 0 (cleanup is best-effort and should not fail CI)
    """
    print("=" * 70)
    print("HuggingFace Cache Cleanup")
    print("=" * 70)

    # Get cache directory
    cache_dir = get_hf_cache_dir()
    print(f"Cache directory: {cache_dir}")

    if not os.path.exists(cache_dir):
        print("Cache directory does not exist - nothing to clean")
        return 0

    print("-" * 70)

    # Find stale artifacts
    print("Scanning for stale artifacts...")
    stale_artifacts = find_stale_artifacts(cache_dir)

    if not stale_artifacts:
        print("✓ No stale cache artifacts found")
        return 0

    # Clean up artifacts
    print(f"Found {len(stale_artifacts)} stale artifact(s) to remove:")
    successful, failed = cleanup_artifacts(stale_artifacts)

    print("-" * 70)

    # Summary
    if failed > 0:
        print(f"⚠ Cleaned up {successful} file(s), {failed} removal(s) failed")
    else:
        print(f"✓ Successfully cleaned up {successful} stale file(s)")

    # Always return 0 - cleanup failures should not fail CI
    return 0
```
**EN:** This block defines `main`. Main cleanup logic. It takes no explicit parameters at the top level. Internally it checks file existence, emits status messages.
**CN:** 该代码块定义了 `main`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会检查文件是否存在、输出状态信息。

### Lines 133-146: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    try:
        exit_code = main()
        sys.exit(exit_code)
    except KeyboardInterrupt:
        print("\nInterrupted by user")
        sys.exit(0)
    except Exception as e:
        print(f"ERROR: Unexpected error during cleanup: {e}")
        import traceback

        traceback.print_exc()
        # Still return 0 - cleanup failures should not fail CI
        sys.exit(0)
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It emits status messages, controls process exit status.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会输出状态信息、控制进程退出状态。

## Key Concepts / 关键概念
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `pathlib`, `sys`, `traceback`, `typing`
- **Third-party modules / 第三方模块**: `huggingface_hub`
