# models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fusions_e2e/models.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for compile tests in compile / fusions_e2e / models; it defines reusable helpers, reference utilities, or backend wrappers shared by multiple test cases. / compile / fusions_e2e / models 对应的编译测试支撑模块；它定义了可复用的辅助函数、参考工具或后端包装器，供多个测试用例共享。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-10)
```python
import pytest

from vllm._aiter_ops import is_aiter_found_and_supported
from vllm.platforms import current_platform
from vllm.utils.flashinfer import has_flashinfer
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from .common import AttentionBackendCase, Matches, ModelFusionInfo, is_blackwell
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest; shared test helpers from .common; and vLLM components like vllm._aiter_ops, vllm.platforms, vllm.utils.flashinfer, vllm.v1.attention.backends.registry.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest；共享测试辅助模块，例如 .common；vLLM 内部组件，例如 vllm._aiter_ops、vllm.platforms、vllm.utils.flashinfer、vllm.v1.attention.backends.registry。

### Constants and module state (lines 13-222)
```python
FLASHINFER_ATTN = pytest.param(
    AttentionBackendCase(
        backend=AttentionBackendEnum.FLASHINFER,
        model_kwargs=dict(kv_cache_dtype="fp8"),
    ),
    id="FLASHINFER",
    marks=pytest.mark.skipif(
        not is_blackwell() or not has_flashinfer(),
        reason="FI backend requires Blackwell and FlashInfer",
    ),
)

TRITON_ATTN = pytest.param(
    AttentionBackendCase(backend=AttentionBackendEnum.TRITON_ATTN), id="TRITON_ATTN"
)

ROCM_ATTN = pytest.param(
    AttentionBackendCase(backend=AttentionBackendEnum.ROCM_ATTN),
    id="ROCM_ATTN",
    marks=pytest.mark.skipif(
        not current_platform.is_rocm(),
        reason="ROCm attention only for AMD",
    ),
)

ROCM_AITER_UNIFIED_ATTN = pytest.param(
    AttentionBackendCase(backend=AttentionBackendEnum.ROCM_AITER_UNIFIED_ATTN),
    id="ROCM_AITER_UNIFIED_ATTN",
    marks=pytest.mark.skipif(
        not is_aiter_found_and_supported(),
        reason="ROCM_AITER_UNIFIED_ATTN only for AMD when AITER is installed",
    ),
)

FLASHINFER_MLA_ATTN = pytest.param(
    AttentionBackendCase(backend=AttentionBackendEnum.FLASHINFER_MLA),
    id="FLASHINFER_MLA",
    marks=pytest.mark.skipif(
        not is_blackwell() or not has_flashinfer(),
        reason="FI backend requires Blackwell and FlashInfer",
# ... excerpt ...
        # async_tp=n_layers * 2,
    ),
)

deepseek_r1_fp4 = ModelFusionInfo(
    model_name="nvidia/DeepSeek-R1-0528-NVFP4-v2",
    matches=lambda n_layers: Matches(
        rms_quant_fusion=0,
        act_quant_fusion=min(3, n_layers),
        attn_quant_fusion=n_layers,
        ar_rms_fusion=n_layers * 2 + 1,
    ),
)

deepseek_v32_fp4 = ModelFusionInfo(
    model_name="nvidia/DeepSeek-V3.2-NVFP4",
    matches=lambda n_layers: Matches(
        rms_quant_fusion=0,
        # silu+quant on dense layers only; MoE hides the act+quant site
        act_quant_fusion=min(3, n_layers),
        # MLA attn + NVFP4 output quant fuses on sparse MLA output path
        attn_quant_fusion=n_layers,
        ar_rms_fusion=n_layers * 2 + 1,
    ),
)

gpt_oss_20b = ModelFusionInfo(
    model_name="openai/gpt-oss-20b",
    matches=lambda n_layers: Matches(
        ar_rms_fusion=n_layers * 2 + 1,
        sequence_parallel=n_layers * 2 + 1,
        async_tp=n_layers * 2,
    ),
    model_kwargs=(
        {"quantization_config": {"moe": {"activation": "mxfp8"}}}
        if is_blackwell()
        else {}
    ),
)
```
**EN:** This block centralizes shared constants and parameter grids, including FLASHINFER_ATTN, TRITON_ATTN, ROCM_ATTN, ROCM_AITER_UNIFIED_ATTN, FLASHINFER_MLA_ATTN. Those values keep later pytest scenarios consistent and make the test matrix easier to audit. Only the key portions are shown here because the block is large.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FLASHINFER_ATTN、TRITON_ATTN、ROCM_ATTN、ROCM_AITER_UNIFIED_ATTN、FLASHINFER_MLA_ATTN。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `pytest`
- `vllm._aiter_ops -> is_aiter_found_and_supported`
- `vllm.platforms -> current_platform`
- `vllm.utils.flashinfer -> has_flashinfer`
- `vllm.v1.attention.backends.registry -> AttentionBackendEnum`
- `.common -> AttentionBackendCase, Matches, ModelFusionInfo, is_blackwell`
