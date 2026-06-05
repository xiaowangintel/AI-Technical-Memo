# aiter_mxfp4_w4a8_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/aiter_mxfp4_w4a8_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-24 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm._aiter_ops import rocm_aiter_ops
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8StaticTensorSym,
    kMxfp4Static,
)

__all__ = [
    "AiterW4A8ExpertsMonolithic",
    "aiter_triton_kernel_w4a8_moe_forward",
]
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. It writes or updates `__all__`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 它会写入或更新 `__all__`。

### Lines 27-71 — function `aiter_triton_kernel_w4a8_moe_forward`
```python
def aiter_triton_kernel_w4a8_moe_forward(
    hidden_states: torch.Tensor,
    w1,  # Tensor or triton_kernels.Tensor
    w2,  # Tensor or triton_kernels.Tensor
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    activation: MoEActivation = MoEActivation.SWIGLUOAI,
    quant_config: FusedMoEQuantConfig | None = None,
    apply_router_weight_on_input: bool = False,
    global_num_experts: int = -1,
    expert_map: torch.Tensor | None = None,
    unpadded_N_w1=None,
    unpadded_K_w1=None,
    unpadded_N_w2=None,
    unpadded_K_w2=None,
):
    assert (
        quant_config is not None
        and quant_config.use_mxfp4_w4a8
        and rocm_aiter_ops.is_enabled()
    )
    from aiter.ops.triton.moe_routing.routing import routing as aiter_routing

    routing_data, gather_idx, scatter_idx = aiter_routing(
        gating_output, topk, sm_first=not renormalize
    )
    return triton_kernel_fused_mxfp4_w4a8_experts(
        None,
        hidden_states,
        w1,
        w2,
        routing_data,
        gather_idx,
        scatter_idx,
        activation=activation.value,
        quant_config=quant_config,
        apply_router_weight_on_input=apply_router_weight_on_input,
        global_num_experts=global_num_experts,
        expert_map=expert_map,
        unpadded_N_w1=unpadded_N_w1,
        unpadded_K_w1=unpadded_K_w1,
        unpadded_N_w2=unpadded_N_w2,
        unpadded_K_w2=unpadded_K_w2,
    )
```
**EN:** This function defines `aiter_triton_kernel_w4a8_moe_forward`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `w1`, `w2`, `gating_output`, `topk`, `renormalize`. Key calls include `aiter_routing`, `triton_kernel_fused_mxfp4_w4a8_experts`, `rocm_aiter_ops.is_enabled`. It writes or updates `routing_data`, `gather_idx`, `scatter_idx`.
**CN:** 该函数定义 `aiter_triton_kernel_w4a8_moe_forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `gating_output`, `topk`, `renormalize`。 关键调用包括 `aiter_routing`, `triton_kernel_fused_mxfp4_w4a8_experts`, `rocm_aiter_ops.is_enabled`。 它会写入或更新 `routing_data`, `gather_idx`, `scatter_idx`。

### Lines 74-163 — function `triton_kernel_fused_mxfp4_w4a8_experts`
```python
def triton_kernel_fused_mxfp4_w4a8_experts(
    output_tensor: torch.Tensor,
    hidden_states: torch.Tensor,
    w1,  # Tensor or triton_kernels.Tensor
    w2,  # Tensor or triton_kernels.Tensor
    routing_data,  # RoutingData
    gather_indx,  # GatherIndx
    scatter_indx,  # ScatterIndx
    activation: str = "silu",
    quant_config: FusedMoEQuantConfig | None = None,
    swiglu_alpha: float = 1.702,
    swiglu_limit: float = 7.0,
    apply_router_weight_on_input: bool = False,
    global_num_experts: int = -1,
    expert_map: torch.Tensor | None = None,
    a1q_scale: torch.Tensor | None = None,
    unpadded_N_w1=None,
    unpadded_K_w1=None,
    unpadded_N_w2=None,
    unpadded_K_w2=None,
) -> torch.Tensor:
    assert quant_config is not None
    # type check, uint8 means mxfp4
    assert hidden_states.dtype == torch.bfloat16
    assert quant_config.w1_bias is None or quant_config.w1_bias.dtype == torch.float32
    assert quant_config.w2_bias is None or quant_config.w2_bias.dtype == torch.float32

    # Shape check: weights are padded (e.g. hidden_size padded for
    # GFX950 swizzle).
    assert hidden_states.shape[-1] == w1.shape[-2]
    assert w2.shape[-1] == w1.shape[1]

    E, _, N = w1.shape

    if global_num_experts == -1:
        global_num_experts = E

    gammas = routing_data.gate_scal if routing_data else None

    from aiter.ops.triton.moe_op_gemm_a8w4 import moe_gemm_a8w4
    from aiter.ops.triton.quant_moe import downcast_to_static_fp8

    assert quant_config.w1_precision is not None, (
        "w1_precision in quant config can't be None"
# ... omitted for brevity ...

    return intermediate_cache3
```
**EN:** This function defines `triton_kernel_fused_mxfp4_w4a8_experts`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `output_tensor`, `hidden_states`, `w1`, `w2`, `routing_data`, `gather_indx`. Key calls include `downcast_to_static_fp8`, `moe_gemm_a8w4`. It writes or updates `E`, `_`, `N`, `gammas`, `hidden_states`, `intermediate_cache1`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `triton_kernel_fused_mxfp4_w4a8_experts`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `output_tensor`, `hidden_states`, `w1`, `w2`, `routing_data`, `gather_indx`。 关键调用包括 `downcast_to_static_fp8`, `moe_gemm_a8w4`。 它会写入或更新 `E`, `_`, `N`, `gammas`, `hidden_states`, `intermediate_cache1`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 166-292 — class `AiterW4A8ExpertsMonolithic`
```python
class AiterW4A8ExpertsMonolithic(mk.FusedMoEExpertsMonolithic):
    """
    Monolithic MXFP4 W4A8 expert using AITER triton kernels.

    This backend uses:
    - aiter.ops.triton.moe_routing.routing for routing
    - aiter.ops.triton.moe_op_gemm_a8w4.moe_gemm_a8w4 for computation

    Weight format: MXFP4 weights with GFX950 swizzle
    Activation: Static FP8 quantization
    """

    def __init__(
        self,
# ... omitted for brevity ...
            unpadded_K_w2=self.moe_config.intermediate_size_per_partition_unpadded,
        )
```
**EN:** This class defines `AiterW4A8ExpertsMonolithic`. It inherits from `mk.FusedMoEExpertsMonolithic`. Monolithic MXFP4 W4A8 expert using AITER triton kernels. Important methods include `__init__`, `apply`, `activation_format`, `supports_expert_map`, `expects_unquantized_inputs`. Key calls include `super.__init__`, `on_gfx950`, `aiter_triton_kernel_w4a8_moe_forward`, `rocm_aiter_ops.is_enabled`, `super`. It writes or updates `topk`, `renormalize`, `SUPPORTED_W_A`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `AiterW4A8ExpertsMonolithic`。 它继承自 `mk.FusedMoEExpertsMonolithic`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `activation_format`, `supports_expert_map`, `expects_unquantized_inputs`。 关键调用包括 `super.__init__`, `on_gfx950`, `aiter_triton_kernel_w4a8_moe_forward`, `rocm_aiter_ops.is_enabled`, `super`。 它会写入或更新 `topk`, `renormalize`, `SUPPORTED_W_A`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 178-188 — method `AiterW4A8ExpertsMonolithic.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(moe_config, quant_config)
        self.topk = moe_config.experts_per_token
        self.renormalize = moe_config.routing_method in (
            RoutingMethodType.Renormalize,
            RoutingMethodType.RenormalizeNaive,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `super`. It writes or updates `topk`, `renormalize`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `topk`, `renormalize`。

### Lines 258-292 — method `AiterW4A8ExpertsMonolithic.apply`
```python
    def apply(
        self,
        hidden_states: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
        router_logits: torch.Tensor,
        activation: MoEActivation,
        global_num_experts: int,
        expert_map: torch.Tensor | None,
        a1q_scale: torch.Tensor | None,
        apply_router_weight_on_input: bool,
        # grouped topk + fused topk bias parameters
        num_expert_group: int | None = None,
        e_score_correction_bias: torch.Tensor | None = None,
        routed_scaling_factor: float | None = None,
        topk_group: int | None = None,
    ) -> torch.Tensor:
        assert self.moe_config.intermediate_size_per_partition_unpadded is not None
        assert self.moe_config.hidden_dim_unpadded is not None
        return aiter_triton_kernel_w4a8_moe_forward(
            hidden_states=hidden_states,
            w1=w1,
            w2=w2,
            gating_output=router_logits,
            topk=self.topk,
            renormalize=self.renormalize,
            global_num_experts=global_num_experts,
            expert_map=expert_map,
            quant_config=self.quant_config,
            apply_router_weight_on_input=apply_router_weight_on_input,
            unpadded_N_w1=self.moe_config.intermediate_size_per_partition_unpadded * 2,
            unpadded_K_w1=self.moe_config.hidden_dim_unpadded,
            unpadded_N_w2=self.moe_config.hidden_dim_unpadded,
            unpadded_K_w2=self.moe_config.intermediate_size_per_partition_unpadded,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`. Key calls include `aiter_triton_kernel_w4a8_moe_forward`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`。 关键调用包括 `aiter_triton_kernel_w4a8_moe_forward`。

### Lines 190-192 — method `AiterW4A8ExpertsMonolithic.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 251-252 — method `AiterW4A8ExpertsMonolithic.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return False  # Expert parallelism not yet supported
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 254-256 — method `AiterW4A8ExpertsMonolithic.expects_unquantized_inputs`
```python
    @property
    def expects_unquantized_inputs(self) -> bool:
        return True
```
**EN:** This method defines `expects_unquantized_inputs`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `expects_unquantized_inputs`。 它管理专家分发、融合内核或 MoE 特有的张量流。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `aiter_triton_kernel_w4a8_moe_forward`, `triton_kernel_fused_mxfp4_w4a8_experts`, `AiterW4A8ExpertsMonolithic` / [CN] 核心符号：`aiter_triton_kernel_w4a8_moe_forward`, `triton_kernel_fused_mxfp4_w4a8_experts`, `AiterW4A8ExpertsMonolithic`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm._aiter_ops`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.quant_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm._aiter_ops`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.quant_utils`
- **Runtime traits**: Triton kernels, distributed collectives / **运行时特征**: Triton kernels, distributed collectives
