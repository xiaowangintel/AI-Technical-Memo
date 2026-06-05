# test_cutlass_mxfp8_grouped_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_cutlass_mxfp8_grouped_mm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_cutlass_mxfp8_grouped_mm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_cutlass_mxfp8_grouped_mm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 6-6)
```python
"""Tests for SM100 CUTLASS MXFP8 grouped MoE kernels."""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-16)
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

### Top-level block starting at line 18 (lines 18-18)
```python
random.seed(42)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 19 (lines 19-19)
```python
set_random_seed(42)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Function `align` (lines 22-23)
```python
def align(val: int, alignment: int = 128) -> int:
    return int((val + alignment - 1) // alignment * alignment)
```
**EN:** This helper function implements the shared logic for align. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 align 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `calc_diff` (lines 27-31)
```python
def calc_diff(x, y):
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
```
**EN:** This helper function implements the shared logic for calc diff. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 calc diff 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `is_sm100_supported` (lines 34-37)
```python
def is_sm100_supported() -> bool:
    return current_platform.is_cuda() and current_platform.is_device_capability_family(
        100
    )
```
**EN:** This helper function implements the shared logic for is sm100 supported. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 is sm100 supported 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `compute_ref_output` (lines 40-61)
```python
def compute_ref_output(
    input_tensor: torch.Tensor,
    weight_list: list[torch.Tensor],
    expert_offsets: list[int],
    expert_offset: int,
    num_experts: int,
) -> torch.Tensor:
    # Build a top-1 routing score so each token maps to its owning expert.
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

### Function `compute_kernel_output` (lines 64-143)
```python
def compute_kernel_output(
    input_tensor: torch.Tensor,
    weight_tensor: torch.Tensor,
    problem_sizes: list[list[int]],
    aux_problem_sizes: list[list[int]],
    expert_offsets: list[int],
    aux_expert_offsets: list[int],
    input_blockscale_offsets: list[int],
    weight_blockscale_offsets: list[int],
    input_blockscale_offset: int,
    n_g: int,
    k_g: int,
    num_experts: int,
    expert_offset: int,
    out_dtype: torch.dtype,
) -> torch.Tensor:
    device = input_tensor.device
    _problem_sizes = torch.tensor(problem_sizes).to(device=device, dtype=torch.int32)
    _aux_problem_sizes = torch.tensor(aux_problem_sizes).to(
        device=device, dtype=torch.int32
    )
    _expert_offsets = torch.tensor(expert_offsets).to(device=device, dtype=torch.int32)
    _aux_expert_offsets = torch.tensor(aux_expert_offsets).to(
        device=device, dtype=torch.int32
    )
    _input_blockscale_offsets = torch.tensor(input_blockscale_offsets).to(
        device=device, dtype=torch.int32
    )
    _weight_blockscale_offsets = torch.tensor(weight_blockscale_offsets).to(
        device=device, dtype=torch.int32
    )

    input_quant = torch.zeros_like(
        input_tensor, dtype=torch.float8_e4m3fn, device=device
    )
    input_scale_factor = torch.zeros(
        (input_blockscale_offset, k_g // 32), dtype=torch.uint8, device=device
    )

    weight_quant = torch.zeros_like(
        weight_tensor, dtype=torch.float8_e4m3fn, device=device
    )
    weight_scale_factor = torch.zeros(
        (num_experts, n_g, k_g // 32), dtype=torch.uint8, device=device
    )

    ops.mxfp8_experts_quant(
        input_tensor,
        _problem_sizes,
        _expert_offsets,
        _input_blockscale_offsets,
        input_quant,
        input_scale_factor,
    )

    ops.mxfp8_experts_quant(
        weight_tensor,
        _aux_problem_sizes,
        _aux_expert_offsets,
        _weight_blockscale_offsets,
        weight_quant,
        weight_scale_factor,
    )
    weight_quant = weight_quant.view(num_experts, n_g, k_g).transpose(1, 2)
    weight_scale_factor = weight_scale_factor.view(
        num_experts, n_g, k_g // 32
    ).transpose(1, 2)

    output = torch.empty((expert_offset, n_g), device=device, dtype=out_dtype)
    ops.cutlass_mxfp8_grouped_mm(
        input_quant,
        weight_quant,
        input_scale_factor,
        weight_scale_factor,
        output,
        _problem_sizes,
        _expert_offsets,
        _input_blockscale_offsets,
    )
    return output
```
**EN:** This helper function implements the shared logic for compute kernel output. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 compute kernel output 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_cutlass_mxfp8_grouped_mm` (lines 146-233)
```python
@pytest.mark.skipif(
    not is_sm100_supported(),
    reason=(
        "cutlass_mxfp8_grouped_mm and mxfp8_experts_quant "
        "are only supported on CUDA SM100"
    ),
)
@pytest.mark.parametrize("num_experts", [8, 16, 32, 64])
@pytest.mark.parametrize("out_dtype", [torch.half, torch.bfloat16])
def test_cutlass_mxfp8_grouped_mm(num_experts, out_dtype):
    device = "cuda"
    alignment = 128
    n_g = random.randint(1, 64) * alignment
    k_g = random.randint(1, 64) * alignment

    expert_offset = 0
    expert_offsets = []
    aux_expert_offset = 0
    aux_expert_offsets = []
    input_blockscale_offset = 0
    input_blockscale_offsets = []
    weight_blockscale_offset = 0
    weight_blockscale_offsets = []
    problem_sizes = []
    aux_problem_sizes = []
    input_list = []
    weight_list = []

    for g in range(num_experts):
        m_g = random.randint(1, 512)
        expert_offsets.append(expert_offset)
        expert_offset += m_g
        aux_expert_offsets.append(aux_expert_offset)
        aux_expert_offset += n_g
        input_blockscale_offsets.append(input_blockscale_offset)
        input_blockscale_offset += align(m_g, 128)
        weight_blockscale_offsets.append(weight_blockscale_offset)
        weight_blockscale_offset += n_g  # n_g already align to 128
        problem_sizes.append([m_g, n_g, k_g])
        aux_problem_sizes.append([n_g, m_g, k_g])
# ... excerpt ...
        weight_list.append(weight_tensor)
    input_tensor = torch.concat(input_list, dim=0)
    weight_tensor = torch.concat(weight_list, dim=0)

    ref_output = compute_ref_output(
        input_tensor=input_tensor,
        weight_list=weight_list,
        expert_offsets=expert_offsets,
        expert_offset=expert_offset,
        num_experts=num_experts,
    )
    output = compute_kernel_output(
        input_tensor=input_tensor,
        weight_tensor=weight_tensor,
        problem_sizes=problem_sizes,
        aux_problem_sizes=aux_problem_sizes,
        expert_offsets=expert_offsets,
        aux_expert_offsets=aux_expert_offsets,
        input_blockscale_offsets=input_blockscale_offsets,
        weight_blockscale_offsets=weight_blockscale_offsets,
        input_blockscale_offset=input_blockscale_offset,
        n_g=n_g,
        k_g=k_g,
        num_experts=num_experts,
        expert_offset=expert_offset,
        out_dtype=out_dtype,
    )

    for g in range(num_experts):
        baseline = ref_output[
            expert_offsets[g] : (expert_offsets[g] + problem_sizes[g][0])
        ]
        actual = output[expert_offsets[g] : (expert_offsets[g] + problem_sizes[g][0])]
        diff = calc_diff(actual, baseline)
        assert diff < 0.001
        print(
            f"m_g={baseline.shape[0]} n_g={n_g} k_g={k_g} num_experts={num_experts}, "
            f"out_dtype={out_dtype}, diff={diff:.5f}: OK"
        )
```
**EN:** This pytest case verifies cutlass mxfp8 grouped mm. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_experts, out_dtype. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 cutlass mxfp8 grouped mm 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 num_experts、out_dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Top-level block starting at line 236 (lines 236-237)
```python
if __name__ == "__main__":
    pytest.main([__file__])
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
