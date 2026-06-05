# cross_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/attention/cross_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: attention execution, backend dispatch, and KV-cache handling / 注意力执行、后端分派与 KV 缓存处理

## Line-by-Line Analysis / 逐行分析
### Lines 3-27 — imports and setup
```python
import functools
from copy import copy

import numpy as np
import torch

from vllm.config import CacheConfig, VllmConfig
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import Attention
from vllm.utils.math_utils import cdiv
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionMetadata,
    AttentionType,
    CommonAttentionMetadata,
    subclass_attention_backend_with_overrides,
)
from vllm.v1.attention.selector import get_attn_backend
from vllm.v1.kv_cache_interface import (
    CrossAttentionSpec,
    KVCacheSpec,
    get_kv_quant_mode,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 30-70 — function `_get_cross_slot_mapping`
```python
def _get_cross_slot_mapping(
    encoder_seq_lens: np.ndarray,
    block_table_tensor: torch.Tensor,
    kv_cache_spec: CrossAttentionSpec,
    device: torch.device,
) -> torch.Tensor:
    """Get cross-attention slot mappings."""

    block_size = kv_cache_spec.block_size
    slot_mappings = []

    # Find indices with non-zero encoder sequence lengths
    # The majority of parallel requests will be running the
    # decoder, so this list should be relatively small.
    active_indices = np.nonzero(encoder_seq_lens)[0]

    for req_index in active_indices:
        encoder_seq_len = encoder_seq_lens[req_index].item()

        # Calculate the number of blocks needed for this request
        num_blocks_needed = cdiv(encoder_seq_len, block_size)

        # Get the block IDs for this request from the tensor
        req_block_ids = block_table_tensor[req_index]

        # Get only the blocks we need (first num_blocks_needed blocks)
        needed_block_ids = req_block_ids[:num_blocks_needed]

        # All needed blocks are allocated
        i_values = torch.arange(encoder_seq_len, dtype=torch.int64, device=device)
        block_indices = i_values // block_size
        block_offsets = i_values % block_size
        block_numbers = needed_block_ids[block_indices]
        slot_mapping = block_numbers * block_size + block_offsets

        slot_mappings.append(slot_mapping)

    if slot_mappings:
        return torch.cat(slot_mappings)
    else:
        return torch.empty(0, dtype=torch.int64, device=device)
```
**EN:** This function defines `_get_cross_slot_mapping`. Get cross-attention slot mappings. The main inputs are `encoder_seq_lens`, `block_table_tensor`, `kv_cache_spec`, `device`. Key calls include `np.nonzero`, `encoder_seq_lens.item`, `cdiv`, `torch.arange`, `slot_mappings.append`, `torch.cat`. It writes or updates `block_size`, `slot_mappings`, `active_indices`, `encoder_seq_len`, `num_blocks_needed`, `req_block_ids`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `_get_cross_slot_mapping`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `encoder_seq_lens`, `block_table_tensor`, `kv_cache_spec`, `device`。 关键调用包括 `np.nonzero`, `encoder_seq_lens.item`, `cdiv`, `torch.arange`, `slot_mappings.append`, `torch.cat`。 它会写入或更新 `block_size`, `slot_mappings`, `active_indices`, `encoder_seq_len`, `num_blocks_needed`, `req_block_ids`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 73-186 — function `create_cross_attention_backend`
```python
@functools.lru_cache
def create_cross_attention_backend(
    underlying_attn_backend: type[AttentionBackend],
) -> type[AttentionBackend]:
    prefix = "CrossAttention_"
    underlying_builder = underlying_attn_backend.get_builder_cls()
    underlying_impl = underlying_attn_backend.get_impl_cls()

    class CrossAttentionBuilder(underlying_builder):  # type: ignore
        def build(
            self,
            common_prefix_len: int,
            common_attn_metadata: CommonAttentionMetadata,
            fast_build: bool = False,
        ) -> AttentionMetadata:
            new_metadata = copy(common_attn_metadata)
            new_metadata.causal = False
            assert new_metadata.encoder_seq_lens_cpu is not None
            max_encoder_len = int(new_metadata.encoder_seq_lens_cpu.max())
            new_metadata.max_seq_len = max_encoder_len
            # Any computed tokens indicates decode step>1 (no chunked prefill).
            # The upper bound is exact for this `> 0` test - prefill rows have
            # num_computed == 0 and decode rows have num_computed > 0.
            query_lens_cpu = (
                common_attn_metadata.query_start_loc_cpu[1:]
                - common_attn_metadata.query_start_loc_cpu[:-1]
            )
            assert common_attn_metadata.seq_lens_cpu_upper_bound is not None
            num_computed_tokens_cpu = (
                common_attn_metadata.seq_lens_cpu_upper_bound - query_lens_cpu
            )
            num_cache_decodes = (num_computed_tokens_cpu > 0).sum().item()
            if num_cache_decodes > 0:
                # CrossAttn KV cache has already been populated on first decoder step,
                # skip slot_mapping calculation for requests that do not need
                # reshape_and_cache.
                num_tokens = num_computed_tokens_cpu.numpy()
                new_metadata.encoder_seq_lens_cpu = np.where(
                    num_tokens > 0, 0, new_metadata.encoder_seq_lens_cpu
                )

            # seq_lens is provided by model runner: initial encoder input length is
            # needed here to know how many tokens to attend to from the cached
            # cross-attention KV cache.
# ... omitted for brevity ...

    return attn_backend
```
**EN:** This function defines `create_cross_attention_backend`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `underlying_attn_backend`. Key calls include `underlying_attn_backend.get_builder_cls`, `underlying_attn_backend.get_impl_cls`, `subclass_attention_backend_with_overrides`, `copy`, `int`, `sum.item`. It writes or updates `prefix`, `underlying_builder`, `underlying_impl`, `attn_backend`, `new_metadata`, `causal`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `create_cross_attention_backend`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `underlying_attn_backend`。 关键调用包括 `underlying_attn_backend.get_builder_cls`, `underlying_attn_backend.get_impl_cls`, `subclass_attention_backend_with_overrides`, `copy`, `int`, `sum.item`。 它会写入或更新 `prefix`, `underlying_builder`, `underlying_impl`, `attn_backend`, `new_metadata`, `causal`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 189-241 — class `CrossAttention`
```python
class CrossAttention(Attention):
    """
    Cross-attention for encoder-decoder models.
    Handles attention between decoder queries and encoder keys/values.
    """

    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        cache_config: CacheConfig | None = None,
        attn_type: str | None = None,
        **kwargs,
# ... omitted for brevity ...
            kv_quant_mode=get_kv_quant_mode(self.kv_cache_dtype),
        )
```
**EN:** This class defines `CrossAttention`. It inherits from `Attention`. Cross-attention for encoder-decoder models. Important methods include `__init__`, `get_kv_cache_spec`. Key calls include `torch.get_default_dtype`, `get_attn_backend`, `create_cross_attention_backend`, `super.__init__`, `CrossAttentionSpec`, `super`. It writes or updates `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `CrossAttention`。 它继承自 `Attention`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `get_kv_cache_spec`。 关键调用包括 `torch.get_default_dtype`, `get_attn_backend`, `create_cross_attention_backend`, `super.__init__`, `CrossAttentionSpec`, `super`。 它会写入或更新 `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 195-232 — method `CrossAttention.__init__`
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        cache_config: CacheConfig | None = None,
        attn_type: str | None = None,
        **kwargs,
    ):
        dtype = torch.get_default_dtype()

        if cache_config is not None:
            kv_cache_dtype = cache_config.cache_dtype
        else:
            kv_cache_dtype = "auto"

        if attn_type is not None:
            assert attn_type == AttentionType.ENCODER_DECODER, (
                "CrossAttention only supports AttentionType.ENCODER_DECODER"
            )

        underlying_attn_backend = get_attn_backend(
            head_size,
            dtype,
            kv_cache_dtype,
            attn_type=AttentionType.ENCODER_DECODER,
        )
        attn_backend = create_cross_attention_backend(underlying_attn_backend)

        super().__init__(
            num_heads=num_heads,
            head_size=head_size,
            scale=scale,
            cache_config=cache_config,
            attn_backend=attn_backend,
            attn_type=AttentionType.ENCODER_DECODER,
            **kwargs,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `num_heads`, `head_size`, `scale`, `cache_config`, `attn_type`, `**kwargs`. Key calls include `torch.get_default_dtype`, `get_attn_backend`, `create_cross_attention_backend`, `super.__init__`, `super`. It writes or updates `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `num_heads`, `head_size`, `scale`, `cache_config`, `attn_type`, `**kwargs`。 关键调用包括 `torch.get_default_dtype`, `get_attn_backend`, `create_cross_attention_backend`, `super.__init__`, `super`。 它会写入或更新 `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 234-241 — method `CrossAttention.get_kv_cache_spec`
```python
    def get_kv_cache_spec(self, vllm_config: VllmConfig) -> KVCacheSpec:
        return CrossAttentionSpec(
            block_size=vllm_config.cache_config.block_size,
            num_kv_heads=self.num_kv_heads,
            head_size=self.head_size,
            dtype=self.kv_cache_torch_dtype,
            kv_quant_mode=get_kv_quant_mode(self.kv_cache_dtype),
        )
```
**EN:** This method defines `get_kv_cache_spec`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `vllm_config`. Key calls include `CrossAttentionSpec`, `get_kv_quant_mode`.
**CN:** 该方法定义 `get_kv_cache_spec`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `vllm_config`。 关键调用包括 `CrossAttentionSpec`, `get_kv_quant_mode`。

## Key Concepts / 关键概念
- [EN] Attention execution, backend dispatch, and kv-cache handling / [CN] 注意力执行、后端分派与 KV 缓存处理
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `_get_cross_slot_mapping`, `create_cross_attention_backend`, `CrossAttention` / [CN] 核心符号：`_get_cross_slot_mapping`, `create_cross_attention_backend`, `CrossAttention`

## Dependencies / 依赖关系
- **External**: `functools`, `copy`, `numpy`, `torch` / **外部依赖**: `functools`, `copy`, `numpy`, `torch`
- **Internal**: `vllm.config`, `vllm.logger`, `vllm.model_executor.layers.attention`, `vllm.utils.math_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.selector`, `vllm.v1.kv_cache_interface` / **内部依赖**: `vllm.config`, `vllm.logger`, `vllm.model_executor.layers.attention`, `vllm.utils.math_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.selector`, `vllm.v1.kv_cache_interface`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
