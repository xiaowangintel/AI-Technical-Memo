# flashattention_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/flashattention_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the flashattention backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 flashattention backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-22: imports
```python
from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional

import numpy as np
import torch
import triton
import triton.language as tl

from sglang.srt.configs.model_config import AttentionArch
from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.layers.radix_attention import AttentionType
from sglang.srt.layers.utils.cp_utils import (
    cp_allgather_and_save_kv_cache,
    cp_attn_forward_extend,
)
from sglang.srt.mem_cache.swa_memory_pool import SWAKVPool
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.server_args import get_global_server_args
from sglang.srt.speculative.spec_info import SpecInput
from sglang.srt.utils import get_compiler_backend
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 24-26: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 28-33: imports
```python
from sgl_kernel import merge_state_v2

from sglang.jit_kernel.flash_attention import (
    flash_attn_varlen_func,
    flash_attn_with_kvcache,
)
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 36-72: class FlashAttentionMetadata
```python
@dataclass
class FlashAttentionMetadata:
    """Metadata to be init once in the model forward pass,
    each layer's forward pass can reuse the metadata.

    For each init metadata function, we will try set up them in below order
    """

    # Sequence lengths for the forward batch
    cache_seqlens_int32: torch.Tensor = None
    # Maximum sequence length for query
    max_seq_len_q: int = 1
    # Maximum sequence length for key
    max_seq_len_k: int = 0
    # Cumulative sequence lengths for query
    cu_seqlens_q: torch.Tensor = None
    # Cumulative sequence lengths for key
    cu_seqlens_k: torch.Tensor = None
    # Window size (typically used by Gemma)
    window_size: tuple = (-1, -1)
    # Page table, the index of KV Cache Tables/Blocks
    page_table: torch.Tensor = None
    # Page table for Sliding Window Attention
    swa_page_table: torch.Tensor = None
    # Precomputed FA3 scheduler metadata (avoids per-layer prepare_varlen_num_blocks)
    scheduler_metadata: torch.Tensor = None

    # Encoder metadata
    # Cumulative sequence lengths for encoder key
    encoder_cu_seqlens_k: torch.Tensor = None
    # Maximum sequence length for encoder key
    encoder_max_seq_len_k: int = 0
    # Sequence lengths for the forward batch
    encoder_lens_int32: torch.Tensor = None
    # Page table for the encoder
    encoder_page_table: torch.Tensor = None
```
**EN:** Dataclass-style container that stores structured runtime state for flash attention metadata.
**CN:** 该数据类风格的容器用于存储 flash attention metadata 的结构化运行时状态。

### Lines 73-79: class LocalAttentionMetadata
```python
    @dataclass
    class LocalAttentionMetadata:
        local_query_start_loc: torch.Tensor = None  # cu_seqlens_q for local attention
        local_seqused_k: torch.Tensor = None  # sequence lengths for local attention
        local_block_table: torch.Tensor = None  # block table for local attention
        local_max_query_len: int = 0  # max query length for local attention
        local_max_seq_len: int = 0  # max sequence length for local attention
```
**EN:** Dataclass-style container that stores structured runtime state for local attention metadata.
**CN:** 该数据类风格的容器用于存储 local attention metadata 的结构化运行时状态。

### Lines 81-84: FlashAttentionMetadata fields
```python
    local_attn_metadata: Optional[LocalAttentionMetadata] = None

    # For sliding window attention topk>1 spec decoding
    swa_spec_metadata: Optional[FlashAttentionMetadata] = None
```
**EN:** Defines class-level fields, defaults, or reusable constants consumed by later methods.
**CN:** 定义后续方法会使用的类级字段、默认值或可复用常量。

### Lines 87-104: class FlashAttentionBackend
```python
class FlashAttentionBackend(AttentionBackend):
    """FlashAttention backend implementation.

    Note about the init:
    - If no spec decoding
        - FlashAttentionBackend will be init once when the server starts.
    - If spec decoding
        - FlashAttentionBackend will be init once for the target worker
        - FlashAttentionMultiStepBackend will be once for the draft worker
            - It will spawn num_steps FlashAttentionBackend for the draft worker

    Note about CUDA Graph:
    - We only support CUDA Graph for Decode (Normal Decode and Draft Decode) and Target Verify.
    - We don't support CUDA Graph for Extend and Draft Extend.
    - When server init, init_cuda_graph_state will be called first and then init_cuda_graph_capture will be called.
    - For each forward batch, init_replay_cuda_graph will be called first and then replay the graph.
    """
```
**EN:** Concrete attention backend that connects flash attention backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 flash attention backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 105-241: method FlashAttentionBackend.__init__
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        skip_prefill: bool = False,
        speculative_step_id=0,
        topk=0,
        speculative_num_steps=0,
        fa_impl_ver=3,
    ):
        super().__init__()

        assert not (
            model_runner.sliding_window_size is not None
            and model_runner.model_config.is_encoder_decoder
        ), "Sliding window and cross attention are not supported together"

        self.is_encoder_decoder = model_runner.model_config.is_encoder_decoder
        self.forward_metadata: FlashAttentionMetadata = None
        # extra metadata for handling speculative decoding topk > 1, extended draft decode and verify
        self.forward_metadata_spec_decode_expand: FlashAttentionMetadata = None
        self.max_context_len = model_runner.model_config.context_len
        self.device = model_runner.device
        self.decode_cuda_graph_metadata = {}
        self.target_verify_metadata = {}
        self.req_to_token = model_runner.req_to_token_pool.req_to_token
        self.kv_cache_dtype = model_runner.kv_cache_dtype
        self.kv_cache_dtype_str = model_runner.server_args.kv_cache_dtype
        self.page_size = model_runner.page_size
        self.use_mla = model_runner.model_config.attention_arch == AttentionArch.MLA
        self.skip_prefill = skip_prefill
        self.attn_cp_size = model_runner.attn_cp_size

# ... omitted 93 lines ...
            and not self.use_mla
        )

        # Skip the FA3 scheduler_metadata precompute (PR #21104) under DP
        # attention. The precomputed buffer can become inconsistent with the
        # num_splits the C++ mha_fwd kernel derives from live cache_seqlens
        # during decode, leading to an OOB read in the split-KV combine kernel
        # (flash_fwd_combine_launch_template.h:52). Leaving scheduler_metadata
        # unset uses the existing per-layer metadata path.
        self._disable_scheduler_metadata_precompute = bool(
            getattr(server_args, "enable_dp_attention", False)
        )
```
**EN:** Initializes the FlashAttentionBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 FlashAttentionBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 243-270: method FlashAttentionBackend._compute_scheduler_metadata
```python
    def _compute_scheduler_metadata(
        self, batch_size, max_seq_len_k, cache_seqlens, cu_seqlens_q
    ):
        """Compute FA3 scheduler metadata for decode.

        Returns the scheduler_metadata tensor, or None if not applicable.
        """
        if self._get_scheduler_metadata is None or self.use_mla:
            return None
        if self._disable_scheduler_metadata_precompute:
            return None
        # Always use window_size=(-1, -1) because scheduler_metadata is only
        # consumed by non-SWA layers (SWA layers skip it in forward_decode).
        return self._get_scheduler_metadata(
            batch_size=batch_size,
            max_seqlen_q=1,
            max_seqlen_k=max_seq_len_k,
            num_heads=self.num_attention_heads,
            num_heads_k=self.num_kv_heads,
            headdim=self.head_dim,
            cache_seqlens=cache_seqlens,
            qkv_dtype=self.kv_cache_dtype,
            cu_seqlens_q=cu_seqlens_q,
            page_size=self.page_size,
            causal=True,
            has_softcap=self.has_softcap,
            num_splits=self.num_splits,
        )
```
**EN:** Implements the compute scheduler metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 compute scheduler metadata 例程。

### Lines 272-613: method FlashAttentionBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        """Initialize forward metadata hence all layers in the forward pass can reuse it."""
        metadata = FlashAttentionMetadata()
        seqlens_in_batch = forward_batch.seq_lens
        batch_size = forward_batch.batch_size
        device = seqlens_in_batch.device

        if forward_batch.forward_mode.is_decode_or_idle():
            # Draft Decode
            if forward_batch.spec_info is not None:
                if self.topk <= 1:
                    metadata.cache_seqlens_int32 = (
                        seqlens_in_batch + (self.speculative_step_id + 1)
                    ).to(torch.int32)
                    metadata.max_seq_len_k = forward_batch.seq_lens_cpu.max().item() + (
                        self.speculative_step_id + 1
                    )
                    metadata.cu_seqlens_q = torch.arange(
                        0, batch_size + 1, dtype=torch.int32, device=device
                    )
                    metadata.cu_seqlens_k = torch.nn.functional.pad(
                        torch.cumsum(
                            metadata.cache_seqlens_int32, dim=0, dtype=torch.int32
                        ),
                        (1, 0),
                    )
                    metadata.page_table = forward_batch.req_to_token_pool.req_to_token[
                        forward_batch.req_pool_indices, : metadata.max_seq_len_k
                    ]
                else:
                    metadata.cache_seqlens_int32 = (seqlens_in_batch).to(torch.int32)
                    metadata.max_seq_len_q = self.topk
# ... omitted 298 lines ...
                draft_decode_set_expand_metadata(
                    cache_seqlens_int32=self.forward_metadata_spec_decode_expand.cache_seqlens_int32,
                    page_table=expand_page_table,
                    last_page_lens=last_page_lens,
                    decode_length=decode_length,
                    cache_loc=cache_loc,
                    topk=self.topk,
                    page_size=self.page_size,
                )
                self.forward_metadata_spec_decode_expand.page_table = expand_page_table

        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 615-1029: method FlashAttentionBackend.forward_extend
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
        sinks: Optional[torch.Tensor] = None,
    ):
        if k is not None:
            assert v is not None

            is_cp_mode = (
                forward_batch.forward_mode.is_context_parallel_extend()
                and forward_batch.attn_cp_metadata is not None
                and self.attn_cp_size > 1
            )

            if save_kv_cache and not is_cp_mode and not self.fa_skip_kv_cache:
                cache_loc = (
                    forward_batch.out_cache_loc
                    if not layer.is_cross_attention
                    else forward_batch.encoder_out_cache_loc
                )
                if not self.use_mla:
                    forward_batch.token_to_kv_pool.set_kv_buffer(
                        layer, cache_loc, k, v, layer.k_scale, layer.v_scale
                    )
# ... omitted 371 lines ...
                        )
                    )
                    o, _ = merge_state_v2_wrapper(
                        o,
                        softmax_lse.T.contiguous(),
                        o_expand,
                        softmax_lse_expand.T.contiguous(),
                    )
                else:
                    o = result

        return o.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1031-1327: method FlashAttentionBackend.forward_decode
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
        sinks: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if k is not None:
            assert v is not None
            if save_kv_cache:
                cache_loc = (
                    forward_batch.out_cache_loc
                    if not layer.is_cross_attention
                    else forward_batch.encoder_out_cache_loc
                )
                if not self.use_mla:
                    forward_batch.token_to_kv_pool.set_kv_buffer(
                        layer, cache_loc, k, v, layer.k_scale, layer.v_scale
                    )
                else:
                    forward_batch.token_to_kv_pool.set_mla_kv_buffer(
                        layer,
                        cache_loc,
                        k,
                        k_rope,
                    )
# ... omitted 253 lines ...
                    ver=self.fa_impl_ver,
                )
                o, _ = merge_state_v2(
                    o,
                    softmax_lse.T.contiguous(),
                    o_expand,
                    softmax_lse_expand.T.contiguous(),
                )
            else:
                o = result

        return o.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1329-1624: method FlashAttentionBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        """Initialize CUDA graph state for the attention backend.

        Args:
            max_bs (int): Maximum batch size to support in CUDA graphs

        This creates fixed-size tensors that will be reused during CUDA graph replay
        to avoid memory allocations.
        """
        max_num_pages = (self.max_context_len + self.page_size - 1) // self.page_size

        # This is being used by normal decode and draft decode when topk == 1
        self.decode_cuda_graph_metadata = {
            "cache_seqlens": torch.zeros(max_bs, dtype=torch.int32, device=self.device),
            "cu_seqlens_q": torch.arange(
                0, max_bs + 1, dtype=torch.int32, device=self.device
            ),
            "cu_seqlens_k": torch.zeros(
                max_bs + 1, dtype=torch.int32, device=self.device
            ),
            "page_table": torch.zeros(
                max_bs,
                max_num_pages,
                dtype=torch.int32,
                device=self.device,
            ),
            "strided_indices": torch.arange(
                0, self.max_context_len, self.page_size, device=self.device
            ),
        }
        # Pre-allocate scheduler_metadata buffer for CUDA graph
        # Size: 1 (semaphore) + round_up(max_bs, 4) * 4 (causal decode vectors)
# ... omitted 252 lines ...
                    device=self.device,
                ),
                "encoder_lens_int32": torch.zeros(
                    max_bs, dtype=torch.int32, device=self.device
                ),
                "encoder_cu_seqlens_k": torch.zeros(
                    max_bs + 1, dtype=torch.int32, device=self.device
                ),
            }
        else:
            # For decoder-only models, skip encoder_metadata allocation
            self.encoder_metadata = {}
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 1626-1890: method FlashAttentionBackend.init_forward_metadata_capture_cuda_graph
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
        """Initialize forward metadata for capturing CUDA graph."""
        metadata = FlashAttentionMetadata()

        # metadata_expand is needed for Spec Decoding when top k > 1
        metadata_expand = FlashAttentionMetadata()

        device = seq_lens.device
        if forward_mode.is_decode_or_idle():
            if spec_info is not None:
                # Draft Decode
                if self.topk <= 1:
                    # When topk = 1, we use the normal decode metadata
                    metadata.cache_seqlens_int32 = self.decode_cuda_graph_metadata[
                        "cache_seqlens"
                    ][:bs]
                    metadata.max_seq_len_k = seq_lens.max().item() + (
                        self.speculative_step_id + 1
                    )
                    metadata.cu_seqlens_q = self.decode_cuda_graph_metadata[
                        "cu_seqlens_q"
                    ][: bs + 1]
                    metadata.cu_seqlens_k = torch.nn.functional.pad(
# ... omitted 221 lines ...
                :encoder_bs
            ]
            metadata.encoder_cu_seqlens_k = self.encoder_metadata[
                "encoder_cu_seqlens_k"
            ][: (encoder_bs + 1)]

            metadata.encoder_page_table = self.encoder_metadata["encoder_page_table"][
                :bs, :
            ]

        self.forward_metadata = metadata
        self.forward_metadata_spec_decode_expand = metadata_expand
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1892-2268: method FlashAttentionBackend.init_forward_metadata_replay_cuda_graph
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
    ):
        """Initialize forward metadata for replaying CUDA graph."""
        seq_lens = seq_lens[:bs]
        seq_lens_cpu = seq_lens_cpu[:bs]
        req_pool_indices = req_pool_indices[:bs]
        device = seq_lens.device
        metadata = None
        metadata_expand = None

        if forward_mode.is_decode_or_idle():

            if spec_info is not None:
                # Draft Decode
                if self.topk <= 1:
                    # When topk = 1, we use the normal decode metadata
                    metadata = self.decode_cuda_graph_metadata[bs]
                    max_len = seq_lens_cpu.max().item()
                    metadata.max_seq_len_k = max_len + self.speculative_step_id + 1
                    max_seq_pages = (
                        metadata.max_seq_len_k + self.page_size - 1
                    ) // self.page_size
# ... omitted 333 lines ...

            # Update the regular page table
            page_table = self.req_to_token[
                req_pool_indices,
                metadata.encoder_max_seq_len_k : (
                    metadata.encoder_max_seq_len_k + metadata.max_seq_len_k
                ),
            ]
            metadata.page_table[:, : metadata.max_seq_len_k].copy_(page_table)

        self.forward_metadata = metadata
        self.forward_metadata_spec_decode_expand = metadata_expand
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2270-2272: method FlashAttentionBackend.get_cuda_graph_seq_len_fill_value
```python
    def get_cuda_graph_seq_len_fill_value(self):
        """Get the fill value for sequence length in CUDA graph."""
        return 1
```
**EN:** Computes and returns get cuda graph seq len fill value from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cuda graph seq len fill value。

### Lines 2274-2316: method FlashAttentionBackend._maybe_init_local_attn_metadata
```python
    def _maybe_init_local_attn_metadata(
        self, forwardbatch: ForwardBatch, metadata: FlashAttentionMetadata, device
    ):
        """Centralized utility to initialize local_attn_metadata if chunked attention is enabled."""
        if not self.has_local_attention:
            metadata.local_attn_metadata = None
            return

        cu_seqlens_q = metadata.cu_seqlens_q
        cache_seqlens_int32 = metadata.cache_seqlens_int32
        if self.use_sliding_window_kv_pool:
            page_table = self.token_to_kv_pool.translate_loc_from_full_to_swa(
                metadata.page_table
            )
        else:
            page_table = metadata.page_table
        if cu_seqlens_q is None or cache_seqlens_int32 is None or page_table is None:
            metadata.local_attn_metadata = None
            return

        cu_seqlens_q_np = cu_seqlens_q.cpu().numpy()
        seq_lens_np = cache_seqlens_int32.cpu().numpy()
        (
            seqlens_q_local_np,
            cu_seqlens_q_local_np,
            seqlens_k_local_np,
            block_table_local,
        ) = make_local_attention_virtual_batches(
            self.attention_chunk_size,
            cu_seqlens_q_np,
            seq_lens_np,
            page_table,
            self.page_size,
        )

        local_metadata = FlashAttentionMetadata.LocalAttentionMetadata(
            local_query_start_loc=torch.from_numpy(cu_seqlens_q_local_np).to(device),
            local_seqused_k=torch.from_numpy(seqlens_k_local_np).to(device),
            local_block_table=block_table_local.to(device),
            local_max_query_len=int(seqlens_q_local_np.max()),
            local_max_seq_len=int(seqlens_k_local_np.max()),
        )
        metadata.local_attn_metadata = local_metadata
```
**EN:** Implements the maybe init local attn metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 maybe init local attn metadata 例程。

### Lines 2318-2375: method FlashAttentionBackend._maybe_update_local_attn_metadata_for_capture
```python
    def _maybe_update_local_attn_metadata_for_capture(
        self, metadata: FlashAttentionMetadata, bs: int
    ):
        """Update local attention metadata during CUDA graph capture phase.

        This method calculates the exact buffer sizes needed for local attention metadata
        during the CUDA graph capture phase, optimizing memory usage by creating views of
        pre-allocated buffers with exactly the sizes needed.
        """
        if not self.has_local_attention:
            return

        seq_lens_capture = metadata.cache_seqlens_int32
        max_seq_len = int(seq_lens_capture.max().item())
        page_table_capture = metadata.page_table

        cu_seqlens_q_np = metadata.cu_seqlens_q.cpu().numpy()
        seqlens_np = seq_lens_capture.cpu().numpy()
        (
            seqlens_q_local_np,
            cu_seqlens_q_local_np,
            seqlens_k_local_np,
            block_table_local_np,
        ) = make_local_attention_virtual_batches(
            self.attention_chunk_size,
            cu_seqlens_q_np,
            seqlens_np,
            page_table_capture,
            self.page_size,
        )

        # Get exact dimensions from the calculation
# ... omitted 14 lines ...

        local_block_table = self.decode_cuda_graph_local_attn_metadata[
            "local_block_table"
        ][:b0, :b1]

        metadata.local_attn_metadata = FlashAttentionMetadata.LocalAttentionMetadata(
            local_query_start_loc=local_query_start_loc,
            local_seqused_k=local_seqused_k,
            local_block_table=local_block_table,
            local_max_query_len=1,
            local_max_seq_len=max_seq_len,
        )
```
**EN:** Implements the maybe update local attn metadata for capture routine used by this attention module.
**CN:** 实现该注意力模块使用的 maybe update local attn metadata for capture 例程。

### Lines 2377-2455: method FlashAttentionBackend._maybe_update_local_attn_metadata_for_replay
```python
    def _maybe_update_local_attn_metadata_for_replay(
        self,
        metadata: FlashAttentionMetadata,
        bs: int,
    ):
        """Update preallocated local attention metadata in-place before CUDA graph replay."""
        if not self.has_local_attention:
            return

        # Access preallocated buffers
        local_q_buf = self.decode_cuda_graph_local_attn_metadata[
            "local_query_start_loc"
        ]
        local_k_buf = self.decode_cuda_graph_local_attn_metadata["local_seqused_k"]
        local_block_buf = self.decode_cuda_graph_local_attn_metadata[
            "local_block_table"
        ]
        cu_seqlens_q = self.decode_cuda_graph_metadata["cu_seqlens_q"]

        # Create a modified version for local attention that only processes the last token
        # This mimics the normal decode pattern
        cu_seqlens_q = torch.arange(
            bs + 1, device=cu_seqlens_q.device, dtype=cu_seqlens_q.dtype
        )
        seqlens = metadata.cache_seqlens_int32[:bs]
        # Slice the page_table to match the batch size and actual sequence length
        # This serves three important purposes:
        # 1. Ensures we only process the actual batch size (bs) and not the maximum batch size
        # 2. Limits the sequence length to prevent processing padding tokens or garbage values
        # 3. Prevents zeros in the block table which can cause garbage output during replay
        #
        # Without this slicing, the pre-allocated page_table may contain zeros or invalid indices
# ... omitted 35 lines ...
        local_q_buf[:q_len].copy_(cu_seqlens_q_local)
        local_q_buf[q_len:].fill_(0)
        local_k_buf[:k_len].copy_(seqlens_k_local)
        local_k_buf[k_len:].fill_(0)
        local_block_buf[:b0, :b1].copy_(block_table_local)
        local_block_buf[b0:, :].fill_(0)
        local_block_buf[:b0, b1:].fill_(0)

        if metadata.local_attn_metadata is not None:
            lam = metadata.local_attn_metadata
            lam.local_max_query_len = int(seqlens_q_local_np.max())
            lam.local_max_seq_len = int(seqlens_k_local_np.max())
```
**EN:** Implements the maybe update local attn metadata for replay routine used by this attention module.
**CN:** 实现该注意力模块使用的 maybe update local attn metadata for replay 例程。

### Lines 2457-2516: method FlashAttentionBackend._init_sliding_window_attn_spec_metadata
```python
    def _init_sliding_window_attn_spec_metadata(
        self,
        metadata: FlashAttentionMetadata,
        metadata_expand: FlashAttentionMetadata,
        metadata_swa: Optional[FlashAttentionMetadata] = None,
    ):
        # TODO: support page_size > 1 for swa spec
        assert (
            self.page_size == 1
        ), "FlashAttention backend doesn't support topk > 1 speculative decoding with page size > 1 sliding window attention"

        cache_seqlens_int32 = (
            metadata.cache_seqlens_int32.repeat_interleave(
                self.speculative_num_draft_tokens
            )
            + metadata_expand.cache_seqlens_int32
        )
        cu_seqlens_k = torch.nn.functional.pad(
            torch.cumsum(cache_seqlens_int32, dim=0, dtype=torch.int32), (1, 0)
        )
        bs = cache_seqlens_int32.shape[0]
        page_table = (
            metadata.page_table.new_zeros(
                (bs, metadata.max_seq_len_k + metadata_expand.page_table.shape[1])
            )
            if metadata_swa is None
            else metadata_swa.page_table
        )

        page_table_a = metadata.page_table
        page_table_b = metadata_expand.page_table
        if self.use_sliding_window_kv_pool:
# ... omitted 16 lines ...
        if metadata_swa is None:
            metadata_swa = FlashAttentionMetadata()
            metadata_swa.max_seq_len_q = 1
            metadata_swa.cu_seqlens_q = metadata_expand.cu_seqlens_q
            metadata_swa.cache_seqlens_int32 = cache_seqlens_int32
            metadata_swa.cu_seqlens_k = cu_seqlens_k
            metadata_swa.page_table = page_table
        else:
            metadata_swa.cache_seqlens_int32.copy_(cache_seqlens_int32)
            metadata_swa.cu_seqlens_k.copy_(cu_seqlens_k)

        metadata.swa_spec_metadata = metadata_swa
```
**EN:** Implements the init sliding window attn spec metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 init sliding window attn spec metadata 例程。

### Lines 2519-2578: function _prepare_swa_spec_page_table_kernel
```python
@triton.jit
def _prepare_swa_spec_page_table_kernel(
    dst_ptr,
    src_a_ptr,
    src_b_ptr,
    seq_len_a_ptr,
    seq_len_b_ptr,
    dst_stride_m,
    dst_stride_n,
    a_stride_m,
    a_stride_n,
    b_stride_m,
    b_stride_n,
    LEN_A: tl.constexpr,
    LEN_B: tl.constexpr,
    REPEAT_STEP: tl.constexpr,
    BLOCK_N: tl.constexpr,
):
    pid_m = tl.program_id(0)
    pid_n = tl.program_id(1)

    idx_a = pid_m // REPEAT_STEP
    idx_b = pid_m
    seq_len_a = tl.load(seq_len_a_ptr + idx_a)
    seq_len_b = tl.load(seq_len_b_ptr + idx_b)

    offs_n = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    total_len = seq_len_a + seq_len_b

    if pid_n * BLOCK_N >= total_len:
        return

# ... omitted 16 lines ...
        a_offs = offs_n
        a_mask = (a_offs < seq_len_a) & (a_offs < LEN_A)
        a_ptr = src_a_ptr + idx_a * a_stride_m + a_offs * a_stride_n
        a_val = tl.load(a_ptr, mask=a_mask, other=0)

        b_offs = offs_n - seq_len_a
        b_mask = (b_offs >= 0) & (b_offs < seq_len_b) & (b_offs < LEN_B)
        b_ptr = src_b_ptr + idx_b * b_stride_m + b_offs * b_stride_n
        b_val = tl.load(b_ptr, mask=b_mask, other=0)

        result = tl.where(offs_n < seq_len_a, a_val, b_val)
        tl.store(dst, result, mask=mask)
```
**EN:** Implements the prepare swa spec page table kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 prepare swa spec page table kernel 例程。

### Lines 2581-2618: function prepare_swa_spec_page_table_triton
```python
def prepare_swa_spec_page_table_triton(
    page_table_dst: torch.Tensor,
    page_table_a: torch.Tensor,
    page_table_b: torch.Tensor,  # expand page table
    seq_len_a: torch.Tensor,
    seq_len_b: torch.Tensor,  # expand seq lens
    speculative_num_draft_tokens: int,
):
    # concat page_table and expand page_table by kv seq length
    bs = seq_len_a.numel()
    bs_expand = seq_len_b.numel()
    assert bs_expand == bs * speculative_num_draft_tokens

    LEN_A = page_table_a.shape[1]
    LEN_B = page_table_b.shape[1]
    LEN_OUT = LEN_A + LEN_B
    REPEAT_STEP = speculative_num_draft_tokens
    BLOCK_N = 256

    grid = (bs_expand, triton.cdiv(LEN_OUT, BLOCK_N))
    _prepare_swa_spec_page_table_kernel[grid](
        page_table_dst,
        page_table_a,
        page_table_b,
        seq_len_a,
        seq_len_b,
        page_table_dst.stride(0),
        page_table_dst.stride(1),
        page_table_a.stride(0),
        page_table_a.stride(1),
        page_table_b.stride(0),
        page_table_b.stride(1),
        LEN_A=LEN_A,
        LEN_B=LEN_B,
        REPEAT_STEP=REPEAT_STEP,
        BLOCK_N=BLOCK_N,
        num_warps=4,
    )
```
**EN:** Prepares prepare swa spec page table triton so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 prepare swa spec page table triton，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 2621-2622: class FlashAttentionMultiStepBackend
```python
class FlashAttentionMultiStepBackend:
```
**EN:** Concrete attention backend that connects flash attention multi step backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 flash attention multi step backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 2623-2643: method FlashAttentionMultiStepBackend.__init__
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        topk: int,
        speculative_num_steps: int,
        fa_impl_ver: int = 3,
    ):
        self.model_runner = model_runner
        self.topk = topk
        self.speculative_num_steps = speculative_num_steps
        self.attn_backends = []
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends.append(
                FlashAttentionBackend(
                    model_runner,
                    speculative_step_id=i,
                    topk=self.topk,
                    speculative_num_steps=self.speculative_num_steps,
                    fa_impl_ver=fa_impl_ver,
                )
            )
```
**EN:** Initializes the FlashAttentionMultiStepBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 FlashAttentionMultiStepBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 2645-2647: method FlashAttentionMultiStepBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_forward_metadata(forward_batch)
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2649-2651: method FlashAttentionMultiStepBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_cuda_graph_state(max_bs, max_num_tokens)
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 2653-2669: method FlashAttentionMultiStepBackend.init_forward_metadata_capture_cuda_graph
```python
    def init_forward_metadata_capture_cuda_graph(
        self,
        forward_batch: ForwardBatch,
    ):
        assert forward_batch.spec_info is not None
        assert forward_batch.spec_info.is_draft_input()

        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_forward_metadata_capture_cuda_graph(
                forward_batch.batch_size,
                forward_batch.batch_size * self.topk,
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                encoder_lens=forward_batch.encoder_lens,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
            )
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2671-2690: method FlashAttentionMultiStepBackend.init_forward_metadata_replay_cuda_graph
```python
    def init_forward_metadata_replay_cuda_graph(
        self, forward_batch: ForwardBatch, bs: int
    ):
        assert forward_batch.spec_info is not None
        assert forward_batch.spec_info.is_draft_input()

        for i in range(self.speculative_num_steps - 1):
            # TODO: incrementally update the metadata for the later steps,
            # so that they do not need to recompute everything from scratch.
            self.attn_backends[i].init_forward_metadata_replay_cuda_graph(
                bs,
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                forward_batch.seq_lens_sum,
                encoder_lens=forward_batch.encoder_lens,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
                seq_lens_cpu=forward_batch.seq_lens_cpu,
                out_cache_loc=forward_batch.out_cache_loc,
            )
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 2693-2788: function _fused_metadata_kernel_general
```python
@triton.jit
def _fused_metadata_kernel_general(
    # Input tensors
    seq_lens,
    seq_lens_stride_0,
    req_to_token,
    req_to_token_stride_0,
    req_to_token_stride_1,
    req_pool_indices,
    req_pool_indices_stride_0,
    # Output buffers
    cache_seqlens_int32,
    cache_seqlens_int32_stride_0,
    cu_seqlens_k,
    cu_seqlens_k_stride_0,
    page_table,
    page_table_stride_0,
    page_table_stride_1,
    swa_page_table,
    swa_page_table_stride_0,
    swa_page_table_stride_1,
    full_to_swa_mapping,
    full_to_swa_mapping_stride_0,
    # Scalar parameters
    B,
    max_seq_pages,
    page_size: tl.constexpr,
    seq_len_delta: tl.constexpr,
    use_swa: tl.constexpr,
    SHIFT: tl.constexpr,
    BLOCK_COLS: tl.constexpr,
):
# ... omitted 52 lines ...
            mask=mask,
            other=0,
            cache_modifier=".cg",
        )
        if page_size == 1:
            swa_val = swa_slot
        else:
            swa_val = swa_slot >> SHIFT
        swa_offsets = (
            i * swa_page_table_stride_0 + col_offsets * swa_page_table_stride_1
        )
        tl.store(swa_page_table + swa_offsets, swa_val, mask=mask, cache_modifier=".cg")
```
**EN:** Implements the fused metadata kernel general routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused metadata kernel general 例程。

### Lines 2791-2850: function _fused_metadata_kernel_ps1_no_swa
```python
@triton.jit
def _fused_metadata_kernel_ps1_no_swa(
    # Input tensors
    seq_lens,
    seq_lens_stride_0,
    req_to_token,
    req_to_token_stride_0,
    req_to_token_stride_1,
    req_pool_indices,
    req_pool_indices_stride_0,
    # Output buffers
    cache_seqlens_int32,
    cache_seqlens_int32_stride_0,
    cu_seqlens_k,
    cu_seqlens_k_stride_0,
    page_table,
    page_table_stride_0,
    page_table_stride_1,
    # Scalar parameters
    B,
    max_seq_pages,
    seq_len_delta: tl.constexpr,
    BLOCK_COLS: tl.constexpr,
):
    pid_b = tl.program_id(0)  # batch index
    pid_c = tl.program_id(1)  # column chunk index

    # 1. Prefix sum (only one block does it)
    if pid_b == 0 and pid_c == 0:
        acc = 0
        for idx in range(B):
            seq = tl.load(seq_lens + idx * seq_lens_stride_0)
# ... omitted 16 lines ...
    col_offsets = col_start + tl.arange(0, BLOCK_COLS)
    mask = col_offsets < max_seq_pages

    # page_size = 1: col_idx = col_offsets
    rt_offsets = row_offset + col_offsets * req_to_token_stride_1
    page_index = tl.load(
        req_to_token + rt_offsets, mask=mask, other=0, cache_modifier=".cg"
    )

    # page_table = page_index // 1 = page_index
    pt_offsets = i * page_table_stride_0 + col_offsets * page_table_stride_1
    tl.store(page_table + pt_offsets, page_index, mask=mask, cache_modifier=".cg")
```
**EN:** Implements the fused metadata kernel ps1 no swa routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused metadata kernel ps1 no swa 例程。

### Lines 2854-2994: function normal_decode_set_metadata
```python
def normal_decode_set_metadata(
    cache_seqlens_int32: torch.Tensor,
    cu_seqlens_k: torch.Tensor,
    page_table: torch.Tensor,
    req_to_token: torch.Tensor,
    req_pool_indices: torch.Tensor,
    strided_indices: torch.Tensor,
    max_seq_pages: torch.Tensor,
    seq_lens: torch.Tensor,
    seq_len_delta: int,
    page_size: int,
    swa_page_table: Optional[torch.Tensor] = None,
    token_to_kv_pool: Optional[SWAKVPool] = None,
):
    """
    Fused Triton implementation that replaces 4-5 sequential CUDA kernels with 1-2 kernels:
      1. cache_seqlens = seq_lens + seq_len_delta (int64→int32 cast)
      2. cu_seqlens_k = cumsum(cache_seqlens) (prefix-sum)
      3. page_indices = req_to_token[pool_idx, stride_idx] (2-D gather)
      4. page_table = page_indices // page_size (floor-divide)
      5. (optional) swa_page_table for sliding window attention

    Achieves ~5.2x speedup on H200 hardware for typical decode workloads.
    """
    assert (
        page_size > 0 and (page_size & (page_size - 1)) == 0
    ), f"page_size must be a power of two, got {page_size}"

    batch_size = cache_seqlens_int32.shape[0]
    device = seq_lens.device

    # Ensure contiguous memory layout for efficient Triton access
# ... omitted 97 lines ...
            full_to_swa_mapping,
            full_to_swa_mapping_stride_0,
            batch_size,
            max_seq_pages,
            page_size,
            seq_len_delta,
            use_swa,
            shift,
            BLOCK_COLS=BLOCK_COLS,
            num_warps=4,
            num_stages=3,
        )
```
**EN:** Implements the normal decode set metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 normal decode set metadata 例程。

### Lines 2997-3017: function draft_decode_set_expand_metadata
```python
@torch.compile(dynamic=True, backend=get_compiler_backend())
def draft_decode_set_expand_metadata(
    cache_seqlens_int32: torch.Tensor,  # Modifies
    page_table: torch.Tensor,  # Modifies
    last_page_lens: torch.Tensor,
    decode_length: int,
    cache_loc: torch.Tensor,
    topk: int,
    page_size: int,
):
    expanded_last_page_lens = last_page_lens.repeat_interleave(topk)
    cache_seqlens_int32.copy_(decode_length + expanded_last_page_lens)
    cache_loc = (cache_loc // page_size).to(torch.int32)
    if cache_loc.dim() == 1:
        cache_loc = cache_loc.unsqueeze(0)
    # Vectorized torch.unique_consecutive: track value change points then scatter
    mask = torch.ones_like(cache_loc, dtype=torch.bool)
    mask[:, 1:] = cache_loc[:, 1:] != cache_loc[:, :-1]
    positions = mask.cumsum(dim=1) - 1
    num_seqs = cache_loc.shape[0]
    page_table[:num_seqs, :].scatter_(1, positions, cache_loc)
```
**EN:** Implements the draft decode set expand metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 draft decode set expand metadata 例程。

### Lines 3074-3223: function make_local_attention_virtual_batches
```python
def make_local_attention_virtual_batches(
    attn_chunk_size: int,
    query_start_loc_np: np.ndarray,
    seq_lens_np: np.ndarray,
    block_table: torch.Tensor,
    page_size: int = 0,
) -> tuple[np.ndarray, np.ndarray, np.ndarray, torch.Tensor]:
    """
    Take in `query_start_loc_np` and `seq_lens_np` and break the sequences into
    local attention blocks, where each block is passed to the attention kernel
    as an independent local ("virtual") batch item.

    Args:
        attn_chunk_size: Size of local attention chunks
        query_start_loc_np: Cumulative sum of query lengths (numpy array)
        seq_lens_np: Sequence lengths (numpy array)
        block_table: Block table for KV cache
        page_size: Size of each page in the KV cache

    Returns:
        seqlens_q_local: Query sequence lengths for local attention
        cu_seqlens_q_local: Cumulative sum of query sequence lengths for local attention
        seqlens_k_local: Key sequence lengths for local attention
        block_table_local: Block table for local attention
    """
    # Adjust attention_chunk_size based on the actual sequence length
    # to avoid index out of bounds errors
    max_seq_len = seq_lens_np.max()
    effective_chunk_size = min(attn_chunk_size, max_seq_len)
    # Make sure effective_chunk_size is divisible by page_size
    effective_chunk_size = (effective_chunk_size // page_size) * page_size
    if effective_chunk_size < page_size:
# ... omitted 106 lines ...

    # NOTE: https://github.com/pytorch/pytorch/pull/160256 causes performance
    # regression when using numpy arrays (batch and block indices) to index into
    # torch tensor (block_table). As a workaround, convert numpy arrays to torch
    # tensor first, which recovers perf.
    batch_indices_torch = torch.from_numpy(batch_indices)
    block_indices_torch = torch.from_numpy(block_indices)
    block_table_local = block_table[batch_indices_torch, block_indices_torch].view(
        virtual_batches, -1
    )

    return seqlens_q_local, cu_seqlens_q_local, seqlens_k_local, block_table_local
```
**EN:** Factory helper that constructs make local attention virtual batches and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 make local attention virtual batches，并校验此执行路径所需的前置条件。

### Lines 3226-3228: function cdiv
```python
def cdiv(a: int, b: int) -> int:
    """Ceiling division."""
    return -(a // -b)
```
**EN:** Implements the cdiv routine used by this attention module.
**CN:** 实现该注意力模块使用的 cdiv 例程。

### Lines 3232-3234: function merge_state_v2_wrapper
```python
@torch._dynamo.disable()
def merge_state_v2_wrapper(o, s_a, o_exp, s_b):
    return merge_state_v2(o, s_a, o_exp, s_b)
```
**EN:** Implements the merge state v2 wrapper routine used by this attention module.
**CN:** 实现该注意力模块使用的 merge state v2 wrapper 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Flash-style fused attention kernels / **CN:** Flash 风格融合注意力内核
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调

## Dependencies / 依赖关系
- `__future__.annotations`
- `dataclasses.dataclass`
- `typing.TYPE_CHECKING`
- `typing.Optional`
- `numpy`
- `torch`
- `triton`
- `triton.language`
- `sglang.srt.configs.model_config.AttentionArch`
- `sglang.srt.layers.attention.base_attn_backend.AttentionBackend`
- `sglang.srt.layers.radix_attention.AttentionType`
- `sglang.srt.layers.utils.cp_utils.cp_allgather_and_save_kv_cache`
- `sglang.srt.layers.utils.cp_utils.cp_attn_forward_extend`
- `sglang.srt.mem_cache.swa_memory_pool.SWAKVPool`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.speculative.spec_info.SpecInput`
- `sglang.srt.utils.get_compiler_backend`
- `sgl_kernel.merge_state_v2`
- `sglang.jit_kernel.flash_attention.flash_attn_varlen_func`
- `sglang.jit_kernel.flash_attention.flash_attn_with_kvcache`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.model_runner.ModelRunner`
- `sgl_kernel.flash_attn.flash_attn_varlen_func`
- `sgl_kernel.flash_attn.flash_attn_with_kvcache`
- `sgl_kernel.flash_attn.get_scheduler_metadata`
- `sglang.jit_kernel.flash_attention_v4.flash_attn_varlen_func`
- `sglang.jit_kernel.flash_attention_v4.flash_attn_with_kvcache`
