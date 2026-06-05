# indexer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/indexer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_prepare_uniform_decode_kernel`, `split_indexer_prefill_chunks`, `DeepseekV32IndexerBackend` for the V1 `attention/backends/mla` subsystem. / 为 V1 的 `attention/backends/mla` 子系统实现 `_prepare_uniform_decode_kernel`, `split_indexer_prefill_chunks`, `DeepseekV32IndexerBackend`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from dataclasses import dataclass

import torch

import vllm.envs as envs
from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.deep_gemm import (
    get_paged_mqa_logits_metadata,
    has_deep_gemm,
)
from vllm.utils.math_utils import cdiv
from vllm.utils.platform_utils import num_compute_units
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionMetadataBuilder,
    CommonAttentionMetadata,
    MultipleOf,
)
from vllm.v1.attention.backends.mla.compressor_utils import get_compressed_slot_mapping
from vllm.v1.attention.backends.utils import (
    split_decodes_and_prefills,
)
from vllm.v1.kv_cache_interface import AttentionSpec, MLAAttentionSpec
from vllm.v1.worker.cp_utils import get_total_cp_world_size

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_prepare_uniform_decode_kernel` function / `_prepare_uniform_decode_kernel` 函数
```python
@triton.jit
def _prepare_uniform_decode_kernel(
    seq_lens_ptr,
    decode_seq_lens_ptr,
    block_table_ptr,
    block_table_stride,
    expanded_block_table_ptr,
    expanded_bt_stride,
    decode_lens_ptr,
    max_decode_len,
    BLOCK_SIZE: tl.constexpr,
):
    idx = tl.program_id(0)
    req_id = idx // max_decode_len
    local_idx = idx % max_decode_len

    # Compute number of KVs attended to by this token.
    seq_len = tl.load(seq_lens_ptr + req_id)
    per_token_seq_len = seq_len - max_decode_len + local_idx + 1
    tl.store(decode_seq_lens_ptr + idx, per_token_seq_len)

    # Copy block table row.
    src = block_table_ptr + req_id * block_table_stride
    dst = expanded_block_table_ptr + idx * expanded_bt_stride
    for i in tl.range(0, expanded_bt_stride, BLOCK_SIZE):
        off = i + tl.arange(0, BLOCK_SIZE)
        mask = off < expanded_bt_stride
        src_block = tl.load(src + off, mask=mask)
        tl.store(dst + off, src_block, mask=mask)

    # All reqs now have decode_len = 1.
    tl.store(decode_lens_ptr + idx, 1)
```
**EN:** This function implements `_prepare_uniform_decode_kernel` within the module. Key calls include `program_id`, `load`, `store`, `range`, `arange`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_prepare_uniform_decode_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `store`, `range`, `arange`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `split_indexer_prefill_chunks` function / `split_indexer_prefill_chunks` 函数
```python
def split_indexer_prefill_chunks(
    seq_lens_cpu: torch.Tensor,
    query_lens_cpu: torch.Tensor,
    workspace_size: int,
    max_logits_bytes: int,
    request_offset: int = 0,
) -> list[tuple[slice, slice]]:
    """
    Split prefill requests into chunks for the sparse indexer, respecting:
    - N constraint: total_seq_lens <= workspace_size (existing O(N) workspace)
    - Logits constraint: M * N * 4 <= max_logits_bytes

    When a single request-level chunk still exceeds the logits budget,
    sub-chunks on the query dimension (M) to bound peak memory.

    Returns list of (req_slice, query_slice) tuples.
    """
    chunks: list[tuple[slice, slice]] = []
    n = len(seq_lens_cpu)
    max_logits_elems = max_logits_bytes // 4
    end = 0

    while end < n:
        start, chunk_m, chunk_n = end, 0, 0

        while end < n:
            q, s = query_lens_cpu[end].item(), seq_lens_cpu[end].item()
            new_m, new_n = chunk_m + q, chunk_n + s
            if new_n <= workspace_size and new_m * new_n <= max_logits_elems:
                chunk_m, chunk_n = new_m, new_n
                end += 1
            else:
                break

        # A single request can exceed the budget, requiring sub-chunking
        # on the query dimension.
        if end == start:
            chunk_m, chunk_n = query_lens_cpu[end].item(), seq_lens_cpu[end].item()
            end += 1

        req_slice = slice(start + request_offset, end + request_offset)
        max_q = max(1, max_logits_elems // chunk_n) if chunk_n > 0 else chunk_m
        for q_off in range(0, chunk_m, max_q):
            sub_m = min(max_q, chunk_m - q_off)
            chunks.append((req_slice, slice(q_off, q_off + sub_m)))

    return chunks
```
**EN:** This function implements `split_indexer_prefill_chunks` within the module. The docstring frames it as: Split prefill requests into chunks for the sparse indexer, respecting: - N constraint: total_seq_lens <= workspace_size (existing O(N) workspace) - Logits constraint: M * N * 4 <= max_logits_bytes When a single request-level chunk still exceeds the logits budget, sub-chunks on the query dimension (M) to bound peak memory. Key calls include `len`, `slice`, `range`, `max`, `min`, `append`. The control flow contains 3 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `split_indexer_prefill_chunks`，其作用域位于the module。 关键调用包括 `len`, `slice`, `range`, `max`, `min`, `append`。 控制流包含 3 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekV32IndexerBackend` class / `DeepseekV32IndexerBackend` 类
```python
class DeepseekV32IndexerBackend(AttentionBackend):
```
**EN:** Introduces the `DeepseekV32IndexerBackend` class on top of `AttentionBackend`. Core methods include `get_name`, `get_supported_kernel_block_sizes`, `get_supported_head_sizes`, `get_builder_cls`, `get_kv_cache_shape`, `get_kv_cache_stride_order`.
**CN:** 这里定义 `DeepseekV32IndexerBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_name`, `get_supported_kernel_block_sizes`, `get_supported_head_sizes`, `get_builder_cls`, `get_kv_cache_shape`, `get_kv_cache_stride_order`。

### `DeepseekV32IndexerBackend.get_builder_cls` method / `DeepseekV32IndexerBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["DeepseekV32IndexerMetadataBuilder"]:
        return DeepseekV32IndexerMetadataBuilder
```
**EN:** This method returns or derives a value within `DeepseekV32IndexerBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekV32IndexerBackend`。

### `DeepseekV32IndexerBackend.get_kv_cache_shape` method / `DeepseekV32IndexerBackend.get_kv_cache_shape` 方法
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,
        head_size: int,
        cache_dtype_str: str = "auto",
    ) -> tuple[int, ...]:
        assert num_kv_heads == 1
        return (num_blocks, block_size, head_size)
```
**EN:** This method returns or derives a value within `DeepseekV32IndexerBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekV32IndexerBackend`。

### `DeepseekV32IndexerBackend.get_kv_cache_stride_order` method / `DeepseekV32IndexerBackend.get_kv_cache_stride_order` 方法
```python
    @staticmethod
    def get_kv_cache_stride_order(
        include_num_layers_dimension: bool = False,
    ) -> tuple[int, ...]:
        if include_num_layers_dimension:
            # DeepseekV32Indexer kernels do not support cross-layer
            # KV cache layout. Identity permutation keeps num_layers
            # first, signaling incompatibility.
            return (0, 1, 2, 3)
        return (0, 1, 2)
```
**EN:** This method returns or derives a value within `DeepseekV32IndexerBackend`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekV32IndexerBackend`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DeepseekV4IndexerBackend` class / `DeepseekV4IndexerBackend` 类
```python
class DeepseekV4IndexerBackend(DeepseekV32IndexerBackend):
```
**EN:** Introduces the `DeepseekV4IndexerBackend` class on top of `DeepseekV32IndexerBackend`. Core methods include `get_name`, `get_supported_kernel_block_sizes`.
**CN:** 这里定义 `DeepseekV4IndexerBackend` 类，其基类包括 `DeepseekV32IndexerBackend`。核心方法包括 `get_name`, `get_supported_kernel_block_sizes`。

### `DeepseekV4IndexerBackend.get_name` method / `DeepseekV4IndexerBackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "DEEPSEEK_V4_INDEXER"
```
**EN:** This method returns or derives a value within `DeepseekV4IndexerBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekV4IndexerBackend`。

### `DeepseekV4IndexerBackend.get_supported_kernel_block_sizes` method / `DeepseekV4IndexerBackend.get_supported_kernel_block_sizes` 方法
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        return [256]
```
**EN:** This method returns or derives a value within `DeepseekV4IndexerBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`DeepseekV4IndexerBackend`。

### `DeepseekV32IndexerPrefillChunkMetadata` class / `DeepseekV32IndexerPrefillChunkMetadata` 类
```python
@dataclass
class DeepseekV32IndexerPrefillChunkMetadata:
    block_table: torch.Tensor
    cu_seqlen_ks: torch.Tensor
    cu_seqlen_ke: torch.Tensor
    cu_seq_lens: torch.Tensor
    token_to_seq: torch.Tensor
    total_seq_lens: int
    token_start: int
    token_end: int
    num_reqs: int
    skip_kv_gather: bool = False
```
**EN:** Uses `@dataclass` to package related state for `DeepseekV32IndexerPrefillChunkMetadata`. Typical fields include `block_table`, `cu_seqlen_ks`, `cu_seqlen_ke`, `cu_seq_lens`, `token_to_seq`, `total_seq_lens`.
**CN:** `DeepseekV32IndexerPrefillChunkMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `block_table`, `cu_seqlen_ks`, `cu_seqlen_ke`, `cu_seq_lens`, `token_to_seq`, `total_seq_lens`。

### `DeepseekV32IndexerPrefillMetadata` class / `DeepseekV32IndexerPrefillMetadata` 类
```python
@dataclass
class DeepseekV32IndexerPrefillMetadata:
    chunks: list[DeepseekV32IndexerPrefillChunkMetadata]
```
**EN:** Uses `@dataclass` to package related state for `DeepseekV32IndexerPrefillMetadata`. Typical fields include `chunks`.
**CN:** `DeepseekV32IndexerPrefillMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `chunks`。

### `DeepSeekV32IndexerDecodeMetadata` class / `DeepSeekV32IndexerDecodeMetadata` 类
```python
@dataclass
class DeepSeekV32IndexerDecodeMetadata:
    block_table: torch.Tensor
    # seq_lens: per-token effective context lengths.
    #   - flatten path / plain decode: 1D (batch_size,)
    #   - native MTP path: 2D (B, next_n) where [b,j] = L_b - next_n + j + 1
    # Both fp8_fp4_paged_mqa_logits and the topk kernels accept both shapes.
    seq_lens: torch.Tensor
    decode_lens: torch.Tensor
    requires_padding: bool
    schedule_metadata: torch.Tensor
```
**EN:** Uses `@dataclass` to package related state for `DeepSeekV32IndexerDecodeMetadata`. Typical fields include `block_table`, `seq_lens`, `decode_lens`, `requires_padding`, `schedule_metadata`.
**CN:** `DeepSeekV32IndexerDecodeMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `block_table`, `seq_lens`, `decode_lens`, `requires_padding`, `schedule_metadata`。

### `DeepseekV32IndexerMetadata` class / `DeepseekV32IndexerMetadata` 类
```python
@dataclass
class DeepseekV32IndexerMetadata:
    # FIXME (zyongye)
    # hacky way to access the data now, need to be in chunked meta
    seq_lens: torch.Tensor
    max_seq_len: int
    slot_mapping: torch.Tensor

    # New for MLA (compared to FlashAttention)
    # For handling prefill decode split
    num_decodes: int
    num_decode_tokens: int
    num_prefills: int
    num_prefill_tokens: int

    decode: DeepSeekV32IndexerDecodeMetadata | None = None
    prefill: DeepseekV32IndexerPrefillMetadata | None = None
```
**EN:** Uses `@dataclass` to package related state for `DeepseekV32IndexerMetadata`. Typical fields include `seq_lens`, `max_seq_len`, `slot_mapping`, `num_decodes`, `num_decode_tokens`, `num_prefills`.
**CN:** `DeepseekV32IndexerMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `seq_lens`, `max_seq_len`, `slot_mapping`, `num_decodes`, `num_decode_tokens`, `num_prefills`。

### `get_max_prefill_buffer_size` function / `get_max_prefill_buffer_size` 函数
```python
def get_max_prefill_buffer_size(vllm_config: VllmConfig):
    max_model_len = vllm_config.model_config.max_model_len
    # NOTE(Chen): 40 is a magic number for controlling the prefill buffer size.
    # Each entry is 128 fp8 bytes and 4 scale bytes for a total of 132 bytes.
    # The flashmla_sparse backend uses a workspace size of 5 * max_model_len.
    # The memory usage of the workspace there is 576 * 2 bytes; so we size this as
    # (576 * 2 // 132) * 5 = 40 to maximize this workspace size while still fitting
    # within the flashmla_sparse workspace.
    # For DeepSeek-V3.2, the max_model_len is 163840.
    #   40 * 163840 * 132 = 865075200 bytes = 825 MB
    return max_model_len * 40
```
**EN:** This function returns or derives a value within the module.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。

### `DeepseekV32IndexerMetadataBuilder` class / `DeepseekV32IndexerMetadataBuilder` 类
```python
class DeepseekV32IndexerMetadataBuilder(AttentionMetadataBuilder):
    reorder_batch_threshold: int = 1
    natively_supported_next_n_fp4: list[int] = [1, 2]
    # TODO (matt): integrate kernel with next_n = 4 support
```
**EN:** Introduces the `DeepseekV32IndexerMetadataBuilder` class on top of `AttentionMetadataBuilder`. Core methods include `get_cudagraph_support`, `__init__`, `_prepare_decode_tensors`, `build`.
**CN:** 这里定义 `DeepseekV32IndexerMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder`。核心方法包括 `get_cudagraph_support`, `__init__`, `_prepare_decode_tensors`, `build`。

### `DeepseekV32IndexerMetadataBuilder.__init__` method / `DeepseekV32IndexerMetadataBuilder.__init__` 方法
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        scheduler_config = self.vllm_config.scheduler_config
        # NOTE(Chen):an estimated max size of flattened_kv. Need to double check.
        self.max_prefill_buffer_size = get_max_prefill_buffer_size(self.vllm_config)
        self.num_speculative_tokens = (
            self.vllm_config.speculative_config.num_speculative_tokens
            if self.vllm_config.speculative_config
            else 0
        )
        self.use_fp4_indexer_cache = (
            self.vllm_config.attention_config.use_fp4_indexer_cache
        )

        assert (
            current_platform.is_device_capability_family(100)
            or not self.use_fp4_indexer_cache
        ), (
            "use_fp4_indexer_cache requires Blackwell datacenter GPUs "
            "(sm_10x, e.g. B200/GB200); sm_120 (consumer Blackwell) and "
            "earlier architectures are not supported."
        )

        next_n = self.num_speculative_tokens + 1
        self.reorder_batch_threshold += self.num_speculative_tokens
        # NOTE(zyongye) fp4 indexer cache only natively supports next_n in
        # natively_supported_next_n_fp4; for other next_n values we fall back
        # to the flattening path. Outside the SM100 datacenter family the FP8
        # paged MQA logits kernel has the same [1, 2] constraint (deepgemm
        # smxx_fp8_fp4_paged_mqa_logits.hpp:233), so flatten there too.
        self.use_flattening = (
            self.use_fp4_indexer_cache
            or not current_platform.is_device_capability_family(100)
        ) and next_n not in self.natively_supported_next_n_fp4

        sm_count = num_compute_units(self.device.index)
        self.num_sms = sm_count

        self.offsets_buffer = torch.arange(
            next_n, device=self.device, dtype=torch.int32
        )
        self.decode_lens_buffer = torch.zeros(
            (scheduler_config.max_num_batched_tokens,),
            dtype=torch.int32,
            device=self.device,
        )
        # Shared workspace for decode seq_lens. Native MTP views this as
        # (B, max_decode_len) at runtime, keeping context_lens contiguous even
        # when max_decode_len is smaller than next_n.
        self.decode_seq_lens_buffer = torch.zeros(
    # ... omitted for brevity ...
        self.scheduler_metadata_buffer = torch.empty(
            (self.num_sms + 1, 2), dtype=torch.int32, device=self.device
        )

        # KV compression. Default to 1 for no compression.
        self.compress_ratio = 1
        # Get compress_ratio for DeepseekV4 support
        if isinstance(self.kv_cache_spec, MLAAttentionSpec):
            self.compress_ratio = self.kv_cache_spec.compress_ratio

        # Pre-allocate buffers for CUDA graph compatibility when
        if self.compress_ratio > 1:
            # compress_ratio > 1 (DeepseekV4)
            # Compressed slot mapping output buffer
            self.compressed_slot_mapping_buffer = torch.zeros(
                (scheduler_config.max_num_batched_tokens,),
                dtype=torch.int64,
                device=self.device,
            )
            # Buffer for compressed seq_lens in decode path
            self.expanded_seq_lens_buffer = torch.zeros(
                (scheduler_config.max_num_batched_tokens,),
                dtype=torch.int32,
                device=self.device,
            )
```
**EN:** This method initializes the object state within `DeepseekV32IndexerMetadataBuilder`. Key calls include `__init__`, `get_max_prefill_buffer_size`, `num_compute_units`, `arange`, `zeros`, `cdiv`. It touches state such as `max_prefill_buffer_size`, `num_speculative_tokens`, `use_fp4_indexer_cache`, `reorder_batch_threshold`, `use_flattening`, `num_sms`, `offsets_buffer`, `decode_lens_buffer`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`DeepseekV32IndexerMetadataBuilder`。 关键调用包括 `__init__`, `get_max_prefill_buffer_size`, `num_compute_units`, `arange`, `zeros`, `cdiv`。 它会读写 `max_prefill_buffer_size`, `num_speculative_tokens`, `use_fp4_indexer_cache`, `reorder_batch_threshold`, `use_flattening`, `num_sms`, `offsets_buffer`, `decode_lens_buffer` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `DeepseekV32IndexerMetadataBuilder._prepare_decode_tensors` method / `DeepseekV32IndexerMetadataBuilder._prepare_decode_tensors` 方法
```python
    def _prepare_decode_tensors(
        self,
        seq_lens: torch.Tensor,
        block_table: torch.Tensor,
        decode_lens: torch.Tensor,
        decode_lens_cpu: torch.Tensor,
        query_start_loc: torch.Tensor,
        num_decodes: int,
        num_decode_tokens: int,
        use_native: bool,
        next_n: int,
        max_decode_len: int,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, int, bool]:
        """Expand seq_lens/block_table/decode_lens for the decode kernels.

        Flatten path (not use_native, max_decode_len > 1):
          Each multi-token decode request is expanded into individual
          single-token entries so the kernel always sees next_n=1.

        Native path (use_native or max_decode_len == 1):
          Plain decode or spec-decode with 2D per-token context lengths.

        Returns (seq_lens, block_table, decode_lens, batch_size, requires_padding).
        seq_lens is 1D (batch_size,) for flatten/plain, 2D (B, max_decode_len)
        for native MTP.
        """
        min_decode_len = int(decode_lens_cpu.min().item())
        if not use_native and max_decode_len > 1:
            assert self.decode_seq_lens_buffer.dim() == 1
            if min_decode_len == max_decode_len:
                # Uniform decode lengths.
                num_decode_tokens = num_decodes * max_decode_len
                _prepare_uniform_decode_kernel[(num_decode_tokens,)](
                    seq_lens,
                    self.decode_seq_lens_buffer,
                    block_table,
                    block_table.stride(0),
                    self.expanded_block_table_buffer,
                    self.expanded_block_table_buffer.stride(0),
                    self.decode_lens_buffer,
                    max_decode_len,
                    BLOCK_SIZE=1024,
                )
                self.decode_seq_lens_buffer[num_decode_tokens:] = 0
                seq_lens = self.decode_seq_lens_buffer[:num_decode_tokens]
                block_table = self.expanded_block_table_buffer[:num_decode_tokens]
                decode_lens = self.decode_lens_buffer[:num_decode_tokens]
                return seq_lens, block_table, decode_lens, num_decode_tokens, False
            else:
                # Variable decode lengths.
    # ... omitted for brevity ...
                return seq_lens, block_table, decode_lens, num_decode_tokens, False
        else:
            # Native path: plain decode (next_n==1) or spec decode
            # with 2D per-token context lengths (next_n > 1).
            #
            # When decode_lens are not truly uniform (e.g. some requests have
            # decode_len < next_n due to padding or short prefills), the simple
            # reshape in sparse_attn_indexer won't work. Use pack_seq_triton
            # (requires_padding) instead.
            requires_padding = min_decode_len != max_decode_len
            if use_native and next_n > 1:
                assert self.decode_seq_lens_buffer.dim() == 1
                # (B, max_decode_len): token j attends to
                # L - max_decode_len + j + 1 KV tokens.
                seq_lens_buffer = self.decode_seq_lens_buffer[
                    : num_decodes * max_decode_len
                ].view(num_decodes, max_decode_len)
                seq_lens_buffer[:] = (
                    seq_lens.unsqueeze(1)
                    - max_decode_len
                    + 1
                    + self.offsets_buffer[:max_decode_len]
                )
                seq_lens = seq_lens_buffer
            return seq_lens, block_table, decode_lens, num_decodes, requires_padding
```
**EN:** This method implements `_prepare_decode_tensors` within `DeepseekV32IndexerMetadataBuilder`. The docstring frames it as: Expand seq_lens/block_table/decode_lens for the decode kernels. Key calls include `int`, `item`, `dim`, `repeat_interleave`, `view`, `min`. It touches state such as `decode_seq_lens_buffer`, `expanded_block_table_buffer`, `decode_lens_buffer`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_prepare_decode_tensors`，其作用域位于`DeepseekV32IndexerMetadataBuilder`。 关键调用包括 `int`, `item`, `dim`, `repeat_interleave`, `view`, `min`。 它会读写 `decode_seq_lens_buffer`, `expanded_block_table_buffer`, `decode_lens_buffer` 等状态。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `DeepseekV32IndexerMetadataBuilder.build` method / `DeepseekV32IndexerMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> DeepseekV32IndexerMetadata:
        num_reqs = common_attn_metadata.num_reqs
        num_tokens = common_attn_metadata.num_actual_tokens
        query_start_loc = common_attn_metadata.query_start_loc
        query_start_loc_cpu = common_attn_metadata.query_start_loc_cpu
        seq_lens = common_attn_metadata.seq_lens
        slot_mapping = common_attn_metadata.slot_mapping
        block_table = common_attn_metadata.block_table_tensor

        num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
            split_decodes_and_prefills(
                common_attn_metadata,
                decode_threshold=self.reorder_batch_threshold,
                require_uniform=not self.use_flattening,
            )
        )

        assert num_decodes + num_prefills == num_reqs
        assert num_decode_tokens + num_prefill_tokens == num_tokens

        compressed_slot_mapping = slot_mapping
        compressed_seq_lens = seq_lens
        if self.compress_ratio > 1:
            compressed_slot_mapping = get_compressed_slot_mapping(
                num_tokens,
                query_start_loc,
                seq_lens,
                block_table,
                self.kv_cache_spec.storage_block_size,
                self.compress_ratio,
                out=self.compressed_slot_mapping_buffer,
            )
            compressed_seq_lens = seq_lens // self.compress_ratio

        prefill_metadata = None
        if num_prefills > 0:
            # This CPU value is an upper bound for async-spec extend rows.  It
            # is safe for chunking/allocation because CUDA metadata below is
            # built from exact device seq_lens and gather ignores the tail.
            assert common_attn_metadata.seq_lens_cpu_upper_bound is not None
            seq_lens_cpu = common_attn_metadata.seq_lens_cpu_upper_bound
            compressed_seq_lens_cpu = (
                seq_lens_cpu // self.compress_ratio
                if self.compress_ratio > 1
                else seq_lens_cpu
    # ... omitted for brevity ...
                    self.kv_cache_spec.storage_block_size,
                    self.num_sms,
                )

            decode_metadata = DeepSeekV32IndexerDecodeMetadata(
                block_table=block_table,
                seq_lens=seq_lens,
                decode_lens=decode_lens,
                requires_padding=requires_padding,
                schedule_metadata=self.scheduler_metadata_buffer,
            )

        attn_metadata = DeepseekV32IndexerMetadata(
            seq_lens=common_attn_metadata.seq_lens,
            max_seq_len=common_attn_metadata.max_seq_len,
            slot_mapping=compressed_slot_mapping,
            num_decodes=num_decodes,
            num_decode_tokens=num_decode_tokens,
            num_prefills=num_prefills,
            num_prefill_tokens=num_prefill_tokens,
            prefill=prefill_metadata,
            decode=decode_metadata,
        )

        return attn_metadata
```
**EN:** This method builds derived structures within `DeepseekV32IndexerMetadataBuilder`. Key calls include `split_decodes_and_prefills`, `DeepseekV32IndexerMetadata`, `get_compressed_slot_mapping`, `diff`, `split_indexer_prefill_chunks`, `DeepseekV32IndexerPrefillMetadata`. It touches state such as `scheduler_metadata_buffer`, `expanded_seq_lens_buffer`. The control flow contains 9 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会构建派生结构，其作用域位于`DeepseekV32IndexerMetadataBuilder`。 关键调用包括 `split_decodes_and_prefills`, `DeepseekV32IndexerMetadata`, `get_compressed_slot_mapping`, `diff`, `split_indexer_prefill_chunks`, `DeepseekV32IndexerPrefillMetadata`。 它会读写 `scheduler_metadata_buffer`, `expanded_seq_lens_buffer` 等状态。 控制流包含 9 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `build_prefill_chunk_metadata` function / `build_prefill_chunk_metadata` 函数
```python
def build_prefill_chunk_metadata(
    start_idx: int,
    end_idx: int,
    query_start_loc: torch.Tensor,
    query_start_loc_cpu: torch.Tensor,
    uncompressed_seq_lens: torch.Tensor,
    compressed_seq_lens: torch.Tensor,
    compressed_seq_lens_cpu: torch.Tensor,
    block_table: torch.Tensor,
    compress_ratio: int,
    query_slice: slice | None = None,
    skip_kv_gather: bool = False,
) -> DeepseekV32IndexerPrefillChunkMetadata | None:
    total_seq_lens = compressed_seq_lens_cpu[start_idx:end_idx].sum().item()
    if total_seq_lens == 0:
        return None

    num_reqs = end_idx - start_idx
    device = block_table.device
    token_to_seq = torch.empty(total_seq_lens, dtype=torch.int32, device=device)

    cu_seq_lens = torch.empty(num_reqs + 1, dtype=torch.int32, device=device)
    # Assigning to slice avoids cpu sync.
    cu_seq_lens[:1] = 0
    torch.cumsum(compressed_seq_lens[start_idx:end_idx], dim=0, out=cu_seq_lens[1:])

    query_start_loc = (
        query_start_loc[start_idx : end_idx + 1] - query_start_loc[start_idx]
    )

    total_query_len = int(
        (query_start_loc_cpu[end_idx] - query_start_loc_cpu[start_idx]).item()
    )
    if query_slice is not None:
        qs_start = query_slice.start
        qs_stop = query_slice.stop
    else:
        qs_start = 0
        qs_stop = total_query_len
    output_query_len = qs_stop - qs_start

    cu_seq_len_ks = torch.empty(output_query_len, dtype=torch.int32, device=device)
    cu_seq_len_ke = torch.empty(output_query_len, dtype=torch.int32, device=device)

    _build_prefill_chunk_metadata_kernel[(num_reqs,)](
        query_start_loc,
        uncompressed_seq_lens[start_idx:end_idx],
        cu_seq_lens,
        token_to_seq,
        cu_seq_len_ks,
        cu_seq_len_ke,
        qs_start,
        qs_stop,
        BLOCK_SIZE=1024,
        COMPRESS_RATIO=compress_ratio,
    )

    token_start = query_start_loc_cpu[start_idx].item()
    if query_slice is not None:
        token_end = token_start + qs_stop
        token_start = token_start + qs_start
        skip_kv_gather = skip_kv_gather or qs_start > 0
    else:
        token_end = query_start_loc_cpu[end_idx].item()

    return DeepseekV32IndexerPrefillChunkMetadata(
        cu_seqlen_ks=cu_seq_len_ks,
        cu_seqlen_ke=cu_seq_len_ke,
        cu_seq_lens=cu_seq_lens,
        token_to_seq=token_to_seq,
        total_seq_lens=total_seq_lens,
        block_table=block_table[start_idx:end_idx],
        token_start=token_start,
        token_end=token_end,
        num_reqs=num_reqs,
        skip_kv_gather=skip_kv_gather,
    )
```
**EN:** This function builds derived structures within the module. Key calls include `item`, `empty`, `cumsum`, `int`, `DeepseekV32IndexerPrefillChunkMetadata`, `sum`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `item`, `empty`, `cumsum`, `int`, `DeepseekV32IndexerPrefillChunkMetadata`, `sum`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_build_prefill_chunk_metadata_kernel` function / `_build_prefill_chunk_metadata_kernel` 函数
```python
@triton.jit
def _build_prefill_chunk_metadata_kernel(
    # Inputs
    query_start_loc_ptr,
    uncompressed_seq_lens_ptr,
    cu_compressed_seq_lens_ptr,
    # Outputs
    token_to_seq_ptr,
    cu_compressed_seq_len_ks_ptr,
    cu_compressed_seq_len_ke_ptr,
    query_slice_start,
    query_slice_stop,
    BLOCK_SIZE: tl.constexpr,
    COMPRESS_RATIO: tl.constexpr,
):
    batch_idx = tl.program_id(0)

    query_start = tl.load(query_start_loc_ptr + batch_idx)
    query_end = tl.load(query_start_loc_ptr + batch_idx + 1)
    query_len = query_end - query_start

    seq_start = tl.load(cu_compressed_seq_lens_ptr + batch_idx)
    seq_end = tl.load(cu_compressed_seq_lens_ptr + batch_idx + 1)
    compressed_seq_len = seq_end - seq_start

    uncompressed_seq_len = tl.load(uncompressed_seq_lens_ptr + batch_idx)
    start_pos = uncompressed_seq_len - query_len

    for i in range(0, query_len, BLOCK_SIZE):
        offset = i + tl.arange(0, BLOCK_SIZE)
        abs_pos = query_start + offset
        mask = (
            (offset < query_len)
            & (abs_pos >= query_slice_start)
            & (abs_pos < query_slice_stop)
        )
        out_pos = abs_pos - query_slice_start

        # Compute cu_seq_len_ks
        tl.store(cu_compressed_seq_len_ks_ptr + out_pos, seq_start, mask=mask)

        # Compute cu_seq_len_ke
        seq_len_per_token = (start_pos + 1 + offset) // COMPRESS_RATIO
        tl.store(
            cu_compressed_seq_len_ke_ptr + out_pos,
            seq_start + seq_len_per_token,
            mask=mask,
        )

    # Compute token_to_seq
    for i in range(0, compressed_seq_len, BLOCK_SIZE):
        offset = i + tl.arange(0, BLOCK_SIZE)
        mask = offset < compressed_seq_len
        tl.store(token_to_seq_ptr + seq_start + offset, batch_idx, mask=mask)
```
**EN:** This function implements `_build_prefill_chunk_metadata_kernel` within the module. Key calls include `program_id`, `load`, `range`, `store`, `arange`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_build_prefill_chunk_metadata_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `range`, `store`, `arange`。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_prepare_uniform_decode_kernel`: top-level helper or orchestration entry point. / `_prepare_uniform_decode_kernel`：顶层辅助函数或编排入口。
- `split_indexer_prefill_chunks`: top-level helper or orchestration entry point. / `split_indexer_prefill_chunks`：顶层辅助函数或编排入口。
- `DeepseekV32IndexerBackend`: central class or interface in this module. / `DeepseekV32IndexerBackend`：本模块中的核心类或接口。
- `DeepseekV4IndexerBackend`: central class or interface in this module. / `DeepseekV4IndexerBackend`：本模块中的核心类或接口。
- `DeepseekV32IndexerPrefillChunkMetadata`: central class or interface in this module. / `DeepseekV32IndexerPrefillChunkMetadata`：本模块中的核心类或接口。
- `DeepseekV32IndexerPrefillMetadata`: central class or interface in this module. / `DeepseekV32IndexerPrefillMetadata`：本模块中的核心类或接口。
- `DeepSeekV32IndexerDecodeMetadata`: central class or interface in this module. / `DeepSeekV32IndexerDecodeMetadata`：本模块中的核心类或接口。
- `DeepseekV32IndexerMetadata`: central class or interface in this module. / `DeepseekV32IndexerMetadata`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config`, `vllm.logger`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.math_utils`, `vllm.utils.platform_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.mla.compressor_utils`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`
