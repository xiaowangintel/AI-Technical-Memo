# indexer.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/dsv4/indexer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages indexer logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 indexer 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-22: imports
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Any, List, Optional, Tuple

import torch
import torch.nn as nn
import torch.nn.functional as F
import triton
import triton.language as tl

from sglang.jit_kernel.deepseek_v4 import (
    fused_q_indexer_rope_hadamard_quant,
    topk_transform_512,
    topk_transform_512_v2,
)
from sglang.srt.configs.deepseek_v4 import DeepSeekV4Config
from sglang.srt.environ import envs
from sglang.srt.layers.attention.dsv4.compressor import Compressor
from sglang.srt.layers.attention.dsv4.metadata import PagedIndexerMetadata
from sglang.srt.layers.linear import ReplicatedLinear
from sglang.srt.state_capturer.indexer_topk import get_global_indexer_capturer
from sglang.srt.utils import add_prefix, is_hip
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 24-31: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.attention.deepseek_v4_backend import DeepseekV4AttnBackend
    from sglang.srt.layers.attention.dsv4.compressor import (
        CompressorBackendMixin,
    )
    from sglang.srt.layers.quantization import QuantizationConfig
    from sglang.srt.mem_cache.deepseek_v4_memory_pool import DeepSeekV4TokenToKVPool
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 34-39: conditional branch
```python
if is_hip():
    FP8_DTYPE = torch.float8_e4m3fnuz
    FP8_MAX = torch.finfo(FP8_DTYPE).max
else:
    FP8_DTYPE = torch.float8_e4m3fn
    FP8_MAX = torch.finfo(FP8_DTYPE).max
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 42-91: function fp8_paged_mqa_logits_torch
```python
def fp8_paged_mqa_logits_torch(
    q_fp8: torch.Tensor,
    kvcache_fp8: torch.Tensor,
    weight: torch.Tensor,
    seq_lens: torch.Tensor,
    page_table: torch.Tensor,
    deep_gemm_metadata: Any,
    max_seq_len: int,
    clean_logits: bool = True,
) -> torch.Tensor:
    _ = deep_gemm_metadata
    batch_size, _, num_heads, head_dim = q_fp8.shape
    block_size = kvcache_fp8.shape[1]

    assert head_dim == 128, "torch reference impl hardcodes DSV4 indexer head_dim=128"
    assert block_size == 64, "torch reference impl hardcodes block_size=64 cache layout"
    assert q_fp8.shape == (batch_size, 1, num_heads, head_dim)
    assert kvcache_fp8.shape[1:] == (block_size, 1, head_dim + 4)
    assert weight.shape == (batch_size, num_heads)
    assert seq_lens.shape == (batch_size,)
    assert page_table.shape[0] == batch_size
    assert clean_logits == False

    logits = page_table.new_empty((batch_size, max_seq_len), dtype=torch.float32)
    for i in range(batch_size):
        q = q_fp8[i, 0]
        q = q.to(torch.float32)
        q_scale = weight[i]
        seq_len = int(seq_lens[i].item())
        assert seq_len <= max_seq_len
        num_pages = (seq_len + block_size - 1) // block_size
        padded_seq_len = num_pages * block_size
# ... omitted 6 lines ...
        kvcache_value = kvcache_value.to(torch.float32)
        kvcache_scale = kvcache_scale.contiguous()
        kvcache_value = kvcache_value.view(padded_seq_len, head_dim)
        kvcache_scale = kvcache_scale.view(padded_seq_len)
        score = F.linear(kvcache_value, q)
        score = F.relu(score)
        score *= q_scale[None, :]
        score = score.sum(dim=1)
        score *= kvcache_scale
        logits[i, :seq_len] = score[:seq_len]

    return logits
```
**EN:** Implements the fp8 paged mqa logits torch routine used by this attention module.
**CN:** 实现该注意力模块使用的 fp8 paged mqa logits torch 例程。

### Lines 94-184: function topk_transform_512_pytorch_vectorized
```python
def topk_transform_512_pytorch_vectorized(
    scores: torch.Tensor,
    seq_lens: torch.Tensor,
    page_tables: torch.Tensor,
    out_page_indices: torch.Tensor,
    page_size: int,
    out_raw_indices: Optional[torch.Tensor] = None,
) -> None:

    TOPK = 512
    batch_size = scores.shape[0]
    max_seq_len = scores.shape[1]
    device = scores.device

    page_bits = (page_size - 1).bit_length() if page_size > 1 else 0
    page_mask = page_size - 1

    positions = (
        torch.arange(max_seq_len, device=device).unsqueeze(0).expand(batch_size, -1)
    )
    valid_mask = positions < seq_lens.unsqueeze(1)

    masked_scores = scores.clone()
    masked_scores[~valid_mask] = float("-inf")

    actual_k = min(TOPK, max_seq_len)
    _, raw_indices = torch.topk(
        masked_scores, k=actual_k, dim=1, largest=True, sorted=False
    )
    raw_indices = raw_indices.to(torch.int32)

    if actual_k < TOPK:
# ... omitted 47 lines ...

    page_indices = torch.where(
        valid_topk, page_indices, torch.tensor(-1, device=device, dtype=torch.int32)
    )

    out_page_indices.copy_(page_indices)

    if out_raw_indices is not None:
        raw_indices = torch.where(
            valid_topk, raw_indices, torch.tensor(-1, device=device, dtype=torch.int32)
        )
        out_raw_indices.copy_(raw_indices)
```
**EN:** Implements the topk transform 512 pytorch vectorized routine used by this attention module.
**CN:** 实现该注意力模块使用的 topk transform 512 pytorch vectorized 例程。

### Lines 187-204: function _fused_scale_kernel
```python
@triton.jit
def _fused_scale_kernel(
    weight_ptr,
    q_scale_ptr,
    out_ptr,
    numel,
    out_scale,
    BLOCK: tl.constexpr,
):
    pid = tl.program_id(0)
    offs = pid * BLOCK + tl.arange(0, BLOCK)
    mask = offs < numel

    w = tl.load(weight_ptr + offs, mask=mask)
    qs = tl.load(q_scale_ptr + offs, mask=mask)

    acc = w.to(tl.float32) * out_scale * qs.to(tl.float32)
    tl.store(out_ptr + offs, acc.to(out_ptr.dtype.element_ty), mask=mask)
```
**EN:** Implements the fused scale kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused scale kernel 例程。

### Lines 207-227: function fused_scale
```python
def fused_scale(
    weight: torch.Tensor,
    out_scale: float,
    q_scale: torch.Tensor,
) -> torch.Tensor:
    assert weight.is_contiguous() and q_scale.is_contiguous()
    B, H = weight.shape
    numel = B * H
    out_dtype = torch.promote_types(weight.dtype, q_scale.dtype)
    out = torch.empty((B, H, 1), device=weight.device, dtype=out_dtype)
    BLOCK = 1024
    grid = (triton.cdiv(numel, BLOCK),)
    _fused_scale_kernel[grid](
        weight,
        q_scale,
        out,
        numel,
        out_scale,
        BLOCK=BLOCK,
    )
    return out
```
**EN:** Implements the fused scale routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused scale 例程。

### Lines 230-230: class C4IndexerBackendMixin
```python
class C4IndexerBackendMixin:
```
**EN:** Concrete attention backend that connects c4 indexer backend mixin to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 c4 indexer backend mixin 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 231-233: method C4IndexerBackendMixin.__init__
```python
    def __init__(self):
        super().__init__()
        self.debug_use_external_c4_sparse_indices: bool = False
```
**EN:** Initializes the C4IndexerBackendMixin instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 C4IndexerBackendMixin 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 235-283: method C4IndexerBackendMixin._forward_prepare_multi_stream
```python
    def _forward_prepare_multi_stream(
        self,
        x: torch.Tensor,
        q_lora: torch.Tensor,
        c4_indexer: C4Indexer,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        token_to_kv_pool: DeepSeekV4TokenToKVPool,
        alt_streams: Optional[List[torch.cuda.Stream]] = None,
        q_lora_ready: Optional[torch.cuda.Event] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        if TYPE_CHECKING:
            assert isinstance(self, CompressorBackendMixin)

        assert alt_streams is not None
        assert len(alt_streams) >= 2
        current_stream = torch.cuda.current_stream()
        stream_q = alt_streams[0]
        stream_weights = alt_streams[1]

        stream_q.wait_stream(current_stream)
        stream_weights.wait_stream(current_stream)

        self.forward_indexer_compressor(
            x=x,
            forward_batch=forward_batch,
            layer_id=c4_indexer.layer_id,
            compressor=c4_indexer.compressor,
        )
        c4_indexer_kv_cache = token_to_kv_pool.get_index_k_with_scale_buffer(
            layer_id=c4_indexer.layer_id,
        )
# ... omitted 5 lines ...
        with torch.cuda.stream(stream_weights):
            weights = c4_indexer.compute_weights(x, skip_scale=True)
            weights_ready = stream_weights.record_event()

        with torch.cuda.stream(stream_q):
            if q_lora_ready is not None:
                stream_q.wait_event(q_lora_ready)
            stream_q.wait_event(weights_ready)
            q_fp8, weights = c4_indexer.compute_q(q_lora, positions, weights)

        current_stream.wait_stream(stream_q)
        return q_fp8, weights, c4_indexer_kv_cache
```
**EN:** Runs the forward-path logic for forward prepare multi stream, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward prepare multi stream 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 285-308: method C4IndexerBackendMixin._forward_prepare_normal
```python
    def _forward_prepare_normal(
        self,
        x: torch.Tensor,
        q_lora: torch.Tensor,
        c4_indexer: C4Indexer,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        token_to_kv_pool: DeepSeekV4TokenToKVPool,
    ) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        if TYPE_CHECKING:
            assert isinstance(self, CompressorBackendMixin)

        weights = c4_indexer.compute_weights(x, skip_scale=True)
        q_fp8, weights = c4_indexer.compute_q(q_lora, positions, weights)
        self.forward_indexer_compressor(
            x=x,
            forward_batch=forward_batch,
            layer_id=c4_indexer.layer_id,
            compressor=c4_indexer.compressor,
        )
        c4_indexer_kv_cache = token_to_kv_pool.get_index_k_with_scale_buffer(
            layer_id=c4_indexer.layer_id,
        )
        return q_fp8, weights, c4_indexer_kv_cache
```
**EN:** Runs the forward-path logic for forward prepare normal, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward prepare normal 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 310-470: method C4IndexerBackendMixin.forward_c4_indexer
```python
    def forward_c4_indexer(
        self,
        x: torch.Tensor,
        q_lora: torch.Tensor,
        c4_indexer: C4Indexer,
        forward_batch: ForwardBatch,
        alt_streams: Optional[List[torch.cuda.Stream]] = None,
        enable_multi_stream: bool = False,
        q_lora_ready: Optional[torch.cuda.Event] = None,
    ) -> None:
        if forward_batch.forward_mode.is_idle():
            return
        # PREP_IN_CG lazy upgrade: this runs from MQALayer._forward_prepare,
        # before attn_backend.forward() would trigger the upgrade.
        self._maybe_upgrade_forward_metadata()
        token_to_kv_pool = forward_batch.token_to_kv_pool

        if TYPE_CHECKING:
            assert isinstance(token_to_kv_pool, DeepSeekV4TokenToKVPool)
            assert isinstance(self, CompressorBackendMixin)

        metadata = self.forward_metadata
        indexer_metadata = metadata.indexer_metadata
        core_metadata = metadata.core_metadata

        from sglang.srt.layers.attention.deepseek_v4_backend import (
            DSV4AttnMetadata,
        )

        assert isinstance(core_metadata, DSV4AttnMetadata)
        assert isinstance(indexer_metadata, PagedIndexerMetadata)

# ... omitted 117 lines ...
            else:
                core_metadata.c4_sparse_page_indices = (
                    token_to_kv_pool.c4_kv_pool.translate_loc_to_hisparse_device(
                        core_metadata.c4_sparse_page_indices
                    )
                )

        if capture_enabled:
            compress_layer_id = token_to_kv_pool.layer_mapping[
                c4_indexer.layer_id
            ].compress_layer_id
            indexer_capturer.capture(compress_layer_id, raw_indices)
```
**EN:** Runs the forward-path logic for forward c4 indexer, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward c4 indexer 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 473-473: class C4Indexer
```python
class C4Indexer(nn.Module):
```
**EN:** Defines the c4 indexer type and the state it exposes to the rest of the attention stack.
**CN:** 定义 c4 indexer 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 474-520: method C4Indexer.__init__
```python
    def __init__(
        self,
        config: DeepSeekV4Config,
        layer_id: int,
        freqs_cis: torch.Tensor,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_streams: Optional[List[torch.cuda.Stream]] = None,
    ):
        super().__init__()
        self.layer_id = layer_id
        self.dim = config.hidden_size
        self.n_heads = config.index_n_heads
        self.head_dim = config.index_head_dim
        self.rope_head_dim = config.qk_rope_head_dim
        self.q_lora_rank = config.q_lora_rank
        self.softmax_scale = self.head_dim**-0.5
        self.n_local_heads = self.n_heads
        self.wq_b = ReplicatedLinear(
            self.q_lora_rank,
            self.n_heads * self.head_dim,
            bias=False,
            quant_config=quant_config,
            params_dtype=torch.bfloat16,
            prefix=add_prefix("wq_b", prefix),
        )
        self.weights_proj = ReplicatedLinear(
            self.dim,
            self.n_heads,
            bias=False,
            quant_config=None,
            params_dtype=torch.bfloat16,
            prefix=add_prefix("weights_proj", prefix),
        )
        self.compressor = Compressor(
            config,
            self.layer_id,
            True,
            freqs_cis,
            compress_ratio=4,
            head_dim=self.head_dim,
            rotate=True,
            prefix=add_prefix("compressor", prefix),
        )
        self.freqs_cis = freqs_cis
        self.weight_scale: float = self.softmax_scale * self.n_heads**-0.5
        self.alt_streams = alt_streams
```
**EN:** Initializes the C4Indexer instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 C4Indexer 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 522-532: method C4Indexer.compute_q
```python
    def compute_q(
        self,
        q_lora: torch.Tensor,
        positions: torch.Tensor,
        weight: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        q, _ = self.wq_b(q_lora)
        q = q.view(-1, self.n_local_heads, self.head_dim)
        return fused_q_indexer_rope_hadamard_quant(
            q, weight, self.weight_scale, self.freqs_cis, positions
        )
```
**EN:** Implements the compute q routine used by this attention module.
**CN:** 实现该注意力模块使用的 compute q 例程。

### Lines 534-538: method C4Indexer.compute_weights
```python
    def compute_weights(self, x: torch.Tensor, skip_scale=False) -> torch.Tensor:
        out, _ = self.weights_proj(x)
        if not skip_scale:
            out = out * self.weight_scale
        return out
```
**EN:** Implements the compute weights routine used by this attention module.
**CN:** 实现该注意力模块使用的 compute weights 例程。

### Lines 540-558: method C4Indexer.forward
```python
    def forward(
        self,
        x: torch.Tensor,
        q_lora: torch.Tensor,
        forward_batch: ForwardBatch,
        enable_multi_stream: bool = False,
        q_lora_ready: Optional[torch.cuda.Event] = None,
    ) -> None:
        if TYPE_CHECKING:
            assert isinstance(forward_batch.attn_backend, DeepseekV4AttnBackend)
        return forward_batch.attn_backend.forward_c4_indexer(
            x=x,
            q_lora=q_lora,
            forward_batch=forward_batch,
            c4_indexer=self,
            alt_streams=self.alt_streams,
            enable_multi_stream=enable_multi_stream,
            q_lora_ready=q_lora_ready,
        )
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局

## Dependencies / 依赖关系
- `__future__.annotations`
- `typing.TYPE_CHECKING`
- `typing.Any`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `triton`
- `triton.language`
- `sglang.jit_kernel.deepseek_v4.fused_q_indexer_rope_hadamard_quant`
- `sglang.jit_kernel.deepseek_v4.topk_transform_512`
- `sglang.jit_kernel.deepseek_v4.topk_transform_512_v2`
- `sglang.srt.configs.deepseek_v4.DeepSeekV4Config`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.attention.dsv4.compressor.Compressor`
- `sglang.srt.layers.attention.dsv4.metadata.PagedIndexerMetadata`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.state_capturer.indexer_topk.get_global_indexer_capturer`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_hip`
- `sglang.srt.layers.attention.deepseek_v4_backend.DeepseekV4AttnBackend`
- `sglang.srt.layers.attention.dsv4.compressor.CompressorBackendMixin`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.mem_cache.deepseek_v4_memory_pool.DeepSeekV4TokenToKVPool`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.layers.attention.deepseek_v4_backend.DSV4AttnMetadata`
- `sglang.srt.layers.attention.dsv4.tilelang_kernel.tilelang_fp8_paged_mqa_logits`
- `deep_gemm.fp8_paged_mqa_logits`
