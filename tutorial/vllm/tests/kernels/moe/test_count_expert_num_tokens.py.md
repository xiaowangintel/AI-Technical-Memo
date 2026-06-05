# test_count_expert_num_tokens.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_count_expert_num_tokens.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_count_expert_num_tokens, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_count_expert_num_tokens 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-5)
```python
"""
Tests compute_expert_num_tokens kernels
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 7-12)
```python
import dataclasses

import pytest
import torch

from vllm.model_executor.layers.fused_moe.utils import count_expert_num_tokens
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, pytest, torch; and vLLM components like vllm.model_executor.layers.fused_moe.utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.utils。

### Class `TestTensors` (lines 15-19)
```python
@dataclasses.dataclass
class TestTensors:
    topk_ids: torch.Tensor
    expert_map: torch.Tensor | None = None
```
**EN:** This dataclass packages the fields needed to describe TestTensors. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 TestTensors 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `TestTensors.to_device` (lines 20-23)
```python
    def to_device(self, device: str):
        self.topk_ids = self.topk_ids.to(device=device)
        if self.expert_map is not None:
            self.expert_map = self.expert_map.to(device=device)
```
**EN:** This method on `TestTensors` implements to device. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestTensors` 中的这个方法实现了 to device。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestTensors.make` (lines 25-38)
```python
    @staticmethod
    def make(
        num_tokens: int,
        num_topk: int,
        num_experts: int,
        device: str,
        topk_ids_dtype: torch.dtype,
    ) -> "TestTensors":
        # make topk ids
        topk_ids = torch.empty((num_tokens, num_topk), device=device, dtype=torch.int64)
        for x in range(num_tokens):
            topk_ids[x] = torch.randperm(num_experts)[:num_topk]
        topk_ids = topk_ids.to(dtype=torch.int64)
        return TestTensors(topk_ids=topk_ids)
```
**EN:** This method on `TestTensors` implements make. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestTensors` 中的这个方法实现了 make。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestTensors.with_ep_rank` (lines 40-50)
```python
    def with_ep_rank(
        self, ep_rank: int, num_global_experts: int, num_local_experts: int, device: str
    ):
        # make an expert map
        expert_map = torch.empty((num_global_experts), device=device, dtype=torch.int32)
        expert_map.fill_(-1)
        s = ep_rank * num_local_experts
        e = s + num_local_experts
        expert_map[s:e] = torch.tensor(list(range(num_local_experts)), device=device)

        return TestTensors(topk_ids=self.topk_ids.clone(), expert_map=expert_map)
```
**EN:** This method on `TestTensors` implements with ep rank. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestTensors` 中的这个方法实现了 with ep rank。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_impl` (lines 53-65)
```python
def ref_impl(tt: TestTensors, expert_num_tokens: torch.Tensor):
    # do the reference in cpu
    tt.to_device("cpu")
    expert_ids, counts = tt.topk_ids.unique(return_counts=True)

    for eid, count in zip(expert_ids, counts):
        if eid != -1 and tt.expert_map is not None:
            eid = tt.expert_map[eid]

        if eid == -1:
            continue

        expert_num_tokens[eid] += count
```
**EN:** This helper acts as a reference implementation for impl. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 impl 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `do_test_compute_expert_num_tokens` (lines 68-110)
```python
def do_test_compute_expert_num_tokens(
    num_tokens: int,
    num_topk: int,
    num_experts: int,
    ep_size: int,
    topk_ids_dtype: torch.dtype,
):
    assert num_topk <= num_experts

    tt = TestTensors.make(
        num_tokens, num_topk, num_experts, topk_ids_dtype=topk_ids_dtype, device="cpu"
    )

    num_global_experts = num_experts
    assert num_global_experts % ep_size == 0
    num_local_experts = num_global_experts // ep_size
    for ep_rank in range(ep_size):
        tt_rank = tt.with_ep_rank(ep_rank, num_global_experts, num_local_experts, "cpu")

        ref_expert_num_tokens = torch.zeros(
            (num_local_experts), device="cpu", dtype=torch.int32
        )
        ref_impl(tt_rank, ref_expert_num_tokens)
        ref_expert_num_tokens = ref_expert_num_tokens.to("cuda")

        tt_rank.to_device("cuda")
        # Test with expert_map
        triton_expert_num_tokens_w_emap = count_expert_num_tokens(
            tt_rank.topk_ids, num_local_experts, tt_rank.expert_map
        )

        # Test without expert map
        topk_ids = tt_rank.expert_map[tt_rank.topk_ids].to(topk_ids_dtype)
        triton_expert_num_tokens_wo_emap = count_expert_num_tokens(
            topk_ids, num_local_experts, expert_map=None
        )

        torch.testing.assert_close(
            ref_expert_num_tokens, triton_expert_num_tokens_w_emap, atol=0, rtol=0
        )
        torch.testing.assert_close(
            ref_expert_num_tokens, triton_expert_num_tokens_wo_emap, atol=0, rtol=0
        )
```
**EN:** This helper function implements the shared logic for do test compute expert num tokens. numeric results are compared against a reference with explicit tolerances.
**CN:** 该辅助函数实现了 do test compute expert num tokens 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_compute_expert_num_tokens` (lines 113-127)
```python
@pytest.mark.parametrize("num_tokens", [1, 4, 8, 11, 127, 128, 3333, 7317])
@pytest.mark.parametrize("num_topk", [2, 6, 8])
@pytest.mark.parametrize("num_experts", [64])
@pytest.mark.parametrize("ep_size", [1, 2, 4])
@pytest.mark.parametrize("topk_ids_dtype", [torch.int64])
def test_compute_expert_num_tokens(
    num_tokens: int,
    num_topk: int,
    num_experts: int,
    ep_size: int,
    topk_ids_dtype: torch.dtype,
):
    do_test_compute_expert_num_tokens(
        num_tokens, num_topk, num_experts, ep_size, topk_ids_dtype
    )
```
**EN:** This pytest case verifies compute expert num tokens. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, num_topk, num_experts, ep_size.
**CN:** 该 pytest 用例验证 compute expert num tokens 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、num_topk、num_experts、ep_size 等 fixture 或输入。

### Function `test_compute_expert_num_tokens_from_numel` (lines 130-143)
```python
@pytest.mark.parametrize("numel", list(range(1, 8192, 111)))
@pytest.mark.parametrize("num_experts", [32])
@pytest.mark.parametrize("ep_size", [2])
@pytest.mark.parametrize("topk_ids_dtype", [torch.int64])
def test_compute_expert_num_tokens_from_numel(
    numel: int, num_experts: int, ep_size: int, topk_ids_dtype: torch.dtype
):
    do_test_compute_expert_num_tokens(
        num_tokens=numel,
        num_topk=1,
        num_experts=num_experts,
        ep_size=ep_size,
        topk_ids_dtype=topk_ids_dtype,
    )
```
**EN:** This pytest case verifies compute expert num tokens from numel. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as numel, num_experts, ep_size, topk_ids_dtype.
**CN:** 该 pytest 用例验证 compute expert num tokens from numel 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 numel、num_experts、ep_size、topk_ids_dtype 等 fixture 或输入。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `dataclasses`
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.utils -> count_expert_num_tokens`
