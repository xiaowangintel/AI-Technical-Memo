# fused_humming_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/fused_humming_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fused MoE utilities for Humming. / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-45 — imports and setup
```python
import json
import math
from typing import TYPE_CHECKING, Any

import torch
from humming import dtypes
from humming.config import GemmType as HummingGemmType
from humming.layer import HummingLayerMeta, HummingMethod

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import envs
from vllm.forward_context import get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.moe_align_block_size import (
    moe_align_block_size,
)
from vllm.model_executor.layers.fused_moe.moe_fused_mul_sum import moe_fused_mul_sum
from vllm.model_executor.layers.fused_moe.moe_permute_unpermute import (
    moe_permute,
    moe_unpermute,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceDelegate,
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.fused_moe.utils import _resize_cache
from vllm.model_executor.layers.quantization.utils.quant_utils import QuantKey
from vllm.platforms import current_platform
from vllm.v1.worker.workspace import current_workspace_manager

if TYPE_CHECKING:
    from vllm.model_executor.layers.fused_moe import RoutedExperts


logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`. The body uses conditional branches to cover different runtime cases.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 48-59 — function `get_humming_moe_gemm_type`
```python
def get_humming_moe_gemm_type() -> str:
    env_gemm_type: str = envs.VLLM_HUMMING_MOE_GEMM_TYPE or ""
    env_gemm_type = env_gemm_type.lower()
    if env_gemm_type == "indexed":
        gemm_type = env_gemm_type
    elif env_gemm_type in ["grouped_contiguous", "grouped"]:
        gemm_type = "grouped_contiguous"
    else:
        gemm_type = "indexed"

    logger.info_once(f"Using {gemm_type} gemm for humming moe")  # noqa
    return gemm_type
```
**EN:** This function defines `get_humming_moe_gemm_type`. It derives and returns computed metadata or outputs needed by later stages. Key calls include `env_gemm_type.lower`, `logger.info_once`. It writes or updates `env_gemm_type`, `gemm_type`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `get_humming_moe_gemm_type`。 它推导并返回后续阶段所需的元数据或输出。 关键调用包括 `env_gemm_type.lower`, `logger.info_once`。 它会写入或更新 `env_gemm_type`, `gemm_type`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 62-423 — class `HummingExpertsBase`
```python
class HummingExpertsBase(mk.FusedMoEExpertsModular):
    def __init__(
        self,
        layer: "RoutedExperts",
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        self.layer = layer
        self.num_experts = self.layer.num_experts
        self.global_num_experts = self.layer.global_num_experts
        self.init_humming_moe()

# ... omitted for brevity ...

        return supported, None if supported else reason
```
**EN:** This class defines `HummingExpertsBase`. It inherits from `mk.FusedMoEExpertsModular`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `apply`, `init_humming_moe`, `get_global_valid_shape_m`, `estimate_local_valid_shape_m`, `humming_gemm_type`. Key calls include `self.init_humming_moe`, `self.is_batched`, `super.__init__`, `HummingMethod.get_default_tuning_configs`, `json.dumps`, `topk_ids.size`. It writes or updates `layer`, `num_experts`, `global_num_experts`, `compute_config`, `w13_tuning_config`, `w2_tuning_config`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `HummingExpertsBase`。 它继承自 `mk.FusedMoEExpertsModular`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `apply`, `init_humming_moe`, `get_global_valid_shape_m`, `estimate_local_valid_shape_m`, `humming_gemm_type`。 关键调用包括 `self.init_humming_moe`, `self.is_batched`, `super.__init__`, `HummingMethod.get_default_tuning_configs`, `json.dumps`, `topk_ids.size`。 它会写入或更新 `layer`, `num_experts`, `global_num_experts`, `compute_config`, `w13_tuning_config`, `w2_tuning_config`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 63-84 — method `HummingExpertsBase.__init__`
```python
    def __init__(
        self,
        layer: "RoutedExperts",
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int | None = None,
        num_dispatchers: int | None = None,
    ):
        self.layer = layer
        self.num_experts = self.layer.num_experts
        self.global_num_experts = self.layer.global_num_experts
        self.init_humming_moe()

        if self.is_batched():
            assert max_num_tokens is not None and num_dispatchers is not None

        super().__init__(
            moe_config=moe_config,
            quant_config=quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=num_dispatchers,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `layer`, `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`. Key calls include `self.init_humming_moe`, `self.is_batched`, `super.__init__`, `super`. It writes or updates `layer`, `num_experts`, `global_num_experts`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `layer`, `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`。 关键调用包括 `self.init_humming_moe`, `self.is_batched`, `super.__init__`, `super`。 它会写入或更新 `layer`, `num_experts`, `global_num_experts`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 358-385 — method `HummingExpertsBase.apply`
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
        assert not apply_router_weight_on_input

        self.main_apply(
            hidden_states=hidden_states,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            workspace1=workspace13,
            workspace2=workspace2,
            expert_tokens_meta=expert_tokens_meta,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `self.main_apply`.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `self.main_apply`。

### Lines 86-108 — method `HummingExpertsBase.init_humming_moe`
```python
    def init_humming_moe(self):
        self.compute_config = {
            "use_batch_invariant": envs.VLLM_BATCH_INVARIANT,
            "use_f16_accum": envs.VLLM_HUMMING_USE_F16_ACCUM,
            "gemm_type": self.humming_gemm_type().value,
        }
        self.w13_tuning_config = HummingMethod.get_default_tuning_configs(
            layer=self.layer,
            use_f16_accum=envs.VLLM_HUMMING_USE_F16_ACCUM,
            use_batch_invariant=envs.VLLM_BATCH_INVARIANT,
            gemm_type=self.humming_gemm_type(),
            sublayer_name="w13",
        )
        self.w2_tuning_config = HummingMethod.get_default_tuning_configs(
            layer=self.layer,
            use_f16_accum=envs.VLLM_HUMMING_USE_F16_ACCUM,
            use_batch_invariant=envs.VLLM_BATCH_INVARIANT,
            gemm_type=self.humming_gemm_type(),
            sublayer_name="w2",
        )
        self.compute_config_str = json.dumps(self.compute_config)
        self.w13_tuning_config_str = json.dumps(self.w13_tuning_config)
        self.w2_tuning_config_str = json.dumps(self.w2_tuning_config)
```
**EN:** This method defines `init_humming_moe`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `HummingMethod.get_default_tuning_configs`, `json.dumps`, `self.humming_gemm_type`. It writes or updates `compute_config`, `w13_tuning_config`, `w2_tuning_config`, `compute_config_str`, `w13_tuning_config_str`, `w2_tuning_config_str`.
**CN:** 该方法定义 `init_humming_moe`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `HummingMethod.get_default_tuning_configs`, `json.dumps`, `self.humming_gemm_type`。 它会写入或更新 `compute_config`, `w13_tuning_config`, `w2_tuning_config`, `compute_config_str`, `w13_tuning_config_str`, `w2_tuning_config_str`。

### Lines 110-116 — method `HummingExpertsBase.get_global_valid_shape_m`
```python
    def get_global_valid_shape_m(self, topk_ids: torch.Tensor):
        num_tokens = topk_ids.size(0)
        ctx = get_forward_context()
        if ctx.dp_metadata is not None:
            num_tokens = ctx.dp_metadata.num_tokens_across_dp_cpu.sum().item()

        return num_tokens * topk_ids.size(1)
```
**EN:** This method defines `get_global_valid_shape_m`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `topk_ids`. Key calls include `topk_ids.size`, `get_forward_context`, `ctx.dp_metadata.num_tokens_across_dp_cpu.sum.item`, `ctx.dp_metadata.num_tokens_across_dp_cpu.sum`. It writes or updates `num_tokens`, `ctx`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `get_global_valid_shape_m`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `topk_ids`。 关键调用包括 `topk_ids.size`, `get_forward_context`, `ctx.dp_metadata.num_tokens_across_dp_cpu.sum.item`, `ctx.dp_metadata.num_tokens_across_dp_cpu.sum`。 它会写入或更新 `num_tokens`, `ctx`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 118-123 — method `HummingExpertsBase.estimate_local_valid_shape_m`
```python
    def estimate_local_valid_shape_m(self, topk_ids: torch.Tensor):
        # estimate shape_m for kernel tuning
        global_valid_shape_m = self.get_global_valid_shape_m(topk_ids)
        num_experts = self.num_experts
        global_num_experts = self.global_num_experts
        return math.ceil(global_valid_shape_m * num_experts / global_num_experts)
```
**EN:** This method defines `estimate_local_valid_shape_m`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `topk_ids`. Key calls include `self.get_global_valid_shape_m`, `math.ceil`. It writes or updates `global_valid_shape_m`, `num_experts`, `global_num_experts`.
**CN:** 该方法定义 `estimate_local_valid_shape_m`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `topk_ids`。 关键调用包括 `self.get_global_valid_shape_m`, `math.ceil`。 它会写入或更新 `global_valid_shape_m`, `num_experts`, `global_num_experts`。

### Lines 125-127 — method `HummingExpertsBase.humming_gemm_type`
```python
    @staticmethod
    def humming_gemm_type() -> HummingGemmType:
        raise NotImplementedError
```
**EN:** This method defines `humming_gemm_type`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `humming_gemm_type`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 426-545 — class `HummingIndexedExperts`
```python
class HummingIndexedExperts(HummingExpertsBase):
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()

    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard

    @staticmethod
    def humming_gemm_type() -> HummingGemmType:
        return HummingGemmType.INDEXED

    def prepare_humming_moe_kwargs(
        self,
# ... omitted for brevity ...
            outputs=buffers["output"],
        )
```
**EN:** This class defines `HummingIndexedExperts`. It inherits from `HummingExpertsBase`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `finalize_weight_and_reduce_impl`, `activation_format`, `humming_gemm_type`, `prepare_humming_moe_kwargs`, `main_apply`. Key calls include `TopKWeightAndReduceNoOP`, `self.estimate_local_valid_shape_m`, `moe_align_block_size`, `topk_ids.size`, `moe_kwargs1.update`, `moe_kwargs2.update`. It writes or updates `valid_shape_m`, `sorted_ids`, `expert_ids`, `num_tokens_padded`, `moe_common_kwargs`, `top_k`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `HummingIndexedExperts`。 它继承自 `HummingExpertsBase`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `finalize_weight_and_reduce_impl`, `activation_format`, `humming_gemm_type`, `prepare_humming_moe_kwargs`, `main_apply`。 关键调用包括 `TopKWeightAndReduceNoOP`, `self.estimate_local_valid_shape_m`, `moe_align_block_size`, `topk_ids.size`, `moe_kwargs1.update`, `moe_kwargs2.update`。 它会写入或更新 `valid_shape_m`, `sorted_ids`, `expert_ids`, `num_tokens_padded`, `moe_common_kwargs`, `top_k`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 427-428 — method `HummingIndexedExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 430-432 — method `HummingIndexedExperts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 434-436 — method `HummingIndexedExperts.humming_gemm_type`
```python
    @staticmethod
    def humming_gemm_type() -> HummingGemmType:
        return HummingGemmType.INDEXED
```
**EN:** This method defines `humming_gemm_type`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `humming_gemm_type`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 438-475 — method `HummingIndexedExperts.prepare_humming_moe_kwargs`
```python
    def prepare_humming_moe_kwargs(
        self,
        topk_ids: torch.Tensor,
        expert_map: torch.Tensor | None,
        expert_tokens_meta: mk.ExpertTokensMetadata | None,
    ) -> tuple[dict[str, Any], dict[str, Any]]:
        valid_shape_m = self.estimate_local_valid_shape_m(topk_ids)

        for min_shape_m, max_shape_m, config in self.w13_tuning_config:
            if valid_shape_m > min_shape_m and valid_shape_m <= max_shape_m:
                moe_block_size = config["block_shape"][0]
                break
        else:
            raise ValueError(f"cannot found moe_block_size for shape {valid_shape_m}")

        sorted_ids, expert_ids, num_tokens_padded = moe_align_block_size(
            topk_ids=topk_ids,
            block_size=moe_block_size,
            num_experts=self.global_num_experts,
            expert_map=expert_map,
            ignore_invalid_experts=True,
        )

        moe_common_kwargs = {
            "sorted_ids": sorted_ids,
            "expert_ids": expert_ids,
            "num_tokens_padded": num_tokens_padded,
            "compute_config": self.compute_config_str,
            "valid_shape_m": valid_shape_m,
        }

        top_k = topk_ids.size(1)
        moe_kwargs1 = {"top_k": top_k, "tuning_config": self.w13_tuning_config_str}
        moe_kwargs2 = {"top_k": 1, "tuning_config": self.w2_tuning_config_str}
        moe_kwargs1.update(moe_common_kwargs)
        moe_kwargs2.update(moe_common_kwargs)

        return moe_kwargs1, moe_kwargs2
```
**EN:** This method defines `prepare_humming_moe_kwargs`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `topk_ids`, `expert_map`, `expert_tokens_meta`. Key calls include `self.estimate_local_valid_shape_m`, `moe_align_block_size`, `topk_ids.size`, `moe_kwargs1.update`, `moe_kwargs2.update`, `ValueError`. It writes or updates `valid_shape_m`, `sorted_ids`, `expert_ids`, `num_tokens_padded`, `moe_common_kwargs`, `top_k`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该方法定义 `prepare_humming_moe_kwargs`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `topk_ids`, `expert_map`, `expert_tokens_meta`。 关键调用包括 `self.estimate_local_valid_shape_m`, `moe_align_block_size`, `topk_ids.size`, `moe_kwargs1.update`, `moe_kwargs2.update`, `ValueError`。 它会写入或更新 `valid_shape_m`, `sorted_ids`, `expert_ids`, `num_tokens_padded`, `moe_common_kwargs`, `top_k`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `get_humming_moe_gemm_type`, `HummingExpertsBase`, `HummingIndexedExperts`, `HummingGroupedExperts` / [CN] 核心符号：`get_humming_moe_gemm_type`, `HummingExpertsBase`, `HummingIndexedExperts`, `HummingGroupedExperts`

## Dependencies / 依赖关系
- **External**: `json`, `math`, `typing`, `torch`, `humming`, `humming.config`, `humming.layer` / **外部依赖**: `json`, `math`, `typing`, `torch`, `humming`, `humming.config`, `humming.layer`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.model_executor.layers.fused_moe.moe_fused_mul_sum`, `vllm.model_executor.layers.fused_moe.moe_permute_unpermute`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.model_executor.layers.fused_moe.moe_fused_mul_sum`, `vllm.model_executor.layers.fused_moe.moe_permute_unpermute`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
