# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `next_power_of_2`, `eagle_step_slot_mapping_metadata_kernel`, `eagle_step_update_slot_mapping_and_metadata` for the V1 `spec_decode` subsystem. / 为 V1 的 `spec_decode` 子系统实现 `next_power_of_2`, `eagle_step_slot_mapping_metadata_kernel`, `eagle_step_update_slot_mapping_and_metadata`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch

from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.v1.attention.backends.utils import (
    CommonAttentionMetadata,
)

PADDING_SLOT_ID = -1
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `PADDING_SLOT_ID`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `PADDING_SLOT_ID`。

### `next_power_of_2` function / `next_power_of_2` 函数
```python
def next_power_of_2(n: int) -> int:
    """Return the smallest power of 2 >= n."""
    if n <= 0:
        return 1
    n -= 1
    n |= n >> 1
    n |= n >> 2
    n |= n >> 4
    n |= n >> 8
    n |= n >> 16
    n |= n >> 32
    return n + 1
```
**EN:** This function implements `next_power_of_2` within the module. The docstring frames it as: Return the smallest power of 2 >= n. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `next_power_of_2`，其作用域位于the module。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `eagle_step_slot_mapping_metadata_kernel` function / `eagle_step_slot_mapping_metadata_kernel` 函数
```python
@triton.jit
def eagle_step_slot_mapping_metadata_kernel(
    positions_ptr,  # [batch_size] - current positions (1D view for M-RoPE)
    block_table_ptr,  # [batch_size, n_blocks_per_req]
    block_table_stride,  # stride for block_table dim 1
    seq_lens_ptr,  # [batch_size] - read and write
    out_clamped_positions_ptr,  # [batch_size] (output)
    out_slot_mapping_ptr,  # [input_batch_size] (output)
    block_size: tl.constexpr,
    max_model_len: tl.constexpr,
    n_blocks_per_req: tl.constexpr,
    PAD_ID: tl.constexpr,
    batch_size,
):
    """
    Fused kernel for EAGLE autoregressive step: updates positions, slot mapping,
    and sequence lengths in a single kernel to reduce launch overhead.

    Launched with input_batch_size threads. Threads with req_idx >= batch_size
    are cudagraph padding slots and only write PADDING_SLOT_ID.

    Each real thread handles one request in the batch. Computes:
    - new_position = position + 1, clamped if exceeds max_model_len
    - slot_mapping from block table lookup
    - seq_lens += 1, or 1 if position exceeds max
    """
    req_idx = tl.program_id(0)

    if req_idx >= batch_size:
        tl.store(out_slot_mapping_ptr + req_idx, PAD_ID)
        return

    # Load current position and increment
    position = tl.load(positions_ptr + req_idx)
    new_position = position + 1

    # Check bounds and compute clamped position
    exceeds_max = new_position >= max_model_len
    clamped_position = tl.where(exceeds_max, 0, new_position)

    # Block table lookup: block_number = position // block_size
    # Clamp block_number to avoid OOB when position is at max
    block_number = clamped_position // block_size
    block_number = tl.minimum(block_number, n_blocks_per_req - 1)

    block_id = tl.load(block_table_ptr + req_idx * block_table_stride + block_number)
    slot_id = block_id * block_size + (clamped_position % block_size)
    slot_id = tl.where(exceeds_max, PAD_ID, slot_id)

    # Update seq_lens: +1 normally, or 1 if exceeded
    seq_len = tl.load(seq_lens_ptr + req_idx)
    new_seq_len = tl.where(exceeds_max, 1, seq_len + 1)
    new_seq_len = tl.minimum(new_seq_len, max_model_len)

    # Store outputs
    tl.store(out_clamped_positions_ptr + req_idx, clamped_position)
    tl.store(out_slot_mapping_ptr + req_idx, slot_id)
    tl.store(seq_lens_ptr + req_idx, new_seq_len)
```
**EN:** This function implements `eagle_step_slot_mapping_metadata_kernel` within the module. The docstring frames it as: Fused kernel for EAGLE autoregressive step: updates positions, slot mapping, and sequence lengths in a single kernel to reduce launch overhead. Key calls include `program_id`, `load`, `where`, `minimum`, `store`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `eagle_step_slot_mapping_metadata_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `where`, `minimum`, `store`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `eagle_step_update_slot_mapping_and_metadata` function / `eagle_step_update_slot_mapping_and_metadata` 函数
```python
def eagle_step_update_slot_mapping_and_metadata(
    positions_1d: torch.Tensor,
    block_table_tensor: torch.Tensor,
    seq_lens: torch.Tensor,
    block_size: int,
    max_model_len: int,
    out_clamped_positions: torch.Tensor,
    out_slot_mapping: torch.Tensor,
    input_batch_size: int | None = None,
) -> None:
    """
    Fused update of slot mapping and metadata for one EAGLE autoregressive step.
    Updates seq_lens in place. Writes to out_clamped_positions and out_slot_mapping.

    When input_batch_size > batch_size, threads beyond batch_size write
    PADDING_SLOT_ID to out_slot_mapping for cudagraph padding.

    Args:
        positions_1d: [batch_size] current positions (use positions[0] for M-RoPE)
        block_table_tensor: [batch_size, n_blocks_per_req]
        seq_lens: [batch_size] updated in place
        block_size: KV cache block size
        max_model_len: max model length for clamping
        out_clamped_positions: [batch_size] output buffer for clamped positions
        out_slot_mapping: [input_batch_size] output buffer for slot mapping
        input_batch_size: total batch size including cudagraph padding;
            defaults to batch_size (no padding)
    """
    batch_size = positions_1d.shape[0]
    if input_batch_size is None:
        input_batch_size = batch_size

    n_blocks_per_req = block_table_tensor.shape[1]
    eagle_step_slot_mapping_metadata_kernel[(input_batch_size,)](
        positions_1d,
        block_table_tensor,
        block_table_tensor.stride(0),
        seq_lens,
        out_clamped_positions,
        out_slot_mapping,
        block_size=block_size,
        max_model_len=max_model_len,
        n_blocks_per_req=n_blocks_per_req,
        PAD_ID=PADDING_SLOT_ID,
        batch_size=batch_size,
    )
```
**EN:** This function implements `eagle_step_update_slot_mapping_and_metadata` within the module. The docstring frames it as: Fused update of slot mapping and metadata for one EAGLE autoregressive step. Key calls include `stride`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `eagle_step_update_slot_mapping_and_metadata`，其作用域位于the module。 关键调用包括 `stride`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `eagle_prepare_inputs_padded_kernel` function / `eagle_prepare_inputs_padded_kernel` 函数
```python
@triton.jit
def eagle_prepare_inputs_padded_kernel(
    cu_num_draft_tokens_ptr,  # [num_reqs]
    valid_sampled_tokens_count_ptr,  # [num_reqs]
    query_start_loc_gpu_ptr,  # [num_reqs + 1]
    token_indices_to_sample_ptr,  # [num_reqs] (output)
    num_rejected_tokens_gpu_ptr,  # [num_reqs] (output)
    num_reqs,  # tl.int32
):
    """
    Fused kernel for Eagle prepare_input_padded. This kernel computes the
    token index to sample for each request, taking into account the number
    of draft tokens and the number of valid sampled tokens (which is one more than
    the number of accepted tokens).
    """
    req_idx = tl.program_id(axis=0)
    if req_idx >= num_reqs:
        return

    # Calculate num_draft_tokens from cu_num_draft_tokens, which is an inclusive
    # cumulative sum (first entry is the first value, not zero).
    cu_draft_curr = tl.load(cu_num_draft_tokens_ptr + req_idx)

    num_draft_tokens = 0
    if req_idx == 0:
        num_draft_tokens = cu_draft_curr
    else:
        cu_draft_prev = tl.load(cu_num_draft_tokens_ptr + req_idx - 1)
        num_draft_tokens = cu_draft_curr - cu_draft_prev

    valid_count = tl.load(valid_sampled_tokens_count_ptr + req_idx)
    num_rejected_tokens = num_draft_tokens + 1 - valid_count
    num_rejected_tokens = tl.where(num_draft_tokens > 0, num_rejected_tokens, 0)

    # query_start_loc[req_idx + 1] is the start position of the next request,
    # which is one past the last token of this request.
    q_last_tok_idx = tl.load(query_start_loc_gpu_ptr + req_idx + 1) - 1

    index_to_sample = q_last_tok_idx - num_rejected_tokens
    tl.store(token_indices_to_sample_ptr + req_idx, index_to_sample)
    tl.store(num_rejected_tokens_gpu_ptr + req_idx, num_rejected_tokens)
```
**EN:** This function implements `eagle_prepare_inputs_padded_kernel` within the module. The docstring frames it as: Fused kernel for Eagle prepare_input_padded. Key calls include `program_id`, `load`, `where`, `store`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `eagle_prepare_inputs_padded_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `where`, `store`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `eagle_prepare_next_token_padded_kernel` function / `eagle_prepare_next_token_padded_kernel` 函数
```python
@triton.jit
def eagle_prepare_next_token_padded_kernel(
    sampled_token_ids_ptr,  # [num_reqs, num_sampled_tokens_per_req]
    discard_request_mask_ptr,  # [num_reqs]
    backup_next_token_ids_ptr,  # [num_reqs]
    next_token_ids_ptr,  # [num_reqs] (output)
    valid_sampled_tokens_count_ptr,  # [num_reqs] (output)
    vocab_size,  # tl.int32
    num_sampled_tokens_per_req,  # tl.int32 (num_spec_tokens + 1)
    num_reqs,  # tl.int32
    stride_sampled_token_ids,  # tl.int32 (stride for dim 0)
    BLOCK_SIZE_TOKENS: tl.constexpr,  # Power-of-2 >= num_sampled_tokens_per_req
):
    """
    Fused kernel for Eagle prepare_next_token_ids_padded. This kernel computes the
    number of valid (1 + accepted) tokens for each request, and the corresponding
    "next" token id to sample from during speculative decoding. This is the
    "last accepted token" from the sampled tokens, or the backup token if no
    tokens were accepted or if the request is marked as discarded.
    """
    req_idx = tl.program_id(axis=0)
    if req_idx >= num_reqs:
        return

    # Check if this request is discarded.
    is_discarded = tl.load(discard_request_mask_ptr + req_idx)

    if is_discarded:
        backup_token = tl.load(backup_next_token_ids_ptr + req_idx)
        valid_count = tl.full((), 0, dtype=tl.uint32)
        tl.store(next_token_ids_ptr + req_idx, backup_token)
        tl.store(valid_sampled_tokens_count_ptr + req_idx, valid_count)
    else:
        # Count the number of valid tokens among the sampled tokens.
        token_offs = tl.arange(0, BLOCK_SIZE_TOKENS)
        token_mask = token_offs < num_sampled_tokens_per_req

        row_ptr = sampled_token_ids_ptr + req_idx * stride_sampled_token_ids
        token_ids = tl.load(row_ptr + token_offs, mask=token_mask, other=-1)

        # Rejected tokens are -1, valid tokens are in [0, vocab_size)
        is_valid_mask = (token_ids != -1) & (token_ids < vocab_size) & token_mask
        valid_count = tl.sum(is_valid_mask)

        if valid_count > 0:
            # Guaranteed to be well-defined since
            # valid_count > 0 implies is_valid_mask is not empty
            last_valid_index = tl.max(tl.where(is_valid_mask, token_offs, -1))

            # Select the token at that index, using a sum trick since
            # we don't want to load again to access token_ids[last_valid_index].
            last_valid_token = tl.sum(
                tl.where(token_offs == last_valid_index, token_ids, 0)
            )
            tl.store(next_token_ids_ptr + req_idx, last_valid_token)
        else:
            # No valid tokens found, use backup token
            backup_token = tl.load(backup_next_token_ids_ptr + req_idx)
            tl.store(next_token_ids_ptr + req_idx, backup_token)

        tl.store(valid_sampled_tokens_count_ptr + req_idx, valid_count)
```
**EN:** This function implements `eagle_prepare_next_token_padded_kernel` within the module. The docstring frames it as: Fused kernel for Eagle prepare_next_token_ids_padded. Key calls include `program_id`, `load`, `full`, `store`, `arange`, `sum`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `eagle_prepare_next_token_padded_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `full`, `store`, `arange`, `sum`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `compute_new_slot_mapping` function / `compute_new_slot_mapping` 函数
```python
def compute_new_slot_mapping(
    cad: CommonAttentionMetadata,
    new_positions: torch.Tensor,
    is_rejected_token_mask: torch.Tensor,
    block_size: int,
    num_new_tokens: int,
    max_model_len: int,
):
    batch_size, n_blocks_per_req = cad.block_table_tensor.shape
    req_indices = torch.arange(batch_size, device=cad.query_start_loc.device)
    req_indices = torch.repeat_interleave(
        req_indices,
        cad.naive_query_lens() + num_new_tokens,
        output_size=len(new_positions),
    )
    # Clamp the positions to prevent an out-of-bounds error when indexing
    # into block_table_tensor.
    clamped_positions = torch.clamp(new_positions, max=max_model_len - 1)
    block_table_indices = (
        req_indices * n_blocks_per_req + clamped_positions // block_size
    )
    block_nums = cad.block_table_tensor.view(-1)[block_table_indices]
    block_offsets = clamped_positions % block_size
    new_slot_mapping = block_nums * block_size + block_offsets
    # Mask out the position ids that exceed the max model length.
    exceeds_max_model_len = new_positions >= max_model_len
    new_slot_mapping.masked_fill_(exceeds_max_model_len, PADDING_SLOT_ID)
    # Mask out rejected tokens to prevent saves to the KV cache.
    new_slot_mapping.masked_fill_(is_rejected_token_mask, PADDING_SLOT_ID)
    return new_slot_mapping
```
**EN:** This function computes derived values within the module. Key calls include `arange`, `repeat_interleave`, `clamp`, `masked_fill_`, `view`, `naive_query_lens`.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `arange`, `repeat_interleave`, `clamp`, `masked_fill_`, `view`, `naive_query_lens`。

### `extend_all_queries_by_N` function / `extend_all_queries_by_N` 函数
```python
def extend_all_queries_by_N(
    common_attn_metadata: CommonAttentionMetadata,
    N: int,
    arange: torch.Tensor,
    new_slot_mapping: torch.Tensor,
) -> CommonAttentionMetadata:
    """
    Creates a new CommonAttentionMetadata with all query lengths increased by N.
    Also all seq lens are increased by N.
    This is useful e.g. in speculative decoding with parallel drafting, where we
    extend each sequence by N tokens and predict all tokens in one pass.
    The slot mapping is computed externally, as it requires more information.
    """
    cad = common_attn_metadata
    # query start loc must be increased by [+0, +N, +2N, ..., +batch_size * N]
    new_query_start_loc = cad.query_start_loc + N * arange[: len(cad.query_start_loc)]
    new_query_start_loc_cpu = cad.query_start_loc_cpu + N * torch.arange(
        len(cad.query_start_loc_cpu), dtype=torch.int32
    )
    new_cad = cad.replace(
        query_start_loc=new_query_start_loc,
        query_start_loc_cpu=new_query_start_loc_cpu,
        seq_lens=cad.seq_lens + N,
        # each request is extended by N tokens -> batch_size * N tokens are added
        num_actual_tokens=cad.num_actual_tokens + cad.batch_size() * N,
        # All query lens increase by N, so max query len increases by N
        max_query_len=cad.max_query_len + N,
        max_seq_len=cad.max_seq_len + N,
        slot_mapping=new_slot_mapping,
    )
    return new_cad
```
**EN:** This function implements `extend_all_queries_by_N` within the module. The docstring frames it as: Creates a new CommonAttentionMetadata with all query lengths increased by N. Key calls include `replace`, `arange`, `len`, `batch_size`.
**CN:** 该函数会实现 `extend_all_queries_by_N`，其作用域位于the module。 关键调用包括 `replace`, `arange`, `len`, `batch_size`。

### `copy_and_expand_eagle_inputs_kernel` function / `copy_and_expand_eagle_inputs_kernel` 函数
```python
@triton.jit
def copy_and_expand_eagle_inputs_kernel(
    # (Padded) Inputs from the target model
    target_token_ids_ptr,  # [total_tokens_in_batch]
    target_positions_ptr,  # [total_tokens_in_batch]
    next_token_ids_ptr,  # [num_reqs]
    # Outputs to the drafting buffers
    out_input_ids_ptr,  # [total_draft_tokens_in_batch] (output)
    out_positions_ptr,  # [total_draft_tokens_in_batch] (output)
    out_is_rejected_token_mask_ptr,  # [total_draft_tokens_in_batch] (output)
    out_is_masked_token_mask_ptr,  # [total_draft_tokens_in_batch] (output)
    out_new_token_indices_ptr,  # [num_padding_slots_per_request * num_reqs] (output)
    out_hidden_state_mapping_ptr,  # [total_tokens_in_batch]
    # Input metadata
    query_start_loc_ptr,  # [num_reqs + 1], last value is the total num input tokens
    query_end_loc_ptr,  # [num_reqs]
    padding_token_id,  # tl.int32
    parallel_drafting_token_id,  # tl.int32
    # Sizing info
    total_input_tokens,  # tl.int32
    num_padding_slots_per_request,  # tl.int32
    shift_input_ids,  # tl.bool
    BLOCK_SIZE_TOKENS: tl.constexpr,  # Blocks along token dim to handle prefills
):
    """
    Copy and expand inputs from the target model to the drafting buffers for Eagle
    speculative decoding. This kernel handles padding slots and parallel drafting
    tokens, if enabled.
    """
    request_idx = tl.program_id(axis=0)
    token_batch_idx = tl.program_id(axis=1)

    # Load query locations
    query_start_loc = tl.load(query_start_loc_ptr + request_idx)
    next_query_start_loc = tl.load(query_start_loc_ptr + request_idx + 1)
    query_end_loc = tl.load(query_end_loc_ptr + request_idx)

    # Calculate number of valid tokens to copy and input offset
    # With shift_input_ids=True, we skip the first token
    # Output layout: each request gets (input_len + num_padding_slots_per_request) slots
    # But with shift, we lose one token per request
    if shift_input_ids:
        num_valid_tokens = query_end_loc - query_start_loc
        input_offset = 1
        output_start = query_start_loc + request_idx * (
            num_padding_slots_per_request - 1
        )
    else:
        num_valid_tokens = query_end_loc - query_start_loc + 1
        input_offset = 0
# ... omitted for brevity ...
        j - num_valid_tokens
    )  # 0 for bonus, 1, 2, ... for parallel drafting
    new_token_out_idx = (
        request_idx * num_padding_slots_per_request + new_token_local_idx
    )

    # Compute hidden state mapping (source index -> destination index)
    # This maps each input position to its corresponding output position
    # Hidden states don't get shifted, so we map all input tokens (including rejected)
    if shift_input_ids:
        num_input_tokens_this_request = next_query_start_loc - query_start_loc
        is_input_region = j < num_input_tokens_this_request
        src_idx = query_start_loc + j
        tl.store(out_hidden_state_mapping_ptr + src_idx, out_idx, mask=is_input_region)

    # Store outputs
    tl.store(out_input_ids_ptr + out_idx, token_ids, mask=in_bounds)
    tl.store(out_positions_ptr + out_idx, positions, mask=in_bounds)
    tl.store(out_is_rejected_token_mask_ptr + out_idx, is_rejected_out, mask=in_bounds)
    tl.store(out_is_masked_token_mask_ptr + out_idx, is_masked_out, mask=in_bounds)
    tl.store(
        out_new_token_indices_ptr + new_token_out_idx,
        out_idx,
        mask=is_new_token_region & in_bounds,
    )
```
**EN:** This function implements `copy_and_expand_eagle_inputs_kernel` within the module. The docstring frames it as: Copy and expand inputs from the target model to the drafting buffers for Eagle speculative decoding. Key calls include `program_id`, `load`, `minimum`, `where`, `store`, `arange`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `copy_and_expand_eagle_inputs_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `minimum`, `where`, `store`, `arange`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `copy_and_expand_dflash_inputs_kernel` function / `copy_and_expand_dflash_inputs_kernel` 函数
```python
@triton.jit
def copy_and_expand_dflash_inputs_kernel(
    # Inputs
    next_token_ids_ptr,  # [num_reqs]
    target_positions_ptr,  # [num_context]
    # Outputs
    out_input_ids_ptr,  # [num_query_total] (output)
    out_context_positions_ptr,  # [num_context] (output)
    out_query_positions_ptr,  # [num_query_total] (output)
    out_context_slot_mapping_ptr,  # [num_context] (output)
    out_query_slot_mapping_ptr,  # [num_query_total] (output)
    out_token_indices_ptr,  # [num_reqs * num_speculative_tokens] (output)
    # Block table
    block_table_ptr,  # [max_reqs, max_blocks]
    block_table_stride,  # stride of block_table dim 0 (in elements)
    # Metadata
    query_start_loc_ptr,  # [num_reqs + 1]
    num_rejected_tokens_ptr,  # [num_reqs] or null (0) when not padded
    # Scalars
    parallel_drafting_token_id,  # tl.int32
    block_size,  # tl.int32
    num_query_per_req,  # tl.int32
    num_speculative_tokens,  # tl.int32
    total_input_tokens,  # tl.int32
    BLOCK_SIZE: tl.constexpr,
    HAS_NUM_REJECTED: tl.constexpr = False,
):
    """
    Fused kernel for DFlash first-pass input setup.

    Per request, this kernel:
      1. Copies context positions from target_positions to
         out_context_positions.
      2. Computes query positions (last_target_pos + 1 + offset) and writes
         them to out_query_positions.
      3. Writes input_ids for query tokens: [next_token, mask, mask, ...].
      4. Computes slot_mapping for context and query positions into separate
         buffers via block_table lookup.
      5. Writes token_indices_to_sample for the mask (speculative) tokens.
    """
    req_idx = tl.program_id(axis=0)
    block_idx = tl.program_id(axis=1)

    # Load context token range for this request
    ctx_start = tl.load(query_start_loc_ptr + req_idx)
    ctx_end = tl.load(query_start_loc_ptr + req_idx + 1)
    num_ctx = ctx_end - ctx_start
    total_tokens = num_ctx + num_query_per_req

    j = block_idx * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
# ... omitted for brevity ...
    # # Clamp block_number to avoid OOB when position is at max
    block_num = tl.minimum(block_num, block_table_stride - 1)
    block_id = tl.load(
        block_table_ptr + req_idx * block_table_stride + block_num,
        mask=in_bounds,
        other=0,
    ).to(tl.int64)
    slot = block_id * block_size + (positions % block_size)
    tl.store(out_context_slot_mapping_ptr + ctx_pos_out, slot, mask=is_ctx)
    tl.store(out_query_slot_mapping_ptr + query_out, slot, mask=is_query)

    # --- Input IDs (query tokens only) ---
    bonus_token = tl.load(next_token_ids_ptr + req_idx)
    is_bonus = is_query & (query_off == 0)
    input_id = tl.where(is_bonus, bonus_token, parallel_drafting_token_id)
    tl.store(out_input_ids_ptr + query_out, input_id, mask=is_query)

    # --- Token indices to sample (mask tokens, skip the bonus token) ---
    is_sample = is_query & (query_off > 0)
    sample_out_idx = req_idx * num_speculative_tokens + (query_off - 1)
    tl.store(
        out_token_indices_ptr + sample_out_idx,
        query_out,
        mask=is_sample,
    )
```
**EN:** This function implements `copy_and_expand_dflash_inputs_kernel` within the module. The docstring frames it as: Fused kernel for DFlash first-pass input setup. Key calls include `program_id`, `load`, `minimum`, `where`, `store`, `to`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `copy_and_expand_dflash_inputs_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `minimum`, `where`, `store`, `to`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `update_num_computed_tokens_for_batch_change` function / `update_num_computed_tokens_for_batch_change` 函数
```python
@torch.compile(dynamic=True, backend=current_platform.simple_compile_backend)
def update_num_computed_tokens_for_batch_change(
    num_computed_tokens: torch.Tensor,
    num_accepted_tokens: torch.Tensor,
    prev_positions: torch.Tensor,
    valid_sampled_token_count: torch.Tensor,
    prev_num_draft_tokens: torch.Tensor,
    cpu_num_computed_tokens: torch.Tensor,
) -> None:
    """Correct num_computed_tokens for async spec decode drift.

    Requests that had drafts: corrected = prev_gpu + valid_count.
    New requests or non-draft (e.g. prefills): use CPU value directly.
    """
    # Clamp because prev_positions can be -1 for new requests
    gather_indices = prev_positions.clamp(min=0)

    valid_counts = valid_sampled_token_count[gather_indices]
    prev_computed = num_computed_tokens[gather_indices]
    prev_drafts = prev_num_draft_tokens[gather_indices]

    participating = (prev_positions >= 0) & (prev_drafts > 0)
    corrected = prev_computed + valid_counts.int()

    n = prev_positions.shape[0]
    num_computed_tokens[:n].copy_(
        torch.where(participating, corrected, cpu_num_computed_tokens)
    )
    num_accepted_tokens.copy_(
        torch.where(participating, valid_counts, num_accepted_tokens)
    )
```
**EN:** This function updates existing state within the module. The docstring frames it as: Correct num_computed_tokens for async spec decode drift. Key calls include `compile`, `clamp`, `copy_`, `int`, `where`.
**CN:** 该函数会更新现有状态，其作用域位于the module。 关键调用包括 `compile`, `clamp`, `copy_`, `int`, `where`。

## Key Concepts / 关键概念
- `next_power_of_2`: top-level helper or orchestration entry point. / `next_power_of_2`：顶层辅助函数或编排入口。
- `eagle_step_slot_mapping_metadata_kernel`: top-level helper or orchestration entry point. / `eagle_step_slot_mapping_metadata_kernel`：顶层辅助函数或编排入口。
- `eagle_step_update_slot_mapping_and_metadata`: top-level helper or orchestration entry point. / `eagle_step_update_slot_mapping_and_metadata`：顶层辅助函数或编排入口。
- `eagle_prepare_inputs_padded_kernel`: top-level helper or orchestration entry point. / `eagle_prepare_inputs_padded_kernel`：顶层辅助函数或编排入口。
- `eagle_prepare_next_token_padded_kernel`: top-level helper or orchestration entry point. / `eagle_prepare_next_token_padded_kernel`：顶层辅助函数或编排入口。
- `compute_new_slot_mapping`: top-level helper or orchestration entry point. / `compute_new_slot_mapping`：顶层辅助函数或编排入口。
- `extend_all_queries_by_N`: top-level helper or orchestration entry point. / `extend_all_queries_by_N`：顶层辅助函数或编排入口。
- `copy_and_expand_eagle_inputs_kernel`: top-level helper or orchestration entry point. / `copy_and_expand_eagle_inputs_kernel`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.platforms`, `vllm.triton_utils`, `vllm.v1.attention.backends.utils`
