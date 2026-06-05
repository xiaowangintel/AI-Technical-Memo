# unquantized_fused_moe_method.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/unquantized_fused_moe_method.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-43 — imports and setup
```python
from collections.abc import Callable
from typing import TYPE_CHECKING

import torch
import torch.nn.functional as F
from torch.nn import Module

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp
from vllm.model_executor.layers.fused_moe.config import (
    FUSED_MOE_UNQUANTIZED_CONFIG,
    FusedMoEConfig,
    FusedMoEQuantConfig,
    biased_moe_quant_config,
)
from vllm.model_executor.layers.fused_moe.fused_moe_method_base import (
    FusedMoEMethodBase,
)
from vllm.model_executor.layers.fused_moe.modular_kernel import (
    FusedMoEExpertsModular,
    FusedMoEPrepareAndFinalizeModular,
)
from vllm.model_executor.layers.fused_moe.oracle.unquantized import (
    UnquantizedMoeBackend,
    convert_to_unquantized_kernel_format,
    make_unquantized_moe_kernel,
    select_unquantized_moe_backend,
)
from vllm.model_executor.layers.fused_moe.runner.shared_experts import (
    SharedExperts,
)
from vllm.model_executor.utils import replace_parameter, set_weight_attrs
from vllm.platforms import current_platform
from vllm.platforms.interface import CpuArchEnum

if TYPE_CHECKING:
    from vllm.model_executor.layers.fused_moe import RoutedExperts

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`. The body uses conditional branches to cover different runtime cases.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 47-387 — class `UnquantizedFusedMoEMethod`
```python
@CustomOp.register("unquantized_fused_moe")
class UnquantizedFusedMoEMethod(FusedMoEMethodBase, CustomOp):
    """MoE method without quantization."""

    # --8<-- [end:unquantized_fused_moe]

    def __init__(self, moe: FusedMoEConfig):
        super().__init__(moe)
        self.unquantized_backend, self.experts_cls = select_unquantized_moe_backend(
            moe_config=self.moe,
        )

    @property
    def is_monolithic(self) -> bool:
# ... omitted for brevity ...
                routed_scaling_factor=layer.routed_scaling_factor,
            )
```
**EN:** This class defines `UnquantizedFusedMoEMethod`. It inherits from `FusedMoEMethodBase`, `CustomOp`. MoE method without quantization. Important methods include `__init__`, `apply`, `forward_native`, `forward_cuda`, `is_monolithic`, `supports_eplb`. Key calls include `CustomOp.register`, `super.__init__`, `select_unquantized_moe_backend`, `ValueError`, `torch.nn.Parameter`, `layer.register_parameter`. It writes or updates `unquantized_backend`, `experts_cls`, `w13_weight`, `w2_weight`, `w13_new`, `w2_new`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `UnquantizedFusedMoEMethod`。 它继承自 `FusedMoEMethodBase`, `CustomOp`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `forward_native`, `forward_cuda`, `is_monolithic`, `supports_eplb`。 关键调用包括 `CustomOp.register`, `super.__init__`, `select_unquantized_moe_backend`, `ValueError`, `torch.nn.Parameter`, `layer.register_parameter`。 它会写入或更新 `unquantized_backend`, `experts_cls`, `w13_weight`, `w2_weight`, `w13_new`, `w2_new`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 53-57 — method `UnquantizedFusedMoEMethod.__init__`
```python
    def __init__(self, moe: FusedMoEConfig):
        super().__init__(moe)
        self.unquantized_backend, self.experts_cls = select_unquantized_moe_backend(
            moe_config=self.moe,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe`. Key calls include `super.__init__`, `select_unquantized_moe_backend`, `super`. It writes or updates `unquantized_backend`, `experts_cls`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe`。 关键调用包括 `super.__init__`, `select_unquantized_moe_backend`, `super`。 它会写入或更新 `unquantized_backend`, `experts_cls`。

### Lines 284-300 — method `UnquantizedFusedMoEMethod.apply`
```python
    def apply(
        self,
        layer: "RoutedExperts",
        x: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        shared_experts: SharedExperts | None,
        shared_experts_input: torch.Tensor | None,
    ) -> torch.Tensor:
        return self.forward(
            layer=layer,
            x=x,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            shared_experts=shared_experts,
            shared_experts_input=shared_experts_input,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`. Key calls include `self.forward`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`。 关键调用包括 `self.forward`。

### Lines 302-324 — method `UnquantizedFusedMoEMethod.forward_native`
```python
    def forward_native(
        self,
        layer: "RoutedExperts",
        x: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        shared_experts: SharedExperts | None,
        shared_experts_input: torch.Tensor | None,
    ) -> torch.Tensor:
        assert self.moe_kernel is not None
        return self.moe_kernel.apply(
            hidden_states=x,
            w1=layer.w13_weight,
            w2=layer.w2_weight,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            activation=layer.activation,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            global_num_experts=layer.global_num_experts,
            expert_map=layer.expert_map,
            shared_experts=shared_experts,
            shared_experts_input=shared_experts_input,
        )
```
**EN:** This method defines `forward_native`. It executes the main forward/runtime path for this component. The main inputs are `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`. Key calls include `self.moe_kernel.apply`.
**CN:** 该方法定义 `forward_native`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`。 关键调用包括 `self.moe_kernel.apply`。

### Lines 326-342 — method `UnquantizedFusedMoEMethod.forward_cuda`
```python
    def forward_cuda(
        self,
        layer: "RoutedExperts",
        x: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        shared_experts: SharedExperts | None,
        shared_experts_input: torch.Tensor | None,
    ) -> torch.Tensor:
        return self.forward_native(
            layer,
            x,
            topk_weights,
            topk_ids,
            shared_experts,
            shared_experts_input,
        )
```
**EN:** This method defines `forward_cuda`. It executes the main forward/runtime path for this component. The main inputs are `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`. Key calls include `self.forward_native`.
**CN:** 该方法定义 `forward_cuda`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`。 关键调用包括 `self.forward_native`。

### Lines 59-64 — method `UnquantizedFusedMoEMethod.is_monolithic`
```python
    @property
    def is_monolithic(self) -> bool:
        # Escape hatch for CPU, which stays on the old monolithic path.
        if self.unquantized_backend == UnquantizedMoeBackend.CPU:
            return True
        return super().is_monolithic
```
**EN:** This method defines `is_monolithic`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `super`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `is_monolithic`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `super`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 66-68 — method `UnquantizedFusedMoEMethod.supports_eplb`
```python
    @property
    def supports_eplb(self) -> bool:
        return True
```
**EN:** This method defines `supports_eplb`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_eplb`。 它管理专家分发、融合内核或 MoE 特有的张量流。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `UnquantizedFusedMoEMethod` / [CN] 核心符号：`UnquantizedFusedMoEMethod`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `typing`, `torch`, `torch.nn.functional`, `torch.nn` / **外部依赖**: `collections.abc`, `typing`, `torch`, `torch.nn.functional`, `torch.nn`
- **Internal**: `vllm.envs`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.oracle.unquantized`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`, `vllm.model_executor.utils`, `vllm.platforms` / **内部依赖**: `vllm.envs`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.oracle.unquantized`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`, `vllm.model_executor.utils`, `vllm.platforms`
- **Runtime traits**: platform-aware dispatch, custom C++/CUDA ops, distributed collectives / **运行时特征**: platform-aware dispatch, custom C++/CUDA ops, distributed collectives
