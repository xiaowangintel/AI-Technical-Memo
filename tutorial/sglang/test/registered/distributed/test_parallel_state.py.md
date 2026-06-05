# test_parallel_state.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_parallel_state.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates parallel state behavior in SGLang's distributed area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 分布式 领域中与 parallel state 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: supporting statements / 辅助语句
```python
"""
Test file to verify the correctness of parallel group calculations.

This test validates that the parallel group initialization creates the correct
groups for different parallelism configurations including:
- Tensor parallelism (TP)
- Pipeline parallelism (PP)
- Attention context parallelism (attn_cp)
- Attention data parallelism (attn_dp)
- MoE expert parallelism (EP)
- MoE data parallelism (moe_dp)

These tests call the ACTUAL initialize_model_parallel() function with mocked
distributed backend to verify the group construction logic.

## How These Tests Work

initialize_model_parallel() creates ALL groups for ALL ranks in a single call.
For example, when creating TP groups with tp_size=2 and world_size=8:

    group_ranks = [[0,1], [2,3], [4,5], [6,7]]  # ALL groups created
    _TP = init_model_parallel_group(group_ranks, local_rank, ...)

ALL ranks call this function and get the same complete group structure. Each rank
then figures out which specific group(s) it belongs to.

Our tests:
1. Mock the distributed backend (no real GPUs needed)
2. Mock init_model_parallel_group to capture the group_ranks parameter
3. Call the real initialize_model_parallel()
4. Verify group_ranks contains the expected complete group structure

We only need to simulate rank 0 because we're testing the group creation logic,
not the per-rank group membership logic.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 37-44: module imports and dependencies / 模块导入与依赖
```python
from __future__ import annotations

import sys
from unittest.mock import Mock, patch

import pytest

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `__future__`, `sys`, `unittest.mock`, `pytest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `__future__`, `sys`, `unittest.mock`, `pytest`。

### Lines 46-50: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=8, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=8, suite="stage-b-test-1-gpu-small-amd")

# Import the actual parallel_state module
parallel_state = pytest.importorskip("sglang.srt.distributed.parallel_state")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci, importorskip.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci, importorskip 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 53-132: test case parallel group construction tp8 attn cp2 (part 1/2) / 测试用例 parallel group construction tp8 attn cp2（第 1/2 部分）
```python
def test_parallel_group_construction_tp8_attn_cp2():
    """
    Test parallel group construction for 8 GPU configuration with:
    - tensor_model_parallel_size = 8
    - attention_context_model_parallel_size = 2

    Expected groups based on docstring example:
        1 tensor model-parallel group:
            [g0, g1, g2, g3, g4, g5, g6, g7]
        4 attention context-parallel groups:
            [g0, g4], [g1, g5], [g2, g6], [g3, g7]

    This test calls the ACTUAL initialize_model_parallel() and verifies the groups.

    Note: We simulate only rank 0 here, but initialize_model_parallel() creates
    ALL groups for ALL ranks in a single call. We capture these groups via mocking
    and verify the complete group structure.
    """
    world_size = 8

    # Mock the distributed backend
    # Note: get_rank() returns 0 because we're testing from a single process,
    # but initialize_model_parallel() still creates all groups for all ranks
    with (
        patch.object(parallel_state, "_WORLD", None),
        patch.object(parallel_state, "_TP", None),
        patch.object(parallel_state, "_ATTN_CP", None),
        patch.object(parallel_state, "_ATTN_TP", None),
        patch.object(parallel_state, "_PP", None),
        patch("torch.distributed.is_initialized", return_value=True),
        patch("torch.distributed.get_world_size", return_value=world_size),
        patch("torch.distributed.get_rank", return_value=0),
        patch("torch.distributed.get_backend", return_value="nccl"),
    ):

        # Mock init_model_parallel_group to capture the groups being created
        created_groups = {}

        def mock_init_model_parallel_group(group_ranks, local_rank, backend, **kwargs):
            group_name = kwargs.get("group_name", "unknown")
            created_groups[group_name] = group_ranks

            # Create a mock group object
            mock_group = Mock()
            mock_group.device_group = Mock()
            return mock_group

        with (
            patch.object(
                parallel_state,
                "init_model_parallel_group",
                side_effect=mock_init_model_parallel_group,
            ),
            patch.object(parallel_state, "get_world_group") as mock_world_group,
        ):

            # Mock world group
            mock_world = Mock()
            mock_world.device_group = Mock()
            mock_world.local_rank = 0
            mock_world_group.return_value = mock_world

            # Call the actual function
            parallel_state.initialize_model_parallel(
                tensor_model_parallel_size=8,
                pipeline_model_parallel_size=1,
                attention_context_model_parallel_size=2,
            )

            # Verify TP groups
            tp_groups = created_groups.get("tp", [])
            assert len(tp_groups) == 1, f"Expected 1 TP group, got {len(tp_groups)}"
            assert tp_groups[0] == [
                0,
                1,
                2,
                3,
                4,
                5,
                6,
```
**EN:** Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - attention_context_model_parallel_size = 2 Expected groups based on docstring example: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 4 attention context-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] This test calls the ACTUAL initialize_model_parallel() and verifies the groups. This test exercises `test_parallel_group_construction_tp8_attn_cp2` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - attention_context_model_parallel_size = 2 Expected groups based on docstring example: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 4 attention context-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] This test calls the ACTUAL initialize_model_parallel() and verifies the groups. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_group_construction_tp8_attn_cp2`。 这一段对应同一逻辑块的第 1 部分。

### Lines 133-154: test case parallel group construction tp8 attn cp2 (part 2/2) / 测试用例 parallel group construction tp8 attn cp2（第 2/2 部分）
```python
                7,
            ], f"Wrong TP group: {tp_groups[0]}"

            # Verify ATTN_CP groups
            attn_cp_groups = created_groups.get("attn_cp", [])
            assert (
                len(attn_cp_groups) == 4
            ), f"Expected 4 ATTN_CP groups, got {len(attn_cp_groups)}"
            expected_attn_cp = [
                [0, 4],
                [1, 5],
                [2, 6],
                [3, 7],
            ]
            assert (
                attn_cp_groups == expected_attn_cp
            ), f"Wrong ATTN_CP groups: {attn_cp_groups}"

            print("TP=8, Attn CP=2 group construction verified")

            # Cleanup
            parallel_state.destroy_model_parallel()
```
**EN:** Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - attention_context_model_parallel_size = 2 Expected groups based on docstring example: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 4 attention context-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] This test calls the ACTUAL initialize_model_parallel() and verifies the groups. This test exercises `test_parallel_group_construction_tp8_attn_cp2` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - attention_context_model_parallel_size = 2 Expected groups based on docstring example: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 4 attention context-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] This test calls the ACTUAL initialize_model_parallel() and verifies the groups. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_group_construction_tp8_attn_cp2`。 这一段对应同一逻辑块的第 2 部分。

### Lines 157-236: test case parallel group construction tp8 moe ep4 cp2 (part 1/2) / 测试用例 parallel group construction tp8 moe ep4 cp2（第 1/2 部分）
```python
def test_parallel_group_construction_tp8_moe_ep4_cp2():
    """
    Test parallel group construction for 8 GPU configuration with:
    - tensor_model_parallel_size = 8
    - expert_model_parallel_size = 4
    - moe_data_model_parallel_size = 2

    Expected groups:
        1 tensor model-parallel group:
            [g0, g1, g2, g3, g4, g5, g6, g7]
        2 MoE expert-parallel groups:
            [g0, g1, g2, g3], [g4, g5, g6, g7]
        4 MoE data-parallel groups:
            [g0, g4], [g1, g5], [g2, g6], [g3, g7]
    """
    world_size = 8

    # Mock the distributed backend
    with (
        patch.object(parallel_state, "_WORLD", None),
        patch.object(parallel_state, "_TP", None),
        patch.object(parallel_state, "_MOE_EP", None),
        patch.object(parallel_state, "_MOE_DP", None),
        patch.object(parallel_state, "_MOE_TP", None),
        patch.object(parallel_state, "_PP", None),
        patch("torch.distributed.is_initialized", return_value=True),
        patch("torch.distributed.get_world_size", return_value=world_size),
        patch("torch.distributed.get_rank", return_value=0),
        patch("torch.distributed.get_backend", return_value="nccl"),
    ):

        # Mock init_model_parallel_group to capture the groups being created
        created_groups = {}

        def mock_init_model_parallel_group(group_ranks, local_rank, backend, **kwargs):
            group_name = kwargs.get("group_name", "unknown")
            created_groups[group_name] = group_ranks

            # Create a mock group object
            mock_group = Mock()
            mock_group.device_group = Mock()
            return mock_group

        with (
            patch.object(
                parallel_state,
                "init_model_parallel_group",
                side_effect=mock_init_model_parallel_group,
            ),
            patch.object(parallel_state, "get_world_group") as mock_world_group,
        ):

            # Mock world group
            mock_world = Mock()
            mock_world.device_group = Mock()
            mock_world.local_rank = 0
            mock_world_group.return_value = mock_world

            # Call the actual function
            parallel_state.initialize_model_parallel(
                tensor_model_parallel_size=8,
                expert_model_parallel_size=4,
                pipeline_model_parallel_size=1,
                moe_data_model_parallel_size=2,
            )

            # Verify TP groups
            tp_groups = created_groups.get("tp", [])
            assert len(tp_groups) == 1, f"Expected 1 TP group, got {len(tp_groups)}"
            assert tp_groups[0] == [
                0,
                1,
                2,
                3,
                4,
                5,
                6,
                7,
            ], f"Wrong TP group: {tp_groups[0]}"

```
**EN:** Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - expert_model_parallel_size = 4 - moe_data_model_parallel_size = 2 Expected groups: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 2 MoE expert-parallel groups: [g0, g1, g2, g3], [g4, g5, g6, g7] 4 MoE data-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] This test exercises `test_parallel_group_construction_tp8_moe_ep4_cp2` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - expert_model_parallel_size = 4 - moe_data_model_parallel_size = 2 Expected groups: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 2 MoE expert-parallel groups: [g0, g1, g2, g3], [g4, g5, g6, g7] 4 MoE data-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_group_construction_tp8_moe_ep4_cp2`。 这一段对应同一逻辑块的第 1 部分。

### Lines 237-268: test case parallel group construction tp8 moe ep4 cp2 (part 2/2) / 测试用例 parallel group construction tp8 moe ep4 cp2（第 2/2 部分）
```python
            # Verify MOE_EP groups
            moe_ep_groups = created_groups.get("moe_ep", [])
            assert (
                len(moe_ep_groups) == 2
            ), f"Expected 2 MOE_EP groups, got {len(moe_ep_groups)}"
            expected_moe_ep = [
                [0, 1, 2, 3],
                [4, 5, 6, 7],
            ]
            assert (
                moe_ep_groups == expected_moe_ep
            ), f"Wrong MOE_EP groups: {moe_ep_groups}"

            # Verify MOE_DP groups
            moe_dp_groups = created_groups.get("moe_dp", [])
            assert (
                len(moe_dp_groups) == 4
            ), f"Expected 4 MOE_DP groups, got {len(moe_dp_groups)}"
            expected_moe_dp = [
                [0, 4],
                [1, 5],
                [2, 6],
                [3, 7],
            ]
            assert (
                moe_dp_groups == expected_moe_dp
            ), f"Wrong MOE_DP groups: {moe_dp_groups}"

            print("TP=8, MoE EP=4, MoE CP=2 group construction verified")

            # Cleanup
            parallel_state.destroy_model_parallel()
```
**EN:** Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - expert_model_parallel_size = 4 - moe_data_model_parallel_size = 2 Expected groups: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 2 MoE expert-parallel groups: [g0, g1, g2, g3], [g4, g5, g6, g7] 4 MoE data-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] This test exercises `test_parallel_group_construction_tp8_moe_ep4_cp2` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - expert_model_parallel_size = 4 - moe_data_model_parallel_size = 2 Expected groups: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 2 MoE expert-parallel groups: [g0, g1, g2, g3], [g4, g5, g6, g7] 4 MoE data-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_group_construction_tp8_moe_ep4_cp2`。 这一段对应同一逻辑块的第 2 部分。

### Lines 271-288: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    # Run tests without requiring GPUs
    import sys

    try:
        test_parallel_group_construction_tp8_attn_cp2()
        test_parallel_group_construction_tp8_moe_ep4_cp2()

        sys.exit(0)
    except AssertionError as e:
        print(f"\n Test failed: {e}")
        sys.exit(1)
    except Exception as e:
        print(f"\n Unexpected error: {e}")
        import traceback

        traceback.print_exc()
        sys.exit(1)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `test_parallel_group_construction_tp8_attn_cp2`: Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - attention_context_model_parallel_size = 2 Expected groups based on docstring example: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 4 attention context-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] This test calls the ACTUAL initialize_model_parallel() and verifies the groups. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_group_construction_tp8_attn_cp2`。
- `test_parallel_group_construction_tp8_moe_ep4_cp2`: Test parallel group construction for 8 GPU configuration with: - tensor_model_parallel_size = 8 - expert_model_parallel_size = 4 - moe_data_model_parallel_size = 2 Expected groups: 1 tensor model-parallel group: [g0, g1, g2, g3, g4, g5, g6, g7] 2 MoE expert-parallel groups: [g0, g1, g2, g3], [g4, g5, g6, g7] 4 MoE data-parallel groups: [g0, g4], [g1, g5], [g2, g6], [g3, g7] / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_group_construction_tp8_moe_ep4_cp2`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `sys`, `unittest.mock`
- **Third-party modules / 第三方模块**: `pytest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 288
