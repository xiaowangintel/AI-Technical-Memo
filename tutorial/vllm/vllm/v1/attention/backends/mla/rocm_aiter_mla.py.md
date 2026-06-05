# rocm_aiter_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/rocm_aiter_mla.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_fp8_mla_prefill_supported`, `AiterMLABackend`, `AiterMLADecodeMetadata` for the V1 `attention/backends/mla` subsystem. / 为 V1 的 `attention/backends/mla` 子系统实现 `_fp8_mla_prefill_supported`, `AiterMLABackend`, `AiterMLADecodeMetadata`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import functools
from dataclasses import dataclass
from typing import ClassVar, Final

import torch

from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import VllmConfig
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.mla_attention import (
    MLACommonBackend,
    MLACommonDecodeMetadata,
    MLACommonImpl,
    MLACommonMetadata,
    MLACommonMetadataBuilder,
    QueryLenSupport,
)
from vllm.triton_utils import tl, triton
from vllm.v1.attention.backend import (
    AttentionCGSupport,
    AttentionLayer,
    CommonAttentionMetadata,
    MultipleOf,
)
from vllm.v1.kv_cache_interface import AttentionSpec

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_fp8_mla_prefill_supported` function / `_fp8_mla_prefill_supported` 函数
```python
@functools.lru_cache(maxsize=1)
def _fp8_mla_prefill_supported() -> bool:
    """Auto-detect FP8 MLA prefill via mla_prefill_ps_asm_fwd + mla_reduce_v1.

    Requires gfx950 plus an AITER build that exports both kernels.  When
    either is missing we silently fall back to ``flash_attn_varlen_func``.
    """
    try:
        from vllm.platforms.rocm import on_gfx950
    except Exception:  # noqa: BLE001
        return False
    if not on_gfx950():
        return False
    try:
        from aiter import mla_prefill_ps_asm_fwd, mla_reduce_v1  # noqa: F401
    except Exception:  # noqa: BLE001
        return False
    return True
```
**EN:** This function implements `_fp8_mla_prefill_supported` within the module. The docstring frames it as: Auto-detect FP8 MLA prefill via mla_prefill_ps_asm_fwd + mla_reduce_v1. Key calls include `lru_cache`, `on_gfx950`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_fp8_mla_prefill_supported`，其作用域位于the module。 关键调用包括 `lru_cache`, `on_gfx950`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AiterMLABackend` class / `AiterMLABackend` 类
```python
class AiterMLABackend(MLACommonBackend):
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
**EN:** Introduces the `AiterMLABackend` class on top of `MLACommonBackend`. Core methods include `get_supported_head_sizes`, `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`.
**CN:** 这里定义 `AiterMLABackend` 类，其基类包括 `MLACommonBackend`。核心方法包括 `get_supported_head_sizes`, `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`。

### `AiterMLABackend.get_supported_head_sizes` method / `AiterMLABackend.get_supported_head_sizes` 方法
```python
    @classmethod
    def get_supported_head_sizes(cls) -> list[int]:
        return []
```
**EN:** This method returns or derives a value within `AiterMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AiterMLABackend`。

### `AiterMLABackend.get_supported_kernel_block_sizes` method / `AiterMLABackend.get_supported_kernel_block_sizes` 方法
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        # The aiter MLA decode kernel always operates with page_size=1
        # internally (the wrapper flattens kv_buffer via .view(-1, 1, 1, H)).
        # We support any kernel_block_size by expanding block-level indices
        # into per-token flat indices in the metadata builder.
        return [MultipleOf(1)]
```
**EN:** This method returns or derives a value within `AiterMLABackend`. Key calls include `MultipleOf`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AiterMLABackend`。 关键调用包括 `MultipleOf`。

### `AiterMLABackend.get_builder_cls` method / `AiterMLABackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["AiterMLAMetadataBuilder"]:
        return AiterMLAMetadataBuilder
```
**EN:** This method returns or derives a value within `AiterMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AiterMLABackend`。

### `AiterMLADecodeMetadata` class / `AiterMLADecodeMetadata` 类
```python
@dataclass
class AiterMLADecodeMetadata(MLACommonDecodeMetadata):
    # The indptr of the paged kv cache, shape: [batch_size + 1]
    paged_kv_indptr: torch.Tensor | None = None
    # The page indices of the paged kv cache
    paged_kv_indices: torch.Tensor | None = None
    # The number of entries in the last page of each request in
    # the paged kv cache, shape: [batch_size]
    paged_kv_last_page_len: torch.Tensor | None = None
    # The query indptr, shape : [num_decode + 1]
    qo_indptr: torch.Tensor | None = None
    # The dtype of MLA out tensor
    attn_out_dtype: torch.dtype = torch.bfloat16
    # The max query output length: int
    max_qo_len: int | None = None
    # Whether persistent MLA metadata was computed (only for qseqlen=1)
    has_persistent_metadata: bool = False
```
**EN:** Uses `@dataclass` to package related state for `AiterMLADecodeMetadata`. Typical fields include `paged_kv_indptr`, `paged_kv_indices`, `paged_kv_last_page_len`, `qo_indptr`, `attn_out_dtype`, `max_qo_len`.
**CN:** `AiterMLADecodeMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `paged_kv_indptr`, `paged_kv_indices`, `paged_kv_last_page_len`, `qo_indptr`, `attn_out_dtype`, `max_qo_len`。

### `AiterMLAMetadata` class / `AiterMLAMetadata` 类
```python
@dataclass
class AiterMLAMetadata(MLACommonMetadata[AiterMLADecodeMetadata]):
    work_meta_data: torch.Tensor | None = None
    work_indptr: torch.Tensor | None = None
    work_info_set: torch.Tensor | None = None
    reduce_indptr: torch.Tensor | None = None
    reduce_final_map: torch.Tensor | None = None
    reduce_partial_map: torch.Tensor | None = None

    # FP8 ASM prefill persistent-scheduling (PS) metadata.  Populated by
    # AiterMLAMetadataBuilder._build_fp8_prefill_ps_metadata when prefill
    # tokens are present and FP8 MLA prefill is supported on the device.
    # Left as None on hosts/configs that fall back to flash_attn_varlen_func.
    fp8_prefill_qo_indptr: torch.Tensor | None = None
    fp8_prefill_kv_indptr: torch.Tensor | None = None
    fp8_prefill_kv_indices: torch.Tensor | None = None
    fp8_prefill_work_indptr: torch.Tensor | None = None
    fp8_prefill_work_info_set: torch.Tensor | None = None
    fp8_prefill_reduce_indptr: torch.Tensor | None = None
    fp8_prefill_reduce_final_map: torch.Tensor | None = None
    fp8_prefill_reduce_partial_map: torch.Tensor | None = None
    fp8_prefill_max_q_len: int | None = None
    fp8_prefill_num_partial_tiles: int | None = None
```
**EN:** Uses `@dataclass` to package related state for `AiterMLAMetadata`. Typical fields include `work_meta_data`, `work_indptr`, `work_info_set`, `reduce_indptr`, `reduce_final_map`, `reduce_partial_map`.
**CN:** `AiterMLAMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `work_meta_data`, `work_indptr`, `work_info_set`, `reduce_indptr`, `reduce_final_map`, `reduce_partial_map`。

### Module constants / 模块常量
```python
_FP8_PREFILL_TILE_Q = 256
```
**EN:** Defines module-level constants or aliases such as `_FP8_PREFILL_TILE_Q`, which are reused by later definitions.
**CN:** 定义 `_FP8_PREFILL_TILE_Q` 等模块级常量或别名，供后续定义复用。

### `AiterMLAMetadataBuilder` class / `AiterMLAMetadataBuilder` 类
```python
class AiterMLAMetadataBuilder(MLACommonMetadataBuilder[AiterMLAMetadata]):
    # TODO(luka, lucas): audit this as part of:
    #  https://github.com/vllm-project/vllm/issues/22945
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.UNIFORM_BATCH
    query_len_support: ClassVar[QueryLenSupport] = QueryLenSupport.UNIFORM
```
**EN:** Introduces the `AiterMLAMetadataBuilder` class on top of `MLACommonMetadataBuilder[AiterMLAMetadata]`. Core methods include `__init__`, `_init_fp8_prefill_ps_buffers`, `_build_fp8_prefill_ps_metadata`, `_build_decode`, `build`.
**CN:** 这里定义 `AiterMLAMetadataBuilder` 类，其基类包括 `MLACommonMetadataBuilder[AiterMLAMetadata]`。核心方法包括 `__init__`, `_init_fp8_prefill_ps_buffers`, `_build_fp8_prefill_ps_metadata`, `_build_decode`, `build`。

### `AiterMLAMetadataBuilder.__init__` method / `AiterMLAMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(
            kv_cache_spec, layer_names, vllm_config, device, AiterMLAMetadata
        )

        self.compilation_config = vllm_config.compilation_config
        self.decode_attn_out_dtype = vllm_config.model_config.dtype

        # Store the kernel block size from the spec. When kernel_block_size=1
        # (no spec-dec), behavior is identical to the original. When > 1
        # (e.g. 16 with Eagle3), we expand block-level indices into per-token
        # flat indices since the aiter kernel always uses page_size=1 internally.
        self.kernel_block_size = kv_cache_spec.block_size

        # In the flat view (.view(-1,1,1,H)), each token is its own page,
        # so max_num_pages_per_req = max_model_len regardless of
        # kernel_block_size.
        max_num_pages_per_req = vllm_config.model_config.max_model_len
        max_num_reqs = vllm_config.scheduler_config.max_num_seqs
        max_num_pages = max_num_reqs * max_num_pages_per_req

        # Preparing persistent buffers
        # TODO: we can disambiguate between decode and mixed-prefill decode here
        # so we can only use the persistent buffer if a cudagraph is actually
        # being used.

        # paged_kv_last_page_len is always 1s (the aiter kernel always sees
        # page_size=1 after .view(-1,1,1,H) flattening), so we create it
        # once and reuse slices in both eager and cudagraph modes.
        self.paged_kv_last_page_len = torch.ones(
            max_num_reqs, dtype=torch.int32, device=device
        )

        # Persistent buffer for paged_kv_indices to avoid blocking boolean mask
        # indexing (block_table_tensor[mask]) which has data-dependent output size.
        self.paged_kv_indices = torch.zeros(
            max_num_pages, dtype=torch.int32, device=device
        )

        from aiter import dtypes, get_mla_metadata_info_v1

        # For num_attention_heads < 16 (e.g. kimi-k2.5 head=8 with TP8),
        # make sure get_mla_metadata_info_v1 / get_mla_metadata_v1 are consistent
        # with the actual tensor shape passed to mla_decode_fwd.
    # ... omitted for brevity ...
        self._mla_reduce_final_map = torch.empty(
            reduce_final_map_size, dtype=reduce_final_map_type, device=device
        )
        self._mla_reduce_partial_map = torch.empty(
            reduce_partial_map_size,
            dtype=reduce_partial_map_type,
            device=device,
        )

        self._fp8_prefill_enabled = _fp8_mla_prefill_supported()
        if self._fp8_prefill_enabled:
            max_prefill_qlen = min(
                vllm_config.model_config.max_model_len,
                vllm_config.scheduler_config.max_num_batched_tokens,
            )
            self._init_fp8_prefill_ps_buffers(max_num_reqs, max_prefill_qlen, device)

        if self.compilation_config.cudagraph_mode.has_full_cudagraphs():
            self.paged_kv_indptr = torch.zeros(
                max_num_reqs + 1, dtype=torch.int32, device=device
            )

            self.qo_indptr = torch.zeros(
                max_num_reqs + 1, dtype=torch.int32, device=device
            )
```
**EN:** This method initializes the object state within `AiterMLAMetadataBuilder`. Key calls include `__init__`, `ones`, `zeros`, `max`, `getattr`, `get`. It touches state such as `compilation_config`, `decode_attn_out_dtype`, `kernel_block_size`, `paged_kv_last_page_len`, `paged_kv_indices`, `_num_attention_heads`, `_mla_work_meta_data`, `_mla_work_indptr`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`AiterMLAMetadataBuilder`。 关键调用包括 `__init__`, `ones`, `zeros`, `max`, `getattr`, `get`。 它会读写 `compilation_config`, `decode_attn_out_dtype`, `kernel_block_size`, `paged_kv_last_page_len`, `paged_kv_indices`, `_num_attention_heads`, `_mla_work_meta_data`, `_mla_work_indptr` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `AiterMLAMetadataBuilder._build_fp8_prefill_ps_metadata` method / `AiterMLAMetadataBuilder._build_fp8_prefill_ps_metadata` 方法
```python
    def _build_fp8_prefill_ps_metadata(
        self,
        metadata: AiterMLAMetadata,
        common_attn_metadata: CommonAttentionMetadata,
    ) -> None:
        """Build per-batch FP8 MLA prefill PS metadata and attach to *metadata*.

        Called from ``build()`` when prefill tokens are present and
        FP8 MLA prefill is enabled (auto-detected via
        ``_fp8_mla_prefill_supported()``).
        """
        from aiter import get_ps_metadata_v1

        prefill = metadata.prefill
        # Caller (build()) only invokes this when prefill tokens exist, so
        # metadata.prefill is guaranteed non-None.  Assert to narrow for mypy.
        assert prefill is not None
        qo_indptr = prefill.query_start_loc
        kv_indptr = qo_indptr  # new tokens: KV length == Q length

        # Reuse the existing CPU view of query_start_loc instead of forcing a
        # device->host copy.  Prefill batches sit at the tail of the request
        # list, so we slice from num_decodes onwards and rebase to zero, the
        # same transform the parent build applies on device tensors.
        num_decodes = metadata.num_decodes
        qsl_cpu = common_attn_metadata.query_start_loc_cpu
        qo_indptr_cpu = (qsl_cpu[num_decodes:] - qsl_cpu[num_decodes]).to(torch.int32)
        kv_indptr_cpu = qo_indptr_cpu.clone()
        seq_lens_cpu = (qo_indptr_cpu[1:] - qo_indptr_cpu[:-1]).to(torch.int32)

        num_head_k = self.num_heads
        # gqa_ratio = 1
        # qhead_granularity = max(gqa_ratio, 1)
        # qlen_granularity = _FP8_PREFILL_TILE_Q // qhead_granularity
        gqa_ratio = 1
        qhead_granularity = 1
        qlen_granularity = _FP8_PREFILL_TILE_Q
        kvlen_granularity = 128
        block_size = 1  # non-paged: each "page" is one token

        get_ps_metadata_v1(
            qo_indptr_cpu,
            kv_indptr_cpu,
            seq_lens_cpu,
            gqa_ratio,
            num_head_k,
            self.fp8_ps_work_metadata,
            self.fp8_ps_work_indptr,
            self.fp8_ps_work_info,
            self.fp8_ps_reduce_indptr,
            self.fp8_ps_reduce_final_map,
            self.fp8_ps_reduce_partial_map,
            qhead_granularity=qhead_granularity,
            qlen_granularity=qlen_granularity,
            kvlen_granularity=kvlen_granularity,
            block_size=block_size,
            is_causal=True,
        )

        total_prefill_tokens = int(qo_indptr_cpu[-1].item())
        kv_indices = torch.arange(
            total_prefill_tokens, device=qo_indptr.device, dtype=torch.int32
        )

        # The actual number of active partial tiles for this batch is the
        # final value of reduce_indptr.  Resolving it here (during metadata
        # build) keeps it off the per-layer forward path where a sync would
        # break CUDA Graph capture.  Using the device-side reduce_indptr is
        # acceptable since build is allowed to incur an occasional sync.
        num_partial_tiles = int(self.fp8_ps_reduce_indptr[-1].item())

        # Attach PS metadata to the metadata object so forward_mha can read it.
        metadata.fp8_prefill_qo_indptr = qo_indptr
        metadata.fp8_prefill_kv_indptr = kv_indptr
        metadata.fp8_prefill_kv_indices = kv_indices
        metadata.fp8_prefill_work_indptr = self.fp8_ps_work_indptr
        metadata.fp8_prefill_work_info_set = self.fp8_ps_work_info
        metadata.fp8_prefill_reduce_indptr = self.fp8_ps_reduce_indptr
        metadata.fp8_prefill_reduce_final_map = self.fp8_ps_reduce_final_map
        metadata.fp8_prefill_reduce_partial_map = self.fp8_ps_reduce_partial_map
        metadata.fp8_prefill_max_q_len = prefill.max_query_len
        metadata.fp8_prefill_num_partial_tiles = num_partial_tiles
```
**EN:** This method implements `_build_fp8_prefill_ps_metadata` within `AiterMLAMetadataBuilder`. The docstring frames it as: Build per-batch FP8 MLA prefill PS metadata and attach to *metadata*. Key calls include `to`, `clone`, `get_ps_metadata_v1`, `int`, `arange`, `item`.
**CN:** 该方法会实现 `_build_fp8_prefill_ps_metadata`，其作用域位于`AiterMLAMetadataBuilder`。 关键调用包括 `to`, `clone`, `get_ps_metadata_v1`, `int`, `arange`, `item`。

### `AiterMLAMetadataBuilder._build_decode` method / `AiterMLAMetadataBuilder._build_decode` 方法
```python
    def _build_decode(
        self,
        block_table_tensor: torch.Tensor,
        seq_lens_device: torch.Tensor,
        max_seq_len: int,
        query_start_loc_cpu: torch.Tensor,
        query_start_loc_device: torch.Tensor,
        num_decode_tokens: int,
        dcp_tot_seq_lens_device: torch.Tensor | None,
    ) -> AiterMLADecodeMetadata:
        device = self.device
        num_reqs = seq_lens_device.size(0)

        # The aiter kernel always operates with page_size=1 (the wrapper
        # flattens kv_buffer). last_page_len is always 1.
        paged_kv_last_page_len = self.paged_kv_last_page_len[:num_reqs]

        # indptr: cumsum of seq_lens (one page per token in the flat view)
        paged_kv_indptr = torch.cat(
            [
                torch.zeros(1, dtype=seq_lens_device.dtype, device=device),
                seq_lens_device.cumsum(dim=0, dtype=torch.int32),
            ]
        )
        qo_len = query_start_loc_cpu[1:] - query_start_loc_cpu[:-1]
        max_qo_len = qo_len.max().item()

        if self.compilation_config.cudagraph_mode.has_full_cudagraphs():
            self.paged_kv_indices.fill_(-1)

        # Expand block_table entries into per-token flat indices.
        # When kernel_block_size=1, this degrades to a direct copy (identical
        # to the original _copy_page_indices_kernel).
        # When kernel_block_size=K>1, block_table entry b covering K tokens
        # gets expanded to flat indices b*K, b*K+1, ..., b*K+(K-1).
        _expand_page_indices_kernel[(num_reqs,)](
            self.paged_kv_indices,
            block_table_tensor,
            block_table_tensor.stride(0),
            paged_kv_indptr,
            seq_lens_device,
            KERNEL_BLOCK_SIZE=self.kernel_block_size,
            BLOCK_SIZE=1024,
        )
        paged_kv_indices = self.paged_kv_indices

        if self.compilation_config.cudagraph_mode.has_full_cudagraphs():
            self.paged_kv_indptr[: 1 + num_reqs].copy_(
                paged_kv_indptr, non_blocking=True
            )
    # ... omitted for brevity ...
                self._mla_reduce_indptr,
                self._mla_reduce_final_map,
                self._mla_reduce_partial_map,
                page_size=1,
                kv_granularity=16,
                max_seqlen_qo=max_qo_len,
                uni_seqlen_qo=max_qo_len,
                fast_mode=True,
            )
            has_persistent_metadata = True

        attn_metadata = AiterMLADecodeMetadata(
            block_table=block_table_tensor,
            seq_lens=seq_lens_device,
            paged_kv_indptr=paged_kv_indptr,
            paged_kv_indices=paged_kv_indices,
            paged_kv_last_page_len=paged_kv_last_page_len,
            qo_indptr=qo_indptr,
            dcp_tot_seq_lens=dcp_tot_seq_lens_device,
            max_qo_len=max_qo_len,
            attn_out_dtype=self.decode_attn_out_dtype,
            has_persistent_metadata=has_persistent_metadata,
        )

        return attn_metadata
```
**EN:** This method implements `_build_decode` within `AiterMLAMetadataBuilder`. Key calls include `size`, `cat`, `item`, `has_full_cudagraphs`, `AiterMLADecodeMetadata`, `fill_`. It touches state such as `qo_indptr`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_build_decode`，其作用域位于`AiterMLAMetadataBuilder`。 关键调用包括 `size`, `cat`, `item`, `has_full_cudagraphs`, `AiterMLADecodeMetadata`, `fill_`。 它会读写 `qo_indptr` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `_expand_page_indices_kernel` function / `_expand_page_indices_kernel` 函数
```python
@triton.jit
def _expand_page_indices_kernel(
    page_indices,
    block_table,
    block_table_stride,
    cu_num_tokens,
    seq_lens,
    KERNEL_BLOCK_SIZE: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    """Expand block table entries into per-token flat page indices.

    The aiter MLA kernel always operates with page_size=1 internally
    (kv_buffer is flattened via .view(-1, 1, 1, H)). This kernel converts
    block-level indices from the block table into individual token positions
    in the flattened KV buffer.

    When KERNEL_BLOCK_SIZE=1: block_idx=t, offset=0, flat=block_id
    (equivalent to a direct copy -- no regression from the original kernel).

    When KERNEL_BLOCK_SIZE=K: block table entry b (covering K tokens)
    is expanded to flat indices b*K, b*K+1, ..., b*K+(K-1).
    """
    req_idx = tl.program_id(0)
    row_ptr = block_table + req_idx * block_table_stride
    start_idx = tl.load(cu_num_tokens + req_idx)
    num_tokens = tl.load(seq_lens + req_idx)

    offset = tl.arange(0, BLOCK_SIZE)
    for i in tl.range(0, num_tokens, BLOCK_SIZE):
        token_offsets = i + offset
        mask = token_offsets < num_tokens

        # Which block in the block table does this token belong to?
        block_idx = token_offsets // KERNEL_BLOCK_SIZE
        # Offset within that block
        offset_in_block = token_offsets % KERNEL_BLOCK_SIZE

        # Load the block ID from the block table
        block_ids = tl.load(row_ptr + block_idx, mask=mask)

        # Compute flat index in the flattened kv_buffer
        flat_indices = block_ids * KERNEL_BLOCK_SIZE + offset_in_block

        tl.store(
            page_indices + start_idx + token_offsets,
            flat_indices,
            mask=mask,
        )
```
**EN:** This function implements `_expand_page_indices_kernel` within the module. The docstring frames it as: Expand block table entries into per-token flat page indices. Key calls include `program_id`, `load`, `arange`, `range`, `store`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_expand_page_indices_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `arange`, `range`, `store`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `AiterMLAHelper` class / `AiterMLAHelper` 类
```python
class AiterMLAHelper:
    """
    AITER MLA implementation requires num_heads >= 16. If num_heads < 16 and
    16 % num_heads == 0, we can pad q to 16 heads; otherwise AITER has to fail.
    """

    _AITER_MIN_MLA_HEADS: Final = 16
    _AITER_UNSUPPORTED_HEADS: ClassVar[tuple[int, ...]] = ()
```
**EN:** Introduces the `AiterMLAHelper` class. Core methods include `check_num_heads_validity`, `is_valid_num_heads`, `get_actual_mla_num_heads`, `get_mla_padded_q`, `get_mla_unpadded_o`. Docstring signal: AITER MLA implementation requires num_heads >= 16.
**CN:** 这里定义 `AiterMLAHelper` 类。核心方法包括 `check_num_heads_validity`, `is_valid_num_heads`, `get_actual_mla_num_heads`, `get_mla_padded_q`, `get_mla_unpadded_o`。

### `AiterMLAHelper.check_num_heads_validity` method / `AiterMLAHelper.check_num_heads_validity` 方法
```python
    @staticmethod
    def check_num_heads_validity(num_heads: int):
        assert AiterMLAHelper.is_valid_num_heads(num_heads), (
            f"Aiter MLA requires that num_heads be multiples or divisors of 16, "
            f"but provided {num_heads} number of heads.\n"
            f"Try adjusting tensor_parallel_size value."
        )
```
**EN:** This method validates assumptions or constraints within `AiterMLAHelper`. Key calls include `is_valid_num_heads`.
**CN:** 该方法会校验前提与约束，其作用域位于`AiterMLAHelper`。 关键调用包括 `is_valid_num_heads`。

### `AiterMLAHelper.is_valid_num_heads` method / `AiterMLAHelper.is_valid_num_heads` 方法
```python
    @staticmethod
    def is_valid_num_heads(num_heads: int) -> bool:
        return (
            num_heads % AiterMLAHelper._AITER_MIN_MLA_HEADS == 0
            if num_heads >= AiterMLAHelper._AITER_MIN_MLA_HEADS
            else AiterMLAHelper._AITER_MIN_MLA_HEADS % num_heads == 0
        )
```
**EN:** This method answers a boolean capability check within `AiterMLAHelper`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会回答布尔能力判断，其作用域位于`AiterMLAHelper`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AiterMLAHelper.get_mla_padded_q` method / `AiterMLAHelper.get_mla_padded_q` 方法
```python
    @staticmethod
    def get_mla_padded_q(num_heads: int, q: torch.Tensor) -> torch.Tensor:
        return (
            q
            if num_heads >= AiterMLAHelper._AITER_MIN_MLA_HEADS
            else q.repeat_interleave(
                AiterMLAHelper._AITER_MIN_MLA_HEADS // num_heads, dim=1
            )
        )
```
**EN:** This method returns or derives a value within `AiterMLAHelper`. Key calls include `repeat_interleave`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AiterMLAHelper`。 关键调用包括 `repeat_interleave`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AiterMLAImpl` class / `AiterMLAImpl` 类
```python
class AiterMLAImpl(MLACommonImpl[AiterMLAMetadata]):
```
**EN:** Introduces the `AiterMLAImpl` class on top of `MLACommonImpl[AiterMLAMetadata]`. Core methods include `__init__`, `_flash_attn_varlen_diff_headdims`, `_mla_fp8_prefill_attn`, `forward_mha`, `forward_mqa`.
**CN:** 这里定义 `AiterMLAImpl` 类，其基类包括 `MLACommonImpl[AiterMLAMetadata]`。核心方法包括 `__init__`, `_flash_attn_varlen_diff_headdims`, `_mla_fp8_prefill_attn`, `forward_mha`, `forward_mqa`。

### `AiterMLAImpl._mla_fp8_prefill_attn` method / `AiterMLAImpl._mla_fp8_prefill_attn` 方法
```python
    def _mla_fp8_prefill_attn(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        attn_metadata: AiterMLAMetadata,
        out: torch.Tensor,
    ) -> None:
        """Run FP8 MLA prefill via mla_prefill_ps_asm_fwd + mla_reduce_v1.

        Q, K, V are already decompressed (post-kv_b_proj), so K and V have
        ``num_heads`` heads (same as Q) and gqa_ratio=1.  Writes the
        result in-place to ``out``, which is the [total_q, nhead * v_head_dim]
        output buffer supplied by ``forward_mha``; no extra allocation or
        copy is required.
        """
        from vllm.platforms import current_platform
        from vllm.v1.worker.workspace import current_workspace_manager

        fp8_dtype = current_platform.fp8_dtype()
        total_q = q.shape[0]
        nhead = self.num_heads
        v_head_dim = self.v_head_dim
        tile_q = _FP8_PREFILL_TILE_Q

        # The FP8 ASM kernel expects FP8 inputs; the q_scale/k_scale/v_scale
        # parameters select per-tensor dequant scales.  Q/K/V arrive as
        # bf16 from kv_b_proj, so cast here (one_scale=1.0 disables scaling).
        if q.dtype != fp8_dtype:
            q = q.to(fp8_dtype)
        if k.dtype != fp8_dtype:
            k = k.to(fp8_dtype)
        if v.dtype != fp8_dtype:
            v = v.to(fp8_dtype)

        one_scale = torch.ones((), dtype=torch.float32, device=q.device)

        # num_partial_tiles is resolved during metadata build to avoid an
        # in-forward .item() sync that would prevent CUDA Graph capture.
        # forward_mha gates the FP8 path on fp8_prefill_qo_indptr being set,
        # and the builder always sets every fp8_prefill_* field together, so
        # num_partial_tiles is non-None here.
        num_partial_tiles = attn_metadata.fp8_prefill_num_partial_tiles
        assert num_partial_tiles is not None

        # Reuse the caller's output buffer to skip the per-call alloc + copy.
        # The ASM and reduce kernels both write to a [total_q, nhead, v_head_dim]
        # view, which aliases the [total_q, nhead * v_head_dim] storage of out.
        out_3d = out.view(total_q, nhead, v_head_dim)

    # ... omitted for brevity ...
            attn_metadata.fp8_prefill_kv_indices,
            attn_metadata.fp8_prefill_work_indptr,
            attn_metadata.fp8_prefill_work_info_set,
            attn_metadata.fp8_prefill_max_q_len,
            self.scale,
            True,  # is_causal
            logits,
            attn_lse,
            out_3d,
            one_scale,
            one_scale,
            one_scale,
        )

        # Phase 2: reduction across KV splits.
        self._mla_reduce_v1(
            logits,
            attn_lse,
            attn_metadata.fp8_prefill_reduce_indptr,
            attn_metadata.fp8_prefill_reduce_final_map,
            attn_metadata.fp8_prefill_reduce_partial_map,
            tile_q,
            out_3d,
            final_lse,
        )
```
**EN:** This method implements `_mla_fp8_prefill_attn` within `AiterMLAImpl`. The docstring frames it as: Run FP8 MLA prefill via mla_prefill_ps_asm_fwd + mla_reduce_v1. Key calls include `fp8_dtype`, `ones`, `view`, `get_simultaneous`, `_mla_prefill_ps_asm_fwd`, `_mla_reduce_v1`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_mla_fp8_prefill_attn`，其作用域位于`AiterMLAImpl`。 关键调用包括 `fp8_dtype`, `ones`, `view`, `get_simultaneous`, `_mla_prefill_ps_asm_fwd`, `_mla_reduce_v1`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `AiterMLAImpl.forward_mha` method / `AiterMLAImpl.forward_mha` 方法
```python
    def forward_mha(
        self,
        q: torch.Tensor,
        kv_c_normed: torch.Tensor,
        k_pe: torch.Tensor,
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: MLACommonMetadata,
        k_scale: torch.Tensor,
        output: torch.Tensor,
    ) -> None:
        """Dispatch prefill to the FP8 ASM kernel when available.

        Falls back to the parent (``flash_attn_varlen_func``) when FP8
        MLA prefill is disabled, PS metadata is missing, or chunked
        context requires two-pass merge.

        The annotation uses the base ``MLACommonMetadata`` to honour LSP
        with ``MLACommonImpl.forward_mha``; the AITER builder always
        produces ``AiterMLAMetadata`` instances at runtime, so we narrow
        with ``isinstance`` before reading the AITER-specific FP8 fields.
        """
        if (
            not self._fp8_prefill_enabled
            or not isinstance(attn_metadata, AiterMLAMetadata)
            or attn_metadata.fp8_prefill_qo_indptr is None
        ):
            return super().forward_mha(
                q,
                kv_c_normed,
                k_pe,
                kv_c_and_k_pe_cache,
                attn_metadata,
                k_scale,
                output,
            )

        assert attn_metadata.prefill is not None
        prefill_metadata = attn_metadata.prefill
        has_context = prefill_metadata.chunked_context is not None

        if has_context:
            return super().forward_mha(
                q,
                kv_c_normed,
                k_pe,
                kv_c_and_k_pe_cache,
                attn_metadata,
                k_scale,
                output,
            )

        kv_nope = self.kv_b_proj(kv_c_normed)[0].view(
            -1, self.num_heads, self.qk_nope_head_dim + self.v_head_dim
        )
        k_nope, v = kv_nope.split([self.qk_nope_head_dim, self.v_head_dim], dim=-1)
        k = self._concat_k_nope_k_pe(k_nope, k_pe)

        self._mla_fp8_prefill_attn(q, k, v, attn_metadata, output)
```
**EN:** This method drives the forward-pass computation within `AiterMLAImpl`. The docstring frames it as: Dispatch prefill to the FP8 ASM kernel when available. Key calls include `view`, `split`, `_concat_k_nope_k_pe`, `_mla_fp8_prefill_attn`, `forward_mha`, `isinstance`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`AiterMLAImpl`。 关键调用包括 `view`, `split`, `_concat_k_nope_k_pe`, `_mla_fp8_prefill_attn`, `forward_mha`, `isinstance`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AiterMLAImpl.forward_mqa` method / `AiterMLAImpl.forward_mqa` 方法
```python
    def forward_mqa(
        self,
        q: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: AiterMLAMetadata,
        layer: AttentionLayer,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        assert kv_c_and_k_pe_cache.numel() > 0
        assert attn_metadata.decode is not None
        assert attn_metadata.decode.max_qo_len is not None

        if type(q) is tuple:
            q = torch.cat(q, dim=-1)

        assert isinstance(q, torch.Tensor)
        B = q.shape[0]

        mla_padded_q = AiterMLAHelper.get_mla_padded_q(self.num_heads, q)
        mla_num_heads = AiterMLAHelper.get_actual_mla_num_heads(self.num_heads)
        o = torch.empty(
            B,
            mla_num_heads,
            self.kv_lora_rank,
            dtype=attn_metadata.decode.attn_out_dtype,
            device=q.device,
        )

        kv_buffer = kv_c_and_k_pe_cache.unsqueeze(2)

        # Build kwargs for mla_decode_fwd. Pass persistent metadata only
        # when it was successfully computed (qseqlen=1 decode steps).
        # For multi-token verification steps (spec-dec), the kernel falls
        # back to computing metadata internally.
        mla_kwargs = dict(
            q_scale=layer._q_scale,
            kv_scale=layer._k_scale,
        )
        if attn_metadata.work_meta_data is not None:
            mla_kwargs.update(
                work_meta_data=attn_metadata.work_meta_data,
                work_indptr=attn_metadata.work_indptr,
                work_info_set=attn_metadata.work_info_set,
                reduce_indptr=attn_metadata.reduce_indptr,
                reduce_final_map=attn_metadata.reduce_final_map,
                reduce_partial_map=attn_metadata.reduce_partial_map,
            )

        rocm_aiter_ops.mla_decode_fwd(
            mla_padded_q,
            kv_buffer,
            o,
            self.scale,
            attn_metadata.decode.qo_indptr,
            attn_metadata.decode.max_qo_len,
            attn_metadata.decode.paged_kv_indptr,
            attn_metadata.decode.paged_kv_indices,
            attn_metadata.decode.paged_kv_last_page_len,
            **mla_kwargs,
        )

        return AiterMLAHelper.get_mla_unpadded_o(self.num_heads, o), None
```
**EN:** This method drives the forward-pass computation within `AiterMLAImpl`. Key calls include `isinstance`, `get_mla_padded_q`, `get_actual_mla_num_heads`, `empty`, `unsqueeze`, `dict`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`AiterMLAImpl`。 关键调用包括 `isinstance`, `get_mla_padded_q`, `get_actual_mla_num_heads`, `empty`, `unsqueeze`, `dict`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_fp8_mla_prefill_supported`: top-level helper or orchestration entry point. / `_fp8_mla_prefill_supported`：顶层辅助函数或编排入口。
- `AiterMLABackend`: central class or interface in this module. / `AiterMLABackend`：本模块中的核心类或接口。
- `AiterMLADecodeMetadata`: central class or interface in this module. / `AiterMLADecodeMetadata`：本模块中的核心类或接口。
- `AiterMLAMetadata`: central class or interface in this module. / `AiterMLAMetadata`：本模块中的核心类或接口。
- `AiterMLAMetadataBuilder`: central class or interface in this module. / `AiterMLAMetadataBuilder`：本模块中的核心类或接口。
- `_expand_page_indices_kernel`: top-level helper or orchestration entry point. / `_expand_page_indices_kernel`：顶层辅助函数或编排入口。
- `AiterMLAHelper`: central class or interface in this module. / `AiterMLAHelper`：本模块中的核心类或接口。
- `AiterMLAImpl`: central class or interface in this module. / `AiterMLAImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `functools`, `dataclasses`, `typing`
- External / 外部依赖: `torch`, `aiter`
- Internal vLLM / 内部依赖: `vllm._aiter_ops`, `vllm.config`, `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.triton_utils`, `vllm.v1.attention.backend`, `vllm.v1.kv_cache_interface`, `vllm.platforms.rocm`, `vllm.platforms`, `vllm.v1.worker.workspace`
