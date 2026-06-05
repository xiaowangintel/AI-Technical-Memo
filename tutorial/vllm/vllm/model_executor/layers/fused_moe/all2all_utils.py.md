# all2all_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/all2all_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-34 — imports and setup
```python
from typing import Any

import torch

from vllm.config import get_current_vllm_config
from vllm.distributed import (
    get_ep_group,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.modular_kernel import (
    FusedMoEPrepareAndFinalize,
)
from vllm.model_executor.layers.fused_moe.prepare_finalize import (
    make_moe_prepare_and_finalize_naive_dp_ep,
    make_moe_prepare_and_finalize_no_dp_ep,
)
from vllm.model_executor.layers.fused_moe.prepare_finalize.flashinfer_nvlink_one_sided import (  # noqa: E501
    FlashInferNVLinkOneSidedPrepareAndFinalize,
)
from vllm.model_executor.layers.fused_moe.prepare_finalize.flashinfer_nvlink_two_sided import (  # noqa: E501
    FlashInferNVLinkTwoSidedPrepareAndFinalize,
)
from vllm.platforms import current_platform
from vllm.utils.import_utils import has_deep_ep, has_mori, has_nixl_ep

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 52-86 — function `maybe_roundup_layer_hidden_size`
```python
def maybe_roundup_layer_hidden_size(
    hidden_size: int,
    act_dtype: torch.dtype,
    moe_parallel_config: FusedMoEParallelConfig,
) -> int:
    """
    Given layer hidden size and MoE configurations, round up hidden_size
    if necessary.

    Args:
        hidden_size: Layer hidden-size
        act_dtype: Data type of the layer activations.
        moe_parallel_config: Fused MoE parallelization strategy configuration.

    Return:
        Rounded up hidden_size if rounding up is required based on the configs
        and all2all backend.
        Original hidden size otherwise.
    """
    if moe_parallel_config.use_deepep_ht_kernels:
        hidden_size = DeepEPHTPrepareAndFinalize.maybe_roundup_layer_hidden_size(
            hidden_size, act_dtype
        )

    if moe_parallel_config.use_deepep_ll_kernels:
        hidden_size = DeepEPLLPrepareAndFinalize.maybe_roundup_layer_hidden_size(
            hidden_size
        )

    if moe_parallel_config.use_nixl_ep_kernels:
        hidden_size = NixlEPPrepareAndFinalize.maybe_roundup_layer_hidden_size(
            hidden_size
        )

    return hidden_size
```
**EN:** This function defines `maybe_roundup_layer_hidden_size`. Given layer hidden size and MoE configurations, round up hidden_size if necessary. The main inputs are `hidden_size`, `act_dtype`, `moe_parallel_config`. Key calls include `DeepEPHTPrepareAndFinalize.maybe_roundup_layer_hidden_size`, `DeepEPLLPrepareAndFinalize.maybe_roundup_layer_hidden_size`, `NixlEPPrepareAndFinalize.maybe_roundup_layer_hidden_size`. It writes or updates `hidden_size`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `maybe_roundup_layer_hidden_size`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_size`, `act_dtype`, `moe_parallel_config`。 关键调用包括 `DeepEPHTPrepareAndFinalize.maybe_roundup_layer_hidden_size`, `DeepEPLLPrepareAndFinalize.maybe_roundup_layer_hidden_size`, `NixlEPPrepareAndFinalize.maybe_roundup_layer_hidden_size`。 它会写入或更新 `hidden_size`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 89-306 — function `maybe_make_prepare_finalize`
```python
def maybe_make_prepare_finalize(
    moe: FusedMoEConfig,
    quant_config: FusedMoEQuantConfig | None,
    routing_tables: tuple[torch.Tensor, torch.Tensor, torch.Tensor] | None = None,
    allow_new_interface: bool = False,
    use_monolithic: bool = False,
) -> FusedMoEPrepareAndFinalize | None:
    # NOTE(rob): we are migrating each quant_method to hold the MK
    # in all cases. The allow_new_interface=False flag allow us to fall
    # back to the old method for methods that have not yet been migrated.
    #
    # In old method:
    #   * maybe_init_modular_kernel() calls this function. If we are
    #     using no Dp/Ep or naive all2all, we return None this function
    #     returns None and no ModularKernelMethod is created. If non-naive
    #     all2all is used, this returns a PrepareAndFinalize object and
    #     a ModularKernelMethod is created.
    # In new method:
    #   * maybe_make_prepare_finalize() is called from the oracle. We
    #     always return a PrepareAndFinalize object and the quant method
    #     holds the ModularKernel.
    if not moe.moe_parallel_config.use_all2all_kernels:
        if not allow_new_interface:
            return None

        # For DP/TP case, fall back to naive P/F.
        if moe.moe_parallel_config.dp_size > 1:
            logger.info_once(
                "Detected DP deployment with no --enable-expert-parallel. "
                "Falling back to AllGather+ReduceScatter dispatch/combine."
            )
            device_communicator = get_ep_group().device_communicator
            assert device_communicator is not None
            assert device_communicator.all2all_manager is not None
            return make_moe_prepare_and_finalize_naive_dp_ep(
                is_sequence_parallel=moe.moe_parallel_config.is_sequence_parallel,
                num_dispatchers=(device_communicator.all2all_manager.world_size),
                use_monolithic=use_monolithic,
            )
        else:
            return make_moe_prepare_and_finalize_no_dp_ep(use_monolithic)

    device_communicator = get_ep_group().device_communicator
    assert device_communicator is not None
# ... omitted for brevity ...

    return prepare_finalize
```
**EN:** This function defines `maybe_make_prepare_finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `moe`, `quant_config`, `routing_tables`, `allow_new_interface`, `use_monolithic`. Key calls include `get_ep_group`, `dict`, `all2all_manager.get_handle`, `DeepEPHTPrepareAndFinalize`, `logger.info_once`, `make_moe_prepare_and_finalize_naive_dp_ep`. It writes or updates `device_communicator`, `all2all_manager`, `prepare_finalize`, `all_to_all_args`, `handle`, `global_to_physical`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `maybe_make_prepare_finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `moe`, `quant_config`, `routing_tables`, `allow_new_interface`, `use_monolithic`。 关键调用包括 `get_ep_group`, `dict`, `all2all_manager.get_handle`, `DeepEPHTPrepareAndFinalize`, `logger.info_once`, `make_moe_prepare_and_finalize_naive_dp_ep`。 它会写入或更新 `device_communicator`, `all2all_manager`, `prepare_finalize`, `all_to_all_args`, `handle`, `global_to_physical`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `maybe_roundup_layer_hidden_size`, `maybe_make_prepare_finalize` / [CN] 核心符号：`maybe_roundup_layer_hidden_size`, `maybe_make_prepare_finalize`

## Dependencies / 依赖关系
- **External**: `typing`, `torch` / **外部依赖**: `typing`, `torch`
- **Internal**: `vllm.config`, `vllm.distributed`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.prepare_finalize`, `vllm.model_executor.layers.fused_moe.prepare_finalize.flashinfer_nvlink_one_sided`, `vllm.model_executor.layers.fused_moe.prepare_finalize.flashinfer_nvlink_two_sided`, `vllm.platforms`, `vllm.utils.import_utils` / **内部依赖**: `vllm.config`, `vllm.distributed`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.prepare_finalize`, `vllm.model_executor.layers.fused_moe.prepare_finalize.flashinfer_nvlink_one_sided`, `vllm.model_executor.layers.fused_moe.prepare_finalize.flashinfer_nvlink_two_sided`, `vllm.platforms`, `vllm.utils.import_utils`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
