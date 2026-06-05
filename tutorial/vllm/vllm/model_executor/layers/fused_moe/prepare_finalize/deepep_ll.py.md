# deepep_ll.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/prepare_finalize/deepep_ll.py`
- **Repository**: vllm-project/vllm
- **Purpose**: distributed MoE preparation/finalization logic; fused Mixture-of-Experts routing, kernels, and runtime helpers / 分布式 MoE 的准备/收尾逻辑；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-31 — imports and setup
```python
from collections.abc import Callable

import deep_ep
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import envs
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.config import FusedMoEQuantConfig
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceDelegate,
)
from vllm.model_executor.layers.fused_moe.utils import (
    moe_kernel_quantize_input,
    normalize_batched_scales_shape,
)
from vllm.v1.worker.ubatching import (
    dbo_current_ubatch_id,
    dbo_enabled,
    dbo_maybe_run_recv_hook,
)

logger = init_logger(__name__)

# DeepEP kernels quantize dispatch inputs in 128 element chunks.
DEEPEP_QUANT_BLOCK_SIZE = 128
DEEPEP_QUANT_BLOCK_SHAPE = [DEEPEP_QUANT_BLOCK_SIZE, DEEPEP_QUANT_BLOCK_SIZE]

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`, `DEEPEP_QUANT_BLOCK_SIZE`, `DEEPEP_QUANT_BLOCK_SHAPE`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`, `DEEPEP_QUANT_BLOCK_SIZE`, `DEEPEP_QUANT_BLOCK_SHAPE`。

### Lines 34-49 — function `dequant_fp8`
```python
def dequant_fp8(
    expert_x_fp8: torch.Tensor, expert_x_scales: torch.Tensor
) -> torch.Tensor:
    """
    Return dequantized tensor in fp32
    """
    # TODO (varun) : Optimize leverage num_tokens_per_expert counts
    assert expert_x_fp8.is_contiguous()
    expert_x_scales = expert_x_scales.contiguous()
    num_experts = expert_x_fp8.size(0)

    expert_x_fp32 = expert_x_fp8.to(torch.float32).view(
        num_experts, -1, DEEPEP_QUANT_BLOCK_SIZE
    )
    expert_x_scales = expert_x_scales.view(num_experts, -1, 1)
    return (expert_x_fp32 * expert_x_scales).view(expert_x_fp8.size())
```
**EN:** This function defines `dequant_fp8`. Return dequantized tensor in fp32 The main inputs are `expert_x_fp8`, `expert_x_scales`. Key calls include `expert_x_fp8.is_contiguous`, `expert_x_scales.contiguous`, `expert_x_fp8.size`, `expert_x_fp8.to.view`, `expert_x_scales.view`, `view`. It writes or updates `expert_x_scales`, `num_experts`, `expert_x_fp32`.
**CN:** 该函数定义 `dequant_fp8`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `expert_x_fp8`, `expert_x_scales`。 关键调用包括 `expert_x_fp8.is_contiguous`, `expert_x_scales.contiguous`, `expert_x_fp8.size`, `expert_x_fp8.to.view`, `expert_x_scales.view`, `view`。 它会写入或更新 `expert_x_scales`, `num_experts`, `expert_x_fp32`。

### Lines 52-448 — class `DeepEPLLPrepareAndFinalize`
```python
class DeepEPLLPrepareAndFinalize(mk.FusedMoEPrepareAndFinalizeModular):
    """
    Prepare/Finalize using DeepEP low-latency kernels.
    """

    # DeepEP low-latency kernels are compiled only for certain
    # specific hidden sizes.
    # NOTE: Keep this list sorted, maybe_roundup_layer_hidden_size depends
    # on it.
    SUPPORTED_HIDDEN_SIZES = [2048, 2560, 3072, 4096, 5120, 6144, 7168, 8192]

    @staticmethod
    def maybe_roundup_layer_hidden_size(hidden_size: int) -> int:
        # Round up hidden size to the closest supported hidden size.
# ... omitted for brevity ...
            do_async=False,
        )
```
**EN:** This class defines `DeepEPLLPrepareAndFinalize`. It inherits from `mk.FusedMoEPrepareAndFinalizeModular`. Prepare/Finalize using DeepEP low-latency kernels. Important methods include `__init__`, `prepare`, `finalize`, `maybe_roundup_layer_hidden_size`, `post_init_setup`, `num_dispatchers`. Key calls include `len`, `all`, `ValueError`, `super.__init__`, `self.topk_indices_dtype`, `_maybe_cast`. It writes or updates `SUPPORTED_HIDDEN_SIZES`, `_supported_hs`, `num_supported_hs`, `buffer`, `max_tokens_per_rank`, `use_fp8_dispatch`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `DeepEPLLPrepareAndFinalize`。 它继承自 `mk.FusedMoEPrepareAndFinalizeModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `prepare`, `finalize`, `maybe_roundup_layer_hidden_size`, `post_init_setup`, `num_dispatchers`。 关键调用包括 `len`, `all`, `ValueError`, `super.__init__`, `self.topk_indices_dtype`, `_maybe_cast`。 它会写入或更新 `SUPPORTED_HIDDEN_SIZES`, `_supported_hs`, `num_supported_hs`, `buffer`, `max_tokens_per_rank`, `use_fp8_dispatch`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 85-120 — method `DeepEPLLPrepareAndFinalize.__init__`
```python
    def __init__(
        self,
        buffer: deep_ep.Buffer,
        max_tokens_per_rank: int,
        num_dispatchers: int,
        use_fp8_dispatch: bool = False,
        global_to_physical: torch.Tensor | None = None,
        physical_to_global: torch.Tensor | None = None,
        local_expert_global_ids: torch.Tensor | None = None,
    ):
        super().__init__()

        self.buffer = buffer
        self.max_tokens_per_rank = max_tokens_per_rank
        self.use_fp8_dispatch = use_fp8_dispatch
        # The dispatch function returns a handle that the combine function
        # requires. We store the handle here so it is available to the
        # combine function.
        self.handles: list[tuple | None] = [None, None]
        self.num_dispatchers_ = num_dispatchers

        topk_indices_dtype = self.topk_indices_dtype()

        def _maybe_cast(tensor: torch.Tensor | None) -> torch.Tensor | None:
            if tensor is None or topk_indices_dtype is None:
                return tensor
            return tensor.to(dtype=topk_indices_dtype)

        self.global_to_physical = _maybe_cast(global_to_physical)
        self.physical_to_global = _maybe_cast(physical_to_global)
        self.local_expert_global_ids = _maybe_cast(local_expert_global_ids)

        # We don't have enough information to determine if we should dispatch
        # activation scales in a packed ue8m0 format during object construction
        # time. This setting is handled by post_init_setup.
        self.use_ue8m0_dispatch = False
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `buffer`, `max_tokens_per_rank`, `num_dispatchers`, `use_fp8_dispatch`, `global_to_physical`, `physical_to_global`. Key calls include `super.__init__`, `self.topk_indices_dtype`, `_maybe_cast`, `tensor.to`, `super`. It writes or updates `buffer`, `max_tokens_per_rank`, `use_fp8_dispatch`, `handles`, `num_dispatchers_`, `topk_indices_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `buffer`, `max_tokens_per_rank`, `num_dispatchers`, `use_fp8_dispatch`, `global_to_physical`, `physical_to_global`。 关键调用包括 `super.__init__`, `self.topk_indices_dtype`, `_maybe_cast`, `tensor.to`, `super`。 它会写入或更新 `buffer`, `max_tokens_per_rank`, `use_fp8_dispatch`, `handles`, `num_dispatchers_`, `topk_indices_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 344-370 — method `DeepEPLLPrepareAndFinalize.prepare`
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
        hook, receiver = self.prepare_async(
            a1,
            topk_weights,
            topk_ids,
            num_experts,
            expert_map,
            apply_router_weight_on_input,
            quant_config,
        )
        hook()
        return receiver()
```
**EN:** This method defines `prepare`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`. Key calls include `self.prepare_async`, `hook`, `receiver`, `NotImplementedError`. It writes or updates `hook`, `receiver`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `prepare`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a1`, `topk_weights`, `topk_ids`, `num_experts`, `expert_map`, `apply_router_weight_on_input`。 关键调用包括 `self.prepare_async`, `hook`, `receiver`, `NotImplementedError`。 它会写入或更新 `hook`, `receiver`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 431-448 — method `DeepEPLLPrepareAndFinalize.finalize`
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
        self._finalize(
            output,
            fused_expert_output,
            topk_weights,
            topk_ids,
            apply_router_weight_on_input,
            weight_and_reduce_impl,
            do_async=False,
        )
```
**EN:** This method defines `finalize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`. Key calls include `self._finalize`.
**CN:** 该方法定义 `finalize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `fused_expert_output`, `topk_weights`, `topk_ids`, `apply_router_weight_on_input`, `weight_and_reduce_impl`。 关键调用包括 `self._finalize`。

### Lines 63-83 — method `DeepEPLLPrepareAndFinalize.maybe_roundup_layer_hidden_size`
```python
    @staticmethod
    def maybe_roundup_layer_hidden_size(hidden_size: int) -> int:
        # Round up hidden size to the closest supported hidden size.
        _supported_hs = DeepEPLLPrepareAndFinalize.SUPPORTED_HIDDEN_SIZES
        # Check sorted
        num_supported_hs = len(_supported_hs)
        assert all(
            [
                _supported_hs[i] < _supported_hs[i + 1]
                for i in range(num_supported_hs - 1)
            ]
        )

        for x in _supported_hs:
            if x >= hidden_size:
                return x

        raise ValueError(
            f"Hidden Size {hidden_size} is greater than the "
            f"maximum supported hidden size {_supported_hs[-1]}"
        )
```
**EN:** This method defines `maybe_roundup_layer_hidden_size`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_size`. Key calls include `len`, `all`, `ValueError`, `range`. It writes or updates `_supported_hs`, `num_supported_hs`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该方法定义 `maybe_roundup_layer_hidden_size`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_size`。 关键调用包括 `len`, `all`, `ValueError`, `range`。 它会写入或更新 `_supported_hs`, `num_supported_hs`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 122-137 — method `DeepEPLLPrepareAndFinalize.post_init_setup`
```python
    def post_init_setup(self, fused_experts: mk.FusedMoEExperts):
        if not fused_experts.supports_packed_ue8m0_act_scales():
            # Early exit.
            return

        if self.use_fp8_dispatch:
            logger.debug_once(
                "Update DeepEPLLPrepareFinalize to do packed ue8m0 scales dispatch."
            )
            self.use_ue8m0_dispatch = True
        else:
            logger.warning_once(
                "DeepEPLLPrepareAndFinalize is setup to dispatch raw/unquantized "
                f"activations despite ({fused_experts.__class__.__name__}) being able "
                "to support quantized activations.",
            )
```
**EN:** This method defines `post_init_setup`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `fused_experts`. Key calls include `fused_experts.supports_packed_ue8m0_act_scales`, `logger.debug_once`, `logger.warning_once`. It writes or updates `use_ue8m0_dispatch`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `post_init_setup`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `fused_experts`。 关键调用包括 `fused_experts.supports_packed_ue8m0_act_scales`, `logger.debug_once`, `logger.warning_once`。 它会写入或更新 `use_ue8m0_dispatch`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 139-140 — method `DeepEPLLPrepareAndFinalize.num_dispatchers`
```python
    def num_dispatchers(self) -> int:
        return self.num_dispatchers_
```
**EN:** This method defines `num_dispatchers`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `num_dispatchers`。 它管理专家分发、融合内核或 MoE 特有的张量流。

## Key Concepts / 关键概念
- [EN] Distributed moe preparation/finalization logic / [CN] 分布式 MoE 的准备/收尾逻辑
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `dequant_fp8`, `DeepEPLLPrepareAndFinalize` / [CN] 核心符号：`dequant_fp8`, `DeepEPLLPrepareAndFinalize`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `deep_ep`, `torch` / **外部依赖**: `collections.abc`, `deep_ep`, `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.v1.worker.ubatching` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.v1.worker.ubatching`
