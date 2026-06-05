# dp_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/dp_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with attention-related tensor transforms for the SGLang SRT runtime. It exposes symbols such as `DpPaddingMode`, `_DpGatheredBufferWrapper`, `set_dp_buffer_len`, and `get_global_dp_buffer` and connects them to backend-specific paths such as `CUDA`, `Triton`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合注意力相关张量变换。它提供了 `DpPaddingMode`、`_DpGatheredBufferWrapper`、`set_dp_buffer_len` 以及 `get_global_dp_buffer` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-52: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import functools
import logging
from contextlib import contextmanager
from enum import IntEnum, auto
from typing import TYPE_CHECKING, List, Optional, Tuple

import torch
import triton
import triton.language as tl

from sglang.srt.distributed import (
    GroupCoordinator,
    get_attn_context_model_parallel_rank,
    get_attn_context_model_parallel_world_size,
    get_attn_cp_group,
    get_attn_tensor_model_parallel_rank,
    get_attn_tensor_model_parallel_world_size,
    get_attn_tp_group,
)
from sglang.srt.distributed import get_moe_dp_group as _get_moe_dp_group
from sglang.srt.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    get_tp_group,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.utils import get_bool_env_var, is_hip

if TYPE_CHECKING:
    from sglang.srt.configs.model_config import ModelConfig
    from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)

if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch

_ATTN_DP_RANK: Optional[int] = None
_ATTN_DP_SIZE: Optional[int] = None
_LOCAL_ATTN_DP_SIZE: Optional[int] = None
_LOCAL_ATTN_DP_RANK: Optional[int] = None
_ENABLE_DP_ATTENTION_FLAG: bool = False

_is_hip = is_hip()
_USE_ROCM700A_WA = _is_hip and get_bool_env_var("SGLANG_USE_ROCM700A")
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `functools`, `logging`, `contextlib.contextmanager`, `enum.IntEnum`, and `enum.auto`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger`, `_ATTN_DP_RANK`, `_ATTN_DP_SIZE`, `_LOCAL_ATTN_DP_SIZE`, and `_LOCAL_ATTN_DP_RANK` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`functools`、`logging`、`contextlib.contextmanager`、`enum.IntEnum` 以及 `enum.auto`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger`、`_ATTN_DP_RANK`、`_ATTN_DP_SIZE`、`_LOCAL_ATTN_DP_SIZE` 以及 `_LOCAL_ATTN_DP_RANK` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 53-59: Class `DpPaddingMode` declaration and shared state
```python
class DpPaddingMode(IntEnum):

    # Padding tokens to max length and then gather tokens using `all_gather_into_tensor`
    MAX_LEN = auto()
    # Padding tokens to sum length and then gather tokens using `all_reduce`
    SUM_LEN = auto()
```
**EN:** This block introduces class `DpPaddingMode` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DpPaddingMode`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 60-62: `DpPaddingMode.is_max_len` predicate for is max len
```python
    def is_max_len(self):
        return self == DpPaddingMode.MAX_LEN
```
**EN:** This block defines `DpPaddingMode.is_max_len` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DpPaddingMode.is_max_len`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 63-65: `DpPaddingMode.is_sum_len` predicate for is sum len
```python
    def is_sum_len(self):
        return self == DpPaddingMode.SUM_LEN
```
**EN:** This block defines `DpPaddingMode.is_sum_len` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DpPaddingMode.is_sum_len`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 66-87: `DpPaddingMode.get_dp_padding_mode` getter for data parallel padding mode
```python
    @classmethod
    def get_dp_padding_mode(
        cls, is_extend_in_batch, global_num_tokens: List[int]
    ) -> DpPaddingMode:
        dp_size = get_attention_dp_size()

        # When is_extend_in_batch and dp_size > 1, use SUM_LEN to avoid padding
        # overhead from uneven token distribution.
        # For dp_size=1, max_len equals sum_len, so prefer MAX_LEN mode
        # to enable symmetric memory optimization (needed for NSA CP, etc.).
        if is_extend_in_batch and dp_size > 1:
            return DpPaddingMode.SUM_LEN

        # we choose the mode that minimizes the communication cost
        # prefer MAX_LEN when communication cost is equal to enable symmetric memory
        max_len = max(global_num_tokens)
        sum_len = sum(global_num_tokens)
        if sum_len * 2 >= max_len * dp_size:
            return cls.MAX_LEN
        else:
            return cls.SUM_LEN
```
**EN:** This block defines `DpPaddingMode.get_dp_padding_mode` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_attention_dp_size`, `max`, and `sum`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dp_size`, `max_len`, and `sum_len` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DpPaddingMode.get_dp_padding_mode`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_attention_dp_size`、`max` 以及 `sum`，说明该流程会编排底层辅助函数或计算内核。 像 `dp_size`、`max_len` 以及 `sum_len` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 88-97: `DpPaddingMode.get_default_mode_in_cuda_graph` getter for default mode in CUDA graph
```python
    @classmethod
    def get_default_mode_in_cuda_graph(cls) -> DpPaddingMode:
        # TODO(kkhuang-amd): noqa, temporary work-around for rocm 7.0.0 alpha
        # it can be safely removed later, once RCCL fixed
        if _USE_ROCM700A_WA:
            return cls.SUM_LEN
        else:
            return cls.MAX_LEN
```
**EN:** This block defines `DpPaddingMode.get_default_mode_in_cuda_graph` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DpPaddingMode.get_default_mode_in_cuda_graph`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 98-108: Class `_DpGatheredBufferWrapper` declaration and shared state
```python
class _DpGatheredBufferWrapper:

    _hidden_size: int
    _dtype: torch.dtype
    _device: torch.device
    _global_dp_buffer_len: int
    _local_dp_buffer_len: int
    _dp_max_padding: bool
    _global_num_tokens: Optional[List[int]]
    _is_extend_in_batch: bool
```
**EN:** This block introduces class `_DpGatheredBufferWrapper` and the state shared by its methods.
**CN:** 该代码块引入类 `_DpGatheredBufferWrapper`，并定义其方法共享的状态。

### Lines 109-114: `_DpGatheredBufferWrapper.set_metadata` setter for metadata
```python
    @classmethod
    def set_metadata(cls, hidden_size: int, dtype: torch.dtype, device: torch.device):
        cls._hidden_size = hidden_size
        cls._dtype = dtype
        cls._device = device
```
**EN:** This block defines `_DpGatheredBufferWrapper.set_metadata` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.set_metadata`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 115-127: `_DpGatheredBufferWrapper.set_dp_buffer_len` setter for data parallel buffer len
```python
    @classmethod
    def set_dp_buffer_len(
        cls,
        global_dp_buffer_len: int,
        local_dp_buffer_len: int,
        dp_max_padding: bool,
        global_num_tokens: Optional[List[int]] = None,
    ):
        cls._global_dp_buffer_len = global_dp_buffer_len
        cls._local_dp_buffer_len = local_dp_buffer_len
        cls._dp_max_padding = dp_max_padding
        cls._global_num_tokens = global_num_tokens
```
**EN:** This block defines `_DpGatheredBufferWrapper.set_dp_buffer_len` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.set_dp_buffer_len`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 128-137: `_DpGatheredBufferWrapper.get_global_dp_buffer` getter for global data parallel buffer
```python
    @classmethod
    def get_global_dp_buffer(cls, group: GroupCoordinator) -> torch.Tensor:
        with use_symmetric_memory(group, disabled=not cls._dp_max_padding):
            buffer = torch.empty(
                (cls._global_dp_buffer_len, cls._hidden_size),
                dtype=cls._dtype,
                device=cls._device,
            )
        return buffer
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_global_dp_buffer` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `use_symmetric_memory` and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_global_dp_buffer`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `use_symmetric_memory` 和 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 138-147: `_DpGatheredBufferWrapper.get_local_dp_buffer` getter for local data parallel buffer
```python
    @classmethod
    def get_local_dp_buffer(cls, group: GroupCoordinator) -> torch.Tensor:
        with use_symmetric_memory(group, disabled=not cls._dp_max_padding):
            buffer = torch.empty(
                (cls._local_dp_buffer_len, cls._hidden_size),
                dtype=cls._dtype,
                device=cls._device,
            )
        return buffer
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_local_dp_buffer` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `use_symmetric_memory` and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_local_dp_buffer`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `use_symmetric_memory` 和 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 148-151: `_DpGatheredBufferWrapper.get_global_dp_buffer_len` getter for global data parallel buffer len
```python
    @classmethod
    def get_global_dp_buffer_len(cls) -> int:
        return cls._global_dp_buffer_len
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_global_dp_buffer_len` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_global_dp_buffer_len`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 152-155: `_DpGatheredBufferWrapper.get_local_dp_buffer_len` getter for local data parallel buffer len
```python
    @classmethod
    def get_local_dp_buffer_len(cls) -> int:
        return cls._local_dp_buffer_len
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_local_dp_buffer_len` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_local_dp_buffer_len`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 156-159: `_DpGatheredBufferWrapper.get_dp_global_num_tokens` getter for data parallel global num tokens
```python
    @classmethod
    def get_dp_global_num_tokens(cls) -> List[int]:
        return cls._global_num_tokens
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_dp_global_num_tokens` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_dp_global_num_tokens`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 160-163: `_DpGatheredBufferWrapper.get_dp_hidden_size` getter for data parallel hidden size
```python
    @classmethod
    def get_dp_hidden_size(cls) -> int:
        return cls._hidden_size
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_dp_hidden_size` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_dp_hidden_size`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 164-167: `_DpGatheredBufferWrapper.get_dp_dtype` getter for data parallel dtype
```python
    @classmethod
    def get_dp_dtype(cls) -> torch.dtype:
        return cls._dtype
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_dp_dtype` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_dp_dtype`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 168-171: `_DpGatheredBufferWrapper.get_dp_device` getter for data parallel device
```python
    @classmethod
    def get_dp_device(cls) -> torch.device:
        return cls._device
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_dp_device` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_dp_device`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 172-175: `_DpGatheredBufferWrapper.set_is_extend_in_batch` setter for is extend in batch
```python
    @classmethod
    def set_is_extend_in_batch(cls, is_extend_in_batch: bool):
        cls._is_extend_in_batch = is_extend_in_batch
```
**EN:** This block defines `_DpGatheredBufferWrapper.set_is_extend_in_batch` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.set_is_extend_in_batch`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 176-179: `_DpGatheredBufferWrapper.get_is_extend_in_batch` getter for is extend in batch
```python
    @classmethod
    def get_is_extend_in_batch(cls) -> bool:
        return cls._is_extend_in_batch
```
**EN:** This block defines `_DpGatheredBufferWrapper.get_is_extend_in_batch` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.get_is_extend_in_batch`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 180-184: `_DpGatheredBufferWrapper.is_dp_max_padding` predicate for is data parallel max padding
```python
    @classmethod
    def is_dp_max_padding(cls) -> bool:
        return cls._dp_max_padding
```
**EN:** This block defines `_DpGatheredBufferWrapper.is_dp_max_padding` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DpGatheredBufferWrapper.is_dp_max_padding`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 185-195: `set_dp_buffer_len` setter for data parallel buffer len
```python
def set_dp_buffer_len(
    global_dp_buffer_len: int,
    local_dp_buffer_len: int,
    dp_max_padding: bool,
    global_num_tokens: Optional[List[int]] = None,
):
    _DpGatheredBufferWrapper.set_dp_buffer_len(
        global_dp_buffer_len, local_dp_buffer_len, dp_max_padding, global_num_tokens
    )
```
**EN:** This block defines `set_dp_buffer_len` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.set_dp_buffer_len`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `set_dp_buffer_len`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.set_dp_buffer_len`，说明该流程会编排底层辅助函数或计算内核。

### Lines 196-199: `get_global_dp_buffer` getter for global data parallel buffer
```python
def get_global_dp_buffer(group: GroupCoordinator) -> torch.Tensor:
    return _DpGatheredBufferWrapper.get_global_dp_buffer(group=group)
```
**EN:** This block defines `get_global_dp_buffer` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_global_dp_buffer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_global_dp_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_global_dp_buffer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 200-203: `get_local_dp_buffer` getter for local data parallel buffer
```python
def get_local_dp_buffer(group: GroupCoordinator) -> torch.Tensor:
    return _DpGatheredBufferWrapper.get_local_dp_buffer(group=group)
```
**EN:** This block defines `get_local_dp_buffer` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_local_dp_buffer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_local_dp_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_local_dp_buffer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 204-207: `get_global_dp_buffer_len` getter for global data parallel buffer len
```python
def get_global_dp_buffer_len() -> int:
    return _DpGatheredBufferWrapper.get_global_dp_buffer_len()
```
**EN:** This block defines `get_global_dp_buffer_len` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_global_dp_buffer_len`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_global_dp_buffer_len`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_global_dp_buffer_len`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 208-211: `get_local_dp_buffer_len` getter for local data parallel buffer len
```python
def get_local_dp_buffer_len() -> int:
    return _DpGatheredBufferWrapper.get_local_dp_buffer_len()
```
**EN:** This block defines `get_local_dp_buffer_len` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_local_dp_buffer_len`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_local_dp_buffer_len`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_local_dp_buffer_len`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 212-215: `get_dp_global_num_tokens` getter for data parallel global num tokens
```python
def get_dp_global_num_tokens() -> List[int]:
    return _DpGatheredBufferWrapper.get_dp_global_num_tokens()
```
**EN:** This block defines `get_dp_global_num_tokens` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_dp_global_num_tokens`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_dp_global_num_tokens`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_dp_global_num_tokens`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 216-219: `get_dp_hidden_size` getter for data parallel hidden size
```python
def get_dp_hidden_size() -> int:
    return _DpGatheredBufferWrapper.get_dp_hidden_size()
```
**EN:** This block defines `get_dp_hidden_size` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_dp_hidden_size`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_dp_hidden_size`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_dp_hidden_size`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 220-223: `get_dp_dtype` getter for data parallel dtype
```python
def get_dp_dtype() -> torch.dtype:
    return _DpGatheredBufferWrapper.get_dp_dtype()
```
**EN:** This block defines `get_dp_dtype` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_dp_dtype`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_dp_dtype`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_dp_dtype`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 224-227: `get_dp_device` getter for data parallel device
```python
def get_dp_device() -> torch.device:
    return _DpGatheredBufferWrapper.get_dp_device()
```
**EN:** This block defines `get_dp_device` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_dp_device`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_dp_device`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_dp_device`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 228-231: `set_is_extend_in_batch` setter for is extend in batch
```python
def set_is_extend_in_batch(is_extend_in_batch: bool):
    _DpGatheredBufferWrapper.set_is_extend_in_batch(is_extend_in_batch)
```
**EN:** This block defines `set_is_extend_in_batch` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.set_is_extend_in_batch`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `set_is_extend_in_batch`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.set_is_extend_in_batch`，说明该流程会编排底层辅助函数或计算内核。

### Lines 232-235: `get_is_extend_in_batch` getter for is extend in batch
```python
def get_is_extend_in_batch() -> bool:
    return _DpGatheredBufferWrapper.get_is_extend_in_batch()
```
**EN:** This block defines `get_is_extend_in_batch` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.get_is_extend_in_batch`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_is_extend_in_batch`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.get_is_extend_in_batch`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 236-239: `is_dp_max_padding` predicate for is data parallel max padding
```python
def is_dp_max_padding() -> bool:
    return _DpGatheredBufferWrapper.is_dp_max_padding()
```
**EN:** This block defines `is_dp_max_padding` and contains the main logic for this step. It mainly invokes `_DpGatheredBufferWrapper.is_dp_max_padding`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_dp_max_padding`，并承载这一阶段的核心逻辑。 它主要调用 `_DpGatheredBufferWrapper.is_dp_max_padding`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 240-256: `compute_dp_attention_world_info` step for data parallel attention world info
```python
def compute_dp_attention_world_info(
    enable_dp_attention, tp_rank, tp_size, dp_size, attn_cp_size: int = 1
):
    attn_dp_size = dp_size if enable_dp_attention else 1
    attn_tp_size = tp_size // attn_dp_size // attn_cp_size
    attn_tp_rank = tp_rank % attn_tp_size

    if not enable_dp_attention:
        attn_dp_rank = 0
    else:
        # Rank layout is (dp, cp, tp) where tp is the fastest-changing dim:
        # tp_rank = (attn_dp_rank * attn_cp_size + attn_cp_rank) * attn_tp_size + attn_tp_rank
        attn_dp_rank = tp_rank // (attn_tp_size * attn_cp_size)

    return attn_tp_rank, attn_tp_size, attn_dp_rank, attn_dp_size
```
**EN:** This block defines `compute_dp_attention_world_info` and contains the main logic for this step. Intermediate names such as `attn_dp_size`, `attn_tp_size`, `attn_tp_rank`, and `attn_dp_rank` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `compute_dp_attention_world_info`，并承载这一阶段的核心逻辑。 像 `attn_dp_size`、`attn_tp_size`、`attn_tp_rank` 以及 `attn_dp_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 257-273: `compute_dp_attention_local_info` step for data parallel attention local info
```python
def compute_dp_attention_local_info(
    enable_dp_attention, tp_rank, tp_size, dp_size, moe_dense_tp_size
):
    if not enable_dp_attention:
        return tp_rank, tp_size, 0

    local_tp_size = moe_dense_tp_size if moe_dense_tp_size else tp_size
    local_tp_rank = tp_rank % local_tp_size
    local_dp_size = max(1, dp_size // (tp_size // local_tp_size))

    local_attn_tp_size = local_tp_size // local_dp_size
    local_attn_dp_rank = local_tp_rank // local_attn_tp_size
    local_attn_tp_rank = local_tp_rank % local_attn_tp_size

    return local_attn_tp_rank, local_attn_tp_size, local_attn_dp_rank
```
**EN:** This block defines `compute_dp_attention_local_info` and contains the main logic for this step. It mainly invokes `max`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `local_tp_size`, `local_tp_rank`, `local_dp_size`, `local_attn_tp_size`, and `local_attn_dp_rank` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `compute_dp_attention_local_info`，并承载这一阶段的核心逻辑。 它主要调用 `max`，说明该流程会编排底层辅助函数或计算内核。 像 `local_tp_size`、`local_tp_rank`、`local_dp_size`、`local_attn_tp_size` 以及 `local_attn_dp_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 274-313: Function `initialize_dp_attention` and its core logic
```python
def initialize_dp_attention(
    server_args: ServerArgs,
    model_config: ModelConfig,
):
    global _ATTN_DP_RANK, _ATTN_DP_SIZE
    global _LOCAL_ATTN_DP_SIZE, _LOCAL_ATTN_DP_RANK, _ENABLE_DP_ATTENTION_FLAG
    enable_dp_attention = server_args.enable_dp_attention
    dp_size = server_args.dp_size
    moe_dense_tp_size = server_args.moe_dense_tp_size
    attn_cp_size = server_args.attn_cp_size

    _ENABLE_DP_ATTENTION_FLAG = enable_dp_attention

    tp_rank = get_tensor_model_parallel_rank()
    tp_size = get_tensor_model_parallel_world_size()

    _, _, _ATTN_DP_RANK, _ = compute_dp_attention_world_info(
        enable_dp_attention, tp_rank, tp_size, dp_size, attn_cp_size
    )
    _, _, _LOCAL_ATTN_DP_RANK = compute_dp_attention_local_info(
        enable_dp_attention, tp_rank, tp_size, dp_size, moe_dense_tp_size
    )

    if enable_dp_attention:
        _ATTN_DP_SIZE = dp_size
        if moe_dense_tp_size is None:
            _LOCAL_ATTN_DP_SIZE = _ATTN_DP_SIZE
        else:
            _LOCAL_ATTN_DP_SIZE = max(1, dp_size // (tp_size // moe_dense_tp_size))
    else:
        _ATTN_DP_SIZE = 1
        _LOCAL_ATTN_DP_SIZE = 1

    _DpGatheredBufferWrapper.set_metadata(
        hidden_size=model_config.hidden_size,
        dtype=model_config.dtype,
        device=torch.device(server_args.device),
    )
```
**EN:** This block defines `initialize_dp_attention` and contains the main logic for this step. It mainly invokes `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`, `compute_dp_attention_world_info`, `compute_dp_attention_local_info`, and `_DpGatheredBufferWrapper.set_metadata`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `enable_dp_attention`, `dp_size`, `moe_dense_tp_size`, `attn_cp_size`, and `_ENABLE_DP_ATTENTION_FLAG` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `initialize_dp_attention`，并承载这一阶段的核心逻辑。 它主要调用 `get_tensor_model_parallel_rank`、`get_tensor_model_parallel_world_size`、`compute_dp_attention_world_info`、`compute_dp_attention_local_info` 以及 `_DpGatheredBufferWrapper.set_metadata`，说明该流程会编排底层辅助函数或计算内核。 像 `enable_dp_attention`、`dp_size`、`moe_dense_tp_size`、`attn_cp_size` 以及 `_ENABLE_DP_ATTENTION_FLAG` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 314-317: `is_dp_attention_enabled` predicate for is data parallel attention enabled
```python
def is_dp_attention_enabled() -> bool:
    return _ENABLE_DP_ATTENTION_FLAG
```
**EN:** This block defines `is_dp_attention_enabled` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_dp_attention_enabled`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 318-321: `is_allocation_symmetric` predicate for is allocation symmetric
```python
def is_allocation_symmetric() -> bool:
    return not is_dp_attention_enabled() or is_dp_max_padding()
```
**EN:** This block defines `is_allocation_symmetric` and contains the main logic for this step. It mainly invokes `is_dp_max_padding` and `is_dp_attention_enabled`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_allocation_symmetric`，并承载这一阶段的核心逻辑。 它主要调用 `is_dp_max_padding` 和 `is_dp_attention_enabled`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 322-325: `get_attention_tp_group` getter for attention tensor parallel group
```python
def get_attention_tp_group() -> GroupCoordinator:
    return get_attn_tp_group()
```
**EN:** This block defines `get_attention_tp_group` and contains the main logic for this step. It mainly invokes `get_attn_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attention_tp_group`，并承载这一阶段的核心逻辑。 它主要调用 `get_attn_tp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 326-329: `get_attention_tp_rank` getter for attention tensor parallel rank
```python
def get_attention_tp_rank() -> int:
    return get_attn_tensor_model_parallel_rank()
```
**EN:** This block defines `get_attention_tp_rank` and contains the main logic for this step. It mainly invokes `get_attn_tensor_model_parallel_rank`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attention_tp_rank`，并承载这一阶段的核心逻辑。 它主要调用 `get_attn_tensor_model_parallel_rank`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 330-333: `get_attention_tp_size` getter for attention tensor parallel size
```python
def get_attention_tp_size() -> int:
    return get_attn_tensor_model_parallel_world_size()
```
**EN:** This block defines `get_attention_tp_size` and contains the main logic for this step. It mainly invokes `get_attn_tensor_model_parallel_world_size`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attention_tp_size`，并承载这一阶段的核心逻辑。 它主要调用 `get_attn_tensor_model_parallel_world_size`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 334-337: `get_attention_cp_group` getter for attention context parallel group
```python
def get_attention_cp_group() -> GroupCoordinator:
    return get_attn_cp_group()
```
**EN:** This block defines `get_attention_cp_group` and contains the main logic for this step. It mainly invokes `get_attn_cp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attention_cp_group`，并承载这一阶段的核心逻辑。 它主要调用 `get_attn_cp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 338-341: `get_attention_cp_rank` getter for attention context parallel rank
```python
def get_attention_cp_rank() -> int:
    return get_attn_context_model_parallel_rank()
```
**EN:** This block defines `get_attention_cp_rank` and contains the main logic for this step. It mainly invokes `get_attn_context_model_parallel_rank`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attention_cp_rank`，并承载这一阶段的核心逻辑。 它主要调用 `get_attn_context_model_parallel_rank`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 342-345: `get_attention_cp_size` getter for attention context parallel size
```python
def get_attention_cp_size() -> int:
    return get_attn_context_model_parallel_world_size()
```
**EN:** This block defines `get_attention_cp_size` and contains the main logic for this step. It mainly invokes `get_attn_context_model_parallel_world_size`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attention_cp_size`，并承载这一阶段的核心逻辑。 它主要调用 `get_attn_context_model_parallel_world_size`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 346-350: `get_attention_dp_rank` getter for attention data parallel rank
```python
def get_attention_dp_rank() -> int:
    assert _ATTN_DP_RANK is not None, "dp attention not initialized!"
    return _ATTN_DP_RANK
```
**EN:** This block defines `get_attention_dp_rank` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attention_dp_rank`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 351-355: `get_attention_dp_size` getter for attention data parallel size
```python
def get_attention_dp_size() -> int:
    assert _ATTN_DP_SIZE is not None, "dp attention not initialized!"
    return _ATTN_DP_SIZE
```
**EN:** This block defines `get_attention_dp_size` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attention_dp_size`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 356-360: `get_local_attention_dp_rank` getter for local attention data parallel rank
```python
def get_local_attention_dp_rank() -> int:
    assert _LOCAL_ATTN_DP_RANK is not None, "dp attention not initialized!"
    return _LOCAL_ATTN_DP_RANK
```
**EN:** This block defines `get_local_attention_dp_rank` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_local_attention_dp_rank`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 361-365: `get_local_attention_dp_size` getter for local attention data parallel size
```python
def get_local_attention_dp_size() -> int:
    assert _LOCAL_ATTN_DP_SIZE is not None, "dp attention not initialized!"
    return _LOCAL_ATTN_DP_SIZE
```
**EN:** This block defines `get_local_attention_dp_size` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_local_attention_dp_size`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 366-386: Function `disable_dp_size` and its core logic
```python
@contextmanager
def disable_dp_size():
    """Patch the tp group temporarily until this function ends.

    This method is for draft workers of speculative decoding to run draft model
    with different tp degree from that of target model workers.

    Args:
        tp_group (GroupCoordinator): the tp group coordinator
    """
    global _ATTN_DP_SIZE
    assert _ATTN_DP_SIZE is not None, "dp attention not initialized!"

    old_dp_size = _ATTN_DP_SIZE
    _ATTN_DP_SIZE = 1
    try:
        yield
    finally:
        _ATTN_DP_SIZE = old_dp_size
```
**EN:** This block defines `disable_dp_size` and contains the main logic for this step. Decorators like `contextmanager` connect it to registration, caching, tracing, or dispatch hooks. Intermediate names such as `old_dp_size` and `_ATTN_DP_SIZE` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `disable_dp_size`，并承载这一阶段的核心逻辑。 像 `contextmanager` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 像 `old_dp_size` 和 `_ATTN_DP_SIZE` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 387-404: `get_dp_local_info` getter for data parallel local info
```python
def get_dp_local_info(forward_batch: ForwardBatch) -> Tuple[torch.Tensor, torch.Tensor]:
    # `get_dp_local_info` is only called in global DP gather and scatter. We use global DP rank here.
    dp_rank = get_attention_dp_rank()

    if forward_batch.dp_local_start_pos is None:
        cumtokens = torch.cumsum(forward_batch.global_num_tokens_gpu, dim=0)
        if dp_rank == 0:
            local_start_pos = torch.zeros_like(cumtokens[0])
        else:
            local_start_pos = cumtokens[dp_rank - 1]
        local_num_tokens = forward_batch.global_num_tokens_gpu[dp_rank]

        forward_batch.dp_local_start_pos = local_start_pos
        forward_batch.dp_local_num_tokens = local_num_tokens

    return forward_batch.dp_local_start_pos, forward_batch.dp_local_num_tokens
```
**EN:** This block defines `get_dp_local_info` and contains the main logic for this step. It mainly invokes `get_attention_dp_rank`, `torch.cumsum`, and `torch.zeros_like`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dp_rank`, `cumtokens`, `local_num_tokens`, and `local_start_pos` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_dp_local_info`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_dp_rank`、`torch.cumsum` 以及 `torch.zeros_like`，说明该流程会编排底层辅助函数或计算内核。 像 `dp_rank`、`cumtokens`、`local_num_tokens` 以及 `local_start_pos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 405-421: `get_dp_local_slice_cpu` getter for data parallel local slice CPU
```python
def get_dp_local_slice_cpu(
    forward_batch: ForwardBatch,
    can_run_graph: bool,
    cuda_graph_batch: Optional[int],
) -> Tuple[int, int]:
    # CPU (start, length) slice for DP-local data in a rank-padded buffer.
    # Returns Python ints (no D2H sync) and handles the cuda-graph-padded layout.
    global_num_tokens = forward_batch.global_num_tokens_cpu
    dp_rank = get_attention_dp_rank()
    local_num_tokens = global_num_tokens[dp_rank]
    if can_run_graph:
        local_start_pos = dp_rank * cuda_graph_batch
    else:
        local_start_pos = sum(global_num_tokens[:dp_rank])
    return local_start_pos, local_num_tokens
```
**EN:** This block defines `get_dp_local_slice_cpu` and contains the main logic for this step. It mainly invokes `get_attention_dp_rank` and `sum`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `global_num_tokens`, `dp_rank`, `local_num_tokens`, and `local_start_pos` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_dp_local_slice_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_dp_rank` 和 `sum`，说明该流程会编排底层辅助函数或计算内核。 像 `global_num_tokens`、`dp_rank`、`local_num_tokens` 以及 `local_start_pos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 422-447: Function `memcpy_triton_kernel` and its core logic
```python
@triton.jit
def memcpy_triton_kernel(
    dst_ptr,
    src_ptr,
    offset_ptr,
    sz_ptr,
    offset_src: tl.constexpr,
    chunk_size,  # multiplied for offset and sz
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0).to(tl.int64)
    offset = tl.load(offset_ptr).to(tl.int64) * chunk_size
    sz = tl.load(sz_ptr).to(tl.int64) * chunk_size

    start_index = pid * BLOCK_SIZE
    offs = tl.arange(0, BLOCK_SIZE)
    mask = start_index + offs < sz

    if offset_src:
        data = tl.load(src_ptr + offset + start_index + offs, mask=mask)
        tl.store(dst_ptr + start_index + offs, data, mask=mask)
    else:
        data = tl.load(src_ptr + start_index + offs, mask=mask)
        tl.store(dst_ptr + offset + start_index + offs, data, mask=mask)
```
**EN:** This block defines `memcpy_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id.to`, `tl.arange`, `tl.load.to`, `tl.load`, and `tl.store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `offset`, `sz`, `start_index`, and `offs` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `memcpy_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id.to`、`tl.arange`、`tl.load.to`、`tl.load` 以及 `tl.store`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`offset`、`sz`、`start_index` 以及 `offs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 448-451: Function `prod` and its core logic
```python
def prod(x):
    return functools.reduce(lambda a, b: a * b, x, 1)
```
**EN:** This block defines `prod` and contains the main logic for this step. It mainly invokes `functools.reduce`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `prod`，并承载这一阶段的核心逻辑。 它主要调用 `functools.reduce`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 452-462: Function `memcpy_triton` and its core logic
```python
def memcpy_triton(dst, src, dim, offset, sz, offset_src):
    max_size = min(src.numel(), dst.numel())
    assert dim == 0, "dim != 0 unsupported"
    assert src.shape[1:] == dst.shape[1:], "src and dst must have same shape"
    chunk_size = prod(src.shape[1:])
    BLOCK_SIZE = 8192
    grid = (triton.cdiv(max_size, BLOCK_SIZE),)

    memcpy_triton_kernel[grid](dst, src, offset, sz, offset_src, chunk_size, BLOCK_SIZE)
```
**EN:** This block defines `memcpy_triton` and contains the main logic for this step. It mainly invokes `min`, `prod`, `memcpy_triton_kernel`, `src.numel`, and `dst.numel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `max_size`, `chunk_size`, `BLOCK_SIZE`, and `grid` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `memcpy_triton`，并承载这一阶段的核心逻辑。 它主要调用 `min`、`prod`、`memcpy_triton_kernel`、`src.numel` 以及 `dst.numel`，说明该流程会编排底层辅助函数或计算内核。 像 `max_size`、`chunk_size`、`BLOCK_SIZE` 以及 `grid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 463-497: Internal helper `_dp_gather_via_all_reduce`
```python
def _dp_gather_via_all_reduce(
    global_tokens: torch.Tensor,
    local_tokens: torch.Tensor,
    forward_batch: ForwardBatch,
    is_partial: bool,
):
    local_start_pos, local_num_tokens = get_dp_local_info(forward_batch)

    global_tokens.fill_(0)
    assert local_tokens.is_contiguous()
    assert global_tokens.is_contiguous()

    if local_tokens.shape[0] > 0 and (is_partial or get_attention_tp_rank() == 0):
        assert (
            local_tokens.untyped_storage() is not global_tokens.untyped_storage()
        ), "aliasing between global_tokens and local_tokens not allowed"

        memcpy_triton(
            global_tokens, local_tokens, 0, local_start_pos, local_num_tokens, False
        )

    # Input IDs are in int 32. We should use inplace_all_reduce for local case because of custom all reduce.
    NUM_GPUS_PER_NODE = 8
    if (
        not local_tokens.dtype.is_floating_point
        and get_tensor_model_parallel_world_size() <= NUM_GPUS_PER_NODE
    ):
        from sglang.srt.distributed.parallel_state import inplace_all_reduce

        inplace_all_reduce(global_tokens, group_name=get_tp_group().unique_name)

    else:
        global_tokens[:] = tensor_model_parallel_all_reduce(global_tokens)
```
**EN:** This block defines `_dp_gather_via_all_reduce` and contains the main logic for this step. It mainly invokes `get_dp_local_info`, `global_tokens.fill_`, `local_tokens.is_contiguous`, `global_tokens.is_contiguous`, and `memcpy_triton`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `local_start_pos`, `local_num_tokens`, and `NUM_GPUS_PER_NODE` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_dp_gather_via_all_reduce`，并承载这一阶段的核心逻辑。 它主要调用 `get_dp_local_info`、`global_tokens.fill_`、`local_tokens.is_contiguous`、`global_tokens.is_contiguous` 以及 `memcpy_triton`，说明该流程会编排底层辅助函数或计算内核。 像 `local_start_pos`、`local_num_tokens` 以及 `NUM_GPUS_PER_NODE` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 498-517: Internal helper `_dp_gather_via_all_gather`
```python
def _dp_gather_via_all_gather(
    global_tokens: torch.Tensor,
    local_tokens: torch.Tensor,
    forward_batch: ForwardBatch,
    is_partial: bool,
):
    if get_attention_tp_size() == 1:
        get_tp_group().all_gather_into_tensor(global_tokens, local_tokens)
        return

    if not is_partial:
        if get_attention_tp_rank() != 0:
            local_tokens.fill_(0)
    scattered_local_tokens = local_tokens.tensor_split(get_attention_tp_size())[
        get_attention_tp_rank()
    ]
    get_attention_tp_group().reduce_scatter_tensor(scattered_local_tokens, local_tokens)
    get_tp_group().all_gather_into_tensor(global_tokens, scattered_local_tokens)
```
**EN:** This block defines `_dp_gather_via_all_gather` and contains the main logic for this step. It mainly invokes `get_attention_tp_group.reduce_scatter_tensor`, `get_tp_group.all_gather_into_tensor`, `get_attention_tp_size`, `local_tokens.tensor_split`, and `get_attention_tp_rank`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `scattered_local_tokens` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_dp_gather_via_all_gather`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_tp_group.reduce_scatter_tensor`、`get_tp_group.all_gather_into_tensor`、`get_attention_tp_size`、`local_tokens.tensor_split` 以及 `get_attention_tp_rank`，说明该流程会编排底层辅助函数或计算内核。 像 `scattered_local_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 518-533: Internal helper `_dp_gather`
```python
def _dp_gather(
    global_tokens: torch.Tensor,
    local_tokens: torch.Tensor,
    forward_batch: ForwardBatch,
    is_partial: bool,
):
    if forward_batch.dp_padding_mode.is_max_len():
        _dp_gather_via_all_gather(
            global_tokens, local_tokens, forward_batch, is_partial
        )
    else:
        _dp_gather_via_all_reduce(
            global_tokens, local_tokens, forward_batch, is_partial
        )
```
**EN:** This block defines `_dp_gather` and contains the main logic for this step. It mainly invokes `forward_batch.dp_padding_mode.is_max_len`, `_dp_gather_via_all_gather`, and `_dp_gather_via_all_reduce`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_dp_gather`，并承载这一阶段的核心逻辑。 它主要调用 `forward_batch.dp_padding_mode.is_max_len`、`_dp_gather_via_all_gather` 以及 `_dp_gather_via_all_reduce`，说明该流程会编排底层辅助函数或计算内核。

### Lines 534-541: Function `dp_gather_partial` and its core logic
```python
def dp_gather_partial(
    global_tokens: torch.Tensor,
    local_tokens: torch.Tensor,
    forward_batch: ForwardBatch,
):
    _dp_gather(global_tokens, local_tokens, forward_batch, is_partial=True)
```
**EN:** This block defines `dp_gather_partial` and contains the main logic for this step. It mainly invokes `_dp_gather`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `dp_gather_partial`，并承载这一阶段的核心逻辑。 它主要调用 `_dp_gather`，说明该流程会编排底层辅助函数或计算内核。

### Lines 542-549: Function `dp_gather_replicate` and its core logic
```python
def dp_gather_replicate(
    global_tokens: torch.Tensor,
    local_tokens: torch.Tensor,
    forward_batch: ForwardBatch,
):
    _dp_gather(global_tokens, local_tokens, forward_batch, is_partial=False)
```
**EN:** This block defines `dp_gather_replicate` and contains the main logic for this step. It mainly invokes `_dp_gather`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `dp_gather_replicate`，并承载这一阶段的核心逻辑。 它主要调用 `_dp_gather`，说明该流程会编排底层辅助函数或计算内核。

### Lines 550-571: Function `dp_scatter` and its core logic
```python
def dp_scatter(
    local_tokens: torch.Tensor,  # output
    global_tokens: torch.Tensor,  # input
    forward_batch: ForwardBatch,
):
    # local_num_tokens is not necessarily the same as local_tokens.shape[0],
    # since local_tokens may be padded for cuda graph
    local_start_pos, local_num_tokens = get_dp_local_info(forward_batch)

    local_tokens.fill_(0)
    assert local_tokens.is_contiguous()
    assert global_tokens.is_contiguous()
    if local_tokens.shape[0] > 0:
        assert (
            local_tokens.untyped_storage() is not global_tokens.untyped_storage()
        ), "aliasing between local_tokens and global_tokens not allowed"

        memcpy_triton(
            local_tokens, global_tokens, 0, local_start_pos, local_num_tokens, True
        )
```
**EN:** This block defines `dp_scatter` and contains the main logic for this step. It mainly invokes `get_dp_local_info`, `local_tokens.fill_`, `local_tokens.is_contiguous`, `global_tokens.is_contiguous`, and `memcpy_triton`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `local_start_pos` and `local_num_tokens` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `dp_scatter`，并承载这一阶段的核心逻辑。 它主要调用 `get_dp_local_info`、`local_tokens.fill_`、`local_tokens.is_contiguous`、`global_tokens.is_contiguous` 以及 `memcpy_triton`，说明该流程会编排底层辅助函数或计算内核。 像 `local_start_pos` 和 `local_num_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 572-582: Function `dp_reduce_scatter_tensor` and its core logic
```python
def dp_reduce_scatter_tensor(output: torch.Tensor, input: torch.Tensor):
    if get_tensor_model_parallel_world_size() == get_attention_dp_size():
        get_tp_group().reduce_scatter_tensor(output, input)
    else:
        scattered_local_tokens = input.tensor_split(
            get_tensor_model_parallel_world_size()
        )[get_tensor_model_parallel_rank()]
        get_tp_group().reduce_scatter_tensor(scattered_local_tokens, input)
        get_attention_tp_group().all_gather_into_tensor(output, scattered_local_tokens)
```
**EN:** This block defines `dp_reduce_scatter_tensor` and contains the main logic for this step. It mainly invokes `get_tensor_model_parallel_world_size`, `get_attention_dp_size`, `get_tp_group.reduce_scatter_tensor`, `get_attention_tp_group.all_gather_into_tensor`, and `input.tensor_split`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `scattered_local_tokens` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `dp_reduce_scatter_tensor`，并承载这一阶段的核心逻辑。 它主要调用 `get_tensor_model_parallel_world_size`、`get_attention_dp_size`、`get_tp_group.reduce_scatter_tensor`、`get_attention_tp_group.all_gather_into_tensor` 以及 `input.tensor_split`，说明该流程会编排底层辅助函数或计算内核。 像 `scattered_local_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 583-586: Function `attn_tp_reduce_scatter_tensor` and its core logic
```python
def attn_tp_reduce_scatter_tensor(output: torch.Tensor, input: torch.Tensor):
    return get_attention_tp_group().reduce_scatter_tensor(output, input)
```
**EN:** This block defines `attn_tp_reduce_scatter_tensor` and contains the main logic for this step. It mainly invokes `get_attention_tp_group.reduce_scatter_tensor` and `get_attention_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `attn_tp_reduce_scatter_tensor`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_tp_group.reduce_scatter_tensor` 和 `get_attention_tp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 587-590: Function `attn_cp_reduce_scatter_tensor` and its core logic
```python
def attn_cp_reduce_scatter_tensor(output: torch.Tensor, input: torch.Tensor):
    return get_attention_cp_group().reduce_scatter_tensor(output, input)
```
**EN:** This block defines `attn_cp_reduce_scatter_tensor` and contains the main logic for this step. It mainly invokes `get_attention_cp_group.reduce_scatter_tensor` and `get_attention_cp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `attn_cp_reduce_scatter_tensor`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_cp_group.reduce_scatter_tensor` 和 `get_attention_cp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 591-594: Function `attn_tp_all_reduce` and its core logic
```python
def attn_tp_all_reduce(input: torch.Tensor):
    return get_attention_tp_group().all_reduce(input)
```
**EN:** This block defines `attn_tp_all_reduce` and contains the main logic for this step. It mainly invokes `get_attention_tp_group.all_reduce` and `get_attention_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `attn_tp_all_reduce`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_tp_group.all_reduce` 和 `get_attention_tp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 595-598: Function `attn_tp_all_gather_into_tensor` and its core logic
```python
def attn_tp_all_gather_into_tensor(output: torch.Tensor, input: torch.Tensor):
    return get_attention_tp_group().all_gather_into_tensor(output, input)
```
**EN:** This block defines `attn_tp_all_gather_into_tensor` and contains the main logic for this step. It mainly invokes `get_attention_tp_group.all_gather_into_tensor` and `get_attention_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `attn_tp_all_gather_into_tensor`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_tp_group.all_gather_into_tensor` 和 `get_attention_tp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 599-602: Function `attn_cp_all_gather_into_tensor` and its core logic
```python
def attn_cp_all_gather_into_tensor(output: torch.Tensor, input: torch.Tensor):
    return get_attention_cp_group().all_gather_into_tensor(output, input)
```
**EN:** This block defines `attn_cp_all_gather_into_tensor` and contains the main logic for this step. It mainly invokes `get_attention_cp_group.all_gather_into_tensor` and `get_attention_cp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `attn_cp_all_gather_into_tensor`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_cp_group.all_gather_into_tensor` 和 `get_attention_cp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 603-607: `get_moe_cp_group` getter for Mixture-of-Experts context parallel group
```python
def get_moe_cp_group() -> GroupCoordinator:
    """Returns the MOE_DP group, which includes CP partners when attn_cp_size > moe_dp_size."""
    return _get_moe_dp_group()
```
**EN:** This block defines `get_moe_cp_group` and contains the main logic for this step. It mainly invokes `_get_moe_dp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_cp_group`，并承载这一阶段的核心逻辑。 它主要调用 `_get_moe_dp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 608-611: `get_moe_cp_rank` getter for Mixture-of-Experts context parallel rank
```python
def get_moe_cp_rank() -> int:
    return _get_moe_dp_group().rank_in_group
```
**EN:** This block defines `get_moe_cp_rank` and contains the main logic for this step. It mainly invokes `_get_moe_dp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_cp_rank`，并承载这一阶段的核心逻辑。 它主要调用 `_get_moe_dp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 612-615: `get_moe_cp_size` getter for Mixture-of-Experts context parallel size
```python
def get_moe_cp_size() -> int:
    return _get_moe_dp_group().world_size
```
**EN:** This block defines `get_moe_cp_size` and contains the main logic for this step. It mainly invokes `_get_moe_dp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_cp_size`，并承载这一阶段的核心逻辑。 它主要调用 `_get_moe_dp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 616-623: `is_enable_moe_cp_allgather` predicate for is enable Mixture-of-Experts context parallel all-gather
```python
def is_enable_moe_cp_allgather() -> bool:
    """True when moe_dp_size < attn_cp_size, requiring allgather across CP ranks before MoE."""
    from sglang.srt.server_args import get_global_server_args

    sa = get_global_server_args()
    return sa.attn_cp_size > sa.moe_dp_size
```
**EN:** This block defines `is_enable_moe_cp_allgather` and contains the main logic for this step. It mainly invokes `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `sa` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_enable_moe_cp_allgather`，并承载这一阶段的核心逻辑。 它主要调用 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 像 `sa` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 624-627: Function `moe_cp_all_gather_into_tensor` and its core logic
```python
def moe_cp_all_gather_into_tensor(output: torch.Tensor, input: torch.Tensor):
    return _get_moe_dp_group().all_gather_into_tensor(output, input)
```
**EN:** This block defines `moe_cp_all_gather_into_tensor` and contains the main logic for this step. It mainly invokes `_get_moe_dp_group.all_gather_into_tensor` and `_get_moe_dp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `moe_cp_all_gather_into_tensor`，并承载这一阶段的核心逻辑。 它主要调用 `_get_moe_dp_group.all_gather_into_tensor` 和 `_get_moe_dp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 628-629: Function `attn_tp_all_gather` and its core logic
```python
def attn_tp_all_gather(output_list: List[torch.Tensor], input: torch.Tensor):
    return get_attention_tp_group().all_gather(input, output_tensor_list=output_list)
```
**EN:** This block defines `attn_tp_all_gather` and contains the main logic for this step. It mainly invokes `get_attention_tp_group.all_gather` and `get_attention_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `attn_tp_all_gather`，并承载这一阶段的核心逻辑。 它主要调用 `get_attention_tp_group.all_gather` 和 `get_attention_tp_group`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `DpPaddingMode`, `_DpGatheredBufferWrapper`, `set_dp_buffer_len`, `get_global_dp_buffer`, and `get_local_dp_buffer`. / **主要符号**：核心入口包括 `DpPaddingMode`、`_DpGatheredBufferWrapper`、`set_dp_buffer_len`、`get_global_dp_buffer` 以及 `get_local_dp_buffer`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Acceleration stack**: The module depends on external acceleration libraries and wraps them behind Python entry points. / **加速栈**：该模块依赖外部加速库，并通过 Python 入口对它们进行封装。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `functools`, `logging`, `contextlib.contextmanager`, `enum.IntEnum`, `enum.auto`, `typing.TYPE_CHECKING`, `typing.List`, `typing.Optional`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`functools`、`logging`、`contextlib.contextmanager`、`enum.IntEnum`、`enum.auto`、`typing.TYPE_CHECKING`、`typing.List`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
- **Internal SGLang modules**: `sglang.srt.distributed.GroupCoordinator`, `sglang.srt.distributed.get_attn_context_model_parallel_rank`, `sglang.srt.distributed.get_attn_context_model_parallel_world_size`, `sglang.srt.distributed.get_attn_cp_group`, `sglang.srt.distributed.get_attn_tensor_model_parallel_rank`, `sglang.srt.distributed.get_attn_tensor_model_parallel_world_size`, `sglang.srt.distributed.get_attn_tp_group`, `sglang.srt.distributed.get_moe_dp_group`, `sglang.srt.distributed.get_tensor_model_parallel_rank`, `sglang.srt.distributed.get_tensor_model_parallel_world_size`, `sglang.srt.distributed.get_tp_group`, and `sglang.srt.distributed.tensor_model_parallel_all_reduce` / **SGLang 内部模块**：`sglang.srt.distributed.GroupCoordinator`、`sglang.srt.distributed.get_attn_context_model_parallel_rank`、`sglang.srt.distributed.get_attn_context_model_parallel_world_size`、`sglang.srt.distributed.get_attn_cp_group`、`sglang.srt.distributed.get_attn_tensor_model_parallel_rank`、`sglang.srt.distributed.get_attn_tensor_model_parallel_world_size`、`sglang.srt.distributed.get_attn_tp_group`、`sglang.srt.distributed.get_moe_dp_group`、`sglang.srt.distributed.get_tensor_model_parallel_rank`、`sglang.srt.distributed.get_tensor_model_parallel_world_size`、`sglang.srt.distributed.get_tp_group` 以及 `sglang.srt.distributed.tensor_model_parallel_all_reduce`
