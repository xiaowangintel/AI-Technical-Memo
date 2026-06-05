# test_expert_placement.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/distributed/test_expert_placement.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Expert Placement behavior in the Distributed test area through focused pytest scenarios. It focuses on scenarios such as Verify Round Robin Pattern, Expert Placement Various Sizes, Expert Placement Edge Cases. / 该文件在 Distributed 测试域中，通过有针对性的 pytest 场景验证 Expert Placement 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-8)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import pytest

from vllm.model_executor.layers.fused_moe.expert_map_manager import (
    determine_expert_map,
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `vllm.model_executor.layers.fused_moe.expert_map_manager`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: verify_round_robin_pattern (lines 11-45)
```python
def verify_round_robin_pattern(expert_map, ep_rank, ep_size, global_num_experts):
    """Verify that the expert map follows the round_robin pattern."""
    # Calculate expected local experts (supporting non-divisible cases)
    base_experts = global_num_experts // ep_size
    remainder = global_num_experts % ep_size

    local_num_experts = base_experts + 1 if ep_rank < remainder else base_experts

    # Expected expert IDs for this rank in round_robin pattern
    # For non-divisible cases, ranks with extra experts start earlier
    expected_expert_ids = []
    for expert_idx in range(local_num_experts):
        global_expert_id = ep_rank + expert_idx * ep_size
        expected_expert_ids.append(global_expert_id)

    # Check that only expected experts are mapped to this rank
    for global_expert_id in range(global_num_experts):
        if global_expert_id in expected_expert_ids:
            local_expert_id = expert_map[global_expert_id]
# ... omitted for brevity ...
                f"Global expert {global_expert_id} should map to local expert "
                f"{expected_local_id}, got {local_expert_id}"
            )
        else:
            assert expert_map[global_expert_id] == -1, (
                f"Global expert {global_expert_id} should not be mapped to this rank"
            )

    # Verify that all local expert IDs are consecutive starting from 0
    local_expert_ids = [expert_map[global_id] for global_id in expected_expert_ids]
    expected_local_ids = list(range(local_num_experts))
    assert local_expert_ids == expected_local_ids, (
        f"Expected local expert IDs {expected_local_ids}, got {local_expert_ids}"
    )
```
**EN:** Verify that the expert map follows the round_robin pattern. It coordinates operations such as `range`, `list`, `expected_expert_ids.append`.
**CN:** 该辅助函数为 Verify Round Robin Pattern 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `range`, `list`, `expected_expert_ids.append` 等操作。

### Test: test_expert_placement_various_sizes (lines 48-112)
```python
@pytest.mark.parametrize("expert_placement_strategy", ["round_robin"])
@pytest.mark.parametrize("world_size", [2, 4])
def test_expert_placement_various_sizes(expert_placement_strategy, world_size):
    """Test round_robin expert placement with various expert counts."""

    # Test with different global_num_experts values
    # Include both divisible and non-divisible cases
    if world_size == 2:
        test_cases = [
            (4, 2),  # 4 experts (divisible)
            (8, 2),  # 8 experts (divisible)
            (9, 2),  # 9 experts (non-divisible)
            (16, 2),  # 16 experts (divisible)
            (17, 2),  # 17 experts (non-divisible)
        ]
    elif world_size == 4:
        test_cases = [
            (8, 4),  # 8 experts (divisible)
            (16, 4),  # 16 experts (divisible)
# ... omitted for brevity ...
                f"rank {ep_rank}: expected {expected_test_local} local"
                f"experts, got {test_local_experts}"
            )

            if test_expert_map is not None:
                assert test_expert_map.shape == (test_global_experts,), (
                    f"Expected expert map shape ({test_global_experts},), "
                    f"got {test_expert_map.shape}"
                )

                # Verify round_robin pattern for this test case
                verify_round_robin_pattern(
                    test_expert_map, ep_rank, test_ep_size, test_global_experts
                )
```
**EN:** Test round_robin expert placement with various expert counts. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `range`, `determine_expert_map` before asserting the expected outcome.
**CN:** 该测试用例验证 Expert Placement Various Sizes 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `range`, `determine_expert_map` 驱动目标逻辑，再断言预期结果。

### Test: test_expert_placement_edge_cases (lines 115-137)
```python
@pytest.mark.parametrize("expert_placement_strategy", ["round_robin"])
@pytest.mark.parametrize("world_size", [2, 4])
def test_expert_placement_edge_cases(expert_placement_strategy, world_size):
    """Test edge cases for round_robin expert placement."""

    # Test case 1: ep_size = 1 (should return None for expert_map)
    local_num_experts, expert_map, _ = determine_expert_map(
        ep_size=1,
        ep_rank=0,
        global_num_experts=8,
        expert_placement_strategy=expert_placement_strategy,
    )
    assert local_num_experts == 8, "For ep_size=1, should get all experts"
    assert expert_map is None, "For ep_size=1, expert_map should be None"

    # Test case 2: ep_size = 0 (should raise assertion)
    with pytest.raises(AssertionError):
        determine_expert_map(
            ep_size=0,
            ep_rank=0,
            global_num_experts=8,
            expert_placement_strategy=expert_placement_strategy,
        )
```
**EN:** Test edge cases for round_robin expert placement. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `determine_expert_map`, `pytest.raises` before asserting the expected outcome.
**CN:** 该测试用例验证 Expert Placement Edge Cases 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `determine_expert_map`, `pytest.raises` 驱动目标逻辑，再断言预期结果。

### Test: test_determine_expert_map_comprehensive (lines 140-246)
```python
def test_determine_expert_map_comprehensive():
    """Test of determine_expert_map function with various configurations."""

    # Test cases: (ep_size, ep_rank, global_num_experts,
    # expert_placement_strategy, expected_local, expected_map_pattern)
    test_cases = [
        # Round robin placement tests
        (
            2,
            0,
            8,
            "round_robin",
            4,
            [0, -1, 1, -1, 2, -1, 3, -1],
        ),  # rank 0 gets even experts
        (
            2,
            1,
            8,
# ... omitted for brevity ...
            f"expected {expected_local} local experts, got {local_num_experts}"
        )

        if expected_map_pattern is None:
            assert expert_map is None, "Expected expert_map to be None"
        else:
            assert expert_map is not None, "Expected expert_map to not be None"
            actual_map = expert_map.tolist()
            assert actual_map == expected_map_pattern, (
                f"ep_size={ep_size}, ep_rank={ep_rank}, "
                f"global_num_experts={global_num_experts}, "
                f"expert_placement_strategy={expert_placement_strategy}: "
                f"expected map {expected_map_pattern}, got {actual_map}"
            )
```
**EN:** Test of determine_expert_map function with various configurations. The body exercises logic via `determine_expert_map`, `expert_map.tolist` before asserting the expected outcome.
**CN:** 该测试用例验证 Determine Expert Map Comprehensive 在特定场景下的行为。 函数体会先通过 `determine_expert_map`, `expert_map.tolist` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.fused_moe.expert_map_manager`
