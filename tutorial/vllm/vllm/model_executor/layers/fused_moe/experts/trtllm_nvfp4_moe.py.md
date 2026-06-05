# trtllm_nvfp4_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/trtllm_nvfp4_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-31 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.fused_moe.utils import trtllm_moe_pack_topk_ids_weights
from vllm.model_executor.layers.quantization.utils.flashinfer_utils import (
    activation_to_flashinfer_int,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kNvfp4Dynamic,
    kNvfp4Static,
)
from vllm.platforms import current_platform
from vllm.utils.flashinfer import has_flashinfer_trtllm_fused_moe

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 34-139 — class `TrtLlmNvFp4ExpertsBase`
```python
class TrtLlmNvFp4ExpertsBase:
    """
    NvFp4 TRTLLM-Gen MoE kernels. Supports modular and monolithic interface.
    """

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        self.moe_config = moe_config
        self.quant_config = quant_config

        self.routing_method_type = self.moe_config.routing_method
# ... omitted for brevity ...
    def supports_expert_map(self) -> bool:
        return False
```
**EN:** This class defines `TrtLlmNvFp4ExpertsBase`. NvFp4 TRTLLM-Gen MoE kernels. Important methods include `__init__`, `process_weights_after_loading`, `activation_format`, `supports_chunking`, `supports_expert_map`. Key calls include `layer.w13_weight_scale_2.data.mul_`, `layer.w2_weight_scale_2.data.mul_`, `layer.register_parameter`, `self.quant_config.a2_gscale.clone`, `torch.nn.Parameter`, `p.is_cuda`. It writes or updates `moe_config`, `quant_config`, `routing_method_type`, `topk`, `intermediate_size_per_partition`, `hidden_dim`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `TrtLlmNvFp4ExpertsBase`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `process_weights_after_loading`, `activation_format`, `supports_chunking`, `supports_expert_map`。 关键调用包括 `layer.w13_weight_scale_2.data.mul_`, `layer.w2_weight_scale_2.data.mul_`, `layer.register_parameter`, `self.quant_config.a2_gscale.clone`, `torch.nn.Parameter`, `p.is_cuda`。 它会写入或更新 `moe_config`, `quant_config`, `routing_method_type`, `topk`, `intermediate_size_per_partition`, `hidden_dim`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 39-67 — method `TrtLlmNvFp4ExpertsBase.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        self.moe_config = moe_config
        self.quant_config = quant_config

        self.routing_method_type = self.moe_config.routing_method
        self.topk = moe_config.experts_per_token
        self.intermediate_size_per_partition = (
            moe_config.intermediate_size_per_partition
        )
        self.hidden_dim = moe_config.hidden_dim
        self.hidden_dim_unpadded = (
            moe_config.hidden_dim_unpadded or moe_config.hidden_dim
        )
        self.local_num_experts = moe_config.num_local_experts
        self.ep_rank = moe_config.moe_parallel_config.ep_rank

        assert self.quant_config.g1_alphas is not None
        assert self.quant_config.a2_gscale is not None
        if moe_config.is_act_and_mul:
            # g1_alpha_s = a13_scale * w13_scale_2
            # a2_gscale = (1 / a2_scale)
            # g1_scale_c = a13_scale * w13_scale_2 / a2_scale
            self.g1_scale_c = self.quant_config.g1_alphas * self.quant_config.a2_gscale
        else:
            self.g1_scale_c = self.quant_config.a2_gscale.clone()
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `self.quant_config.a2_gscale.clone`. It writes or updates `moe_config`, `quant_config`, `routing_method_type`, `topk`, `intermediate_size_per_partition`, `hidden_dim`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `self.quant_config.a2_gscale.clone`。 它会写入或更新 `moe_config`, `quant_config`, `routing_method_type`, `topk`, `intermediate_size_per_partition`, `hidden_dim`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 69-85 — method `TrtLlmNvFp4ExpertsBase.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.w13_weight_scale_2.data.mul_(layer.w13_input_scale)
        layer.w2_weight_scale_2.data.mul_(layer.w2_input_scale)
        # Recompute g1_scale_c since g1_alphas was just fused in-place.
        # Register as a layer parameter so EPLB rearranges it alongside
        # other expert weights.
        assert self.quant_config.g1_alphas is not None
        assert self.quant_config.a2_gscale is not None
        if self.moe_config.is_act_and_mul:
            g1_scale_c = self.quant_config.g1_alphas * self.quant_config.a2_gscale
        else:
            g1_scale_c = self.quant_config.a2_gscale.clone()
        layer.register_parameter(
            "g1_scale_c",
            torch.nn.Parameter(g1_scale_c, requires_grad=False),
        )
        self.g1_scale_c = layer.g1_scale_c
```
**EN:** This method defines `process_weights_after_loading`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `layer`. Key calls include `layer.w13_weight_scale_2.data.mul_`, `layer.w2_weight_scale_2.data.mul_`, `layer.register_parameter`, `self.quant_config.a2_gscale.clone`, `torch.nn.Parameter`. It writes or updates `g1_scale_c`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `process_weights_after_loading`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `layer`。 关键调用包括 `layer.w13_weight_scale_2.data.mul_`, `layer.w2_weight_scale_2.data.mul_`, `layer.register_parameter`, `self.quant_config.a2_gscale.clone`, `torch.nn.Parameter`。 它会写入或更新 `g1_scale_c`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 131-133 — method `TrtLlmNvFp4ExpertsBase.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 135-136 — method `TrtLlmNvFp4ExpertsBase.supports_chunking`
```python
    def supports_chunking(self) -> bool:
        return False
```
**EN:** This method defines `supports_chunking`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_chunking`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 138-139 — method `TrtLlmNvFp4ExpertsBase.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return False
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 142-237 — class `TrtLlmNvFp4ExpertsModular`
```python
class TrtLlmNvFp4ExpertsModular(TrtLlmNvFp4ExpertsBase, mk.FusedMoEExpertsModular):
    """
    Modular version of the implementation (just the experts).
    """

    @staticmethod
    def _supports_parallel_config(moe_parallel_config: FusedMoEParallelConfig) -> bool:
        """The modular implementation supports all parallel configs."""
        return True

    def workspace_shapes(
        self,
        M: int,
        N: int,
# ... omitted for brevity ...
            output=output,
        )
```
**EN:** This class defines `TrtLlmNvFp4ExpertsModular`. It inherits from `TrtLlmNvFp4ExpertsBase`, `mk.FusedMoEExpertsModular`. Modular version of the implementation (just the experts). Important methods include `apply`, `workspace_shapes`, `finalize_weight_and_reduce_impl`. Key calls include `TopKWeightAndReduceNoOP`, `self._supports_activation`, `trtllm_moe_pack_topk_ids_weights`, `flashinfer.fused_moe.trtllm_fp4_block_scale_routed_moe`, `a1q_scale.view.reshape`, `self.quant_config.w1_scale.view`. It writes or updates `workspace1`, `workspace2`, `output`, `packed_tensor`.
**CN:** 该类定义了 `TrtLlmNvFp4ExpertsModular`。 它继承自 `TrtLlmNvFp4ExpertsBase`, `mk.FusedMoEExpertsModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `apply`, `workspace_shapes`, `finalize_weight_and_reduce_impl`。 关键调用包括 `TopKWeightAndReduceNoOP`, `self._supports_activation`, `trtllm_moe_pack_topk_ids_weights`, `flashinfer.fused_moe.trtllm_fp4_block_scale_routed_moe`, `a1q_scale.view.reshape`, `self.quant_config.w1_scale.view`。 它会写入或更新 `workspace1`, `workspace2`, `output`, `packed_tensor`。

### Lines 177-237 — method `TrtLlmNvFp4ExpertsModular.apply`
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
        import flashinfer

        assert self._supports_activation(activation)
        assert a1q_scale is not None
        assert self.quant_config.w1_scale is not None
        assert self.quant_config.w2_scale is not None

        # Pack topk ids and weights into format expected by the kernel.
        packed_tensor = trtllm_moe_pack_topk_ids_weights(topk_ids, topk_weights)

        # Invoke kernel.
        flashinfer.fused_moe.trtllm_fp4_block_scale_routed_moe(
            topk_ids=packed_tensor,
            routing_bias=None,
            hidden_states=hidden_states,
            hidden_states_scale=a1q_scale.view(torch.float8_e4m3fn).reshape(
                *hidden_states.shape[:-1], -1
            ),
            gemm1_weights=w1,
            gemm1_weights_scale=self.quant_config.w1_scale.view(torch.float8_e4m3fn),
            gemm1_bias=None,
            gemm1_alpha=None,
            gemm1_beta=None,
            gemm1_clamp_limit=None,
            gemm2_weights=w2,
            gemm2_weights_scale=self.quant_config.w2_scale.view(torch.float8_e4m3fn),
# ... omitted for brevity ...
            output=output,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `self._supports_activation`, `trtllm_moe_pack_topk_ids_weights`, `flashinfer.fused_moe.trtllm_fp4_block_scale_routed_moe`, `a1q_scale.view.reshape`, `self.quant_config.w1_scale.view`, `self.quant_config.w2_scale.view`. It writes or updates `packed_tensor`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `self._supports_activation`, `trtllm_moe_pack_topk_ids_weights`, `flashinfer.fused_moe.trtllm_fp4_block_scale_routed_moe`, `a1q_scale.view.reshape`, `self.quant_config.w1_scale.view`, `self.quant_config.w2_scale.view`。 它会写入或更新 `packed_tensor`。

### Lines 152-172 — method `TrtLlmNvFp4ExpertsModular.workspace_shapes`
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
        # The workspaces for this implementation are managed by flashinfer.
        workspace1 = (0,)
        workspace2 = (0,)

        # Hidden states are Nvfp4, packed into int8 dtype, so we
        # need to multiply K by 2 to get the output shape right.
        assert self.hidden_dim == K * 2
        output = (M, self.hidden_dim)

        return (workspace1, workspace2, output)
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. It writes or updates `workspace1`, `workspace2`, `output`.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 它会写入或更新 `workspace1`, `workspace2`, `output`。

### Lines 174-175 — method `TrtLlmNvFp4ExpertsModular.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 240-349 — class `TrtLlmNvFp4ExpertsMonolithic`
```python
class TrtLlmNvFp4ExpertsMonolithic(
    TrtLlmNvFp4ExpertsBase, mk.FusedMoEExpertsMonolithic
):
    """
    Monolithic version of the kernel (router + experts).
    """

    @staticmethod
    def _supports_parallel_config(moe_parallel_config: FusedMoEParallelConfig) -> bool:
        """The modular implementation should be used for the Dp/Ep or EPLB case."""
        return (
            not moe_parallel_config.use_all2all_kernels
            and not moe_parallel_config.enable_eplb
        )
# ... omitted for brevity ...
            activation_type=activation_to_flashinfer_int(activation),
        )[0]
```
**EN:** This class defines `TrtLlmNvFp4ExpertsMonolithic`. It inherits from `TrtLlmNvFp4ExpertsBase`, `mk.FusedMoEExpertsMonolithic`. Monolithic version of the kernel (router + experts). Important methods include `apply`. Key calls include `self._supports_activation`, `e_score_correction_bias.to`, `flashinfer.fused_moe.trtllm_fp4_block_scale_moe`, `a1q_scale.view.reshape`, `self.quant_config.w1_scale.view`, `self.quant_config.w2_scale.view`. It writes or updates `e_score_correction_bias`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `TrtLlmNvFp4ExpertsMonolithic`。 它继承自 `TrtLlmNvFp4ExpertsBase`, `mk.FusedMoEExpertsMonolithic`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `apply`。 关键调用包括 `self._supports_activation`, `e_score_correction_bias.to`, `flashinfer.fused_moe.trtllm_fp4_block_scale_moe`, `a1q_scale.view.reshape`, `self.quant_config.w1_scale.view`, `self.quant_config.w2_scale.view`。 它会写入或更新 `e_score_correction_bias`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 280-349 — method `TrtLlmNvFp4ExpertsMonolithic.apply`
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
        import flashinfer

        assert self._supports_activation(activation)
        assert a1q_scale is not None
        assert self.quant_config.w1_scale is not None
        assert self.quant_config.w2_scale is not None
        assert (
            apply_router_weight_on_input
            and self.routing_method_type == RoutingMethodType.Llama4
        ) or (
            not apply_router_weight_on_input
            and self.routing_method_type != RoutingMethodType.Llama4
        )

        # Currently FI requires bfloat16 routing bias.
        # https://github.com/flashinfer-ai/flashinfer/issues/2909
        if e_score_correction_bias is not None:
            e_score_correction_bias = e_score_correction_bias.to(torch.bfloat16)

        # Invoke kernel.
        # NOTE: Activation padding and output
        # truncation are handled by the MoE runner's
        return flashinfer.fused_moe.trtllm_fp4_block_scale_moe(
            routing_logits=router_logits,
            routing_bias=e_score_correction_bias,
            hidden_states=hidden_states,
            hidden_states_scale=a1q_scale.view(torch.float8_e4m3fn).reshape(
# ... omitted for brevity ...
            activation_type=activation_to_flashinfer_int(activation),
        )[0]
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`. Key calls include `self._supports_activation`, `e_score_correction_bias.to`, `flashinfer.fused_moe.trtllm_fp4_block_scale_moe`, `a1q_scale.view.reshape`, `self.quant_config.w1_scale.view`, `self.quant_config.w2_scale.view`. It writes or updates `e_score_correction_bias`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`。 关键调用包括 `self._supports_activation`, `e_score_correction_bias.to`, `flashinfer.fused_moe.trtllm_fp4_block_scale_moe`, `a1q_scale.view.reshape`, `self.quant_config.w1_scale.view`, `self.quant_config.w2_scale.view`。 它会写入或更新 `e_score_correction_bias`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `TrtLlmNvFp4ExpertsBase`, `TrtLlmNvFp4ExpertsModular`, `TrtLlmNvFp4ExpertsMonolithic` / [CN] 核心符号：`TrtLlmNvFp4ExpertsBase`, `TrtLlmNvFp4ExpertsModular`, `TrtLlmNvFp4ExpertsMonolithic`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.flashinfer` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.flashinfer`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
