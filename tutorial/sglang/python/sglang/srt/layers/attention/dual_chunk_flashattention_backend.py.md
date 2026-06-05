# dual_chunk_flashattention_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/dual_chunk_flashattention_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the dual chunk flashattention backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 dual chunk flashattention backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 2-2: docstring
```python
"""Attention layer with Dual chunk flash attention and sparse attention."""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 4-25: imports
```python
import functools
import logging
import math
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Tuple

import torch
import torch.nn.functional as F
from sgl_kernel.sparse_flash_attn import (
    convert_vertical_slash_indexes,
    convert_vertical_slash_indexes_mergehead,
    sparse_attn_func,
)

from sglang.jit_kernel.flash_attention import (
    flash_attn_varlen_func,
    flash_attn_with_kvcache,
)
from sglang.srt.distributed.parallel_state import get_tensor_model_parallel_rank
from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.layers.attention.flashattention_backend import FlashAttentionMetadata
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 27-29: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 32-32: module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 35-101: class DualChunkFlashAttentionMetadata
```python
@dataclass
class DualChunkFlashAttentionMetadata:
    """Metadata for FlashAttentionBackend.

    NOTE: Any python object stored here is not updated when it is
    cuda-graph replayed. If you have values that need to be changed
    dynamically, it should be stored in tensor. The tensor has to be
    updated from `CUDAGraphRunner.forward` API.
    """

    # (batch_size,). The sequence length per sequence. Sequence length means
    # the computed tokens + new tokens None if it is a decoding.
    seq_lens: Optional[List[int]] = None
    # seq_lens stored as a tensor.
    seq_lens_tensor: Optional[torch.Tensor] = None
    # Maximum sequence length among prefill batch. 0 if there are decoding
    # requests only.
    max_seq_len: int = None

    # (batch_size,). The orig sequence length per sequence.
    orig_seq_lens: Optional[List[int]] = None

    # orig_seq_lens stored as a tensor.
    orig_seq_lens_tensor: Optional[torch.Tensor] = None

    # Block addresses per sequence. (Seq id -> list of physical block)
    # E.g., [0, 1, 2] means tokens are stored in 0th, 1st, and 2nd blocks
    # in the kv cache. Each block can contain up to block_size tokens.
    # 2nd dimensions are padded up to max_blocks_per_seq if it is cuda-graph
    # captured.
    block_tables: Optional[torch.Tensor] = None

# ... omitted 23 lines ...

    # Max sequence length for succ attention.
    max_seq_len_succ: Optional[int] = None

    # (batch_size, num_blocks). Block table for succ attention.
    block_tables_succ: Optional[torch.Tensor] = None

    # (batch_size,). Sequence lengths for inter attention.
    seq_lens_inter: Optional[torch.Tensor] = None

    # Max sequence length for inter attention.
    max_seq_len_inter: Optional[int] = None
```
**EN:** Dataclass-style container that stores structured runtime state for dual chunk flash attention metadata.
**CN:** 该数据类风格的容器用于存储 dual chunk flash attention metadata 的结构化运行时状态。

### Lines 104-104: class DualChunkFlashAttentionBackend
```python
class DualChunkFlashAttentionBackend(AttentionBackend):
```
**EN:** Concrete attention backend that connects dual chunk flash attention backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 dual chunk flash attention backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 105-160: method DualChunkFlashAttentionBackend.__init__
```python
    def __init__(
        self,
        model_runner: "ModelRunner",
    ) -> None:
        self.forward_metadata: FlashAttentionMetadata = None
        self.device = model_runner.device
        self.max_context_len = model_runner.model_config.context_len
        self.num_heads = model_runner.model_config.get_num_attention_heads(
            model_runner.server_args.tp_size
        )
        self.num_kv_heads = model_runner.model_config.get_num_kv_heads(
            model_runner.server_args.tp_size
        )
        self.head_size = model_runner.model_config.head_dim

        self.req_to_token = model_runner.req_to_token_pool.req_to_token
        self.kv_cache_dtype = model_runner.kv_cache_dtype
        self.kv_cache_dtype_str = model_runner.server_args.kv_cache_dtype
        self.page_size = model_runner.page_size

        assert self.num_heads % self.num_kv_heads == 0
        self.num_queries_per_kv = self.num_heads // self.num_kv_heads

        dual_chunk_attention_config = getattr(
            model_runner.model_config.hf_config, "dual_chunk_attention_config", None
        )
        assert dual_chunk_attention_config is not None
        self.chunk_size = dual_chunk_attention_config.get("chunk_size", 8192)
        self.local_size = dual_chunk_attention_config.get("local_size", 1024)
        self.original_max_position_embeddings = dual_chunk_attention_config.get(
            "original_max_position_embeddings", 0
        )
# ... omitted 12 lines ...
            "sparse_attention_threshold", 32768
        )
        self.sparse_attention_last_q = dual_chunk_attention_config.get(
            "sparse_attention_last_q", 64
        )
        self.dual_chunk_attention_config = dual_chunk_attention_config

        if self.sparse_attention_enabled:
            self.arange = torch.arange(self.sparse_attention_last_q, device="cuda")
            self.last_q_mask = (
                self.arange[None, None, :, None] >= self.arange[None, None, None, :]
            )
```
**EN:** Initializes the DualChunkFlashAttentionBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 DualChunkFlashAttentionBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 162-169: method DualChunkFlashAttentionBackend.get_sparse_attention_config
```python
    @functools.lru_cache()
    def get_sparse_attention_config(self, layer_idx) -> List[Dict[str, Any]]:
        layer_sparse_attention_config = {
            int(i): j for i, j in self.sparse_attention_config[layer_idx].items()
        }
        start_head = self.num_heads * get_tensor_model_parallel_rank()
        end_head = start_head + self.num_heads
        return [layer_sparse_attention_config[i] for i in range(start_head, end_head)]
```
**EN:** Computes and returns get sparse attention config from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get sparse attention config。

### Lines 171-297: method DualChunkFlashAttentionBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        """Initialize forward metadata hence all layers in the forward pass can reuse it."""

        forward_mode: ForwardMode = forward_batch.forward_mode
        assert forward_mode.is_prefill() or forward_mode.is_decode()
        batch_size = forward_batch.batch_size

        metadata = DualChunkFlashAttentionMetadata()
        metadata.seq_lens_tensor = forward_batch.seq_lens.to(torch.int32)
        metadata.seq_lens = forward_batch.seq_lens.tolist()
        metadata.max_seq_len = forward_batch.seq_lens.max().item()

        metadata.orig_seq_lens_tensor = forward_batch.orig_seq_lens
        metadata.orig_seq_lens = forward_batch.orig_seq_lens.tolist()

        metadata.block_tables = forward_batch.req_to_token_pool.req_to_token[
            forward_batch.req_pool_indices, : metadata.max_seq_len
        ]
        # Convert the block table to a strided format.
        if self.page_size > 1:
            strided_indices = torch.arange(
                0, metadata.block_tables.shape[1], self.page_size, device=self.device
            )
            metadata.block_tables = (
                metadata.block_tables[:, strided_indices] // self.page_size
            )

        metadata.query_start_loc = torch.zeros(
            batch_size + 1, dtype=torch.int32, device=metadata.seq_lens_tensor.device
        )
        if forward_mode.is_prefill():
            metadata.query_start_loc[1:] = torch.cumsum(
# ... omitted 83 lines ...
                        start + (metadata.max_seq_len_succ - 1) // self.page_size + 1,
                        (cache_seq_lens[i] - 1) // self.page_size + 1,
                    )
                    block_tables_succ[i, : end - start] = metadata.block_tables[
                        i, start:end
                    ]
                metadata.block_tables_succ = block_tables_succ

            metadata.seq_lens_inter = (chunk_num_curr - 1).clip(min=0) * chunk_len
            metadata.max_seq_len_inter = metadata.seq_lens_inter.max().item()

        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 299-410: method DualChunkFlashAttentionBackend.forward_extend
```python
    def forward_extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: "RadixAttention",
        forward_batch: ForwardBatch,
        save_kv_cache=True,
    ):
        # Use precomputed metadata across all layers
        metadata = self.forward_metadata

        (
            query,
            query_succ,
            query_inter,
            query_succ_critical,
            query_inter_critical,
        ) = torch.split(q, q.shape[-1] // 5, dim=-1)

        # Reshape the query, key, and value tensors.
        query = query.view(-1, self.num_heads, self.head_size)
        query_succ = query_succ.view(-1, self.num_heads, self.head_size)
        query_inter = query_inter.view(-1, self.num_heads, self.head_size)
        query_succ_critical = query_succ_critical.view(
            -1, self.num_heads, self.head_size
        )
        query_inter_critical = query_inter_critical.view(
            -1, self.num_heads, self.head_size
        )
        key = k.view(-1, self.num_kv_heads, self.head_size)
        value = v.view(-1, self.num_kv_heads, self.head_size)
# ... omitted 68 lines ...
                cu_seqlens_q=metadata.query_start_loc,
                cu_seqlens_k=metadata.seq_start_loc,
                orig_seq_lens=metadata.orig_seq_lens,
                scaling_factor=metadata.scaling_factor,
                softmax_scale=layer.scaling,
                causal=True,
                window_size=(-1, -1),
                block_table=metadata.block_tables,
                chunk_size=self.chunk_size,
                local_size=self.local_size,
            )
        return o.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 412-487: method DualChunkFlashAttentionBackend.forward_decode
```python
    def forward_decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: "RadixAttention",
        forward_batch: ForwardBatch,
        save_kv_cache=True,
    ) -> torch.Tensor:
        # Use precomputed metadata across all layers
        metadata = self.forward_metadata

        (
            query,
            query_succ,
            query_inter,
            query_succ_critical,
            query_inter_critical,
        ) = torch.split(q, q.shape[-1] // 5, dim=-1)

        # Reshape the query, key, and value tensors.
        query = query.view(-1, self.num_heads, self.head_size)
        query_succ = query_succ.view(-1, self.num_heads, self.head_size)
        query_inter = query_inter.view(-1, self.num_heads, self.head_size)
        query_succ_critical = query_succ_critical.view(
            -1, self.num_heads, self.head_size
        )
        query_inter_critical = query_inter_critical.view(
            -1, self.num_heads, self.head_size
        )
        key = k.view(-1, self.num_kv_heads, self.head_size)
        value = v.view(-1, self.num_kv_heads, self.head_size)
# ... omitted 32 lines ...
            key_cache,
            value_cache,
            block_table=metadata.block_tables,
            cache_seqlens=metadata.seq_lens_tensor,
            softmax_scale=layer.scaling,
            causal=True,
            chunk_size=self.chunk_size,
            local_size=self.local_size,
            original_max_position_embeddings=self.original_max_position_embeddings,
            decode_meta=metadata,
        ).squeeze(1)
        return o.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 489-533: method DualChunkFlashAttentionBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        """Initialize CUDA graph state for the attention backend.

        Args:
            max_bs (int): Maximum batch size to support in CUDA graphs

        This creates fixed-size tensors that will be reused during CUDA graph replay
        to avoid memory allocations.
        """
        self.decode_metadata = {
            "seq_lens_tensor": torch.zeros(
                max_bs, dtype=torch.int32, device=self.device
            ),
            "orig_seq_lens_tensor": torch.zeros(
                max_bs, dtype=torch.int32, device=self.device
            ),
            "scaling_factor": torch.zeros(
                max_bs, dtype=torch.float32, device=self.device
            ),
            "block_tables": torch.zeros(
                max_bs,
                (self.max_context_len - 1) // self.page_size + 1,
                dtype=torch.int32,
                device=self.device,
            ),
            "block_tables_intra": torch.zeros(
                max_bs,
                (self.max_context_len - 1) // self.page_size + 1,
                dtype=torch.int32,
                device=self.device,
            ),
            "seq_lens_intra": torch.zeros(
                max_bs, dtype=torch.int32, device=self.device
            ),
            "block_tables_succ": torch.zeros(
                max_bs,
                (self.max_context_len - 1) // self.page_size + 1,
                dtype=torch.int32,
                device=self.device,
            ),
            "seq_lens_succ": torch.zeros(max_bs, dtype=torch.int32, device=self.device),
            "seq_lens_inter": torch.zeros(
                max_bs, dtype=torch.int32, device=self.device
            ),
        }
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 535-581: method DualChunkFlashAttentionBackend.init_forward_metadata_capture_cuda_graph
```python
    def init_forward_metadata_capture_cuda_graph(
        self,
        bs: int,
        num_tokens: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: ForwardMode,
        spec_info: Optional[None],
    ):
        metadata = DualChunkFlashAttentionMetadata()

        if forward_mode.is_decode_or_idle():
            if self.original_max_position_embeddings > 0:
                metadata.scaling_factor = self.decode_metadata["scaling_factor"][:bs]

            metadata.seq_lens_tensor = self.decode_metadata["seq_lens_tensor"][:bs]
            metadata.orig_seq_lens_tensor = self.decode_metadata[
                "orig_seq_lens_tensor"
            ][:bs]
            metadata.max_seq_len = self.max_context_len
            metadata.block_tables = self.decode_metadata["block_tables"][
                req_pool_indices, :
            ]

            # intra
            metadata.max_seq_len_intra = self.max_context_len
            metadata.seq_lens_intra = self.decode_metadata["seq_lens_intra"][:bs]

            metadata.block_tables_intra = self.decode_metadata["block_tables_intra"][
                :bs, :
            ]

            # succ
            metadata.seq_lens_succ = self.decode_metadata["seq_lens_succ"][:bs]
            metadata.max_seq_len_succ = self.max_context_len

            metadata.block_tables_succ = self.decode_metadata["block_tables_succ"][
                :bs, :
            ]

            metadata.seq_lens_inter = self.decode_metadata["seq_lens_inter"][:bs]
            metadata.max_seq_len_inter = self.max_context_len

            self.decode_metadata[bs] = metadata

        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 583-671: method DualChunkFlashAttentionBackend.init_forward_metadata_replay_cuda_graph
```python
    def init_forward_metadata_replay_cuda_graph(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_sum: int,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: ForwardMode,
        spec_info: Optional[None],
        seq_lens_cpu: Optional[torch.Tensor],
        out_cache_loc: torch.Tensor = None,
    ):
        """Initialize forward metadata for replaying CUDA graph."""
        assert forward_mode.is_decode()
        seq_lens = seq_lens[:bs]
        req_pool_indices = req_pool_indices[:bs]
        metadata = self.decode_metadata[bs]

        metadata.seq_lens_tensor.copy_(seq_lens.to(torch.int32))
        metadata.seq_lens = seq_lens.tolist()
        metadata.max_seq_len = seq_lens.max().item()

        metadata.orig_seq_lens_tensor.copy_(seq_lens)
        metadata.orig_seq_lens = seq_lens.tolist()

        block_tables = self.req_to_token[req_pool_indices, : metadata.max_seq_len]
        # Convert the block table to a strided format.
        if self.page_size > 1:
            strided_indices = torch.arange(
                0, block_tables.shape[1], self.page_size, device=self.device
            )
            block_tables = block_tables[:, strided_indices] // self.page_size
# ... omitted 45 lines ...
                    start + (metadata.max_seq_len_succ - 1) // self.page_size + 1,
                    (cache_seq_lens[i] - 1) // self.page_size + 1,
                )
                metadata.block_tables_succ[i, : end - start] = metadata.block_tables[
                    i, start:end
                ]

        seq_lens_inter = (chunk_num_curr - 1).clip(min=0) * chunk_len
        metadata.seq_lens_inter.copy_(seq_lens_inter)
        metadata.max_seq_len_inter = metadata.seq_lens_inter.max().item()

        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 673-675: method DualChunkFlashAttentionBackend.get_cuda_graph_seq_len_fill_value
```python
    def get_cuda_graph_seq_len_fill_value(self):
        """Get the fill value for sequence length in CUDA graph."""
        return 1
```
**EN:** Computes and returns get cuda graph seq len fill value from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cuda graph seq len fill value。

### Lines 677-832: method DualChunkFlashAttentionBackend._dual_chunk_flash_attn_prefill
```python
    def _dual_chunk_flash_attn_prefill(
        self,
        q,
        q_succ,
        q_inter,
        q_succ_critical,
        q_inter_critical,
        k,
        v,
        cu_seqlens_q,
        cu_seqlens_k,
        orig_seq_lens: List[int],
        scaling_factor: torch.Tensor,
        softmax_scale: float,
        causal: Optional[bool] = True,
        window_size: Tuple[int, int] = (-1, -1),
        block_table: Optional[torch.Tensor] = None,
        chunk_size: int = 8192,
        local_size: int = 1024,
    ):
        if not causal:
            raise ValueError("Dual Chunk Attention does not support causal=False")
        if window_size != (-1, -1):
            raise ValueError("Dual Chunk Attention does not support window_size")

        cu_seqlens_q_cpu = cu_seqlens_q.cpu().tolist()
        cu_seqlens_k_cpu = cu_seqlens_k.cpu().tolist()
        all_outputs = []

        for i in range(0, len(cu_seqlens_q_cpu) - 1):
            qs = cu_seqlens_q_cpu[i]
            qe = cu_seqlens_q_cpu[i : i + 2][-1]
# ... omitted 112 lines ...
                        current_v_head,
                        current_block_table,
                        softmax_scale,
                        chunk_size,
                        local_size,
                        scaling_factor[i].item(),
                        ke - ks,
                        sparse_attn_enabled=sparse_attn_enabled,
                    )
                    current_output[:, head_id : head_id + 1, :] = current_out
            all_outputs.append(current_output)
        return torch.cat(all_outputs, dim=0)
```
**EN:** Implements the dual chunk flash attn prefill routine used by this attention module.
**CN:** 实现该注意力模块使用的 dual chunk flash attn prefill 例程。

### Lines 834-1330: method DualChunkFlashAttentionBackend._dual_chunk_flash_attn_prefill_func
```python
    def _dual_chunk_flash_attn_prefill_func(
        self,
        q,
        q_succ,
        q_inter,
        q_succ_critical,
        q_inter_critical,
        k,
        v,
        block_table,
        softmax_scale: float,
        chunk_size: int,
        local_size: int,
        scaling_factor: float,
        k_length: int,
        sparse_attn_enabled: Optional[bool] = True,
        heads_vertical_size=None,
        heads_slash_size=None,
        group_size=None,
    ):
        flash_results = []
        chunk_len = chunk_size - local_size

        if block_table is not None:
            block_size = v.shape[1]
            if chunk_len % block_size != 0:
                raise ValueError("chunk_len must be divisible by block_size.")
        else:
            block_size = 1

        if self.original_max_position_embeddings > 0:
            softmax_scale = softmax_scale * scaling_factor
# ... omitted 453 lines ...
                        vertical_indices=inter_vertical_indices,
                        slash_indices=inter_slash_indices,
                        sparse_attn_enabled=sparse_attn_enabled,
                    )
                flash_per_chunk.append(flash_result)

            flash_results.append(flash_per_chunk)
            begin = end

        attn_output = self._merge_attn_outputs(flash_results)
        del flash_results
        return attn_output
```
**EN:** Implements the dual chunk flash attn prefill func routine used by this attention module.
**CN:** 实现该注意力模块使用的 dual chunk flash attn prefill func 例程。

### Lines 1332-1425: method DualChunkFlashAttentionBackend._do_flash_attn
```python
    def _do_flash_attn(
        self,
        query_states: torch.Tensor,
        key_states: torch.Tensor,
        value_states: torch.Tensor,
        softmax_scale: float,
        causal: bool = True,
        max_seqlen_k: Optional[int] = None,
        stage: str = "intra",
        vertical_indices: Optional[torch.Tensor] = None,
        slash_indices: Optional[torch.Tensor] = None,
        vertical_indices_count: Optional[torch.Tensor] = None,
        slash_indices_count: Optional[torch.Tensor] = None,
        mergehead_softmax_scale: Optional[float] = None,
        sparse_attn_enabled: Optional[bool] = False,
    ):
        if max_seqlen_k is None:
            max_seqlen_k = key_states.shape[0]

        q_len = query_states.shape[0]
        q_heads = query_states.shape[1]
        h_dim = query_states.shape[-1]

        if sparse_attn_enabled:
            assert slash_indices is not None
            if stage == "intra":
                assert causal
            else:
                assert not causal

            query_states = query_states.unsqueeze(0).transpose(1, 2)
            key_states = key_states.unsqueeze(0).transpose(1, 2)
# ... omitted 50 lines ...
                device=query_states.device,
            ),
            max_seqlen_q=query_states.shape[0],
            cu_seqlens_k=torch.tensor(
                [0, max_seqlen_k], dtype=torch.int32, device=query_states.device
            ),
            max_seqlen_k=max_seqlen_k,
            causal=causal,
            return_softmax_lse=True,
        )
        softmax_lse = softmax_lse.view(q_len, q_heads, 1).transpose(0, 2).float()
        return output, softmax_lse
```
**EN:** Implements the do flash attn routine used by this attention module.
**CN:** 实现该注意力模块使用的 do flash attn 例程。

### Lines 1427-1467: method DualChunkFlashAttentionBackend._merge_attn_outputs
```python
    def _merge_attn_outputs(
        self,
        flash_results: List[List[Tuple[torch.Tensor, torch.Tensor]]],
        return_lse: Optional[bool] = False,
    ) -> torch.Tensor:
        attn_outputs_all = []
        logits_all = []

        for flash_per_chunk in flash_results:
            if len(flash_per_chunk) == 1:
                attn_outputs_all.append(flash_per_chunk[0][0])
                if return_lse:
                    logits_all.append(flash_per_chunk[0][1])
                continue

            attn_outputs = torch.stack(
                [flash_attn_output[0] for flash_attn_output in flash_per_chunk]
            )
            logits = torch.stack(
                [flash_attn_output[1] for flash_attn_output in flash_per_chunk]
            )
            logits = logits.to(torch.float32)

            if return_lse:
                max_val = torch.max(logits, dim=0).values
                diff = torch.abs(logits[0] - logits[1])
                log_sum_exp = max_val + torch.log1p(torch.exp(-diff))
                logits_all.append(log_sum_exp)

            max_logits = torch.max(logits, dim=0).values
            stable_logits = logits - max_logits.unsqueeze(0)
            lse_s = torch.exp(stable_logits).detach()
            lse_sum = torch.sum(lse_s, dim=0)
            lse_s /= lse_sum
            attn_outputs *= lse_s.unsqueeze(-1).transpose(2, 3).squeeze(1)
            attn_outputs_all.append(attn_outputs.sum(dim=0))

        if return_lse:
            return (torch.cat(attn_outputs_all, dim=0), torch.cat(logits_all, dim=-1))
        else:
            return torch.cat(attn_outputs_all, dim=0)
```
**EN:** Implements the merge attn outputs routine used by this attention module.
**CN:** 实现该注意力模块使用的 merge attn outputs 例程。

### Lines 1469-1561: method DualChunkFlashAttentionBackend._dual_chunk_flash_attn_decoding
```python
    def _dual_chunk_flash_attn_decoding(
        self,
        query: torch.Tensor,
        query_succ: torch.Tensor,
        query_inter: torch.Tensor,
        key_cache: torch.Tensor,
        value_cache: torch.Tensor,
        block_table: torch.Tensor,
        cache_seqlens: torch.Tensor,
        softmax_scale: float,
        causal: bool,
        chunk_size: int,
        local_size: int,
        original_max_position_embeddings: int,
        decode_meta: DualChunkFlashAttentionMetadata,
    ):
        if not causal:
            raise ValueError("Dual Chunk Attention does not support causal=False")

        block_size = value_cache.shape[1]
        chunk_len = chunk_size - local_size
        if chunk_len % block_size != 0:
            raise ValueError("chunk_len must be divisible by block_size.")
        if original_max_position_embeddings > 0:
            assert decode_meta.scaling_factor is not None
            scaling_factor = decode_meta.scaling_factor
            query = (query * scaling_factor.view(-1, 1, 1, 1)).to(
                query.dtype
            )  # possible for numerical issue, need to fused in the kernel
            query_succ = (query_succ * scaling_factor.view(-1, 1, 1, 1)).to(query.dtype)
            query_inter = (query_inter * scaling_factor.view(-1, 1, 1, 1)).to(
                query.dtype
# ... omitted 49 lines ...
            softmax_lses_list.append(inter_softmax_lse)
        outputs = torch.stack(outputs_list, dim=0)
        del outputs_list
        softmax_lses = torch.stack(softmax_lses_list, dim=0).to(torch.float32)
        del softmax_lses_list
        max_logits = torch.max(softmax_lses, dim=0).values
        stable_logits = softmax_lses - max_logits.unsqueeze(0)
        lse_s = torch.exp(stable_logits).detach()
        lse_sum = torch.sum(lse_s, dim=0)
        lse_s /= lse_sum
        outputs *= lse_s.unsqueeze(-1).transpose(2, 3)
        return outputs.sum(0)
```
**EN:** Implements the dual chunk flash attn decoding routine used by this attention module.
**CN:** 实现该注意力模块使用的 dual chunk flash attn decoding 例程。

### Lines 1563-1586: method DualChunkFlashAttentionBackend._dual_chunk_flash_attn_decoding_with_exp_sums
```python
    def _dual_chunk_flash_attn_decoding_with_exp_sums(
        self,
        query: torch.Tensor,
        key_cache: torch.Tensor,
        value_cache: torch.Tensor,
        block_table: torch.Tensor,
        cache_seqlens: torch.Tensor,
        softmax_scale: float,
        causal: bool,
    ):
        out, softmax_lse, *rest_expand = flash_attn_with_kvcache(
            q=query,
            k_cache=key_cache,
            v_cache=value_cache,
            page_table=block_table,
            cache_seqlens=cache_seqlens,
            softmax_scale=softmax_scale,
            causal=causal,
            return_softmax_lse=True,
        )
        mask = cache_seqlens == 0
        out[mask] = 0
        softmax_lse[mask] = -float("inf")
        return out, softmax_lse
```
**EN:** Implements the dual chunk flash attn decoding with exp sums routine used by this attention module.
**CN:** 实现该注意力模块使用的 dual chunk flash attn decoding with exp sums 例程。

### Lines 1589-1682: function _vertical_slash_sparse_attention
```python
def _vertical_slash_sparse_attention(
    query: torch.Tensor,  # [BATCH, N_HEADS, N_CTX, D_HEAD]
    key: torch.Tensor,  # [BATCH, N_HEADS, N_KV_CTX, D_HEAD]
    value: torch.Tensor,  # [BATCH, N_HEADS, N_KV_CTX, D_HEAD]
    v_idx: torch.Tensor,  # [BATCH, N_HEADS, NNZ_V]
    s_idx: torch.Tensor,  # [BATCH, N_HEADS, NNZ_S]
    softmax_scale: float,
    causal: bool = True,
    stage: str = "intra",
    block_size_M: int = 64,
    block_size_N: int = 64,
    vertical_indices_count: torch.Tensor = None,  # [N_HEADS,]
    slash_indices_count: torch.Tensor = None,
):
    if stage == "intra":
        assert causal
    else:
        assert not causal

    batch_size, num_heads, context_size, head_dim = query.shape
    _, _, kv_seq_len, _ = key.shape

    if head_dim not in [16, 32, 64, 128, 256, 512]:
        target_dim = 2 ** math.ceil(math.log2(head_dim)) - head_dim
        query = F.pad(query, [0, target_dim, 0, 0, 0, 0, 0, 0])
        key = F.pad(key, [0, target_dim, 0, 0, 0, 0, 0, 0])
        value = F.pad(value, [0, target_dim, 0, 0, 0, 0, 0, 0])

    v_idx = (
        v_idx.to(torch.int32)
        .reshape((batch_size, num_heads, -1))
        .sort(dim=-1, descending=False)[0]
# ... omitted 50 lines ...
        v,
        block_count,
        block_offset,
        column_count,
        column_index,
        causal=causal,
        softmax_scale=softmax_scale,
        return_softmax_lse=True,
    )
    out = out.transpose(1, 2).contiguous()
    softmax_lse = lse.reshape(*lse.shape, 1)
    return (out[..., :context_size, :head_dim], softmax_lse[..., :context_size, :])
```
**EN:** Implements the vertical slash sparse attention routine used by this attention module.
**CN:** 实现该注意力模块使用的 vertical slash sparse attention 例程。

### Lines 1685-1697: function _sum_all_diagonal_matrix
```python
def _sum_all_diagonal_matrix(mat: torch.tensor):
    h, n, m = mat.shape
    # Zero matrix used for padding
    zero_mat = torch.zeros((h, n, n), device=mat.device)
    # pads the matrix on left and right
    mat_padded = torch.cat((zero_mat, mat, zero_mat), -1)
    # Change the strides
    mat_strided = mat_padded.as_strided(
        (1, n, n + m), (n * (2 * n + m), 2 * n + m + 1, 1)
    )
    # Sums the resulting matrix's columns
    sum_diags = torch.sum(mat_strided, 1)
    return sum_diags[:, 1:]  # drop left bottom corner
```
**EN:** Implements the sum all diagonal matrix routine used by this attention module.
**CN:** 实现该注意力模块使用的 sum all diagonal matrix 例程。

### Lines 1700-1703: function _get_block
```python
def _get_block(block_table: torch.Tensor, block_size: int, begin: int, end: int):
    begin_block = begin // block_size
    end_block = (end - 1) // block_size + 1
    return block_table[begin_block:end_block]
```
**EN:** Implements the get block routine used by this attention module.
**CN:** 实现该注意力模块使用的 get block 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `functools`
- `logging`
- `math`
- `dataclasses.dataclass`
- `typing.TYPE_CHECKING`
- `typing.Any`
- `typing.Dict`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn.functional`
- `sgl_kernel.sparse_flash_attn.convert_vertical_slash_indexes`
- `sgl_kernel.sparse_flash_attn.convert_vertical_slash_indexes_mergehead`
- `sgl_kernel.sparse_flash_attn.sparse_attn_func`
- `sglang.jit_kernel.flash_attention.flash_attn_varlen_func`
- `sglang.jit_kernel.flash_attention.flash_attn_with_kvcache`
- `sglang.srt.distributed.parallel_state.get_tensor_model_parallel_rank`
- `sglang.srt.layers.attention.base_attn_backend.AttentionBackend`
- `sglang.srt.layers.attention.flashattention_backend.FlashAttentionMetadata`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.model_runner.ModelRunner`
