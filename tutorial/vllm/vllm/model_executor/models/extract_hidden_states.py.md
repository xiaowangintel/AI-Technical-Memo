# extract_hidden_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/extract_hidden_states.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Utility model wrapper that exposes hidden-state extraction paths for downstream tasks. / 工具型模型封装，向下游任务暴露隐藏状态提取路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 11-39)
```python
from collections.abc import Iterable
from dataclasses import replace
from typing import ClassVar

import torch
import torch.nn as nn

from vllm.config import CacheConfig, VllmConfig, get_current_vllm_config
from vllm.config.cache import CacheDType
from vllm.forward_context import get_forward_context
from vllm.model_executor.layers.attention.attention import set_default_quant_scales
from vllm.model_executor.layers.attention.kv_transfer_utils import (
    maybe_transfer_kv_layer,
)
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.model_executor.models.utils import maybe_prefix
from vllm.utils.torch_utils import is_quantized_kv_cache, kv_cache_dtype_str_to_dtype
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadataBuilder,
    AttentionType,
    CommonAttentionMetadata,
)
from vllm.v1.kv_cache_interface import (
    AttentionSpec,
    HiddenStateCacheSpec,
    KVCacheSpec,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `unified_kv_cache_update` (lines 44-72)
```python
def unified_kv_cache_update(
    to_cache: torch.Tensor,
    layer_name: str,
) -> torch.Tensor:
    """
    Returns a dummy that is passed to unified_attention to signal a side effect and
    the data dependency between them to ensure torch.compile preserves ordering.
    """
    forward_context = get_forward_context()
    attn_layer = forward_context.no_compile_layers[layer_name]
    kv_cache = attn_layer.kv_cache

    slot_mapping = forward_context.slot_mapping
    assert isinstance(slot_mapping, dict), (
        f"Expected slot_mapping to be a dict, got {type(slot_mapping)}. "
    )
    layer_slot_mapping = slot_mapping.get(layer_name)
    if layer_slot_mapping is not None:
        assert hasattr(attn_layer.impl, "do_kv_cache_update"), (
            f"{attn_layer.impl.__class__.__name__} does not support kv cache update"
        )
        attn_layer.impl.do_kv_cache_update(
            attn_layer,
            to_cache,
            kv_cache,
            layer_slot_mapping,
        )

    return torch.empty(0, device=kv_cache.device, dtype=kv_cache.dtype)
```
**EN:** Function `unified_kv_cache_update` verifies runtime/model configuration and applies compatibility fixes before the rest of the stack executes. The docstring says: Returns a dummy that is passed to unified_attention to signal a side effect and the data dependency between them to ensure torch.compile preserves ordering.
**CN:** Function `unified_kv_cache_update` 会在其余栈执行之前校验运行时/模型配置，并应用兼容性修补。 文档字符串表达的核心意思是：Returns a dummy that is passed to unified_attention to signal a side effect and the data dependency between them to ensure torch.compile preserves ordering。

### Function `dummy_attention` (lines 76-78)
```python
@maybe_transfer_kv_layer
def dummy_attention(layer_name, _placeholder):
    # Note: layer_name arg required by @maybe_transfer_kv_layer
    return _placeholder
```
**EN:** Function `dummy_attention` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `dummy_attention` 封装了该模块中的一段可复用核心逻辑。

### Function `basic_cache` (lines 81-87)
```python
def basic_cache(
    to_cache: torch.Tensor,  # shape: [seq_len, num_heads, head_size]
    kv_cache: torch.Tensor,  # shape: [num_blocks, block_size, num_heads, head_size]
    slot_mapping: torch.Tensor,  # shape: [seq_len]
):
    block_size = kv_cache.shape[1]
    kv_cache[slot_mapping // block_size, slot_mapping % block_size] = to_cache
```
**EN:** Function `basic_cache` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `basic_cache` 封装了该模块中的一段可复用核心逻辑。

### Class `CacheOnlyAttentionBackend` (lines 93-145)
```python
class CacheOnlyAttentionBackend(AttentionBackend):
    """Attention backend that only caches KV without computing attention."""

    supported_dtypes: ClassVar[list[torch.dtype]] = [
        torch.float16,
        torch.bfloat16,
        torch.float32,
    ]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "bfloat16",
    ]
    forward_includes_kv_cache_update: bool = False

    @staticmethod
    def get_name() -> str:
        return "CACHE_ONLY_ATTN"

    @classmethod
    def supports_attn_type(cls, attn_type: str) -> bool:
        return attn_type == AttentionType.DECODER

    @classmethod
    def supports_mm_prefix(cls) -> bool:
        return True
```
**EN:** Class `CacheOnlyAttentionBackend` organizes related behavior for this model family or helper component. It inherits from AttentionBackend. Key methods include get_name, supports_attn_type, supports_mm_prefix, get_impl_cls, get_kv_cache_shape, get_builder_cls.
**CN:** 类 `CacheOnlyAttentionBackend` 用于组织该模型族或辅助组件的相关行为。 它继承自 AttentionBackend。 关键方法包括 get_name, supports_attn_type, supports_mm_prefix, get_impl_cls, get_kv_cache_shape, get_builder_cls。

### Method `CacheOnlyAttentionBackend.get_name` (lines 108-109)
```python
    @staticmethod
    def get_name() -> str:
        return "CACHE_ONLY_ATTN"
```
**EN:** Method `CacheOnlyAttentionBackend.get_name` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `CacheOnlyAttentionBackend.get_name` 封装了该模块中的一段可复用核心逻辑。

### Method `CacheOnlyAttentionBackend.supports_attn_type` (lines 112-113)
```python
    @classmethod
    def supports_attn_type(cls, attn_type: str) -> bool:
        return attn_type == AttentionType.DECODER
```
**EN:** Method `CacheOnlyAttentionBackend.supports_attn_type` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `CacheOnlyAttentionBackend.supports_attn_type` 封装了该模块中的一段可复用核心逻辑。

### Method `CacheOnlyAttentionBackend.supports_mm_prefix` (lines 116-117)
```python
    @classmethod
    def supports_mm_prefix(cls) -> bool:
        return True
```
**EN:** Method `CacheOnlyAttentionBackend.supports_mm_prefix` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `CacheOnlyAttentionBackend.supports_mm_prefix` 封装了该模块中的一段可复用核心逻辑。

### Method `CacheOnlyAttentionBackend.get_impl_cls` (lines 120-121)
```python
    @staticmethod
    def get_impl_cls() -> type["CacheOnlyAttentionImpl"]:
        return CacheOnlyAttentionImpl
```
**EN:** Method `CacheOnlyAttentionBackend.get_impl_cls` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `CacheOnlyAttentionBackend.get_impl_cls` 封装了该模块中的一段可复用核心逻辑。

### Class `CacheOnlyAttentionMetadata` (lines 148-150)
```python
class CacheOnlyAttentionMetadata:
    def __init__(self, slot_mapping: torch.Tensor):
        self.slot_mapping = slot_mapping
```
**EN:** Class `CacheOnlyAttentionMetadata` organizes related behavior for this model family or helper component. Key methods include __init__.
**CN:** 类 `CacheOnlyAttentionMetadata` 用于组织该模型族或辅助组件的相关行为。 关键方法包括 __init__。

### Method `CacheOnlyAttentionMetadata.__init__` (lines 149-150)
```python
    def __init__(self, slot_mapping: torch.Tensor):
        self.slot_mapping = slot_mapping
```
**EN:** Method `CacheOnlyAttentionMetadata.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `CacheOnlyAttentionMetadata.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `CacheOnlyAttentionMetadataBuilder` (lines 153-184)
```python
class CacheOnlyAttentionMetadataBuilder(
    AttentionMetadataBuilder[CacheOnlyAttentionMetadata]
):
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)

    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> CacheOnlyAttentionMetadata:
        use_cascade = common_prefix_len > 0
        if use_cascade:
            raise NotImplementedError(
                "Cascade attention not supported by CacheOnlyAttention"
            )
        causal = common_attn_metadata.causal
        if not causal:
```
**EN:** Class `CacheOnlyAttentionMetadataBuilder` organizes related behavior for this model family or helper component. It inherits from AttentionMetadataBuilder[CacheOnlyAttentionMetadata]. Key methods include __init__, build.
**CN:** 类 `CacheOnlyAttentionMetadataBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 AttentionMetadataBuilder[CacheOnlyAttentionMetadata]。 关键方法包括 __init__, build。

### Method `CacheOnlyAttentionMetadataBuilder.__init__` (lines 156-163)
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)
```
**EN:** Method `CacheOnlyAttentionMetadataBuilder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `CacheOnlyAttentionMetadataBuilder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `CacheOnlyAttentionMetadataBuilder.build` (lines 165-184)
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> CacheOnlyAttentionMetadata:
        use_cascade = common_prefix_len > 0
        if use_cascade:
            raise NotImplementedError(
                "Cascade attention not supported by CacheOnlyAttention"
            )
        causal = common_attn_metadata.causal
        if not causal:
            raise NotImplementedError(
                "Non-causal attention not supported by CacheOnlyAttention"
            )

        return CacheOnlyAttentionMetadata(
            slot_mapping=common_attn_metadata.slot_mapping,
        )
```
**EN:** Method `CacheOnlyAttentionMetadataBuilder.build` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `CacheOnlyAttentionMetadataBuilder.build` 封装了该模块中的一段可复用核心逻辑。

### Class `CacheOnlyAttentionImpl` (lines 187-228)
```python
class CacheOnlyAttentionImpl(AttentionImpl):
    """Attention implementation that only caches KV states."""

    def __init__(
        self,
        num_heads: int,
        head_size: int,
        kv_cache_dtype: str,
        kv_cache_torch_dtype: torch.dtype,
        attn_type: AttentionType = AttentionType.DECODER,
    ) -> None:
        self.num_heads = num_heads
        self.head_size = head_size
        self.kv_cache_dtype = kv_cache_dtype
        self.kv_cache_torch_dtype = kv_cache_torch_dtype

        if attn_type != AttentionType.DECODER:
            raise NotImplementedError(f"Unsupported attention type: {attn_type}")
        if is_quantized_kv_cache(kv_cache_dtype):
            raise NotImplementedError("Quantized KV cache not supported")

        self.num_queries_per_kv = 1

    def do_kv_cache_update(
        self,
```
**EN:** Class `CacheOnlyAttentionImpl` organizes related behavior for this model family or helper component. It inherits from AttentionImpl. Key methods include __init__, do_kv_cache_update, forward.
**CN:** 类 `CacheOnlyAttentionImpl` 用于组织该模型族或辅助组件的相关行为。 它继承自 AttentionImpl。 关键方法包括 __init__, do_kv_cache_update, forward。

### Method `CacheOnlyAttentionImpl.__init__` (lines 190-208)
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        kv_cache_dtype: str,
        kv_cache_torch_dtype: torch.dtype,
        attn_type: AttentionType = AttentionType.DECODER,
    ) -> None:
        self.num_heads = num_heads
        self.head_size = head_size
        self.kv_cache_dtype = kv_cache_dtype
        self.kv_cache_torch_dtype = kv_cache_torch_dtype

        if attn_type != AttentionType.DECODER:
            raise NotImplementedError(f"Unsupported attention type: {attn_type}")
        if is_quantized_kv_cache(kv_cache_dtype):
            raise NotImplementedError("Quantized KV cache not supported")

        self.num_queries_per_kv = 1
```
**EN:** Method `CacheOnlyAttentionImpl.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `CacheOnlyAttentionImpl.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `CacheOnlyAttentionImpl.do_kv_cache_update` (lines 210-224)
```python
    def do_kv_cache_update(
        self,
        layer,
        to_cache,
        kv_cache,
        slot_mapping,
    ):
        assert to_cache.dtype == self.kv_cache_torch_dtype, (
            f"Data to cache must be {self.kv_cache_torch_dtype}, got {to_cache.dtype}"
        )
        assert kv_cache.dtype == self.kv_cache_torch_dtype, (
            f"KV cache must be {self.kv_cache_torch_dtype}, got {kv_cache.dtype}"
        )

        basic_cache(to_cache, kv_cache, slot_mapping)
```
**EN:** Method `CacheOnlyAttentionImpl.do_kv_cache_update` verifies runtime/model configuration and applies compatibility fixes before the rest of the stack executes.
**CN:** Method `CacheOnlyAttentionImpl.do_kv_cache_update` 会在其余栈执行之前校验运行时/模型配置，并应用兼容性修补。

### Method `CacheOnlyAttentionImpl.forward` (lines 226-228)
```python
    def forward(self, *args, **kwargs):
        # Empty implementation of abstract method
        pass
```
**EN:** Method `CacheOnlyAttentionImpl.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `CacheOnlyAttentionImpl.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `CacheOnlyAttentionLayer` (lines 234-330)
```python
class CacheOnlyAttentionLayer(nn.Module, AttentionLayerBase):
    """Attention layer that only caches key/value states without computing attention."""

    def __init__(
        self,
        num_heads: int,
        head_size: int,
        cache_config: CacheConfig | None = None,
        prefix: str = "",
        attn_type: str = AttentionType.DECODER,
    ):
        super().__init__()

        self.num_heads = num_heads
        self.head_size = head_size
        self.layer_name = prefix

        vllm_config = get_current_vllm_config()

        # KV cache configuration
        cache_config = cache_config or vllm_config.cache_config
        if cache_config is not None:
            kv_cache_dtype = cache_config.cache_dtype
            self.block_size = cache_config.block_size
        else:
```
**EN:** Class `CacheOnlyAttentionLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module, AttentionLayerBase. Key methods include __init__, forward, get_attn_backend, get_kv_cache_spec.
**CN:** 类 `CacheOnlyAttentionLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、AttentionLayerBase。 关键方法包括 __init__, forward, get_attn_backend, get_kv_cache_spec。

### Method `CacheOnlyAttentionLayer.__init__` (lines 237-295)
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        cache_config: CacheConfig | None = None,
        prefix: str = "",
        attn_type: str = AttentionType.DECODER,
    ):
        super().__init__()

        self.num_heads = num_heads
        self.head_size = head_size
        self.layer_name = prefix

        vllm_config = get_current_vllm_config()

        # KV cache configuration
        cache_config = cache_config or vllm_config.cache_config
        if cache_config is not None:
            kv_cache_dtype = cache_config.cache_dtype
            self.block_size = cache_config.block_size
        else:
            kv_cache_dtype = "auto"
            self.block_size = 16

        assert kv_cache_dtype in ["auto", "bfloat16", "float16"], (
            "CacheOnlyAttentionLayer doesn't currently support quantized kv cache but"
            f"kv cache dtype was set to {kv_cache_dtype}"
        )
        self.kv_cache_torch_dtype = kv_cache_dtype_str_to_dtype(
            kv_cache_dtype, vllm_config.model_config
        )

        # Initialize KV cache quantization attributes
        set_default_quant_scales(self, register_buffer=True)

        # Attention backend
        self.attn_backend = CacheOnlyAttentionBackend
        impl_cls = self.attn_backend.get_impl_cls()
        self.impl = impl_cls(
            num_heads,
            head_size,
            kv_cache_dtype,
            self.kv_cache_torch_dtype,
            attn_type,
        )

        assert not self.attn_backend.forward_includes_kv_cache_update, (
            "KV cache update should be independent of forward"
        )

        # Placeholder KV cache (replaced by bind_kv_cache)
        self.kv_cache = torch.tensor([])

        # Register in compilation context
        compilation_config = vllm_config.compilation_config
        if prefix in compilation_config.static_forward_context:
            raise ValueError(f"Duplicate layer name: {prefix}")
        compilation_config.static_forward_context[prefix] = self
```
**EN:** Method `CacheOnlyAttentionLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `CacheOnlyAttentionLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `CacheOnlyAttentionLayer.forward` (lines 297-318)
```python
    def forward(self, to_cache: torch.Tensor) -> torch.Tensor:
        """Cache hidden states as KV pairs without computing attention.

        Args:
            to_cache: The tensor to insert into the kv cache.
                shape [num_tokens, num_heads, head_size]

        Returns:
            Dummy output tensor (not used)
        """
        # Note: we set num_heads to num_hidden_layers and
        # head_size to hidden_size for hidden states storage
        output = torch.empty(0, device=to_cache.device, dtype=to_cache.dtype)

        # Note: dummy_out is used to force torch.compile to preserve ordering between
        # cache update and attention op (which triggers kv_connector transfer)
        dummy_out = unified_kv_cache_update(to_cache, self.layer_name)

        # Triggers kv_connector transfer via decorator
        _ = dummy_attention(self.layer_name, dummy_out)

        return output
```
**EN:** Method `CacheOnlyAttentionLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Cache hidden states as KV pairs without computing attention.
**CN:** Method `CacheOnlyAttentionLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Cache hidden states as KV pairs without computing attention。

### Method `CacheOnlyAttentionLayer.get_attn_backend` (lines 320-321)
```python
    def get_attn_backend(self) -> type[AttentionBackend]:
        return self.attn_backend
```
**EN:** Method `CacheOnlyAttentionLayer.get_attn_backend` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `CacheOnlyAttentionLayer.get_attn_backend` 封装了该模块中的一段可复用核心逻辑。

### Method `CacheOnlyAttentionLayer.get_kv_cache_spec` (lines 323-330)
```python
    def get_kv_cache_spec(self, vllm_config: VllmConfig) -> KVCacheSpec:
        # Re-read block_size: hybrid models may bump it after __init__.
        return HiddenStateCacheSpec(
            block_size=vllm_config.cache_config.block_size,
            num_kv_heads=self.num_heads,
            head_size=self.head_size,
            dtype=self.kv_cache_torch_dtype,
        )
```
**EN:** Method `CacheOnlyAttentionLayer.get_kv_cache_spec` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `CacheOnlyAttentionLayer.get_kv_cache_spec` 封装了该模块中的一段可复用核心逻辑。

### Class `ExtractHiddenStatesModel` (lines 336-391)
```python
class ExtractHiddenStatesModel(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        self.vllm_config = vllm_config
        self.hf_config = vllm_config.speculative_config.draft_model_config.hf_config
        self.hidden_size = vllm_config.model_config.get_hidden_size()
        self.target_num_hidden_layers = (
            vllm_config.model_config.get_total_num_hidden_layers()
        )
        self.num_hidden_states = len(
            getattr(self.hf_config, "eagle_aux_hidden_state_layer_ids", [])
        )

        cache_config = vllm_config.cache_config

        # Hidden states dtype should be independent of KV cache dtype.
        if cache_config is not None and is_quantized_kv_cache(cache_config.cache_dtype):
            cache_config = replace(cache_config, cache_dtype="auto")

        # Create a single cache-only attention layer
        # Note: We set num_heads <- self.num_hidden_states
        # and head_size <- hidden_size so that we can insert
        # the hidden states directly into the cache without
        # reshaping
```
**EN:** Class `ExtractHiddenStatesModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward, load_weights.
**CN:** 类 `ExtractHiddenStatesModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward, load_weights。

### Method `ExtractHiddenStatesModel.__init__` (lines 337-372)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        self.vllm_config = vllm_config
        self.hf_config = vllm_config.speculative_config.draft_model_config.hf_config
        self.hidden_size = vllm_config.model_config.get_hidden_size()
        self.target_num_hidden_layers = (
            vllm_config.model_config.get_total_num_hidden_layers()
        )
        self.num_hidden_states = len(
            getattr(self.hf_config, "eagle_aux_hidden_state_layer_ids", [])
        )

        cache_config = vllm_config.cache_config

        # Hidden states dtype should be independent of KV cache dtype.
        if cache_config is not None and is_quantized_kv_cache(cache_config.cache_dtype):
            cache_config = replace(cache_config, cache_dtype="auto")

        # Create a single cache-only attention layer
        # Note: We set num_heads <- self.num_hidden_states
        # and head_size <- hidden_size so that we can insert
        # the hidden states directly into the cache without
        # reshaping
        self.cache_only_layers = nn.ModuleDict(
            {
                str(self.target_num_hidden_layers): CacheOnlyAttentionLayer(
                    num_heads=self.num_hidden_states,
                    head_size=self.hidden_size,
                    cache_config=cache_config,
                    prefix=maybe_prefix(
                        prefix, f"cache_only_layers.{self.target_num_hidden_layers}"
                    ),
                )
            }
        )
```
**EN:** Method `ExtractHiddenStatesModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ExtractHiddenStatesModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ExtractHiddenStatesModel.forward` (lines 374-387)
```python
    def forward(self, hidden_states: torch.Tensor) -> None:
        """Process and cache hidden states.

        Args:
            hidden_states: Hidden states from target model
                          shape: [num_tokens, num_hidden_states, hidden_size]

        Returns:
            Tuple of (dummy_output, dummy_output) - both unused
        """

        # Call dummy attention layer to cache hidden states
        # Output is ignored - we only care about the KV cache side effects
        _ = self.cache_only_layers[str(self.target_num_hidden_layers)](hidden_states)
```
**EN:** Method `ExtractHiddenStatesModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Process and cache hidden states.
**CN:** Method `ExtractHiddenStatesModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Process and cache hidden states。

### Method `ExtractHiddenStatesModel.load_weights` (lines 389-391)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        """No weights to load for this dummy model."""
        return set()
```
**EN:** Method `ExtractHiddenStatesModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders. The docstring says: No weights to load for this dummy model.
**CN:** Method `ExtractHiddenStatesModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。 文档字符串表达的核心意思是：No weights to load for this dummy model。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable`, `from dataclasses import replace`, `from typing import ClassVar`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import CacheConfig, VllmConfig, get_current_vllm_config`, `from vllm.config.cache import CacheDType`, `from vllm.forward_context import get_forward_context`, `from vllm.model_executor.layers.attention.attention import set_default_quant_scales`, `from vllm.model_executor.layers.attention.kv_transfer_utils import (`, `from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase`, `from vllm.model_executor.models.utils import maybe_prefix`, `from vllm.utils.torch_utils import is_quantized_kv_cache, kv_cache_dtype_str_to_dtype`, `from vllm.v1.attention.backend import (`, `from vllm.v1.kv_cache_interface import (`
- **Module note / 模块说明**: **EN:** Hidden States Extractor Model.  This model extracts and caches hidden states from the target model without performing actual token generation. It's used with the extract_hidden_states speculative decoding method. **CN:** 模块文档字符串给出的原始说明是：Hidden States Extractor Model.  This model extracts and caches hidden states from the target model without performing actual token generation. It's used with the extract_hidden_states speculative decoding method.。
