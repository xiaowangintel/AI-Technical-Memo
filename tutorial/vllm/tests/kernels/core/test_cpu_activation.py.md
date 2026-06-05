# test_cpu_activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/core/test_cpu_activation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / core / test_cpu_activation, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / core / test_cpu_activation 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-10)
```python
import pytest
import torch

from tests.kernels.allclose_default import get_default_atol, get_default_rtol
from tests.kernels.utils import opcheck
from vllm.platforms import CpuArchEnum, current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.allclose_default, tests.kernels.utils; and vLLM components like vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.allclose_default、tests.kernels.utils；vLLM 内部组件，例如 vllm.platforms、vllm.utils.torch_utils。

### Top-level block starting at line 12 (lines 12-13)
```python
if not current_platform.is_cpu():
    pytest.skip("skipping CPU-only tests", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 15-22)
```python
from vllm.model_executor.layers.activation import (
    GELU,
    FastGELU,
    GeluAndMul,
    NewGELU,
    QuickGELU,
    SiluAndMul,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in and vLLM components like vllm.model_executor.layers.activation.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括vLLM 内部组件，例如 vllm.model_executor.layers.activation。

### Constants and module state (lines 24-27)
```python
DTYPES = [torch.bfloat16, torch.float32]
NUM_TOKENS = [7, 83]
D = [512, 2048]
SEEDS = [0]
```
**EN:** This block centralizes shared constants and parameter grids, including DTYPES, NUM_TOKENS, D, SEEDS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DTYPES、NUM_TOKENS、D、SEEDS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_cpu_act_and_mul` (lines 30-65)
```python
@pytest.mark.parametrize(
    ("activation_cls", "fn"),
    [
        (SiluAndMul, torch.ops._C.silu_and_mul),
        (GeluAndMul, torch.ops._C.gelu_and_mul),
        (GeluAndMul, torch.ops._C.gelu_tanh_and_mul),
    ],
)
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("d", D)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@torch.inference_mode()
def test_cpu_act_and_mul(
    default_vllm_config,
    activation_cls: type[torch.nn.Module],
    fn: object,
    num_tokens: int,
    d: int,
    dtype: torch.dtype,
    seed: int,
) -> None:
    set_random_seed(seed)
    x = torch.randn(num_tokens, 2 * d, dtype=dtype)

    layer = activation_cls()
    out = layer(x)
    ref_out = layer.forward_native(x)

    torch.testing.assert_close(
        out, ref_out, atol=get_default_atol(out), rtol=get_default_rtol(out)
    )

    output_shape = x.shape[:-1] + (x.shape[-1] // 2,)
    raw_out = torch.empty(output_shape, dtype=x.dtype, device=x.device)
    opcheck(fn, (raw_out, x))
```
**EN:** This pytest case verifies CPU act and mul. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, activation_cls, fn, num_tokens. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 CPU act and mul 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、activation_cls、fn、num_tokens 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

### Function `test_cpu_unary_activation` (lines 68-111)
```python
@pytest.mark.parametrize(
    ("activation_cls", "fn", "op_args"),
    [
        (NewGELU, torch.ops._C.gelu_new, ()),
        (FastGELU, torch.ops._C.gelu_fast, ()),
        (QuickGELU, torch.ops._C.gelu_quick, ()),
        pytest.param(
            GELU,
            getattr(torch.ops._C, "activation_lut_bf16", None),
            ("gelu",),
            marks=pytest.mark.skipif(
                current_platform.get_cpu_architecture() != CpuArchEnum.ARM,
                reason="activation_lut_bf16 is only built on Arm CPU",
            ),
        ),
    ],
)
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("d", D)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@torch.inference_mode()
def test_cpu_unary_activation(
    default_vllm_config,
    activation_cls: type[torch.nn.Module],
    fn: object,
    op_args: tuple[str, ...],
    num_tokens: int,
    d: int,
    dtype: torch.dtype,
    seed: int,
) -> None:
    set_random_seed(seed)
    x = torch.randn(num_tokens, d, dtype=dtype)
    layer = activation_cls()
    out = layer(x)
    ref_out = layer.forward_native(x)
    torch.testing.assert_close(
        out, ref_out, atol=get_default_atol(out), rtol=get_default_rtol(out)
    )
    # gelu with activation_lut_bf16 only makes sense for BF16
    if not (activation_cls is GELU and dtype != torch.bfloat16):
        raw_out = torch.empty_like(x)
        opcheck(fn, (raw_out, x, *op_args))
```
**EN:** This pytest case verifies CPU unary activation. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, activation_cls, fn, op_args. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 CPU unary activation 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、activation_cls、fn、op_args 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `tests.kernels.allclose_default -> get_default_atol, get_default_rtol`
- `tests.kernels.utils -> opcheck`
- `vllm.platforms -> CpuArchEnum, current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
- `vllm.model_executor.layers.activation -> GELU, FastGELU, GeluAndMul, NewGELU, QuickGELU, SiluAndMul`
