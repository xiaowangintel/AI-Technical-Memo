# no_dp_ep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/prepare_finalize/no_dp_ep.py`
- **Repository**: vllm-project/vllm
- **Purpose**: distributed MoE preparation/finalization logic; fused Mixture-of-Experts routing, kernels, and runtime helpers / 分布式 MoE 的准备/收尾逻辑；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-11 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.model_executor.layers.fused_moe.config import FusedMoEQuantConfig
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceContiguous,
    TopKWeightAndReduceDelegate,
)
from vllm.model_executor.layers.fused_moe.utils import moe_kernel_quantize_input
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 14-37 — function `_quantize_input`
```python
def _quantize_input(
    a1: torch.Tensor,
    quant_config: FusedMoEQuantConfig,
    defer_input_quant: bool = False,
) -> tuple[torch.Tensor, torch.Tensor | None]:
    # Defer input quant to moe kernel for backends (e.g. AITER, FI)
    # which use a single kernel call for quant + experts.
    if defer_input_quant:
        return a1, None

    input_sf = (
        quant_config.a1_gscale if quant_config.use_nvfp4_w4a4 else quant_config.a1_scale
    )
    a1q, a1q_scale = moe_kernel_quantize_input(
        a1,
        input_sf,
        quant_dtype=quant_config.quant_dtype,
        per_act_token_quant=quant_config.per_act_token_quant,
        block_shape=quant_config.block_shape,
        is_scale_swizzled=quant_config.is_scale_swizzled,
        mx_alignment=quant_config.mx_alignment,
    )

    return a1q, a1q_scale
```
**EN:** This function defines `_quantize_input`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a1`, `quant_config`, `defer_input_quant`. Key calls include `moe_kernel_quantize_input`. It writes or updates `input_sf`, `a1q`, `a1q_scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_quantize_input`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a1`, `quant_config`, `defer_input_quant`。 关键调用包括 `moe_kernel_quantize_input`。 它会写入或更新 `input_sf`, `a1q`, `a1q_scale`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 40-98 — class `MoEPrepareAndFinalizeNoDPEPModular`
```python
class MoEPrepareAndFinalizeNoDPEPModular(mk.FusedMoEPrepareAndFinalizeModular):
    @property
    def activation_format(self) -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard

    def max_num_tokens_per_rank(self) -> int | None:
        return None

    def topk_indices_dtype(self) -> torch.dtype | None:
        return None

    def num_dispatchers(self) -> int:
        return 1

# ... omitted for brevity ...
            apply_router_weight_on_input=apply_router_weight_on_input,
        )
```
**EN:** This class defines `MoEPrepareAndFinalizeNoDPEPModular`. It inherits from `mk.FusedMoEPrepareAndFinalizeModular`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`, `num_dispatchers`. Key calls include `_quantize_input`, `isinstance`, `weight_and_reduce_impl.apply`, `topk_ids.size`, `TopKWeightAndReduceContiguous`, `topk_weights.to`. It writes or updates `a1q`, `a1q_scale`, `topk`, `a1`, `weight_and_reduce_impl`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `MoEPrepareAndFinalizeNoDPEPModular`。 它继承自 `mk.FusedMoEPrepareAndFinalizeModular`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`, `num_dispatchers`。 关键调用包括 `_quantize_input`, `isinstance`, `weight_and_reduce_impl.apply`, `topk_ids.size`, `TopKWeightAndReduceContiguous`, `topk_weights.to`。 它会写入或更新 `a1q`, `a1q_scale`, `topk`, `a1`, `weight_and_reduce_impl`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 57-79 — method `MoEPrepareAndFinalizeNoDPEPModular.prepare`
```python
    def prepare(
        self,
        a1: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        num_experts: int,
        expert_map: torch.Tensor | None,
        apply_router_weight_on_input: bool,
        quant_config: FusedMoEQuantConfig,
        defer_input_quant: bool = False,
    ) -> mk.PrepareResultType:
        if apply_router_weight_on_input:
            topk = topk_ids.size(1)
            # TODO: this only works for topK=1, will need to update for topK>1
            assert topk == 1, (
                "apply_router_weight_on_input is only implemented for topk=1"
            )
            # Note: do not use inplace for shared experts overlap
            a1 = a1 * topk_weights.to(a1.dtype)

        a1q, a1q_scale = _quantize_input(a1, quant_config, defer_input_quant)

        return a1q, a1q_scale, None, None, None
```
**EN:** This method defines `prepare`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`. Key calls include `_quantize_input`, `topk_ids.size`, `topk_weights.to`. It writes or updates `a1q`, `a1q_scale`, `topk`, `a1`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `prepare`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`。 关键调用包括 `_quantize_input`, `topk_ids.size`, `topk_weights.to`。 它会写入或更新 `a1q`, `a1q_scale`, `topk`, `a1`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 81-98 — method `MoEPrepareAndFinalizeNoDPEPModular.finalize`
```python
    def finalize(
        self,
        output: torch.Tensor,
        fused_expert_output: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        apply_router_weight_on_input: bool,
        weight_and_reduce_impl: mk.TopKWeightAndReduce,
    ) -> None:
        if isinstance(weight_and_reduce_impl, TopKWeightAndReduceDelegate):
            weight_and_reduce_impl = TopKWeightAndReduceContiguous()
        weight_and_reduce_impl.apply(
            output=output,
            fused_expert_output=fused_expert_output,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            apply_router_weight_on_input=apply_router_weight_on_input,
        )
```
**EN:** This method defines `finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`. Key calls include `isinstance`, `weight_and_reduce_impl.apply`, `TopKWeightAndReduceContiguous`. It writes or updates `weight_and_reduce_impl`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`。 关键调用包括 `isinstance`, `weight_and_reduce_impl.apply`, `TopKWeightAndReduceContiguous`。 它会写入或更新 `weight_and_reduce_impl`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 41-43 — method `MoEPrepareAndFinalizeNoDPEPModular.activation_format`
```python
    @property
    def activation_format(self) -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 45-46 — method `MoEPrepareAndFinalizeNoDPEPModular.max_num_tokens_per_rank`
```python
    def max_num_tokens_per_rank(self) -> int | None:
        return None
```
**EN:** This method defines `max_num_tokens_per_rank`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `max_num_tokens_per_rank`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 48-49 — method `MoEPrepareAndFinalizeNoDPEPModular.topk_indices_dtype`
```python
    def topk_indices_dtype(self) -> torch.dtype | None:
        return None
```
**EN:** This method defines `topk_indices_dtype`. It computes routing scores or expert-selection behavior for MoE execution.
**CN:** 该方法定义 `topk_indices_dtype`。 它为 MoE 执行计算路由得分或专家选择行为。

### Lines 51-52 — method `MoEPrepareAndFinalizeNoDPEPModular.num_dispatchers`
```python
    def num_dispatchers(self) -> int:
        return 1
```
**EN:** This method defines `num_dispatchers`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `num_dispatchers`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 101-132 — class `MoEPrepareAndFinalizeNoDPEPMonolithic`
```python
class MoEPrepareAndFinalizeNoDPEPMonolithic(mk.FusedMoEPrepareAndFinalizeMonolithic):
    @property
    def activation_format(self) -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard

    def max_num_tokens_per_rank(self) -> int | None:
        return None

    def topk_indices_dtype(self) -> torch.dtype | None:
        return None

    def num_dispatchers(self) -> int:
        return 1

# ... omitted for brevity ...
    ) -> torch.Tensor:
        return fused_expert_output
```
**EN:** This class defines `MoEPrepareAndFinalizeNoDPEPMonolithic`. It inherits from `mk.FusedMoEPrepareAndFinalizeMonolithic`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`, `num_dispatchers`. Key calls include `_quantize_input`. It writes or updates `a1q`, `a1q_scale`.
**CN:** 该类定义了 `MoEPrepareAndFinalizeNoDPEPMonolithic`。 它继承自 `mk.FusedMoEPrepareAndFinalizeMonolithic`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`, `num_dispatchers`。 关键调用包括 `_quantize_input`。 它会写入或更新 `a1q`, `a1q_scale`。

### Lines 118-126 — method `MoEPrepareAndFinalizeNoDPEPMonolithic.prepare`
```python
    def prepare(
        self,
        a1: torch.Tensor,
        router_logits: torch.Tensor,
        quant_config: FusedMoEQuantConfig,
        defer_input_quant: bool = False,
    ) -> mk.PrepareMonolithicResultType:
        a1q, a1q_scale = _quantize_input(a1, quant_config, defer_input_quant)
        return a1q, a1q_scale, router_logits
```
**EN:** This method defines `prepare`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a1`, `router_logits`, `quant_config`, `defer_input_quant`. Key calls include `_quantize_input`. It writes or updates `a1q`, `a1q_scale`.
**CN:** 该方法定义 `prepare`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a1`, `router_logits`, `quant_config`, `defer_input_quant`。 关键调用包括 `_quantize_input`。 它会写入或更新 `a1q`, `a1q_scale`。

### Lines 128-132 — method `MoEPrepareAndFinalizeNoDPEPMonolithic.finalize`
```python
    def finalize(
        self,
        fused_expert_output: torch.Tensor,
    ) -> torch.Tensor:
        return fused_expert_output
```
**EN:** This method defines `finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `fused_expert_output`.
**CN:** 该方法定义 `finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `fused_expert_output`。

### Lines 102-104 — method `MoEPrepareAndFinalizeNoDPEPMonolithic.activation_format`
```python
    @property
    def activation_format(self) -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 106-107 — method `MoEPrepareAndFinalizeNoDPEPMonolithic.max_num_tokens_per_rank`
```python
    def max_num_tokens_per_rank(self) -> int | None:
        return None
```
**EN:** This method defines `max_num_tokens_per_rank`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `max_num_tokens_per_rank`。 它管理专家分发、融合内核或 MoE 特有的张量流。

## Key Concepts / 关键概念
- [EN] Distributed moe preparation/finalization logic / [CN] 分布式 MoE 的准备/收尾逻辑
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `_quantize_input`, `MoEPrepareAndFinalizeNoDPEPModular`, `MoEPrepareAndFinalizeNoDPEPMonolithic`, `make_moe_prepare_and_finalize_no_dp_ep` / [CN] 核心符号：`_quantize_input`, `MoEPrepareAndFinalizeNoDPEPModular`, `MoEPrepareAndFinalizeNoDPEPMonolithic`, `make_moe_prepare_and_finalize_no_dp_ep`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`
