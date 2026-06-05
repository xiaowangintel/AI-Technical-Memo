# rocm_aiter_mla_sparse.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/rocm_aiter_mla_sparse.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_convert_req_index_to_global_index_kernel`, `triton_convert_req_index_to_global_index`, `generate_sparse_seqlen_kernel` for the V1 `attention/backends/mla` subsystem. / 为 V1 的 `attention/backends/mla` 子系统实现 `_convert_req_index_to_global_index_kernel`, `triton_convert_req_index_to_global_index`, `generate_sparse_seqlen_kernel`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from dataclasses import dataclass
from typing import TYPE_CHECKING, ClassVar

import numpy as np
import torch

from vllm import _custom_ops as ops
from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import VllmConfig
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.mla_attention import (
    get_mla_dims,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionLayer,
    AttentionMetadata,
    AttentionMetadataBuilder,
    CommonAttentionMetadata,
    MultipleOf,
    SparseMLAAttentionImpl,
)
from vllm.v1.attention.backends.mla.rocm_aiter_mla import (
    AiterMLAHelper,
)
from vllm.v1.kv_cache_interface import AttentionSpec

if TYPE_CHECKING:
    from vllm.model_executor.models.deepseek_v2 import Indexer
logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_convert_req_index_to_global_index_kernel` function / `_convert_req_index_to_global_index_kernel` 函数
```python
@triton.jit
def _convert_req_index_to_global_index_kernel(
    req_id_ptr,  # int32 [num_tokens]
    block_table_ptr,  # int32 [num_requests, max_num_blocks_per_req]
    token_indices_ptr,  # int32 [num_tokens, NUM_TOPK_TOKENS]
    cu_seqlens_ptr,  # int32 [num_tokens + 1]
    out_ptr,  # int32 [num_tokens, NUM_TOPK_TOKENS]
    # shapes (compile-time where possible)
    max_num_blocks_per_req: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
    BLOCK_N: tl.constexpr,  # tile width along columns
    # strides (in elements)
    bt_stride0,
    bt_stride1,
    ti_stride0,
    ti_stride1,
):
    # program_id(0) -> token_id (row)
    # program_id(1) -> tile index along columns
    token_id = tl.program_id(0)
    tile_id = tl.program_id(1)

    # Each program covers BLOCK_N consecutive columns
    indice_id = tile_id * BLOCK_N + tl.arange(0, BLOCK_N)

    # Load request id for this token (no mask: grid is exact)
    req = tl.load(req_id_ptr + token_id)

    # Load cumulative sequence lengths to get starting index of this request
    seq_start = tl.load(cu_seqlens_ptr + token_id)
    seq_end = tl.load(cu_seqlens_ptr + token_id + 1)

    if tile_id * BLOCK_N + seq_start >= seq_end:
        return

    # Load token indices for this tile
    ti_ptr = token_indices_ptr + token_id * ti_stride0 + indice_id * ti_stride1
    tok = tl.load(ti_ptr)  # int32

    # Only token == -1 should propagate as -1
    is_invalid_tok = tok < 0

    # Compute block id and in-block offset
    block_id = tok // BLOCK_SIZE
    inblock_off = tok % BLOCK_SIZE

    # Guard block_table access
    valid_block = (block_id < max_num_blocks_per_req) & (block_id >= 0)
    bt_ptr = block_table_ptr + req * bt_stride0 + block_id * bt_stride1
    base = tl.load(bt_ptr, mask=valid_block, other=0)

    # # If token == -1 OR block_id OOB, output 0; else base * BLOCK_SIZE + offset
    out_val = tl.where(
        is_invalid_tok | (~valid_block), 0, base * BLOCK_SIZE + inblock_off
    )
    out_ptr_ij = out_ptr + seq_start + indice_id
    out_ptr_ij_mask = (seq_start + indice_id) < seq_end

    # store the results with mask
    tl.store(out_ptr_ij, out_val, mask=out_ptr_ij_mask)
```
**EN:** This function implements `_convert_req_index_to_global_index_kernel` within the module. Key calls include `program_id`, `load`, `where`, `store`, `arange`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_convert_req_index_to_global_index_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `where`, `store`, `arange`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `triton_convert_req_index_to_global_index` function / `triton_convert_req_index_to_global_index` 函数
```python
def triton_convert_req_index_to_global_index(
    req_id: torch.Tensor,  # int32 [num_tokens]
    block_table: torch.Tensor,  # int32 [num_requests, max_num_blocks_per_req]
    token_indices: torch.Tensor,  # int32 [num_tokens, NUM_TOPK_TOKENS]
    cu_seqlens: torch.Tensor,  # int32 [num_tokens + 1]
    paged_kv_indices: torch.Tensor,  # int32 [num_tokens * topk] out_buffer
    BLOCK_SIZE: int = 64,
    NUM_TOPK_TOKENS: int = 2048,
    BLOCK_N: int = 128,  # tile width along columns
):
    """
    out[token_id, indice_id] =
        block_table[req_id[token_id],
            token_indices[token_id, indice_id] // BLOCK_SIZE] * BLOCK_SIZE
        + token_indices[token_id, indice_id] % BLOCK_SIZE

    Only when token_indices[token_id, indice_id] == -1 do we output -1.
    For safety, we also output -1 if the derived block_id would be
        out-of-bounds.
    """
    assert req_id.dtype == torch.int32
    assert block_table.dtype == torch.int32
    assert token_indices.dtype == torch.int32
    assert token_indices.shape[1] == NUM_TOPK_TOKENS
    assert NUM_TOPK_TOKENS % BLOCK_N == 0, (
        f"NUM_TOPK_TOKENS ({NUM_TOPK_TOKENS}) must be divisible byBLOCK_N ({BLOCK_N})"
    )
    # print("req_id: ", req_id, flush=True)
    num_tokens = req_id.shape[0]
    _, max_num_blocks_per_req = block_table.shape
    tiles_per_row = NUM_TOPK_TOKENS // BLOCK_N

    # Ensure contiguous tensors on the same device
    req_id_c = req_id.contiguous()
    block_table_c = block_table.contiguous()
    token_indices_c = token_indices.contiguous()

    # Strides in elements
    bt_stride0, bt_stride1 = block_table_c.stride()
    ti_stride0, ti_stride1 = token_indices_c.stride()

    # Exact 2D grid: tokens × column tiles
    grid = (num_tokens, tiles_per_row)

    _convert_req_index_to_global_index_kernel[grid](
        req_id_c,
        block_table_c,
        token_indices_c,
        cu_seqlens,
        paged_kv_indices,
        # shapes / constexprs
        max_num_blocks_per_req,
        BLOCK_SIZE,
        BLOCK_N,
        # strides
        bt_stride0,
        bt_stride1,
        ti_stride0,
        ti_stride1,
    )
    return
```
**EN:** This function implements `triton_convert_req_index_to_global_index` within the module. The docstring frames it as: out[token_id, indice_id] = block_table[req_id[token_id], token_indices[token_id, indice_id] // BLOCK_SIZE] * BLOCK_SIZE + token_indices[token_id, indice_id] % BLOCK_SIZE Only when token_indices[token_id, indice_id] == -1 do we output -1. Key calls include `contiguous`, `stride`.
**CN:** 该函数会实现 `triton_convert_req_index_to_global_index`，其作用域位于the module。 关键调用包括 `contiguous`, `stride`。

### `generate_sparse_seqlen_kernel` function / `generate_sparse_seqlen_kernel` 函数
```python
@triton.jit
def generate_sparse_seqlen_kernel(
    seq_len_ptr,  # [num_seq]
    cu_query_lens_ptr,  # [num_seq]
    out_ptr,  # [num_query_tokens]
    topk_token: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    seq_id = tl.program_id(0)
    query_offset = tl.program_id(1) * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    query_start = tl.load(cu_query_lens_ptr + seq_id)
    query_end = tl.load(cu_query_lens_ptr + seq_id + 1)
    if query_start + tl.program_id(1) * BLOCK_SIZE > query_end:
        return
    query_len = query_end - query_start
    query_mask = query_offset + query_start < query_end
    seq_len = tl.load(seq_len_ptr + seq_id)
    # Just return since the out_ptr is zero initialized.
    if seq_len == 0:
        return
    context_start_point = seq_len - query_len
    sparse_seqlen = context_start_point + query_offset
    sparse_seqlen_masked = tl.where(
        sparse_seqlen + 1 < topk_token, sparse_seqlen + 1, topk_token
    )
    tl.store(
        out_ptr + query_start + query_offset, sparse_seqlen_masked, mask=query_mask
    )
```
**EN:** This function implements `generate_sparse_seqlen_kernel` within the module. Key calls include `program_id`, `load`, `where`, `store`, `arange`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `generate_sparse_seqlen_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `where`, `store`, `arange`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `generate_sparse_seqlen_triton` function / `generate_sparse_seqlen_triton` 函数
```python
def generate_sparse_seqlen_triton(
    query_lens: torch.Tensor,
    seq_lens: torch.Tensor,
    cu_query_lens: torch.Tensor,
    topk_token: int,
    num_tokens: int,
    max_query_len: int,
):
    num_seqs = query_lens.size(0)
    # zero initialize the tensor to make sure invalid positions will be zero
    out = torch.zeros([num_tokens], dtype=torch.int32, device=query_lens.device)
    block_size = 64
    num_block_per_row = triton.cdiv(max_query_len, block_size)
    grid = (
        num_seqs,
        num_block_per_row,
    )
    generate_sparse_seqlen_kernel[grid](
        seq_lens,
        cu_query_lens,
        out,
        topk_token,
        block_size,
    )
    return out
```
**EN:** This function implements `generate_sparse_seqlen_triton` within the module. Key calls include `size`, `zeros`, `cdiv`.
**CN:** 该函数会实现 `generate_sparse_seqlen_triton`，其作用域位于the module。 关键调用包括 `size`, `zeros`, `cdiv`。

### `fetch_id_to_ragged_kernel` function / `fetch_id_to_ragged_kernel` 函数
```python
@triton.jit
def fetch_id_to_ragged_kernel(
    in_tensor_ptr,  # [num_seq, topk]
    cumsum_ptr,  # [num_seq + 1]
    out_tensor_ptr,  # [max_num_seq * topk]
    in_tensor_ptr_stride,
    TOPK: tl.constexpr,
    TOKEN_NUM: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    seq_id = tl.program_id(0)
    block_id = tl.program_id(1)
    offset = tl.arange(0, BLOCK_SIZE)
    token_start = tl.load(cumsum_ptr + seq_id)
    token_end = tl.load(cumsum_ptr + seq_id + 1)
    token_num = token_end - token_start
    row_offset = block_id * BLOCK_SIZE
    if row_offset >= token_num:
        return
    in_tensor_offset = seq_id * in_tensor_ptr_stride + row_offset + offset
    in_tensor_mask = (row_offset + offset) < TOPK
    in_tensor_val = tl.load(in_tensor_ptr + in_tensor_offset, mask=in_tensor_mask)
    out_tensor_offset = token_start + row_offset + offset
    out_tensor_mask = (out_tensor_offset < token_end) & in_tensor_mask
    tl.store(out_tensor_ptr + out_tensor_offset, in_tensor_val, mask=out_tensor_mask)
```
**EN:** This function implements `fetch_id_to_ragged_kernel` within the module. Key calls include `program_id`, `arange`, `load`, `store`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `fetch_id_to_ragged_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `arange`, `load`, `store`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `fetch_id_to_ragged_triton` function / `fetch_id_to_ragged_triton` 函数
```python
def fetch_id_to_ragged_triton(
    in_tensor: torch.Tensor, cumsum: torch.Tensor, out_tensor: torch.Tensor, topk
):
    num_tokens = in_tensor.size(0)
    block_size = 64
    num_block_per_row = triton.cdiv(topk, block_size)
    grid = (
        num_tokens,
        num_block_per_row,
    )
    fetch_id_to_ragged_kernel[grid](
        in_tensor, cumsum, out_tensor, in_tensor.stride(0), topk, num_tokens, block_size
    )
```
**EN:** This function implements `fetch_id_to_ragged_triton` within the module. Key calls include `size`, `cdiv`, `stride`.
**CN:** 该函数会实现 `fetch_id_to_ragged_triton`，其作用域位于the module。 关键调用包括 `size`, `cdiv`, `stride`。

### `ROCMAiterMLASparseBackend` class / `ROCMAiterMLASparseBackend` 类
```python
class ROCMAiterMLASparseBackend(AttentionBackend):
    supported_dtypes: ClassVar[list[torch.dtype]] = [torch.float16, torch.bfloat16]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "float16",
        "bfloat16",
        "fp8",
        "fp8_e4m3",
    ]
```
**EN:** Introduces the `ROCMAiterMLASparseBackend` class on top of `AttentionBackend`. Core methods include `get_supported_kernel_block_sizes`, `get_name`, `get_metadata_cls`, `get_builder_cls`, `get_impl_cls`, `get_kv_cache_shape`.
**CN:** 这里定义 `ROCMAiterMLASparseBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_supported_kernel_block_sizes`, `get_name`, `get_metadata_cls`, `get_builder_cls`, `get_impl_cls`, `get_kv_cache_shape`。

### `ROCMAiterMLASparseBackend.get_supported_kernel_block_sizes` method / `ROCMAiterMLASparseBackend.get_supported_kernel_block_sizes` 方法
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        return [1, 64]
```
**EN:** This method returns or derives a value within `ROCMAiterMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`ROCMAiterMLASparseBackend`。

### `ROCMAiterMLASparseBackend.get_builder_cls` method / `ROCMAiterMLASparseBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["ROCMAiterMLASparseMetadataBuilder"]:
        return ROCMAiterMLASparseMetadataBuilder
```
**EN:** This method returns or derives a value within `ROCMAiterMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`ROCMAiterMLASparseBackend`。

### `ROCMAiterMLASparseBackend.get_kv_cache_shape` method / `ROCMAiterMLASparseBackend.get_kv_cache_shape` 方法
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,  # assumed to be 1 for MLA
        head_size: int,
        cache_dtype_str: str = "auto",
    ) -> tuple[int, ...]:
        return (num_blocks, block_size, head_size)
```
**EN:** This method returns or derives a value within `ROCMAiterMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`ROCMAiterMLASparseBackend`。

### `ROCMAiterMLASparseMetadata` class / `ROCMAiterMLASparseMetadata` 类
```python
@dataclass
class ROCMAiterMLASparseMetadata(AttentionMetadata):
    num_reqs: int
    max_query_len: int
    max_seq_len: int

    num_actual_tokens: int  # Number of tokens excluding padding.
    query_start_loc: torch.Tensor
    slot_mapping: torch.Tensor

    block_table: torch.Tensor
    req_id_per_token: torch.Tensor

    qo_indptr: torch.Tensor
    paged_kv_last_page_len: torch.Tensor
    paged_kv_indices: torch.Tensor
    paged_kv_indptr: torch.Tensor
    attn_out_dtype: torch.dtype

    block_size: int = 1
    topk_tokens: int = 2048

    # Persistent MLA metadata (only populated when persistent mode is enabled,
    # i.e. when the aiter sparse decode kernel supports work-stealing splits).
    work_meta_data: torch.Tensor | None = None
    work_indptr: torch.Tensor | None = None
    work_info_set: torch.Tensor | None = None
    reduce_indptr: torch.Tensor | None = None
    reduce_final_map: torch.Tensor | None = None
    reduce_partial_map: torch.Tensor | None = None
```
**EN:** Uses `@dataclass` to package related state for `ROCMAiterMLASparseMetadata`. Typical fields include `num_reqs`, `max_query_len`, `max_seq_len`, `num_actual_tokens`, `query_start_loc`, `slot_mapping`.
**CN:** `ROCMAiterMLASparseMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_reqs`, `max_query_len`, `max_seq_len`, `num_actual_tokens`, `query_start_loc`, `slot_mapping`。

### `ROCMAiterMLASparseMetadataBuilder` class / `ROCMAiterMLASparseMetadataBuilder` 类
```python
@dataclass
class ROCMAiterMLASparseMetadataBuilder(
    AttentionMetadataBuilder[ROCMAiterMLASparseMetadata]
):
    _cudagraph_support: ClassVar[AttentionCGSupport] = (
        AttentionCGSupport.UNIFORM_SINGLE_TOKEN_DECODE
    )
```
**EN:** Uses `@dataclass` to package related state for `ROCMAiterMLASparseMetadataBuilder`. Typical fields include `_cudagraph_support`.
**CN:** `ROCMAiterMLASparseMetadataBuilder` 使用 `@dataclass` 打包相关状态。典型字段包括 `_cudagraph_support`。

### `ROCMAiterMLASparseMetadataBuilder.__init__` method / `ROCMAiterMLASparseMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        self.kv_cache_spec = kv_cache_spec
        self.model_config = vllm_config.model_config
        self.model_dtype = vllm_config.model_config.dtype
        parallel_config = vllm_config.parallel_config
        self.device = device
        max_num_batched_tokens = vllm_config.scheduler_config.max_num_batched_tokens

        self.num_heads = self.model_config.get_num_attention_heads(parallel_config)
        self.mla_dims = get_mla_dims(self.model_config)
        self.topk_tokens = vllm_config.model_config.hf_config.index_topk
        self.max_model_len_tensor = torch.tensor(
            [self.model_config.max_model_len], device=device, dtype=torch.int32
        )
        # this is ignored by `flash_mla_with_kvcache` if indices not None
        self.dummy_block_table = torch.empty(
            (1, 1), dtype=torch.int32, device=self.device
        )

        self.req_id_per_token_buffer = torch.empty(
            (vllm_config.scheduler_config.max_num_batched_tokens,),
            dtype=torch.int32,
            device=device,
        )
        self.qo_indptr = torch.arange(
            0, max_num_batched_tokens + 1, dtype=torch.int32, device=device
        )
        self.paged_kv_last_page_len = torch.ones(
            max_num_batched_tokens, dtype=torch.int32, device=device
        )

        # These two needs to be calculated in runtime,
        # but we still needs to prepare the buffer
        self.paged_kv_indices = torch.zeros(
            [max_num_batched_tokens * self.topk_tokens],
            dtype=torch.int32,
            device=device,
        )
        self.paged_kv_indptr = torch.zeros(
            [max_num_batched_tokens + 1], dtype=torch.int32, device=device
        )

        # ----- Persistent MLA metadata buffers -----
        # The aiter sparse decode kernel supports a "persistent" path that
    # ... omitted for brevity ...
            q_dtype,
            kv_dtype,
            is_sparse=True,
            fast_mode=True,
        )
        self._mla_work_meta_data = torch.empty(
            work_meta_data_size, dtype=work_meta_data_type, device=device
        )
        self._mla_work_indptr = torch.empty(
            work_indptr_size, dtype=work_indptr_type, device=device
        )
        self._mla_work_info_set = torch.empty(
            work_info_set_size, dtype=work_info_set_type, device=device
        )
        self._mla_reduce_indptr = torch.empty(
            reduce_indptr_size, dtype=reduce_indptr_type, device=device
        )
        self._mla_reduce_final_map = torch.empty(
            reduce_final_map_size, dtype=reduce_final_map_type, device=device
        )
        self._mla_reduce_partial_map = torch.empty(
            reduce_partial_map_size,
            dtype=reduce_partial_map_type,
            device=device,
        )
```
**EN:** This method initializes the object state within `ROCMAiterMLASparseMetadataBuilder`. Key calls include `get_num_attention_heads`, `get_mla_dims`, `tensor`, `empty`, `arange`, `ones`. It touches state such as `kv_cache_spec`, `model_config`, `model_dtype`, `device`, `num_heads`, `mla_dims`, `topk_tokens`, `max_model_len_tensor`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`ROCMAiterMLASparseMetadataBuilder`。 关键调用包括 `get_num_attention_heads`, `get_mla_dims`, `tensor`, `empty`, `arange`, `ones`。 它会读写 `kv_cache_spec`, `model_config`, `model_dtype`, `device`, `num_heads`, `mla_dims`, `topk_tokens`, `max_model_len_tensor` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `ROCMAiterMLASparseMetadataBuilder.build` method / `ROCMAiterMLASparseMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> ROCMAiterMLASparseMetadata:
        num_tokens = common_attn_metadata.num_actual_tokens
        starts = np.asarray(common_attn_metadata.query_start_loc_cpu, dtype=np.int32)
        seg_lengths = np.diff(starts)
        req_id_per_token = np.repeat(
            np.arange(seg_lengths.shape[0], dtype=np.int32), seg_lengths
        )
        # Zero-fill for cudagraphs
        self.req_id_per_token_buffer.fill_(0)
        self.paged_kv_indices.fill_(0)
        self.paged_kv_indptr.fill_(0)
        self.req_id_per_token_buffer[: req_id_per_token.shape[0]].copy_(
            torch.from_numpy(req_id_per_token), non_blocking=True
        )
        query_lens = (
            common_attn_metadata.query_start_loc[1:]
            - common_attn_metadata.query_start_loc[:-1]
        )
        seq_lens = common_attn_metadata.seq_lens
        sparse_seqlen = generate_sparse_seqlen_triton(
            query_lens,
            seq_lens,
            common_attn_metadata.query_start_loc,
            self.topk_tokens,
            num_tokens,
            common_attn_metadata.max_query_len,
        )

        torch.cumsum(sparse_seqlen, dim=0, out=self.paged_kv_indptr[1 : num_tokens + 1])
        self.paged_kv_indptr[num_tokens + 1 :].fill_(self.paged_kv_indptr[num_tokens])

        req_id_per_token = self.req_id_per_token_buffer[:num_tokens]
        qo_indptr = self.qo_indptr[: num_tokens + 1]
        paged_kv_last_page_len = self.paged_kv_last_page_len[:num_tokens]
        paged_kv_indptr = self.paged_kv_indptr[: num_tokens + 1]
        paged_kv_indices = self.paged_kv_indices[: num_tokens * self.topk_tokens]

        # ----- Compute persistent MLA metadata -----
        # The aiter sparse decode kernel uses qseqlen=1 (each query token is
        # treated as its own batch entry), so persistent metadata can always
        # be precomputed here. The kernel switches to the persistent
        # work-stealing path automatically when work_meta_data is non-None.
        from aiter import get_mla_metadata_v1

        get_mla_metadata_v1(
    # ... omitted for brevity ...

        metadata = ROCMAiterMLASparseMetadata(
            num_reqs=common_attn_metadata.num_reqs,
            max_query_len=common_attn_metadata.max_query_len,
            max_seq_len=common_attn_metadata.max_seq_len,
            num_actual_tokens=common_attn_metadata.num_actual_tokens,
            query_start_loc=common_attn_metadata.query_start_loc,
            slot_mapping=common_attn_metadata.slot_mapping,
            block_table=common_attn_metadata.block_table_tensor,
            req_id_per_token=req_id_per_token,
            block_size=self.kv_cache_spec.block_size,
            attn_out_dtype=self.model_dtype,
            topk_tokens=self.topk_tokens,
            qo_indptr=qo_indptr,
            paged_kv_last_page_len=paged_kv_last_page_len,
            paged_kv_indices=paged_kv_indices,
            paged_kv_indptr=paged_kv_indptr,
            work_meta_data=self._mla_work_meta_data,
            work_indptr=self._mla_work_indptr,
            work_info_set=self._mla_work_info_set,
            reduce_indptr=self._mla_reduce_indptr,
            reduce_final_map=self._mla_reduce_final_map,
            reduce_partial_map=self._mla_reduce_partial_map,
        )
        return metadata
```
**EN:** This method builds derived structures within `ROCMAiterMLASparseMetadataBuilder`. Key calls include `asarray`, `diff`, `repeat`, `fill_`, `copy_`, `generate_sparse_seqlen_triton`. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会构建派生结构，其作用域位于`ROCMAiterMLASparseMetadataBuilder`。 关键调用包括 `asarray`, `diff`, `repeat`, `fill_`, `copy_`, `generate_sparse_seqlen_triton`。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `reference_mla_sparse_prefill` function / `reference_mla_sparse_prefill` 函数
```python
def reference_mla_sparse_prefill(
    q: torch.Tensor, kv: torch.Tensor, indices: torch.Tensor, sm_scale: float, d_v: int
) -> tuple[torch.Tensor, torch.Tensor]:
    import math

    def log2sumexp2(a: torch.Tensor, dim: int) -> torch.Tensor:
        return torch.logsumexp(a * math.log(2), dim=dim) * math.log2(math.e)

    skv = kv.shape[0]
    sq = q.shape[0]
    topk = indices.shape[-1]
    dqk = q.shape[-1]
    indices = indices[:, 0, :]  # [s_q, topk]
    invalid_indices_mask = (indices < 0) | (indices >= skv)
    indices[invalid_indices_mask] = 0
    qs = q  # [s_q, h_q, d_qk]
    kvs = kv[:, 0, :][indices].view(sq, topk, dqk)  # [s_q, topk, d_qk]

    attn_score = (qs @ kvs.transpose(1, 2)).float()  # [s_q, h_q, topk]
    attn_score.masked_fill_(invalid_indices_mask.unsqueeze(1), float("-inf"))
    attn_score *= sm_scale * math.log2(math.e)
    lse = log2sumexp2(attn_score, dim=-1)  # [s_q, h_q]
    attn_score = torch.exp2(attn_score - lse.unsqueeze(-1))  # [s_q, h_q, topk]
    result = attn_score.to(q.dtype) @ kvs[:, :, :d_v]
    return (result, lse)
```
**EN:** This function implements `reference_mla_sparse_prefill` within the module. Key calls include `view`, `float`, `masked_fill_`, `log2sumexp2`, `exp2`, `unsqueeze`.
**CN:** 该函数会实现 `reference_mla_sparse_prefill`，其作用域位于the module。 关键调用包括 `view`, `float`, `masked_fill_`, `log2sumexp2`, `exp2`, `unsqueeze`。

### `ROCMAiterMLASparseImpl` class / `ROCMAiterMLASparseImpl` 类
```python
class ROCMAiterMLASparseImpl(SparseMLAAttentionImpl[ROCMAiterMLASparseMetadata]):
```
**EN:** Introduces the `ROCMAiterMLASparseImpl` class on top of `SparseMLAAttentionImpl[ROCMAiterMLASparseMetadata]`. Core methods include `__init__`, `_forward_mla`, `forward_mqa`.
**CN:** 这里定义 `ROCMAiterMLASparseImpl` 类，其基类包括 `SparseMLAAttentionImpl[ROCMAiterMLASparseMetadata]`。核心方法包括 `__init__`, `_forward_mla`, `forward_mqa`。

### `ROCMAiterMLASparseImpl.__init__` method / `ROCMAiterMLASparseImpl.__init__` 方法
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        num_kv_heads: int,
        alibi_slopes: list[float] | None,
        sliding_window: int | None,
        kv_cache_dtype: str,
        logits_soft_cap: float | None,
        attn_type: str,
        kv_sharing_target_layer_name: str | None,
        # MLA Specific Arguments
        topk_indice_buffer: torch.Tensor | None = None,
        indexer: "Indexer | None" = None,
        **mla_args,
    ) -> None:
        AiterMLAHelper.check_num_heads_validity(num_heads)

        self.num_heads = num_heads
        self.head_size = head_size
        self.scale = float(scale)
        self.num_kv_heads = num_kv_heads
        self.kv_cache_dtype = kv_cache_dtype
        self.kv_lora_rank: int = mla_args["kv_lora_rank"]
        self.softmax_scale = scale
        assert indexer is not None
        self.topk_indices_buffer: torch.Tensor | None = indexer.topk_indices_buffer
```
**EN:** This method initializes the object state within `ROCMAiterMLASparseImpl`. Key calls include `check_num_heads_validity`, `float`. It touches state such as `num_heads`, `head_size`, `scale`, `num_kv_heads`, `kv_cache_dtype`, `kv_lora_rank`, `softmax_scale`, `topk_indices_buffer`.
**CN:** 该方法会初始化对象状态，其作用域位于`ROCMAiterMLASparseImpl`。 关键调用包括 `check_num_heads_validity`, `float`。 它会读写 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `kv_cache_dtype`, `kv_lora_rank`, `softmax_scale`, `topk_indices_buffer` 等状态。

### `ROCMAiterMLASparseImpl._forward_mla` method / `ROCMAiterMLASparseImpl._forward_mla` 方法
```python
    def _forward_mla(
        self,
        layer: AttentionLayer,
        q: torch.Tensor,  # [sq, heads, d_qk]
        kv_c_and_k_pe_cache: torch.Tensor,  # [blocks, heads, d_qk]
        attn_metadata: ROCMAiterMLASparseMetadata,
    ) -> torch.Tensor:
        num_tokens = q.shape[0]
        mla_num_heads = AiterMLAHelper.get_actual_mla_num_heads(self.num_heads)
        output = torch.empty(
            [num_tokens, mla_num_heads, self.kv_lora_rank],
            dtype=attn_metadata.attn_out_dtype,
            device=q.device,
        )

        # Build kwargs and forward the persistent MLA metadata when it has
        # been computed. The aiter mla_decode_fwd switches to its
        # work-stealing persistent kernel path when work_meta_data is given.
        mla_kwargs: dict = dict(
            q_scale=layer._q_scale,
            kv_scale=layer._k_scale,
        )
        if attn_metadata.work_meta_data is not None:
            mla_kwargs.update(
                work_meta_data=attn_metadata.work_meta_data,
                work_indptr=attn_metadata.work_indptr,
                work_info_set=attn_metadata.work_info_set,
                reduce_indptr=attn_metadata.reduce_indptr,
                reduce_final_map=attn_metadata.reduce_final_map,
                reduce_partial_map=attn_metadata.reduce_partial_map,
            )

        rocm_aiter_ops.mla_decode_fwd(
            q,
            kv_c_and_k_pe_cache,
            output,
            self.scale,
            attn_metadata.qo_indptr,
            1,
            attn_metadata.paged_kv_indptr,
            attn_metadata.paged_kv_indices,
            attn_metadata.paged_kv_last_page_len,
            **mla_kwargs,
        )

        return AiterMLAHelper.get_mla_unpadded_o(self.num_heads, output)
```
**EN:** This method implements `_forward_mla` within `ROCMAiterMLASparseImpl`. Key calls include `get_actual_mla_num_heads`, `empty`, `dict`, `mla_decode_fwd`, `get_mla_unpadded_o`, `update`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_forward_mla`，其作用域位于`ROCMAiterMLASparseImpl`。 关键调用包括 `get_actual_mla_num_heads`, `empty`, `dict`, `mla_decode_fwd`, `get_mla_unpadded_o`, `update`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ROCMAiterMLASparseImpl.forward_mqa` method / `ROCMAiterMLASparseImpl.forward_mqa` 方法
```python
    def forward_mqa(
        self,
        q: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: ROCMAiterMLASparseMetadata,
        layer: AttentionLayer,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        # NOTE(lucas): for the sparse FlashMLA kernels the kernels want to use
        # MQA 576/512 approach for both prefill and decode

        # Concatenate q if it's a tuple (ql_nope, q_pe)
        if isinstance(q, tuple):
            q = torch.cat(q, dim=-1)

        num_actual_toks = attn_metadata.num_actual_tokens

        # Get topk indices
        assert self.topk_indices_buffer is not None
        topk_indices = self.topk_indices_buffer[:num_actual_toks]

        triton_convert_req_index_to_global_index(
            attn_metadata.req_id_per_token,
            attn_metadata.block_table,
            topk_indices,
            attn_metadata.paged_kv_indptr,
            attn_metadata.paged_kv_indices,
            BLOCK_SIZE=attn_metadata.block_size,
            NUM_TOPK_TOKENS=attn_metadata.topk_tokens,
        )

        # write the latent and rope to kv cache
        fp8_attention = self.kv_cache_dtype.startswith("fp8")
        if fp8_attention:
            original_q_shape = q.shape
            kv_c_and_k_pe_cache = kv_c_and_k_pe_cache.view(current_platform.fp8_dtype())
            q, _ = ops.scaled_fp8_quant(q.view(q.shape[0], -1), layer._q_scale)
            q = q.view(original_q_shape)
        mla_padded_q = AiterMLAHelper.get_mla_padded_q(self.num_heads, q)
        attn_out = self._forward_mla(
            layer, mla_padded_q, kv_c_and_k_pe_cache, attn_metadata
        )

        return attn_out, None
```
**EN:** This method drives the forward-pass computation within `ROCMAiterMLASparseImpl`. Key calls include `isinstance`, `triton_convert_req_index_to_global_index`, `startswith`, `get_mla_padded_q`, `_forward_mla`, `cat`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`ROCMAiterMLASparseImpl`。 关键调用包括 `isinstance`, `triton_convert_req_index_to_global_index`, `startswith`, `get_mla_padded_q`, `_forward_mla`, `cat`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_convert_req_index_to_global_index_kernel`: top-level helper or orchestration entry point. / `_convert_req_index_to_global_index_kernel`：顶层辅助函数或编排入口。
- `triton_convert_req_index_to_global_index`: top-level helper or orchestration entry point. / `triton_convert_req_index_to_global_index`：顶层辅助函数或编排入口。
- `generate_sparse_seqlen_kernel`: top-level helper or orchestration entry point. / `generate_sparse_seqlen_kernel`：顶层辅助函数或编排入口。
- `generate_sparse_seqlen_triton`: top-level helper or orchestration entry point. / `generate_sparse_seqlen_triton`：顶层辅助函数或编排入口。
- `fetch_id_to_ragged_kernel`: top-level helper or orchestration entry point. / `fetch_id_to_ragged_kernel`：顶层辅助函数或编排入口。
- `fetch_id_to_ragged_triton`: top-level helper or orchestration entry point. / `fetch_id_to_ragged_triton`：顶层辅助函数或编排入口。
- `ROCMAiterMLASparseBackend`: central class or interface in this module. / `ROCMAiterMLASparseBackend`：本模块中的核心类或接口。
- `ROCMAiterMLASparseMetadata`: central class or interface in this module. / `ROCMAiterMLASparseMetadata`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`, `math`
- External / 外部依赖: `numpy`, `torch`, `aiter`
- Internal vLLM / 内部依赖: `vllm`, `vllm._aiter_ops`, `vllm.config`, `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.platforms`, `vllm.triton_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.mla.rocm_aiter_mla`, `vllm.v1.kv_cache_interface`, `vllm.model_executor.models.deepseek_v2`
