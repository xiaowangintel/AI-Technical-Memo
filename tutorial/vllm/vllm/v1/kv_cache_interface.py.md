# kv_cache_interface.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_cache_interface.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `KVQuantMode`, `get_kv_quant_mode`, `is_quantized_kv_cache` for the V1 `v1` subsystem. / 为 V1 的 `v1` 子系统实现 `KVQuantMode`, `get_kv_quant_mode`, `is_quantized_kv_cache`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from __future__ import annotations

import copy
from collections import Counter
from dataclasses import dataclass, fields, replace
from enum import Enum, IntEnum
from math import prod
from typing import TYPE_CHECKING

import torch
from typing_extensions import Self

from vllm.logger import init_logger
from vllm.utils.math_utils import cdiv, round_up
from vllm.utils.torch_utils import get_dtype_size, nvfp4_kv_cache_full_dim
from vllm.v1.attention.backends.registry import MambaAttentionBackendEnum

if TYPE_CHECKING:
    from vllm.config import VllmConfig

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `KVQuantMode` class / `KVQuantMode` 类
```python
class KVQuantMode(IntEnum):
    """KV cache quantization mode.

    Used by attention backends and kernels to dispatch quantization logic
    without string matching on ``kv_cache_dtype``.
    """

    NONE = 0
    FP8_PER_TENSOR = 1  # per-tensor scales (current fp8 path)
    INT8_PER_TOKEN_HEAD = 2  # per-token-head dynamic scales for int8
    FP8_PER_TOKEN_HEAD = 3  # per-token-head dynamic scales for fp8
    NVFP4 = 4  # packed fp4 data + fp8 block scales
```
**EN:** Defines the `KVQuantMode` enum used to normalize modes or options across the subsystem. Representative members: `NONE`, `FP8_PER_TENSOR`, `INT8_PER_TOKEN_HEAD`, `FP8_PER_TOKEN_HEAD`, `NVFP4`.
**CN:** `KVQuantMode` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`NONE`, `FP8_PER_TENSOR`, `INT8_PER_TOKEN_HEAD`, `FP8_PER_TOKEN_HEAD`, `NVFP4`。

### `KVQuantMode.is_per_token_head` method / `KVQuantMode.is_per_token_head` 方法
```python
    @property
    def is_per_token_head(self) -> bool:
        """True for any per-token-head quantization mode."""
        return self in (
            KVQuantMode.INT8_PER_TOKEN_HEAD,
            KVQuantMode.FP8_PER_TOKEN_HEAD,
        )
```
**EN:** This method answers a boolean capability check within `KVQuantMode`. The docstring frames it as: True for any per-token-head quantization mode.
**CN:** 该方法会回答布尔能力判断，其作用域位于`KVQuantMode`。

### `KVQuantMode.is_nvfp4` method / `KVQuantMode.is_nvfp4` 方法
```python
    @property
    def is_nvfp4(self) -> bool:
        """True for NVFP4 packed quantization mode."""
        return self == KVQuantMode.NVFP4
```
**EN:** This method answers a boolean capability check within `KVQuantMode`. The docstring frames it as: True for NVFP4 packed quantization mode.
**CN:** 该方法会回答布尔能力判断，其作用域位于`KVQuantMode`。

### `get_kv_quant_mode` function / `get_kv_quant_mode` 函数
```python
def get_kv_quant_mode(kv_cache_dtype: str) -> KVQuantMode:
    """Map a ``kv_cache_dtype`` string to a :class:`KVQuantMode`."""
    if kv_cache_dtype == "int8_per_token_head":
        return KVQuantMode.INT8_PER_TOKEN_HEAD
    if kv_cache_dtype == "fp8_per_token_head":
        return KVQuantMode.FP8_PER_TOKEN_HEAD
    if kv_cache_dtype == "nvfp4":
        return KVQuantMode.NVFP4
    if isinstance(kv_cache_dtype, str) and kv_cache_dtype.startswith("fp8"):
        return KVQuantMode.FP8_PER_TENSOR
    return KVQuantMode.NONE
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Map a ``kv_cache_dtype`` string to a :class:`KVQuantMode`. Key calls include `isinstance`, `startswith`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `isinstance`, `startswith`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `is_quantized_kv_cache` function / `is_quantized_kv_cache` 函数
```python
def is_quantized_kv_cache(kv_cache_dtype: str) -> bool:
    return get_kv_quant_mode(kv_cache_dtype) != KVQuantMode.NONE
```
**EN:** This function answers a boolean capability check within the module. Key calls include `get_kv_quant_mode`.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。 关键调用包括 `get_kv_quant_mode`。

### `kv_cache_uses_per_token_head_scales` function / `kv_cache_uses_per_token_head_scales` 函数
```python
def kv_cache_uses_per_token_head_scales(kv_cache_dtype: str) -> bool:
    """Return True if *kv_cache_dtype* needs per-token-head scales."""
    return get_kv_quant_mode(kv_cache_dtype).is_per_token_head
```
**EN:** This function implements `kv_cache_uses_per_token_head_scales` within the module. The docstring frames it as: Return True if *kv_cache_dtype* needs per-token-head scales. Key calls include `get_kv_quant_mode`.
**CN:** 该函数会实现 `kv_cache_uses_per_token_head_scales`，其作用域位于the module。 关键调用包括 `get_kv_quant_mode`。

### `KVCacheSpecKind` class / `KVCacheSpecKind` 类
```python
class KVCacheSpecKind(str, Enum):
    FULL_ATTENTION = "full_attention"
    MLA_ATTENTION = "mla_attention"
    SLIDING_WINDOW = "sliding_window"
    SLIDING_WINDOW_MLA = "sliding_window_mla"
    MAMBA = "mamba"
    CHUNKED_LOCAL_ATTENTION = "chunked_local_attention"
    SINK_FULL_ATTENTION = "sink_full_attention"
    ENCODER_ONLY_ATTENTION = "encoder_only_attention"
    CROSS_ATTENTION = "cross_attention"
    UNKNOWN = "unknown"
```
**EN:** Defines the `KVCacheSpecKind` enum used to normalize modes or options across the subsystem. Representative members: `FULL_ATTENTION`, `MLA_ATTENTION`, `SLIDING_WINDOW`, `SLIDING_WINDOW_MLA`, `MAMBA`, `CHUNKED_LOCAL_ATTENTION`.
**CN:** `KVCacheSpecKind` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`FULL_ATTENTION`, `MLA_ATTENTION`, `SLIDING_WINDOW`, `SLIDING_WINDOW_MLA`, `MAMBA`, `CHUNKED_LOCAL_ATTENTION`。

### `KVCacheSpec` class / `KVCacheSpec` 类
```python
@dataclass(frozen=True)
class KVCacheSpec:
    """
    A base class for specifying the KV cache format of one layer.
    """

    # number of tokens in a block
    block_size: int
```
**EN:** Introduces the `KVCacheSpec` class. Core methods include `page_size_bytes`, `storage_block_size`, `max_memory_usage_bytes`, `copy_with_new_block_size`, `merge`. Docstring signal: A base class for specifying the KV cache format of one layer.
**CN:** 这里定义 `KVCacheSpec` 类。核心方法包括 `page_size_bytes`, `storage_block_size`, `max_memory_usage_bytes`, `copy_with_new_block_size`, `merge`。

### `KVCacheSpec.page_size_bytes` method / `KVCacheSpec.page_size_bytes` 方法
```python
    @property
    def page_size_bytes(self) -> int:
        """
        The size of a page with `block_size` tokens in bytes.

        Returns:
            The page size
        """
        raise NotImplementedError
```
**EN:** This method implements `page_size_bytes` within `KVCacheSpec`. The docstring frames it as: The size of a page with `block_size` tokens in bytes.
**CN:** 该方法会实现 `page_size_bytes`，其作用域位于`KVCacheSpec`。

### `KVCacheSpec.max_memory_usage_bytes` method / `KVCacheSpec.max_memory_usage_bytes` 方法
```python
    def max_memory_usage_bytes(self, vllm_config: VllmConfig) -> int:
        """
        The maximum possible memory usage of this KV cache in bytes.

        Returns:
            The KV cache size in bytes
        """
        raise NotImplementedError
```
**EN:** This method implements `max_memory_usage_bytes` within `KVCacheSpec`. The docstring frames it as: The maximum possible memory usage of this KV cache in bytes.
**CN:** 该方法会实现 `max_memory_usage_bytes`，其作用域位于`KVCacheSpec`。

### `KVCacheSpec.merge` method / `KVCacheSpec.merge` 方法
```python
    @classmethod
    def merge(cls, specs: list[Self]) -> Self:
        """
        Merge a list of KVCacheSpec objects into a single KVCacheSpec object.
        """
        assert all(spec == specs[0] for spec in specs[1:]), (
            "All layers in the same KV cache group must be the same."
        )
        return copy.deepcopy(specs[0])
```
**EN:** This method implements `merge` within `KVCacheSpec`. The docstring frames it as: Merge a list of KVCacheSpec objects into a single KVCacheSpec object. Key calls include `all`, `deepcopy`.
**CN:** 该方法会实现 `merge`，其作用域位于`KVCacheSpec`。 关键调用包括 `all`, `deepcopy`。

### `AttentionSpec` class / `AttentionSpec` 类
```python
@dataclass(frozen=True, kw_only=True)
class AttentionSpec(KVCacheSpec):
    num_kv_heads: int
    head_size: int
    dtype: torch.dtype
    kv_quant_mode: KVQuantMode = KVQuantMode.NONE
    page_size_padded: int | None = None
```
**EN:** Introduces the `AttentionSpec` class on top of `KVCacheSpec`. Core methods include `page_size_bytes`, `real_page_size_bytes`.
**CN:** 这里定义 `AttentionSpec` 类，其基类包括 `KVCacheSpec`。核心方法包括 `page_size_bytes`, `real_page_size_bytes`。

### `AttentionSpec.page_size_bytes` method / `AttentionSpec.page_size_bytes` 方法
```python
    @property
    def page_size_bytes(self) -> int:
        real_page_size = self.real_page_size_bytes
        # Per-token-head scales are stored in separate tensors managed
        # by the attention backend, but the memory is carved from the
        # raw KV cache allocation so it must be budgeted here.
        if self.kv_quant_mode.is_per_token_head:
            real_page_size += (
                2 * self.block_size * self.num_kv_heads * get_dtype_size(torch.float32)
            )
        if self.page_size_padded is not None:
            assert self.page_size_padded >= real_page_size
            return self.page_size_padded
        return real_page_size
```
**EN:** This method implements `page_size_bytes` within `AttentionSpec`. Key calls include `get_dtype_size`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `page_size_bytes`，其作用域位于`AttentionSpec`。 关键调用包括 `get_dtype_size`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionSpec.real_page_size_bytes` method / `AttentionSpec.real_page_size_bytes` 方法
```python
    @property
    def real_page_size_bytes(self) -> int:
        if self.kv_quant_mode.is_nvfp4:
            # Packed layout: fp4 data + fp8 block scales per head.
            full_dim = nvfp4_kv_cache_full_dim(self.head_size)
            return (
                2
                * self.block_size
                * self.num_kv_heads
                * full_dim
                * get_dtype_size(self.dtype)
            )
        return (
            2
            * self.block_size
            * self.num_kv_heads
            * self.head_size
            * get_dtype_size(self.dtype)
        )
```
**EN:** This method implements `real_page_size_bytes` within `AttentionSpec`. Key calls include `nvfp4_kv_cache_full_dim`, `get_dtype_size`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `real_page_size_bytes`，其作用域位于`AttentionSpec`。 关键调用包括 `nvfp4_kv_cache_full_dim`, `get_dtype_size`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FullAttentionSpec` class / `FullAttentionSpec` 类
```python
@dataclass(frozen=True, kw_only=True)
class FullAttentionSpec(AttentionSpec):
    """
    When hybrid allocator is disabled and the model contains both full
    attention layers and sliding window attention layers, sliding
    window attention are regarded as full attention in KV cache manager
    (blocks are allocated for all tokens), while computed as sliding window
    attention in model runner.
    In this case, we use FullAttentionSpec and record the sliding window size.
    """

    head_size_v: int = None  # type: ignore[assignment]

    sliding_window: int | None = None
    """
    Default to None for not using sliding window attention.
    """
    attention_chunk_size: int | None = None
```
**EN:** Introduces the `FullAttentionSpec` class on top of `AttentionSpec`. Core methods include `__post_init__`, `max_memory_usage_bytes`, `merge_window_sizes`, `merge`, `real_page_size_bytes`. Docstring signal: When hybrid allocator is disabled and the model contains both full attention layers and sliding window attention layers, sliding window attention are regarded as full attention in KV cache manager (blocks are allocated for all tokens), while computed as sliding window attention in model runner.
**CN:** 这里定义 `FullAttentionSpec` 类，其基类包括 `AttentionSpec`。核心方法包括 `__post_init__`, `max_memory_usage_bytes`, `merge_window_sizes`, `merge`, `real_page_size_bytes`。

### `FullAttentionSpec.merge_window_sizes` method / `FullAttentionSpec.merge_window_sizes` 方法
```python
    @classmethod
    def merge_window_sizes(cls, window_sizes: set[int]) -> int | None:
        if len(window_sizes) == 0:
            return None
        elif len(window_sizes) == 1:
            return window_sizes.pop()
        else:
            raise ValueError(
                "All attention layers in the same KV cache group must have the "
                "same window size."
            )
```
**EN:** This method implements `merge_window_sizes` within `FullAttentionSpec`. Key calls include `len`, `pop`, `ValueError`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `merge_window_sizes`，其作用域位于`FullAttentionSpec`。 关键调用包括 `len`, `pop`, `ValueError`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FullAttentionSpec.merge` method / `FullAttentionSpec.merge` 方法
```python
    @classmethod
    def merge(cls, specs: list[Self]) -> Self:
        """
        Merge a list of FullAttentionSpec objects into a single
        FullAttentionSpec object.
        """
        assert all(isinstance(spec, FullAttentionSpec) for spec in specs), (
            "All attention layers in the same KV cache group must be FullAttentionSpec."
        )

        sliding_window = set(
            spec.sliding_window for spec in specs if spec.sliding_window is not None
        )
        attention_chunk_size = set(
            spec.attention_chunk_size
            for spec in specs
            if spec.attention_chunk_size is not None
        )
        assert not any(isinstance(spec, MLAAttentionSpec) for spec in specs), (
            "MLAAttentionSpec should be merged in MLAAttentionSpec.merge"
        )
        merged_spec = cls(
            block_size=specs[0].block_size,
            num_kv_heads=specs[0].num_kv_heads,
            head_size=specs[0].head_size,
            head_size_v=specs[0].head_size_v,
            dtype=specs[0].dtype,
            kv_quant_mode=specs[0].kv_quant_mode,
            page_size_padded=specs[0].page_size_padded,
            sliding_window=cls.merge_window_sizes(sliding_window),
            attention_chunk_size=cls.merge_window_sizes(attention_chunk_size),
        )
        for spec in specs:
            for f in fields(AttentionSpec):
                assert getattr(spec, f.name) == getattr(merged_spec, f.name), (
                    "All attention layers in the same KV cache group must have "
                    "the same attention spec."
                )
        assert (merged_spec.sliding_window is not None) + (
            merged_spec.attention_chunk_size is not None
        ) <= 1, (
            "Model with both sliding window layers and chunked local attention "
            "layers is not supported."
        )
        return merged_spec
```
**EN:** This method implements `merge` within `FullAttentionSpec`. The docstring frames it as: Merge a list of FullAttentionSpec objects into a single FullAttentionSpec object. Key calls include `all`, `set`, `cls`, `any`, `fields`, `isinstance`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `merge`，其作用域位于`FullAttentionSpec`。 关键调用包括 `all`, `set`, `cls`, `any`, `fields`, `isinstance`。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `FullAttentionSpec.real_page_size_bytes` method / `FullAttentionSpec.real_page_size_bytes` 方法
```python
    @property
    def real_page_size_bytes(self) -> int:
        if self.kv_quant_mode.is_nvfp4:
            # Packed layout per head: fp4 data + fp8 block scales.
            # fp4 data: head_size//2 bytes (2 fp4 values per byte)
            # fp8 block scale: head_size//16 bytes (1 scale per 16 elements)
            last_dim = nvfp4_kv_cache_full_dim(
                self.head_size
            ) + nvfp4_kv_cache_full_dim(self.head_size_v)
            return (
                self.block_size
                * self.num_kv_heads
                * last_dim
                * get_dtype_size(self.dtype)
            )
        return (
            self.block_size
            * self.num_kv_heads
            * (self.head_size + self.head_size_v)
            * get_dtype_size(self.dtype)
        )
```
**EN:** This method implements `real_page_size_bytes` within `FullAttentionSpec`. Key calls include `get_dtype_size`, `nvfp4_kv_cache_full_dim`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `real_page_size_bytes`，其作用域位于`FullAttentionSpec`。 关键调用包括 `get_dtype_size`, `nvfp4_kv_cache_full_dim`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_apply_alignment_padding` function / `_apply_alignment_padding` 函数
```python
def _apply_alignment_padding(spec: MLAAttentionSpec | SlidingWindowMLASpec):
    if spec.alignment is None:
        return
    actual_page_size = spec.real_page_size_bytes
    padded_page_size = round_up(actual_page_size, spec.alignment)
    if padded_page_size != actual_page_size:
        object.__setattr__(spec, "page_size_padded", padded_page_size)
```
**EN:** This function implements `_apply_alignment_padding` within the module. Key calls include `round_up`, `__setattr__`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_apply_alignment_padding`，其作用域位于the module。 关键调用包括 `round_up`, `__setattr__`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TQFullAttentionSpec` class / `TQFullAttentionSpec` 类
```python
@dataclass(frozen=True, kw_only=True)
class TQFullAttentionSpec(FullAttentionSpec):
    """FullAttentionSpec with TQ-aware page size.

    Python equivalent of the C++ TQ4FullAttentionSpec. Overrides
    real_page_size_bytes to use TQ slot bytes instead of the raw
    head_size * dtype formula.
    """

    tq_slot_size: int = 0
```
**EN:** Introduces the `TQFullAttentionSpec` class on top of `FullAttentionSpec`. Core methods include `real_page_size_bytes`, `merge`. Docstring signal: FullAttentionSpec with TQ-aware page size.
**CN:** 这里定义 `TQFullAttentionSpec` 类，其基类包括 `FullAttentionSpec`。核心方法包括 `real_page_size_bytes`, `merge`。

### `TQFullAttentionSpec.real_page_size_bytes` method / `TQFullAttentionSpec.real_page_size_bytes` 方法
```python
    @property
    def real_page_size_bytes(self) -> int:
        if self.tq_slot_size > 0:
            return self.block_size * self.num_kv_heads * self.tq_slot_size
        return super().real_page_size_bytes
```
**EN:** This method implements `real_page_size_bytes` within `TQFullAttentionSpec`. Key calls include `super`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `real_page_size_bytes`，其作用域位于`TQFullAttentionSpec`。 关键调用包括 `super`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TQFullAttentionSpec.merge` method / `TQFullAttentionSpec.merge` 方法
```python
    @classmethod
    def merge(cls, specs: list[Self]) -> Self:
        merged = super().merge(specs)
        assert all(s.tq_slot_size == specs[0].tq_slot_size for s in specs), (
            "All TQ layers in the same KV cache group must use the same tq_slot_size."
        )
        return replace(merged, tq_slot_size=specs[0].tq_slot_size)
```
**EN:** This method implements `merge` within `TQFullAttentionSpec`. Key calls include `merge`, `all`, `replace`, `super`.
**CN:** 该方法会实现 `merge`，其作用域位于`TQFullAttentionSpec`。 关键调用包括 `merge`, `all`, `replace`, `super`。

### `MLAAttentionSpec` class / `MLAAttentionSpec` 类
```python
@dataclass(frozen=True, kw_only=True)
class MLAAttentionSpec(FullAttentionSpec):
    # TODO(Lucas/Chen): less hacky way to do this
    cache_dtype_str: str | None = None
    # DeepseekV4 only fields. Non-DeepseekV4 MLA models leave these at defaults.
    alignment: int | None = None  # Default to None for no padding.
    compress_ratio: int = 1  # Default to 1 for no compression.
    model_version: str | None = None
```
**EN:** Introduces the `MLAAttentionSpec` class on top of `FullAttentionSpec`. Core methods include `__post_init__`, `storage_block_size`, `real_page_size_bytes`, `merge`.
**CN:** 这里定义 `MLAAttentionSpec` 类，其基类包括 `FullAttentionSpec`。核心方法包括 `__post_init__`, `storage_block_size`, `real_page_size_bytes`, `merge`。

### `MLAAttentionSpec.storage_block_size` method / `MLAAttentionSpec.storage_block_size` 方法
```python
    @property
    def storage_block_size(self) -> int:
        return self.block_size // self.compress_ratio
```
**EN:** This method implements `storage_block_size` within `MLAAttentionSpec`.
**CN:** 该方法会实现 `storage_block_size`，其作用域位于`MLAAttentionSpec`。

### `MLAAttentionSpec.real_page_size_bytes` method / `MLAAttentionSpec.real_page_size_bytes` 方法
```python
    @property
    def real_page_size_bytes(self) -> int:
        if self.cache_dtype_str == "fp8_ds_mla":
            if self.model_version == "deepseek_v4":
                # DeepseekV4: 448B NoPE + 128B RoPE + 8B fp8 scale = 584B per token.
                # head_size stays semantic (512); bytes are determined here.
                return self.storage_block_size * 584
            # V3.2 main MLA: 656-byte custom layout (kv_lora_rank=512 +
            # qk_rope_head_dim=64, head_size=576). See flashmla_sparse.py.
            return self.block_size * 656
        return (
            self.storage_block_size
            * self.num_kv_heads
            * self.head_size
            * get_dtype_size(self.dtype)
        )
```
**EN:** This method implements `real_page_size_bytes` within `MLAAttentionSpec`. Key calls include `get_dtype_size`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `real_page_size_bytes`，其作用域位于`MLAAttentionSpec`。 关键调用包括 `get_dtype_size`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MLAAttentionSpec.merge` method / `MLAAttentionSpec.merge` 方法
```python
    @classmethod
    def merge(cls, specs: list[Self]) -> Self:
        assert all(isinstance(spec, MLAAttentionSpec) for spec in specs), (
            "All attention layers in the same KV cache group must be MLAAttentionSpec."
        )
        cache_dtype_str_set = set(spec.cache_dtype_str for spec in specs)
        compress_ratio_set = set(spec.compress_ratio for spec in specs)
        model_version_set = set(spec.model_version for spec in specs)
        assert (
            len(cache_dtype_str_set) == 1
            and len(compress_ratio_set) == 1
            and len(model_version_set) == 1
        ), (
            "All attention layers in the same KV cache group must use the same "
            "quantization method, compress ratio, and model version."
        )
        return cls(
            block_size=specs[0].block_size,
            num_kv_heads=specs[0].num_kv_heads,
            head_size=specs[0].head_size,
            dtype=specs[0].dtype,
            kv_quant_mode=specs[0].kv_quant_mode,
            page_size_padded=specs[0].page_size_padded,
            cache_dtype_str=cache_dtype_str_set.pop(),
            compress_ratio=compress_ratio_set.pop(),
            model_version=model_version_set.pop(),
        )
```
**EN:** This method implements `merge` within `MLAAttentionSpec`. Key calls include `all`, `set`, `cls`, `isinstance`, `len`, `pop`.
**CN:** 该方法会实现 `merge`，其作用域位于`MLAAttentionSpec`。 关键调用包括 `all`, `set`, `cls`, `isinstance`, `len`, `pop`。

### `HiddenStateCacheSpec` class / `HiddenStateCacheSpec` 类
```python
@dataclass(frozen=True, kw_only=True)
class HiddenStateCacheSpec(MLAAttentionSpec):
    """Marker for hidden-state cache layers used by extract_hidden_states."""

    pass
```
**EN:** Introduces the `HiddenStateCacheSpec` class on top of `MLAAttentionSpec`. Core methods include its methods defined below. Docstring signal: Marker for hidden-state cache layers used by extract_hidden_states.
**CN:** 这里定义 `HiddenStateCacheSpec` 类，其基类包括 `MLAAttentionSpec`。核心方法包括 下方定义的方法。

### `ChunkedLocalAttentionSpec` class / `ChunkedLocalAttentionSpec` 类
```python
@dataclass(frozen=True, kw_only=True)
class ChunkedLocalAttentionSpec(AttentionSpec):
    attention_chunk_size: int
```
**EN:** Introduces the `ChunkedLocalAttentionSpec` class on top of `AttentionSpec`. Core methods include `max_admission_blocks_per_request`, `max_memory_usage_bytes`.
**CN:** 这里定义 `ChunkedLocalAttentionSpec` 类，其基类包括 `AttentionSpec`。核心方法包括 `max_admission_blocks_per_request`, `max_memory_usage_bytes`。

### `ChunkedLocalAttentionSpec.max_admission_blocks_per_request` method / `ChunkedLocalAttentionSpec.max_admission_blocks_per_request` 方法
```python
    def max_admission_blocks_per_request(
        self, max_num_batched_tokens: int, max_model_len: int
    ) -> int:
        """Per-request admission cap, in blocks.

        Single source of truth for both startup pool sizing
        (`max_memory_usage_bytes`) and the runtime admission gate, so requests
        admitted by startup can also be admitted at runtime.
        """
        # During chunked prefill, we hold KV for at most one chunk window.
        num_tokens = min(
            self.attention_chunk_size + max_num_batched_tokens, max_model_len
        )
        return cdiv(num_tokens, self.block_size)
```
**EN:** This method implements `max_admission_blocks_per_request` within `ChunkedLocalAttentionSpec`. The docstring frames it as: Per-request admission cap, in blocks. Key calls include `min`, `cdiv`.
**CN:** 该方法会实现 `max_admission_blocks_per_request`，其作用域位于`ChunkedLocalAttentionSpec`。 关键调用包括 `min`, `cdiv`。

### `ChunkedLocalAttentionSpec.max_memory_usage_bytes` method / `ChunkedLocalAttentionSpec.max_memory_usage_bytes` 方法
```python
    def max_memory_usage_bytes(self, vllm_config: VllmConfig) -> int:
        max_model_len = vllm_config.model_config.max_model_len
        max_num_batched_tokens = vllm_config.scheduler_config.max_num_batched_tokens
        max_blocks = self.max_admission_blocks_per_request(
            max_num_batched_tokens=max_num_batched_tokens, max_model_len=max_model_len
        )
        return max_blocks * self.page_size_bytes
```
**EN:** This method implements `max_memory_usage_bytes` within `ChunkedLocalAttentionSpec`. Key calls include `max_admission_blocks_per_request`.
**CN:** 该方法会实现 `max_memory_usage_bytes`，其作用域位于`ChunkedLocalAttentionSpec`。 关键调用包括 `max_admission_blocks_per_request`。

### `SlidingWindowSpec` class / `SlidingWindowSpec` 类
```python
@dataclass(frozen=True, kw_only=True)
class SlidingWindowSpec(AttentionSpec):
    sliding_window: int
    head_size_v: int = None  # type: ignore[assignment]
```
**EN:** Introduces the `SlidingWindowSpec` class on top of `AttentionSpec`. Core methods include `__post_init__`, `real_page_size_bytes`, `max_admission_blocks_per_request`, `max_memory_usage_bytes`.
**CN:** 这里定义 `SlidingWindowSpec` 类，其基类包括 `AttentionSpec`。核心方法包括 `__post_init__`, `real_page_size_bytes`, `max_admission_blocks_per_request`, `max_memory_usage_bytes`。

### `SlidingWindowSpec.real_page_size_bytes` method / `SlidingWindowSpec.real_page_size_bytes` 方法
```python
    @property
    def real_page_size_bytes(self) -> int:
        # Mirror ``FullAttentionSpec.real_page_size_bytes`` for NVFP4 KV cache.
        if self.kv_quant_mode.is_nvfp4:
            last_dim = nvfp4_kv_cache_full_dim(
                self.head_size
            ) + nvfp4_kv_cache_full_dim(self.head_size_v)
            return (
                self.block_size
                * self.num_kv_heads
                * last_dim
                * get_dtype_size(self.dtype)
            )
        return (
            self.block_size
            * self.num_kv_heads
            * (self.head_size + self.head_size_v)
            * get_dtype_size(self.dtype)
        )
```
**EN:** This method implements `real_page_size_bytes` within `SlidingWindowSpec`. Key calls include `get_dtype_size`, `nvfp4_kv_cache_full_dim`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `real_page_size_bytes`，其作用域位于`SlidingWindowSpec`。 关键调用包括 `get_dtype_size`, `nvfp4_kv_cache_full_dim`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SlidingWindowSpec.max_admission_blocks_per_request` method / `SlidingWindowSpec.max_admission_blocks_per_request` 方法
```python
    def max_admission_blocks_per_request(
        self, max_num_batched_tokens: int, max_model_len: int
    ) -> int:
        """Per-request admission cap, in blocks.

        Single source of truth for both startup pool sizing
        (`max_memory_usage_bytes`) and the runtime admission gate. Per-request
        real-held blocks plateau at this bound because
        `SlidingWindowManager.remove_skipped_blocks` runs from `allocate_slots`
        before each chunk's `get_num_blocks_to_allocate`.
        """
        # During chunked prefill, we hold KV for the last `sliding_window-1`
        # computed tokens plus the newly scheduled tokens, and never more
        # than `max_model_len`.
        num_tokens = min(
            self.sliding_window - 1 + max_num_batched_tokens, max_model_len
        )
        # +1 because the sliding window may not start from the beginning of
        # the block. E.g. block size 4 and num_token 4 needs two blocks
        # [XXCD][EF] to store the 6-token window [CDEF].
        return cdiv(num_tokens, self.block_size) + 1
```
**EN:** This method implements `max_admission_blocks_per_request` within `SlidingWindowSpec`. The docstring frames it as: Per-request admission cap, in blocks. Key calls include `min`, `cdiv`.
**CN:** 该方法会实现 `max_admission_blocks_per_request`，其作用域位于`SlidingWindowSpec`。 关键调用包括 `min`, `cdiv`。

### `SlidingWindowSpec.max_memory_usage_bytes` method / `SlidingWindowSpec.max_memory_usage_bytes` 方法
```python
    def max_memory_usage_bytes(self, vllm_config: VllmConfig) -> int:
        assert vllm_config.parallel_config.decode_context_parallel_size == 1, (
            "DCP not support sliding window."
        )
        max_model_len = vllm_config.model_config.max_model_len
        max_num_batched_tokens = vllm_config.scheduler_config.max_num_batched_tokens
        max_blocks = self.max_admission_blocks_per_request(
            max_num_batched_tokens=max_num_batched_tokens, max_model_len=max_model_len
        )
        return max_blocks * self.page_size_bytes
```
**EN:** This method implements `max_memory_usage_bytes` within `SlidingWindowSpec`. Key calls include `max_admission_blocks_per_request`.
**CN:** 该方法会实现 `max_memory_usage_bytes`，其作用域位于`SlidingWindowSpec`。 关键调用包括 `max_admission_blocks_per_request`。

### `SlidingWindowMLASpec` class / `SlidingWindowMLASpec` 类
```python
@dataclass(frozen=True, kw_only=True)
class SlidingWindowMLASpec(SlidingWindowSpec):
    """Sliding window attention with MLA cache format."""

    cache_dtype_str: str | None = None
    # DeepseekV4-only: see MLAAttentionSpec.model_version.
    alignment: int | None = None  # Default to None for no padding.
    compress_ratio: int = 1
    model_version: str | None = None
```
**EN:** Introduces the `SlidingWindowMLASpec` class on top of `SlidingWindowSpec`. Core methods include `__post_init__`, `storage_block_size`, `real_page_size_bytes`, `merge`. Docstring signal: Sliding window attention with MLA cache format.
**CN:** 这里定义 `SlidingWindowMLASpec` 类，其基类包括 `SlidingWindowSpec`。核心方法包括 `__post_init__`, `storage_block_size`, `real_page_size_bytes`, `merge`。

### `SlidingWindowMLASpec.storage_block_size` method / `SlidingWindowMLASpec.storage_block_size` 方法
```python
    @property
    def storage_block_size(self) -> int:
        return self.block_size // self.compress_ratio
```
**EN:** This method implements `storage_block_size` within `SlidingWindowMLASpec`.
**CN:** 该方法会实现 `storage_block_size`，其作用域位于`SlidingWindowMLASpec`。

### `SlidingWindowMLASpec.real_page_size_bytes` method / `SlidingWindowMLASpec.real_page_size_bytes` 方法
```python
    @property
    def real_page_size_bytes(self) -> int:
        if self.model_version == "deepseek_v4":
            # DeepseekV4: 448B NoPE + 128B RoPE + 8B fp8 scale = 584B per token.
            return self.storage_block_size * 584
        assert self.model_version is None, (
            f"Unsupported model version: {self.model_version}"
        )
        return (
            self.storage_block_size
            * self.num_kv_heads
            * self.head_size
            * get_dtype_size(self.dtype)
        )
```
**EN:** This method implements `real_page_size_bytes` within `SlidingWindowMLASpec`. Key calls include `get_dtype_size`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `real_page_size_bytes`，其作用域位于`SlidingWindowMLASpec`。 关键调用包括 `get_dtype_size`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SlidingWindowMLASpec.merge` method / `SlidingWindowMLASpec.merge` 方法
```python
    @classmethod
    def merge(cls, specs: list[Self]) -> Self:
        assert all(isinstance(spec, SlidingWindowMLASpec) for spec in specs), (
            "All attention layers in the same KV cache group must be "
            "SlidingWindowMLASpec."
        )
        cache_dtype_str_set = set(spec.cache_dtype_str for spec in specs)
        compress_ratio_set = set(spec.compress_ratio for spec in specs)
        model_version_set = set(spec.model_version for spec in specs)
        sliding_window_set = set(spec.sliding_window for spec in specs)
        assert (
            len(cache_dtype_str_set) == 1
            and len(compress_ratio_set) == 1
            and len(model_version_set) == 1
            and len(sliding_window_set) == 1
        ), (
            "All attention layers in the same KV cache group must use the same "
            "quantization method, compress ratio, model version and sliding "
            "window size."
        )
        return cls(
            block_size=specs[0].block_size,
            num_kv_heads=specs[0].num_kv_heads,
            head_size=specs[0].head_size,
            dtype=specs[0].dtype,
            page_size_padded=specs[0].page_size_padded,
            sliding_window=sliding_window_set.pop(),
            cache_dtype_str=cache_dtype_str_set.pop(),
            compress_ratio=compress_ratio_set.pop(),
            model_version=model_version_set.pop(),
        )
```
**EN:** This method implements `merge` within `SlidingWindowMLASpec`. Key calls include `all`, `set`, `cls`, `isinstance`, `len`, `pop`.
**CN:** 该方法会实现 `merge`，其作用域位于`SlidingWindowMLASpec`。 关键调用包括 `all`, `set`, `cls`, `isinstance`, `len`, `pop`。

### `MambaSpec` class / `MambaSpec` 类
```python
@dataclass(frozen=True)
class MambaSpec(KVCacheSpec):
    shapes: tuple[tuple[int, ...], ...]
    dtypes: tuple[torch.dtype]
    page_size_padded: int | None = None
    mamba_type: MambaAttentionBackendEnum = MambaAttentionBackendEnum.MAMBA2
    mamba_cache_mode: str = "none"
    num_speculative_blocks: int = 0
```
**EN:** Introduces the `MambaSpec` class on top of `KVCacheSpec`. Core methods include `page_size_bytes`, `max_memory_usage_bytes`.
**CN:** 这里定义 `MambaSpec` 类，其基类包括 `KVCacheSpec`。核心方法包括 `page_size_bytes`, `max_memory_usage_bytes`。

### `MambaSpec.page_size_bytes` method / `MambaSpec.page_size_bytes` 方法
```python
    @property
    def page_size_bytes(self) -> int:
        page_size = sum(
            prod(shape) * get_dtype_size(dtype)
            for (shape, dtype) in zip(self.shapes, self.dtypes)
        )
        if self.page_size_padded is not None:
            assert self.page_size_padded >= page_size
            return self.page_size_padded
        return page_size
```
**EN:** This method implements `page_size_bytes` within `MambaSpec`. Key calls include `sum`, `prod`, `get_dtype_size`, `zip`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `page_size_bytes`，其作用域位于`MambaSpec`。 关键调用包括 `sum`, `prod`, `get_dtype_size`, `zip`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MambaSpec.max_memory_usage_bytes` method / `MambaSpec.max_memory_usage_bytes` 方法
```python
    def max_memory_usage_bytes(self, vllm_config: VllmConfig) -> int:
        if vllm_config.cache_config.mamba_cache_mode == "all":
            max_model_len = vllm_config.model_config.max_model_len
            return (
                cdiv(max_model_len, self.block_size) + self.num_speculative_blocks
            ) * self.page_size_bytes
        elif vllm_config.cache_config.mamba_cache_mode == "align":
            return self.page_size_bytes * (2 + self.num_speculative_blocks)
        else:
            return self.page_size_bytes * (1 + self.num_speculative_blocks)
```
**EN:** This method implements `max_memory_usage_bytes` within `MambaSpec`. Key calls include `cdiv`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `max_memory_usage_bytes`，其作用域位于`MambaSpec`。 关键调用包括 `cdiv`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EncoderOnlyAttentionSpec` class / `EncoderOnlyAttentionSpec` 类
```python
@dataclass(frozen=True)
class EncoderOnlyAttentionSpec(AttentionSpec):
```
**EN:** Introduces the `EncoderOnlyAttentionSpec` class on top of `AttentionSpec`. Core methods include `max_memory_usage_bytes`.
**CN:** 这里定义 `EncoderOnlyAttentionSpec` 类，其基类包括 `AttentionSpec`。核心方法包括 `max_memory_usage_bytes`。

### `EncoderOnlyAttentionSpec.max_memory_usage_bytes` method / `EncoderOnlyAttentionSpec.max_memory_usage_bytes` 方法
```python
    def max_memory_usage_bytes(self, vllm_config: VllmConfig) -> int:
        # Encoder-only layers do not need KV cache
        return 0
```
**EN:** This method implements `max_memory_usage_bytes` within `EncoderOnlyAttentionSpec`.
**CN:** 该方法会实现 `max_memory_usage_bytes`，其作用域位于`EncoderOnlyAttentionSpec`。

### `CrossAttentionSpec` class / `CrossAttentionSpec` 类
```python
@dataclass(frozen=True)
class CrossAttentionSpec(AttentionSpec):
    """
    KV cache spec for cross-attention layers in encoder-decoder models.
    """
```
**EN:** Introduces the `CrossAttentionSpec` class on top of `AttentionSpec`. Core methods include `max_memory_usage_bytes`. Docstring signal: KV cache spec for cross-attention layers in encoder-decoder models.
**CN:** 这里定义 `CrossAttentionSpec` 类，其基类包括 `AttentionSpec`。核心方法包括 `max_memory_usage_bytes`。

### `CrossAttentionSpec.max_memory_usage_bytes` method / `CrossAttentionSpec.max_memory_usage_bytes` 方法
```python
    def max_memory_usage_bytes(self, vllm_config: VllmConfig) -> int:
        # For cross-attention, we need to cache encoder states
        # Get encoder length (e.g., 1500 for Whisper).
        max_encoder_len = vllm_config.scheduler_config.max_num_encoder_input_tokens
        return cdiv(max_encoder_len, self.block_size) * self.page_size_bytes
```
**EN:** This method implements `max_memory_usage_bytes` within `CrossAttentionSpec`. Key calls include `cdiv`.
**CN:** 该方法会实现 `max_memory_usage_bytes`，其作用域位于`CrossAttentionSpec`。 关键调用包括 `cdiv`。

### `SinkFullAttentionSpec` class / `SinkFullAttentionSpec` 类
```python
@dataclass(frozen=True)
class SinkFullAttentionSpec(FullAttentionSpec):
    sink_len: int | None = None
```
**EN:** Introduces the `SinkFullAttentionSpec` class on top of `FullAttentionSpec`. Core methods include `merge`.
**CN:** 这里定义 `SinkFullAttentionSpec` 类，其基类包括 `FullAttentionSpec`。核心方法包括 `merge`。

### `SinkFullAttentionSpec.merge` method / `SinkFullAttentionSpec.merge` 方法
```python
    @classmethod
    def merge(cls, specs: list[Self]) -> Self:
        """
        Merge a list of FullAttentionSpec objects into a single
        FullAttentionSpec object.
        """
        assert all(isinstance(spec, FullAttentionSpec) for spec in specs), (
            "All attention layers in the same KV cache group must be FullAttentionSpec."
        )

        sliding_window = set(
            spec.sliding_window for spec in specs if spec.sliding_window is not None
        )
        attention_chunk_size = set(
            spec.attention_chunk_size
            for spec in specs
            if spec.attention_chunk_size is not None
        )
        assert not any(isinstance(spec, MLAAttentionSpec) for spec in specs), (
            "MLAAttentionSpec should be merged in MLAAttentionSpec.merge"
        )
        merged_spec = cls(
            block_size=specs[0].block_size,
            num_kv_heads=specs[0].num_kv_heads,
            head_size=specs[0].head_size,
            head_size_v=specs[0].head_size_v,
            sink_len=specs[0].sink_len,
            dtype=specs[0].dtype,
            kv_quant_mode=specs[0].kv_quant_mode,
            page_size_padded=specs[0].page_size_padded,
            sliding_window=cls.merge_window_sizes(sliding_window),
            attention_chunk_size=cls.merge_window_sizes(attention_chunk_size),
        )
        for spec in specs:
            for f in fields(AttentionSpec):
                assert getattr(spec, f.name) == getattr(merged_spec, f.name), (
                    "All attention layers in the same KV cache group must have "
                    "the same attention spec."
                )
        assert (merged_spec.sliding_window is not None) + (
            merged_spec.attention_chunk_size is not None
        ) <= 1, (
            "Model with both sliding window layers and chunked local attention "
            "layers is not supported."
        )
        return merged_spec
```
**EN:** This method implements `merge` within `SinkFullAttentionSpec`. The docstring frames it as: Merge a list of FullAttentionSpec objects into a single FullAttentionSpec object. Key calls include `all`, `set`, `cls`, `any`, `fields`, `isinstance`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `merge`，其作用域位于`SinkFullAttentionSpec`。 关键调用包括 `all`, `set`, `cls`, `any`, `fields`, `isinstance`。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `UniformTypeKVCacheSpecs` class / `UniformTypeKVCacheSpecs` 类
```python
@dataclass(frozen=True)
class UniformTypeKVCacheSpecs(KVCacheSpec):
    """
    A KV cache spec for multiple layers with the same type of attention. Here,
    same types means always need the same number of token slots. For example,
    sliding window attentions with different window sizes are not the same type
    and should not be merged into one UniformTypeKVCacheSpecs.
    """

    kv_cache_specs: dict[str, KVCacheSpec]
```
**EN:** Introduces the `UniformTypeKVCacheSpecs` class on top of `KVCacheSpec`. Core methods include `page_size_bytes`, `max_memory_usage_bytes`, `is_uniform_type`, `from_specs`, `get_page_sizes`, `get_num_layer_tuples`. Docstring signal: A KV cache spec for multiple layers with the same type of attention.
**CN:** 这里定义 `UniformTypeKVCacheSpecs` 类，其基类包括 `KVCacheSpec`。核心方法包括 `page_size_bytes`, `max_memory_usage_bytes`, `is_uniform_type`, `from_specs`, `get_page_sizes`, `get_num_layer_tuples`。

### `UniformTypeKVCacheSpecs.max_memory_usage_bytes` method / `UniformTypeKVCacheSpecs.max_memory_usage_bytes` 方法
```python
    def max_memory_usage_bytes(self, vllm_config: VllmConfig) -> int:
        max_num_pages = max(
            cdiv(spec.max_memory_usage_bytes(vllm_config), spec.page_size_bytes)
            for spec in self.kv_cache_specs.values()
        )
        return max_num_pages * self.page_size_bytes
```
**EN:** This method implements `max_memory_usage_bytes` within `UniformTypeKVCacheSpecs`. Key calls include `max`, `cdiv`, `max_memory_usage_bytes`, `values`.
**CN:** 该方法会实现 `max_memory_usage_bytes`，其作用域位于`UniformTypeKVCacheSpecs`。 关键调用包括 `max`, `cdiv`, `max_memory_usage_bytes`, `values`。

### `UniformTypeKVCacheSpecs.is_uniform_type` method / `UniformTypeKVCacheSpecs.is_uniform_type` 方法
```python
    @classmethod
    def is_uniform_type(cls, kv_cache_specs: dict[str, KVCacheSpec]) -> bool:
        """
        Whether all layers have the same type of KV cache spec.
        """
        block_sizes = set(spec.block_size for spec in kv_cache_specs.values())
        if len(block_sizes) > 1:
            # Different block sizes, not uniform.
            return False
        one_spec = next(iter(kv_cache_specs.values()))
        # NOTE: Check subclasses before parent classes since isinstance()
        # returns True for subclasses.
        if isinstance(one_spec, SlidingWindowMLASpec):
            # SlidingWindowMLASpec is uniform if all specs are SlidingWindowMLASpec
            # with the same sliding_window size.
            return all(
                isinstance(spec, SlidingWindowMLASpec)
                and spec.sliding_window == one_spec.sliding_window
                for spec in kv_cache_specs.values()
            )
        elif isinstance(one_spec, FullAttentionSpec):
            return all(
                isinstance(spec, FullAttentionSpec) for spec in kv_cache_specs.values()
            )
        elif isinstance(one_spec, CrossAttentionSpec):
            return all(
                isinstance(spec, CrossAttentionSpec) for spec in kv_cache_specs.values()
            )
        elif isinstance(one_spec, SlidingWindowSpec):
            return all(
                isinstance(spec, SlidingWindowSpec)
                and spec.sliding_window == one_spec.sliding_window
                for spec in kv_cache_specs.values()
            )
        elif isinstance(one_spec, ChunkedLocalAttentionSpec):
            return all(
                isinstance(spec, ChunkedLocalAttentionSpec)
                and spec.attention_chunk_size == one_spec.attention_chunk_size
                for spec in kv_cache_specs.values()
            )
        elif isinstance(one_spec, MambaSpec):
            return all(
                isinstance(spec, MambaSpec)
                and spec.num_speculative_blocks == one_spec.num_speculative_blocks
                for spec in kv_cache_specs.values()
            )
        else:
            # NOTE(Chen): Please add new branches for new KV cache spec types.
            raise NotImplementedError(
                f"Unsupported KV cache spec type: {type(one_spec)}"
            )
```
**EN:** This method answers a boolean capability check within `UniformTypeKVCacheSpecs`. The docstring frames it as: Whether all layers have the same type of KV cache spec. Key calls include `set`, `next`, `isinstance`, `len`, `iter`, `all`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会回答布尔能力判断，其作用域位于`UniformTypeKVCacheSpecs`。 关键调用包括 `set`, `next`, `isinstance`, `len`, `iter`, `all`。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UniformTypeKVCacheSpecs.from_specs` method / `UniformTypeKVCacheSpecs.from_specs` 方法
```python
    @classmethod
    def from_specs(cls, kv_cache_specs: dict[str, KVCacheSpec]) -> Self | None:
        """
        Return a SameTypeKVCacheSpecs object if all layers have the same type
        of KV cache spec. Return None if not.
        """
        if cls.is_uniform_type(kv_cache_specs):
            block_size = next(iter(kv_cache_specs.values())).block_size
            return cls(block_size=block_size, kv_cache_specs=kv_cache_specs)
        else:
            return None
```
**EN:** This method reconstructs data from another representation within `UniformTypeKVCacheSpecs`. The docstring frames it as: Return a SameTypeKVCacheSpecs object if all layers have the same type of KV cache spec. Key calls include `is_uniform_type`, `cls`, `next`, `iter`, `values`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`UniformTypeKVCacheSpecs`。 关键调用包括 `is_uniform_type`, `cls`, `next`, `iter`, `values`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_kv_cache_spec_kind` function / `get_kv_cache_spec_kind` 函数
```python
def get_kv_cache_spec_kind(kv_cache_spec: KVCacheSpec) -> KVCacheSpecKind:
    if isinstance(kv_cache_spec, UniformTypeKVCacheSpecs):
        inner_kinds = {
            get_kv_cache_spec_kind(spec)
            for spec in kv_cache_spec.kv_cache_specs.values()
        }
        if len(inner_kinds) == 1:
            return next(iter(inner_kinds))
        return KVCacheSpecKind.UNKNOWN
    # Keep subclass checks before base classes so specialized specs keep their
    # more precise kind.
    if isinstance(kv_cache_spec, SlidingWindowMLASpec):
        return KVCacheSpecKind.SLIDING_WINDOW_MLA
    if isinstance(kv_cache_spec, MLAAttentionSpec):
        return KVCacheSpecKind.MLA_ATTENTION
    if isinstance(kv_cache_spec, SinkFullAttentionSpec):
        return KVCacheSpecKind.SINK_FULL_ATTENTION
    if isinstance(kv_cache_spec, FullAttentionSpec):
        return KVCacheSpecKind.FULL_ATTENTION
    if isinstance(kv_cache_spec, ChunkedLocalAttentionSpec):
        return KVCacheSpecKind.CHUNKED_LOCAL_ATTENTION
    if isinstance(kv_cache_spec, SlidingWindowSpec):
        return KVCacheSpecKind.SLIDING_WINDOW
    if isinstance(kv_cache_spec, MambaSpec):
        return KVCacheSpecKind.MAMBA
    if isinstance(kv_cache_spec, EncoderOnlyAttentionSpec):
        return KVCacheSpecKind.ENCODER_ONLY_ATTENTION
    if isinstance(kv_cache_spec, CrossAttentionSpec):
        return KVCacheSpecKind.CROSS_ATTENTION
    return KVCacheSpecKind.UNKNOWN
```
**EN:** This function returns or derives a value within the module. Key calls include `isinstance`, `get_kv_cache_spec_kind`, `len`, `next`, `values`, `iter`. The control flow contains 11 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `isinstance`, `get_kv_cache_spec_kind`, `len`, `next`, `values`, `iter`。 控制流包含 11 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_kv_cache_spec_sliding_window` function / `get_kv_cache_spec_sliding_window` 函数
```python
def get_kv_cache_spec_sliding_window(kv_cache_spec: KVCacheSpec) -> int | None:
    if isinstance(kv_cache_spec, UniformTypeKVCacheSpecs):
        inner_windows = {
            get_kv_cache_spec_sliding_window(spec)
            for spec in kv_cache_spec.kv_cache_specs.values()
        }
        return next(iter(inner_windows)) if len(inner_windows) == 1 else None
    if isinstance(kv_cache_spec, SlidingWindowSpec):
        return kv_cache_spec.sliding_window
    return None
```
**EN:** This function returns or derives a value within the module. Key calls include `isinstance`, `get_kv_cache_spec_sliding_window`, `next`, `values`, `len`, `iter`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `isinstance`, `get_kv_cache_spec_sliding_window`, `next`, `values`, `len`, `iter`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheTensor` class / `KVCacheTensor` 类
```python
@dataclass
class KVCacheTensor:
    """
    A class for specifying how the workers should initialize the KV cache.
    """

    size: int  # size of the KV cache tensor in bytes
    shared_by: list[str]  # layer names that share the same KV cache tensor
```
**EN:** Uses `@dataclass` to package related state for `KVCacheTensor`. Typical fields include `size`, `shared_by`.
**CN:** `KVCacheTensor` 使用 `@dataclass` 打包相关状态。典型字段包括 `size`, `shared_by`。

### `KVCacheGroupSpec` class / `KVCacheGroupSpec` 类
```python
@dataclass
class KVCacheGroupSpec:
    """
    Represents a group of model layers that share the same KV cache block table.
    These layers are regarded as one layer in the KV cache manager.
    """

    # The names of model layers in this group
    layer_names: list[str]
    # The KV cache spec of this manager layer
    kv_cache_spec: KVCacheSpec
    # Whether this group contains EAGLE/MTP draft attention layers.
    is_eagle_group: bool = False
```
**EN:** Uses `@dataclass` to package related state for `KVCacheGroupSpec`. Typical fields include `layer_names`, `kv_cache_spec`, `is_eagle_group`.
**CN:** `KVCacheGroupSpec` 使用 `@dataclass` 打包相关状态。典型字段包括 `layer_names`, `kv_cache_spec`, `is_eagle_group`。

### `KVCacheConfig` class / `KVCacheConfig` 类
```python
@dataclass
class KVCacheConfig:
    """
    The KV cache configuration of a model.
    """

    num_blocks: int
    """The number of KV cache blocks"""
    kv_cache_tensors: list[KVCacheTensor]
    """How should model runner initialize the KV cache tensors for each layer"""
    kv_cache_groups: list[KVCacheGroupSpec]
    """
    The kv cache groups of the model.
    For models with only one type of attention, there is only one group that
    contains all layers.
    For models with multiple types of attention, there will be multiple groups,
    see `_get_kv_cache_config_uniform_page_size` for more details.
    """
```
**EN:** Uses `@dataclass` to package related state for `KVCacheConfig`. Typical fields include `num_blocks`, `kv_cache_tensors`, `kv_cache_groups`.
**CN:** `KVCacheConfig` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_blocks`, `kv_cache_tensors`, `kv_cache_groups`。

### `KVCacheConfig.has_mamba_layers` method / `KVCacheConfig.has_mamba_layers` 方法
```python
    @property
    def has_mamba_layers(self) -> bool:
        return any(isinstance(g.kv_cache_spec, MambaSpec) for g in self.kv_cache_groups)
```
**EN:** This method implements `has_mamba_layers` within `KVCacheConfig`. Key calls include `any`, `isinstance`.
**CN:** 该方法会实现 `has_mamba_layers`，其作用域位于`KVCacheConfig`。 关键调用包括 `any`, `isinstance`。

### `KVCacheConfig.needs_kv_cache_zeroing` method / `KVCacheConfig.needs_kv_cache_zeroing` 方法
```python
    @property
    def needs_kv_cache_zeroing(self) -> bool:
        return self.has_mamba_layers
```
**EN:** This method implements `needs_kv_cache_zeroing` within `KVCacheConfig`.
**CN:** 该方法会实现 `needs_kv_cache_zeroing`，其作用域位于`KVCacheConfig`。

## Key Concepts / 关键概念
- `KVQuantMode`: central class or interface in this module. / `KVQuantMode`：本模块中的核心类或接口。
- `get_kv_quant_mode`: top-level helper or orchestration entry point. / `get_kv_quant_mode`：顶层辅助函数或编排入口。
- `is_quantized_kv_cache`: top-level helper or orchestration entry point. / `is_quantized_kv_cache`：顶层辅助函数或编排入口。
- `kv_cache_uses_per_token_head_scales`: top-level helper or orchestration entry point. / `kv_cache_uses_per_token_head_scales`：顶层辅助函数或编排入口。
- `KVCacheSpecKind`: central class or interface in this module. / `KVCacheSpecKind`：本模块中的核心类或接口。
- `KVCacheSpec`: central class or interface in this module. / `KVCacheSpec`：本模块中的核心类或接口。
- `AttentionSpec`: central class or interface in this module. / `AttentionSpec`：本模块中的核心类或接口。
- `FullAttentionSpec`: central class or interface in this module. / `FullAttentionSpec`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `__future__`, `copy`, `collections`, `dataclasses`, `enum`, `math`, `typing`
- External / 外部依赖: `torch`, `typing_extensions`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.utils.math_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backends.registry`, `vllm.config`
