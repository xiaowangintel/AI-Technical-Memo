# test_gptq.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_gptq.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_gptq, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_gptq 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-7)
```python
import torch

from tests.kernels.utils import opcheck
from vllm import _custom_ops as ops  # noqa: F401
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as torch; shared test helpers from tests.kernels.utils; and vLLM components like vllm.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 torch；共享测试辅助模块，例如 tests.kernels.utils；vLLM 内部组件，例如 vllm。

### Function `test_gptq_shuffle_opcheck` (lines 10-16)
```python
def test_gptq_shuffle_opcheck():
    weight = torch.randint(
        -2000000, 2000000, (1792, 4096), device="cuda", dtype=torch.int32
    )
    perm = torch.empty((0,), device="cuda", dtype=torch.int32)
    bit = 4
    opcheck(torch.ops._C.gptq_shuffle, (weight, perm, bit))
```
**EN:** This pytest case verifies gptq shuffle opcheck. it also validates that the custom operator entry point is wired correctly.
**CN:** 该 pytest 用例验证 gptq shuffle opcheck 的行为。 它还会校验自定义算子入口是否正确接线。

### Function `test_gptq_gemm_opcheck` (lines 19-35)
```python
def test_gptq_gemm_opcheck():
    a = torch.rand((240, 4096), device="cuda", dtype=torch.float16)
    weight = torch.randint(
        -2000000, 2000000, (512, 6144), device="cuda", dtype=torch.int32
    )
    zeros = torch.zeros((32, 768), device="cuda", dtype=torch.int32)
    scales = torch.rand((32, 6144), device="cuda", dtype=torch.float16)
    idx = torch.empty((0,), device="cuda", dtype=torch.int32)
    use_exllama = True
    bit = 4
    # Test both GPTQv1 and GPTQv2 format
    opcheck(
        torch.ops._C.gptq_gemm, (a, weight, zeros, scales, idx, use_exllama, True, bit)
    )
    opcheck(
        torch.ops._C.gptq_gemm, (a, weight, zeros, scales, idx, use_exllama, False, bit)
    )
```
**EN:** This pytest case verifies gptq gemm opcheck. it also validates that the custom operator entry point is wired correctly.
**CN:** 该 pytest 用例验证 gptq gemm opcheck 的行为。 它还会校验自定义算子入口是否正确接线。

## Key Concepts / 关键概念
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `torch`
- `tests.kernels.utils -> opcheck`
- `vllm -> _custom_ops`
