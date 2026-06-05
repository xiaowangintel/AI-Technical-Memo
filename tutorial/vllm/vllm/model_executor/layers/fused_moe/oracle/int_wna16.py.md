# int_wna16.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/oracle/int_wna16.py`
- **Repository**: vllm-project/vllm
- **Purpose**: reference/oracle quantization behavior for MoE kernels; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 内核的参考量化行为；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-32 — imports and setup
```python
from enum import Enum
from typing import TYPE_CHECKING

import torch

import vllm._custom_ops as ops
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.experts.marlin_moe import (
    BatchedMarlinExperts,
    MarlinExperts,
)
from vllm.model_executor.layers.quantization.base_config import QuantizationConfig
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    marlin_act_int8_process_scales,
    marlin_moe_permute_scales,
    marlin_permute_bias,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
)

if TYPE_CHECKING:
    from vllm.model_executor.layers.quantization.auto_gptq import AutoGPTQConfig

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`. The body uses conditional branches to cover different runtime cases.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 35-37 — class `WNA16MoEBackend`
```python
class WNA16MoEBackend(Enum):
    MARLIN = "MARLIN"
    BATCHED_MARLIN = "BATCHED_MARLIN"
```
**EN:** This class defines `WNA16MoEBackend`. It inherits from `Enum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `MARLIN`, `BATCHED_MARLIN`.
**CN:** 该类定义了 `WNA16MoEBackend`。 它继承自 `Enum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `MARLIN`, `BATCHED_MARLIN`。

### Lines 40-59 — function `backend_to_kernel_cls`
```python
def backend_to_kernel_cls(
    backend: WNA16MoEBackend,
) -> list[type[mk.FusedMoEExperts]]:
    """Return the experts class for the given backend, or None for NONE."""
    if backend == WNA16MoEBackend.MARLIN:
        from vllm.model_executor.layers.fused_moe.experts.marlin_moe import (
            MarlinExperts,
        )

        return [MarlinExperts]

    elif backend == WNA16MoEBackend.BATCHED_MARLIN:
        from vllm.model_executor.layers.fused_moe.experts.marlin_moe import (
            BatchedMarlinExperts,
        )

        return [BatchedMarlinExperts]

    else:
        raise ValueError(f"Unknown WNA16 MoE backend: {backend.value}")
```
**EN:** This function defines `backend_to_kernel_cls`. Return the experts class for the given backend, or None for NONE. The main inputs are `backend`. Key calls include `ValueError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `backend_to_kernel_cls`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `backend`。 关键调用包括 `ValueError`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 62-70 — function `_get_priority_backends`
```python
def _get_priority_backends() -> list[WNA16MoEBackend]:
    """
    Get available backends in priority order based on platform and config.
    """
    _AVAILABLE_BACKENDS = [
        WNA16MoEBackend.MARLIN,
        WNA16MoEBackend.BATCHED_MARLIN,
    ]
    return _AVAILABLE_BACKENDS
```
**EN:** This function defines `_get_priority_backends`. Get available backends in priority order based on platform and config. It writes or updates `_AVAILABLE_BACKENDS`.
**CN:** 该函数定义 `_get_priority_backends`。 该函数/方法的文档字符串直接说明了它的职责。 它会写入或更新 `_AVAILABLE_BACKENDS`。

### Lines 73-143 — function `select_wna16_moe_backend`
```python
def select_wna16_moe_backend(
    config: FusedMoEConfig,
    weight_key: QuantKey,
    weight_bits: int,
) -> tuple[WNA16MoEBackend, type[mk.FusedMoEExperts]]:
    """Select the WNA16 MoE backend.

    Args:
        config: the shared ``FusedMoEConfig`` for this layer.
        weight_bits: quantization bit-width (4 or 8). 8-bit weights are not
            supported by the modular Marlin kernel, so ``NONE`` is returned.

    Returns:
        A tuple of (``WNA16MoEBackend``, experts class or ``None``).
    """

    activation_format = (
        mk.FusedMoEActivationFormat.BatchedExperts
        if config.moe_parallel_config.use_batched_activation_format
        else mk.FusedMoEActivationFormat.Standard
    )

    def _make_log_backend(backend: WNA16MoEBackend):
        return f"Using '{backend.value}' WNA16 MoE backend."

    def _make_log_unsupported(backend: WNA16MoEBackend, reason: str | None) -> str:
        if reason:
            return (
                f"WNA16 MoE backend '{backend.value}' does not support the "
                f"deployment configuration since {reason}."
            )
        return (
            f"WNA16 MoE backend '{backend.value}' does not support the "
            "deployment configuration."
        )

    def _return_or_raise(
        backend: WNA16MoEBackend,
        config: FusedMoEConfig,
        weight_key: QuantKey | None,
        activation_key: QuantKey | None,
        activation_format: mk.FusedMoEActivationFormat,
    ) -> tuple[WNA16MoEBackend, type[mk.FusedMoEExperts]]:
        reason: str | None = None
# ... omitted for brevity ...
        "No WNA16 MoE backend supports the deployment configuration."
    )
```
**EN:** This function defines `select_wna16_moe_backend`. Select the WNA16 MoE backend. The main inputs are `config`, `weight_key`, `weight_bits`. Key calls include `_get_priority_backends`, `NotImplementedError`, `backend_to_kernel_cls`, `ValueError`, `k_cls.is_supported_config`, `_make_log_unsupported`. It writes or updates `activation_format`, `AVAILABLE_BACKENDS`, `reason`, `activation_key`, `supported`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `select_wna16_moe_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `config`, `weight_key`, `weight_bits`。 关键调用包括 `_get_priority_backends`, `NotImplementedError`, `backend_to_kernel_cls`, `ValueError`, `k_cls.is_supported_config`, `_make_log_unsupported`。 它会写入或更新 `activation_format`, `AVAILABLE_BACKENDS`, `reason`, `activation_key`, `supported`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 146-205 — function `make_wna16_moe_kernel`
```python
def make_wna16_moe_kernel(
    moe_quant_config: FusedMoEQuantConfig,
    moe_config: FusedMoEConfig,
    experts_cls: type[mk.FusedMoEExperts] | None,
    layer: torch.nn.Module,
    is_k_full: bool,
    w13_g_idx: torch.Tensor | None,
    w2_g_idx: torch.Tensor | None,
    w13_g_idx_sort_indices: torch.Tensor | None,
    w2_g_idx_sort_indices: torch.Tensor | None,
    routing_tables: tuple[torch.Tensor, torch.Tensor, torch.Tensor] | None = None,
) -> mk.FusedMoEKernel:
    # Currently, we only support MarlinExperts and BatchedMarlinExperts
    assert experts_cls in (MarlinExperts, BatchedMarlinExperts)

    from vllm.model_executor.layers.fused_moe.all2all_utils import (
        maybe_make_prepare_finalize,
    )

    prepare_finalize = maybe_make_prepare_finalize(
        moe=moe_config,
        quant_config=moe_quant_config,
        routing_tables=routing_tables,
        allow_new_interface=True,
    )
    assert prepare_finalize is not None
    assert isinstance(prepare_finalize, mk.FusedMoEPrepareAndFinalizeModular)

    if prepare_finalize.activation_format == mk.FusedMoEActivationFormat.BatchedExperts:
        assert experts_cls == BatchedMarlinExperts
        max_num_tokens = prepare_finalize.max_num_tokens_per_rank()
        assert max_num_tokens is not None
        experts: mk.FusedMoEExperts = BatchedMarlinExperts(
            max_num_tokens=max_num_tokens,
            num_dispatchers=prepare_finalize.num_dispatchers(),
            moe_config=moe_config,
            quant_config=moe_quant_config,
            w13_g_idx=w13_g_idx,
            w2_g_idx=w2_g_idx,
            w13_g_idx_sort_indices=w13_g_idx_sort_indices,
            w2_g_idx_sort_indices=w2_g_idx_sort_indices,
            is_k_full=is_k_full,
        )
    else:
# ... omitted for brevity ...
        inplace=not moe_config.disable_inplace,
    )
```
**EN:** This function defines `make_wna16_moe_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `moe_quant_config`, `moe_config`, `experts_cls`, `layer`, `is_k_full`, `w13_g_idx`. Key calls include `maybe_make_prepare_finalize`, `isinstance`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `BatchedMarlinExperts`, `MarlinExperts`. It writes or updates `prepare_finalize`, `max_num_tokens`, `experts`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_wna16_moe_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `moe_quant_config`, `moe_config`, `experts_cls`, `layer`, `is_k_full`, `w13_g_idx`。 关键调用包括 `maybe_make_prepare_finalize`, `isinstance`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `BatchedMarlinExperts`, `MarlinExperts`。 它会写入或更新 `prepare_finalize`, `max_num_tokens`, `experts`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 213-375 — function `_process_weights_marlin`
```python
def _process_weights_marlin(
    layer: torch.nn.Module,
    quant_config: "AutoGPTQConfig",
    input_dtype: torch.dtype | None,
    w13_qweight: torch.Tensor,
    w2_qweight: torch.Tensor,
    w13_scales: torch.Tensor,
    w2_scales: torch.Tensor,
    w13_g_idx: torch.Tensor,
    w2_g_idx: torch.Tensor,
    w13_bias: torch.Tensor | None = None,
    w2_bias: torch.Tensor | None = None,
) -> tuple[
    torch.Tensor,  # w13_qweight
    torch.Tensor,  # w2_qweight
    torch.Tensor,  # w13_scales
    torch.Tensor,  # w2_scales
    torch.Tensor,  # w13_g_idx
    torch.Tensor,  # w2_g_idx
    torch.Tensor,  # w13_g_idx_sort_indices
    torch.Tensor,  # w2_g_idx_sort_indices
    torch.Tensor | None,  # w13_input_global_scale
    torch.Tensor | None,  # w2_input_global_scale
    torch.Tensor | None,  # w13_bias
    torch.Tensor | None,  # w2_bias
]:
    """Standard Marlin weight post-processing shared by MARLIN and
    BATCHED_MARLIN backends.

    Steps
    -----
    1. Optional FP8 preprocessing of packed weights / scales.
    2. Sort / reset g_idx tensors for act-order handling.
    3. Repack weights via ``gptq_marlin_moe_repack``.
    4. Permute scales (and optionally extract INT8 global scales).
    5. Permute bias tensors.
    """
    is_a_8bit = input_dtype is not None and input_dtype.itemsize == 1

    marlin_w13_qweight: torch.Tensor
    marlin_w2_qweight: torch.Tensor
    marlin_w13_scales: torch.Tensor
    marlin_w2_scales: torch.Tensor
    w13_g_idx_sort_indices: torch.Tensor | None = None
# ... omitted for brevity ...
        w2_bias_out,
    )
```
**EN:** This function defines `_process_weights_marlin`. Standard Marlin weight post-processing shared by MARLIN and BATCHED_MARLIN backends. The main inputs are `layer`, `quant_config`, `input_dtype`, `w13_qweight`, `w2_qweight`, `w13_scales`. Key calls include `ops.gptq_marlin_moe_repack`, `marlin_moe_permute_scales`, `ops.marlin_int4_fp8_preprocess`, `torch.empty_like`, `range`, `torch.nn.Parameter`. It writes or updates `is_a_8bit`, `marlin_w13_qweight`, `marlin_w2_qweight`, `marlin_w13_scales`, `marlin_w2_scales`, `w13_g_idx_sort_indices`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `_process_weights_marlin`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `layer`, `quant_config`, `input_dtype`, `w13_qweight`, `w2_qweight`, `w13_scales`。 关键调用包括 `ops.gptq_marlin_moe_repack`, `marlin_moe_permute_scales`, `ops.marlin_int4_fp8_preprocess`, `torch.empty_like`, `range`, `torch.nn.Parameter`。 它会写入或更新 `is_a_8bit`, `marlin_w13_qweight`, `marlin_w2_qweight`, `marlin_w13_scales`, `marlin_w2_scales`, `w13_g_idx_sort_indices`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 378-443 — function `convert_to_wna16_moe_kernel_format`
```python
def convert_to_wna16_moe_kernel_format(
    backend: WNA16MoEBackend,
    layer: torch.nn.Module,
    quant_config: QuantizationConfig,
    input_dtype: torch.dtype | None,
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    w13_g_idx: torch.Tensor,
    w2_g_idx: torch.Tensor,
    w13_bias: torch.Tensor | None = None,
    w2_bias: torch.Tensor | None = None,
) -> tuple[
    torch.Tensor,  # w13_qweight
    torch.Tensor,  # w2_qweight
    torch.Tensor,  # w13_scales
    torch.Tensor,  # w2_scales
    torch.Tensor | None,  # w13_g_idx
    torch.Tensor | None,  # w2_g_idx
    torch.Tensor | None,  # w13_g_idx_sort_indices
    torch.Tensor | None,  # w2_g_idx_sort_indices
    torch.Tensor | None,  # w13_input_global_scale
    torch.Tensor | None,  # w2_input_global_scale
    torch.Tensor | None,  # w13_bias
    torch.Tensor | None,  # w2_bias
]:
    """Dispatch weight post-processing to the appropriate per-backend handler.

    To add a new backend, implement a ``_process_weights_<name>`` helper and
    add a branch here.

    Args:
        backend: the selected ``WNA16MoEBackend``.
        layer: the ``FusedMoE`` layer whose parameters are being prepared.
        quant_config: the ``QuantizationConfig`` for this layer.
        input_dtype: optional activation dtype, usually should be 16 bit.
    """
    if backend in (
        WNA16MoEBackend.MARLIN,
        WNA16MoEBackend.BATCHED_MARLIN,
    ):
        from vllm.model_executor.layers.quantization.auto_gptq import (
            AutoGPTQConfig,
# ... omitted for brevity ...
    else:
        raise ValueError(f"Unsupported wna16 MoE backend: {backend.value}")
```
**EN:** This function defines `convert_to_wna16_moe_kernel_format`. Dispatch weight post-processing to the appropriate per-backend handler. The main inputs are `backend`, `layer`, `quant_config`, `input_dtype`, `w13`, `w2`. Key calls include `_process_weights_marlin`, `ValueError`, `isinstance`, `TypeError`, `type`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `convert_to_wna16_moe_kernel_format`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `backend`, `layer`, `quant_config`, `input_dtype`, `w13`, `w2`。 关键调用包括 `_process_weights_marlin`, `ValueError`, `isinstance`, `TypeError`, `type`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Reference/oracle quantization behavior for moe kernels / [CN] MoE 内核的参考量化行为
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `WNA16MoEBackend`, `backend_to_kernel_cls`, `_get_priority_backends`, `select_wna16_moe_backend` / [CN] 核心符号：`WNA16MoEBackend`, `backend_to_kernel_cls`, `_get_priority_backends`, `select_wna16_moe_backend`

## Dependencies / 依赖关系
- **External**: `enum`, `typing`, `torch` / **外部依赖**: `enum`, `typing`, `torch`
- **Internal**: `vllm._custom_ops`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.marlin_moe`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.auto_gptq` / **内部依赖**: `vllm._custom_ops`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.marlin_moe`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.auto_gptq`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
