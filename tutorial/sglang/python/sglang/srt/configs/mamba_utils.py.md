# mamba_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/mamba_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides core runtime configuration logic for mamba utils, including shared helpers, loading paths, or registry behavior. / 该模块提供与 mamba utils 相关的核心运行时配置逻辑，包括通用辅助函数、加载流程或注册表行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Comments and module notes
```python
# Copyright 2025 SGLang Team
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
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 13-13: Documents the scope
```python
"""Common config utils for mamba2 - NemotronH, FalconH1, Qwen3Next, LFM2, etc."""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 14-14: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 15-24: Imports dependencies
```python
import logging
from abc import ABC
from dataclasses import dataclass, field
from typing import List, Optional

import numpy as np
import torch

from sglang.srt.distributed.utils import divide
from sglang.srt.environ import envs
```
**EN:** This block groups related imports for the module, including logging, abc.ABC, dataclasses.dataclass, dataclasses.field, typing.List, and 5 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 logging, abc.ABC, dataclasses.dataclass, dataclasses.field, typing.List 等 5 项，为后续代码准备所需名称。

### Lines 25-25: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-26: Declares logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 27-28: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 29-38: Defines function extra_groups_for_head_shards
```python
def extra_groups_for_head_shards(ngroups: int, tp_size: int):
    """Compute the increase in group numbers to account for
    replication in order to accompany the head shards."""

    # in the case ngoups % tp_size == 0, this will be zero
    if ngroups % tp_size == 0:
        return 0

    # for n_groups == 1, this is exactly tp_size - n_groups
    return tp_size - ngroups
```
**EN:** This block defines function `extra_groups_for_head_shards`. Parameters: ngroups, tp_size. Compute the increase in group numbers to account for replication in order to accompany the head shards.
**CN:** 该代码块定义函数 `extra_groups_for_head_shards`。 参数包括 ngroups、tp_size。 文档字符串摘要：Compute the increase in group numbers to account for replication in order to accompany the head shards.

### Lines 39-40: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 41-42: Declares class Mamba2StateDType
```python
@dataclass(kw_only=True, frozen=True)
class Mamba2StateDType:
```
**EN:** This block introduces class `Mamba2StateDType` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `Mamba2StateDType`，作为模块中的可复用抽象。

### Lines 43-44: Declares conv, temporal
```python
    conv: torch.dtype
    temporal: torch.dtype
```
**EN:** This block initializes a related set of values in the Mamba2StateDType, including conv, temporal. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Mamba2StateDType 中初始化一组相关值，包括 conv, temporal。将这些赋值集中在一起有助于理解周边配置。

### Lines 45-46: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 47-107: Defines function mamba2_state_dtype
```python
def mamba2_state_dtype(config=None) -> Mamba2StateDType:
    """
    Get mamba2 state dtype from config or environment variable.

    Priority (from highest to lowest):
    1. Environment variable SGLANG_MAMBA_SSM_DTYPE
    2. Config file (config.mamba_ssm_dtype or config.text_config.mamba_ssm_dtype)
    3. Default "float32"

    Args:
        config: Optional config object (PretrainedConfig). If provided, will read
                mamba_ssm_dtype from it. For VL models, reads from text_config.

    Returns:
        Mamba2StateDType with conv and temporal dtypes
    """
    dtype_map = {
        "float32": torch.float32,
        "bfloat16": torch.bfloat16,
        "float16": torch.float16,
    }
    conv_dtype = dtype_map.get(envs.SGLANG_MAMBA_CONV_DTYPE.get(), torch.bfloat16)

    # Get SSM dtype: default -> config -> env var
    ssm_dtype = torch.float32  # Step 1: Default value

    # Step 2: Try to read from config
    if config is not None:
        config_dtype = None
        if hasattr(config, "text_config") and hasattr(
            config.text_config, "mamba_ssm_dtype"
        ):
            # VL model: read from text_config
            config_dtype = config.text_config.mamba_ssm_dtype
        elif hasattr(config, "mamba_ssm_dtype"):
            # Text model: read from root config
            config_dtype = config.mamba_ssm_dtype

        if config_dtype is not None:
            if config_dtype not in dtype_map:
                logger.warning(
                    f"Invalid mamba_ssm_dtype '{config_dtype}' in config. "
                    f"Must be one of {list(dtype_map.keys())}. Using default 'float32'."
                )
            else:
                ssm_dtype = dtype_map[config_dtype]

    # Step 3: Check environment variable, if not None, override
    env_ssm_dtype = envs.SGLANG_MAMBA_SSM_DTYPE.get()
    if env_ssm_dtype is not None:
        if env_ssm_dtype not in dtype_map:
            logger.warning(
                f"Invalid mamba_ssm_dtype '{env_ssm_dtype}' from environment variable. "
                f"Must be one of {list(dtype_map.keys())}. Using default 'float32'."
            )
        else:
            ssm_dtype = dtype_map[env_ssm_dtype]

    logger.debug(f"Mamba2 state dtype: conv_dtype={conv_dtype}, ssm_dtype={ssm_dtype}")

    return Mamba2StateDType(conv=conv_dtype, temporal=ssm_dtype)
```
**EN:** This block defines function `mamba2_state_dtype`. Parameters: config. Get mamba2 state dtype from config or environment variable. Priority (from highest to lowest): 1.
**CN:** 该代码块定义函数 `mamba2_state_dtype`。 参数包括 config。 文档字符串摘要：Get mamba2 state dtype from config or environment variable. Priority (from highest to lowest): 1.

### Lines 108-109: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 110-111: Declares class BaseLinearStateParams
```python
@dataclass(kw_only=True, frozen=True)
class BaseLinearStateParams(ABC):
```
**EN:** This block introduces class `BaseLinearStateParams` as a reusable abstraction inside the module. It inherits from ABC.
**CN:** 该代码块声明类 `BaseLinearStateParams`，作为模块中的可复用抽象。 它继承自 ABC。

### Lines 112-113: Declares dtype, layers
```python
    dtype: Mamba2StateDType = field(default_factory=lambda: mamba2_state_dtype(None))
    layers: list[int]
```
**EN:** This block initializes a related set of values in the BaseLinearStateParams, including dtype, layers. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 BaseLinearStateParams 中初始化一组相关值，包括 dtype, layers。将这些赋值集中在一起有助于理解周边配置。

### Lines 114-114: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseLinearStateParams, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseLinearStateParams 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 115-125: Defines function BaseLinearStateParams.mamba_cache_per_req
```python
    @property
    def mamba_cache_per_req(self) -> int:
        conv_numel = int(
            np.sum([np.prod(conv_shape) for conv_shape in self.shape.conv])
        )

        ssm_numel = int(np.prod(self.shape.temporal))
        return (
            conv_numel * self.dtype.conv.itemsize
            + ssm_numel * self.dtype.temporal.itemsize
        ) * len(self.layers)
```
**EN:** This block defines function `BaseLinearStateParams.mamba_cache_per_req`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `BaseLinearStateParams.mamba_cache_per_req`。 参数包括 self。 装饰器包括 property。

### Lines 126-127: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 128-129: Declares class Mamba2StateShape
```python
@dataclass(kw_only=True, frozen=True)
class Mamba2StateShape:
```
**EN:** This block introduces class `Mamba2StateShape` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `Mamba2StateShape`，作为模块中的可复用抽象。

### Lines 130-131: Declares conv, temporal
```python
    conv: list[tuple[int, int]]
    temporal: tuple[int, int, int]
```
**EN:** This block initializes a related set of values in the Mamba2StateShape, including conv, temporal. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Mamba2StateShape 中初始化一组相关值，包括 conv, temporal。将这些赋值集中在一起有助于理解周边配置。

### Lines 132-132: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Mamba2StateShape, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Mamba2StateShape 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 133-139: Declares intermediate_size, conv_dim, ssm_state_size, num_heads, head_dim, and 2 more
```python
    intermediate_size: int
    conv_dim: int
    ssm_state_size: int
    num_heads: int
    head_dim: int
    state_size: int
    conv_kernel: int
```
**EN:** This block initializes a related set of values in the Mamba2StateShape, including intermediate_size, conv_dim, ssm_state_size, num_heads, head_dim, and 2 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Mamba2StateShape 中初始化一组相关值，包括 intermediate_size, conv_dim, ssm_state_size, num_heads, head_dim 等 2 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 140-140: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Mamba2StateShape, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Mamba2StateShape 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 141-177: Defines function Mamba2StateShape.create
```python
    @staticmethod
    def create(
        *,
        tp_world_size: int,
        intermediate_size: int,
        n_groups: int,
        num_heads: int,
        head_dim: int,
        state_size: int,
        conv_kernel: int,
    ) -> "Mamba2StateShape":
        # if n_groups is not divisible by world_size, need to extend the shards
        # to ensure all groups needed by a head is sharded along with it
        if n_groups % tp_world_size != 0:
            extra_groups = extra_groups_for_head_shards(n_groups, tp_world_size)
            n_groups += extra_groups
        # heads and n_groups are TP-ed
        conv_dim = intermediate_size + 2 * n_groups * state_size

        # contiguous along 'dim' axis
        conv_state_shape = divide(conv_dim, tp_world_size), conv_kernel - 1

        # These are not TP-ed as they depend on A, dt_bias, D
        # - they are typically small
        #   e.g., QWen3-Next: (32, 128, 128)
        temporal_state_shape = (divide(num_heads, tp_world_size), head_dim, state_size)
        return Mamba2StateShape(
            conv=[conv_state_shape],
            temporal=temporal_state_shape,
            intermediate_size=intermediate_size,
            conv_dim=conv_dim,
            ssm_state_size=state_size,
            num_heads=num_heads,
            head_dim=head_dim,
            state_size=state_size,
            conv_kernel=conv_kernel,
        )
```
**EN:** This block defines function `Mamba2StateShape.create`. It takes no explicit parameters. Decorators: staticmethod.
**CN:** 该代码块定义函数 `Mamba2StateShape.create`。 它没有显式参数。 装饰器包括 staticmethod。

### Lines 178-179: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 180-181: Declares class Mamba2CacheParams
```python
@dataclass(kw_only=True, frozen=True)
class Mamba2CacheParams(BaseLinearStateParams):
```
**EN:** This block introduces class `Mamba2CacheParams` as a reusable abstraction inside the module. It inherits from BaseLinearStateParams.
**CN:** 该代码块声明类 `Mamba2CacheParams`，作为模块中的可复用抽象。 它继承自 BaseLinearStateParams。

### Lines 182-182: Declares shape
```python
    shape: Mamba2StateShape
```
**EN:** This statement initializes shape in the Mamba2CacheParams. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Mamba2CacheParams 中初始化 shape。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 183-184: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 185-186: Declares class KimiLinearStateShape
```python
@dataclass(kw_only=True, frozen=True)
class KimiLinearStateShape:
```
**EN:** This block introduces class `KimiLinearStateShape` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `KimiLinearStateShape`，作为模块中的可复用抽象。

### Lines 187-188: Declares conv, temporal
```python
    conv: List[tuple[int, int]]
    temporal: tuple[int, int, int]
```
**EN:** This block initializes a related set of values in the KimiLinearStateShape, including conv, temporal. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 KimiLinearStateShape 中初始化一组相关值，包括 conv, temporal。将这些赋值集中在一起有助于理解周边配置。

### Lines 189-189: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearStateShape, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearStateShape 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 190-195: Declares num_heads, head_dim, num_k_heads, head_k_dim, conv_kernel, and 1 more
```python
    num_heads: int
    head_dim: int
    num_k_heads: int
    head_k_dim: int
    conv_kernel: int
    num_spec: int
```
**EN:** This block initializes a related set of values in the KimiLinearStateShape, including num_heads, head_dim, num_k_heads, head_k_dim, conv_kernel, and 1 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 KimiLinearStateShape 中初始化一组相关值，包括 num_heads, head_dim, num_k_heads, head_k_dim, conv_kernel 等 1 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 196-196: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearStateShape, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearStateShape 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 197-234: Defines function KimiLinearStateShape.create
```python
    @staticmethod
    def create(
        *,
        tp_world_size: int,
        num_heads: int,
        head_dim: int,
        num_k_heads: Optional[int] = None,
        head_k_dim: Optional[int] = None,
        conv_kernel_size: int = 4,
        num_spec: int = 0,
    ) -> "KimiLinearStateShape":
        if num_k_heads is None:
            num_k_heads = num_heads
        if head_k_dim is None:
            head_k_dim = head_dim

        proj_size = num_heads * head_dim
        proj_k_size = num_k_heads * head_k_dim

        conv_state_shape = (divide(proj_size, tp_world_size), conv_kernel_size - 1)
        conv_state_k_shape = (divide(proj_k_size, tp_world_size), conv_kernel_size - 1)
        temporal_state_shape = (divide(num_heads, tp_world_size), head_dim, head_dim)

        conv_state_shape = (
            conv_state_shape[1],
            conv_state_shape[0] + conv_state_k_shape[0] * 2,
        )

        return KimiLinearStateShape(
            conv=[conv_state_shape],
            temporal=temporal_state_shape,
            num_heads=num_heads,
            head_dim=head_dim,
            num_k_heads=num_k_heads,
            head_k_dim=head_k_dim,
            conv_kernel=conv_kernel_size,
            num_spec=num_spec,
        )
```
**EN:** This block defines function `KimiLinearStateShape.create`. It takes no explicit parameters. Decorators: staticmethod.
**CN:** 该代码块定义函数 `KimiLinearStateShape.create`。 它没有显式参数。 装饰器包括 staticmethod。

### Lines 235-236: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 237-238: Declares class KimiLinearCacheParams
```python
@dataclass(kw_only=True, frozen=True)
class KimiLinearCacheParams(BaseLinearStateParams):
```
**EN:** This block introduces class `KimiLinearCacheParams` as a reusable abstraction inside the module. It inherits from BaseLinearStateParams.
**CN:** 该代码块声明类 `KimiLinearCacheParams`，作为模块中的可复用抽象。 它继承自 BaseLinearStateParams。

### Lines 239-239: Declares shape
```python
    shape: KimiLinearStateShape
```
**EN:** This statement initializes shape in the KimiLinearCacheParams. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 KimiLinearCacheParams 中初始化 shape。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

## Key Concepts / 关键概念
- **Classes / 类**: `Mamba2StateDType`, `BaseLinearStateParams`, `Mamba2StateShape`, `Mamba2CacheParams`, `KimiLinearStateShape`, `KimiLinearCacheParams`
- **Functions / 函数**: `extra_groups_for_head_shards`, `mamba2_state_dtype`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `abc`, `dataclasses`, `logging`, `typing`
- **Third-Party / 第三方**: `numpy`, `torch`
- **Local Modules / 本地模块**: `sglang.srt.distributed.utils`, `sglang.srt.environ`
