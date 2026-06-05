# test_rocm_skinny_gemms.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_rocm_skinny_gemms.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_rocm_skinny_gemms, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_rocm_skinny_gemms 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-12)
```python
import math

import pytest
import torch

import vllm._custom_ops as ops
from tests.kernels.quant_utils import ref_dynamic_per_tensor_fp8_quant
from vllm.platforms import current_platform
from vllm.platforms.rocm import on_gfx950
from vllm.utils.platform_utils import num_compute_units
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as math, pytest, torch; shared test helpers from tests.kernels.quant_utils; and vLLM components like vllm._custom_ops, vllm.platforms, vllm.platforms.rocm, vllm.utils.platform_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 math、pytest、torch；共享测试辅助模块，例如 tests.kernels.quant_utils；vLLM 内部组件，例如 vllm._custom_ops、vllm.platforms、vllm.platforms.rocm、vllm.utils.platform_utils。

### Constants and module state (lines 14-109)
```python
DTYPES = [torch.bfloat16, torch.float16]
BIAS_MODES = [0, 1, 2]
# Specific (N, K, M) combinations for targeted testing
NKM_FACTORS_LLMM1 = [
    # Small, medium, large cases
    (1, 8, 16),
    (1, 32, 64),
    (1, 128, 256),
    (1, 512, 1024),
    (1, 2048, 4096),
    # Edge cases with specific K sizes
    (1, 6144, 1024),
    (1, 8192, 2048),
    # Very large case
    (1, 4096, 8192),
]

NKM_FACTORS_WVSPLITK = [
    # Different batch sizes with key dimensions
    (1, 32, 16),
    (1, 64, 64),
    (2, 256, 256),
    (3, 1024, 1024),
    (4, 4096, 4096),
    (4, 4096, 4096 + 1),
    (4, 4096 + 16, 4096),
    (4, 4096 + 16, 4096 + 1),
    # Extended K values
    (1, 9216, 512),
    (2, 10240, 1024),
    (4, 16384, 8192),
    (4, 16384 * 2, 8192),
    (4, 16384 * 2, 8192 + 1),
    (4, 16384 * 2 + 16, 8192),
    (4, 16384 * 2 + 16, 8192 + 1),
    # Minimum M constraint validation (m >= 8)
    (1, 64, 8),
    (2, 128, 8),
    (4, 256, 8),
]
# ... excerpt ...
    128,
]
K_FACTORS_WVSPLITKRC = [2880, 2880 + 8, 3072, 3072 + 8]
M_FACTORS_WVSPLITKRC = [128, 128 + 16, 256, 256 + 16, 640, 640 + 16]

NKM_FACTORS_WVSPLITK_FP8 = [
    # FP8-specific cases with K % 16 == 0
    (1, 16, 16),
    (1, 32, 16 + 16),
    (1, 64, 64),
    (1, 64, 64 + 16),
    (1, 64 + 16, 64),
    (1, 64 + 16, 64 + 16),
    (4, 64, 64),
    (4, 64, 64 + 16),
    (4, 64 + 16, 64),
    (4, 64 + 16, 64 + 16),
    (2, 512, 512),
    (3, 512, 512),
    (3, 512, 512 + 16),
    (4, 512, 512),
    (3, 2048, 2048),
    (3, 2048, 2048 + 16),
    (4, 2048 + 16, 2048),
    (4, 2048 + 16, 2048 + 16),
    (4, 4096, 4096),
    (4, 16400, 2048),
    (4, 16400, 2048 + 16),
    # Extended FP8 dimensions not covered by WVSPLITK
    (1, 14336, 1024),
    (2, 24576, 2048),
    (4, 32768, 28672),
    (4, 32768 * 2, 28672),
    (4, 32768 * 2, 28672 + 16),
    (4, 32768 * 2 + 16, 28672),
    (4, 32768 * 2 + 16, 28672 + 16),
]

SEEDS = [0]
```
**EN:** This block centralizes shared constants and parameter grids, including DTYPES, BIAS_MODES, NKM_FACTORS_LLMM1, NKM_FACTORS_WVSPLITK, N_FACTORS_WVSPLITKRC. Those values keep later pytest scenarios consistent and make the test matrix easier to audit. Only the key portions are shown here because the block is large.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DTYPES、BIAS_MODES、NKM_FACTORS_LLMM1、NKM_FACTORS_WVSPLITK、N_FACTORS_WVSPLITKRC。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。 由于该代码块较大，这里只展示关键片段。

### Function `pad_fp8` (lines 112-116)
```python
def pad_fp8(weight):
    num_pad = 256 // weight.element_size()
    import torch.nn.functional as F

    return F.pad(weight, (0, num_pad), "constant", 0)[..., :-num_pad]
```
**EN:** This helper function implements the shared logic for pad FP8. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 pad FP8 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_rocm_wvsplitkrc_kernel` (lines 119-172)
```python
@pytest.mark.parametrize("xnorm", [False, True])
@pytest.mark.parametrize("n", N_FACTORS_WVSPLITKRC)
@pytest.mark.parametrize("k", K_FACTORS_WVSPLITKRC)
@pytest.mark.parametrize("m", M_FACTORS_WVSPLITKRC)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("padded_a", [False, True])
@pytest.mark.parametrize("bias_mode", BIAS_MODES)
@pytest.mark.skipif(not current_platform.is_rocm(), reason="only test for rocm")
@pytest.mark.skipif(not on_gfx950(), reason="only meant for gfx950")
def test_rocm_wvsplitkrc_kernel(xnorm, n, k, m, dtype, seed, padded_a, bias_mode):
    torch.manual_seed(seed)
    cu_count = num_compute_units()

    # Next ^2 of n
    N_p2 = 1 << (n - 1).bit_length()
    # With 64 Ms per CU (each of 4 SIMDs working on a 16x16 tile),
    # and each working on a 512-shard of K, how many CUs would we need?
    rndup_cus = ((m + 64 - 1) // 64) * ((k + 512 - 1) // 512)
    # How many of 4 waves in a group can work on same 16 Ms at same time?
    # This reduces the Ms each group works on, i.e. increasing the number of CUs needed.
    GrpsShrB = min(N_p2 // 16, 4)
    # Given the above, how many CUs would we need?
    CuNeeded = rndup_cus * GrpsShrB
    # candidate for atomic reduce count splitk?
    fits_wvsplitkrc = (N_p2 * m * ((k + 512 - 1) // 512)) <= 128 * 1024 * 12
    fits_wvsplitkrc &= CuNeeded <= cu_count

    if not fits_wvsplitkrc:
        pytest.skip("Too large for wvSplitKrc")

    xavier = (
        math.sqrt(2 / k) if xnorm else 1
    )  # normalize to avoid large output-bias deltas
    A = (torch.rand(n, k, dtype=dtype, device="cuda") * 2 - 1) * xavier
    B = (torch.rand(m, k, dtype=dtype, device="cuda") * 2 - 1) * xavier
    if padded_a:
        A = pad_fp8(A)

    BIAS = None
    if bias_mode == 1:
        BIAS = torch.rand(m, dtype=dtype, device="cuda") * 2 - 1
    elif bias_mode == 2:
        BIAS = torch.rand(n, m, dtype=dtype, device="cuda") * 2 - 1
    elif bias_mode == 3:
        BIAS = torch.rand(1, m, dtype=dtype, device="cuda") * 2 - 1

    ref_out = torch.nn.functional.linear(A, B, BIAS)
    out = ops.wvSplitKrc(A, B, cu_count, BIAS)

    if xnorm:
        torch.testing.assert_close(out, ref_out, atol=1e-3, rtol=1e-8)
    else:
        torch.testing.assert_close(out, ref_out, atol=1e-3, rtol=1e-2)
```
**EN:** This pytest case verifies ROCm wvsplitkrc kernel. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as xnorm, n, k, m. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 ROCm wvsplitkrc kernel 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 xnorm、n、k、m 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_rocm_llmm1_kernel` (lines 175-192)
```python
@pytest.mark.parametrize("n,k,m", NKM_FACTORS_LLMM1)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("rows_per_block", [2, 4, 8, 16])
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.skipif(not current_platform.is_rocm(), reason="only test for rocm")
@torch.inference_mode()
def test_rocm_llmm1_kernel(n, k, m, dtype, rows_per_block, seed):
    torch.manual_seed(seed)
    # TODO: Zero-centering the inputs causes errors for LLMM1!
    #      Without that the numbers quickly saturate, and may
    #      be giving false matches.
    A = torch.rand(n, k, dtype=dtype, device="cuda")
    B = torch.rand(m, k, dtype=dtype, device="cuda")

    ref_out = torch.matmul(A, B.t())
    out = ops.LLMM1(B, A, rows_per_block)

    torch.testing.assert_close(out, ref_out, atol=1e-8, rtol=1e-2)
```
**EN:** This pytest case verifies ROCm llmm1 kernel. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as n, k, m, dtype. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 ROCm llmm1 kernel 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 n、k、m、dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_rocm_wvsplitk_kernel` (lines 195-231)
```python
@pytest.mark.parametrize("xnorm", [False, True])
@pytest.mark.parametrize("n,k,m", NKM_FACTORS_WVSPLITK)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.skipif(not current_platform.is_rocm(), reason="only test for rocm")
@pytest.mark.parametrize("bias_mode", BIAS_MODES)
@pytest.mark.parametrize("padded_a", [False, True])
@pytest.mark.parametrize("padded_b", [False, True])
def test_rocm_wvsplitk_kernel(
    xnorm, n, k, m, dtype, seed, bias_mode, padded_a, padded_b
):
    torch.manual_seed(seed)
    cu_count = num_compute_units()

    xavier = (
        math.sqrt(2 / k) if xnorm else 1
    )  # normalize to avoid large output-bias deltas
    A = (torch.rand(n, k, dtype=dtype, device="cuda") * 2 - 1) * xavier
    B = (torch.rand(m, k, dtype=dtype, device="cuda") * 2 - 1) * xavier

    BIAS = None
    if bias_mode == 1:
        BIAS = torch.rand(m, dtype=dtype, device="cuda") * 2 - 1
    elif bias_mode == 2:
        BIAS = torch.rand(n, m, dtype=dtype, device="cuda") * 2 - 1

    if padded_a:
        A = pad_fp8(A)
    if padded_b:
        B = pad_fp8(B)

    ref_out = torch.nn.functional.linear(A, B, BIAS)
    out = ops.wvSplitK(B, A.view(-1, A.size(-1)), cu_count, BIAS)

    # Accumulation error in fp16 GEMM scales with sqrt(K)
    atol = torch.finfo(dtype).eps * math.sqrt(k)
    torch.testing.assert_close(out, ref_out, atol=atol, rtol=1e-2)
```
**EN:** This pytest case verifies ROCm wvsplitk kernel. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as xnorm, n, k, m. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 ROCm wvsplitk kernel 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 xnorm、n、k、m 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_rocm_wvsplitk_fp8_kernel` (lines 234-274)
```python
@pytest.mark.parametrize("xnorm", [False, True])
@pytest.mark.parametrize("n,k,m", NKM_FACTORS_WVSPLITK_FP8)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("padded_a", [False, True])
@pytest.mark.parametrize("padded_b", [False, True])
@pytest.mark.parametrize("biased", [False, True])
@pytest.mark.skipif(
    not (current_platform.is_rocm() and current_platform.supports_fp8()),
    reason="only test for rocm fp8",
)
def test_rocm_wvsplitk_fp8_kernel(
    xnorm, n, k, m, dtype, seed, padded_a, padded_b, biased
):
    torch.manual_seed(seed)

    xavier = math.sqrt(2 / k) if xnorm else 1  # normalize to avoid large deltas
    A = (torch.rand(n, k, device="cuda") * 2 - 1) * xavier
    B = (torch.rand(m, k, device="cuda") * 2 - 1) * xavier

    A, scale_a = ref_dynamic_per_tensor_fp8_quant(A)
    B, scale_b = ref_dynamic_per_tensor_fp8_quant(B)
    if padded_b:
        B = pad_fp8(B)
    if padded_a:
        A = pad_fp8(A)

    BIAS = None if (not biased) else (torch.rand(m, dtype=dtype, device="cuda") * 2 - 1)

    ref_out = torch._scaled_mm(
        A, B.t(), out_dtype=dtype, scale_a=scale_a, scale_b=scale_b, bias=BIAS
    )
    out = ops.wvSplitKQ(B, A, dtype, scale_a, scale_b, num_compute_units(), BIAS)

    if xnorm:
        torch.testing.assert_close(out, ref_out, atol=1e-3, rtol=1e-8)
    elif k >= 32 * 1024:
        # wider pytrch thresh for large-K & no xnorm
        torch.testing.assert_close(out, ref_out, atol=0.07, rtol=5e-2)
    else:
        torch.testing.assert_close(out, ref_out, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies ROCm wvsplitk FP8 kernel. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as xnorm, n, k, m. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 ROCm wvsplitk FP8 kernel 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 xnorm、n、k、m 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `math`
- `pytest`
- `torch`
- `vllm._custom_ops`
- `tests.kernels.quant_utils -> ref_dynamic_per_tensor_fp8_quant`
- `vllm.platforms -> current_platform`
- `vllm.platforms.rocm -> on_gfx950`
- `vllm.utils.platform_utils -> num_compute_units`
