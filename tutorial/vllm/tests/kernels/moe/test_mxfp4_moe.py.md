# test_mxfp4_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_mxfp4_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_mxfp4_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_mxfp4_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 4-4)
```python
"""Tests for SM100 CUTLASS MXFP4 x MXFP4 grouped MoE kernels."""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 6-14)
```python
import random

import pytest
import torch

from tests.kernels.utils import torch_moe_single
from vllm import _custom_ops as ops
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as random, pytest, torch; shared test helpers from tests.kernels.utils; and vLLM components like vllm, vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 random、pytest、torch；共享测试辅助模块，例如 tests.kernels.utils；vLLM 内部组件，例如 vllm、vllm.platforms、vllm.utils.torch_utils。

### Top-level block starting at line 16 (lines 16-16)
```python
random.seed(42)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 17 (lines 17-17)
```python
set_random_seed(42)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 19-19)
```python
MXFP4_BLOCK_SIZE = 32
```
**EN:** This block centralizes shared constants and parameter grids, including MXFP4_BLOCK_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MXFP4_BLOCK_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `align` (lines 22-23)
```python
def align(val: int, alignment: int = 128) -> int:
    return int((val + alignment - 1) // alignment * alignment)
```
**EN:** This helper function implements the shared logic for align. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 align 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `calc_diff` (lines 26-30)
```python
def calc_diff(x, y):
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
```
**EN:** This helper function implements the shared logic for calc diff. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 calc diff 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `is_sm100_supported` (lines 33-36)
```python
def is_sm100_supported() -> bool:
    return current_platform.is_cuda() and current_platform.is_device_capability_family(
        100
    )
```
**EN:** This helper function implements the shared logic for is sm100 supported. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 is sm100 supported 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `compute_ref_output` (lines 39-60)
```python
def compute_ref_output(
    input_tensor: torch.Tensor,
    weight_list: list[torch.Tensor],
    expert_offsets: list[int],
    expert_offset: int,
    num_experts: int,
) -> torch.Tensor:
    """Reference output using torch_moe_single with top-1 routing."""
    score = torch.full(
        (expert_offset, num_experts),
        -1e9,
        device=input_tensor.device,
        dtype=torch.float32,
    )
    for g in range(num_experts):
        start = expert_offsets[g]
        end = expert_offsets[g + 1] if g + 1 < num_experts else expert_offset
        score[start:end, g] = 0.0

    return torch_moe_single(
        input_tensor, torch.stack(weight_list, dim=0), score, topk=1
    )
```
**EN:** This helper function implements the shared logic for compute ref output. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 compute ref output 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_cutlass_mxfp4_grouped_mm` (lines 63-200)
```python
@pytest.mark.skipif(
    not is_sm100_supported(),
    reason="cutlass_mxfp4_group_mm requires CUDA SM100",
)
@pytest.mark.parametrize("num_experts", [8, 16, 32])
@pytest.mark.parametrize("out_dtype", [torch.bfloat16])
def test_cutlass_mxfp4_grouped_mm(num_experts, out_dtype):
    """
    Test the MXFP4 grouped GEMM kernel by:
    1. Creating random per-expert inputs and weights
    2. Quantizing both to MXFP4 using the CUDA kernel
    3. Running the CUTLASS grouped GEMM
    4. Comparing against BF16 reference
    """
    device = "cuda"
    alignment = 128
    # N and K must be multiples of 128 for clean swizzle layout
    n_g = random.randint(1, 16) * alignment
    k_g = random.randint(1, 16) * alignment

    expert_offset = 0
    expert_offsets_input = []
    problem_sizes = []
    input_list = []
    weight_list = []

    for g in range(num_experts):
        m_g = random.randint(1, 256)
        expert_offsets_input.append(expert_offset)
        expert_offset += m_g
        problem_sizes.append([m_g, n_g, k_g])

        input_list.append(
            torch.normal(0.0, std=0.5, size=(m_g, k_g), device=device, dtype=out_dtype)
        )
        weight_list.append(
            torch.normal(0.0, std=0.5, size=(n_g, k_g), device=device, dtype=out_dtype)
        )

    input_tensor = torch.concat(input_list, dim=0)  # [M_total, K]
# ... excerpt ...

    # Run the MXFP4 grouped GEMM
    ops.cutlass_mxfp4_moe_mm(
        output,
        input_quant,
        weight_quant,
        input_sf,
        weight_sf_3d,
        _problem_sizes,
        _expert_offsets,
        _input_bs,
    )

    # Reference: BF16 matmul
    ref_output = compute_ref_output(
        input_tensor=input_tensor,
        weight_list=weight_list,
        expert_offsets=expert_offsets_input,
        expert_offset=expert_offset,
        num_experts=num_experts,
    )

    # Compare per-expert
    for g in range(num_experts):
        start = expert_offsets_input[g]
        end = expert_offsets_input[g + 1] if g + 1 < num_experts else expert_offset
        if start == end:
            continue
        baseline = ref_output[start:end]
        actual = output[start:end]
        diff = calc_diff(actual, baseline)
        print(
            f"m_g={end - start} n_g={n_g} k_g={k_g} "
            f"num_experts={num_experts}, "
            f"out_dtype={out_dtype}, diff={diff:.5f}"
        )
        # FP4 quantization is very lossy (~4 bits precision)
        # Comparing quantized vs full-precision gives cosine diff of 0.05-0.15
        assert diff < 0.15, f"Expert {g}: diff={diff:.5f} exceeds threshold"
```
**EN:** This pytest case verifies cutlass mxfp4 grouped mm. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_experts, out_dtype. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 cutlass mxfp4 grouped mm 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 num_experts、out_dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `test_mxfp4_experts_quant_basic` (lines 203-244)
```python
@pytest.mark.skipif(
    not is_sm100_supported(),
    reason="mxfp4_experts_quant requires CUDA SM100",
)
def test_mxfp4_experts_quant_basic():
    """
    Basic smoke test for the MXFP4 experts quantization kernel.
    """
    device = "cuda"
    num_experts = 4
    k = 256
    tokens_per_expert = 16

    total_tokens = tokens_per_expert * num_experts
    input_tensor = torch.randn(total_tokens, k, device=device, dtype=torch.bfloat16) / 5

    expert_offsets = [i * tokens_per_expert for i in range(num_experts + 1)]
    blockscale_offsets = [
        align(i * tokens_per_expert, 128) for i in range(num_experts + 1)
    ]

    _expert_offsets = torch.tensor(expert_offsets, device=device, dtype=torch.int32)
    _blockscale_offsets = torch.tensor(
        blockscale_offsets, device=device, dtype=torch.int32
    )

    output, output_sf = ops.mxfp4_experts_quant(
        input_tensor,
        _expert_offsets,
        _blockscale_offsets,
        num_experts,
        topk=1,
    )

    assert output.shape == (total_tokens, k // 2)
    assert output.dtype == torch.uint8
    assert output_sf.dtype == torch.uint8
    assert output.any(), "Quantized output is all zeros"
    print(
        f"MXFP4 experts quant: output shape={output.shape}, sf shape={output_sf.shape}"
    )
    print("PASSED")
```
**EN:** This pytest case verifies mxfp4 experts quant basic. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 mxfp4 experts quant basic 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Top-level block starting at line 247 (lines 247-248)
```python
if __name__ == "__main__":
    pytest.main([__file__, "-v", "-s"])
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `random`
- `pytest`
- `torch`
- `tests.kernels.utils -> torch_moe_single`
- `vllm -> _custom_ops`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
