# test_layernorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/core/test_layernorm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / core / test_layernorm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / core / test_layernorm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-11)
```python
import pytest
import torch

from tests.kernels.quant_utils import FP8_DTYPE
from tests.kernels.utils import opcheck
from vllm.model_executor.layers.layernorm import GemmaRMSNorm, RMSNorm
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.quant_utils, tests.kernels.utils; and vLLM components like vllm.model_executor.layers.layernorm, vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.quant_utils、tests.kernels.utils；vLLM 内部组件，例如 vllm.model_executor.layers.layernorm、vllm.platforms、vllm.utils.torch_utils。

### Top-level block starting at line 13 (lines 13-18)
```python
if current_platform.is_rocm():
    from vllm.platforms.rocm import on_gfx90a

    on_mi250 = on_gfx90a()
else:
    on_mi250 = False
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 20-27)
```python
DTYPES = [torch.half, torch.bfloat16, torch.float]
NUM_TOKENS = [7, 83, 4096]  # Arbitrary values for testing
HIDDEN_SIZES = [8, 768, 769, 5120, 5125, 8192]  # Arbitrary values for testing
ADD_RESIDUAL = [False, True] if not on_mi250 else [True]
SEEDS = [0]
CUDA_DEVICES = [
    f"cuda:{i}" for i in range(1 if torch.accelerator.device_count() == 1 else 2)
]
```
**EN:** This block centralizes shared constants and parameter grids, including DTYPES, NUM_TOKENS, HIDDEN_SIZES, ADD_RESIDUAL, SEEDS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DTYPES、NUM_TOKENS、HIDDEN_SIZES、ADD_RESIDUAL、SEEDS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_rms_norm` (lines 30-81)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("add_residual", ADD_RESIDUAL)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("strided_input", [False, True])
@torch.inference_mode()
def test_rms_norm(
    default_vllm_config,
    num_tokens: int,
    hidden_size: int,
    add_residual: bool,
    dtype: torch.dtype,
    seed: int,
    device: str,
    strided_input: bool,
) -> None:
    set_random_seed(seed)
    torch.set_default_device(device)
    layer = RMSNorm(hidden_size).to(dtype=dtype)
    layer.weight.data.normal_(mean=1.0, std=0.1)
    scale = 1 / (2 * hidden_size)
    last_dim = 2 * hidden_size if strided_input else hidden_size
    x = torch.randn(num_tokens, last_dim, dtype=dtype)
    x = x[..., :hidden_size]
    assert x.is_contiguous() != strided_input
    x *= scale
    residual = torch.randn_like(x) * scale if add_residual else None

    # NOTE(woosuk): The reference implementation should be executed first
    # because the custom kernel is in-place.
    ref_out = layer.forward_native(x, residual)
    out = layer(x, residual)
    # NOTE(woosuk): LayerNorm operators (including RMS) typically have larger
    # numerical errors than other operators because they involve reductions.
    # Therefore, we use a larger tolerance.
    if add_residual:
        torch.testing.assert_close(out[0], ref_out[0], atol=1e-2, rtol=1e-2)
        torch.testing.assert_close(out[1], ref_out[1], atol=1e-2, rtol=1e-2)
    else:
        torch.testing.assert_close(out, ref_out, atol=1e-2, rtol=1e-2)

    if residual is not None:
        opcheck(
            torch.ops._C.fused_add_rms_norm,
            (x, residual, layer.weight.data, layer.variance_epsilon),
        )
    else:
        opcheck(
            torch.ops._C.rms_norm, (out, x, layer.weight.data, layer.variance_epsilon)
        )
```
**EN:** This pytest case verifies rms norm. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, num_tokens, hidden_size, add_residual. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 rms norm 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、num_tokens、hidden_size、add_residual 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

### Function `test_fused_rms_norm_quant` (lines 84-164)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("add_residual", ADD_RESIDUAL)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("quant_scale", [0.01, 1.0, 10.0])
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("strided_input", [False, True])
def test_fused_rms_norm_quant(
    num_tokens: int,
    hidden_size: int,
    add_residual: bool,
    dtype: torch.dtype,
    quant_scale: float,
    seed: int,
    device: str,
    strided_input: bool,
) -> None:
    set_random_seed(seed)
    torch.set_default_device(device)

    weight = torch.empty(hidden_size, dtype=dtype).normal_(mean=1.0, std=0.1)
    scale = 1 / (2 * hidden_size)
    last_dim = 2 * hidden_size if strided_input else hidden_size
    x_base = torch.randn(num_tokens, last_dim, dtype=dtype)
    x = x_base[..., :hidden_size]
    assert x.is_contiguous() != strided_input

    x *= scale
    if add_residual:
        residual = torch.randn_like(x) * scale
        residual_fused = residual.clone()
    else:
        residual = residual_fused = None

    out_norm = torch.empty_like(x)
    out_quant = torch.empty_like(x, dtype=FP8_DTYPE)
    out_quant_fused = torch.empty_like(out_quant)

    quant_scale_t = torch.tensor(quant_scale, dtype=torch.float32)
# ... excerpt ...
        torch.ops._C.fused_add_rms_norm_static_fp8_quant(
            out_quant_fused, x, residual_fused, weight, quant_scale_t, 1e-6
        )

        # Unfused kernel is in-place so it goes second
        # Also use a separate clone of x to avoid modifying the input
        x_unfused_base = x_base.clone()
        x_unfused = x_unfused_base[..., :hidden_size]
        assert x_unfused.is_contiguous() != strided_input
        torch.ops._C.fused_add_rms_norm(x_unfused, residual, weight, 1e-6)
        torch.ops._C.static_scaled_fp8_quant(
            out_quant, x_unfused.contiguous(), quant_scale_t
        )

        torch.accelerator.synchronize()
        torch.testing.assert_close(residual_fused, residual, atol=1e-2, rtol=1e-2)
        opcheck(
            torch.ops._C.fused_add_rms_norm_static_fp8_quant,
            (out_quant_fused, x, residual_fused, weight, quant_scale_t, 1e-6),
        )
    else:
        torch.ops._C.rms_norm_static_fp8_quant(
            out_quant_fused, x, weight, quant_scale_t, 1e-6
        )

        torch.ops._C.rms_norm(out_norm, x, weight, 1e-6)
        torch.ops._C.static_scaled_fp8_quant(out_quant, out_norm, quant_scale_t)

        opcheck(
            torch.ops._C.rms_norm_static_fp8_quant,
            (out_quant_fused, x, weight, quant_scale_t, 1e-6),
        )

    torch.testing.assert_close(
        out_quant.to(dtype=torch.float32),
        out_quant_fused.to(dtype=torch.float32),
        atol=1e-3,
        rtol=1e-3,
    )
```
**EN:** This pytest case verifies fused rms norm quant. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, add_residual, dtype. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 fused rms norm quant 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、add_residual、dtype 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_gemma_rms_norm_mixed_input_weight_dtype` (lines 167-192)
```python
@torch.inference_mode()
def test_gemma_rms_norm_mixed_input_weight_dtype(default_vllm_config) -> None:
    if not torch.cuda.is_available():
        pytest.skip("CUDA required")

    device = CUDA_DEVICES[0]
    torch.set_default_device(device)

    num_tokens, hidden_size = 32, 1024
    x = torch.randn(num_tokens, hidden_size, dtype=torch.bfloat16, device=device)
    layer = GemmaRMSNorm(hidden_size, eps=1e-6).to(device=device)
    layer.weight.data.normal_(mean=0.0, std=0.1)

    # Gemma uses fp32 weight parameter while activations can be bf16.
    assert layer.weight.dtype == torch.float32
    out = layer(x)

    x_fp32 = x.float()
    weight_fp32 = layer.weight.data.float() + 1.0
    variance = x_fp32.pow(2).mean(dim=-1, keepdim=True)
    ref = (x_fp32 * torch.rsqrt(variance + layer.variance_epsilon) * weight_fp32).to(
        x.dtype
    )

    assert out.dtype == x.dtype
    torch.testing.assert_close(out, ref, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies gemma rms norm mixed input weight dtype. it consumes fixtures or inputs such as default_vllm_config. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 gemma rms norm mixed input weight dtype 的行为。 它会使用诸如 default_vllm_config 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `tests.kernels.quant_utils -> FP8_DTYPE`
- `tests.kernels.utils -> opcheck`
- `vllm.model_executor.layers.layernorm -> GemmaRMSNorm, RMSNorm`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
