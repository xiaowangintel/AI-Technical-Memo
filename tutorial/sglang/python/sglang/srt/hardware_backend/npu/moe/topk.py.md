# topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/moe/topk.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for backend-specific runtime support inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的后端运行时支持支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module setup and shared state / 模块设置与共享状态
```python
from typing import TYPE_CHECKING, Optional

import torch
from sgl_kernel_npu.norm.l1_norm import l1_norm

from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location_dispatch import topk_ids_logical_to_physical
from sglang.srt.layers.moe.topk import StandardTopKOutput, select_experts
from sglang.srt.state_capturer.routed_experts import get_global_experts_capturer

if TYPE_CHECKING:
    from sglang.srt.eplb.expert_location_dispatch import ExpertLocationDispatchInfo
    from sglang.srt.layers.moe.topk import TopKConfig, TopKOutput
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `sgl_kernel_npu.norm.l1_norm`, `sglang.srt.eplb.expert_distribution`, `sglang.srt.eplb.expert_location_dispatch`, `sglang.srt.layers.moe.topk`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `sgl_kernel_npu.norm.l1_norm`, `sglang.srt.eplb.expert_distribution`, `sglang.srt.eplb.expert_location_dispatch`, `sglang.srt.layers.moe.topk`。

### Lines 16-92: Function `fused_topk_npu` / 函数 `fused_topk_npu`
```python
def fused_topk_npu(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    topk_config: "TopKConfig",
    num_token_non_padded: Optional[torch.Tensor] = None,
    expert_location_dispatch_info: Optional["ExpertLocationDispatchInfo"] = None,
    layer_id: Optional[int] = None,
) -> "TopKOutput":

    use_grouped_topk = topk_config.use_grouped_topk
    renormalize = topk_config.renormalize
    correction_bias = topk_config.correction_bias

    # Fast path: simple top-k without grouped routing and bias
    if not use_grouped_topk and correction_bias is None:
        topk_weights, topk_ids, _ = torch.ops.npu.npu_moe_gating_top_k_softmax(
            router_logits,
            k=topk_config.top_k,
        )

        if renormalize:
            topk_weights = l1_norm(
                topk_weights
                if topk_config.num_fused_shared_experts == 0
                else topk_weights[:, :-1]
            )
        topk_weights = topk_weights.to(torch.float32)

# ... omitted for brevity ...
        cap.capture(
            layer_id=layer_id,
            topk_indices=topk_ids,
        )

    return StandardTopKOutput(topk_weights, topk_ids, router_logits)
```
**EN:** This function implements `fused_topk_npu`. It primarily calls `get_global_expert_distribution_recorder.on_select_experts`, `StandardTopKOutput`, `torch.ops.npu.npu_moe_gating_top_k_softmax`, `topk_weights.to`, `topk_ids_logical_to_physical`, `cap.capture` to complete its work. State updates are written into `use_grouped_topk`, `renormalize`, `correction_bias`, `topk_weights`, `topk_ids`, `topk_config.torch_native`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `fused_topk_npu`。 它主要通过调用 `get_global_expert_distribution_recorder.on_select_experts`, `StandardTopKOutput`, `torch.ops.npu.npu_moe_gating_top_k_softmax`, `topk_weights.to`, `topk_ids_logical_to_physical`, `cap.capture` 来完成任务。 状态更新主要写入 `use_grouped_topk`, `renormalize`, `correction_bias`, `topk_weights`, `topk_ids`, `topk_config.torch_native`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `fused_topk_npu`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.eplb.expert_distribution`, `sglang.srt.eplb.expert_location_dispatch`, `sglang.srt.layers.moe.topk`, `sglang.srt.state_capturer.routed_experts`
- **External / 外部依赖**: `torch`, `sgl_kernel_npu.norm.l1_norm`
- **Standard library / 标准库**: `typing`
