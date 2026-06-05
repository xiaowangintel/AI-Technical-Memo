# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `is_valid_kv_cache_layout`, `get_kv_cache_layout`, `set_kv_cache_layout` for the V1 `attention/backends` subsystem. / 为 V1 的 `attention/backends` 子系统实现 `is_valid_kv_cache_layout`, `get_kv_cache_layout`, `set_kv_cache_layout`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import functools
from collections.abc import Callable
from dataclasses import dataclass, field, fields, make_dataclass
from typing import (
    TYPE_CHECKING,
    Any,
    Literal,
    Protocol,
    get_args,
)

import numpy as np
import torch
from typing_extensions import runtime_checkable

from vllm.config import VllmConfig, get_layers_from_vllm_config
from vllm.utils.math_utils import cdiv
from vllm.v1.kv_cache_interface import KVCacheSpec, MambaSpec

if TYPE_CHECKING:
    from vllm.v1.core.sched.output import SchedulerOutput
    from vllm.v1.worker.gpu_input_batch import InputBatch

import vllm.envs as envs
from vllm.distributed.kv_transfer.kv_connector.utils import (
    get_kv_connector_cache_layout,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    CommonAttentionMetadata,
    subclass_attention_backend,
)

logger = init_logger(__name__)
KVCacheLayoutType = Literal["NHD", "HND"]
_KV_CACHE_LAYOUT_OVERRIDE: KVCacheLayoutType | None = None

PAD_SLOT_ID = -1
NULL_BLOCK_ID = 0
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `KVCacheLayoutType`, `_KV_CACHE_LAYOUT_OVERRIDE`, `PAD_SLOT_ID`, `NULL_BLOCK_ID`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `KVCacheLayoutType`, `_KV_CACHE_LAYOUT_OVERRIDE`, `PAD_SLOT_ID`, `NULL_BLOCK_ID`。

### `PerLayerParameters` class / `PerLayerParameters` 类
```python
@dataclass
class PerLayerParameters:
    """
    Currently, FlashInfer backend only support models in which all layers share
    the same values for the following hyperparameters. Should not be used for
    trtllm-gen backend since it supports different values for the following
    hyperparameters.
    """

    window_left: int
    logits_soft_cap: float | None
    sm_scale: float
    has_sinks: bool = False
    # has same params for all layers
    has_same_window_lefts: bool | None = field(default=None, compare=False)
    has_same_all_params: bool | None = field(default=None, compare=False)
```
**EN:** Uses `@dataclass` to package related state for `PerLayerParameters`. Typical fields include `window_left`, `logits_soft_cap`, `sm_scale`, `has_sinks`, `has_same_window_lefts`, `has_same_all_params`.
**CN:** `PerLayerParameters` 使用 `@dataclass` 打包相关状态。典型字段包括 `window_left`, `logits_soft_cap`, `sm_scale`, `has_sinks`, `has_same_window_lefts`, `has_same_all_params`。

### `make_local_attention_virtual_batches` function / `make_local_attention_virtual_batches` 函数
```python
def make_local_attention_virtual_batches(
    attn_chunk_size: int,
    common_attn_metadata: CommonAttentionMetadata,
    block_size: int = 0,
) -> tuple[CommonAttentionMetadata, Callable[[torch.Tensor], torch.Tensor]]:
    query_start_loc_np = common_attn_metadata.query_start_loc_cpu.numpy()
    seq_lens_np = common_attn_metadata.seq_lens_cpu.numpy()
    block_table = common_attn_metadata.block_table_tensor
    device = common_attn_metadata.query_start_loc.device

    q_seqlens = query_start_loc_np[1:] - query_start_loc_np[:-1]
    actual_batch_size = seq_lens_np.shape[0]

    # Handle if we are starting in the middle of a local attention block,
    #  we assume q_seqlens > 0 (for all elements), for each batch idx we compute
    #  the number of tokens that are not in the first local attention block and
    #  then we can simply use a cdiv for the rest.
    # For example if we have:
    #   attn_chunk_size = 4
    #   q_seqlens = [4, 10, 5]
    #   k_seqlens = [6, 17, 9]
    # Then we would get:
    #   new_tokens_in_first_block = [2, 1, 4]
    #   local_blocks = [2, 4, 2]
    q_tokens_in_first_block = np.minimum(
        attn_chunk_size - ((seq_lens_np - q_seqlens) % attn_chunk_size), q_seqlens
    ).astype(np.int32)
    tokens_in_last_block = attn_chunk_size + (seq_lens_np % -attn_chunk_size)
    local_blocks = 1 + cdiv(q_seqlens - q_tokens_in_first_block, attn_chunk_size)

    # Once we know the number of local blocks we can compute the request spans
    #  for each batch idx, we can figure out the number of "virtual" requests we
    #  have to make,
    # For the above example we would get:
    #   seqlens_q_local = [2, 2, 1, 4, 4, 1, 4, 1]
    #
    # First Get batched arange. (E.g., [2, 4, 2] -> [0, 1, 0, 1, 2, 3, 0, 1])
    #   (TODO: make a utility to share this code with _prepare_inputs)
    # arange step 1. [2, 4, 2] -> [2, 6, 8]
    cu_num_blocks = np.cumsum(local_blocks)
    virtual_batches = cu_num_blocks[-1]
    # arange step 2. [2, 6, 8] -> [0, 0, 2, 2, 2, 2, 6, 6]
    block_offsets = np.repeat(cu_num_blocks - local_blocks, local_blocks)
    # arange step 3. [0, 1, 0, 1, 2, 3, 0, 1]
    arange = np.arange(virtual_batches, dtype=np.int32) - block_offsets
    # also compute reverse arange (i.e. [1, 0, 3, 2, 1, 0, 1, 0])
    rarange = np.repeat(local_blocks, local_blocks) - arange - 1
    # Then we can compute the seqlens_q_local, handling the fact that the
    #  first and last blocks could be partial
    seqlens_q_local = np.repeat(q_seqlens - q_tokens_in_first_block, local_blocks)
# ... omitted for brevity ...
    # Save as a lambda so we can return this for update_block_table
    make_block_table = lambda block_table: block_table[
        batch_indices_torch, block_indices_torch
    ].view(virtual_batches, -1)
    block_table_local = make_block_table(block_table)

    query_start_loc_cpu = torch.from_numpy(cu_seqlens_q_local)
    seq_lens_cpu = torch.from_numpy(seqlens_k_local)
    max_seq_len = int(seq_lens_cpu.max())

    return CommonAttentionMetadata(
        query_start_loc_cpu=query_start_loc_cpu,
        query_start_loc=query_start_loc_cpu.to(device=device, non_blocking=True),
        seq_lens=seq_lens_cpu.to(device=device, non_blocking=True),
        num_reqs=len(seq_lens_cpu),
        num_actual_tokens=common_attn_metadata.num_actual_tokens,
        max_query_len=seqlens_q_local.max(),
        max_seq_len=max_seq_len,
        block_table_tensor=block_table_local,
        slot_mapping=common_attn_metadata.slot_mapping,
        causal=True,
        seq_lens_cpu_upper_bound=common_attn_metadata.seq_lens_cpu_upper_bound,
        _seq_lens_cpu=seq_lens_cpu,
        _num_computed_tokens_cpu=torch.from_numpy(num_computed_tokens_local),
    ), make_block_table
```
**EN:** This function implements `make_local_attention_virtual_batches` within the module. Key calls include `numpy`, `astype`, `cumsum`, `repeat`, `empty`, `full`. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `make_local_attention_virtual_batches`，其作用域位于the module。 关键调用包括 `numpy`, `astype`, `cumsum`, `repeat`, `empty`, `full`。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `make_kv_sharing_fast_prefill_common_attn_metadata` function / `make_kv_sharing_fast_prefill_common_attn_metadata` 函数
```python
def make_kv_sharing_fast_prefill_common_attn_metadata(
    common_attn_metadata: CommonAttentionMetadata,
) -> CommonAttentionMetadata:
    if common_attn_metadata.max_query_len == 1:
        # All requests are decode (assume 1 token for now)
        # Skip computing fast prefill path
        return common_attn_metadata

    assert common_attn_metadata.logits_indices_padded is not None
    assert common_attn_metadata.num_logits_indices is not None

    logits_indices_padded = common_attn_metadata.logits_indices_padded
    num_logits_indices = common_attn_metadata.num_logits_indices
    # Get rid of CUDAGraph padding, if any
    logits_indices = logits_indices_padded[:num_logits_indices]
    num_reqs = common_attn_metadata.num_reqs
    query_start_loc = common_attn_metadata.query_start_loc
    # Example inputs
    # num_reqs: 3
    # generation_indices:  [14, 18, 19, 27]
    # query_start_loc: [0, 15, 20, 28]
    # seq_lens:        [41, 31, 40]

    # Find how many decode indices belong to each request
    # request_ids: [0, 1, 1, 2]
    request_ids = torch.bucketize(logits_indices, query_start_loc[1:], right=True)

    # Figure out how many tokens are in each request
    # num_decode_tokens: [1, 2, 1]
    # Avoid `torch.bincount` here — on CUDA it forces a sync to determine
    # the output size (even with `minlength`, the kernel must confirm no
    # value exceeds the bound). `scatter_add_` into a preallocated buffer
    # is equivalent and stays async.
    num_decode_tokens = torch.zeros(
        num_reqs, dtype=request_ids.dtype, device=request_ids.device
    )
    num_decode_tokens.scatter_add_(
        0, request_ids.to(num_decode_tokens.dtype), torch.ones_like(request_ids)
    )

    # Calculate new query_start_loc with tokens in generation_indices
    # decode_query_start_loc: [0, 1, 3, 4]
    decode_query_start_loc = torch.empty(
        num_reqs + 1, device=query_start_loc.device, dtype=query_start_loc.dtype
    )

    decode_query_start_loc[:1].fill_(0)  # Avoid sync from scalar assignment.
    decode_query_start_loc[1:] = torch.cumsum(num_decode_tokens, dim=0)
    decode_max_query_len = int(num_decode_tokens.max().item())
    total_num_decode_tokens = int(num_decode_tokens.sum().item())

    common_attn_metadata = CommonAttentionMetadata(
        query_start_loc=decode_query_start_loc,
        query_start_loc_cpu=decode_query_start_loc.to("cpu", non_blocking=True),
        seq_lens=common_attn_metadata.seq_lens,
        num_reqs=num_reqs,
        num_actual_tokens=total_num_decode_tokens,
        max_query_len=decode_max_query_len,
        max_seq_len=common_attn_metadata.max_seq_len,
        block_table_tensor=common_attn_metadata.block_table_tensor,
        slot_mapping=common_attn_metadata.slot_mapping,
        causal=True,
        seq_lens_cpu_upper_bound=common_attn_metadata.seq_lens_cpu_upper_bound,
        _seq_lens_cpu=common_attn_metadata._seq_lens_cpu,
        _num_computed_tokens_cpu=common_attn_metadata._num_computed_tokens_cpu,
    )
    return common_attn_metadata
```
**EN:** This function implements `make_kv_sharing_fast_prefill_common_attn_metadata` within the module. Key calls include `bucketize`, `zeros`, `scatter_add_`, `empty`, `fill_`, `cumsum`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `make_kv_sharing_fast_prefill_common_attn_metadata`，其作用域位于the module。 关键调用包括 `bucketize`, `zeros`, `scatter_add_`, `empty`, `fill_`, `cumsum`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `split_decodes_prefills_and_extends` function / `split_decodes_prefills_and_extends` 函数
```python
def split_decodes_prefills_and_extends(
    common_attn_metadata: CommonAttentionMetadata,
    decode_threshold: int = 1,
) -> tuple[int, int, int, int, int, int]:
    """
    Assuming a reordered batch, finds the boundary between prefill and decode
    requests.

    Args:
        common_attn_metadata: CommonAttentionMetadata object containing the
            batch metadata.
        decode_threshold: The maximum query length to be considered a decode.

    Returns:
        num_decodes: The number of decode requests.
        num_extends: The number of extend requests.
        num_prefills: The number of prefill requests.
        num_decode_tokens: The number of tokens in the decode requests.
        num_extend_tokens: The number of tokens in the extend requests.
        num_prefill_tokens: The number of tokens in the prefill requests.
    """
    max_query_len = common_attn_metadata.max_query_len
    num_reqs = common_attn_metadata.num_reqs
    num_tokens = common_attn_metadata.num_actual_tokens
    query_start_loc = common_attn_metadata.query_start_loc_cpu
    # Upper bound is exact for prefill rows; decode rows still satisfy
    # seq_len > query_len under the optimistic bound, so `seq_lens ==
    # query_lens` identifies prefills correctly either way.
    assert common_attn_metadata.seq_lens_cpu_upper_bound is not None
    seq_lens = common_attn_metadata.seq_lens_cpu_upper_bound

    if max_query_len <= decode_threshold:
        return num_reqs, 0, 0, num_tokens, 0, 0

    query_lens = query_start_loc[1:] - query_start_loc[:-1]
    is_prefill_or_extend = query_lens > decode_threshold
    is_prefill = (seq_lens == query_lens) & is_prefill_or_extend
    first_extend = is_prefill_or_extend.int().argmax(dim=-1).item()
    first_prefill = is_prefill.int().argmax(dim=-1).item()
    num_decodes = first_extend
    num_decode_tokens = query_start_loc[first_extend].item()
    if not torch.any(is_prefill_or_extend):
        return (num_decodes, 0, 0, num_decode_tokens, 0, 0)

    num_prefills_or_extends = num_reqs - num_decodes
    num_prefill_or_extend_tokens = num_tokens - num_decode_tokens
    if not torch.any(is_prefill):
        return (
            num_decodes,
            num_prefills_or_extends,
            0,
            num_decode_tokens,
            num_prefill_or_extend_tokens,
            0,
        )

    num_extends = first_prefill - num_decodes
    num_prefills = num_reqs - first_prefill

    num_prefill_tokens = num_tokens - query_start_loc[first_prefill]
    num_extend_tokens = num_prefill_or_extend_tokens - num_prefill_tokens
    return (
        num_decodes,
        num_extends,
        num_prefills,
        num_decode_tokens,
        num_extend_tokens,
        num_prefill_tokens,
    )
```
**EN:** This function implements `split_decodes_prefills_and_extends` within the module. The docstring frames it as: Assuming a reordered batch, finds the boundary between prefill and decode requests. Key calls include `item`, `any`, `argmax`, `int`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `split_decodes_prefills_and_extends`，其作用域位于the module。 关键调用包括 `item`, `any`, `argmax`, `int`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `split_decodes_and_prefills` function / `split_decodes_and_prefills` 函数
```python
def split_decodes_and_prefills(
    common_attn_metadata: CommonAttentionMetadata,
    decode_threshold: int = 1,
    require_uniform: bool = False,
    treat_short_extends_as_decodes: bool = True,
) -> tuple[int, int, int, int]:
    """
    Assuming a reordered batch, finds the boundary between prefill and decode
    requests.

    The batch is expected to be ordered as:
        decode → short_extend → long_extend → prefill

    Args:
        common_attn_metadata: CommonAttentionMetadata object containing the
            batch metadata.
        decode_threshold: The maximum query length to be considered a decode.
        require_uniform: If True, requires that all decode requests have the
            same query length. When set, some queries may be considered prefills
            even if they are <= decode_threshold, in order to ensure uniformity.
        treat_short_extends_as_decodes: If True (default), short extends
            (query_len <= threshold but still prefilling) are counted as
            decodes. If False, they are counted as prefills.

    Returns:
        num_decodes: The number of decode requests.
        num_prefills: The number of prefill requests.
        num_decode_tokens: The number of tokens in the decode requests.
        num_prefill_tokens: The number of tokens in the prefill requests.
    """
    max_query_len = common_attn_metadata.max_query_len
    num_reqs = common_attn_metadata.num_reqs
    num_tokens = common_attn_metadata.num_actual_tokens
    query_start_loc = common_attn_metadata.query_start_loc_cpu

    if (
        max_query_len <= decode_threshold
        and (not require_uniform or decode_threshold <= 1)
        and treat_short_extends_as_decodes
    ):
        return num_reqs, 0, num_tokens, 0

    query_lens = query_start_loc[1:] - query_start_loc[:-1]
    if query_lens[0].item() > decode_threshold:
        # first request is not decode, so no decode requests
        return 0, num_reqs, 0, num_tokens

    if require_uniform:
        # check if we are in a padded uniform batch; this is used for full-CGs, some
        # requests may have a query length of 0 but since they are padding its fine
        # to treat them as decodes (ensures num_decodes matches the captured size)
        if torch.all((query_lens == query_lens[0]) | (query_lens == 0)):
            return num_reqs, 0, num_tokens, 0  # all decodes
        is_prefill = query_lens != query_lens[0]
    else:
        is_prefill = query_lens > decode_threshold

    if not treat_short_extends_as_decodes:
        assert common_attn_metadata.is_prefilling is not None
        is_prefill |= common_attn_metadata.is_prefilling

    if not torch.any(is_prefill):
        return num_reqs, 0, num_tokens, 0

    first_prefill = is_prefill.int().argmax(dim=-1).item()
    num_decodes = first_prefill
    num_prefills = num_reqs - num_decodes
    num_decode_tokens = query_start_loc[first_prefill].item()
    num_prefill_tokens = num_tokens - num_decode_tokens
    return (num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens)
```
**EN:** This function implements `split_decodes_and_prefills` within the module. The docstring frames it as: Assuming a reordered batch, finds the boundary between prefill and decode requests. Key calls include `item`, `all`, `any`, `argmax`, `int`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `split_decodes_and_prefills`，其作用域位于the module。 关键调用包括 `item`, `all`, `any`, `argmax`, `int`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `split_prefill_chunks` function / `split_prefill_chunks` 函数
```python
def split_prefill_chunks(
    seq_lens_cpu: torch.Tensor, workspace_size: int, request_offset: int = 0
) -> list[tuple[int, int]]:
    """
    Split the prefill requests into chunks such that the total sequence length
    of each chunk is less than or equal to the workspace size.

    Args:
        seq_lens_cpu: The sequence lengths of the prefill requests on CPU.
        workspace_size: The maximum workspace size (in tokens) per chunk.
        request_offset: The offset to add to the request indices.
    Returns:
        A list of tuples of (reqs_start, reqs_end) representing chunk boundaries.
    """
    chunk_bounds = []
    i, n = 0, len(seq_lens_cpu)
    assert torch.all(seq_lens_cpu <= workspace_size).item()

    while i < n:
        start, chunk_total = i, 0
        while i < n and (chunk_total + (s := seq_lens_cpu[i].item())) <= workspace_size:
            chunk_total += s
            i += 1
        chunk_bounds.append((start + request_offset, i + request_offset))
    return chunk_bounds
```
**EN:** This function implements `split_prefill_chunks` within the module. The docstring frames it as: Split the prefill requests into chunks such that the total sequence length of each chunk is less than or equal to the workspace size. Key calls include `item`, `len`, `append`, `all`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `split_prefill_chunks`，其作用域位于the module。 关键调用包括 `item`, `len`, `append`, `all`。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `reorder_batch_to_split_decodes_and_prefills` function / `reorder_batch_to_split_decodes_and_prefills` 函数
```python
def reorder_batch_to_split_decodes_and_prefills(
    input_batch: "InputBatch",
    scheduler_output: "SchedulerOutput",
    decode_threshold: int = 1,
) -> bool:
    """
    Reorders the batch to split into prefill and decode requests; places all
    requests with <= decode_threshold tokens at the front of the batch.

    The batch is reordered into 4 regions:
        decode:        (num_scheduled <= threshold AND is not prefilling)
        short_extend:  (num_scheduled <= threshold AND is chunked prefilling)
        long_extend:   (num_scheduled > threshold AND is chunked prefilling)
        prefill:       (num_computed == 0)   # First chunks

    Returns:
        True if the batch was modified, False otherwise.
    """
    num_reqs = len(input_batch.req_ids)
    num_scheduled_tokens = [
        scheduler_output.num_scheduled_tokens[id] for id in input_batch.req_ids
    ]
    num_scheduled_tokens_np = np.array(num_scheduled_tokens)
    num_computed_tokens_np = input_batch.num_computed_tokens_cpu[:num_reqs]
    num_prompt_tokens_np = input_batch.num_prompt_tokens[:num_reqs]

    has_context = num_computed_tokens_np > 0
    is_below_threshold = num_scheduled_tokens_np <= decode_threshold
    done_prefilling = num_computed_tokens_np >= num_prompt_tokens_np

    # Mutually exclusive categories (exactly one True per request):
    # 1. No context yet -> prefill
    # 2. Has context, above threshold -> long_extend
    # 3. Has context, below threshold, still prefilling -> short_extend
    # 4. Has context, below threshold, done prefilling -> decode
    is_pure_prefill = ~has_context
    is_long_extend = has_context & ~is_below_threshold
    is_short_extend = has_context & is_below_threshold & ~done_prefilling
    is_decode = has_context & is_below_threshold & done_prefilling

    # Desired order: decode → short_extend → long_extend → prefill
    req_regions = np.zeros(num_reqs, dtype=np.int32)  # 0 = decode by default
    req_regions[is_short_extend] = 1
    req_regions[is_long_extend] = 2
    req_regions[is_pure_prefill] = 3

    num_decodes = int(is_decode.sum())
    num_short_extends = int(is_short_extend.sum())
    num_long_extends = int(is_long_extend.sum())
    num_prefills = int(is_pure_prefill.sum())

    target_regions = np.repeat(
        [0, 1, 2, 3],
        [num_decodes, num_short_extends, num_long_extends, num_prefills],
    ).astype(np.int32)

    needs_swap = req_regions != target_regions

    if not needs_swap.any():
        return False

    # Extract indices that need swapping and sort by target region
    orig_indices = np.where(needs_swap)[0]
    sorted_order = np.argsort(req_regions[needs_swap], kind="stable")
    src_indices = orig_indices[sorted_order]

    src_dest_map = {int(src): int(dst) for src, dst in zip(src_indices, orig_indices)}

    for src in src_dest_map:
        dst = src_dest_map[src]
        while src != dst:
            input_batch.swap_states(src, dst)
            # Mark dst as done by updating its destination to itself
            next_dst = src_dest_map.get(dst, dst)
            src_dest_map[dst] = dst
            dst = next_dst

    return True
```
**EN:** This function implements `reorder_batch_to_split_decodes_and_prefills` within the module. The docstring frames it as: Reorders the batch to split into prefill and decode requests; places all requests with <= decode_threshold tokens at the front of the batch. Key calls include `len`, `array`, `zeros`, `int`, `astype`, `argsort`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `reorder_batch_to_split_decodes_and_prefills`，其作用域位于the module。 关键调用包括 `len`, `array`, `zeros`, `int`, `astype`, `argsort`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `reshape_query_for_spec_decode` function / `reshape_query_for_spec_decode` 函数
```python
def reshape_query_for_spec_decode(query: torch.Tensor, batch_size: int) -> torch.Tensor:
    """
    Reshapes the query tensor for the specified batch size, so that
    it has shape (batch_size, seq_len, num_heads, head_dim).
    """
    assert query.dim() == 3, f"query must be 3D, got {query.dim()}D"
    total_tokens = query.shape[0]
    num_heads = query.shape[1]
    head_dim = query.shape[2]
    assert total_tokens % batch_size == 0, (
        f"{total_tokens=} is not divisible by {batch_size=}"
    )
    seq_len = total_tokens // batch_size
    return query.view(batch_size, seq_len, num_heads, head_dim)
```
**EN:** This function implements `reshape_query_for_spec_decode` within the module. The docstring frames it as: Reshapes the query tensor for the specified batch size, so that it has shape (batch_size, seq_len, num_heads, head_dim). Key calls include `view`, `dim`.
**CN:** 该函数会实现 `reshape_query_for_spec_decode`，其作用域位于the module。 关键调用包括 `view`, `dim`。

### `reshape_attn_output_for_spec_decode` function / `reshape_attn_output_for_spec_decode` 函数
```python
def reshape_attn_output_for_spec_decode(attn_output: torch.Tensor) -> torch.Tensor:
    """
    Reshapes the attention output tensor, so that
    the batch_size and seq_len dimensions are combined.
    """
    if attn_output.dim() == 3:
        # Already in the correct shape
        return attn_output
    assert attn_output.dim() == 4, f"attn_output must be 4D, got {attn_output.dim()}D"
    total_tokens = attn_output.shape[0] * attn_output.shape[1]
    return attn_output.view(total_tokens, attn_output.shape[2], attn_output.shape[3])
```
**EN:** This function implements `reshape_attn_output_for_spec_decode` within the module. The docstring frames it as: Reshapes the attention output tensor, so that the batch_size and seq_len dimensions are combined. Key calls include `view`, `dim`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `reshape_attn_output_for_spec_decode`，其作用域位于the module。 关键调用包括 `view`, `dim`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVSharingFastPrefillMetadata` class / `KVSharingFastPrefillMetadata` 类
```python
@runtime_checkable
class KVSharingFastPrefillMetadata(Protocol):
    logits_indices_padded: torch.Tensor | None = None
    num_logits_indices: int | None = None
```
**EN:** Declares the `KVSharingFastPrefillMetadata` interface. Downstream implementations are expected to provide methods such as the abstract API defined here.
**CN:** `KVSharingFastPrefillMetadata` 声明了一组接口约定。下游实现需要提供 这里定义的抽象 API 等方法。

### `create_fast_prefill_custom_backend` function / `create_fast_prefill_custom_backend` 函数
```python
def create_fast_prefill_custom_backend(
    prefix: str,
    underlying_attn_backend: type[AttentionBackend],
) -> type[AttentionBackend]:
    underlying_builder = underlying_attn_backend.get_builder_cls()

    class FastPrefillAttentionBuilder(underlying_builder):  # type: ignore
        def build(
            self,
            common_prefix_len: int,
            common_attn_metadata: CommonAttentionMetadata,
            fast_build: bool = False,
        ) -> AttentionMetadata:
            new_common_attn_metadata = (
                make_kv_sharing_fast_prefill_common_attn_metadata(common_attn_metadata)
            )
            metadata = super().build(
                common_prefix_len, new_common_attn_metadata, fast_build
            )

            class KVSharingFastPrefillAttentionMetadata(
                metadata.__class__,  #  type: ignore
                KVSharingFastPrefillMetadata,
            ):
                def __init__(self, metadata, common_attn_metadata):
                    # Shallow copy all fields in metadata cls
                    for _field in fields(metadata.__class__):
                        setattr(self, _field.name, getattr(metadata, _field.name))

                    self.logits_indices_padded = (
                        common_attn_metadata.logits_indices_padded
                    )
                    self.num_logits_indices = common_attn_metadata.num_logits_indices

            return KVSharingFastPrefillAttentionMetadata(metadata, common_attn_metadata)

    attn_backend = subclass_attention_backend(
        name_prefix=prefix,
        attention_backend_cls=underlying_attn_backend,
        builder_cls=FastPrefillAttentionBuilder,
    )

    return attn_backend
```
**EN:** This function creates a new object or plan within the module. Key calls include `get_builder_cls`, `subclass_attention_backend`, `make_kv_sharing_fast_prefill_common_attn_metadata`, `build`, `KVSharingFastPrefillAttentionMetadata`, `super`. It touches state such as `logits_indices_padded`, `num_logits_indices`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会创建新的对象或计划，其作用域位于the module。 关键调用包括 `get_builder_cls`, `subclass_attention_backend`, `make_kv_sharing_fast_prefill_common_attn_metadata`, `build`, `KVSharingFastPrefillAttentionMetadata`, `super`。 它会读写 `logits_indices_padded`, `num_logits_indices` 等状态。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `compute_causal_conv1d_metadata` function / `compute_causal_conv1d_metadata` 函数
```python
def compute_causal_conv1d_metadata(
    query_start_loc_p_cpu: torch.Tensor,
    *,
    device: torch.device,
):
    # Needed for causal_conv1d. Use the CPU query_start_loc to avoid DtoH sync.
    assert query_start_loc_p_cpu.device.type == "cpu"
    seqlens = query_start_loc_p_cpu.diff()
    nums_dict = {}  # type: ignore
    batch_ptr = None
    token_chunk_offset_ptr = None
    for BLOCK_M in [8]:  # cover all BLOCK_M values
        nums = -(-seqlens // BLOCK_M)
        nums_dict[BLOCK_M] = {}
        nums_dict[BLOCK_M]["nums"] = nums
        nums_dict[BLOCK_M]["tot"] = nums.sum().item()
        mlist = torch.from_numpy(np.repeat(np.arange(len(nums)), nums))
        nums_dict[BLOCK_M]["mlist"] = mlist
        mlist_len = len(nums_dict[BLOCK_M]["mlist"])
        nums_dict[BLOCK_M]["mlist_len"] = mlist_len
        MAX_NUM_PROGRAMS = max(1024, mlist_len) * 2
        offsetlist = []  # type: ignore
        for idx, num in enumerate(nums):
            offsetlist.extend(range(num))
        offsetlist = torch.tensor(offsetlist, dtype=torch.int32)
        nums_dict[BLOCK_M]["offsetlist"] = offsetlist

        if batch_ptr is None:
            # Update default value after class definition
            batch_ptr = torch.full(
                (MAX_NUM_PROGRAMS,), PAD_SLOT_ID, dtype=torch.int32, device=device
            )
            token_chunk_offset_ptr = torch.full(
                (MAX_NUM_PROGRAMS,), PAD_SLOT_ID, dtype=torch.int32, device=device
            )
        else:
            if batch_ptr.nelement() < MAX_NUM_PROGRAMS:
                batch_ptr.resize_(MAX_NUM_PROGRAMS).fill_(PAD_SLOT_ID)
                token_chunk_offset_ptr.resize_(  # type: ignore
                    MAX_NUM_PROGRAMS
                ).fill_(PAD_SLOT_ID)

        batch_ptr[0:mlist_len].copy_(mlist, non_blocking=True)
        token_chunk_offset_ptr[  # type: ignore
            0:mlist_len
        ].copy_(offsetlist, non_blocking=True)
        nums_dict[BLOCK_M]["batch_ptr"] = batch_ptr
        nums_dict[BLOCK_M]["token_chunk_offset_ptr"] = token_chunk_offset_ptr  # type: ignore

    return nums_dict, batch_ptr, token_chunk_offset_ptr
```
**EN:** This function computes derived values within the module. Key calls include `diff`, `item`, `from_numpy`, `len`, `enumerate`, `tensor`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `diff`, `item`, `from_numpy`, `len`, `enumerate`, `tensor`。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `is_valid_kv_cache_layout`: top-level helper or orchestration entry point. / `is_valid_kv_cache_layout`：顶层辅助函数或编排入口。
- `get_kv_cache_layout`: top-level helper or orchestration entry point. / `get_kv_cache_layout`：顶层辅助函数或编排入口。
- `set_kv_cache_layout`: top-level helper or orchestration entry point. / `set_kv_cache_layout`：顶层辅助函数或编排入口。
- `PerLayerParameters`: central class or interface in this module. / `PerLayerParameters`：本模块中的核心类或接口。
- `get_per_layer_parameters`: top-level helper or orchestration entry point. / `get_per_layer_parameters`：顶层辅助函数或编排入口。
- `infer_global_hyperparameters`: top-level helper or orchestration entry point. / `infer_global_hyperparameters`：顶层辅助函数或编排入口。
- `make_local_attention_virtual_batches`: top-level helper or orchestration entry point. / `make_local_attention_virtual_batches`：顶层辅助函数或编排入口。
- `make_kv_sharing_fast_prefill_common_attn_metadata`: top-level helper or orchestration entry point. / `make_kv_sharing_fast_prefill_common_attn_metadata`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `functools`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`, `typing_extensions`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.utils.math_utils`, `vllm.v1.kv_cache_interface`, `vllm.v1.core.sched.output`, `vllm.v1.worker.gpu_input_batch`, `vllm.envs`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.logger`, `vllm.model_executor.layers.attention_layer_base`, `vllm.v1.attention.backend`
