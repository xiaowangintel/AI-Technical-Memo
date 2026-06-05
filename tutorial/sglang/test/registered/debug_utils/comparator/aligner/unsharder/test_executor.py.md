# test_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/unsharder/test_executor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on unsharder executor in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 unsharder executor 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Import dependencies
```python
import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.unsharder.executor import (
    UnsharderResult,
    _apply_unshard,
    _verify_replicated_group,
    execute_unsharder_plan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.planner import (
    compute_unsharder_plan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
    AxisInfo,
    CpThdConcatParams,
    PickParams,
    ReduceSumParams,
    UnsharderPlan,
)
from sglang.srt.debug_utils.comparator.dims_spec import (
    DimSpec,
    ParallelAxis,
    parse_dims,
)
from sglang.srt.debug_utils.comparator.output_types import ReplicatedCheckResult
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 30-30: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 33-37: Define helper: name tensors
```python
def _name_tensors(
    tensors: list[torch.Tensor], dim_specs: list[DimSpec]
) -> list[torch.Tensor]:
    names: list[str] = [s.sanitized_name for s in dim_specs]
    return [t.refine_names(*names) for t in tensors]
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 40-40: Define class TestExecuteUnsharderPlan
```python
class TestExecuteUnsharderPlan:
```
**EN:** This declaration introduces the `TestExecuteUnsharderPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteUnsharderPlan` 测试类，并说明它通过继承承担的职责。

### Lines 41-58: Run test: tp4 concat
```python
    def test_tp4_concat(self) -> None:
        full_tensor = torch.randn(2, 8, 16)
        shards = list(full_tensor.chunk(4, dim=1))

        dim_specs = parse_dims("b h[tp] d").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=4)} for i in range(4)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1

        named_shards: list[torch.Tensor] = _name_tensors(shards, dim_specs)
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_shards
        )
        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
        assert unsharder_result.replicated_checks == []
```
**EN:** This test method exercises tp4 concat and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tp4 concat 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 60-89: Run test: scrambled world ranks correct result
```python
    def test_scrambled_world_ranks_correct_result(self) -> None:
        full_tensor = torch.randn(4, 8)
        shards = list(full_tensor.chunk(4, dim=0))

        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4)},
        ]
        dim_specs = parse_dims("h[tp] d").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1

        tensors_ordered_by_world_rank = _name_tensors(
            [
                shards[2],  # world_rank=0, axis_rank=2
                shards[0],  # world_rank=1, axis_rank=0
                shards[3],  # world_rank=2, axis_rank=3
                shards[1],  # world_rank=3, axis_rank=1
            ],
            dim_specs,
        )

        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], tensors_ordered_by_world_rank
        )
        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
        assert unsharder_result.replicated_checks == []
```
**EN:** This test method exercises scrambled world ranks correct result and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 scrambled world ranks correct result 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 91-127: Run test: single step reduces tensor count
```python
    def test_single_step_reduces_tensor_count(self) -> None:
        """8 tensors with 2 groups of 4 produce 2 output tensors."""
        full_a = torch.randn(4, 8)
        full_b = torch.randn(4, 8)
        shards_a = list(full_a.chunk(4, dim=0))
        shards_b = list(full_b.chunk(4, dim=0))

        dim_specs = parse_dims("s[cp] h[tp]").dims
        parallel_infos = []
        for cp_rank in range(2):
            for tp_rank in range(4):
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=4),
                    }
                )

        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 2

        tensors: list[torch.Tensor] = []
        for cp_rank in range(2):
            source = shards_a if cp_rank == 0 else shards_b
            for tp_rank in range(4):
                tensors.append(source[tp_rank])

        named_tensors: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        intermediate_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_tensors
        )
        assert len(intermediate_result.tensors) == 4

        final_result: UnsharderResult = execute_unsharder_plan(
            plans[1], intermediate_result.tensors
        )
        assert len(final_result.tensors) == 1
```
**EN:** This test method exercises single step reduces tensor count and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single step reduces tensor count 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 129-158: Run test: cp tp concat
```python
    def test_cp_tp_concat(self) -> None:
        """CP=2 + TP=2: multi-step unshard reconstructs original tensor."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16)

        cp_chunks = list(full_tensor.chunk(2, dim=1))
        tensors: list[torch.Tensor] = []
        parallel_infos = []
        for cp_rank in range(2):
            tp_chunks = list(cp_chunks[cp_rank].chunk(2, dim=2))
            for tp_rank in range(2):
                tensors.append(tp_chunks[tp_rank])
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                    }
                )

        dim_specs = parse_dims("b s[cp] h[tp]").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 2

        current: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        for plan in plans:
            unsharder_result: UnsharderResult = execute_unsharder_plan(plan, current)
            current = unsharder_result.tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises cp tp concat and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp tp concat 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 160-200: Run test: cp tp scrambled
```python
    def test_cp_tp_scrambled(self) -> None:
        """Scrambled world_ranks for CP=2 + TP=2 still reconstruct correctly."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16)

        cp_chunks = list(full_tensor.chunk(2, dim=1))
        shard_map: dict[tuple[int, int], torch.Tensor] = {}
        for cp_rank in range(2):
            tp_chunks = list(cp_chunks[cp_rank].chunk(2, dim=2))
            for tp_rank in range(2):
                shard_map[(cp_rank, tp_rank)] = tp_chunks[tp_rank]

        scrambled_assignment = [
            (1, 1),  # world_rank=0
            (0, 0),  # world_rank=1
            (1, 0),  # world_rank=2
            (0, 1),  # world_rank=3
        ]

        tensors: list[torch.Tensor] = []
        parallel_infos = []
        for cp_rank, tp_rank in scrambled_assignment:
            tensors.append(shard_map[(cp_rank, tp_rank)])
            parallel_infos.append(
                {
                    ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                    ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                }
            )

        dim_specs = parse_dims("b s[cp] h[tp]").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 2

        current: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        for plan in plans:
            unsharder_result: UnsharderResult = execute_unsharder_plan(plan, current)
            current = unsharder_result.tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises cp tp scrambled and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp tp scrambled 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 202-214: Run test: unsupported params type raises
```python
    def test_unsupported_params_type_raises(self) -> None:
        """_apply_unshard raises ValueError for unknown params type."""

        class _FakeParams:
            pass

        with pytest.raises(ValueError, match="Unsupported unshard"):
            _apply_unshard(
                _FakeParams(),
                [torch.randn(2, 2)],
                axis=ParallelAxis.TP,
                group_index=0,
            )
```
**EN:** This test method exercises unsupported params type raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unsupported params type raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 216-254: Run test: cp tp ep three axis concat
```python
    def test_cp_tp_ep_three_axis_concat(self) -> None:
        """CP=2 + TP=2 + EP=2: three-step unshard reconstructs original tensor."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16, 32)

        ep_chunks = list(full_tensor.chunk(2, dim=1))
        shard_map: dict[tuple[int, int, int], torch.Tensor] = {}
        for ep_rank in range(2):
            cp_chunks = list(ep_chunks[ep_rank].chunk(2, dim=2))
            for cp_rank in range(2):
                tp_chunks = list(cp_chunks[cp_rank].chunk(2, dim=3))
                for tp_rank in range(2):
                    shard_map[(ep_rank, cp_rank, tp_rank)] = tp_chunks[tp_rank]

        tensors: list[torch.Tensor] = []
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for ep_rank in range(2):
            for cp_rank in range(2):
                for tp_rank in range(2):
                    tensors.append(shard_map[(ep_rank, cp_rank, tp_rank)])
                    parallel_infos.append(
                        {
                            ParallelAxis.EP: AxisInfo(axis_rank=ep_rank, axis_size=2),
                            ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                            ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                        }
                    )

        dim_specs = parse_dims("b e[ep] s[cp] h[tp]").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 3

        current: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        for plan in plans:
            unsharder_result: UnsharderResult = execute_unsharder_plan(plan, current)
            current = unsharder_result.tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises cp tp ep three axis concat and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp tp ep three axis concat 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 256-303: Run test: cp tp ep scrambled three axis
```python
    def test_cp_tp_ep_scrambled_three_axis(self) -> None:
        """Scrambled ranks for CP=2 + TP=2 + EP=2 still reconstruct correctly."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16, 32)

        ep_chunks = list(full_tensor.chunk(2, dim=1))
        shard_map: dict[tuple[int, int, int], torch.Tensor] = {}
        for ep_rank in range(2):
            cp_chunks = list(ep_chunks[ep_rank].chunk(2, dim=2))
            for cp_rank in range(2):
                tp_chunks = list(cp_chunks[cp_rank].chunk(2, dim=3))
                for tp_rank in range(2):
                    shard_map[(ep_rank, cp_rank, tp_rank)] = tp_chunks[tp_rank]

        scrambled_assignment = [
            (1, 0, 1),  # world_rank=0
            (0, 1, 0),  # world_rank=1
            (1, 1, 0),  # world_rank=2
            (0, 0, 0),  # world_rank=3
            (0, 1, 1),  # world_rank=4
            (1, 0, 0),  # world_rank=5
            (0, 0, 1),  # world_rank=6
            (1, 1, 1),  # world_rank=7
        ]

        tensors: list[torch.Tensor] = []
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for ep_rank, cp_rank, tp_rank in scrambled_assignment:
            tensors.append(shard_map[(ep_rank, cp_rank, tp_rank)])
            parallel_infos.append(
                {
                    ParallelAxis.EP: AxisInfo(axis_rank=ep_rank, axis_size=2),
                    ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                    ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                }
            )

        dim_specs = parse_dims("b e[ep] s[cp] h[tp]").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 3

        current: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        for plan in plans:
            unsharder_result: UnsharderResult = execute_unsharder_plan(plan, current)
            current = unsharder_result.tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises cp tp ep scrambled three axis and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp tp ep scrambled three axis 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 306-306: Define class TestPickOperation
```python
class TestPickOperation:
```
**EN:** This declaration introduces the `TestPickOperation` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPickOperation` 测试类，并说明它通过继承承担的职责。

### Lines 307-328: Run test: pick single group
```python
    def test_pick_single_group(self) -> None:
        """PickParams picks the first tensor from a single group."""
        tensor = torch.randn(4, 8)
        dim_specs = parse_dims("h d # tp:replicated").dims
        replicated = frozenset({ParallelAxis.TP})
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2)},
        ]

        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )
        assert len(plans) == 1
        assert isinstance(plans[0].params, PickParams)

        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], [tensor, tensor.clone()]
        )
        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), tensor)
        assert all(c.passed for c in unsharder_result.replicated_checks)
```
**EN:** This test method exercises pick single group and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 pick single group 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 330-367: Run test: pick multiple groups
```python
    def test_pick_multiple_groups(self) -> None:
        """PickParams with multiple groups picks one from each."""
        dim_specs = parse_dims("h[tp] # cp:replicated").dims
        replicated = frozenset({ParallelAxis.CP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]

        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )
        pick_plans = [p for p in plans if isinstance(p.params, PickParams)]
        assert len(pick_plans) == 1
        assert pick_plans[0].axis == ParallelAxis.CP

        tensor = torch.randn(4)
        tensors = [tensor.clone() for _ in range(4)]

        unsharder_result: UnsharderResult = execute_unsharder_plan(
            pick_plans[0], tensors
        )
        assert len(unsharder_result.tensors) == 2
        assert all(c.passed for c in unsharder_result.replicated_checks)
```
**EN:** This test method exercises pick multiple groups and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 pick multiple groups 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 369-400: Run test: replicated tp sharded cp e2e
```python
    def test_replicated_tp_sharded_cp_e2e(self) -> None:
        """CP2 TP2, dims='b s[cp] d # tp:replicated': replicated TP pick + sharded CP concat round-trip."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16)
        cp_chunks = list(full_tensor.chunk(2, dim=1))

        tensors: list[torch.Tensor] = []
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for tp_rank in range(2):
                tensors.append(cp_chunks[cp_rank].clone())
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                    }
                )

        dim_specs = parse_dims("b s[cp] d # tp:replicated").dims
        replicated = frozenset({ParallelAxis.TP})
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )
        assert len(plans) == 2

        current: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        for plan in plans:
            unsharder_result: UnsharderResult = execute_unsharder_plan(plan, current)
            current = unsharder_result.tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises replicated tp sharded cp e2e and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replicated tp sharded cp e2e 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 402-433: Run test: fully replicated e2e
```python
    def test_fully_replicated_e2e(self) -> None:
        """CP2 TP2, dims='b h d # cp:replicated tp:replicated': fully replicated -> 2 pick steps -> 1 tensor."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16)

        tensors: list[torch.Tensor] = []
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for tp_rank in range(2):
                tensors.append(full_tensor.clone())
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                    }
                )

        dim_specs = parse_dims("b h d # cp:replicated tp:replicated").dims
        replicated = frozenset({ParallelAxis.CP, ParallelAxis.TP})
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )
        assert len(plans) == 2
        assert all(isinstance(p.params, PickParams) for p in plans)

        current: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        for plan in plans:
            unsharder_result: UnsharderResult = execute_unsharder_plan(plan, current)
            current = unsharder_result.tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises fully replicated e2e and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fully replicated e2e 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 436-436: Define class TestVerifyReplicatedGroup
```python
class TestVerifyReplicatedGroup:
```
**EN:** This declaration introduces the `TestVerifyReplicatedGroup` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestVerifyReplicatedGroup` 测试类，并说明它通过继承承担的职责。

### Lines 437-453: Run test: fails on mismatch
```python
    def test_fails_on_mismatch(self) -> None:
        """_verify_replicated_group returns failed check when replicas differ."""
        tensor_a = torch.ones(4)
        tensor_b = torch.ones(4) + 0.1

        checks: list[ReplicatedCheckResult] = _verify_replicated_group(
            [tensor_a, tensor_b],
            axis=ParallelAxis.TP,
            group_index=0,
        )
        assert len(checks) == 1
        assert checks[0].axis == "tp"
        assert checks[0].group_index == 0
        assert checks[0].compared_index == 1
        assert checks[0].baseline_index == 0
        assert not checks[0].passed
        assert checks[0].diff.max_abs_diff == pytest.approx(0.1, abs=1e-5)
```
**EN:** This test method exercises fails on mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fails on mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 455-465: Run test: passes when identical
```python
    def test_passes_when_identical(self) -> None:
        """_verify_replicated_group returns passed check for identical replicas."""
        tensor = torch.randn(4, 8)

        checks: list[ReplicatedCheckResult] = _verify_replicated_group(
            [tensor, tensor.clone()],
            axis=ParallelAxis.TP,
            group_index=0,
        )
        assert len(checks) == 1
        assert checks[0].passed
```
**EN:** This test method exercises passes when identical and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 passes when identical 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 467-483: Run test: multiple mismatches
```python
    def test_multiple_mismatches(self) -> None:
        """_verify_replicated_group reports each differing replica."""
        baseline = torch.zeros(4)
        other_a = torch.ones(4)
        other_b = torch.ones(4) * 2

        checks: list[ReplicatedCheckResult] = _verify_replicated_group(
            [baseline, other_a, other_b],
            axis=ParallelAxis.CP,
            group_index=1,
        )
        assert len(checks) == 2
        assert checks[0].compared_index == 1
        assert not checks[0].passed
        assert checks[1].compared_index == 2
        assert not checks[1].passed
        assert checks[1].diff.max_abs_diff == pytest.approx(2.0, abs=1e-5)
```
**EN:** This test method exercises multiple mismatches and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple mismatches 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 485-506: Run test: execute returns replicated checks
```python
    def test_execute_returns_replicated_checks(self) -> None:
        """execute_unsharder_plan returns replicated checks for mismatch."""
        dim_specs = parse_dims("h d # tp:replicated").dims
        replicated = frozenset({ParallelAxis.TP})
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )

        tensor_a = torch.zeros(4)
        tensor_b = torch.ones(4)

        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], [tensor_a, tensor_b]
        )
        assert len(unsharder_result.tensors) == 1
        assert len(unsharder_result.replicated_checks) == 1
        assert not unsharder_result.replicated_checks[0].passed
        assert torch.allclose(unsharder_result.tensors[0].rename(None), tensor_a)
```
**EN:** This test method exercises execute returns replicated checks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 execute returns replicated checks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 508-519: Run test: atol boundary within
```python
    def test_atol_boundary_within(self) -> None:
        """Difference exactly at atol (1e-6) -> passed."""
        baseline = torch.zeros(4)
        other = torch.full((4,), 1e-6)

        checks: list[ReplicatedCheckResult] = _verify_replicated_group(
            [baseline, other],
            axis=ParallelAxis.TP,
            group_index=0,
        )
        assert len(checks) == 1
        assert checks[0].passed
```
**EN:** This test method exercises atol boundary within and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 atol boundary within 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 521-533: Run test: atol boundary exceeded
```python
    def test_atol_boundary_exceeded(self) -> None:
        """Difference just above atol (1e-6 + 1e-9) -> failed."""
        baseline = torch.zeros(4)
        other = torch.full((4,), 1e-6 + 1e-9)

        checks: list[ReplicatedCheckResult] = _verify_replicated_group(
            [baseline, other],
            axis=ParallelAxis.TP,
            group_index=0,
        )
        assert len(checks) == 1
        assert not checks[0].passed
        assert checks[0].compared_index == 1
```
**EN:** This test method exercises atol boundary exceeded and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 atol boundary exceeded 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 535-551: Run test: recompute pseudo mismatch
```python
    def test_recompute_pseudo_mismatch(self) -> None:
        """_verify_replicated_group returns failed check for RECOMPUTE_PSEUDO axis mismatch."""
        tensor_a = torch.ones(4)
        tensor_b = torch.ones(4) + 0.1

        checks: list[ReplicatedCheckResult] = _verify_replicated_group(
            [tensor_a, tensor_b],
            axis=ParallelAxis.RECOMPUTE_PSEUDO,
            group_index=0,
        )
        assert len(checks) == 1
        assert checks[0].axis == "recompute_pseudo"
        assert checks[0].group_index == 0
        assert checks[0].compared_index == 1
        assert checks[0].baseline_index == 0
        assert not checks[0].passed
        assert checks[0].diff.max_abs_diff == pytest.approx(0.1, abs=1e-5)
```
**EN:** This test method exercises recompute pseudo mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 recompute pseudo mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 554-554: Define class TestThdCpConcat
```python
class TestThdCpConcat:
```
**EN:** This declaration introduces the `TestThdCpConcat` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestThdCpConcat` 测试类，并说明它通过继承承担的职责。

### Lines 555-569: Run test: single seq
```python
    def test_single_seq(self) -> None:
        """Single seq THD unshard: 2 ranks → per-seq concat."""
        rank0 = torch.tensor([1, 2, 3]).refine_names("t")
        rank1 = torch.tensor([4, 5, 6]).refine_names("t")

        plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=[3]),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, [rank0, rank1])

        assert len(unsharder_result.tensors) == 1
        expected = torch.tensor([1, 2, 3, 4, 5, 6])
        assert torch.equal(unsharder_result.tensors[0].rename(None), expected)
```
**EN:** This test method exercises single seq and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single seq 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 571-600: Run test: multi seq
```python
    def test_multi_seq(self) -> None:
        """Multi-seq THD unshard: 2 ranks, seq_lens=[50, 32, 46]."""
        # rank0: [seqA_r0(50) | seqB_r0(32) | pad_r0(46)]
        # rank1: [seqA_r1(50) | seqB_r1(32) | pad_r1(46)]
        seq_a_r0 = torch.arange(0, 50)
        seq_b_r0 = torch.arange(100, 132)
        pad_r0 = torch.full((46,), -1)
        rank0 = torch.cat([seq_a_r0, seq_b_r0, pad_r0]).refine_names("t")

        seq_a_r1 = torch.arange(50, 100)
        seq_b_r1 = torch.arange(132, 164)
        pad_r1 = torch.full((46,), -2)
        rank1 = torch.cat([seq_a_r1, seq_b_r1, pad_r1]).refine_names("t")

        plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=[50, 32, 46]),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, [rank0, rank1])

        assert len(unsharder_result.tensors) == 1
        unsharded: torch.Tensor = unsharder_result.tensors[0].rename(None)

        # seqA: r0(50) + r1(50) = 100 tokens, values 0..99
        assert torch.equal(unsharded[:100], torch.cat([seq_a_r0, seq_a_r1]))
        # seqB: r0(32) + r1(32) = 64 tokens
        assert torch.equal(unsharded[100:164], torch.cat([seq_b_r0, seq_b_r1]))
        # pad: r0(46) + r1(46) = 92 tokens
        assert torch.equal(unsharded[164:256], torch.cat([pad_r0, pad_r1]))
```
**EN:** This test method exercises multi seq and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi seq 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 602-628: Run test: with hidden dim
```python
    def test_with_hidden_dim(self) -> None:
        """THD unshard with trailing hidden dim: shape [T, H]."""
        torch.manual_seed(42)
        hidden: int = 4
        # rank0: [seqA_r0(3, 4) | seqB_r0(2, 4)]
        # rank1: [seqA_r1(3, 4) | seqB_r1(2, 4)]
        seq_a_r0 = torch.randn(3, hidden)
        seq_b_r0 = torch.randn(2, hidden)
        rank0 = torch.cat([seq_a_r0, seq_b_r0]).refine_names("t", "h")

        seq_a_r1 = torch.randn(3, hidden)
        seq_b_r1 = torch.randn(2, hidden)
        rank1 = torch.cat([seq_a_r1, seq_b_r1]).refine_names("t", "h")

        plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=[3, 2]),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, [rank0, rank1])

        assert len(unsharder_result.tensors) == 1
        unsharded: torch.Tensor = unsharder_result.tensors[0].rename(None)

        assert unsharded.shape == (10, hidden)
        assert torch.equal(unsharded[:6], torch.cat([seq_a_r0, seq_a_r1]))
        assert torch.equal(unsharded[6:10], torch.cat([seq_b_r0, seq_b_r1]))
```
**EN:** This test method exercises with hidden dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with hidden dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 630-661: Run test: with leading batch dim
```python
    def test_with_leading_batch_dim(self) -> None:
        """THD unshard with leading batch dim: shape [B, T, H], t is dim=1."""
        torch.manual_seed(42)
        batch: int = 2
        hidden: int = 4
        # rank0: [seqA_r0(3) | seqB_r0(2)] per batch item
        # rank1: [seqA_r1(3) | seqB_r1(2)] per batch item
        seq_a_r0 = torch.randn(batch, 3, hidden)
        seq_b_r0 = torch.randn(batch, 2, hidden)
        rank0 = torch.cat([seq_a_r0, seq_b_r0], dim=1).refine_names("b", "t", "h")

        seq_a_r1 = torch.randn(batch, 3, hidden)
        seq_b_r1 = torch.randn(batch, 2, hidden)
        rank1 = torch.cat([seq_a_r1, seq_b_r1], dim=1).refine_names("b", "t", "h")

        plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=[3, 2]),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, [rank0, rank1])

        assert len(unsharder_result.tensors) == 1
        unsharded: torch.Tensor = unsharder_result.tensors[0].rename(None)

        assert unsharded.shape == (batch, 10, hidden)
        # seqA: r0(3) + r1(3) = 6 tokens per batch
        assert torch.equal(unsharded[:, :6, :], torch.cat([seq_a_r0, seq_a_r1], dim=1))
        # seqB: r0(2) + r1(2) = 4 tokens per batch
        assert torch.equal(
            unsharded[:, 6:10, :], torch.cat([seq_b_r0, seq_b_r1], dim=1)
        )
```
**EN:** This test method exercises with leading batch dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with leading batch dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 664-664: Define class TestReduceSum
```python
class TestReduceSum:
```
**EN:** This declaration introduces the `TestReduceSum` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestReduceSum` 测试类，并说明它通过继承承担的职责。

### Lines 665-686: Run test: basic tp2 reduce
```python
    def test_basic_tp2_reduce(self) -> None:
        """2 partial tensors sum to full tensor."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8)
        part_a = full_tensor * 0.6
        part_b = full_tensor * 0.4

        dim_specs = parse_dims("h[tp:partial] d").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1
        assert isinstance(plans[0].params, ReduceSumParams)

        named_parts: list[torch.Tensor] = _name_tensors([part_a, part_b], dim_specs)
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_parts
        )

        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
```
**EN:** This test method exercises basic tp2 reduce and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic tp2 reduce 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 688-707: Run test: tp4 reduce
```python
    def test_tp4_reduce(self) -> None:
        """4 partial tensors sum to full tensor."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8)
        parts: list[torch.Tensor] = [full_tensor * 0.25 for _ in range(4)]

        dim_specs = parse_dims("h[tp:partial] d").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=4)} for i in range(4)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1

        named_parts: list[torch.Tensor] = _name_tensors(parts, dim_specs)
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_parts
        )

        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
```
**EN:** This test method exercises tp4 reduce and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tp4 reduce 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 709-738: Run test: multi axis concat then reduce
```python
    def test_multi_axis_concat_then_reduce(self) -> None:
        """CP concat + TP reduce end-to-end."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16)

        cp_chunks = list(full_tensor.chunk(2, dim=1))
        # Each CP chunk is held as partial sums across TP ranks
        tensors: list[torch.Tensor] = []
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for tp_rank in range(2):
                tensors.append(cp_chunks[cp_rank] * 0.5)
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                    }
                )

        dim_specs = parse_dims("b s[cp] h[tp:partial]").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 2

        current: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        for plan in plans:
            unsharder_result: UnsharderResult = execute_unsharder_plan(plan, current)
            current = unsharder_result.tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises multi axis concat then reduce and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi axis concat then reduce 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 740-766: Run test: reduce scrambled ranks
```python
    def test_reduce_scrambled_ranks(self) -> None:
        """Scrambled rank order — sum is commutative so result is the same."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8)
        parts: list[torch.Tensor] = [
            full_tensor * 0.1,
            full_tensor * 0.2,
            full_tensor * 0.3,
            full_tensor * 0.4,
        ]

        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4)},
        ]
        dim_specs = parse_dims("h[tp:partial] d").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        named_parts: list[torch.Tensor] = _name_tensors(parts, dim_specs)
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_parts
        )

        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
```
**EN:** This test method exercises reduce scrambled ranks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reduce scrambled ranks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 768-788: Run test: reduce preserves named dims
```python
    def test_reduce_preserves_named_dims(self) -> None:
        """Named tensor dimensions are preserved through reduce_sum."""
        dim_specs = parse_dims("h[tp:partial] d").dims
        part_a = torch.randn(4, 8).refine_names("h", "d")
        part_b = torch.randn(4, 8).refine_names("h", "d")

        plan = UnsharderPlan(
            axis=ParallelAxis.TP,
            params=ReduceSumParams(),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plan, [part_a, part_b]
        )

        assert len(unsharder_result.tensors) == 1
        assert unsharder_result.tensors[0].names == ("h", "d")
        expected = (part_a.rename(None) + part_b.rename(None)).refine_names("h", "d")
        assert torch.allclose(
            unsharder_result.tensors[0].rename(None), expected.rename(None)
        )
```
**EN:** This test method exercises reduce preserves named dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reduce preserves named dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 790-806: Run test: recompute pseudo mismatch
```python
    def test_recompute_pseudo_mismatch(self) -> None:
        """_verify_replicated_group returns failed check for RECOMPUTE_PSEUDO axis mismatch."""
        tensor_a = torch.ones(4)
        tensor_b = torch.ones(4) + 0.1

        checks: list[ReplicatedCheckResult] = _verify_replicated_group(
            [tensor_a, tensor_b],
            axis=ParallelAxis.RECOMPUTE_PSEUDO,
            group_index=0,
        )
        assert len(checks) == 1
        assert checks[0].axis == "recompute_pseudo"
        assert checks[0].group_index == 0
        assert checks[0].compared_index == 1
        assert checks[0].baseline_index == 0
        assert not checks[0].passed
        assert checks[0].diff.max_abs_diff == pytest.approx(0.1, abs=1e-5)
```
**EN:** This test method exercises recompute pseudo mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 recompute pseudo mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 809-809: Define class TestThdCpConcat
```python
class TestThdCpConcat:
```
**EN:** This declaration introduces the `TestThdCpConcat` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestThdCpConcat` 测试类，并说明它通过继承承担的职责。

### Lines 810-824: Run test: single seq
```python
    def test_single_seq(self) -> None:
        """Single seq THD unshard: 2 ranks → per-seq concat."""
        rank0 = torch.tensor([1, 2, 3]).refine_names("t")
        rank1 = torch.tensor([4, 5, 6]).refine_names("t")

        plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=[3]),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, [rank0, rank1])

        assert len(unsharder_result.tensors) == 1
        expected = torch.tensor([1, 2, 3, 4, 5, 6])
        assert torch.equal(unsharder_result.tensors[0].rename(None), expected)
```
**EN:** This test method exercises single seq and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single seq 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 826-855: Run test: multi seq
```python
    def test_multi_seq(self) -> None:
        """Multi-seq THD unshard: 2 ranks, seq_lens=[50, 32, 46]."""
        # rank0: [seqA_r0(50) | seqB_r0(32) | pad_r0(46)]
        # rank1: [seqA_r1(50) | seqB_r1(32) | pad_r1(46)]
        seq_a_r0 = torch.arange(0, 50)
        seq_b_r0 = torch.arange(100, 132)
        pad_r0 = torch.full((46,), -1)
        rank0 = torch.cat([seq_a_r0, seq_b_r0, pad_r0]).refine_names("t")

        seq_a_r1 = torch.arange(50, 100)
        seq_b_r1 = torch.arange(132, 164)
        pad_r1 = torch.full((46,), -2)
        rank1 = torch.cat([seq_a_r1, seq_b_r1, pad_r1]).refine_names("t")

        plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=[50, 32, 46]),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, [rank0, rank1])

        assert len(unsharder_result.tensors) == 1
        unsharded: torch.Tensor = unsharder_result.tensors[0].rename(None)

        # seqA: r0(50) + r1(50) = 100 tokens, values 0..99
        assert torch.equal(unsharded[:100], torch.cat([seq_a_r0, seq_a_r1]))
        # seqB: r0(32) + r1(32) = 64 tokens
        assert torch.equal(unsharded[100:164], torch.cat([seq_b_r0, seq_b_r1]))
        # pad: r0(46) + r1(46) = 92 tokens
        assert torch.equal(unsharded[164:256], torch.cat([pad_r0, pad_r1]))
```
**EN:** This test method exercises multi seq and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi seq 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 857-883: Run test: with hidden dim
```python
    def test_with_hidden_dim(self) -> None:
        """THD unshard with trailing hidden dim: shape [T, H]."""
        torch.manual_seed(42)
        hidden: int = 4
        # rank0: [seqA_r0(3, 4) | seqB_r0(2, 4)]
        # rank1: [seqA_r1(3, 4) | seqB_r1(2, 4)]
        seq_a_r0 = torch.randn(3, hidden)
        seq_b_r0 = torch.randn(2, hidden)
        rank0 = torch.cat([seq_a_r0, seq_b_r0]).refine_names("t", "h")

        seq_a_r1 = torch.randn(3, hidden)
        seq_b_r1 = torch.randn(2, hidden)
        rank1 = torch.cat([seq_a_r1, seq_b_r1]).refine_names("t", "h")

        plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=[3, 2]),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, [rank0, rank1])

        assert len(unsharder_result.tensors) == 1
        unsharded: torch.Tensor = unsharder_result.tensors[0].rename(None)

        assert unsharded.shape == (10, hidden)
        assert torch.equal(unsharded[:6], torch.cat([seq_a_r0, seq_a_r1]))
        assert torch.equal(unsharded[6:10], torch.cat([seq_b_r0, seq_b_r1]))
```
**EN:** This test method exercises with hidden dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with hidden dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 885-916: Run test: with leading batch dim
```python
    def test_with_leading_batch_dim(self) -> None:
        """THD unshard with leading batch dim: shape [B, T, H], t is dim=1."""
        torch.manual_seed(42)
        batch: int = 2
        hidden: int = 4
        # rank0: [seqA_r0(3) | seqB_r0(2)] per batch item
        # rank1: [seqA_r1(3) | seqB_r1(2)] per batch item
        seq_a_r0 = torch.randn(batch, 3, hidden)
        seq_b_r0 = torch.randn(batch, 2, hidden)
        rank0 = torch.cat([seq_a_r0, seq_b_r0], dim=1).refine_names("b", "t", "h")

        seq_a_r1 = torch.randn(batch, 3, hidden)
        seq_b_r1 = torch.randn(batch, 2, hidden)
        rank1 = torch.cat([seq_a_r1, seq_b_r1], dim=1).refine_names("b", "t", "h")

        plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=[3, 2]),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(plan, [rank0, rank1])

        assert len(unsharder_result.tensors) == 1
        unsharded: torch.Tensor = unsharder_result.tensors[0].rename(None)

        assert unsharded.shape == (batch, 10, hidden)
        # seqA: r0(3) + r1(3) = 6 tokens per batch
        assert torch.equal(unsharded[:, :6, :], torch.cat([seq_a_r0, seq_a_r1], dim=1))
        # seqB: r0(2) + r1(2) = 4 tokens per batch
        assert torch.equal(
            unsharded[:, 6:10, :], torch.cat([seq_b_r0, seq_b_r1], dim=1)
        )
```
**EN:** This test method exercises with leading batch dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with leading batch dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 919-919: Define class TestReduceSum
```python
class TestReduceSum:
```
**EN:** This declaration introduces the `TestReduceSum` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestReduceSum` 测试类，并说明它通过继承承担的职责。

### Lines 920-941: Run test: basic tp2 reduce
```python
    def test_basic_tp2_reduce(self) -> None:
        """2 partial tensors sum to full tensor."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8)
        part_a = full_tensor * 0.6
        part_b = full_tensor * 0.4

        dim_specs = parse_dims("h[tp:partial] d").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1
        assert isinstance(plans[0].params, ReduceSumParams)

        named_parts: list[torch.Tensor] = _name_tensors([part_a, part_b], dim_specs)
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_parts
        )

        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
```
**EN:** This test method exercises basic tp2 reduce and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic tp2 reduce 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 943-962: Run test: tp4 reduce
```python
    def test_tp4_reduce(self) -> None:
        """4 partial tensors sum to full tensor."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8)
        parts: list[torch.Tensor] = [full_tensor * 0.25 for _ in range(4)]

        dim_specs = parse_dims("h[tp:partial] d").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=4)} for i in range(4)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1

        named_parts: list[torch.Tensor] = _name_tensors(parts, dim_specs)
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_parts
        )

        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
```
**EN:** This test method exercises tp4 reduce and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tp4 reduce 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 964-993: Run test: multi axis concat then reduce
```python
    def test_multi_axis_concat_then_reduce(self) -> None:
        """CP concat + TP reduce end-to-end."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16)

        cp_chunks = list(full_tensor.chunk(2, dim=1))
        # Each CP chunk is held as partial sums across TP ranks
        tensors: list[torch.Tensor] = []
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for tp_rank in range(2):
                tensors.append(cp_chunks[cp_rank] * 0.5)
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                    }
                )

        dim_specs = parse_dims("b s[cp] h[tp:partial]").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 2

        current: list[torch.Tensor] = _name_tensors(tensors, dim_specs)
        for plan in plans:
            unsharder_result: UnsharderResult = execute_unsharder_plan(plan, current)
            current = unsharder_result.tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises multi axis concat then reduce and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi axis concat then reduce 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 995-1021: Run test: reduce scrambled ranks
```python
    def test_reduce_scrambled_ranks(self) -> None:
        """Scrambled rank order — sum is commutative so result is the same."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8)
        parts: list[torch.Tensor] = [
            full_tensor * 0.1,
            full_tensor * 0.2,
            full_tensor * 0.3,
            full_tensor * 0.4,
        ]

        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4)},
        ]
        dim_specs = parse_dims("h[tp:partial] d").dims
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        named_parts: list[torch.Tensor] = _name_tensors(parts, dim_specs)
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_parts
        )

        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
```
**EN:** This test method exercises reduce scrambled ranks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reduce scrambled ranks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1023-1043: Run test: reduce preserves named dims
```python
    def test_reduce_preserves_named_dims(self) -> None:
        """Named tensor dimensions are preserved through reduce_sum."""
        dim_specs = parse_dims("h[tp:partial] d").dims
        part_a = torch.randn(4, 8).refine_names("h", "d")
        part_b = torch.randn(4, 8).refine_names("h", "d")

        plan = UnsharderPlan(
            axis=ParallelAxis.TP,
            params=ReduceSumParams(),
            groups=[[0, 1]],
        )
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plan, [part_a, part_b]
        )

        assert len(unsharder_result.tensors) == 1
        assert unsharder_result.tensors[0].names == ("h", "d")
        expected = (part_a.rename(None) + part_b.rename(None)).refine_names("h", "d")
        assert torch.allclose(
            unsharder_result.tensors[0].rename(None), expected.rename(None)
        )
```
**EN:** This test method exercises reduce preserves named dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reduce preserves named dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1046-1046: Define class TestFusedDimExecutor
```python
class TestFusedDimExecutor:
```
**EN:** This declaration introduces the `TestFusedDimExecutor` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFusedDimExecutor` 测试类，并说明它通过继承承担的职责。

### Lines 1047-1067: Run test: fused tp2 concat
```python
    def test_fused_tp2_concat(self) -> None:
        """Fused dim "t (num_heads*head_dim)[tp]": TP=2 concat on fused axis."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 128)  # t=4, nh*hd=128

        shards = list(full_tensor.chunk(2, dim=1))

        dim_specs = parse_dims("t (num_heads*head_dim)[tp]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1

        named_shards: list[torch.Tensor] = _name_tensors(shards, dim_specs)
        unsharder_result: UnsharderResult = execute_unsharder_plan(
            plans[0], named_shards
        )

        assert len(unsharder_result.tensors) == 1
        assert torch.allclose(unsharder_result.tensors[0].rename(None), full_tensor)
```
**EN:** This test method exercises fused tp2 concat and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused tp2 concat 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1070-1071: Expose unittest entrypoint
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.unsharder.executor`, `sglang.srt.debug_utils.comparator.aligner.unsharder.planner`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
