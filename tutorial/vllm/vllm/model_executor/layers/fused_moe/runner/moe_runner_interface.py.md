# moe_runner_interface.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/runner/moe_runner_interface.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE runner orchestration and shared-expert execution; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 执行器编排与共享专家执行；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-13 — imports and setup
```python
from abc import ABC, abstractmethod

import torch

from vllm.model_executor.custom_op import PluggableLayer
from vllm.model_executor.layers.fused_moe.fused_moe_method_base import (
    FusedMoEMethodBase,
)
from vllm.model_executor.layers.fused_moe.runner.shared_experts import (
    SharedExperts,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 16-46 — class `MoERunnerInterface`
```python
class MoERunnerInterface(PluggableLayer, ABC):
    """
    Abstract base class for Mixture of Experts (MoE) runners.

    This class defines the interface that all MoE runner implementations must follow.
    MoE runners are responsible for executing the forward pass of MoE layers, handling
    expert routing, and managing tensor parallel operations.
    """

    @abstractmethod
    def forward(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
# ... omitted for brevity ...
    def _replace_quant_method(self, quant_method: FusedMoEMethodBase):
        raise NotImplementedError
```
**EN:** This class defines `MoERunnerInterface`. It inherits from `PluggableLayer`, `ABC`. Abstract base class for Mixture of Experts (MoE) runners. Important methods include `forward`, `is_internal_router`, `shared_experts`, `_replace_quant_method`.
**CN:** 该类定义了 `MoERunnerInterface`。 它继承自 `PluggableLayer`, `ABC`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `forward`, `is_internal_router`, `shared_experts`, `_replace_quant_method`。

### Lines 25-32 — method `MoERunnerInterface.forward`
```python
    @abstractmethod
    def forward(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        input_ids: torch.Tensor | None = None,
    ) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `router_logits`, `input_ids`.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `router_logits`, `input_ids`。

### Lines 34-36 — method `MoERunnerInterface.is_internal_router`
```python
    @abstractmethod
    def is_internal_router(self) -> bool:
        raise NotImplementedError
```
**EN:** This method defines `is_internal_router`. It computes routing scores or expert-selection behavior for MoE execution.
**CN:** 该方法定义 `is_internal_router`。 它为 MoE 执行计算路由得分或专家选择行为。

### Lines 38-41 — method `MoERunnerInterface.shared_experts`
```python
    @property
    @abstractmethod
    def shared_experts(self) -> SharedExperts | None:
        raise NotImplementedError
```
**EN:** This method defines `shared_experts`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `shared_experts`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 44-46 — method `MoERunnerInterface._replace_quant_method`
```python
    @abstractmethod
    def _replace_quant_method(self, quant_method: FusedMoEMethodBase):
        raise NotImplementedError
```
**EN:** This method defines `_replace_quant_method`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `quant_method`.
**CN:** 该方法定义 `_replace_quant_method`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `quant_method`。

## Key Concepts / 关键概念
- [EN] Moe runner orchestration and shared-expert execution / [CN] MoE 执行器编排与共享专家执行
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `MoERunnerInterface` / [CN] 核心符号：`MoERunnerInterface`

## Dependencies / 依赖关系
- **External**: `abc`, `torch` / **外部依赖**: `abc`, `torch`
- **Internal**: `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.runner.shared_experts` / **内部依赖**: `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
