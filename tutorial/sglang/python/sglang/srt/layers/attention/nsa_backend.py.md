# nsa_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/nsa_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the nsa backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 nsa backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-39: imports
```python
from __future__ import annotations

from dataclasses import dataclass
from enum import IntEnum, auto
from typing import TYPE_CHECKING, Dict, List, Literal, Optional, Tuple, TypeAlias

import torch

from sglang.srt.configs.model_config import get_nsa_index_topk, is_deepseek_nsa
from sglang.srt.environ import envs
from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.layers.attention.nsa.dequant_k_cache import dequantize_k_cache_paged
from sglang.srt.layers.attention.nsa.nsa_backend_mtp_precompute import (
    NativeSparseAttnBackendMTPPrecomputeMixin,
    PrecomputedMetadata,
    compute_cu_seqlens,
)
from sglang.srt.layers.attention.nsa.nsa_indexer import BaseIndexerMetadata
from sglang.srt.layers.attention.nsa.quant_k_cache import quantize_k_cache
from sglang.srt.layers.attention.nsa.transform_index import (
    transform_index_page_table_decode,
    transform_index_page_table_prefill,
)
from sglang.srt.layers.attention.nsa.utils import (
    can_nsa_prefill_cp_round_robin_split,
    compute_nsa_seqlens,
    is_nsa_enable_prefill_cp,
    nsa_cp_round_robin_split_data,
    nsa_cp_round_robin_split_q_seqs,
    pad_nsa_cache_seqlens,
)
from sglang.srt.layers.attention.utils import (
    concat_mla_absorb_q_general,
    mla_quantize_and_rope_for_fp8,
    seqlens_expand_triton,
)
from sglang.srt.layers.dp_attention import get_attention_tp_size
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.utils import is_cuda, is_hip
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 41-44: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner
    from sglang.srt.speculative.spec_info import SpecInput
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 47-47: module constants
```python
_is_hip = is_hip()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 49-67: conditional branch
```python
if _is_hip:
    from sglang.srt.layers.attention.nsa.triton_kernel import get_valid_kv_indices

    try:
        from aiter import (  # noqa: F401
            flash_attn_varlen_func,
            mha_batch_prefill_func,
            paged_attention_ragged,
        )
        from aiter.mla import mla_decode_fwd, mla_prefill_fwd  # noqa: F401
    except ImportError:
        print(
            "aiter is AMD specific kernel library. Please make sure aiter is installed on your AMD device."
        )
else:
    from sglang.jit_kernel.flash_attention import (
        flash_attn_varlen_func,
        flash_attn_with_kvcache,
    )
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 70-78: function _to_2d_context_lens
```python
def _to_2d_context_lens(seqlens_32: torch.Tensor, batch_size: int) -> torch.Tensor:
    # Always normalize to (N_total, 1) layout, to avoid deadlock at deep_gemm.fp8_paged_mqa_logits
    if seqlens_32.dim() == 2:
        if seqlens_32.size(1) == 1:
            return seqlens_32
        # Fall through and re-flatten if the caller already gave us a (bs, next_n)
        # view — we want (N_total, 1) regardless.
        seqlens_32 = seqlens_32.reshape(-1)
    return seqlens_32.contiguous().view(-1, 1)
```
**EN:** Implements the to 2d context lens routine used by this attention module.
**CN:** 实现该注意力模块使用的 to 2d context lens 例程。

### Lines 82-86: module constants
```python
global_workspace_buffer = None

# Control whether to use fused metadata copy kernel for cuda graph replay (default: enabled)
# Set SGLANG_USE_FUSED_METADATA_COPY=0 or false to disable
_USE_FUSED_METADATA_COPY = envs.SGLANG_USE_FUSED_METADATA_COPY.get() and not _is_hip
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 89-95: class NSAFlashMLAMetadata
```python
@dataclass(frozen=True)
class NSAFlashMLAMetadata:
    """Metadata only needed by FlashMLA"""

    flashmla_metadata: torch.Tensor
    num_splits: torch.Tensor
```
**EN:** Dataclass-style container that stores structured runtime state for nsaflash mlametadata.
**CN:** 该数据类风格的容器用于存储 nsaflash mlametadata 的结构化运行时状态。

### Lines 96-100: method NSAFlashMLAMetadata.slice
```python
    def slice(self, sli):
        return NSAFlashMLAMetadata(
            flashmla_metadata=self.flashmla_metadata,
            num_splits=self.num_splits[sli],
        )
```
**EN:** Implements the slice routine used by this attention module.
**CN:** 实现该注意力模块使用的 slice 例程。

### Lines 102-104: method NSAFlashMLAMetadata.copy_
```python
    def copy_(self, other: "NSAFlashMLAMetadata"):
        self.flashmla_metadata.copy_(other.flashmla_metadata)
        self.num_splits.copy_(other.num_splits)
```
**EN:** Implements the copy routine used by this attention module.
**CN:** 实现该注意力模块使用的 copy 例程。

### Lines 107-158: class NSAMetadata
```python
@dataclass(frozen=True)
class NSAMetadata:
    page_size: int

    # Sequence lengths for the forward batch
    cache_seqlens_int32: torch.Tensor
    # Maximum sequence length for query
    max_seq_len_q: int
    # Maximum sequence length for key
    max_seq_len_k: int
    # Cumulative sequence lengths for query
    cu_seqlens_q: torch.Tensor
    # Cumulative sequence lengths for key
    cu_seqlens_k: torch.Tensor
    # Page table, the index of KV Cache Tables/Blocks
    # this table is always with page_size = 1
    page_table_1: torch.Tensor

    # NOTE(dark): This will property be used in:
    # 1. dense decode/prefill, we use paged flash attention, need real_page_table
    # 2. sparse decode/prefill, indexer need real_page_table to compute the score
    real_page_table: torch.Tensor

    # NSA metadata (nsa prefill are expanded)
    nsa_cache_seqlens_int32: torch.Tensor  # this seqlens is clipped to `topk`
    nsa_cu_seqlens_q: torch.Tensor  # must be arange(0, len(nsa_cu_seqlens_k))
    nsa_cu_seqlens_k: torch.Tensor  # cumsum of `nsa_cache_seqlens_int32`
    nsa_extend_seq_lens_list: List[int]
    nsa_seqlens_expanded: torch.Tensor  # expanded, unclipped `seqlens`
    nsa_max_seqlen_q: Literal[1] = 1  # always 1 for decode, variable for extend

    flashmla_metadata: Optional[NSAFlashMLAMetadata] = None
# ... omitted 8 lines ...
    # The offset of topk indices in ragged kv, prefill only
    # shape: (seq_lens_sum,)
    topk_indices_offset: Optional[torch.Tensor] = None

    # k_start and k_end in kv cache for each token.
    indexer_k_start_end: Optional[Tuple[torch.Tensor, torch.Tensor]] = None
    # seq lens for each batch.
    indexer_seq_lens_cpu: Optional[torch.Tensor] = None
    # seq lens for each batch.
    indexer_seq_lens: Optional[torch.Tensor] = None
    # batch index for each token.
    token_to_batch_idx: Optional[torch.Tensor] = None
```
**EN:** Dataclass-style container that stores structured runtime state for nsametadata.
**CN:** 该数据类风格的容器用于存储 nsametadata 的结构化运行时状态。

### Lines 161-165: class TopkTransformMethod
```python
class TopkTransformMethod(IntEnum):
    # Transform topk indices to indices to the page table (page_size = 1)
    PAGED = auto()
    # Transform topk indices to indices to ragged kv (non-paged)
    RAGGED = auto()
```
**EN:** Enumeration that names execution modes, backend choices, or other symbolic options used by this module.
**CN:** 该枚举为本模块使用的执行模式、后端选择或其他符号化选项命名。

### Lines 168-170: function _compiled_cat
```python
@torch.compile
def _compiled_cat(tensors: list[torch.Tensor], dim: int = -1) -> torch.Tensor:
    return torch.cat(tensors, dim=dim)
```
**EN:** Implements the compiled cat routine used by this attention module.
**CN:** 实现该注意力模块使用的 compiled cat 例程。

### Lines 173-186: function _cat
```python
def _cat(tensors: list[torch.Tensor], dim: int = -1) -> torch.Tensor:
    """
    Concatenate two tensors along the last dimension.
    Use this function to concatenate q_nope and q_rope or k_nope and k_rope.
    """
    assert len(tensors) == 2

    qk_nope, qk_rope = tensors
    assert qk_nope.ndim == 3 and qk_rope.ndim == 3

    torch._dynamo.mark_dynamic(qk_nope, 0)
    torch._dynamo.mark_dynamic(qk_rope, 0)

    return _compiled_cat([qk_nope, qk_rope], dim=dim)
```
**EN:** Implements the cat routine used by this attention module.
**CN:** 实现该注意力模块使用的 cat 例程。

### Lines 189-195: class NSAIndexerMetadata
```python
@dataclass(frozen=True)
class NSAIndexerMetadata(BaseIndexerMetadata):
    attn_metadata: NSAMetadata
    topk_transform_method: TopkTransformMethod
    paged_mqa_schedule_metadata: Optional[torch.Tensor] = None
    force_unfused_topk: bool = False
```
**EN:** Dataclass-style container that stores structured runtime state for nsaindexer metadata.
**CN:** 该数据类风格的容器用于存储 nsaindexer metadata 的结构化运行时状态。

### Lines 196-197: method NSAIndexerMetadata.get_seqlens_int32
```python
    def get_seqlens_int32(self) -> torch.Tensor:
        return self.attn_metadata.cache_seqlens_int32
```
**EN:** Computes and returns get seqlens int32 from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get seqlens int32。

### Lines 199-200: method NSAIndexerMetadata.get_page_table_64
```python
    def get_page_table_64(self) -> torch.Tensor:
        return self.attn_metadata.real_page_table
```
**EN:** Computes and returns get page table 64 from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get page table 64。

### Lines 202-203: method NSAIndexerMetadata.get_page_table_1
```python
    def get_page_table_1(self) -> torch.Tensor:
        return self.attn_metadata.page_table_1
```
**EN:** Computes and returns get page table 1 from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get page table 1。

### Lines 205-206: method NSAIndexerMetadata.get_seqlens_expanded
```python
    def get_seqlens_expanded(self) -> torch.Tensor:
        return self.attn_metadata.nsa_seqlens_expanded
```
**EN:** Computes and returns get seqlens expanded from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get seqlens expanded。

### Lines 208-209: method NSAIndexerMetadata.get_cu_seqlens_k
```python
    def get_cu_seqlens_k(self) -> torch.Tensor:
        return self.attn_metadata.cu_seqlens_k
```
**EN:** Computes and returns get cu seqlens k from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cu seqlens k。

### Lines 211-212: method NSAIndexerMetadata.get_indexer_kvcache_range
```python
    def get_indexer_kvcache_range(self) -> Tuple[torch.Tensor, torch.Tensor]:
        return self.attn_metadata.indexer_k_start_end
```
**EN:** Computes and returns get indexer kvcache range from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer kvcache range。

### Lines 214-215: method NSAIndexerMetadata.get_indexer_seq_len
```python
    def get_indexer_seq_len(self) -> torch.Tensor:
        return self.attn_metadata.indexer_seq_lens
```
**EN:** Computes and returns get indexer seq len from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer seq len。

### Lines 217-218: method NSAIndexerMetadata.get_indexer_seq_len_cpu
```python
    def get_indexer_seq_len_cpu(self) -> torch.Tensor:
        return self.attn_metadata.indexer_seq_lens_cpu
```
**EN:** Computes and returns get indexer seq len cpu from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer seq len cpu。

### Lines 220-221: method NSAIndexerMetadata.get_nsa_extend_len_cpu
```python
    def get_nsa_extend_len_cpu(self) -> List[int]:
        return self.attn_metadata.nsa_extend_seq_lens_list
```
**EN:** Computes and returns get nsa extend len cpu from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get nsa extend len cpu。

### Lines 223-224: method NSAIndexerMetadata.get_token_to_batch_idx
```python
    def get_token_to_batch_idx(self) -> torch.Tensor:
        return self.attn_metadata.token_to_batch_idx
```
**EN:** Computes and returns get token to batch idx from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get token to batch idx。

### Lines 226-290: method NSAIndexerMetadata.topk_transform
```python
    def topk_transform(
        self,
        logits: torch.Tensor,
        topk: int,
        ks: Optional[torch.Tensor] = None,
        cu_seqlens_q: torch.Tensor = None,
        ke_offset: torch.Tensor = None,
        batch_idx_list: List[int] = None,
        topk_indices_offset_override: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        from sgl_kernel import (
            fast_topk_transform_fused,
            fast_topk_transform_ragged_fused,
            fast_topk_v2,
        )

        if topk_indices_offset_override is not None:
            cu_topk_indices_offset = topk_indices_offset_override
            cu_seqlens_q_topk = None
        elif cu_seqlens_q is not None:
            cu_seqlens_q = cu_seqlens_q.to(torch.int32)
            cu_seqlens_q_topk = compute_cu_seqlens(cu_seqlens_q)
            cu_topk_indices_offset = torch.repeat_interleave(
                cu_seqlens_q_topk[:-1],
                cu_seqlens_q,
            )
        else:
            cu_seqlens_q_topk = self.attn_metadata.cu_seqlens_q
            cu_topk_indices_offset = self.attn_metadata.topk_indices_offset
        if ke_offset is not None:
            seq_lens_topk = ke_offset
        else:
# ... omitted 21 lines ...
                    "RAGGED topk_transform requires topk_indices_offset; "
                    "expected extend-without-speculative metadata."
                )
            return fast_topk_transform_ragged_fused(
                score=logits,
                lengths=seq_lens_topk,
                topk_indices_offset=cu_topk_indices_offset,
                topk=topk,
                row_starts=ks,
            )
        else:
            assert False, f"Unsupported {self.topk_transform_method = }"
```
**EN:** Implements the topk transform routine used by this attention module.
**CN:** 实现该注意力模块使用的 topk transform 例程。

### Lines 293-295: module constants
```python
_NSA_IMPL_T: TypeAlias = Literal[
    "flashmla_sparse", "flashmla_kv", "fa3", "tilelang", "trtllm"
]
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 298-300: class NativeSparseAttnBackend
```python
class NativeSparseAttnBackend(
    NativeSparseAttnBackendMTPPrecomputeMixin, AttentionBackend
):
```
**EN:** Concrete attention backend that connects native sparse attn backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 native sparse attn backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 301-393: method NativeSparseAttnBackend.__init__
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        skip_prefill: bool = False,
        speculative_step_id=0,
        topk=0,
        speculative_num_steps=0,
    ):
        super().__init__()
        self.forward_metadata: NSAMetadata
        self.device = model_runner.device
        assert isinstance(model_runner.page_size, int)
        self.real_page_size = model_runner.page_size
        self.num_splits = (
            1 if model_runner.server_args.enable_deterministic_inference else 0
        )
        self.use_nsa = is_deepseek_nsa(model_runner.model_config.hf_config)
        assert self.use_nsa, "NSA backend only supports DeepSeek NSA"
        self.nsa_kv_cache_store_fp8 = (
            model_runner.token_to_kv_pool.nsa_kv_cache_store_fp8
        )
        self.nsa_index_topk = get_nsa_index_topk(model_runner.model_config.hf_config)
        self.max_context_len = model_runner.model_config.context_len
        self.num_q_heads = (
            model_runner.model_config.num_attention_heads // get_attention_tp_size()
        )
        self.kv_cache_dim = model_runner.token_to_kv_pool.kv_cache_dim
        self.qk_nope_head_dim = model_runner.model_config.qk_nope_head_dim
        self.kv_lora_rank = model_runner.model_config.kv_lora_rank
        self.qk_rope_head_dim = model_runner.model_config.qk_rope_head_dim

        assert model_runner.req_to_token_pool is not None
# ... omitted 49 lines ...
        # Allocate global workspace buffer for TRT-LLM kernels (ragged attention on SM100/B200, or trtllm decode)
        if self.device_sm_major >= 10 or self.nsa_decode_impl == "trtllm":
            global global_workspace_buffer
            if global_workspace_buffer is None:
                global_workspace_buffer = torch.empty(
                    envs.SGLANG_FLASHINFER_WORKSPACE_SIZE.get(),
                    dtype=torch.uint8,
                    device=model_runner.device,
                )
            self.workspace_buffer = global_workspace_buffer
        else:
            self.workspace_buffer = None
```
**EN:** Initializes the NativeSparseAttnBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 NativeSparseAttnBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 395-401: method NativeSparseAttnBackend.get_device_int32_arange
```python
    def get_device_int32_arange(self, l: int) -> torch.Tensor:
        if l > len(self._arange_buf):
            next_pow_of_2 = 1 << (l - 1).bit_length()
            self._arange_buf = torch.arange(
                next_pow_of_2, device=self.device, dtype=torch.int32
            )
        return self._arange_buf[:l]
```
**EN:** Computes and returns get device int32 arange from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get device int32 arange。

### Lines 403-411: method NativeSparseAttnBackend._transform_table_1_to_real
```python
    def _transform_table_1_to_real(self, page_table: torch.Tensor) -> torch.Tensor:
        page_size = self.real_page_size
        if page_size == 1:
            return page_table
        max_seqlen_k = page_table.shape[1]
        strided_indices = torch.arange(
            0, max_seqlen_k, page_size, device=page_table.device, dtype=torch.int32
        )
        return page_table[:, strided_indices] // page_size
```
**EN:** Implements the transform table 1 to real routine used by this attention module.
**CN:** 实现该注意力模块使用的 transform table 1 to real 例程。

### Lines 413-702: method NativeSparseAttnBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        """Init the metadata for a forward pass."""
        batch_size = forward_batch.batch_size
        device = forward_batch.seq_lens.device

        if forward_batch.forward_mode.is_target_verify():
            draft_token_num = self.speculative_num_draft_tokens
        else:
            draft_token_num = 0

        cache_seqlens_int32 = (forward_batch.seq_lens + draft_token_num).to(torch.int32)
        cu_seqlens_k = compute_cu_seqlens(cache_seqlens_int32)
        assert forward_batch.seq_lens_cpu is not None
        max_seqlen_k = int(forward_batch.seq_lens_cpu.max().item() + draft_token_num)
        # [b, max_seqlen_k]
        page_table = forward_batch.req_to_token_pool.req_to_token[
            forward_batch.req_pool_indices, :max_seqlen_k
        ]

        page_table_1_flattened = None
        topk_indices_offset = None

        # Centralized dispatch: decide all strategies for this batch
        self.set_nsa_prefill_impl(forward_batch)
        nsa_impl_for_batch = (
            self.nsa_decode_impl
            if (
                forward_batch.forward_mode.is_decode_or_idle()
                or forward_batch.forward_mode.is_target_verify()
                or forward_batch.forward_mode.is_draft_extend(include_v2=True)
            )
            else self.nsa_prefill_impl
# ... omitted 246 lines ...
            nsa_cu_seqlens_k=nsa_cu_seqlens_k,
            nsa_seqlens_expanded=seqlens_expanded,
            nsa_extend_seq_lens_list=extend_seq_lens_cpu,
            real_page_table=self._transform_table_1_to_real(page_table),
            nsa_max_seqlen_q=1,
            topk_indices_offset=topk_indices_offset,
            indexer_k_start_end=indexer_k_start_end,
            indexer_seq_lens_cpu=indexer_seq_lens_cpu,
            indexer_seq_lens=indexer_seq_lens,
            token_to_batch_idx=token_to_batch_idx,
        )
        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 704-778: method NativeSparseAttnBackend._cal_indexer_k_start_end
```python
    def _cal_indexer_k_start_end(
        self,
        forward_batch: ForwardBatch,
        bs_idx: Optional[List[int]] = None,
    ):
        if not forward_batch.forward_mode.is_extend_without_speculative():
            return None, None
        if forward_batch.batch_size == 0 or (bs_idx is not None and len(bs_idx) == 0):
            empty_t = torch.empty(0, dtype=torch.int32, device=self.device)
            return (empty_t, empty_t), empty_t

        # Suppose there are two requests, with extend_seq_len = [3, 2]
        # and seq_lens = [10, 4]
        # The logits matrix looks like this, with * representing the valid logits
        # and - representing the invalid logits:
        #
        #  ********--|----
        #  *********-|----
        #  **********|----
        #  ----------|***-
        #  ----------|****
        #
        # ks = [0, 0, 0, 10, 10]
        # ke = [8, 9, 10, 13, 14]
        ks_list = []
        ke_list = []
        token_to_batch_idx = []

        q_offset = 0
        k_offset = 0

        assert (
# ... omitted 31 lines ...
                q_offset += extend_seq_len
                k_offset += seq_len

        ks = torch.cat(ks_list, dim=0)
        ke = torch.cat(ke_list, dim=0)
        token_to_batch_idx = torch.cat(token_to_batch_idx, dim=0)
        if bs_idx is not None:
            assert can_nsa_prefill_cp_round_robin_split(forward_batch)
            ks = nsa_cp_round_robin_split_data(ks)
            ke = nsa_cp_round_robin_split_data(ke)
            token_to_batch_idx = nsa_cp_round_robin_split_data(token_to_batch_idx)
        return (ks, ke), token_to_batch_idx
```
**EN:** Implements the cal indexer k start end routine used by this attention module.
**CN:** 实现该注意力模块使用的 cal indexer k start end 例程。

### Lines 780-818: method NativeSparseAttnBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        """Initialize CUDA graph state for the attention backend.

        Args:
            max_bs (int): Maximum batch size to support in CUDA graphs

        This creates fixed-size tensors that will be reused during CUDA graph replay
        to avoid memory allocations.
        """
        self.decode_cuda_graph_metadata: Dict = {
            "cache_seqlens": torch.ones(
                max_num_tokens, dtype=torch.int32, device=self.device
            ),
            "cu_seqlens_q": torch.arange(
                0, max_bs + 1, dtype=torch.int32, device=self.device
            ),
            "cu_seqlens_k": torch.zeros(
                max_bs + 1, dtype=torch.int32, device=self.device
            ),
            # fake page_table for sparse_prefill
            # Add extra columns for speculative draft tokens to avoid
            # overflow during target_verify when max_seqlen_k = seq_len + num_draft_tokens
            "page_table": torch.zeros(
                max_num_tokens,
                self.max_context_len + (self.speculative_num_draft_tokens or 0),
                dtype=torch.int32,
                device=self.device,
            ),
            "flashmla_metadata": (
                self._compute_flashmla_metadata(
                    cache_seqlens=torch.ones(
                        max_num_tokens, dtype=torch.int32, device=self.device
                    ),
                    seq_len_q=1,
                )
                if self.nsa_decode_impl == "flashmla_kv"
                else None
            ),
        }
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 820-974: method NativeSparseAttnBackend.init_forward_metadata_capture_cuda_graph
```python
    def init_forward_metadata_capture_cuda_graph(
        self,
        bs: int,
        num_tokens: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: ForwardMode,
        spec_info: Optional[SpecInput],
    ):
        self.set_nsa_prefill_impl(forward_batch=None)

        """Initialize forward metadata for capturing CUDA graph."""
        if forward_mode.is_decode_or_idle():
            # Normal Decode
            # Get sequence information
            cache_seqlens_int32 = seq_lens.to(torch.int32)
            cu_seqlens_k = compute_cu_seqlens(cache_seqlens_int32)

            # Use max context length for seq_len_k
            page_table_1 = self.decode_cuda_graph_metadata["page_table"][:bs, :]
            max_seqlen_q = 1
            max_seqlen_k = page_table_1.shape[1]

            # Precompute page table
            # Precompute cumulative sequence lengths

            # NOTE(dark): this is always arange, since we are decoding
            cu_seqlens_q = self.decode_cuda_graph_metadata["cu_seqlens_q"][: bs + 1]
            nsa_cache_seqlens_int32 = compute_nsa_seqlens(
                cache_seqlens_int32, nsa_index_topk=self.nsa_index_topk
            )
# ... omitted 111 lines ...
            page_table_1=page_table_1,
            flashmla_metadata=flashmla_metadata,
            paged_mqa_schedule_metadata=paged_mqa_schedule_metadata,
            nsa_cache_seqlens_int32=nsa_cache_seqlens_int32,
            nsa_cu_seqlens_q=nsa_cu_seqlens_q,
            nsa_cu_seqlens_k=nsa_cu_seqlens_k,
            nsa_seqlens_expanded=seqlens_expanded,
            real_page_table=real_page_table,
            nsa_extend_seq_lens_list=nsa_extend_seq_lens_list,
        )
        self.decode_cuda_graph_metadata[bs] = metadata
        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 976-1144: method NativeSparseAttnBackend.init_forward_metadata_replay_cuda_graph
```python
    def init_forward_metadata_replay_cuda_graph(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_sum: int,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: ForwardMode,
        spec_info: Optional[SpecInput],
        seq_lens_cpu: Optional[torch.Tensor],
        out_cache_loc: Optional[torch.Tensor] = None,
        actual_forward_mode: Optional[ForwardMode] = None,
    ):
        """Initialize forward metadata for replaying CUDA graph."""
        assert seq_lens_cpu is not None

        self.set_nsa_prefill_impl(forward_batch=None)

        seq_lens = seq_lens[:bs]
        seq_lens_cpu = seq_lens_cpu[:bs]
        req_pool_indices = req_pool_indices[:bs]

        # Normal Decode
        metadata: NSAMetadata = self.decode_cuda_graph_metadata[bs]
        if forward_mode.is_decode_or_idle():
            # Normal Decode
            max_len = int(seq_lens_cpu.max().item())

            cache_seqlens = seq_lens.to(torch.int32)
            metadata.cache_seqlens_int32.copy_(cache_seqlens)
            metadata.cu_seqlens_k[1:].copy_(
                torch.cumsum(cache_seqlens, dim=0, dtype=torch.int32)
# ... omitted 125 lines ...
        if self.nsa_decode_impl == "flashmla_kv":
            flashmla_metadata = metadata.flashmla_metadata.slice(
                slice(0, seqlens_expanded_size + 1)
            )
            flashmla_metadata.copy_(
                self._compute_flashmla_metadata(
                    cache_seqlens=nsa_cache_seqlens,
                    seq_len_q=1,
                )
            )

        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1146-1325: method NativeSparseAttnBackend.init_forward_metadata_replay_cuda_graph_from_precomputed
```python
    def init_forward_metadata_replay_cuda_graph_from_precomputed(
        self,
        bs: int,
        precomputed: PrecomputedMetadata,
        forward_mode: ForwardMode,
    ):
        """Fast path: copy precomputed metadata to this backend's metadata.

        This function only performs copy operations, no computation.

        Args:
            bs: Batch size
            precomputed: Precomputed metadata to copy from
            forward_mode: Forward mode
        """
        self.set_nsa_prefill_impl(forward_batch=None)

        metadata = self.decode_cuda_graph_metadata[bs]

        # Track whether fused kernel succeeded
        fused_kernel_succeeded = False

        # Use fused CUDA kernel for all copy operations
        if _USE_FUSED_METADATA_COPY:
            try:
                from sglang.jit_kernel.fused_metadata_copy import (
                    fused_metadata_copy_cuda,
                )

                # Map forward_mode to integer enum
                if forward_mode.is_decode_or_idle():
                    mode_int = 0  # DECODE
# ... omitted 136 lines ...
                    seqlens_32_2d, 64, deep_gemm.get_num_sms()
                )
                if metadata.paged_mqa_schedule_metadata is None:
                    object.__setattr__(
                        metadata, "paged_mqa_schedule_metadata", new_schedule
                    )
                else:
                    metadata.paged_mqa_schedule_metadata.copy_(new_schedule)
            except (ImportError, ModuleNotFoundError):
                pass

        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph from precomputed, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph from precomputed 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1327-1535: method NativeSparseAttnBackend.forward_extend
```python
    def forward_extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        # For multi-head latent attention
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        topk_indices: Optional[torch.Tensor] = None,
        cos_sin_cache: Optional[torch.Tensor] = None,
        is_neox: Optional[bool] = False,
        llama_4_scaling: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        causal = not layer.is_cross_attention
        metadata = self.forward_metadata
        assert causal, "NSA is causal only"

        nsa_impl = (
            self.nsa_decode_impl
            if (
                forward_batch.forward_mode.is_target_verify()
                or forward_batch.forward_mode.is_draft_extend(include_v2=True)
            )
            else self.nsa_prefill_impl
        )

        if nsa_impl == "trtllm" and not self.use_mha:
            return self._forward_trtllm(
# ... omitted 165 lines ...
            if q_rope is not None:
                q_all = torch.cat([q_nope, q_rope], dim=-1)
            return self._forward_aiter_extend(
                q_all=q_all,
                kv_cache=kv_cache,
                page_table_1=page_table_1,
                layer=layer,
            )
        else:
            raise ValueError(
                f"Unsupported {nsa_impl = } for forward_extend. Consider using an other attention backend."
            )
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1537-1693: method NativeSparseAttnBackend.forward_decode
```python
    def forward_decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        # For multi-head latent attention
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        topk_indices: Optional[torch.Tensor] = None,
        cos_sin_cache: Optional[torch.Tensor] = None,
        is_neox: Optional[bool] = False,
        llama_4_scaling: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        causal = not layer.is_cross_attention
        metadata = self.forward_metadata
        assert causal, "NSA is causal only"

        if self.nsa_decode_impl == "trtllm":
            return self._forward_trtllm(
                q,
                k,
                v,
                layer,
                forward_batch,
                metadata.cache_seqlens_int32,
                save_kv_cache,
                q_rope,
                k_rope,
# ... omitted 113 lines ...
                q_all = torch.cat([q_nope, q_rope], dim=-1)
            return self._forward_aiter(
                q_all=q_all,
                kv_cache=kv_cache,
                page_table_1=page_table_1,
                layer=layer,
                metadata=metadata,
                bs=forward_batch.batch_size,
            )

        else:
            assert False, f"Unsupported {self.nsa_decode_impl = }"
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1695-1731: method NativeSparseAttnBackend._forward_fa3
```python
    def _forward_fa3(
        self,
        q_rope: torch.Tensor,
        kv_cache: torch.Tensor,
        v_head_dim: int,
        q_nope: torch.Tensor,
        page_table: torch.Tensor,
        cache_seqlens: torch.Tensor,
        cu_seqlens_q: torch.Tensor,
        cu_seqlens_k: torch.Tensor,
        max_seqlen_q: int,
        sm_scale: float,
        logit_cap: float,
        page_size: int,
    ) -> torch.Tensor:
        k_rope_cache = kv_cache[:, :, v_head_dim:]
        c_kv_cache = kv_cache[:, :, :v_head_dim]
        qk_rope_dim = k_rope_cache.shape[-1]
        k_rope_cache = k_rope_cache.view(-1, page_size, 1, qk_rope_dim)
        c_kv_cache = c_kv_cache.view(-1, page_size, 1, v_head_dim)
        o = flash_attn_with_kvcache(
            q=q_rope,
            k_cache=k_rope_cache,
            v_cache=c_kv_cache,
            qv=q_nope,
            page_table=page_table,
            cache_seqlens=cache_seqlens,
            cu_seqlens_q=cu_seqlens_q,
            cu_seqlens_k_new=cu_seqlens_k,
            max_seqlen_q=max_seqlen_q,
            softmax_scale=sm_scale,
            causal=True,
            softcap=logit_cap,
            return_softmax_lse=False,
            num_splits=self.num_splits,
        )
        return o  # type: ignore
```
**EN:** Runs the forward-path logic for forward fa3, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward fa3 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1733-1780: method NativeSparseAttnBackend._forward_flashmla_sparse
```python
    def _forward_flashmla_sparse(
        self,
        q_all: torch.Tensor,
        kv_cache: torch.Tensor,
        v_head_dim: int,
        page_table_1: torch.Tensor,
        sm_scale: float,
    ) -> torch.Tensor:
        from sgl_kernel.flash_mla import flash_mla_sparse_fwd

        # FlashMLA sparse kernel requires num_heads to be a multiple of 64 (Hopper) or 128 (Blackwell)
        # When using TP, num_heads might be smaller (e.g., 256//8=32)
        num_tokens, num_heads, head_dim = q_all.shape

        # Determine required padding based on GPU architecture (use cached value)
        required_padding = 128 if self.device_sm_major >= 10 else 64

        need_padding = num_heads % required_padding != 0

        if need_padding:
            assert required_padding % num_heads == 0, (
                f"num_heads {num_heads} cannot be padded to {required_padding}. "
                f"TP size may be too large for this model."
            )

            # Pad q to required size
            q_padded = q_all.new_zeros((num_tokens, required_padding, head_dim))
            q_padded[:, :num_heads, :] = q_all
            q_input = q_padded
        else:
            q_input = q_all

        # indices shape must be (s_q, h_kv=1, topk), keep h_kv=1 unchanged
        indices_input = page_table_1.unsqueeze(1)

        o, _, _ = flash_mla_sparse_fwd(
            q=q_input,
            kv=kv_cache,
            indices=indices_input,
            sm_scale=sm_scale,
            d_v=v_head_dim,
        )

        # Trim output back to original num_heads if we padded
        if need_padding:
            o = o[:, :num_heads, :]

        return o
```
**EN:** Runs the forward-path logic for forward flashmla sparse, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward flashmla sparse 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1782-1841: method NativeSparseAttnBackend._forward_flashmla_kv
```python
    def _forward_flashmla_kv(
        self,
        q_all: torch.Tensor,
        kv_cache: torch.Tensor,
        v_head_dim: int,
        sm_scale: float,
        layer,
        metadata: NSAMetadata,
        page_table_1,
    ) -> torch.Tensor:
        from sgl_kernel.flash_mla import flash_mla_with_kvcache

        cache_seqlens = metadata.nsa_cache_seqlens_int32
        assert metadata.flashmla_metadata is not None

        # TODO the 2nd dim is seq_len_q, need to be >1 when MTP
        q_all = q_all.view(-1, 1, layer.tp_q_head_num, layer.head_dim)
        num_q_heads = q_all.shape[2]
        target_q_heads = self.flashmla_kv_num_q_heads
        if target_q_heads != num_q_heads:
            # Pad q heads to match FlashMLA decode supported head-count variants.
            q_input = q_all.new_zeros(
                q_all.shape[0], q_all.shape[1], target_q_heads, q_all.shape[3]
            )
            q_input[:, :, :num_q_heads, :] = q_all
        else:
            q_input = q_all

        kv_cache = kv_cache.view(-1, self.real_page_size, 1, self.kv_cache_dim)
        assert self.real_page_size == 64, "only page size 64 is supported"

        if not self.nsa_kv_cache_store_fp8:
# ... omitted 16 lines ...
            indices=indices,
            # doc says it is not used, but if pass in None then error
            block_table=torch.empty(
                (q_all.shape[0], 0), dtype=torch.int32, device=q_all.device
            ),
            is_fp8_kvcache=True,
        )

        if target_q_heads != num_q_heads:
            o = o[:, :, :num_q_heads, :]

        return o
```
**EN:** Runs the forward-path logic for forward flashmla kv, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward flashmla kv 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1843-1906: method NativeSparseAttnBackend._forward_standard_mha
```python
    def _forward_standard_mha(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        metadata: NSAMetadata,
    ) -> torch.Tensor:
        """Standard MHA using FlashAttention varlen for MHA_ONE_SHOT mode."""
        q = q.view(-1, layer.tp_q_head_num, layer.head_dim)
        k = k.view(-1, layer.tp_k_head_num, layer.head_dim)
        v = v.view(-1, layer.tp_v_head_num, layer.v_head_dim)

        # MHA_ONE_SHOT: k/v include all tokens (prefix + current)
        cu_seqlens_q = metadata.cu_seqlens_q
        cu_seqlens_k = metadata.cu_seqlens_k
        max_seqlen_k = metadata.max_seq_len_k
        causal = True

        # Verify batch sizes match (length of cu_seqlens should be batch_size + 1)
        assert len(cu_seqlens_q) == len(cu_seqlens_k), (
            f"batch_size mismatch: cu_seqlens_q has {len(cu_seqlens_q)-1} requests, "
            f"cu_seqlens_k has {len(cu_seqlens_k)-1} requests"
        )

        # Use TRTLLm ragged attention for SM100 (Blackwell/B200) to avoid FA4 accuracy issues
        if self.device_sm_major >= 10:
            import flashinfer

            seq_lens = metadata.cache_seqlens_int32
            return flashinfer.prefill.trtllm_ragged_attention_deepseek(
# ... omitted 20 lines ...
        # Use FA3 for SM90 (Hopper/H200)
        return flash_attn_varlen_func(
            q=q,
            k=k,
            v=v,
            cu_seqlens_q=cu_seqlens_q,
            cu_seqlens_k=cu_seqlens_k,
            max_seqlen_q=metadata.max_seq_len_q,
            max_seqlen_k=max_seqlen_k,
            softmax_scale=layer.scaling,
            causal=causal,
        )
```
**EN:** Runs the forward-path logic for forward standard mha, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward standard mha 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1908-1924: method NativeSparseAttnBackend._forward_tilelang
```python
    def _forward_tilelang(
        self,
        q_all: torch.Tensor,
        kv_cache: torch.Tensor,
        v_head_dim: int,
        page_table_1: torch.Tensor,
        sm_scale: float,
    ) -> torch.Tensor:
        from sglang.srt.layers.attention.nsa.tilelang_kernel import tilelang_sparse_fwd

        return tilelang_sparse_fwd(
            q=q_all,
            kv=kv_cache,
            indices=page_table_1.unsqueeze(1),
            sm_scale=sm_scale,
            d_v=v_head_dim,
        )
```
**EN:** Runs the forward-path logic for forward tilelang, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward tilelang 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1926-1982: method NativeSparseAttnBackend._forward_aiter
```python
    def _forward_aiter(
        self,
        q_all: torch.Tensor,
        kv_cache: torch.Tensor,
        page_table_1: torch.Tensor,
        layer: RadixAttention,
        metadata: NSAMetadata,
        bs: int,
    ) -> torch.Tensor:
        q = q_all.reshape(-1, layer.tp_q_head_num * layer.head_dim)

        if layer.head_dim != layer.v_head_dim:
            o = q.new_empty((q.shape[0], layer.tp_q_head_num * layer.v_head_dim))
        else:
            o = torch.empty_like(q)

        if self.need_pad_heads:
            q_kernel = q.view(
                -1, layer.tp_q_head_num, layer.head_dim
            ).repeat_interleave(self.head_repeat_factor, dim=1)
            o_kernel = q.new_empty(
                (
                    q.shape[0],
                    layer.tp_q_head_num * self.head_repeat_factor,
                    layer.v_head_dim,
                )
            )
        else:
            q_kernel = q.view(-1, layer.tp_q_head_num, layer.head_dim)
            o_kernel = o.view(-1, layer.tp_q_head_num, layer.v_head_dim)

        kv_indptr = self.kv_indptr
# ... omitted 13 lines ...
            kv_indptr,
            kv_indices,
            metadata.cu_seqlens_q,
            metadata.max_seq_len_q,
            sm_scale=layer.scaling,
            logit_cap=layer.logit_cap,
        )

        if self.need_pad_heads:
            o = o_kernel[:, :: self.head_repeat_factor, :]

        return o
```
**EN:** Runs the forward-path logic for forward aiter, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward aiter 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1984-2050: method NativeSparseAttnBackend._forward_aiter_extend
```python
    def _forward_aiter_extend(
        self,
        q_all: torch.Tensor,
        kv_cache: torch.Tensor,
        page_table_1: torch.Tensor,
        layer: RadixAttention,
    ) -> torch.Tensor:
        num_tokens = q_all.shape[0]
        q = q_all.reshape(-1, layer.tp_q_head_num * layer.head_dim)

        if layer.head_dim != layer.v_head_dim:
            o = q.new_empty((num_tokens, layer.tp_q_head_num * layer.v_head_dim))
        else:
            o = torch.empty_like(q)

        if self.need_pad_heads:
            q_kernel = q.view(
                -1, layer.tp_q_head_num, layer.head_dim
            ).repeat_interleave(self.head_repeat_factor, dim=1)
            o_kernel = q.new_empty(
                (
                    num_tokens,
                    layer.tp_q_head_num * self.head_repeat_factor,
                    layer.v_head_dim,
                )
            )
        else:
            q_kernel = q.view(-1, layer.tp_q_head_num, layer.head_dim)
            o_kernel = o.view(-1, layer.tp_q_head_num, layer.v_head_dim)

        non_minus1_mask = page_table_1 != -1
        non_minus1_counts = non_minus1_mask.sum(dim=1)
# ... omitted 23 lines ...
            kv_indptr,
            kv_indices,
            cu_seqlens_q,
            1,  # max_seq_len_q = 1 for per-token attention
            sm_scale=layer.scaling,
            logit_cap=layer.logit_cap,
        )

        if self.need_pad_heads:
            o = o_kernel[:, :: self.head_repeat_factor, :]

        return o
```
**EN:** Runs the forward-path logic for forward aiter extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward aiter extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2052-2176: method NativeSparseAttnBackend._forward_trtllm
```python
    def _forward_trtllm(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        seq_lens: torch.Tensor,
        save_kv_cache=True,
        # For multi-head latent attention
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        topk_indices: Optional[torch.Tensor] = None,
        cos_sin_cache: Optional[torch.Tensor] = None,
        is_neox: Optional[bool] = False,
        llama_4_scaling: Optional[torch.Tensor] = None,
        is_prefill: bool = False,
    ) -> torch.Tensor:
        """Forward using TRT-LLM sparse MLA kernel."""
        import flashinfer.decode

        metadata = self.forward_metadata

        merge_query = q_rope is not None
        if self.kv_cache_dtype == torch.float8_e4m3fn:
            # For FP8 path, we quantize the query and rope parts and merge them into a single tensor
            # Note: rope application in deepseek_v2.py:forward_absorb_prepare is skipped for FP8 decode path of this trtllm_mla backend
            assert q_rope is not None, "For FP8 path q_rope should not be None."
            assert k_rope is not None, "For FP8 path k_rope should not be None."
            assert (
                cos_sin_cache is not None
            ), "For FP8 path cos_sin_cache should not be None."
# ... omitted 81 lines ...
            kv_lora_rank=self.kv_lora_rank,
            qk_rope_head_dim=self.qk_rope_head_dim,
            block_tables=block_tables,
            seq_lens=seq_lens,
            max_seq_len=metadata.max_seq_len_k,
            sparse_mla_top_k=self.nsa_index_topk,
            bmm1_scale=bmm1_scale,
            backend="trtllm-gen",
            skip_softmax_threshold_scale_factor=envs.SGLANG_SKIP_SOFTMAX_DECODE_THRESHOLD_SCALE_FACTOR.get(),
        )
        # Output: [batch, q_len=1, heads, v_dim] -> [batch, heads, v_dim]
        return out.squeeze(1)
```
**EN:** Runs the forward-path logic for forward trtllm, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward trtllm 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2178-2197: method NativeSparseAttnBackend._pad_topk_indices
```python
    def _pad_topk_indices(
        self, topk_indices: torch.Tensor, num_tokens: int
    ) -> torch.Tensor:
        current_tokens = topk_indices.shape[0]
        if current_tokens == num_tokens:
            return topk_indices

        assert current_tokens <= num_tokens, (
            f"topk_indices rows ({current_tokens}) > num_tokens ({num_tokens}); "
            "this indicates a mismatch between indexer output and q layout."
        )

        pad_size = num_tokens - current_tokens
        padding = torch.full(
            (pad_size, topk_indices.shape[1]),
            -1,
            dtype=topk_indices.dtype,
            device=topk_indices.device,
        )
        return torch.cat([topk_indices, padding], dim=0)
```
**EN:** Implements the pad topk indices routine used by this attention module.
**CN:** 实现该注意力模块使用的 pad topk indices 例程。

### Lines 2199-2201: method NativeSparseAttnBackend.get_cuda_graph_seq_len_fill_value
```python
    def get_cuda_graph_seq_len_fill_value(self):
        """Get the fill value for sequence length in CUDA graph."""
        return 1
```
**EN:** Computes and returns get cuda graph seq len fill value from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cuda graph seq len fill value。

### Lines 2203-2251: method NativeSparseAttnBackend.set_nsa_prefill_impl
```python
    def set_nsa_prefill_impl(self, forward_batch: Optional[ForwardBatch] = None):
        """
        Decide all attention prefill dispatch strategies for this batch.
        """
        from sglang.srt.utils import get_device_sm, is_blackwell

        # Decide MHA vs MLA
        if forward_batch and forward_batch.forward_mode.is_extend_without_speculative():
            # Check if sequence meets criteria for MHA_ONE_SHOT
            assert forward_batch.seq_lens_cpu is not None
            max_kv_len = forward_batch.seq_lens_cpu.max().item()
            sum_seq_lens = sum(forward_batch.seq_lens_cpu)
            device_sm = get_device_sm()

            # Requirements: H200/B200, short sequences, supported dtype, fits in chunk
            self.use_mha = (
                (
                    device_sm == 90 or (device_sm >= 100 and device_sm < 110)
                )  # SM90/SM100 only
                and max_kv_len
                <= envs.SGLANG_NSA_PREFILL_DENSE_ATTN_KV_LEN_THRESHOLD.get()  # Short enough for MHA
                and forward_batch.token_to_kv_pool.dtype
                in [torch.bfloat16, torch.float8_e4m3fn]
                and sum_seq_lens
                <= forward_batch.get_max_chunk_capacity()  # Fits in chunk
                and (not is_nsa_enable_prefill_cp())  # CP not enabled
                and (forward_batch.hisparse_coordinator is None)
            )
        else:
            self.use_mha = False  # Decode/verify always use MLA

        # Set MLA implementation only if not using MHA
# ... omitted 5 lines ...
                    and forward_batch.forward_mode == ForwardMode.EXTEND
                ):
                    total_kv_tokens = forward_batch.seq_lens_sum
                    total_q_tokens = forward_batch.extend_num_tokens
                    # Heuristic based on benchmarking flashmla_kv vs flashmla_sparse + dequantize_k_cache_paged
                    if total_kv_tokens < total_q_tokens * 512:
                        self.nsa_prefill_impl = "flashmla_sparse"
                        return
                self.nsa_prefill_impl = "flashmla_kv"
            else:
                # bf16 kv cache
                self.nsa_prefill_impl = "flashmla_sparse"
```
**EN:** Updates set nsa prefill impl on the active object so later attention steps observe the latest runtime state.
**CN:** 更新活动对象上的 set nsa prefill impl，以便后续注意力步骤读取最新运行时状态。

### Lines 2253-2269: method NativeSparseAttnBackend.get_topk_transform_method
```python
    def get_topk_transform_method(
        self, forward_mode: Optional[ForwardMode] = None
    ) -> TopkTransformMethod:
        """
        SGLANG_NSA_FUSE_TOPK controls whether to fuse the topk transform into the topk kernel.
        This method is used to select the topk transform method which can be fused or unfused.
        """
        if (
            # disable for MTP
            self.nsa_kv_cache_store_fp8
            and self.nsa_prefill_impl == "flashmla_sparse"
            and forward_mode == ForwardMode.EXTEND
        ):
            topk_transform_method = TopkTransformMethod.RAGGED
        else:
            topk_transform_method = TopkTransformMethod.PAGED
        return topk_transform_method
```
**EN:** Computes and returns get topk transform method from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get topk transform method。

### Lines 2271-2285: method NativeSparseAttnBackend.get_indexer_metadata
```python
    def get_indexer_metadata(
        self, layer_id: int, forward_batch: ForwardBatch
    ) -> NSAIndexerMetadata:
        force_unfused = (
            forward_batch.hisparse_coordinator is not None
            and forward_batch.forward_mode.is_decode_or_idle()
        )
        return NSAIndexerMetadata(
            attn_metadata=self.forward_metadata,
            topk_transform_method=self.get_topk_transform_method(
                forward_batch.forward_mode
            ),
            paged_mqa_schedule_metadata=self.forward_metadata.paged_mqa_schedule_metadata,
            force_unfused_topk=force_unfused,
        )
```
**EN:** Computes and returns get indexer metadata from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer metadata。

### Lines 2287-2306: method NativeSparseAttnBackend._compute_flashmla_metadata
```python
    def _compute_flashmla_metadata(self, cache_seqlens: torch.Tensor, seq_len_q: int):
        from sgl_kernel.flash_mla import get_mla_metadata

        num_heads_q = self.flashmla_kv_num_q_heads

        flashmla_metadata, num_splits = get_mla_metadata(
            cache_seqlens=cache_seqlens,
            # TODO doc says `num_q_tokens_per_q_seq * num_heads_q // num_heads_k`
            #      but the name looks like need seq_len_q?
            num_q_tokens_per_head_k=seq_len_q * num_heads_q // 1,
            num_heads_k=1,
            num_heads_q=num_heads_q,
            is_fp8_kvcache=True,
            topk=self.nsa_index_topk,
        )

        return NSAFlashMLAMetadata(
            flashmla_metadata=flashmla_metadata,
            num_splits=num_splits,
        )
```
**EN:** Implements the compute flashmla metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 compute flashmla metadata 例程。

### Lines 2309-2310: class NativeSparseAttnMultiStepBackend
```python
class NativeSparseAttnMultiStepBackend:
```
**EN:** Concrete attention backend that connects native sparse attn multi step backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 native sparse attn multi step backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 2311-2326: method NativeSparseAttnMultiStepBackend.__init__
```python
    def __init__(
        self, model_runner: ModelRunner, topk: int, speculative_num_steps: int
    ):
        self.model_runner = model_runner
        self.topk = topk
        self.speculative_num_steps = speculative_num_steps
        self.attn_backends = []
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends.append(
                NativeSparseAttnBackend(
                    model_runner,
                    speculative_step_id=i,
                    topk=self.topk,
                    speculative_num_steps=self.speculative_num_steps,
                )
            )
```
**EN:** Initializes the NativeSparseAttnMultiStepBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 NativeSparseAttnMultiStepBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 2328-2330: method NativeSparseAttnMultiStepBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_forward_metadata(forward_batch)
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2332-2334: method NativeSparseAttnMultiStepBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_cuda_graph_state(max_bs, max_num_tokens)
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 2336-2346: method NativeSparseAttnMultiStepBackend.init_forward_metadata_capture_cuda_graph
```python
    def init_forward_metadata_capture_cuda_graph(self, forward_batch: ForwardBatch):
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_forward_metadata_capture_cuda_graph(
                forward_batch.batch_size,
                forward_batch.batch_size * self.topk,
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                encoder_lens=None,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
            )
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2348-2520: method NativeSparseAttnMultiStepBackend.init_forward_metadata_replay_cuda_graph
```python
    def init_forward_metadata_replay_cuda_graph(
        self, forward_batch: ForwardBatch, bs: int
    ):
        if envs.SGLANG_NSA_ENABLE_MTP_PRECOMPUTE_METADATA.get():
            # Precompute metadata once (shared across all backends)
            precomputed = self.attn_backends[0]._precompute_replay_metadata(
                bs=bs,
                req_pool_indices=forward_batch.req_pool_indices,
                seq_lens=forward_batch.seq_lens,
                seq_lens_cpu=forward_batch.seq_lens_cpu,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
            )

            # Use multi-backend fused copy when we have 3 or more backends
            # This is 3x faster than calling the single-backend copy 3 times
            if self.speculative_num_steps > 3:
                try:
                    from sglang.jit_kernel.fused_metadata_copy import (
                        fused_metadata_copy_multi_cuda,
                    )

                    metadata0 = self.attn_backends[0].decode_cuda_graph_metadata[bs]
                    metadata1 = self.attn_backends[1].decode_cuda_graph_metadata[bs]
                    metadata2 = self.attn_backends[2].decode_cuda_graph_metadata[bs]

                    # Set nsa_prefill_impl for first 3 backends (required by the method)
                    for i in range(3):
                        self.attn_backends[i].set_nsa_prefill_impl(forward_batch=None)

                    # Prepare FlashMLA tensors if needed
                    flashmla_num_splits_src = None
# ... omitted 129 lines ...
            for i in range(self.speculative_num_steps - 1):
                self.attn_backends[i].init_forward_metadata_replay_cuda_graph(
                    bs=bs,
                    req_pool_indices=forward_batch.req_pool_indices,
                    seq_lens=forward_batch.seq_lens,
                    seq_lens_sum=forward_batch.seq_lens_sum,
                    encoder_lens=None,
                    forward_mode=ForwardMode.DECODE,
                    spec_info=forward_batch.spec_info,
                    seq_lens_cpu=forward_batch.seq_lens_cpu,
                    out_cache_loc=None,
                )
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** FlashInfer execution path / **CN:** FlashInfer 执行路径
- **EN:** Flash-style fused attention kernels / **CN:** Flash 风格融合注意力内核
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划

## Dependencies / 依赖关系
- `__future__.annotations`
- `dataclasses.dataclass`
- `enum.IntEnum`
- `enum.auto`
- `typing.TYPE_CHECKING`
- `typing.Dict`
- `typing.List`
- `typing.Literal`
- `typing.Optional`
- `typing.Tuple`
- `typing.TypeAlias`
- `torch`
- `sglang.srt.configs.model_config.get_nsa_index_topk`
- `sglang.srt.configs.model_config.is_deepseek_nsa`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.attention.base_attn_backend.AttentionBackend`
- `sglang.srt.layers.attention.nsa.dequant_k_cache.dequantize_k_cache_paged`
- `sglang.srt.layers.attention.nsa.nsa_backend_mtp_precompute.NativeSparseAttnBackendMTPPrecomputeMixin`
- `sglang.srt.layers.attention.nsa.nsa_backend_mtp_precompute.PrecomputedMetadata`
- `sglang.srt.layers.attention.nsa.nsa_backend_mtp_precompute.compute_cu_seqlens`
- `sglang.srt.layers.attention.nsa.nsa_indexer.BaseIndexerMetadata`
- `sglang.srt.layers.attention.nsa.quant_k_cache.quantize_k_cache`
- `sglang.srt.layers.attention.nsa.transform_index.transform_index_page_table_decode`
- `sglang.srt.layers.attention.nsa.transform_index.transform_index_page_table_prefill`
- `sglang.srt.layers.attention.nsa.utils.can_nsa_prefill_cp_round_robin_split`
- `sglang.srt.layers.attention.nsa.utils.compute_nsa_seqlens`
- `sglang.srt.layers.attention.nsa.utils.is_nsa_enable_prefill_cp`
- `sglang.srt.layers.attention.nsa.utils.nsa_cp_round_robin_split_data`
- `sglang.srt.layers.attention.nsa.utils.nsa_cp_round_robin_split_q_seqs`
- `sglang.srt.layers.attention.nsa.utils.pad_nsa_cache_seqlens`
- `sglang.srt.layers.attention.utils.concat_mla_absorb_q_general`
- `sglang.srt.layers.attention.utils.mla_quantize_and_rope_for_fp8`
- `sglang.srt.layers.attention.utils.seqlens_expand_triton`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_hip`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.model_runner.ModelRunner`
- `sglang.srt.speculative.spec_info.SpecInput`
- `sglang.srt.layers.attention.nsa.triton_kernel.get_valid_kv_indices`
- `sglang.jit_kernel.flash_attention.flash_attn_varlen_func`
- `sglang.jit_kernel.flash_attention.flash_attn_with_kvcache`
- `aiter.flash_attn_varlen_func`
- `aiter.mha_batch_prefill_func`
- `aiter.paged_attention_ragged`
- `aiter.mla.mla_decode_fwd`
- `aiter.mla.mla_prefill_fwd`
- `sgl_kernel.fast_topk_transform_fused`
- `sgl_kernel.fast_topk_transform_ragged_fused`
- `sgl_kernel.fast_topk_v2`
- `sgl_kernel.flash_mla.flash_mla_sparse_fwd`
- `sgl_kernel.flash_mla.flash_mla_with_kvcache`
- `sglang.srt.layers.attention.nsa.tilelang_kernel.tilelang_sparse_fwd`
- `flashinfer.decode`
- `sglang.srt.utils.get_device_sm`
- `sglang.srt.utils.is_blackwell`
- `sgl_kernel.flash_mla.get_mla_metadata`
- `flashinfer`
- `deep_gemm`
- `sglang.jit_kernel.fused_metadata_copy.fused_metadata_copy_cuda`
- `sglang.jit_kernel.fused_metadata_copy.fused_metadata_copy_multi_cuda`
