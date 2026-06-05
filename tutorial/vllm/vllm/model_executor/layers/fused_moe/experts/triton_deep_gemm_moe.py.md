# triton_deep_gemm_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/triton_deep_gemm_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-21 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe import (
    DeepGemmExperts,
    _valid_deep_gemm,
    _valid_deep_gemm_shape,
)
from vllm.model_executor.layers.fused_moe.experts.fallback import FallbackExperts
from vllm.model_executor.layers.fused_moe.experts.triton_moe import TritonExperts
from vllm.utils.deep_gemm import (
    is_deep_gemm_e8m0_used,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 24-86 — class `TritonOrDeepGemmExperts`
```python
class TritonOrDeepGemmExperts(FallbackExperts):
    """DeepGemm with fallback to Triton for low latency shapes."""

    def __init__(self, moe_config: FusedMoEConfig, quant_config: FusedMoEQuantConfig):
        super().__init__(
            experts=DeepGemmExperts(moe_config, quant_config),
            fallback_experts=TritonExperts(moe_config, quant_config),
        )

    @staticmethod
    def get_clses() -> tuple[
        type[mk.FusedMoEExpertsModular],
        type[mk.FusedMoEExpertsModular],
    ]:
# ... omitted for brevity ...
        else:
            return self.fallback_experts
```
**EN:** This class defines `TritonOrDeepGemmExperts`. It inherits from `FallbackExperts`. DeepGemm with fallback to Triton for low latency shapes. Important methods include `__init__`, `get_clses`, `workspace_shapes`, `_select_experts_impl`. Key calls include `super.__init__`, `is_deep_gemm_e8m0_used`, `_valid_deep_gemm_shape`, `self.experts.workspace_shapes`, `self.fallback_experts.workspace_shapes`, `_valid_deep_gemm`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `TritonOrDeepGemmExperts`。 它继承自 `FallbackExperts`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `get_clses`, `workspace_shapes`, `_select_experts_impl`。 关键调用包括 `super.__init__`, `is_deep_gemm_e8m0_used`, `_valid_deep_gemm_shape`, `self.experts.workspace_shapes`, `self.fallback_experts.workspace_shapes`, `_valid_deep_gemm`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 27-31 — method `TritonOrDeepGemmExperts.__init__`
```python
    def __init__(self, moe_config: FusedMoEConfig, quant_config: FusedMoEQuantConfig):
        super().__init__(
            experts=DeepGemmExperts(moe_config, quant_config),
            fallback_experts=TritonExperts(moe_config, quant_config),
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `super`, `DeepGemmExperts`, `TritonExperts`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `super`, `DeepGemmExperts`, `TritonExperts`。

### Lines 33-38 — method `TritonOrDeepGemmExperts.get_clses`
```python
    @staticmethod
    def get_clses() -> tuple[
        type[mk.FusedMoEExpertsModular],
        type[mk.FusedMoEExpertsModular],
    ]:
        return (DeepGemmExperts, TritonExperts)
```
**EN:** This method defines `get_clses`. It derives and returns computed metadata or outputs needed by later stages.
**CN:** 该方法定义 `get_clses`。 它推导并返回后续阶段所需的元数据或输出。

### Lines 40-75 — method `TritonOrDeepGemmExperts.workspace_shapes`
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
        # Note: the deep gemm workspaces are strictly larger than the triton
        # workspaces so we can be pessimistic here and allocate for DeepGemm
        # even if we fall back to triton later, e.g. if expert maps are set.
        if is_deep_gemm_e8m0_used() or _valid_deep_gemm_shape(M, N, K):
            return self.experts.workspace_shapes(
                M,
                N,
                K,
                topk,
                global_num_experts,
                local_num_experts,
                expert_tokens_meta,
                activation,
            )
        else:
            return self.fallback_experts.workspace_shapes(
                M,
                N,
                K,
                topk,
                global_num_experts,
                local_num_experts,
                expert_tokens_meta,
                activation,
            )
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. Key calls include `is_deep_gemm_e8m0_used`, `_valid_deep_gemm_shape`, `self.experts.workspace_shapes`, `self.fallback_experts.workspace_shapes`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 关键调用包括 `is_deep_gemm_e8m0_used`, `_valid_deep_gemm_shape`, `self.experts.workspace_shapes`, `self.fallback_experts.workspace_shapes`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 77-86 — method `TritonOrDeepGemmExperts._select_experts_impl`
```python
    def _select_experts_impl(
        self,
        hidden_states: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
    ) -> mk.FusedMoEExpertsModular:
        if is_deep_gemm_e8m0_used() or _valid_deep_gemm(hidden_states, w1, w2):
            return self.experts
        else:
            return self.fallback_experts
```
**EN:** This method defines `_select_experts_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_states`, `w1`, `w2`. Key calls include `is_deep_gemm_e8m0_used`, `_valid_deep_gemm`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `_select_experts_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_states`, `w1`, `w2`。 关键调用包括 `is_deep_gemm_e8m0_used`, `_valid_deep_gemm`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `TritonOrDeepGemmExperts` / [CN] 核心符号：`TritonOrDeepGemmExperts`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe`, `vllm.model_executor.layers.fused_moe.experts.fallback`, `vllm.model_executor.layers.fused_moe.experts.triton_moe`, `vllm.utils.deep_gemm` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe`, `vllm.model_executor.layers.fused_moe.experts.fallback`, `vllm.model_executor.layers.fused_moe.experts.triton_moe`, `vllm.utils.deep_gemm`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
