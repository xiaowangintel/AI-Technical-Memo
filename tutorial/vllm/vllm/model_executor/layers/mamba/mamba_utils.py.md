# mamba_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/mamba/mamba_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for Mamba/state-space layers and kernels. / 汇总Mamba/状态空间模型层与内核的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-19)
```python
import functools
from collections.abc import Callable
from dataclasses import dataclass
from typing import Literal, TypeAlias

import torch

import vllm.envs as envs
from vllm.config.cache import MambaDType
from vllm.config.model import ModelDType
from vllm.distributed import divide
from vllm.logger import init_logger
from vllm.utils.torch_utils import (
    STR_DTYPE_TO_TORCH_DTYPE,
    get_kv_cache_torch_dtype,
)
```
**EN:** This opening block pulls in external dependencies such as `functools`, `collections`, `dataclasses`, `typing`, `torch` and internal modules such as `vllm.envs`, `vllm.config.cache`, `vllm.config.model`, `vllm.distributed`, `vllm.logger`, `vllm.utils.torch_utils`. That import mix shows the file is part of the Mamba/state-space layers and kernels stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `functools`, `collections`, `dataclasses`, `typing`, `torch`）以及内部模块（如 `vllm.envs`, `vllm.config.cache`, `vllm.config.model`, `vllm.distributed`, `vllm.logger`, `vllm.utils.torch_utils`）。这些导入关系表明该文件属于Mamba/状态空间模型层与内核栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 21-287)
```python
logger = init_logger(__name__)

ConvStateLayoutType = Literal["SD", "DS"]


@functools.lru_cache
def get_conv_state_layout() -> ConvStateLayoutType:
    """Return the SSM conv state layout.

    SD = (state_len, dim) — dim is the innermost contiguous dimension.
    DS = (dim, state_len) — TP-sharded dim is on dim-1 (like HND for KV
         cache), consistent with SSM temporal state layout.
    """
    layout: ConvStateLayoutType | None = envs.VLLM_SSM_CONV_STATE_LAYOUT
    if layout is not None:
        logger.info_once(
            "VLLM_SSM_CONV_STATE_LAYOUT env detected. "
            "Setting SSM conv state layout to %s.",
            layout,
        )
        return layout

    return "SD"


def is_conv_state_dim_first() -> bool:
    """True when the conv state is stored as (dim, state_len) per block."""
    return get_conv_state_layout() == "DS"


class MambaStateDtypeCalculator:
    @classmethod
    def linear_attention_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
    ) -> tuple[torch.dtype, ...]:
        state_dtype = get_kv_cache_torch_dtype(mamba_cache_dtype, model_dtype)
        return (state_dtype,)

    @classmethod
    def mamba1_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
# ... truncated for analysis ...
    Attributes:
        start_addr (int): Starting address for the memory copy operation.
        num_elements (int): Number of elements to copy from the starting address.
    """

    start_addr: int
    num_elements: int


MambaStateCopyFunc: TypeAlias = Callable[
    [torch.Tensor, list[int], int, int], MambaCopySpec
]
```
**EN:** This block defines module-level metadata or constants such as `logger`, `ConvStateLayoutType`, `MambaStateCopyFunc`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the Mamba/state-space layers and kernels pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `ConvStateLayoutType`, `MambaStateCopyFunc`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在Mamba/状态空间模型层与内核流程中复用。

### Function `get_conv_state_layout` (lines 27-43)
```python
def get_conv_state_layout() -> ConvStateLayoutType:
    """Return the SSM conv state layout.

    SD = (state_len, dim) — dim is the innermost contiguous dimension.
    DS = (dim, state_len) — TP-sharded dim is on dim-1 (like HND for KV
         cache), consistent with SSM temporal state layout.
    """
    layout: ConvStateLayoutType | None = envs.VLLM_SSM_CONV_STATE_LAYOUT
    if layout is not None:
        logger.info_once(
            "VLLM_SSM_CONV_STATE_LAYOUT env detected. "
            "Setting SSM conv state layout to %s.",
            layout,
        )
        return layout

    return "SD"
```
**EN:** Defines function `get_conv_state_layout` with signature `get_conv_state_layout() -> ConvStateLayoutType`. It mainly works with object context only; returns a derived property or capability check. The body uses branching. Key calls include `logger.info_once`.
**CN:** 定义函数 `get_conv_state_layout`，其签名为 `get_conv_state_layout() -> ConvStateLayoutType`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含分支判断。关键调用包括 `logger.info_once`。

### Function `is_conv_state_dim_first` (lines 46-48)
```python
def is_conv_state_dim_first() -> bool:
    """True when the conv state is stored as (dim, state_len) per block."""
    return get_conv_state_layout() == "DS"
```
**EN:** Defines function `is_conv_state_dim_first` with signature `is_conv_state_dim_first() -> bool`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `get_conv_state_layout`.
**CN:** 定义函数 `is_conv_state_dim_first`，其签名为 `is_conv_state_dim_first() -> bool`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `get_conv_state_layout`。

### Function `get_conv_copy_spec` (lines 299-328)
```python
def get_conv_copy_spec(
    state: torch.Tensor,
    block_ids: list[int],
    cur_block_idx: int,
    num_accepted_tokens: int,
) -> MambaCopySpec:
    """Return a MambaCopySpec for copying a convolutional state slice.

    Works for both SD layout ``(num_blocks, state_len, dim)`` and
    DS layout ``(num_blocks, dim, state_len)``.
    """
    src_block_id = block_ids[cur_block_idx]
    offset = num_accepted_tokens - 1
    if is_conv_state_dim_first():
        # DS layout: (num_blocks, dim, state_len) — state_len is last.
        if offset > 0:
            # Slicing along the last dim yields a non-contiguous view
            # because features (dim) are strided by state_len.
            raise NotImplementedError(
                "DS conv state layout does not yet support speculative "
                "decoding with mamba_cache_mode='align' "
                "(num_accepted_tokens > 1)."
            )
        src_state = state[src_block_id]
    else:
        # SD layout: (num_blocks, state_len, dim) — dim contiguous.
        src_state = state[src_block_id, offset:]
    return MambaCopySpec(
        start_addr=src_state.data_ptr(), num_elements=src_state.numel()
    )
```
**EN:** Defines function `get_conv_copy_spec` with signature `get_conv_copy_spec(state: torch.Tensor, block_ids: list[int], cur_block_idx: int, num_accepted_tokens: int) -> MambaCopySpec`. It mainly works with `state`, `block_ids`, `cur_block_idx`, `num_accepted_tokens`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `is_conv_state_dim_first`, `MambaCopySpec`, `NotImplementedError`, `src_state.data_ptr`, `src_state.numel`.
**CN:** 定义函数 `get_conv_copy_spec`，其签名为 `get_conv_copy_spec(state: torch.Tensor, block_ids: list[int], cur_block_idx: int, num_accepted_tokens: int) -> MambaCopySpec`。它主要围绕 `state`, `block_ids`, `cur_block_idx`, `num_accepted_tokens` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `is_conv_state_dim_first`, `MambaCopySpec`, `NotImplementedError`, `src_state.data_ptr`, `src_state.numel`。

### Function `get_temporal_copy_spec` (lines 331-342)
```python
def get_temporal_copy_spec(
    state: torch.Tensor,
    block_ids: list[int],
    cur_block_idx: int,
    num_accepted_tokens: int,
) -> MambaCopySpec:
    """Return a MambaCopySpec for copying a temporal state slice."""
    src_block_id = block_ids[cur_block_idx + num_accepted_tokens - 1]
    src_state = state[src_block_id]
    return MambaCopySpec(
        start_addr=src_state.data_ptr(), num_elements=src_state.numel()
    )
```
**EN:** Defines function `get_temporal_copy_spec` with signature `get_temporal_copy_spec(state: torch.Tensor, block_ids: list[int], cur_block_idx: int, num_accepted_tokens: int) -> MambaCopySpec`. It mainly works with `state`, `block_ids`, `cur_block_idx`, `num_accepted_tokens`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `MambaCopySpec`, `src_state.data_ptr`, `src_state.numel`.
**CN:** 定义函数 `get_temporal_copy_spec`，其签名为 `get_temporal_copy_spec(state: torch.Tensor, block_ids: list[int], cur_block_idx: int, num_accepted_tokens: int) -> MambaCopySpec`。它主要围绕 `state`, `block_ids`, `cur_block_idx`, `num_accepted_tokens` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `MambaCopySpec`, `src_state.data_ptr`, `src_state.numel`。

### Class `MambaStateDtypeCalculator` overview (lines 51-125)
```python
class MambaStateDtypeCalculator:
    @classmethod
    def linear_attention_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
    ) -> tuple[torch.dtype, ...]:
        state_dtype = get_kv_cache_torch_dtype(mamba_cache_dtype, model_dtype)
        return (state_dtype,)

    @classmethod
    def mamba1_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
        mamba_ssm_cache_dtype: MambaDType,
    ) -> tuple[torch.dtype, ...]:
        return cls._mamba_state_dtype(
            model_dtype, mamba_cache_dtype, mamba_ssm_cache_dtype
        )

    @classmethod
    def mamba2_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
```
**EN:** Defines class `MambaStateDtypeCalculator` with base classes no explicit base class and decorators none. It acts as a Mamba/SSM module building block and exposes 7 direct methods, with notable entries `linear_attention_state_dtype`, `mamba1_state_dtype`, `mamba2_state_dtype`, `_mamba_state_dtype`, `short_conv_state_dtype`, `gated_delta_net_state_dtype`.
**CN:** 定义类 `MambaStateDtypeCalculator`，其基类为 无显式基类，装饰器为 无。它在整体实现中充当Mamba/状态空间模块构件，并直接暴露 7 个方法，较重要的包括 `linear_attention_state_dtype`, `mamba1_state_dtype`, `mamba2_state_dtype`, `_mamba_state_dtype`, `short_conv_state_dtype`, `gated_delta_net_state_dtype`。

### Method `MambaStateDtypeCalculator.linear_attention_state_dtype` (lines 53-59)
```python
    def linear_attention_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
    ) -> tuple[torch.dtype, ...]:
        state_dtype = get_kv_cache_torch_dtype(mamba_cache_dtype, model_dtype)
        return (state_dtype,)
```
**EN:** Defines function `MambaStateDtypeCalculator.linear_attention_state_dtype` with signature `linear_attention_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`. It mainly works with `model_dtype`, `mamba_cache_dtype`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `get_kv_cache_torch_dtype`.
**CN:** 定义函数 `MambaStateDtypeCalculator.linear_attention_state_dtype`，其签名为 `linear_attention_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`。它主要围绕 `model_dtype`, `mamba_cache_dtype` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `get_kv_cache_torch_dtype`。

### Method `MambaStateDtypeCalculator.mamba1_state_dtype` (lines 62-70)
```python
    def mamba1_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
        mamba_ssm_cache_dtype: MambaDType,
    ) -> tuple[torch.dtype, ...]:
        return cls._mamba_state_dtype(
            model_dtype, mamba_cache_dtype, mamba_ssm_cache_dtype
        )
```
**EN:** Defines function `MambaStateDtypeCalculator.mamba1_state_dtype` with signature `mamba1_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType, mamba_ssm_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`. It mainly works with `model_dtype`, `mamba_cache_dtype`, `mamba_ssm_cache_dtype`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `cls._mamba_state_dtype`.
**CN:** 定义函数 `MambaStateDtypeCalculator.mamba1_state_dtype`，其签名为 `mamba1_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType, mamba_ssm_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`。它主要围绕 `model_dtype`, `mamba_cache_dtype`, `mamba_ssm_cache_dtype` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls._mamba_state_dtype`。

### Method `MambaStateDtypeCalculator.mamba2_state_dtype` (lines 73-81)
```python
    def mamba2_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
        mamba_ssm_cache_dtype: MambaDType,
    ) -> tuple[torch.dtype, ...]:
        return cls._mamba_state_dtype(
            model_dtype, mamba_cache_dtype, mamba_ssm_cache_dtype
        )
```
**EN:** Defines function `MambaStateDtypeCalculator.mamba2_state_dtype` with signature `mamba2_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType, mamba_ssm_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`. It mainly works with `model_dtype`, `mamba_cache_dtype`, `mamba_ssm_cache_dtype`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `cls._mamba_state_dtype`.
**CN:** 定义函数 `MambaStateDtypeCalculator.mamba2_state_dtype`，其签名为 `mamba2_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType, mamba_ssm_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`。它主要围绕 `model_dtype`, `mamba_cache_dtype`, `mamba_ssm_cache_dtype` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls._mamba_state_dtype`。

### Method `MambaStateDtypeCalculator._mamba_state_dtype` (lines 84-96)
```python
    def _mamba_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
        mamba_ssm_cache_dtype: MambaDType,
    ) -> tuple[torch.dtype, ...]:
        conv_state_dtype = get_kv_cache_torch_dtype(mamba_cache_dtype, model_dtype)
        if mamba_ssm_cache_dtype == "auto":
            temporal_state_dtype = conv_state_dtype
        else:
            temporal_state_dtype = STR_DTYPE_TO_TORCH_DTYPE[mamba_ssm_cache_dtype]

        return (conv_state_dtype, temporal_state_dtype)
```
**EN:** Defines function `MambaStateDtypeCalculator._mamba_state_dtype` with signature `_mamba_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType, mamba_ssm_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`. It mainly works with `model_dtype`, `mamba_cache_dtype`, `mamba_ssm_cache_dtype`; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include `get_kv_cache_torch_dtype`.
**CN:** 定义函数 `MambaStateDtypeCalculator._mamba_state_dtype`，其签名为 `_mamba_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType, mamba_ssm_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`。它主要围绕 `model_dtype`, `mamba_cache_dtype`, `mamba_ssm_cache_dtype` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 `get_kv_cache_torch_dtype`。

### Method `MambaStateDtypeCalculator.short_conv_state_dtype` (lines 99-105)
```python
    def short_conv_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
    ) -> tuple[torch.dtype, ...]:
        conv_state_dtype = get_kv_cache_torch_dtype(mamba_cache_dtype, model_dtype)
        return (conv_state_dtype,)
```
**EN:** Defines function `MambaStateDtypeCalculator.short_conv_state_dtype` with signature `short_conv_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`. It mainly works with `model_dtype`, `mamba_cache_dtype`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `get_kv_cache_torch_dtype`.
**CN:** 定义函数 `MambaStateDtypeCalculator.short_conv_state_dtype`，其签名为 `short_conv_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType) -> tuple[torch.dtype, ...]`。它主要围绕 `model_dtype`, `mamba_cache_dtype` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `get_kv_cache_torch_dtype`。

### Method `MambaStateDtypeCalculator.gated_delta_net_state_dtype` (lines 108-116)
```python
    def gated_delta_net_state_dtype(
        cls,
        model_dtype: ModelDType | torch.dtype,
        mamba_cache_dtype: MambaDType,
        mamba_ssm_cache_dtype: MambaDType = "auto",
    ) -> tuple[torch.dtype, torch.dtype]:
        return cls._mamba_state_dtype(
            model_dtype, mamba_cache_dtype, mamba_ssm_cache_dtype
        )
```
**EN:** Defines function `MambaStateDtypeCalculator.gated_delta_net_state_dtype` with signature `gated_delta_net_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType, mamba_ssm_cache_dtype: MambaDType='auto') -> tuple[torch.dtype, torch.dtype]`. It mainly works with `model_dtype`, `mamba_cache_dtype`, `mamba_ssm_cache_dtype`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `cls._mamba_state_dtype`.
**CN:** 定义函数 `MambaStateDtypeCalculator.gated_delta_net_state_dtype`，其签名为 `gated_delta_net_state_dtype(cls, model_dtype: ModelDType | torch.dtype, mamba_cache_dtype: MambaDType, mamba_ssm_cache_dtype: MambaDType='auto') -> tuple[torch.dtype, torch.dtype]`。它主要围绕 `model_dtype`, `mamba_cache_dtype`, `mamba_ssm_cache_dtype` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls._mamba_state_dtype`。

### Class `MambaStateShapeCalculator` overview (lines 128-267)
```python
class MambaStateShapeCalculator:
    @classmethod
    def linear_attention_state_shape(
        cls,
        num_heads: int,
        tp_size: int,
        head_dim: int,
    ) -> tuple[tuple[int, int, int], ...]:
        state_shape = (num_heads // tp_size, head_dim, head_dim)
        return (state_shape,)

    @staticmethod
    def _orient_conv_shape(dim: int, state_len: int) -> tuple[int, int]:
        """Return (dim, state_len) for DS layout, (state_len, dim) for SD."""
        if is_conv_state_dim_first():
            return (dim, state_len)
        return (state_len, dim)

    @classmethod
    def mamba1_state_shape(
        cls,
        tp_world_size: int,
        intermediate_size: int,
        state_size: int,
        conv_kernel: int,
```
**EN:** Defines class `MambaStateShapeCalculator` with base classes no explicit base class and decorators none. It acts as a Mamba/SSM module building block and exposes 8 direct methods, with notable entries `linear_attention_state_shape`, `_orient_conv_shape`, `mamba1_state_shape`, `mamba2_state_shape`, `short_conv_state_shape`, `extra_groups_for_head_shards`.
**CN:** 定义类 `MambaStateShapeCalculator`，其基类为 无显式基类，装饰器为 无。它在整体实现中充当Mamba/状态空间模块构件，并直接暴露 8 个方法，较重要的包括 `linear_attention_state_shape`, `_orient_conv_shape`, `mamba1_state_shape`, `mamba2_state_shape`, `short_conv_state_shape`, `extra_groups_for_head_shards`。

### Method `MambaStateShapeCalculator.mamba1_state_shape` (lines 147-159)
```python
    def mamba1_state_shape(
        cls,
        tp_world_size: int,
        intermediate_size: int,
        state_size: int,
        conv_kernel: int,
    ) -> tuple[tuple[int, int], tuple[int, int]]:
        conv_dim = divide(intermediate_size, tp_world_size)
        conv_state_shape = cls._orient_conv_shape(conv_dim, conv_kernel - 1)

        temporal_state_shape = (divide(intermediate_size, tp_world_size), state_size)

        return conv_state_shape, temporal_state_shape
```
**EN:** Defines function `MambaStateShapeCalculator.mamba1_state_shape` with signature `mamba1_state_shape(cls, tp_world_size: int, intermediate_size: int, state_size: int, conv_kernel: int) -> tuple[tuple[int, int], tuple[int, int]]`. It mainly works with `tp_world_size`, `intermediate_size`, `state_size`, `conv_kernel`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `divide`, `cls._orient_conv_shape`.
**CN:** 定义函数 `MambaStateShapeCalculator.mamba1_state_shape`，其签名为 `mamba1_state_shape(cls, tp_world_size: int, intermediate_size: int, state_size: int, conv_kernel: int) -> tuple[tuple[int, int], tuple[int, int]]`。它主要围绕 `tp_world_size`, `intermediate_size`, `state_size`, `conv_kernel` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `divide`, `cls._orient_conv_shape`。

### Method `MambaStateShapeCalculator.mamba2_state_shape` (lines 162-187)
```python
    def mamba2_state_shape(
        cls,
        tp_world_size: int,
        intermediate_size: int,
        n_groups: int,
        num_heads: int,
        head_dim: int,
        state_size: int,
        conv_kernel: int,
        num_spec: int = 0,
    ) -> tuple[tuple[int, int], tuple[int, int, int]]:
        # if n_groups is not divisible by world_size, need to extend the shards
        # to ensure all groups needed by a head is sharded along with it
        n_groups = n_groups + cls.extra_groups_for_head_shards(n_groups, tp_world_size)
        # heads and n_groups are TP-ed
        conv_dim = intermediate_size + 2 * n_groups * state_size

        conv_state_shape = cls._orient_conv_shape(
            divide(conv_dim, tp_world_size), conv_kernel - 1 + num_spec
        )

        # These are not TP-ed as they depend on A, dt_bias, D
        # - they are typically small
        #   e.g., (h_heads, head_dim, state_size) = (128, 64, 128)
        temporal_state_shape = (divide(num_heads, tp_world_size), head_dim, state_size)
        return conv_state_shape, temporal_state_shape
```
**EN:** Defines function `MambaStateShapeCalculator.mamba2_state_shape` with signature `mamba2_state_shape(cls, tp_world_size: int, intermediate_size: int, n_groups: int, num_heads: int, head_dim: int, state_size: int, conv_kernel: int, num_spec: int=0) -> tuple[tuple[int, int], tuple[int, int, int]]`. It mainly works with `tp_world_size`, `intermediate_size`, `n_groups`, `num_heads`, `head_dim`, `state_size`, `conv_kernel`, `num_spec`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `cls._orient_conv_shape`, `cls.extra_groups_for_head_shards`, `divide`.
**CN:** 定义函数 `MambaStateShapeCalculator.mamba2_state_shape`，其签名为 `mamba2_state_shape(cls, tp_world_size: int, intermediate_size: int, n_groups: int, num_heads: int, head_dim: int, state_size: int, conv_kernel: int, num_spec: int=0) -> tuple[tuple[int, int], tuple[int, int, int]]`。它主要围绕 `tp_world_size`, `intermediate_size`, `n_groups`, `num_heads`, `head_dim`, `state_size`, `conv_kernel`, `num_spec` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls._orient_conv_shape`, `cls.extra_groups_for_head_shards`, `divide`。

### Method `MambaStateShapeCalculator.short_conv_state_shape` (lines 190-198)
```python
    def short_conv_state_shape(
        cls,
        tp_world_size: int,
        intermediate_size: int,
        conv_kernel: int,
    ) -> tuple[tuple[int, int]]:
        conv_dim = divide(intermediate_size, tp_world_size)
        conv_state_shape = cls._orient_conv_shape(conv_dim, conv_kernel - 1)
        return (conv_state_shape,)
```
**EN:** Defines function `MambaStateShapeCalculator.short_conv_state_shape` with signature `short_conv_state_shape(cls, tp_world_size: int, intermediate_size: int, conv_kernel: int) -> tuple[tuple[int, int]]`. It mainly works with `tp_world_size`, `intermediate_size`, `conv_kernel`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `divide`, `cls._orient_conv_shape`.
**CN:** 定义函数 `MambaStateShapeCalculator.short_conv_state_shape`，其签名为 `short_conv_state_shape(cls, tp_world_size: int, intermediate_size: int, conv_kernel: int) -> tuple[tuple[int, int]]`。它主要围绕 `tp_world_size`, `intermediate_size`, `conv_kernel` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `divide`, `cls._orient_conv_shape`。

### Method `MambaStateShapeCalculator.extra_groups_for_head_shards` (lines 201-210)
```python
    def extra_groups_for_head_shards(cls, ngroups: int, tp_size: int):
        """Compute the increase in group numbers to account for
        replication in order to accompany the head shards."""

        # in the case ngoups % tp_size == 0, this will be zero
        if ngroups % tp_size == 0:
            return 0

        # for n_groups == 1, this is exactly tp_size - n_groups
        return tp_size - ngroups
```
**EN:** Defines function `MambaStateShapeCalculator.extra_groups_for_head_shards` with signature `extra_groups_for_head_shards(cls, ngroups: int, tp_size: int)`. It mainly works with `ngroups`, `tp_size`; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include no notable helper calls.
**CN:** 定义函数 `MambaStateShapeCalculator.extra_groups_for_head_shards`，其签名为 `extra_groups_for_head_shards(cls, ngroups: int, tp_size: int)`。它主要围绕 `ngroups`, `tp_size` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 无显著辅助调用。

### Method `MambaStateShapeCalculator.gated_delta_net_state_shape` (lines 213-234)
```python
    def gated_delta_net_state_shape(
        cls,
        tp_world_size: int,
        num_k_heads: int,
        num_v_heads: int,
        head_k_dim: int,
        head_v_dim: int,
        conv_kernel_size: int,
        num_spec: int = 0,
    ):
        conv_dim = head_k_dim * num_k_heads * 2 + head_v_dim * num_v_heads
        conv_state_shape = cls._orient_conv_shape(
            divide(conv_dim, tp_world_size),
            conv_kernel_size - 1 + num_spec,
        )

        temporal_state_shape = (
            divide(num_v_heads, tp_world_size),
            head_v_dim,
            head_k_dim,
        )
        return conv_state_shape, temporal_state_shape
```
**EN:** Defines function `MambaStateShapeCalculator.gated_delta_net_state_shape` with signature `gated_delta_net_state_shape(cls, tp_world_size: int, num_k_heads: int, num_v_heads: int, head_k_dim: int, head_v_dim: int, conv_kernel_size: int, num_spec: int=0)`. It mainly works with `tp_world_size`, `num_k_heads`, `num_v_heads`, `head_k_dim`, `head_v_dim`, `conv_kernel_size`, `num_spec`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `cls._orient_conv_shape`, `divide`.
**CN:** 定义函数 `MambaStateShapeCalculator.gated_delta_net_state_shape`，其签名为 `gated_delta_net_state_shape(cls, tp_world_size: int, num_k_heads: int, num_v_heads: int, head_k_dim: int, head_v_dim: int, conv_kernel_size: int, num_spec: int=0)`。它主要围绕 `tp_world_size`, `num_k_heads`, `num_v_heads`, `head_k_dim`, `head_v_dim`, `conv_kernel_size`, `num_spec` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls._orient_conv_shape`, `divide`。

### Method `MambaStateShapeCalculator.kda_state_shape` (lines 237-267)
```python
    def kda_state_shape(
        cls,
        tp_world_size: int,
        num_heads: int,
        head_dim: int,
        num_k_heads: int | None = None,
        head_k_dim: int | None = None,
        conv_kernel_size: int = 4,
        num_spec: int = 0,
    ) -> tuple[tuple[int, int], tuple[int, int], tuple[int, int], tuple[int, int, int]]:
        if num_k_heads is None:
            num_k_heads = num_heads
        if head_k_dim is None:
            head_k_dim = head_dim

        proj_size = num_heads * head_dim
        proj_k_size = num_k_heads * head_k_dim

        conv_state_shape = cls._orient_conv_shape(
            divide(proj_size, tp_world_size), conv_kernel_size - 1
        )
        conv_state_k_shape = cls._orient_conv_shape(
            divide(proj_k_size, tp_world_size), conv_kernel_size - 1
        )
        recurrent_state_shape = (divide(num_heads, tp_world_size), head_dim, head_dim)
        return (
            conv_state_shape,
            conv_state_k_shape,
            conv_state_k_shape,
            recurrent_state_shape,
        )
```
**EN:** Defines function `MambaStateShapeCalculator.kda_state_shape` with signature `kda_state_shape(cls, tp_world_size: int, num_heads: int, head_dim: int, num_k_heads: int | None=None, head_k_dim: int | None=None, conv_kernel_size: int=4, num_spec: int=0) -> tuple[tuple[int, int], tuple[int, int], tuple[int, int], tuple[int, int, int]]`. It mainly works with `tp_world_size`, `num_heads`, `head_dim`, `num_k_heads`, `head_k_dim`, `conv_kernel_size`, `num_spec`; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include `cls._orient_conv_shape`, `divide`.
**CN:** 定义函数 `MambaStateShapeCalculator.kda_state_shape`，其签名为 `kda_state_shape(cls, tp_world_size: int, num_heads: int, head_dim: int, num_k_heads: int | None=None, head_k_dim: int | None=None, conv_kernel_size: int=4, num_spec: int=0) -> tuple[tuple[int, int], tuple[int, int], tuple[int, int], tuple[int, int, int]]`。它主要围绕 `tp_world_size`, `num_heads`, `head_dim`, `num_k_heads`, `head_k_dim`, `conv_kernel_size`, `num_spec` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 `cls._orient_conv_shape`, `divide`。

### Class `MambaCopySpec` overview (lines 271-282)
```python
class MambaCopySpec:
    """
    Data class specifying the memory-copy parameters for Mamba states used for
    prefix caching in align mode.

    Attributes:
        start_addr (int): Starting address for the memory copy operation.
        num_elements (int): Number of elements to copy from the starting address.
    """

    start_addr: int
    num_elements: int
```
**EN:** Defines class `MambaCopySpec` with base classes no explicit base class and decorators `dataclass`. It acts as a Mamba/SSM module building block and exposes 0 direct methods, with notable entries no direct methods. Its docstring says: Data class specifying the memory-copy parameters for Mamba states used for prefix caching in align mode.
**CN:** 定义类 `MambaCopySpec`，其基类为 无显式基类，装饰器为 `dataclass`。它在整体实现中充当Mamba/状态空间模块构件，并直接暴露 0 个方法，较重要的包括 无直接方法。 文档字符串进一步说明了该类的定位。

### Class `MambaStateCopyFuncCalculator` overview (lines 345-373)
```python
class MambaStateCopyFuncCalculator:
    @classmethod
    def linear_attention_state_copy_func(cls):
        return (get_temporal_copy_spec,)

    @classmethod
    def mamba1_state_copy_func(cls):
        return (get_conv_copy_spec, get_temporal_copy_spec)

    @classmethod
    def mamba2_state_copy_func(cls):
        return get_conv_copy_spec, get_temporal_copy_spec

    @classmethod
    def short_conv_state_copy_func(cls):
        return (get_conv_copy_spec,)

    @classmethod
    def gated_delta_net_state_copy_func(cls):
        return (get_conv_copy_spec, get_temporal_copy_spec)

    @classmethod
    def kda_state_copy_func(cls):
        return (
            get_conv_copy_spec,
```
**EN:** Defines class `MambaStateCopyFuncCalculator` with base classes no explicit base class and decorators none. It acts as a Mamba/SSM module building block and exposes 6 direct methods, with notable entries `linear_attention_state_copy_func`, `mamba1_state_copy_func`, `mamba2_state_copy_func`, `short_conv_state_copy_func`, `gated_delta_net_state_copy_func`, `kda_state_copy_func`.
**CN:** 定义类 `MambaStateCopyFuncCalculator`，其基类为 无显式基类，装饰器为 无。它在整体实现中充当Mamba/状态空间模块构件，并直接暴露 6 个方法，较重要的包括 `linear_attention_state_copy_func`, `mamba1_state_copy_func`, `mamba2_state_copy_func`, `short_conv_state_copy_func`, `gated_delta_net_state_copy_func`, `kda_state_copy_func`。

### Method `MambaStateCopyFuncCalculator.linear_attention_state_copy_func` (lines 347-348)
```python
    def linear_attention_state_copy_func(cls):
        return (get_temporal_copy_spec,)
```
**EN:** Defines function `MambaStateCopyFuncCalculator.linear_attention_state_copy_func` with signature `linear_attention_state_copy_func(cls)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MambaStateCopyFuncCalculator.linear_attention_state_copy_func`，其签名为 `linear_attention_state_copy_func(cls)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MambaStateCopyFuncCalculator.mamba1_state_copy_func` (lines 351-352)
```python
    def mamba1_state_copy_func(cls):
        return (get_conv_copy_spec, get_temporal_copy_spec)
```
**EN:** Defines function `MambaStateCopyFuncCalculator.mamba1_state_copy_func` with signature `mamba1_state_copy_func(cls)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MambaStateCopyFuncCalculator.mamba1_state_copy_func`，其签名为 `mamba1_state_copy_func(cls)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MambaStateCopyFuncCalculator.mamba2_state_copy_func` (lines 355-356)
```python
    def mamba2_state_copy_func(cls):
        return get_conv_copy_spec, get_temporal_copy_spec
```
**EN:** Defines function `MambaStateCopyFuncCalculator.mamba2_state_copy_func` with signature `mamba2_state_copy_func(cls)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MambaStateCopyFuncCalculator.mamba2_state_copy_func`，其签名为 `mamba2_state_copy_func(cls)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MambaStateCopyFuncCalculator.short_conv_state_copy_func` (lines 359-360)
```python
    def short_conv_state_copy_func(cls):
        return (get_conv_copy_spec,)
```
**EN:** Defines function `MambaStateCopyFuncCalculator.short_conv_state_copy_func` with signature `short_conv_state_copy_func(cls)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MambaStateCopyFuncCalculator.short_conv_state_copy_func`，其签名为 `short_conv_state_copy_func(cls)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MambaStateCopyFuncCalculator.gated_delta_net_state_copy_func` (lines 363-364)
```python
    def gated_delta_net_state_copy_func(cls):
        return (get_conv_copy_spec, get_temporal_copy_spec)
```
**EN:** Defines function `MambaStateCopyFuncCalculator.gated_delta_net_state_copy_func` with signature `gated_delta_net_state_copy_func(cls)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MambaStateCopyFuncCalculator.gated_delta_net_state_copy_func`，其签名为 `gated_delta_net_state_copy_func(cls)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MambaStateCopyFuncCalculator.kda_state_copy_func` (lines 367-373)
```python
    def kda_state_copy_func(cls):
        return (
            get_conv_copy_spec,
            get_conv_copy_spec,
            get_conv_copy_spec,
            get_temporal_copy_spec,
        )
```
**EN:** Defines function `MambaStateCopyFuncCalculator.kda_state_copy_func` with signature `kda_state_copy_func(cls)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MambaStateCopyFuncCalculator.kda_state_copy_func`，其签名为 `kda_state_copy_func(cls)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

## Key Concepts / 关键概念
- **EN:** The file belongs to the Mamba/SSM path, so recurrent state updates, scan-style kernels, or short convolutions are recurring themes.
  **CN:** 该文件位于 Mamba/状态空间模型路径中，因此循环状态更新、scan 风格内核或短卷积是反复出现的主题。
- **EN:** Top-level classes include `MambaStateDtypeCalculator`, `MambaStateShapeCalculator`, `MambaCopySpec`, `MambaStateCopyFuncCalculator`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `MambaStateDtypeCalculator`, `MambaStateShapeCalculator`, `MambaCopySpec`, `MambaStateCopyFuncCalculator`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_conv_state_layout`, `is_conv_state_dim_first`, `get_conv_copy_spec`, `get_temporal_copy_spec` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_conv_state_layout`, `is_conv_state_dim_first`, `get_conv_copy_spec`, `get_temporal_copy_spec` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `functools`, `collections`, `dataclasses`, `typing`, `torch`
- **Internal / 内部**: `vllm.envs`, `vllm.config.cache`, `vllm.config.model`, `vllm.distributed`, `vllm.logger`, `vllm.utils.torch_utils`
