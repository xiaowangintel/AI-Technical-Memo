# fused_moe_method_base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/fused_moe_method_base.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-27 — imports and setup
```python
from abc import abstractmethod
from typing import TYPE_CHECKING

import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.modular_kernel import (
    FusedMoEExpertsModular,
    FusedMoEPrepareAndFinalizeModular,
)
from vllm.model_executor.layers.quantization.base_config import (
    QuantizeMethodBase,
)

if TYPE_CHECKING:
    from vllm.model_executor.layers.fused_moe.runner.shared_experts import SharedExperts

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`. The body uses conditional branches to cover different runtime cases.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 30-181 — class `FusedMoEMethodBase`
```python
class FusedMoEMethodBase(QuantizeMethodBase):
    def __init__(self, moe: FusedMoEConfig):
        super().__init__()
        self.moe: FusedMoEConfig = moe
        self.moe_quant_config: FusedMoEQuantConfig | None = None
        self.moe_kernel: mk.FusedMoEKernel | None = None

    @property
    def supports_internal_mk(self) -> bool:
        # NOTE(rob): temporary attribute to indicate support for
        # completed migration to the new internal MK interface.
        return self.moe_kernel is not None

    @property
# ... omitted for brevity ...
    ) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This class defines `FusedMoEMethodBase`. It inherits from `QuantizeMethodBase`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `apply`, `supports_internal_mk`, `mk_can_overlap_shared_experts`, `create_weights`, `uses_weight_scale_2_pattern`. Key calls include `super.__init__`, `maybe_make_prepare_finalize`, `ValueError`, `maybe_roundup_layer_hidden_size`, `isinstance`, `self.moe_kernel.prepare_finalize.topk_indices_dtype`. It writes or updates `moe`, `moe_quant_config`, `moe_kernel`, `pf`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `FusedMoEMethodBase`。 它继承自 `QuantizeMethodBase`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `apply`, `supports_internal_mk`, `mk_can_overlap_shared_experts`, `create_weights`, `uses_weight_scale_2_pattern`。 关键调用包括 `super.__init__`, `maybe_make_prepare_finalize`, `ValueError`, `maybe_roundup_layer_hidden_size`, `isinstance`, `self.moe_kernel.prepare_finalize.topk_indices_dtype`。 它会写入或更新 `moe`, `moe_quant_config`, `moe_kernel`, `pf`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 31-35 — method `FusedMoEMethodBase.__init__`
```python
    def __init__(self, moe: FusedMoEConfig):
        super().__init__()
        self.moe: FusedMoEConfig = moe
        self.moe_quant_config: FusedMoEQuantConfig | None = None
        self.moe_kernel: mk.FusedMoEKernel | None = None
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe`. Key calls include `super.__init__`, `super`. It writes or updates `moe`, `moe_quant_config`, `moe_kernel`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `moe`, `moe_quant_config`, `moe_kernel`。

### Lines 163-172 — method `FusedMoEMethodBase.apply`
```python
    def apply(
        self,
        layer: "RoutedExperts",  # type: ignore[name-defined] # noqa: F821
        x: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        shared_experts: "SharedExperts | None",
        shared_experts_input: torch.Tensor | None,
    ) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`。

### Lines 37-41 — method `FusedMoEMethodBase.supports_internal_mk`
```python
    @property
    def supports_internal_mk(self) -> bool:
        # NOTE(rob): temporary attribute to indicate support for
        # completed migration to the new internal MK interface.
        return self.moe_kernel is not None
```
**EN:** This method defines `supports_internal_mk`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_internal_mk`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 43-49 — method `FusedMoEMethodBase.mk_can_overlap_shared_experts`
```python
    @property
    def mk_can_overlap_shared_experts(self) -> bool:
        # NOTE(rob): temporary attribute to indicate support for
        # completed migration to the new internal MK interface.
        return (
            self.moe_kernel is not None and self.moe_kernel.can_overlap_shared_experts
        )
```
**EN:** This method defines `mk_can_overlap_shared_experts`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `mk_can_overlap_shared_experts`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 51-61 — method `FusedMoEMethodBase.create_weights`
```python
    @abstractmethod
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        raise NotImplementedError
```
**EN:** This method defines `create_weights`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`.
**CN:** 该方法定义 `create_weights`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`。

### Lines 63-71 — method `FusedMoEMethodBase.uses_weight_scale_2_pattern`
```python
    def uses_weight_scale_2_pattern(self) -> bool:
        """
        Returns True if this quantization method uses 'weight_scale_2' pattern
        for per-tensor weight scales (e.g., FP4 variants), False otherwise.

        This method should be overridden by subclasses that use the
        'weight_scale_2' pattern instead of the standard 'weight_scale' pattern.
        """
        return False
```
**EN:** This method defines `uses_weight_scale_2_pattern`. Returns True if this quantization method uses 'weight_scale_2' pattern for per-tensor weight scales (e.g., FP4 variants), False otherwise.
**CN:** 该方法定义 `uses_weight_scale_2_pattern`。 该函数/方法的文档字符串直接说明了它的职责。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `FusedMoEMethodBase` / [CN] 核心符号：`FusedMoEMethodBase`

## Dependencies / 依赖关系
- **External**: `abc`, `typing`, `torch` / **外部依赖**: `abc`, `typing`, `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.fused_moe.runner.shared_experts` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
