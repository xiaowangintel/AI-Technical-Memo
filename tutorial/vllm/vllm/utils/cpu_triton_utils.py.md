# cpu_triton_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/cpu_triton_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Contains replacement functions to fallback Triton usages in CPU backend / 该模块围绕 `cpu_triton_utils` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-10)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

"""
Contains replacement functions to fallback Triton usages in CPU backend
"""

from collections.abc import Callable

import torch
```
**EN:** Sets up the module with standard-library support such as `collections.abc`, external packages such as `torch`. It prepares the symbols later used by `_FuncWrapper`, `_compute_slot_mapping_kernel_impl`, `_ensure_int64`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `_FuncWrapper`, `_compute_slot_mapping_kernel_impl`, `_ensure_int64` 提供上下文。

### _FuncWrapper (lines 13-18)
```python
class _FuncWrapper:
    def __init__(self, func: Callable) -> None:
        self.func = func

    def __getitem__(self, *args, **kwargs) -> Callable:
        return self.func
```
**EN:** Defines the `_FuncWrapper` class used by this module. Key methods include `__init__`.
**CN:** `_FuncWrapper` 是该文件中的核心类，用于封装与 `_FuncWrapper` 相关的状态和行为。 关键方法包括 `__init__`。

### _compute_slot_mapping_kernel_impl (lines 22-44)
```python
def _compute_slot_mapping_kernel_impl(
    num_tokens: int,
    max_num_tokens: int,
    query_start_loc: torch.Tensor,  # [num_reqs + 1], int32
    positions: torch.Tensor,  # [num_tokens], int64
    block_table: torch.Tensor,  # [max_num_reqs, max_num_blocks_per_req], int32
    block_table_stride: int,  # max_num_blocks_per_req
    block_size: int,
    slot_mapping: torch.Tensor,  # [max_num_tokens], int64
    TOTAL_CP_WORLD_SIZE: int,
    TOTAL_CP_RANK: int,
    CP_KV_CACHE_INTERLEAVE_SIZE: int,
    PAD_ID: int,
    BLOCK_SIZE: int,
) -> None:
    assert TOTAL_CP_WORLD_SIZE == 1, "Context Parallelism is not supported on CPU."
    torch.ops._C.compute_slot_mapping_kernel_impl(
        query_start_loc,
        positions,
        block_table,
        slot_mapping,
        block_size,
    )
```
**EN:** `_compute_slot_mapping_kernel_impl` implements helper logic used by this module. It mainly works with `num_tokens`, `max_num_tokens`, `query_start_loc`, `positions`. Inside the body, it relies on `torch.ops._C.compute_slot_mapping_kernel_impl` to complete the main steps.
**CN:** `_compute_slot_mapping_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `num_tokens`, `max_num_tokens`, `query_start_loc`, `positions` 等参数。 实现过程中会调用 `torch.ops._C.compute_slot_mapping_kernel_impl` 等函数完成关键步骤。

### _ensure_int64 (lines 50-51)
```python
def _ensure_int64(t: torch.Tensor) -> torch.Tensor:
    return t if t.dtype == torch.int64 else t.to(torch.int64)
```
**EN:** `_ensure_int64` implements helper logic used by this module. It mainly works with `t`. Inside the body, it relies on `t.to` to complete the main steps.
**CN:** `_ensure_int64` 负责实现本模块使用的辅助逻辑。 它主要处理 `t` 等参数。 实现过程中会调用 `t.to` 等函数完成关键步骤。

### _eagle_prepare_inputs_padded_kernel_impl (lines 54-79)
```python
def _eagle_prepare_inputs_padded_kernel_impl(
    cu_num_draft_tokens,
    valid_sampled_tokens_count,
    query_start_loc_gpu,
    token_indices_to_sample,
    num_rejected_tokens_gpu,
    num_reqs,
):
    # C++ expects int64 for cu_num_draft_tokens, valid_sampled_tokens_count,
    # and num_rejected_tokens_gpu, but Python allocates them as int32.
    orig_rejected_dtype = num_rejected_tokens_gpu.dtype
    rejected_i64 = (
        num_rejected_tokens_gpu
        if orig_rejected_dtype == torch.int64
        else num_rejected_tokens_gpu.to(torch.int64)
    )
    torch.ops._C.eagle_prepare_inputs_padded_kernel_impl(
        _ensure_int64(cu_num_draft_tokens),
        _ensure_int64(valid_sampled_tokens_count),
        query_start_loc_gpu,
        token_indices_to_sample,
        rejected_i64,
        num_reqs,
    )
    if orig_rejected_dtype != torch.int64:
        num_rejected_tokens_gpu.copy_(rejected_i64.to(orig_rejected_dtype))
```
**EN:** `_eagle_prepare_inputs_padded_kernel_impl` implements helper logic used by this module. It mainly works with `cu_num_draft_tokens`, `valid_sampled_tokens_count`, `query_start_loc_gpu`, `token_indices_to_sample`. Inside the body, it relies on `torch.ops._C.eagle_prepare_inputs_padded_kernel_impl`, `num_rejected_tokens_gpu.to`, `_ensure_int64` to complete the main steps.
**CN:** `_eagle_prepare_inputs_padded_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `cu_num_draft_tokens`, `valid_sampled_tokens_count`, `query_start_loc_gpu`, `token_indices_to_sample` 等参数。 实现过程中会调用 `torch.ops._C.eagle_prepare_inputs_padded_kernel_impl`, `num_rejected_tokens_gpu.to`, `_ensure_int64` 等函数完成关键步骤。

### _eagle_prepare_next_token_padded_kernel_impl (lines 82-113)
```python
def _eagle_prepare_next_token_padded_kernel_impl(
    sampled_token_ids,
    discard_request_mask,
    backup_next_token_ids,
    next_token_ids,
    valid_sampled_tokens_count,
    vocab_size,
    num_sampled_tokens_per_req,
    num_reqs,
    stride=None,
    BLOCK_SIZE_TOKENS=None,
):
    # C++ reads all integer tensors as int64_t*. Output tensors are written
    # in-place so we create int64 copies, call C++, and copy back.
    orig_next_dtype = next_token_ids.dtype
    orig_valid_dtype = valid_sampled_tokens_count.dtype
    next_i64 = _ensure_int64(next_token_ids)
    valid_i64 = _ensure_int64(valid_sampled_tokens_count)
    torch.ops._C.eagle_prepare_next_token_padded_kernel_impl(
        _ensure_int64(sampled_token_ids),
        discard_request_mask,
        _ensure_int64(backup_next_token_ids),
        next_i64,
        valid_i64,
        vocab_size,
        num_sampled_tokens_per_req,
        num_reqs,
    )
    if orig_next_dtype != torch.int64:
        next_token_ids.copy_(next_i64.to(orig_next_dtype))
    if orig_valid_dtype != torch.int64:
        valid_sampled_tokens_count.copy_(valid_i64.to(orig_valid_dtype))
```
**EN:** `_eagle_prepare_next_token_padded_kernel_impl` implements helper logic used by this module. It mainly works with `sampled_token_ids`, `discard_request_mask`, `backup_next_token_ids`, `next_token_ids`. Inside the body, it relies on `_ensure_int64`, `torch.ops._C.eagle_prepare_next_token_padded_kernel_impl`, `next_token_ids.copy_` to complete the main steps.
**CN:** `_eagle_prepare_next_token_padded_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `sampled_token_ids`, `discard_request_mask`, `backup_next_token_ids`, `next_token_ids` 等参数。 实现过程中会调用 `_ensure_int64`, `torch.ops._C.eagle_prepare_next_token_padded_kernel_impl`, `next_token_ids.copy_` 等函数完成关键步骤。

### _eagle_step_slot_mapping_metadata_kernel_impl (lines 116-141)
```python
def _eagle_step_slot_mapping_metadata_kernel_impl(
    positions,
    block_table,
    stride,
    seq_lens,
    out_clamped_positions,
    out_slot_mapping,
    block_size,
    max_model_len,
    n_blocks_per_req,
    PAD_ID,
    batch_size=None,
):
    assert batch_size is None or batch_size == positions.shape[0], (
        f"batch_size mismatch: {batch_size} vs positions.shape[0]={positions.shape[0]}"
    )
    torch.ops._C.eagle_step_slot_mapping_metadata_kernel_impl(
        positions,
        block_table,
        seq_lens,
        out_clamped_positions,
        out_slot_mapping,
        block_size,
        max_model_len,
        PAD_ID,
    )
```
**EN:** `_eagle_step_slot_mapping_metadata_kernel_impl` implements helper logic used by this module. It mainly works with `positions`, `block_table`, `stride`, `seq_lens`. Inside the body, it relies on `torch.ops._C.eagle_step_slot_mapping_metadata_kernel_impl` to complete the main steps.
**CN:** `_eagle_step_slot_mapping_metadata_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `positions`, `block_table`, `stride`, `seq_lens` 等参数。 实现过程中会调用 `torch.ops._C.eagle_step_slot_mapping_metadata_kernel_impl` 等函数完成关键步骤。

### _copy_and_expand_eagle_inputs_kernel_impl (lines 144-196)
```python
def _copy_and_expand_eagle_inputs_kernel_impl(
    target_token_ids_ptr,
    target_positions_ptr,
    next_token_ids_ptr,
    out_input_ids_ptr,
    out_positions_ptr,
    out_is_rejected_token_mask_ptr,
    out_is_masked_token_mask_ptr,
    out_new_token_indices_ptr,
    out_hidden_state_mapping_ptr,
    query_start_loc_ptr,
    query_end_loc_ptr,
    padding_token_id,
    parallel_drafting_token_id,
    total_input_tokens,
    num_padding_slots_per_request,
    shift_input_ids,
    BLOCK_SIZE_TOKENS=None,
    BLOCK_SIZE_REQS=None,
):
    """Adapter between Triton kernel call convention and C++ implementation.

    The Triton kernel uses '_ptr' suffixed parameter names and compile-time
    constants (BLOCK_SIZE_TOKENS, BLOCK_SIZE_REQS) which are not needed by
    # ...
        shift_input_ids,
    )
    if orig_ids_dtype != torch.int64:
        out_input_ids_ptr.copy_(out_ids_i64.to(orig_ids_dtype))
    if orig_pos_dtype != torch.int64:
        out_positions_ptr.copy_(out_pos_i64.to(orig_pos_dtype))
```
**EN:** `_copy_and_expand_eagle_inputs_kernel_impl`: Adapter between Triton kernel call convention and C++ implementation. It mainly works with `target_token_ids_ptr`, `target_positions_ptr`, `next_token_ids_ptr`, `out_input_ids_ptr`. Inside the body, it relies on `_ensure_int64`, `torch.ops._C.copy_and_expand_eagle_inputs_kernel_impl`, `out_input_ids_ptr.copy_` to complete the main steps.
**CN:** `_copy_and_expand_eagle_inputs_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `target_token_ids_ptr`, `target_positions_ptr`, `next_token_ids_ptr`, `out_input_ids_ptr` 等参数。 实现过程中会调用 `_ensure_int64`, `torch.ops._C.copy_and_expand_eagle_inputs_kernel_impl`, `out_input_ids_ptr.copy_` 等函数完成关键步骤。

### _rejection_greedy_sample_kernel_impl (lines 199-229)
```python
def _rejection_greedy_sample_kernel_impl(
    output_token_ids,
    cu_num_draft_tokens,
    draft_token_ids,
    target_argmax,
    bonus_token_ids,
    is_greedy,
    max_spec_len,
    uniform_probs=None,
    synthetic_conditional_rates=None,
    SYNTHETIC_MODE=False,
):
    # C++ kernel expects int64 for all integer tensors.
    # Note: uniform_probs, synthetic_conditional_rates, and SYNTHETIC_MODE are
    # passed by the rejection sampler for synthetic mode support, but are not
    # yet implemented in the C++ CPU kernel. We accept them here to maintain
    # compatibility with the kernel calling convention.
    assert not SYNTHETIC_MODE, "Synthetic acceptance not supported with CPU sampling"
    orig_dtype = output_token_ids.dtype
    output_token_ids_i64 = _ensure_int64(output_token_ids)
    torch.ops._C.rejection_greedy_sample_kernel_impl(
        output_token_ids_i64,
        _ensure_int64(cu_num_draft_tokens),
        _ensure_int64(draft_token_ids),
        _ensure_int64(target_argmax),
        _ensure_int64(bonus_token_ids),
        is_greedy,
        max_spec_len,
    )
    if orig_dtype != torch.int64:
        output_token_ids.copy_(output_token_ids_i64.to(orig_dtype))
```
**EN:** `_rejection_greedy_sample_kernel_impl` implements helper logic used by this module. It mainly works with `output_token_ids`, `cu_num_draft_tokens`, `draft_token_ids`, `target_argmax`. Inside the body, it relies on `_ensure_int64`, `torch.ops._C.rejection_greedy_sample_kernel_impl`, `output_token_ids.copy_` to complete the main steps.
**CN:** `_rejection_greedy_sample_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `output_token_ids`, `cu_num_draft_tokens`, `draft_token_ids`, `target_argmax` 等参数。 实现过程中会调用 `_ensure_int64`, `torch.ops._C.rejection_greedy_sample_kernel_impl`, `output_token_ids.copy_` 等函数完成关键步骤。

### _rejection_random_sample_kernel_impl (lines 232-273)
```python
def _rejection_random_sample_kernel_impl(
    output_token_ids,
    cu_num_draft_tokens,
    draft_token_ids,
    draft_probs,
    target_probs,
    bonus_token_ids,
    recovered_token_ids,
    uniform_probs,
    is_greedy,
    max_spec_len,
    vocab_size,
    synthetic_conditional_rates=None,
    NO_DRAFT_PROBS=False,
    SYNTHETIC_MODE=False,
):
    # C++ kernel expects int64 for all integer tensors and float32 for probs.
    # uniform_probs is intentionally float64 in Python to avoid exact-zero
    # samples; cast to float32 here for C++ compatibility.
    # Note: synthetic_conditional_rates and SYNTHETIC_MODE are passed by the
    # rejection sampler for synthetic mode support, but are not yet implemented
    # in the C++ CPU kernel. We accept them here to maintain compatibility with
    # the kernel calling convention.
    assert not SYNTHETIC_MODE, "Synthetic acceptance not supported with CPU sampling"
    # ...
        max_spec_len,
        vocab_size,
        NO_DRAFT_PROBS,
    )
    if orig_dtype != torch.int64:
        output_token_ids.copy_(output_token_ids_i64.to(orig_dtype))
```
**EN:** `_rejection_random_sample_kernel_impl` implements helper logic used by this module. It mainly works with `output_token_ids`, `cu_num_draft_tokens`, `draft_token_ids`, `draft_probs`. Inside the body, it relies on `_ensure_int64`, `torch.ops._C.rejection_random_sample_kernel_impl`, `uniform_probs.to` to complete the main steps.
**CN:** `_rejection_random_sample_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `output_token_ids`, `cu_num_draft_tokens`, `draft_token_ids`, `draft_probs` 等参数。 实现过程中会调用 `_ensure_int64`, `torch.ops._C.rejection_random_sample_kernel_impl`, `uniform_probs.to` 等函数完成关键步骤。

### _expand_kernel_impl (lines 276-290)
```python
def _expand_kernel_impl(
    output,
    input_val,
    cu_num_tokens,
    replace_from,
    replace_to,
    MAX_NUM_TOKENS=None,
):
    torch.ops._C.expand_kernel_impl(
        _ensure_int64(output),
        _ensure_int64(input_val),
        _ensure_int64(cu_num_tokens),
        replace_from,
        replace_to,
    )
```
**EN:** `_expand_kernel_impl` implements helper logic used by this module. It mainly works with `output`, `input_val`, `cu_num_tokens`, `replace_from`. Inside the body, it relies on `torch.ops._C.expand_kernel_impl`, `_ensure_int64` to complete the main steps.
**CN:** `_expand_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `output`, `input_val`, `cu_num_tokens`, `replace_from` 等参数。 实现过程中会调用 `torch.ops._C.expand_kernel_impl`, `_ensure_int64` 等函数完成关键步骤。

### _sample_recovered_tokens_kernel_impl (lines 293-318)
```python
def _sample_recovered_tokens_kernel_impl(
    output_token_ids,
    cu_num_draft_tokens,
    draft_token_ids,
    draft_probs,
    target_probs,
    inv_q,
    vocab_size,
    BLOCK_SIZE=None,
    NO_DRAFT_PROBS=False,
):
    # C++ reads integer tensors as int64_t*; ensure correct dtype.
    orig_dtype = output_token_ids.dtype
    output_i64 = _ensure_int64(output_token_ids)
    torch.ops._C.sample_recovered_tokens_kernel_impl(
        output_i64,
        _ensure_int64(cu_num_draft_tokens),
        _ensure_int64(draft_token_ids),
        draft_probs,
        target_probs,
        inv_q,
        vocab_size,
        NO_DRAFT_PROBS,
    )
    if orig_dtype != torch.int64:
        output_token_ids.copy_(output_i64.to(orig_dtype))
```
**EN:** `_sample_recovered_tokens_kernel_impl` implements helper logic used by this module. It mainly works with `output_token_ids`, `cu_num_draft_tokens`, `draft_token_ids`, `draft_probs`. Inside the body, it relies on `_ensure_int64`, `torch.ops._C.sample_recovered_tokens_kernel_impl`, `output_token_ids.copy_` to complete the main steps.
**CN:** `_sample_recovered_tokens_kernel_impl` 负责实现本模块使用的辅助逻辑。 它主要处理 `output_token_ids`, `cu_num_draft_tokens`, `draft_token_ids`, `draft_probs` 等参数。 实现过程中会调用 `_ensure_int64`, `torch.ops._C.sample_recovered_tokens_kernel_impl`, `output_token_ids.copy_` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`_FuncWrapper`**: Core class that organizes module behavior. / **`_FuncWrapper`**：组织模块行为的核心类。
- **`_compute_slot_mapping_kernel_impl`**: Key helper or entry point in this file. / **`_compute_slot_mapping_kernel_impl`**：本文件中的关键辅助函数或入口。
- **`_ensure_int64`**: Key helper or entry point in this file. / **`_ensure_int64`**：本文件中的关键辅助函数或入口。
- **`_eagle_prepare_inputs_padded_kernel_impl`**: Key helper or entry point in this file. / **`_eagle_prepare_inputs_padded_kernel_impl`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: collections.abc
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: None / 无
