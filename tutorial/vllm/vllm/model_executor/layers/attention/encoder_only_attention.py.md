# encoder_only_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/attention/encoder_only_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: attention execution, backend dispatch, and KV-cache handling / 注意力执行、后端分派与 KV 缓存处理

## Line-by-Line Analysis / 逐行分析
### Lines 3-19 — imports and setup
```python
import functools
from copy import copy

import torch

from vllm.config import CacheConfig
from vllm.config.vllm import VllmConfig
from vllm.model_executor.layers.attention import Attention
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionMetadata,
    AttentionType,
    CommonAttentionMetadata,
    subclass_attention_backend,
)
from vllm.v1.attention.selector import get_attn_backend
from vllm.v1.kv_cache_interface import KVCacheSpec
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 22-48 — function `create_encoder_only_attention_backend`
```python
@functools.lru_cache
def create_encoder_only_attention_backend(
    underlying_attn_backend: type[AttentionBackend],
) -> type[AttentionBackend]:
    prefix = "EncoderOnlyAttention_"
    underlying_builder = underlying_attn_backend.get_builder_cls()

    class EncoderOnlyAttentionBuilder(underlying_builder):  # type: ignore
        def build(
            self,
            common_prefix_len: int,
            common_attn_metadata: CommonAttentionMetadata,
            fast_build: bool = False,
        ) -> AttentionMetadata:
            new_common_attn_metadata = copy(common_attn_metadata)
            new_common_attn_metadata.causal = False
            return super().build(
                common_prefix_len, new_common_attn_metadata, fast_build
            )

    attn_backend = subclass_attention_backend(
        name_prefix=prefix,
        attention_backend_cls=underlying_attn_backend,
        builder_cls=EncoderOnlyAttentionBuilder,
    )

    return attn_backend
```
**EN:** This function defines `create_encoder_only_attention_backend`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `underlying_attn_backend`. Key calls include `underlying_attn_backend.get_builder_cls`, `subclass_attention_backend`, `copy`, `super.build`, `super`. It writes or updates `prefix`, `underlying_builder`, `attn_backend`, `new_common_attn_metadata`, `causal`.
**CN:** 该函数定义 `create_encoder_only_attention_backend`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `underlying_attn_backend`。 关键调用包括 `underlying_attn_backend.get_builder_cls`, `subclass_attention_backend`, `copy`, `super.build`, `super`。 它会写入或更新 `prefix`, `underlying_builder`, `attn_backend`, `new_common_attn_metadata`, `causal`。

### Lines 51-98 — class `EncoderOnlyAttention`
```python
class EncoderOnlyAttention(Attention):
    """
    Encoder attention is a special case that doesn't need a KV Cache.
    """

    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        cache_config: CacheConfig | None = None,
        attn_type: str | None = None,
        **kwargs,
    ):
# ... omitted for brevity ...
        # Does not need KV cache
        return None
```
**EN:** This class defines `EncoderOnlyAttention`. It inherits from `Attention`. Encoder attention is a special case that doesn't need a KV Cache. Important methods include `__init__`, `get_kv_cache_spec`. Key calls include `torch.get_default_dtype`, `get_attn_backend`, `create_encoder_only_attention_backend`, `super.__init__`, `super`. It writes or updates `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `EncoderOnlyAttention`。 它继承自 `Attention`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `get_kv_cache_spec`。 关键调用包括 `torch.get_default_dtype`, `get_attn_backend`, `create_encoder_only_attention_backend`, `super.__init__`, `super`。 它会写入或更新 `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 56-94 — method `EncoderOnlyAttention.__init__`
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

        underlying_attn_backend = get_attn_backend(
            head_size,
            dtype,
            kv_cache_dtype,
            attn_type=AttentionType.ENCODER_ONLY,
        )

        attn_backend = create_encoder_only_attention_backend(underlying_attn_backend)

        if attn_type is not None:
            assert attn_type == AttentionType.ENCODER_ONLY, (
                "EncoderOnlyAttention only supports AttentionType.ENCODER_ONLY"
            )

        super().__init__(
            num_heads=num_heads,
            head_size=head_size,
            scale=scale,
            cache_config=cache_config,
            attn_backend=attn_backend,
            attn_type=AttentionType.ENCODER_ONLY,
            **kwargs,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `num_heads`, `head_size`, `scale`, `cache_config`, `attn_type`, `**kwargs`. Key calls include `torch.get_default_dtype`, `get_attn_backend`, `create_encoder_only_attention_backend`, `super.__init__`, `super`. It writes or updates `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `num_heads`, `head_size`, `scale`, `cache_config`, `attn_type`, `**kwargs`。 关键调用包括 `torch.get_default_dtype`, `get_attn_backend`, `create_encoder_only_attention_backend`, `super.__init__`, `super`。 它会写入或更新 `dtype`, `underlying_attn_backend`, `attn_backend`, `kv_cache_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 96-98 — method `EncoderOnlyAttention.get_kv_cache_spec`
```python
    def get_kv_cache_spec(self, vllm_config: VllmConfig) -> KVCacheSpec | None:
        # Does not need KV cache
        return None
```
**EN:** This method defines `get_kv_cache_spec`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `vllm_config`.
**CN:** 该方法定义 `get_kv_cache_spec`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `vllm_config`。

## Key Concepts / 关键概念
- [EN] Attention execution, backend dispatch, and kv-cache handling / [CN] 注意力执行、后端分派与 KV 缓存处理
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `create_encoder_only_attention_backend`, `EncoderOnlyAttention` / [CN] 核心符号：`create_encoder_only_attention_backend`, `EncoderOnlyAttention`

## Dependencies / 依赖关系
- **External**: `functools`, `copy`, `torch` / **外部依赖**: `functools`, `copy`, `torch`
- **Internal**: `vllm.config`, `vllm.config.vllm`, `vllm.model_executor.layers.attention`, `vllm.v1.attention.backend`, `vllm.v1.attention.selector`, `vllm.v1.kv_cache_interface` / **内部依赖**: `vllm.config`, `vllm.config.vllm`, `vllm.model_executor.layers.attention`, `vllm.v1.attention.backend`, `vllm.v1.attention.selector`, `vllm.v1.kv_cache_interface`
