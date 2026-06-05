# test_allspark_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_allspark_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_allspark_gemm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_allspark_gemm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-16)
```python
import pytest
import torch

from tests.kernels.utils import DEFAULT_OPCHECK_TEST_UTILS, opcheck
from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.allspark_utils import (
    ALLSPARK_AMPERE_K_ALIGN,
    ALLSPARK_AMPERE_M_CUBLAS_THRESHOLD,
    ALLSPARK_AMPERE_N_ALIGN,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import quantize_weights
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
from vllm.utils.platform_utils import num_compute_units
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.utils; and vLLM components like vllm, vllm.model_executor.layers.quantization.utils.allspark_utils, vllm.model_executor.layers.quantization.utils.quant_utils, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.utils；vLLM 内部组件，例如 vllm、vllm.model_executor.layers.quantization.utils.allspark_utils、vllm.model_executor.layers.quantization.utils.quant_utils、vllm.platforms。

### Function `is_gptq_allspark_supported` (lines 19-28)
```python
def is_gptq_allspark_supported(min_capability: int, max_capability: int) -> bool:
    if not current_platform.is_cuda():
        return False

    capability = current_platform.get_device_capability()
    assert capability is not None

    return (
        capability.to_int() >= min_capability and capability.to_int() <= max_capability
    )
```
**EN:** This helper function implements the shared logic for is gptq allspark supported. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 is gptq allspark supported 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Constants and module state (lines 31-43)
```python
MNK_FACTORS = [
    (1, 4, 8),
    (13, 17, 67),
    (26, 37, 13),
    (48, 16, 24),
    (67, 13, 88),
    (257, 13, 11),
    (658, 13, 11),
    (1033, 9, 17),
]

DTYPES = [torch.float16, torch.bfloat16]
HAS_ZP_OPTS = [False, True]
```
**EN:** This block centralizes shared constants and parameter grids, including MNK_FACTORS, DTYPES, HAS_ZP_OPTS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MNK_FACTORS、DTYPES、HAS_ZP_OPTS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `compute_max_diff` (lines 46-49)
```python
def compute_max_diff(output, output_ref):
    return torch.mean(torch.abs(output - output_ref)) / torch.mean(
        torch.abs(output_ref)
    )
```
**EN:** This helper function implements the shared logic for compute max diff. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 compute max diff 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `rand_data` (lines 52-53)
```python
def rand_data(shape, dtype=torch.float16):
    return torch.randn(shape, dtype=dtype, device="cuda")
```
**EN:** This helper function implements the shared logic for rand data. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 rand data 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_gptq_allspark_gemm_ampere` (lines 56-128)
```python
@pytest.mark.skipif(
    not is_gptq_allspark_supported(80, 89),
    reason="AllSpark Ampere kernel is not supported on this GPU type.",
)
@pytest.mark.parametrize("mnk_factors", MNK_FACTORS)
@pytest.mark.parametrize("group_size", [-1])
@pytest.mark.parametrize("has_zp", HAS_ZP_OPTS)
@pytest.mark.parametrize("dtype", DTYPES)
def test_gptq_allspark_gemm_ampere(mnk_factors, group_size, has_zp, dtype):
    m_factor, n_factor, k_factor = mnk_factors
    m = m_factor
    n = n_factor * ALLSPARK_AMPERE_N_ALIGN
    k = k_factor * ALLSPARK_AMPERE_K_ALIGN

    input = rand_data((m, k), dtype=dtype)
    weight = rand_data((k, n), dtype=dtype)

    # Quantize (and apply act_order if provided)
    w_ref, qw, s, zp = quantize_weights(
        weight, scalar_types.uint8b128, group_size, has_zp
    )

    qw = qw.to(torch.uint8)
    if has_zp:
        zp = zp.to(dtype)
    properties = torch.cuda.get_device_properties(qw.device.index)
    sm_count = num_compute_units(qw.device.index)
    sm_version = properties.major * 10 + properties.minor

    n_32align = (n + 32 - 1) // 32 * 32

    qw_reorder, s_reorder, zp_reorder = ops.allspark_repack_weight(qw, s, zp, has_zp)
    opcheck(
        torch.ops._C.rearrange_kn_weight_as_n32k16_order,
        (qw, s, zp, has_zp, qw_reorder, s_reorder, zp_reorder, k, n, n_32align),
    )

    opcheck(
        torch.ops._C.allspark_w8a16_gemm,
        (
            input,
            qw_reorder,
            s_reorder,
            zp_reorder,
            n,
            group_size,
            sm_count,
            sm_version,
            ALLSPARK_AMPERE_M_CUBLAS_THRESHOLD,
            has_zp,
            True,
        ),
        test_utils=DEFAULT_OPCHECK_TEST_UTILS,
    )
    output = ops.allspark_w8a16_gemm(
        input,
        qw_reorder,
        s_reorder,
        zp_reorder,
        n,
        group_size,
        sm_count,
        sm_version,
        ALLSPARK_AMPERE_M_CUBLAS_THRESHOLD,
        has_zp,
        True,
    )

    output_ref = torch.matmul(input, w_ref)
    torch.accelerator.synchronize()
    max_diff = compute_max_diff(output, output_ref)

    assert max_diff < 0.04
```
**EN:** This pytest case verifies gptq allspark gemm ampere. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as mnk_factors, group_size, has_zp, dtype. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly.
**CN:** 该 pytest 用例验证 gptq allspark gemm ampere 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 mnk_factors、group_size、has_zp、dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `tests.kernels.utils -> DEFAULT_OPCHECK_TEST_UTILS, opcheck`
- `vllm -> _custom_ops`
- `vllm.model_executor.layers.quantization.utils.allspark_utils -> ALLSPARK_AMPERE_K_ALIGN, ALLSPARK_AMPERE_M_CUBLAS_THRESHOLD, ALLSPARK_AMPERE_N_ALIGN`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> quantize_weights`
- `vllm.platforms -> current_platform`
- `vllm.scalar_type -> scalar_types`
- `vllm.utils.platform_utils -> num_compute_units`
