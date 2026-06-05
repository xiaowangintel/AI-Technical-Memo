# test_eplb_algo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/distributed/test_eplb_algo.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Eplb Algo behavior in the Distributed test area through focused pytest scenarios. It focuses on scenarios such as Basic Rebalance, Single GPU Case, Equal Weights. / 该文件在 Distributed 测试域中，通过有针对性的 pytest 场景验证 Eplb Algo 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-9)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import numpy as np
import pytest
import torch

from vllm.distributed.eplb.eplb_state import compute_logical_maps
from vllm.distributed.eplb.policy.default import DefaultEplbPolicy
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `numpy`, `pytest`, `vllm.distributed.eplb.eplb_state`, `vllm.distributed.eplb.policy.default`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_basic_rebalance (lines 12-72)
```python
def test_basic_rebalance():
    """Test basic rebalancing functionality"""
    # Example from https://github.com/deepseek-ai/eplb
    weight = torch.tensor(
        [
            [90, 132, 40, 61, 104, 165, 39, 4, 73, 56, 183, 86],
            [20, 107, 104, 64, 19, 197, 187, 157, 172, 86, 16, 27],
        ]
    )

    num_layers = weight.shape[0]
    num_replicas = 16
    num_groups = 4
    num_nodes = 2
    num_gpus = 8

    phy2log = DefaultEplbPolicy.rebalance_experts(
        weight, num_replicas, num_groups, num_nodes, num_gpus
    )
# ... omitted for brevity ...

    # Verify expected output
    expected_phy2log = torch.tensor(
        [
            [5, 6, 5, 7, 8, 4, 3, 4, 10, 9, 10, 2, 0, 1, 11, 1],
            [7, 10, 6, 8, 6, 11, 8, 9, 2, 4, 5, 1, 5, 0, 3, 1],
        ]
    )
    assert torch.all(phy2log == expected_phy2log)

    expected_logcnt = torch.tensor(
        [[1, 2, 1, 1, 2, 2, 1, 1, 1, 1, 2, 1], [1, 2, 1, 1, 1, 2, 2, 1, 2, 1, 1, 1]]
    )
    assert torch.all(logcnt == expected_logcnt)
```
**EN:** Test basic rebalancing functionality The body exercises logic via `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` before asserting the expected outcome.
**CN:** 该测试用例验证 Basic Rebalance 在特定场景下的行为。 函数体会先通过 `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` 驱动目标逻辑，再断言预期结果。

### Test: test_single_gpu_case (lines 75-95)
```python
def test_single_gpu_case():
    """Test single GPU case"""
    weight = torch.tensor([[10, 20, 30, 40]])
    num_replicas = 4
    num_groups = 1
    num_nodes = 1
    num_gpus = 1

    phy2log = DefaultEplbPolicy.rebalance_experts(
        weight, num_replicas, num_groups, num_nodes, num_gpus
    )
    log2phy, logcnt = compute_logical_maps(phy2log, weight.shape[-1])

    # Verify shapes
    assert phy2log.shape == (1, 4)
    assert log2phy.shape[0] == 1
    assert log2phy.shape[1] == 4
    assert logcnt.shape == (1, 4)

    # Verify all logical experts are mapped
    assert set(phy2log[0].tolist()) == {0, 1, 2, 3}
```
**EN:** Test single GPU case The body exercises logic via `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` before asserting the expected outcome.
**CN:** 该测试用例验证 Single GPU Case 在特定场景下的行为。 函数体会先通过 `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` 驱动目标逻辑，再断言预期结果。

### Test: test_equal_weights (lines 98-119)
```python
def test_equal_weights():
    """Test case with equal weights"""
    weight = torch.tensor([[50, 50, 50, 50, 50, 50, 50, 50]])
    num_replicas = 8
    num_groups = 2
    num_nodes = 2
    num_gpus = 4

    phy2log = DefaultEplbPolicy.rebalance_experts(
        weight, num_replicas, num_groups, num_nodes, num_gpus
    )
    _, logcnt = compute_logical_maps(phy2log, weight.shape[-1])

    # Verify shapes
    assert phy2log.shape == (1, 8)
    assert logcnt.shape == (1, 8)

    # With equal weights, each expert should have exactly one replica
    assert torch.all(logcnt == 1), (
        "With equal weights and no replication, "
        "each expert should have exactly 1 replica"
    )
```
**EN:** Test case with equal weights The body exercises logic via `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` before asserting the expected outcome.
**CN:** 该测试用例验证 Equal Weights 在特定场景下的行为。 函数体会先通过 `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` 驱动目标逻辑，再断言预期结果。

### Test: test_extreme_weight_imbalance (lines 122-142)
```python
def test_extreme_weight_imbalance():
    """Test extreme weight imbalance case"""
    weight = torch.tensor([[1000, 1, 1, 1, 1, 1, 1, 1]])
    num_replicas = 12
    num_groups = 2
    num_nodes = 2
    num_gpus = 4

    phy2log = DefaultEplbPolicy.rebalance_experts(
        weight, num_replicas, num_groups, num_nodes, num_gpus
    )
    _, logcnt = compute_logical_maps(phy2log, weight.shape[-1])

    # Verify shapes
    assert phy2log.shape == (1, 12)
    assert logcnt.shape == (1, 8)

    # Expert with highest weight (index 0) should have more replicas
    assert logcnt[0, 0] > logcnt[0, 1], (
        "Expert with highest weight should have more replicas"
    )
```
**EN:** Test extreme weight imbalance case The body exercises logic via `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` before asserting the expected outcome.
**CN:** 该测试用例验证 Extreme Weight Imbalance 在特定场景下的行为。 函数体会先通过 `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` 驱动目标逻辑，再断言预期结果。

### Test: test_multiple_layers (lines 145-175)
```python
def test_multiple_layers():
    """Test multiple layers case"""
    weight = torch.tensor(
        [
            [10, 20, 30, 40, 50, 60],  # First layer
            [60, 50, 40, 30, 20, 10],  # Second layer (opposite weight pattern)
            [25, 25, 25, 25, 25, 25],  # Third layer (equal weights)
        ]
    )
    num_replicas = 8
    num_groups = 2
    num_nodes = 2
    num_gpus = 4

    phy2log = DefaultEplbPolicy.rebalance_experts(
        weight, num_replicas, num_groups, num_nodes, num_gpus
    )
    _, logcnt = compute_logical_maps(phy2log, weight.shape[-1])

    # Verify shapes
    assert phy2log.shape == (3, 8)
    assert logcnt.shape == (3, 6)

    # Verify expert allocation is reasonable for each layer
    for layer in range(3):
        assert torch.all(phy2log[layer] >= 0) and torch.all(phy2log[layer] < 6), (
            f"Layer {layer} physical to logical mappingshould be in range [0, 6)"
        )
        assert torch.sum(logcnt[layer]) == num_replicas, (
            f"Layer {layer} total replicas should be {num_replicas}"
        )
```
**EN:** Test multiple layers case The body exercises logic via `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` before asserting the expected outcome.
**CN:** 该测试用例验证 Multiple Layers 在特定场景下的行为。 函数体会先通过 `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` 驱动目标逻辑，再断言预期结果。

### Test: test_parameter_validation (lines 178-193)
```python
def test_parameter_validation():
    """Test parameter validation"""
    weight = torch.tensor([[10, 20, 30, 40]])

    # Test non-divisible case - this should handle normally without throwing
    # errors because the function will fall back to global load balancing
    # strategy
    phy2log = DefaultEplbPolicy.rebalance_experts(weight, 8, 3, 2, 4)
    _, logcnt = compute_logical_maps(phy2log, weight.shape[-1])
    assert phy2log.shape == (1, 8)
    assert logcnt.shape == (1, 4)

    # Test cases that will actually cause errors:
    # num_physical_experts not divisible by num_gpus
    with pytest.raises(AssertionError):
        DefaultEplbPolicy.rebalance_experts(weight, 7, 2, 2, 4)  # 7 not divisible by 4
```
**EN:** Test parameter validation The body exercises logic via `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` before asserting the expected outcome.
**CN:** 该测试用例验证 Parameter Validation 在特定场景下的行为。 函数体会先通过 `torch.tensor`, `DefaultEplbPolicy.rebalance_experts`, `compute_logical_maps` 驱动目标逻辑，再断言预期结果。

### Helper: _make_phy_replicas_idx_from_phy2log (lines 360-370)
```python
def _make_phy_replicas_idx_from_phy2log(phy2log: np.ndarray) -> np.ndarray:
    """Create replicas indices mapping from phy2log."""
    pr = np.zeros_like(phy2log, dtype=np.int64)
    for layer in range(phy2log.shape[0]):
        seen: dict[int, int] = {}
        row = phy2log[layer].tolist()
        for i, expert in enumerate(row):
            r = seen.get(expert, 0)
            pr[layer, i] = r
            seen[expert] = r + 1
    return pr
```
**EN:** Create replicas indices mapping from phy2log. It coordinates operations such as `np.zeros_like`, `range`, `phy2log[layer].tolist`.
**CN:** 该辅助函数为 Make Phy Replicas Idx From Phy2log 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `np.zeros_like`, `range`, `phy2log[layer].tolist` 等操作。

### Helper: _validate_intragpu_rearrangement (lines 373-421)
```python
def _validate_intragpu_rearrangement(
    old_global_expert_indices: np.ndarray,
    new_phy2log: np.ndarray,
    new_phy_replicas_idx: np.ndarray,
    post_phy2log: np.ndarray,
    post_phy_replicas_idx: np.ndarray,
    num_ranks: int,
    slots_per_gpu: int,
):
    # Per-GPU checks
    for gpu_idx in range(num_ranks):
        start = gpu_idx * slots_per_gpu
        end = start + slots_per_gpu
        old_seg = old_global_expert_indices[0, start:end]
        new_seg = new_phy2log[0, start:end]
        new_rnk = new_phy_replicas_idx[0, start:end]
        post_seg = post_phy2log[0, start:end]
        post_rnk = post_phy_replicas_idx[0, start:end]

# ... omitted for brevity ...
        new_ranks_for_expert: dict[int, list[int]] = {}
        for v, r in zip(new_list, new_rnk.tolist()):
            new_ranks_for_expert.setdefault(v, []).append(r)
        for expert in remained:
            old_pos = old_list.index(expert)
            assert post_list[old_pos] == expert, (
                f"Expert {expert} on GPU {gpu_idx} should stay at old slot {old_pos}"
            )
            # Rank at preserved slot must be one of the ranks
            # the expert has in new mapping
            assert post_rnk.tolist()[old_pos] in new_ranks_for_expert[expert], (
                f"Rank for expert {expert} at preserved slot on GPU {gpu_idx} "
                "must come from new mapping"
            )
```
**EN:** Implements a reusable helper for Validate Intragpu Rearrangement, reducing duplication across related tests. It coordinates operations such as `range`, `old_seg.tolist`, `new_seg.tolist`.
**CN:** 该辅助函数为 Validate Intragpu Rearrangement 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `range`, `old_seg.tolist`, `new_seg.tolist` 等操作。

### Additional scenarios (summary)
```python
test_small_scale_hierarchical
test_global_load_balance_fallback
test_device_compatibility
test_additional_cases
test_compute_logical_maps_with_negative_indices
Conditional block
test_preserve_intragpu_slots
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `numpy`, `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.distributed.eplb.eplb_state`, `vllm.distributed.eplb.policy.default`
