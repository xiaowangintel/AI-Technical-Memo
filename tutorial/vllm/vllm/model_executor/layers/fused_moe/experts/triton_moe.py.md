# triton_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/triton_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Triton-based MoE expert implementations. / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-49 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import _custom_ops as ops
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin import (
    LoRAExpertsMixin,
)
from vllm.model_executor.layers.fused_moe.fused_moe import (
    _prepare_expert_assignment,
    invoke_fused_moe_triton_kernel,
    invoke_fused_moe_wna16_triton_kernel,
    try_get_optimal_moe_config,
)
from vllm.model_executor.layers.fused_moe.moe_align_block_size import (
    moe_align_block_size,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.fused_moe.utils import (
    _resize_cache,
    moe_kernel_quantize_input,
)
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    is_deep_gemm_e8m0_used,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8Dynamic128Sym,
    kFp8DynamicTensorSym,
    kFp8DynamicTokenSym,
    kFp8Static128BlockSym,
    kFp8StaticChannelSym,
    kFp8StaticTensorSym,
    kInt8DynamicTokenSym,
    kInt8StaticChannelSym,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 52-366 — class `TritonExperts`
```python
class TritonExperts(LoRAExpertsMixin, mk.FusedMoEExpertsModular):
    """Triton-based fused MoE expert implementation."""

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        # Whether quantized MOE runs natively, or through
        # higher-precision + activation QDQ.
        self.quantization_emulation = False
        super().__init__(moe_config, quant_config)

    @staticmethod
# ... omitted for brevity ...
    def moe_sum(self, input: torch.Tensor, output: torch.Tensor) -> None:
        ops.moe_sum(input, output)
```
**EN:** This class defines `TritonExperts`. It inherits from `LoRAExpertsMixin`, `mk.FusedMoEExpertsModular`. Triton-based fused MoE expert implementation. Important methods include `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`. Key calls include `super.__init__`, `current_platform.supports_fp8`, `TopKWeightAndReduceNoOP`, `self.adjust_N_for_activation`, `hidden_states.is_contiguous`, `self.moe_problem_size`. It writes or updates `quantization_emulation`, `device_supports_int8`, `supported`, `activation_out_dim`, `workspace1`, `workspace2`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `TritonExperts`。 它继承自 `LoRAExpertsMixin`, `mk.FusedMoEExpertsModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`。 关键调用包括 `super.__init__`, `current_platform.supports_fp8`, `TopKWeightAndReduceNoOP`, `self.adjust_N_for_activation`, `hidden_states.is_contiguous`, `self.moe_problem_size`。 它会写入或更新 `quantization_emulation`, `device_supports_int8`, `supported`, `activation_out_dim`, `workspace1`, `workspace2`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 55-63 — method `TritonExperts.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        # Whether quantized MOE runs natively, or through
        # higher-precision + activation QDQ.
        self.quantization_emulation = False
        super().__init__(moe_config, quant_config)
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `super`. It writes or updates `quantization_emulation`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `quantization_emulation`。

### Lines 149-363 — method `TritonExperts.apply`
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
        # Check constraints.
        if self.quant_config.use_int4_w4a16:
            assert hidden_states.size(-1) // 2 == w1.size(2), "Hidden size mismatch"
        else:
            assert hidden_states.size(-1) == w1.size(2), (
                f"Hidden size mismatch {hidden_states.size(-1)} != {w1.size(2)}"
            )

        assert hidden_states.is_contiguous(), "Hidden_states must be contiguous"
        assert hidden_states.dim() == 2
        assert w1.stride(-1) == 1, "Stride of last dimension must be 1"
        assert w2.stride(-1) == 1, "Stride of last dimension must be 1"
        assert hidden_states.dtype in [
            torch.float32,
            torch.float16,
            torch.bfloat16,
            torch.float8_e4m3fn,
            torch.float8_e4m3fnuz,
        ]

        E, num_tokens, N, K, top_k_num = self.moe_problem_size(
            hidden_states, w1, w2, topk_ids
        )

        if global_num_experts == -1:
            global_num_experts = E
# ... omitted for brevity ...
        # separate function is required for MoE + LoRA
        self.moe_sum(intermediate_cache3, output)
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `hidden_states.is_contiguous`, `self.moe_problem_size`, `try_get_optimal_moe_config`, `_resize_cache`, `self.adjust_N_for_activation`, `_prepare_expert_assignment`. It writes or updates `E`, `num_tokens`, `N`, `K`, `top_k_num`, `config`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `hidden_states.is_contiguous`, `self.moe_problem_size`, `try_get_optimal_moe_config`, `_resize_cache`, `self.adjust_N_for_activation`, `_prepare_expert_assignment`。 它会写入或更新 `E`, `num_tokens`, `N`, `K`, `top_k_num`, `config`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 65-67 — method `TritonExperts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 126-127 — method `TritonExperts.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return True
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 129-130 — method `TritonExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 132-147 — method `TritonExperts.workspace_shapes`
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
        activation_out_dim = self.adjust_N_for_activation(N, activation)
        workspace1 = (M, topk, max(activation_out_dim, K))
        workspace2 = (M, topk, max(N, K))
        output = (M, K)
        return (workspace1, workspace2, output)
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. Key calls include `self.adjust_N_for_activation`, `max`. It writes or updates `activation_out_dim`, `workspace1`, `workspace2`, `output`.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 关键调用包括 `self.adjust_N_for_activation`, `max`。 它会写入或更新 `activation_out_dim`, `workspace1`, `workspace2`, `output`。

### Lines 369-541 — class `TritonWNA16Experts`
```python
class TritonWNA16Experts(TritonExperts):
    @staticmethod
    def _supports_current_device() -> bool:
        raise NotImplementedError(
            "TritonWNA16Experts is not yet used by an Oracle. "
            "This method should not be called."
        )

    @staticmethod
    def _supports_no_act_and_mul() -> bool:
        raise NotImplementedError(
            "TritonWNA16Experts is not yet used by an Oracle. "
            "This method should not be called."
        )
# ... omitted for brevity ...
        # separate function is required for MoE + LoRA
        self.moe_sum(intermediate_cache3, output)
```
**EN:** This class defines `TritonWNA16Experts`. It inherits from `TritonExperts`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `apply`. Key calls include `NotImplementedError`, `hidden_states.is_contiguous`, `self.moe_problem_size`, `try_get_optimal_moe_config`, `_resize_cache`, `self.adjust_N_for_activation`. It writes or updates `E`, `num_tokens`, `N`, `K`, `top_k_num`, `config`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `TritonWNA16Experts`。 它继承自 `TritonExperts`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `apply`。 关键调用包括 `NotImplementedError`, `hidden_states.is_contiguous`, `self.moe_problem_size`, `try_get_optimal_moe_config`, `_resize_cache`, `self.adjust_N_for_activation`。 它会写入或更新 `E`, `num_tokens`, `N`, `K`, `top_k_num`, `config`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 408-541 — method `TritonWNA16Experts.apply`
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
        # Check constraints.
        if self.quant_config.use_int4_w4a16:
            assert hidden_states.size(-1) // 2 == w1.size(2), "Hidden size mismatch"
        else:
            assert hidden_states.size(-1) == w1.size(2), (
                f"Hidden size mismatch {hidden_states.size(-1)} != {w1.size(2)}"
            )

        assert hidden_states.is_contiguous(), "Hidden_states must be contiguous"
        assert hidden_states.dim() == 2
        assert w1.stride(-1) == 1, "Stride of last dimension must be 1"
        assert w2.stride(-1) == 1, "Stride of last dimension must be 1"
        assert hidden_states.dtype in [
            torch.float32,
            torch.float16,
            torch.bfloat16,
            torch.float8_e4m3fn,
            torch.float8_e4m3fnuz,
        ]

        E, num_tokens, N, K, top_k_num = self.moe_problem_size(
            hidden_states, w1, w2, topk_ids
        )

        if global_num_experts == -1:
            global_num_experts = E
# ... omitted for brevity ...
        # separate function is required for MoE + LoRA
        self.moe_sum(intermediate_cache3, output)
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `hidden_states.is_contiguous`, `self.moe_problem_size`, `try_get_optimal_moe_config`, `_resize_cache`, `self.adjust_N_for_activation`, `moe_align_block_size`. It writes or updates `E`, `num_tokens`, `N`, `K`, `top_k_num`, `config`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `hidden_states.is_contiguous`, `self.moe_problem_size`, `try_get_optimal_moe_config`, `_resize_cache`, `self.adjust_N_for_activation`, `moe_align_block_size`。 它会写入或更新 `E`, `num_tokens`, `N`, `K`, `top_k_num`, `config`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `TritonExperts`, `TritonWNA16Experts` / [CN] 核心符号：`TritonExperts`, `TritonWNA16Experts`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.fp8_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.fp8_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
