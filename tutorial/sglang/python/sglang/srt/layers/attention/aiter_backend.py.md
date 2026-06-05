# aiter_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/aiter_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the aiter backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 aiter backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-1: imports
```python
from __future__ import annotations
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 3-5: docstring
```python
"""
end to end attention solution with aiter kernels
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 7-29: imports
```python
import logging
from dataclasses import dataclass
from enum import Enum, auto
from typing import TYPE_CHECKING, Optional

import torch
import triton

from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.layers.attention.triton_ops.aiter_unified_attention import (
    scatter_ragged_to_page_table_kernel,
    scatter_req_to_token_to_page_table_kernel,
)
from sglang.srt.layers.attention.utils import (
    create_flashinfer_kv_indices_triton,
    create_flashmla_kv_indices_triton,
)
from sglang.srt.layers.dp_attention import (
    get_attention_tp_size,
    is_dp_attention_enabled,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.utils import is_gfx95_supported
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 31-34: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner
    from sglang.srt.speculative.spec_info import SpecInput
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 36-53: guarded import or fallback
```python
try:
    from aiter import (
        flash_attn_varlen_func,
        get_mla_metadata_info_v1,
        get_mla_metadata_v1,
        get_ps_metadata_info_v1,
        get_ps_metadata_v1,
        mha_batch_prefill_func,
        mla_prefill_ps_asm_fwd,
        mla_reduce_v1,
        paged_attention_ragged,
    )
    from aiter.mla import mla_decode_fwd, mla_prefill_fwd
    from aiter.ops.triton.attention.unified_attention import unified_attention
except ImportError:
    print(
        "aiter is AMD specific kernel library. Please make sure aiter is installed on your AMD device."
    )
```
**EN:** Attempts an optional import or guarded operation and provides a fallback path when the fast path is unavailable.
**CN:** 尝试可选导入或受保护操作，并在快速路径不可用时提供回退逻辑。

### Lines 55-62: imports
```python
from sglang.srt.configs.model_config import AttentionArch
from sglang.srt.layers.attention.utils import (
    launch_reshape_and_cache_flash,
    pad_sequence_with_mask,
)
from sglang.srt.layers.quantization.fp8_kernel import fp8_dtype
from sglang.srt.mem_cache.swa_memory_pool import SWAKVPool
from sglang.srt.utils import get_bool_env_var
```
**EN:** Imports neighboring SGLang modules so this file can reuse shared attention abstractions and utilities.
**CN:** 导入相邻的 SGLang 模块，以复用共享的注意力抽象和工具函数。

### Lines 64-80: module constants
```python
logger = logging.getLogger(__name__)

# Use aiter mla persist design for fp8-kv cache
_use_mla_ps_kernel = get_bool_env_var("SGLANG_AITER_MLA_PERSIST", "True")

# Use fp8 prefill only on gfx95
_use_fp8_prefill_attn = (
    get_bool_env_var("SGLANG_AITER_FP8_PREFILL_ATTN", "True") and is_gfx95_supported()
)

# Persist
# fast_mode=True if _use_mla_ps_kernel else False
# intra_batch_mode=False if _use_mla_ps_kernel else True

# fake non-ps, intra_batch_mode needs to be True for non-ps-mode
fast_mode = False
intra_batch_mode = True if _use_mla_ps_kernel else False
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 83-85: class WrapperDispatch
```python
class WrapperDispatch(Enum):
    SLIDING_WINDOW = auto()
    CROSS_ATTENTION = auto()
```
**EN:** Enumeration that names execution modes, backend choices, or other symbolic options used by this module.
**CN:** 该枚举为本模块使用的执行模式、后端选择或其他符号化选项命名。

### Lines 88-108: class ForwardMetadata
```python
@dataclass
class ForwardMetadata:
    kv_indptr: torch.Tensor
    kv_indices: torch.Tensor
    qo_indptr: torch.Tensor
    kv_last_page_len: torch.Tensor
    max_q_len: int
    max_kv_len: Optional[int]
    work_metadata: Optional[torch.Tensor] = None
    work_info_set: Optional[torch.Tensor] = None
    work_indptr: Optional[torch.Tensor] = None
    reduce_indptr: Optional[torch.Tensor] = None
    reduce_final_map: Optional[torch.Tensor] = None
    reduce_partial_map: Optional[torch.Tensor] = None
    num_kv_splits: Optional[int] = None
    run_graph: Optional[bool] = True
    custom_mask: Optional[torch.Tensor] = None
    mask_indptr: Optional[torch.Tensor] = None
    max_extend_len: Optional[int] = None
    fp8_prefill_kv_indices: Optional[torch.Tensor] = None
    swa_page_table: Optional[torch.Tensor] = None
```
**EN:** Dataclass-style container that stores structured runtime state for forward metadata.
**CN:** 该数据类风格的容器用于存储 forward metadata 的结构化运行时状态。

### Lines 111-114: module constants
```python
global_workspace_buffer = None


_AITER_PARTITION_SIZE_ROCM = 256
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 117-117: class AiterAttnBackend
```python
class AiterAttnBackend(AttentionBackend):
```
**EN:** Concrete attention backend that connects aiter attn backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 aiter attn backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 118-301: method AiterAttnBackend.__init__
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        skip_prefill: bool = False,
        kv_indptr_buf: Optional[torch.Tensor] = None,
        topk: int = 1,
    ):
        super().__init__()
        # Lazy import to avoid the initialization of cuda context
        from sglang.srt.layers.attention.triton_ops.extend_attention import (
            extend_attention_fwd,
        )

        self.input_dtype = model_runner.model_config.dtype

        self.page_size = model_runner.server_args.page_size

        self.extend_attention_fwd = torch.compiler.disable(extend_attention_fwd)

        self.device = model_runner.device
        self.is_multimodal = model_runner.model_config.is_multimodal
        self.num_draft_tokens = model_runner.server_args.speculative_num_draft_tokens
        self.speculative_num_steps = model_runner.server_args.speculative_num_steps
        self.topk = topk
        self.num_head = (
            model_runner.model_config.num_attention_heads // get_attention_tp_size()
        )
        self.head_dim = model_runner.model_config.head_dim
        self.num_kv_head = model_runner.model_config.get_num_kv_heads(
            get_attention_tp_size()
        )
        self.kv_cache_dtype = model_runner.kv_cache_dtype
# ... omitted 140 lines ...
                self.num_head_padded == 16 or self.num_head_padded == 128
            ) and self.kv_cache_dtype is not fp8_dtype:
                _use_mla_ps_kernel = False
                fast_mode = False
                intra_batch_mode = False

            self.max_split_per_batch = 32 if _use_mla_ps_kernel else None

            if self.num_draft_tokens is None and _use_mla_ps_kernel:
                self.max_split_per_batch = 64

            self.fix_max_split_per_batch = self.max_split_per_batch
```
**EN:** Initializes the AiterAttnBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 AiterAttnBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 303-316: method AiterAttnBackend._get_aiter_paged_ragged_kv_cache_dtype
```python
    def _get_aiter_paged_ragged_kv_cache_dtype(self) -> str:
        """``kv_cache_dtype`` string for ``paged_attention_ragged`` (aiter ``pa/pa_ragged.py``).

        **Behavior change:** we no longer upcast FP8 KV to the activations dtype for this decode path.
        Paged K/V stay in native FP8 storage; we pass ``\"fp8_e4m3\"`` so the kernel dequants on read
        (``k_scale`` / ``v_scale``) instead of widening the cache to bf16/fp16 for ``\"auto\"``.

        **Context (short):** aiter accepts ``auto`` / ``fp8`` / ``fp8_e4m3`` only (not ``fp8_e5m2``).
        On HIP, ``configure_kv_cache_dtype`` maps CLI ``fp8_e5m2`` and ``fp8_e4m3`` to ``fp8_dtype``;
        return ``\"fp8_e4m3\"`` when ``self.kv_cache_dtype == fp8_dtype``, else ``\"auto\"``.
        """
        if self.kv_cache_dtype != fp8_dtype:
            return "auto"
        return "fp8_e4m3"
```
**EN:** Implements the get aiter paged ragged kv cache dtype routine used by this attention module.
**CN:** 实现该注意力模块使用的 get aiter paged ragged kv cache dtype 例程。

### Lines 318-379: method AiterAttnBackend.make_mla_decode_meta_data_buffer
```python
    def make_mla_decode_meta_data_buffer(self, max_seqlen_qo, batch_size):
        nhead = self.num_head_padded
        dtype = self.kv_cache_dtype

        if self.enable_dp_attention:
            gpu = torch.cuda.current_device()
            device_properties = torch.cuda.get_device_properties(gpu)
            cu_num = device_properties.multi_processor_count
            self.max_split_per_batch = min(
                (cu_num + batch_size - 1) // batch_size, self.fix_max_split_per_batch
            )

        (
            (work_meta_data_size, work_meta_data_type),
            (work_indptr_size, work_indptr_type),
            (work_info_set_size, work_info_set_type),
            (reduce_indptr_size, reduce_indptr_type),
            (reduce_final_map_size, reduce_final_map_type),
            (reduce_partial_map_size, reduce_partial_map_type),
        ) = get_mla_metadata_info_v1(
            batch_size,
            max_seqlen_qo,
            nhead,
            dtype,
            dtype,
            is_sparse=False,
            fast_mode=fast_mode,
            num_kv_splits=self.max_split_per_batch,
            intra_batch_mode=intra_batch_mode,
        )

        # aiter implementation
# ... omitted 18 lines ...
        reduce_partial_map = torch.empty(
            reduce_partial_map_size, dtype=reduce_partial_map_type, device="cuda"
        )

        return (
            work_metadata,
            work_indptr,
            work_info_set,
            reduce_indptr,
            reduce_final_map,
            reduce_partial_map,
        )
```
**EN:** Factory helper that constructs make mla decode meta data buffer and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 make mla decode meta data buffer，并校验此执行路径所需的前置条件。

### Lines 381-423: method AiterAttnBackend.make_mla_meta_data
```python
    def make_mla_meta_data(
        self,
        qo_indptr,
        kv_indptr,
        kv_last_page_len,
        work_metadata,
        work_info_set,
        work_indptr,
        reduce_indptr,
        reduce_final_map,
        reduce_partial_map,
        max_q_len,
        fast_mode,
        max_split_per_batch,
        intra_batch_mode,
    ):

        nhead_kv = 1
        page_size = self.page_size
        dtype = self.kv_cache_dtype

        meta = get_mla_metadata_v1(
            qo_indptr,
            kv_indptr,
            kv_last_page_len,
            self.num_head_padded // nhead_kv,
            nhead_kv,
            False,
            work_metadata,
            work_info_set,
            work_indptr,
            reduce_indptr,
            reduce_final_map,
            reduce_partial_map,
            kv_granularity=max(page_size, 16),
            max_seqlen_qo=max_q_len,
            uni_seqlen_qo=max_q_len,
            fast_mode=fast_mode,
            max_split_per_batch=max_split_per_batch,
            intra_batch_mode=intra_batch_mode,
            dtype_q=dtype,
            dtype_kv=dtype,
        )
```
**EN:** Factory helper that constructs make mla meta data and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 make mla meta data，并校验此执行路径所需的前置条件。

### Lines 425-467: method AiterAttnBackend.make_mla_prefill_ps_meta_data_buffer
```python
    def make_mla_prefill_ps_meta_data_buffer(
        self, batch_size: int, max_qlen: int, qlen_granularity: int
    ):
        (
            (work_meta_data_size, work_meta_data_type),
            (work_indptr_size, work_indptr_type),
            (work_info_size, work_info_type),
            (reduce_indptr_size, reduce_indptr_type),
            (reduce_final_map_size, reduce_final_map_type),
            (reduce_partial_map_size, reduce_partial_map_type),
        ) = get_ps_metadata_info_v1(
            batch_size=batch_size,
            num_head_k=self.num_kv_head,
            max_qlen=max_qlen,
            qlen_granularity=qlen_granularity,
        )

        device = self.device
        work_metadata_ptrs = torch.empty(
            work_meta_data_size, dtype=work_meta_data_type, device=device
        )
        work_indptr = torch.empty(
            work_indptr_size, dtype=work_indptr_type, device=device
        )
        work_info = torch.empty(work_info_size, dtype=work_info_type, device=device)
        reduce_indptr = torch.empty(
            reduce_indptr_size, dtype=reduce_indptr_type, device=device
        )
        reduce_final_map = torch.empty(
            reduce_final_map_size, dtype=reduce_final_map_type, device=device
        )
        reduce_partial_map = torch.empty(
            reduce_partial_map_size, dtype=reduce_partial_map_type, device=device
        )

        return (
            work_metadata_ptrs,
            work_indptr,
            work_info,
            reduce_indptr,
            reduce_final_map,
            reduce_partial_map,
        )
```
**EN:** Factory helper that constructs make mla prefill ps meta data buffer and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 make mla prefill ps meta data buffer，并校验此执行路径所需的前置条件。

### Lines 469-511: method AiterAttnBackend.make_mla_prefill_ps_meta_data
```python
    def make_mla_prefill_ps_meta_data(
        self,
        qo_indptr: torch.Tensor,
        kv_indptr: torch.Tensor,
        seq_lens: torch.Tensor,
        work_metadata: torch.Tensor,
        work_indptr: torch.Tensor,
        work_info: torch.Tensor,
        reduce_indptr: torch.Tensor,
        reduce_final_map: torch.Tensor,
        reduce_partial_map: torch.Tensor,
        is_causal: bool = True,
    ):
        gqa_ratio = self.num_head // self.num_kv_head
        num_heads_k = self.num_kv_head
        tile_q = 256
        qhead_granularity = gqa_ratio
        qlen_granularity = tile_q // qhead_granularity
        kvlen_granularity = max(128, self.page_size)
        block_size = self.page_size

        qo_indptr_cpu = qo_indptr.to("cpu", dtype=torch.int32)
        kv_indptr_cpu = kv_indptr.to("cpu", dtype=torch.int32)
        seq_lens_cpu = seq_lens.to("cpu", dtype=torch.int32)

        get_ps_metadata_v1(
            qo_indptr_cpu,
            kv_indptr_cpu,
            seq_lens_cpu,
            gqa_ratio,
            num_heads_k,
            work_metadata,
            work_indptr,
            work_info,
            reduce_indptr,
            reduce_final_map,
            reduce_partial_map,
            qhead_granularity=qhead_granularity,
            qlen_granularity=qlen_granularity,
            kvlen_granularity=kvlen_granularity,
            block_size=block_size,
            is_causal=is_causal,
        )
```
**EN:** Factory helper that constructs make mla prefill ps meta data and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 make mla prefill ps meta data，并校验此执行路径所需的前置条件。

### Lines 514-522: method AiterAttnBackend._transform_table_1_to_real
```python
    def _transform_table_1_to_real(self, page_table: torch.Tensor) -> torch.Tensor:
        page_size = self.page_size
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

### Lines 524-579: method AiterAttnBackend._build_unified_page_table_from_spec
```python
    def _build_unified_page_table_from_spec(
        self,
        spec_info,
        bs: int,
        dest_buf: Optional[torch.Tensor] = None,
        swa_dest_buf: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        """Convert ragged (token-level) kv_indices from spec_info into a 2D
        block-level page_table of shape (bs, max_num_blocks_per_seq).
        unified_attention expects max_seqlen_k = page_table.shape[1] *
        page_size to be a captured constant, so rows are sized to the
        backend-level max_num_blocks_per_seq regardless of seqused_k.
        """
        kv_indptr = spec_info.kv_indptr
        kv_flat = spec_info.kv_indices
        page_size = self.page_size
        max_blocks = (self.max_context_len + page_size - 1) // page_size

        swa_slot_mapping = None
        swa_page_table = None

        if dest_buf is not None:
            # The scatter kernel fills [0, num_blocks) and loads past that use
            # other=0, so the tail is 0-filled. Under graph replay rows > bs
            # are stale but unified_attention only walks rows [0, bs).
            page_table = dest_buf
        else:
            page_table = torch.zeros(
                bs, max_blocks, dtype=torch.int32, device=self.device
            )

        if self.use_sliding_window_kv_pool:
# ... omitted 12 lines ...
            kv_flat,
            kv_indptr,
            page_table,
            page_table.stride(0),
            swa_page_table,
            swa_slot_mapping,
            PAGE_SIZE=page_size,
            BLOCK_SIZE=BLOCK_SIZE,
            HAS_SWA=(swa_slot_mapping is not None),
        )

        return page_table, swa_page_table
```
**EN:** Implements the build unified page table from spec routine used by this attention module.
**CN:** 实现该注意力模块使用的 build unified page table from spec 例程。

### Lines 581-644: method AiterAttnBackend._build_verify_unified_metadata
```python
    def _build_verify_unified_metadata(
        self,
        bs: int,
        seq_lens: torch.Tensor,
        req_pool_indices: torch.Tensor,
        draft_num: int,
        page_table_dest: Optional[torch.Tensor] = None,
        swa_page_table_dest: Optional[torch.Tensor] = None,
    ):
        """Build the 2D block page_table + qo_indptr for EAGLE target_verify
        through unified_attention. Assumes the new draft K/V have already been
        written by set_kv_buffer, so req_to_token[rp, :seq_lens[i]+draft_num]
        covers both the prefix and the freshly committed draft tokens. Returns
        (page_table, qo_indptr, max_q_len=draft_num).
        """
        device = seq_lens.device
        qo_indptr = self.qo_indptr[: bs + 1]
        qo_indptr[: bs + 1] = torch.arange(
            0,
            (1 + bs) * draft_num,
            step=draft_num,
            dtype=torch.int32,
            device=device,
        )

        page_size = self.page_size
        max_blocks = (self.max_context_len + page_size - 1) // page_size

        swa_slot_mapping = None
        swa_page_table = None

        if page_table_dest is not None:
# ... omitted 20 lines ...
            page_table,
            self.req_to_token.stride(0),
            page_table.stride(0),
            swa_page_table,
            swa_slot_mapping,
            DRAFT_NUM=draft_num,
            PAGE_SIZE=page_size,
            BLOCK_SIZE=BLOCK_SIZE,
            HAS_SWA=(swa_slot_mapping is not None),
        )

        return page_table, qo_indptr, draft_num, swa_page_table
```
**EN:** Implements the build verify unified metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 build verify unified metadata 例程。

### Lines 646-679: method AiterAttnBackend._resolve_v2_num_draft_tokens
```python
    def _resolve_v2_num_draft_tokens(
        self,
        extend_seq_lens: Optional[torch.Tensor] = None,
        extend_seq_lens_cpu: Optional[list[int]] = None,
    ) -> int:
        """Resolve fixed per-request extend length for DRAFT_EXTEND_V2."""
        num_draft_tokens = self.num_draft_tokens
        if num_draft_tokens is None:
            if extend_seq_lens is not None and extend_seq_lens.numel() > 0:
                # Avoid list scans in hot path when tensor lengths are already available.
                num_draft_tokens = int(extend_seq_lens[0].item())
            elif extend_seq_lens_cpu:
                num_draft_tokens = max(extend_seq_lens_cpu)
            else:
                raise ValueError(
                    "DRAFT_EXTEND_V2 requires speculative_num_draft_tokens or "
                    "non-empty extend_seq_lens/extend_seq_lens_cpu."
                )

        num_draft_tokens = int(num_draft_tokens)
        if extend_seq_lens is not None and extend_seq_lens.numel() > 0:
            if not torch.all(extend_seq_lens == num_draft_tokens):
                raise ValueError(
                    "DRAFT_EXTEND_V2 expects fixed extend length per request; got "
                    f"extend_seq_lens={extend_seq_lens}, expected all == {num_draft_tokens}."
                )
        if extend_seq_lens_cpu and any(
            x != num_draft_tokens for x in extend_seq_lens_cpu
        ):
            raise ValueError(
                "DRAFT_EXTEND_V2 expects fixed extend length per request; got "
                f"{extend_seq_lens_cpu}, expected all == {num_draft_tokens}."
            )
        return num_draft_tokens
```
**EN:** Implements the resolve v2 num draft tokens routine used by this attention module.
**CN:** 实现该注意力模块使用的 resolve v2 num draft tokens 例程。

### Lines 681-692: method AiterAttnBackend._get_kv_indices_scratch
```python
    def _get_kv_indices_scratch(
        self, required_tokens: int, device: torch.device
    ) -> torch.Tensor:
        if (
            self._kv_indices_scratch is None
            or self._kv_indices_scratch.device != device
            or self._kv_indices_scratch.numel() < required_tokens
        ):
            self._kv_indices_scratch = torch.empty(
                required_tokens, dtype=torch.int32, device=device
            )
        return self._kv_indices_scratch[:required_tokens]
```
**EN:** Implements the get kv indices scratch routine used by this attention module.
**CN:** 实现该注意力模块使用的 get kv indices scratch 例程。

### Lines 694-705: method AiterAttnBackend._set_uniform_qo_indptr
```python
    def _set_uniform_qo_indptr(
        self, bs: int, tokens_per_req: int, device: torch.device
    ) -> torch.Tensor:
        qo_indptr = self.qo_indptr[: bs + 1]
        qo_indptr[: bs + 1] = torch.arange(
            0,
            bs * tokens_per_req + 1,
            step=tokens_per_req,
            dtype=torch.int32,
            device=device,
        )
        return qo_indptr
```
**EN:** Implements the set uniform qo indptr routine used by this attention module.
**CN:** 实现该注意力模块使用的 set uniform qo indptr 例程。

### Lines 707-712: method AiterAttnBackend._ensure_spec_v2_topk_supported
```python
    def _ensure_spec_v2_topk_supported(self):
        if self.topk > 1:
            raise NotImplementedError(
                "AiterAttnBackend SPEC_V2 path currently supports topk <= 1 only. "
                f"Got topk={self.topk}."
            )
```
**EN:** Implements the ensure spec v2 topk supported routine used by this attention module.
**CN:** 实现该注意力模块使用的 ensure spec v2 topk supported 例程。

### Lines 714-742: method AiterAttnBackend._mla_decode_fwd_with_head_pad
```python
    def _mla_decode_fwd_with_head_pad(
        self,
        q: torch.Tensor,
        k_buffer_flat: torch.Tensor,
        layer,
        **kwargs,
    ):
        """Wrap mla_decode_fwd with head-dimension padding for num_head < 16.

        When head_repeat_factor > 1 (i.e. num_head is 4 or 8), q is
        repeat-interleaved to reach num_head_padded (16) before the kernel
        call, and the corresponding output columns are sliced back afterward.
        q / o must already be shaped (..., num_head, head_dim).
        """
        if self.head_repeat_factor > 1:
            q_in = q.repeat_interleave(self.head_repeat_factor, dim=1)
            o = q.new_empty(
                (q.shape[0], self.num_head_padded, layer.v_head_dim),
                dtype=self.input_dtype,
            )
            mla_decode_fwd(q_in, k_buffer_flat, o, **kwargs)
            return o[:, :: self.head_repeat_factor, :]
        else:
            o = q.new_empty(
                (q.shape[0], layer.tp_q_head_num, layer.v_head_dim),
                dtype=self.input_dtype,
            )
            mla_decode_fwd(q, k_buffer_flat, o, **kwargs)
            return o
```
**EN:** Implements the mla decode fwd with head pad routine used by this attention module.
**CN:** 实现该注意力模块使用的 mla decode fwd with head pad 例程。

### Lines 744-817: method AiterAttnBackend.mla_fp8_prefill_attn
```python
    def mla_fp8_prefill_attn(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
    ):
        total_q = q.shape[0]
        nhead = layer.tp_q_head_num
        v_head_dim = layer.v_head_dim

        if q.dtype != fp8_dtype:
            q = q.to(fp8_dtype)
        if k.dtype != fp8_dtype:
            k = k.to(fp8_dtype)
        if v.dtype != fp8_dtype:
            v = v.to(fp8_dtype)
        one_scale = torch.ones((), dtype=torch.float32, device=q.device)

        tile_q = 256
        reduce_indptr = self.forward_metadata.reduce_indptr
        reduce_final_map = self.forward_metadata.reduce_final_map
        reduce_partial_map = self.forward_metadata.reduce_partial_map

        logits = torch.empty(
            (reduce_partial_map.size(0) * tile_q, nhead, v_head_dim),
            dtype=torch.float32,
            device=q.device,
        )
        attn_lse = torch.empty(
            (reduce_partial_map.size(0) * tile_q, nhead),
            dtype=torch.float32,
# ... omitted 30 lines ...
        )
        mla_reduce_v1(
            logits,
            attn_lse,
            reduce_indptr,
            reduce_final_map,
            reduce_partial_map,
            tile_q,
            output,
            final_lse,
        )
        return output
```
**EN:** Implements the mla fp8 prefill attn routine used by this attention module.
**CN:** 实现该注意力模块使用的 mla fp8 prefill attn 例程。

### Lines 819-1372: method AiterAttnBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        """Init auxiliary variables for aiter attention backend."""

        bs = forward_batch.batch_size
        kv_indptr = self.kv_indptr
        spec_info = forward_batch.spec_info
        qo_indptr = None
        kv_last_page_len = None
        max_q_len = None
        max_kv_len = None

        work_metadata = None
        work_indptr = None
        work_info_set = None
        reduce_indptr = None
        reduce_final_map = None
        reduce_partial_map = None

        num_kv_splits = None
        swa_page_table = None
        max_kv_len = forward_batch.seq_lens_cpu.max().item()

        if forward_batch.forward_mode.is_decode_or_idle():
            if spec_info is None or forward_batch.forward_mode.is_idle():
                kv_indptr[1 : bs + 1] = torch.cumsum(forward_batch.seq_lens, dim=0)
                kv_indptr = kv_indptr[: bs + 1]

                if not self.use_triton_unified_attention:
                    kv_indices = self._get_kv_indices_scratch(
                        forward_batch.seq_lens_sum, forward_batch.seq_lens.device
                    )
                    create_flashinfer_kv_indices_triton[(bs,)](
# ... omitted 510 lines ...
                        )
                    )

                self.forward_metadata = ForwardMetadata(
                    self.indices_updater_prefill.kv_indptr,
                    self.indices_updater_prefill.kv_indices,
                    None,
                    None,
                    max(forward_batch.extend_seq_lens_cpu),
                    forward_batch.seq_lens_cpu.max().item(),
                    swa_page_table=swa_page_table,
                )
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1374-1453: method AiterAttnBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(
        self,
        max_bs: int,
        max_num_tokens: int,
        kv_indices_buf: Optional[torch.Tensor] = None,
    ):
        # PR #20978 pads max_bs beyond pool_size for higher cuda-graph
        # coverage. Reallocate indptr buffers so they fit the padded max_bs.
        # See: https://github.com/sgl-project/sglang/pull/20978
        if max_bs + 1 > self.kv_indptr.shape[0]:
            self.kv_indptr = torch.zeros(
                (max_bs + 1,), dtype=torch.int32, device=self.device
            )
            self.qo_indptr = torch.zeros(
                (max_bs + 1,), dtype=torch.int32, device=self.device
            )
            self.mask_indptr = torch.zeros(
                (max_bs + 1,), dtype=torch.int64, device=self.device
            )
            if hasattr(self, "qo_indptr_"):
                self.qo_indptr_ = torch.zeros(
                    (max_bs + 1,), dtype=torch.int32, device=self.device
                )

        self.cuda_graph_kv_last_page_len = torch.ones(
            max_bs, dtype=torch.int32, device=self.device
        )
        if kv_indices_buf is None:
            max_num_blocks_per_seq = (
                self.max_context_len + self.page_size - 1
            ) // self.page_size
            self.cuda_graph_kv_indices = torch.zeros(
# ... omitted 36 lines ...
            self.reduce_final_map = None
            self.reduce_partial_map = None

        if self.use_sliding_window_kv_pool:
            max_num_blocks_per_seq = (
                self.max_context_len + self.page_size - 1
            ) // self.page_size
            self.cuda_graph_swa_page_table = torch.zeros(
                (max_bs, max_num_blocks_per_seq),
                dtype=torch.int32,
                device=self.device,
            )
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 1455-1885: method AiterAttnBackend.init_forward_metadata_capture_cuda_graph
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

        num_kv_splits = None
        # num_kv_splits_indptr = None

        work_metadata = None
        work_info_set = None
        work_indptr = None

        reduce_indptr = None
        reduce_final_map = None
        reduce_partial_map = None

        swa_page_table = None

        max_kv_len = torch.max(seq_lens).item()

        if forward_mode.is_decode_or_idle():
            qo_indptr = None
            kv_last_page_len = None
            max_q_len = None

            if spec_info is None or (
# ... omitted 387 lines ...
                    kv_indptr,
                    kv_indices,
                    qo_indptr,
                    None,
                    num_tokens_per_bs,
                    None,
                    custom_mask=None,
                    mask_indptr=None,
                    max_extend_len=num_tokens_per_bs,
                )
        else:
            raise ValueError(f"Invalid mode: {forward_mode=}")
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1887-2309: method AiterAttnBackend.init_forward_metadata_replay_cuda_graph
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
    ):

        num_kv_splits = None
        # num_kv_splits_indptr = None

        work_metadata = None
        work_info_set = None
        work_indptr = None

        reduce_indptr = None
        reduce_final_map = None
        reduce_partial_map = None

        swa_page_table = None
        max_kv_len = seq_lens_cpu.max().item()

        if forward_mode.is_decode_or_idle():
            qo_indptr = None
            kv_last_page_len = None
            max_q_len = None

            if spec_info is None or (
# ... omitted 379 lines ...
                max_kv_len,
                work_metadata=work_metadata,
                work_info_set=work_info_set,
                work_indptr=work_indptr,
                reduce_indptr=reduce_indptr,
                reduce_final_map=reduce_final_map,
                reduce_partial_map=reduce_partial_map,
                num_kv_splits=num_kv_splits,
            )

        else:
            raise ValueError("Invalid forward mode")
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2311-2312: method AiterAttnBackend.get_cuda_graph_seq_len_fill_value
```python
    def get_cuda_graph_seq_len_fill_value(self):
        return 1 if self.num_draft_tokens is None else self.num_draft_tokens
```
**EN:** Computes and returns get cuda graph seq len fill value from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cuda graph seq len fill value。

### Lines 2314-2320: method AiterAttnBackend.update_verify_buffers_to_fill_after_draft
```python
    def update_verify_buffers_to_fill_after_draft(
        self, spec_info: SpecInput, cuda_graph_bs: Optional[int]
    ):
        # AITER verify path does not require post-draft buffer patching currently.
        # This override prevents overlap-plan stream mode from failing with the
        # base class NotImplementedError.
        pass
```
**EN:** Updates update verify buffers to fill after draft on the active object so later attention steps observe the latest runtime state.
**CN:** 更新活动对象上的 update verify buffers to fill after draft，以便后续注意力步骤读取最新运行时状态。

### Lines 2322-2774: method AiterAttnBackend.forward_extend
```python
    def forward_extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        sinks=None,
    ):
        self.logits_soft_cap = layer.logit_cap

        cache_loc = (
            forward_batch.out_cache_loc
            if not layer.is_cross_attention
            else forward_batch.encoder_out_cache_loc
        )

        k_descale = None
        v_descale = None
        if self.kv_cache_dtype == fp8_dtype:
            k_descale = layer.k_scale if layer.k_scale is not None else self.k_scale
            v_descale = layer.v_scale if layer.v_scale is not None else self.k_scale

        if k is not None:
            assert v is not None
            if save_kv_cache:
                # Only use SWA-specific kv cache write (reshape_and_cache_flash) when
                # both unified attention and sliding window kv pool are active.
                # Non-SWA models (e.g. Qwen3-VL) enabled via SGLANG_USE_AITER_UNIFIED_ATTN
                # use standard set_kv_buffer, as they lack SWA-specific attributes
                # like full_to_swa_index_mapping.
# ... omitted 409 lines ...
                q_descale=q_descale,
                k_descale=k_descale,
                v_descale=v_descale,
            )

            # The fp8bf16 aiter prefill kernel returns bf16 even when the
            # model computes in fp16. Cast back so the attention output keeps
            # the same dtype as the rest of the model activations.
            if o.dtype != self.input_dtype:
                o = o.to(self.input_dtype)

            return o.view(-1, layer.tp_q_head_num * layer.head_dim)
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2776-2959: method AiterAttnBackend.forward_decode
```python
    def forward_decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        sinks=None,
    ):
        q = q.reshape(-1, layer.tp_q_head_num * layer.qk_head_dim)

        k_descale = None
        v_descale = None
        if self.kv_cache_dtype == fp8_dtype:
            k_descale = layer.k_scale if layer.k_scale is not None else self.k_scale
            v_descale = layer.v_scale if layer.v_scale is not None else self.k_scale

        if save_kv_cache:
            # Only use SWA-specific kv cache write (reshape_and_cache_flash) when
            # both unified attention and sliding window kv pool are active.
            # Non-SWA models (e.g. Qwen3-VL) enabled via SGLANG_USE_AITER_UNIFIED_ATTN
            # use standard set_kv_buffer, as they lack SWA-specific attributes
            # like full_to_swa_index_mapping.
            if self.use_triton_unified_attention and self.use_sliding_window_kv_pool:
                token_to_kv_pool = forward_batch.token_to_kv_pool
                k_cache, v_cache = forward_batch.token_to_kv_pool.get_kv_buffer(
                    layer.layer_id
                )
                slot_mapping_swa = token_to_kv_pool.full_to_swa_index_mapping

                launch_reshape_and_cache_flash(
# ... omitted 140 lines ...
                    self.max_num_partitions,
                    None,
                    aiter_kv_str,
                    "NHD",
                    self.logits_soft_cap,
                    self.k_scale,
                    self.v_scale,
                    None,
                    _AITER_PARTITION_SIZE_ROCM,
                )

        return o
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2962-2962: class AiterIndicesUpdaterPrefill
```python
class AiterIndicesUpdaterPrefill:
```
**EN:** Defines the aiter indices updater prefill type and the state it exposes to the rest of the attention stack.
**CN:** 定义 aiter indices updater prefill 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 2963-2986: method AiterIndicesUpdaterPrefill.__init__
```python
    def __init__(self, model_runner: ModelRunner, attn_backend: AttentionBackend):
        # Parse Constants
        self.num_qo_heads = (
            model_runner.model_config.num_attention_heads // get_attention_tp_size()
        )
        self.num_kv_heads = model_runner.model_config.get_num_kv_heads(
            get_attention_tp_size()
        )
        self.head_dim = model_runner.model_config.head_dim
        self.data_type = model_runner.kv_cache_dtype
        self.q_data_type = model_runner.dtype
        self.sliding_window_size = model_runner.sliding_window_size
        self.attn_backend = attn_backend

        # Buffers and wrappers
        self.kv_indptr = attn_backend.kv_indptr
        self.kv_last_page_len = attn_backend.kv_last_page_len
        self.qo_indptr = attn_backend.qo_indptr
        self.req_to_token = model_runner.req_to_token_pool.req_to_token
        self.update = self.update_single_wrapper

        self.kv_indices = None
        self.max_q_len = 0
        self.max_kv_len = 0
```
**EN:** Initializes the AiterIndicesUpdaterPrefill instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 AiterIndicesUpdaterPrefill 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 2988-2998: method AiterIndicesUpdaterPrefill.update
```python
    def update(
        self,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_sum: int,
        prefix_lens: torch.Tensor,
        encoder_lens: Optional[torch.Tensor],
        spec_info: Optional[SpecInput],
    ):
        # Keep the signature for type checking. It will be assigned during runtime.
        raise NotImplementedError()
```
**EN:** Implements the update routine used by this attention module.
**CN:** 实现该注意力模块使用的 update 例程。

### Lines 3000-3061: method AiterIndicesUpdaterPrefill.update_single_wrapper
```python
    def update_single_wrapper(
        self,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_sum: int,
        prefix_lens: torch.Tensor,
        encoder_lens: Optional[torch.Tensor],
        spec_info: Optional[SpecInput],
    ):

        kv_start_idx = None
        kv_indptr = self.kv_indptr
        qo_indptr = self.qo_indptr
        paged_kernel_lens = seq_lens
        paged_kernel_lens_sum = seq_lens_sum

        bs = len(req_pool_indices)
        if spec_info is None:
            # Normal extend
            kv_indptr[1 : bs + 1] = torch.cumsum(paged_kernel_lens, dim=0)
            kv_indptr = kv_indptr[: bs + 1]

            # (TODO: Kk) WA - CI test_moe_eval_accuracy_large.py
            # mha_batch_prefill reads 128 data to do computatoin
            # if real data is not long enough then original padding value 0 is used
            # but the 0 location will be made nan (noqa) in cuda graph capture mode
            # this will cause the output tensor value becomes nan
            # WA is to assure that last index of pool not changed
            kv_indices = torch.empty(
                paged_kernel_lens_sum + 256,
                dtype=torch.int32,
                device=req_pool_indices.device,
# ... omitted 18 lines ...
            custom_mask = None
        else:
            kv_indices, kv_indptr, qo_indptr, custom_mask = (
                spec_info.generate_attn_arg_prefill(
                    req_pool_indices,
                    paged_kernel_lens,
                    paged_kernel_lens_sum,
                    self.req_to_token,
                )
            )

        self.kv_indices = kv_indices
```
**EN:** Updates update single wrapper on the active object so later attention steps observe the latest runtime state.
**CN:** 更新活动对象上的 update single wrapper，以便后续注意力步骤读取最新运行时状态。

### Lines 3064-3064: class AiterMlaIndicesUpdaterPrefill
```python
class AiterMlaIndicesUpdaterPrefill:
```
**EN:** Defines the aiter mla indices updater prefill type and the state it exposes to the rest of the attention stack.
**CN:** 定义 aiter mla indices updater prefill 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 3065-3078: method AiterMlaIndicesUpdaterPrefill.__init__
```python
    def __init__(self, model_runner: ModelRunner, attn_backend: AttentionBackend):
        # Parse Constants
        self.attn_backend = attn_backend

        # Buffers and wrappers
        self.req_to_token = model_runner.req_to_token_pool.req_to_token
        self.update = self.update_single_wrapper

        self.kv_indptr = None
        self.kv_indices = None
        self.qo_indptr = None
        self.kv_last_page_len = None
        self.max_q_len = 0
        self.max_kv_len = 0
```
**EN:** Initializes the AiterMlaIndicesUpdaterPrefill instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 AiterMlaIndicesUpdaterPrefill 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 3080-3091: method AiterMlaIndicesUpdaterPrefill.update
```python
    def update(
        self,
        req_pool_indices: torch.Tensor,
        kv_lens: torch.Tensor,
        kv_lens_sum: int,
        extend_lens: torch.Tensor,
        max_q_len: int,
        max_kv_len: int,
        spec_info: Optional[SpecInput],
    ):
        # Keep the signature for type checking. It will be assigned during runtime.
        raise NotImplementedError()
```
**EN:** Implements the update routine used by this attention module.
**CN:** 实现该注意力模块使用的 update 例程。

### Lines 3093-3143: method AiterMlaIndicesUpdaterPrefill.update_single_wrapper
```python
    def update_single_wrapper(
        self,
        req_pool_indices: torch.Tensor,
        kv_lens: torch.Tensor,
        kv_lens_sum: int,
        extend_lens: torch.Tensor,
        max_q_len: int,
        max_kv_len: int,
        spec_info: Optional[SpecInput],
    ):
        bs = len(req_pool_indices)

        kv_indptr = self.attn_backend.kv_indptr

        if spec_info is None:
            # Normal extend
            kv_indptr[1 : bs + 1] = torch.cumsum(kv_lens, dim=0)
            kv_indptr = kv_indptr[: bs + 1]
            kv_indices = torch.empty(
                kv_lens_sum,
                dtype=torch.int32,
                device=req_pool_indices.device,
            )
            create_flashinfer_kv_indices_triton[(bs,)](
                self.req_to_token,
                req_pool_indices,
                kv_lens,
                kv_indptr,
                None,
                kv_indices,
                self.req_to_token.stride(0),
            )
# ... omitted 7 lines ...
                    req_pool_indices,
                    kv_lens,
                    kv_lens_sum,
                    self.req_to_token,
                )
            )

        self.kv_indptr = kv_indptr
        self.kv_indices = kv_indices
        self.qo_indptr = qo_indptr
        self.max_q_len = max_q_len
        self.max_kv_len = max_kv_len
```
**EN:** Updates update single wrapper on the active object so later attention steps observe the latest runtime state.
**CN:** 更新活动对象上的 update single wrapper，以便后续注意力步骤读取最新运行时状态。

### Lines 3146-3151: class AiterMultiStepDraftBackend
```python
class AiterMultiStepDraftBackend:
    """
    Wrap multiple triton attention backends as one for multiple consecutive
    draft decoding steps.
    """
```
**EN:** Concrete attention backend that connects aiter multi step draft backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 aiter multi step draft backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 3152-3189: method AiterMultiStepDraftBackend.__init__
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        topk: int,
        speculative_num_steps: int,
    ):
        from sglang.srt.speculative.spec_utils import generate_draft_decode_kv_indices

        self.topk = topk
        self.speculative_num_steps = speculative_num_steps
        self.generate_draft_decode_kv_indices = generate_draft_decode_kv_indices
        max_bs = model_runner.req_to_token_pool.size * self.topk
        self.kv_indptr = torch.zeros(
            (
                self.speculative_num_steps,
                max_bs + 1,
            ),
            dtype=torch.int32,
            device=model_runner.device,
        )
        self.attn_backends = []
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends.append(
                AiterAttnBackend(
                    model_runner,
                    skip_prefill=True,
                    kv_indptr_buf=self.kv_indptr[i],
                    topk=topk,
                )
            )
        self.max_context_len = self.attn_backends[0].max_context_len
        self.num_head = (
            model_runner.model_config.num_attention_heads // get_attention_tp_size()
        )
        self.device = model_runner.device
        # Cached variables for generate_draft_decode_kv_indices
        self.pool_len = model_runner.req_to_token_pool.req_to_token.shape[1]
        self.page_size = model_runner.server_args.page_size
```
**EN:** Initializes the AiterMultiStepDraftBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 AiterMultiStepDraftBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 3191-3221: method AiterMultiStepDraftBackend.common_template
```python
    def common_template(
        self, forward_batch: ForwardBatch, kv_indices_buffer: torch.Tensor, call_fn: int
    ):
        num_seqs = forward_batch.batch_size
        bs = self.topk * num_seqs
        seq_lens_sum = forward_batch.seq_lens_sum

        self.generate_draft_decode_kv_indices[
            (self.speculative_num_steps, num_seqs, self.topk)
        ](
            forward_batch.req_pool_indices,
            forward_batch.req_to_token_pool.req_to_token,
            forward_batch.seq_lens,
            kv_indices_buffer,
            self.kv_indptr,
            forward_batch.positions,
            self.pool_len,
            kv_indices_buffer.shape[1],
            self.kv_indptr.shape[1],
            triton.next_power_of_2(num_seqs),
            triton.next_power_of_2(self.speculative_num_steps),
            triton.next_power_of_2(bs),
            self.page_size,
        )

        for i in range(self.speculative_num_steps - 1):
            forward_batch.spec_info.kv_indptr = self.kv_indptr[i, : bs + 1]
            forward_batch.spec_info.kv_indices = kv_indices_buffer[i][
                : seq_lens_sum * self.topk + bs * (i + 1)
            ]
            call_fn(i, forward_batch)
```
**EN:** Implements the common template routine used by this attention module.
**CN:** 实现该注意力模块使用的 common template 例程。

### Lines 3223-3242: method AiterMultiStepDraftBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        kv_indices = torch.empty(
            (
                self.speculative_num_steps,
                forward_batch.batch_size * self.topk * self.max_context_len,
            ),
            dtype=torch.int32,
            device=self.device,
        )

        def call_fn(i, forward_batch):
            forward_batch.spec_info.kv_indptr = (
                forward_batch.spec_info.kv_indptr.clone()
            )
            forward_batch.spec_info.kv_indices = (
                forward_batch.spec_info.kv_indices.clone()
            )
            self.attn_backends[i].init_forward_metadata(forward_batch)

        self.common_template(forward_batch, kv_indices, call_fn)
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 3244-3253: method AiterMultiStepDraftBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        self.cuda_graph_kv_indices = torch.zeros(
            (self.speculative_num_steps, max_num_tokens * self.max_context_len),
            dtype=torch.int32,
            device=self.device,
        )
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_cuda_graph_state(
                max_bs, max_num_tokens, kv_indices_buf=self.cuda_graph_kv_indices[i]
            )
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 3255-3267: method AiterMultiStepDraftBackend.init_forward_metadata_capture_cuda_graph
```python
    def init_forward_metadata_capture_cuda_graph(self, forward_batch: ForwardBatch):
        def call_fn(i, forward_batch):
            self.attn_backends[i].init_forward_metadata_capture_cuda_graph(
                forward_batch.batch_size,
                forward_batch.batch_size * self.topk,
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                encoder_lens=None,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
            )

        self.common_template(forward_batch, self.cuda_graph_kv_indices, call_fn)
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 3269-3284: method AiterMultiStepDraftBackend.init_forward_metadata_replay_cuda_graph
```python
    def init_forward_metadata_replay_cuda_graph(
        self, forward_batch: ForwardBatch, bs: int
    ):
        def call_fn(i, forward_batch):
            self.attn_backends[i].init_forward_metadata_replay_cuda_graph(
                bs,
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                seq_lens_sum=-1,
                encoder_lens=None,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
                seq_lens_cpu=forward_batch.seq_lens_cpu,
            )

        self.common_template(forward_batch, self.cuda_graph_kv_indices, call_fn)
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
- `logging`
- `dataclasses.dataclass`
- `enum.Enum`
- `enum.auto`
- `typing.TYPE_CHECKING`
- `typing.Optional`
- `torch`
- `triton`
- `sglang.srt.layers.attention.base_attn_backend.AttentionBackend`
- `sglang.srt.layers.attention.triton_ops.aiter_unified_attention.scatter_ragged_to_page_table_kernel`
- `sglang.srt.layers.attention.triton_ops.aiter_unified_attention.scatter_req_to_token_to_page_table_kernel`
- `sglang.srt.layers.attention.utils.create_flashinfer_kv_indices_triton`
- `sglang.srt.layers.attention.utils.create_flashmla_kv_indices_triton`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.utils.is_gfx95_supported`
- `sglang.srt.configs.model_config.AttentionArch`
- `sglang.srt.layers.attention.utils.launch_reshape_and_cache_flash`
- `sglang.srt.layers.attention.utils.pad_sequence_with_mask`
- `sglang.srt.layers.quantization.fp8_kernel.fp8_dtype`
- `sglang.srt.mem_cache.swa_memory_pool.SWAKVPool`
- `sglang.srt.utils.get_bool_env_var`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.model_runner.ModelRunner`
- `sglang.srt.speculative.spec_info.SpecInput`
- `aiter.flash_attn_varlen_func`
- `aiter.get_mla_metadata_info_v1`
- `aiter.get_mla_metadata_v1`
- `aiter.get_ps_metadata_info_v1`
- `aiter.get_ps_metadata_v1`
- `aiter.mha_batch_prefill_func`
- `aiter.mla_prefill_ps_asm_fwd`
- `aiter.mla_reduce_v1`
- `aiter.paged_attention_ragged`
- `aiter.mla.mla_decode_fwd`
- `aiter.mla.mla_prefill_fwd`
- `aiter.ops.triton.attention.unified_attention.unified_attention`
- `sglang.srt.layers.attention.triton_ops.extend_attention.extend_attention_fwd`
- `sglang.srt.speculative.spec_utils.generate_draft_decode_kv_indices`
