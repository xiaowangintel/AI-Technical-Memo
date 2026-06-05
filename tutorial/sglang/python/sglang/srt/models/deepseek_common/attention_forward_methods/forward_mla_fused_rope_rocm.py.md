# forward_mla_fused_rope_rocm.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_common/attention_forward_methods/forward_mla_fused_rope_rocm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides a ROCm-specific fused RoPE preparation path for DeepSeek MLA attention. / 该模块为 DeepSeek MLA 注意力提供 ROCm 专用的融合 RoPE 预处理路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
from __future__ import annotations
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 16-17: Conditional setup: TYPE_CHECKING / 条件初始化：TYPE_CHECKING
```python
if TYPE_CHECKING:
    from sglang.srt.models.deepseek_v2 import DeepseekV2AttentionMLA
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 19-20: Conditional setup: _is_cuda / 条件初始化：_is_cuda
```python
if _is_cuda:
    from sgl_kernel import bmm_fp8
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 22-25: Conditional setup: _is_hip / 条件初始化：_is_hip
```python
if _is_hip:
    from sglang.srt.layers.attention.triton_ops.rocm_mla_decode_rope import (
        decode_attention_fwd_grouped_rope,
    )
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 28-227: Class: DeepseekMLARocmForwardMixin / 类：DeepseekMLARocmForwardMixin
```python
class DeepseekMLARocmForwardMixin:

    def init_mla_fused_rope_rocm_forward(self: DeepseekV2AttentionMLA):
        self.rocm_fused_decode_mla = get_bool_env_var(
            "SGLANG_ROCM_FUSED_DECODE_MLA", "false"
        )

    def forward_absorb_fused_mla_rope_prepare(
        self: DeepseekV2AttentionMLA,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        zero_allocator: BumpAllocator,
    ):
        enable_rope_fusion = (
            os.getenv("SGLANG_FUSED_MLA_ENABLE_ROPE_FUSION", "1") == "1"
        )
        # NOTE: hidden_states can be a tuple for some quantization paths.
        # For shape/device/dtype, use the first tensor; still pass the original
        # hidden_states through linear ops which may accept tuple inputs.
        hidden_states_tensor = (
            hidden_states[0] if isinstance(hidden_states, tuple) else hidden_states
        )

        q_len = hidden_states_tensor.shape[0]
        q_input = hidden_states_tensor.new_empty(
            q_len, self.num_local_heads, self.kv_lora_rank + self.qk_rope_head_dim
        )
        if self.q_lora_rank is not None:
            q, latent_cache = self.fused_qkv_a_proj_with_mqa(hidden_states)[0].split(
                [self.q_lora_rank, self.kv_lora_rank + self.qk_rope_head_dim], dim=-1
            )
            q = self.q_a_layernorm(q)
            q = self.q_b_proj(q)[0].view(-1, self.num_local_heads, self.qk_head_dim)
        else:
            q = self.q_proj(hidden_states)[0].view(
                -1, self.num_local_heads, self.qk_head_dim
            )
            latent_cache = self.kv_a_proj_with_mqa(hidden_states)[0]
        q_nope, q_pe = q.split([self.qk_nope_head_dim, self.qk_rope_head_dim], dim=-1)
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek M L A Rocm Forward Mixin, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek M L A Rocm Forward Mixin，用于封装该模型组件的状态与方法。

## Key Concepts / 关键概念
- **EN:** Backend-aware dispatch that selects specialized attention implementations for different hardware and execution modes. / **CN:** 具备后端感知能力的分派机制，可针对不同硬件与执行模式选择特化注意力实现。
- **EN:** Shared utilities that reduce duplication across DeepSeek-family model integrations. / **CN:** 在 DeepSeek 系列模型集成中复用的共享工具，以减少重复实现。

## Dependencies / 依赖关系
- `__future__: annotations`
- `os`
- `typing: TYPE_CHECKING`
- `torch`
- `sglang.srt.layers.quantization.fp8_kernel: per_tensor_quant_mla_fp8`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.models.deepseek_common.utils: _is_cuda, _is_hip`
- `sglang.srt.utils: BumpAllocator, get_bool_env_var`
