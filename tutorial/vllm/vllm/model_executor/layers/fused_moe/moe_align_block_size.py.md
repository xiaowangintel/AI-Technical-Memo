# moe_align_block_size.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/moe_align_block_size.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-8 — imports and setup
```python
import torch

from vllm import _custom_ops as ops
from vllm.triton_utils import triton
from vllm.utils.math_utils import round_up
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 11-103 — function `moe_align_block_size`
```python
def moe_align_block_size(
    topk_ids: torch.Tensor,
    block_size: int,
    num_experts: int,
    expert_map: torch.Tensor | None = None,
    pad_sorted_ids: bool = False,
    ignore_invalid_experts: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Aligns the token distribution across experts to be compatible with block
    size for matrix multiplication.

    Note: In the case of expert_parallel, moe_align_block_size initially
    considers all experts as valid and aligns all tokens appropriately.
    Before the function returns it marks the experts_ids that are not in
    the current GPU rank as -1 so the MoE matmuls could skip those blocks.
    This requires the num_experts input arg to be the num global experts.

    Parameters:
    - topk_ids: A tensor of shape [total_tokens, top_k] representing the
        top-k expert indices for each token.
    - block_size: The block size used in block matrix multiplication.
    - num_experts: The total number of experts.
    - expert_map: A tensor of shape [num_experts] that maps the expert index
        from the global space to the local index space of the current
        expert parallel shard. If the expert is not in the current expert
        parallel shard, the mapping is set to -1.
    - pad_sorted_ids: A flag indicating whether the sorted_token_ids length
        should be padded to a multiple of block_size,
    - ignore_invalid_experts: A flag indicating whether to ignore invalid
        experts. When False, all expert_ids in topk_ids will participate in
        counting and ranking, but invalid experts in expert_ids will be marked
        as -1. When True, all invalid expert_ids in topk_ids will be ignored
        and will not participate in counting or ranking, and there will be no
        -1 in expert_ids.

    Returns:
    - sorted_token_ids: A tensor containing the sorted token indices according
        to their allocated expert.
    - expert_ids: A tensor indicating the assigned expert index for each block.
    - num_tokens_post_padded: The total number of tokens after padding,
        ensuring divisibility by block_size.

    This function pads the number of tokens that each expert needs to process
# ... omitted for brevity ...

    return sorted_ids, expert_ids, num_tokens_post_pad
```
**EN:** This function defines `moe_align_block_size`. Aligns the token distribution across experts to be compatible with block size for matrix multiplication. The main inputs are `topk_ids`, `block_size`, `num_experts`, `expert_map`, `pad_sorted_ids`, `ignore_invalid_experts`. Key calls include `torch.empty`, `triton.cdiv`, `ops.moe_align_block_size`, `topk_ids.numel`, `round_up`, `min`. It writes or updates `max_num_tokens_padded`, `sorted_ids`, `max_num_m_blocks`, `expert_ids`, `num_tokens_post_pad`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `moe_align_block_size`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `topk_ids`, `block_size`, `num_experts`, `expert_map`, `pad_sorted_ids`, `ignore_invalid_experts`。 关键调用包括 `torch.empty`, `triton.cdiv`, `ops.moe_align_block_size`, `topk_ids.numel`, `round_up`, `min`。 它会写入或更新 `max_num_tokens_padded`, `sorted_ids`, `max_num_m_blocks`, `expert_ids`, `num_tokens_post_pad`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 106-192 — function `batched_moe_align_block_size`
```python
def batched_moe_align_block_size(
    max_tokens_per_batch: int, block_size: int, expert_num_tokens: torch.Tensor
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Given num_batches, max_tokens_per_batch, block_size and the number of
    valid-tokens in each batch, prepare sorted_token_ids, expert_ids and
    num_tokens_post_pad. sorted_token_ids, expert_ids and num_tokens_post_pad
    have the same semantics as in moe_align_block_size.

    This function is intended to be a drop in replacement for
    moe_align_batch_size for the batched case.

    Parameters:
    - max_tokens_per_batch (int): Number of tokens in each batch (both
        valid and invalid).
    - block_size (int): block_size to align the data to.
    - expert_num_tokens (torch.Tensor): expert_num_tokens[i], indicates
        the number of valid tokens in batch i.

    Returns:
    - sorted_token_ids (torch.Tensor): Torch tensor of size
        (num_batches * max_tokens_per_batch) indicating the token indices for
        that block.
    - expert_ids (torch.Tensor): Torch tensor of size
        ceil((num_batches * max_tokens_per_batch) / block_size) indicating
        what expert to use for each block.
    - num_tokens_post_pad (torch.Tensor): Torch tensor of size 1
        indicating the number of valid blocks with actual data to
        process. This is represented in terms of num tokens.
    Example:
    Let num_batches=5, max_tokens_per_batch=8, block_size=4, and
    expert_num_tokens=[2, 3, 0, 6, 8]. This expert_num_tokens tensor
    indicates that,
     - The first 2 tokens in the 0th batch are valid and the rest 6 are
     invalid (i.e. in the 2D hidden_states tensor of shape,
     [num_batches * max_tokens_per_batch, K], indices 0, 1 are valid)
     - The first 3 tokens in the 1st batch are valid. i.e. indices 8, 9, 10
     - 0 tokens in the 2nd batch are valid
     - first 6 tokens in the  3rd batch are valid. i.e. indices,
     24, 25, 26, 27, 28, 29
     - so on ...

     In this case,
      sorted_token_ids will be [0, 1, 40, 40,
# ... omitted for brevity ...

    return sorted_ids, expert_ids, num_tokens_post_pad
```
**EN:** This function defines `batched_moe_align_block_size`. Given num_batches, max_tokens_per_batch, block_size and the number of valid-tokens in each batch, prepare sorted_token_ids, expert_ids and num_tokens_post_pad. The main inputs are `max_tokens_per_batch`, `block_size`, `expert_num_tokens`. Key calls include `expert_num_tokens.size`, `torch.empty`, `ops.batched_moe_align_block_size`, `round_up`. It writes or updates `B`, `device`, `max_num_tokens_padded`, `sorted_ids`, `max_num_m_blocks`, `expert_ids`.
**CN:** 该函数定义 `batched_moe_align_block_size`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `max_tokens_per_batch`, `block_size`, `expert_num_tokens`。 关键调用包括 `expert_num_tokens.size`, `torch.empty`, `ops.batched_moe_align_block_size`, `round_up`。 它会写入或更新 `B`, `device`, `max_num_tokens_padded`, `sorted_ids`, `max_num_m_blocks`, `expert_ids`。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `moe_align_block_size`, `batched_moe_align_block_size` / [CN] 核心符号：`moe_align_block_size`, `batched_moe_align_block_size`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm`, `vllm.triton_utils`, `vllm.utils.math_utils` / **内部依赖**: `vllm`, `vllm.triton_utils`, `vllm.utils.math_utils`
- **Runtime traits**: Triton kernels, distributed collectives / **运行时特征**: Triton kernels, distributed collectives
