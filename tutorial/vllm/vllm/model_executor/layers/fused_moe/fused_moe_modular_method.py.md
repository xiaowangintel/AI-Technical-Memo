# fused_moe_modular_method.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/fused_moe_modular_method.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-23 — imports and setup
```python
import torch

from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.fused_moe_method_base import (
    FusedMoEMethodBase,
)
from vllm.model_executor.layers.fused_moe.modular_kernel import (
    FusedMoEKernel,
    FusedMoEPrepareAndFinalizeModular,
)
from vllm.model_executor.layers.fused_moe.runner.shared_experts import (
    SharedExperts,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 27-107 — class `FusedMoEModularMethod`
```python
@CustomOp.register("modular_fused_moe")
class FusedMoEModularMethod(FusedMoEMethodBase, CustomOp):
    # --8<-- [end:modular_fused_moe]

    def __init__(
        self, old_quant_method: FusedMoEMethodBase, moe_kernel: FusedMoEKernel
    ):
        super().__init__(old_quant_method.moe)
        self.moe_quant_config = old_quant_method.moe_quant_config
        self.moe_kernel = moe_kernel
        self.disable_expert_map = getattr(
            old_quant_method,
            "disable_expert_map",
            not self.moe_kernel.supports_expert_map(),
# ... omitted for brevity ...
            shared_experts_input=shared_experts_input,
        )
```
**EN:** This class defines `FusedMoEModularMethod`. It inherits from `FusedMoEMethodBase`, `CustomOp`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `apply`, `make`, `supports_eplb`, `method_name`, `create_weights`. Key calls include `CustomOp.register`, `super.__init__`, `getattr`, `logger.debug`, `FusedMoEModularMethod`, `self.moe_kernel.apply`. It writes or updates `moe_quant_config`, `moe_kernel`, `disable_expert_map`, `old_quant_method`.
**CN:** 该类定义了 `FusedMoEModularMethod`。 它继承自 `FusedMoEMethodBase`, `CustomOp`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `apply`, `make`, `supports_eplb`, `method_name`, `create_weights`。 关键调用包括 `CustomOp.register`, `super.__init__`, `getattr`, `logger.debug`, `FusedMoEModularMethod`, `self.moe_kernel.apply`。 它会写入或更新 `moe_quant_config`, `moe_kernel`, `disable_expert_map`, `old_quant_method`。

### Lines 31-43 — method `FusedMoEModularMethod.__init__`
```python
    def __init__(
        self, old_quant_method: FusedMoEMethodBase, moe_kernel: FusedMoEKernel
    ):
        super().__init__(old_quant_method.moe)
        self.moe_quant_config = old_quant_method.moe_quant_config
        self.moe_kernel = moe_kernel
        self.disable_expert_map = getattr(
            old_quant_method,
            "disable_expert_map",
            not self.moe_kernel.supports_expert_map(),
        )
        self.old_quant_method = old_quant_method
        logger.debug("Swapping out %s", self.old_quant_method.__class__.__name__)
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `old_quant_method`, `moe_kernel`. Key calls include `super.__init__`, `getattr`, `logger.debug`, `super`, `self.moe_kernel.supports_expert_map`. It writes or updates `moe_quant_config`, `moe_kernel`, `disable_expert_map`, `old_quant_method`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `old_quant_method`, `moe_kernel`。 关键调用包括 `super.__init__`, `getattr`, `logger.debug`, `super`, `self.moe_kernel.supports_expert_map`。 它会写入或更新 `moe_quant_config`, `moe_kernel`, `disable_expert_map`, `old_quant_method`。

### Lines 85-107 — method `FusedMoEModularMethod.apply`
```python
    def apply(
        self,
        layer: "RoutedExperts",  # type: ignore[name-defined] # noqa: F821
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
            global_num_experts=layer.global_num_experts,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            expert_map=None if self.disable_expert_map else layer.expert_map,
            shared_experts=shared_experts,
            shared_experts_input=shared_experts_input,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`. Key calls include `self.moe_kernel.apply`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`。 关键调用包括 `self.moe_kernel.apply`。

### Lines 45-59 — method `FusedMoEModularMethod.make`
```python
    @staticmethod
    def make(
        moe_layer: torch.nn.Module,
        old_quant_method: FusedMoEMethodBase,
        prepare_finalize: FusedMoEPrepareAndFinalizeModular,
        inplace: bool = False,
    ) -> "FusedMoEModularMethod":
        return FusedMoEModularMethod(
            old_quant_method,
            FusedMoEKernel(
                prepare_finalize,
                old_quant_method.select_gemm_impl(prepare_finalize, moe_layer),
                inplace=inplace,
            ),
        )
```
**EN:** This method defines `make`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `moe_layer`, `old_quant_method`, `prepare_finalize`, `inplace`. Key calls include `FusedMoEModularMethod`, `FusedMoEKernel`, `old_quant_method.select_gemm_impl`.
**CN:** 该方法定义 `make`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `moe_layer`, `old_quant_method`, `prepare_finalize`, `inplace`。 关键调用包括 `FusedMoEModularMethod`, `FusedMoEKernel`, `old_quant_method.select_gemm_impl`。

### Lines 61-63 — method `FusedMoEModularMethod.supports_eplb`
```python
    @property
    def supports_eplb(self) -> bool:
        return self.old_quant_method.supports_eplb
```
**EN:** This method defines `supports_eplb`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_eplb`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 65-67 — method `FusedMoEModularMethod.method_name`
```python
    @property
    def method_name(self) -> str:
        return self.old_quant_method.method_name
```
**EN:** This method defines `method_name`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `method_name`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 69-78 — method `FusedMoEModularMethod.create_weights`
```python
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

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `FusedMoEModularMethod` / [CN] 核心符号：`FusedMoEModularMethod`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.runner.shared_experts` / **内部依赖**: `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`
