# batched.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/prepare_finalize/batched.py`
- **Repository**: vllm-project/vllm
- **Purpose**: distributed MoE preparation/finalization logic; fused Mixture-of-Experts routing, kernels, and runtime helpers / 分布式 MoE 的准备/收尾逻辑；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-15 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.model_executor.layers.fused_moe.config import FusedMoEQuantConfig
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceDelegate,
    TopKWeightAndReduceNaiveBatched,
)
from vllm.model_executor.layers.fused_moe.utils import (
    moe_kernel_quantize_input,
    normalize_scales_shape,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 18-171 — class `BatchedPrepareAndFinalize`
```python
class BatchedPrepareAndFinalize(mk.FusedMoEPrepareAndFinalizeModular):
    """
    A reference prepare/finalize class that reorganizes the tokens into
    expert batched format, i.e. E x max_num_tokens x K.  This is the format
    that the batched dispatch/combine kernels use.
    """

    def __init__(
        self,
        max_num_tokens: int,
        num_local_experts: int,
        num_dispatchers: int,
        rank: int,
    ):
# ... omitted for brevity ...
            apply_router_weight_on_input=apply_router_weight_on_input,
        )
```
**EN:** This class defines `BatchedPrepareAndFinalize`. It inherits from `mk.FusedMoEPrepareAndFinalizeModular`. A reference prepare/finalize class that reorganizes the tokens into expert batched format, i.e. Important methods include `__init__`, `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`. Key calls include `super.__init__`, `a1.size`, `topk_ids.size`, `torch.zeros`, `normalize_scales_shape`, `range`. It writes or updates `max_num_tokens`, `num_local_experts`, `rank`, `num_dispatchers_`, `num_tokens`, `hidden_dim`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `BatchedPrepareAndFinalize`。 它继承自 `mk.FusedMoEPrepareAndFinalizeModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`。 关键调用包括 `super.__init__`, `a1.size`, `topk_ids.size`, `torch.zeros`, `normalize_scales_shape`, `range`。 它会写入或更新 `max_num_tokens`, `num_local_experts`, `rank`, `num_dispatchers_`, `num_tokens`, `hidden_dim`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 25-36 — method `BatchedPrepareAndFinalize.__init__`
```python
    def __init__(
        self,
        max_num_tokens: int,
        num_local_experts: int,
        num_dispatchers: int,
        rank: int,
    ):
        super().__init__()
        self.max_num_tokens = max_num_tokens
        self.num_local_experts = num_local_experts
        self.rank = rank
        self.num_dispatchers_ = num_dispatchers
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `max_num_tokens`, `num_local_experts`, `num_dispatchers`, `rank`. Key calls include `super.__init__`, `super`. It writes or updates `max_num_tokens`, `num_local_experts`, `rank`, `num_dispatchers_`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `max_num_tokens`, `num_local_experts`, `num_dispatchers`, `rank`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `max_num_tokens`, `num_local_experts`, `rank`, `num_dispatchers_`。

### Lines 54-152 — method `BatchedPrepareAndFinalize.prepare`
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
        if defer_input_quant:
            raise NotImplementedError(
                f"{self.__class__.__name__} does not support defer_input_quant=True. "
                "Please select an MoE kernel that accepts quantized inputs."
            )
        assert a1.dim() == 2
        assert topk_ids.dim() == 2
        assert topk_ids.size(0) == a1.size(0)

        if apply_router_weight_on_input:
            topk = topk_ids.size(1)
            # TODO: this only works for topK=1, will need to update for topK>1
            assert topk == 1, (
                "apply_router_weight_on_input is only implemented for topk=1"
            )
            a1.mul_(topk_weights.to(a1.dtype))

        num_tokens, hidden_dim = a1.size()
        topk = topk_ids.size(1)

        tokens_per_expert = torch.zeros(num_experts, dtype=torch.int, device=a1.device)

        num_local_experts = self.num_local_experts

        if quant_config.quant_dtype is None:
            b_type = a1.dtype
        else:
            b_type = quant_config.quant_dtype

        b_a1 = torch.zeros(
            (num_local_experts, self.max_num_tokens, hidden_dim),
            dtype=b_type,
            device=a1.device,
# ... omitted for brevity ...

        return b_a1, b_a1_scale, expert_tokens_meta, None, None
```
**EN:** This method defines `prepare`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`. Key calls include `a1.size`, `topk_ids.size`, `torch.zeros`, `normalize_scales_shape`, `range`, `mk.ExpertTokensMetadata`. It writes or updates `num_tokens`, `hidden_dim`, `topk`, `tokens_per_expert`, `num_local_experts`, `b_a1`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该方法定义 `prepare`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`。 关键调用包括 `a1.size`, `topk_ids.size`, `torch.zeros`, `normalize_scales_shape`, `range`, `mk.ExpertTokensMetadata`。 它会写入或更新 `num_tokens`, `hidden_dim`, `topk`, `tokens_per_expert`, `num_local_experts`, `b_a1`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 154-171 — method `BatchedPrepareAndFinalize.finalize`
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
            weight_and_reduce_impl = TopKWeightAndReduceNaiveBatched(self.rank)
        weight_and_reduce_impl.apply(
            output=output,
            fused_expert_output=fused_expert_output,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            apply_router_weight_on_input=apply_router_weight_on_input,
        )
```
**EN:** This method defines `finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`. Key calls include `isinstance`, `weight_and_reduce_impl.apply`, `TopKWeightAndReduceNaiveBatched`. It writes or updates `weight_and_reduce_impl`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`。 关键调用包括 `isinstance`, `weight_and_reduce_impl.apply`, `TopKWeightAndReduceNaiveBatched`。 它会写入或更新 `weight_and_reduce_impl`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 38-40 — method `BatchedPrepareAndFinalize.activation_format`
```python
    @property
    def activation_format(self) -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.BatchedExperts
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 42-43 — method `BatchedPrepareAndFinalize.max_num_tokens_per_rank`
```python
    def max_num_tokens_per_rank(self) -> int | None:
        return self.max_num_tokens
```
**EN:** This method defines `max_num_tokens_per_rank`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `max_num_tokens_per_rank`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 45-46 — method `BatchedPrepareAndFinalize.topk_indices_dtype`
```python
    def topk_indices_dtype(self) -> torch.dtype | None:
        return None
```
**EN:** This method defines `topk_indices_dtype`. It computes routing scores or expert-selection behavior for MoE execution.
**CN:** 该方法定义 `topk_indices_dtype`。 它为 MoE 执行计算路由得分或专家选择行为。

## Key Concepts / 关键概念
- [EN] Distributed moe preparation/finalization logic / [CN] 分布式 MoE 的准备/收尾逻辑
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `BatchedPrepareAndFinalize` / [CN] 核心符号：`BatchedPrepareAndFinalize`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`
