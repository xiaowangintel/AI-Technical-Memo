# test_onednn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/test_onednn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / test_onednn, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / test_onednn 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-9)
```python
import pytest
import torch

from tests.kernels.utils import to_int8
from vllm import _custom_ops as ops
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.utils; and vLLM components like vllm, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.utils；vLLM 内部组件，例如 vllm、vllm.platforms。

### Top-level block starting at line 11 (lines 11-12)
```python
if not current_platform.is_cpu():
    pytest.skip("skipping CPU-only tests", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 14-26)
```python
NK_FACTORS = [
    (256, 128),
    (4096, 4096),
    (16384, 4096),
    (1023, 491),
    (1001, 15),
]
M_FACTORS = [
    (16, 1, 32, 128, 64),
    (1, 17, 1, 31, 17),
]
CACHE_SIZES = [2]
DTYPE = [torch.bfloat16]
```
**EN:** This block centralizes shared constants and parameter grids, including NK_FACTORS, M_FACTORS, CACHE_SIZES, DTYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NK_FACTORS、M_FACTORS、CACHE_SIZES、DTYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `rand_int8` (lines 29-30)
```python
def rand_int8(shape: tuple, device: str = "cpu"):
    return to_int8(torch.rand(shape, device=device) * 255 - 128)
```
**EN:** This helper function implements the shared logic for rand int8. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 rand int8 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_int8_scaled_mm` (lines 33-50)
```python
def ref_int8_scaled_mm(
    a: torch.Tensor,
    b: torch.Tensor,
    scale_a: torch.Tensor,
    scale_b: torch.Tensor,
    azp: torch.Tensor | None,
    bias: torch.Tensor | None,
    output_type: torch.dtype,
):
    if azp is not None:
        a = a.to(dtype=torch.float32) - azp.to(dtype=torch.float32)
    output = torch.mm(
        (scale_a * a.to(dtype=torch.float32)), (scale_b * b.to(dtype=torch.float32))
    )
    if bias is not None:
        output += bias.float()

    return output.to(dtype=output_type)
```
**EN:** This helper acts as a reference implementation for int8 scaled mm. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 int8 scaled mm 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `onednn_int8_gemm_test_helper` (lines 53-107)
```python
def onednn_int8_gemm_test_helper(
    primitive_cache_size: int,
    m: int,
    n: int,
    k: int,
    per_tensor_a_quant: bool,
    per_tensor_b_quant: bool,
    use_azp: bool,
    use_bias: bool,
    out_dtype: torch.dtype = torch.bfloat16,
    device: str = "cpu",
):
    # Test for a oneDNN kernel with per-tensor / per-token activation
    # quantization and per-tensor / per-output channel weight quantization.
    a = to_int8(torch.randn((m, k), device=device) * 5)
    b = to_int8(torch.randn((n, k), device=device).t() * 5)

    a_scales_shape = (1, 1) if per_tensor_a_quant else (m, 1)
    b_scales_shape = (1, 1) if per_tensor_b_quant else (1, n)

    scale_a = torch.randn(a_scales_shape, device=device, dtype=torch.float32)
    scale_b = torch.randn(b_scales_shape, device=device, dtype=torch.float32)

    if use_azp:
        azp = torch.rand(a_scales_shape, dtype=torch.float32) * 10 + 1.5
        azp = (azp / scale_a).round().to(dtype=torch.int32)
        azp_adj = scale_b * b.sum(dim=0, keepdim=True, dtype=torch.float32)
    else:
        azp = None
        azp_adj = None

    bias = torch.rand((n,), device=device, dtype=out_dtype) * 10 if use_bias else None

    handler = ops.create_onednn_scaled_mm(
        b,
        scale_b,
        out_dtype,
        not per_tensor_a_quant,
        use_azp,
        primitive_cache_size,
    )

    out = torch.zeros((m, n), dtype=out_dtype)
    ops.onednn_scaled_mm(handler, a, out, scale_a, azp, azp_adj, bias)
    baseline = ref_int8_scaled_mm(a, b, scale_a, scale_b, azp, bias, out_dtype)

    torch.testing.assert_close(out, baseline, rtol=1e-1, atol=1e0)

    if use_bias:
        # To test runtime bias setting
        out = torch.zeros((m, n), dtype=out_dtype)
        ops.onednn_scaled_mm(handler, a, out, scale_a, azp, azp_adj, None)
        baseline = ref_int8_scaled_mm(a, b, scale_a, scale_b, azp, None, out_dtype)

        torch.testing.assert_close(out, baseline, rtol=1e-1, atol=1e0)
```
**EN:** This helper function implements the shared logic for onednn int8 gemm test helper. numeric results are compared against a reference with explicit tolerances.
**CN:** 该辅助函数实现了 onednn int8 gemm test helper 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较。

### Function `onednn_gemm_test_helper` (lines 110-154)
```python
def onednn_gemm_test_helper(
    primitive_cache_size: int,
    m: int,
    n: int,
    k: int,
    use_bias: bool,
    use_stride: bool,
    dtype: torch.dtype = torch.bfloat16,
    device: str = "cpu",
):
    if use_stride:
        a = torch.rand((m, 2 * k), dtype=dtype, device=device) * 1.5
        a = a[:, :k]
    else:
        a = torch.rand((m, k), dtype=dtype, device=device) * 1.5

    b = torch.rand((n, k), dtype=dtype, device=device) * 1.5

    if use_bias:
        bias = torch.rand((n,), device=device, dtype=dtype) * 5
        bias_f32 = bias.float()
    else:
        bias = None
        bias_f32 = None

    handler = ops.create_onednn_mm(
        b.t(),
        primitive_cache_size,
    )

    out = ops.onednn_mm(handler, a, bias)
    baseline = torch.nn.functional.linear(a.float(), b.float(), bias_f32).to(
        dtype=a.dtype
    )

    torch.testing.assert_close(out, baseline)

    if use_bias:
        # To test runtime bias setting
        out = ops.onednn_mm(handler, a, None)
        baseline = torch.nn.functional.linear(a.float(), b.float(), None).to(
            dtype=a.dtype
        )

        torch.testing.assert_close(out, baseline)
```
**EN:** This helper function implements the shared logic for onednn gemm test helper. numeric results are compared against a reference with explicit tolerances.
**CN:** 该辅助函数实现了 onednn gemm test helper 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_onednn_int8_scaled_gemm` (lines 157-187)
```python
@pytest.mark.parametrize("n,k", NK_FACTORS)
@pytest.mark.parametrize("m_list", M_FACTORS)
@pytest.mark.parametrize("per_tensor_a_scale", [True, False])
@pytest.mark.parametrize("per_tensor_b_scale", [True, False])
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.parametrize("use_azp", [True, False])
@pytest.mark.parametrize("output_type", DTYPE)
@pytest.mark.parametrize("primitive_cache_size", CACHE_SIZES)
def test_onednn_int8_scaled_gemm(
    n: int,
    k: int,
    m_list: tuple[int, ...],
    per_tensor_a_scale: bool,
    per_tensor_b_scale: bool,
    use_bias: bool,
    use_azp: bool,
    output_type: torch.dtype,
    primitive_cache_size: int,
):
    for m in m_list:
        onednn_int8_gemm_test_helper(
            primitive_cache_size=primitive_cache_size,
            m=m,
            n=n,
            k=k,
            per_tensor_a_quant=per_tensor_a_scale,
            per_tensor_b_quant=per_tensor_b_scale,
            use_bias=use_bias,
            use_azp=use_azp,
            out_dtype=output_type,
        )
```
**EN:** This pytest case verifies onednn int8 scaled gemm. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as n, k, m_list, per_tensor_a_scale.
**CN:** 该 pytest 用例验证 onednn int8 scaled gemm 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 n、k、m_list、per_tensor_a_scale 等 fixture 或输入。

### Function `test_onednn_gemm` (lines 190-214)
```python
@pytest.mark.parametrize("n,k", NK_FACTORS)
@pytest.mark.parametrize("m_list", M_FACTORS)
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.parametrize("use_stride", [True, False])
@pytest.mark.parametrize("dtype", DTYPE)
@pytest.mark.parametrize("primitive_cache_size", CACHE_SIZES)
def test_onednn_gemm(
    n: int,
    k: int,
    m_list: tuple[int, ...],
    use_bias: bool,
    use_stride: bool,
    dtype: torch.dtype,
    primitive_cache_size: int,
):
    for m in m_list:
        onednn_gemm_test_helper(
            primitive_cache_size=primitive_cache_size,
            m=m,
            n=n,
            k=k,
            use_bias=use_bias,
            use_stride=use_stride,
            dtype=dtype,
        )
```
**EN:** This pytest case verifies onednn gemm. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as n, k, m_list, use_bias.
**CN:** 该 pytest 用例验证 onednn gemm 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 n、k、m_list、use_bias 等 fixture 或输入。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `tests.kernels.utils -> to_int8`
- `vllm -> _custom_ops`
- `vllm.platforms -> current_platform`
