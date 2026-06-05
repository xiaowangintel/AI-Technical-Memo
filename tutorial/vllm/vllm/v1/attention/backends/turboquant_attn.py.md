# turboquant_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/turboquant_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: TurboQuant attention backend for vLLM. / 该模块位于 `attention/backends` 子系统，主要围绕 `_build_hadamard`, `_build_hadamard_cached`, `TurboQuantAttentionBackend` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""TurboQuant attention backend for vLLM.

Prefill: Standard scaled dot-product attention on uncompressed K/V,
         then quantize K and store K+V into combined cache slot.
Decode:  Compute TQ attention scores from compressed cache,
         unpack FP16 values, softmax + weighted sum.

Cache layout (no leading 2 dimension):
  (num_blocks, block_size, num_kv_heads, slot_size)
  where slot_size = key_packed_size + value_fp16_size

Per-head per-position slot layout:
  [key_packed (kps bytes) | value_fp16 (D*2 bytes)]
  For turboquant_k3v4_nc head_dim=256: [100 bytes key | 512 bytes value] = 612
"""

import functools
import math
from dataclasses import dataclass
from typing import Any, ClassVar

import torch
import torch.nn.functional as F

from vllm.config import get_current_vllm_config
from vllm.config.cache import CacheDType
from vllm.model_executor.layers.quantization.turboquant.centroids import (
    get_centroids,
)
from vllm.triton_utils import triton
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionImpl,
    AttentionLayer,
    AttentionMetadata,
    AttentionMetadataBuilder,
    AttentionType,
    CommonAttentionMetadata,
    MultipleOf,
)
from vllm.v1.attention.backends.fa_utils import (
    get_flash_attn_version,
    is_flash_attn_varlen_func_available,
)
from vllm.v1.attention.backends.utils import split_decodes_and_prefills
from vllm.v1.attention.ops.triton_turboquant_decode import (
    _tq_full_dequant_kv,
    _use_fp8_e4b15,
    triton_turboquant_decode_attention,
)
from vllm.v1.attention.ops.triton_turboquant_store import triton_turboquant_store
from vllm.v1.worker.workspace import (
    current_workspace_manager,
    is_workspace_manager_initialized,
)

_HAS_FLASH_ATTN = is_flash_attn_varlen_func_available()
if _HAS_FLASH_ATTN:
    from vllm.v1.attention.backends.fa_utils import flash_attn_varlen_func

# Continuation prefill: for small continuation chunks (q_len ≤ threshold),
# use the TQ decode kernel directly instead of full-dequant + flash_attn.
# do_kv_cache_update already stored all tokens to TQ cache, so the decode
# kernel can read them efficiently. This avoids O(cached_len) dequant work
# per continuation, eliminating the O(N²/chunk_size) collapse at long context.
_CONTINUATION_DECODE_THRESHOLD = 128
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `_HAS_FLASH_ATTN`, `_CONTINUATION_DECODE_THRESHOLD`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `_HAS_FLASH_ATTN`, `_CONTINUATION_DECODE_THRESHOLD`。

### `_build_hadamard` function / `_build_hadamard` 函数
```python
def _build_hadamard(d: int, device_str: str) -> torch.Tensor:
    """Orthonormal Hadamard matrix (Sylvester construction), cached per (d, device).

    Precomputed D×D matrix enables matmul-based WHT — single cuBLAS GEMM
    instead of log2(D) butterfly kernel launches. 64KB for D=128.
    """
    # Normalize device string so "cuda" and "cuda:0" hit the same cache entry.
    return _build_hadamard_cached(d, str(torch.device(device_str)))
```
**EN:** This function implements `_build_hadamard` within the module. The docstring frames it as: Orthonormal Hadamard matrix (Sylvester construction), cached per (d, device). Key calls include `_build_hadamard_cached`, `str`, `device`.
**CN:** 该函数会实现 `_build_hadamard`，其作用域位于the module。 关键调用包括 `_build_hadamard_cached`, `str`, `device`。

### `_build_hadamard_cached` function / `_build_hadamard_cached` 函数
```python
@functools.cache
def _build_hadamard_cached(d: int, device_str: str) -> torch.Tensor:
    H = torch.tensor([[1.0]])
    while H.shape[0] < d:
        H = torch.cat([torch.cat([H, H], 1), torch.cat([H, -H], 1)], 0)
    return (H / math.sqrt(d)).to(torch.device(device_str))
```
**EN:** This function implements `_build_hadamard_cached` within the module. Key calls include `tensor`, `to`, `cat`, `device`, `sqrt`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_build_hadamard_cached`，其作用域位于the module。 关键调用包括 `tensor`, `to`, `cat`, `device`, `sqrt`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `TurboQuantAttentionBackend` class / `TurboQuantAttentionBackend` 类
```python
class TurboQuantAttentionBackend(AttentionBackend):
    """Attention backend using TurboQuant KV-cache compression."""

    accept_output_buffer: bool = True
    forward_includes_kv_cache_update: bool = False

    supported_dtypes: ClassVar[list[torch.dtype]] = [
        torch.float16,
        torch.bfloat16,
    ]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "turboquant_k8v4",
        "turboquant_4bit_nc",
        "turboquant_k3v4_nc",
        "turboquant_3bit_nc",
    ]
```
**EN:** Introduces the `TurboQuantAttentionBackend` class on top of `AttentionBackend`. Core methods include `get_name`, `get_supported_kernel_block_sizes`, `supports_attn_type`, `supports_per_head_quant_scales`, `get_impl_cls`, `get_builder_cls`. Docstring signal: Attention backend using TurboQuant KV-cache compression.
**CN:** 这里定义 `TurboQuantAttentionBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_name`, `get_supported_kernel_block_sizes`, `supports_attn_type`, `supports_per_head_quant_scales`, `get_impl_cls`, `get_builder_cls`。

### `TurboQuantAttentionBackend.get_builder_cls` method / `TurboQuantAttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["TurboQuantMetadataBuilder"]:
        return TurboQuantMetadataBuilder
```
**EN:** This method returns or derives a value within `TurboQuantAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TurboQuantAttentionBackend`。

### `TurboQuantAttentionBackend.get_kv_cache_shape` method / `TurboQuantAttentionBackend.get_kv_cache_shape` 方法
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,
        head_size: int,
        cache_dtype_str: str = "turboquant_4bit_nc",
    ) -> tuple[int, ...]:
        """Combined K+V cache shape — no leading 2 dimension.

        Standard attention backends use (2, num_blocks, block_size, num_kv_heads,
        head_dim) with a leading 2 to separate K and V. TurboQuant packs K+V
        into a single interleaved slot per head per position, so the cache is:

            (num_blocks, block_size, num_kv_heads, slot_size_aligned)

        Each slot = [key_packed | value_packed | padding].
        This is safe because TQ has its own get_kv_cache_shape override and
        never shares cache tensors with other backends. Layers that fall back
        to native dtype via kv_cache_dtype_skip_layers get their own
        standard-shaped cache allocation.

        head_size is the model's real head_dim. slot_size_aligned is computed
        from the TQ config to ensure correct cache allocation for all head dims.
        """
        from vllm.model_executor.layers.quantization.turboquant.config import (
            TurboQuantConfig,
        )

        tq_config = TurboQuantConfig.from_cache_dtype(cache_dtype_str, head_size)
        return (num_blocks, block_size, num_kv_heads, tq_config.slot_size_aligned)
```
**EN:** This method returns or derives a value within `TurboQuantAttentionBackend`. The docstring frames it as: Combined K+V cache shape — no leading 2 dimension. Key calls include `from_cache_dtype`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TurboQuantAttentionBackend`。 关键调用包括 `from_cache_dtype`。

### `TurboQuantAttentionBackend.supports_kv_cache_dtype` method / `TurboQuantAttentionBackend.supports_kv_cache_dtype` 方法
```python
    @classmethod
    def supports_kv_cache_dtype(cls, kv_cache_dtype: CacheDType | None) -> bool:
        if kv_cache_dtype is None:
            return False
        return kv_cache_dtype.startswith("turboquant_")
```
**EN:** This method implements `supports_kv_cache_dtype` within `TurboQuantAttentionBackend`. Key calls include `startswith`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `supports_kv_cache_dtype`，其作用域位于`TurboQuantAttentionBackend`。 关键调用包括 `startswith`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TurboQuantMetadata` class / `TurboQuantMetadata` 类
```python
@dataclass
class TurboQuantMetadata(AttentionMetadata):
    """Metadata for TurboQuant attention."""

    seq_lens: torch.Tensor  # (num_reqs,) — total context length per request
    slot_mapping: torch.Tensor  # (num_tokens,) — cache slot for each token
    block_table: torch.Tensor  # (num_reqs, max_num_blocks)
    query_start_loc: torch.Tensor  # (num_reqs + 1,) — cu_seqlens for queries
    num_actual_tokens: int = 0  # actual tokens (excluding padding)
    max_query_len: int = 0  # longest query in batch
    max_seq_len: int = 0  # longest context in batch
    is_prefill: bool = False
    num_decodes: int = 0  # number of decode requests (first in batch)
    num_decode_tokens: int = 0  # tokens from decode requests
    # CPU-resident copies used by the prefill path for per-request iteration
    # without per-step D2H syncs.
    query_start_loc_cpu: torch.Tensor | None = None
    seq_lens_cpu: torch.Tensor | None = None
```
**EN:** Uses `@dataclass` to package related state for `TurboQuantMetadata`. Typical fields include `seq_lens`, `slot_mapping`, `block_table`, `query_start_loc`, `num_actual_tokens`, `max_query_len`.
**CN:** `TurboQuantMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `seq_lens`, `slot_mapping`, `block_table`, `query_start_loc`, `num_actual_tokens`, `max_query_len`。

### `TurboQuantMetadataBuilder` class / `TurboQuantMetadataBuilder` 类
```python
class TurboQuantMetadataBuilder(AttentionMetadataBuilder[TurboQuantMetadata]):
    """Builds TurboQuantMetadata from scheduler output."""

    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.UNIFORM_BATCH
```
**EN:** Introduces the `TurboQuantMetadataBuilder` class on top of `AttentionMetadataBuilder[TurboQuantMetadata]`. Core methods include `__init__`, `build_for_cudagraph_capture`, `build`. Docstring signal: Builds TurboQuantMetadata from scheduler output.
**CN:** 这里定义 `TurboQuantMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[TurboQuantMetadata]`。核心方法包括 `__init__`, `build_for_cudagraph_capture`, `build`。

### `TurboQuantMetadataBuilder.__init__` method / `TurboQuantMetadataBuilder.__init__` 方法
```python
    def __init__(self, kv_cache_spec, layer_names, vllm_config, device):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)
        self._init_reorder_batch_threshold(1, supports_spec_as_decode=False)
```
**EN:** This method initializes the object state within `TurboQuantMetadataBuilder`. Key calls include `__init__`, `_init_reorder_batch_threshold`, `super`.
**CN:** 该方法会初始化对象状态，其作用域位于`TurboQuantMetadataBuilder`。 关键调用包括 `__init__`, `_init_reorder_batch_threshold`, `super`。

### `TurboQuantMetadataBuilder.build_for_cudagraph_capture` method / `TurboQuantMetadataBuilder.build_for_cudagraph_capture` 方法
```python
    def build_for_cudagraph_capture(
        self, common_attn_metadata: CommonAttentionMetadata
    ) -> TurboQuantMetadata:
        attn_metadata = self.build(0, common_attn_metadata)
        # Set seq_lens to 1 so CUDA graph capture is fast
        # (real seq_lens are filled at replay time).
        attn_metadata.seq_lens.fill_(1)
        return attn_metadata
```
**EN:** This method builds derived structures within `TurboQuantMetadataBuilder`. Key calls include `build`, `fill_`.
**CN:** 该方法会构建派生结构，其作用域位于`TurboQuantMetadataBuilder`。 关键调用包括 `build`, `fill_`。

### `TurboQuantMetadataBuilder.build` method / `TurboQuantMetadataBuilder.build` 方法
```python
    def build(self, common_prefix_len, common_attn_metadata, fast_build=False):
        """Build TurboQuantMetadata from common attention metadata."""
        cam = common_attn_metadata

        # With reorder_batch_threshold=1, the model runner guarantees
        # decodes come first in the batch. split_decodes_and_prefills
        # finds the boundary (operates on CPU tensors — no GPU sync).
        assert self.reorder_batch_threshold is not None
        num_decodes, num_prefills, num_decode_tokens, _ = split_decodes_and_prefills(
            cam, decode_threshold=self.reorder_batch_threshold
        )

        return TurboQuantMetadata(
            seq_lens=cam.seq_lens,
            slot_mapping=cam.slot_mapping,
            block_table=cam.block_table_tensor,
            query_start_loc=cam.query_start_loc,
            num_actual_tokens=cam.num_actual_tokens,
            max_query_len=cam.max_query_len,
            max_seq_len=cam.max_seq_len,
            is_prefill=(cam.max_query_len > 1),
            num_decodes=num_decodes,
            num_decode_tokens=num_decode_tokens,
            query_start_loc_cpu=cam.query_start_loc_cpu,
            seq_lens_cpu=cam.seq_lens_cpu_upper_bound,
        )
```
**EN:** This method builds derived structures within `TurboQuantMetadataBuilder`. The docstring frames it as: Build TurboQuantMetadata from common attention metadata. Key calls include `split_decodes_and_prefills`, `TurboQuantMetadata`.
**CN:** 该方法会构建派生结构，其作用域位于`TurboQuantMetadataBuilder`。 关键调用包括 `split_decodes_and_prefills`, `TurboQuantMetadata`。

### `TurboQuantAttentionImpl` class / `TurboQuantAttentionImpl` 类
```python
class TurboQuantAttentionImpl(AttentionImpl["TurboQuantMetadata"]):
    """TurboQuant attention implementation.

    Vectorized PyTorch: batch quantize/store, vectorized bit-unpack
    decode with einsum scores and value gather.
    """

    supports_quant_query_input: bool = False
```
**EN:** Introduces the `TurboQuantAttentionImpl` class on top of `AttentionImpl['TurboQuantMetadata']`. Core methods include `__init__`, `_flash_attn_varlen`, `_ensure_on_device`, `do_kv_cache_update`, `forward`, `_store_kv`. Docstring signal: TurboQuant attention implementation.
**CN:** 这里定义 `TurboQuantAttentionImpl` 类，其基类包括 `AttentionImpl['TurboQuantMetadata']`。核心方法包括 `__init__`, `_flash_attn_varlen`, `_ensure_on_device`, `do_kv_cache_update`, `forward`, `_store_kv`。

### `TurboQuantAttentionImpl.forward` method / `TurboQuantAttentionImpl.forward` 方法
```python
    def forward(
        self,
        layer: AttentionLayer,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: "TurboQuantMetadata",
        output: torch.Tensor | None = None,
        output_scale: torch.Tensor | None = None,
        output_block_scale: torch.Tensor | None = None,
    ) -> torch.Tensor:
        num_tokens = query.shape[0]

        if output is None:
            output = torch.zeros(
                num_tokens,
                self.num_heads * self.head_size,
                dtype=query.dtype,
                device=query.device,
            )

        if attn_metadata is None:
            return output.fill_(0)

        # Slice to actual tokens
        N = attn_metadata.num_actual_tokens
        if N <= 0:
            return output.fill_(0)

        q = query[:N].view(N, self.num_heads, self.head_size)

        # Get TQ buffers, ensure on device (one-time migration).
        # Use Any-typed alias for dynamic _tq_* attrs set by _ensure_on_device.
        tq_layer: Any = layer
        device = q.device
        self._ensure_on_device(tq_layer, device)
        Pi = tq_layer._tq_Pi
        PiT = tq_layer._tq_PiT
        centroids = tq_layer._tq_centroids

        # Compute attention (KV cache was already updated by do_kv_cache_update)
        # With reorder_batch_threshold=1, decodes come first in the batch.
        # num_decodes/num_decode_tokens from metadata give the split point.
        num_decodes = attn_metadata.num_decodes
        num_decode_tokens = attn_metadata.num_decode_tokens

        if not attn_metadata.is_prefill:
            # Pure decode batch — fast path
            attn_out = self._decode_attention(
    # ... omitted for brevity ...
                seq_lens_cpu=attn_metadata.seq_lens_cpu[num_decodes:]
                if attn_metadata.seq_lens_cpu is not None
                else None,
            )
            k = key[:N].view(N, self.num_kv_heads, self.head_size)
            v = value[:N].view(N, self.num_kv_heads, self.head_size)
            attn_out[num_decode_tokens:] = self._prefill_attention(
                q[num_decode_tokens:],
                k[num_decode_tokens:],
                v[num_decode_tokens:],
                kv_cache,
                prefill_meta,
                Pi,
                centroids,
                PiT,
                layer=layer,
            )

        # Write into output buffer: attn_out is (N, Hq, D)
        # output may be 2D (N, Hq*D) or 3D (N, Hq, D)
        if output.ndim == 3:
            output[:N] = attn_out.to(output.dtype)
        else:
            output[:N] = attn_out.reshape(N, -1).to(output.dtype)
        return output
```
**EN:** This method drives the forward-pass computation within `TurboQuantAttentionImpl`. Key calls include `view`, `_ensure_on_device`, `zeros`, `fill_`, `_decode_attention`, `to`. The control flow contains 9 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会驱动前向计算流程，其作用域位于`TurboQuantAttentionImpl`。 关键调用包括 `view`, `_ensure_on_device`, `zeros`, `fill_`, `_decode_attention`, `to`。 控制流包含 9 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `TurboQuantAttentionImpl._prefill_attention` method / `TurboQuantAttentionImpl._prefill_attention` 方法
```python
    def _prefill_attention(
        self,
        query: torch.Tensor,  # (N, Hq, D)
        key: torch.Tensor,  # (N, Hk, D)
        value: torch.Tensor,  # (N, Hk, D)
        kv_cache: torch.Tensor,  # (num_blocks, block_size, Hk, slot_size)
        attn_metadata: TurboQuantMetadata,
        Pi: torch.Tensor,
        centroids: torch.Tensor,
        PiT: torch.Tensor | None = None,
        layer: Any = None,
    ) -> torch.Tensor:
        N, Hq, D = query.shape

        # Fast path: use flash_attn for first-chunk prefills (all K/V in batch).
        # max_query_len == max_seq_len means no request has prior cached KV.
        # Both are Python ints — no GPU sync.
        if _HAS_FLASH_ATTN and attn_metadata.max_query_len == attn_metadata.max_seq_len:
            return self._flash_attn_varlen(
                q=query,
                k=key,
                v=value,
                cu_seqlens_q=attn_metadata.query_start_loc,
                cu_seqlens_k=attn_metadata.query_start_loc,
                max_seqlen_q=attn_metadata.max_query_len,
                max_seqlen_k=attn_metadata.max_query_len,
            )

        # Continuation or no flash_attn: per-request attention.
        # For continuation chunks (seq_len > q_len), we must attend to
        # previously cached K/V from the TQ cache, not just the current
        # chunk's raw K/V.
        Hk = key.shape[1]
        use_gqa = Hk < Hq
        query_start_loc = attn_metadata.query_start_loc
        num_reqs = query_start_loc.shape[0] - 1

        output = torch.zeros(N, Hq, D, device=query.device, dtype=query.dtype)

        # Prefer the CPU-resident copies from the metadata if populated —
        # otherwise `.tolist()` on GPU tensors forces a synchronizing copy.
        if attn_metadata.query_start_loc_cpu is not None:
            qsl = attn_metadata.query_start_loc_cpu.tolist()
        else:
            qsl = query_start_loc.tolist()
        if attn_metadata.seq_lens_cpu is not None:
            seq_lens_list = attn_metadata.seq_lens_cpu.tolist()
        else:
            seq_lens_list = attn_metadata.seq_lens.tolist()

    # ... omitted for brevity ...
                        mse_bits=self.tq_config.key_mse_bits,
                        key_packed_size=self.tq_config.key_packed_size,
                        value_quant_bits=(self.tq_config.effective_value_quant_bits),
                        key_fp8=self.tq_config.key_fp8,
                        norm_correction=self.tq_config.norm_correction,
                        PiT=PiT,
                    )
                else:
                    # Large continuation: dequant cached K/V and use
                    # flash_attn for better throughput.
                    out = self._continuation_prefill(
                        layer,
                        q_seq,
                        k_seq,
                        v_seq,
                        kv_cache,
                        attn_metadata.block_table[i : i + 1],
                        cached_len,
                        seq_len,
                        Pi,
                        centroids,
                    )
                output[q_start:q_end] = out.to(query.dtype)

        return output
```
**EN:** This method implements `_prefill_attention` within `TurboQuantAttentionImpl`. Key calls include `zeros`, `getattr`, `range`, `_flash_attn_varlen`, `tolist`, `hasattr`. It touches state such as `_cu_2`, `_arange_cache`. The control flow contains 9 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_prefill_attention`，其作用域位于`TurboQuantAttentionImpl`。 关键调用包括 `zeros`, `getattr`, `range`, `_flash_attn_varlen`, `tolist`, `hasattr`。 它会读写 `_cu_2`, `_arange_cache` 等状态。 控制流包含 9 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `TurboQuantAttentionImpl._continuation_prefill` method / `TurboQuantAttentionImpl._continuation_prefill` 方法
```python
    def _continuation_prefill(
        self,
        layer: Any,
        query: torch.Tensor,  # (q_len, Hq, D)
        key_chunk: torch.Tensor,  # (q_len, Hk, D)
        val_chunk: torch.Tensor,  # (q_len, Hk, D)
        kv_cache: torch.Tensor,  # (num_blocks, block_size, Hk, slot_size)
        block_table: torch.Tensor,  # (1, max_num_blocks)
        cached_len: int,
        seq_len: int,
        Pi: torch.Tensor,
        centroids: torch.Tensor,
    ) -> torch.Tensor:
        """Handle continuation chunk by dequanting cached K/V from TQ cache.

        Dequants previously cached K/V, concatenates with the current
        chunk's raw K/V, then runs flash_attn with causal masking.
        """
        q_len, Hq, D = query.shape
        Hk = key_chunk.shape[1]
        device = query.device
        block_size = kv_cache.shape[1]
        BLOCK_D = triton.next_power_of_2(D)

        mse_bytes = self._mse_bytes
        val_data_bytes = self._val_data_bytes

        # Dequant cached K/V from TQ cache
        # Allocate slightly over to align to block_size for the grid.
        # Reuse cached buffers to avoid per-call allocation (~16MB at 8K).
        alloc_len = math.ceil(cached_len / block_size) * block_size
        buf_shape = (1, Hk, alloc_len, D)
        # Use WorkspaceManager for dequant buffers.
        # Shared across all layers — saves 60× memory at long context.
        # Required for CUDA Graph capture (per-layer growth incompatible with CG).
        k_buf, v_buf = current_workspace_manager().get_simultaneous(
            (buf_shape, torch.float16),
            (buf_shape, torch.float16),
        )
        # Skip .zero_() — kernel writes all positions up to cached_len,
        # and we only read [:cached_len] afterwards.
        k_cached = k_buf[:, :, :alloc_len, :]
        v_cached = v_buf[:, :, :alloc_len, :]

        grid = (alloc_len, 1 * Hk)
        _tq_full_dequant_kv[grid](
            kv_cache,
            block_table,
            centroids,
            k_cached,
    # ... omitted for brevity ...
                v=v_full,
                cu_seqlens_q=cu_seqlens_q,
                cu_seqlens_k=cu_seqlens_k,
                max_seqlen_q=q_len,
                max_seqlen_k=seq_len,
            )
        else:
            # SDPA fallback: expand KV for GQA, build causal mask
            q_t = query.transpose(0, 1).unsqueeze(0)  # (1, Hq, q_len, D)
            k_t = k_full.transpose(0, 1).unsqueeze(0)  # (1, Hk, seq_len, D)
            v_t = v_full.transpose(0, 1).unsqueeze(0)  # (1, Hk, seq_len, D)
            # Build causal mask: query position p can attend to K position j
            # where j <= cached_len + p (p is 0-indexed within chunk)
            q_pos = torch.arange(q_len, device=device).unsqueeze(1) + cached_len
            k_pos = torch.arange(seq_len, device=device).unsqueeze(0)
            mask = k_pos <= q_pos  # (q_len, seq_len)
            out = F.scaled_dot_product_attention(
                q_t,
                k_t,
                v_t,
                attn_mask=mask,
                scale=self.scale,
                enable_gqa=(Hk < Hq),
            )  # (1, Hq, q_len, D)
            return out[0].transpose(0, 1)  # (q_len, Hq, D)
```
**EN:** This method implements `_continuation_prefill` within `TurboQuantAttentionImpl`. The docstring frames it as: Handle continuation chunk by dequanting cached K/V from TQ cache. Key calls include `next_power_of_2`, `get_simultaneous`, `transpose`, `empty`, `to`, `ceil`. It touches state such as `_cu_2_q`, `_cu_2_k`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_continuation_prefill`，其作用域位于`TurboQuantAttentionImpl`。 关键调用包括 `next_power_of_2`, `get_simultaneous`, `transpose`, `empty`, `to`, `ceil`。 它会读写 `_cu_2_q`, `_cu_2_k` 等状态。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `_build_hadamard`: top-level helper or orchestration entry point. / `_build_hadamard`：顶层辅助函数或编排入口。
- `_build_hadamard_cached`: top-level helper or orchestration entry point. / `_build_hadamard_cached`：顶层辅助函数或编排入口。
- `TurboQuantAttentionBackend`: central class or interface in this module. / `TurboQuantAttentionBackend`：本模块中的核心类或接口。
- `TurboQuantMetadata`: central class or interface in this module. / `TurboQuantMetadata`：本模块中的核心类或接口。
- `TurboQuantMetadataBuilder`: central class or interface in this module. / `TurboQuantMetadataBuilder`：本模块中的核心类或接口。
- `TurboQuantAttentionImpl`: central class or interface in this module. / `TurboQuantAttentionImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `functools`, `math`, `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.cache`, `vllm.model_executor.layers.quantization.turboquant.centroids`, `vllm.triton_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.fa_utils`, `vllm.v1.attention.backends.utils`, `vllm.v1.attention.ops.triton_turboquant_decode`, `vllm.v1.attention.ops.triton_turboquant_store`, `vllm.v1.worker.workspace`, `vllm.model_executor.layers.quantization.turboquant.config`
