# test_silu_mul_fp8_quant_deep_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_silu_mul_fp8_quant_deep_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_silu_mul_fp8_quant_deep_gemm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_silu_mul_fp8_quant_deep_gemm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-23)
```python
import random

import pytest
import torch

from vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe import (
    persistent_masked_m_silu_mul_quant,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    get_fp8_min_max,
)
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import (
    DeepGemmQuantScaleFMT,
    has_deep_gemm,
    transform_sf_into_required_layout,
)
from vllm.utils.math_utils import cdiv, round_up
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as random, pytest, torch; and vLLM components like vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe, vllm.model_executor.layers.quantization.utils.quant_utils, vllm.platforms, vllm.utils.deep_gemm.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 random、pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe、vllm.model_executor.layers.quantization.utils.quant_utils、vllm.platforms、vllm.utils.deep_gemm。

### Constants and module state (lines 25-52)
```python
fp8_dtype = current_platform.fp8_dtype()

CASES = [
    (1, 1, 128, fp8_dtype),
    (1, 4, 128 * 1, fp8_dtype),
    (2, 4, 128 * 2, fp8_dtype),
    (1, 4, 128 * 3, fp8_dtype),
    (8, 16, 128 * 4, fp8_dtype),
    (8, 16, 128 * 5, fp8_dtype),
    (8, 16, 128 * 6, fp8_dtype),
    (8, 16, 128 * 7, fp8_dtype),
    (8, 16, 128 * 8, fp8_dtype),
    (8, 16, 128 * 9, fp8_dtype),
    (8, 64, 7168, fp8_dtype),
    (8, 128, 128 * 33, fp8_dtype),
    (1, 4, 128 * 10, fp8_dtype),
    (8, 128, 7168, fp8_dtype),
    (8, 512, 7168, fp8_dtype),
    (8, 1024, 7168, fp8_dtype),
    (17, 31, 768, fp8_dtype),
    (32, 64, 256, fp8_dtype),
    (256, 8, 7168, fp8_dtype),
    (256, 32, 7168, fp8_dtype),
    (256, 64, 7168, fp8_dtype),
    # Only add a few fnuz tests to help with long CI times.
    (8, 512, 7168, torch.float8_e4m3fnuz),
    (8, 1024, 7168, torch.float8_e4m3fnuz),
]
```
**EN:** This block centralizes shared constants and parameter grids, including CASES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 CASES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `as_uint8` (lines 55-58)
```python
def as_uint8(x) -> torch.Tensor:
    return (
        torch.empty(x.shape, dtype=x.dtype, device=x.device).copy_(x).view(torch.uint8)
    )
```
**EN:** This helper function implements the shared logic for as uint8. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 as uint8 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `silu` (lines 61-68)
```python
def silu(x: torch.Tensor) -> torch.Tensor:
    x_f32 = x.to(torch.float32)
    act_f32 = x_f32 / (1.0 + torch.exp(-x_f32))
    if current_platform.is_cuda():
        # C++ kernel returns bf16
        return act_f32.to(torch.bfloat16)
    # Triton fallback stays in f32
    return act_f32
```
**EN:** This helper function implements the shared logic for silu. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 silu 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `do_quant` (lines 71-117)
```python
def do_quant(x: torch.Tensor, group_size: int, ceil_ue8m0: bool):
    fp8_min_val, fp8_max_val = get_fp8_min_max()
    eps_bf16 = torch.tensor([1e-10], device=x.device, dtype=torch.bfloat16)
    one_bf16 = torch.tensor([1.0], device=x.device, dtype=torch.bfloat16)
    fp8_max_bf16 = torch.tensor([fp8_max_val], device=x.device, dtype=torch.bfloat16)
    fp8_min_bf16 = torch.tensor([fp8_min_val], device=x.device, dtype=torch.bfloat16)
    fp8_max_inv = one_bf16 / fp8_max_bf16
    assert fp8_max_inv.dtype == torch.bfloat16

    assert x.size(-1) % group_size == 0
    num_groups = x.numel() // group_size
    x_og_shape = x.shape

    if current_platform.is_cuda():
        # C++ kernel computes entirely in bf16
        x = x.to(torch.bfloat16)
        x = x.view((-1, group_size))
        amax = x.abs().amax(dim=1).clamp(min=eps_bf16)
        assert amax.dtype == torch.bfloat16
        s = amax * fp8_max_inv

        if ceil_ue8m0:
            s = torch.exp2(
                torch.ceil(torch.log2(s).to(torch.bfloat16)).to(torch.bfloat16)
            ).to(torch.bfloat16)

        inv_s = one_bf16 / s
        inv_s = inv_s.view((num_groups, 1))
        xq = torch.clamp(
            x * inv_s, min=fp8_min_bf16.item(), max=fp8_max_bf16.item()
        ).to(fp8_dtype)
    else:
        # Triton fallback computes in f32. Use multiply-by-reciprocal
        # to match Triton's constexpr evaluation of 1.0/fp8_max.
        fp8_min_f, fp8_max_f = get_fp8_min_max()

        x = x.to(torch.float32).view((-1, group_size))
        amax = x.abs().amax(dim=1).clamp(min=1e-10)
        s = amax * (1.0 / fp8_max_f)
        if ceil_ue8m0:
            s = torch.exp2(torch.ceil(torch.log2(s)))
        inv_s = (1.0 / s).view((num_groups, 1))
        xq = torch.clamp(x * inv_s, min=fp8_min_f, max=fp8_max_f).to(fp8_dtype)

    xq = xq.view(x_og_shape)
    xs = s.view((-1, xq.size(-1) // group_size))
    return xq, xs
```
**EN:** This helper function implements the shared logic for do quant. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 do quant 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `silu_mul_quant` (lines 120-135)
```python
def silu_mul_quant(
    gate: torch.Tensor, up: torch.Tensor, group_size: int, ceil_ue8m0: bool
) -> tuple[torch.Tensor, torch.Tensor]:
    assert gate.size(-1) % group_size == 0
    assert up.size(-1) % group_size == 0

    assert gate.dtype == torch.bfloat16
    assert up.dtype == torch.bfloat16

    act = silu(gate)

    # act & mul
    a_m = act * up

    q, s = do_quant(a_m, group_size, ceil_ue8m0)
    return q, s
```
**EN:** This helper function implements the shared logic for silu mul quant. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 silu mul quant 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `pack_scales` (lines 138-154)
```python
def pack_scales(x: torch.Tensor, tokens_per_expert: torch.Tensor) -> torch.Tensor:
    """
    pack float32 scales into a int32 tensor
    """
    assert x.dtype == torch.float32
    E, T, G = x.size()

    # Add i32_padding here so we can view it as a i32 tensor later on.
    i32_padding = round_up(G, 4) - G
    ref_s_i8 = torch.empty((E, T, G + i32_padding), dtype=torch.uint8, device="cuda")
    for e in range(E):
        nt = tokens_per_expert[e].item()
        ref_s_i8[e, :nt, :G] = x[e, :nt].view(torch.int32) >> 23

    ref_s_i32 = ref_s_i8.view(torch.int32)

    return ref_s_i32
```
**EN:** This helper function implements the shared logic for pack scales. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 pack scales 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `ref_with_scale_fmt` (lines 157-197)
```python
def ref_with_scale_fmt(
    E: int,
    T: int,
    H: int,
    group_size: int,
    tokens_per_expert: torch.Tensor,
    gate: torch.Tensor,
    up: torch.Tensor,
    scale_fmt: DeepGemmQuantScaleFMT,
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    The precision types of the operations triggered by this function
    match closely with the kernel implementation so we compare more
    accurately.
    """
    scale_dtype = (
        torch.int32 if scale_fmt == DeepGemmQuantScaleFMT.UE8M0 else torch.float32
    )
    ceil_ue8m0 = scale_fmt in [
        DeepGemmQuantScaleFMT.UE8M0,
        DeepGemmQuantScaleFMT.FLOAT32_CEIL_UE8M0,
    ]

    ref_q = torch.empty((E, T, H), dtype=fp8_dtype, device="cuda")
    ref_s_f32 = torch.empty(
        (E, T, cdiv(H, group_size)), dtype=torch.float32, device="cuda"
    )

    for e in range(E):
        nt = tokens_per_expert[e].item()
        if nt == 0:
            continue
        ref_q[e, :nt], ref_s_f32[e, :nt] = silu_mul_quant(
            gate[e, :nt], up[e, :nt], group_size, ceil_ue8m0=ceil_ue8m0
        )

    if scale_dtype == torch.float32:
        return ref_q, ref_s_f32

    assert scale_dtype == torch.int32
    return ref_q, pack_scales(ref_s_f32, tokens_per_expert)
```
**EN:** This helper acts as a reference implementation for with scale fmt. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数为 with scale fmt 提供参考实现。 结尾处的断言会固定预期行为或计算图形态。

### Function `token_random` (lines 200-210)
```python
def token_random(E, T, H2, tokens_per_expert):
    """
    Initialize each token in a random range so we test a range of
    scale values.
    """
    y = torch.empty((E, T, H2), dtype=torch.bfloat16, device="cuda")
    for e in range(E):
        for t in range(tokens_per_expert[e].item()):
            exp = random.choice(range(1, 20))
            y[e, t].uniform_(-(2**exp), 2**exp)
    return y
```
**EN:** This helper function implements the shared logic for token random. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 token random 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_silu_mul_fp8_quant_deep_gemm` (lines 213-321)
```python
@pytest.mark.parametrize("E,T,H,fp8_type", CASES)
@torch.inference_mode()
def test_silu_mul_fp8_quant_deep_gemm(E: int, T: int, H: int, fp8_type: torch.dtype):
    group_size = 128
    set_random_seed(42)

    tokens_per_expert = torch.randint(
        low=0,
        high=T,
        size=(E,),
        dtype=torch.int32,
        device="cuda",
    )

    # Input tensor of shape (E, T, 2*H)
    y = token_random(E, T, 2 * H, tokens_per_expert)

    gate = y[..., :H].to(torch.bfloat16)
    up = y[..., H:].to(torch.bfloat16)

    scale_fmts = [
        DeepGemmQuantScaleFMT.FLOAT32,
        DeepGemmQuantScaleFMT.FLOAT32_CEIL_UE8M0,
    ]
    # UE8M0 (int32 packed) scales require the C++ kernel which is
    # not available on ROCm (#ifndef USE_ROCM).
    # https://github.com/ROCm/aiter/issues/2420
    if current_platform.is_cuda():
        scale_fmts.append(DeepGemmQuantScaleFMT.UE8M0)

    # Run the SiLU V2 kernel
    for scale_fmt in scale_fmts:
        y_q, y_s = persistent_masked_m_silu_mul_quant(
            y,
            tokens_per_expert,
            group_size=group_size,
            quant_scale_fmt=scale_fmt,
        )

        ref_y_q, ref_y_s = ref_with_scale_fmt(
# ... excerpt ...
            torch.int32 if scale_fmt == DeepGemmQuantScaleFMT.UE8M0 else torch.float32
        )
        assert y_s.dtype == expected_scale_dtype
        assert ref_y_s.dtype == expected_scale_dtype

        for e in range(E):
            nt = tokens_per_expert[e].item()

            if current_platform.is_rocm():
                # On ROCm the Triton fallback kernel uses f32 math
                # intrinsics (tl.exp) that may differ from PyTorch's
                # torch.exp by 1 ULP.  At FP8 quantization
                # boundaries this can flip one representable value.
                # Allow 1 FP8 quantum of tolerance.
                torch.testing.assert_close(
                    y_q[e, :nt].to(torch.float32),
                    ref_y_q[e, :nt].to(torch.float32),
                    atol=32.0,
                    rtol=0.2,
                )
            else:
                torch.testing.assert_close(
                    y_q[e, :nt].to(torch.float32),
                    ref_y_q[e, :nt].to(torch.float32),
                )

            if scale_fmt == DeepGemmQuantScaleFMT.UE8M0:
                G = H // group_size
                y_s_sliced = as_uint8(y_s[e])
                ref_s_sliced = as_uint8(ref_y_s[e])
                torch.testing.assert_close(y_s_sliced[:nt, :G], ref_s_sliced[:nt, :G])
                if dg_scales is not None:
                    dg_sliced = as_uint8(dg_scales[e])
                    torch.testing.assert_close(y_s_sliced[:nt, :G], dg_sliced[:nt, :G])
            else:
                torch.testing.assert_close(
                    y_s[e, :nt],
                    ref_y_s[e, :nt],
                )
```
**EN:** This pytest case verifies silu mul FP8 quant deep gemm. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as E, T, H, fp8_type. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 silu mul FP8 quant deep gemm 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 E、T、H、fp8_type 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `random`
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe -> persistent_masked_m_silu_mul_quant`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> get_fp8_min_max`
- `vllm.platforms -> current_platform`
- `vllm.utils.deep_gemm -> DeepGemmQuantScaleFMT, has_deep_gemm, transform_sf_into_required_layout`
- `vllm.utils.math_utils -> cdiv, round_up`
- `vllm.utils.torch_utils -> set_random_seed`
