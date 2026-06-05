# trtllm_mha_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/trtllm_mha_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the trtllm mha backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 trtllm mha backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-1: imports
```python
from __future__ import annotations
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 3-6: docstring
```python
"""
Support attention backend for TRTLLM MHA kernels from flashinfer.
The kernel supports sm100 only, with sliding window and attention sink features.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 8-26: imports
```python
import logging
from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.environ import envs
from sglang.srt.layers.attention.flashinfer_backend import (
    FlashInferAttnBackend,
    FlashInferMultiStepDraftBackend,
)
from sglang.srt.layers.attention.triton_ops.trtllm_fp8_kv_kernel import (
    fused_fp8_set_kv_buffer,
)
from sglang.srt.layers.attention.utils import canonicalize_stride
from sglang.srt.mem_cache.swa_memory_pool import SWAKVPool
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.utils import is_flashinfer_available
from sglang.srt.utils.common import is_sm90_supported, is_sm120_supported
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 28-28: module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 30-31: conditional branch
```python
if is_flashinfer_available():
    import flashinfer
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 33-36: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner
    from sglang.srt.speculative.spec_info import SpecInput
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 41-44: module constants
```python
DEFAULT_WORKSPACE_SIZE_MB = 512

# Reuse this workspace buffer across all TRTLLM MHA wrappers
global_zero_init_workspace_buffer = None
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 47-62: class TRTLLMMHAMetadata
```python
@dataclass
class TRTLLMMHAMetadata:
    # Sequence lengths for the forward batch
    cache_seqlens_int32: torch.Tensor = None
    # Maximum sequence length for query
    max_seq_len_q: int = 1
    # Maximum sequence length for key
    max_seq_len_k: int = 0
    # Cumulative sequence lengths for `query
    cu_seqlens_q: torch.Tensor = None
    # Cumulative sequence lengths for key
    cu_seqlens_k: torch.Tensor = None
    # Page table, the index of KV Cache Tables/Blocks
    page_table: torch.Tensor = None
    # Page table for SWA layers (translated from full pool indices to SWA pool indices)
    swa_page_table: torch.Tensor = None
```
**EN:** Dataclass-style container that stores structured runtime state for trtllmmhametadata.
**CN:** 该数据类风格的容器用于存储 trtllmmhametadata 的结构化运行时状态。

### Lines 65-67: class TRTLLMHAAttnBackend
```python
class TRTLLMHAAttnBackend(FlashInferAttnBackend):
    """TRTLLM MHA attention kernel from flashinfer."""
```
**EN:** Concrete attention backend that connects trtllmhaattn backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 trtllmhaattn backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 68-148: method TRTLLMHAAttnBackend.__init__
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        skip_prefill: bool = False,
        kv_indptr_buf: Optional[torch.Tensor] = None,
        kv_last_page_len_buf: Optional[torch.Tensor] = None,
        speculative_step_id: int = 0,
    ):
        # Capture workspace size before super().__init__() to preserve user's
        # SGLANG_FLASHINFER_WORKSPACE_SIZE setting (may be overridden by parent)
        env_var = envs.SGLANG_FLASHINFER_WORKSPACE_SIZE
        workspace_size_bytes = (
            env_var.get()
            if env_var.is_set()
            else DEFAULT_WORKSPACE_SIZE_MB * 1024 * 1024
        )

        super().__init__(
            model_runner, skip_prefill, kv_indptr_buf, kv_last_page_len_buf
        )

        config = model_runner.model_config

        # MHA-specific dimensions
        self.max_context_len = model_runner.model_config.context_len
        self.hidden_size = config.hidden_size

        # Runtime parameters
        self.data_type = model_runner.kv_cache_dtype
        self.q_data_type = model_runner.dtype
        self.page_size = model_runner.page_size
        self.req_to_token = model_runner.req_to_token_pool.req_to_token
# ... omitted 37 lines ...
        # Forward metadata
        self.forward_metadata: Optional[TRTLLMMHAMetadata] = None

        # Init backend (XQA or TRTLLM-GEN)
        # We need to specify q_type and out_type for different backend
        # XQA: (q_type must be bf16)
        #   KV bf16: q_type = bf16, out_type=model_runner.dtype
        #   KV fp8: q_type = bf16, out_type=model_runner.dtype
        # TRTLLM-GEN:
        #   KV bf16: q_type = bf16, out_type=model_runner.dtype
        #   KV fp8: q_type = fp8, out_type=model_runner.dtype
        self.is_xqa_impl = is_sm90_supported() or is_sm120_supported()
```
**EN:** Initializes the TRTLLMHAAttnBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TRTLLMHAAttnBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 150-159: method TRTLLMHAAttnBackend._maybe_translate_swa
```python
    def _maybe_translate_swa(
        self, token_indices: torch.Tensor
    ) -> Optional[torch.Tensor]:
        """Translate full-pool token indices to SWA-pool indices, or return None."""
        if not self.use_sliding_window_kv_pool:
            return None
        shape = token_indices.shape
        return self._swa_kv_pool.translate_loc_from_full_to_swa(
            token_indices.reshape(-1)
        ).reshape(shape)
```
**EN:** Implements the maybe translate swa routine used by this attention module.
**CN:** 实现该注意力模块使用的 maybe translate swa 例程。

### Lines 161-167: method TRTLLMHAAttnBackend._alloc_swa_page_table
```python
    def _alloc_swa_page_table(
        self, max_bs: int, max_num_pages: int
    ) -> Optional[torch.Tensor]:
        """Allocate a SWA page_table buffer, or return None for non-SWA models."""
        if not self.use_sliding_window_kv_pool:
            return None
        return torch.zeros(max_bs, max_num_pages, dtype=torch.int32, device=self.device)
```
**EN:** Implements the alloc swa page table routine used by this attention module.
**CN:** 实现该注意力模块使用的 alloc swa page table 例程。

### Lines 169-179: method TRTLLMHAAttnBackend._copy_swa_page_table
```python
    def _copy_swa_page_table(
        self,
        metadata: TRTLLMMHAMetadata,
        page_indices: torch.Tensor,
        num_pages: int,
    ):
        """Translate and copy SWA page indices into metadata. No-op for non-SWA."""
        if metadata.swa_page_table is None:
            return
        swa_indices = self._maybe_translate_swa(page_indices)
        metadata.swa_page_table[:, :num_pages].copy_(swa_indices // self.page_size)
```
**EN:** Implements the copy swa page table routine used by this attention module.
**CN:** 实现该注意力模块使用的 copy swa page table 例程。

### Lines 181-195: method TRTLLMHAAttnBackend._get_layer_cache_loc
```python
    def _get_layer_cache_loc(
        self,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        """Return cache locations in the correct index space for the given layer."""
        if self.use_sliding_window_kv_pool:
            _, is_swa = self._swa_kv_pool.layers_mapping[layer.layer_id]
            if is_swa:
                if forward_batch.out_cache_loc_swa is not None:
                    return forward_batch.out_cache_loc_swa
                return self._swa_kv_pool.translate_loc_from_full_to_swa(
                    forward_batch.out_cache_loc
                )
        return forward_batch.out_cache_loc
```
**EN:** Implements the get layer cache loc routine used by this attention module.
**CN:** 实现该注意力模块使用的 get layer cache loc 例程。

### Lines 197-203: method TRTLLMHAAttnBackend._bind_swa_page_table
```python
    def _bind_swa_page_table(
        self, metadata: TRTLLMMHAMetadata, source: dict, key: str, bs: int
    ):
        """Bind a pre-allocated SWA page_table slice to metadata for CUDA graph."""
        buf = source.get(key)
        if buf is not None:
            metadata.swa_page_table = buf[:bs, :]
```
**EN:** Implements the bind swa page table routine used by this attention module.
**CN:** 实现该注意力模块使用的 bind swa page table 例程。

### Lines 205-214: method TRTLLMHAAttnBackend._get_layer_page_table
```python
    def _get_layer_page_table(
        self, layer: RadixAttention, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        """Return the correct page_table for the given layer (SWA or full)."""
        swa_pt = self.forward_metadata.swa_page_table
        if swa_pt is not None:
            _, is_swa = self._swa_kv_pool.layers_mapping[layer.layer_id]
            if is_swa:
                return swa_pt
        return self.forward_metadata.page_table
```
**EN:** Implements the get layer page table routine used by this attention module.
**CN:** 实现该注意力模块使用的 get layer page table 例程。

### Lines 216-306: method TRTLLMHAAttnBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(
        self,
        max_bs: int,
        max_num_tokens: int,
        kv_indices_buf: Optional[torch.Tensor] = None,
    ):
        """Initialize CUDA graph state for TRTLLM MHA."""
        max_num_pages = (self.max_context_len + self.page_size - 1) // self.page_size
        self.decode_cuda_graph_metadata = {
            "cache_seqlens": torch.zeros(max_bs, dtype=torch.int32, device=self.device),
            "page_table": torch.zeros(
                max_bs,
                max_num_pages,
                dtype=torch.int32,
                device=self.device,
            ),
            "swa_page_table": self._alloc_swa_page_table(max_bs, max_num_pages),
            "strided_indices": torch.arange(
                0, self.max_context_len, self.page_size, device=self.device
            ),
        }

        if (
            self.speculative_num_draft_tokens is not None
            and self.speculative_num_draft_tokens > 0
        ):
            self.decode_cuda_graph_metadata["cu_seqlens_q"] = torch.arange(
                0, max_bs + 1, dtype=torch.int32, device=self.device
            )
            self.decode_cuda_graph_metadata["cu_seqlens_k"] = torch.zeros(
                max_bs + 1, dtype=torch.int32, device=self.device
            )
# ... omitted 47 lines ...
                ),
                "page_table": torch.zeros(
                    max_bs,
                    max_num_pages,
                    dtype=torch.int32,
                    device=self.device,
                ),
                "swa_page_table": self._alloc_swa_page_table(max_bs, max_num_pages),
                "strided_indices": torch.arange(
                    0, self.max_context_len, self.page_size, device=self.device
                ),
            }
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 308-443: method TRTLLMHAAttnBackend.init_forward_metadata_capture_cuda_graph
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
        """Initialize metadata for CUDA graph capture."""
        metadata = TRTLLMMHAMetadata()
        device = seq_lens.device

        if forward_mode.is_decode_or_idle():
            if spec_info is not None:
                # Draft Decode
                # Here we only support topk = 1 for now.
                metadata.cache_seqlens_int32 = self.decode_cuda_graph_metadata[
                    "cache_seqlens"
                ][:bs]
                metadata.max_seq_len_k = seq_lens.max().item() + (
                    self.speculative_step_id + 1
                )
                metadata.cu_seqlens_q = self.decode_cuda_graph_metadata["cu_seqlens_q"][
                    : bs + 1
                ]
                metadata.cu_seqlens_k = torch.nn.functional.pad(
                    torch.cumsum(
                        metadata.cache_seqlens_int32, dim=0, dtype=torch.int32
                    ),
                    (1, 0),
# ... omitted 92 lines ...
            metadata.max_seq_len_k = seq_lens.max().item()

            metadata.page_table = self.draft_extend_metadata["page_table"][:bs, :]
            self._bind_swa_page_table(
                metadata,
                self.draft_extend_metadata,
                "swa_page_table",
                bs,
            )

            self.draft_extend_metadata[bs] = metadata
        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 445-548: method TRTLLMHAAttnBackend.init_forward_metadata_replay_cuda_graph
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
        """Replay CUDA graph with new inputs."""
        seq_lens = seq_lens[:bs]
        seq_lens_cpu = seq_lens_cpu[:bs]
        req_pool_indices = req_pool_indices[:bs]
        metadata = None
        if forward_mode.is_decode_or_idle():
            if spec_info is not None:
                # Draft Decode
                # Here we only support topk = 1 for now.
                metadata = self.decode_cuda_graph_metadata[bs]
                max_len = seq_lens_cpu.max().item()
                metadata.max_seq_len_k = max_len + self.speculative_step_id + 1

                max_seq_pages = (
                    metadata.max_seq_len_k + self.page_size - 1
                ) // self.page_size

                metadata.cache_seqlens_int32.copy_(
                    seq_lens + self.speculative_step_id + 1
                )
            else:
# ... omitted 60 lines ...
            )

            max_seq_pages = (
                metadata.max_seq_len_k + self.page_size - 1
            ) // self.page_size
            page_indices = self.req_to_token[
                req_pool_indices[:, None],
                self.draft_extend_metadata["strided_indices"][:max_seq_pages],
            ]
            metadata.page_table[:, :max_seq_pages].copy_(page_indices // self.page_size)
            self._copy_swa_page_table(metadata, page_indices, max_seq_pages)
        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 550-552: method TRTLLMHAAttnBackend.get_cuda_graph_seq_len_fill_value
```python
    def get_cuda_graph_seq_len_fill_value(self) -> int:
        """Get the fill value for sequence lengths in CUDA graph."""
        return 1
```
**EN:** Computes and returns get cuda graph seq len fill value from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cuda graph seq len fill value。

### Lines 554-556: method TRTLLMHAAttnBackend._should_use_fused_fp8_path
```python
    def _should_use_fused_fp8_path(self, save_kv_cache: bool, k: torch.Tensor) -> bool:
        """Check if we should use the fused FP8 KV cache write path."""
        return save_kv_cache and k is not None and self.data_type == torch.float8_e4m3fn
```
**EN:** Implements the should use fused fp8 path routine used by this attention module.
**CN:** 实现该注意力模块使用的 should use fused fp8 path 例程。

### Lines 558-582: method TRTLLMHAAttnBackend._fused_fp8_set_kv_buffer
```python
    def _fused_fp8_set_kv_buffer(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        **kwargs,
    ):
        """Fused FP8 quantization and KV cache write."""
        cache_loc = self._get_layer_cache_loc(layer, forward_batch)

        # Get K/V cache buffers from token_to_kv_pool
        k_cache, v_cache = forward_batch.token_to_kv_pool.get_kv_buffer(layer.layer_id)

        fused_fp8_set_kv_buffer(
            k=k,
            v=v,
            k_cache=k_cache,
            v_cache=v_cache,
            cache_loc=cache_loc,
            k_scale=layer.k_scale,  # May be None
            v_scale=layer.v_scale,  # May be None
            page_size=self.page_size,
        )
```
**EN:** Implements the fused fp8 set kv buffer routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused fp8 set kv buffer 例程。

### Lines 584-695: method TRTLLMHAAttnBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        """Initialize the metadata for a forward pass."""

        metadata = TRTLLMMHAMetadata()
        seqlens_in_batch = forward_batch.seq_lens
        batch_size = forward_batch.batch_size
        device = seqlens_in_batch.device

        if forward_batch.forward_mode.is_decode_or_idle():
            if forward_batch.spec_info is not None:
                # Draft Decode
                # Here we only support topk = 1 for now.
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
                # Normal Decode
# ... omitted 68 lines ...
            self.strided_indices = torch.arange(
                0, metadata.page_table.shape[1], self.page_size, device=self.device
            )
            metadata.page_table = (
                metadata.page_table[:, self.strided_indices] // self.page_size
            )
            if metadata.swa_page_table is not None:
                metadata.swa_page_table = (
                    metadata.swa_page_table[:, self.strided_indices] // self.page_size
                )

        self.forward_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 697-782: method TRTLLMHAAttnBackend.forward_decode
```python
    def forward_decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        **kwargs,
    ) -> torch.Tensor:
        """Run forward for decode using TRTLLM MHA kernel."""
        cache_loc = forward_batch.out_cache_loc

        use_fused_fp8_path = self._should_use_fused_fp8_path(save_kv_cache, k)

        if use_fused_fp8_path:
            # Use fused FP8 quantization + KV cache write path
            self._fused_fp8_set_kv_buffer(
                q=q,
                k=k,
                v=v,
                layer=layer,
                forward_batch=forward_batch,
            )
            k = None
            v = None
        else:
            # Use original set_kv_buffer path
            if save_kv_cache and k is not None:
                forward_batch.token_to_kv_pool.set_kv_buffer(
                    layer, cache_loc, k, v, layer.k_scale, layer.v_scale
                )
# ... omitted 42 lines ...
            block_tables=page_table,
            seq_lens=self.forward_metadata.cache_seqlens_int32,
            max_seq_len=self.max_context_len,
            bmm1_scale=bmm1_scale,
            bmm2_scale=bmm2_scale,
            window_left=layer.sliding_window_size,
            sinks=attention_sink,
            skip_softmax_threshold_scale_factor=envs.SGLANG_SKIP_SOFTMAX_DECODE_THRESHOLD_SCALE_FACTOR.get(),
            out_dtype=self.q_data_type,  # model_runner.dtype
        )

        return o.view(-1, layer.tp_q_head_num * layer.head_dim)
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 784-885: method TRTLLMHAAttnBackend.forward_extend
```python
    def forward_extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        **kwargs,
    ):
        cache_loc = forward_batch.out_cache_loc

        use_fused_fp8_path = self._should_use_fused_fp8_path(save_kv_cache, k)

        if use_fused_fp8_path:
            # Use fused FP8 quantization + KV cache write path
            self._fused_fp8_set_kv_buffer(
                q=q,
                k=k,
                v=v,
                layer=layer,
                forward_batch=forward_batch,
            )
            k = None
            v = None
        else:
            # Use original set_kv_buffer path
            if save_kv_cache and k is not None:
                forward_batch.token_to_kv_pool.set_kv_buffer(
                    layer, cache_loc, k, v, layer.k_scale, layer.v_scale
                )

# ... omitted 58 lines ...
                bmm1_scale=bmm1_scale,
                bmm2_scale=bmm2_scale,
                batch_size=self.forward_metadata.cu_seqlens_q.shape[0] - 1,
                cum_seq_lens_q=self.forward_metadata.cu_seqlens_q,
                cum_seq_lens_kv=self.forward_metadata.cu_seqlens_k,
                window_left=layer.sliding_window_size,
                sinks=attention_sink,
                skip_softmax_threshold_scale_factor=envs.SGLANG_SKIP_SOFTMAX_PREFILL_THRESHOLD_SCALE_FACTOR.get(),
                out_dtype=self.q_data_type,  # model_runner.dtype
            )

        return o.view(-1, layer.tp_q_head_num * layer.head_dim)
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 888-890: class TRTLLMHAAttnMultiStepDraftBackend
```python
class TRTLLMHAAttnMultiStepDraftBackend(FlashInferMultiStepDraftBackend):
    """Multi-step TRTLLM MHA attention kernel used by EAGLE."""
```
**EN:** Concrete attention backend that connects trtllmhaattn multi step draft backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 trtllmhaattn multi step draft backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 891-902: method TRTLLMHAAttnMultiStepDraftBackend.__init__
```python
    def __init__(
        self, model_runner: ModelRunner, topk: int, speculative_num_steps: int
    ):
        super().__init__(model_runner, topk, speculative_num_steps)
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i] = TRTLLMHAAttnBackend(
                model_runner,
                skip_prefill=True,
                kv_indptr_buf=self.kv_indptr[i],
                kv_last_page_len_buf=self.kv_last_page_len,
                speculative_step_id=i,
            )
```
**EN:** Initializes the TRTLLMHAAttnMultiStepDraftBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TRTLLMHAAttnMultiStepDraftBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 904-906: method TRTLLMHAAttnMultiStepDraftBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_forward_metadata(forward_batch)
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 908-910: method TRTLLMHAAttnMultiStepDraftBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i].init_cuda_graph_state(max_bs, max_num_tokens)
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 912-928: method TRTLLMHAAttnMultiStepDraftBackend.init_forward_metadata_capture_cuda_graph
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

### Lines 930-947: method TRTLLMHAAttnMultiStepDraftBackend.init_forward_metadata_replay_cuda_graph
```python
    def init_forward_metadata_replay_cuda_graph(
        self, forward_batch: ForwardBatch, bs: int
    ):
        assert forward_batch.spec_info is not None
        assert forward_batch.spec_info.is_draft_input()

        for i in range(self.speculative_num_steps - 1):

            self.attn_backends[i].init_forward_metadata_replay_cuda_graph(
                bs,
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                forward_batch.seq_lens_sum,
                encoder_lens=forward_batch.encoder_lens,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
                seq_lens_cpu=forward_batch.seq_lens_cpu,
            )
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** FlashInfer execution path / **CN:** FlashInfer 执行路径
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调

## Dependencies / 依赖关系
- `__future__.annotations`
- `logging`
- `dataclasses.dataclass`
- `typing.TYPE_CHECKING`
- `typing.Optional`
- `torch`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.attention.flashinfer_backend.FlashInferAttnBackend`
- `sglang.srt.layers.attention.flashinfer_backend.FlashInferMultiStepDraftBackend`
- `sglang.srt.layers.attention.triton_ops.trtllm_fp8_kv_kernel.fused_fp8_set_kv_buffer`
- `sglang.srt.layers.attention.utils.canonicalize_stride`
- `sglang.srt.mem_cache.swa_memory_pool.SWAKVPool`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.utils.is_flashinfer_available`
- `sglang.srt.utils.common.is_sm90_supported`
- `sglang.srt.utils.common.is_sm120_supported`
- `flashinfer`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.model_runner.ModelRunner`
- `sglang.srt.speculative.spec_info.SpecInput`
