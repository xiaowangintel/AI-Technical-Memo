# mxfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/oracle/mxfp4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: reference/oracle quantization behavior for MoE kernels; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 内核的参考量化行为；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-46 — imports and setup
```python
from enum import Enum
from typing import TYPE_CHECKING, Literal, Union

import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import envs
from vllm.config import get_current_vllm_config
from vllm.config.kernel import MoEBackend
from vllm.config.quantization import QuantizationConfigArgs
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe import (
    FusedMoEConfig,
)
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEQuantConfig,
    FusedMoEQuantDesc,
    RoutingMethodType,
    mxfp4_mxfp8_moe_quant_config,
    mxfp4_w4a8_moe_quant_config,
    mxfp4_w4a16_moe_quant_config,
    ocp_mx_moe_quant_config,
)
from vllm.model_executor.layers.quantization.utils.mxfp4_utils import _swizzle_mxfp4
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8Dynamic128Sym,
    kFp8StaticTensorSym,
    kMxfp4Static,
    kMxfp8Dynamic,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import all_close_1d
from vllm.platforms import current_platform
from vllm.utils.import_utils import has_triton_kernels
from vllm.utils.math_utils import round_up

if TYPE_CHECKING:
    from vllm.model_executor.layers.fused_moe import RoutedExperts


logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`. The body uses conditional branches to cover different runtime cases.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 59-87 — class `Mxfp4MoeBackend`
```python
class Mxfp4MoeBackend(Enum):
    NONE = "None"
    # DeepGEMM FP8xFP4 backend (SM100+)
    DEEPGEMM_MXFP4 = "DEEPGEMM_MXFP4"
    # FlashInfer TRTLLM backends
    FLASHINFER_TRTLLM_MXFP4_MXFP8 = "FLASHINFER_TRTLLM_MXFP4_MXFP8"
    FLASHINFER_TRTLLM_MXFP4_BF16 = "FLASHINFER_TRTLLM_MXFP4_BF16"
    # FlashInfer CUTLASS backends
    FLASHINFER_CUTLASS_MXFP4_MXFP8 = "FLASHINFER_CUTLASS_MXFP4_MXFP8"
    FLASHINFER_CUTLASS_MXFP4_BF16 = "FLASHINFER_CUTLASS_MXFP4_BF16"
    # Marlin
    BATCHED_MARLIN = "BATCHED_MARLIN"
    MARLIN = "MARLIN"
    # ROCm AITER backends
# ... omitted for brevity ...
    # Humming
    HUMMING = "HUMMING"
```
**EN:** This class defines `Mxfp4MoeBackend`. It inherits from `Enum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `NONE`, `DEEPGEMM_MXFP4`, `FLASHINFER_TRTLLM_MXFP4_MXFP8`, `FLASHINFER_TRTLLM_MXFP4_BF16`, `FLASHINFER_CUTLASS_MXFP4_MXFP8`, `FLASHINFER_CUTLASS_MXFP4_BF16`.
**CN:** 该类定义了 `Mxfp4MoeBackend`。 它继承自 `Enum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `NONE`, `DEEPGEMM_MXFP4`, `FLASHINFER_TRTLLM_MXFP4_MXFP8`, `FLASHINFER_TRTLLM_MXFP4_BF16`, `FLASHINFER_CUTLASS_MXFP4_MXFP8`, `FLASHINFER_CUTLASS_MXFP4_BF16`。

### Lines 91-94 — constant `AITER_BACKENDS`
```python
AITER_BACKENDS = (
    Mxfp4MoeBackend.AITER_MXFP4_BF16,
    Mxfp4MoeBackend.AITER_MXFP4_FP8,
)
```
**EN:** This constant defines `AITER_BACKENDS`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `AITER_BACKENDS`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 98-101 — constant `TRTLLM_BACKENDS`
```python
TRTLLM_BACKENDS = (
    Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_BF16,
    Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_MXFP8,
)
```
**EN:** This constant defines `TRTLLM_BACKENDS`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `TRTLLM_BACKENDS`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 103-106 — constant `TRITON_BACKENDS`
```python
TRITON_BACKENDS = (
    Mxfp4MoeBackend.TRITON,
    Mxfp4MoeBackend.TRITON_UNFUSED,
)
```
**EN:** This constant defines `TRITON_BACKENDS`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `TRITON_BACKENDS`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 109-216 — function `backend_to_kernel_cls`
```python
def backend_to_kernel_cls(
    backend: Mxfp4MoeBackend,
) -> list[type[mk.FusedMoEExperts]]:
    if backend == Mxfp4MoeBackend.DEEPGEMM_MXFP4:
        from vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe import (
            DeepGemmFP4Experts,
        )

        return [DeepGemmFP4Experts]

    elif backend in (
        Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_BF16,
        Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_MXFP8,
    ):
        from vllm.model_executor.layers.fused_moe.experts.trtllm_mxfp4_moe import (
            TrtLlmMxfp4ExpertsModular,
            TrtLlmMxfp4ExpertsMonolithic,
        )

        # NOTE: prefer Monolithic > Modular, so return Monolithic first.
        return [TrtLlmMxfp4ExpertsMonolithic, TrtLlmMxfp4ExpertsModular]

    elif backend in (
        Mxfp4MoeBackend.FLASHINFER_CUTLASS_MXFP4_BF16,
        Mxfp4MoeBackend.FLASHINFER_CUTLASS_MXFP4_MXFP8,
    ):
        from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutlass_moe import (  # noqa: E501
            FlashInferExperts,
        )

        return [FlashInferExperts]

    elif backend == Mxfp4MoeBackend.TRITON:
        from vllm.model_executor.layers.fused_moe.experts.gpt_oss_triton_kernels_moe import (  # noqa: E501
            OAITritonExperts,
            OAITritonMxfp4ExpertsMonolithic,
        )

        # NOTE: prefer Monolithic > Modular, so return Monolithic first.
        return [OAITritonMxfp4ExpertsMonolithic, OAITritonExperts]

    elif backend == Mxfp4MoeBackend.TRITON_UNFUSED:
        from vllm.model_executor.layers.fused_moe.experts.gpt_oss_triton_kernels_moe import (  # noqa: E501
            UnfusedOAITritonExperts,
# ... omitted for brevity ...
    else:
        raise ValueError(f"Unknown MXFP4 MoE backend: {backend.value}")
```
**EN:** This function defines `backend_to_kernel_cls`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `backend`. Key calls include `ValueError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `backend_to_kernel_cls`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `backend`。 关键调用包括 `ValueError`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 219-255 — function `map_mxfp4_backend`
```python
def map_mxfp4_backend(runner_backend: MoEBackend) -> list[Mxfp4MoeBackend]:
    """Map a moe_backend string to its candidate Mxfp4MoeBackends.

    Vendor families return all activation variants; the caller picks one
    via ``activation_key`` and ``is_supported_config``.
    """
    mapping: dict[str, list[Mxfp4MoeBackend]] = {
        "deep_gemm": [Mxfp4MoeBackend.DEEPGEMM_MXFP4],
        "flashinfer_trtllm": [
            Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_BF16,
            Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_MXFP8,
        ],
        "flashinfer_trtllm_afp8": [Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_MXFP8],
        "flashinfer_cutlass": [
            Mxfp4MoeBackend.FLASHINFER_CUTLASS_MXFP4_BF16,
            Mxfp4MoeBackend.FLASHINFER_CUTLASS_MXFP4_MXFP8,
        ],
        "flashinfer_cutlass_afp8": [Mxfp4MoeBackend.FLASHINFER_CUTLASS_MXFP4_MXFP8],
        "triton": [Mxfp4MoeBackend.TRITON],
        "triton_unfused": [Mxfp4MoeBackend.TRITON_UNFUSED],
        "humming": [Mxfp4MoeBackend.HUMMING],
        "marlin": [Mxfp4MoeBackend.MARLIN],
        "aiter": [
            Mxfp4MoeBackend.AITER_MXFP4_BF16,
            Mxfp4MoeBackend.AITER_MXFP4_FP8,
        ],
        "aiter_mxfp4_fp8": [Mxfp4MoeBackend.AITER_MXFP4_FP8],
        "xpu": [Mxfp4MoeBackend.XPU],
        "cpu": [Mxfp4MoeBackend.CPU],
        "emulation": [Mxfp4MoeBackend.EMULATION],
    }
    if backends := mapping.get(runner_backend):
        return backends
    raise ValueError(
        f"moe_backend='{runner_backend}' is not supported for MXFP4 MoE. "
        f"Expected one of {list(mapping.keys())}."
    )
```
**EN:** This function defines `map_mxfp4_backend`. Map a moe_backend string to its candidate Mxfp4MoeBackends. The main inputs are `runner_backend`. Key calls include `ValueError`, `mapping.get`, `list`, `mapping.keys`. It writes or updates `mapping`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `map_mxfp4_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `runner_backend`。 关键调用包括 `ValueError`, `mapping.get`, `list`, `mapping.keys`。 它会写入或更新 `mapping`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 258-277 — function `_get_priority_backends_for_gpt_oss`
```python
def _get_priority_backends_for_gpt_oss() -> list[Mxfp4MoeBackend]:
    """Available backends in priority order, BF16-act variant before
    activation-quantized variant within each vendor family."""
    _AVAILABLE_BACKENDS = [
        Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_BF16,
        Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_MXFP8,
        Mxfp4MoeBackend.AITER_MXFP4_BF16,
        Mxfp4MoeBackend.AITER_MXFP4_FP8,
        Mxfp4MoeBackend.TRITON,
        Mxfp4MoeBackend.FLASHINFER_CUTLASS_MXFP4_BF16,
        Mxfp4MoeBackend.FLASHINFER_CUTLASS_MXFP4_MXFP8,
        # TRITON_UNFUSED has bug with MTP support
        # TODO re-enable after kernel is fixed
        # TRITON_UNFUSED
        Mxfp4MoeBackend.MARLIN,
        Mxfp4MoeBackend.BATCHED_MARLIN,
        Mxfp4MoeBackend.XPU,
        Mxfp4MoeBackend.EMULATION,
    ]
    return _AVAILABLE_BACKENDS
```
**EN:** This function defines `_get_priority_backends_for_gpt_oss`. Available backends in priority order, BF16-act variant before activation-quantized variant within each vendor family. It writes or updates `_AVAILABLE_BACKENDS`.
**CN:** 该函数定义 `_get_priority_backends_for_gpt_oss`。 该函数/方法的文档字符串直接说明了它的职责。 它会写入或更新 `_AVAILABLE_BACKENDS`。

### Lines 280-297 — function `_get_priority_backends`
```python
def _get_priority_backends() -> list[Mxfp4MoeBackend]:
    """
    Get available backends in priority order. SM100+ prefers DeepGEMM FP4 /
    TRTLLM MXFP8; SM90 falls through to Triton_unfused or Marlin (the
    backend-level ``is_supported_config`` check filters by device capability).
    """
    if current_platform.is_rocm():
        return [Mxfp4MoeBackend.AITER_MXFP4_BF16]
    _AVAILABLE_BACKENDS = [
        Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_MXFP8,
        Mxfp4MoeBackend.DEEPGEMM_MXFP4,
        # TRITON_UNFUSED has bug with MTP support
        # TODO re-enable after kernel is fixed
        # TRITON_UNFUSED
        Mxfp4MoeBackend.MARLIN,
        Mxfp4MoeBackend.BATCHED_MARLIN,
    ]
    return _AVAILABLE_BACKENDS
```
**EN:** This function defines `_get_priority_backends`. Get available backends in priority order. Key calls include `current_platform.is_rocm`. It writes or updates `_AVAILABLE_BACKENDS`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_get_priority_backends`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `current_platform.is_rocm`。 它会写入或更新 `_AVAILABLE_BACKENDS`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 300-311 — function `_backend_activation_key`
```python
def _backend_activation_key(backend: Mxfp4MoeBackend) -> QuantKey | None:
    """Map backend to its activation key (FP8, MXFP8, or None for BF16)."""
    if backend == Mxfp4MoeBackend.DEEPGEMM_MXFP4:
        return kFp8Dynamic128Sym
    if backend in (
        Mxfp4MoeBackend.FLASHINFER_TRTLLM_MXFP4_MXFP8,
        Mxfp4MoeBackend.FLASHINFER_CUTLASS_MXFP4_MXFP8,
    ):
        return kMxfp8Dynamic
    if backend == Mxfp4MoeBackend.AITER_MXFP4_FP8:
        return kFp8StaticTensorSym
    return None  # BF16 activation
```
**EN:** This function defines `_backend_activation_key`. Map backend to its activation key (FP8, MXFP8, or None for BF16). The main inputs are `backend`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_backend_activation_key`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `backend`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 314-319 — function `_user_moe_activation_override`
```python
def _user_moe_activation_override() -> QuantKey | None:
    """User's MoE activation override from quantization_config, or None."""
    args = get_current_vllm_config().model_config.quantization_config
    if not isinstance(args, QuantizationConfigArgs) or args.moe is None:
        return None
    return args.moe.activation
```
**EN:** This function defines `_user_moe_activation_override`. User's MoE activation override from quantization_config, or None. Key calls include `get_current_vllm_config`, `isinstance`. It writes or updates `args`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_user_moe_activation_override`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `get_current_vllm_config`, `isinstance`。 它会写入或更新 `args`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 322-336 — function `_resolve_activation_key`
```python
def _resolve_activation_key(
    model_activation_key: QuantKey | None,
) -> QuantKey | None:
    """Combine the model-supplied activation key with the user override.
    Raises on conflict (both set and disagreeing)."""
    user_override = _user_moe_activation_override()
    if user_override is None:
        return model_activation_key
    if model_activation_key is None or model_activation_key == user_override:
        return user_override
    raise ValueError(
        f"checkpoint declares MoE activation={model_activation_key} but "
        f"quantization_config.moe.activation={user_override}; remove the "
        f"override or align it with the checkpoint."
    )
```
**EN:** This function defines `_resolve_activation_key`. Combine the model-supplied activation key with the user override. The main inputs are `model_activation_key`. Key calls include `_user_moe_activation_override`, `ValueError`. It writes or updates `user_override`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_resolve_activation_key`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `model_activation_key`。 关键调用包括 `_user_moe_activation_override`, `ValueError`。 它会写入或更新 `user_override`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 339-340 — function `_make_log_backend`
```python
def _make_log_backend(backend: Mxfp4MoeBackend) -> str:
    return f"Using '{backend.value}' Mxfp4 MoE backend."
```
**EN:** This function defines `_make_log_backend`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `backend`.
**CN:** 该函数定义 `_make_log_backend`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `backend`。

### Lines 343-348 — function `_make_log_unsupported`
```python
def _make_log_unsupported(backend: Mxfp4MoeBackend, reason: str | None) -> str:
    base = (
        f"Mxfp4 MoE backend '{backend.value}' does not support the "
        f"deployment configuration"
    )
    return f"{base} since {reason}." if reason else f"{base}."
```
**EN:** This function defines `_make_log_unsupported`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `backend`, `reason`. It writes or updates `base`.
**CN:** 该函数定义 `_make_log_unsupported`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `backend`, `reason`。 它会写入或更新 `base`。

## Key Concepts / 关键概念
- [EN] Reference/oracle quantization behavior for moe kernels / [CN] MoE 内核的参考量化行为
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `Mxfp4MoeBackend`, `backend_to_kernel_cls`, `map_mxfp4_backend`, `_get_priority_backends_for_gpt_oss` / [CN] 核心符号：`Mxfp4MoeBackend`, `backend_to_kernel_cls`, `map_mxfp4_backend`, `_get_priority_backends_for_gpt_oss`

## Dependencies / 依赖关系
- **External**: `enum`, `typing`, `torch` / **外部依赖**: `enum`, `typing`, `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.config`, `vllm.config.kernel`, `vllm.config.quantization`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.config`, `vllm.config.kernel`, `vllm.config.quantization`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
