# test_moe_align_block_size.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_moe_align_block_size.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_moe_align_block_size, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_moe_align_block_size 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""Tests for the MOE align block size function.

Run `pytest tests/kernels/moe/test_moe_align_block_size.py`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-16)
```python
import pytest
import torch

from vllm.model_executor.layers.fused_moe.moe_align_block_size import (
    batched_moe_align_block_size,
    moe_align_block_size,
)
from vllm.utils.math_utils import cdiv, round_up
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.model_executor.layers.fused_moe.moe_align_block_size, vllm.utils.math_utils, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.moe_align_block_size、vllm.utils.math_utils、vllm.utils.torch_utils。

### Constants and module state (lines 18-21)
```python
NUM_TOKENS = [1, 3, 256, 2256, 4096]
NUM_EXPERTS = [32, 160, 256, 257]
TOP_KS = [1, 2, 16, 32]
BLOCK_SIZES = [32, 128]
```
**EN:** This block centralizes shared constants and parameter grids, including NUM_TOKENS, NUM_EXPERTS, TOP_KS, BLOCK_SIZES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NUM_TOKENS、NUM_EXPERTS、TOP_KS、BLOCK_SIZES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Top-level block starting at line 22 (lines 22-22)
```python
set_random_seed(0)
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Function `_group_tokens_by_expert` (lines 25-46)
```python
def _group_tokens_by_expert(
    sorted_ids: torch.Tensor,
    expert_ids: torch.Tensor,
    block_size: int,
    valid_length: int,
    total_tokens: int,
) -> dict:
    num_blocks = valid_length // block_size
    expert_tokens: dict[int, list[int]] = {}

    for block_idx in range(num_blocks):
        expert_id = expert_ids[block_idx].item()
        block_start = block_idx * block_size
        block_end = min(block_start + block_size, valid_length)

        block_tokens = sorted_ids[block_start:block_end]
        valid_tokens = block_tokens[block_tokens < total_tokens]

        if expert_id not in expert_tokens:
            expert_tokens[expert_id] = []
        expert_tokens[expert_id].extend(valid_tokens.tolist())
    return expert_tokens
```
**EN:** This helper function implements the shared logic for group tokens by expert. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 group tokens by expert 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_verify_expert_level_sorting` (lines 49-89)
```python
def _verify_expert_level_sorting(
    actual_sorted_ids: torch.Tensor,
    golden_sorted_ids: torch.Tensor,
    expert_ids: torch.Tensor,
    block_size: int,
    valid_length: int,
    total_tokens: int,
):
    """
    Verify that actual_sorted_ids follows the correct expert-level sorting.
    The kerne limplementation may or may not preserve original token order
    in topk_ids in the final sorted_ids however this does not impact quality.
    """
    # Group tokens by expert from the golden implementation
    golden_expert_tokens = _group_tokens_by_expert(
        golden_sorted_ids, expert_ids, block_size, valid_length, total_tokens
    )

    actual_expert_tokens = _group_tokens_by_expert(
        actual_sorted_ids, expert_ids, block_size, valid_length, total_tokens
    )

    assert set(golden_expert_tokens.keys()) == set(actual_expert_tokens.keys()), (
        f"Expert IDs mismatch: golden={set(golden_expert_tokens.keys())}, "
        f"actual={set(actual_expert_tokens.keys())}"
    )

    for expert_id in golden_expert_tokens:
        golden_tokens = torch.tensor(
            golden_expert_tokens[expert_id], device=actual_sorted_ids.device
        )
        actual_tokens = torch.tensor(
            actual_expert_tokens[expert_id], device=actual_sorted_ids.device
        )
        assert torch.equal(
            torch.sort(golden_tokens)[0], torch.sort(actual_tokens)[0]
        ), (
            f"Expert {expert_id} token mismatch: "
            f"golden={golden_expert_tokens[expert_id]}, "
            f"actual={actual_expert_tokens[expert_id]}"
        )
```
**EN:** This helper function implements the shared logic for verify expert level sorting. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 verify expert level sorting 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `torch_moe_align_block_size` (lines 92-181)
```python
def torch_moe_align_block_size(
    topk_ids: torch.Tensor,
    block_size: int,
    num_experts: int,
    expert_map: torch.Tensor | None = None,
    pad_sorted_ids: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Golden torch implementation of moe_align_block_size.

    This function aligns the token distribution across experts to be compatible
    with block size for matrix multiplication by sorting tokens by expert and
    padding to block boundaries.
    """
    max_num_tokens_padded = topk_ids.numel() + num_experts * (block_size - 1)
    if pad_sorted_ids:
        max_num_tokens_padded = round_up(max_num_tokens_padded, block_size)
    if topk_ids.numel() < num_experts:
        max_num_tokens_padded = topk_ids.numel() * block_size

    flattened_token_indices = torch.arange(
        topk_ids.numel(), device=topk_ids.device, dtype=torch.int32
    )
    flattened_expert_ids = topk_ids.flatten()
    sorted_expert_ids, sort_indices = torch.sort(flattened_expert_ids, stable=True)
    sorted_token_indices = flattened_token_indices[sort_indices]

    expert_token_counts = torch.zeros(
        num_experts, dtype=torch.int64, device=topk_ids.device
    )
    for expert_id in range(num_experts):
        mask = sorted_expert_ids == expert_id
        expert_token_counts[expert_id] = mask.sum()

    expert_padded_counts = torch.zeros(
        num_experts, dtype=torch.int64, device=topk_ids.device
    )
    for expert_id in range(num_experts):
        original_count = expert_token_counts[expert_id]
        if expert_map is not None and expert_map[expert_id] == -1:
# ... excerpt ...
    )
    max_num_blocks = (max_num_tokens_padded + block_size - 1) // block_size
    expert_ids = torch.full(
        (max_num_blocks,), -1, dtype=torch.int32, device=topk_ids.device
    )

    current_pos = 0
    current_block = 0
    for expert_id in range(num_experts):
        if expert_map is not None and expert_map[expert_id] == -1:
            continue

        expert_mask = sorted_expert_ids == expert_id
        expert_tokens = sorted_token_indices[expert_mask]
        num_expert_tokens = expert_tokens.shape[0]

        if num_expert_tokens > 0:
            sorted_token_ids[current_pos : current_pos + num_expert_tokens] = (
                expert_tokens
            )

            expert_blocks_needed = expert_padded_counts[expert_id] // block_size

            expert_id_new = expert_id
            if expert_map is not None:
                expert_id_new = expert_map[expert_id]
            expert_ids[current_block : current_block + expert_blocks_needed] = (
                expert_id_new
            )

            current_pos += expert_padded_counts[expert_id]
            current_block += expert_blocks_needed

    total_padded_tokens = expert_padded_counts.sum()
    num_tokens_post_pad = torch.tensor(
        [total_padded_tokens], dtype=torch.int32, device=topk_ids.device
    )

    return sorted_token_ids, expert_ids, num_tokens_post_pad
```
**EN:** This helper function implements the shared logic for torch MoE align block size. It keeps the surrounding module logic factored into a reusable unit. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 torch MoE align block size 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。 由于该代码块较大，这里只展示关键片段。

### Function `test_moe_align_block_size` (lines 184-242)
```python
@pytest.mark.parametrize("m", NUM_TOKENS)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("num_experts", NUM_EXPERTS)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("pad_sorted_ids", [False, True])
def test_moe_align_block_size(
    m: int, topk: int, num_experts: int, block_size: int, pad_sorted_ids: bool
):
    """Test moe_align_block_size without expert mapping"""
    topk_ids = torch.zeros((m, topk), device="cuda", dtype=torch.int32)
    for i in range(m):
        experts = torch.randperm(num_experts, device="cuda")[:topk]
        topk_ids[i] = experts

    actual_sorted_ids, actual_expert_ids, actual_num_tokens = moe_align_block_size(
        topk_ids=topk_ids,
        block_size=block_size,
        num_experts=num_experts,
        pad_sorted_ids=pad_sorted_ids,
    )
    golden_sorted_ids, golden_expert_ids, golden_num_tokens = (
        torch_moe_align_block_size(
            topk_ids=topk_ids,
            block_size=block_size,
            num_experts=num_experts,
            pad_sorted_ids=pad_sorted_ids,
        )
    )

    torch.testing.assert_close(actual_num_tokens, golden_num_tokens, atol=0, rtol=0)
    torch.testing.assert_close(actual_expert_ids, golden_expert_ids, atol=0, rtol=0)

    # For sorted_token_ids, verify block-level correctness rather than exact
    # order Tokens within each expert's blocks can be in any order, but expert
    # regions must be correct
    _verify_expert_level_sorting(
        actual_sorted_ids,
        golden_sorted_ids,
        actual_expert_ids,
        block_size,
        actual_num_tokens.item(),
        m * topk,
    )

    total_tokens = m * topk
    assert actual_num_tokens.item() % block_size == 0, (
        "num_tokens_post_pad should be divisible by block_size"
    )
    assert actual_num_tokens.item() >= total_tokens, (
        "num_tokens_post_pad should be at least total_tokens"
    )
    valid_tokens = actual_sorted_ids[actual_sorted_ids < total_tokens]
    assert len(valid_tokens) == total_tokens, (
        f"Should have exactly {total_tokens} valid tokens, got {len(valid_tokens)}"
    )
    actual_num_blocks = cdiv(int(actual_num_tokens.item()), block_size)
    assert (actual_expert_ids[:actual_num_blocks] >= 0).all() and (
        actual_expert_ids[:actual_num_blocks] < num_experts
    ).all(), "expert_ids should contain valid expert indices"
```
**EN:** This pytest case verifies MoE align block size. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, topk, num_experts, block_size. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 MoE align block size 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 m、topk、num_experts、block_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。

### Function `test_moe_align_block_size_with_expert_map` (lines 245-288)
```python
@pytest.mark.parametrize("m", [16, 32, 2048])
@pytest.mark.parametrize("topk", [2, 4])
@pytest.mark.parametrize("num_experts", [8, 64])
@pytest.mark.parametrize("block_size", [64])
def test_moe_align_block_size_with_expert_map(
    m: int, topk: int, num_experts: int, block_size: int
):
    """Test moe_align_block_size with expert mapping (EP scenario)"""
    topk_ids = torch.zeros((m, topk), device="cuda", dtype=torch.int32)
    for i in range(m):
        experts = torch.randperm(num_experts, device="cuda")[:topk]
        topk_ids[i] = experts

    expert_map = torch.full((num_experts,), -1, device="cuda", dtype=torch.int32)
    local_experts = list(range(0, num_experts, 2))
    for i, expert_id in enumerate(local_experts):
        expert_map[expert_id] = i

    actual_sorted_ids, actual_expert_ids, actual_num_tokens = moe_align_block_size(
        topk_ids=topk_ids,
        block_size=block_size,
        num_experts=num_experts,
        expert_map=expert_map,
        ignore_invalid_experts=True,
    )
    golden_sorted_ids, golden_expert_ids, golden_num_tokens = (
        torch_moe_align_block_size(
            topk_ids=topk_ids,
            block_size=block_size,
            num_experts=num_experts,
            expert_map=expert_map,
        )
    )

    torch.testing.assert_close(actual_num_tokens, golden_num_tokens, atol=0, rtol=0)
    torch.testing.assert_close(actual_expert_ids, golden_expert_ids, atol=0, rtol=0)
    _verify_expert_level_sorting(
        actual_sorted_ids,
        golden_sorted_ids,
        actual_expert_ids,
        block_size,
        actual_num_tokens.item(),
        m * topk,
    )
```
**EN:** This pytest case verifies MoE align block size with expert map. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, topk, num_experts, block_size. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 MoE align block size with expert map 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、topk、num_experts、block_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。

### Function `test_moe_align_block_size_deterministic` (lines 291-316)
```python
def test_moe_align_block_size_deterministic():
    m, topk, num_experts, block_size = 128, 2, 32, 64

    torch.manual_seed(42)
    topk_ids = torch.randint(
        0, num_experts, (m, topk), device="cuda", dtype=torch.int32
    )

    # expect the results to be reproducible
    results = []
    for _ in range(5):
        sorted_ids, expert_ids, num_tokens = moe_align_block_size(
            topk_ids=topk_ids, block_size=block_size, num_experts=num_experts
        )
        results.append((sorted_ids.clone(), expert_ids.clone(), num_tokens.clone()))

    for i in range(1, len(results)):
        assert torch.equal(results[0][0], results[i][0]), (
            "sorted_ids should be deterministic"
        )
        assert torch.equal(results[0][1], results[i][1]), (
            "expert_ids should be deterministic"
        )
        assert torch.equal(results[0][2], results[i][2]), (
            "num_tokens should be deterministic"
        )
```
**EN:** This pytest case verifies MoE align block size deterministic. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 MoE align block size deterministic 的行为。 结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

### Function `test_batched_moe_align_block_size` (lines 319-409)
```python
@pytest.mark.parametrize("max_tokens_per_batch", [13, 16, 512])
@pytest.mark.parametrize("num_experts", [8, 16, 32, 64])
@pytest.mark.parametrize("block_size", [8, 16, 32, 64])
@pytest.mark.parametrize("simulate_empty_batches", [False, True])
def test_batched_moe_align_block_size(
    max_tokens_per_batch: int,
    num_experts: int,
    block_size: int,
    simulate_empty_batches: bool,
):
    def ref_outputs(
        expert_num_tokens: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        E = expert_num_tokens.size(0)

        # Round up so each batch can be split to blocks evenly.
        Msum = round_up(max_tokens_per_batch, block_size) * E
        ref_sorted_ids = torch.empty((Msum,), dtype=torch.int32)
        ref_expert_ids = torch.empty((Msum // block_size,), dtype=torch.int32)
        ref_num_tokens_post_pad = torch.empty((1,), dtype=torch.int32)

        # Initialize
        sentinel = E * max_tokens_per_batch
        ref_sorted_ids.fill_(sentinel)
        ref_expert_ids.fill_(-1)

        # Fill ref_sorted_ids
        i = 0
        for expert_id, expert_nt in enumerate(expert_num_tokens):
            token_offset = expert_id * max_tokens_per_batch
            for j in range(expert_nt):
                ref_sorted_ids[i] = token_offset + j
                i += 1
            # round up i to the next block_size
            i = round_up(i, block_size)

        ref_num_tokens_post_pad[0] = i

        # Fill expert_ids
        nt_ceil_sum = 0
# ... excerpt ...
        )

    # Compute expert_num_tokens
    expert_num_tokens = torch.randint(
        low=0,
        high=max_tokens_per_batch,
        size=(num_experts,),
        device="cpu",
        dtype=torch.int32,
    )
    if simulate_empty_batches:
        # mark half the batches to have 0 tokens
        zero_batches = torch.randperm(num_experts)[: num_experts // 2]
        expert_num_tokens[zero_batches] = 0

    # ref outputs
    ref_sorted_ids, ref_expert_ids, ref_num_tokens_post_pad = ref_outputs(
        expert_num_tokens
    )

    # outputs
    sorted_ids, expert_ids, num_tokens_post_pad = batched_moe_align_block_size(
        max_tokens_per_batch, block_size, expert_num_tokens.to("cuda")
    )

    assert ref_sorted_ids.size() == sorted_ids.size(), (
        f"{ref_sorted_ids.size()} vs {sorted_ids.size()}"
    )
    assert ref_expert_ids.size() == expert_ids.size(), (
        f"{ref_expert_ids.size()} vs {expert_ids.size()}"
    )
    assert ref_num_tokens_post_pad.size() == num_tokens_post_pad.size(), (
        f"{ref_num_tokens_post_pad.size()} vs {num_tokens_post_pad.size()}"
    )
    torch.testing.assert_close(ref_sorted_ids, sorted_ids, atol=0, rtol=0)
    torch.testing.assert_close(ref_expert_ids, expert_ids, atol=0, rtol=0)
    torch.testing.assert_close(
        ref_num_tokens_post_pad, num_tokens_post_pad, atol=0, rtol=0
    )
```
**EN:** This pytest case verifies batched MoE align block size. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as max_tokens_per_batch, num_experts, block_size, simulate_empty_batches. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 batched MoE align block size 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 max_tokens_per_batch、num_experts、block_size、simulate_empty_batches 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.moe_align_block_size -> batched_moe_align_block_size, moe_align_block_size`
- `vllm.utils.math_utils -> cdiv, round_up`
- `vllm.utils.torch_utils -> set_random_seed`
