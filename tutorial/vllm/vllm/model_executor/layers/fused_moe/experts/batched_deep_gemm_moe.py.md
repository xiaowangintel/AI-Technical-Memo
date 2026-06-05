# batched_deep_gemm_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/batched_deep_gemm_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-37 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.forward_context import get_forward_context, is_forward_context_available
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceDelegate,
)
from vllm.model_executor.layers.fused_moe.utils import _resize_cache
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    get_fp8_min_max,
    kFp8Dynamic128Sym,
    kFp8Static128BlockSym,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.deep_gemm import (
    DeepGemmQuantScaleFMT,
    fp8_m_grouped_gemm_nt_masked,
    get_mk_alignment_for_contiguous_layout,
    is_deep_gemm_e8m0_used,
    is_deep_gemm_supported,
)
from vllm.utils.math_utils import cdiv, round_up

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 40-54 — function `scales_shape_stride_dtype`
```python
def scales_shape_stride_dtype(
    E: int, T: int, G: int, quant_scale_fmt: DeepGemmQuantScaleFMT
) -> tuple[tuple[int, ...], tuple[int, ...], torch.dtype]:
    shape = (E, T, G)
    strides = (T * G, 1, T)
    if quant_scale_fmt in [
        DeepGemmQuantScaleFMT.FLOAT32,
        DeepGemmQuantScaleFMT.FLOAT32_CEIL_UE8M0,
    ]:
        return shape, strides, torch.float32

    assert quant_scale_fmt == DeepGemmQuantScaleFMT.UE8M0
    shape = (E, T, cdiv(G, 4))
    strides = (T * cdiv(G, 4), 1, T)
    return shape, strides, torch.int32
```
**EN:** This function defines `scales_shape_stride_dtype`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `E`, `T`, `G`, `quant_scale_fmt`. Key calls include `cdiv`. It writes or updates `shape`, `strides`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `scales_shape_stride_dtype`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `E`, `T`, `G`, `quant_scale_fmt`。 关键调用包括 `cdiv`。 它会写入或更新 `shape`, `strides`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 57-131 — function `_silu_mul_fp8_quant_deep_gemm`
```python
@triton.jit
def _silu_mul_fp8_quant_deep_gemm(
    # Pointers ------------------------------------------------------------
    input_ptr,  # 16-bit activations (E, T, 2*H)
    y_q_ptr,  # fp8 quantized activations (E, T, H)
    y_s_ptr,  # 16-bit scales (E, T, G)
    counts_ptr,  # int32 num tokens per expert (E)
    # Sizes ---------------------------------------------------------------
    H: tl.constexpr,  # hidden dimension (per output)
    GROUP_SIZE: tl.constexpr,  # elements per group (usually 128)
    # Strides for input (elements) ---------------------------------------
    stride_i_e,
    stride_i_t,
    stride_i_h,
    # Strides for y_q (elements) -----------------------------------------
    stride_yq_e,
    stride_yq_t,
    stride_yq_h,
    # Strides for y_s (elements) -----------------------------------------
    stride_ys_e,
    stride_ys_t,
    stride_ys_g,
    # Stride for counts (elements)
    stride_counts_e,
    # Numeric params ------------------------------------------------------
    eps: tl.constexpr,
    fp8_min: tl.constexpr,
    fp8_max: tl.constexpr,
    ceil_ue8m0: tl.constexpr,
    # Meta ---------------------------------------------------------------
    BLOCK: tl.constexpr,
    NUM_STAGES: tl.constexpr,
):
    G = H // GROUP_SIZE

    # map program id -> (e, g)
    pid = tl.program_id(0)
    e = pid // G
    g = pid % G

    e = e.to(tl.int64)
    g = g.to(tl.int64)

    # number of valid tokens for this expert
    n_tokens = tl.load(counts_ptr + e * stride_counts_e).to(tl.int64)

    cols = tl.arange(0, BLOCK).to(tl.int64)
    mask = cols < BLOCK

    base_input_offset = e * stride_i_e + g * GROUP_SIZE * stride_i_h
    base_gate_offset = base_input_offset + cols * stride_i_h
    base_up_offset = base_input_offset + H * stride_i_h + cols * stride_i_h
    base_yq_offset = e * stride_yq_e + g * GROUP_SIZE * stride_yq_h + cols * stride_yq_h
    base_ys_offset = e * stride_ys_e + g * stride_ys_g

    for t in tl.range(0, n_tokens, num_stages=NUM_STAGES):
# ... omitted for brevity ...
        tl.store(y_q_ptr + base_yq_offset + t * stride_yq_t, y_q, mask=mask)
        tl.store(y_s_ptr + base_ys_offset + t * stride_ys_t, y_s)
```
**EN:** This function defines `_silu_mul_fp8_quant_deep_gemm`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `input_ptr`, `y_q_ptr`, `y_s_ptr`, `counts_ptr`, `H`, `GROUP_SIZE`. Key calls include `tl.program_id`, `e.to`, `g.to`, `tl.load.to`, `tl.arange.to`, `tl.range`. It writes or updates `G`, `pid`, `e`, `g`, `n_tokens`, `cols`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `_silu_mul_fp8_quant_deep_gemm`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `input_ptr`, `y_q_ptr`, `y_s_ptr`, `counts_ptr`, `H`, `GROUP_SIZE`。 关键调用包括 `tl.program_id`, `e.to`, `g.to`, `tl.load.to`, `tl.arange.to`, `tl.range`。 它会写入或更新 `G`, `pid`, `e`, `g`, `n_tokens`, `cols`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 134-266 — function `persistent_masked_m_silu_mul_quant`
```python
def persistent_masked_m_silu_mul_quant(
    y: torch.Tensor,  # (E, T, 2*H)
    tokens_per_expert: torch.Tensor,  # (E,) number of valid tokens per expert
    num_parallel_tokens=16,
    group_size: int = 128,
    quant_scale_fmt: DeepGemmQuantScaleFMT = DeepGemmQuantScaleFMT.FLOAT32,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Quantize silu(y[..., :H]) * y[..., H:] to FP8 with group per-token scales
    y has shape (E, T, 2*H). The first half of the last dimension is
    silu-activated, multiplied by the second half, then quantized into FP8.
    We launch a fixed grid of threads to accommodate CUDA graphs. Let `P2`
    be a parallelization factor for persistent_masked_m_silu_mul_quant over the
    hidden dimension.

    Let `expert_offsets = [0] + [num_tokens.cumsum()]` and
    `total_tokens = expert_offsets[-1]`.
    persistent_masked_m_silu_mul_quant launches `total_tokens x P2` number of
    thread blocks. Each thread block contains `NUM_WARPS` warps.

    Every thread block needs to find it's corresponding expert by warp-parallel scanning
    over the `expert_offsets` array.

    The i-th warp in the first thread block processes
    `[i * warp_chunk_size, (i + 1) * warp_chunk_size]` groups
    sequentially, where `warp_chunk_size = ((H / GROUP_SIZE) / P2) / NUM_WARPS`,
    pipelining loads and computes.

    The shared memory layout for 4 warps with a 2-stage pipeline for SiLU V2
    can is visualized like so:

                         stage0                              stage1
    ┌─────┬───┬─────┬───┬─────┬───┬─────┬───┬─────┬───┬─────┬───┬─────┬───┬─────┬───┐
    │gate0│up0│gate1│up1│gate2│up2│gate3│up3│gate0│up0│gate1│up1│gate2│up2│gate3│up3│
    └─────┴───┴─────┴───┴─────┴───┴─────┴───┴─────┴───┴─────┴───┴─────┴───┴─────┴───┘

    with the main difference between V1 and V2 being the global load
    stride between warps, and between half-warps. Regarding the latter stride,
    we assign the first half warp of every warp for `gate` loads and the second
    half-warp to `up` loads.

    Returns `(y_q, y_s)` where
    * `y_q`: FP8 tensor, shape (E, T, H), same layout as y[..., :H]
    * `y_s` depends on quant_scale_fmt,
      - quant_scale_fmt == FLOAT32,
# ... omitted for brevity ...

    return y_q, y_s
```
**EN:** This function defines `persistent_masked_m_silu_mul_quant`. Quantize silu(y[..., :H]) * y[..., H:] to FP8 with group per-token scales y has shape (E, T, 2*H). The main inputs are `y`, `tokens_per_expert`, `num_parallel_tokens`, `group_size`, `quant_scale_fmt`. Key calls include `tokens_per_expert.to`, `current_platform.fp8_dtype`, `torch.empty`, `scales_shape_stride_dtype`, `torch.empty_strided`, `current_platform.get_device_capability`. It writes or updates `E`, `T`, `H2`, `H`, `G`, `tokens_per_expert`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `persistent_masked_m_silu_mul_quant`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `y`, `tokens_per_expert`, `num_parallel_tokens`, `group_size`, `quant_scale_fmt`。 关键调用包括 `tokens_per_expert.to`, `current_platform.fp8_dtype`, `torch.empty`, `scales_shape_stride_dtype`, `torch.empty_strided`, `current_platform.get_device_capability`。 它会写入或更新 `E`, `T`, `H2`, `H`, `G`, `tokens_per_expert`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 269-448 — class `BatchedDeepGemmExperts`
```python
class BatchedDeepGemmExperts(mk.FusedMoEExpertsModular):
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int,
        num_dispatchers: int,
    ):
        """
        max_num_tokens: Maximum number of tokens from a DP Rank
        num_dispatchers: The number of DP dispatchers.
        quant_config: Quantization configuration
        """
        super().__init__(
# ... omitted for brevity ...
            expected_m,
        )
```
**EN:** This class defines `BatchedDeepGemmExperts`. It inherits from `mk.FusedMoEExpertsModular`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `apply`, `activation_format`, `supports_expert_map`, `supports_packed_ue8m0_act_scales`, `finalize_weight_and_reduce_impl`. Key calls include `super.__init__`, `is_deep_gemm_supported`, `TopKWeightAndReduceDelegate`, `self.adjust_N_for_activation`, `dp_meta.num_tokens_across_dp_cpu.sum.item`, `round_up`. It writes or updates `SUPPORTED_W_A`, `num_dispatchers`, `num_experts`, `max_num_tokens`, `activation_out_dim`, `workspace13`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `BatchedDeepGemmExperts`。 它继承自 `mk.FusedMoEExpertsModular`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `apply`, `activation_format`, `supports_expert_map`, `supports_packed_ue8m0_act_scales`, `finalize_weight_and_reduce_impl`。 关键调用包括 `super.__init__`, `is_deep_gemm_supported`, `TopKWeightAndReduceDelegate`, `self.adjust_N_for_activation`, `dp_meta.num_tokens_across_dp_cpu.sum.item`, `round_up`。 它会写入或更新 `SUPPORTED_W_A`, `num_dispatchers`, `num_experts`, `max_num_tokens`, `activation_out_dim`, `workspace13`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 270-289 — method `BatchedDeepGemmExperts.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int,
        num_dispatchers: int,
    ):
        """
        max_num_tokens: Maximum number of tokens from a DP Rank
        num_dispatchers: The number of DP dispatchers.
        quant_config: Quantization configuration
        """
        super().__init__(
            moe_config=moe_config,
            quant_config=quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=num_dispatchers,
        )
        assert self.block_shape == get_mk_alignment_for_contiguous_layout()
        assert self.quant_config.use_fp8_w8a8
```
**EN:** This method defines `__init__`. max_num_tokens: Maximum number of tokens from a DP Rank num_dispatchers: The number of DP dispatchers. The main inputs are `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`. Key calls include `super.__init__`, `get_mk_alignment_for_contiguous_layout`, `super`.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`。 关键调用包括 `super.__init__`, `get_mk_alignment_for_contiguous_layout`, `super`。

### Lines 386-448 — method `BatchedDeepGemmExperts.apply`
```python
    def apply(
        self,
        output: torch.Tensor,
        hidden_states: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        activation: MoEActivation,
        global_num_experts: int,
        expert_map: torch.Tensor | None,
        a1q_scale: torch.Tensor | None,
        a2_scale: torch.Tensor | None,
        workspace13: torch.Tensor,
        workspace2: torch.Tensor,
        expert_tokens_meta: mk.ExpertTokensMetadata | None,
        apply_router_weight_on_input: bool,
    ):
        assert expert_tokens_meta is not None
        expert_num_tokens = expert_tokens_meta.expert_num_tokens

        assert hidden_states.ndim == 3
        assert self.block_shape is not None

        a1q = hidden_states
        _, N, K = w1.size()

        assert w2.size(1) == K

        E, max_num_tokens, N, K, _ = self.moe_problem_size(
            hidden_states, w1, w2, topk_ids
        )

        workspace1 = _resize_cache(workspace13, (E, max_num_tokens, N))

        expected_m = self.estimate_expected_m(
            global_num_experts=global_num_experts,
            max_tokens_per_expert=max_num_tokens,
            topk=topk_ids.size(-1),
        )

        fp8_m_grouped_gemm_nt_masked(
            (a1q, a1q_scale),
            (w1, self.w1_scale),
# ... omitted for brevity ...
            expected_m,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `w1.size`, `self.moe_problem_size`, `_resize_cache`, `self.estimate_expected_m`, `fp8_m_grouped_gemm_nt_masked`, `DeepGemmQuantScaleFMT.from_oracle`. It writes or updates `expert_num_tokens`, `a1q`, `_`, `N`, `K`, `E`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `w1.size`, `self.moe_problem_size`, `_resize_cache`, `self.estimate_expected_m`, `fp8_m_grouped_gemm_nt_masked`, `DeepGemmQuantScaleFMT.from_oracle`。 它会写入或更新 `expert_num_tokens`, `a1q`, `_`, `N`, `K`, `E`。

### Lines 291-293 — method `BatchedDeepGemmExperts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.BatchedExperts
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 319-320 — method `BatchedDeepGemmExperts.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return False
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 322-329 — method `BatchedDeepGemmExperts.supports_packed_ue8m0_act_scales`
```python
    def supports_packed_ue8m0_act_scales(self) -> bool:
        """
        DeepGemm supports packed ue8m0 activation scales format in devices == sm100
        """
        return (
            is_deep_gemm_e8m0_used()
            and current_platform.is_device_capability_family(100)
        )
```
**EN:** This method defines `supports_packed_ue8m0_act_scales`. DeepGemm supports packed ue8m0 activation scales format in devices == sm100 Key calls include `is_deep_gemm_e8m0_used`, `current_platform.is_device_capability_family`.
**CN:** 该方法定义 `supports_packed_ue8m0_act_scales`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `is_deep_gemm_e8m0_used`, `current_platform.is_device_capability_family`。

### Lines 331-333 — method `BatchedDeepGemmExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        # Let PrepareAndFinalize::finalize() decide the impl.
        return TopKWeightAndReduceDelegate()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceDelegate`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceDelegate`。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `scales_shape_stride_dtype`, `_silu_mul_fp8_quant_deep_gemm`, `persistent_masked_m_silu_mul_quant`, `BatchedDeepGemmExperts` / [CN] 核心符号：`scales_shape_stride_dtype`, `_silu_mul_fp8_quant_deep_gemm`, `persistent_masked_m_silu_mul_quant`, `BatchedDeepGemmExperts`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`
- **Runtime traits**: platform-aware dispatch, custom C++/CUDA ops, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, custom C++/CUDA ops, Triton kernels, distributed collectives
