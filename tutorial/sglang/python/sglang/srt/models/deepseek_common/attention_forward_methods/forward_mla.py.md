# forward_mla.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_common/attention_forward_methods/forward_mla.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the MLA-based forward path used by DeepSeek attention execution. / 该模块实现基于 MLA 的前向路径，用于 DeepSeek 注意力执行。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
from __future__ import annotations
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 43-44: Conditional setup: TYPE_CHECKING / 条件初始化：TYPE_CHECKING
```python
if TYPE_CHECKING:
    from sglang.srt.models.deepseek_v2 import DeepseekV2AttentionMLA
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 46-72: Conditional setup: _is_cuda / 条件初始化：_is_cuda
```python
if _is_cuda:
    from sgl_kernel import bmm_fp8 as _raw_bmm_fp8

    from sglang.srt.utils.custom_op import register_custom_op

    # TODO(yuwei): remove this wrapper after sgl-kernel registers its own fake/meta impl
    # Wrap bmm_fp8 as a custom op so torch.compile does not trace into
    # torch.cuda.current_blas_handle() (which returns a non-Tensor).
    @register_custom_op(mutates_args=["out"])
    def _bmm_fp8_op(
        A: torch.Tensor,
        B: torch.Tensor,
        out: torch.Tensor,
        A_scale: torch.Tensor,
        B_scale: torch.Tensor,
    ) -> None:
        _raw_bmm_fp8(A, B, A_scale, B_scale, out.dtype, out)

    def bmm_fp8(A, B, A_scale, B_scale, dtype, out=None):
        if out is None:
            out = torch.empty(
                (A.shape[0], A.shape[1], B.shape[2]),
                device=A.device,
                dtype=dtype,
            )
        _bmm_fp8_op(A, B, out, A_scale, B_scale)
        return out
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 75-111: Conditional setup: _use_aiter / 条件初始化：_use_aiter
```python
if _use_aiter:
    # aiter ROCm/aiter#2958 renamed the public `fused_qk_rmsnorm` in
    # `aiter.ops.fused_qk_norm_rope_cache_quant` to a private `_fused_qk_rmsnorm`
    # and introduced a unified entry point in `aiter.ops.fused_qk_rmsnorm_group_quant`
    # with a different (in-place, kwarg-only, no-return) signature. Probe for the
    # new symbol first so SGLang works with both pre- and post-#2958 aiter without
    # requiring the docker pin to be bumped atomically.
    try:
        from aiter.ops.enum import QuantType as _AiterQuantType
        from aiter.ops.fused_qk_rmsnorm_group_quant import (
            fused_qk_rmsnorm as _aiter_fused_qk_rmsnorm_unified,
        )

        def fused_qk_rmsnorm_bf16(q, q_weight, q_eps, k, k_weight, k_eps):
            q_out = torch.empty_like(q)
            k_out = torch.empty_like(k)
            _aiter_fused_qk_rmsnorm_unified(
                q_out_quantized=q_out,
                k_out=k_out,
                q=q,
                q_weight=q_weight,
                q_epsilon=q_eps,
                k=k,
                k_weight=k_weight,
                k_epsilon=k_eps,
                quant_type=_AiterQuantType.No,
            )
            return q_out, k_out

    except ImportError:
        from aiter.ops.fused_qk_norm_rope_cache_quant import (
            fused_qk_rmsnorm as fused_qk_rmsnorm_bf16,
        )

    from aiter.ops.triton.batched_gemm_a8w8_a_per_token_group_prequant_w_per_batched_tensor_quant import (
        batched_gemm_a8w8_a_per_token_group_prequant_w_per_batched_tensor_quant,
    )
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 112-123: Conditional setup: _use_aiter_gfx95 / 条件初始化：_use_aiter_gfx95
```python
if _use_aiter_gfx95:
    from aiter.ops.triton.fused_fp8_quant import (
        fused_flatten_fp8_group_quant,
        fused_rms_fp8_group_quant,
    )

    from sglang.srt.layers.quantization.rocm_mxfp4_utils import (
        batched_gemm_afp4wfp4_pre_quant,
        fused_flatten_mxfp4_quant,
        fused_rms_mxfp4_quant,
    )
    from sglang.srt.layers.rocm_linear_utils import fused_qk_rope_cat_and_cache_mla
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 126-725: Class: DeepseekMLAForwardMixin / 类：DeepseekMLAForwardMixin
```python
class DeepseekMLAForwardMixin:

    def init_mla_forward(self: DeepseekV2AttentionMLA):
        self.flashinfer_mla_disable_ragged = (
            get_global_server_args().flashinfer_mla_disable_ragged
        )

    def forward_absorb_prepare(
        self: DeepseekV2AttentionMLA,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        zero_allocator: BumpAllocator,
        llama_4_scaling: Optional[torch.Tensor] = None,
        prev_topk_indices: Optional[torch.Tensor] = None,
    ):
        from sglang.srt.model_executor.cuda_graph_runner import get_is_capture_mode

        q_lora = None
        topk_indices = None
        if self.q_lora_rank is not None:
            q, latent_cache = (
                get_attn_tp_context()
                .fetch_qkv_latent()
                .split(
                    [self.q_lora_rank, self.kv_lora_rank + self.qk_rope_head_dim],
                    dim=-1,
                )
            )
            k_nope = latent_cache[..., : self.kv_lora_rank]

            # overlap qk norm
            if self.alt_stream is not None and get_is_capture_mode():
                current_stream = torch.cuda.current_stream()
                self.alt_stream.wait_stream(current_stream)
                q = self.q_a_layernorm(q)
                with torch.cuda.stream(self.alt_stream):
                    k_nope = self.kv_a_layernorm(k_nope)
                current_stream.wait_stream(self.alt_stream)
            else:
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek M L A Forward Mixin, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek M L A Forward Mixin，用于封装该模型组件的状态与方法。

## Key Concepts / 关键概念
- **EN:** Backend-aware dispatch that selects specialized attention implementations for different hardware and execution modes. / **CN:** 具备后端感知能力的分派机制，可针对不同硬件与执行模式选择特化注意力实现。
- **EN:** Shared utilities that reduce duplication across DeepSeek-family model integrations. / **CN:** 在 DeepSeek 系列模型集成中复用的共享工具，以减少重复实现。

## Dependencies / 依赖关系
- `__future__: annotations`
- `typing: TYPE_CHECKING, Optional`
- `torch`
- `sglang.srt.compilation.piecewise_context_manager: is_in_piecewise_cuda_graph`
- `sglang.srt.layers: deep_gemm_wrapper`
- `sglang.srt.layers.attention.nsa.utils: nsa_use_prefill_cp`
- `sglang.srt.layers.communicator: get_attn_tp_context`
- `sglang.srt.layers.quantization.fp8_kernel: fp8_dtype, per_tensor_quant_mla_fp8, per_token_group_quant_mla_deep_gemm_masked_fp8`
- `sglang.srt.lora.deepseek_mla_correction: apply_q_correction`
- `sglang.srt.lora.deepseek_mla_correction: apply_v_correction`
- `sglang.srt.lora.deepseek_mla_correction: is_kv_b_lora_active`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.models.deepseek_common.utils: FORWARD_ABSORB_CORE_ATTENTION_BACKENDS, _is_cpu, _is_cublas_ge_129, _is_cuda, _is_gfx95_supported, _is_hip, _is_musa, _use_aiter, _use_aiter_gfx95`
- `sglang.srt.server_args: get_global_server_args`
- `sglang.srt.state_capturer.indexer_topk: maybe_capture_indexer_topk`
- `sglang.srt.utils: BumpAllocator`
