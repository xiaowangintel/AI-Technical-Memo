# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module organizes Python logic for `utilities`, including reusable helpers, configuration, or integration code. / 该模块组织与 `utilities` 相关的 Python 逻辑，包括可复用辅助函数、配置或集成代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Comments and local context
````python
# Copyright 2025 SGLang Team. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 16-19: Imports and module setup
````python
import functools
from typing import Dict, Tuple

import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 21-21: Constants and configuration
````python
_cache_buf: Dict[Tuple[str, torch.device], torch.Tensor] = {}
````
**EN:** This block defines shared constants or configuration values such as `_cache_buf`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_cache_buf`），供后续函数或控制流程复用。

### Lines 24-30: `_get_cache_buf` definition
````python
def _get_cache_buf(name: str, bytes: int, device: torch.device) -> torch.Tensor:
    key = (name, device)
    buf = _cache_buf.get(key)
    if buf is None:
        buf = torch.empty(bytes, dtype=torch.uint8, device=device)
        _cache_buf[key] = buf
    return buf
````
**EN:** This section defines `_get_cache_buf` and implements the core logic associated with get cache buf.
**CN:** 该部分定义 `_get_cache_buf`，并实现与 get cache buf 相关的核心逻辑。

### Lines 33-37: `_to_tensor_scalar_tuple` definition
````python
def _to_tensor_scalar_tuple(x):
    if isinstance(x, torch.Tensor):
        return (x, 0)
    else:
        return (None, x)
````
**EN:** This section defines `_to_tensor_scalar_tuple` and implements the core logic associated with to tensor scalar tuple.
**CN:** 该部分定义 `_to_tensor_scalar_tuple`，并实现与 to tensor scalar tuple 相关的核心逻辑。

### Lines 40-54: `cache_once` definition
````python
def cache_once(fn):
    """
    NOTE: `functools.lru_cache` is not compatible with `torch.compile`
    So we manually implement a simple cache_once decorator to replace it.
    """
    result_map = {}

    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        key = (args, tuple(sorted(kwargs.items())))
        if key not in result_map:
            result_map[key] = fn(*args, **kwargs)
        return result_map[key]

    return wrapper
````
**EN:** This section defines `cache_once` and implements the core logic associated with cache once. Docstring summary: NOTE: `functools.lru_cache` is not compatible with `torch.compile` So we manually implement a simple cache_once decorator to replace it.
**CN:** 该部分定义 `cache_once`，并实现与 cache once 相关的核心逻辑。 文档字符串摘要：NOTE: `functools.lru_cache` is not compatible with `torch.compile` So we manually implement a simple cache_once decorator to replace it.

### Lines 57-66: `is_arch_support_pdl` definition
````python
@cache_once
def is_arch_support_pdl() -> bool:
    if getattr(torch.version, "hip", None) or getattr(torch.version, "musa", None):
        return False
    try:
        device = torch.cuda.current_device()
        major, _ = torch.cuda.get_device_capability(device)
    except Exception:
        return False
    return major >= 9
````
**EN:** This section defines `is_arch_support_pdl` and implements the core logic associated with is arch support pdl.
**CN:** 该部分定义 `is_arch_support_pdl`，并实现与 is arch support pdl 相关的核心逻辑。

## Key Concepts / 关键概念
- **Role / 角色**: Reusable Python module / 可复用 Python 模块
- **Primary symbols / 主要符号**: `_get_cache_buf`, `_to_tensor_scalar_tuple`, `cache_once`, `is_arch_support_pdl`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `functools`, `torch`, `typing`
