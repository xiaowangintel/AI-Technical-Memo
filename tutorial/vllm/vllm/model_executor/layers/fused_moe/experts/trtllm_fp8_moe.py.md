# trtllm_fp8_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/trtllm_fp8_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-33 — imports and setup
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
    kFp8Dynamic128Sym,
    kFp8Static128BlockSym,
    kFp8StaticTensorSym,
    kMxfp8Dynamic,
    kMxfp8Static,
)
from vllm.platforms import current_platform
from vllm.utils.flashinfer import has_flashinfer_trtllm_fused_moe

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 36-95 — class `TrtLlmFp8ExpertsBase`
```python
class TrtLlmFp8ExpertsBase:
    """
    Fp8 TRTLLM-Gen MoE kernels. Shared base for modular and monolithic
    interfaces.
    """

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        self.routing_method_type = moe_config.routing_method
        self.topk = moe_config.experts_per_token
        self.intermediate_size_per_partition = (
# ... omitted for brevity ...
    def supports_expert_map(self) -> bool:
        return False
```
**EN:** This class defines `TrtLlmFp8ExpertsBase`. Fp8 TRTLLM-Gen MoE kernels. Important methods include `__init__`, `activation_format`, `supports_chunking`, `supports_expert_map`. Key calls include `p.is_cuda`, `p.is_device_capability_family`, `has_flashinfer_trtllm_fused_moe`. It writes or updates `routing_method_type`, `topk`, `intermediate_size_per_partition`, `hidden_dim`, `local_num_experts`, `ep_rank`.
**CN:** 该类定义了 `TrtLlmFp8ExpertsBase`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `activation_format`, `supports_chunking`, `supports_expert_map`。 关键调用包括 `p.is_cuda`, `p.is_device_capability_family`, `has_flashinfer_trtllm_fused_moe`。 它会写入或更新 `routing_method_type`, `topk`, `intermediate_size_per_partition`, `hidden_dim`, `local_num_experts`, `ep_rank`。

### Lines 42-57 — method `TrtLlmFp8ExpertsBase.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        self.routing_method_type = moe_config.routing_method
        self.topk = moe_config.experts_per_token
        self.intermediate_size_per_partition = (
            moe_config.intermediate_size_per_partition
        )
        self.hidden_dim = moe_config.hidden_dim
        self.local_num_experts = moe_config.num_local_experts
        self.ep_rank = moe_config.moe_parallel_config.ep_rank

        self.moe_config = moe_config
        self.quant_config = quant_config
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. It writes or updates `routing_method_type`, `topk`, `intermediate_size_per_partition`, `hidden_dim`, `local_num_experts`, `ep_rank`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 它会写入或更新 `routing_method_type`, `topk`, `intermediate_size_per_partition`, `hidden_dim`, `local_num_experts`, `ep_rank`。

### Lines 59-61 — method `TrtLlmFp8ExpertsBase.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 91-92 — method `TrtLlmFp8ExpertsBase.supports_chunking`
```python
    def supports_chunking(self) -> bool:
        return False
```
**EN:** This method defines `supports_chunking`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_chunking`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 94-95 — method `TrtLlmFp8ExpertsBase.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return False
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 98-214 — class `TrtLlmFp8ExpertsModular`
```python
class TrtLlmFp8ExpertsModular(TrtLlmFp8ExpertsBase, mk.FusedMoEExpertsModular):
    """
    Fp8 TRTLLM-Gen MoE kernels. Supports modular interface.
    """

    @staticmethod
    def _supports_quant_scheme(
        weight_key: QuantKey | None,
        activation_key: QuantKey | None,
    ) -> bool:
        """Supports Fp8 block and MXFP8."""
        SUPPORTED_W_A = [
            (kFp8Static128BlockSym, kFp8Dynamic128Sym),
            (kMxfp8Static, kMxfp8Dynamic),
# ... omitted for brevity ...
            output=output,
        )
```
**EN:** This class defines `TrtLlmFp8ExpertsModular`. It inherits from `TrtLlmFp8ExpertsBase`, `mk.FusedMoEExpertsModular`. Fp8 TRTLLM-Gen MoE kernels. Important methods include `apply`, `moe_problem_size`, `workspace_shapes`, `finalize_weight_and_reduce_impl`. Key calls include `super.moe_problem_size`, `TopKWeightAndReduceNoOP`, `trtllm_moe_pack_topk_ids_weights`, `flashinfer.fused_moe.trtllm_fp8_block_scale_routed_moe`, `w1.dim`, `a1.size`. It writes or updates `SUPPORTED_W_A`, `workspace1`, `workspace2`, `output`, `packed_topk_ids`, `is_mxfp8`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `TrtLlmFp8ExpertsModular`。 它继承自 `TrtLlmFp8ExpertsBase`, `mk.FusedMoEExpertsModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `apply`, `moe_problem_size`, `workspace_shapes`, `finalize_weight_and_reduce_impl`。 关键调用包括 `super.moe_problem_size`, `TopKWeightAndReduceNoOP`, `trtllm_moe_pack_topk_ids_weights`, `flashinfer.fused_moe.trtllm_fp8_block_scale_routed_moe`, `w1.dim`, `a1.size`。 它会写入或更新 `SUPPORTED_W_A`, `workspace1`, `workspace2`, `output`, `packed_topk_ids`, `is_mxfp8`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 154-214 — method `TrtLlmFp8ExpertsModular.apply`
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
        from flashinfer.fused_moe import Fp8QuantizationType, WeightLayout

        # Pack topk ids and weights into format expected by the kernel.
        packed_topk_ids = trtllm_moe_pack_topk_ids_weights(topk_ids, topk_weights)

        assert a1q_scale is not None

        is_mxfp8 = self.quant_config.block_shape == [1, 32]
        if is_mxfp8:
            fp8_quant_type = Fp8QuantizationType.MxFp8
            use_shuffled_weight = True
            weight_layout = WeightLayout.MajorK
            hidden_states_scale = a1q_scale
        else:
            fp8_quant_type = Fp8QuantizationType.DeepSeekFp8
            use_shuffled_weight = True
            weight_layout = WeightLayout.BlockMajorK
            hidden_states_scale = a1q_scale.t().contiguous()

        flashinfer.fused_moe.trtllm_fp8_block_scale_routed_moe(
            topk_ids=packed_topk_ids,
            routing_bias=None,
            hidden_states=hidden_states,
            hidden_states_scale=hidden_states_scale,
            gemm1_weights=w1,
# ... omitted for brevity ...
            output=output,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `trtllm_moe_pack_topk_ids_weights`, `flashinfer.fused_moe.trtllm_fp8_block_scale_routed_moe`, `a1q_scale.t.contiguous`, `a1q_scale.t`. It writes or updates `packed_topk_ids`, `is_mxfp8`, `fp8_quant_type`, `use_shuffled_weight`, `weight_layout`, `hidden_states_scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `trtllm_moe_pack_topk_ids_weights`, `flashinfer.fused_moe.trtllm_fp8_block_scale_routed_moe`, `a1q_scale.t.contiguous`, `a1q_scale.t`。 它会写入或更新 `packed_topk_ids`, `is_mxfp8`, `fp8_quant_type`, `use_shuffled_weight`, `weight_layout`, `hidden_states_scale`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 115-131 — method `TrtLlmFp8ExpertsModular.moe_problem_size`
```python
    def moe_problem_size(
        self,
        a1: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
        topk_ids: torch.Tensor,
    ) -> tuple[int, int, int, int, int]:
        """Override to handle 4D BlockMajorK weights (E, K/bk, Mn, bk)."""
        if w1.dim() == 4:
            # BlockMajorK: (E, K/bk, Mn, bk)
            E = w1.shape[0]
            N = w1.shape[2]
            K = a1.size(-1)
            M = a1.size(0) if a1.dim() == 2 else a1.size(1)
            topk = topk_ids.size(1)
            return E, M, N, K, topk
        return super().moe_problem_size(a1, w1, w2, topk_ids)
```
**EN:** This method defines `moe_problem_size`. Override to handle 4D BlockMajorK weights (E, K/bk, Mn, bk). The main inputs are `a1`, `w1`, `w2`, `topk_ids`. Key calls include `super.moe_problem_size`, `w1.dim`, `a1.size`, `topk_ids.size`, `super`, `a1.dim`. It writes or updates `E`, `N`, `K`, `M`, `topk`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `moe_problem_size`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `a1`, `w1`, `w2`, `topk_ids`。 关键调用包括 `super.moe_problem_size`, `w1.dim`, `a1.size`, `topk_ids.size`, `super`, `a1.dim`。 它会写入或更新 `E`, `N`, `K`, `M`, `topk`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 133-149 — method `TrtLlmFp8ExpertsModular.workspace_shapes`
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
        output = (M, K)

        return (workspace1, workspace2, output)
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. It writes or updates `workspace1`, `workspace2`, `output`.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 它会写入或更新 `workspace1`, `workspace2`, `output`。

### Lines 151-152 — method `TrtLlmFp8ExpertsModular.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 217-482 — class `TrtLlmFp8ExpertsMonolithic`
```python
class TrtLlmFp8ExpertsMonolithic(TrtLlmFp8ExpertsBase, mk.FusedMoEExpertsMonolithic):
    """
    Fp8 TRTLLM-Gen MoE kernels. Supports monolithic interface.
    """

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(moe_config, quant_config)

        # Make additional scales for per-tensor interface.
        if self.quant_config.is_per_tensor:
# ... omitted for brevity ...
                f"supported in {self.__class__.__name__}."
            )
```
**EN:** This class defines `TrtLlmFp8ExpertsMonolithic`. It inherits from `TrtLlmFp8ExpertsBase`, `mk.FusedMoEExpertsMonolithic`. Fp8 TRTLLM-Gen MoE kernels. Important methods include `__init__`, `apply`. Key calls include `super.__init__`, `flashinfer.fused_moe.trtllm_fp8_block_scale_moe`, `activation_to_flashinfer_int`, `flashinfer.fused_moe.trtllm_fp8_per_tensor_scale_moe`, `squeeze`, `a1q_scale.t.contiguous`. It writes or updates `SUPPORTED_W_A`, `is_mxfp8`, `activation_type`, `out`, `w1_scale`, `a1_scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `TrtLlmFp8ExpertsMonolithic`。 它继承自 `TrtLlmFp8ExpertsBase`, `mk.FusedMoEExpertsMonolithic`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`。 关键调用包括 `super.__init__`, `flashinfer.fused_moe.trtllm_fp8_block_scale_moe`, `activation_to_flashinfer_int`, `flashinfer.fused_moe.trtllm_fp8_per_tensor_scale_moe`, `squeeze`, `a1q_scale.t.contiguous`。 它会写入或更新 `SUPPORTED_W_A`, `is_mxfp8`, `activation_type`, `out`, `w1_scale`, `a1_scale`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 222-246 — method `TrtLlmFp8ExpertsMonolithic.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(moe_config, quant_config)

        # Make additional scales for per-tensor interface.
        if self.quant_config.is_per_tensor:
            w1_scale = self.quant_config.w1_scale
            assert w1_scale is not None
            a1_scale = self.quant_config.a1_scale
            assert a1_scale is not None
            w2_scale = self.quant_config.w2_scale
            assert w2_scale is not None
            a2_scale = self.quant_config.a2_scale
            assert a2_scale is not None

            self._g1_alphas = (w1_scale * a1_scale).squeeze()
            self._g2_alphas = (w2_scale * a2_scale).squeeze()
            self._g1_scale_c = (
                self._g1_alphas / self.quant_config.a2_scale
                if moe_config.is_act_and_mul
                else torch.ones_like(self._g1_alphas) / self.quant_config.a2_scale
            )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `squeeze`, `super`, `torch.ones_like`. It writes or updates `w1_scale`, `a1_scale`, `w2_scale`, `a2_scale`, `_g1_alphas`, `_g2_alphas`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `squeeze`, `super`, `torch.ones_like`。 它会写入或更新 `w1_scale`, `a1_scale`, `w2_scale`, `a2_scale`, `_g1_alphas`, `_g2_alphas`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 430-482 — method `TrtLlmFp8ExpertsMonolithic.apply`
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
        if self.quant_config.block_shape is not None:
            return self._apply_block_scale(
                hidden_states,
                w1,
                w2,
                router_logits,
                activation,
                global_num_experts,
                expert_map,
                a1q_scale,
                apply_router_weight_on_input,
                num_expert_group=num_expert_group,
                e_score_correction_bias=e_score_correction_bias,
                routed_scaling_factor=routed_scaling_factor,
                topk_group=topk_group,
            )
        elif self.quant_config.is_per_tensor:
            return self._apply_per_tensor(
                hidden_states,
                w1,
                w2,
                router_logits,
                activation,
                global_num_experts,
                expert_map,
                a1q_scale,
                apply_router_weight_on_input,
# ... omitted for brevity ...
                f"supported in {self.__class__.__name__}."
            )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`. Key calls include `self._apply_block_scale`, `self._apply_per_tensor`, `NotImplementedError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `router_logits`, `activation`, `global_num_experts`。 关键调用包括 `self._apply_block_scale`, `self._apply_per_tensor`, `NotImplementedError`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `TrtLlmFp8ExpertsBase`, `TrtLlmFp8ExpertsModular`, `TrtLlmFp8ExpertsMonolithic` / [CN] 核心符号：`TrtLlmFp8ExpertsBase`, `TrtLlmFp8ExpertsModular`, `TrtLlmFp8ExpertsMonolithic`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.flashinfer` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.flashinfer`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
