# xpu_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/xpu_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-23 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8DynamicTensorSym,
    kFp8StaticTensorSym,
    kMxfp4Static,
    kMxfp8Dynamic,
    kMxfp8Static,
)
from vllm.platforms import current_platform
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 29-33 — function `prepare_fp8_moe_layer_for_xpu`
```python
def prepare_fp8_moe_layer_for_xpu(
    w13: torch.Tensor,
    w2: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    return w13.transpose(-1, -2).contiguous(), w2.transpose(-1, -2).contiguous()
```
**EN:** This function defines `prepare_fp8_moe_layer_for_xpu`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `w13`, `w2`. Key calls include `w13.transpose.contiguous`, `w2.transpose.contiguous`, `w13.transpose`, `w2.transpose`.
**CN:** 该函数定义 `prepare_fp8_moe_layer_for_xpu`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `w13`, `w2`。 关键调用包括 `w13.transpose.contiguous`, `w2.transpose.contiguous`, `w13.transpose`, `w2.transpose`。

### Lines 36-155 — class `XPUExperts`
```python
class XPUExperts(mk.FusedMoEExpertsModular):
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config,
            quant_config,
            max_num_tokens,
            num_dispatchers,
        )
# ... omitted for brevity ...
            is_mxfp8=self.is_mxfp8,
        )
```
**EN:** This class defines `XPUExperts`. It inherits from `mk.FusedMoEExpertsModular`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `apply`, `expects_unquantized_inputs`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`. Key calls include `super.__init__`, `current_platform.is_xpu`, `TopKWeightAndReduceNoOP`, `topk_ids.size`, `xpu_fused_moe`, `super`. It writes or updates `is_fp8`, `is_mxfp4`, `is_mxfp8`, `SUPPORTED_W_A`, `workspace1`, `workspace2`.
**CN:** 该类定义了 `XPUExperts`。 它继承自 `mk.FusedMoEExpertsModular`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `apply`, `expects_unquantized_inputs`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`。 关键调用包括 `super.__init__`, `current_platform.is_xpu`, `TopKWeightAndReduceNoOP`, `topk_ids.size`, `xpu_fused_moe`, `super`。 它会写入或更新 `is_fp8`, `is_mxfp4`, `is_mxfp8`, `SUPPORTED_W_A`, `workspace1`, `workspace2`。

### Lines 37-52 — method `XPUExperts.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config,
            quant_config,
            max_num_tokens,
            num_dispatchers,
        )
        self.is_fp8 = False
        self.is_mxfp4 = False
        self.is_mxfp8 = False
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`. Key calls include `super.__init__`, `super`. It writes or updates `is_fp8`, `is_mxfp4`, `is_mxfp8`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `is_fp8`, `is_mxfp4`, `is_mxfp8`。

### Lines 117-155 — method `XPUExperts.apply`
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
        topk = topk_ids.size(-1)
        xpu_fused_moe(
            hidden_states=hidden_states,
            w13=w1,
            w13_scales=self.w1_scale,
            w13_bias=self.w1_bias,
            w2=w2,
            w2_scales=self.w2_scale,
            w2_bias=self.w2_bias,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            n_experts_per_token=topk,
            activation=activation.value,
            num_experts=self.moe_config.num_local_experts,
            ep_rank=self.moe_config.ep_rank,
            ep_size=self.moe_config.ep_size,
            output=output,
            is_fp8=self.is_fp8,
            is_mxfp4=self.is_mxfp4,
            is_mxfp8=self.is_mxfp8,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `topk_ids.size`, `xpu_fused_moe`. It writes or updates `topk`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `topk_ids.size`, `xpu_fused_moe`。 它会写入或更新 `topk`。

### Lines 54-56 — method `XPUExperts.expects_unquantized_inputs`
```python
    @property
    def expects_unquantized_inputs(self) -> bool:
        return True
```
**EN:** This method defines `expects_unquantized_inputs`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `expects_unquantized_inputs`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 58-60 — method `XPUExperts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 95-96 — method `XPUExperts.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return True
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 98-99 — method `XPUExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 158-183 — class `XPUExpertsFp8`
```python
class XPUExpertsFp8(XPUExperts):
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config,
            quant_config,
            max_num_tokens,
            num_dispatchers,
        )
# ... omitted for brevity ...
        ]
        return (weight_key, activation_key) in SUPPORTED_W_A
```
**EN:** This class defines `XPUExpertsFp8`. It inherits from `XPUExperts`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`. Key calls include `super.__init__`, `super`. It writes or updates `is_fp8`, `SUPPORTED_W_A`.
**CN:** 该类定义了 `XPUExpertsFp8`。 它继承自 `XPUExperts`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `is_fp8`, `SUPPORTED_W_A`。

### Lines 159-172 — method `XPUExpertsFp8.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config,
            quant_config,
            max_num_tokens,
            num_dispatchers,
        )
        self.is_fp8 = True
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`. Key calls include `super.__init__`, `super`. It writes or updates `is_fp8`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `is_fp8`。

### Lines 186-212 — class `XPUExpertsMxfp8`
```python
class XPUExpertsMxfp8(XPUExpertsFp8):
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config,
            quant_config,
            max_num_tokens,
            num_dispatchers,
        )
# ... omitted for brevity ...
        ]
        return (weight_key, activation_key) in SUPPORTED_W_A
```
**EN:** This class defines `XPUExpertsMxfp8`. It inherits from `XPUExpertsFp8`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`. Key calls include `super.__init__`, `super`. It writes or updates `is_mxfp8`, `SUPPORTED_W_A`.
**CN:** 该类定义了 `XPUExpertsMxfp8`。 它继承自 `XPUExpertsFp8`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `is_mxfp8`, `SUPPORTED_W_A`。

### Lines 187-201 — method `XPUExpertsMxfp8.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config,
            quant_config,
            max_num_tokens,
            num_dispatchers,
        )
        assert quant_config.quant_dtype == "mxfp8"
        self.is_mxfp8 = True
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`. Key calls include `super.__init__`, `super`. It writes or updates `is_mxfp8`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `is_mxfp8`。

### Lines 215-239 — class `XPUExpertsMXFp4`
```python
class XPUExpertsMXFp4(XPUExperts):
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config,
            quant_config,
            max_num_tokens,
            num_dispatchers,
        )
# ... omitted for brevity ...
        ]
        return (weight_key, activation_key) in SUPPORTED_W_A
```
**EN:** This class defines `XPUExpertsMXFp4`. It inherits from `XPUExperts`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`. Key calls include `super.__init__`, `super`. It writes or updates `is_mxfp4`, `SUPPORTED_W_A`.
**CN:** 该类定义了 `XPUExpertsMXFp4`。 它继承自 `XPUExperts`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `is_mxfp4`, `SUPPORTED_W_A`。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `prepare_fp8_moe_layer_for_xpu`, `XPUExperts`, `XPUExpertsFp8`, `XPUExpertsMxfp8` / [CN] 核心符号：`prepare_fp8_moe_layer_for_xpu`, `XPUExperts`, `XPUExpertsFp8`, `XPUExpertsMxfp8`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
