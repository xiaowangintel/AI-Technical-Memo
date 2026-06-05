# lora_context.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/lora_context.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-8 — imports and setup
```python
from dataclasses import dataclass

import torch

from vllm.lora.punica_wrapper.punica_base import PunicaWrapperBase
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 11-51 — class `MoELoRAContext`
```python
@dataclass
class MoELoRAContext:
    """
    Carries all LoRA state for one MoE forward pass.

    Built by FusedMoEWithLoRA.forward() and propagated explicitly through the
    modular kernel path (FusedMoEKernel -> FusedMoEExpertsModular.apply) so
    that TritonExperts.apply() can compute the LoRA contribution inline,
    replacing the decorator-based monkey-patch approach.
    """

    # LoRA weight tensors (same shapes as FusedMoEWithLoRA attributes)
    w13_lora_a_stacked: tuple[torch.Tensor, ...]
    w13_lora_b_stacked: tuple[torch.Tensor, ...]
# ... omitted for brevity ...
    # fall back to punica_wrapper.token_mapping_meta.
    local_token_lora_mapping: torch.Tensor | None = None
```
**EN:** This class defines `MoELoRAContext`. Carries all LoRA state for one MoE forward pass. It writes or updates `w13_lora_a_stacked`, `w13_lora_b_stacked`, `w2_lora_a_stacked`, `w2_lora_b_stacked`, `adapter_enabled`, `max_loras`.
**CN:** 该类定义了 `MoELoRAContext`。 该类的文档字符串说明了它在当前模块中的职责。 它会写入或更新 `w13_lora_a_stacked`, `w13_lora_b_stacked`, `w2_lora_a_stacked`, `w2_lora_b_stacked`, `adapter_enabled`, `max_loras`。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `MoELoRAContext` / [CN] 核心符号：`MoELoRAContext`

## Dependencies / 依赖关系
- **External**: `dataclasses`, `torch` / **外部依赖**: `dataclasses`, `torch`
- **Internal**: `vllm.lora.punica_wrapper.punica_base` / **内部依赖**: `vllm.lora.punica_wrapper.punica_base`
