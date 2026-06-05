# test_nvfp4_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_nvfp4_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_nvfp4_quant, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_nvfp4_quant 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-9)
```python
import pytest
import torch

from vllm import _custom_ops as ops
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm, vllm.platforms, vllm.scalar_type, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm、vllm.platforms、vllm.scalar_type、vllm.utils.torch_utils。

### Top-level block starting at line 11 (lines 11-15)
```python
if not current_platform.has_device_capability(100):
    pytest.skip(
        reason="Nvfp4 Requires compute capability of 10 or above.",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 17-70)
```python
DTYPES = [torch.float16, torch.bfloat16]
SHAPES = [(128, 64), (128, 128), (256, 64), (256, 128)]
PAD_SHAPES = [
    (90, 64),
    (150, 64),
    (128, 48),
    (128, 80),
    (150, 80),
    (90, 48),
    (90, 128),
    (150, 128),
    (150, 48),
    (90, 80),
    (128, 512),
    (128, 1024),
    (128, 2048),
    (64, 7168),
    (64, 7152),
    (32, 14336),
]
SEEDS = [42]
CUDA_DEVICES = ["cuda:0"]

FLOAT4_E2M1_MAX = scalar_types.float4_e2m1f.max()
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max

# E2M1 to float
# 0111 -> 6
# 0110 -> 4
# 0101 -> 3
# 0100 -> 2
# 0011 -> 1.5
# 0010 -> 1
# 0001 -> 0.5
# 0000 -> 0
E2M1_TO_FLOAT32 = [
    0.0,
    0.5,
    1.0,
    1.5,
    2.0,
    3.0,
    4.0,
    6.0,
    0.0,
    -0.5,
    -1.0,
    -1.5,
    -2.0,
    -3.0,
    -4.0,
    -6.0,
]
BLOCK_SIZE = 16
```
**EN:** This block centralizes shared constants and parameter grids, including DTYPES, SHAPES, PAD_SHAPES, SEEDS, CUDA_DEVICES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DTYPES、SHAPES、PAD_SHAPES、SEEDS、CUDA_DEVICES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `cast_from_fp4` (lines 73-80)
```python
def cast_from_fp4(x, m, n):
    # The fp4 values are packed in uint8 as [v_1st | v_2nd]
    v_2nd = x & 0xF
    v_1st = (x >> 4) & 0xF
    c = torch.stack((v_2nd, v_1st), dim=-1)
    out = torch.tensor([E2M1_TO_FLOAT32[x] for x in c.flatten()])
    out = out.reshape(m, n).to(torch.float32)
    return out
```
**EN:** This helper function implements the shared logic for cast from fp4. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 cast from fp4 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `cast_to_fp4` (lines 83-94)
```python
def cast_to_fp4(x):
    sign = torch.sign(x)
    x = torch.abs(x)
    x[(x >= 0.0) & (x <= 0.25)] = 0.0
    x[(x > 0.25) & (x < 0.75)] = 0.5
    x[(x >= 0.75) & (x <= 1.25)] = 1.0
    x[(x > 1.25) & (x < 1.75)] = 1.5
    x[(x >= 1.75) & (x <= 2.5)] = 2.0
    x[(x > 2.5) & (x < 3.5)] = 3.0
    x[(x >= 3.5) & (x <= 5.0)] = 4.0
    x[x > 5.0] = 6.0
    return x * sign
```
**EN:** This helper function implements the shared logic for cast to fp4. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 cast to fp4 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `get_reciprocal` (lines 97-103)
```python
def get_reciprocal(x):
    if isinstance(x, torch.Tensor):
        return torch.where(x == 0, torch.tensor(0.0, dtype=x.dtype), 1.0 / x)
    elif isinstance(x, (float, int)):
        return 0.0 if x == 0 else 1.0 / x
    else:
        raise TypeError("Input must be a float, int, or a torch.Tensor.")
```
**EN:** This helper function implements the shared logic for reciprocal. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 reciprocal 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_nvfp4_quant` (lines 106-118)
```python
def ref_nvfp4_quant(x, global_scale):
    assert global_scale.dtype == torch.float32
    assert x.ndim == 2
    m, n = x.shape
    x = torch.reshape(x, (m, n // BLOCK_SIZE, BLOCK_SIZE))
    vec_max = torch.max(torch.abs(x), dim=-1, keepdim=True)[0].to(torch.float32)
    scale = global_scale * (vec_max * get_reciprocal(FLOAT4_E2M1_MAX))
    scale = scale.to(torch.float8_e4m3fn).to(torch.float32)
    output_scale = get_reciprocal(scale * get_reciprocal(global_scale))

    scaled_x = x.to(torch.float32) * output_scale
    clipped_x = torch.clamp(scaled_x, -6.0, 6.0).reshape(m, n)
    return cast_to_fp4(clipped_x), scale.squeeze(-1)
```
**EN:** This helper acts as a reference implementation for nvfp4 quant. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数为 nvfp4 quant 提供参考实现。 结尾处的断言会固定预期行为或计算图形态。

### Function `recover_swizzled_scales` (lines 121-130)
```python
def recover_swizzled_scales(scale, m, n):
    round_up = lambda x, y: (x + y - 1) // y * y
    rounded_m = round_up(m, 128)
    scale_n = n // BLOCK_SIZE
    rounded_n = round_up(scale_n, 4)
    # Recover the swizzled scaling factor to linear layout
    tmp = torch.reshape(scale, (1, rounded_m // 128, rounded_n // 4, 32, 4, 4))
    tmp = torch.permute(tmp, (0, 1, 4, 3, 2, 5))
    result = torch.reshape(tmp, (rounded_m, rounded_n)).to(torch.float32)
    return result[:m, :scale_n]
```
**EN:** This helper function implements the shared logic for recover swizzled scales. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 recover swizzled scales 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_quantize_to_fp4` (lines 133-159)
```python
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("shape", SHAPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@torch.inference_mode()
def test_quantize_to_fp4(
    dtype: torch.dtype,
    shape: tuple[int, int],
    seed: int,
    device: str,
) -> None:
    set_random_seed(seed)
    torch.set_default_device(device)

    m, n = shape

    x = torch.randn((m, n), dtype=dtype)
    tensor_amax = torch.abs(x).max().to(torch.float32)
    global_scale = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / tensor_amax
    out_ref, scale_ref = ref_nvfp4_quant(x, global_scale)

    out, out_scale = ops.scaled_fp4_quant(x, global_scale)
    scale_ans = recover_swizzled_scales(out_scale, m, n)
    out_ans = cast_from_fp4(out, m, n)

    torch.testing.assert_close(out_ans, out_ref)
    torch.testing.assert_close(scale_ans, scale_ref)
```
**EN:** This pytest case verifies quantize to fp4. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dtype, shape, seed, device. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 quantize to fp4 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 dtype、shape、seed、device 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_python_util_matches_cpp_allocation` (lines 162-205)
```python
@pytest.mark.parametrize(
    "shape",
    [(32, 4096), (128, 4096), (1, 64), (127, 1024), (256, 16384)],
)
@pytest.mark.parametrize("is_sf_swizzled_layout", [True, False])
@torch.inference_mode()
def test_python_util_matches_cpp_allocation(
    shape: tuple[int, int],
    is_sf_swizzled_layout: bool,
) -> None:
    """
    Verify that the Python utility (create_fp4_output_tensors) allocates
    tensors with the same shapes and dtypes as the C++ functional variant
    (scaled_fp4_quant_func).
    """
    from vllm._custom_ops import create_fp4_output_tensors

    torch.set_default_device("cuda:0")
    m, n = shape
    input_tensor = torch.randn((m, n), dtype=torch.bfloat16)
    input_scale = torch.tensor([1.0], dtype=torch.float32, device="cuda:0")

    # C++ functional variant allocates internally
    cpp_out, cpp_scale = torch.ops._C.scaled_fp4_quant(
        input_tensor, input_scale, is_sf_swizzled_layout
    )

    # Python utility
    py_out, py_scale = create_fp4_output_tensors(
        m, n, torch.device("cuda:0"), is_sf_swizzled_layout
    )

    assert py_out.shape == cpp_out.shape, (
        f"Output shape mismatch: Python {py_out.shape} vs C++ {cpp_out.shape}"
    )
    assert py_out.dtype == cpp_out.dtype, (
        f"Output dtype mismatch: Python {py_out.dtype} vs C++ {cpp_out.dtype}"
    )
    assert py_scale.shape == cpp_scale.shape, (
        f"Scale shape mismatch: Python {py_scale.shape} vs C++ {cpp_scale.shape}"
    )
    assert py_scale.dtype == cpp_scale.dtype, (
        f"Scale dtype mismatch: Python {py_scale.dtype} vs C++ {cpp_scale.dtype}"
    )
```
**EN:** This pytest case verifies python util matches cpp allocation. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as shape, is_sf_swizzled_layout. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 python util matches cpp allocation 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 shape、is_sf_swizzled_layout 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_quantize_to_fp4_padded` (lines 208-227)
```python
@pytest.mark.parametrize("pad_shape", PAD_SHAPES)
@torch.inference_mode()
def test_quantize_to_fp4_padded(pad_shape: tuple[int, int]) -> None:
    dtype = torch.float16
    set_random_seed(42)
    torch.set_default_device("cuda:0")

    m, n = pad_shape

    x = torch.randn((m, n), dtype=dtype)

    tensor_amax = torch.abs(x).max().to(torch.float32)
    global_scale = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / tensor_amax
    out_ref, scale_ref = ref_nvfp4_quant(x, global_scale)

    out, out_scale = ops.scaled_fp4_quant(x, global_scale)
    scale_ans = recover_swizzled_scales(out_scale, m, n)
    out_ans = cast_from_fp4(out, m, n)
    torch.testing.assert_close(out_ans, out_ref)
    torch.testing.assert_close(scale_ans, scale_ref)
```
**EN:** This pytest case verifies quantize to fp4 padded. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as pad_shape. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 quantize to fp4 padded 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 pad_shape 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_quantize_to_fp4_padded_no_sf_swizzled` (lines 230-249)
```python
@pytest.mark.parametrize("pad_shape", PAD_SHAPES)
@torch.inference_mode()
def test_quantize_to_fp4_padded_no_sf_swizzled(pad_shape: tuple[int, int]) -> None:
    dtype = torch.float16
    set_random_seed(42)
    torch.set_default_device("cuda:0")

    m, n = pad_shape

    x = torch.randn((m, n), dtype=dtype)

    tensor_amax = torch.abs(x).max().to(torch.float32)
    global_scale = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / tensor_amax
    out_ref, scale_ref = ref_nvfp4_quant(x, global_scale)

    out, out_scale = ops.scaled_fp4_quant(x, global_scale, is_sf_swizzled_layout=False)
    scale_ans = out_scale.to(torch.float32)
    out_ans = cast_from_fp4(out, m, n)
    torch.testing.assert_close(out_ans, out_ref)
    torch.testing.assert_close(scale_ans, scale_ref)
```
**EN:** This pytest case verifies quantize to fp4 padded no sf swizzled. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as pad_shape. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 quantize to fp4 padded no sf swizzled 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 pad_shape 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm -> _custom_ops`
- `vllm.platforms -> current_platform`
- `vllm.scalar_type -> scalar_types`
- `vllm.utils.torch_utils -> set_random_seed`
