# test_eplb_execute.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/distributed/test_eplb_execute.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Eplb Execute behavior in the Distributed test area through focused pytest scenarios. It focuses on scenarios such as Create Expert Indices With Redundancy, Create Expert Weights, Create Redundancy Config. / 该文件在 Distributed 测试域中，通过有针对性的 pytest 场景验证 Eplb Execute 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-25)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import random

import pytest
import torch
import torch.distributed

from vllm.config import VllmConfig, set_current_vllm_config
from vllm.distributed.eplb.eplb_communicator import (
    create_eplb_communicator,
    has_nixl,
)
from vllm.distributed.eplb.rebalance_execute import (
    move_from_buffer,
    rearrange_expert_weights_inplace,
    transfer_layer,
)
from vllm.distributed.parallel_state import (
    ensure_model_parallel_initialized,
    get_tp_group,
)

from .eplb_utils import distributed_run, set_env_vars_and_device
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `random`, `pytest`, `torch`, `vllm.config`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: create_expert_indices_with_redundancy (lines 28-62)
```python
def create_expert_indices_with_redundancy(
    num_layers: int,
    num_logical_experts: int,
    total_physical_experts: int,
    redundancy_config: list[int],  # redundancy for each logical expert
) -> torch.Tensor:
    """
    Create expert indices with redundancy.

    Args:
        num_layers: number of layers
        num_logical_experts: number of logical experts
        total_physical_experts: total number of physical experts
        redundancy_config: redundancy for each logical expert

    Returns:
        indices: Shape (num_layers, total_physical_experts)
    """
    assert sum(redundancy_config) == total_physical_experts
# ... omitted for brevity ...
    indices = torch.zeros(num_layers, total_physical_experts, dtype=torch.long)

    for layer in range(num_layers):
        physical_pos = 0
        for logical_expert_id, redundancy in enumerate(redundancy_config):
            for _ in range(redundancy):
                indices[layer, physical_pos] = logical_expert_id
                physical_pos += 1

    # Shuffle the indices at dim 1
    for layer in range(num_layers):
        indices[layer] = indices[layer][torch.randperm(indices.shape[1])]

    return indices
```
**EN:** Create expert indices with redundancy. It coordinates operations such as `torch.zeros`, `range`, `sum`.
**CN:** 该辅助函数为 Create Expert Indices With Redundancy 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.zeros`, `range`, `sum` 等操作。

### Helper: create_expert_weights (lines 65-114)
```python
def create_expert_weights(
    num_layers: int,
    num_local_experts: int,
    hidden_sizes: list[int],
    rank: int,
    device: torch.device,
    physical_to_logical_mapping: torch.Tensor,
) -> list[list[torch.Tensor]]:
    """
    Create fake expert weights tensor for testing.

    Use `arange` to generate predictable weights values, based on logical
    expert ID.
    All replicas of the same logical expert should have the same weights.

    Args:
        physical_to_logical_mapping: Shape (num_layers, num_local_experts)
            mapping[layer, physical_pos] = logical_expert_id
    """
# ... omitted for brevity ...
                # (so that all replicas of the same logical expert have the
                # same weights)
                base_value = logical_expert_id * 1000 + layer * 100 + weight_idx * 10
                weight_tensor[local_expert] = torch.arange(
                    base_value,
                    base_value + hidden_size,
                    device=device,
                    dtype=torch.float32,
                )

            layer_weights.append(weight_tensor)
        expert_weights.append(layer_weights)

    return expert_weights
```
**EN:** Create fake expert weights tensor for testing. It coordinates operations such as `range`, `enumerate`, `expert_weights.append`.
**CN:** 该辅助函数为 Create Expert Weights 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `range`, `enumerate`, `expert_weights.append` 等操作。

### Test: test_rearrange_expert_weights_with_redundancy (lines 505-551)
```python
@pytest.mark.parametrize(
    "world_size,num_layers,num_local_experts,num_logical_experts",
    [
        # 2 GPU, 2 experts per GPU
        # 3 logical experts, 4 physical experts, 1 redundant experts
        (2, 1, 2, 3),
        # 2 GPU, 3 experts per GPU
        # 4 logical experts, 6 physical experts, 2 redundant experts
        (2, 2, 3, 4),
        # 2 GPU, 8 experts per GPU
        # 16 logical experts, 16 physical experts, 0 redundant experts
        (2, 4, 8, 16),
        # 4 GPU, 2 experts per GPU
        # 6 logical experts, 8 physical experts, 2 redundant experts
        (4, 1, 2, 6),
        # 4 GPU, 2 experts per GPU
        # 5 logical experts, 8 physical experts, 3 redundant experts
        (4, 2, 2, 5),
        # 4 GPU, 8 experts per GPU
# ... omitted for brevity ...
    """Test the functionality of rearranging expert weights with redundancy."""

    if eplb_communicator == "nixl" and not has_nixl():
        pytest.skip("NIXL is not available")
    if torch.accelerator.device_count() < world_size:
        pytest.skip(f"Need at least {world_size} GPUs to run the test")
    distributed_run(
        _test_rearrange_expert_weights_with_redundancy,
        world_size,
        num_layers,
        num_local_experts,
        num_logical_experts,
        eplb_communicator,
    )
```
**EN:** Test the functionality of rearranging expert weights with redundancy. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `distributed_run`, `pytest.skip` before asserting the expected outcome.
**CN:** 该测试用例验证 Rearrange Expert Weights With Redundancy 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `distributed_run`, `pytest.skip` 驱动目标逻辑，再断言预期结果。

### Test: test_async_transfer_layer_without_mtp (lines 632-662)
```python
@pytest.mark.parametrize(
    "world_size,num_layers,num_local_experts,num_logical_experts",
    [
        (2, 2, 2, 3),
    ],
)
@pytest.mark.parametrize(
    "eplb_communicator", ["torch_nccl", "torch_gloo", "pynccl", "nixl"]
)
def test_async_transfer_layer_without_mtp(
    world_size: int,
    num_layers: int,
    num_local_experts: int,
    num_logical_experts: int,
    eplb_communicator: str,
):
    """Exercise async EPLB transfer path without MTP/spec decode."""

    if eplb_communicator == "nixl" and not has_nixl():
        pytest.skip("NIXL is not available")
    if torch.accelerator.device_count() < world_size:
        pytest.skip(f"Need at least {world_size} GPUs to run the test")

    distributed_run(
        _test_async_transfer_layer_without_mtp_worker,
        world_size,
        num_layers,
        num_local_experts,
        num_logical_experts,
        eplb_communicator,
    )
```
**EN:** Exercise async EPLB transfer path without MTP/spec decode. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `distributed_run`, `pytest.skip` before asserting the expected outcome.
**CN:** 该测试用例验证 Async Transfer Layer Without Mtp 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `distributed_run`, `pytest.skip` 驱动目标逻辑，再断言预期结果。

### Test: test_rearrange_expert_weights_no_change (lines 665-677)
```python
@pytest.mark.parametrize("world_size", [2, 4])
def test_rearrange_expert_weights_no_change(world_size):
    """
    Test that when the indices do not change, the weights should remain
    unchanged.
    """

    if torch.accelerator.device_count() < world_size:
        pytest.skip(f"Need at least {world_size} GPUs to run the test")
    distributed_run(
        _test_rearrange_expert_weights_no_change,
        world_size,
    )
```
**EN:** Test that when the indices do not change, the weights should remain unchanged. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `distributed_run`, `torch.accelerator.device_count` before asserting the expected outcome.
**CN:** 该测试用例验证 Rearrange Expert Weights No Change 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `distributed_run`, `torch.accelerator.device_count` 驱动目标逻辑，再断言预期结果。

### Test: test_rearrange_expert_weights_profile_mode (lines 765-774)
```python
@pytest.mark.parametrize("world_size", [2, 4])
def test_rearrange_expert_weights_profile_mode(world_size):
    """Test profile mode (should not copy actual weights)"""

    if torch.accelerator.device_count() < world_size:
        pytest.skip(f"Need at least {world_size} GPUs to run the test")
    distributed_run(
        _test_rearrange_expert_weights_profile_mode,
        world_size,
    )
```
**EN:** Test profile mode (should not copy actual weights) Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `distributed_run`, `torch.accelerator.device_count` before asserting the expected outcome.
**CN:** 该测试用例验证 Rearrange Expert Weights Profile Mode 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `distributed_run`, `torch.accelerator.device_count` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
create_redundancy_config
verify_expert_weights_after_shuffle
verify_redundant_experts_have_same_weights
assert_verification_synced
create_eplb_communicator_or_raise
_test_async_transfer_layer_without_mtp_worker
_test_rearrange_expert_weights_with_redundancy
_test_rearrange_expert_weights_no_change
_test_rearrange_expert_weights_profile_mode
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
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`
- **Third-party / 第三方依赖**: `pytest`, `torch`, `torch.distributed`
- **vLLM internal / vLLM 内部依赖**: `vllm.config`, `vllm.distributed.eplb.eplb_communicator`, `vllm.distributed.eplb.rebalance_execute`, `vllm.distributed.parallel_state`
- **Local test utilities / 本地测试辅助**: `.eplb_utils`
