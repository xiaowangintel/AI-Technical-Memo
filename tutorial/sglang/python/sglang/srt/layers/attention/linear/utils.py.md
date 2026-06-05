# utils.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/linear/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides utils utilities shared by multiple attention implementations in SGLang. / 该模块提供 SGLang 多种注意力实现共享的 utils 工具函数。
## Line-by-Line Analysis / 逐行分析
### Lines 1-7: imports
```python
from __future__ import annotations

import logging
from enum import Enum
from typing import TYPE_CHECKING, Optional

from sglang.srt.utils.common import rank0_log
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 9-10: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 12-12: module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 15-20: class LinearAttnKernelBackend
```python
class LinearAttnKernelBackend(Enum):
    TRITON = "triton"
    CUTEDSL = "cutedsl"
    FLASHINFER = "flashinfer"
    CUSTOM = "custom"
```
**EN:** Enumeration that names execution modes, backend choices, or other symbolic options used by this module.
**CN:** 该枚举为本模块使用的执行模式、后端选择或其他符号化选项命名。

### Lines 21-23: method LinearAttnKernelBackend._missing_
```python
    @classmethod
    def _missing_(cls, value):
        return cls.CUSTOM
```
**EN:** Implements the missing routine used by this attention module.
**CN:** 实现该注意力模块使用的 missing 例程。

### Lines 25-26: method LinearAttnKernelBackend.is_triton
```python
    def is_triton(self):
        return self == LinearAttnKernelBackend.TRITON
```
**EN:** Checks whether is triton is valid for the current configuration or execution state.
**CN:** 检查在当前配置或执行状态下 is triton 是否成立。

### Lines 28-29: method LinearAttnKernelBackend.is_cutedsl
```python
    def is_cutedsl(self):
        return self == LinearAttnKernelBackend.CUTEDSL
```
**EN:** Checks whether is cutedsl is valid for the current configuration or execution state.
**CN:** 检查在当前配置或执行状态下 is cutedsl 是否成立。

### Lines 31-32: method LinearAttnKernelBackend.is_flashinfer
```python
    def is_flashinfer(self):
        return self == LinearAttnKernelBackend.FLASHINFER
```
**EN:** Checks whether is flashinfer is valid for the current configuration or execution state.
**CN:** 检查在当前配置或执行状态下 is flashinfer 是否成立。

### Lines 34-35: method LinearAttnKernelBackend.is_custom
```python
    def is_custom(self):
        return self == LinearAttnKernelBackend.CUSTOM
```
**EN:** Checks whether is custom is valid for the current configuration or execution state.
**CN:** 检查在当前配置或执行状态下 is custom 是否成立。

### Lines 38-39: module constants
```python
LINEAR_ATTN_DECODE_BACKEND: Optional[LinearAttnKernelBackend] = None
LINEAR_ATTN_PREFILL_BACKEND: Optional[LinearAttnKernelBackend] = None
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 42-53: function initialize_linear_attn_config
```python
def initialize_linear_attn_config(server_args: ServerArgs):
    global LINEAR_ATTN_DECODE_BACKEND
    global LINEAR_ATTN_PREFILL_BACKEND

    base = server_args.linear_attn_backend
    decode = server_args.linear_attn_decode_backend or base
    prefill = server_args.linear_attn_prefill_backend or base

    LINEAR_ATTN_DECODE_BACKEND = LinearAttnKernelBackend(decode)
    LINEAR_ATTN_PREFILL_BACKEND = LinearAttnKernelBackend(prefill)

    rank0_log(f"Linear attention kernel backend: decode={decode}, prefill={prefill}")
```
**EN:** Implements the initialize linear attn config routine used by this attention module.
**CN:** 实现该注意力模块使用的 initialize linear attn config 例程。

### Lines 56-63: function get_linear_attn_decode_backend
```python
def get_linear_attn_decode_backend() -> LinearAttnKernelBackend:
    global LINEAR_ATTN_DECODE_BACKEND
    if LINEAR_ATTN_DECODE_BACKEND is None:
        logger.warning(
            "LINEAR_ATTN_DECODE_BACKEND is not initialized, using triton backend"
        )
        LINEAR_ATTN_DECODE_BACKEND = LinearAttnKernelBackend.TRITON
    return LINEAR_ATTN_DECODE_BACKEND
```
**EN:** Computes and returns get linear attn decode backend from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get linear attn decode backend。

### Lines 66-73: function get_linear_attn_prefill_backend
```python
def get_linear_attn_prefill_backend() -> LinearAttnKernelBackend:
    global LINEAR_ATTN_PREFILL_BACKEND
    if LINEAR_ATTN_PREFILL_BACKEND is None:
        logger.warning(
            "LINEAR_ATTN_PREFILL_BACKEND is not initialized, using triton backend"
        )
        LINEAR_ATTN_PREFILL_BACKEND = LinearAttnKernelBackend.TRITON
    return LINEAR_ATTN_PREFILL_BACKEND
```
**EN:** Computes and returns get linear attn prefill backend from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get linear attn prefill backend。

## Key Concepts / 关键概念
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** FlashInfer execution path / **CN:** FlashInfer 执行路径

## Dependencies / 依赖关系
- `__future__.annotations`
- `logging`
- `enum.Enum`
- `typing.TYPE_CHECKING`
- `typing.Optional`
- `sglang.srt.utils.common.rank0_log`
- `sglang.srt.server_args.ServerArgs`
