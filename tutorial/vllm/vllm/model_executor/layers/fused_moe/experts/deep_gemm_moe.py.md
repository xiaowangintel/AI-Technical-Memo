# deep_gemm_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/deep_gemm_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-46 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.deep_gemm_utils import (
    compute_aligned_M,
    deepgemm_moe_permute,
    deepgemm_unpermute_and_reduce,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.fused_moe.utils import _resize_cache
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    per_token_group_quant_fp8,
    per_token_group_quant_fp8_packed_for_deepgemm,
    silu_mul_per_token_group_quant_fp8_colmajor,
)
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    silu_mul_quant_fp8_packed_triton as fused_silu_mul_fp8_quant_packed,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8Dynamic128Sym,
    kFp8Static128BlockSym,
    kMxfp4Static,
)
from vllm.utils.deep_gemm import (
    DeepGemmQuantScaleFMT,
    get_mk_alignment_for_contiguous_layout,
    is_deep_gemm_supported,
    m_grouped_fp8_fp4_gemm_nt_contiguous,
    m_grouped_fp8_gemm_nt_contiguous,
)
from vllm.utils.import_utils import has_deep_gemm

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 49-51 — function `_valid_deep_gemm_shape`
```python
def _valid_deep_gemm_shape(M: int, N: int, K: int) -> bool:
    align = get_mk_alignment_for_contiguous_layout()[0]
    return align <= M and N % align == 0 and K % align == 0
```
**EN:** This function defines `_valid_deep_gemm_shape`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`. Key calls include `get_mk_alignment_for_contiguous_layout`. It writes or updates `align`.
**CN:** 该函数定义 `_valid_deep_gemm_shape`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`。 关键调用包括 `get_mk_alignment_for_contiguous_layout`。 它会写入或更新 `align`。

### Lines 54-118 — function `_valid_deep_gemm`
```python
def _valid_deep_gemm(
    hidden_states: torch.Tensor, w1: torch.Tensor, w2: torch.Tensor
) -> bool:
    """
    Check if the given problem size is supported by the DeepGemm grouped
    gemm kernel.  All of M, N, K and the quantization block_shape must be
    aligned by `dg.get_m_alignment_for_contiguous_layout()`.
    """
    if not has_deep_gemm():
        logger.debug_once("DeepGemm disabled: deep_gemm not available.")
        return False

    M = hidden_states.size(0)
    _, K, N = w2.size()

    align = get_mk_alignment_for_contiguous_layout()[0]

    if not _valid_deep_gemm_shape(M, N, K):
        logger.debug_once(
            "DeepGemm disabled due to unaligned problem size. "
            "M: %s, N: %s, K: %s. M should >= %s "
            "and N and K must be multiples of %s. "
            "This is not an error and we will fall back to triton.",
            M,
            N,
            K,
            align,
            align,
        )
        return False
    elif N <= 512:
        logger.debug_once(
            "DeepGemm disabled for N <= 512. M: %s, N: %s, K: %s. "
            "This means we will fallback to triton "
            "for this specific shape for further speed up.",
            M,
            N,
            K,
        )
        return False

    if w1.dtype != torch.float8_e4m3fn or w2.dtype != torch.float8_e4m3fn:
        logger.debug_once(
            "DeepGemm disabled: invalid weight dtype(s). w1.dtype: %s, w2.dtype: %s",
# ... omitted for brevity ...

    return True
```
**EN:** This function defines `_valid_deep_gemm`. Check if the given problem size is supported by the DeepGemm grouped gemm kernel. The main inputs are `hidden_states`, `w1`, `w2`. Key calls include `hidden_states.size`, `w2.size`, `has_deep_gemm`, `logger.debug_once`, `get_mk_alignment_for_contiguous_layout`, `_valid_deep_gemm_shape`. It writes or updates `M`, `_`, `K`, `N`, `align`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_valid_deep_gemm`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `w1`, `w2`。 关键调用包括 `hidden_states.size`, `w2.size`, `has_deep_gemm`, `logger.debug_once`, `get_mk_alignment_for_contiguous_layout`, `_valid_deep_gemm_shape`。 它会写入或更新 `M`, `_`, `K`, `N`, `align`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 121-329 — class `DeepGemmExperts`
```python
class DeepGemmExperts(mk.FusedMoEExpertsModular):
    """DeepGemm-based fused MoE expert implementation."""

    def __init__(self, moe_config: FusedMoEConfig, quant_config: FusedMoEQuantConfig):
        super().__init__(moe_config=moe_config, quant_config=quant_config)
        assert quant_config.block_shape == get_mk_alignment_for_contiguous_layout()
        assert quant_config.quant_dtype == torch.float8_e4m3fn
        assert not quant_config.per_act_token_quant
        assert not quant_config.per_out_ch_quant

        self.gemm1_clamp_limit = quant_config.gemm1_clamp_limit

    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
# ... omitted for brevity ...
            output=output,
        )
```
**EN:** This class defines `DeepGemmExperts`. It inherits from `mk.FusedMoEExpertsModular`. DeepGemm-based fused MoE expert implementation. Important methods include `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`. Key calls include `super.__init__`, `is_deep_gemm_supported`, `TopKWeightAndReduceNoOP`, `compute_aligned_M`, `self.adjust_N_for_activation`, `DeepGemmQuantScaleFMT.from_oracle`. It writes or updates `gemm1_clamp_limit`, `SUPPORTED_W_A`, `block_m`, `M_sum`, `activation_out_dim`, `workspace1`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `DeepGemmExperts`。 它继承自 `mk.FusedMoEExpertsModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`。 关键调用包括 `super.__init__`, `is_deep_gemm_supported`, `TopKWeightAndReduceNoOP`, `compute_aligned_M`, `self.adjust_N_for_activation`, `DeepGemmQuantScaleFMT.from_oracle`。 它会写入或更新 `gemm1_clamp_limit`, `SUPPORTED_W_A`, `block_m`, `M_sum`, `activation_out_dim`, `workspace1`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 124-131 — method `DeepGemmExperts.__init__`
```python
    def __init__(self, moe_config: FusedMoEConfig, quant_config: FusedMoEQuantConfig):
        super().__init__(moe_config=moe_config, quant_config=quant_config)
        assert quant_config.block_shape == get_mk_alignment_for_contiguous_layout()
        assert quant_config.quant_dtype == torch.float8_e4m3fn
        assert not quant_config.per_act_token_quant
        assert not quant_config.per_out_ch_quant

        self.gemm1_clamp_limit = quant_config.gemm1_clamp_limit
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `get_mk_alignment_for_contiguous_layout`, `super`. It writes or updates `gemm1_clamp_limit`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `get_mk_alignment_for_contiguous_layout`, `super`。 它会写入或更新 `gemm1_clamp_limit`。

### Lines 246-329 — method `DeepGemmExperts.apply`
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
        assert a1q_scale is not None
        assert a2_scale is None
        assert self.block_shape is not None
        assert self.w1_scale is not None
        assert self.w2_scale is not None

        a1q = hidden_states
        _, N, K = w1.size()

        local_num_experts = w1.size(0)
        if global_num_experts == -1:
            global_num_experts = local_num_experts

        assert w2.size(1) == K

        M_sum = compute_aligned_M(
            M=topk_ids.size(0),
            num_topk=topk_ids.size(1),
            local_num_experts=local_num_experts,
            alignment=get_mk_alignment_for_contiguous_layout()[0],
            expert_tokens_meta=expert_tokens_meta,
        )

        a1q_perm = _resize_cache(
            workspace13.view(dtype=torch.float8_e4m3fn), (M_sum, K)
        )
# ... omitted for brevity ...
            output=output,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `w1.size`, `compute_aligned_M`, `_resize_cache`, `deepgemm_moe_permute`, `m_grouped_fp8_gemm_nt_contiguous`, `self.adjust_N_for_activation`. It writes or updates `a1q`, `_`, `N`, `K`, `local_num_experts`, `M_sum`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `w1.size`, `compute_aligned_M`, `_resize_cache`, `deepgemm_moe_permute`, `m_grouped_fp8_gemm_nt_contiguous`, `self.adjust_N_for_activation`。 它会写入或更新 `a1q`, `_`, `N`, `K`, `local_num_experts`, `M_sum`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 133-135 — method `DeepGemmExperts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 167-168 — method `DeepGemmExperts.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return True
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 170-171 — method `DeepGemmExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 173-195 — method `DeepGemmExperts.workspace_shapes`
```python
    def workspace_shapes(
        self,
        M: int,
        N: int,
        K: int,
        topk: int,
        global_num_experts: int,
        local_num_experts: int,
        expert_tokens_meta: mk.ExpertTokensMetadata | None,
        activation: MoEActivation,
    ) -> tuple[tuple[int, ...], tuple[int, ...], tuple[int, ...]]:
        assert self.block_shape is not None
        block_m = self.block_shape[0]
        M_sum = compute_aligned_M(
            M, topk, local_num_experts, block_m, expert_tokens_meta
        )
        assert M_sum % block_m == 0

        activation_out_dim = self.adjust_N_for_activation(N, activation)
        workspace1 = (M_sum, max(activation_out_dim, K))
        workspace2 = (M_sum, max(N, K))
        output = (M, K)
        return (workspace1, workspace2, output)
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. Key calls include `compute_aligned_M`, `self.adjust_N_for_activation`, `max`. It writes or updates `block_m`, `M_sum`, `activation_out_dim`, `workspace1`, `workspace2`, `output`.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 关键调用包括 `compute_aligned_M`, `self.adjust_N_for_activation`, `max`。 它会写入或更新 `block_m`, `M_sum`, `activation_out_dim`, `workspace1`, `workspace2`, `output`。

### Lines 332-552 — class `DeepGemmFP4Experts`
```python
class DeepGemmFP4Experts(mk.FusedMoEExpertsModular):
    """DeepGemm-based fused MoE expert implementation for FP4 weights.

    Uses m_grouped_fp8_fp4_gemm_nt_contiguous with FP8 activations and
    MXFP4 (FP4 E2M1 packed as uint8) weights. Requires SM100+ (Blackwell).
    """

    # FP8 activation block size (hardcoded since mxfp4_w4a8 quant config
    # does not set a block_shape on the activation descriptor).
    _ACT_BLOCK_K = 128
    # FP4 weight block size
    _WEIGHT_BLOCK_K = 32

    def __init__(self, moe_config: FusedMoEConfig, quant_config: FusedMoEQuantConfig):
# ... omitted for brevity ...
            output=output,
        )
```
**EN:** This class defines `DeepGemmFP4Experts`. It inherits from `mk.FusedMoEExpertsModular`. DeepGemm-based fused MoE expert implementation for FP4 weights. Important methods include `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`. Key calls include `super.__init__`, `TopKWeightAndReduceNoOP`, `compute_aligned_M`, `self.adjust_N_for_activation`, `DeepGemmQuantScaleFMT.from_oracle`, `input.size`. It writes or updates `_ACT_BLOCK_K`, `_WEIGHT_BLOCK_K`, `gemm1_clamp_limit`, `SUPPORTED_W_A`, `block_m`, `M_sum`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `DeepGemmFP4Experts`。 它继承自 `mk.FusedMoEExpertsModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`。 关键调用包括 `super.__init__`, `TopKWeightAndReduceNoOP`, `compute_aligned_M`, `self.adjust_N_for_activation`, `DeepGemmQuantScaleFMT.from_oracle`, `input.size`。 它会写入或更新 `_ACT_BLOCK_K`, `_WEIGHT_BLOCK_K`, `gemm1_clamp_limit`, `SUPPORTED_W_A`, `block_m`, `M_sum`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 345-351 — method `DeepGemmFP4Experts.__init__`
```python
    def __init__(self, moe_config: FusedMoEConfig, quant_config: FusedMoEQuantConfig):
        super().__init__(moe_config=moe_config, quant_config=quant_config)
        assert quant_config.weight_quant_dtype == "mxfp4"
        assert not quant_config.per_act_token_quant
        assert not quant_config.per_out_ch_quant

        self.gemm1_clamp_limit = quant_config.gemm1_clamp_limit
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `super`. It writes or updates `gemm1_clamp_limit`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `gemm1_clamp_limit`。

### Lines 455-552 — method `DeepGemmFP4Experts.apply`
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
        assert a1q_scale is not None
        assert a2_scale is None
        assert self.w1_scale is not None
        assert self.w2_scale is not None

        a1q = hidden_states
        _, N, _ = w1.size()
        # K comes from activations (full hidden dim), not from w1 which is
        # packed FP4 (E, N, K//2).
        K = a1q.size(1)

        local_num_experts = w1.size(0)
        if global_num_experts == -1:
            global_num_experts = local_num_experts

        M_sum = compute_aligned_M(
            M=topk_ids.size(0),
            num_topk=topk_ids.size(1),
            local_num_experts=local_num_experts,
            alignment=get_mk_alignment_for_contiguous_layout()[0],
            expert_tokens_meta=expert_tokens_meta,
        )

        a1q_perm = _resize_cache(
            workspace13.view(dtype=torch.float8_e4m3fn), (M_sum, K)
        )
# ... omitted for brevity ...
            output=output,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `w1.size`, `a1q.size`, `compute_aligned_M`, `_resize_cache`, `deepgemm_moe_permute`, `m_grouped_fp8_fp4_gemm_nt_contiguous`. It writes or updates `a1q`, `_`, `N`, `K`, `local_num_experts`, `M_sum`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `w1.size`, `a1q.size`, `compute_aligned_M`, `_resize_cache`, `deepgemm_moe_permute`, `m_grouped_fp8_fp4_gemm_nt_contiguous`。 它会写入或更新 `a1q`, `_`, `N`, `K`, `local_num_experts`, `M_sum`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 353-355 — method `DeepGemmFP4Experts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `_valid_deep_gemm_shape`, `_valid_deep_gemm`, `DeepGemmExperts`, `DeepGemmFP4Experts` / [CN] 核心符号：`_valid_deep_gemm_shape`, `_valid_deep_gemm`, `DeepGemmExperts`, `DeepGemmFP4Experts`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.deep_gemm_utils`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.utils.deep_gemm` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.deep_gemm_utils`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.utils.deep_gemm`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
