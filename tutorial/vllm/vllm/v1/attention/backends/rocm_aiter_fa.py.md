# rocm_aiter_fa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/rocm_aiter_fa.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Attention layer with AiterFlashAttention. / 该模块位于 `attention/backends` 子系统，主要围绕 `AiterFlashAttentionDecodeMetadata`, `AiterFlashAttentionPrefillMetadata`, `AiterChunkSlidingWindowMetadata` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Attention layer with AiterFlashAttention."""

from dataclasses import dataclass
from typing import ClassVar

import torch

from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import VllmConfig, get_layers_from_vllm_config
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import Attention
from vllm.platforms import current_platform
from vllm.platforms.interface import DeviceCapability
from vllm.utils.math_utils import cdiv
from vllm.utils.platform_utils import num_compute_units
from vllm.utils.torch_utils import is_quantized_kv_cache
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionImpl,
    AttentionLayer,
    AttentionMetadataBuilder,
    AttentionType,
    CommonAttentionMetadata,
    MultipleOf,
)
from vllm.v1.attention.backends.utils import (
    split_decodes_prefills_and_extends,
)
from vllm.v1.attention.ops.merge_attn_states import merge_attn_states
from vllm.v1.kv_cache_interface import AttentionSpec

_PARTITION_SIZE_ROCM = 256
_CP_TOKENS_PER_ITER_ROCM = 32 * 1024
if current_platform.is_rocm():
    from vllm.triton_utils import tl, triton

    def block_size(x, head_dim):
        return min(65536 // x.element_size(), triton.next_power_of_2(head_dim))

    def num_programs(total_tokens):
        return min(total_tokens, num_compute_units())

    @triton.jit
    def cp_mha_gather_cache_kernel(
        key_cache_ptr,  # [num_blocks, page_size, num_head, head_size]
        value_cache_ptr,  # [num_blocks, page_size, num_head, head_size]
        key_ptr,  # [num_tokens, num_heads, head_size]
        value_ptr,  # [num_tokens, num_heads, head_size]
# ... omitted for brevity ...
            QUANT = True
        grid = (
            num_tokens,
            num_kv_heads,
        )
        reshape_and_cache_shuffle_kernel[grid](
            key,
            value,
            new_key_cache,
            new_value_cache,
            slot_mapping,
            k_scales,
            v_scales,
            x,
            key.stride(0),
            value.stride(0),
            block_size,
            head_size,
            num_kv_heads,
            BLOCK_SIZE=head_size,
            QUANT=QUANT,
        )


logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `_PARTITION_SIZE_ROCM`, `_CP_TOKENS_PER_ITER_ROCM`, `logger`. This excerpt omits repetitive declarations.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `_PARTITION_SIZE_ROCM`, `_CP_TOKENS_PER_ITER_ROCM`, `logger`。 该片段省略了重复性声明。

### `AiterFlashAttentionDecodeMetadata` class / `AiterFlashAttentionDecodeMetadata` 类
```python
@dataclass
class AiterFlashAttentionDecodeMetadata:
    max_query_len: int
```
**EN:** Uses `@dataclass` to package related state for `AiterFlashAttentionDecodeMetadata`. Typical fields include `max_query_len`.
**CN:** `AiterFlashAttentionDecodeMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `max_query_len`。

### `AiterFlashAttentionPrefillMetadata` class / `AiterFlashAttentionPrefillMetadata` 类
```python
@dataclass
class AiterFlashAttentionPrefillMetadata:
    max_query_len: int
    max_seq_len: int
    query_start_loc: torch.Tensor
```
**EN:** Uses `@dataclass` to package related state for `AiterFlashAttentionPrefillMetadata`. Typical fields include `max_query_len`, `max_seq_len`, `query_start_loc`.
**CN:** `AiterFlashAttentionPrefillMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `max_query_len`, `max_seq_len`, `query_start_loc`。

### `AiterChunkSlidingWindowMetadata` class / `AiterChunkSlidingWindowMetadata` 类
```python
@dataclass
class AiterChunkSlidingWindowMetadata:
    swa_cu_seqlens: torch.Tensor
    swa_seq_starts: torch.Tensor
    swa_token_to_batch: torch.Tensor
    swa_max_seqlens: int
    swa_total_tokens: int
    swa_workspace: torch.Tensor
```
**EN:** Uses `@dataclass` to package related state for `AiterChunkSlidingWindowMetadata`. Typical fields include `swa_cu_seqlens`, `swa_seq_starts`, `swa_token_to_batch`, `swa_max_seqlens`, `swa_total_tokens`, `swa_workspace`.
**CN:** `AiterChunkSlidingWindowMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `swa_cu_seqlens`, `swa_seq_starts`, `swa_token_to_batch`, `swa_max_seqlens`, `swa_total_tokens`, `swa_workspace`。

### `AiterChunkContextMetadata` class / `AiterChunkContextMetadata` 类
```python
@dataclass
class AiterChunkContextMetadata:
    workspace: torch.Tensor
    cu_seq_lens_chunk: torch.Tensor
    chunk_starts: torch.Tensor
    token_to_batch: torch.Tensor
    max_seq_lens: list[int]
    num_chunks: int
    total_token_per_batch: list[int]
    swa_metadata: AiterChunkSlidingWindowMetadata | None
```
**EN:** Uses `@dataclass` to package related state for `AiterChunkContextMetadata`. Typical fields include `workspace`, `cu_seq_lens_chunk`, `chunk_starts`, `token_to_batch`, `max_seq_lens`, `num_chunks`.
**CN:** `AiterChunkContextMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `workspace`, `cu_seq_lens_chunk`, `chunk_starts`, `token_to_batch`, `max_seq_lens`, `num_chunks`。

### `AiterFlashAttentionChunkPrefillMetadata` class / `AiterFlashAttentionChunkPrefillMetadata` 类
```python
@dataclass
class AiterFlashAttentionChunkPrefillMetadata:
    max_query_len: int
    max_seq_len: int
    query_start_loc: torch.Tensor
    chunk_context_metadata: AiterChunkContextMetadata
```
**EN:** Uses `@dataclass` to package related state for `AiterFlashAttentionChunkPrefillMetadata`. Typical fields include `max_query_len`, `max_seq_len`, `query_start_loc`, `chunk_context_metadata`.
**CN:** `AiterFlashAttentionChunkPrefillMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `max_query_len`, `max_seq_len`, `query_start_loc`, `chunk_context_metadata`。

### `AiterFlashAttentionMetadata` class / `AiterFlashAttentionMetadata` 类
```python
@dataclass
class AiterFlashAttentionMetadata:
    # NOTE(sang): Definition of context_len, query_len, and seq_len.
    # |---------- N-1 iteration --------|
    # |---------------- N iteration ---------------------|
    # |- tokenA -|......................|-- newTokens ---|
    # |---------- context_len ----------|
    # |-------------------- seq_len ---------------------|
    #                                   |-- query_len ---|

    num_actual_tokens: int  # Number of tokens excluding padding.
    query_start_loc: torch.Tensor
    max_seq_len: int
    seq_lens: torch.Tensor
    slot_mapping: torch.Tensor
    block_table: torch.Tensor
    causal: bool

    # prefill and decode split
    num_decodes: int
    num_decode_tokens: int
    num_prefills: int
    num_extends: int
    num_extend_tokens: int

    decode_metadata: AiterFlashAttentionDecodeMetadata | None
    prefill_metadata: AiterFlashAttentionPrefillMetadata | None
    extend_metadata: AiterFlashAttentionChunkPrefillMetadata | None

    # For cascade attention.
    use_cascade: bool

    # Only for fp8 shuffle layout kv cache, we allocate kv_scale for each layer
    # since we might integrate per token quant for kv cache in the future.
    k_scale: dict[str, torch.Tensor] | None
    v_scale: dict[str, torch.Tensor] | None
```
**EN:** Uses `@dataclass` to package related state for `AiterFlashAttentionMetadata`. Typical fields include `num_actual_tokens`, `query_start_loc`, `max_seq_len`, `seq_lens`, `slot_mapping`, `block_table`.
**CN:** `AiterFlashAttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_actual_tokens`, `query_start_loc`, `max_seq_len`, `seq_lens`, `slot_mapping`, `block_table`。

### `AiterFlashAttentionMetadataBuilder` class / `AiterFlashAttentionMetadataBuilder` 类
```python
class AiterFlashAttentionMetadataBuilder(
    AttentionMetadataBuilder[AiterFlashAttentionMetadata]
):
    _cudagraph_support = AttentionCGSupport.UNIFORM_BATCH
```
**EN:** Introduces the `AiterFlashAttentionMetadataBuilder` class on top of `AttentionMetadataBuilder[AiterFlashAttentionMetadata]`. Core methods include `__init__`, `build_for_cudagraph_capture`, `build`, `build_for_drafting`, `use_cascade_attention`.
**CN:** 这里定义 `AiterFlashAttentionMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[AiterFlashAttentionMetadata]`。核心方法包括 `__init__`, `build_for_cudagraph_capture`, `build`, `build_for_drafting`, `use_cascade_attention`。

### `AiterFlashAttentionMetadataBuilder.__init__` method / `AiterFlashAttentionMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)

        self.model_config = vllm_config.model_config
        self.parallel_config = vllm_config.parallel_config
        self.cache_config = vllm_config.cache_config

        self.num_heads_q = self.model_config.get_num_attention_heads(
            self.parallel_config
        )
        self.num_heads_kv = self.model_config.get_num_kv_heads(self.parallel_config)
        self.headdim = self.model_config.get_head_size()
        self.block_size = kv_cache_spec.block_size
        # Sliding window size to be used with the AOT scheduler will be
        # populated on first build() call.
        self.aot_sliding_window: tuple[int, int] | None = None
        self._init_reorder_batch_threshold(1, supports_spec_as_decode=True)

        sliding_window_configs: set[tuple[int, int] | None] = set()
        layers = get_layers_from_vllm_config(self.vllm_config, Attention)
        for name, layer in layers.items():
            if name not in layer_names:
                continue
            assert isinstance(layer.impl, AiterFlashAttentionImpl), (
                "Aiter Flash Attention Metadata Builder can only be used "
                "with Aiter Flash Attention Impl."
            )
            sliding_window_configs.add(layer.impl.sliding_window)

        while len(sliding_window_configs) > 0:
            sliding_window_config = sliding_window_configs.pop()
            if sliding_window_config is not None and sliding_window_config[0] != -1:
                assert self.aot_sliding_window is None, (
                    "Aiter Flash ATTENTION can only support one valid sliding window!"
                )
                self.aot_sliding_window = sliding_window_config

        self.extend_workspace = torch.empty(
            [2, _CP_TOKENS_PER_ITER_ROCM, self.num_heads_kv, self.headdim],
            dtype=self.model_config.dtype,
            device=device,
        )
        self.scale = torch.tensor([1.0], dtype=torch.float, device=self.device)
```
**EN:** This method initializes the object state within `AiterFlashAttentionMetadataBuilder`. Key calls include `__init__`, `get_num_attention_heads`, `get_num_kv_heads`, `get_head_size`, `_init_reorder_batch_threshold`, `set`. It touches state such as `model_config`, `parallel_config`, `cache_config`, `num_heads_q`, `num_heads_kv`, `headdim`, `block_size`, `aot_sliding_window`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`AiterFlashAttentionMetadataBuilder`。 关键调用包括 `__init__`, `get_num_attention_heads`, `get_num_kv_heads`, `get_head_size`, `_init_reorder_batch_threshold`, `set`。 它会读写 `model_config`, `parallel_config`, `cache_config`, `num_heads_q`, `num_heads_kv`, `headdim`, `block_size`, `aot_sliding_window` 等状态。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `AiterFlashAttentionMetadataBuilder.build_for_cudagraph_capture` method / `AiterFlashAttentionMetadataBuilder.build_for_cudagraph_capture` 方法
```python
    def build_for_cudagraph_capture(
        self, common_attn_metadata: CommonAttentionMetadata
    ):
        return self.build(
            common_prefix_len=0, common_attn_metadata=common_attn_metadata
        )
```
**EN:** This method builds derived structures within `AiterFlashAttentionMetadataBuilder`. Key calls include `build`.
**CN:** 该方法会构建派生结构，其作用域位于`AiterFlashAttentionMetadataBuilder`。 关键调用包括 `build`。

### `AiterFlashAttentionMetadataBuilder.build` method / `AiterFlashAttentionMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> "AiterFlashAttentionMetadata":
        assert self.reorder_batch_threshold is not None
        split_ret = split_decodes_prefills_and_extends(
            common_attn_metadata,
            decode_threshold=self.reorder_batch_threshold,
        )
        # Allocate scales for fp8 shuffle kv cache with shuffle_kv_cache enabled
        if (
            rocm_aiter_ops.is_shuffle_kv_cache_enabled()
            and self.scale.numel() == 1
            and is_quantized_kv_cache(self.vllm_config.cache_config.cache_dtype)
        ):
            layers = get_layers_from_vllm_config(self.vllm_config, Attention)
            first_layer_name = [k for k in layers][0]
            kv_cache_shape = self.vllm_config.compilation_config.static_forward_context[
                first_layer_name
            ].kv_cache.shape
            num_blocks = kv_cache_shape[1]
            self.scale = torch.ones(
                [num_blocks, self.num_heads_kv, self.block_size],
                dtype=torch.float32,
                device=self.device,
            )
        (
            num_decodes,
            num_extends,
            num_prefills,
            num_decode_tokens,
            num_extend_tokens,
            _,
        ) = split_ret

        query_start_loc_cpu = common_attn_metadata.query_start_loc_cpu

        # Only copy seq_lens to CPU when prefill or extend is present to avoid a
        # blocking device→host transfer.
        seq_lens = (
            common_attn_metadata.seq_lens.cpu()
            if num_prefills > 0 or num_extends > 0
            else None
        )

        query_lens_cpu = query_start_loc_cpu[1:] - query_start_loc_cpu[:-1]

        decode_metadata = None
    # ... omitted for brevity ...
            )

        use_cascade = common_prefix_len > 0

        attn_metadata = AiterFlashAttentionMetadata(
            num_actual_tokens=common_attn_metadata.num_actual_tokens,
            query_start_loc=common_attn_metadata.query_start_loc,
            max_seq_len=common_attn_metadata.max_seq_len,
            seq_lens=common_attn_metadata.seq_lens,
            block_table=common_attn_metadata.block_table_tensor,
            causal=common_attn_metadata.causal,
            slot_mapping=common_attn_metadata.slot_mapping,
            num_decodes=num_decodes,
            num_decode_tokens=num_decode_tokens,
            num_prefills=num_prefills,
            num_extends=num_extends,
            num_extend_tokens=num_extend_tokens,
            decode_metadata=decode_metadata,
            prefill_metadata=prefill_metadata,
            extend_metadata=extend_metadata,
            use_cascade=use_cascade,
            k_scale=self.scale,
            v_scale=self.scale,
        )
        return attn_metadata
```
**EN:** This method builds derived structures within `AiterFlashAttentionMetadataBuilder`. Key calls include `split_decodes_prefills_and_extends`, `AiterFlashAttentionMetadata`, `is_shuffle_kv_cache_enabled`, `is_quantized_kv_cache`, `get_layers_from_vllm_config`, `ones`. It touches state such as `scale`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会构建派生结构，其作用域位于`AiterFlashAttentionMetadataBuilder`。 关键调用包括 `split_decodes_prefills_and_extends`, `AiterFlashAttentionMetadata`, `is_shuffle_kv_cache_enabled`, `is_quantized_kv_cache`, `get_layers_from_vllm_config`, `ones`。 它会读写 `scale` 等状态。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `AiterFlashAttentionMetadataBuilder.build_for_drafting` method / `AiterFlashAttentionMetadataBuilder.build_for_drafting` 方法
```python
    def build_for_drafting(
        self,
        common_attn_metadata: CommonAttentionMetadata,
        draft_index: int,
    ) -> AiterFlashAttentionMetadata:
        """
        Build attention metadata for draft model without CPU-GPU sync.

        During EAGLE drafting all requests are uniform decodes, so we can
        skip split_decodes_prefills_and_extends() and avoid all .cpu() /
        .item() calls that would otherwise break CUDA graph capture.
        """
        num_reqs = common_attn_metadata.num_reqs
        num_tokens = common_attn_metadata.num_actual_tokens

        decode_metadata = AiterFlashAttentionDecodeMetadata(
            max_query_len=common_attn_metadata.max_query_len,
        )

        return AiterFlashAttentionMetadata(
            num_actual_tokens=num_tokens,
            query_start_loc=common_attn_metadata.query_start_loc,
            max_seq_len=common_attn_metadata.max_seq_len,
            seq_lens=common_attn_metadata.seq_lens,
            block_table=common_attn_metadata.block_table_tensor,
            causal=common_attn_metadata.causal,
            slot_mapping=common_attn_metadata.slot_mapping,
            num_decodes=num_reqs,
            num_decode_tokens=num_tokens,
            num_prefills=0,
            num_extends=0,
            num_extend_tokens=0,
            decode_metadata=decode_metadata,
            prefill_metadata=None,
            extend_metadata=None,
            use_cascade=False,
            k_scale=self.scale,
            v_scale=self.scale,
        )
```
**EN:** This method builds derived structures within `AiterFlashAttentionMetadataBuilder`. The docstring frames it as: Build attention metadata for draft model without CPU-GPU sync. Key calls include `AiterFlashAttentionDecodeMetadata`, `AiterFlashAttentionMetadata`.
**CN:** 该方法会构建派生结构，其作用域位于`AiterFlashAttentionMetadataBuilder`。 关键调用包括 `AiterFlashAttentionDecodeMetadata`, `AiterFlashAttentionMetadata`。

### `AiterFlashAttentionBackend` class / `AiterFlashAttentionBackend` 类
```python
class AiterFlashAttentionBackend(AttentionBackend):
    supported_dtypes: ClassVar[list[torch.dtype]] = [torch.float16, torch.bfloat16]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "float16",
        "bfloat16",
        "fp8",
        "fp8_e4m3",
        "fp8_e5m2",
    ]
```
**EN:** Introduces the `AiterFlashAttentionBackend` class on top of `AttentionBackend`. Core methods include `supports_attn_type`, `get_supported_kernel_block_sizes`, `get_supported_head_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`.
**CN:** 这里定义 `AiterFlashAttentionBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `supports_attn_type`, `get_supported_kernel_block_sizes`, `get_supported_head_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`。

### `AiterFlashAttentionBackend.supports_attn_type` method / `AiterFlashAttentionBackend.supports_attn_type` 方法
```python
    @classmethod
    def supports_attn_type(cls, attn_type: str) -> bool:
        """ENCODER_DECODER is not supported because the prefill path uses
        flash_attn_varlen_func with cu_seqlens_k set to decoder
        query_start_loc (not encoder seq lens) and causal=True, both of
        which are incorrect for cross-attention layers.
        """
        return attn_type in (AttentionType.DECODER,)
```
**EN:** This method implements `supports_attn_type` within `AiterFlashAttentionBackend`. The docstring frames it as: ENCODER_DECODER is not supported because the prefill path uses flash_attn_varlen_func with cu_seqlens_k set to decoder query_start_loc (not encoder seq lens) and causal=True, both of which are incorrect for cross-attention layers.
**CN:** 该方法会实现 `supports_attn_type`，其作用域位于`AiterFlashAttentionBackend`。

### `AiterFlashAttentionBackend.get_builder_cls` method / `AiterFlashAttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["AiterFlashAttentionMetadataBuilder"]:
        return AiterFlashAttentionMetadataBuilder
```
**EN:** This method returns or derives a value within `AiterFlashAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AiterFlashAttentionBackend`。

### `AiterFlashAttentionBackend.get_kv_cache_shape` method / `AiterFlashAttentionBackend.get_kv_cache_shape` 方法
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,
        head_size: int,
        cache_dtype_str: str = "auto",
    ) -> tuple[int, ...]:
        if block_size % 16 != 0:
            raise ValueError("Block size must be a multiple of 16.")
        return (2, num_blocks, block_size, num_kv_heads, head_size)
```
**EN:** This method returns or derives a value within `AiterFlashAttentionBackend`. Key calls include `ValueError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AiterFlashAttentionBackend`。 关键调用包括 `ValueError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AiterFlashAttentionBackend.supports_compute_capability` method / `AiterFlashAttentionBackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, capability: DeviceCapability) -> bool:
        from vllm.platforms.rocm import on_mi3xx

        # DeviceCapability is currently created using torch.cuda.get_device_capability()
        # which is known to be buggy on rocm systems. on_mi3xx uses amd-smi which is
        # more reliable.
        return on_mi3xx()
```
**EN:** This method implements `supports_compute_capability` within `AiterFlashAttentionBackend`. Key calls include `on_mi3xx`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`AiterFlashAttentionBackend`。 关键调用包括 `on_mi3xx`。

### `AiterFlashAttentionImpl` class / `AiterFlashAttentionImpl` 类
```python
class AiterFlashAttentionImpl(AttentionImpl):
```
**EN:** Introduces the `AiterFlashAttentionImpl` class on top of `AttentionImpl`. Core methods include `__init__`, `extend_for_sliding_window`, `extend_forward`, `forward`, `do_kv_cache_update`, `fused_rope_kvcache_supported`.
**CN:** 这里定义 `AiterFlashAttentionImpl` 类，其基类包括 `AttentionImpl`。核心方法包括 `__init__`, `extend_for_sliding_window`, `extend_forward`, `forward`, `do_kv_cache_update`, `fused_rope_kvcache_supported`。

### `AiterFlashAttentionImpl.extend_forward` method / `AiterFlashAttentionImpl.extend_forward` 方法
```python
    def extend_forward(
        self,
        attn_metadata: AiterFlashAttentionMetadata,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        key_cache: torch.Tensor,
        value_cache: torch.Tensor,
        output: torch.Tensor,
        cu_seqlens_q: torch.Tensor,
        max_seqlen_q: int,
        min_seqlen_q: int,
        max_seqlen_k: int,
        block_table: torch.Tensor,
        slot_mapping: torch.Tensor,
        k_scale: torch.Tensor,
        v_scale: torch.Tensor,
    ):
        if self.sliding_window[0] != -1:
            self.extend_for_sliding_window(
                attn_metadata,
                query,
                key_cache,
                value_cache,
                output,
                cu_seqlens_q,
                max_seqlen_q,
                block_table,
                k_scale,
                v_scale,
            )
            return
        out, lse = rocm_aiter_ops.flash_attn_varlen_func(
            q=query,
            k=key,
            v=value,
            cu_seqlens_q=cu_seqlens_q,
            cu_seqlens_k=cu_seqlens_q,
            max_seqlen_q=max_seqlen_q,
            max_seqlen_k=max_seqlen_q,
            min_seqlen_q=min_seqlen_q,
            dropout_p=0.0,
            softmax_scale=self.scale,
            causal=True,
            window_size=self.sliding_window,
            alibi_slopes=self.alibi_slopes,
            return_lse=True,
        )
        assert attn_metadata.extend_metadata is not None
        chunk_context_metadata = attn_metadata.extend_metadata.chunk_context_metadata
    # ... omitted for brevity ...
            )
            if chunked_output is None:
                chunked_output = suf_out
                chunked_lse = suf_lse
            else:
                tmp_output = torch.empty_like(out)
                tmp_lse = torch.empty_like(lse)
                merge_attn_states(
                    output=tmp_output,
                    output_lse=tmp_lse,
                    prefix_output=chunked_output,
                    prefix_lse=chunked_lse,
                    suffix_output=suf_out,
                    suffix_lse=suf_lse,
                )
                chunked_output = tmp_output
                chunked_lse = tmp_lse

        merge_attn_states(
            output=output,
            prefix_output=chunked_output,
            prefix_lse=chunked_lse,
            suffix_output=out,
            suffix_lse=lse,
        )
```
**EN:** This method implements `extend_forward` within `AiterFlashAttentionImpl`. Key calls include `flash_attn_varlen_func`, `range`, `merge_attn_states`, `extend_for_sliding_window`, `cp_mha_gather_cache`, `empty_like`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `extend_forward`，其作用域位于`AiterFlashAttentionImpl`。 关键调用包括 `flash_attn_varlen_func`, `range`, `merge_attn_states`, `extend_for_sliding_window`, `cp_mha_gather_cache`, `empty_like`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `AiterFlashAttentionImpl.forward` method / `AiterFlashAttentionImpl.forward` 方法
```python
    def forward(
        self,
        layer: torch.nn.Module,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: AiterFlashAttentionMetadata,
        output: torch.Tensor,
        output_scale: torch.Tensor | None = None,
        output_block_scale: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Forward pass with AiterFlashAttention.

        Args:
            query: shape = [num_tokens, num_heads, head_size]
            key: shape = [num_tokens, num_kv_heads, head_size]
            value: shape = [num_tokens, num_kv_heads, head_size]
            kv_cache: shape =
                [2, num_blocks, block_size, num_kv_heads, head_size]
            attn_metadata: Metadata for attention.
        Returns:
            shape = [num_tokens, num_heads * head_size]
        NOTE: FP8 quantization, flash-attn expect the size of
              {q,k,v}_descale to be (num_sequences, num_kv_heads).
              We use torch's .expand() to avoid duplicating values
        """
        if output_scale is not None or output_block_scale is not None:
            raise NotImplementedError(
                "fused output quantization is not yet supported "
                "for AiterFlashAttentionImpl"
            )

        if attn_metadata is None:
            # Profiling run.
            return output.fill_(0)

        # IMPORTANT!
        # NOTE(woosuk): With piece-wise CUDA graphs, this method is
        # executed in eager-mode PyTorch. Thus, we need to be careful
        # about any CPU overhead in this method. For example, `view`
        # and `slice` (or `[:n]`) operations are surprisingly slow even
        # in the case they do not invoke any GPU ops.
        # Minimize the PyTorch ops in this method as much as possible.
        # Whenever making a change in this method, please benchmark the
        # performance to make sure it does not introduce any overhead.
        num_actual_tokens = attn_metadata.num_actual_tokens
        key_cache, value_cache = kv_cache.unbind(0)

        if is_quantized_kv_cache(self.kv_cache_dtype):
    # ... omitted for brevity ...
                        query[:num_decode_tokens],
                        key_cache,
                        value_cache,
                        self.scale,
                        attn_metadata.block_table[:num_decodes],
                        attn_metadata.query_start_loc[:num_decodes],
                        attn_metadata.seq_lens[:num_decodes],
                        attn_metadata.max_seq_len,
                        self.alibi_slopes,
                        self.kv_cache_dtype,
                        "NHD",
                        self.logits_soft_cap,
                        layer._k_scale,
                        layer._v_scale,
                        None,
                        _PARTITION_SIZE_ROCM,
                        1,
                        self.sliding_window[0] + 1,
                    )
        else:
            raise NotImplementedError(
                "Cascade attention is not implemented for ROCM AITER"
            )

        return output
```
**EN:** This method drives the forward-pass computation within `AiterFlashAttentionImpl`. The docstring frames it as: Forward pass with AiterFlashAttention. Key calls include `unbind`, `is_quantized_kv_cache`, `NotImplementedError`, `fill_`, `view`, `fp8_dtype`. The control flow contains 16 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会驱动前向计算流程，其作用域位于`AiterFlashAttentionImpl`。 关键调用包括 `unbind`, `is_quantized_kv_cache`, `NotImplementedError`, `fill_`, `view`, `fp8_dtype`。 控制流包含 16 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `AiterFlashAttentionImpl.do_kv_cache_update` method / `AiterFlashAttentionImpl.do_kv_cache_update` 方法
```python
    def do_kv_cache_update(
        self,
        layer: AttentionLayer,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        slot_mapping: torch.Tensor,
    ):
        key_cache, value_cache = kv_cache.unbind(0)

        # key and value may be None in the case of cross attention. They are
        # calculated once based on the output from the encoder and then cached
        # in KV cache.
        if is_quantized_kv_cache(self.kv_cache_dtype):
            key_cache = key_cache.view(current_platform.fp8_dtype())
            value_cache = value_cache.view(current_platform.fp8_dtype())
        # Reshape the input keys and values and store them in the cache.
        # Skip this if sharing KV cache with an earlier attention layer.
        # NOTE(woosuk): Here, key and value are padded while slot_mapping
        # is not padded. However, we don't need to do
        # key[:num_actual_tokens] and value[:num_actual_tokens] because
        # the reshape_and_cache_flash op uses the slot_mapping's shape
        # to determine the number of actual tokens.
        if rocm_aiter_ops.is_shuffle_kv_cache_enabled():
            # We may calculate per token quant scale in
            # reshape_and_cache_shuffle_triton which might differ from
            # vllm's style when shuffle layout is used.
            k_scale = layer._k_scale
            v_scale = layer._v_scale
            assert k_scale is not None and v_scale is not None, (
                "k_scale and v_scale are required for shuffled update"
            )
            # TODO: Add correct KV cache handling for hybrid model. KV cache
            # may not be contiguous if mamba state exists.
            reshape_and_cache_shuffle_triton(
                key,
                value,
                key_cache,
                value_cache,
                slot_mapping,
                self.kv_cache_dtype,
                k_scale,
                v_scale,
            )
        else:
            torch.ops._C_cache_ops.reshape_and_cache_flash(
                key,
                value,
                key_cache,
                value_cache,
                slot_mapping,
                self.kv_cache_dtype,
                layer._k_scale,
                layer._v_scale,
            )
```
**EN:** This method implements `do_kv_cache_update` within `AiterFlashAttentionImpl`. Key calls include `unbind`, `is_quantized_kv_cache`, `is_shuffle_kv_cache_enabled`, `view`, `reshape_and_cache_shuffle_triton`, `reshape_and_cache_flash`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_kv_cache_update`，其作用域位于`AiterFlashAttentionImpl`。 关键调用包括 `unbind`, `is_quantized_kv_cache`, `is_shuffle_kv_cache_enabled`, `view`, `reshape_and_cache_shuffle_triton`, `reshape_and_cache_flash`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AiterFlashAttentionImpl.do_rope_and_kv_cache_update` method / `AiterFlashAttentionImpl.do_rope_and_kv_cache_update` 方法
```python
    def do_rope_and_kv_cache_update(
        self,
        layer: AttentionLayer,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        positions: torch.Tensor,
        cos_sin_cache: torch.Tensor,
        is_neox: bool,
        kv_cache: torch.Tensor,
        layer_slot_mapping: torch.Tensor,
    ):
        key_cache, value_cache = kv_cache.unbind(0)
        flash_layout = True

        is_fp8_kv_cache = is_quantized_kv_cache(self.kv_cache_dtype)
        if is_fp8_kv_cache:
            key_cache = key_cache.view(current_platform.fp8_dtype())
            value_cache = value_cache.view(current_platform.fp8_dtype())

        rocm_aiter_ops.triton_rope_and_cache(
            query,
            key,
            value,
            positions,
            cos_sin_cache,
            is_neox,
            key_cache,
            value_cache,
            layer_slot_mapping,
            layer._k_scale,
            layer._v_scale,
            flash_layout,
            is_fp8_kv_cache,
        )
```
**EN:** This method implements `do_rope_and_kv_cache_update` within `AiterFlashAttentionImpl`. Key calls include `unbind`, `is_quantized_kv_cache`, `triton_rope_and_cache`, `view`, `fp8_dtype`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_rope_and_kv_cache_update`，其作用域位于`AiterFlashAttentionImpl`。 关键调用包括 `unbind`, `is_quantized_kv_cache`, `triton_rope_and_cache`, `view`, `fp8_dtype`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `AiterFlashAttentionDecodeMetadata`: central class or interface in this module. / `AiterFlashAttentionDecodeMetadata`：本模块中的核心类或接口。
- `AiterFlashAttentionPrefillMetadata`: central class or interface in this module. / `AiterFlashAttentionPrefillMetadata`：本模块中的核心类或接口。
- `AiterChunkSlidingWindowMetadata`: central class or interface in this module. / `AiterChunkSlidingWindowMetadata`：本模块中的核心类或接口。
- `AiterChunkContextMetadata`: central class or interface in this module. / `AiterChunkContextMetadata`：本模块中的核心类或接口。
- `AiterFlashAttentionChunkPrefillMetadata`: central class or interface in this module. / `AiterFlashAttentionChunkPrefillMetadata`：本模块中的核心类或接口。
- `AiterFlashAttentionMetadata`: central class or interface in this module. / `AiterFlashAttentionMetadata`：本模块中的核心类或接口。
- `AiterFlashAttentionMetadataBuilder`: central class or interface in this module. / `AiterFlashAttentionMetadataBuilder`：本模块中的核心类或接口。
- `AiterFlashAttentionBackend`: central class or interface in this module. / `AiterFlashAttentionBackend`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `torch`, `aiter`
- Internal vLLM / 内部依赖: `vllm._aiter_ops`, `vllm.config`, `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.attention`, `vllm.platforms`, `vllm.platforms.interface`, `vllm.utils.math_utils`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`
