# interface.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/platforms/interface.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for interface in the multimodal generation stack. Key symbols include `AttentionBackendEnum`, `PlatformEnum`, `CpuArchEnum`. / 该模块包含多模态生成体系中与 interface 相关的运行时支持代码。 关键符号包括 `AttentionBackendEnum`, `PlatformEnum`, `CpuArchEnum`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from vllm: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/platforms/interface.py
from __future__ import annotations

import enum
import random
from collections.abc import Callable
from functools import lru_cache
from typing import TYPE_CHECKING, Any, NamedTuple

import numpy as np
import torch
# ...
        AttentionImpl,
    )

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 27-55: Class `AttentionBackendEnum` / 类 `AttentionBackendEnum`
```python
class AttentionBackendEnum(enum.Enum):
    FA2 = enum.auto()
    FA = enum.auto()
    SLIDING_TILE_ATTN = enum.auto()
    TORCH_SDPA = enum.auto()
    SAGE_ATTN = enum.auto()
    SAGE_ATTN_3 = enum.auto()
    VIDEO_SPARSE_ATTN = enum.auto()
    SPARSE_VIDEO_GEN_2_ATTN = enum.auto()
    VMOBA_ATTN = enum.auto()
    AITER = enum.auto()
    AITER_SAGE = enum.auto()
    SLA_ATTN = enum.auto()
    SAGE_SLA_ATTN = enum.auto()
# ...
            AttentionBackendEnum.VMOBA_ATTN,
            AttentionBackendEnum.SLA_ATTN,
            AttentionBackendEnum.SAGE_SLA_ATTN,
        }
```
**EN:** This class models `AttentionBackendEnum` as a specialization of `enum.Enum`. Important methods include `__str__`, `is_sparse`.
**CN:** 该类实现 `AttentionBackendEnum`，并继承/扩展 `enum.Enum`。 其中较重要的方法包括 `__str__`, `is_sparse`。

### Lines 58-68: Class `PlatformEnum` / 类 `PlatformEnum`
```python
class PlatformEnum(enum.Enum):
    CUDA = enum.auto()
    ROCM = enum.auto()
    TPU = enum.auto()
    CPU = enum.auto()
    MPS = enum.auto()
    NPU = enum.auto()
    MUSA = enum.auto()
    XPU = enum.auto()
    OOT = enum.auto()
    UNSPECIFIED = enum.auto()
```
**EN:** This class models `PlatformEnum` as a specialization of `enum.Enum`.
**CN:** 该类实现 `PlatformEnum`，并继承/扩展 `enum.Enum`。

### Lines 71-74: Class `CpuArchEnum` / 类 `CpuArchEnum`
```python
class CpuArchEnum(enum.Enum):
    X86 = enum.auto()
    ARM = enum.auto()
    UNSPECIFIED = enum.auto()
```
**EN:** This class models `CpuArchEnum` as a specialization of `enum.Enum`.
**CN:** 该类实现 `CpuArchEnum`，并继承/扩展 `enum.Enum`。

### Lines 77-91: Class `DeviceCapability` / 类 `DeviceCapability`
```python
class DeviceCapability(NamedTuple):
    major: int
    minor: int

    def as_version_str(self) -> str:
        return f"{self.major}.{self.minor}"

    def to_int(self) -> int:
        """
        Express device capability as an integer ``<major><minor>``.

        It is assumed that the minor version is always a single digit.
        """
        assert 0 <= self.minor < 10
        return self.major * 10 + self.minor
```
**EN:** This class models `DeviceCapability` as a specialization of `NamedTuple`. Important methods include `as_version_str`, `to_int`.
**CN:** 该类实现 `DeviceCapability`，并继承/扩展 `NamedTuple`。 其中较重要的方法包括 `as_version_str`, `to_int`。

### Lines 94-415: Class `Platform` / 类 `Platform`
```python
class Platform:
    _enum: PlatformEnum
    device_name: str
    device_type: str
    device: torch.device | None = None  # Dummy attribute for compatibility

    # available dispatch keys:
    # check https://github.com/pytorch/pytorch/blob/313dac6c1ca0fa0cde32477509cce32089f8532a/torchgen/model.py#L134 # noqa
    # use "CPU" as a fallback for platforms not registered in PyTorch
    dispatch_key: str = "CPU"

    # The torch.compile backend for compiling simple and
    # standalone functions. The default value is "inductor" to keep
    # the same behavior as PyTorch.
# ...

    def get_attn_backend(self, *args, **kwargs) -> AttentionImpl:
        attention_cls_str = self.get_attn_backend_cls_str(*args, **kwargs)
        return resolve_obj_by_qualname(attention_cls_str)
```
**EN:** This class models `Platform`. Important methods include `is_cuda`, `is_npu`, `is_rocm`, `is_tpu`.
**CN:** 该类实现 `Platform`。 其中较重要的方法包括 `is_cuda`, `is_npu`, `is_rocm`, `is_tpu`。

### Lines 418-420: Class `UnspecifiedPlatform` / 类 `UnspecifiedPlatform`
```python
class UnspecifiedPlatform(Platform):
    _enum = PlatformEnum.UNSPECIFIED
    device_type = ""
```
**EN:** This class models `UnspecifiedPlatform` as a specialization of `Platform`.
**CN:** 该类实现 `UnspecifiedPlatform`，并继承/扩展 `Platform`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`
- **External / 外部**: `__future__`, `numpy`, `torch`
- **Stdlib / 标准库**: `enum`, `random`, `collections.abc`, `functools`, `typing`
