# mk_objects.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/modular_kernel_tools/mk_objects.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / moe / modular_kernel_tools / mk_objects; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / moe / modular_kernel_tools / mk_objects 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-45)
```python
from dataclasses import dataclass

import torch

# Fused experts and PrepareFinalize imports
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.model_executor.layers.fused_moe import TritonExperts
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe import (
    BatchedDeepGemmExperts,
)
from vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe import DeepGemmExperts
from vllm.model_executor.layers.fused_moe.experts.fused_batched_moe import (
    BatchedTritonExperts,
    NaiveBatchedExperts,
)
from vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe import (
    TritonOrDeepGemmExperts,
)
from vllm.model_executor.layers.fused_moe.prepare_finalize import (
    MoEPrepareAndFinalizeNoDPEPModular,
)
from vllm.model_executor.layers.quantization.utils.nvfp4_utils import (
    cutlass_fp4_supported,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    cutlass_fp8_supported,
)
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import is_deep_gemm_supported
from vllm.utils.flashinfer import (
    has_flashinfer_cutlass_fused_moe,
    has_flashinfer_nvlink_one_sided,
)
from vllm.utils.import_utils import (
    has_aiter,
    has_deep_ep,
    has_deep_gemm,
    has_mori,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, torch; and vLLM components like vllm.model_executor.layers.fused_moe.modular_kernel, vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.fused_moe.config, vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、torch；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.modular_kernel、vllm.model_executor.layers.fused_moe、vllm.model_executor.layers.fused_moe.config、vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe。

### Class `TestMoEQuantConfig` (lines 48-53)
```python
@dataclass
class TestMoEQuantConfig:
    quant_dtype: torch.dtype | str | None
    per_out_ch_quant: bool
    per_act_token_quant: bool
    block_shape: list[int] | None
```
**EN:** This dataclass packages the fields needed to describe TestMoEQuantConfig. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 TestMoEQuantConfig 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Class `PrepareFinalizeInfo` (lines 56-62)
```python
@dataclass
class PrepareFinalizeInfo:
    activation_format: mk.FusedMoEActivationFormat
    supported_dtypes: list[torch.dtype | str]
    blocked_quantization_support: bool
    backend: str | None
    supports_apply_weight_on_input: bool = True
```
**EN:** This dataclass packages the fields needed to describe PrepareFinalizeInfo. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 PrepareFinalizeInfo 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Class `ExpertInfo` (lines 65-73)
```python
@dataclass
class ExpertInfo:
    activation_format: mk.FusedMoEActivationFormat
    supported_dtypes: list[torch.dtype | str]
    blocked_quantization_support: bool
    supports_expert_map: bool
    needs_matching_quant: bool = False
    needs_deep_gemm: bool = False
    needs_aiter: bool = False
```
**EN:** This dataclass packages the fields needed to describe ExpertInfo. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 ExpertInfo 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Constants and module state (lines 76-95)
```python
PREPARE_FINALIZE_INFO: dict[
    mk.FusedMoEPrepareAndFinalizeModular, PrepareFinalizeInfo
] = {}
EXPERT_INFO: dict[mk.FusedMoEExpertsModular, ExpertInfo] = {}
MK_ALL_PREPARE_FINALIZE_TYPES: list[mk.FusedMoEPrepareAndFinalizeModular] = []
MK_MULTI_GPU_PREPARE_FINALIZE_TYPES: list[mk.FusedMoEPrepareAndFinalizeModular] = []
MK_SINGLE_GPU_PREPARE_FINALIZE_TYPES: list[mk.FusedMoEPrepareAndFinalizeModular] = []
MK_FUSED_EXPERT_TYPES: list[mk.FusedMoEExpertsModular] = []

standard_format = mk.FusedMoEActivationFormat.Standard
batched_format = mk.FusedMoEActivationFormat.BatchedExperts
common_float_types: list[torch.dtype | str] = [
    torch.float8_e4m3fn,
    torch.bfloat16,
    torch.float16,
    torch.float32,
]
common_float_and_int_types = common_float_types + [torch.int8]
nvfp4_types = ["nvfp4"]
fp8_types = [torch.float8_e4m3fn]
```
**EN:** This block centralizes shared constants and parameter grids, including PREPARE_FINALIZE_INFO, EXPERT_INFO, MK_ALL_PREPARE_FINALIZE_TYPES, MK_MULTI_GPU_PREPARE_FINALIZE_TYPES, MK_SINGLE_GPU_PREPARE_FINALIZE_TYPES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 PREPARE_FINALIZE_INFO、EXPERT_INFO、MK_ALL_PREPARE_FINALIZE_TYPES、MK_MULTI_GPU_PREPARE_FINALIZE_TYPES、MK_SINGLE_GPU_PREPARE_FINALIZE_TYPES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `register_prepare_and_finalize` (lines 98-124)
```python
def register_prepare_and_finalize(
    kind,
    activation_format: mk.FusedMoEActivationFormat,
    supported_dtypes: list[torch.dtype | str],
    blocked_quantization_support: bool,
    backend: str | None,
    force_multigpu: bool = False,
    supports_apply_weight_on_input: bool = True,
):
    global PREPARE_FINALIZE_INFO
    global MK_ALL_PREPARE_FINALIZE_TYPES
    global MK_MULTI_GPU_PREPARE_FINALIZE_TYPES
    global MK_SINGLE_GPU_PREPARE_FINALIZE_TYPES
    assert kind not in PREPARE_FINALIZE_INFO

    PREPARE_FINALIZE_INFO[kind] = PrepareFinalizeInfo(
        activation_format,
        supported_dtypes,
        blocked_quantization_support,
        backend,
        supports_apply_weight_on_input,
    )
    MK_ALL_PREPARE_FINALIZE_TYPES.append(kind)
    if backend is not None or force_multigpu:
        MK_MULTI_GPU_PREPARE_FINALIZE_TYPES.append(kind)
    else:
        MK_SINGLE_GPU_PREPARE_FINALIZE_TYPES.append(kind)
```
**EN:** This helper function implements the shared logic for register prepare and finalize. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 register prepare and finalize 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `register_experts` (lines 127-151)
```python
def register_experts(
    kind,
    activation_format: mk.FusedMoEActivationFormat,
    supported_dtypes: list[torch.dtype | str],
    blocked_quantization_support: bool,
    supports_expert_map: bool,
    needs_matching_quant: bool = False,
    needs_deep_gemm: bool = False,
    needs_aiter: bool = False,
):
    global EXPERT_INFO
    global MK_FUSED_EXPERT_TYPES
    assert kind not in EXPERT_INFO

    EXPERT_INFO[kind] = ExpertInfo(
        activation_format,
        supported_dtypes,
        blocked_quantization_support,
        supports_expert_map,
        needs_matching_quant,
        needs_deep_gemm,
        needs_aiter,
    )

    MK_FUSED_EXPERT_TYPES.append(kind)
```
**EN:** This helper function implements the shared logic for register experts. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 register experts 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `prepare_finalize_info` (lines 154-157)
```python
def prepare_finalize_info(kind) -> PrepareFinalizeInfo:
    info = PREPARE_FINALIZE_INFO.get(kind)
    assert info is not None
    return info
```
**EN:** This helper function implements the shared logic for prepare finalize info. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 prepare finalize info 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `expert_info` (lines 160-163)
```python
def expert_info(kind) -> ExpertInfo:
    info = EXPERT_INFO.get(kind)
    assert info is not None
    return info
```
**EN:** This helper function implements the shared logic for expert info. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 expert info 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Top-level block starting at line 166 (lines 166-172)
```python
register_prepare_and_finalize(
    MoEPrepareAndFinalizeNoDPEPModular,
    standard_format,
    common_float_types,
    blocked_quantization_support=True,
    backend=None,
)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 174 (lines 174-181)
```python
register_experts(
    BatchedTritonExperts,
    batched_format,
    common_float_types,
    blocked_quantization_support=True,
    supports_expert_map=False,
    needs_matching_quant=True,
)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 183 (lines 183-190)
```python
register_experts(
    TritonExperts,
    standard_format,
    common_float_and_int_types,
    blocked_quantization_support=True,
    supports_expert_map=True,
    needs_matching_quant=True,
)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 192 (lines 192-198)
```python
register_experts(
    NaiveBatchedExperts,
    batched_format,
    common_float_and_int_types,
    blocked_quantization_support=True,
    supports_expert_map=True,
)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 201 (lines 201-223)
```python
if has_deep_ep() and not current_platform.has_device_capability(100):
    from vllm.model_executor.layers.fused_moe.prepare_finalize.deepep_ht import (
        DeepEPHTPrepareAndFinalize,
    )
    from vllm.model_executor.layers.fused_moe.prepare_finalize.deepep_ll import (
        DeepEPLLPrepareAndFinalize,
    )

    register_prepare_and_finalize(
        DeepEPHTPrepareAndFinalize,
        standard_format,
        common_float_types,
        blocked_quantization_support=True,
        backend="deepep_high_throughput",
    )

    register_prepare_and_finalize(
        DeepEPLLPrepareAndFinalize,
        batched_format,
        common_float_types,
        blocked_quantization_support=True,
        backend="deepep_low_latency",
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 225 (lines 225-237)
```python
if has_mori():
    from vllm.model_executor.layers.fused_moe.prepare_finalize.mori import (
        MoriPrepareAndFinalize,
    )

    register_prepare_and_finalize(
        MoriPrepareAndFinalize,
        standard_format,
        fp8_types,
        blocked_quantization_support=True,
        backend="mori",
        supports_apply_weight_on_input=False,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 239 (lines 239-267)
```python
if has_flashinfer_cutlass_fused_moe() and current_platform.has_device_capability(100):
    from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutlass_moe import (
        FlashInferExperts,
    )
    from vllm.model_executor.layers.fused_moe.prepare_finalize.flashinfer_nvlink_two_sided import (  # noqa: E501
        FlashInferNVLinkTwoSidedPrepareAndFinalize,
    )

    register_prepare_and_finalize(
        FlashInferNVLinkTwoSidedPrepareAndFinalize,
        standard_format,
        nvfp4_types + fp8_types,
        blocked_quantization_support=True,
        backend=None,
        force_multigpu=True,
        supports_apply_weight_on_input=False,
    )

    register_experts(
        FlashInferExperts,
        standard_format,
        nvfp4_types + fp8_types,
        blocked_quantization_support=True,
        # Note: this is a hack to get it to run for now
        supports_expert_map=True,
    )
else:
    FlashInferCutlassMoEPrepareAndFinalize = None
    FlashInferExperts = None
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 269 (lines 269-285)
```python
if (
    has_flashinfer_nvlink_one_sided()
    and has_flashinfer_cutlass_fused_moe()
    and current_platform.has_device_capability(100)
):
    from vllm.model_executor.layers.fused_moe.prepare_finalize.flashinfer_nvlink_one_sided import (  # noqa: E501
        FlashInferNVLinkOneSidedPrepareAndFinalize,
    )

    register_prepare_and_finalize(
        FlashInferNVLinkOneSidedPrepareAndFinalize,
        standard_format,
        nvfp4_types,
        blocked_quantization_support=False,
        backend="flashinfer_nvlink_one_sided",
        supports_apply_weight_on_input=False,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 287 (lines 287-298)
```python
if has_flashinfer_cutlass_fused_moe() and current_platform.has_device_capability(100):
    from vllm.model_executor.layers.fused_moe.experts.trtllm_nvfp4_moe import (
        TrtLlmNvFp4ExpertsModular,
    )

    register_experts(
        TrtLlmNvFp4ExpertsModular,
        standard_format,
        nvfp4_types,
        blocked_quantization_support=False,
        supports_expert_map=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 300 (lines 300-314)
```python
if has_aiter():
    from vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe import (
        AiterExperts,
    )

    register_experts(
        AiterExperts,
        standard_format,
        fp8_types,
        blocked_quantization_support=True,
        supports_expert_map=True,
        needs_aiter=True,
    )
else:
    AiterExperts = None
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 316 (lines 316-343)
```python
if has_deep_gemm() and is_deep_gemm_supported():
    register_experts(
        BatchedDeepGemmExperts,
        batched_format,
        fp8_types,
        blocked_quantization_support=True,
        supports_expert_map=False,
        needs_matching_quant=False,
        needs_deep_gemm=True,
    )
    register_experts(
        DeepGemmExperts,
        standard_format,
        fp8_types,
        blocked_quantization_support=True,
        supports_expert_map=True,
        needs_matching_quant=False,
        needs_deep_gemm=True,
    )
    register_experts(
        TritonOrDeepGemmExperts,
        standard_format,
        common_float_and_int_types,
        blocked_quantization_support=True,
        supports_expert_map=True,
        needs_matching_quant=True,
        needs_deep_gemm=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 345 (lines 345-367)
```python
if cutlass_fp8_supported():
    from vllm.model_executor.layers.fused_moe import (
        CutlassBatchedExpertsFp8,
        CutlassExpertsFp8,
    )

    register_experts(
        CutlassExpertsFp8,
        standard_format,
        fp8_types,
        blocked_quantization_support=False,
        supports_expert_map=False,
    )
    register_experts(
        CutlassBatchedExpertsFp8,
        batched_format,
        fp8_types,
        blocked_quantization_support=False,
        supports_expert_map=False,
    )
else:
    CutlassBatchedExpertsFp8 = None
    CutlassExpertsFp8 = None
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 369 (lines 369-382)
```python
if cutlass_fp4_supported():
    from vllm.model_executor.layers.fused_moe.experts.cutlass_moe import (
        CutlassExpertsFp4,
    )

    register_experts(
        CutlassExpertsFp4,
        standard_format,
        nvfp4_types,
        blocked_quantization_support=True,
        supports_expert_map=False,
    )
else:
    CutlassExpertsFp4 = None
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 384-424)
```python
MK_QUANT_CONFIGS: list[TestMoEQuantConfig | None] = [
    None,
    # per-channel / per-column weights and per-tensor activations
    TestMoEQuantConfig(
        quant_dtype=torch.float8_e4m3fn,
        per_out_ch_quant=True,
        per_act_token_quant=False,
        block_shape=None,
    ),
    # per-channel / per-column weights and per-token activations
    TestMoEQuantConfig(
        quant_dtype=torch.float8_e4m3fn,
        per_out_ch_quant=True,
        per_act_token_quant=True,
        block_shape=None,
    ),
    # per-tensor weights and per-tensor activations
    TestMoEQuantConfig(
        quant_dtype=torch.float8_e4m3fn,
        per_out_ch_quant=False,
        per_act_token_quant=False,
        block_shape=None,
    ),
    # per-tensor weights and per-token activations
    TestMoEQuantConfig(
        quant_dtype=torch.float8_e4m3fn,
        per_out_ch_quant=False,
        per_act_token_quant=True,
        block_shape=None,
    ),
    # block-quantized weights and 128 block per-token activations
    TestMoEQuantConfig(
        quant_dtype=torch.float8_e4m3fn,
        per_out_ch_quant=False,
        per_act_token_quant=False,
        block_shape=[128, 128],
    ),
    # TODO (varun) : Should we test the following combinations ?
    # block-quantized weights and per-token activations
    # block-quantized weights and per-tensor activations
]
```
**EN:** This block centralizes shared constants and parameter grids, including MK_QUANT_CONFIGS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MK_QUANT_CONFIGS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Top-level block starting at line 426 (lines 426-434)
```python
if cutlass_fp4_supported() or has_flashinfer_cutlass_fused_moe():
    MK_QUANT_CONFIGS += [
        TestMoEQuantConfig(
            quant_dtype="nvfp4",
            per_out_ch_quant=False,
            per_act_token_quant=False,
            block_shape=None,
        ),
    ]
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `_slice` (lines 437-440)
```python
def _slice(rank: int, num_local_experts: int, t: torch.Tensor) -> torch.Tensor:
    s = rank * num_local_experts
    e = s + num_local_experts
    return t[s:e]
```
**EN:** This helper function implements the shared logic for slice. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 slice 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_cutlass_strides` (lines 443-452)
```python
def make_cutlass_strides(
    e: int,
    n: int,
    k: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    ab_strides1 = torch.full((e,), k, device="cuda", dtype=torch.int64)
    ab_strides2 = torch.full((e,), n, device="cuda", dtype=torch.int64)
    c_strides1 = torch.full((e,), 2 * n, device="cuda", dtype=torch.int64)
    c_strides2 = torch.full((e,), k, device="cuda", dtype=torch.int64)
    return ab_strides1, ab_strides2, c_strides1, c_strides2
```
**EN:** This helper function implements the shared logic for cutlass strides. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 cutlass strides 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_fused_experts` (lines 455-485)
```python
def make_fused_experts(
    fused_experts_type: mk.FusedMoEExpertsModular,
    moe: FusedMoEConfig,
    quant_config: FusedMoEQuantConfig,
    num_dispatchers: int,
    N: int,
) -> mk.FusedMoEExpertsModular:
    if (
        fused_experts_type.activation_format()
        == mk.FusedMoEActivationFormat.BatchedExperts
    ):
        kwargs = {
            "moe_config": moe,
            "quant_config": quant_config,
            "max_num_tokens": moe.max_num_tokens,
            "num_dispatchers": num_dispatchers,
        }
    else:
        kwargs = {
            "moe_config": moe,
            "quant_config": quant_config,
        }

    torch.set_printoptions(threshold=0, edgeitems=0, linewidth=10000)

    print(f"Making {fused_experts_type.__class__.__name__} {kwargs} ...")
    experts = fused_experts_type(**kwargs)

    torch.set_printoptions(threshold=1000, edgeitems=5, linewidth=80)

    return experts
```
**EN:** This helper function implements the shared logic for fused experts. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 fused experts 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `dataclasses -> dataclass`
- `torch`
- `vllm.model_executor.layers.fused_moe.modular_kernel`
- `vllm.model_executor.layers.fused_moe -> TritonExperts`
- `vllm.model_executor.layers.fused_moe.config -> FusedMoEConfig, FusedMoEQuantConfig`
- `vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe -> BatchedDeepGemmExperts`
- `vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe -> DeepGemmExperts`
- `vllm.model_executor.layers.fused_moe.experts.fused_batched_moe -> BatchedTritonExperts, NaiveBatchedExperts`
- `vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe -> TritonOrDeepGemmExperts`
- `vllm.model_executor.layers.fused_moe.prepare_finalize -> MoEPrepareAndFinalizeNoDPEPModular`
- `vllm.model_executor.layers.quantization.utils.nvfp4_utils -> cutlass_fp4_supported`
- `vllm.model_executor.layers.quantization.utils.w8a8_utils -> cutlass_fp8_supported`
- `vllm.platforms -> current_platform`
- `vllm.utils.deep_gemm -> is_deep_gemm_supported`
- `vllm.utils.flashinfer -> has_flashinfer_cutlass_fused_moe, has_flashinfer_nvlink_one_sided`
- `vllm.utils.import_utils -> has_aiter, has_deep_ep, has_deep_gemm, has_mori`
