# naive_dp_ep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/prepare_finalize/naive_dp_ep.py`
- **Repository**: vllm-project/vllm
- **Purpose**: distributed MoE preparation/finalization logic; fused Mixture-of-Experts routing, kernels, and runtime helpers / 分布式 MoE 的准备/收尾逻辑；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-13 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.distributed import get_ep_group
from vllm.model_executor.layers.fused_moe.config import FusedMoEQuantConfig
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceContiguous,
    TopKWeightAndReduceDelegate,
)
from vllm.model_executor.layers.fused_moe.utils import moe_kernel_quantize_input
from vllm.utils.flashinfer import nvfp4_block_scale_interleave
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 16-52 — function `_quantize_and_setup_dispatch`
```python
def _quantize_and_setup_dispatch(
    a1: torch.Tensor,
    quant_config: FusedMoEQuantConfig,
    defer_input_quant: bool = False,
) -> tuple[torch.Tensor, list[torch.Tensor] | None]:
    # Defer input quantization to the MoE kernel.
    if defer_input_quant:
        a1q = a1
        a1q_scale = None
    else:
        input_sf = (
            quant_config.a1_gscale
            if quant_config.use_nvfp4_w4a4
            else quant_config.a1_scale
        )

        # NOTE: swizzling pads the scales to multiple of 128
        # which makes the scales tensor different shape than
        # the hidden states, breaking the A2A kernel. So, we
        # delay the swizzling until after the A2A.
        a1q, a1q_scale = a1q, a1q_scale = moe_kernel_quantize_input(
            a1,
            input_sf,
            quant_dtype=quant_config.quant_dtype,
            per_act_token_quant=quant_config.per_act_token_quant,
            block_shape=quant_config.block_shape,
            is_scale_swizzled=False,
            mx_alignment=quant_config.mx_alignment,
        )

    # Skip gathering scales if we have static quantization
    # (the scale is a scalar, replicated on all ranks) or
    # if quantization is deferred.
    skip_gather_scales = a1q_scale is None or a1q_scale.ndim == 0
    scales = None if skip_gather_scales else [a1q_scale]

    return a1q, scales
```
**EN:** This function defines `_quantize_and_setup_dispatch`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a1`, `quant_config`, `defer_input_quant`. Key calls include `moe_kernel_quantize_input`. It writes or updates `skip_gather_scales`, `scales`, `a1q`, `a1q_scale`, `input_sf`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_quantize_and_setup_dispatch`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a1`, `quant_config`, `defer_input_quant`。 关键调用包括 `moe_kernel_quantize_input`。 它会写入或更新 `skip_gather_scales`, `scales`, `a1q`, `a1q_scale`, `input_sf`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 55-68 — function `_unwrap_scale_and_prepare_for_moe`
```python
def _unwrap_scale_and_prepare_for_moe(
    scales: list[torch.Tensor] | None,
    quant_config: FusedMoEQuantConfig,
) -> torch.Tensor:
    assert scales is not None and len(scales) == 1
    a1q_scale = scales[0]
    # Apply swizzling after a2a if the MoE kernel needs it.
    if quant_config.quant_dtype == "nvfp4" and quant_config.is_scale_swizzled:
        assert a1q_scale is not None
        if a1q_scale.element_size() == 1:
            a1q_scale = a1q_scale.view(torch.uint8)
        a1q_scale = nvfp4_block_scale_interleave(a1q_scale)

    return a1q_scale
```
**EN:** This function defines `_unwrap_scale_and_prepare_for_moe`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `scales`, `quant_config`. Key calls include `nvfp4_block_scale_interleave`, `len`, `a1q_scale.element_size`, `a1q_scale.view`. It writes or updates `a1q_scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_unwrap_scale_and_prepare_for_moe`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `scales`, `quant_config`。 关键调用包括 `nvfp4_block_scale_interleave`, `len`, `a1q_scale.element_size`, `a1q_scale.view`。 它会写入或更新 `a1q_scale`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 71-208 — class `MoEPrepareAndFinalizeNaiveDPEPModular`
```python
class MoEPrepareAndFinalizeNaiveDPEPModular(mk.FusedMoEPrepareAndFinalizeModular):
    """
    Naive Prepare/Finalize for Dp/Ep case for Modular Kernels.

    Uses Torch AR/RS or AR for dispatch/combine operations, applied
    to the topk weights and ids.
    """

    def __init__(
        self,
        is_sequence_parallel: bool = False,
        num_dispatchers: int = 1,
    ) -> None:
        super().__init__()
# ... omitted for brevity ...
            get_ep_group().combine(out, is_sequence_parallel=self.is_sequence_parallel)
        )
```
**EN:** This class defines `MoEPrepareAndFinalizeNaiveDPEPModular`. It inherits from `mk.FusedMoEPrepareAndFinalizeModular`. Naive Prepare/Finalize for Dp/Ep case for Modular Kernels. Important methods include `__init__`, `prepare`, `finalize`, `set_lora_context`, `activation_format`, `max_num_tokens_per_rank`. Key calls include `super.__init__`, `_quantize_and_setup_dispatch`, `get_ep_group.dispatch`, `isinstance`, `weight_and_reduce_impl.apply`, `output.copy_`. It writes or updates `is_sequence_parallel`, `_num_dispatchers`, `_lora_context`, `a1q`, `scales`, `lora_ctx`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `MoEPrepareAndFinalizeNaiveDPEPModular`。 它继承自 `mk.FusedMoEPrepareAndFinalizeModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `prepare`, `finalize`, `set_lora_context`, `activation_format`, `max_num_tokens_per_rank`。 关键调用包括 `super.__init__`, `_quantize_and_setup_dispatch`, `get_ep_group.dispatch`, `isinstance`, `weight_and_reduce_impl.apply`, `output.copy_`。 它会写入或更新 `is_sequence_parallel`, `_num_dispatchers`, `_lora_context`, `a1q`, `scales`, `lora_ctx`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 79-91 — method `MoEPrepareAndFinalizeNaiveDPEPModular.__init__`
```python
    def __init__(
        self,
        is_sequence_parallel: bool = False,
        num_dispatchers: int = 1,
    ) -> None:
        super().__init__()
        self.is_sequence_parallel = is_sequence_parallel
        self._num_dispatchers = num_dispatchers
        # Set by FusedMoEWithLoRA.set_mapping() when LoRA is active. When
        # present, prepare() dispatches the per-token LoRA mapping alongside
        # hidden_states and writes the gathered result back to the context so
        # experts can use the per-rank-local mapping.
        self._lora_context = None
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `is_sequence_parallel`, `num_dispatchers`. Key calls include `super.__init__`, `super`. It writes or updates `is_sequence_parallel`, `_num_dispatchers`, `_lora_context`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `is_sequence_parallel`, `num_dispatchers`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `is_sequence_parallel`, `_num_dispatchers`, `_lora_context`。

### Lines 112-184 — method `MoEPrepareAndFinalizeNaiveDPEPModular.prepare`
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
        """Quantize and Dispatch Topk Weights and Topk Ids."""

        if apply_router_weight_on_input:
            topk = topk_ids.size(1)
            assert topk == 1, (
                "apply_router_weight_on_input is only implemented for topk=1"
            )
            # Note: do not use inplace for shared experts overlap
            a1 = a1 * topk_weights.to(a1.dtype)

        a1q, scales = _quantize_and_setup_dispatch(a1, quant_config, defer_input_quant)

        # When LoRA is active, dispatch the per-token LoRA id along with
        # hidden_states so every rank receives the correct mapping for the
        # tokens it ends up processing. The punica_wrapper stores indices as
        # int64 but the moe_lora_align_block_size kernel expects int32, so
        # pull the pre-cast view from token_mapping_meta.
        lora_ctx = self._lora_context
        local_token_lora_mapping = None
        if lora_ctx is not None:
            local_token_lora_mapping = (
                lora_ctx.punica_wrapper.token_mapping_meta.token_lora_mapping[
                    : a1.shape[0]
                ]
            )

        extra_tensors: list[torch.Tensor] | None = None
        if scales is not None:
            extra_tensors = list(scales)
        if local_token_lora_mapping is not None:
            if extra_tensors is None:
                extra_tensors = []
            extra_tensors.append(local_token_lora_mapping)
# ... omitted for brevity ...

        return a1q, a1q_scale, None, topk_ids, topk_weights
```
**EN:** This method defines `prepare`. Quantize and Dispatch Topk Weights and Topk Ids. The main inputs are `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`. Key calls include `_quantize_and_setup_dispatch`, `get_ep_group.dispatch`, `topk_ids.size`, `list`, `extra_tensors.append`, `topk_weights.to`. It writes or updates `a1q`, `scales`, `lora_ctx`, `local_token_lora_mapping`, `extra_tensors`, `res`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `prepare`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`。 关键调用包括 `_quantize_and_setup_dispatch`, `get_ep_group.dispatch`, `topk_ids.size`, `list`, `extra_tensors.append`, `topk_weights.to`。 它会写入或更新 `a1q`, `scales`, `lora_ctx`, `local_token_lora_mapping`, `extra_tensors`, `res`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 186-208 — method `MoEPrepareAndFinalizeNaiveDPEPModular.finalize`
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

        out = weight_and_reduce_impl.apply(
            output=None,
            fused_expert_output=fused_expert_output,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            apply_router_weight_on_input=apply_router_weight_on_input,
        )

        output.copy_(
            get_ep_group().combine(out, is_sequence_parallel=self.is_sequence_parallel)
        )
```
**EN:** This method defines `finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`. Key calls include `isinstance`, `weight_and_reduce_impl.apply`, `output.copy_`, `TopKWeightAndReduceContiguous`, `get_ep_group.combine`, `get_ep_group`. It writes or updates `out`, `weight_and_reduce_impl`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`。 关键调用包括 `isinstance`, `weight_and_reduce_impl.apply`, `output.copy_`, `TopKWeightAndReduceContiguous`, `get_ep_group.combine`, `get_ep_group`。 它会写入或更新 `out`, `weight_and_reduce_impl`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 93-94 — method `MoEPrepareAndFinalizeNaiveDPEPModular.set_lora_context`
```python
    def set_lora_context(self, ctx) -> None:
        self._lora_context = ctx
```
**EN:** This method defines `set_lora_context`. It updates buffers, cached values, or configuration-dependent state. The main inputs are `ctx`. It writes or updates `_lora_context`.
**CN:** 该方法定义 `set_lora_context`。 它更新缓冲区、缓存值或依赖配置的状态。 其主要输入参数包括 `ctx`。 它会写入或更新 `_lora_context`。

### Lines 96-98 — method `MoEPrepareAndFinalizeNaiveDPEPModular.activation_format`
```python
    @property
    def activation_format(self) -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 100-101 — method `MoEPrepareAndFinalizeNaiveDPEPModular.max_num_tokens_per_rank`
```python
    def max_num_tokens_per_rank(self) -> int | None:
        return None
```
**EN:** This method defines `max_num_tokens_per_rank`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `max_num_tokens_per_rank`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 211-280 — class `MoEPrepareAndFinalizeNaiveDPEPMonolithic`
```python
class MoEPrepareAndFinalizeNaiveDPEPMonolithic(mk.FusedMoEPrepareAndFinalizeMonolithic):
    """
    Naive Prepare/Finalize for Dp/Ep case for Modular Kernels.

    Uses Torch AR/RS or AR for dispatch/combine operations, applied
    to the router logits (the MoE kernel runs the router internally).
    """

    def __init__(
        self,
        is_sequence_parallel: bool = False,
        num_dispatchers: int = 1,
    ) -> None:
        super().__init__()
# ... omitted for brevity ...
        )
        return out
```
**EN:** This class defines `MoEPrepareAndFinalizeNaiveDPEPMonolithic`. It inherits from `mk.FusedMoEPrepareAndFinalizeMonolithic`. Naive Prepare/Finalize for Dp/Ep case for Modular Kernels. Important methods include `__init__`, `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`. Key calls include `super.__init__`, `_quantize_and_setup_dispatch`, `get_ep_group.dispatch_router_logits`, `get_ep_group.combine`, `_unwrap_scale_and_prepare_for_moe`, `super`. It writes or updates `is_sequence_parallel`, `_num_dispatchers`, `a1q`, `scales`, `res`, `out`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `MoEPrepareAndFinalizeNaiveDPEPMonolithic`。 它继承自 `mk.FusedMoEPrepareAndFinalizeMonolithic`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`。 关键调用包括 `super.__init__`, `_quantize_and_setup_dispatch`, `get_ep_group.dispatch_router_logits`, `get_ep_group.combine`, `_unwrap_scale_and_prepare_for_moe`, `super`。 它会写入或更新 `is_sequence_parallel`, `_num_dispatchers`, `a1q`, `scales`, `res`, `out`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 219-226 — method `MoEPrepareAndFinalizeNaiveDPEPMonolithic.__init__`
```python
    def __init__(
        self,
        is_sequence_parallel: bool = False,
        num_dispatchers: int = 1,
    ) -> None:
        super().__init__()
        self.is_sequence_parallel = is_sequence_parallel
        self._num_dispatchers = num_dispatchers
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `is_sequence_parallel`, `num_dispatchers`. Key calls include `super.__init__`, `super`. It writes or updates `is_sequence_parallel`, `_num_dispatchers`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `is_sequence_parallel`, `num_dispatchers`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `is_sequence_parallel`, `_num_dispatchers`。

### Lines 244-271 — method `MoEPrepareAndFinalizeNaiveDPEPMonolithic.prepare`
```python
    def prepare(
        self,
        a1: torch.Tensor,
        router_logits: torch.Tensor,
        quant_config: FusedMoEQuantConfig,
        defer_input_quant: bool = False,
    ) -> mk.PrepareMonolithicResultType:
        """Quantize and Dispatch Router Logits."""

        a1q, scales = _quantize_and_setup_dispatch(a1, quant_config, defer_input_quant)

        res = get_ep_group().dispatch_router_logits(
            a1q,
            router_logits,
            is_sequence_parallel=self.is_sequence_parallel,
            extra_tensors=scales,
        )

        if scales is None:
            assert len(res) == 2
            a1q, router_logits = res
            a1q_scale = None
        else:
            assert len(res) == 3
            a1q, router_logits, scales = res
            a1q_scale = _unwrap_scale_and_prepare_for_moe(scales, quant_config)

        return a1q, a1q_scale, router_logits
```
**EN:** This method defines `prepare`. Quantize and Dispatch Router Logits. The main inputs are `a1`, `router_logits`, `quant_config`, `defer_input_quant`. Key calls include `_quantize_and_setup_dispatch`, `get_ep_group.dispatch_router_logits`, `_unwrap_scale_and_prepare_for_moe`, `get_ep_group`, `len`. It writes or updates `a1q`, `scales`, `res`, `router_logits`, `a1q_scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `prepare`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `a1`, `router_logits`, `quant_config`, `defer_input_quant`。 关键调用包括 `_quantize_and_setup_dispatch`, `get_ep_group.dispatch_router_logits`, `_unwrap_scale_and_prepare_for_moe`, `get_ep_group`, `len`。 它会写入或更新 `a1q`, `scales`, `res`, `router_logits`, `a1q_scale`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 273-280 — method `MoEPrepareAndFinalizeNaiveDPEPMonolithic.finalize`
```python
    def finalize(
        self,
        fused_expert_output: torch.Tensor,
    ) -> torch.Tensor:
        out = get_ep_group().combine(
            fused_expert_output, is_sequence_parallel=self.is_sequence_parallel
        )
        return out
```
**EN:** This method defines `finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `fused_expert_output`. Key calls include `get_ep_group.combine`, `get_ep_group`. It writes or updates `out`.
**CN:** 该方法定义 `finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `fused_expert_output`。 关键调用包括 `get_ep_group.combine`, `get_ep_group`。 它会写入或更新 `out`。

## Key Concepts / 关键概念
- [EN] Distributed moe preparation/finalization logic / [CN] 分布式 MoE 的准备/收尾逻辑
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `_quantize_and_setup_dispatch`, `_unwrap_scale_and_prepare_for_moe`, `MoEPrepareAndFinalizeNaiveDPEPModular`, `MoEPrepareAndFinalizeNaiveDPEPMonolithic` / [CN] 核心符号：`_quantize_and_setup_dispatch`, `_unwrap_scale_and_prepare_for_moe`, `MoEPrepareAndFinalizeNaiveDPEPModular`, `MoEPrepareAndFinalizeNaiveDPEPMonolithic`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.distributed`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.utils.flashinfer` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.distributed`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.utils.flashinfer`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
