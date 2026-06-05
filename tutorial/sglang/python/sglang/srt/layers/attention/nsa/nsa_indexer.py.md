# nsa_indexer.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/nsa/nsa_indexer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages nsa indexer logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 nsa indexer 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-36: imports
```python
from __future__ import annotations

import contextlib
import logging
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Tuple, Union

import torch
from einops import rearrange

from sglang.jit_kernel.fused_store_index_cache import (
    can_use_nsa_fused_store,
    fused_store_index_k_cache,
)
from sglang.srt.environ import envs
from sglang.srt.layers.attention.nsa.utils import (
    aiter_can_use_preshuffle_paged_mqa,
    is_nsa_enable_prefill_cp,
    is_nsa_prefill_cp_in_seq_split,
)
from sglang.srt.layers.dp_attention import attn_tp_all_gather_into_tensor
from sglang.srt.layers.layernorm import LayerNorm
from sglang.srt.layers.quantization.fp8_kernel import fp8_dtype, is_fp8_fnuz
from sglang.srt.layers.utils import MultiPlatformOp
from sglang.srt.state_capturer.indexer_topk import (
    maybe_capture_indexer_topk,
)
from sglang.srt.utils import (
    add_prefix,
    ceil_align,
    get_bool_env_var,
    is_cuda,
    is_gfx95_supported,
    is_hip,
    is_npu,
)
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 38-38: module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 40-40: global
```python
global _use_multi_stream
```
**EN:** Implements supporting module logic that does not fit into a named function or class.
**CN:** 实现不属于命名函数或类的辅助模块逻辑。

### Lines 41-50: module constants
```python
_is_cuda = is_cuda()
_is_hip = is_hip()
_is_npu = is_npu()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
_is_fp8_fnuz = is_fp8_fnuz()
_is_gfx95_supported = is_gfx95_supported()
# Whether the aiter preshuffle paged-MQA path (page_size=64 + Preshuffle=True +
# KVBlockSize=64) can be used. Falls back to the legacy page_size=1 / KVBlockSize=1
# path when the gluon kernel is unavailable (Triton<3.5 and no AOT bundle).
_use_aiter_preshuffle = aiter_can_use_preshuffle_paged_mqa()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 51-56: conditional branch
```python
if _use_aiter and not _use_aiter_preshuffle:
    logger.warning(
        "ROCm NSA indexer: aiter preshuffle paged-MQA path is unavailable "
        "(needs Triton>=3.5.0 or AITER_ENABLE_AOT_GLUON_PA_MQA_LOGITS=1); "
        "falling back to legacy page_size=1 / KVBlockSize=1 path."
    )
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 57-61: conditional branch
```python
if _is_cuda:
    try:
        import deep_gemm
    except ImportError as e:
        deep_gemm = e
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 63-64: conditional branch
```python
if _use_aiter:
    from aiter.ops.cache import indexer_k_quant_and_cache
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 66-68: conditional branch
```python
if is_npu():
    import torch_npu
    from sglang.srt.hardware_backend.npu.utils import get_indexer_weight_stream
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 70-83: imports
```python
from sglang.srt.distributed import (
    get_attn_context_model_parallel_rank,
    get_attn_context_model_parallel_world_size,
)
from sglang.srt.distributed.parallel_state import get_pp_group
from sglang.srt.layers import deep_gemm_wrapper
from sglang.srt.layers.communicator import ScatterMode
from sglang.srt.layers.linear import ReplicatedLinear
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.rotary_embedding import get_rope_wrapper
from sglang.srt.layers.utils.cp_utils import cp_all_gather_rerange_output
from sglang.srt.model_executor.cuda_graph_runner import get_is_capture_mode
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.server_args import get_global_server_args
```
**EN:** Imports neighboring SGLang modules so this file can reuse shared attention abstractions and utilities.
**CN:** 导入相邻的 SGLang 模块，以复用共享的注意力抽象和工具函数。

### Lines 85-85: module constants
```python
_use_ag_after_qlora = envs.SGLANG_USE_AG_AFTER_QLORA.get()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 86-87: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.memory_pool import NSATokenToKVPool
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 90-90: module constants
```python
DUAL_STREAM_TOKEN_THRESHOLD = 1024 if _is_cuda else 0
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 93-93: class BaseIndexerMetadata
```python
class BaseIndexerMetadata(ABC):
```
**EN:** Container class that stores batch-specific metadata and helper methods for base indexer metadata.
**CN:** 该容器类存储批次相关元数据以及 base indexer metadata 所需的辅助方法。

### Lines 94-98: method BaseIndexerMetadata.get_seqlens_int32
```python
    @abstractmethod
    def get_seqlens_int32(self) -> torch.Tensor:
        """
        Return: (batch_size,) int32 tensor
        """
```
**EN:** Computes and returns get seqlens int32 from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get seqlens int32。

### Lines 100-105: method BaseIndexerMetadata.get_page_table_64
```python
    @abstractmethod
    def get_page_table_64(self) -> torch.Tensor:
        """
        Return: (batch_size, num_blocks) int32, page table.
                The page size of the table is 64.
        """
```
**EN:** Computes and returns get page table 64 from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get page table 64。

### Lines 107-112: method BaseIndexerMetadata.get_page_table_1
```python
    @abstractmethod
    def get_page_table_1(self) -> torch.Tensor:
        """
        Return: (batch_size, num_blocks) int32, page table.
                The page size of the table is 1.
        """
```
**EN:** Computes and returns get page table 1 from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get page table 1。

### Lines 114-118: method BaseIndexerMetadata.get_seqlens_expanded
```python
    @abstractmethod
    def get_seqlens_expanded(self) -> torch.Tensor:
        """
        Return: (sum_extend_seq_len,) int32 tensor
        """
```
**EN:** Computes and returns get seqlens expanded from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get seqlens expanded。

### Lines 120-123: method BaseIndexerMetadata.get_indexer_kvcache_range
```python
    def get_indexer_kvcache_range(self) -> Tuple[torch.Tensor, torch.Tensor]:
        """
        Return: (tokens, ), (tokens, ) int32, k_start and k_end in kv cache(token,xxx) for each token.
        """
```
**EN:** Computes and returns get indexer kvcache range from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer kvcache range。

### Lines 125-128: method BaseIndexerMetadata.get_indexer_seq_len_cpu
```python
    def get_indexer_seq_len_cpu(self) -> torch.Tensor:
        """
        Return: seq lens for each batch.
        """
```
**EN:** Computes and returns get indexer seq len cpu from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer seq len cpu。

### Lines 130-133: method BaseIndexerMetadata.get_indexer_seq_len
```python
    def get_indexer_seq_len(self) -> torch.Tensor:
        """
        Return: seq lens for each batch.
        """
```
**EN:** Computes and returns get indexer seq len from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer seq len。

### Lines 135-138: method BaseIndexerMetadata.get_nsa_extend_len_cpu
```python
    def get_nsa_extend_len_cpu(self) -> List[int]:
        """
        Return: extend seq lens for each batch.
        """
```
**EN:** Computes and returns get nsa extend len cpu from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get nsa extend len cpu。

### Lines 140-143: method BaseIndexerMetadata.get_token_to_batch_idx
```python
    def get_token_to_batch_idx(self) -> torch.Tensor:
        """
        Return: batch idx for each token.
        """
```
**EN:** Computes and returns get token to batch idx from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get token to batch idx。

### Lines 145-161: method BaseIndexerMetadata.topk_transform
```python
    @abstractmethod
    def topk_transform(
        self,
        logits: torch.Tensor,
        topk: int,
    ) -> torch.Tensor:
        """
        Perform topk selection on the logits and possibly transform the result.

        NOTE that attention backend may override this function to do some
        transformation, which means the result of this topk_transform may not
        be the topk indices of the input logits.

        Return: Anything, since it will be passed to the attention backend
                for further processing on sparse attention computation.
                Don't assume it is the topk indices of the input logits.
        """
```
**EN:** Implements the topk transform routine used by this attention module.
**CN:** 实现该注意力模块使用的 topk transform 例程。

### Lines 164-175: function rotate_activation
```python
def rotate_activation(x: torch.Tensor) -> torch.Tensor:
    # from sgl_kernel import hadamard_transform
    if _is_hip:
        from fast_hadamard_transform import hadamard_transform
    else:
        from sglang.jit_kernel.hadamard import hadamard_transform

    hidden_size = x.size(-1)
    assert (
        hidden_size & (hidden_size - 1)
    ) == 0, "Hidden size must be a power of 2 for Hadamard transform."
    return hadamard_transform(x, scale=hidden_size**-0.5)
```
**EN:** Implements the rotate activation routine used by this attention module.
**CN:** 实现该注意力模块使用的 rotate activation 例程。

### Lines 178-178: class Indexer
```python
class Indexer(MultiPlatformOp):
```
**EN:** Defines the indexer type and the state it exposes to the rest of the attention stack.
**CN:** 定义 indexer 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 179-258: method Indexer.__init__
```python
    def __init__(
        self,
        hidden_size: int,
        index_n_heads: int,
        index_head_dim: int,
        rope_head_dim: int,
        index_topk: int,
        q_lora_rank: int,
        max_position_embeddings: int,
        rope_theta: float,
        layer_id: int,
        scale_fmt: Optional[str],
        block_size: int = 128,
        rope_scaling: Optional[Dict[str, Any]] = None,
        is_neox_style: bool = True,
        prefix: str = "",
        quant_config: Optional[QuantizationConfig] = None,
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.hidden_size = hidden_size
        self.n_heads = index_n_heads
        self.head_dim = index_head_dim
        self.rope_head_dim = rope_head_dim
        self.index_topk = index_topk
        self.q_lora_rank = q_lora_rank
        self.layer_id = layer_id
        self.alt_stream = alt_stream
        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()
        if self.nsa_enable_prefill_cp:
            self.cp_size = get_attn_context_model_parallel_world_size()
            self.cp_rank = get_attn_context_model_parallel_rank()
# ... omitted 36 lines ...
        self.rotary_emb = get_rope_wrapper(
            rope_head_dim,
            rotary_dim=rope_head_dim,
            max_position=max_position_embeddings,
            base=rope_theta,  # type: ignore
            rope_scaling=rope_scaling,
            is_neox_style=is_neox_style,
            device=get_global_server_args().device,
        )
        self.block_size = block_size
        self.scale_fmt = scale_fmt
        self.softmax_scale = self.head_dim**-0.5
```
**EN:** Initializes the Indexer instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 Indexer 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 260-273: method Indexer._with_real_sm_count
```python
    @contextlib.contextmanager
    def _with_real_sm_count(self):
        # When pipeline parallelism is enabled, each PP rank initiates a recv operation after the _pp_launch_batch
        # request to receive the PP proxy tensor or output from the previous stage, occupying one SM resource.
        # Model execution runs in parallel with the recv operation, so the SMs available to the indexer must be reduced
        # by 1. Currently, the last rank starts the send result + recv request only after waiting for execution results.
        if self.logits_with_pp_recv:
            pp_recv_sm_count = 1
            with deep_gemm_wrapper.configure_deep_gemm_num_sms(
                self.sm_count - pp_recv_sm_count
            ):
                yield
        else:
            yield
```
**EN:** Implements the with real sm count routine used by this attention module.
**CN:** 实现该注意力模块使用的 with real sm count 例程。

### Lines 275-290: method Indexer._weights_proj_bf16_in_fp32_out
```python
    def _weights_proj_bf16_in_fp32_out(
        self, x: Union[torch.Tensor, Tuple[torch.Tensor, ...]]
    ) -> torch.Tensor:
        # aiter (ROCm gfx95): extract the passthrough bf16 tensor from the
        # 3-tuple (fp8, scale, bf16) produced by fused_rms_fp8_group_quant,
        # avoiding an expensive FP8-to-bf16 dequantization.
        if _use_aiter and _is_gfx95_supported and isinstance(x, tuple) and len(x) == 3:
            x = x[2]
        if _is_cuda:
            return torch.mm(x, self.weights_proj.weight.t(), out_dtype=torch.float32)

        weights, _ = self.weights_proj(x)
        if _is_hip:
            # Return bf16; multiplying with q_scale promotes back to fp32.
            return weights
        return weights.float()
```
**EN:** Implements the weights proj bf16 in fp32 out routine used by this attention module.
**CN:** 实现该注意力模块使用的 weights proj bf16 in fp32 out 例程。

### Lines 292-298: method Indexer._project_and_scale_head_gates
```python
    @torch.compile(dynamic=True)
    def _project_and_scale_head_gates(
        self, x: Union[torch.Tensor, Tuple[torch.Tensor, ...]]
    ):
        weights = self._weights_proj_bf16_in_fp32_out(x)
        weights = weights * self.n_heads**-0.5
        return weights
```
**EN:** Implements the project and scale head gates routine used by this attention module.
**CN:** 实现该注意力模块使用的 project and scale head gates 例程。

### Lines 300-307: method Indexer._get_logits_head_gate
```python
    @torch.compile(dynamic=True)
    def _get_logits_head_gate(
        self, x: Union[torch.Tensor, Tuple[torch.Tensor, ...]], q_scale: torch.Tensor
    ):
        weights = self._weights_proj_bf16_in_fp32_out(x)
        weights = weights * self.n_heads**-0.5
        weights = weights.unsqueeze(-1) * q_scale * self.softmax_scale
        return weights
```
**EN:** Implements the get logits head gate routine used by this attention module.
**CN:** 实现该注意力模块使用的 get logits head gate 例程。

### Lines 309-313: method Indexer._apply_q_scale_and_softmax_scale
```python
    @torch.compile(dynamic=True)
    def _apply_q_scale_and_softmax_scale(
        self, weights: torch.Tensor, q_scale: torch.Tensor
    ):
        return weights.unsqueeze(-1) * q_scale * self.softmax_scale
```
**EN:** Implements the apply q scale and softmax scale routine used by this attention module.
**CN:** 实现该注意力模块使用的 apply q scale and softmax scale 例程。

### Lines 315-405: method Indexer._get_q_k_bf16
```python
    def _get_q_k_bf16(
        self,
        q_lora: torch.Tensor,
        x: torch.Tensor,
        positions: torch.Tensor,
        enable_dual_stream: bool,
        forward_batch: ForwardBatch,
    ):
        if enable_dual_stream:
            current_stream = torch.cuda.current_stream()
            self.alt_stream.wait_stream(current_stream)

            with deep_gemm_wrapper.configure_deep_gemm_num_sms(
                self.half_device_sm_count
            ):
                query, _ = self.wq_b(q_lora)
                query = rearrange(query, "l (h d) -> l h d", d=self.head_dim)
                q_rope, _ = torch.split(
                    query,
                    [self.rope_head_dim, self.head_dim - self.rope_head_dim],
                    dim=-1,
                )
            with torch.cuda.stream(self.alt_stream):
                # TODO we should also put DeepGEMM half SM here?
                key, _ = self.wk(x)
                key = self.k_norm(key)

                k_rope, _ = torch.split(
                    key,
                    [self.rope_head_dim, self.head_dim - self.rope_head_dim],
                    dim=-1,
                )
# ... omitted 47 lines ...
            query = rotate_activation(query)
            key = rotate_activation(key)

        # allgather+rerrange
        if forward_batch.attn_cp_metadata is not None and self.nsa_enable_prefill_cp:
            key = cp_all_gather_rerange_output(
                key.contiguous(),
                self.cp_size,
                forward_batch,
                torch.cuda.current_stream(),
            )
        return query, key
```
**EN:** Implements the get q k bf16 routine used by this attention module.
**CN:** 实现该注意力模块使用的 get q k bf16 例程。

### Lines 407-424: method Indexer._get_k_bf16
```python
    def _get_k_bf16(
        self,
        x: torch.Tensor,
        positions: torch.Tensor,
        enable_dual_stream: bool,
    ):
        # Compute only key, skip query
        key, _ = self.wk(x)
        key = self.k_norm(key)
        k_rope, _ = torch.split(
            key, [self.rope_head_dim, self.head_dim - self.rope_head_dim], dim=-1
        )

        _, k_rope = self.rotary_emb(positions, k_rope, k_rope)
        self._update_rope_guarded(key[..., : self.rope_head_dim], k_rope)
        key = rotate_activation(key)

        return key
```
**EN:** Implements the get k bf16 routine used by this attention module.
**CN:** 实现该注意力模块使用的 get k bf16 例程。

### Lines 426-432: method Indexer._update_rope_guarded
```python
    @staticmethod
    def _update_rope_guarded(dst: torch.Tensor, src: torch.Tensor) -> None:
        # On AMD with in-place RoPE kernels, self-aliasing can occur;
        # skip write-back when src/dst tensors point to a single memory.
        if src.data_ptr() == dst.data_ptr():
            return
        dst.copy_(src)
```
**EN:** Implements the update rope guarded routine used by this attention module.
**CN:** 实现该注意力模块使用的 update rope guarded 例程。

### Lines 434-553: method Indexer._get_topk_paged
```python
    def _get_topk_paged(
        self,
        forward_batch: ForwardBatch,
        layer_id: int,
        q_fp8: torch.Tensor,
        weights: torch.Tensor,
        metadata: BaseIndexerMetadata,
    ) -> torch.Tensor:
        if TYPE_CHECKING:
            assert isinstance(forward_batch.token_to_kv_pool, NSATokenToKVPool)

        page_size = forward_batch.token_to_kv_pool.page_size
        # NOTE(dark): blocksize = 64 is hardcoded in deep_gemm
        if _is_hip:
            if _use_aiter_preshuffle:
                assert (
                    page_size % 16 == 0
                ), f"HIP preshuffle requires page_size to be a multiple of 16, got {page_size}"
            else:
                assert (
                    page_size == 1
                ), f"HIP legacy NSA path requires page_size == 1, got {page_size}"
        else:
            assert page_size == 64, "only support page size 64"
        # NOTE(dark): this support extend/decode/decode+graph
        if _is_hip and not _use_aiter_preshuffle:
            block_tables = metadata.get_page_table_1()
        else:
            block_tables = metadata.get_page_table_64()

        max_seq_len = block_tables.shape[1] * page_size
        kv_cache_fp8 = forward_batch.token_to_kv_pool.get_index_k_with_scale_buffer(
# ... omitted 76 lines ...
        topk_result = metadata.topk_transform(logits, self.index_topk)
        # Restore possible padding exist in the hidden states.
        if not _is_hip and q_offset < q_fp8.shape[0]:
            pad_len = q_fp8.shape[0] - q_offset
            padding = torch.full(
                (pad_len, topk_result.shape[1]),
                -1,
                dtype=topk_result.dtype,
                device=topk_result.device,
            )
            topk_result = torch.cat([topk_result, padding], dim=0)
        return topk_result
```
**EN:** Implements the get topk paged routine used by this attention module.
**CN:** 实现该注意力模块使用的 get topk paged 例程。

### Lines 555-572: method Indexer._should_chunk_mqa_logits
```python
    def _should_chunk_mqa_logits(
        self, num_q: int, num_k: int, device: torch.device
    ) -> Tuple[bool, int]:
        """
        Detect whether we need to chunk the MQA logits computation to avoid OOM
        Return: (need_chunk, free_mem)
        """
        # Quick static check for normal batches
        if num_q * num_k < 8_000_000:  # 8M elements ≈ 32MB logits
            return False, 0

        free_mem, total_mem = torch.cuda.mem_get_info(device)
        bytes_per_elem = 4  # float32
        logits_bytes = num_q * num_k * bytes_per_elem

        # Logits should not exceed 50% of free memory or 30% of total memory
        need_chunk = (logits_bytes * 2 > free_mem) or (logits_bytes > total_mem * 0.3)
        return need_chunk, free_mem
```
**EN:** Implements the should chunk mqa logits routine used by this attention module.
**CN:** 实现该注意力模块使用的 should chunk mqa logits 例程。

### Lines 574-754: method Indexer._get_topk_ragged
```python
    def _get_topk_ragged(
        self,
        enable_dual_stream: bool,
        forward_batch: ForwardBatch,
        layer_id: int,
        q_fp8: torch.Tensor,
        weights: torch.Tensor,
        metadata: BaseIndexerMetadata,
    ) -> torch.Tensor:
        if TYPE_CHECKING:
            assert isinstance(forward_batch.token_to_kv_pool, NSATokenToKVPool)

        assert forward_batch.forward_mode.is_extend_without_speculative()

        page_size = forward_batch.token_to_kv_pool.page_size
        if _is_hip:
            if _use_aiter_preshuffle:
                assert (
                    page_size % 16 == 0
                ), f"HIP preshuffle requires page_size to be a multiple of 16, got {page_size}"
            else:
                assert (
                    page_size == 1
                ), f"HIP legacy NSA path requires page_size == 1, got {page_size}"
        else:
            assert page_size == 64, "only support page size 64"

        assert len(weights.shape) == 3
        assert (
            forward_batch.seq_lens_cpu is not None
            and forward_batch.extend_seq_lens_cpu is not None
        )
# ... omitted 137 lines ...
                logits_chunk,
                self.index_topk,
                ks=ks[start:end],
                cu_seqlens_q=cu_seqlens_q_chunk,
                ke_offset=lengths_chunk,
                batch_idx_list=batch_idx_chunk,
                topk_indices_offset_override=topk_offset_chunk,
            )
            topk_result[start:end] = raw_topk_chunk
            start = end

        return topk_result
```
**EN:** Implements the get topk ragged routine used by this attention module.
**CN:** 实现该注意力模块使用的 get topk ragged 例程。

### Lines 756-796: method Indexer._forward_cuda_k_only
```python
    def _forward_cuda_k_only(
        self,
        x: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        layer_id: int,
        act_quant,
        enable_dual_stream: bool,
        metadata: BaseIndexerMetadata,
        return_indices: bool = True,
    ) -> Optional[torch.Tensor]:
        assert forward_batch.forward_mode.is_extend_without_speculative()
        x_meta = x[0] if isinstance(x, tuple) else x

        # Fast path: only compute and store k cache, skip all q and weights ops
        key = self._get_k_bf16(x, positions, enable_dual_stream)

        if not forward_batch.out_cache_loc.is_contiguous():
            forward_batch.out_cache_loc = forward_batch.out_cache_loc.contiguous()

        self._store_index_k_cache(
            forward_batch=forward_batch,
            layer_id=layer_id,
            key=key,
            act_quant=act_quant,
        )

        # MHA doesn't need topk_indices
        if not return_indices:
            return None

        # MLA: use dummy logits with topk kernel's fast path to generate indices
        # When length <= 2048, naive_topk_cuda directly generates [0,1,...,length-1,-1,...]
        seq_lens_expanded = metadata.get_seqlens_expanded()
        dummy_logits = torch.zeros(
            seq_lens_expanded.shape[0],
            self.index_topk,
            dtype=torch.float32,
            device=x_meta.device,
        )
        return metadata.topk_transform(dummy_logits, self.index_topk)
```
**EN:** Runs the forward-path logic for forward cuda k only, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward cuda k only 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 798-945: method Indexer._get_topk_ragged_with_cp
```python
    def _get_topk_ragged_with_cp(
        self,
        forward_batch: ForwardBatch,
        layer_id: int,
        q_fp8: torch.Tensor,
        weights: torch.Tensor,
        metadata: BaseIndexerMetadata,
        kv_len: int,
        actual_seq_q: int,
        cp_index: List[Tuple[int, int, int]] = None,
    ) -> torch.Tensor:
        if TYPE_CHECKING:
            assert isinstance(forward_batch.token_to_kv_pool, NSATokenToKVPool)

        page_size = forward_batch.token_to_kv_pool.page_size
        assert page_size == 64, "only support page size 64"
        assert len(weights.shape) == 3
        weights = weights.squeeze(-1)
        k_fp8_list = []
        k_scale_list = []
        ks_list = []
        ke_offset_list = []
        offset = 0
        actual_seq_q_list = []
        batch_idx_list = []

        block_tables = metadata.get_page_table_64()

        assert (
            forward_batch.seq_lens_cpu is not None
            and forward_batch.extend_seq_lens_cpu is not None
        )
# ... omitted 104 lines ...
            actual_seq_q = torch.tensor([actual_seq_q], dtype=torch.int32).to(
                device="cuda", non_blocking=True
            )
            topk_result = metadata.topk_transform(
                logits,
                self.index_topk,
                ks=ks,
                cu_seqlens_q=actual_seq_q,
                ke_offset=ke_offset,
            )

        return topk_result
```
**EN:** Implements the get topk ragged with cp routine used by this attention module.
**CN:** 实现该注意力模块使用的 get topk ragged with cp 例程。

### Lines 947-1028: method Indexer.forward_indexer
```python
    def forward_indexer(
        self,
        q_fp8: torch.Tensor,
        weights: torch.Tensor,
        forward_batch: ForwardBatch,
        topk: int,
        layer_id: int,
    ) -> Optional[torch.Tensor]:
        if not _is_npu:
            from sglang.srt.layers.attention.nsa.tilelang_kernel import fp8_index

        page_size = forward_batch.token_to_kv_pool.page_size
        assert page_size == 64, "only support page size 64"

        assert len(weights.shape) == 3
        weights = weights.squeeze(-1)

        # logits = deep_gemm.fp8_mqa_logits(q_fp8, kv_fp8, weights, ks, ke)
        k_fp8_list = []
        k_scale_list = []

        topk_indices_list = []

        block_tables = forward_batch.req_to_token_pool.req_to_token[
            forward_batch.req_pool_indices, :
        ]
        strided_indices = torch.arange(
            0, block_tables.shape[-1], page_size, device="cuda"
        )
        block_tables = block_tables[:, strided_indices] // page_size

        q_len_start = 0
# ... omitted 38 lines ...

            pad_len = ceil_align(topk_indices.shape[-1], 2048) - topk_indices.shape[-1]
            topk_indices = torch.nn.functional.pad(
                topk_indices, (0, pad_len), "constant", -1
            )

            topk_indices_list.append(topk_indices)

            q_len_start = q_len_end

        topk_indices = torch.cat(topk_indices_list, dim=0)
        return topk_indices
```
**EN:** Runs the forward-path logic for forward indexer, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward indexer 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1030-1104: method Indexer._store_index_k_cache
```python
    def _store_index_k_cache(
        self,
        forward_batch: ForwardBatch,
        layer_id: int,
        key: torch.Tensor,
        *,
        act_quant=None,  # fallback only
    ) -> None:
        """
        Store NSA indexer K cache for current step.

        Preferred: fused_store_index_k_cache(key, cache, out_cache_loc, page_size)
        Fallback : act_quant(key) + token_to_kv_pool.set_index_k_scale_buffer(...)
        """

        # Fast path: JIT fused store (CUDA, page_size=64, non-fnuz)
        if (
            _is_cuda
            and (not _is_fp8_fnuz)
            and can_use_nsa_fused_store(
                key.dtype,
                forward_batch.out_cache_loc.dtype,
                forward_batch.token_to_kv_pool.page_size,
            )
        ):
            # NOTE: wrapper already normalizes shape/contiguity and asserts dtypes.
            buf = forward_batch.token_to_kv_pool.get_index_k_with_scale_buffer(
                layer_id=layer_id
            )
            fused_store_index_k_cache(
                key,
                buf,
# ... omitted 31 lines ...
        k_fp8, k_scale = act_quant(key, self.block_size, self.scale_fmt)

        out_loc = forward_batch.out_cache_loc
        if not out_loc.is_contiguous():
            out_loc = out_loc.contiguous()

        forward_batch.token_to_kv_pool.set_index_k_scale_buffer(
            layer_id=layer_id,
            loc=out_loc,
            index_k=k_fp8,
            index_k_scale=k_scale,
        )
```
**EN:** Implements the store index k cache routine used by this attention module.
**CN:** 实现该注意力模块使用的 store index k cache 例程。

### Lines 1106-1117: method Indexer.forward_xpu
```python
    def forward_xpu(
        self,
        x: torch.Tensor,
        q_lora: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        layer_id: int,
        return_indices: bool = True,
    ) -> Optional[torch.Tensor]:
        return self.forward_cuda(
            x, q_lora, positions, forward_batch, layer_id, return_indices
        )
```
**EN:** Runs the forward-path logic for forward xpu, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward xpu 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1119-1352: method Indexer.forward_cuda
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        q_lora: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        layer_id: int,
        return_indices: bool = True,
    ) -> Optional[torch.Tensor]:
        if _is_hip:
            from sglang.srt.layers.attention.nsa.tilelang_kernel import act_quant
        elif not _is_npu:
            from sglang.srt.layers.attention.nsa.triton_kernel import act_quant

        if TYPE_CHECKING:
            assert isinstance(forward_batch.token_to_kv_pool, NSATokenToKVPool)

        # When upstream uses fused FP8 RMSNorm+quant, activations may be passed as
        # a tuple like (x_fp8, x_scale[, y]). Use `x_meta` for shape/device queries.
        x_meta = x[0] if isinstance(x, tuple) else x

        metadata = forward_batch.attn_backend.get_indexer_metadata(
            layer_id, forward_batch
        )

        enable_dual_stream = (
            self.alt_stream is not None
            and get_is_capture_mode()
            and q_lora.shape[0] > 0
            and q_lora.shape[0] <= DUAL_STREAM_TOKEN_THRESHOLD
        )

# ... omitted 190 lines ...
                        weights,
                        metadata,
                    )
        else:
            topk_result = self.forward_indexer(
                q_fp8.contiguous(),
                weights,
                forward_batch,
                topk=self.index_topk,
                layer_id=layer_id,
            )
        return maybe_capture_indexer_topk(layer_id, topk_result)
```
**EN:** Runs the forward-path logic for forward cuda, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward cuda 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1354-1633: method Indexer.forward_npu
```python
    def forward_npu(
        self,
        x: torch.Tensor,
        q_lora: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        layer_id: int,
        layer_scatter_modes=None,
        dynamic_scale: torch.Tensor = None,
    ) -> torch.Tensor:
        if forward_batch.attn_backend.forward_metadata.seq_lens_cpu_int is None:
            actual_seq_lengths_kv = forward_batch.attn_backend.forward_metadata.seq_lens
        else:
            actual_seq_lengths_kv = (
                forward_batch.attn_backend.forward_metadata.seq_lens_cpu_int
            )
        is_prefill = (
            forward_batch.forward_mode.is_extend()
            and not forward_batch.forward_mode.is_draft_extend_v2()
            and not forward_batch.forward_mode.is_target_verify()
            and not forward_batch.forward_mode.is_draft_extend()
        )

        bs = q_lora.shape[0]

        if self.rotary_emb.is_neox_style:
            if not hasattr(forward_batch, "npu_indexer_sin_cos_cache"):
                cos_sin = self.rotary_emb.cos_sin_cache[positions]
                cos, sin = cos_sin.chunk(2, dim=-1)
                cos = cos.repeat(1, 2).view(-1, 1, 1, self.rope_head_dim)
                sin = sin.repeat(1, 2).view(-1, 1, 1, self.rope_head_dim)
                forward_batch.npu_indexer_sin_cos_cache = (sin, cos)
# ... omitted 236 lines ...
                weights=weights,
                actual_seq_lengths_query=actual_seq_lengths_q.to(torch.int32),
                actual_seq_lengths_key=actual_seq_lengths_kv.to(k.device).to(
                    torch.int32
                ),
                block_table=block_table,
                layout_query="TND",
                layout_key="PA_BSND",
                sparse_count=self.index_topk,
                sparse_mode=3,
            )
            return topk_indices[0]
```
**EN:** Runs the forward-path logic for forward npu, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward npu 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 1635-1688: method Indexer.do_npu_cp_balance_indexer
```python
    def do_npu_cp_balance_indexer(
        self,
        q,
        past_key_states,
        indexer_weights,
        actual_seq_lengths_q,
        actual_seq_lengths_kv,
        block_table,
    ):
        q_prev, q_next = torch.split(q, (q.size(0) + 1) // 2, dim=0)
        weights_prev, weights_next = None, None
        if indexer_weights is not None:
            weights_prev, weights_next = torch.split(
                indexer_weights, (indexer_weights.size(0) + 1) // 2, dim=0
            )
            weights_prev = weights_prev.contiguous().view(-1, weights_prev.shape[-1])
            weights_next = weights_next.contiguous().view(-1, weights_next.shape[-1])

        actual_seq_lengths_q_prev, actual_seq_lengths_q_next = actual_seq_lengths_q
        actual_seq_lengths_kv_prev, actual_seq_lengths_kv_next = actual_seq_lengths_kv

        topk_indices_prev = torch_npu.npu_lightning_indexer(
            query=q_prev,
            key=past_key_states,
            weights=weights_prev,
            actual_seq_lengths_query=actual_seq_lengths_q_prev.to(
                device=q.device, dtype=torch.int32
            ),
            actual_seq_lengths_key=actual_seq_lengths_kv_prev.to(
                device=q.device, dtype=torch.int32
            ),
            block_table=block_table,
# ... omitted 10 lines ...
                device=q.device, dtype=torch.int32
            ),
            actual_seq_lengths_key=actual_seq_lengths_kv_next.to(
                device=q.device, dtype=torch.int32
            ),
            block_table=block_table,
            layout_query="TND",
            layout_key="PA_BSND",
            sparse_count=self.index_topk,
            sparse_mode=3,
        )
        return topk_indices_prev[0], topk_indices_next[0]
```
**EN:** Implements the do npu cp balance indexer routine used by this attention module.
**CN:** 实现该注意力模块使用的 do npu cp balance indexer 例程。

### Lines 1691-1704: function scattered_to_tp_attn_full
```python
def scattered_to_tp_attn_full(
    hidden_states: torch.Tensor,
    forward_batch,
) -> torch.Tensor:
    hidden_states, local_hidden_states = (
        torch.empty(
            (forward_batch.input_ids.shape[0], hidden_states.shape[1]),
            dtype=hidden_states.dtype,
            device=hidden_states.device,
        ),
        hidden_states,
    )
    attn_tp_all_gather_into_tensor(hidden_states, local_hidden_states.contiguous())
    return hidden_states
```
**EN:** Implements the scattered to tp attn full routine used by this attention module.
**CN:** 实现该注意力模块使用的 scattered to tp attn full 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局

## Dependencies / 依赖关系
- `__future__.annotations`
- `contextlib`
- `logging`
- `abc.ABC`
- `abc.abstractmethod`
- `typing.TYPE_CHECKING`
- `typing.Any`
- `typing.Dict`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `torch`
- `einops.rearrange`
- `sglang.jit_kernel.fused_store_index_cache.can_use_nsa_fused_store`
- `sglang.jit_kernel.fused_store_index_cache.fused_store_index_k_cache`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.attention.nsa.utils.aiter_can_use_preshuffle_paged_mqa`
- `sglang.srt.layers.attention.nsa.utils.is_nsa_enable_prefill_cp`
- `sglang.srt.layers.attention.nsa.utils.is_nsa_prefill_cp_in_seq_split`
- `sglang.srt.layers.dp_attention.attn_tp_all_gather_into_tensor`
- `sglang.srt.layers.layernorm.LayerNorm`
- `sglang.srt.layers.quantization.fp8_kernel.fp8_dtype`
- `sglang.srt.layers.quantization.fp8_kernel.is_fp8_fnuz`
- `sglang.srt.layers.utils.MultiPlatformOp`
- `sglang.srt.state_capturer.indexer_topk.maybe_capture_indexer_topk`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.ceil_align`
- `sglang.srt.utils.get_bool_env_var`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_gfx95_supported`
- `sglang.srt.utils.is_hip`
- `sglang.srt.utils.is_npu`
- `sglang.srt.distributed.get_attn_context_model_parallel_rank`
- `sglang.srt.distributed.get_attn_context_model_parallel_world_size`
- `sglang.srt.distributed.parallel_state.get_pp_group`
- `sglang.srt.layers.deep_gemm_wrapper`
- `sglang.srt.layers.communicator.ScatterMode`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.rotary_embedding.get_rope_wrapper`
- `sglang.srt.layers.utils.cp_utils.cp_all_gather_rerange_output`
- `sglang.srt.model_executor.cuda_graph_runner.get_is_capture_mode`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.server_args.get_global_server_args`
- `aiter.ops.cache.indexer_k_quant_and_cache`
- `torch_npu`
- `sglang.srt.hardware_backend.npu.utils.get_indexer_weight_stream`
- `sglang.srt.mem_cache.memory_pool.NSATokenToKVPool`
- `deep_gemm`
- `fast_hadamard_transform.hadamard_transform`
- `sglang.jit_kernel.hadamard.hadamard_transform`
- `aiter.ops.triton.pa_mqa_logits.deepgemm_fp8_paged_mqa_logits`
- `sglang.srt.layers.attention.nsa.tilelang_kernel.fp8_index`
- `sglang.srt.layers.attention.nsa.tilelang_kernel.act_quant`
- `sglang.srt.layers.attention.nsa.triton_kernel.act_quant`
- `aiter.ops.triton.fp8_mqa_logits.fp8_mqa_logits`
