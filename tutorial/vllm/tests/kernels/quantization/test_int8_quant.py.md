# test_int8_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_int8_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_int8_quant, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_int8_quant 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-10)
```python
import pytest
import torch

from tests.kernels.quant_utils import ref_dynamic_per_token_quant
from tests.kernels.utils import opcheck
from vllm._custom_ops import scaled_int8_quant
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.quant_utils, tests.kernels.utils; and vLLM components like vllm._custom_ops, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.quant_utils、tests.kernels.utils；vLLM 内部组件，例如 vllm._custom_ops、vllm.utils.torch_utils。

### Constants and module state (lines 12-16)
```python
DTYPES = [torch.bfloat16, torch.float]
HIDDEN_SIZES = [17, 1024, 1025, 1026, 5137, 8193]
NUM_TOKENS = [1, 7, 4096]
SEEDS = [0]
SCALE = [0.1, 2.1]
```
**EN:** This block centralizes shared constants and parameter grids, including DTYPES, HIDDEN_SIZES, NUM_TOKENS, SEEDS, SCALE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DTYPES、HIDDEN_SIZES、NUM_TOKENS、SEEDS、SCALE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `opcheck_int8_quant_static` (lines 19-23)
```python
def opcheck_int8_quant_static(output, input, scale, azp=None):
    if azp is None:
        opcheck(torch.ops._C.static_scaled_int8_quant, (output, input, scale, None))
    else:
        opcheck(torch.ops._C.static_scaled_int8_quant, (output, input, scale, azp))
```
**EN:** This helper function implements the shared logic for opcheck int8 quant static. it also validates that the custom operator entry point is wired correctly.
**CN:** 该辅助函数实现了 opcheck int8 quant static 所需的共享逻辑。 它还会校验自定义算子入口是否正确接线。

### Function `opcheck_int8_quant_dynamic` (lines 26-38)
```python
def opcheck_int8_quant_dynamic(output, input, symmetric=True):
    scale = torch.empty(
        (input.numel() // input.shape[-1], 1), device=input.device, dtype=torch.float32
    )
    if symmetric:
        opcheck(torch.ops._C.dynamic_scaled_int8_quant, (output, input, scale, None))
    else:
        azp = torch.empty(
            (input.numel() // input.shape[-1], 1),
            device=input.device,
            dtype=torch.int32,
        )
        opcheck(torch.ops._C.dynamic_scaled_int8_quant, (output, input, scale, azp))
```
**EN:** This helper function implements the shared logic for opcheck int8 quant dynamic. it also validates that the custom operator entry point is wired correctly.
**CN:** 该辅助函数实现了 opcheck int8 quant dynamic 所需的共享逻辑。 它还会校验自定义算子入口是否正确接线。

### Function `test_dynamic_scaled_int8_quant` (lines 41-62)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@torch.inference_mode()
def test_dynamic_scaled_int8_quant(
    num_tokens: int, hidden_size: int, dtype: torch.dtype, seed: int
) -> None:
    set_random_seed(seed)

    x = torch.rand(num_tokens, hidden_size, dtype=dtype, device="cuda") * 1000

    # reference
    ref_out, ref_scales = ref_dynamic_per_token_quant(x, torch.int8)
    # kernel
    ops_out, ops_scales, _ = scaled_int8_quant(x)

    torch.testing.assert_close(ops_scales, ref_scales)
    # big atol to account for rounding errors
    torch.testing.assert_close(ops_out, ref_out, atol=1, rtol=0.0)

    opcheck_int8_quant_dynamic(ops_out, x)
```
**EN:** This pytest case verifies dynamic scaled int8 quant. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, dtype, seed. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 dynamic scaled int8 quant 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、dtype、seed 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_dynamic_scaled_int8_azp_quant` (lines 65-102)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@torch.inference_mode()
def test_dynamic_scaled_int8_azp_quant(
    num_tokens: int, hidden_size: int, dtype: torch.dtype, seed: int
) -> None:
    set_random_seed(seed)
    int8_traits = torch.iinfo(torch.int8)

    x = torch.rand(num_tokens, hidden_size, dtype=dtype, device="cuda") * 1000 - 300

    x_token_max, _ = x.to(dtype=torch.float32).max(dim=1, keepdim=True)
    x_token_min, _ = x.to(dtype=torch.float32).min(dim=1, keepdim=True)

    # calculate scale and azp, and adjust the range
    scales = (x_token_max - x_token_min) / torch.tensor(255.0)
    azps = torch.round(torch.tensor(-128.0) - x_token_min / scales).to(torch.int32)

    torch_out = (
        ((x / scales).round() + azps)
        .clamp(int8_traits.min, int8_traits.max)
        .to(torch.int8)
    )
    assert torch_out.min() >= int8_traits.min and torch_out.max() <= int8_traits.max

    ops_out, scales_out, azp_out = scaled_int8_quant(x, symmetric=False)

    if not torch.allclose(scales_out, scales):
        print(torch.argmax(torch.abs(scales_out - scales)))
    torch.testing.assert_close(scales_out, scales)
    # big atol to account for rounding errors
    torch.testing.assert_close(azp_out, azps, atol=1, rtol=0.0)
    # if AZP is off by 1, after rounding-to-even, the output may be off by 2
    torch.testing.assert_close(ops_out, torch_out, atol=2, rtol=0.0)

    opcheck_int8_quant_dynamic(ops_out, x, False)
```
**EN:** This pytest case verifies dynamic scaled int8 azp quant. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, dtype, seed. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 dynamic scaled int8 azp quant 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、dtype、seed 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_static_scaled_int8_quant` (lines 105-129)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("scale", SCALE)
@torch.inference_mode()
def test_static_scaled_int8_quant(
    num_tokens: int, hidden_size: int, dtype: torch.dtype, seed: int, scale: float
) -> None:
    set_random_seed(seed)
    int8_traits = torch.iinfo(torch.int8)

    x = torch.rand(num_tokens, hidden_size, dtype=dtype, device="cuda") * 1000
    scale_arg = torch.tensor([scale], dtype=torch.float32, device="cuda")

    out1 = (
        (x / scale_arg).round().clamp(int8_traits.min, int8_traits.max).to(torch.int8)
    )
    out2, scale2, _ = scaled_int8_quant(x, scale_arg)
    assert scale2 is scale_arg

    # big atol to account for rounding errors
    torch.testing.assert_close(out1, out2, atol=1, rtol=0.0)

    opcheck_int8_quant_static(out2, x, scale_arg)
```
**EN:** This pytest case verifies static scaled int8 quant. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, dtype, seed. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 static scaled int8 quant 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、dtype、seed 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_static_scaled_int8_azp_quant` (lines 132-167)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("scale", SCALE)
@pytest.mark.parametrize("azp", [-255, 54])
@torch.inference_mode()
def test_static_scaled_int8_azp_quant(
    num_tokens: int,
    hidden_size: int,
    dtype: torch.dtype,
    seed: int,
    scale: float,
    azp: int,
) -> None:
    set_random_seed(seed)
    int8_traits = torch.iinfo(torch.int8)

    x = torch.rand(num_tokens, hidden_size, dtype=dtype, device="cuda") * 1000 - 300

    out1 = (
        ((x / scale).round() + azp)
        .clamp(int8_traits.min, int8_traits.max)
        .to(torch.int8)
    )
    scale_arg = torch.tensor([scale], dtype=torch.float32, device="cuda")
    azp_arg = torch.tensor([azp], dtype=torch.int32, device="cuda")

    out2, scale2, azp2 = scaled_int8_quant(x, scale_arg, azp_arg, symmetric=False)
    assert scale2 is scale_arg
    assert azp2 is azp_arg

    # big atol to account for rounding errors
    torch.testing.assert_close(out1, out2, atol=1, rtol=0.0)

    opcheck_int8_quant_static(out2, x, scale_arg, azp_arg)
```
**EN:** This pytest case verifies static scaled int8 azp quant. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, dtype, seed. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 static scaled int8 azp quant 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、dtype、seed 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_static_scaled_int8_azp_quant_saturating_cast` (lines 170-195)
```python
@pytest.mark.parametrize("is_max", [True, False])
@torch.inference_mode()
def test_static_scaled_int8_azp_quant_saturating_cast(is_max: bool) -> None:
    # Test that the saturating cast works correctly for values near i32 max/min

    from numpy import inf, nextafter

    int32_traits = torch.iinfo(torch.int32)
    val = float(int32_traits.max if is_max else int32_traits.min)

    x_vals = [[nextafter(val, inf), val + 1, val, val - 1, nextafter(val, -inf)]]
    x = torch.tensor(x_vals, dtype=torch.float32, device="cuda")

    # The calculation in the kernel is: cast<int8>(cast<int32>(x / scale) + azp)
    # where cast<T> is a saturating cast to type T.
    # Scale is set to 1.0 so that the input values are the ones that are cast.
    # AZP is set to 0 to make sure the int8 saturating cast is tested as well.
    scale = torch.scalar_tensor(1.0, dtype=torch.float32, device="cuda")
    azp = torch.scalar_tensor(0, dtype=torch.int32, device="cuda")

    int8_traits = torch.iinfo(torch.int8)
    val_i8 = int8_traits.max if is_max else int8_traits.min
    expected = torch.full((1, 5), val_i8, dtype=torch.int8, device="cuda")

    out, _, _ = scaled_int8_quant(x, scale, azp, symmetric=False)
    torch.testing.assert_close(expected, out, atol=0, rtol=0)
```
**EN:** This pytest case verifies static scaled int8 azp quant saturating cast. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as is_max. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 static scaled int8 azp quant saturating cast 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 is_max 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `tests.kernels.quant_utils -> ref_dynamic_per_token_quant`
- `tests.kernels.utils -> opcheck`
- `vllm._custom_ops -> scaled_int8_quant`
- `vllm.utils.torch_utils -> set_random_seed`
