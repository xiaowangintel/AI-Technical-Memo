# unquantized.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/oracle/unquantized.py`
- **Repository**: vllm-project/vllm
- **Purpose**: reference/oracle quantization behavior for MoE kernels; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 内核的参考量化行为；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-29 — imports and setup
```python
from enum import Enum

import torch
from torch.nn import Module

import vllm.envs as envs
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm._aiter_ops import rocm_aiter_ops
from vllm.config.kernel import MoEBackend
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.quantization.utils.flashinfer_utils import (
    FlashinferMoeBackend,
    convert_moe_weights_to_flashinfer_trtllm_block_layout,
    get_flashinfer_moe_backend,
    swap_w13_to_w31,
)
from vllm.platforms import current_platform

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 32-41 — class `UnquantizedMoeBackend`
```python
class UnquantizedMoeBackend(Enum):
    FLASHINFER_TRTLLM = "FlashInfer TRTLLM"
    FLASHINFER_CUTLASS = "FlashInfer CUTLASS"
    AITER = "ROCm AITER"
    TRITON = "TRITON"
    BATCHED_TRITON = "BATCHED_TRITON"
    CPU = "CPU"
    XPU = "XPU"
    TPU = "TPU"
    OOT = "OOT"
```
**EN:** This class defines `UnquantizedMoeBackend`. It inherits from `Enum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `FLASHINFER_TRTLLM`, `FLASHINFER_CUTLASS`, `AITER`, `TRITON`, `BATCHED_TRITON`, `CPU`.
**CN:** 该类定义了 `UnquantizedMoeBackend`。 它继承自 `Enum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `FLASHINFER_TRTLLM`, `FLASHINFER_CUTLASS`, `AITER`, `TRITON`, `BATCHED_TRITON`, `CPU`。

### Lines 44-81 — function `_get_priority_backends`
```python
def _get_priority_backends(moe_config: FusedMoEConfig) -> list[UnquantizedMoeBackend]:
    """
    Get available backends in priority order based on platform and config.

    This function can be extended to become more complex as needed.
    """

    def _move_to_back(
        backends: list[UnquantizedMoeBackend],
        backend: UnquantizedMoeBackend,
    ) -> None:
        backends.append(backends.pop(backends.index(backend)))

    if current_platform.is_rocm():
        _AVAILABLE_BACKENDS = [
            UnquantizedMoeBackend.AITER,
            UnquantizedMoeBackend.TRITON,
            UnquantizedMoeBackend.BATCHED_TRITON,
        ]
    elif current_platform.is_cuda():
        _AVAILABLE_BACKENDS = [
            UnquantizedMoeBackend.FLASHINFER_TRTLLM,
            UnquantizedMoeBackend.FLASHINFER_CUTLASS,
            UnquantizedMoeBackend.TRITON,
            UnquantizedMoeBackend.BATCHED_TRITON,
        ]

        # HACK: Qwen3.5 has crash with FLASHINFER_CUTLASS BF16 if DEP.
        # Updating the oracle querying logic is out of the scope of this
        # PR. Need to fix the kernel or update structure in follow up.
        if moe_config.moe_parallel_config.dp_size > 1:
            _move_to_back(_AVAILABLE_BACKENDS, UnquantizedMoeBackend.FLASHINFER_CUTLASS)

    elif current_platform.is_xpu():
        _AVAILABLE_BACKENDS = [UnquantizedMoeBackend.XPU]
    elif current_platform.is_cpu():
        _AVAILABLE_BACKENDS = [UnquantizedMoeBackend.CPU]
    return _AVAILABLE_BACKENDS
```
**EN:** This function defines `_get_priority_backends`. Get available backends in priority order based on platform and config. The main inputs are `moe_config`. Key calls include `current_platform.is_rocm`, `backends.append`, `current_platform.is_cuda`, `backends.pop`, `current_platform.is_xpu`, `backends.index`. It writes or updates `_AVAILABLE_BACKENDS`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_get_priority_backends`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `moe_config`。 关键调用包括 `current_platform.is_rocm`, `backends.append`, `current_platform.is_cuda`, `backends.pop`, `current_platform.is_xpu`, `backends.index`。 它会写入或更新 `_AVAILABLE_BACKENDS`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 84-128 — function `backend_to_kernel_cls`
```python
def backend_to_kernel_cls(
    backend: UnquantizedMoeBackend,
) -> type[mk.FusedMoEExperts]:
    if backend == UnquantizedMoeBackend.FLASHINFER_TRTLLM:
        from vllm.model_executor.layers.fused_moe.experts.trtllm_bf16_moe import (
            TrtLlmBf16Experts,
        )

        return TrtLlmBf16Experts

    elif backend == UnquantizedMoeBackend.FLASHINFER_CUTLASS:
        from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutlass_moe import (  # noqa: E501
            FlashInferExperts,
        )

        return FlashInferExperts

    elif backend == UnquantizedMoeBackend.AITER:
        from vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe import (
            AiterExperts,
        )

        return AiterExperts

    elif backend == UnquantizedMoeBackend.TRITON:
        from vllm.model_executor.layers.fused_moe.experts.triton_moe import (
            TritonExperts,
        )

        return TritonExperts

    elif backend == UnquantizedMoeBackend.BATCHED_TRITON:
        from vllm.model_executor.layers.fused_moe.experts.fused_batched_moe import (
            BatchedTritonExperts,
        )

        return BatchedTritonExperts

    elif backend == UnquantizedMoeBackend.XPU:
        from vllm.model_executor.layers.fused_moe.experts.xpu_moe import XPUExperts

        return XPUExperts

    else:
        raise ValueError(f"Unknown unquantized MoE backend: {backend.value}")
```
**EN:** This function defines `backend_to_kernel_cls`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `backend`. Key calls include `ValueError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `backend_to_kernel_cls`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `backend`。 关键调用包括 `ValueError`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 131-144 — function `map_unquantized_backend`
```python
def map_unquantized_backend(runner_backend: MoEBackend) -> UnquantizedMoeBackend:
    """Map user's MoEBackend to UnquantizedMoeBackend."""
    mapping = {
        "triton": UnquantizedMoeBackend.TRITON,
        "flashinfer_trtllm": UnquantizedMoeBackend.FLASHINFER_TRTLLM,
        "flashinfer_cutlass": UnquantizedMoeBackend.FLASHINFER_CUTLASS,
        "aiter": UnquantizedMoeBackend.AITER,
    }
    if backend := mapping.get(runner_backend):
        return backend
    raise ValueError(
        f"moe_backend='{runner_backend}' is not supported for unquantized MoE. "
        f"Expected one of {list(mapping.keys())}."
    )
```
**EN:** This function defines `map_unquantized_backend`. Map user's MoEBackend to UnquantizedMoeBackend. The main inputs are `runner_backend`. Key calls include `ValueError`, `mapping.get`, `list`, `mapping.keys`. It writes or updates `mapping`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `map_unquantized_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `runner_backend`。 关键调用包括 `ValueError`, `mapping.get`, `list`, `mapping.keys`。 它会写入或更新 `mapping`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 147-292 — function `select_unquantized_moe_backend`
```python
def select_unquantized_moe_backend(
    moe_config: FusedMoEConfig,
) -> tuple[UnquantizedMoeBackend, type[mk.FusedMoEExperts] | None]:
    """
    Select the primary Unquantized MoE backend.
    Note: Shape-specific fallbacks may still occur at runtime.
    """

    if current_platform.is_cpu():
        # TODO: migrate to MK structure.
        return UnquantizedMoeBackend.CPU, None

    if current_platform.is_tpu():
        return UnquantizedMoeBackend.TPU, None

    if current_platform.is_out_of_tree():
        return UnquantizedMoeBackend.OOT, None

    if moe_config.is_lora_enabled:
        return UnquantizedMoeBackend.TRITON, backend_to_kernel_cls(
            UnquantizedMoeBackend.TRITON
        )

    # NOTE: the kernels are selected in the following order.
    AVAILABLE_BACKENDS = _get_priority_backends(moe_config)

    # NOTE(rob): We need to peak into the P/F selection to determine
    # if we are using the batched or standard expert format, which
    # if not ideal. Once we unify TP + DP/EP, we can select P/F first.
    activation_format = (
        mk.FusedMoEActivationFormat.BatchedExperts
        if moe_config.moe_parallel_config.use_batched_activation_format
        else mk.FusedMoEActivationFormat.Standard
    )

    def _make_log_backend(backend: UnquantizedMoeBackend) -> str:
        available_strs = [b.value for b in AVAILABLE_BACKENDS]
        return (
            f"Using {backend.value} Unquantized MoE backend out "
            f"of potential backends: {available_strs}."
        )

    def _make_log_unsupported(
        backend: UnquantizedMoeBackend, reason: str | None
# ... omitted for brevity ...
        "No Unquantized MoE backend supports the deployment configuration."
    )
```
**EN:** This function defines `select_unquantized_moe_backend`. Select the primary Unquantized MoE backend. The main inputs are `moe_config`. Key calls include `current_platform.is_cpu`, `current_platform.is_tpu`, `current_platform.is_out_of_tree`, `_get_priority_backends`, `envs.is_set`, `NotImplementedError`. It writes or updates `AVAILABLE_BACKENDS`, `activation_format`, `runner_backend`, `available_strs`, `k_cls`, `supported`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `select_unquantized_moe_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `moe_config`。 关键调用包括 `current_platform.is_cpu`, `current_platform.is_tpu`, `current_platform.is_out_of_tree`, `_get_priority_backends`, `envs.is_set`, `NotImplementedError`。 它会写入或更新 `AVAILABLE_BACKENDS`, `activation_format`, `runner_backend`, `available_strs`, `k_cls`, `supported`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 295-320 — function `convert_to_unquantized_kernel_format`
```python
def convert_to_unquantized_kernel_format(
    unquantized_backend: UnquantizedMoeBackend,
    layer: Module,
    w13_weight: torch.Tensor,
    w2_weight: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    if unquantized_backend == UnquantizedMoeBackend.AITER:
        w13_weight, w2_weight = rocm_aiter_ops.shuffle_weights(w13_weight, w2_weight)

    elif unquantized_backend == UnquantizedMoeBackend.FLASHINFER_CUTLASS:
        if layer.moe_config.is_act_and_mul:
            # Swap halves to arrange as [w3; w1] (kernel expectation)
            # Non-gated MoE: w13 is a single projection, no need to swap.
            w13_weight = swap_w13_to_w31(w13_weight)

    elif unquantized_backend == UnquantizedMoeBackend.FLASHINFER_TRTLLM:
        # Swap halves to arrange as [w3; w1] (kernel expectation)
        w13_weight = swap_w13_to_w31(w13_weight)
        _cache_permute_indices: dict[torch.Size, torch.Tensor] = {}
        w13_weight, w2_weight = convert_moe_weights_to_flashinfer_trtllm_block_layout(
            _cache_permute_indices,
            w13_weight,
            w2_weight,
        )

    return w13_weight.contiguous(), w2_weight.contiguous()
```
**EN:** This function defines `convert_to_unquantized_kernel_format`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `unquantized_backend`, `layer`, `w13_weight`, `w2_weight`. Key calls include `rocm_aiter_ops.shuffle_weights`, `w13_weight.contiguous`, `w2_weight.contiguous`, `swap_w13_to_w31`, `convert_moe_weights_to_flashinfer_trtllm_block_layout`. It writes or updates `w13_weight`, `w2_weight`, `_cache_permute_indices`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `convert_to_unquantized_kernel_format`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `unquantized_backend`, `layer`, `w13_weight`, `w2_weight`。 关键调用包括 `rocm_aiter_ops.shuffle_weights`, `w13_weight.contiguous`, `w2_weight.contiguous`, `swap_w13_to_w31`, `convert_moe_weights_to_flashinfer_trtllm_block_layout`。 它会写入或更新 `w13_weight`, `w2_weight`, `_cache_permute_indices`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 323-365 — function `make_unquantized_moe_kernel`
```python
def make_unquantized_moe_kernel(
    quant_config: FusedMoEQuantConfig,
    moe_config: FusedMoEConfig,
    backend: UnquantizedMoeBackend,
    experts_cls: type[mk.FusedMoEExperts],
    routing_tables: tuple[torch.Tensor, torch.Tensor, torch.Tensor] | None = None,
) -> mk.FusedMoEKernel:
    # Create Prepare/Finalize
    is_monolithic = issubclass(experts_cls, mk.FusedMoEExpertsMonolithic)
    prepare_finalize = maybe_make_prepare_finalize(
        moe=moe_config,
        quant_config=quant_config,
        routing_tables=routing_tables,
        allow_new_interface=True,
        use_monolithic=is_monolithic,
    )
    assert prepare_finalize is not None

    logger.info_once("Using %s", prepare_finalize.__class__.__name__)

    # Create Experts
    if prepare_finalize.activation_format == mk.FusedMoEActivationFormat.BatchedExperts:
        max_num_tokens = prepare_finalize.max_num_tokens_per_rank()
        assert max_num_tokens is not None
        experts = experts_cls(
            moe_config=moe_config,
            quant_config=quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=prepare_finalize.num_dispatchers(),
        )
    else:
        experts = experts_cls(
            moe_config=moe_config,
            quant_config=quant_config,
        )

    kernel = mk.FusedMoEKernel(
        prepare_finalize,
        experts,
        inplace=(not moe_config.disable_inplace and not is_monolithic),
    )

    return kernel
```
**EN:** This function defines `make_unquantized_moe_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `quant_config`, `moe_config`, `backend`, `experts_cls`, `routing_tables`. Key calls include `issubclass`, `maybe_make_prepare_finalize`, `logger.info_once`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `experts_cls`. It writes or updates `is_monolithic`, `prepare_finalize`, `kernel`, `max_num_tokens`, `experts`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_unquantized_moe_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `quant_config`, `moe_config`, `backend`, `experts_cls`, `routing_tables`。 关键调用包括 `issubclass`, `maybe_make_prepare_finalize`, `logger.info_once`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `experts_cls`。 它会写入或更新 `is_monolithic`, `prepare_finalize`, `kernel`, `max_num_tokens`, `experts`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Reference/oracle quantization behavior for moe kernels / [CN] MoE 内核的参考量化行为
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `UnquantizedMoeBackend`, `_get_priority_backends`, `backend_to_kernel_cls`, `map_unquantized_backend` / [CN] 核心符号：`UnquantizedMoeBackend`, `_get_priority_backends`, `backend_to_kernel_cls`, `map_unquantized_backend`

## Dependencies / 依赖关系
- **External**: `enum`, `torch`, `torch.nn` / **外部依赖**: `enum`, `torch`, `torch.nn`
- **Internal**: `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm._aiter_ops`, `vllm.config.kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.platforms` / **内部依赖**: `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm._aiter_ops`, `vllm.config.kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.platforms`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
