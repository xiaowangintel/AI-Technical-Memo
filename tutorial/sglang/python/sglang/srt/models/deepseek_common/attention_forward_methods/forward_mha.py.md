# forward_mha.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_common/attention_forward_methods/forward_mha.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the multi-head-attention-oriented forward path used by DeepSeek attention execution. / 该模块实现面向多头注意力的前向路径，用于 DeepSeek 注意力执行。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
from __future__ import annotations
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 23-25: Assignment: _use_fp8_prefill_attn / 赋值：_use_fp8_prefill_attn
```python
_use_fp8_prefill_attn = (
    get_bool_env_var("SGLANG_AITER_FP8_PREFILL_ATTN", "True") and _use_aiter_gfx95
)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 27-28: Conditional setup: TYPE_CHECKING / 条件初始化：TYPE_CHECKING
```python
if TYPE_CHECKING:
    from sglang.srt.models.deepseek_v2 import DeepseekV2AttentionMLA
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 30-31: Conditional setup: _is_cuda / 条件初始化：_is_cuda
```python
if _is_cuda:
    from sgl_kernel import concat_mla_k, merge_state_v2
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 33-37: Conditional setup: _use_aiter_gfx95 / 条件初始化：_use_aiter_gfx95
```python
if _use_aiter_gfx95:
    from aiter.ops.triton.fused_fp8_quant import fused_rms_fp8_group_quant

    from sglang.srt.layers.quantization.fp8_kernel import fp8_dtype
    from sglang.srt.layers.quantization.rocm_mxfp4_utils import fused_rms_mxfp4_quant
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 40-44: Function: _resolve_attn_backend() / 函数：_resolve_attn_backend()
```python
def _resolve_attn_backend(forward_batch: ForwardBatch):
    backend = forward_batch.attn_backend
    if isinstance(backend, TboAttnBackend):
        backend = backend.primary
    return backend
```
**EN:** This function implements resolve attn backend for the surrounding model/runtime logic. Key parameters include forward_batch.
**CN:** 该函数实现了 resolve attn backend 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 96-541: Class: DeepseekMHAForwardMixin / 类：DeepseekMHAForwardMixin
```python
class DeepseekMHAForwardMixin:

    def init_mha_forward(self: DeepseekV2AttentionMLA):
        self.disable_chunked_prefix_cache = (
            get_global_server_args().disable_chunked_prefix_cache
        )

        # TODO: Design a finer way to determine the threshold
        self.chunked_prefix_cache_threshold = (
            envs.SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD.get()
        )

    def forward_normal_prepare(
        self: DeepseekV2AttentionMLA,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        zero_allocator: BumpAllocator,
    ):
        if self.q_lora_rank is not None:
            q, latent_cache = (
                get_attn_tp_context()
                .fetch_qkv_latent()
                .split(
                    [self.q_lora_rank, self.kv_lora_rank + self.qk_rope_head_dim],
                    dim=-1,
                )
            )

            # NSA Indexer: cache quantized keys, auto-skip topk for sequences <= nsa_index_topk

            if self.use_nsa:
                # NSA requires unquantized q_lora for the indexer. When q_b_proj is FP8
                # on gfx95, we can still use fused RMSNorm+FP8 quant, but MUST request
                # the unquantized output for q_lora; otherwise q_lora becomes the (fp8,scale)
                # tuple.
                if (
                    _use_aiter_gfx95
                    and self.q_b_proj.weight.dtype == torch.float8_e4m3fn
                ):
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek M H A Forward Mixin, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek M H A Forward Mixin，用于封装该模型组件的状态与方法。

## Key Concepts / 关键概念
- **EN:** Backend-aware dispatch that selects specialized attention implementations for different hardware and execution modes. / **CN:** 具备后端感知能力的分派机制，可针对不同硬件与执行模式选择特化注意力实现。
- **EN:** Shared utilities that reduce duplication across DeepSeek-family model integrations. / **CN:** 在 DeepSeek 系列模型集成中复用的共享工具，以减少重复实现。

## Dependencies / 依赖关系
- `__future__: annotations`
- `typing: TYPE_CHECKING`
- `torch`
- `sglang.srt.environ: envs`
- `sglang.srt.layers.attention.nsa.dequant_k_cache: dequantize_k_cache_paged`
- `sglang.srt.layers.attention.tbo_backend: TboAttnBackend`
- `sglang.srt.layers.attention.utils: concat_and_cast_mha_k_triton`
- `sglang.srt.layers.communicator: get_attn_tp_context`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.models.deepseek_common.utils: _is_cuda, _is_hip, _is_musa, _is_npu, _use_aiter_gfx95`
- `sglang.srt.server_args: get_global_server_args`
- `sglang.srt.utils: BumpAllocator, get_bool_env_var, next_power_of_2`
