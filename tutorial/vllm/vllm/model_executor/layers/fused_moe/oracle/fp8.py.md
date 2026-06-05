# fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/oracle/fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: reference/oracle quantization behavior for MoE kernels; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 内核的参考量化行为；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-39 — imports and setup
```python
from enum import Enum

import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import envs
from vllm._aiter_ops import rocm_aiter_ops
from vllm.config.kernel import MoEBackend
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
    fp8_w8a8_moe_quant_config,
    fp8_w8a16_moe_quant_config,
)
from vllm.model_executor.layers.quantization.utils.flashinfer_utils import (
    FlashinferMoeBackend,
    get_flashinfer_moe_backend,
    prepare_fp8_moe_layer_for_fi,
)
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    prepare_fp8_moe_layer_for_deepgemm,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_fp8 import (
    prepare_fp8_moe_layer_for_marlin,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8Dynamic128Sym,
    kFp8Static128BlockSym,
)
from vllm.platforms import current_platform

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 42-55 — class `Fp8MoeBackend`
```python
class Fp8MoeBackend(Enum):
    NONE = "NONE"
    FLASHINFER_TRTLLM = "FLASHINFER_TRTLLM"
    FLASHINFER_CUTLASS = "FLASHINFER_CUTLASS"
    DEEPGEMM = "DEEPGEMM"
    BATCHED_DEEPGEMM = "BATCHED_DEEPGEMM"
    MARLIN = "MARLIN"
    TRITON = "TRITON"
    BATCHED_TRITON = "BATCHED_TRITON"
    AITER = "AITER"
    VLLM_CUTLASS = "VLLM_CUTLASS"
    BATCHED_VLLM_CUTLASS = "BATCHED_VLLM_CUTLASS"
    XPU = "XPU"
    CPU = "CPU"
```
**EN:** This class defines `Fp8MoeBackend`. It inherits from `Enum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `NONE`, `FLASHINFER_TRTLLM`, `FLASHINFER_CUTLASS`, `DEEPGEMM`, `BATCHED_DEEPGEMM`, `MARLIN`.
**CN:** 该类定义了 `Fp8MoeBackend`。 它继承自 `Enum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `NONE`, `FLASHINFER_TRTLLM`, `FLASHINFER_CUTLASS`, `DEEPGEMM`, `BATCHED_DEEPGEMM`, `MARLIN`。

### Lines 58-108 — function `_get_priority_backends`
```python
def _get_priority_backends(
    moe_config: FusedMoEConfig,
    weight_key: QuantKey | None,
    activation_key: QuantKey | None,
) -> list[Fp8MoeBackend]:
    """
    Get available backends in priority order based on platform and config.

    This function can be extended to become more complex as needed.
    """

    _AVAILABLE_BACKENDS = [
        Fp8MoeBackend.AITER,
        Fp8MoeBackend.FLASHINFER_TRTLLM,
        Fp8MoeBackend.FLASHINFER_CUTLASS,
        Fp8MoeBackend.DEEPGEMM,
        Fp8MoeBackend.VLLM_CUTLASS,
        Fp8MoeBackend.TRITON,
        Fp8MoeBackend.MARLIN,
        Fp8MoeBackend.BATCHED_DEEPGEMM,
        Fp8MoeBackend.BATCHED_VLLM_CUTLASS,
        Fp8MoeBackend.BATCHED_TRITON,
        Fp8MoeBackend.XPU,
        Fp8MoeBackend.CPU,
    ]

    def _move_to_front(backends: list[Fp8MoeBackend], backend: Fp8MoeBackend) -> None:
        backends.insert(0, backends.pop(backends.index(backend)))

    # On Hopper for Block Fp8, prefer Triton for TP and FI CUTLASS for EP.
    if (
        current_platform.is_cuda()
        and current_platform.is_device_capability(90)
        and activation_key == kFp8Dynamic128Sym
        and weight_key == kFp8Static128BlockSym
    ):
        if moe_config.moe_parallel_config.ep_size > 1:
            _move_to_front(_AVAILABLE_BACKENDS, Fp8MoeBackend.FLASHINFER_CUTLASS)
        else:
            _move_to_front(_AVAILABLE_BACKENDS, Fp8MoeBackend.TRITON)

    if current_platform.is_xpu():
        # XPU platform supports TritonExperts and XPUExpertsFp8,
        # move XPU backend to the front.
# ... omitted for brevity ...

    return _AVAILABLE_BACKENDS
```
**EN:** This function defines `_get_priority_backends`. Get available backends in priority order based on platform and config. The main inputs are `moe_config`, `weight_key`, `activation_key`. Key calls include `current_platform.is_xpu`, `current_platform.is_cpu`, `backends.insert`, `current_platform.is_cuda`, `current_platform.is_device_capability`, `_move_to_front`. It writes or updates `_AVAILABLE_BACKENDS`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_get_priority_backends`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `moe_config`, `weight_key`, `activation_key`。 关键调用包括 `current_platform.is_xpu`, `current_platform.is_cpu`, `backends.insert`, `current_platform.is_cuda`, `current_platform.is_device_capability`, `_move_to_front`。 它会写入或更新 `_AVAILABLE_BACKENDS`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 111-201 — function `backend_to_kernel_cls`
```python
def backend_to_kernel_cls(
    backend: Fp8MoeBackend,
) -> list[type[mk.FusedMoEExperts]]:
    if backend == Fp8MoeBackend.FLASHINFER_TRTLLM:
        from vllm.model_executor.layers.fused_moe.experts.trtllm_fp8_moe import (  # noqa: E501
            TrtLlmFp8ExpertsModular,
            TrtLlmFp8ExpertsMonolithic,
        )

        return [TrtLlmFp8ExpertsMonolithic, TrtLlmFp8ExpertsModular]

    elif backend == Fp8MoeBackend.FLASHINFER_CUTLASS:
        from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutlass_moe import (  # noqa: E501
            FlashInferExperts,
        )

        return [FlashInferExperts]

    elif backend == Fp8MoeBackend.DEEPGEMM:
        from vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe import (
            TritonOrDeepGemmExperts,
        )

        return [TritonOrDeepGemmExperts]

    elif backend == Fp8MoeBackend.BATCHED_DEEPGEMM:
        from vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe import (
            BatchedDeepGemmExperts,
        )

        return [BatchedDeepGemmExperts]

    elif backend == Fp8MoeBackend.MARLIN:
        from vllm.model_executor.layers.fused_moe.experts.marlin_moe import (
            MarlinExperts,
        )

        return [MarlinExperts]

    elif backend == Fp8MoeBackend.TRITON:
        from vllm.model_executor.layers.fused_moe.experts.triton_moe import (
            TritonExperts,
        )

# ... omitted for brevity ...
    else:
        raise ValueError(f"Unknown FP8 MoE backend: {backend.value}")
```
**EN:** This function defines `backend_to_kernel_cls`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `backend`. Key calls include `ValueError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `backend_to_kernel_cls`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `backend`。 关键调用包括 `ValueError`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 204-220 — function `map_fp8_backend`
```python
def map_fp8_backend(runner_backend: MoEBackend) -> Fp8MoeBackend:
    """Map user's MoEBackend to Fp8MoeBackend."""
    mapping = {
        "triton": Fp8MoeBackend.TRITON,
        "deep_gemm": Fp8MoeBackend.DEEPGEMM,
        "cutlass": Fp8MoeBackend.VLLM_CUTLASS,
        "flashinfer_trtllm": Fp8MoeBackend.FLASHINFER_TRTLLM,
        "flashinfer_cutlass": Fp8MoeBackend.FLASHINFER_CUTLASS,
        "marlin": Fp8MoeBackend.MARLIN,
        "aiter": Fp8MoeBackend.AITER,
    }
    if backend := mapping.get(runner_backend):
        return backend
    raise ValueError(
        f"moe_backend='{runner_backend}' is not supported for FP8 MoE. "
        f"Expected one of {list(mapping.keys())}."
    )
```
**EN:** This function defines `map_fp8_backend`. Map user's MoEBackend to Fp8MoeBackend. The main inputs are `runner_backend`. Key calls include `ValueError`, `mapping.get`, `list`, `mapping.keys`. It writes or updates `mapping`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `map_fp8_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `runner_backend`。 关键调用包括 `ValueError`, `mapping.get`, `list`, `mapping.keys`。 它会写入或更新 `mapping`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 223-419 — function `select_fp8_moe_backend`
```python
def select_fp8_moe_backend(
    config: FusedMoEConfig,
    weight_key: QuantKey | None,
    activation_key: QuantKey | None,
    allow_vllm_cutlass: bool = False,
) -> tuple[Fp8MoeBackend, type[mk.FusedMoEExperts] | None]:
    """
    Select the primary FP8 MoE backend
    Note: Shape-specific fallbacks may still occur at runtime.
    """

    # NOTE: the kernels are selected in the following order.
    AVAILABLE_BACKENDS = _get_priority_backends(config, weight_key, activation_key)

    # NOTE(rob): We need to peak into the P/F selection to determine
    # if we are using the batched or standard expert format, which
    # if not ideal. Once we unify TP + DP/EP, we can select P/F first.
    activation_format = (
        mk.FusedMoEActivationFormat.BatchedExperts
        if config.moe_parallel_config.use_batched_activation_format
        else mk.FusedMoEActivationFormat.Standard
    )

    def _make_log_backend(backend: Fp8MoeBackend):
        available_backend_strs = [b.value for b in AVAILABLE_BACKENDS]
        return (
            f"Using {backend.value} Fp8 MoE backend out "
            f"of potential backends: {available_backend_strs}."
        )

    def _make_log_unsupported(backend: Fp8MoeBackend, reason: str | None) -> str:
        if reason:
            return (
                f"FP8 MoE backend {backend.value} does not support the "
                f"deployment configuration since {reason}."
            )
        else:
            return (
                f"FP8 MoE backend '{backend.value}' does not support the "
                "deployment configuration."
            )

    def _return_or_raise(
        backend: Fp8MoeBackend,
# ... omitted for brevity ...

    return Fp8MoeBackend.NONE, None
```
**EN:** This function defines `select_fp8_moe_backend`. Select the primary FP8 MoE backend Note: Shape-specific fallbacks may still occur at runtime. The main inputs are `config`, `weight_key`, `activation_key`, `allow_vllm_cutlass`. Key calls include `_get_priority_backends`, `envs.is_set`, `backend_to_kernel_cls`, `ValueError`, `map_fp8_backend`, `_return_or_raise`. It writes or updates `AVAILABLE_BACKENDS`, `activation_format`, `runner_backend`, `available_backend_strs`, `requested_backend`, `backend`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `select_fp8_moe_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `config`, `weight_key`, `activation_key`, `allow_vllm_cutlass`。 关键调用包括 `_get_priority_backends`, `envs.is_set`, `backend_to_kernel_cls`, `ValueError`, `map_fp8_backend`, `_return_or_raise`。 它会写入或更新 `AVAILABLE_BACKENDS`, `activation_format`, `runner_backend`, `available_backend_strs`, `requested_backend`, `backend`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 422-502 — function `convert_to_fp8_moe_kernel_format`
```python
def convert_to_fp8_moe_kernel_format(
    fp8_backend: Fp8MoeBackend,
    layer: torch.nn.Module,
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    w13_input_scale: torch.Tensor | None,
    w2_input_scale: torch.Tensor | None,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    block_quant = hasattr(layer, "weight_block_size")
    if fp8_backend in [Fp8MoeBackend.DEEPGEMM, Fp8MoeBackend.BATCHED_DEEPGEMM]:
        assert block_quant
        w13, w2, w13_scale, w2_scale = prepare_fp8_moe_layer_for_deepgemm(
            w13,
            w2,
            w13_scale,
            w2_scale,
            tuple(layer.weight_block_size),
        )
    elif fp8_backend == Fp8MoeBackend.AITER:
        w13, w2 = rocm_aiter_ops.shuffle_weights(w13, w2)
    elif fp8_backend == Fp8MoeBackend.MARLIN:
        weight_block_size = getattr(layer, "weight_block_size", None)
        if weight_block_size == [1, 32]:
            from vllm.model_executor.layers.quantization.utils.marlin_utils_fp8 import (
                prepare_mxfp8_moe_layer_for_marlin,
            )

            w13, w2, w13_scale, w2_scale = prepare_mxfp8_moe_layer_for_marlin(
                layer,
                w13,
                w2,
                w13_scale,
                w2_scale,
            )
        else:
            w13, w2, w13_scale, w2_scale = prepare_fp8_moe_layer_for_marlin(
                layer,
                w13,
                w2,
                w13_scale,
                w2_scale,
            )
# ... omitted for brevity ...

    return w13, w2, w13_scale, w2_scale
```
**EN:** This function defines `convert_to_fp8_moe_kernel_format`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `fp8_backend`, `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`. Key calls include `hasattr`, `prepare_fp8_moe_layer_for_deepgemm`, `tuple`, `rocm_aiter_ops.shuffle_weights`, `getattr`, `prepare_mxfp8_moe_layer_for_marlin`. It writes or updates `block_quant`, `w13`, `w2`, `w13_scale`, `w2_scale`, `weight_block_size`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `convert_to_fp8_moe_kernel_format`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `fp8_backend`, `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`。 关键调用包括 `hasattr`, `prepare_fp8_moe_layer_for_deepgemm`, `tuple`, `rocm_aiter_ops.shuffle_weights`, `getattr`, `prepare_mxfp8_moe_layer_for_marlin`。 它会写入或更新 `block_quant`, `w13`, `w2`, `w13_scale`, `w2_scale`, `weight_block_size`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 505-586 — function `make_fp8_moe_quant_config`
```python
def make_fp8_moe_quant_config(
    fp8_backend: Fp8MoeBackend,
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    a1_scale: torch.Tensor | None,
    a2_scale: torch.Tensor | None,
    block_shape: list[int] | None = None,
    per_act_token_quant: bool = False,
    per_out_ch_quant: bool = False,
    swiglu_limit: float | None = None,
) -> FusedMoEQuantConfig:
    """
    Create FusedMoEQuantConfig for the specified FP8 Backend.
    The FusedMoEQuantConfig holds the scales that are used
    at runtime by the Modular Kernel abstraction.

    Note that certain kernels (e.g. Flashinfer CUTLASS) need
    special Quant configs to handle non-standard inputs to
    their kernel interfaces.

    In a future PR, we will have this function should be
    a method of the modular kernel itself.
    """

    # MARLIN is mixed precision W8A16 config.
    if fp8_backend == Fp8MoeBackend.MARLIN:
        return fp8_w8a16_moe_quant_config(
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            block_shape=block_shape,
        )

    # CPU is mixed precision W8A16 config.
    if fp8_backend == Fp8MoeBackend.CPU:
        return fp8_w8a16_moe_quant_config(
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            block_shape=block_shape,
        )

    # Flashinfer CUTLASS per-tensor uses single dq scale
    # (alpha = w_scale * a_scale) and inverse a2 scale.
    if fp8_backend == Fp8MoeBackend.FLASHINFER_CUTLASS and block_shape is None:
        assert a1_scale is not None and a2_scale is not None
# ... omitted for brevity ...
        gemm1_clamp_limit=swiglu_limit,
    )
```
**EN:** This function defines `make_fp8_moe_quant_config`. Create FusedMoEQuantConfig for the specified FP8 Backend. The main inputs are `fp8_backend`, `w1_scale`, `w2_scale`, `a1_scale`, `a2_scale`, `block_shape`. Key calls include `fp8_w8a8_moe_quant_config`, `fp8_w8a16_moe_quant_config`, `FusedMoEQuantConfig.make`, `squeeze`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_fp8_moe_quant_config`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `fp8_backend`, `w1_scale`, `w2_scale`, `a1_scale`, `a2_scale`, `block_shape`。 关键调用包括 `fp8_w8a8_moe_quant_config`, `fp8_w8a16_moe_quant_config`, `FusedMoEQuantConfig.make`, `squeeze`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 589-633 — function `make_fp8_moe_kernel`
```python
def make_fp8_moe_kernel(
    moe_quant_config: FusedMoEQuantConfig,
    moe_config: FusedMoEConfig,
    experts_cls: type[mk.FusedMoEExperts],
    fp8_backend: Fp8MoeBackend,
    routing_tables: tuple[torch.Tensor, torch.Tensor, torch.Tensor] | None = None,
) -> mk.FusedMoEKernel:
    # Create Prepare/Finalize.
    prepare_finalize = maybe_make_prepare_finalize(
        moe=moe_config,
        quant_config=moe_quant_config,
        routing_tables=routing_tables,
        allow_new_interface=True,
        use_monolithic=issubclass(experts_cls, mk.FusedMoEExpertsMonolithic),
    )
    assert prepare_finalize is not None

    logger.info_once("Using %s", prepare_finalize.__class__.__name__)

    # Create Experts.
    if prepare_finalize.activation_format == mk.FusedMoEActivationFormat.BatchedExperts:
        max_num_tokens = prepare_finalize.max_num_tokens_per_rank()
        assert max_num_tokens is not None
        experts = experts_cls(
            moe_config=moe_config,
            quant_config=moe_quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=prepare_finalize.num_dispatchers(),
        )
    else:
        experts = experts_cls(
            moe_config=moe_config,
            quant_config=moe_quant_config,
        )

    kernel = mk.FusedMoEKernel(
        prepare_finalize,
        experts,
        inplace=(
            not moe_config.disable_inplace
            and fp8_backend != Fp8MoeBackend.FLASHINFER_CUTLASS
        ),
    )

    return kernel
```
**EN:** This function defines `make_fp8_moe_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `moe_quant_config`, `moe_config`, `experts_cls`, `fp8_backend`, `routing_tables`. Key calls include `maybe_make_prepare_finalize`, `logger.info_once`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `experts_cls`, `issubclass`. It writes or updates `prepare_finalize`, `kernel`, `max_num_tokens`, `experts`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_fp8_moe_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `moe_quant_config`, `moe_config`, `experts_cls`, `fp8_backend`, `routing_tables`。 关键调用包括 `maybe_make_prepare_finalize`, `logger.info_once`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `experts_cls`, `issubclass`。 它会写入或更新 `prepare_finalize`, `kernel`, `max_num_tokens`, `experts`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Reference/oracle quantization behavior for moe kernels / [CN] MoE 内核的参考量化行为
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `Fp8MoeBackend`, `_get_priority_backends`, `backend_to_kernel_cls`, `map_fp8_backend` / [CN] 核心符号：`Fp8MoeBackend`, `_get_priority_backends`, `backend_to_kernel_cls`, `map_fp8_backend`

## Dependencies / 依赖关系
- **External**: `enum`, `torch` / **外部依赖**: `enum`, `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm._aiter_ops`, `vllm.config.kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp8` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm._aiter_ops`, `vllm.config.kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp8`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
