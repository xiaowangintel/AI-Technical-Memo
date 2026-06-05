# flashinfer_nvlink_two_sided.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/prepare_finalize/flashinfer_nvlink_two_sided.py`
- **Repository**: vllm-project/vllm
- **Purpose**: distributed MoE preparation/finalization logic; fused Mixture-of-Experts routing, kernels, and runtime helpers / 分布式 MoE 的准备/收尾逻辑；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-14 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.distributed import get_ep_group
from vllm.distributed.device_communicators.base_device_communicator import (
    All2AllManagerBase,
)
from vllm.forward_context import get_forward_context
from vllm.model_executor.layers.fused_moe.config import FusedMoEQuantConfig
from vllm.model_executor.layers.fused_moe.utils import moe_kernel_quantize_input
from vllm.utils.flashinfer import nvfp4_block_scale_interleave
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 17-20 — function `get_local_sizes`
```python
def get_local_sizes():
    dp_metadata = get_forward_context().dp_metadata
    assert dp_metadata is not None
    return dp_metadata.get_chunk_sizes_across_dp_rank()
```
**EN:** This function defines `get_local_sizes`. It derives and returns computed metadata or outputs needed by later stages. Key calls include `dp_metadata.get_chunk_sizes_across_dp_rank`, `get_forward_context`. It writes or updates `dp_metadata`.
**CN:** 该函数定义 `get_local_sizes`。 它推导并返回后续阶段所需的元数据或输出。 关键调用包括 `dp_metadata.get_chunk_sizes_across_dp_rank`, `get_forward_context`。 它会写入或更新 `dp_metadata`。

### Lines 23-122 — class `FlashInferNVLinkTwoSidedPrepareAndFinalize`
```python
class FlashInferNVLinkTwoSidedPrepareAndFinalize(mk.FusedMoEPrepareAndFinalizeModular):
    """Base class for FlashInfer MoE prepare and finalize operations."""

    all2all_manager: All2AllManagerBase

    def __init__(
        self,
        num_dispatchers: int = 1,
    ):
        super().__init__()
        self.num_dispatchers_ = num_dispatchers
        device_communicator = get_ep_group().device_communicator
        assert device_communicator is not None
        assert device_communicator.all2all_manager is not None
# ... omitted for brevity ...
        )
        output.copy_(fused_expert_output)
```
**EN:** This class defines `FlashInferNVLinkTwoSidedPrepareAndFinalize`. It inherits from `mk.FusedMoEPrepareAndFinalizeModular`. Base class for FlashInfer MoE prepare and finalize operations. Important methods include `__init__`, `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`. Key calls include `super.__init__`, `self._apply_router_weight_on_input`, `get_local_sizes`, `topk_ids.size`, `flashinfer_alltoall_dispatch`, `flashinfer_alltoall_combine`. It writes or updates `all2all_manager`, `num_dispatchers_`, `device_communicator`, `global_num_tokens_cpu`, `top_k`, `alltoall_info`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `FlashInferNVLinkTwoSidedPrepareAndFinalize`。 它继承自 `mk.FusedMoEPrepareAndFinalizeModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `prepare`, `finalize`, `activation_format`, `max_num_tokens_per_rank`, `topk_indices_dtype`。 关键调用包括 `super.__init__`, `self._apply_router_weight_on_input`, `get_local_sizes`, `topk_ids.size`, `flashinfer_alltoall_dispatch`, `flashinfer_alltoall_combine`。 它会写入或更新 `all2all_manager`, `num_dispatchers_`, `device_communicator`, `global_num_tokens_cpu`, `top_k`, `alltoall_info`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 28-37 — method `FlashInferNVLinkTwoSidedPrepareAndFinalize.__init__`
```python
    def __init__(
        self,
        num_dispatchers: int = 1,
    ):
        super().__init__()
        self.num_dispatchers_ = num_dispatchers
        device_communicator = get_ep_group().device_communicator
        assert device_communicator is not None
        assert device_communicator.all2all_manager is not None
        self.all2all_manager = device_communicator.all2all_manager
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `num_dispatchers`. Key calls include `super.__init__`, `get_ep_group`, `super`. It writes or updates `num_dispatchers_`, `device_communicator`, `all2all_manager`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `num_dispatchers`。 关键调用包括 `super.__init__`, `get_ep_group`, `super`。 它会写入或更新 `num_dispatchers_`, `device_communicator`, `all2all_manager`。

### Lines 70-102 — method `FlashInferNVLinkTwoSidedPrepareAndFinalize.prepare`
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
        self._apply_router_weight_on_input(
            a1, topk_weights, topk_ids, apply_router_weight_on_input
        )
        global_num_tokens_cpu = get_local_sizes()
        top_k = topk_ids.size(1)

        (self.alltoall_info, topk_ids, topk_weights, a1q, a1q_scale) = (
            flashinfer_alltoall_dispatch(
                self.all2all_manager,
                global_num_tokens_cpu,
                a1,
                quant_config.a1_gscale,
                topk_ids,
                topk_weights,
                top_k,
                num_experts,
                quant_config,
                defer_input_quant=defer_input_quant,
            )
        )

        return a1q, a1q_scale, None, topk_ids, topk_weights
```
**EN:** This method defines `prepare`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`. Key calls include `self._apply_router_weight_on_input`, `get_local_sizes`, `topk_ids.size`, `flashinfer_alltoall_dispatch`. It writes or updates `global_num_tokens_cpu`, `top_k`, `alltoall_info`, `topk_ids`, `topk_weights`, `a1q`.
**CN:** 该方法定义 `prepare`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`。 关键调用包括 `self._apply_router_weight_on_input`, `get_local_sizes`, `topk_ids.size`, `flashinfer_alltoall_dispatch`。 它会写入或更新 `global_num_tokens_cpu`, `top_k`, `alltoall_info`, `topk_ids`, `topk_weights`, `a1q`。

### Lines 104-122 — method `FlashInferNVLinkTwoSidedPrepareAndFinalize.finalize`
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
        top_k = topk_ids.size(1)
        token_count = output.shape[0]
        fused_expert_output = flashinfer_alltoall_combine(
            self.all2all_manager,
            fused_expert_output,
            top_k=top_k,
            token_count=token_count,
            alltoall_info=self.alltoall_info,
        )
        output.copy_(fused_expert_output)
```
**EN:** This method defines `finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`. Key calls include `topk_ids.size`, `flashinfer_alltoall_combine`, `output.copy_`. It writes or updates `top_k`, `token_count`, `fused_expert_output`.
**CN:** 该方法定义 `finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`。 关键调用包括 `topk_ids.size`, `flashinfer_alltoall_combine`, `output.copy_`。 它会写入或更新 `top_k`, `token_count`, `fused_expert_output`。

### Lines 39-41 — method `FlashInferNVLinkTwoSidedPrepareAndFinalize.activation_format`
```python
    @property
    def activation_format(self) -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 43-44 — method `FlashInferNVLinkTwoSidedPrepareAndFinalize.max_num_tokens_per_rank`
```python
    def max_num_tokens_per_rank(self) -> int | None:
        return None
```
**EN:** This method defines `max_num_tokens_per_rank`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `max_num_tokens_per_rank`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 46-47 — method `FlashInferNVLinkTwoSidedPrepareAndFinalize.topk_indices_dtype`
```python
    def topk_indices_dtype(self) -> torch.dtype | None:
        return None
```
**EN:** This method defines `topk_indices_dtype`. It computes routing scores or expert-selection behavior for MoE execution.
**CN:** 该方法定义 `topk_indices_dtype`。 它为 MoE 执行计算路由得分或专家选择行为。

### Lines 125-216 — function `flashinfer_alltoall_dispatch`
```python
def flashinfer_alltoall_dispatch(
    all2all_manager: All2AllManagerBase,
    global_num_tokens_cpu: list[int],
    x: torch.Tensor,
    gs: torch.Tensor,
    topk_ids: torch.Tensor,
    topk_weights: torch.Tensor,
    top_k: int,
    num_experts: int,
    quant_config: FusedMoEQuantConfig,
    defer_input_quant: bool = False,
):
    from flashinfer.comm.trtllm_alltoall import MnnvlMoe

    assert all2all_manager.ensure_alltoall_workspace_initialized(), (  # type: ignore[attr-defined]
        "FlashInfer AllToAll workspace not available"
    )

    ep_rank = all2all_manager.rank
    ep_size = all2all_manager.world_size
    max_num_token = (
        max(global_num_tokens_cpu) if global_num_tokens_cpu is not None else x.shape[0]
    )
    orig_topk_weights_dtype = topk_weights.dtype
    alltoall_info, topk_ids, topk_weights, _ = (
        MnnvlMoe.mnnvl_moe_alltoallv_prepare_without_allgather(
            topk_ids,
            topk_weights,
            None,
            all2all_manager.prepare_workspace_tensor,  # type: ignore[attr-defined]
            max_num_token,
            ep_rank,
            ep_size,
            num_experts,
            num_experts,
            top_k,
        )
    )
    topk_weights = topk_weights.view(dtype=orig_topk_weights_dtype)

    if not defer_input_quant:
        x, x_sf = moe_kernel_quantize_input(
            x,
            gs,
# ... omitted for brevity ...
        )
    return alltoall_info, topk_ids, topk_weights, x, x_sf
```
**EN:** This function defines `flashinfer_alltoall_dispatch`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `all2all_manager`, `global_num_tokens_cpu`, `x`, `gs`, `topk_ids`, `topk_weights`. Key calls include `all2all_manager.ensure_alltoall_workspace_initialized`, `MnnvlMoe.mnnvl_moe_alltoallv_prepare_without_allgather`, `topk_weights.view`, `max`, `moe_kernel_quantize_input`, `MnnvlMoe.mnnvl_moe_alltoallv`. It writes or updates `ep_rank`, `ep_size`, `max_num_token`, `orig_topk_weights_dtype`, `alltoall_info`, `topk_ids`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `flashinfer_alltoall_dispatch`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `all2all_manager`, `global_num_tokens_cpu`, `x`, `gs`, `topk_ids`, `topk_weights`。 关键调用包括 `all2all_manager.ensure_alltoall_workspace_initialized`, `MnnvlMoe.mnnvl_moe_alltoallv_prepare_without_allgather`, `topk_weights.view`, `max`, `moe_kernel_quantize_input`, `MnnvlMoe.mnnvl_moe_alltoallv`。 它会写入或更新 `ep_rank`, `ep_size`, `max_num_token`, `orig_topk_weights_dtype`, `alltoall_info`, `topk_ids`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 219-239 — function `flashinfer_alltoall_combine`
```python
def flashinfer_alltoall_combine(
    all2all_manager: All2AllManagerBase,
    output: torch.Tensor,
    top_k: int,
    token_count: int,
    alltoall_info,
):
    from flashinfer.comm.trtllm_alltoall import MnnvlMoe

    assert all2all_manager.ensure_alltoall_workspace_initialized(), (  # type: ignore[attr-defined]
        "FlashInfer AllToAll workspace not available"
    )
    return MnnvlMoe.mnnvl_moe_alltoallv_combine(
        output,
        alltoall_info,
        all2all_manager.workspace_tensor,  # type: ignore[attr-defined]
        ep_rank=all2all_manager.rank,
        ep_size=all2all_manager.world_size,
        top_k=top_k,
        token_count=token_count,
    )
```
**EN:** This function defines `flashinfer_alltoall_combine`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `all2all_manager`, `output`, `top_k`, `token_count`, `alltoall_info`. Key calls include `all2all_manager.ensure_alltoall_workspace_initialized`, `MnnvlMoe.mnnvl_moe_alltoallv_combine`.
**CN:** 该函数定义 `flashinfer_alltoall_combine`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `all2all_manager`, `output`, `top_k`, `token_count`, `alltoall_info`。 关键调用包括 `all2all_manager.ensure_alltoall_workspace_initialized`, `MnnvlMoe.mnnvl_moe_alltoallv_combine`。

## Key Concepts / 关键概念
- [EN] Distributed moe preparation/finalization logic / [CN] 分布式 MoE 的准备/收尾逻辑
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `get_local_sizes`, `FlashInferNVLinkTwoSidedPrepareAndFinalize`, `flashinfer_alltoall_dispatch`, `flashinfer_alltoall_combine` / [CN] 核心符号：`get_local_sizes`, `FlashInferNVLinkTwoSidedPrepareAndFinalize`, `flashinfer_alltoall_dispatch`, `flashinfer_alltoall_combine`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.distributed`, `vllm.distributed.device_communicators.base_device_communicator`, `vllm.forward_context`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.utils.flashinfer` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.distributed`, `vllm.distributed.device_communicators.base_device_communicator`, `vllm.forward_context`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.utils.flashinfer`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
