# cutlass_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/cutlass_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: CUTLASS based Fused MoE kernels. / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-50 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import _custom_ops as ops
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import (
    MoEActivation,
    apply_moe_activation,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.moe_permute_unpermute import (
    moe_permute,
    moe_unpermute,
)
from vllm.model_executor.layers.fused_moe.prepare_finalize import (
    MoEPrepareAndFinalizeNoDPEPModular,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceDelegate,
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.fused_moe.utils import (
    _resize_cache,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8DynamicTensorSym,
    kFp8DynamicTokenSym,
    kFp8StaticChannelSym,
    kFp8StaticTensorSym,
    kMxfp4Dynamic,
    kMxfp4Static,
    kNvfp4Dynamic,
    kNvfp4Static,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    cutlass_group_gemm_supported,
)
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 53-264 — function `run_cutlass_moe_fp8`
```python
def run_cutlass_moe_fp8(
    output: torch.Tensor,
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_ids: torch.Tensor,
    activation: MoEActivation,
    global_num_experts: int,
    expert_map: torch.Tensor | None,
    w1_scale: torch.Tensor | None,
    w2_scale: torch.Tensor | None,
    a1q_scale: torch.Tensor | None,
    a2_scale: torch.Tensor | None,
    ab_strides1: torch.Tensor,
    ab_strides2: torch.Tensor,
    c_strides1: torch.Tensor,
    c_strides2: torch.Tensor,
    workspace13: torch.Tensor,
    workspace2: torch.Tensor,
    expert_num_tokens: torch.Tensor | None,
    out_dtype: torch.dtype,
    per_act_token: bool,
    per_out_ch: bool,
    use_batched_format: bool,
    topk_weights: torch.Tensor | None,
):
    a1q = hidden_states

    assert activation.is_gated, "Only gated activation is supported"
    assert w1_scale is not None
    assert w2_scale is not None
    assert w1.dtype == torch.float8_e4m3fn
    assert w2.dtype == torch.float8_e4m3fn
    assert a1q.size(-1) == w1.size(2), "Hidden size mismatch w1"
    assert w1.size(1) == w2.size(2) * 2, "Hidden size mismatch w2"
    assert (
        w1_scale.dim() == 1 or w1_scale.size(1) == 1 or w1_scale.shape[1] == w1.size(1)
    ), "W1 scale shape mismatch"
    assert (
        w2_scale.dim() == 1 or w2_scale.size(1) == 1 or w2_scale.shape[1] == w2.size(1)
    ), "W2 scale shape mismatch"
    assert w1.size(0) == w2.size(0), "Expert number mismatch"
    assert (
        a1q_scale is None
# ... omitted for brevity ...
            expert_first_token_offset=expert_first_token_offset,
        )
```
**EN:** This function defines `run_cutlass_moe_fp8`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_ids`, `activation`. Key calls include `a1q.size`, `topk_ids.size`, `w1.size`, `ops.cutlass_moe_mm`, `apply_moe_activation`, `ops.scaled_fp8_quant`. It writes or updates `a1q`, `M`, `padded_M`, `_`, `K`, `N`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `run_cutlass_moe_fp8`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_ids`, `activation`。 关键调用包括 `a1q.size`, `topk_ids.size`, `w1.size`, `ops.cutlass_moe_mm`, `apply_moe_activation`, `ops.scaled_fp8_quant`。 它会写入或更新 `a1q`, `M`, `padded_M`, `_`, `K`, `N`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 267-384 — class `CutlassExpertsFp8Base`
```python
class CutlassExpertsFp8Base(mk.FusedMoEExpertsModular):
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config=moe_config,
            quant_config=quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=num_dispatchers,
        )
# ... omitted for brevity ...
            topk_weights,
        )
```
**EN:** This class defines `CutlassExpertsFp8Base`. It inherits from `mk.FusedMoEExpertsModular`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `apply`, `finalize_weight_and_reduce_impl`. Key calls include `super.__init__`, `torch.full`, `cutlass_group_gemm_supported`, `TopKWeightAndReduceDelegate`, `run_cutlass_moe_fp8`, `self.activation_format`. It writes or updates `e`, `n`, `k`, `device`, `ab_strides1_c_strides2`, `ab_strides2`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `CutlassExpertsFp8Base`。 它继承自 `mk.FusedMoEExpertsModular`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `apply`, `finalize_weight_and_reduce_impl`。 关键调用包括 `super.__init__`, `torch.full`, `cutlass_group_gemm_supported`, `TopKWeightAndReduceDelegate`, `run_cutlass_moe_fp8`, `self.activation_format`。 它会写入或更新 `e`, `n`, `k`, `device`, `ab_strides1_c_strides2`, `ab_strides2`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 268-295 — method `CutlassExpertsFp8Base.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        super().__init__(
            moe_config=moe_config,
            quant_config=quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=num_dispatchers,
        )
        assert quant_config.use_fp8_w8a8

        e = moe_config.num_local_experts
        n = moe_config.intermediate_size_per_partition
        k = moe_config.hidden_dim
        device = moe_config.device
        ab_strides1_c_strides2 = torch.full((e,), k, device=device, dtype=torch.int64)
        ab_strides2 = torch.full((e,), n, device=device, dtype=torch.int64)
        c_strides1 = torch.full((e,), 2 * n, device=device, dtype=torch.int64)

        self.out_dtype = moe_config.in_dtype
        self.ab_strides1 = ab_strides1_c_strides2
        self.ab_strides2 = ab_strides2
        self.c_strides1 = c_strides1
        self.c_strides2 = ab_strides1_c_strides2
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`. Key calls include `super.__init__`, `torch.full`, `super`. It writes or updates `e`, `n`, `k`, `device`, `ab_strides1_c_strides2`, `ab_strides2`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`。 关键调用包括 `super.__init__`, `torch.full`, `super`。 它会写入或更新 `e`, `n`, `k`, `device`, `ab_strides1_c_strides2`, `ab_strides2`。

### Lines 329-384 — method `CutlassExpertsFp8Base.apply`
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
        assert self.w1_zp is None, "w1_zp is not supported in CUTLASS MoE"
        assert self.w2_zp is None, "w2_zp is not supported in CUTLASS MoE"

        expert_num_tokens = None
        if expert_tokens_meta is not None:
            expert_num_tokens = expert_tokens_meta.expert_num_tokens

        use_batched_format = (
            self.activation_format() == mk.FusedMoEActivationFormat.BatchedExperts
        )

        in_dtype = hidden_states.dtype
        run_cutlass_moe_fp8(
            output,
            hidden_states,
            w1,
            w2,
            topk_ids,
            activation,
            global_num_experts,
            expert_map,
            self.w1_scale,
            self.w2_scale,
            a1q_scale,
            a2_scale,
            self.ab_strides1,
# ... omitted for brevity ...
            topk_weights,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `run_cutlass_moe_fp8`, `self.activation_format`. It writes or updates `expert_num_tokens`, `use_batched_format`, `in_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `run_cutlass_moe_fp8`, `self.activation_format`。 它会写入或更新 `expert_num_tokens`, `use_batched_format`, `in_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 325-327 — method `CutlassExpertsFp8Base.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        # Let PrepareAndFinalize::finalize() decide the impl.
        return TopKWeightAndReduceDelegate()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceDelegate`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceDelegate`。

### Lines 387-431 — class `CutlassExpertsFp8`
```python
class CutlassExpertsFp8(CutlassExpertsFp8Base):
    """CUTLASS FP8 fused MoE expert implementation."""

    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard

    @staticmethod
    def _supports_parallel_config(moe_parallel_config: FusedMoEParallelConfig) -> bool:
        # CutlassExpertsFp8 does not support expert map, which is
        # needed for STANDARD activation format kernels in DP/EP mode.
        # Note that the BATCHED activation format does not use
        # the expert map for identifying experts.
        return not (
# ... omitted for brevity ...
        output = (M, K)
        return (workspace1, workspace2, output)
```
**EN:** This class defines `CutlassExpertsFp8`. It inherits from `CutlassExpertsFp8Base`. CUTLASS FP8 fused MoE expert implementation. Important methods include `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_dtype`, `workspace_shapes`. Key calls include `TopKWeightAndReduceNoOP`, `self.adjust_N_for_activation`, `max`. It writes or updates `activation_out_dim`, `workspace1`, `workspace2`, `output`.
**CN:** 该类定义了 `CutlassExpertsFp8`。 它继承自 `CutlassExpertsFp8Base`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_dtype`, `workspace_shapes`。 关键调用包括 `TopKWeightAndReduceNoOP`, `self.adjust_N_for_activation`, `max`。 它会写入或更新 `activation_out_dim`, `workspace1`, `workspace2`, `output`。

### Lines 390-392 — method `CutlassExpertsFp8.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 406-407 — method `CutlassExpertsFp8.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return False
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 409-411 — method `CutlassExpertsFp8.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        # topk weights and reduction are fused in moe_unpermute cuda kernel
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 413-414 — method `CutlassExpertsFp8.workspace_dtype`
```python
    def workspace_dtype(self, act_dtype: torch.dtype) -> torch.dtype:
        return self.out_dtype if self.out_dtype is not None else act_dtype
```
**EN:** This method defines `workspace_dtype`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `act_dtype`.
**CN:** 该方法定义 `workspace_dtype`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `act_dtype`。

### Lines 416-431 — method `CutlassExpertsFp8.workspace_shapes`
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
        workspace1 = (M * topk, max(N, K))
        workspace2 = (M * topk, max(activation_out_dim, K))
        output = (M, K)
        return (workspace1, workspace2, output)
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. Key calls include `self.adjust_N_for_activation`, `max`. It writes or updates `activation_out_dim`, `workspace1`, `workspace2`, `output`.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 关键调用包括 `self.adjust_N_for_activation`, `max`。 它会写入或更新 `activation_out_dim`, `workspace1`, `workspace2`, `output`。

### Lines 434-476 — class `CutlassBatchedExpertsFp8`
```python
class CutlassBatchedExpertsFp8(CutlassExpertsFp8Base):
    """Batched CUTLASS FP8 fused MoE expert implementation."""

    @staticmethod
    def _supports_parallel_config(moe_parallel_config: FusedMoEParallelConfig) -> bool:
        # BATCHED activation format works with EP because
        # expert_map is not used to identify experts (the
        # info is encoded/managed by the P/F logic).
        return True

    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.BatchedExperts

# ... omitted for brevity ...
        output = (experts_per_worker, M, K)
        return (workspace1, workspace2, output)
```
**EN:** This class defines `CutlassBatchedExpertsFp8`. It inherits from `CutlassExpertsFp8Base`. Batched CUTLASS FP8 fused MoE expert implementation. Important methods include `activation_format`, `supports_expert_map`, `workspace_dtype`, `workspace_shapes`. Key calls include `self.adjust_N_for_activation`, `max`. It writes or updates `num_dp`, `experts_per_worker`, `activation_out_dim`, `workspace1`, `workspace2`, `output`.
**CN:** 该类定义了 `CutlassBatchedExpertsFp8`。 它继承自 `CutlassExpertsFp8Base`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `activation_format`, `supports_expert_map`, `workspace_dtype`, `workspace_shapes`。 关键调用包括 `self.adjust_N_for_activation`, `max`。 它会写入或更新 `num_dp`, `experts_per_worker`, `activation_out_dim`, `workspace1`, `workspace2`, `output`。

### Lines 444-446 — method `CutlassBatchedExpertsFp8.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.BatchedExperts
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `run_cutlass_moe_fp8`, `CutlassExpertsFp8Base`, `CutlassExpertsFp8`, `CutlassBatchedExpertsFp8` / [CN] 核心符号：`run_cutlass_moe_fp8`, `CutlassExpertsFp8Base`, `CutlassExpertsFp8`, `CutlassBatchedExpertsFp8`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.moe_permute_unpermute`, `vllm.model_executor.layers.fused_moe.prepare_finalize`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.moe_permute_unpermute`, `vllm.model_executor.layers.fused_moe.prepare_finalize`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
