# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/triton_ops/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides Triton kernels or kernel wrappers for LoRA-specific matrix, embedding, or expert operations. It focuses on GPU-efficient execution for high-throughput adapter inference. / 该文件提供了面向 LoRA 的 Triton 内核或内核封装，用于矩阵、Embedding 或专家相关运算。它重点优化高吞吐适配器推理时的 GPU 执行效率。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33: Module header, imports, and shared constants
```python
from .chunked_embedding_lora_a import chunked_embedding_lora_a_forward
from .chunked_sgmv_expand import chunked_sgmv_lora_expand_forward
from .chunked_sgmv_shrink import chunked_sgmv_lora_shrink_forward
from .embedding_lora_a import embedding_lora_a_fwd
from .fused_moe_lora_kernel import fused_moe_lora
from .gate_up_lora_b import gate_up_lora_b_fwd
from .kv_b_lora_absorbed import (
    step_a_q_fwd,
    step_a_v_fwd,
    step_b_q_fwd,
    step_b_v_fwd,
)
from .qkv_lora_b import qkv_lora_b_fwd
from .sgemm_lora_a import sgemm_lora_a_fwd
from .sgemm_lora_b import sgemm_lora_b_fwd
from .virtual_experts import merged_experts_fused_moe_lora_add

__all__ = [
    "gate_up_lora_b_fwd",
    "qkv_lora_b_fwd",
    "sgemm_lora_a_fwd",
    "sgemm_lora_b_fwd",
    "chunked_sgmv_lora_shrink_forward",
    "chunked_sgmv_lora_expand_forward",
    "fused_moe_lora",
    "chunked_embedding_lora_a_forward",
    "embedding_lora_a_fwd",
    "merged_experts_fused_moe_lora_add",
    "step_a_q_fwd",
    "step_a_v_fwd",
    "step_b_q_fwd",
    "step_b_v_fwd",
]
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Triton GPU kernels / Triton GPU 内核

## Dependencies / 依赖关系
### Internal / 内部
- None / 无
### External / 外部
- `chunked_embedding_lora_a`
- `chunked_sgmv_expand`
- `chunked_sgmv_shrink`
- `embedding_lora_a`
- `fused_moe_lora_kernel`
- `gate_up_lora_b`
- `kv_b_lora_absorbed`
- `qkv_lora_b`
- `sgemm_lora_a`
- `sgemm_lora_b`
- `virtual_experts`
