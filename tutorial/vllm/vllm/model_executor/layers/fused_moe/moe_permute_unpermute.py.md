# moe_permute_unpermute.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/moe_permute_unpermute.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-4 — imports and setup
```python
import torch
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 7-97 — function `moe_permute`
```python
def moe_permute(
    hidden_states: torch.Tensor,
    a1q_scale: torch.Tensor | None,
    topk_ids: torch.Tensor,
    n_expert: int,
    n_local_expert: int = -1,
    expert_map: torch.Tensor | None = None,
    permuted_hidden_states: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor | None, torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    This function expands and permutes activation to gather uncontinuous tokens
      for each expert.
    Parameters:
    - hidden_states (torch.Tensor): The input tensor to the MoE layer.
    - a1q_scale (Optional[torch.Tensor]): quant scale for hidden_states
    - topk_ids (torch.Tensor): topk expert route id for each token.
    - n_expert (int): The number of expert.
    - n_local_expert (int): The number of expert in current EP rank.
    - expert_map (Optional[torch.Tensor]):  A tensor mapping expert indices
        from the global expert space to the local expert space of the expert
        parallel shard.
    - permuted_hidden_states (Optional[torch.Tensor]): Optional output tensor.
        If None, the output tensor will be created in this function.
    Returns:
    - permuted_hidden_states (torch.Tensor): permuted activation.
    - a1q_scale (Optional[torch.Tensor]): permuted quant scale for hidden_states
        if original scale not per-tensor scaling
    - expert_first_token_offset (torch.Tensor): offset of the first token
       of each expert for standard grouped gemm.
    - inv_permuted_idx (torch.Tensor): idx map for moe_unpermute.
    - permuted_idx (torch.Tensor): idx map from hidden to permuted_hidden.
    """
    n_token, n_hidden = hidden_states.size()
    topk = topk_ids.size(1)
    assert (n_hidden * hidden_states.element_size()) % 16 == 0, (
        "permue kernel need hidden dim align to 16B"
    )
    permuted_row_size = n_token * topk
    if n_local_expert == -1:
        n_local_expert = n_expert
    if permuted_hidden_states is None:
        permuted_hidden_states = torch.empty(
            (permuted_row_size, n_hidden),
            dtype=hidden_states.dtype,
# ... omitted for brevity ...
        permuted_idx,
    )
```
**EN:** This function defines `moe_permute`. This function expands and permutes activation to gather uncontinuous tokens for each expert. The main inputs are `hidden_states`, `a1q_scale`, `topk_ids`, `n_expert`, `n_local_expert`, `expert_map`. Key calls include `hidden_states.size`, `topk_ids.size`, `torch.arange.reshape`, `torch.empty`, `torch.full`, `topk_ids.to`. It writes or updates `n_token`, `n_hidden`, `topk`, `permuted_row_size`, `token_expert_indices`, `expert_first_token_offset`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `moe_permute`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `a1q_scale`, `topk_ids`, `n_expert`, `n_local_expert`, `expert_map`。 关键调用包括 `hidden_states.size`, `topk_ids.size`, `torch.arange.reshape`, `torch.empty`, `torch.full`, `topk_ids.to`。 它会写入或更新 `n_token`, `n_hidden`, `topk`, `permuted_row_size`, `token_expert_indices`, `expert_first_token_offset`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 100-134 — function `moe_unpermute`
```python
def moe_unpermute(
    out: torch.Tensor,
    permuted_hidden_states: torch.Tensor,
    topk_weights: torch.Tensor,
    inv_permuted_idx: torch.Tensor,
    expert_first_token_offset: torch.Tensor | None = None,
) -> None:
    """
    This function expands and permutes activation to gathering uncontinuous
      tokens for each expert.
    Parameters:
    - out (torch.Tensor): output tensor
    - permuted_hidden_states (torch.Tensor): permuted activation.
    - topk_weights (torch.Tensor): topk expert route weight for each token.
    - inv_permuted_idx (torch.Tensor): row idx map for moe_unpermute.
    - expert_first_token_offset (Optional[torch.Tensor]): offset of the first
      token of each expert for grouped gemm.
    Returns:
    - hidden_states (torch.Tensor): The reduced and unpermuted activation
      tensor.
    """
    topk = topk_weights.size(1)
    n_hidden = permuted_hidden_states.size(-1)
    assert (n_hidden * permuted_hidden_states.element_size()) % 16 == 0, (
        "unpermue kernel need hidden dim align to 16B"
    )

    torch.ops._moe_C.moe_unpermute(
        permuted_hidden_states,
        topk_weights,
        inv_permuted_idx,
        expert_first_token_offset,
        topk,
        out,
    )
```
**EN:** This function defines `moe_unpermute`. This function expands and permutes activation to gathering uncontinuous tokens for each expert. The main inputs are `out`, `permuted_hidden_states`, `topk_weights`, `inv_permuted_idx`, `expert_first_token_offset`. Key calls include `topk_weights.size`, `permuted_hidden_states.size`, `torch.ops._moe_C.moe_unpermute`, `permuted_hidden_states.element_size`. It writes or updates `topk`, `n_hidden`.
**CN:** 该函数定义 `moe_unpermute`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `out`, `permuted_hidden_states`, `topk_weights`, `inv_permuted_idx`, `expert_first_token_offset`。 关键调用包括 `topk_weights.size`, `permuted_hidden_states.size`, `torch.ops._moe_C.moe_unpermute`, `permuted_hidden_states.element_size`。 它会写入或更新 `topk`, `n_hidden`。

### Lines 137-138 — function `moe_permute_unpermute_supported`
```python
def moe_permute_unpermute_supported():
    return torch.ops._moe_C.moe_permute_unpermute_supported()
```
**EN:** This function defines `moe_permute_unpermute_supported`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `torch.ops._moe_C.moe_permute_unpermute_supported`.
**CN:** 该函数定义 `moe_permute_unpermute_supported`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `torch.ops._moe_C.moe_permute_unpermute_supported`。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `moe_permute`, `moe_unpermute`, `moe_permute_unpermute_supported` / [CN] 核心符号：`moe_permute`, `moe_unpermute`, `moe_permute_unpermute_supported`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: none / **内部依赖**: 无
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
