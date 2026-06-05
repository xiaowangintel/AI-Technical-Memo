# cpu_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/cpu_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: CPU FP8 W8A16 and MXFP4 W4A16 fused MoE experts. / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-22 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm._custom_ops import CPUQuantMethod, fused_experts_cpu
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8Dynamic128Sym,
    kFp8Static128BlockSym,
    kMxfp4Static,
)
from vllm.platforms import current_platform
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 25-32 — function `prepare_fp8_moe_layer_for_cpu`
```python
def prepare_fp8_moe_layer_for_cpu(
    w13: torch.Tensor,
    w2: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    """VNNI-prepack FP8 MoE weights for CPU kernel."""
    packed_w13 = torch.ops._C.convert_weight_packed(w13)
    packed_w2 = torch.ops._C.convert_weight_packed(w2)
    return packed_w13, packed_w2
```
**EN:** This function defines `prepare_fp8_moe_layer_for_cpu`. VNNI-prepack FP8 MoE weights for CPU kernel. The main inputs are `w13`, `w2`. Key calls include `torch.ops._C.convert_weight_packed`. It writes or updates `packed_w13`, `packed_w2`.
**CN:** 该函数定义 `prepare_fp8_moe_layer_for_cpu`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `w13`, `w2`。 关键调用包括 `torch.ops._C.convert_weight_packed`。 它会写入或更新 `packed_w13`, `packed_w2`。

### Lines 35-174 — class `CPUExpertsFp8`
```python
class CPUExpertsFp8(mk.FusedMoEExpertsMonolithic):
    """CPU FP8 W8A16 block-quantized monolithic MoE experts."""

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(
            moe_config,
            quant_config,
        )

    @property
# ... omitted for brevity ...
            True,  # is_vnni
        )
```
**EN:** This class defines `CPUExpertsFp8`. It inherits from `mk.FusedMoEExpertsMonolithic`. CPU FP8 W8A16 block-quantized monolithic MoE experts. Important methods include `__init__`, `apply`, `expects_unquantized_inputs`, `activation_format`, `supports_expert_map`. Key calls include `super.__init__`, `current_platform.is_cpu`, `select_experts`, `fused_experts_cpu`, `list`, `super`. It writes or updates `SUPPORTED_W_A`, `topk_weights`, `topk_ids`, `block_shape`.
**CN:** 该类定义了 `CPUExpertsFp8`。 它继承自 `mk.FusedMoEExpertsMonolithic`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `expects_unquantized_inputs`, `activation_format`, `supports_expert_map`。 关键调用包括 `super.__init__`, `current_platform.is_cpu`, `select_experts`, `fused_experts_cpu`, `list`, `super`。 它会写入或更新 `SUPPORTED_W_A`, `topk_weights`, `topk_ids`, `block_shape`。

### Lines 38-46 — method `CPUExpertsFp8.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(
            moe_config,
            quant_config,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `super`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `super`。

### Lines 106-174 — method `CPUExpertsFp8.apply`
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
        from vllm.model_executor.layers.fused_moe.cpu_fused_moe import (
            select_experts,
        )

        topk_weights, topk_ids = select_experts(
            hidden_states=hidden_states,
            router_logits=router_logits,
            use_grouped_topk=num_expert_group is not None,
            top_k=self.moe_config.experts_per_token,
            renormalize=self.moe_config.routing_method
            in (
                RoutingMethodType.Renormalize,
                RoutingMethodType.RenormalizeNaive,
            ),
            topk_group=topk_group,
            num_expert_group=num_expert_group,
            scoring_func="softmax",
            routed_scaling_factor=(
                routed_scaling_factor if routed_scaling_factor is not None else 1.0
            ),
            e_score_correction_bias=e_score_correction_bias,
        )

        block_shape = (
            list(self.quant_config.block_shape)
            if self.quant_config.block_shape
            else (
# ... omitted for brevity ...
            True,  # is_vnni
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`. Key calls include `select_experts`, `fused_experts_cpu`, `list`. It writes or updates `topk_weights`, `topk_ids`, `block_shape`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`。 关键调用包括 `select_experts`, `fused_experts_cpu`, `list`。 它会写入或更新 `topk_weights`, `topk_ids`, `block_shape`。

### Lines 48-50 — method `CPUExpertsFp8.expects_unquantized_inputs`
```python
    @property
    def expects_unquantized_inputs(self) -> bool:
        return True
```
**EN:** This method defines `expects_unquantized_inputs`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `expects_unquantized_inputs`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 52-54 — method `CPUExpertsFp8.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 103-104 — method `CPUExpertsFp8.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return False
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 177-188 — function `prepare_mxfp4_moe_layer_for_cpu`
```python
def prepare_mxfp4_moe_layer_for_cpu(
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_scale: torch.Tensor,
    w2_scale: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """VNNI-prepack MXFP4 MoE weights and repack scales for CPU AMX kernel."""
    packed_w13 = torch.ops._C.convert_weight_packed(w13)
    packed_w2 = torch.ops._C.convert_weight_packed(w2)
    packed_w13_scale = torch.ops._C.convert_scale_packed(w13_scale)
    packed_w2_scale = torch.ops._C.convert_scale_packed(w2_scale)
    return packed_w13, packed_w2, packed_w13_scale, packed_w2_scale
```
**EN:** This function defines `prepare_mxfp4_moe_layer_for_cpu`. VNNI-prepack MXFP4 MoE weights and repack scales for CPU AMX kernel. The main inputs are `w13`, `w2`, `w13_scale`, `w2_scale`. Key calls include `torch.ops._C.convert_weight_packed`, `torch.ops._C.convert_scale_packed`. It writes or updates `packed_w13`, `packed_w2`, `packed_w13_scale`, `packed_w2_scale`.
**CN:** 该函数定义 `prepare_mxfp4_moe_layer_for_cpu`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `w13`, `w2`, `w13_scale`, `w2_scale`。 关键调用包括 `torch.ops._C.convert_weight_packed`, `torch.ops._C.convert_scale_packed`。 它会写入或更新 `packed_w13`, `packed_w2`, `packed_w13_scale`, `packed_w2_scale`。

### Lines 191-326 — class `CPUExpertsMxfp4`
```python
class CPUExpertsMxfp4(mk.FusedMoEExpertsMonolithic):
    """CPU MXFP4 W4A16 monolithic MoE experts."""

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(
            moe_config,
            quant_config,
        )

    @property
# ... omitted for brevity ...
            True,  # is_vnni
        )
```
**EN:** This class defines `CPUExpertsMxfp4`. It inherits from `mk.FusedMoEExpertsMonolithic`. CPU MXFP4 W4A16 monolithic MoE experts. Important methods include `__init__`, `apply`, `expects_unquantized_inputs`, `activation_format`, `supports_expert_map`. Key calls include `super.__init__`, `current_platform.is_cpu`, `select_experts`, `getattr`, `fused_experts_cpu`, `super`. It writes or updates `SUPPORTED_W_A`, `topk_weights`, `topk_ids`, `w1_bias`, `w2_bias`, `alpha`.
**CN:** 该类定义了 `CPUExpertsMxfp4`。 它继承自 `mk.FusedMoEExpertsMonolithic`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `expects_unquantized_inputs`, `activation_format`, `supports_expert_map`。 关键调用包括 `super.__init__`, `current_platform.is_cpu`, `select_experts`, `getattr`, `fused_experts_cpu`, `super`。 它会写入或更新 `SUPPORTED_W_A`, `topk_weights`, `topk_ids`, `w1_bias`, `w2_bias`, `alpha`。

### Lines 194-202 — method `CPUExpertsMxfp4.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(
            moe_config,
            quant_config,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `super`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `super`。

### Lines 262-326 — method `CPUExpertsMxfp4.apply`
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
        from vllm.model_executor.layers.fused_moe.cpu_fused_moe import (
            select_experts,
        )

        topk_weights, topk_ids = select_experts(
            hidden_states=hidden_states,
            router_logits=router_logits,
            use_grouped_topk=num_expert_group is not None,
            top_k=self.moe_config.experts_per_token,
            renormalize=self.moe_config.routing_method
            in (
                RoutingMethodType.Renormalize,
                RoutingMethodType.RenormalizeNaive,
            ),
            topk_group=topk_group,
            num_expert_group=num_expert_group,
            scoring_func="softmax",
            routed_scaling_factor=(
                routed_scaling_factor if routed_scaling_factor is not None else 1.0
            ),
            e_score_correction_bias=e_score_correction_bias,
        )

        # Get bias and swiglu params from quant config
        w1_bias = self.quant_config.w1_bias
        w2_bias = self.quant_config.w2_bias
        alpha = getattr(self.quant_config, "gemm1_alpha", None)
# ... omitted for brevity ...
            True,  # is_vnni
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`. Key calls include `select_experts`, `getattr`, `fused_experts_cpu`. It writes or updates `topk_weights`, `topk_ids`, `w1_bias`, `w2_bias`, `alpha`, `limit`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`。 关键调用包括 `select_experts`, `getattr`, `fused_experts_cpu`。 它会写入或更新 `topk_weights`, `topk_ids`, `w1_bias`, `w2_bias`, `alpha`, `limit`。

### Lines 204-206 — method `CPUExpertsMxfp4.expects_unquantized_inputs`
```python
    @property
    def expects_unquantized_inputs(self) -> bool:
        return True
```
**EN:** This method defines `expects_unquantized_inputs`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `expects_unquantized_inputs`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 208-210 — method `CPUExpertsMxfp4.activation_format`
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
- [EN] Core symbols: `prepare_fp8_moe_layer_for_cpu`, `CPUExpertsFp8`, `prepare_mxfp4_moe_layer_for_cpu`, `CPUExpertsMxfp4` / [CN] 核心符号：`prepare_fp8_moe_layer_for_cpu`, `CPUExpertsFp8`, `prepare_mxfp4_moe_layer_for_cpu`, `CPUExpertsMxfp4`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm._custom_ops`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm._custom_ops`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`
- **Runtime traits**: platform-aware dispatch, custom C++/CUDA ops, distributed collectives / **运行时特征**: platform-aware dispatch, custom C++/CUDA ops, distributed collectives
