# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/__init__.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Package export surface for fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子的包导出入口

## Line-by-Line Analysis / 逐行分析
### Lines 9-25 — imports and setup
```python
from .chunk import chunk_gated_delta_rule
from .fused_gdn_prefill_post_conv import fused_post_conv_prep
from .fused_recurrent import (
    fused_recurrent_gated_delta_rule,
    fused_recurrent_gated_delta_rule_packed_decode,
)
from .fused_sigmoid_gating import fused_sigmoid_gating_delta_rule_update
from .layernorm_guard import RMSNormGated

__all__ = [
    "RMSNormGated",
    "chunk_gated_delta_rule",
    "fused_recurrent_gated_delta_rule",
    "fused_recurrent_gated_delta_rule_packed_decode",
    "fused_post_conv_prep",
    "fused_sigmoid_gating_delta_rule_update",
]
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. It writes or updates `__all__`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 它会写入或更新 `__all__`。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子

## Dependencies / 依赖关系
- **External**: `chunk`, `fused_gdn_prefill_post_conv`, `fused_recurrent`, `fused_sigmoid_gating`, `layernorm_guard` / **外部依赖**: `chunk`, `fused_gdn_prefill_post_conv`, `fused_recurrent`, `fused_sigmoid_gating`, `layernorm_guard`
- **Internal**: none / **内部依赖**: 无
