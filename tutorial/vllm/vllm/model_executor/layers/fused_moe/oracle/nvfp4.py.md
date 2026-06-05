# nvfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/oracle/nvfp4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: reference/oracle quantization behavior for MoE kernels; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 内核的参考量化行为；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-38 — imports and setup
```python
from enum import Enum

import torch

import vllm.envs as envs
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.config.kernel import MoEBackend
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
    nvfp4_moe_quant_config,
    nvfp4_w4a16_moe_quant_config,
)
from vllm.model_executor.layers.quantization.utils.flashinfer_fp4_moe import (
    prepare_nvfp4_moe_layer_for_fi_or_cutlass,
    prepare_nvfp4_moe_layer_for_flashinfer_cutedsl,
)
from vllm.model_executor.layers.quantization.utils.flashinfer_utils import (
    FlashinferMoeBackend,
    get_flashinfer_moe_backend,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_fp4 import (
    prepare_nvfp4_moe_layer_for_marlin,
)
from vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils import (
    kE2M1ToFloat_handle,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 41-48 — class `NvFp4MoeBackend`
```python
class NvFp4MoeBackend(Enum):
    FLASHINFER_TRTLLM = "FLASHINFER_TRTLLM"
    FLASHINFER_CUTLASS = "FLASHINFER_CUTLASS"
    FLASHINFER_CUTEDSL = "FLASHINFER_CUTEDSL"
    FLASHINFER_CUTEDSL_BATCHED = "FLASHINFER_CUTEDSL_BATCHED"
    VLLM_CUTLASS = "VLLM_CUTLASS"
    MARLIN = "MARLIN"
    EMULATION = "EMULATION"
```
**EN:** This class defines `NvFp4MoeBackend`. It inherits from `Enum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `FLASHINFER_TRTLLM`, `FLASHINFER_CUTLASS`, `FLASHINFER_CUTEDSL`, `FLASHINFER_CUTEDSL_BATCHED`, `VLLM_CUTLASS`, `MARLIN`.
**CN:** 该类定义了 `NvFp4MoeBackend`。 它继承自 `Enum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `FLASHINFER_TRTLLM`, `FLASHINFER_CUTLASS`, `FLASHINFER_CUTEDSL`, `FLASHINFER_CUTEDSL_BATCHED`, `VLLM_CUTLASS`, `MARLIN`。

### Lines 51-56 — constant `FLASHINFER_NVFP4_MOE_BACKENDS`
```python
FLASHINFER_NVFP4_MOE_BACKENDS = [
    NvFp4MoeBackend.FLASHINFER_TRTLLM,
    NvFp4MoeBackend.FLASHINFER_CUTLASS,
    NvFp4MoeBackend.FLASHINFER_CUTEDSL,
    NvFp4MoeBackend.FLASHINFER_CUTEDSL_BATCHED,
]
```
**EN:** This constant defines `FLASHINFER_NVFP4_MOE_BACKENDS`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `FLASHINFER_NVFP4_MOE_BACKENDS`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 65-70 — function `is_global_sf_supported_for_nvfp4_backend`
```python
def is_global_sf_supported_for_nvfp4_backend(backend: NvFp4MoeBackend) -> bool:
    # Checks whether `backend` supports quantizing with scaling factors
    # of all experts in Expert Parallel Mode when all experts are not
    # on the same rank.

    return backend in FLASHINFER_NVFP4_MOE_BACKENDS
```
**EN:** This function defines `is_global_sf_supported_for_nvfp4_backend`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `backend`.
**CN:** 该函数定义 `is_global_sf_supported_for_nvfp4_backend`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `backend`。

### Lines 73-129 — function `backend_to_kernel_cls`
```python
def backend_to_kernel_cls(
    backend: NvFp4MoeBackend,
) -> list[type[mk.FusedMoEExperts]]:
    if backend == NvFp4MoeBackend.FLASHINFER_TRTLLM:
        from vllm.model_executor.layers.fused_moe.experts.trtllm_nvfp4_moe import (
            TrtLlmNvFp4ExpertsModular,
            TrtLlmNvFp4ExpertsMonolithic,
        )

        # NOTE: prefer Monolthic > Modular, so return Monolithic first.
        return [
            TrtLlmNvFp4ExpertsMonolithic,
            TrtLlmNvFp4ExpertsModular,
        ]

    elif backend == NvFp4MoeBackend.FLASHINFER_CUTLASS:
        from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutlass_moe import (  # noqa: E501
            FlashInferExperts,
        )

        return [FlashInferExperts]

    elif backend == NvFp4MoeBackend.FLASHINFER_CUTEDSL:
        from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutedsl_moe import (  # noqa: E501
            FlashInferCuteDSLExperts,
        )

        return [FlashInferCuteDSLExperts]

    elif backend == NvFp4MoeBackend.FLASHINFER_CUTEDSL_BATCHED:
        from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutedsl_batched_moe import (  # noqa: E501
            FlashInferCuteDSLBatchedExperts,
        )

        return [FlashInferCuteDSLBatchedExperts]

    elif backend == NvFp4MoeBackend.VLLM_CUTLASS:
        from vllm.model_executor.layers.fused_moe.experts.cutlass_moe import (
            CutlassExpertsFp4,
        )

        return [CutlassExpertsFp4]

    elif backend == NvFp4MoeBackend.MARLIN:
# ... omitted for brevity ...
    else:
        raise ValueError(f"Unknown NvFP4 MoE backend: {backend.value}")
```
**EN:** This function defines `backend_to_kernel_cls`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `backend`. Key calls include `ValueError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `backend_to_kernel_cls`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `backend`。 关键调用包括 `ValueError`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 132-147 — function `map_nvfp4_backend`
```python
def map_nvfp4_backend(runner_backend: MoEBackend) -> NvFp4MoeBackend:
    """Map user's MoEBackend to NvFp4MoeBackend."""
    mapping = {
        "cutlass": NvFp4MoeBackend.VLLM_CUTLASS,
        "flashinfer_trtllm": NvFp4MoeBackend.FLASHINFER_TRTLLM,
        "flashinfer_cutlass": NvFp4MoeBackend.FLASHINFER_CUTLASS,
        "flashinfer_cutedsl": NvFp4MoeBackend.FLASHINFER_CUTEDSL,
        "marlin": NvFp4MoeBackend.MARLIN,
        "emulation": NvFp4MoeBackend.EMULATION,
    }
    if backend := mapping.get(runner_backend):
        return backend
    raise ValueError(
        f"moe_backend='{runner_backend}' is not supported for NvFP4 MoE. "
        f"Expected one of {list(mapping.keys())}."
    )
```
**EN:** This function defines `map_nvfp4_backend`. Map user's MoEBackend to NvFp4MoeBackend. The main inputs are `runner_backend`. Key calls include `ValueError`, `mapping.get`, `list`, `mapping.keys`. It writes or updates `mapping`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `map_nvfp4_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `runner_backend`。 关键调用包括 `ValueError`, `mapping.get`, `list`, `mapping.keys`。 它会写入或更新 `mapping`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 150-286 — function `select_nvfp4_moe_backend`
```python
def select_nvfp4_moe_backend(
    config: FusedMoEConfig,
    weight_key: QuantKey | None,
    activation_key: QuantKey | None,
) -> tuple[NvFp4MoeBackend, type[mk.FusedMoEExperts]]:
    """
    Select the primary NvFP4 MoE backend
    Note: Shape-specific fallbacks may still occur at runtime.
    """

    # NOTE: the kernels are selected in the following order.
    AVAILABLE_BACKENDS = [
        NvFp4MoeBackend.FLASHINFER_TRTLLM,
        NvFp4MoeBackend.FLASHINFER_CUTEDSL,
        NvFp4MoeBackend.FLASHINFER_CUTEDSL_BATCHED,
        NvFp4MoeBackend.FLASHINFER_CUTLASS,
        NvFp4MoeBackend.VLLM_CUTLASS,
        NvFp4MoeBackend.MARLIN,
        NvFp4MoeBackend.EMULATION,
    ]

    use_batched = config.moe_parallel_config.use_batched_activation_format
    activation_format = (
        mk.FusedMoEActivationFormat.BatchedExperts
        if use_batched
        else mk.FusedMoEActivationFormat.Standard
    )

    def _make_log_backend(backend: NvFp4MoeBackend):
        available_backend_strs = [b.value for b in AVAILABLE_BACKENDS]
        return (
            f"Using '{backend.value}' NvFp4 MoE backend out "
            f"of potential backends: {available_backend_strs}."
        )

    def _make_log_unsupported(backend: NvFp4MoeBackend, reason: str | None) -> str:
        if reason:
            return (
                f"NvFp4 MoE backend '{backend.value}' does not support the "
                f"deployment configuration since {reason}."
            )
        else:
            return (
                f"NvFp4 MoE backend '{backend.value}' does not support the "
# ... omitted for brevity ...
        "No NvFp4 MoE backend supports the deployment configuration."
    )
```
**EN:** This function defines `select_nvfp4_moe_backend`. Select the primary NvFP4 MoE backend Note: Shape-specific fallbacks may still occur at runtime. The main inputs are `config`, `weight_key`, `activation_key`. Key calls include `envs.is_set`, `NotImplementedError`, `backend_to_kernel_cls`, `ValueError`, `map_nvfp4_backend`, `_return_or_raise`. It writes or updates `AVAILABLE_BACKENDS`, `use_batched`, `activation_format`, `runner_backend`, `available_backend_strs`, `requested_backend`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `select_nvfp4_moe_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `config`, `weight_key`, `activation_key`。 关键调用包括 `envs.is_set`, `NotImplementedError`, `backend_to_kernel_cls`, `ValueError`, `map_nvfp4_backend`, `_return_or_raise`。 它会写入或更新 `AVAILABLE_BACKENDS`, `use_batched`, `activation_format`, `runner_backend`, `available_backend_strs`, `requested_backend`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 289-418 — function `convert_to_nvfp4_moe_kernel_format`
```python
def convert_to_nvfp4_moe_kernel_format(
    nvfp4_backend: NvFp4MoeBackend,
    layer: torch.nn.Module,
    w13: torch.Tensor,
    w13_scale: torch.Tensor,
    w13_scale_2: torch.Tensor,
    a13_scale: torch.Tensor | None,
    w2: torch.Tensor,
    w2_scale: torch.Tensor,
    w2_scale_2: torch.Tensor,
    a2_scale: torch.Tensor | None,
    is_act_and_mul: bool,
) -> tuple[
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
]:
    if nvfp4_backend == NvFp4MoeBackend.FLASHINFER_CUTEDSL:
        (
            w13,
            w13_scale,
            w13_scale_2,
            a13_scale,
            w2,
            w2_scale,
            w2_scale_2,
            a2_scale,
        ) = prepare_nvfp4_moe_layer_for_flashinfer_cutedsl(
            layer=layer,
            w13=w13,
            w13_scale=w13_scale,
            w13_scale_2=w13_scale_2,
            a13_scale=a13_scale,
            w2=w2,
            w2_scale=w2_scale,
            w2_scale_2=w2_scale_2,
            a2_scale=a2_scale,
        )
    elif (
# ... omitted for brevity ...
        a2_scale,
    )
```
**EN:** This function defines `convert_to_nvfp4_moe_kernel_format`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `nvfp4_backend`, `layer`, `w13`, `w13_scale`, `w13_scale_2`, `a13_scale`. Key calls include `prepare_nvfp4_moe_layer_for_flashinfer_cutedsl`, `prepare_nvfp4_moe_layer_for_fi_or_cutlass`, `prepare_nvfp4_moe_layer_for_marlin`, `kE2M1ToFloat_handle.val.to`, `ValueError`, `logger.warning_once`. It writes or updates `w13`, `w13_scale`, `w13_scale_2`, `a13_scale`, `w2`, `w2_scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `convert_to_nvfp4_moe_kernel_format`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `nvfp4_backend`, `layer`, `w13`, `w13_scale`, `w13_scale_2`, `a13_scale`。 关键调用包括 `prepare_nvfp4_moe_layer_for_flashinfer_cutedsl`, `prepare_nvfp4_moe_layer_for_fi_or_cutlass`, `prepare_nvfp4_moe_layer_for_marlin`, `kE2M1ToFloat_handle.val.to`, `ValueError`, `logger.warning_once`。 它会写入或更新 `w13`, `w13_scale`, `w13_scale_2`, `a13_scale`, `w2`, `w2_scale`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 421-468 — function `make_nvfp4_moe_quant_config`
```python
def make_nvfp4_moe_quant_config(
    backend: NvFp4MoeBackend,
    w13_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    w13_scale_2: torch.Tensor,
    w2_scale_2: torch.Tensor,
    a13_scale: torch.Tensor,
    a2_scale: torch.Tensor,
) -> FusedMoEQuantConfig:
    if backend == NvFp4MoeBackend.MARLIN:
        return nvfp4_w4a16_moe_quant_config(
            g1_alphas=w13_scale_2,
            g2_alphas=w2_scale_2,
            w1_scale=w13_scale,
            w2_scale=w2_scale,
        )
    elif backend == NvFp4MoeBackend.EMULATION:
        return nvfp4_moe_quant_config(
            g1_alphas=w13_scale_2,
            g2_alphas=w2_scale_2,
            a1_gscale=a13_scale,
            a2_gscale=a2_scale,
            w1_scale=w13_scale,
            w2_scale=w2_scale,
        )

    # Pass w13_scale_2 / w2_scale_2 directly as g1/g2_alphas.
    # The expert's process_weights_after_loading will fuse activation
    # scales in-place. Since the quant config references the same tensor
    # as the registered parameter, EPLB rearrangement stays in sync.
    return nvfp4_moe_quant_config(
        g1_alphas=w13_scale_2,
        g2_alphas=w2_scale_2,
        a1_gscale=(1.0 / a13_scale),
        a2_gscale=(1.0 / a2_scale),
        w1_scale=w13_scale,
        w2_scale=w2_scale,
        # NOTE(rob): this is a hack until the MoE kernels
        # create their own quant configs. TRTLLM kernel
        # does not accept swizzled input quant scales.
        is_scale_swizzled=(
            backend
            not in (
                NvFp4MoeBackend.FLASHINFER_TRTLLM,
                NvFp4MoeBackend.FLASHINFER_CUTEDSL,
            )
        ),
    )
```
**EN:** This function defines `make_nvfp4_moe_quant_config`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `backend`, `w13_scale`, `w2_scale`, `w13_scale_2`, `w2_scale_2`, `a13_scale`. Key calls include `nvfp4_moe_quant_config`, `nvfp4_w4a16_moe_quant_config`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_nvfp4_moe_quant_config`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `backend`, `w13_scale`, `w2_scale`, `w13_scale_2`, `w2_scale_2`, `a13_scale`。 关键调用包括 `nvfp4_moe_quant_config`, `nvfp4_w4a16_moe_quant_config`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 471-512 — function `make_nvfp4_moe_kernel`
```python
def make_nvfp4_moe_kernel(
    moe_quant_config: FusedMoEQuantConfig,
    moe_config: FusedMoEConfig,
    experts_cls: type[mk.FusedMoEExperts],
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
        inplace=False,
    )

    # TODO(rob): update inplace logic to be part of the kernel.
    return kernel
```
**EN:** This function defines `make_nvfp4_moe_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `moe_quant_config`, `moe_config`, `experts_cls`, `routing_tables`. Key calls include `maybe_make_prepare_finalize`, `logger.info_once`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `experts_cls`, `issubclass`. It writes or updates `prepare_finalize`, `kernel`, `max_num_tokens`, `experts`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_nvfp4_moe_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `moe_quant_config`, `moe_config`, `experts_cls`, `routing_tables`。 关键调用包括 `maybe_make_prepare_finalize`, `logger.info_once`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `experts_cls`, `issubclass`。 它会写入或更新 `prepare_finalize`, `kernel`, `max_num_tokens`, `experts`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Reference/oracle quantization behavior for moe kernels / [CN] MoE 内核的参考量化行为
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `NvFp4MoeBackend`, `is_global_sf_supported_for_nvfp4_backend`, `backend_to_kernel_cls`, `map_nvfp4_backend` / [CN] 核心符号：`NvFp4MoeBackend`, `is_global_sf_supported_for_nvfp4_backend`, `backend_to_kernel_cls`, `map_nvfp4_backend`

## Dependencies / 依赖关系
- **External**: `enum`, `torch` / **外部依赖**: `enum`, `torch`
- **Internal**: `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.config.kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.flashinfer_fp4_moe`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp4`, `vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils` / **内部依赖**: `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.config.kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.flashinfer_fp4_moe`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp4`, `vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
