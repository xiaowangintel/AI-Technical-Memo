# nccl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/nccl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-14)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from __future__ import annotations

import importlib.util
import os

import torch

import vllm.envs as envs
from vllm.logger import init_logger

logger = init_logger(__name__)
```
**EN:** Sets up the module with standard-library support such as `__future__`, `importlib.util`, `os`, external packages such as `torch`, vLLM modules such as `vllm.envs`, `vllm.logger`. It prepares the symbols later used by `find_nccl_library`, `find_nccl_include_paths`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `find_nccl_library`, `find_nccl_include_paths` 提供上下文。

### find_nccl_library (lines 17-35)
```python
def find_nccl_library() -> str:
    """Return NCCL/RCCL shared library name to load.

    Uses `VLLM_NCCL_SO_PATH` if set; otherwise chooses by torch backend.
    """
    so_file = envs.VLLM_NCCL_SO_PATH
    if so_file:
        logger.info(
            "Found nccl from environment variable VLLM_NCCL_SO_PATH=%s", so_file
        )
    else:
        if torch.version.cuda is not None:
            so_file = "libnccl.so.2"
        elif torch.version.hip is not None:
            so_file = "librccl.so.1"
        else:
            raise ValueError("NCCL only supports CUDA and ROCm backends.")
        logger.debug_once("Found nccl from library %s", so_file)
    return so_file
```
**EN:** `find_nccl_library`: Return NCCL/RCCL shared library name to load. Inside the body, it relies on `logger.info`, `logger.debug_once`, `ValueError` to complete the main steps.
**CN:** `find_nccl_library` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `logger.info`, `logger.debug_once`, `ValueError` 等函数完成关键步骤。

### find_nccl_include_paths (lines 38-64)
```python
def find_nccl_include_paths() -> list[str] | None:
    """Return possible include paths containing `nccl.h`.

    Considers `VLLM_NCCL_INCLUDE_PATH` and the `nvidia-nccl-cuXX` package.
    """
    paths: list[str] = []
    inc = envs.VLLM_NCCL_INCLUDE_PATH
    if inc and os.path.isdir(inc):
        paths.append(inc)

    try:
        spec = importlib.util.find_spec("nvidia.nccl")
        if spec and (locs := getattr(spec, "submodule_search_locations", None)):
            for loc in locs:
                inc_dir = os.path.join(loc, "include")
                if os.path.exists(os.path.join(inc_dir, "nccl.h")):
                    paths.append(inc_dir)
    except Exception as e:
        logger.debug("Failed to find nccl include path from nvidia.nccl package: %s", e)

    seen: set[str] = set()
    out: list[str] = []
    for p in paths:
        if p and p not in seen:
            out.append(p)
            seen.add(p)
    return out or None
```
**EN:** `find_nccl_include_paths`: Return possible include paths containing `nccl.h`. Inside the body, it relies on `os.path.isdir`, `paths.append`, `importlib.util.find_spec` to complete the main steps.
**CN:** `find_nccl_include_paths` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `os.path.isdir`, `paths.append`, `importlib.util.find_spec` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`find_nccl_library`**: Key helper or entry point in this file. / **`find_nccl_library`**：本文件中的关键辅助函数或入口。
- **`find_nccl_include_paths`**: Key helper or entry point in this file. / **`find_nccl_include_paths`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: __future__, importlib.util, os
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.logger
