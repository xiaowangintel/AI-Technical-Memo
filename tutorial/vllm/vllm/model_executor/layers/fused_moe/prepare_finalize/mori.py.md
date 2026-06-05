# mori.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/prepare_finalize/mori.py`
- **Repository**: vllm-project/vllm
- **Purpose**: distributed MoE preparation/finalization logic; fused Mixture-of-Experts routing, kernels, and runtime helpers / 分布式 MoE 的准备/收尾逻辑；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-12 — imports and setup
```python
import mori
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.config import FusedMoEQuantConfig
from vllm.platforms import current_platform

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 15-124 — class `MoriPrepareAndFinalize`
```python
class MoriPrepareAndFinalize(mk.FusedMoEPrepareAndFinalizeModular):
    """
    Prepare/Finalize using MoRI kernels.
    """

    def __init__(
        self,
        mori_op: mori.ops.EpDispatchCombineOp,
        max_tokens_per_rank: int,
        num_dispatchers: int,
        use_fp8_dispatch: bool = False,
    ):
        super().__init__()
        self.mori_op = mori_op
# ... omitted for brevity ...
        )[0]
        output.copy_(result[:num_token])
```
**EN:** This class defines `MoriPrepareAndFinalize`. It inherits from `mk.FusedMoEPrepareAndFinalizeModular`. Prepare/Finalize using MoRI kernels. Important methods include `__init__`, `prepare`, `finalize`, `activation_format`, `output_is_reduced`, `num_dispatchers`. Key calls include `super.__init__`, `self.mori_op.dispatch`, `mk.ExpertTokensMetadata`, `output.copy_`, `self.mori_op.combine`, `super`. It writes or updates `mori_op`, `num_dispatchers_`, `max_tokens_per_rank`, `use_fp8_dispatch`, `scale`, `dispatch_a1`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `MoriPrepareAndFinalize`。 它继承自 `mk.FusedMoEPrepareAndFinalizeModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `prepare`, `finalize`, `activation_format`, `output_is_reduced`, `num_dispatchers`。 关键调用包括 `super.__init__`, `self.mori_op.dispatch`, `mk.ExpertTokensMetadata`, `output.copy_`, `self.mori_op.combine`, `super`。 它会写入或更新 `mori_op`, `num_dispatchers_`, `max_tokens_per_rank`, `use_fp8_dispatch`, `scale`, `dispatch_a1`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 20-31 — method `MoriPrepareAndFinalize.__init__`
```python
    def __init__(
        self,
        mori_op: mori.ops.EpDispatchCombineOp,
        max_tokens_per_rank: int,
        num_dispatchers: int,
        use_fp8_dispatch: bool = False,
    ):
        super().__init__()
        self.mori_op = mori_op
        self.num_dispatchers_ = num_dispatchers
        self.max_tokens_per_rank = max_tokens_per_rank
        self.use_fp8_dispatch = use_fp8_dispatch
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `mori_op`, `max_tokens_per_rank`, `num_dispatchers`, `use_fp8_dispatch`. Key calls include `super.__init__`, `super`. It writes or updates `mori_op`, `num_dispatchers_`, `max_tokens_per_rank`, `use_fp8_dispatch`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `mori_op`, `max_tokens_per_rank`, `num_dispatchers`, `use_fp8_dispatch`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `mori_op`, `num_dispatchers_`, `max_tokens_per_rank`, `use_fp8_dispatch`。

### Lines 52-107 — method `MoriPrepareAndFinalize.prepare`
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
        """
        Returns a tuple of:
        - quantized + dispatched a.
        - Optional quantized + dispatched a1_scales.
        - Optional ExpertTokensMetadata containing gpu/cpu tensors
          as big as the number of local experts with the information about the
          number of tokens assigned to each local expert.
        - Optional dispatched expert topk IDs
        - Optional dispatched expert topk weight
        """
        assert not apply_router_weight_on_input, (
            "mori does not support apply_router_weight_on_input=True now."
        )
        scale = None
        # When defer_input_quant is True, the expert kernel handles
        # quantization internally, so skip FP8 dispatch quantization.
        if self.use_fp8_dispatch and not defer_input_quant:
            from aiter import QuantType, get_hip_quant

            if quant_config.is_block_quantized:
                quant_func = get_hip_quant(QuantType.per_1x128)
                a1, scale = quant_func(a1, quant_dtype=current_platform.fp8_dtype())
            elif quant_config.is_per_act_token:
                quant_func = get_hip_quant(QuantType.per_Token)
                a1, scale = quant_func(a1, quant_dtype=current_platform.fp8_dtype())

        (
            dispatch_a1,
            dispatch_weights,
            dispatch_scale,
            dispatch_ids,
            dispatch_recv_token_num,
        ) = self.mori_op.dispatch(a1, topk_weights, scale, topk_ids)
# ... omitted for brevity ...
            dispatch_weights,
        )
```
**EN:** This method defines `prepare`. Returns a tuple of: - quantized + dispatched a. The main inputs are `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`. Key calls include `self.mori_op.dispatch`, `mk.ExpertTokensMetadata`, `get_hip_quant`, `quant_func`, `current_platform.fp8_dtype`. It writes or updates `scale`, `dispatch_a1`, `dispatch_weights`, `dispatch_scale`, `dispatch_ids`, `dispatch_recv_token_num`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `prepare`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`。 关键调用包括 `self.mori_op.dispatch`, `mk.ExpertTokensMetadata`, `get_hip_quant`, `quant_func`, `current_platform.fp8_dtype`。 它会写入或更新 `scale`, `dispatch_a1`, `dispatch_weights`, `dispatch_scale`, `dispatch_ids`, `dispatch_recv_token_num`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 109-124 — method `MoriPrepareAndFinalize.finalize`
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
        num_token = output.shape[0]
        result = self.mori_op.combine(
            fused_expert_output,
            None,
            topk_ids,
        )[0]
        output.copy_(result[:num_token])
```
**EN:** This method defines `finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`. Key calls include `output.copy_`, `self.mori_op.combine`. It writes or updates `num_token`, `result`.
**CN:** 该方法定义 `finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`。 关键调用包括 `output.copy_`, `self.mori_op.combine`。 它会写入或更新 `num_token`, `result`。

### Lines 33-35 — method `MoriPrepareAndFinalize.activation_format`
```python
    @property
    def activation_format(self) -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 37-38 — method `MoriPrepareAndFinalize.output_is_reduced`
```python
    def output_is_reduced(self) -> bool:
        return True
```
**EN:** This method defines `output_is_reduced`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `output_is_reduced`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 40-41 — method `MoriPrepareAndFinalize.num_dispatchers`
```python
    def num_dispatchers(self):
        return self.num_dispatchers_
```
**EN:** This method defines `num_dispatchers`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `num_dispatchers`。 它管理专家分发、融合内核或 MoE 特有的张量流。

## Key Concepts / 关键概念
- [EN] Distributed moe preparation/finalization logic / [CN] 分布式 MoE 的准备/收尾逻辑
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `MoriPrepareAndFinalize` / [CN] 核心符号：`MoriPrepareAndFinalize`

## Dependencies / 依赖关系
- **External**: `mori`, `torch` / **外部依赖**: `mori`, `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.platforms` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.platforms`
- **Runtime traits**: platform-aware dispatch / **运行时特征**: platform-aware dispatch
