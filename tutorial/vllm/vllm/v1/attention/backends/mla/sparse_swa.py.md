# sparse_swa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/sparse_swa.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_layer_type_for`, `DeepseekV4SWACache`, `DeepseekSparseSWABackend` for the V1 `attention/backends/mla` subsystem. / 为 V1 的 `attention/backends/mla` 子系统实现 `_layer_type_for`, `DeepseekV4SWACache`, `DeepseekSparseSWABackend`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from dataclasses import dataclass
from typing import ClassVar, cast

import torch

from vllm.config import CacheConfig, VllmConfig, get_current_vllm_config
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionMetadataBuilder,
    CommonAttentionMetadata,
    MultipleOf,
)
from vllm.v1.attention.backends.utils import split_decodes_and_prefills
from vllm.v1.attention.ops.flashmla import FlashMLASchedMeta, get_mla_metadata
from vllm.v1.kv_cache_interface import (
    KVCacheSpec,
    MLAAttentionSpec,
    SlidingWindowMLASpec,
)

# DeepseekV4 decode layer types, keyed by compress_ratio. Each type has a distinct
# (topk, extra_topk, extra_page_block_size) config, so they cannot share a
# FlashMLA tile-scheduler plan. Within a type, all ~60 DeepseekV4 layers share one
# plan per step because b / s_q / h_q / page_block_sizes / topks are identical.
_LAYER_TYPE_SWAONLY = "swaonly"
_LAYER_TYPE_C4A = "c4a"
_LAYER_TYPE_C128A = "c128a"
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `_LAYER_TYPE_SWAONLY`, `_LAYER_TYPE_C4A`, `_LAYER_TYPE_C128A`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `_LAYER_TYPE_SWAONLY`, `_LAYER_TYPE_C4A`, `_LAYER_TYPE_C128A`。

### `_layer_type_for` function / `_layer_type_for` 函数
```python
def _layer_type_for(compress_ratio: int) -> str:
    if compress_ratio <= 1:
        return _LAYER_TYPE_SWAONLY
    if compress_ratio == 4:
        return _LAYER_TYPE_C4A
    if compress_ratio == 128:
        return _LAYER_TYPE_C128A
    raise ValueError(
        f"Unsupported DeepseekV4 compress_ratio={compress_ratio}; "
        "expected 1, 4, or 128."
    )
```
**EN:** This function implements `_layer_type_for` within the module. Key calls include `ValueError`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_layer_type_for`，其作用域位于the module。 关键调用包括 `ValueError`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekV4SWACache` class / `DeepseekV4SWACache` 类
```python
class DeepseekV4SWACache(torch.nn.Module, AttentionLayerBase):
```
**EN:** Introduces the `DeepseekV4SWACache` class on top of `torch.nn.Module`, `AttentionLayerBase`. Core methods include `__init__`, `get_kv_cache_spec`, `forward`, `get_attn_backend`.
**CN:** 这里定义 `DeepseekV4SWACache` 类，其基类包括 `torch.nn.Module`, `AttentionLayerBase`。核心方法包括 `__init__`, `get_kv_cache_spec`, `forward`, `get_attn_backend`。

### `DeepseekV4SWACache.__init__` method / `DeepseekV4SWACache.__init__` 方法
```python
    def __init__(
        self,
        head_dim: int,
        window_size: int,
        dtype: torch.dtype,
        prefix: str,
        cache_config: CacheConfig,
    ):
        super().__init__()
        self.kv_cache = torch.tensor([])
        self.head_dim = head_dim
        self.window_size = window_size
        self.prefix = prefix
        self.cache_config = cache_config
        self.dtype = dtype
        compilation_config = get_current_vllm_config().compilation_config
        if prefix in compilation_config.static_forward_context:
            raise ValueError(f"Duplicate layer name: {prefix}")
        compilation_config.static_forward_context[prefix] = self

        # Block size is constrained by tensor sharing between SWA and C4A KV blocks.
        # Since both block types share the same physical tensor, they must use the
        # same page size. The C4A KV block shape [256//4, head_dim] = [64, head_dim]
        # determines the SWA block size of 64 tokens per block.
        # TODO(yifan): make SWA block size automatically determined and configurable.
        self.block_size = 64
        assert self.dtype == torch.uint8
```
**EN:** This method initializes the object state within `DeepseekV4SWACache`. Key calls include `__init__`, `tensor`, `get_current_vllm_config`, `ValueError`, `super`. It touches state such as `kv_cache`, `head_dim`, `window_size`, `prefix`, `cache_config`, `dtype`, `block_size`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`DeepseekV4SWACache`。 关键调用包括 `__init__`, `tensor`, `get_current_vllm_config`, `ValueError`, `super`。 它会读写 `kv_cache`, `head_dim`, `window_size`, `prefix`, `cache_config`, `dtype`, `block_size` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekV4SWACache.get_kv_cache_spec` method / `DeepseekV4SWACache.get_kv_cache_spec` 方法
```python
    def get_kv_cache_spec(self, vllm_config: VllmConfig) -> KVCacheSpec:
        return SlidingWindowMLASpec(
            block_size=self.block_size,
            num_kv_heads=1,
            head_size=self.head_dim,
            dtype=self.dtype,
            sliding_window=self.window_size,
            cache_dtype_str=self.cache_config.cache_dtype,
            alignment=576,  # NOTE: FlashMLA requires 576B alignment
            model_version="deepseek_v4",
        )
```
**EN:** This method returns or derives a value within `DeepseekV4SWACache`. Key calls include `SlidingWindowMLASpec`.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekV4SWACache`。 关键调用包括 `SlidingWindowMLASpec`。

### `DeepseekV4SWACache.forward` method / `DeepseekV4SWACache.forward` 方法
```python
    def forward(self): ...
```
**EN:** This method drives the forward-pass computation within `DeepseekV4SWACache`.
**CN:** 该方法会驱动前向计算流程，其作用域位于`DeepseekV4SWACache`。

### `DeepseekSparseSWABackend` class / `DeepseekSparseSWABackend` 类
```python
class DeepseekSparseSWABackend(AttentionBackend):
```
**EN:** Introduces the `DeepseekSparseSWABackend` class on top of `AttentionBackend`. Core methods include `get_name`, `get_supported_kernel_block_sizes`, `get_preferred_block_size`, `get_supported_head_sizes`, `get_builder_cls`, `get_kv_cache_shape`.
**CN:** 这里定义 `DeepseekSparseSWABackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_name`, `get_supported_kernel_block_sizes`, `get_preferred_block_size`, `get_supported_head_sizes`, `get_builder_cls`, `get_kv_cache_shape`。

### `DeepseekSparseSWABackend.get_builder_cls` method / `DeepseekSparseSWABackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["DeepseekSparseSWAMetadataBuilder"]:
        if current_platform.is_rocm():
            from vllm.v1.attention.backends.mla.rocm_aiter_mla_sparse_dsv4 import (
                DeepseekV4ROCMAiterSparseSWAMetadataBuilder,
            )

            return DeepseekV4ROCMAiterSparseSWAMetadataBuilder
        return DeepseekSparseSWAMetadataBuilder
```
**EN:** This method returns or derives a value within `DeepseekSparseSWABackend`. Key calls include `is_rocm`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekSparseSWABackend`。 关键调用包括 `is_rocm`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekSparseSWABackend.get_kv_cache_shape` method / `DeepseekSparseSWABackend.get_kv_cache_shape` 方法
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,
        head_size: int,
        cache_dtype_str: str = "auto",
    ) -> tuple[int, ...]:
        assert num_kv_heads == 1
        if cache_dtype_str == "fp8_ds_mla":
            # DeepseekV4 SWA: 584B per token (448 NoPE + 128 RoPE + 8 fp8 scale).
            # head_size passed in is the semantic head_dim (512).
            return (num_blocks, block_size, 584)
        else:
            return (num_blocks, block_size, head_size)
```
**EN:** This method returns or derives a value within `DeepseekSparseSWABackend`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekSparseSWABackend`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekSparseSWABackend.get_kv_cache_stride_order` method / `DeepseekSparseSWABackend.get_kv_cache_stride_order` 方法
```python
    @staticmethod
    def get_kv_cache_stride_order(
        include_num_layers_dimension: bool = False,
    ) -> tuple[int, ...]:
        if include_num_layers_dimension:
            return (0, 1, 2, 3)
        return (0, 1, 2)
```
**EN:** This method returns or derives a value within `DeepseekSparseSWABackend`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekSparseSWABackend`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekSparseSWAMetadata` class / `DeepseekSparseSWAMetadata` 类
```python
@dataclass
class DeepseekSparseSWAMetadata:
    block_table: torch.Tensor
    slot_mapping: torch.Tensor
    block_size: int
    seq_lens: torch.Tensor | None = None  # [num_seqs]
    query_start_loc: torch.Tensor | None = None  # [num_seqs + 1]
    query_start_loc_cpu: torch.Tensor | None = None  # [num_seqs + 1]

    is_valid_token: torch.Tensor | None = None  # [num_tokens]
    token_to_req_indices: torch.Tensor | None = None  # [num_tokens]
    decode_swa_indices: torch.Tensor | None = None  # [num_decode_tokens, window_size]
    decode_swa_lens: torch.Tensor | None = None  # [num_decode_tokens]

    # Number of decode/prefill requests/tokens (batch is reordered: decodes first)
    num_decodes: int = 0
    num_prefills: int = 0
    num_decode_tokens: int = 0
    num_prefill_tokens: int = 0

    # Pre-computed prefill metadata shared across all DeepseekV4 attention layers.
    prefill_seq_lens: torch.Tensor | None = None
    prefill_gather_lens: torch.Tensor | None = None

    # Per-layer-type FlashMLA tile-scheduler metadata. One FlashMLASchedMeta
    # per present DeepseekV4 layer type, shared across all ~60 layers of that type
    # within a decode step. The first forward call of a given type triggers
    # the in-kernel planner (which also allocates tile_scheduler_metadata and
    # num_splits via PyTorch's graph-aware allocator); subsequent same-type
    # calls skip planning and reuse the plan. Fresh instance per build(), so
    # have_initialized is always False at the start of a step and the plan
    # is re-derived from current seq_lens / topk_length on replay.
    # None for layer types the model does not use (or when num_decode_tokens
    # is zero).
    tile_sched_swaonly: "FlashMLASchedMeta | None" = None
    tile_sched_c4a: "FlashMLASchedMeta | None" = None
    tile_sched_c128a: "FlashMLASchedMeta | None" = None
```
**EN:** Uses `@dataclass` to package related state for `DeepseekSparseSWAMetadata`. Typical fields include `block_table`, `slot_mapping`, `block_size`, `seq_lens`, `query_start_loc`, `query_start_loc_cpu`.
**CN:** `DeepseekSparseSWAMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `block_table`, `slot_mapping`, `block_size`, `seq_lens`, `query_start_loc`, `query_start_loc_cpu`。

### `DeepseekSparseSWAMetadataBuilder` class / `DeepseekSparseSWAMetadataBuilder` 类
```python
class DeepseekSparseSWAMetadataBuilder(AttentionMetadataBuilder):
    """Builds metadata for DeepseekV4 SWA cache.

    Similar to the indexer, this handles mixed batches by:
    1. Using split_decodes_and_prefills() to determine the boundary
    2. Building separate metadata for decode and prefill portions

    Supports:
    - Mixed decode/prefill batches
    - MTP (Multi-Token Prediction) where decode has query_len > 1
    - Chunked prefill (aligns with the indexer's chunking)
    """

    # Base threshold: query_len <= 1 is decode
    reorder_batch_threshold: int = 1
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.UNIFORM_BATCH
```
**EN:** Introduces the `DeepseekSparseSWAMetadataBuilder` class on top of `AttentionMetadataBuilder`. Core methods include `__init__`, `build`, `build_tile_scheduler`, `_build_deepseek_v4_metadata`. Docstring signal: Builds metadata for DeepseekV4 SWA cache.
**CN:** 这里定义 `DeepseekSparseSWAMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder`。核心方法包括 `__init__`, `build`, `build_tile_scheduler`, `_build_deepseek_v4_metadata`。

### `DeepseekSparseSWAMetadataBuilder.__init__` method / `DeepseekSparseSWAMetadataBuilder.__init__` 方法
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        assert isinstance(self.kv_cache_spec, SlidingWindowMLASpec | MLAAttentionSpec)
        mla_spec = cast(SlidingWindowMLASpec | MLAAttentionSpec, self.kv_cache_spec)
        self.head_size = mla_spec.head_size  # Already considered quantization.
        self.compress_ratio = mla_spec.compress_ratio
        self.block_size = mla_spec.block_size

        # Handle MTP: adjust decode_threshold like the indexer does
        self.num_speculative_tokens = (
            self.vllm_config.speculative_config.num_speculative_tokens
            if self.vllm_config.speculative_config
            else 0
        )
        # With MTP, decode can have query_len up to 1 + num_speculative_tokens.
        # Must match the threshold used by the indexer and flashmla_sparse so
        # that all backends agree on the decode/prefill split.
        self.decode_threshold = (
            self.reorder_batch_threshold + self.num_speculative_tokens
        )

        hf_config = self.vllm_config.model_config.hf_config
        assert hasattr(hf_config, "sliding_window")
        self.window_size = hf_config.sliding_window

        # Detect which DeepseekV4 layer types this model uses so we only build a
        # FlashMLA tile-scheduler plan for types that will actually be called.
        # Models without compress_ratios (pure SWA) fall back to swaonly.
        compress_ratios = getattr(hf_config, "compress_ratios", None) or [1]
        self._layer_types: set[str] = set()
        for ratio in compress_ratios:
            self._layer_types.add(_layer_type_for(int(ratio)))

        max_tokens = self.vllm_config.scheduler_config.max_num_batched_tokens
        self.token_to_req_indices = torch.zeros(
            max_tokens,
            dtype=torch.int32,
            device=self.device,
        )
        self.decode_swa_indices = torch.zeros(
            max_tokens,
            1,
            self.window_size,
            dtype=torch.int32,
            device=self.device,
        )
        self.decode_swa_lens = torch.zeros(
            max_tokens,
            dtype=torch.int32,
            device=self.device,
        )
        self.is_valid_token = torch.zeros(
            max_tokens,
            dtype=torch.bool,
            device=self.device,
        )
```
**EN:** This method initializes the object state within `DeepseekSparseSWAMetadataBuilder`. Key calls include `__init__`, `isinstance`, `cast`, `hasattr`, `set`, `zeros`. It touches state such as `head_size`, `compress_ratio`, `block_size`, `num_speculative_tokens`, `decode_threshold`, `window_size`, `_layer_types`, `token_to_req_indices`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`DeepseekSparseSWAMetadataBuilder`。 关键调用包括 `__init__`, `isinstance`, `cast`, `hasattr`, `set`, `zeros`。 它会读写 `head_size`, `compress_ratio`, `block_size`, `num_speculative_tokens`, `decode_threshold`, `window_size`, `_layer_types`, `token_to_req_indices` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekSparseSWAMetadataBuilder.build` method / `DeepseekSparseSWAMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> DeepseekSparseSWAMetadata:
        """Build SWA metadata for mixed decode/prefill batches.

        The batch is assumed to be reordered with decodes first (by vLLM scheduler).
        We use split_decodes_and_prefills() to find the boundary, then build
        separate window_topk_idxs for each portion.

        For prefill, we use chunked prefill to align with the indexer's chunking.
        """
        num_reqs = common_attn_metadata.num_reqs
        seq_lens = common_attn_metadata.seq_lens
        query_start_loc = common_attn_metadata.query_start_loc
        query_start_loc_cpu = common_attn_metadata.query_start_loc_cpu
        block_table = common_attn_metadata.block_table_tensor
        slot_mapping = common_attn_metadata.slot_mapping

        # Split into decode and prefill portions using configurable threshold
        (num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens) = (
            split_decodes_and_prefills(
                common_attn_metadata, decode_threshold=self.decode_threshold
            )
        )

        # NOTE: Ensure all metadata tensors maintain fixed memory addresses
        # for CUDA graph compatibility.
        query_lens = query_start_loc_cpu[1:] - query_start_loc_cpu[:-1]
        x = torch.repeat_interleave(torch.arange(num_reqs), query_lens).pin_memory()
        token_to_req_indices = self.token_to_req_indices[: x.shape[0]]
        token_to_req_indices.copy_(x, non_blocking=True)

        is_valid_token = self.is_valid_token[: slot_mapping.shape[0]]
        is_valid_token.copy_(slot_mapping >= 0)

        if num_decode_tokens > 0:
            self.decode_swa_lens[num_decode_tokens:] = 0
            _compute_swa_indices_and_lens_kernel[(num_decode_tokens,)](
                self.decode_swa_indices,
                self.decode_swa_indices.stride(0),
                self.decode_swa_lens,
                self.window_size,
                query_start_loc,
                seq_lens,
                token_to_req_indices,
                is_valid_token,
                block_table,
                block_table.stride(0),
                self.block_size,
                TRITON_BLOCK_SIZE=1024,
            )

        # Pre-compute DeepseekV4 prefill metadata shared across all attention layers.
        deepseek_v4_fields = self._build_deepseek_v4_metadata(
            num_decodes,
            num_prefills,
            seq_lens,
            query_start_loc,
        )

        # Per-layer-type tile-scheduler plan holders. Empty FlashMLASchedMeta
        # per present DeepseekV4 layer type; the first flash_mla_with_kvcache call of
        # each type triggers the planner and all same-type layers reuse the
        # resulting plan for the rest of the step.
        tile_sched = self.build_tile_scheduler(num_decode_tokens)

        return DeepseekSparseSWAMetadata(
            seq_lens=seq_lens,
            query_start_loc=query_start_loc,
            query_start_loc_cpu=query_start_loc_cpu,
            block_table=block_table,
            slot_mapping=slot_mapping,
            is_valid_token=is_valid_token,
            token_to_req_indices=token_to_req_indices,
            decode_swa_indices=self.decode_swa_indices[:num_decode_tokens],
            decode_swa_lens=self.decode_swa_lens[:num_decode_tokens],
            block_size=self.block_size,
            num_decodes=num_decodes,
            num_prefills=num_prefills,
            num_decode_tokens=num_decode_tokens,
            num_prefill_tokens=num_prefill_tokens,
            tile_sched_swaonly=tile_sched[_LAYER_TYPE_SWAONLY],
            tile_sched_c4a=tile_sched[_LAYER_TYPE_C4A],
            tile_sched_c128a=tile_sched[_LAYER_TYPE_C128A],
            **deepseek_v4_fields,
        )
```
**EN:** This method builds derived structures within `DeepseekSparseSWAMetadataBuilder`. The docstring frames it as: Build SWA metadata for mixed decode/prefill batches. Key calls include `split_decodes_and_prefills`, `pin_memory`, `copy_`, `_build_deepseek_v4_metadata`, `build_tile_scheduler`, `DeepseekSparseSWAMetadata`. It touches state such as `decode_swa_lens`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`DeepseekSparseSWAMetadataBuilder`。 关键调用包括 `split_decodes_and_prefills`, `pin_memory`, `copy_`, `_build_deepseek_v4_metadata`, `build_tile_scheduler`, `DeepseekSparseSWAMetadata`。 它会读写 `decode_swa_lens` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekSparseSWAMetadataBuilder.build_tile_scheduler` method / `DeepseekSparseSWAMetadataBuilder.build_tile_scheduler` 方法
```python
    def build_tile_scheduler(
        self, num_decode_tokens: int
    ) -> dict[str, FlashMLASchedMeta | None]:
        """Allocate one empty ``FlashMLASchedMeta`` per present DeepseekV4 layer type.

        Returned instances have ``tile_scheduler_metadata`` / ``num_splits``
        set to ``None``; the FlashMLA C++ decode path will allocate them and
        run the tile-scheduler planner on the first ``flash_mla_with_kvcache``
        call of each type. Subsequent same-type calls reuse the plan because
        the tensors (and ``have_initialized``) are populated on the struct.

        Returns all-``None`` when there are no decode tokens this step, so
        ``_forward_decode`` sees a clean sentinel.
        """
        out: dict[str, FlashMLASchedMeta | None] = {
            _LAYER_TYPE_SWAONLY: None,
            _LAYER_TYPE_C4A: None,
            _LAYER_TYPE_C128A: None,
        }
        if num_decode_tokens == 0 or current_platform.is_rocm():
            return out
        for layer_type in self._layer_types:
            # get_mla_metadata() is the official FlashMLA entry point that
            # returns a fresh empty FlashMLASchedMeta; using it keeps this
            # call site aligned with the rest of the vLLM FlashMLA backends
            # that already go through the same stub.
            out[layer_type] = get_mla_metadata()[0]
        return out
```
**EN:** This method builds derived structures within `DeepseekSparseSWAMetadataBuilder`. The docstring frames it as: Allocate one empty ``FlashMLASchedMeta`` per present DeepseekV4 layer type. Key calls include `is_rocm`, `get_mla_metadata`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`DeepseekSparseSWAMetadataBuilder`。 关键调用包括 `is_rocm`, `get_mla_metadata`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `_compute_prefill_metadata_kernel` function / `_compute_prefill_metadata_kernel` 函数
```python
@triton.jit
def _compute_prefill_metadata_kernel(
    # Outputs
    prefill_gather_lens_ptr,
    # Inputs
    seq_lens_ptr,
    query_start_loc_ptr,
    num_prefills,
    num_decodes,
    window_size,
    BLOCK_SIZE: tl.constexpr,
):
    """Compute prefill gather_lens in a single pass."""
    offset = tl.arange(0, BLOCK_SIZE)
    mask = offset < num_prefills

    seq_len = tl.load(seq_lens_ptr + num_decodes + offset, mask=mask)
    qsl_start = tl.load(query_start_loc_ptr + num_decodes + offset, mask=mask)
    qsl_end = tl.load(query_start_loc_ptr + num_decodes + offset + 1, mask=mask)

    query_len = qsl_end - qsl_start
    prefix_len = seq_len - query_len
    gather_len = query_len + tl.minimum(prefix_len, window_size - 1)

    tl.store(prefill_gather_lens_ptr + offset, gather_len, mask=mask)
```
**EN:** This function implements `_compute_prefill_metadata_kernel` within the module. The docstring frames it as: Compute prefill gather_lens in a single pass. Key calls include `arange`, `load`, `store`, `minimum`.
**CN:** 该函数会实现 `_compute_prefill_metadata_kernel`，其作用域位于the module。 关键调用包括 `arange`, `load`, `store`, `minimum`。

### `_compute_swa_indices_and_lens_kernel` function / `_compute_swa_indices_and_lens_kernel` 函数
```python
@triton.jit
def _compute_swa_indices_and_lens_kernel(
    swa_indices_ptr,
    swa_indices_stride,
    swa_lens_ptr,
    window_size,
    query_start_loc_ptr,
    seq_lens_ptr,
    token_to_req_indices_ptr,
    is_valid_token_ptr,
    block_table_ptr,
    block_table_stride,
    block_size,
    TRITON_BLOCK_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(0)
    is_valid = tl.load(is_valid_token_ptr + token_idx)
    if not is_valid:
        tl.store(swa_lens_ptr + token_idx, 0)
        return

    req_idx = tl.load(token_to_req_indices_ptr + token_idx)

    query_start = tl.load(query_start_loc_ptr + req_idx)
    query_end = tl.load(query_start_loc_ptr + req_idx + 1)
    query_len = query_end - query_start

    seq_len = tl.load(seq_lens_ptr + req_idx)
    prefix_len = seq_len - query_len

    pos = prefix_len + token_idx - query_start
    start_pos = tl.maximum(pos - window_size + 1, 0)
    end_pos = pos + 1

    swa_len = end_pos - start_pos
    tl.store(swa_lens_ptr + token_idx, swa_len)

    for i in range(0, window_size, TRITON_BLOCK_SIZE):
        offset = i + tl.arange(0, TRITON_BLOCK_SIZE)

        pos_offset = start_pos + offset
        block_indices = pos_offset // block_size
        block_numbers = tl.load(
            block_table_ptr + req_idx * block_table_stride + block_indices,
            mask=pos_offset < end_pos,
        )
        block_offsets = pos_offset % block_size
        slot_ids = block_numbers * block_size + block_offsets

        slot_ids = tl.where(offset < swa_len, slot_ids, -1)
        tl.store(
            swa_indices_ptr + token_idx * swa_indices_stride + offset,
            slot_ids,
            mask=offset < window_size,
        )
```
**EN:** This function implements `_compute_swa_indices_and_lens_kernel` within the module. Key calls include `program_id`, `load`, `maximum`, `store`, `range`, `where`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_compute_swa_indices_and_lens_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `maximum`, `store`, `range`, `where`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_layer_type_for`: top-level helper or orchestration entry point. / `_layer_type_for`：顶层辅助函数或编排入口。
- `DeepseekV4SWACache`: central class or interface in this module. / `DeepseekV4SWACache`：本模块中的核心类或接口。
- `DeepseekSparseSWABackend`: central class or interface in this module. / `DeepseekSparseSWABackend`：本模块中的核心类或接口。
- `DeepseekSparseSWAMetadata`: central class or interface in this module. / `DeepseekSparseSWAMetadata`：本模块中的核心类或接口。
- `DeepseekSparseSWAMetadataBuilder`: central class or interface in this module. / `DeepseekSparseSWAMetadataBuilder`：本模块中的核心类或接口。
- `_compute_prefill_metadata_kernel`: top-level helper or orchestration entry point. / `_compute_prefill_metadata_kernel`：顶层辅助函数或编排入口。
- `_compute_swa_indices_and_lens_kernel`: top-level helper or orchestration entry point. / `_compute_swa_indices_and_lens_kernel`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.model_executor.layers.attention_layer_base`, `vllm.platforms`, `vllm.triton_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.v1.attention.ops.flashmla`, `vllm.v1.kv_cache_interface`, `vllm.v1.attention.backends.mla.rocm_aiter_mla_sparse_dsv4`
