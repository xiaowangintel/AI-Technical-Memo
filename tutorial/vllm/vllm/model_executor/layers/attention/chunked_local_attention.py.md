# chunked_local_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/attention/chunked_local_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: attention execution, backend dispatch, and KV-cache handling / 注意力执行、后端分派与 KV 缓存处理

## Line-by-Line Analysis / 逐行分析
### Lines 3-27 — imports and setup
```python
import functools

import torch

from vllm.config import CacheConfig
from vllm.config.vllm import VllmConfig
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionMetadataBuilder,
    CommonAttentionMetadata,
    subclass_attention_backend,
)
from vllm.v1.attention.backends.utils import (
    make_local_attention_virtual_batches,
)
from vllm.v1.attention.selector import get_attn_backend
from vllm.v1.kv_cache_interface import (
    AttentionSpec,
    ChunkedLocalAttentionSpec,
    KVCacheSpec,
    get_kv_quant_mode,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 30-78 — function `create_chunked_local_attention_backend`
```python
@functools.lru_cache
def create_chunked_local_attention_backend(
    underlying_attn_backend: type[AttentionBackend],
    attention_chunk_size: int,
) -> type[AttentionBackend]:
    prefix = f"ChunkedLocalAttention_{attention_chunk_size}_"

    underlying_builder = underlying_attn_backend.get_builder_cls()
    assert issubclass(underlying_builder, AttentionMetadataBuilder)

    class ChunkedLocalAttentionBuilder(underlying_builder):  # type: ignore
        @classmethod
        def get_cudagraph_support(
            cls: type["AttentionMetadataBuilder"],
            vllm_config: VllmConfig,
            kv_cache_spec: AttentionSpec,
        ) -> AttentionCGSupport:
            # Explicit override in case the underlying builder specialized this getter.
            # @override omitted only because of mypy limitation due to type variable.
            return AttentionCGSupport.NEVER

        def build(
            self,
            common_prefix_len: int,
            common_attn_metadata: CommonAttentionMetadata,
            fast_build: bool = False,
        ):
            cm, make_virtual_batches_block_table = make_local_attention_virtual_batches(
                attention_chunk_size,
                common_attn_metadata,
                self.kv_cache_spec.block_size,
            )
            metadata = super().build(common_prefix_len, cm, fast_build)
            metadata.make_virtual_batches_block_table = make_virtual_batches_block_table
            return metadata

        def update_block_table(
            self, metadata, blk_table: torch.Tensor, slot_mapping: torch.Tensor
        ):
            blk_table = metadata.make_virtual_batches_block_table(blk_table)
            return super().update_block_table(metadata, blk_table, slot_mapping)

    attn_backend = subclass_attention_backend(
        name_prefix=prefix,
# ... omitted for brevity ...

    return attn_backend
```
**EN:** This function defines `create_chunked_local_attention_backend`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `underlying_attn_backend`, `attention_chunk_size`. Key calls include `underlying_attn_backend.get_builder_cls`, `issubclass`, `subclass_attention_backend`, `make_local_attention_virtual_batches`, `super.build`, `metadata.make_virtual_batches_block_table`. It writes or updates `prefix`, `underlying_builder`, `attn_backend`, `cm`, `make_virtual_batches_block_table`, `metadata`.
**CN:** 该函数定义 `create_chunked_local_attention_backend`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `underlying_attn_backend`, `attention_chunk_size`。 关键调用包括 `underlying_attn_backend.get_builder_cls`, `issubclass`, `subclass_attention_backend`, `make_local_attention_virtual_batches`, `super.build`, `metadata.make_virtual_batches_block_table`。 它会写入或更新 `prefix`, `underlying_builder`, `attn_backend`, `cm`, `make_virtual_batches_block_table`, `metadata`。

### Lines 81-129 — class `ChunkedLocalAttention`
```python
class ChunkedLocalAttention(Attention):
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        attention_chunk_size: int,
        num_kv_heads: int | None = None,
        alibi_slopes: list[float] | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        kv_sharing_target_layer_name: str | None = None,
        prefix: str = "",
    ):
# ... omitted for brevity ...
            attention_chunk_size=self.attention_chunk_size,
        )
```
**EN:** This class defines `ChunkedLocalAttention`. It inherits from `Attention`. It handles attention-specific state, metadata, or tensor transforms. Important methods include `__init__`, `get_kv_cache_spec`. Key calls include `torch.get_default_dtype`, `get_attn_backend`, `create_chunked_local_attention_backend`, `super.__init__`, `ChunkedLocalAttentionSpec`, `super`. It writes or updates `attention_chunk_size`, `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `ChunkedLocalAttention`。 它继承自 `Attention`。 它处理注意力相关的状态、元数据或张量变换。 重要方法包括 `__init__`, `get_kv_cache_spec`。 关键调用包括 `torch.get_default_dtype`, `get_attn_backend`, `create_chunked_local_attention_backend`, `super.__init__`, `ChunkedLocalAttentionSpec`, `super`。 它会写入或更新 `attention_chunk_size`, `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 82-118 — method `ChunkedLocalAttention.__init__`
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        attention_chunk_size: int,
        num_kv_heads: int | None = None,
        alibi_slopes: list[float] | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        kv_sharing_target_layer_name: str | None = None,
        prefix: str = "",
    ):
        self.attention_chunk_size = attention_chunk_size
        dtype = torch.get_default_dtype()
        if cache_config is not None:
            kv_cache_dtype = cache_config.cache_dtype
        else:
            kv_cache_dtype = "auto"

        underlying_attn_backend = get_attn_backend(head_size, dtype, kv_cache_dtype)
        attn_backend = create_chunked_local_attention_backend(
            underlying_attn_backend, attention_chunk_size
        )

        super().__init__(
            num_heads=num_heads,
            head_size=head_size,
            scale=scale,
            num_kv_heads=num_kv_heads,
            alibi_slopes=alibi_slopes,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=prefix,
            kv_sharing_target_layer_name=kv_sharing_target_layer_name,
            attn_backend=attn_backend,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `num_heads`, `head_size`, `scale`, `attention_chunk_size`, `num_kv_heads`, `alibi_slopes`. Key calls include `torch.get_default_dtype`, `get_attn_backend`, `create_chunked_local_attention_backend`, `super.__init__`, `super`. It writes or updates `attention_chunk_size`, `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `num_heads`, `head_size`, `scale`, `attention_chunk_size`, `num_kv_heads`, `alibi_slopes`。 关键调用包括 `torch.get_default_dtype`, `get_attn_backend`, `create_chunked_local_attention_backend`, `super.__init__`, `super`。 它会写入或更新 `attention_chunk_size`, `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 120-129 — method `ChunkedLocalAttention.get_kv_cache_spec`
```python
    def get_kv_cache_spec(self, vllm_config: VllmConfig) -> KVCacheSpec:
        assert self.attention_chunk_size
        return ChunkedLocalAttentionSpec(
            block_size=vllm_config.cache_config.block_size,
            num_kv_heads=self.num_kv_heads,
            head_size=self.head_size,
            dtype=self.kv_cache_torch_dtype,
            kv_quant_mode=get_kv_quant_mode(self.kv_cache_dtype),
            attention_chunk_size=self.attention_chunk_size,
        )
```
**EN:** This method defines `get_kv_cache_spec`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `vllm_config`. Key calls include `ChunkedLocalAttentionSpec`, `get_kv_quant_mode`.
**CN:** 该方法定义 `get_kv_cache_spec`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `vllm_config`。 关键调用包括 `ChunkedLocalAttentionSpec`, `get_kv_quant_mode`。

## Key Concepts / 关键概念
- [EN] Attention execution, backend dispatch, and kv-cache handling / [CN] 注意力执行、后端分派与 KV 缓存处理
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `create_chunked_local_attention_backend`, `ChunkedLocalAttention` / [CN] 核心符号：`create_chunked_local_attention_backend`, `ChunkedLocalAttention`

## Dependencies / 依赖关系
- **External**: `functools`, `torch` / **外部依赖**: `functools`, `torch`
- **Internal**: `vllm.config`, `vllm.config.vllm`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.quantization`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.v1.attention.selector`, `vllm.v1.kv_cache_interface` / **内部依赖**: `vllm.config`, `vllm.config.vllm`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.quantization`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.v1.attention.selector`, `vllm.v1.kv_cache_interface`
