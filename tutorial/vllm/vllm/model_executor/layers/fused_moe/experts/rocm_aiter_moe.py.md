# rocm_aiter_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/rocm_aiter_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-30 — imports and setup
```python
from enum import IntEnum
from functools import lru_cache

import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm._aiter_ops import rocm_aiter_ops
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FUSED_MOE_UNQUANTIZED_CONFIG,
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.fused_moe.utils import disable_inplace
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8Dynamic128Sym,
    kFp8DynamicTensorSym,
    kFp8DynamicTokenSym,
    kFp8Static128BlockSym,
    kFp8StaticChannelSym,
    kFp8StaticTensorSym,
    kMxfp4Static,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 33-46 — class `QuantMethod`
```python
class QuantMethod(IntEnum):
    # This allows interfacing with AITER QuantType Enum
    # without importing the QuantType from AITER globally.

    # Note that these quantization methods are
    # supported in AITER package. However,
    # not all are used in this module.

    NO = 0  # a16w16
    PER_TENSOR = 1  # w8a8 (pre_Tensor)
    PER_TOKEN = 2  # w8a8/w8a4 (per_Token)
    BLOCK_1X32 = 3  # fp4x2
    BLOCK_1X128 = 4  # block quantized w8a8 (per_1x128)
    BLOCK_128x128 = 5  # block quantized w8a8 (per_128x128)
```
**EN:** This class defines `QuantMethod`. It inherits from `IntEnum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `NO`, `PER_TENSOR`, `PER_TOKEN`, `BLOCK_1X32`, `BLOCK_1X128`, `BLOCK_128x128`.
**CN:** 该类定义了 `QuantMethod`。 它继承自 `IntEnum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `NO`, `PER_TENSOR`, `PER_TOKEN`, `BLOCK_1X32`, `BLOCK_1X128`, `BLOCK_128x128`。

### Lines 49-53 — class `ActivationMethod`
```python
class ActivationMethod(IntEnum):
    # This allows interfacing with AITER ActivationType enum
    # without importing the ActivationType enum from AITER globally.
    SILU = 0
    GELU = 1
```
**EN:** This class defines `ActivationMethod`. It inherits from `IntEnum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `SILU`, `GELU`.
**CN:** 该类定义了 `ActivationMethod`。 它继承自 `IntEnum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `SILU`, `GELU`。

### Lines 59-109 — function `init_aiter_topK_meta_data`
```python
@lru_cache(maxsize=1)
def init_aiter_topK_meta_data(
    n_routed_experts: int,
    n_shared_experts: int,
    top_k: int,
    tp_rank: int,
    tp_size: int,
    shared_experts_score: float = 1.0,
    max_num_tokens: int = 32768,
    is_EP: bool = False,
):
    global aiter_topK_meta_data
    fake_expertid = n_routed_experts + n_shared_experts

    # all layers reuse same buffer
    # This extra element when EP is enabled is used as a sentinel
    # to mask out shared expert processing for tokens not owned by
    # the current EP rank. This is necessary to avoid double-processing
    # of shared experts.
    total_topk_ids = torch.empty(
        (max_num_tokens, top_k + n_shared_experts + is_EP),
        dtype=torch.int32,
        device="cuda",
    )
    ns_topk_ids, s_topk_ids = total_topk_ids.split(
        [top_k, n_shared_experts + is_EP], dim=1
    )
    shared_expert_ids = [n_routed_experts + i for i in range(n_shared_experts + is_EP)]
    if is_EP:
        s_topk_ids_list = [
            [fake_expertid] * (n_shared_experts + is_EP)
        ] * max_num_tokens
        for i in range(tp_rank, max_num_tokens, tp_size):
            s_topk_ids_list[i] = shared_expert_ids
    else:
        s_topk_ids_list = [
            list(range(n_routed_experts, fake_expertid))
        ] * max_num_tokens
    s_topk_ids[:] = torch.tensor(s_topk_ids_list, dtype=torch.int32, device="cuda")

    total_topk_weights = torch.empty(
        (max_num_tokens, top_k + n_shared_experts + is_EP),
        dtype=torch.float32,
        device="cuda",
# ... omitted for brevity ...
    assert aiter_topK_meta_data is None, "AITER topK meta data is already initialized"
    aiter_topK_meta_data = (total_topk_weights, total_topk_ids)
```
**EN:** This function defines `init_aiter_topK_meta_data`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `n_routed_experts`, `n_shared_experts`, `top_k`, `tp_rank`, `tp_size`, `shared_experts_score`. Key calls include `lru_cache`, `torch.empty`, `total_topk_ids.split`, `torch.tensor`, `total_topk_weights.split`, `s_topk_weights.fill_`. It writes or updates `fake_expertid`, `total_topk_ids`, `ns_topk_ids`, `s_topk_ids`, `shared_expert_ids`, `total_topk_weights`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `init_aiter_topK_meta_data`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `n_routed_experts`, `n_shared_experts`, `top_k`, `tp_rank`, `tp_size`, `shared_experts_score`。 关键调用包括 `lru_cache`, `torch.empty`, `total_topk_ids.split`, `torch.tensor`, `total_topk_weights.split`, `s_topk_weights.fill_`。 它会写入或更新 `fake_expertid`, `total_topk_ids`, `ns_topk_ids`, `s_topk_ids`, `shared_expert_ids`, `total_topk_weights`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 112-158 — function `inject_shared_expert_weights`
```python
def inject_shared_expert_weights(
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    topk: int,
    num_fused_shared_experts: int,
    shared_expert_weights: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Merge routed topk results with the shared expert buffer and inject
    dynamic per-token shared expert gate values for AITER fusion.

    For routers that already return the combined buffer (e.g. GroupedTopKRouter
    via rocm_aiter_grouped_topk), only the dynamic weight injection is needed.
    For routers that return only routed slots (e.g. FusedTopKRouter), this also
    copies the routed results into the pre-allocated combined buffer.
    """
    if num_fused_shared_experts == 0:
        return topk_weights, topk_ids

    assert aiter_topK_meta_data is not None, (
        "aiter_topK_meta_data is not initialized but "
        "num_fused_shared_experts > 0. Ensure init_aiter_topK_meta_data "
        "is called before routing."
    )

    total_topk_weights, total_topk_ids = aiter_topK_meta_data
    token = topk_weights.shape[0]

    assert total_topk_weights.shape[0] >= token, (
        f"AITER topK meta data supports {total_topk_weights.shape[0]} "
        f"tokens, but got {token} tokens."
    )

    total_topk_weights_slice = total_topk_weights[:token]
    total_topk_ids_slice = total_topk_ids[:token]

    if topk_weights.shape[1] == topk:
        total_topk_weights_slice[:, :topk] = topk_weights
        total_topk_ids_slice[:, :topk] = topk_ids
        topk_weights = total_topk_weights_slice
        topk_ids = total_topk_ids_slice

    if shared_expert_weights is not None:
        topk_weights[:, topk : topk + num_fused_shared_experts] = shared_expert_weights[
            :token
        ]

    return topk_weights, topk_ids
```
**EN:** This function defines `inject_shared_expert_weights`. Merge routed topk results with the shared expert buffer and inject dynamic per-token shared expert gate values for AITER fusion. The main inputs are `topk_weights`, `topk_ids`, `topk`, `num_fused_shared_experts`, `shared_expert_weights`. It writes or updates `total_topk_weights`, `total_topk_ids`, `token`, `total_topk_weights_slice`, `total_topk_ids_slice`, `topk_weights`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `inject_shared_expert_weights`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `topk_weights`, `topk_ids`, `topk`, `num_fused_shared_experts`, `shared_expert_weights`。 它会写入或更新 `total_topk_weights`, `total_topk_ids`, `token`, `total_topk_weights_slice`, `total_topk_ids_slice`, `topk_weights`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 161-231 — function `rocm_aiter_grouped_topk`
```python
def rocm_aiter_grouped_topk(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: int = 0,
    topk_group: int = 0,
    scoring_func: str = "softmax",
    routed_scaling_factor: float = 1.0,
    e_score_correction_bias: torch.Tensor | None = None,
    num_fused_shared_experts: int = 0,
) -> tuple[torch.Tensor, torch.Tensor]:
    token = hidden_states.shape[0]
    device = hidden_states.device
    if (
        rocm_aiter_ops.is_fusion_moe_shared_experts_enabled()
        and num_fused_shared_experts > 0
    ):
        assert aiter_topK_meta_data is not None, (
            "AITER topK meta data is not initialized. "
            "Please ensure that init_aiter_topK_meta_data "
            "is called before this function."
        )
        total_topk_weights, total_topk_ids = aiter_topK_meta_data
        assert total_topk_weights.shape[0] >= token, (
            f"AITER topK meta data support {total_topk_weights.shape[0]} "
            f"tokens which is determined by max_num_batched_tokens, "
            f"but got {token} tokens now."
        )
        total_topk_weights = total_topk_weights[:token]
        total_topk_ids = total_topk_ids[:token]
        topk_weights, _ = total_topk_weights.split(
            [topk, total_topk_weights.shape[1] - topk], dim=1
        )
        topk_ids, _ = total_topk_ids.split(
            [topk, total_topk_ids.shape[1] - topk], dim=1
        )
    else:
        topk_ids = torch.empty((token, topk), dtype=torch.int32, device=device)
        topk_weights = torch.empty((token, topk), dtype=torch.float32, device=device)

    if e_score_correction_bias is not None:
        rocm_aiter_ops.biased_grouped_topk(
            gating_output,
# ... omitted for brevity ...
        return total_topk_weights, total_topk_ids
    return topk_weights, topk_ids
```
**EN:** This function defines `rocm_aiter_grouped_topk`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `hidden_states`, `gating_output`, `topk`, `renormalize`, `num_expert_group`, `topk_group`. Key calls include `rocm_aiter_ops.is_fusion_moe_shared_experts_enabled`, `total_topk_weights.split`, `total_topk_ids.split`, `torch.empty`, `rocm_aiter_ops.biased_grouped_topk`, `rocm_aiter_ops.grouped_topk`. It writes or updates `token`, `device`, `total_topk_weights`, `total_topk_ids`, `topk_weights`, `_`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `rocm_aiter_grouped_topk`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `hidden_states`, `gating_output`, `topk`, `renormalize`, `num_expert_group`, `topk_group`。 关键调用包括 `rocm_aiter_ops.is_fusion_moe_shared_experts_enabled`, `total_topk_weights.split`, `total_topk_ids.split`, `torch.empty`, `rocm_aiter_ops.biased_grouped_topk`, `rocm_aiter_ops.grouped_topk`。 它会写入或更新 `token`, `device`, `total_topk_weights`, `total_topk_ids`, `topk_weights`, `_`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 234-363 — function `rocm_aiter_fused_experts`
```python
def rocm_aiter_fused_experts(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    moe_config: FusedMoEConfig,
    activation: MoEActivation = MoEActivation.SILU,
    apply_router_weight_on_input: bool = False,
    expert_map: torch.Tensor | None = None,
    quant_config: FusedMoEQuantConfig | None = None,
    a1q_scale: torch.Tensor | None = None,
    num_local_tokens: torch.Tensor | None = None,
    output_dtype: torch.dtype | None = None,
) -> torch.Tensor:
    """ROCm AITER fused MoE expert computation."""
    if quant_config is None:
        quant_config = FUSED_MOE_UNQUANTIZED_CONFIG

    if activation == MoEActivation.SILU:
        activation_method = ActivationMethod.SILU
    elif activation == MoEActivation.GELU:
        activation_method = ActivationMethod.GELU
    elif activation == MoEActivation.SWIGLUOAI:
        activation_method = rocm_aiter_ops.get_aiter_activation_type("swiglu")
    else:
        raise ValueError(f"Unsupported activation: {activation}")

    # All AITER Fused MoE kernels are expecting the following datatypes
    topk_weights = topk_weights.to(torch.float32)
    topk_ids = topk_ids.to(torch.int32)

    expert_mask = expert_map if expert_map is not None else None

    # w8a8 per-channel quantization
    if (
        quant_config.per_act_token_quant
        and apply_router_weight_on_input
        and quant_config.use_fp8_w8a8
    ):
        # AITER tkw1 kernel for FP8 models with `apply_router_weight_on_input`
        # This applies topk_weights on the GEMM output of the first FC layer
        #  rather than the second FC.
        assert topk_weights.dim() == 2, (
# ... omitted for brevity ...
            bias2=quant_config.w2_bias if quant_config.use_mxfp4_w4a16 else None,
        )
```
**EN:** This function defines `rocm_aiter_fused_experts`. ROCm AITER fused MoE expert computation. The main inputs are `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`, `moe_config`. Key calls include `topk_weights.to`, `topk_ids.to`, `rocm_aiter_ops.asm_moe_tkw1`, `rocm_aiter_ops.fused_moe`, `topk_weights.dim`, `rocm_aiter_ops.get_aiter_activation_type`. It writes or updates `topk_weights`, `topk_ids`, `expert_mask`, `quant_config`, `activation_method`, `quant_method`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `rocm_aiter_fused_experts`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`, `moe_config`。 关键调用包括 `topk_weights.to`, `topk_ids.to`, `rocm_aiter_ops.asm_moe_tkw1`, `rocm_aiter_ops.fused_moe`, `topk_weights.dim`, `rocm_aiter_ops.get_aiter_activation_type`。 它会写入或更新 `topk_weights`, `topk_ids`, `expert_mask`, `quant_config`, `activation_method`, `quant_method`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 366-503 — class `AiterExperts`
```python
class AiterExperts(mk.FusedMoEExpertsModular):
    @property
    def expects_unquantized_inputs(self) -> bool:
        # When paired with MoRI, the prepare/finalize handles FP8
        # quantization during dispatch to reduce network traffic,
        # so we should not defer input quantization.
        # Otherwise, AITER fused MoE kernels handle input quantization
        # internally via a single fused kernel.
        return not self.moe_config.use_mori_kernels

    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard

# ... omitted for brevity ...
        else:
            output.copy_(result)
```
**EN:** This class defines `AiterExperts`. It inherits from `mk.FusedMoEExpertsModular`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `apply`, `expects_unquantized_inputs`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`. Key calls include `rocm_aiter_ops.is_fused_moe_enabled`, `TopKWeightAndReduceNoOP`, `rocm_aiter_fused_experts`, `output.is_contiguous`, `result.is_contiguous`, `disable_inplace`. It writes or updates `SUPPORTED_W_A`, `workspace1`, `workspace2`, `output`, `result`, `num_local_tokens`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `AiterExperts`。 它继承自 `mk.FusedMoEExpertsModular`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `apply`, `expects_unquantized_inputs`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`。 关键调用包括 `rocm_aiter_ops.is_fused_moe_enabled`, `TopKWeightAndReduceNoOP`, `rocm_aiter_fused_experts`, `output.is_contiguous`, `result.is_contiguous`, `disable_inplace`。 它会写入或更新 `SUPPORTED_W_A`, `workspace1`, `workspace2`, `output`, `result`, `num_local_tokens`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 449-503 — method `AiterExperts.apply`
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
        # TODO(rob): rocm_aiter_fused_experts uses self.quant_config's
        # a_scales for static quantization. Update this to fit better
        # with the interface once all quant integrations are complete.

        if expert_tokens_meta is not None:
            num_local_tokens = expert_tokens_meta.expert_num_tokens
        else:
            num_local_tokens = None

        result = rocm_aiter_fused_experts(
            hidden_states=hidden_states,
            w1=w1,
            w2=w2,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            activation=activation,
            apply_router_weight_on_input=apply_router_weight_on_input,
            expert_map=expert_map,
            quant_config=self.quant_config,
            moe_config=self.moe_config,
            a1q_scale=a1q_scale,
            num_local_tokens=num_local_tokens,
            output_dtype=output.dtype,
        )
        # avoid redundant copy when output is a view of the result
        if (
# ... omitted for brevity ...
        else:
            output.copy_(result)
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `rocm_aiter_fused_experts`, `output.is_contiguous`, `result.is_contiguous`, `disable_inplace`, `output.set_`, `output.copy_`. It writes or updates `result`, `num_local_tokens`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `rocm_aiter_fused_experts`, `output.is_contiguous`, `result.is_contiguous`, `disable_inplace`, `output.set_`, `output.copy_`。 它会写入或更新 `result`, `num_local_tokens`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 367-374 — method `AiterExperts.expects_unquantized_inputs`
```python
    @property
    def expects_unquantized_inputs(self) -> bool:
        # When paired with MoRI, the prepare/finalize handles FP8
        # quantization during dispatch to reduce network traffic,
        # so we should not defer input quantization.
        # Otherwise, AITER fused MoE kernels handle input quantization
        # internally via a single fused kernel.
        return not self.moe_config.use_mori_kernels
```
**EN:** This method defines `expects_unquantized_inputs`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `expects_unquantized_inputs`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 376-378 — method `AiterExperts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 426-427 — method `AiterExperts.supports_expert_map`
```python
    def supports_expert_map(self):
        return True
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 429-430 — method `AiterExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 432-447 — method `AiterExperts.workspace_shapes`
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
        # Workspaces are managed internally by AITER.
        workspace1 = (0,)
        workspace2 = (0,)
        output = (M, K)
        return (workspace1, workspace2, output)
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. It writes or updates `workspace1`, `workspace2`, `output`.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 它会写入或更新 `workspace1`, `workspace2`, `output`。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `QuantMethod`, `ActivationMethod`, `init_aiter_topK_meta_data`, `inject_shared_expert_weights` / [CN] 核心符号：`QuantMethod`, `ActivationMethod`, `init_aiter_topK_meta_data`, `inject_shared_expert_weights`

## Dependencies / 依赖关系
- **External**: `enum`, `functools`, `torch` / **外部依赖**: `enum`, `functools`, `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm._aiter_ops`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm._aiter_ops`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
