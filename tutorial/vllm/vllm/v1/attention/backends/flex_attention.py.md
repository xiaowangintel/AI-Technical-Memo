# flex_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/flex_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Attention layer with FlexAttention. / 该模块位于 `attention/backends` 子系统，主要围绕 `_offsets_to_doc_ids_tensor`, `pad_to_multiple`, `FlexAttentionBackend` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Attention layer with FlexAttention."""

import math
from collections.abc import Callable
from dataclasses import dataclass
from functools import cached_property
from typing import ClassVar, NamedTuple

import torch
import torch._dynamo.decorators
import torch.nn.functional as F
from torch.nn.attention.flex_attention import (
    BlockMask,
    _mask_mod_signature,
    _score_mod_signature,
    and_masks,
    create_block_mask,
    flex_attention,
    or_masks,
)

import vllm.envs as envs
from vllm.config import VllmConfig
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.utils.math_utils import cdiv
from vllm.utils.torch_utils import is_quantized_kv_cache, is_torch_equal_or_newer
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionImpl,
    AttentionMetadataBuilder,
    AttentionType,
    CommonAttentionMetadata,
    MultipleOf,
)
from vllm.v1.kv_cache_interface import AttentionSpec, EncoderOnlyAttentionSpec

logger = init_logger(__name__)

torch._dynamo.config.recompile_limit = 16
create_block_mask_compiled = torch.compile(
    create_block_mask, fullgraph=True, mode="reduce-overhead"
)
flex_attention_compiled = torch.compile(flex_attention, fullgraph=True)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `create_block_mask_compiled`, `flex_attention_compiled`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `create_block_mask_compiled`, `flex_attention_compiled`。

### `_offsets_to_doc_ids_tensor` function / `_offsets_to_doc_ids_tensor` 函数
```python
def _offsets_to_doc_ids_tensor(
    offsets_cpu: torch.Tensor, device: torch.device
) -> torch.Tensor:
    # Build on CPU (so `repeat_interleave` doesn't force a GPU->CPU sync to
    # learn the data-dependent output length) and upload non-blocking.
    counts = offsets_cpu[1:] - offsets_cpu[:-1]
    doc_ids = torch.repeat_interleave(
        torch.arange(len(counts), dtype=torch.int32), counts
    )
    return doc_ids.to(device, non_blocking=True)
```
**EN:** This function implements `_offsets_to_doc_ids_tensor` within the module. Key calls include `repeat_interleave`, `to`, `arange`, `len`.
**CN:** 该函数会实现 `_offsets_to_doc_ids_tensor`，其作用域位于the module。 关键调用包括 `repeat_interleave`, `to`, `arange`, `len`。

### `pad_to_multiple` function / `pad_to_multiple` 函数
```python
def pad_to_multiple(x: torch.Tensor, multiple: int, dim: int):
    difference = (multiple - (x.shape[dim] % multiple)) % multiple
    if difference == 0:
        return x

    dim = dim if dim >= 0 else x.ndim + dim
    pad_list = []

    for i in range(x.ndim - 1, dim - 1, -1):
        if i == dim:
            pad_list.extend([0, difference])
        else:
            pad_list.extend([0, 0])

    return F.pad(x, pad_list, mode="constant", value=0)
```
**EN:** This function implements `pad_to_multiple` within the module. Key calls include `range`, `pad`, `extend`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `pad_to_multiple`，其作用域位于the module。 关键调用包括 `range`, `pad`, `extend`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `FlexAttentionBackend` class / `FlexAttentionBackend` 类
```python
class FlexAttentionBackend(AttentionBackend):
    supported_dtypes: ClassVar[list[torch.dtype]] = [
        torch.float16,
        torch.bfloat16,
        torch.float32,
    ]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "float16",
        "bfloat16",
    ]

    forward_includes_kv_cache_update: bool = False
```
**EN:** Introduces the `FlexAttentionBackend` class on top of `AttentionBackend`. Core methods include `get_name`, `supports_non_causal`, `supports_attn_type`, `supports_batch_invariance`, `supports_mm_prefix`, `get_impl_cls`.
**CN:** 这里定义 `FlexAttentionBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_name`, `supports_non_causal`, `supports_attn_type`, `supports_batch_invariance`, `supports_mm_prefix`, `get_impl_cls`。

### `FlexAttentionBackend.supports_attn_type` method / `FlexAttentionBackend.supports_attn_type` 方法
```python
    @classmethod
    def supports_attn_type(cls, attn_type: str) -> bool:
        """FlexAttention supports both decoder and encoder-only attention."""
        return attn_type in (AttentionType.DECODER, AttentionType.ENCODER_ONLY)
```
**EN:** This method implements `supports_attn_type` within `FlexAttentionBackend`. The docstring frames it as: FlexAttention supports both decoder and encoder-only attention.
**CN:** 该方法会实现 `supports_attn_type`，其作用域位于`FlexAttentionBackend`。

### `FlexAttentionBackend.supports_mm_prefix` method / `FlexAttentionBackend.supports_mm_prefix` 方法
```python
    @classmethod
    def supports_mm_prefix(cls) -> bool:
        """FlexAttention supports full attention for image tokens."""
        return True
```
**EN:** This method implements `supports_mm_prefix` within `FlexAttentionBackend`. The docstring frames it as: FlexAttention supports full attention for image tokens.
**CN:** 该方法会实现 `supports_mm_prefix`，其作用域位于`FlexAttentionBackend`。

### `FlexAttentionBackend.get_kv_cache_shape` method / `FlexAttentionBackend.get_kv_cache_shape` 方法
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,
        head_size: int,
        cache_dtype_str: str = "auto",
    ) -> tuple[int, ...]:
        return (2, num_blocks, block_size, num_kv_heads, head_size)
```
**EN:** This method returns or derives a value within `FlexAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlexAttentionBackend`。

### `FlexAttentionBackend.get_builder_cls` method / `FlexAttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["FlexAttentionMetadataBuilder"]:
        return FlexAttentionMetadataBuilder
```
**EN:** This method returns or derives a value within `FlexAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlexAttentionBackend`。

### `physical_to_logical_mapping` function / `physical_to_logical_mapping` 函数
```python
def physical_to_logical_mapping(
    block_table: torch.Tensor,
    seq_lens: torch.Tensor,
    block_size: int,
    total_blocks: int,
) -> torch.Tensor:
    """
    Creates an inverse mapping from physical block locations to logical indices.

    The original block_table maps from logical blocks to physical locations:

    Logical to Physical (Original block_table):
    ┌───────────────────────────────────────────┐
    │ Request 0:                                │
    │                                           │
    │ Logical Blocks:  0  1  2  3  4  5  6  7   │
    │                  │  │  │  │  │  │  │  │   │
    │                  v  v  v  v  v  v  v  v   │
    │ Physical Blocks: 3  5  1  7  4  2  0  6   │
    └───────────────────────────────────────────┘

    This function creates the inverse mapping:

    Physical to Logical (Inverse mapping):
    ┌───────────────────────────────────────────┐
    │ Request 0:                                │
    │                                           │
    │ Physical Blocks: 0  1  2  3  4  5  6  7   │
    │                  │  │  │  │  │  │  │  │   │
    │                  v  v  v  v  v  v  v  v   │
    │ Logical Blocks:  6  2  5  0  4  1  7  3   │
    └───────────────────────────────────────────┘

    If multiple logical blocks map to the same physical block,
    this function returns the latest (maximum) logical block index.

    If a physical block is not mapped to by any logical block,
    its value in the result will be -1.

    IMPORTANT: Garbage Value Protection
    ────────────────────────────────────
    The block_table tensor may contain garbage values in unused positions
    (beyond the actual sequence length). For example, if a sequence only
    needs 3 blocks but the table has space for 8:

        block_table[0] = [10, 25, 7, 999, 1234, 888, ...]
                                    ^^^^^^^^^^^^^^^^^^^^
                                    garbage values

    These garbage values can cause issues because:
# ... omitted for brevity ...
    max_reqs, max_num_blocks = block_table.shape
    device = block_table.device

    physical_to_logical = torch.full(
        (max_reqs, total_blocks), -1, dtype=torch.long, device=device
    )

    # Only process valid blocks to avoid garbage values
    num_blocks_per_seq: torch.Tensor = cdiv(seq_lens, block_size)
    mask = (
        torch.arange(max_num_blocks, device=device)[None, :]
        < num_blocks_per_seq[:, None]
    )

    valid_block_table = torch.where(mask, block_table, 0)
    valid_logical_indices = torch.where(
        mask, torch.arange(max_num_blocks, device=device)[None, :], 0
    )

    physical_to_logical.scatter_reduce_(
        -1, valid_block_table.to(torch.int64), valid_logical_indices, reduce="amax"
    )
    # NB - Seems like block 0 is always empty so we reset it manually
    physical_to_logical[:, 0] = -1
    return physical_to_logical
```
**EN:** This function implements `physical_to_logical_mapping` within the module. The docstring frames it as: Creates an inverse mapping from physical block locations to logical indices. Key calls include `full`, `cdiv`, `where`, `scatter_reduce_`, `to`, `arange`. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `physical_to_logical_mapping`，其作用域位于the module。 关键调用包括 `full`, `cdiv`, `where`, `scatter_reduce_`, `to`, `arange`。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `unique_static_unsorted` function / `unique_static_unsorted` 函数
```python
def unique_static_unsorted(
    x: torch.Tensor,
    *,
    M: int,  # maximum positive value (0 is “skip me”)
    dim: int = -1,  # axis along which to deduplicate
    ignored_val: int = 0,  # value to ignore
    pad_val: int = -1,  # sentinel for unused slots
) -> torch.Tensor:
    """
    - Keeps the first occurrence of each non-zero value while preserving order,
      then left-packs those uniques and fills the rest with `pad_val`.
    - Returns (packed, keep_mask) with the *same shape* as `x`.
    - Requires that all values be in the range [0, M]
    - Skips ignored_val

    Works on CPU or GPU, no Python loops, O(B·N) time / O(B·M) memory.

    Example:
    x =[3, 1, 0, 1, 2], M=3, ignored_val=0 => [3, 1, 2, -1, -1]
    """
    if not (-1 <= pad_val <= M):
        raise ValueError("`pad_val` must lie in [-1, M]")

    # ── move `dim` to the end so we can treat tensor as [B, N] ──────────
    dim = dim % x.ndim
    x_perm = x.movedim(dim, -1)  # shape [..., N]
    B, N = x_perm.numel() // x_perm.shape[-1], x_perm.shape[-1]
    x_flat = x_perm.reshape(B, N)  # [B, N]

    device = x.device
    idx = torch.arange(N, device=device).expand(B, N)  # per-row indices

    # ── build first-occurrence table for every v ∈ [0, M] ───────────────
    first_idx = torch.full((B, M + 1), N, device=device)  # “∞”
    # scatter_reduce_: first_idx[b, v] = min(first_idx[b, v], i) for each i
    first_idx.scatter_reduce_(1, x_flat, idx, reduce="amin")

    # ── keep mask: first occurrence *and* value ≠ 0 ─────────────────────
    keep = (x_flat != ignored_val) & (idx == first_idx.gather(1, x_flat))  # [B, N]

    # ── left-pack uniques into a fresh tensor ───────────────────────────
    # Route non-kept entries to a garbage slot at column N so we can do a
    # single scatter rather than using torch.nonzero (which would force a
    # GPU->CPU sync to enumerate kept positions).
    dest_pos = torch.cumsum(keep.to(torch.long), dim=1) - 1  # where to go
    dest_pos = torch.where(keep, dest_pos, N)
    packed_extended = torch.full((B, N + 1), pad_val, device=device, dtype=x_flat.dtype)
    packed_flat = packed_extended.scatter_(1, dest_pos, x_flat)[:, :N]

    # ── restore original layout ─────────────────────────────────────────
    packed = packed_flat.reshape(x_perm.shape).movedim(-1, dim)
    return packed
```
**EN:** This function implements `unique_static_unsorted` within the module. The docstring frames it as: - Keeps the first occurrence of each non-zero value while preserving order, then left-packs those uniques and fills the rest with `pad_val`. Key calls include `movedim`, `reshape`, `expand`, `full`, `scatter_reduce_`, `where`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `unique_static_unsorted`，其作用域位于the module。 关键调用包括 `movedim`, `reshape`, `expand`, `full`, `scatter_reduce_`, `where`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `causal_mask_mod` function / `causal_mask_mod` 函数
```python
def causal_mask_mod(
    b: torch.Tensor, h: torch.Tensor, q_idx: torch.Tensor, kv_idx: torch.Tensor
):
    return q_idx >= kv_idx
```
**EN:** This function implements `causal_mask_mod` within the module.
**CN:** 该函数会实现 `causal_mask_mod`，其作用域位于the module。

### `bidirectional_mask_mod` function / `bidirectional_mask_mod` 函数
```python
def bidirectional_mask_mod(
    b: torch.Tensor, h: torch.Tensor, q_idx: torch.Tensor, kv_idx: torch.Tensor
):
    return q_idx >= 0
```
**EN:** This function implements `bidirectional_mask_mod` within the module.
**CN:** 该函数会实现 `bidirectional_mask_mod`，其作用域位于the module。

### Module constants / 模块常量
```python
_block_sparsity_hint_signature = Callable[
    [torch.Tensor, torch.Tensor, int], torch.Tensor
]
```
**EN:** Defines module-level constants or aliases such as `_block_sparsity_hint_signature`, which are reused by later definitions.
**CN:** 定义 `_block_sparsity_hint_signature` 等模块级常量或别名，供后续定义复用。

### `BlockSparsityHint` class / `BlockSparsityHint` 类
```python
class BlockSparsityHint(NamedTuple):
    """This prunes KV blocks from the BlockMask before the flex_attention kernel
    is invoked, so that blocks that are fully masked never get loaded.
    Use this with custom mask_mods that are sparse to avoid
    the kernel iterating over all KV blocks unnecessarily.

    Attributes:
        hint_fn: (q_block_idx [num_tokens, 1], kv_block_idx [1, num_kv_blocks],
            block_size int) -> bool Tensor [num_tokens, num_kv_blocks].
            Returns True for block pairs that may contain non-masked elements.
    """

    hint_fn: _block_sparsity_hint_signature
```
**EN:** Introduces the `BlockSparsityHint` class on top of `NamedTuple`. Core methods include its methods defined below. Docstring signal: This prunes KV blocks from the BlockMask before the flex_attention kernel is invoked, so that blocks that are fully masked never get loaded.
**CN:** 这里定义 `BlockSparsityHint` 类，其基类包括 `NamedTuple`。核心方法包括 下方定义的方法。

### `copy_to_persistent` function / `copy_to_persistent` 函数
```python
def copy_to_persistent(dst, src):
    sliced = dst[tuple(slice(0, s) for s in src.shape)]
    sliced.copy_(src)
    return sliced
```
**EN:** This function implements `copy_to_persistent` within the module. Key calls include `copy_`, `tuple`, `slice`.
**CN:** 该函数会实现 `copy_to_persistent`，其作用域位于the module。 关键调用包括 `copy_`, `tuple`, `slice`。

### `FlexAttentionMetadata` class / `FlexAttentionMetadata` 类
```python
@dataclass
class FlexAttentionMetadata:
    causal: bool
    num_actual_tokens: int  # Number of tokens excluding padding.
    max_query_len: int
    query_start_loc: torch.Tensor
    # CPU-resident copy of query_start_loc used to derive doc_ids without a
    # GPU->CPU sync from repeat_interleave's data-dependent output size.
    query_start_loc_cpu: torch.Tensor
    max_seq_len: int
    seq_lens: torch.Tensor
    block_table: torch.Tensor
    slot_mapping: torch.Tensor

    use_cascade: bool
    common_prefix_len: int
    cu_prefix_query_lens: torch.Tensor | None
    prefix_kv_lens: torch.Tensor | None
    suffix_kv_lens: torch.Tensor | None

    # Block info
    total_cache_tokens: int
    block_size: int
    max_possible_sequence_length: int
    num_reqs: int
    physical_to_logical: torch.Tensor
    decode_offset: torch.Tensor
    num_blocks_per_seq: torch.Tensor
    persistent_kv_indices: torch.Tensor
    persistent_kv_num_blocks: torch.Tensor
    persistent_doc_ids: torch.Tensor

    # For logging.
    num_input_tokens: int = 0  # Number of tokens including padding.

    # Flex Metadata
    num_blocks = 0
    block_mask: BlockMask | None = None
    score_mod: _score_mod_signature | None = None
    logical_mask_mod: _mask_mod_signature = causal_mask_mod
    uses_paged_kv: bool = True
    doc_ids: torch.Tensor | None = None
    direct_build: bool = True
    q_block_size: int = 16
    kv_block_size: int = 16
    transformed_score_mod: _score_mod_signature | None = None
    sliding_window: int | None = None
    mm_prefix_range: dict[int, list[tuple[int, int]]] | None = None
    block_sparsity_hint: BlockSparsityHint | None = None
```
**EN:** Uses `@dataclass` to package related state for `FlexAttentionMetadata`. Typical fields include `causal`, `num_actual_tokens`, `max_query_len`, `query_start_loc`, `query_start_loc_cpu`, `max_seq_len`.
**CN:** `FlexAttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `causal`, `num_actual_tokens`, `max_query_len`, `query_start_loc`, `query_start_loc_cpu`, `max_seq_len`。

### `FlexAttentionMetadata._convert_physical_to_logical` method / `FlexAttentionMetadata._convert_physical_to_logical` 方法
```python
    def _convert_physical_to_logical(
        self,
        request_lookup: torch.Tensor,
        q_idx: torch.Tensor,
        physical_kv_idx: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        """Convert physical indices to logical indices for both query and kv.

        NB is_within_lower_bound: do sequences start on block_boundaries?

        Returns:
            tuple of (is_valid, logical_q_idx, logical_kv_idx)
        """
        # Map query indices to corresponding request indices
        q_req = request_lookup[q_idx]

        # Convert physical KV indices to logical indices
        physical_kv_block = physical_kv_idx // self.block_size
        physical_kv_offset = physical_kv_idx % self.block_size
        logical_block_idx = self.physical_to_logical[q_req, physical_kv_block]
        logical_kv_idx = logical_block_idx * self.block_size + physical_kv_offset

        # Determine valid kv indices
        live_block = logical_block_idx >= 0
        within_upper_bound = logical_kv_idx < self.seq_lens[q_req]
        within_lower_bound = logical_kv_idx >= 0
        is_valid = live_block & within_upper_bound & within_lower_bound

        # Convert physical query indices to logical indices
        local_q_idx = q_idx - self.query_start_loc[q_req]
        logical_q_idx = local_q_idx + self.decode_offset[q_req]

        return is_valid, logical_q_idx, logical_kv_idx
```
**EN:** This method implements `_convert_physical_to_logical` within `FlexAttentionMetadata`. The docstring frames it as: Convert physical indices to logical indices for both query and kv.
**CN:** 该方法会实现 `_convert_physical_to_logical`，其作用域位于`FlexAttentionMetadata`。

### `FlexAttentionMetadata.get_prefix_lm_mask_mod` method / `FlexAttentionMetadata.get_prefix_lm_mask_mod` 方法
```python
    def get_prefix_lm_mask_mod(self) -> _mask_mod_signature:
        """Creates the prefix LM mask_mod function for FlexAttention."""

        assert self.doc_ids is not None
        request_lookup = self.doc_ids

        def prefix_lm_mask_mod(
            b: torch.Tensor,
            h: torch.Tensor,
            cu_q_idx: torch.Tensor,
            q_idx: torch.Tensor,
            kv_idx: torch.Tensor,
        ):
            mask = torch.zeros_like(q_idx, dtype=torch.bool)
            for req, doc_range_lst in (self.mm_prefix_range or {}).items():
                req_mask = request_lookup[cu_q_idx] == req
                for start, end in doc_range_lst:
                    doc_mask_q = (q_idx >= start) & (q_idx <= end)
                    doc_mask_kv = (kv_idx >= start) & (kv_idx <= end)
                    mask = mask | (req_mask & doc_mask_q & doc_mask_kv)
            return mask

        def final_mask_mod(
            b: torch.Tensor,
            h: torch.Tensor,
            q_idx: torch.Tensor,
            physical_kv_idx: torch.Tensor,
        ) -> torch.Tensor:
            (is_valid, logical_q_idx, logical_kv_idx) = (
                self._convert_physical_to_logical(self.doc_ids, q_idx, physical_kv_idx)
            )
            return torch.where(
                is_valid,
                prefix_lm_mask_mod(b, h, q_idx, logical_q_idx, logical_kv_idx),
                False,
            )

        return final_mask_mod
```
**EN:** This method returns or derives a value within `FlexAttentionMetadata`. The docstring frames it as: Creates the prefix LM mask_mod function for FlexAttention. Key calls include `zeros_like`, `items`, `_convert_physical_to_logical`, `where`, `prefix_lm_mask_mod`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlexAttentionMetadata`。 关键调用包括 `zeros_like`, `items`, `_convert_physical_to_logical`, `where`, `prefix_lm_mask_mod`。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `FlexAttentionMetadata._build_block_mask_direct` method / `FlexAttentionMetadata._build_block_mask_direct` 方法
```python
    def _build_block_mask_direct(self) -> BlockMask:
        """Direct block mask construction for paged KV cache attention.

        This method constructs the block mask directly using
        BlockMask.from_kv_blocks which is much more efficient than the
        generic create_block_mask approach.

        The direct path works as follows:
        1. For each query token, fetch blocks from block_table using max_seq_len
           and exclude out of sliding window blocks if needed.
           (this fetches more blocks than needed for shorter sequences)
        2. Group query tokens into chunks of q_block_size
        3. For each group, deduplicate the blocks using unique_static_unsorted
        4. Create BlockMask using the deduplicated block indices

        Over-estimation occurs when a group of q_block_size tokens contains
        multiple sequence IDs (doc_ids). In this case, we fetch ALL blocks for
        each sequence represented in the group, even though individual query
        tokens may only need a subset of those blocks based on causal masking
        and their position.

        """
        page_to_block_ratio = self.kv_block_size // self.block_size
        if page_to_block_ratio != 1:
            raise ValueError(
                f"FlexAttention currently requires the cache block size "
                f"({self.block_size}) to be equal to the kv_block_size "
                f"({self.kv_block_size}). Please check your model's "
                f"configuration."
            )

        used_pages = self.block_table[
            self.doc_ids, : cdiv(self.max_seq_len, self.block_size)
        ]

        custom_hint = self.block_sparsity_hint is not None

        if self.sliding_window or custom_hint:
            device = used_pages.device
            assert self.doc_ids is not None
            token_indices = torch.arange(
                self.doc_ids.shape[0], device=device, dtype=torch.long
            )
            logical_q_idx = (
                token_indices
                - self.query_start_loc[self.doc_ids]
                + self.decode_offset[self.doc_ids]
            )

            if self.sliding_window:
    # ... omitted for brevity ...
            used_pages_padded.shape[0] // self.q_block_size, -1
        )
        used_pages_padded = used_pages_padded // page_to_block_ratio
        kv_indices = unique_static_unsorted(
            (used_pages_padded.long()), M=self.num_blocks
        ).to(torch.int32)
        kv_indices = copy_to_persistent(self.persistent_kv_indices, kv_indices)

        kv_num_blocks = (kv_indices >= 0).sum(dim=-1).to(torch.int32)
        kv_num_blocks = copy_to_persistent(self.persistent_kv_num_blocks, kv_num_blocks)

        block_mask_kwargs = {
            "seq_lengths": (self.num_actual_tokens, self.total_cache_tokens),
            "kv_num_blocks": kv_num_blocks[None, None],
            "kv_indices": kv_indices[None, None],
            "full_kv_num_blocks": None,
            "full_kv_indices": None,
            "BLOCK_SIZE": (self.q_block_size, self.kv_block_size),
            "mask_mod": self.mask_mod,
        }

        # compute_q_blocks parameter is available in PyTorch 2.9+
        if is_torch_equal_or_newer("2.9.0.dev0"):
            block_mask_kwargs["compute_q_blocks"] = False
        return BlockMask.from_kv_blocks(**block_mask_kwargs)
```
**EN:** This method implements `_build_block_mask_direct` within `FlexAttentionMetadata`. The docstring frames it as: Direct block mask construction for paged KV cache attention. Key calls include `pad_to_multiple`, `reshape`, `to`, `copy_to_persistent`, `is_torch_equal_or_newer`, `from_kv_blocks`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_build_block_mask_direct`，其作用域位于`FlexAttentionMetadata`。 关键调用包括 `pad_to_multiple`, `reshape`, `to`, `copy_to_persistent`, `is_torch_equal_or_newer`, `from_kv_blocks`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `FlexAttentionMetadata.build_block_mask` method / `FlexAttentionMetadata.build_block_mask` 方法
```python
    def build_block_mask(self) -> BlockMask:
        mask_mod = self.get_mask_mod()
        kv_len = (
            self.total_cache_tokens if self.uses_paged_kv else self.num_actual_tokens
        )
        return create_block_mask_compiled(
            mask_mod,
            None,
            None,
            self.num_actual_tokens,
            kv_len,
            device=self.block_table.device,
            BLOCK_SIZE=(self.q_block_size, self.kv_block_size),
        )
```
**EN:** This method builds derived structures within `FlexAttentionMetadata`. Key calls include `get_mask_mod`, `create_block_mask_compiled`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`FlexAttentionMetadata`。 关键调用包括 `get_mask_mod`, `create_block_mask_compiled`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlexAttentionMetadataBuilder` class / `FlexAttentionMetadataBuilder` 类
```python
class FlexAttentionMetadataBuilder(AttentionMetadataBuilder[FlexAttentionMetadata]):
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.ALWAYS
```
**EN:** Introduces the `FlexAttentionMetadataBuilder` class on top of `AttentionMetadataBuilder[FlexAttentionMetadata]`. Core methods include `__init__`, `_get_block_sizes`, `build_for_cudagraph_capture`, `build`, `use_cascade_attention`.
**CN:** 这里定义 `FlexAttentionMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[FlexAttentionMetadata]`。核心方法包括 `__init__`, `_get_block_sizes`, `build_for_cudagraph_capture`, `build`, `use_cascade_attention`。

### `FlexAttentionMetadataBuilder.__init__` method / `FlexAttentionMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)

        self.model_config = vllm_config.model_config
        self.parallel_config = vllm_config.parallel_config
        self.cache_config = vllm_config.cache_config

        self.num_heads_q = self.model_config.get_num_attention_heads(
            self.parallel_config
        )
        self.num_heads_kv = self.model_config.get_num_kv_heads(self.parallel_config)
        self.headdim = self.model_config.get_head_size()
        self.block_size = kv_cache_spec.block_size
        self.kv_cache_spec = kv_cache_spec
        supports_small_blocks = is_torch_equal_or_newer("2.9.0.dev0")
        self.direct_build: bool = supports_small_blocks

        self.q_block_size, self.kv_block_size = self._get_block_sizes(
            vllm_config.attention_config,
            supports_small_blocks,
            self.block_size,
        )

        if self.direct_build and self.kv_block_size != self.block_size:
            self.direct_build = False

        self.max_model_len = self.model_config.max_model_len
        max_num_seqs = vllm_config.scheduler_config.max_num_seqs
        max_num_batched_tokens = vllm_config.scheduler_config.max_num_batched_tokens
        self.max_num_query_groups = cdiv(max_num_batched_tokens, self.q_block_size)
        max_num_pages_per_seq = cdiv(self.max_model_len, self.block_size)
        self.max_num_kv_indices = self.q_block_size * max_num_pages_per_seq
        self.persistent_kv_num_blocks = torch.empty(
            self.max_num_query_groups, dtype=torch.int32, device=device
        )
        self.persistent_offset_tensor = torch.empty(
            max_num_seqs, dtype=torch.int32, device=device
        )
        self.persistent_doc_ids = torch.empty(
            max_num_batched_tokens, dtype=torch.int32, device=device
        )

        # initialize later when we can access block_table
        self.persistent_physical_to_logical = None
        self.persistent_kv_indices = None
```
**EN:** This method initializes the object state within `FlexAttentionMetadataBuilder`. Key calls include `__init__`, `get_num_attention_heads`, `get_num_kv_heads`, `get_head_size`, `is_torch_equal_or_newer`, `_get_block_sizes`. It touches state such as `model_config`, `parallel_config`, `cache_config`, `num_heads_q`, `num_heads_kv`, `headdim`, `block_size`, `kv_cache_spec`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`FlexAttentionMetadataBuilder`。 关键调用包括 `__init__`, `get_num_attention_heads`, `get_num_kv_heads`, `get_head_size`, `is_torch_equal_or_newer`, `_get_block_sizes`。 它会读写 `model_config`, `parallel_config`, `cache_config`, `num_heads_q`, `num_heads_kv`, `headdim`, `block_size`, `kv_cache_spec` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlexAttentionMetadataBuilder._get_block_sizes` method / `FlexAttentionMetadataBuilder._get_block_sizes` 方法
```python
    @staticmethod
    def _get_block_sizes(
        attn_cfg,
        supports_small_blocks: bool,
        cache_block_size: int,
    ) -> tuple[int, int]:
        q_block_size = 16 if supports_small_blocks else 128
        kv_block_size = cache_block_size if supports_small_blocks else 128

        q_block_size = attn_cfg.flex_attn_q_block_size or q_block_size
        if (q_block_size & (q_block_size - 1)) != 0 or (
            attn_cfg.flex_attn_block_m is not None
            and q_block_size % attn_cfg.flex_attn_block_m != 0
        ):
            raise ValueError(
                f"flex_attn_q_block_size must be a power of 2 "
                f"and divisible by flex_attn_block_m, got "
                f"{q_block_size}, {attn_cfg.flex_attn_block_m}"
            )

        kv_block_size = attn_cfg.flex_attn_kv_block_size or kv_block_size
        if (kv_block_size & (kv_block_size - 1)) != 0 or (
            attn_cfg.flex_attn_block_n is not None
            and kv_block_size % attn_cfg.flex_attn_block_n != 0
        ):
            raise ValueError(
                f"flex_attn_kv_block_size must be a power of 2 "
                f"and divisible by flex_attn_block_n, got "
                f"{kv_block_size}, {attn_cfg.flex_attn_block_n}"
            )

        return q_block_size, kv_block_size
```
**EN:** This method implements `_get_block_sizes` within `FlexAttentionMetadataBuilder`. Key calls include `ValueError`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_get_block_sizes`，其作用域位于`FlexAttentionMetadataBuilder`。 关键调用包括 `ValueError`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlexAttentionMetadataBuilder.build_for_cudagraph_capture` method / `FlexAttentionMetadataBuilder.build_for_cudagraph_capture` 方法
```python
    def build_for_cudagraph_capture(
        self, common_attn_metadata: CommonAttentionMetadata
    ) -> FlexAttentionMetadata:
        # Use actual max_seq_len (not max_model_len) to avoid torch.compile
        # recompilation during CUDA graph capture.
        assert common_attn_metadata.seq_lens_cpu_upper_bound is not None
        common_attn_metadata.max_seq_len = int(
            common_attn_metadata.seq_lens_cpu_upper_bound.max().item()
        )
        return self.build(
            common_prefix_len=0, common_attn_metadata=common_attn_metadata
        )
```
**EN:** This method builds derived structures within `FlexAttentionMetadataBuilder`. Key calls include `int`, `build`, `item`, `max`.
**CN:** 该方法会构建派生结构，其作用域位于`FlexAttentionMetadataBuilder`。 关键调用包括 `int`, `build`, `item`, `max`。

### `FlexAttentionMetadataBuilder.build` method / `FlexAttentionMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> FlexAttentionMetadata:
        num_reqs = common_attn_metadata.num_reqs
        num_actual_tokens = common_attn_metadata.num_actual_tokens
        max_query_len = common_attn_metadata.max_query_len

        max_seq_len = common_attn_metadata.max_seq_len
        query_start_loc = common_attn_metadata.query_start_loc
        query_start_loc_cpu = common_attn_metadata.query_start_loc_cpu
        seq_lens = common_attn_metadata.seq_lens
        block_table_tensor = common_attn_metadata.block_table_tensor
        slot_mapping = common_attn_metadata.slot_mapping
        num_blocks_per_seq = cdiv(seq_lens, self.block_size)

        use_cascade = common_prefix_len > 0
        cu_prefix_query_lens = None
        prefix_kv_lens = None
        suffix_kv_lens = None
        if use_cascade:
            raise NotImplementedError(
                "Cascade prefix attention is not yet implemented "
                "for FlexAttention backend"
            )

        block_size = self.kv_cache_spec.block_size
        max_possible_seq_len = self.model_config.max_model_len
        num_gpu_blocks = self.cache_config.num_gpu_blocks

        assert num_gpu_blocks is not None, (
            "FlexAttention requires num_gpu_blocks to be set"
        )
        total_cache_tokens = num_gpu_blocks * block_size

        inverse_block_table = physical_to_logical_mapping(
            block_table_tensor, seq_lens, block_size, num_gpu_blocks
        )
        if self.persistent_physical_to_logical is None:
            max_num_seqs = self.vllm_config.scheduler_config.max_num_seqs
            self.persistent_physical_to_logical = torch.empty(
                max_num_seqs,
                num_gpu_blocks,
                dtype=torch.long,
                device=self.device,
            )

        if self.persistent_kv_indices is None:
    # ... omitted for brevity ...
            total_cache_tokens=total_cache_tokens,
            decode_offset=offset_tensor,
            num_blocks_per_seq=num_blocks_per_seq,
            uses_paged_kv=uses_paged_kv,
            # FIXME(Isotr0py): direct build has issue to build bidirectional
            # attention block mask for encoder-only models, disable it temporarily.
            # see: https://github.com/vllm-project/vllm/pull/27329#issuecomment-3431484053
            direct_build=self.direct_build and uses_paged_kv,
            q_block_size=self.q_block_size,
            kv_block_size=self.kv_block_size,
            persistent_kv_indices=self.persistent_kv_indices,
            persistent_kv_num_blocks=self.persistent_kv_num_blocks,
            persistent_doc_ids=self.persistent_doc_ids,
        )

        # Pre-build block_mask so it is ready before CUDA graph capture.
        # Without this, the lazy build in forward() would run non-graph-safe
        # ops (e.g. torch.nonzero) inside capture.
        if out.block_mask is None:
            if out.direct_build:
                out.block_mask = out._build_block_mask_direct()
            else:
                out.block_mask = out.build_block_mask()

        return out
```
**EN:** This method builds derived structures within `FlexAttentionMetadataBuilder`. Key calls include `cdiv`, `physical_to_logical_mapping`, `copy_to_persistent`, `compute_num_computed_tokens`, `FlexAttentionMetadata`, `NotImplementedError`. It touches state such as `persistent_physical_to_logical`, `persistent_kv_indices`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会构建派生结构，其作用域位于`FlexAttentionMetadataBuilder`。 关键调用包括 `cdiv`, `physical_to_logical_mapping`, `copy_to_persistent`, `compute_num_computed_tokens`, `FlexAttentionMetadata`, `NotImplementedError`。 它会读写 `persistent_physical_to_logical`, `persistent_kv_indices` 等状态。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `FlexAttentionImpl` class / `FlexAttentionImpl` 类
```python
class FlexAttentionImpl(AttentionImpl):
    sliding_window: int | None
    alibi_slopes: torch.Tensor | None
    logits_soft_cap: float | None
    mm_prefix_range: dict[int, list[tuple[int, int]]] | None = None
    logical_mask_mod: _mask_mod_signature | None = None
    block_sparsity_hint: BlockSparsityHint | None = None
```
**EN:** Introduces the `FlexAttentionImpl` class on top of `AttentionImpl`. Core methods include `__init__`, `view_as_4d`, `do_kv_cache_update`, `forward`.
**CN:** 这里定义 `FlexAttentionImpl` 类，其基类包括 `AttentionImpl`。核心方法包括 `__init__`, `view_as_4d`, `do_kv_cache_update`, `forward`。

### `FlexAttentionImpl.__init__` method / `FlexAttentionImpl.__init__` 方法
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
        logits_soft_cap: float | None = None,
        attn_type: AttentionType = AttentionType.DECODER,
        kv_sharing_target_layer_name: str | None = None,
        block_m: int | None = None,
        block_n: int | None = None,
        **kwargs,
    ) -> None:
        self.num_heads = num_heads
        self.head_size = head_size
        self.scale = float(scale)
        self.num_kv_heads = num_kv_heads
        self.attn_type = attn_type

        if attn_type not in (AttentionType.ENCODER_ONLY, AttentionType.DECODER):
            raise NotImplementedError(
                f"FlexAttention does not support {attn_type} attention"
            )

        if alibi_slopes is not None:
            raise NotImplementedError(
                "FlexAttention does not support alibi slopes yet."
            )
        else:
            self.alibi_slopes = None

        self.sliding_window = sliding_window

        self.kv_cache_dtype = kv_cache_dtype
        self.logits_soft_cap = logits_soft_cap
        if self.logits_soft_cap is not None:
            raise NotImplementedError(
                "FlexAttention does not support logits soft cap yet."
            )

        assert self.num_heads % self.num_kv_heads == 0
        self.num_queries_per_kv = self.num_heads // self.num_kv_heads

        if kv_sharing_target_layer_name is not None:
            raise NotImplementedError("FlexAttention does not support kv sharing yet.")

        if is_quantized_kv_cache(self.kv_cache_dtype):
            raise NotImplementedError(
                "FlexAttention does not support quantized kv-cache. Yet"
            )

        self.block_m = 16 if envs.VLLM_BATCH_INVARIANT else None
        self.block_n = 16 if envs.VLLM_BATCH_INVARIANT else None

        if block_m is not None:
            self.block_m = block_m
        if block_n is not None:
            self.block_n = block_n
```
**EN:** This method initializes the object state within `FlexAttentionImpl`. Key calls include `float`, `is_quantized_kv_cache`, `NotImplementedError`. It touches state such as `num_heads`, `head_size`, `scale`, `num_kv_heads`, `attn_type`, `sliding_window`, `kv_cache_dtype`, `logits_soft_cap`. The control flow contains 9 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`FlexAttentionImpl`。 关键调用包括 `float`, `is_quantized_kv_cache`, `NotImplementedError`。 它会读写 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `attn_type`, `sliding_window`, `kv_cache_dtype`, `logits_soft_cap` 等状态。 控制流包含 9 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlexAttentionImpl.view_as_4d` method / `FlexAttentionImpl.view_as_4d` 方法
```python
    @staticmethod
    def view_as_4d(tensor: torch.Tensor) -> torch.Tensor:
        """View a 3d tensor as 4D."""
        if tensor.ndim == 4:
            return tensor
        assert tensor.ndim == 3
        return tensor[None, :, :, :]
```
**EN:** This method implements `view_as_4d` within `FlexAttentionImpl`. The docstring frames it as: View a 3d tensor as 4D. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `view_as_4d`，其作用域位于`FlexAttentionImpl`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlexAttentionImpl.do_kv_cache_update` method / `FlexAttentionImpl.do_kv_cache_update` 方法
```python
    def do_kv_cache_update(
        self,
        layer: torch.nn.Module,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        slot_mapping: torch.Tensor,
    ) -> None:
        if self.attn_type == AttentionType.ENCODER_ONLY:
            return

        key_cache, value_cache = kv_cache.unbind(0)
        torch.ops._C_cache_ops.reshape_and_cache_flash(
            key,
            value,
            key_cache,
            value_cache,
            slot_mapping,
            self.kv_cache_dtype,
            layer._k_scale,
            layer._v_scale,
        )
```
**EN:** This method implements `do_kv_cache_update` within `FlexAttentionImpl`. Key calls include `unbind`, `reshape_and_cache_flash`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_kv_cache_update`，其作用域位于`FlexAttentionImpl`。 关键调用包括 `unbind`, `reshape_and_cache_flash`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlexAttentionImpl.forward` method / `FlexAttentionImpl.forward` 方法
```python
    def forward(
        self,
        layer: torch.nn.Module,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: FlexAttentionMetadata,
        output: torch.Tensor,
        output_scale: torch.Tensor | None = None,
        output_block_scale: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Forward pass with FLexAttention.

        Args:
            query: shape = [num_tokens, num_heads, head_size]
            key: shape = [num_tokens, num_kv_heads, head_size]
            value: shape = [num_tokens, num_kv_heads, head_size]
            kv_cache: shape =
                [2, num_blocks, block_size, num_kv_heads, head_size]
            attn_metadata: Metadata for attention.
        Returns:
            shape = [num_tokens, num_heads * head_size]
        """
        if output_scale is not None or output_block_scale is not None:
            raise NotImplementedError(
                "fused output quantization is not yet supported for FlexAttentionImpl"
            )

        enable_gqa = self.num_kv_heads != self.num_heads

        if attn_metadata is None:
            # Profiling run.
            return output.fill_(0)
            # query = self.view_as_4d(query).permute(0, 2, 1, 3)
            # return torch.empty_like(query)

        num_actual_tokens = attn_metadata.num_actual_tokens

        needs_rebuild_block_mask = False
        if attn_metadata.sliding_window != self.sliding_window:
            attn_metadata.sliding_window = self.sliding_window
            if attn_metadata.direct_build:
                # update mask mod in attention metadata
                attn_metadata.mask_mod = attn_metadata.get_mask_mod()
            needs_rebuild_block_mask = True

        if self.mm_prefix_range != getattr(attn_metadata, "mm_prefix_range", None):
            self.mm_prefix_range = attn_metadata.mm_prefix_range
            attn_metadata.mask_mod = attn_metadata.get_mask_mod()
    # ... omitted for brevity ...
        kernel_options = get_kernel_options(
            query, block_m, block_n, attn_metadata.direct_build
        )

        if self.block_m is not None:
            kernel_options["BLOCK_M"] = self.block_m
        if self.block_n is not None:
            kernel_options["BLOCK_N"] = self.block_n
        if envs.VLLM_BATCH_INVARIANT:
            kernel_options["IS_DIVISIBLE"] = False
        out = flex_attention_compiled(
            query,
            key_tensor,
            value_tensor,
            attn_metadata.transformed_score_mod,
            attn_metadata.block_mask,
            self.scale,
            enable_gqa=enable_gqa,
            kernel_options=kernel_options,
        )

        # Flex doesn't have an out variant today, rely on epilogue fusion
        out = out.permute(0, 2, 1, 3).squeeze(0)
        output[:num_actual_tokens, :, :].copy_(out)
        return output
```
**EN:** This method drives the forward-pass computation within `FlexAttentionImpl`. The docstring frames it as: Forward pass with FLexAttention. Key calls include `getattr`, `get_kernel_options`, `flex_attention_compiled`, `squeeze`, `copy_`, `NotImplementedError`. It touches state such as `mm_prefix_range`. The control flow contains 14 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会驱动前向计算流程，其作用域位于`FlexAttentionImpl`。 关键调用包括 `getattr`, `get_kernel_options`, `flex_attention_compiled`, `squeeze`, `copy_`, `NotImplementedError`。 它会读写 `mm_prefix_range` 等状态。 控制流包含 14 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `get_kernel_options` function / `get_kernel_options` 函数
```python
def get_kernel_options(
    query, block_m, block_n, use_direct_build: bool
) -> dict[str, int | bool]:
    kernel_options: dict[str, int | bool] = {
        "FORCE_USE_FLEX_ATTENTION": True,
    }

    def ensure_divisible(candidate: int, block_size: int) -> int:
        """Pick a kernel block size that divides the logical block."""
        if block_size <= 0:
            return candidate
        candidate = min(candidate, block_size)
        if candidate <= 0:
            return block_size
        if block_size % candidate == 0:
            return candidate

        candidate = math.gcd(candidate, block_size)
        if candidate <= 1:
            return block_size
        return candidate

    if use_direct_build:
        kernel_options["BLOCK_M"] = block_m
        kernel_options["BLOCK_N"] = block_n
        return kernel_options
    else:
        preferred_block = 32 if query.dtype == torch.float32 else 64
        block_lower_bound = 16

        block_m_candidate = ensure_divisible(preferred_block, block_m)
        block_n_candidate = ensure_divisible(preferred_block, block_n)

        if torch.cuda.is_available():
            device_props = torch.cuda.get_device_properties()
            # ROCm doesn't expose shared_memory_per_block_optin attribute
            # AMD GPUs typically have 64KB LDS (Local Data Share) per workgroup
            if hasattr(device_props, "shared_memory_per_block_optin"):
                max_shared_memory = device_props.shared_memory_per_block_optin
            elif current_platform.is_rocm():
                # ROCm fallback: use 64KB
                max_shared_memory = 65536
            else:
                raise RuntimeError(
                    "Unable to determine shared memory size on this hardware."
                )

            if max_shared_memory < 144 * 1024:
                block_m_candidate = ensure_divisible(
                    max(1, block_m_candidate // 2), block_m
                )
                block_n_candidate = ensure_divisible(
                    max(1, block_n_candidate // 2), block_n
                )

        block_m_candidate = max(block_m_candidate, block_lower_bound)
        block_n_candidate = max(block_n_candidate, block_lower_bound)

        kernel_options["BLOCK_M"] = block_m_candidate
        kernel_options["BLOCK_N"] = block_n_candidate

    return kernel_options
```
**EN:** This function returns or derives a value within the module. Key calls include `min`, `gcd`, `ensure_divisible`, `is_available`, `max`, `get_device_properties`. The control flow contains 10 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `min`, `gcd`, `ensure_divisible`, `is_available`, `max`, `get_device_properties`。 控制流包含 10 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_offsets_to_doc_ids_tensor`: top-level helper or orchestration entry point. / `_offsets_to_doc_ids_tensor`：顶层辅助函数或编排入口。
- `pad_to_multiple`: top-level helper or orchestration entry point. / `pad_to_multiple`：顶层辅助函数或编排入口。
- `FlexAttentionBackend`: central class or interface in this module. / `FlexAttentionBackend`：本模块中的核心类或接口。
- `physical_to_logical_mapping`: top-level helper or orchestration entry point. / `physical_to_logical_mapping`：顶层辅助函数或编排入口。
- `unique_static_unsorted`: top-level helper or orchestration entry point. / `unique_static_unsorted`：顶层辅助函数或编排入口。
- `causal_mask_mod`: top-level helper or orchestration entry point. / `causal_mask_mod`：顶层辅助函数或编排入口。
- `bidirectional_mask_mod`: top-level helper or orchestration entry point. / `bidirectional_mask_mod`：顶层辅助函数或编排入口。
- `BlockSparsityHint`: central class or interface in this module. / `BlockSparsityHint`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `math`, `collections`, `dataclasses`, `functools`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config`, `vllm.config.cache`, `vllm.logger`, `vllm.platforms`, `vllm.utils.math_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.kv_cache_interface`
