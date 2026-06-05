# marlin_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/marlin_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fused MoE utilities for GPTQ. / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-54 — imports and setup
```python
from collections.abc import Callable

import torch

import vllm._custom_ops as ops
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.model_executor.layers.fused_moe.activation import (
    MoEActivation,
    apply_moe_activation,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin import (
    LoRAExpertsMixin,
)
from vllm.model_executor.layers.fused_moe.moe_align_block_size import (
    batched_moe_align_block_size,
    moe_align_block_size,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceDelegate,
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.fused_moe.utils import (
    _resize_cache,
    disable_inplace,
    swiglu_limit_func,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    get_marlin_input_dtype,
    marlin_make_workspace_new,
    marlin_moe_intermediate_size,
    marlin_quant_input,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8Static128BlockSym,
    kFp8StaticChannelSym,
    kFp8StaticTensorSym,
    kInt4Static,
    kInt8Static,
    kMxfp4Static,
    kMxfp8Static,
    kNvfp4Static,
)
from vllm.platforms import current_platform
from vllm.scalar_type import ScalarType, scalar_types
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 57-222 — function `_fused_marlin_moe`
```python
def _fused_marlin_moe(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    bias1: torch.Tensor | None,
    bias2: torch.Tensor | None,
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    topk_weights: torch.Tensor,
    num_topk: int,
    quant_type: ScalarType,
    apply_router_weight_on_input: bool,
    expert_map: torch.Tensor | None,
    block_size_m: int,
    sorted_token_ids: torch.Tensor,
    expert_ids: torch.Tensor,
    num_tokens_post_padded: torch.Tensor,
    activation: MoEActivation = MoEActivation.SILU,
    activation_func: Callable[
        [MoEActivation, torch.Tensor, torch.Tensor], None
    ] = apply_moe_activation,
    input_global_scale1: torch.Tensor | None = None,
    input_global_scale2: torch.Tensor | None = None,
    global_scale1: torch.Tensor | None = None,
    global_scale2: torch.Tensor | None = None,
    g_idx1: torch.Tensor | None = None,
    g_idx2: torch.Tensor | None = None,
    sort_indices1: torch.Tensor | None = None,
    sort_indices2: torch.Tensor | None = None,
    w1_zeros: torch.Tensor | None = None,
    w2_zeros: torch.Tensor | None = None,
    workspace: torch.Tensor | None = None,
    intermediate_cache13: torch.Tensor | None = None,
    intermediate_cache2: torch.Tensor | None = None,
    output: torch.Tensor | None = None,
    input_dtype: torch.dtype | None = None,
    is_k_full: bool = True,
    clamp_limit: float | None = None,
) -> torch.Tensor:
    assert hidden_states.ndim == 2
    M, K = hidden_states.size()
    N = marlin_moe_intermediate_size(w1, w2)
    w13_num_shards = 2 if activation.is_gated else 1
    if workspace is None:
# ... omitted for brevity ...

    return output
```
**EN:** This function defines `_fused_marlin_moe`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_states`, `w1`, `w2`, `bias1`, `bias2`, `w1_scale`. Key calls include `hidden_states.size`, `marlin_moe_intermediate_size`, `_resize_cache`, `ops.moe_wna16_marlin_gemm`, `marlin_make_workspace_new`, `torch.empty`. It writes or updates `M`, `K`, `N`, `w13_num_shards`, `intermediate_cache1`, `intermediate_cache3`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_fused_marlin_moe`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `bias1`, `bias2`, `w1_scale`。 关键调用包括 `hidden_states.size`, `marlin_moe_intermediate_size`, `_resize_cache`, `ops.moe_wna16_marlin_gemm`, `marlin_make_workspace_new`, `torch.empty`。 它会写入或更新 `M`, `K`, `N`, `w13_num_shards`, `intermediate_cache1`, `intermediate_cache3`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 225-387 — function `fused_marlin_moe`
```python
def fused_marlin_moe(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    bias1: torch.Tensor | None,
    bias2: torch.Tensor | None,
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    quant_type_id: int,
    apply_router_weight_on_input: bool = False,
    global_num_experts: int = -1,
    activation: MoEActivation = MoEActivation.SILU,
    activation_func: Callable[
        [MoEActivation, torch.Tensor, torch.Tensor], None
    ] = apply_moe_activation,
    moe_sum: Callable[[torch.Tensor, torch.Tensor], None] | None = None,
    expert_map: torch.Tensor | None = None,
    input_global_scale1: torch.Tensor | None = None,
    input_global_scale2: torch.Tensor | None = None,
    global_scale1: torch.Tensor | None = None,
    global_scale2: torch.Tensor | None = None,
    g_idx1: torch.Tensor | None = None,
    g_idx2: torch.Tensor | None = None,
    sort_indices1: torch.Tensor | None = None,
    sort_indices2: torch.Tensor | None = None,
    w1_zeros: torch.Tensor | None = None,
    w2_zeros: torch.Tensor | None = None,
    workspace: torch.Tensor | None = None,
    intermediate_cache13: torch.Tensor | None = None,
    intermediate_cache2: torch.Tensor | None = None,
    is_k_full: bool = True,
    output: torch.Tensor | None = None,
    input_dtype: torch.dtype | None = None,
    inplace: bool = False,
    clamp_limit: float | None = None,
) -> torch.Tensor:
    """
    This function computes a Mixture of Experts (MoE) layer using two sets of
    weights, w1 and w2, and top-k gating mechanism.

    Parameters:
    - hidden_states (torch.Tensor): The input tensor to the MoE layer.
# ... omitted for brevity ...
    else:
        return moe_sum(moe_output, output)
```
**EN:** This function defines `fused_marlin_moe`. This function computes a Mixture of Experts (MoE) layer using two sets of weights, w1 and w2, and top-k gating mechanism. The main inputs are `hidden_states`, `w1`, `w2`, `bias1`, `bias2`, `w1_scale`. Key calls include `ScalarType.from_id`, `hidden_states.size`, `w1.size`, `topk_ids.size`, `hidden_states.is_contiguous`, `w1.is_contiguous`. It writes or updates `quant_type`, `bit4_scalar_types`, `num_bits`, `M`, `K`, `E`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `fused_marlin_moe`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `bias1`, `bias2`, `w1_scale`。 关键调用包括 `ScalarType.from_id`, `hidden_states.size`, `w1.size`, `topk_ids.size`, `hidden_states.is_contiguous`, `w1.is_contiguous`。 它会写入或更新 `quant_type`, `bit4_scalar_types`, `num_bits`, `M`, `K`, `E`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 390-546 — function `batched_fused_marlin_moe`
```python
def batched_fused_marlin_moe(
    hidden_states: torch.Tensor,
    expert_num_tokens: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    bias1: torch.Tensor | None,
    bias2: torch.Tensor | None,
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    quant_type_id: int,
    apply_router_weight_on_input: bool = False,
    global_num_experts: int = -1,
    activation: MoEActivation = MoEActivation.SILU,
    expert_map: torch.Tensor | None = None,
    global_scale1: torch.Tensor | None = None,
    global_scale2: torch.Tensor | None = None,
    g_idx1: torch.Tensor | None = None,
    g_idx2: torch.Tensor | None = None,
    sort_indices1: torch.Tensor | None = None,
    sort_indices2: torch.Tensor | None = None,
    w1_zeros: torch.Tensor | None = None,
    w2_zeros: torch.Tensor | None = None,
    workspace: torch.Tensor | None = None,
    intermediate_cache13: torch.Tensor | None = None,
    intermediate_cache2: torch.Tensor | None = None,
    is_k_full: bool = True,
    output: torch.Tensor | None = None,
    inplace: bool = False,
    clamp_limit: float | None = None,
) -> torch.Tensor:
    """
    This function massages the inputs so the batched hidden_states can be
    presented as a 2D contiguous tensor that could be used with
    _fused_marlin_moe.

    Note that both batched_fused_marlin_moe and fused_marlin_moe ultimately
    use `ops.moe_wna16_marlin_gemm` for the gemm operation and
    `ops.moe_mna16_marlin_gemm` supports only 2D contiguous hidden_states.
    Note that the moe_align_block_size function indicates,
        - What rows of the A matrix (hidden_states) to access during the
        matmul, via sorted_ids output.
        - What expert_id to use for each block matmul, via expert_ids output.

    In the batched version, the tokens are already grouped/batched by experts
# ... omitted for brevity ...

    return output
```
**EN:** This function defines `batched_fused_marlin_moe`. This function massages the inputs so the batched hidden_states can be presented as a 2D contiguous tensor that could be used with _fused_marlin_moe. The main inputs are `hidden_states`, `expert_num_tokens`, `w1`, `w2`, `bias1`, `bias2`. Key calls include `ScalarType.from_id`, `hidden_states.size`, `hidden_states.view.size`, `w1.size`, `hidden_states.is_contiguous`, `w1.is_contiguous`. It writes or updates `quant_type`, `bit4_scalar_types`, `num_bits`, `B`, `BATCH_TOKENS_MAX`, `K`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `batched_fused_marlin_moe`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `expert_num_tokens`, `w1`, `w2`, `bias1`, `bias2`。 关键调用包括 `ScalarType.from_id`, `hidden_states.size`, `hidden_states.view.size`, `w1.size`, `hidden_states.is_contiguous`, `w1.is_contiguous`。 它会写入或更新 `quant_type`, `bit4_scalar_types`, `num_bits`, `B`, `BATCH_TOKENS_MAX`, `K`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 549-674 — class `MarlinExpertsBase`
```python
class MarlinExpertsBase(mk.FusedMoEExpertsModular):
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
        w13_g_idx: torch.Tensor | None = None,
        w2_g_idx: torch.Tensor | None = None,
        w13_g_idx_sort_indices: torch.Tensor | None = None,
        w2_g_idx_sort_indices: torch.Tensor | None = None,
        is_k_full: bool = True,
    ):
        # TODO (varun) : Enable activation quantization
# ... omitted for brevity ...

        return E, M, N, K, topk
```
**EN:** This class defines `MarlinExpertsBase`. It inherits from `mk.FusedMoEExpertsModular`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `quant_type_id`, `moe_problem_size`. Key calls include `get_marlin_input_dtype`, `super.__init__`, `w1.size`, `a1.size`, `marlin_moe_intermediate_size`, `topk_ids.size`. It writes or updates `w13_g_idx`, `w2_g_idx`, `w13_g_idx_sort_indices`, `w2_g_idx_sort_indices`, `is_k_full`, `input_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `MarlinExpertsBase`。 它继承自 `mk.FusedMoEExpertsModular`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `quant_type_id`, `moe_problem_size`。 关键调用包括 `get_marlin_input_dtype`, `super.__init__`, `w1.size`, `a1.size`, `marlin_moe_intermediate_size`, `topk_ids.size`。 它会写入或更新 `w13_g_idx`, `w2_g_idx`, `w13_g_idx_sort_indices`, `w2_g_idx_sort_indices`, `is_k_full`, `input_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 550-582 — method `MarlinExpertsBase.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
        w13_g_idx: torch.Tensor | None = None,
        w2_g_idx: torch.Tensor | None = None,
        w13_g_idx_sort_indices: torch.Tensor | None = None,
        w2_g_idx_sort_indices: torch.Tensor | None = None,
        is_k_full: bool = True,
    ):
        # TODO (varun) : Enable activation quantization
        assert (
            quant_config.use_mxfp4_w4a16
            or quant_config.use_nvfp4_w4a16
            or quant_config.use_int4_w4a16
            or quant_config.use_fp8_w8a16
        ), "Supports only {mxfp,nvfp,int}4_w4a16 or fp8_w8a16"
        self.w13_g_idx = w13_g_idx
        self.w2_g_idx = w2_g_idx
        self.w13_g_idx_sort_indices = w13_g_idx_sort_indices
        self.w2_g_idx_sort_indices = w2_g_idx_sort_indices
        self.is_k_full = is_k_full
        self.input_dtype = get_marlin_input_dtype()
        self.gemm1_clamp_limit = quant_config.gemm1_clamp_limit

        super().__init__(
            moe_config=moe_config,
            quant_config=quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=num_dispatchers,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`, `w13_g_idx`, `w2_g_idx`. Key calls include `get_marlin_input_dtype`, `super.__init__`, `super`. It writes or updates `w13_g_idx`, `w2_g_idx`, `w13_g_idx_sort_indices`, `w2_g_idx_sort_indices`, `is_k_full`, `input_dtype`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`, `w13_g_idx`, `w2_g_idx`。 关键调用包括 `get_marlin_input_dtype`, `super.__init__`, `super`。 它会写入或更新 `w13_g_idx`, `w2_g_idx`, `w13_g_idx_sort_indices`, `w2_g_idx_sort_indices`, `is_k_full`, `input_dtype`。

### Lines 635-647 — method `MarlinExpertsBase.quant_type_id`
```python
    @property
    def quant_type_id(self) -> int:
        if self.quant_config.use_int4_w4a16:
            return scalar_types.uint4b8.id
        elif self.quant_config.use_mxfp4_w4a16 or self.quant_config.use_nvfp4_w4a16:
            return scalar_types.float4_e2m1f.id
        elif (
            self.quant_config.use_fp8_w8a16
            and current_platform.fp8_dtype() == torch.float8_e4m3fn
        ):
            return scalar_types.float8_e4m3fn.id
        else:
            raise NotImplementedError("Unsupported quantization type.")
```
**EN:** This method defines `quant_type_id`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `NotImplementedError`, `current_platform.fp8_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `quant_type_id`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `NotImplementedError`, `current_platform.fp8_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 649-674 — method `MarlinExpertsBase.moe_problem_size`
```python
    def moe_problem_size(
        self,
        a1: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
        topk_ids: torch.Tensor,
    ) -> tuple[int, int, int, int, int]:
        assert w1.dim() == 3 and w2.dim() == 3

        E = w1.size(0)
        K = a1.size(-1)
        N = marlin_moe_intermediate_size(w1, w2)

        if a1.dim() == 2:
            # Make sure we are using the correct a1 (pre-permute).
            assert topk_ids.size(0) == a1.size(0), f"{topk_ids.size(0)} != {a1.size(0)}"
            M = a1.size(0)
        else:
            assert a1.dim() == 3
            assert a1.size(0) == E, f"{a1.size(0)} == {E}"
            M = a1.size(1)  # This is max_num_tokens

        assert topk_ids.dim() == 2
        topk = topk_ids.size(1)

        return E, M, N, K, topk
```
**EN:** This method defines `moe_problem_size`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a1`, `w1`, `w2`, `topk_ids`. Key calls include `w1.size`, `a1.size`, `marlin_moe_intermediate_size`, `topk_ids.size`, `a1.dim`, `topk_ids.dim`. It writes or updates `E`, `K`, `N`, `topk`, `M`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `moe_problem_size`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a1`, `w1`, `w2`, `topk_ids`。 关键调用包括 `w1.size`, `a1.size`, `marlin_moe_intermediate_size`, `topk_ids.size`, `a1.dim`, `topk_ids.dim`。 它会写入或更新 `E`, `K`, `N`, `topk`, `M`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 677-876 — class `MarlinExperts`
```python
class MarlinExperts(LoRAExpertsMixin, MarlinExpertsBase):
    """Marlin-based fused MoE expert implementation."""

    def supports_expert_map(self) -> bool:
        return True

    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()

    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard

    def workspace_shapes(
# ... omitted for brevity ...
    def moe_sum(self, input: torch.Tensor, output: torch.Tensor) -> None:
        ops.moe_sum(input, output)
```
**EN:** This class defines `MarlinExperts`. It inherits from `LoRAExpertsMixin`, `MarlinExpertsBase`. Marlin-based fused MoE expert implementation. Important methods include `apply`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `activation_format`, `workspace_shapes`, `moe_sum`. Key calls include `TopKWeightAndReduceNoOP`, `hidden_states.size`, `topk_ids.size`, `fused_marlin_moe`, `ops.moe_sum`, `max`. It writes or updates `workspace1`, `workspace2`, `output`, `ctx`, `M`, `top_k_num`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `MarlinExperts`。 它继承自 `LoRAExpertsMixin`, `MarlinExpertsBase`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `apply`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `activation_format`, `workspace_shapes`, `moe_sum`。 关键调用包括 `TopKWeightAndReduceNoOP`, `hidden_states.size`, `topk_ids.size`, `fused_marlin_moe`, `ops.moe_sum`, `max`。 它会写入或更新 `workspace1`, `workspace2`, `output`, `ctx`, `M`, `top_k_num`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 722-873 — method `MarlinExperts.apply`
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
        assert self.w1_scale is not None
        assert self.w2_scale is not None

        ctx = self._lora_context
        if ctx is None:
            fused_marlin_moe(
                hidden_states=hidden_states,
                w1=w1,
                w2=w2,
                bias1=self.w1_bias,
                bias2=self.w2_bias,
                w1_scale=self.w1_scale,
                w2_scale=self.w2_scale,
                topk_weights=topk_weights,
                topk_ids=topk_ids,
                global_scale1=self.g1_alphas,
                global_scale2=self.g2_alphas,
                quant_type_id=self.quant_type_id,
                apply_router_weight_on_input=apply_router_weight_on_input,
                global_num_experts=global_num_experts,
                activation=activation,
                activation_func=self.activation,
                moe_sum=self.moe_sum,
                expert_map=expert_map,
                output=output,
                # Workspaces are swapped in workspace_shapes() to account for proper
# ... omitted for brevity ...
            clamp_limit=self.gemm1_clamp_limit,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `hidden_states.size`, `topk_ids.size`, `fused_marlin_moe`, `self.apply_w13_lora`, `lora_state.update`, `self.activation`. It writes or updates `ctx`, `M`, `top_k_num`, `lora_state`, `sorted_token_ids_lora`, `expert_ids_lora`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `hidden_states.size`, `topk_ids.size`, `fused_marlin_moe`, `self.apply_w13_lora`, `lora_state.update`, `self.activation`。 它会写入或更新 `ctx`, `M`, `top_k_num`, `lora_state`, `sorted_token_ids_lora`, `expert_ids_lora`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 680-681 — method `MarlinExperts.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return True
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 683-684 — method `MarlinExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 686-688 — method `MarlinExperts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 690-720 — method `MarlinExperts.workspace_shapes`
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
        # Modular Kernel provisions output buffer from workspace1. However in
        # the fused_marlin_moe() function, the final torch.sum(), is defined
        # essentially as,
        # `torch.sum(workspace1, dim=1, out=output)`
        # Having overlapping input and output tensors for torch.sum seems
        # error prone and depends on how the torch.sum is implemented.
        # For this reason we swap let the output buffer provision from
        # workspace2.

        # Workspace/IntermediateCache allocation matching fused_marlin_moe()
        # workspace1 = (M * topk * max(2 * N, K),)
        # workspace2 = (M * topk, N)

        # Workspace/IntermediateCache allocation accounting for output buffer
        # provisioning
        workspace1 = (M * topk, max(N, K))
        workspace2 = (M * topk * max(2 * N, K),)
        output = (M, K)

        return (workspace1, workspace2, output)
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. Key calls include `max`. It writes or updates `workspace1`, `workspace2`, `output`.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 关键调用包括 `max`。 它会写入或更新 `workspace1`, `workspace2`, `output`。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `_fused_marlin_moe`, `fused_marlin_moe`, `batched_fused_marlin_moe`, `MarlinExpertsBase` / [CN] 核心符号：`_fused_marlin_moe`, `fused_marlin_moe`, `batched_fused_marlin_moe`, `MarlinExpertsBase`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `torch` / **外部依赖**: `collections.abc`, `torch`
- **Internal**: `vllm._custom_ops`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin`, `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils` / **内部依赖**: `vllm._custom_ops`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin`, `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
