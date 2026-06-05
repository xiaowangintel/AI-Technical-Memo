# trtllm_mla_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/trtllm_mla_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the trtllm mla backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 trtllm mla backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
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
Support attention backend for TRTLLM MLA kernels from flashinfer.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 7-33: imports
```python
import logging
import math
from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional, Union

import torch
import triton
import triton.language as tl

from sglang.jit_kernel.fixup_zero_kv import fixup_zero_kv_rows
from sglang.srt.compilation.piecewise_context_manager import is_in_piecewise_cuda_graph
from sglang.srt.environ import envs
from sglang.srt.layers.attention.flashinfer_mla_backend import (
    FlashInferMLAAttnBackend,
    FlashInferMLAMultiStepDraftBackend,
)
from sglang.srt.layers.attention.utils import (
    concat_mla_absorb_q_general,
    create_flashmla_kv_indices_triton,
    get_num_page_per_block_flashmla,
    mla_quantize_and_rope_for_fp8,
)
from sglang.srt.layers.dp_attention import get_attention_tp_size
from sglang.srt.layers.quantization.fp8_kernel import scaled_fp8_quant
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import is_flashinfer_available, is_float4_e2m1fn_x2
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 35-36: conditional branch
```python
if is_flashinfer_available():
    import flashinfer
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 38-41: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner
    from sglang.srt.speculative.spec_info import SpecInput
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 43-54: module constants
```python
logger = logging.getLogger(__name__)

# Constants
DEFAULT_WORKSPACE_SIZE_MB = 150  # Memory workspace size in MB

# Block constraint from flashinfer requirements
# From flashinfer.decode._check_trtllm_gen_mla_shape:
#   block_num % (128 / block_size) == 0
# This imposes that the total number of blocks must be divisible by
# (128 / block_size). We capture the 128 constant here so we can
# compute the LCM with other padding constraints.
TRTLLM_BLOCK_CONSTRAINT = 128
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 57-127: function pad_draft_extend_query_kernel
```python
@triton.jit
def pad_draft_extend_query_kernel(
    q_ptr,  # Input query tensor [total_seq_len, num_heads, head_dim]
    padded_q_ptr,  # Output padded query tensor [batch_size, max_seq_len, num_heads, head_dim]
    seq_lens_q_ptr,  # Sequence lengths for each sequence [batch_size]
    cumsum_ptr,  # Cumulative sum of sequence lengths [batch_size + 1]
    batch_size,
    max_seq_len,
    num_heads,
    head_dim,
    BLOCK_SIZE: tl.constexpr,
):
    """Triton kernel for padding draft extended query tensor with parallelized head and dim processing."""
    # Use 3D program IDs: (batch_seq, head_block, dim_block)
    batch_seq_pid = tl.program_id(0)
    head_pid = tl.program_id(1)
    dim_pid = tl.program_id(2)

    batch_id = batch_seq_pid // max_seq_len
    seq_pos = batch_seq_pid % max_seq_len

    if batch_id >= batch_size:
        return

    # Load sequence length for this batch
    seq_len = tl.load(seq_lens_q_ptr + batch_id)

    if seq_pos >= seq_len:
        return

    # Load cumulative sum to get start position in input tensor
    input_start = tl.load(cumsum_ptr + batch_id)
# ... omitted 27 lines ...
        batch_id * max_seq_len * num_heads * head_dim
        + seq_pos * num_heads * head_dim
        + (head_start + tl.arange(0, BLOCK_SIZE))[:, None] * head_dim
        + (dim_start + tl.arange(0, BLOCK_SIZE))[None, :]
    )

    # Store data
    tl.store(
        padded_q_ptr + output_offset,
        data,
        mask=head_mask[:, None] & dim_mask[None, :],
    )
```
**EN:** Implements the pad draft extend query kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 pad draft extend query kernel 例程。

### Lines 130-198: function unpad_draft_extend_output_kernel
```python
@triton.jit
def unpad_draft_extend_output_kernel(
    raw_out_ptr,  # Input raw output tensor (batch_size, token_per_batch, tp_q_head_num, v_head_dim)
    output_ptr,  # Output tensor (-1, tp_q_head_num, v_head_dim)
    num_accept_tokens_ptr,  # Accept lengths for each sequence [batch_size]
    cumsum_ptr,  # Cumulative sum of accept lengths [batch_size + 1]
    batch_size,
    token_per_batch,
    tp_q_head_num,
    v_head_dim,
    BLOCK_SIZE: tl.constexpr,
):
    """Triton kernel for unpadding draft extended output tensor with parallelized head and dim processing."""
    batch_seq_pid = tl.program_id(0)
    head_pid = tl.program_id(1)
    dim_pid = tl.program_id(2)

    batch_id = batch_seq_pid // token_per_batch
    seq_pos = batch_seq_pid % token_per_batch

    if batch_id >= batch_size:
        return

    # Load accept length for this batch
    accept_len = tl.load(num_accept_tokens_ptr + batch_id)

    if seq_pos >= accept_len:
        return

    # Load cumulative sum to get start position in output tensor
    output_start = tl.load(cumsum_ptr + batch_id)
    output_pos = output_start + seq_pos
# ... omitted 25 lines ...
    output_offset = (
        output_pos * tp_q_head_num * v_head_dim
        + (head_start + tl.arange(0, BLOCK_SIZE))[:, None] * v_head_dim
        + (dim_start + tl.arange(0, BLOCK_SIZE))[None, :]
    )

    # Store data
    tl.store(
        output_ptr + output_offset,
        data,
        mask=head_mask[:, None] & dim_mask[None, :],
    )
```
**EN:** Implements the unpad draft extend output kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 unpad draft extend output kernel 例程。

### Lines 201-228: function _quantize_fp8_qkv
```python
def _quantize_fp8_qkv(q, k, v, layer):
    q = q.to(torch.float8_e4m3fn)

    k_scale = getattr(layer, "k_scale_float", None)
    if k_scale is None:
        k_scale = 1.0
    if k_scale != 1.0:
        assert hasattr(layer, "k_scale"), "k_scale is not set"
        k_2d, _ = scaled_fp8_quant(
            k.reshape(-1, k.shape[-1]).contiguous(), layer.k_scale
        )
        k = k_2d.reshape(k.shape)
    else:
        k = k.to(torch.float8_e4m3fn)

    v_scale = getattr(layer, "v_scale_float", None)
    if v_scale is None:
        v_scale = 1.0
    if v_scale != 1.0:
        assert hasattr(layer, "v_scale"), "v_scale is not set"
        v_2d, _ = scaled_fp8_quant(
            v.reshape(-1, v.shape[-1]).contiguous(), layer.v_scale
        )
        v = v_2d.reshape(v.shape)
    else:
        v = v.to(torch.float8_e4m3fn)

    return q, k, v, k_scale, v_scale
```
**EN:** Implements the quantize fp8 qkv routine used by this attention module.
**CN:** 实现该注意力模块使用的 quantize fp8 qkv 例程。

### Lines 231-231: module constants
```python
global_zero_init_workspace_buffer = None
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 234-241: class TRTLLMMLAPrefillMetadata
```python
@dataclass
class TRTLLMMLAPrefillMetadata:
    """Metadata for TRTLLM MLA prefill operations."""

    max_seq_len: int
    cum_seq_lens: torch.Tensor
    seq_lens: torch.Tensor
    fallback_to_flashinfer_impl: bool = False
```
**EN:** Dataclass-style container that stores structured runtime state for trtllmmlaprefill metadata.
**CN:** 该数据类风格的容器用于存储 trtllmmlaprefill metadata 的结构化运行时状态。

### Lines 244-254: class TRTLLMMLADecodeMetadata
```python
@dataclass
class TRTLLMMLADecodeMetadata:
    """Metadata for TRTLLM MLA decode operations."""

    block_kv_indices: Optional[torch.Tensor] = None
    max_seq_len_k: Optional[int] = None
    max_seq_len_q: Optional[int] = None
    sum_seq_lens_q: Optional[int] = None
    cu_seqlens_q: Optional[torch.Tensor] = None
    seq_lens_q: Optional[torch.Tensor] = None
    seq_lens_k: Optional[torch.Tensor] = None
```
**EN:** Dataclass-style container that stores structured runtime state for trtllmmladecode metadata.
**CN:** 该数据类风格的容器用于存储 trtllmmladecode metadata 的结构化运行时状态。

### Lines 257-259: class TRTLLMMLABackend
```python
class TRTLLMMLABackend(FlashInferMLAAttnBackend):
    """TRTLLM MLA attention kernel from flashinfer."""
```
**EN:** Concrete attention backend that connects trtllmmlabackend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 trtllmmlabackend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 260-318: method TRTLLMMLABackend.__init__
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        skip_prefill: bool = False,
        kv_indptr_buf: Optional[torch.Tensor] = None,
        q_indptr_decode_buf: Optional[torch.Tensor] = None,
    ):
        super().__init__(
            model_runner,
            skip_prefill,
            kv_indptr_buf,
            q_indptr_decode_buf,
        )

        config = model_runner.model_config

        # Model parameters
        self.num_q_heads = config.num_attention_heads // get_attention_tp_size()
        self.num_kv_heads = config.get_num_kv_heads(get_attention_tp_size())
        self.num_local_heads = config.num_attention_heads // get_attention_tp_size()

        # MLA-specific dimensions
        self.kv_lora_rank = config.kv_lora_rank
        self.qk_nope_head_dim = config.qk_nope_head_dim
        self.qk_rope_head_dim = config.qk_rope_head_dim
        self.v_head_dim = config.v_head_dim
        self.kv_cache_dim = self.kv_lora_rank + self.qk_rope_head_dim

        # Runtime parameters
        self.scaling = config.scaling
        self.data_type = model_runner.kv_cache_dtype
        self.q_data_type = model_runner.dtype
# ... omitted 15 lines ...
        self.decode_cuda_graph_metadata = {}
        self.decode_cuda_graph_kv_indices = None
        self.padded_q_buffer = None
        self.unpad_output_buffer = None
        self.forward_prefill_metadata: Optional[TRTLLMMLAPrefillMetadata] = None
        self.forward_decode_metadata: Union[TRTLLMMLADecodeMetadata, None] = None

        self.disable_chunked_prefix_cache = (
            get_global_server_args().disable_chunked_prefix_cache
        )

        self.num_draft_tokens = model_runner.server_args.speculative_num_draft_tokens
```
**EN:** Initializes the TRTLLMMLABackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TRTLLMMLABackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 320-341: method TRTLLMMLABackend._calc_padded_blocks
```python
    def _calc_padded_blocks(self, max_seq_len: int) -> int:
        """
        Calculate padded block count that satisfies both TRT-LLM and Triton constraints.

        Args:
            max_seq_len: Maximum sequence length in tokens

        Returns:
            Number of blocks padded to satisfy all constraints
        """
        blocks = triton.cdiv(max_seq_len, self.page_size)

        # Apply dual constraints (take LCM to satisfy both):
        # 1. TRT-LLM: block_num % (128 / page_size) == 0
        # 2. Triton: number of pages per block
        trtllm_constraint = TRTLLM_BLOCK_CONSTRAINT // self.page_size
        triton_constraint = get_num_page_per_block_flashmla(self.page_size)
        constraint_lcm = math.lcm(trtllm_constraint, triton_constraint)

        if blocks % constraint_lcm != 0:
            blocks = triton.cdiv(blocks, constraint_lcm) * constraint_lcm
        return blocks
```
**EN:** Implements the calc padded blocks routine used by this attention module.
**CN:** 实现该注意力模块使用的 calc padded blocks 例程。

### Lines 343-379: method TRTLLMMLABackend._create_block_kv_indices
```python
    def _create_block_kv_indices(
        self,
        batch_size: int,
        max_blocks: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        device: torch.device,
    ) -> torch.Tensor:
        """
        Create block KV indices tensor using Triton kernel.

        Args:
            batch_size: Batch size
            max_blocks: Maximum number of blocks per sequence
            req_pool_indices: Request pool indices
            seq_lens: Sequence lengths
            device: Target device

        Returns:
            Block KV indices tensor
        """
        block_kv_indices = torch.full(
            (batch_size, max_blocks), -1, dtype=torch.int32, device=device
        )

        create_flashmla_kv_indices_triton[(batch_size,)](
            self.req_to_token,
            req_pool_indices,
            seq_lens,
            None,
            block_kv_indices,
            self.req_to_token.stride(0),
            max_blocks,
            PAGED_SIZE=self.page_size,
        )

        return block_kv_indices
```
**EN:** Implements the create block kv indices routine used by this attention module.
**CN:** 实现该注意力模块使用的 create block kv indices 例程。

### Lines 381-426: method TRTLLMMLABackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(
        self,
        max_bs: int,
        max_num_tokens: int,
        kv_indices_buf: Optional[torch.Tensor] = None,
    ):
        """Initialize CUDA graph state for TRTLLM MLA."""

        max_blocks_per_seq = self._calc_padded_blocks(self.max_context_len)

        self.decode_cuda_graph_kv_indices = torch.full(
            (max_bs, max_blocks_per_seq), -1, dtype=torch.int32, device=self.device
        )
        num_tokens_per_bs = max_num_tokens // max_bs

        if is_float4_e2m1fn_x2(self.data_type):
            # Buffer for padded query: (max_bs, max_draft_tokens, num_q_heads, v_head_dim)
            self.store_dtype = torch.uint8
            self.padded_q_buffer = torch.zeros(
                (max_bs, num_tokens_per_bs // 2, self.num_q_heads, self.kv_cache_dim),
                dtype=self.store_dtype,
                device=self.device,
            )

            # Buffer for unpadded output: (max_num_tokens, num_q_heads, v_head_dim)
            self.unpad_output_buffer = torch.zeros(
                (max_num_tokens // 2, self.num_q_heads, 512),
                dtype=self.store_dtype,
                device=self.device,
            )
        else:
            # Buffer for padded query: (max_bs, max_draft_tokens, num_q_heads, v_head_dim)
            self.padded_q_buffer = torch.zeros(
                (max_bs, num_tokens_per_bs, self.num_q_heads, self.kv_cache_dim),
                dtype=self.data_type,
                device=self.device,
            )

            # Buffer for unpadded output: (max_num_tokens, num_q_heads, v_head_dim)
            self.unpad_output_buffer = torch.zeros(
                (max_num_tokens, self.num_q_heads, 512),
                dtype=self.data_type,
                device=self.device,
            )

        super().init_cuda_graph_state(max_bs, max_num_tokens, kv_indices_buf)
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 428-509: method TRTLLMMLABackend.init_forward_metadata_capture_cuda_graph
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

        # Delegate to parent for non-decode modes.
        if (
            not forward_mode.is_decode_or_idle()
            and not forward_mode.is_target_verify()
            and not forward_mode.is_draft_extend(include_v2=True)
        ):
            return super().init_forward_metadata_capture_cuda_graph(
                bs,
                num_tokens,
                req_pool_indices,
                seq_lens,
                encoder_lens,
                forward_mode,
                spec_info,
            )

        metadata = TRTLLMMLADecodeMetadata()

        if forward_mode.is_target_verify():
            seq_lens = seq_lens + self.num_draft_tokens
# ... omitted 38 lines ...
            None,
            block_kv_indices,
            self.req_to_token.stride(0),
            max_blocks_per_seq,
            PAGED_SIZE=self.page_size,
        )

        metadata.block_kv_indices = block_kv_indices
        metadata.max_seq_len_k = self.max_context_len

        self.decode_cuda_graph_metadata[bs] = metadata
        self.forward_decode_metadata = metadata
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 511-574: method TRTLLMMLABackend.init_forward_metadata_replay_cuda_graph
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
        # Delegate to parent for non-decode modes.
        if (
            not forward_mode.is_decode_or_idle()
            and not forward_mode.is_target_verify()
            and not forward_mode.is_draft_extend(include_v2=True)
        ):
            return super().init_forward_metadata_replay_cuda_graph(
                bs,
                req_pool_indices,
                seq_lens,
                seq_lens_sum,
                encoder_lens,
                forward_mode,
                spec_info,
                seq_lens_cpu,
            )

        metadata = self.decode_cuda_graph_metadata[bs]

        if forward_mode.is_target_verify():
# ... omitted 20 lines ...

        # Update block indices for new sequences.
        create_flashmla_kv_indices_triton[(bs,)](
            self.req_to_token,
            req_pool_indices[:bs],
            seq_lens,
            None,
            metadata.block_kv_indices,
            self.req_to_token.stride(0),
            metadata.block_kv_indices.shape[1],
            PAGED_SIZE=self.page_size,
        )
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 576-578: method TRTLLMMLABackend.get_cuda_graph_seq_len_fill_value
```python
    def get_cuda_graph_seq_len_fill_value(self) -> int:
        """Get the fill value for sequence lengths in CUDA graph."""
        return 1
```
**EN:** Computes and returns get cuda graph seq len fill value from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cuda graph seq len fill value。

### Lines 580-673: method TRTLLMMLABackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        """Initialize the metadata for a forward pass."""
        # Delegate to parent for non-decode modes.
        if (
            forward_batch.forward_mode.is_extend()
            and not forward_batch.forward_mode.is_target_verify()
            and not forward_batch.forward_mode.is_draft_extend(include_v2=True)
        ):
            # For extend batch with prefix length > 0, fallback to ragged kernel implemented in flashinfer MLA backend
            # when chunked prefix cache is disabled.
            # Also fallback to flashinfer MLA backend when in piecewise cuda graph, since it only supports MLA forward mode.
            has_prefix = any(forward_batch.extend_prefix_lens_cpu)
            fallback_to_flashinfer_impl = (
                self.disable_chunked_prefix_cache and has_prefix
            ) or is_in_piecewise_cuda_graph()
            if fallback_to_flashinfer_impl:
                super().init_forward_metadata(forward_batch)

            seq_lens = forward_batch.seq_lens - forward_batch.extend_prefix_lens
            cum_seq_lens_q = torch.cat(
                (
                    torch.zeros(
                        1, dtype=torch.int32, device=forward_batch.seq_lens.device
                    ),
                    torch.cumsum(seq_lens, dim=0),
                )
            ).int()
            max_seq_len = max(forward_batch.extend_seq_lens_cpu)
            self.forward_prefill_metadata = TRTLLMMLAPrefillMetadata(
                max_seq_len,
                cum_seq_lens_q,
                seq_lens,
# ... omitted 50 lines ...
                forward_batch.req_pool_indices,
                seq_lens,
                seq_lens.device,
            )

            self.forward_decode_metadata.block_kv_indices = block_kv_indices
            self.forward_decode_metadata.max_seq_len_k = int(max_seq)
            self.forward_decode_metadata.batch_size = bs

            forward_batch.decode_trtllm_mla_metadata = self.forward_decode_metadata
        else:
            return super().init_forward_metadata(forward_batch)
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 675-676: method TRTLLMMLABackend.init_mha_chunk_metadata
```python
    def init_mha_chunk_metadata(self, forward_batch: ForwardBatch):
        super().init_mha_chunk_metadata(forward_batch, disable_flashinfer_ragged=True)
```
**EN:** Prepares init mha chunk metadata so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init mha chunk metadata，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 678-708: method TRTLLMMLABackend.pad_draft_extend_query
```python
    def pad_draft_extend_query(
        self,
        q: torch.Tensor,
        padded_q: torch.Tensor,
        seq_lens_q: torch.Tensor,
        cu_seqlens_q: torch.Tensor,
    ) -> torch.Tensor:
        """Pad draft extended query using Triton kernel."""
        batch_size = cu_seqlens_q.shape[0] - 1
        max_seq_len_q = padded_q.shape[1]
        num_heads = padded_q.shape[2]
        head_dim = padded_q.shape[3]

        # Launch Triton kernel with 3D grid for parallelized head and dim processing
        BLOCK_SIZE = 64
        num_head_blocks = triton.cdiv(num_heads, BLOCK_SIZE)
        num_dim_blocks = triton.cdiv(head_dim, BLOCK_SIZE)
        grid = (batch_size * max_seq_len_q, num_head_blocks, num_dim_blocks)

        pad_draft_extend_query_kernel[grid](
            q_ptr=q,
            padded_q_ptr=padded_q,
            seq_lens_q_ptr=seq_lens_q,
            cumsum_ptr=cu_seqlens_q,
            batch_size=batch_size,
            max_seq_len=max_seq_len_q,
            num_heads=num_heads,
            head_dim=head_dim,
            BLOCK_SIZE=BLOCK_SIZE,
        )
        return padded_q
```
**EN:** Implements the pad draft extend query routine used by this attention module.
**CN:** 实现该注意力模块使用的 pad draft extend query 例程。

### Lines 710-756: method TRTLLMMLABackend.unpad_draft_extend_output
```python
    def unpad_draft_extend_output(
        self,
        raw_out: torch.Tensor,
        cu_seqlens_q: torch.Tensor,
        seq_lens_q: torch.Tensor,
        sum_seq_lens_q: int,
    ) -> torch.Tensor:
        """Unpad draft extended output using Triton kernel."""
        # raw_out: (batch_size, token_per_batch, layer.tp_q_head_num, layer.v_head_dim)
        batch_size = seq_lens_q.shape[0]
        token_per_batch = raw_out.shape[1]  # max_seq_len
        tp_q_head_num = raw_out.shape[2]  # num_heads
        v_head_dim = raw_out.shape[3]  # head_dim
        total_tokens = sum_seq_lens_q

        # Check if we're in CUDA graph mode (buffers are pre-allocated)
        if self.unpad_output_buffer is not None:
            # Use pre-allocated buffer for CUDA graph compatibility
            output = self.unpad_output_buffer[:total_tokens, :, :].to(
                dtype=raw_out.dtype
            )
        else:
            # Dynamic allocation for non-CUDA graph mode
            output = torch.empty(
                (total_tokens, tp_q_head_num, v_head_dim),
                dtype=raw_out.dtype,
                device=raw_out.device,
            )

        # Launch Triton kernel with 3D grid for parallelized head and dim processing
        BLOCK_SIZE = 64
        num_head_blocks = triton.cdiv(tp_q_head_num, BLOCK_SIZE)
        num_dim_blocks = triton.cdiv(v_head_dim, BLOCK_SIZE)
        grid = (batch_size * token_per_batch, num_head_blocks, num_dim_blocks)

        unpad_draft_extend_output_kernel[grid](
            raw_out_ptr=raw_out,
            output_ptr=output,
            num_accept_tokens_ptr=seq_lens_q,
            cumsum_ptr=cu_seqlens_q,
            batch_size=batch_size,
            token_per_batch=token_per_batch,
            tp_q_head_num=tp_q_head_num,
            v_head_dim=v_head_dim,
            BLOCK_SIZE=BLOCK_SIZE,
        )
        return output[:total_tokens, :, :]
```
**EN:** Implements the unpad draft extend output routine used by this attention module.
**CN:** 实现该注意力模块使用的 unpad draft extend output 例程。

### Lines 758-777: method TRTLLMMLABackend._compute_decode_bmm1_scale
```python
    def _compute_decode_bmm1_scale(self, layer: RadixAttention) -> float:
        """BMM1 scale ``q_scale * k_scale * softmax_scale``. k_scale only
        applies when the KV cache stores FP8."""
        q_scale = 1.0
        if self.data_type == torch.float8_e4m3fn:
            k_scale = (
                layer.k_scale_float
                if getattr(layer, "k_scale_float", None) is not None
                else 1.0
            )
        else:
            if getattr(layer, "k_scale_float", None) is not None:
                logger.warning_once(
                    "Checkpoint has k_scale but KV cache dtype is not FP8. "
                    "Ignoring k_scale for BMM1 (k_scale=%.4f, kv_dtype=%s).",
                    layer.k_scale_float,
                    self.data_type,
                )
            k_scale = 1.0
        return q_scale * k_scale * layer.scaling
```
**EN:** Implements the compute decode bmm1 scale routine used by this attention module.
**CN:** 实现该注意力模块使用的 compute decode bmm1 scale 例程。

### Lines 779-814: method TRTLLMMLABackend._run_decode_kernel
```python
    def _run_decode_kernel(
        self,
        query: torch.Tensor,
        kv_cache: torch.Tensor,
        block_tables: torch.Tensor,
        seq_lens: torch.Tensor,
        max_seq_len: int,
        layer: RadixAttention,
    ) -> torch.Tensor:
        """Hook for subclasses to swap the decode/spec-verify kernel."""

        # Scale computation for TRTLLM MLA kernel BMM1 operation:
        # The final BMM1 scale is computed as: q_scale * k_scale * softmax_scale
        # Scale components:
        # - q_scale: Query scaling factor (set to 1.0 for both FP16/FP8 paths)
        # - k_scale: Key scaling factor from model checkpoint. Only applied when KV cache
        #   stores FP8-quantized values, to compensate for the quantization scaling.
        #   For BF16/FP16 KV cache, k_scale must be 1.0 since values are unscaled.
        # - softmax_scale: Attention softmax scaling = 1/sqrt(head_dim), pre-computed as layer.scaling
        bmm1_scale = self._compute_decode_bmm1_scale(layer)
        seq_lens_i32 = (
            seq_lens if seq_lens.dtype == torch.int32 else seq_lens.to(torch.int32)
        )
        return flashinfer.decode.trtllm_batch_decode_with_kv_cache_mla(
            query=query,
            kv_cache=kv_cache,
            workspace_buffer=self.workspace_buffer,
            qk_nope_head_dim=self.qk_nope_head_dim,
            kv_lora_rank=self.kv_lora_rank,
            qk_rope_head_dim=self.qk_rope_head_dim,
            block_tables=block_tables,
            seq_lens=seq_lens_i32,
            max_seq_len=max_seq_len,
            bmm1_scale=bmm1_scale,
            skip_softmax_threshold_scale_factor=envs.SGLANG_SKIP_SOFTMAX_DECODE_THRESHOLD_SCALE_FACTOR.get(),
        )
```
**EN:** Implements the run decode kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 run decode kernel 例程。

### Lines 816-859: method TRTLLMMLABackend._run_prefill_kernel
```python
    def _run_prefill_kernel(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        batch_size: int,
        cum_seq_lens_q: torch.Tensor,
        max_q_len: int,
        seq_lens_kv: torch.Tensor,
        cum_seq_lens_kv: torch.Tensor,
        max_kv_len: int,
        is_causal: bool,
        return_lse: bool,
        out_buffer: torch.Tensor,
        o_sf_scale: float = 1.0,
    ):
        """Hook for subclasses to swap the ragged prefill kernel. Q/K/V arrive
        in model-native dtype; subclasses do any kernel-specific quantization.
        Returns the output tensor or ``(output, lse)`` if ``return_lse``."""
        q_scale = k_scale = v_scale = 1.0
        if self.data_type == torch.float8_e4m3fn:
            q, k, v, k_scale, v_scale = _quantize_fp8_qkv(q, k, v, layer)
        return flashinfer.prefill.trtllm_ragged_attention_deepseek(
            query=q,
            key=k,
            value=v,
            workspace_buffer=self.workspace_buffer,
            batch_size=batch_size,
            window_left=-1,
            enable_pdl=False,
            max_q_len=max_q_len,
            bmm1_scale=q_scale * k_scale * layer.scaling,
            bmm2_scale=v_scale,
            cum_seq_lens_q=cum_seq_lens_q,
            cum_seq_lens_kv=cum_seq_lens_kv,
            seq_lens=seq_lens_kv,
            max_kv_len=max_kv_len,
            is_causal=is_causal,
            return_lse=return_lse,
            o_sf_scale=o_sf_scale,
            out=out_buffer,
            skip_softmax_threshold_scale_factor=envs.SGLANG_SKIP_SOFTMAX_PREFILL_THRESHOLD_SCALE_FACTOR.get(),
        )
```
**EN:** Implements the run prefill kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 run prefill kernel 例程。

### Lines 861-955: method TRTLLMMLABackend.forward_decode
```python
    def forward_decode(
        self,
        q: torch.Tensor,  # q_nope
        k: torch.Tensor,  # k_nope
        v: torch.Tensor,  # not used in this backend
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        cos_sin_cache: Optional[torch.Tensor] = None,
        is_neox: Optional[bool] = False,
        llama_4_scaling: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        """Run forward for decode using TRTLLM MLA kernel."""
        merge_query = q_rope is not None
        if self.data_type == torch.float8_e4m3fn:
            # For FP8 path, we quantize the query and rope parts and merge them into a single tensor
            # Note: rope application in deepseek_v2.py:forward_absorb_prepare is skipped for FP8 decode path of this trtllm_mla backend
            assert all(
                x is not None for x in [q_rope, k_rope, cos_sin_cache]
            ), "For FP8 path and using flashinfer.rope.mla_rope_quantize we need all of q_rope, k_rope and cos_sin_cache to be not None."
            q, k, k_rope = mla_quantize_and_rope_for_fp8(
                q,
                q_rope,
                k.squeeze(1),
                k_rope.squeeze(1),
                forward_batch.positions,
                cos_sin_cache,
                is_neox,
                self.kv_lora_rank,
                self.qk_rope_head_dim,
# ... omitted 51 lines ...
        raw_out = self._run_decode_kernel(
            query=query,
            kv_cache=kv_cache,
            block_tables=metadata.block_kv_indices,
            seq_lens=forward_batch.seq_lens,
            max_seq_len=metadata.max_seq_len_k,
            layer=layer,
        )

        # Reshape output directly without slicing
        output = raw_out.view(-1, layer.tp_q_head_num * layer.v_head_dim)
        return output
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 957-1212: method TRTLLMMLABackend.forward_extend
```python
    def forward_extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        cos_sin_cache: Optional[torch.Tensor] = None,
        is_neox: Optional[bool] = False,
        llama_4_scaling: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        if (
            self.forward_prefill_metadata is not None
            and self.forward_prefill_metadata.fallback_to_flashinfer_impl
        ):
            return super().forward_extend(
                q, k, v, layer, forward_batch, save_kv_cache, q_rope, k_rope
            )

        # TODO refactor to avoid code duplication
        merge_query = q_rope is not None
        if (
            self.data_type == torch.float8_e4m3fn
        ) and forward_batch.forward_mode.is_target_verify():
            # For FP8 path, we quantize the query and rope parts and merge them into a single tensor
            # Note: rope application in deepseek_v2.py:forward_absorb_prepare is skipped for FP8 decode path of this trtllm_mla backend
            assert all(
                x is not None for x in [q_rope, k_rope, cos_sin_cache]
# ... omitted 212 lines ...
                layer=layer,
                batch_size=forward_batch.batch_size,
                cum_seq_lens_q=self.forward_prefill_metadata.cum_seq_lens,
                max_q_len=self.forward_prefill_metadata.max_seq_len,
                seq_lens_kv=self.forward_prefill_metadata.seq_lens,
                cum_seq_lens_kv=self.forward_prefill_metadata.cum_seq_lens,
                max_kv_len=self.forward_prefill_metadata.max_seq_len,
                is_causal=True,
                return_lse=forward_batch.mha_return_lse,
                out_buffer=out,
                o_sf_scale=1.0,
            )
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1215-1217: class TRTLLMMLAMultiStepDraftBackend
```python
class TRTLLMMLAMultiStepDraftBackend(FlashInferMLAMultiStepDraftBackend):
    """Multi-step draft backend for TRT-LLM MLA used by EAGLE."""
```
**EN:** Concrete attention backend that connects trtllmmlamulti step draft backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 trtllmmlamulti step draft backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 1218-1229: method TRTLLMMLAMultiStepDraftBackend.__init__
```python
    def __init__(
        self, model_runner: "ModelRunner", topk: int, speculative_num_steps: int
    ):
        super().__init__(model_runner, topk, speculative_num_steps)

        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i] = TRTLLMMLABackend(
                model_runner,
                skip_prefill=True,
                kv_indptr_buf=self.kv_indptr[i],
                q_indptr_decode_buf=self.q_indptr_decode,
            )
```
**EN:** Initializes the TRTLLMMLAMultiStepDraftBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TRTLLMMLAMultiStepDraftBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

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
- `math`
- `dataclasses.dataclass`
- `typing.TYPE_CHECKING`
- `typing.Optional`
- `typing.Union`
- `torch`
- `triton`
- `triton.language`
- `sglang.jit_kernel.fixup_zero_kv.fixup_zero_kv_rows`
- `sglang.srt.compilation.piecewise_context_manager.is_in_piecewise_cuda_graph`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.attention.flashinfer_mla_backend.FlashInferMLAAttnBackend`
- `sglang.srt.layers.attention.flashinfer_mla_backend.FlashInferMLAMultiStepDraftBackend`
- `sglang.srt.layers.attention.utils.concat_mla_absorb_q_general`
- `sglang.srt.layers.attention.utils.create_flashmla_kv_indices_triton`
- `sglang.srt.layers.attention.utils.get_num_page_per_block_flashmla`
- `sglang.srt.layers.attention.utils.mla_quantize_and_rope_for_fp8`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.quantization.fp8_kernel.scaled_fp8_quant`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.is_flashinfer_available`
- `sglang.srt.utils.is_float4_e2m1fn_x2`
- `flashinfer`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.model_runner.ModelRunner`
- `sglang.srt.speculative.spec_info.SpecInput`
