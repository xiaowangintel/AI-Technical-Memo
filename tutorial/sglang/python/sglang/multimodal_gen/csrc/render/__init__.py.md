# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/csrc/render/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `_get_build_directory`, `_is_recoverable_load_error`, and `load_extension_with_recovery`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于多模态生成模块。它围绕 `_get_build_directory`、`_is_recoverable_load_error` 和 `load_extension_with_recovery` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

import os
import shutil
import sys
from pathlib import Path
from typing import Any, Sequence

import torch

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `__future__`, `os`, `shutil`, `sys`, `pathlib`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`os`、`shutil`、`sys`、`pathlib` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 13-13: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 16-36: `_get_build_directory` implementation / `_get_build_directory` 实现
```python
def _get_build_directory(name: str) -> Path:
    try:
        from torch.utils.cpp_extension import _get_build_directory

        return Path(_get_build_directory(name, False))
    except (ImportError, AttributeError):
        from torch.utils.cpp_extension import get_default_build_root

        root = os.environ.get("TORCH_EXTENSIONS_DIR") or get_default_build_root()
        if "TORCH_EXTENSIONS_DIR" not in os.environ:
            cu_str = (
                "cpu"
                if torch.version.cuda is None
                else f"cu{torch.version.cuda.replace('.', '')}"
            )
            py_str = (
                f"py{sys.version_info.major}{sys.version_info.minor}"
                f"{getattr(sys, 'abiflags', '')}"
            )
            root = os.path.join(root, f"{py_str}_{cu_str}")
        return Path(root) / name
```
**EN:** This block defines function `_get_build_directory`. It retrieves build directory. Key calls include `Path`, `_get_build_directory`, `os.environ.get`, `get_default_build_root`, and `os.path.join`. The implementation branches on conditions, handles exceptional paths. Parameters such as `name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_get_build_directory`。 它用于获取build directory。 关键调用包括 `Path`、`_get_build_directory`、`os.environ.get`、`get_default_build_root` 和 `os.path.join`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `name` 等参数驱动。

### Lines 39-82: `_is_recoverable_load_error` implementation / `_is_recoverable_load_error` 实现
```python
def _is_recoverable_load_error(
    exc: BaseException, name: str, build_directory: Path
) -> bool:
    message = str(exc).lower()
    current = exc.__cause__ or exc.__context__
    while current is not None:
        message += f"\n{current}".lower()
        current = current.__cause__ or current.__context__

    if any(
        marker in message
        for marker in (
            "error building extension",
            "error compiling objects for extension",
            "ninja",
            "nvcc",
            "gcc",
            "g++",
            "fatal error:",
            "compilation terminated",
        )
    ):
        return False

    if not any(
        marker in message
        for marker in (str(build_directory / f"{name}.so").lower(), f"{name}.so")
    ):
        return False

    return any(
        marker in message
        for marker in (
            "undefined symbol",
            "cannot open shared object file",
            "no such file or directory",
            "file too short",
            "invalid elf header",
            "wrong elf class",
            "elf load command",
            "dlopen",
            "version `glibcxx",
        )
    )
```
**EN:** This block defines function `_is_recoverable_load_error`. It handles is recoverable load error logic. Key calls include `str.lower`, `any`, `lower`, and `str`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `exc`, `name`, and `build_directory` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_is_recoverable_load_error`。 它用于处理 is recoverable load error 相关逻辑。 关键调用包括 `str.lower`、`any`、`lower` 和 `str`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `exc`、`name` 和 `build_directory` 等参数驱动。

### Lines 85-127: `load_extension_with_recovery` implementation / `load_extension_with_recovery` 实现
```python
def load_extension_with_recovery(
    name: str,
    sources: Sequence[str],
    extra_cflags: Sequence[str] | None = None,
    extra_cuda_cflags: Sequence[str] | None = None,
    verbose: bool = False,
) -> Any:
    from torch.utils.cpp_extension import load

    try:
        return load(
            name=name,
            sources=list(sources),
            extra_cflags=None if extra_cflags is None else list(extra_cflags),
            extra_cuda_cflags=(
                None if extra_cuda_cflags is None else list(extra_cuda_cflags)
            ),
            verbose=verbose,
        )
    except Exception as exc:
        build_directory = _get_build_directory(name)
        if not _is_recoverable_load_error(exc, name, build_directory):
            raise

        logger.warning(
            "Detected a stale or broken JIT extension for %s at %s; clearing "
            "its cache and retrying once.",
            name,
            build_directory,
        )
        sys.modules.pop(name, None)
        if build_directory.exists():
            shutil.rmtree(build_directory)

        return load(
            name=name,
            sources=list(sources),
            extra_cflags=None if extra_cflags is None else list(extra_cflags),
            extra_cuda_cflags=(
                None if extra_cuda_cflags is None else list(extra_cuda_cflags)
            ),
            verbose=verbose,
        )
```
**EN:** This block defines function `load_extension_with_recovery`. It loads extension with recovery. Key calls include `load`, `_get_build_directory`, `logger.warning`, `sys.modules.pop`, and `build_directory.exists`. The implementation branches on conditions, handles exceptional paths. Parameters such as `name`, `sources`, `extra_cflags`, `extra_cuda_cflags`, and `verbose` drive the behavior in this section.
**CN:** 该代码块定义了函数 `load_extension_with_recovery`。 它用于加载extension with recovery。 关键调用包括 `load`、`_get_build_directory`、`logger.warning`、`sys.modules.pop` 和 `build_directory.exists`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `name`、`sources`、`extra_cflags`、`extra_cuda_cflags` 和 `verbose` 等参数驱动。

### Lines 130-130: supporting statements / 辅助语句
```python
__all__ = ["load_extension_with_recovery"]
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `__all__`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `__all__` 等名称。

## Key Concepts / 关键概念
- `_get_build_directory`: Top-level function that retrieves build directory. / 顶层函数，用于获取build directory。
- `_is_recoverable_load_error`: Top-level function that handles is recoverable load error logic. / 顶层函数，用于处理 is recoverable load error 相关逻辑。
- `load_extension_with_recovery`: Top-level function that loads extension with recovery. / 顶层函数，用于加载extension with recovery。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `os`, `shutil`, `sys`, `pathlib`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.utils.cpp_extension`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 130
