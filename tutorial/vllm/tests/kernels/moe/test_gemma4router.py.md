# test_gemma4router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_gemma4router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_gemma4router, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_gemma4router 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-9)
```python
import pytest
import torch

from vllm.model_executor.models.gemma4 import (
    gemma4_fused_routing_kernel_triton,
    gemma4_routing_function_torch,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.model_executor.models.gemma4.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.model_executor.models.gemma4。

### Function `sort_by_id` (lines 12-14)
```python
def sort_by_id(w, ids):
    order = ids.argsort(dim=-1)
    return w.gather(1, order), ids.gather(1, order)
```
**EN:** This helper function implements the shared logic for sort by id. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 sort by id 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_gemma4_routing_kernel_triton` (lines 19-57)
```python
@pytest.mark.parametrize("num_tokens", [1, 2, 2048, 250000])
@pytest.mark.parametrize("num_experts", [128])  # gemma4 moe experts
@pytest.mark.parametrize("topk", [8])  # gemma4 topk
@pytest.mark.parametrize("dtype", [torch.bfloat16, torch.half, torch.float32])
def test_gemma4_routing_kernel_triton(
    num_tokens: int,
    num_experts: int,
    topk: int,
    dtype: torch.dtype,
):
    torch.manual_seed(0)

    gating = torch.randn(num_tokens, num_experts, dtype=dtype, device="cuda")
    scales = torch.rand(num_experts, dtype=torch.float32, device="cuda")

    ref_w, ref_ids = gemma4_routing_function_torch(gating, topk, scales)
    tri_w, tri_ids = gemma4_fused_routing_kernel_triton(gating, topk, scales)

    # Sort by expert id — to remove tie-breaking differences
    ref_ws, ref_is = sort_by_id(ref_w, ref_ids)
    tri_ws, tri_is = sort_by_id(tri_w, tri_ids)

    ids_match = (ref_is == tri_is).all().item()
    weights_match = torch.allclose(ref_ws, tri_ws, atol=1e-2, rtol=1e-2)
    all_match = ids_match and weights_match
    max_err = (ref_ws - tri_ws).abs().max().item()
    print(
        f"T={num_tokens:5d} E={num_experts:4d} K={topk} "
        f"{str(dtype).split('.')[-1]:7s} ids={ids_match} max_Δweight={max_err:.2e}"
    )
    if not all_match:
        bad = (ref_is != tri_is).any(dim=-1).nonzero(as_tuple=True)[0]
        if len(bad):
            r = bad[0].item()
            print(
                f"  first bad row {r}: ref_ids={ref_ids[r].tolist()} "
                f"tri_ids={tri_ids[r].tolist()}"
            )
        assert all_match
```
**EN:** This pytest case verifies gemma4 routing kernel triton. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, num_experts, topk, dtype. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 gemma4 routing kernel triton 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、num_experts、topk、dtype 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.models.gemma4 -> gemma4_fused_routing_kernel_triton, gemma4_routing_function_torch`
