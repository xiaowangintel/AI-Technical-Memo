# test_moe_permute_unpermute.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_moe_permute_unpermute.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_moe_permute_unpermute, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_moe_permute_unpermute 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""Tests for the MOE permute/unpermute kernel

Run `pytest tests/kernels/test_moe_permute_unpermute.py`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-22)
```python
import numpy as np
import pytest
import torch

from vllm.model_executor.layers.fused_moe import fused_topk
from vllm.model_executor.layers.fused_moe.expert_map_manager import (
    determine_expert_map,
)
from vllm.model_executor.layers.fused_moe.moe_permute_unpermute import (
    moe_permute,
    moe_permute_unpermute_supported,
    moe_unpermute,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as numpy, pytest, torch; and vLLM components like vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.fused_moe.expert_map_manager, vllm.model_executor.layers.fused_moe.moe_permute_unpermute, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 numpy、pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe、vllm.model_executor.layers.fused_moe.expert_map_manager、vllm.model_executor.layers.fused_moe.moe_permute_unpermute、vllm.platforms。

### Constants and module state (lines 24-26)
```python
NUM_EXPERTS = [16, 64, 256]
TOP_KS = [2, 6, 8]
EP_SIZE = [1, 4, 16]
```
**EN:** This block centralizes shared constants and parameter grids, including NUM_EXPERTS, TOP_KS, EP_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NUM_EXPERTS、TOP_KS、EP_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Top-level block starting at line 27 (lines 27-27)
```python
set_random_seed(0)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 29 (lines 29-33)
```python
if current_platform.is_rocm():
    pytest.skip(
        "moe_permute_unpermute_supported is not defined for ROCm",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `torch_permute` (lines 36-85)
```python
def torch_permute(
    hidden_states: torch.Tensor,
    topk_ids: torch.Tensor,
    #   token_expert_indices: torch.Tensor,
    topk: int,
    n_expert: int,
    n_local_expert: int,
    start_expert: int,
    expert_map: torch.Tensor | None = None,
) -> list[torch.Tensor]:
    n_token = hidden_states.shape[0]
    if expert_map is not None:
        is_local_expert = expert_map[topk_ids] != -1
        not_local_expert = expert_map[topk_ids] == -1
        topk_ids = is_local_expert * (topk_ids - start_expert) + not_local_expert * (
            topk_ids + n_expert
        )
    token_expert_indices = torch.arange(
        0, n_token * topk, dtype=torch.int32, device=hidden_states.device
    ).reshape((n_token, topk))

    sorted_topk_ids, sorted_indices = torch.sort(topk_ids.flatten(), stable=True)
    dst_row_id2src_row_id_map = token_expert_indices.flatten()[sorted_indices]

    expert_first_token_offset = torch.zeros(
        n_local_expert + 1, dtype=torch.int64, device="cuda"
    )
    idx = 0
    for i in range(0, n_local_expert):
        cnt = 0
        while idx < sorted_topk_ids.numel() and sorted_topk_ids[idx] == i:
            cnt += 1
            idx += 1
        expert_first_token_offset[i + 1] = expert_first_token_offset[i] + cnt

    _, src2dst_idx = torch.sort(dst_row_id2src_row_id_map)
    valid_row_idx = []
    permuted_hidden_states = hidden_states[dst_row_id2src_row_id_map // topk, ...]
    src_row_id2dst_row_id_map = torch.arange(
        0, n_token * topk, device="cuda", dtype=torch.int32
    )[src2dst_idx].reshape((n_token, topk))
    valid_row_idx += [i for i in range(expert_first_token_offset[-1])]
    dst_row_id2src_row_id_map[expert_first_token_offset[-1] :] = n_token * topk
    return [
        permuted_hidden_states,
        expert_first_token_offset,
        src_row_id2dst_row_id_map,
        dst_row_id2src_row_id_map,
        valid_row_idx,
    ]
```
**EN:** This helper function implements the shared logic for torch permute. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 torch permute 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `torch_unpermute` (lines 88-113)
```python
def torch_unpermute(
    permuted_hidden_states: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    token_expert_indices: torch.Tensor,
    src_row_id2dst_row_id_map: torch.Tensor,
    valid_row_idx: torch.Tensor,
    topk: int,
    n_expert: int,
) -> torch.Tensor:
    # ignore invalid row
    n_hidden = permuted_hidden_states.shape[1]
    mask = torch.zeros(permuted_hidden_states.shape[0], dtype=bool, device="cuda")
    mask[valid_row_idx] = True
    permuted_hidden_states[~mask] = 0

    permuted_hidden_states = permuted_hidden_states[
        src_row_id2dst_row_id_map.flatten(), ...
    ]
    permuted_hidden_states = permuted_hidden_states.view(-1, topk, n_hidden)
    output = (
        (permuted_hidden_states * topk_weights.unsqueeze(2))
        .sum(1)
        .to(permuted_hidden_states.dtype)
    )
    return output
```
**EN:** This helper function implements the shared logic for torch unpermute. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 torch unpermute 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_moe_permute_unpermute` (lines 116-211)
```python
@pytest.mark.parametrize("n_token", [1, 33, 1024, 5000])
@pytest.mark.parametrize("n_hidden", [2048, 7168])
@pytest.mark.parametrize("n_expert", NUM_EXPERTS)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("ep_size", EP_SIZE)
def test_moe_permute_unpermute(
    n_token: int,
    n_hidden: int,
    topk: int,
    n_expert: int,
    ep_size: int,
    dtype: torch.dtype,
):
    if not moe_permute_unpermute_supported():
        pytest.skip("moe_permute_unpermute is not supported on this platform.")
    ep_rank = np.random.randint(0, ep_size)
    expert_map = None
    n_local_expert = n_expert
    if ep_size != 1:
        n_local_expert, expert_map, _ = determine_expert_map(ep_size, ep_rank, n_expert)
        expert_map = expert_map.cuda()
    start_expert = n_local_expert * ep_rank
    set_random_seed(0)
    hidden_states = torch.randn((n_token, n_hidden), device="cuda").to(dtype)
    gating_output = torch.randn((n_token, n_expert), device="cuda").to(dtype)
    topk_weights, topk_ids, token_expert_indices = fused_topk(
        hidden_states, gating_output, topk, False
    )
    (
        gold_permuted_hidden_states,
        gold_expert_first_token_offset,
        gold_inv_permuted_idx,
        gold_permuted_idx,
        valid_row_idx,
    ) = torch_permute(
        hidden_states,
        topk_ids,
        # token_expert_indices,
        topk,
# ... excerpt ...
        n_local_expert=n_local_expert,
        expert_map=expert_map,
    )

    # check expert_first_token_offset
    torch.testing.assert_close(
        gold_expert_first_token_offset, expert_first_token_offset, atol=0, rtol=0
    )
    # check src_row_id2dst_row_id_map
    torch.testing.assert_close(
        gold_inv_permuted_idx.flatten(), inv_permuted_idx, atol=0, rtol=0
    )

    # check permuted_hidden_states, only valid token
    torch.testing.assert_close(
        gold_permuted_hidden_states[valid_row_idx],
        permuted_hidden_states[valid_row_idx],
        atol=0,
        rtol=0,
    )
    # add a random tensor to simulate group gemm
    result0 = 0.5 * permuted_hidden_states + torch.randn_like(permuted_hidden_states)
    result4 = torch.empty_like(hidden_states)
    moe_unpermute(
        result4, result0, topk_weights, inv_permuted_idx, expert_first_token_offset
    )

    gold4 = torch_unpermute(
        result0,
        topk_weights,
        topk_ids,
        token_expert_indices,
        inv_permuted_idx,
        valid_row_idx,
        topk,
        n_local_expert,
    )
    # check unpermuted hidden
    torch.testing.assert_close(result4, gold4, atol=2e-2, rtol=0)
```
**EN:** This pytest case verifies MoE permute unpermute. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as n_token, n_hidden, topk, n_expert. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 MoE permute unpermute 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 n_token、n_hidden、topk、n_expert 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `numpy`
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe -> fused_topk`
- `vllm.model_executor.layers.fused_moe.expert_map_manager -> determine_expert_map`
- `vllm.model_executor.layers.fused_moe.moe_permute_unpermute -> moe_permute, moe_permute_unpermute_supported, moe_unpermute`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
