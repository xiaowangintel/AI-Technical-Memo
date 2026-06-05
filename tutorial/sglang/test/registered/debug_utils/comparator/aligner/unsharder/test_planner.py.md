# test_planner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/unsharder/test_planner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on unsharder planner in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 unsharder planner 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Import dependencies
```python
import sys

import pytest

from sglang.srt.debug_utils.comparator.aligner.unsharder.planner import (
    _compute_dependent_axes,
    _is_dependent_axis,
    _is_jointly_determined,
    _validate_explicit_replicated,
    _validate_replicated_axes_orthogonal,
    compute_unsharder_plan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
    AxisInfo,
    ConcatParams,
    PickParams,
    ReduceSumParams,
)
from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis, parse_dims
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 22-22: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 25-25: Define class TestComputeUnsharderPlan
```python
class TestComputeUnsharderPlan:
```
**EN:** This declaration introduces the `TestComputeUnsharderPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeUnsharderPlan` 测试类，并说明它通过继承承担的职责。

### Lines 26-36: Run test: tp4 plan
```python
    def test_tp4_plan(self) -> None:
        dim_specs = parse_dims("b s h[tp] d").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=4)} for i in range(4)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
        assert plans[0].params.dim_name == "h"
        assert plans[0].groups == [[0, 1, 2, 3]]
```
**EN:** This test method exercises tp4 plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tp4 plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 38-45: Run test: inconsistent axis size raises
```python
    def test_inconsistent_axis_size_raises(self) -> None:
        dim_specs = parse_dims("h[tp]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        with pytest.raises(ValueError, match="Inconsistent axis_size"):
            compute_unsharder_plan(dim_specs, parallel_infos)
```
**EN:** This test method exercises inconsistent axis size raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 inconsistent axis size raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 47-53: Run test: missing axis in all parallel infos skipped
```python
    def test_missing_axis_in_all_parallel_infos_skipped(self) -> None:
        """Axis in dims but absent from all parallel_infos -> axis_size=1, auto-skip.
        But CP is active and undeclared → raises undeclared error."""
        dim_specs = parse_dims("h[tp]").dims
        parallel_infos = [{ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2)}]
        with pytest.raises(ValueError, match="not declared"):
            compute_unsharder_plan(dim_specs, parallel_infos)
```
**EN:** This test method exercises missing axis in all parallel infos skipped and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 missing axis in all parallel infos skipped 场景，并验证观测到的行为是否符合预期契约。

### Lines 55-58: Run test: empty parallel infos raises
```python
    def test_empty_parallel_infos_raises(self) -> None:
        dim_specs = parse_dims("h[tp]").dims
        with pytest.raises(ValueError, match="must not be empty"):
            compute_unsharder_plan(dim_specs, [])
```
**EN:** This test method exercises empty parallel infos raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 empty parallel infos raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 60-71: Run test: scrambled world ranks
```python
    def test_scrambled_world_ranks(self) -> None:
        """world_rank order != axis_rank order."""
        dim_specs = parse_dims("h[tp]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4)},
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1
        assert plans[0].groups == [[1, 3, 0, 2]]
```
**EN:** This test method exercises scrambled world ranks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 scrambled world ranks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 73-77: Run test: no sharded axes returns empty
```python
    def test_no_sharded_axes_returns_empty(self) -> None:
        dim_specs = parse_dims("b s d").dims
        parallel_infos = [{}]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert plans == []
```
**EN:** This test method exercises no sharded axes returns empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no sharded axes returns empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 79-104: Run test: multi axis plan
```python
    def test_multi_axis_plan(self) -> None:
        """Multi-axis (TP + CP) produces a 2-step plan."""
        dim_specs = parse_dims("s[cp] h[tp]").dims
        parallel_infos = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 2
        assert plans[0].axis == ParallelAxis.CP
        assert plans[1].axis == ParallelAxis.TP
```
**EN:** This test method exercises multi axis plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi axis plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 106-132: Run test: cp tp plan
```python
    def test_cp_tp_plan(self) -> None:
        """CP=2 + TP=4 produces correct 2-step plan with correct groups."""
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

        cp_plan = plans[0]
        assert cp_plan.axis == ParallelAxis.CP
        assert len(cp_plan.groups) == 4
        for group in cp_plan.groups:
            assert len(group) == 2

        tp_plan = plans[1]
        assert tp_plan.axis == ParallelAxis.TP
        assert len(tp_plan.groups) == 1
        assert len(tp_plan.groups[0]) == 4
```
**EN:** This test method exercises cp tp plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp tp plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 134-168: Run test: cp tp scrambled ranks
```python
    def test_cp_tp_scrambled_ranks(self) -> None:
        """Scrambled rank assignment still produces correct plan."""
        dim_specs = parse_dims("s[cp] h[tp]").dims
        parallel_infos = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 2

        cp_plan = plans[0]
        assert cp_plan.axis == ParallelAxis.CP
        assert len(cp_plan.groups) == 2
        for group in cp_plan.groups:
            assert len(group) == 2

        tp_plan = plans[1]
        assert tp_plan.axis == ParallelAxis.TP
        assert len(tp_plan.groups) == 1
        assert len(tp_plan.groups[0]) == 2
```
**EN:** This test method exercises cp tp scrambled ranks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp tp scrambled ranks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 170-179: Run test: axis rank coverage incomplete raises
```python
    def test_axis_rank_coverage_incomplete_raises(self) -> None:
        """TP size=4 but only ranks 0,1,3 provided (missing rank 2)."""
        dim_specs = parse_dims("h[tp]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4)},
        ]
        with pytest.raises(ValueError, match="axis_rank coverage.*incomplete"):
            compute_unsharder_plan(dim_specs, parallel_infos)
```
**EN:** This test method exercises axis rank coverage incomplete raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 axis rank coverage incomplete raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 181-191: Run test: reduction partial returns reduce sum
```python
    def test_reduction_partial_returns_reduce_sum(self) -> None:
        dim_specs = parse_dims("h[tp:partial]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
        assert isinstance(plans[0].params, ReduceSumParams)
        assert plans[0].groups == [[0, 1]]
```
**EN:** This test method exercises reduction partial returns reduce sum and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reduction partial returns reduce sum 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 193-203: Run test: reduction partial tp4
```python
    def test_reduction_partial_tp4(self) -> None:
        """TP=4 with partial reduction produces a single ReduceSumParams step."""
        dim_specs = parse_dims("h[tp:partial]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=4)} for i in range(4)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 1
        assert isinstance(plans[0].params, ReduceSumParams)
        assert plans[0].groups == [[0, 1, 2, 3]]
```
**EN:** This test method exercises reduction partial tp4 and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reduction partial tp4 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 205-224: Run test: multi axis with reduction on one
```python
    def test_multi_axis_with_reduction_on_one(self) -> None:
        """CP concat + TP reduce produces a 2-step plan."""
        dim_specs = parse_dims("s[cp] h[tp:partial]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for tp_rank in range(2):
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                    }
                )

        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 2
        assert plans[0].axis == ParallelAxis.CP
        assert isinstance(plans[0].params, ConcatParams)
        assert plans[1].axis == ParallelAxis.TP
        assert isinstance(plans[1].params, ReduceSumParams)
```
**EN:** This test method exercises multi axis with reduction on one and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi axis with reduction on one 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 226-239: Run test: reduction scrambled ranks
```python
    def test_reduction_scrambled_ranks(self) -> None:
        """Scrambled world_rank order with partial reduction."""
        dim_specs = parse_dims("h[tp:partial]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4)},
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 1
        assert isinstance(plans[0].params, ReduceSumParams)
        assert plans[0].groups == [[1, 3, 0, 2]]
```
**EN:** This test method exercises reduction scrambled ranks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reduction scrambled ranks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 241-248: Run test: ordering zigzag accepted
```python
    def test_ordering_zigzag_accepted(self) -> None:
        dim_specs = parse_dims("s[cp:zigzag]").dims
        parallel_infos = [
            {ParallelAxis.CP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.CP
```
**EN:** This test method exercises ordering zigzag accepted and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ordering zigzag accepted 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 250-257: Run test: ordering natural accepted
```python
    def test_ordering_natural_accepted(self) -> None:
        dim_specs = parse_dims("s[cp:natural]").dims
        parallel_infos = [
            {ParallelAxis.CP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.CP
```
**EN:** This test method exercises ordering natural accepted and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ordering natural accepted 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 259-293: Run test: three axis plan
```python
    def test_three_axis_plan(self) -> None:
        """EP=2 + CP=2 + TP=2 produces a 3-step plan."""
        dim_specs = parse_dims("b e[ep] s[cp] h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for ep_rank in range(2):
            for cp_rank in range(2):
                for tp_rank in range(2):
                    parallel_infos.append(
                        {
                            ParallelAxis.EP: AxisInfo(axis_rank=ep_rank, axis_size=2),
                            ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                            ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                        }
                    )

        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 3
        assert plans[0].axis == ParallelAxis.EP
        assert plans[1].axis == ParallelAxis.CP
        assert plans[2].axis == ParallelAxis.TP

        # Step 0 (EP): 8 tensors → 4 (groups of 2)
        assert len(plans[0].groups) == 4
        for group in plans[0].groups:
            assert len(group) == 2

        # Step 1 (CP): 4 tensors → 2 (groups of 2)
        assert len(plans[1].groups) == 2
        for group in plans[1].groups:
            assert len(group) == 2

        # Step 2 (TP): 2 tensors → 1 (single group of 2)
        assert len(plans[2].groups) == 1
        assert len(plans[2].groups[0]) == 2
```
**EN:** This test method exercises three axis plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three axis plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 295-327: Run test: same dim cp sp plan
```python
    def test_same_dim_cp_sp_plan(self) -> None:
        """t[cp:zigzag,sp] with CP=2 SP=2: SP unshards first (inner), then CP."""
        dim_specs = parse_dims("t[cp:zigzag,sp] 1 h").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for sp_rank in range(2):
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.SP: AxisInfo(axis_rank=sp_rank, axis_size=2),
                    }
                )

        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 2

        # SP unshards first (rightmost modifier = innermost shard)
        sp_plan = plans[0]
        assert sp_plan.axis == ParallelAxis.SP
        assert isinstance(sp_plan.params, ConcatParams)
        assert sp_plan.params.dim_name == "t"
        assert len(sp_plan.groups) == 2
        for group in sp_plan.groups:
            assert len(group) == 2

        # CP unshards second (leftmost modifier = outermost shard)
        cp_plan = plans[1]
        assert cp_plan.axis == ParallelAxis.CP
        assert isinstance(cp_plan.params, ConcatParams)
        assert cp_plan.params.dim_name == "t"
        assert len(cp_plan.groups) == 1
        assert len(cp_plan.groups[0]) == 2
```
**EN:** This test method exercises same dim cp sp plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 same dim cp sp plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 329-364: Run test: same dim cp sp with thd
```python
    def test_same_dim_cp_sp_with_thd(self) -> None:
        """t[cp:zigzag,sp] with THD: SP → ConcatParams, CP → CpThdConcatParams."""
        from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
            CpThdConcatParams,
        )

        dim_specs = parse_dims("t[cp:zigzag,sp] h").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for sp_rank in range(2):
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.SP: AxisInfo(axis_rank=sp_rank, axis_size=2),
                    }
                )

        thd_global_seq_lens: list[int] = [100, 64]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, thd_global_seq_lens=thd_global_seq_lens
        )

        assert len(plans) == 2

        # SP unshards first: plain concat (SP is not CP, no THD special handling)
        sp_plan = plans[0]
        assert sp_plan.axis == ParallelAxis.SP
        assert isinstance(sp_plan.params, ConcatParams)
        assert sp_plan.params.dim_name == "t"

        # CP unshards second: THD concat because dim is 't' + axis is CP + thd_global_seq_lens provided
        cp_plan = plans[1]
        assert cp_plan.axis == ParallelAxis.CP
        assert isinstance(cp_plan.params, CpThdConcatParams)
        assert cp_plan.params.dim_name == "t"
        assert cp_plan.params.seq_lens_per_rank == [50, 32]
```
**EN:** This test method exercises same dim cp sp with thd and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 same dim cp sp with thd 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 366-375: Run test: sp in dims but not in parallel info
```python
    def test_sp_in_dims_but_not_in_parallel_info(self) -> None:
        """s[sp] in dims but SP absent from parallel_info (SP disabled), should auto-skip."""
        dim_specs = parse_dims("s[sp] b h[tp]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
```
**EN:** This test method exercises sp in dims but not in parallel info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sp in dims but not in parallel info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 377-382: Run test: all dims sharded but single gpu
```python
    def test_all_dims_sharded_but_single_gpu(self) -> None:
        """Single GPU (TP=1, CP=1), dims has s[cp] h[tp] but parallel_info is empty."""
        dim_specs = parse_dims("b s[cp] h[tp] d").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [{}]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)
        assert plans == []
```
**EN:** This test method exercises all dims sharded but single gpu and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all dims sharded but single gpu 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 384-398: Run test: sharded axis missing from rank raises
```python
    def test_sharded_axis_missing_from_rank_raises(self) -> None:
        """A world_rank missing a sharded axis raises ValueError."""
        dim_specs = parse_dims("s[cp] h[tp]").dims
        parallel_infos = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                # missing TP — sharded axis absent from rank
            },
        ]
        with pytest.raises(ValueError, match="missing parallel_info"):
            compute_unsharder_plan(dim_specs, parallel_infos)
```
**EN:** This test method exercises sharded axis missing from rank raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 sharded axis missing from rank raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 400-423: Run test: tp sharded etp dependent auto resolved
```python
    def test_tp_sharded_etp_dependent_auto_resolved(self) -> None:
        """dims=h[tp], active={TP, ETP, EP}, EP replicated, etp depends on tp → plan succeeds."""
        dim_specs = parse_dims("b h[tp] d # ep:replicated").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for tp_rank in range(2):
            for ep_rank in range(2):
                parallel_infos.append(
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                        ParallelAxis.ETP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                        ParallelAxis.EP: AxisInfo(axis_rank=ep_rank, axis_size=2),
                    }
                )

        plans = compute_unsharder_plan(
            dim_specs,
            parallel_infos,
            explicit_replicated_axes=frozenset({ParallelAxis.EP}),
        )

        axes_in_plan = [p.axis for p in plans]
        assert ParallelAxis.TP in axes_in_plan
        assert ParallelAxis.EP in axes_in_plan
        assert ParallelAxis.ETP not in axes_in_plan
```
**EN:** This test method exercises tp sharded etp dependent auto resolved and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tp sharded etp dependent auto resolved 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 425-465: Run test: edp jointly determined by tp and cp
```python
    def test_edp_jointly_determined_by_tp_and_cp(self) -> None:
        """dims=t[cp:zigzag,sp] h # tp:replicated, EDP determined by (TP,CP) jointly → plan succeeds.

        Simulates tp=2, cp=2, ep=1, etp=1 on 4 GPUs.
        """
        dim_specs = parse_dims("t[cp:zigzag,sp] h # tp:replicated").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=2, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=3, axis_size=4),
            },
        ]
        plans = compute_unsharder_plan(
            dim_specs,
            parallel_infos,
            explicit_replicated_axes=frozenset({ParallelAxis.TP}),
        )
        axes_in_plan = [p.axis for p in plans]
        assert ParallelAxis.CP in axes_in_plan
        assert ParallelAxis.TP in axes_in_plan
        assert ParallelAxis.EDP not in axes_in_plan
```
**EN:** This test method exercises edp jointly determined by tp and cp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 edp jointly determined by tp and cp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 468-468: Define class TestExplicitReplicatedAxes
```python
class TestExplicitReplicatedAxes:
```
**EN:** This declaration introduces the `TestExplicitReplicatedAxes` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExplicitReplicatedAxes` 测试类，并说明它通过继承承担的职责。

### Lines 469-504: Run test: replicated tp with sharded cp
```python
    def test_replicated_tp_with_sharded_cp(self) -> None:
        """CP2 TP2, dims='b s[cp] d # tp:replicated' → PickPlan(TP) + ConcatPlan(CP)."""
        dim_specs = parse_dims("b s[cp] d # tp:replicated").dims
        replicated = frozenset({ParallelAxis.TP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )

        assert len(plans) == 2
        assert plans[0].axis == ParallelAxis.TP
        assert isinstance(plans[0].params, PickParams)
        assert len(plans[0].groups) == 2
        for group in plans[0].groups:
            assert len(group) == 2

        assert plans[1].axis == ParallelAxis.CP
        assert isinstance(plans[1].params, ConcatParams)
        assert plans[1].params.dim_name == "s"
```
**EN:** This test method exercises replicated tp with sharded cp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replicated tp with sharded cp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 506-535: Run test: fully replicated
```python
    def test_fully_replicated(self) -> None:
        """CP2 TP2, dims='b h d # cp:replicated tp:replicated' → PickPlan(CP) + PickPlan(TP)."""
        dim_specs = parse_dims("b h d # cp:replicated tp:replicated").dims
        replicated = frozenset({ParallelAxis.CP, ParallelAxis.TP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )

        assert len(plans) == 2
        assert all(isinstance(p.params, PickParams) for p in plans)
        axes = {p.axis for p in plans}
        assert axes == {ParallelAxis.CP, ParallelAxis.TP}
```
**EN:** This test method exercises fully replicated and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fully replicated 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 537-565: Run test: multiple replicated one sharded
```python
    def test_multiple_replicated_one_sharded(self) -> None:
        """CP2 TP2 EP2, dims='h[tp] # cp:replicated ep:replicated'."""
        dim_specs = parse_dims("h[tp] # cp:replicated ep:replicated").dims
        replicated = frozenset({ParallelAxis.CP, ParallelAxis.EP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for ep_rank in range(2):
                for tp_rank in range(2):
                    parallel_infos.append(
                        {
                            ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                            ParallelAxis.EP: AxisInfo(axis_rank=ep_rank, axis_size=2),
                            ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                        }
                    )

        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )

        assert len(plans) == 3
        pick_plans = [p for p in plans if isinstance(p.params, PickParams)]
        concat_plans = [p for p in plans if isinstance(p.params, ConcatParams)]
        assert len(pick_plans) == 2
        assert len(concat_plans) == 1
        assert concat_plans[0].axis == ParallelAxis.TP

        replicated_axes_in_plan = {p.axis for p in pick_plans}
        assert replicated_axes_in_plan == {ParallelAxis.CP, ParallelAxis.EP}
```
**EN:** This test method exercises multiple replicated one sharded and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple replicated one sharded 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 567-597: Run test: replicated scrambled ranks
```python
    def test_replicated_scrambled_ranks(self) -> None:
        """Scrambled world_rank order with explicit replicated axis."""
        dim_specs = parse_dims("h[tp] # cp:replicated").dims
        replicated = frozenset({ParallelAxis.CP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )

        assert len(plans) == 2
        assert plans[0].axis == ParallelAxis.CP
        assert isinstance(plans[0].params, PickParams)
        assert plans[1].axis == ParallelAxis.TP
        assert isinstance(plans[1].params, ConcatParams)
```
**EN:** This test method exercises replicated scrambled ranks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replicated scrambled ranks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 599-616: Run test: replicated axis inconsistent size raises
```python
    def test_replicated_axis_inconsistent_size_raises(self) -> None:
        """Replicated axis with inconsistent sizes raises ValueError."""
        dim_specs = parse_dims("h[tp] # cp:replicated").dims
        replicated = frozenset({ParallelAxis.CP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=4),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="Inconsistent axis_size"):
            compute_unsharder_plan(
                dim_specs, parallel_infos, explicit_replicated_axes=replicated
            )
```
**EN:** This test method exercises replicated axis inconsistent size raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 replicated axis inconsistent size raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 618-635: Run test: replicated axis missing from rank raises
```python
    def test_replicated_axis_missing_from_rank_raises(self) -> None:
        """A rank missing a replicated axis that other ranks have raises ValueError."""
        dim_specs = parse_dims("h[tp] # cp:replicated").dims
        replicated = frozenset({ParallelAxis.CP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                # missing CP — replicated axis absent from this rank
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="missing parallel_info"):
            compute_unsharder_plan(
                dim_specs, parallel_infos, explicit_replicated_axes=replicated
            )
```
**EN:** This test method exercises replicated axis missing from rank raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 replicated axis missing from rank raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 637-649: Run test: recompute pseudo auto replicated
```python
    def test_recompute_pseudo_auto_replicated(self) -> None:
        """RECOMPUTE_PSEUDO is auto-replicated without explicit declaration."""
        dim_specs = parse_dims("h d").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.RECOMPUTE_PSEUDO: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.RECOMPUTE_PSEUDO: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.RECOMPUTE_PSEUDO
        assert isinstance(plans[0].params, PickParams)
        assert plans[0].groups == [[0, 1]]
```
**EN:** This test method exercises recompute pseudo auto replicated and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 recompute pseudo auto replicated 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 651-666: Run test: recompute pseudo explicit replicated also works
```python
    def test_recompute_pseudo_explicit_replicated_also_works(self) -> None:
        """RECOMPUTE_PSEUDO with explicit # recompute_pseudo:replicated also works."""
        dim_specs = parse_dims("h d # recompute_pseudo:replicated").dims
        replicated = frozenset({ParallelAxis.RECOMPUTE_PSEUDO})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.RECOMPUTE_PSEUDO: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.RECOMPUTE_PSEUDO: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.RECOMPUTE_PSEUDO
        assert isinstance(plans[0].params, PickParams)
        assert plans[0].groups == [[0, 1]]
```
**EN:** This test method exercises recompute pseudo explicit replicated also works and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 recompute pseudo explicit replicated also works 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 668-690: Run test: undeclared active axis raises
```python
    def test_undeclared_active_axis_raises(self) -> None:
        """Active axis not declared as sharded or replicated raises ValueError."""
        dim_specs = parse_dims("b s[cp] d").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="tp.*not declared"):
            compute_unsharder_plan(dim_specs, parallel_infos)
```
**EN:** This test method exercises undeclared active axis raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 undeclared active axis raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 692-702: Run test: replicated not in parallel infos raises
```python
    def test_replicated_not_in_parallel_infos_raises(self) -> None:
        """Declaring replicated axis not in parallel_infos raises ValueError."""
        dim_specs = parse_dims("h[tp] # ep:replicated").dims
        replicated = frozenset({ParallelAxis.EP})
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        with pytest.raises(ValueError, match="not found in parallel_infos"):
            compute_unsharder_plan(
                dim_specs, parallel_infos, explicit_replicated_axes=replicated
            )
```
**EN:** This test method exercises replicated not in parallel infos raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 replicated not in parallel infos raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 704-714: Run test: explicit replicated conflicts with sharded raises
```python
    def test_explicit_replicated_conflicts_with_sharded_raises(self) -> None:
        """Planner-level defense: replicated overlaps sharded → ValueError."""
        dim_specs = parse_dims("h[tp]").dims
        replicated = frozenset({ParallelAxis.TP})
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        with pytest.raises(ValueError, match="both sharded and replicated"):
            compute_unsharder_plan(
                dim_specs, parallel_infos, explicit_replicated_axes=replicated
            )
```
**EN:** This test method exercises explicit replicated conflicts with sharded raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 explicit replicated conflicts with sharded raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 717-717: Define class TestComputeUnsharderPlanFusedDims
```python
class TestComputeUnsharderPlanFusedDims:
```
**EN:** This declaration introduces the `TestComputeUnsharderPlanFusedDims` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeUnsharderPlanFusedDims` 测试类，并说明它通过继承承担的职责。

### Lines 718-730: Run test: fused dim tp2
```python
    def test_fused_dim_tp2(self) -> None:
        """Fused dim "(num_heads*head_dim)[tp]" should unshard on the fused tensor name."""
        dim_specs = parse_dims("t (num_heads*head_dim)[tp]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
        assert isinstance(plans[0].params, ConcatParams)
        assert plans[0].params.dim_name == "num_heads___head_dim"
        assert plans[0].groups == [[0, 1]]
```
**EN:** This test method exercises fused dim tp2 and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused dim tp2 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 732-743: Run test: fused dim modifier on second sub
```python
    def test_fused_dim_modifier_on_second_sub(self) -> None:
        """Modifier on fused dim: "(a*b)[tp]" should produce concat plan."""
        dim_specs = parse_dims("t (a*b)[tp]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
        assert isinstance(plans[0].params, ConcatParams)
        assert plans[0].params.dim_name == "a___b"
```
**EN:** This test method exercises fused dim modifier on second sub and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused dim modifier on second sub 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 745-757: Run test: fused dim no modifier
```python
    def test_fused_dim_no_modifier(self) -> None:
        """Fused dim without modifier + explicit replicated TP → PickParams."""
        dim_specs = parse_dims("t (a*b) # tp:replicated").dims
        replicated = frozenset({ParallelAxis.TP})
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )

        assert len(plans) == 1
        assert isinstance(plans[0].params, PickParams)
```
**EN:** This test method exercises fused dim no modifier and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused dim no modifier 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 759-769: Run test: fused dim with reduction
```python
    def test_fused_dim_with_reduction(self) -> None:
        """Fused dim with partial reduction: "(a*b)[tp:partial]"."""
        dim_specs = parse_dims("t (a*b)[tp:partial]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(dim_specs, parallel_infos)

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
        assert isinstance(plans[0].params, ReduceSumParams)
```
**EN:** This test method exercises fused dim with reduction and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused dim with reduction 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 772-772: Define class TestAxisContainment
```python
class TestAxisContainment:
```
**EN:** This declaration introduces the `TestAxisContainment` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAxisContainment` 测试类，并说明它通过继承承担的职责。

### Lines 773-806: Run test: tp replicated auto resolves dependent axes
```python
    def test_tp_replicated_auto_resolves_dependent_axes(self) -> None:
        """tp:replicated + attn_tp/moe_tp active but undeclared → no error, correct pick."""
        dim_specs = parse_dims("t h # tp:replicated").dims
        replicated = frozenset({ParallelAxis.TP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, explicit_replicated_axes=replicated
        )

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
        assert isinstance(plans[0].params, PickParams)
        assert plans[0].groups == [[0, 1, 2, 3]]
```
**EN:** This test method exercises tp replicated auto resolves dependent axes and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tp replicated auto resolves dependent axes 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 808-833: Run test: independent axis still requires declaration
```python
    def test_independent_axis_still_requires_declaration(self) -> None:
        """cp independent of tp → cp undeclared still raises."""
        dim_specs = parse_dims("t h # tp:replicated").dims
        replicated = frozenset({ParallelAxis.TP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="cp.*not declared"):
            compute_unsharder_plan(
                dim_specs, parallel_infos, explicit_replicated_axes=replicated
            )
```
**EN:** This test method exercises independent axis still requires declaration and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 independent axis still requires declaration 场景，并验证观测到的行为是否符合预期契约。

### Lines 835-868: Run test: backward compat explicit children
```python
    def test_backward_compat_explicit_children(self) -> None:
        """Both tp:replicated and attn_tp:replicated → ValueError (not orthogonal)."""
        dim_specs = parse_dims(
            "t h # tp:replicated attn_tp:replicated moe_tp:replicated"
        ).dims
        replicated = frozenset(
            {ParallelAxis.TP, ParallelAxis.ATTN_TP, ParallelAxis.MOE_TP}
        )
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="not orthogonal"):
            compute_unsharder_plan(
                dim_specs, parallel_infos, explicit_replicated_axes=replicated
            )
```
**EN:** This test method exercises backward compat explicit children and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 backward compat explicit children 场景，并验证观测到的行为是否符合预期契约。

### Lines 871-871: Define class TestDpFilteredAxis
```python
class TestDpFilteredAxis:
```
**EN:** This declaration introduces the `TestDpFilteredAxis` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDpFilteredAxis` 测试类，并说明它通过继承承担的职责。

### Lines 872-873: Document the class `TestDpFilteredAxis`
```python
    """Tests for dp_filtered_axis parameter: DP axis handled by upstream DP filter
    should be excluded from unsharder validation."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDpFilteredAxis`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDpFilteredAxis`的设计意图。

### Lines 875-884: Run test: dp filtered skips undeclared error
```python
    def test_dp_filtered_skips_undeclared_error(self) -> None:
        """DP active but dp_filtered_axis=DP → no error, no DP plan produced."""
        dim_specs = parse_dims("b h d").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2)},
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, dp_filtered_axis=ParallelAxis.DP
        )
        assert plans == []
```
**EN:** This test method exercises dp filtered skips undeclared error and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp filtered skips undeclared error 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 886-905: Run test: dp filtered with sharded tp
```python
    def test_dp_filtered_with_sharded_tp(self) -> None:
        """DP2 + TP2, dims='t h[tp]', dp_filtered_axis=DP → only TP concat plan."""
        dim_specs = parse_dims("t h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, dp_filtered_axis=ParallelAxis.DP
        )

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
        assert isinstance(plans[0].params, ConcatParams)
```
**EN:** This test method exercises dp filtered with sharded tp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp filtered with sharded tp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 907-930: Run test: dp filtered with replicated tp
```python
    def test_dp_filtered_with_replicated_tp(self) -> None:
        """DP2 + TP2, dims='b h # tp:replicated', dp_filtered_axis=DP → only TP pick plan."""
        dim_specs = parse_dims("b h # tp:replicated").dims
        replicated = frozenset({ParallelAxis.TP})
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(
            dim_specs,
            parallel_infos,
            explicit_replicated_axes=replicated,
            dp_filtered_axis=ParallelAxis.DP,
        )

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
        assert isinstance(plans[0].params, PickParams)
```
**EN:** This test method exercises dp filtered with replicated tp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp filtered with replicated tp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 932-947: Run test: dp filtered does not affect other undeclared
```python
    def test_dp_filtered_does_not_affect_other_undeclared(self) -> None:
        """DP filtered + EP active but undeclared (independent of TP) → still raises for EP."""
        dim_specs = parse_dims("t h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EP: AxisInfo(axis_rank=ep_rank, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
            }
            for tp_rank in range(2)
            for ep_rank in range(2)
        ]
        with pytest.raises(ValueError, match="ep.*not declared"):
            compute_unsharder_plan(
                dim_specs, parallel_infos, dp_filtered_axis=ParallelAxis.DP
            )
```
**EN:** This test method exercises dp filtered does not affect other undeclared and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dp filtered does not affect other undeclared 场景，并验证观测到的行为是否符合预期契约。

### Lines 949-961: Run test: dp filtered none still raises for undeclared dp
```python
    def test_dp_filtered_none_still_raises_for_undeclared_dp(self) -> None:
        """Default dp_filtered_axis=None, DP active but undeclared (independent of TP) → raises."""
        dim_specs = parse_dims("t h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.DP: AxisInfo(axis_rank=dp_rank, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
            }
            for tp_rank in range(2)
            for dp_rank in range(2)
        ]
        with pytest.raises(ValueError, match="dp.*not declared"):
            compute_unsharder_plan(dim_specs, parallel_infos)
```
**EN:** This test method exercises dp filtered none still raises for undeclared dp and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dp filtered none still raises for undeclared dp 场景，并验证观测到的行为是否符合预期契约。

### Lines 963-981: Run test: dp filtered custom alias
```python
    def test_dp_filtered_custom_alias(self) -> None:
        """dp_filtered_axis=MOE_DP (custom alias) skips undeclared error for moe_dp."""
        dim_specs = parse_dims("t h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.MOE_DP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.MOE_DP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, dp_filtered_axis=ParallelAxis.MOE_DP
        )

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
```
**EN:** This test method exercises dp filtered custom alias and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp filtered custom alias 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 983-994: Run test: dp filtered not in parallel infos is harmless
```python
    def test_dp_filtered_not_in_parallel_infos_is_harmless(self) -> None:
        """dp_filtered_axis=DP but DP not in parallel_infos → no error, no effect."""
        dim_specs = parse_dims("t h[tp]").dims
        parallel_infos = [
            {ParallelAxis.TP: AxisInfo(axis_rank=i, axis_size=2)} for i in range(2)
        ]
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, dp_filtered_axis=ParallelAxis.DP
        )

        assert len(plans) == 1
        assert plans[0].axis == ParallelAxis.TP
```
**EN:** This test method exercises dp filtered not in parallel infos is harmless and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp filtered not in parallel infos is harmless 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 996-1015: Run test: dp filtered with multi axis sharding
```python
    def test_dp_filtered_with_multi_axis_sharding(self) -> None:
        """DP2 + TP2 + CP2, dims='s[cp] h[tp]', dp_filtered_axis=DP → CP+TP plans only."""
        dim_specs = parse_dims("s[cp] h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for tp_rank in range(2):
                parallel_infos.append(
                    {
                        ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                    }
                )
        plans = compute_unsharder_plan(
            dim_specs, parallel_infos, dp_filtered_axis=ParallelAxis.DP
        )

        assert len(plans) == 2
        assert plans[0].axis == ParallelAxis.CP
        assert plans[1].axis == ParallelAxis.TP
```
**EN:** This test method exercises dp filtered with multi axis sharding and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp filtered with multi axis sharding 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1018-1018: Define class TestIsDependentAxis
```python
class TestIsDependentAxis:
```
**EN:** This declaration introduces the `TestIsDependentAxis` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestIsDependentAxis` 测试类，并说明它通过继承承担的职责。

### Lines 1019-1041: Run test: child determined by parent
```python
    def test_child_determined_by_parent(self) -> None:
        """attn_tp uniquely determined by tp → dependent."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert _is_dependent_axis(
            parallel_infos, parent=ParallelAxis.TP, child=ParallelAxis.ATTN_TP
        )
```
**EN:** This test method exercises child determined by parent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child determined by parent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1043-1065: Run test: child not determined by parent
```python
    def test_child_not_determined_by_parent(self) -> None:
        """dp varies independently of tp → not dependent."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.DP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.DP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert not _is_dependent_axis(
            parallel_infos, parent=ParallelAxis.TP, child=ParallelAxis.DP
        )
```
**EN:** This test method exercises child not determined by parent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child not determined by parent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1067-1075: Run test: parent absent from all infos
```python
    def test_parent_absent_from_all_infos(self) -> None:
        """Parent axis not in any info → vacuously True."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.DP: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        assert _is_dependent_axis(
            parallel_infos, parent=ParallelAxis.TP, child=ParallelAxis.DP
        )
```
**EN:** This test method exercises parent absent from all infos and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parent absent from all infos 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1077-1085: Run test: child absent from all infos
```python
    def test_child_absent_from_all_infos(self) -> None:
        """Child axis not in any info → vacuously True."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        assert _is_dependent_axis(
            parallel_infos, parent=ParallelAxis.TP, child=ParallelAxis.DP
        )
```
**EN:** This test method exercises child absent from all infos and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child absent from all infos 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1087-1097: Run test: single info always dependent
```python
    def test_single_info_always_dependent(self) -> None:
        """With one info entry, any pair is trivially dependent."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
            },
        ]
        assert _is_dependent_axis(
            parallel_infos, parent=ParallelAxis.TP, child=ParallelAxis.DP
        )
```
**EN:** This test method exercises single info always dependent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single info always dependent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1099-1117: Run test: child missing from some infos but consistent
```python
    def test_child_missing_from_some_infos_but_consistent(self) -> None:
        """Child absent from some infos but consistent where present → dependent."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                # ATTN_TP absent here
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
        ]
        assert _is_dependent_axis(
            parallel_infos, parent=ParallelAxis.TP, child=ParallelAxis.ATTN_TP
        )
```
**EN:** This test method exercises child missing from some infos but consistent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child missing from some infos but consistent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1119-1123: Run test: empty parallel infos
```python
    def test_empty_parallel_infos(self) -> None:
        """No infos → vacuously True."""
        assert _is_dependent_axis(
            [], parent=ParallelAxis.TP, child=ParallelAxis.ATTN_TP
        )
```
**EN:** This test method exercises empty parallel infos and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty parallel infos 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1125-1139: Run test: same parent rank different child ranks
```python
    def test_same_parent_rank_different_child_ranks(self) -> None:
        """Explicit conflict: parent_rank=0 maps to child_rank=0 and child_rank=1."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert not _is_dependent_axis(
            parallel_infos, parent=ParallelAxis.TP, child=ParallelAxis.ATTN_TP
        )
```
**EN:** This test method exercises same parent rank different child ranks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 same parent rank different child ranks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1142-1142: Define class TestComputeDependentAxes
```python
class TestComputeDependentAxes:
```
**EN:** This declaration introduces the `TestComputeDependentAxes` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeDependentAxes` 测试类，并说明它通过继承承担的职责。

### Lines 1143-1160: Run test: dependent child found
```python
    def test_dependent_child_found(self) -> None:
        """parent={TP}, candidate={ETP}, etp depends on tp → returns {ETP}."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.ETP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.ETP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        result = _compute_dependent_axes(
            parent_axes={ParallelAxis.TP},
            candidate_axes={ParallelAxis.ETP},
            parallel_infos=parallel_infos,
        )
        assert result == frozenset({ParallelAxis.ETP})
```
**EN:** This test method exercises dependent child found and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dependent child found 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1162-1187: Run test: independent child not found
```python
    def test_independent_child_not_found(self) -> None:
        """parent={TP}, candidate={CP}, cp independent → returns empty."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        result = _compute_dependent_axes(
            parent_axes={ParallelAxis.TP},
            candidate_axes={ParallelAxis.CP},
            parallel_infos=parallel_infos,
        )
        assert result == frozenset()
```
**EN:** This test method exercises independent child not found and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 independent child not found 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1189-1222: Run test: multiple parents
```python
    def test_multiple_parents(self) -> None:
        """parent={TP, EP}, candidate={ETP, MOE_EP}, both dependent → returns both."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.ETP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.MOE_EP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.ETP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.MOE_EP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.ETP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.MOE_EP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.ETP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.MOE_EP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        result = _compute_dependent_axes(
            parent_axes={ParallelAxis.TP, ParallelAxis.EP},
            candidate_axes={ParallelAxis.ETP, ParallelAxis.MOE_EP},
            parallel_infos=parallel_infos,
        )
        assert result == frozenset({ParallelAxis.ETP, ParallelAxis.MOE_EP})
```
**EN:** This test method exercises multiple parents and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple parents 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1225-1225: Define class TestValidateExplicitReplicated
```python
class TestValidateExplicitReplicated:
```
**EN:** This declaration introduces the `TestValidateExplicitReplicated` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestValidateExplicitReplicated` 测试类，并说明它通过继承承担的职责。

### Lines 1226-1238: Run test: valid all axes declared
```python
    def test_valid_all_axes_declared(self) -> None:
        """All axes declared as sharded or replicated → no error."""
        _validate_explicit_replicated(
            explicit_replicated_axes=frozenset({ParallelAxis.CP}),
            sharded_axes={ParallelAxis.TP},
            all_axes={ParallelAxis.TP, ParallelAxis.CP},
            parallel_infos=[
                {
                    ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                    ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                },
            ],
        )
```
**EN:** This test method exercises valid all axes declared and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 valid all axes declared 场景，并验证观测到的行为是否符合预期契约。

### Lines 1240-1250: Run test: replicated not in all axes raises
```python
    def test_replicated_not_in_all_axes_raises(self) -> None:
        """Declaring replicated axis absent from all_axes → ValueError."""
        with pytest.raises(ValueError, match="not found in parallel_infos"):
            _validate_explicit_replicated(
                explicit_replicated_axes=frozenset({ParallelAxis.EP}),
                sharded_axes={ParallelAxis.TP},
                all_axes={ParallelAxis.TP},
                parallel_infos=[
                    {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
                ],
            )
```
**EN:** This test method exercises replicated not in all axes raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 replicated not in all axes raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 1252-1262: Run test: replicated conflicts with sharded raises
```python
    def test_replicated_conflicts_with_sharded_raises(self) -> None:
        """Same axis declared sharded and replicated → ValueError."""
        with pytest.raises(ValueError, match="both sharded and replicated"):
            _validate_explicit_replicated(
                explicit_replicated_axes=frozenset({ParallelAxis.TP}),
                sharded_axes={ParallelAxis.TP},
                all_axes={ParallelAxis.TP},
                parallel_infos=[
                    {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
                ],
            )
```
**EN:** This test method exercises replicated conflicts with sharded raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 replicated conflicts with sharded raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 1264-1289: Run test: undeclared active axis raises
```python
    def test_undeclared_active_axis_raises(self) -> None:
        """Active axis not sharded/replicated/implicitly_replicated → ValueError."""
        with pytest.raises(ValueError, match="dp.*not declared"):
            _validate_explicit_replicated(
                explicit_replicated_axes=frozenset({ParallelAxis.TP}),
                sharded_axes=set(),
                all_axes={ParallelAxis.TP, ParallelAxis.DP},
                parallel_infos=[
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                    },
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                    },
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=1, axis_size=2),
                    },
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=1, axis_size=2),
                    },
                ],
            )
```
**EN:** This test method exercises undeclared active axis raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 undeclared active axis raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 1291-1319: Run test: dependent child implicitly replicated
```python
    def test_dependent_child_implicitly_replicated(self) -> None:
        """Child axis dependent on replicated parent → no error (implicitly replicated)."""
        _validate_explicit_replicated(
            explicit_replicated_axes=frozenset({ParallelAxis.TP}),
            sharded_axes=set(),
            all_axes={ParallelAxis.TP, ParallelAxis.ATTN_TP, ParallelAxis.MOE_TP},
            parallel_infos=[
                {
                    ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4),
                    ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
                    ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
                },
                {
                    ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4),
                    ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
                    ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
                },
                {
                    ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4),
                    ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
                    ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
                },
                {
                    ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4),
                    ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
                    ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
                },
            ],
        )
```
**EN:** This test method exercises dependent child implicitly replicated and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dependent child implicitly replicated 场景，并验证观测到的行为是否符合预期契约。

### Lines 1321-1334: Run test: dp filtered axis excluded from undeclared
```python
    def test_dp_filtered_axis_excluded_from_undeclared(self) -> None:
        """dp_filtered_axis is exempt from undeclared check."""
        _validate_explicit_replicated(
            explicit_replicated_axes=frozenset(),
            sharded_axes={ParallelAxis.TP},
            all_axes={ParallelAxis.TP, ParallelAxis.DP},
            parallel_infos=[
                {
                    ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                    ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                },
            ],
            dp_filtered_axis=ParallelAxis.DP,
        )
```
**EN:** This test method exercises dp filtered axis excluded from undeclared and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dp filtered axis excluded from undeclared 场景，并验证观测到的行为是否符合预期契约。

### Lines 1336-1353: Run test: dp filtered does not exempt other axes
```python
    def test_dp_filtered_does_not_exempt_other_axes(self) -> None:
        """dp_filtered_axis=DP, but EP still undeclared (independent of TP) → raises."""
        with pytest.raises(ValueError, match="ep.*not declared"):
            _validate_explicit_replicated(
                explicit_replicated_axes=frozenset(),
                sharded_axes={ParallelAxis.TP},
                all_axes={ParallelAxis.TP, ParallelAxis.DP, ParallelAxis.EP},
                parallel_infos=[
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                        ParallelAxis.EP: AxisInfo(axis_rank=ep_rank, axis_size=2),
                    }
                    for tp_rank in range(2)
                    for ep_rank in range(2)
                ],
                dp_filtered_axis=ParallelAxis.DP,
            )
```
**EN:** This test method exercises dp filtered does not exempt other axes and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dp filtered does not exempt other axes 场景，并验证观测到的行为是否符合预期契约。

### Lines 1355-1380: Run test: independent child not implicitly replicated
```python
    def test_independent_child_not_implicitly_replicated(self) -> None:
        """Child axis independent of replicated parent → still raises."""
        with pytest.raises(ValueError, match="dp.*not declared"):
            _validate_explicit_replicated(
                explicit_replicated_axes=frozenset({ParallelAxis.TP}),
                sharded_axes=set(),
                all_axes={ParallelAxis.TP, ParallelAxis.DP},
                parallel_infos=[
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                    },
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=0, axis_size=2),
                    },
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=1, axis_size=2),
                    },
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                        ParallelAxis.DP: AxisInfo(axis_rank=1, axis_size=2),
                    },
                ],
            )
```
**EN:** This test method exercises independent child not implicitly replicated and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 independent child not implicitly replicated 场景，并验证观测到的行为是否符合预期契约。

### Lines 1382-1399: Run test: sharded axis determines undeclared implicitly sharded
```python
    def test_sharded_axis_determines_undeclared_implicitly_sharded(self) -> None:
        """TP sharded, ETP dependent on TP → no error (implicitly sharded)."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for tp_rank in range(2):
            for ep_rank in range(2):
                parallel_infos.append(
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                        ParallelAxis.ETP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                        ParallelAxis.EP: AxisInfo(axis_rank=ep_rank, axis_size=2),
                    }
                )
        _validate_explicit_replicated(
            explicit_replicated_axes=frozenset({ParallelAxis.EP}),
            sharded_axes={ParallelAxis.TP},
            all_axes={ParallelAxis.TP, ParallelAxis.ETP, ParallelAxis.EP},
            parallel_infos=parallel_infos,
        )
```
**EN:** This test method exercises sharded axis determines undeclared implicitly sharded and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 sharded axis determines undeclared implicitly sharded 场景，并验证观测到的行为是否符合预期契约。

### Lines 1401-1427: Run test: sharded axis does not resolve independent child
```python
    def test_sharded_axis_does_not_resolve_independent_child(self) -> None:
        """TP sharded, CP active but independent of TP → still raises."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="cp.*not declared"):
            _validate_explicit_replicated(
                explicit_replicated_axes=frozenset(),
                sharded_axes={ParallelAxis.TP},
                all_axes={ParallelAxis.TP, ParallelAxis.CP},
                parallel_infos=parallel_infos,
            )
```
**EN:** This test method exercises sharded axis does not resolve independent child and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 sharded axis does not resolve independent child 场景，并验证观测到的行为是否符合预期契约。

### Lines 1429-1436: Run test: no axes at all
```python
    def test_no_axes_at_all(self) -> None:
        """Empty axes sets → no error."""
        _validate_explicit_replicated(
            explicit_replicated_axes=frozenset(),
            sharded_axes=set(),
            all_axes=set(),
            parallel_infos=[{}],
        )
```
**EN:** This test method exercises no axes at all and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 no axes at all 场景，并验证观测到的行为是否符合预期契约。

### Lines 1438-1480: Run test: jointly determined axis passes
```python
    def test_jointly_determined_axis_passes(self) -> None:
        """EDP determined by (TP, CP) jointly but not by either alone → no error.

        Simulates tp=2, cp=2, ep=1, etp=1 on 4 GPUs where edp_size=4
        and edp_rank = unique per (tp_rank, cp_rank) combination.
        """
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=2, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=3, axis_size=4),
            },
        ]
        _validate_explicit_replicated(
            explicit_replicated_axes=frozenset({ParallelAxis.TP}),
            sharded_axes={ParallelAxis.CP, ParallelAxis.SP},
            all_axes={
                ParallelAxis.TP,
                ParallelAxis.CP,
                ParallelAxis.SP,
                ParallelAxis.EDP,
            },
            parallel_infos=parallel_infos,
        )
```
**EN:** This test method exercises jointly determined axis passes and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 jointly determined axis passes 场景，并验证观测到的行为是否符合预期契约。

### Lines 1482-1502: Run test: jointly undetermined axis still raises
```python
    def test_jointly_undetermined_axis_still_raises(self) -> None:
        """Axis not determined even by the combination of all declared axes → raises.

        DP is orthogonal to TP (each TP rank pairs with both DP ranks),
        so (TP,) cannot determine DP.
        """
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                ParallelAxis.DP: AxisInfo(axis_rank=dp_rank, axis_size=2),
            }
            for tp_rank in range(2)
            for dp_rank in range(2)
        ]
        with pytest.raises(ValueError, match="dp.*not declared"):
            _validate_explicit_replicated(
                explicit_replicated_axes=frozenset(),
                sharded_axes={ParallelAxis.TP},
                all_axes={ParallelAxis.TP, ParallelAxis.DP},
                parallel_infos=parallel_infos,
            )
```
**EN:** This test method exercises jointly undetermined axis still raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 jointly undetermined axis still raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 1505-1505: Define class TestIsJointlyDetermined
```python
class TestIsJointlyDetermined:
```
**EN:** This declaration introduces the `TestIsJointlyDetermined` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestIsJointlyDetermined` 测试类，并说明它通过继承承担的职责。

### Lines 1506-1534: Run test: edp determined by tp and cp
```python
    def test_edp_determined_by_tp_and_cp(self) -> None:
        """EDP rank = unique per (TP, CP) combination → True."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=2, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=3, axis_size=4),
            },
        ]
        assert _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises edp determined by tp and cp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 edp determined by tp and cp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1536-1550: Run test: dp not determined by tp alone
```python
    def test_dp_not_determined_by_tp_alone(self) -> None:
        """DP is orthogonal to TP → False."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=2),
                ParallelAxis.DP: AxisInfo(axis_rank=dp_rank, axis_size=2),
            }
            for tp_rank in range(2)
            for dp_rank in range(2)
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP}),
            child=ParallelAxis.DP,
        )
```
**EN:** This test method exercises dp not determined by tp alone and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp not determined by tp alone 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1552-1558: Run test: empty parallel infos returns false
```python
    def test_empty_parallel_infos_returns_false(self) -> None:
        """No parallel_info entries → False (no evidence)."""
        assert not _is_jointly_determined(
            [],
            parent_axes=frozenset({ParallelAxis.TP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises empty parallel infos returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty parallel infos returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1560-1569: Run test: child absent from infos returns false
```python
    def test_child_absent_from_infos_returns_false(self) -> None:
        """Child axis not present in any info → False."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises child absent from infos returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child absent from infos returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1571-1583: Run test: empty parent axes returns false
```python
    def test_empty_parent_axes_returns_false(self) -> None:
        """Empty parent_axes → False (no parents to determine child)."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset(),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises empty parent axes returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty parent axes returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1585-1601: Run test: single parent determines child
```python
    def test_single_parent_determines_child(self) -> None:
        """Single parent tp_rank uniquely maps to edp_rank → True (degenerate joint case)."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises single parent determines child and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single parent determines child 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1603-1621: Run test: conflict returns false
```python
    def test_conflict_returns_false(self) -> None:
        """Same (tp_rank, cp_rank) maps to different edp_rank → False."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=4),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises conflict returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 conflict returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1623-1651: Run test: two parents jointly determine child
```python
    def test_two_parents_jointly_determine_child(self) -> None:
        """(tp_rank, cp_rank) tuple uniquely determines edp_rank → True."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=2, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=3, axis_size=4),
            },
        ]
        assert _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises two parents jointly determine child and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 two parents jointly determine child 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1653-1670: Run test: three parents jointly determine child
```python
    def test_three_parents_jointly_determine_child(self) -> None:
        """(tp, cp, ep) triple uniquely determines edp → True."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=tp, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=cp, axis_size=2),
                ParallelAxis.EP: AxisInfo(axis_rank=ep, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=tp * 4 + cp * 2 + ep, axis_size=8),
            }
            for tp in range(2)
            for cp in range(2)
            for ep in range(2)
        ]
        assert _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP, ParallelAxis.EP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises three parents jointly determine child and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three parents jointly determine child 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1672-1694: Run test: parent partially absent causes ambiguity
```python
    def test_parent_partially_absent_causes_ambiguity(self) -> None:
        """Some infos lack a parent axis → False, even if child values differ.

        When cp is missing from some infos, the joint determination is
        incomplete because we cannot construct a full parent key.
        """
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=4),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                # cp absent — parent key is incomplete
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=4),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises parent partially absent causes ambiguity and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parent partially absent causes ambiguity 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1696-1714: Run test: partial parent first info missing returns false
```python
    def test_partial_parent_first_info_missing_returns_false(self) -> None:
        """First info lacks a parent axis; second info has all parents → False."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                # cp absent
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises partial parent first info missing returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 partial parent first info missing returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1716-1736: Run test: universally absent parent ignored remaining determines
```python
    def test_universally_absent_parent_ignored_remaining_determines(self) -> None:
        """Parent axis absent from ALL infos is ignored; remaining parent determines child → True.

        Models the real scenario where DP (size 1) is in declared_axes but
        filtered out of all parallel_infos by normalize_parallel_info.
        """
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises universally absent parent ignored remaining determines and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 universally absent parent ignored remaining determines 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1738-1752: Run test: all parents universally absent returns false
```python
    def test_all_parents_universally_absent_returns_false(self) -> None:
        """Every parent axis absent from ALL infos → no active parents → False."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises all parents universally absent returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all parents universally absent returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1754-1770: Run test: universally absent parent remaining conflict returns false
```python
    def test_universally_absent_parent_remaining_conflict_returns_false(self) -> None:
        """Parent axis absent from ALL infos ignored, but remaining parent has conflict → False."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises universally absent parent remaining conflict returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 universally absent parent remaining conflict returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1772-1793: Run test: partial parent matching child still returns false
```python
    def test_partial_parent_matching_child_still_returns_false(self) -> None:
        """Even when child values match across infos, incomplete parent → False.

        Ensures the check is about parent completeness, not child conflict.
        """
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                # cp absent — but edp_rank is SAME as first info
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises partial parent matching child still returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 partial parent matching child still returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1795-1812: Run test: many infos consistent joint mapping
```python
    def test_many_infos_consistent_joint_mapping(self) -> None:
        """8 ranks with (tp, cp) consistently mapping to edp → True."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=tp, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=cp, axis_size=2),
                ParallelAxis.EP: AxisInfo(axis_rank=ep, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=tp * 2 + cp, axis_size=4),
            }
            for tp in range(2)
            for cp in range(2)
            for ep in range(2)
        ]
        assert _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises many infos consistent joint mapping and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 many infos consistent joint mapping 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1814-1837: Run test: partial parent middle info missing returns false
```python
    def test_partial_parent_middle_info_missing_returns_false(self) -> None:
        """Middle info in a 3-info list lacks a parent → False."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=3),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                # cp absent
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=3),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=2, axis_size=3),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises partial parent middle info missing returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 partial parent middle info missing returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1839-1865: Run test: child absent from some infos still true
```python
    def test_child_absent_from_some_infos_still_true(self) -> None:
        """Child absent from some infos but consistent where present → True.

        Infos without the child are skipped; no parent completeness issue.
        """
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                # edp absent — this info is skipped
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises child absent from some infos still true and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child absent from some infos still true 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1867-1877: Run test: child absent from all infos returns false
```python
    def test_child_absent_from_all_infos_returns_false(self) -> None:
        """Child not present in any info → mapping is empty → False."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP}),
            child=ParallelAxis.CP,
        )
```
**EN:** This test method exercises child absent from all infos returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child absent from all infos returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1879-1901: Run test: parent present in some but missing with child returns false
```python
    def test_parent_present_in_some_but_missing_with_child_returns_false(self) -> None:
        """Parent present in some infos but absent in an info that has child.

        This is the potential false-positive scenario: an info has child but
        not all active parents, so the parent key cannot be fully constructed.
        """
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                # TP present in first info so it's active, but absent here
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.EDP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        assert not _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises parent present in some but missing with child returns false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parent present in some but missing with child returns false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1903-1915: Run test: single info with all axes returns true
```python
    def test_single_info_with_all_axes_returns_true(self) -> None:
        """Single info entry with parent and child → trivially determined → True."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=1),
                ParallelAxis.EDP: AxisInfo(axis_rank=0, axis_size=1),
            },
        ]
        assert _is_jointly_determined(
            parallel_infos,
            parent_axes=frozenset({ParallelAxis.TP}),
            child=ParallelAxis.EDP,
        )
```
**EN:** This test method exercises single info with all axes returns true and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single info with all axes returns true 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1918-1918: Define class TestReplicatedAxesOrthogonality
```python
class TestReplicatedAxesOrthogonality:
```
**EN:** This declaration introduces the `TestReplicatedAxesOrthogonality` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestReplicatedAxesOrthogonality` 测试类，并说明它通过继承承担的职责。

### Lines 1919-1920: Document the class `TestReplicatedAxesOrthogonality`
```python
    """Tests for _validate_replicated_axes_orthogonal: every pair of explicitly
    replicated axes must be fully orthogonal (no dependency relationship)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestReplicatedAxesOrthogonality`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestReplicatedAxesOrthogonality`的设计意图。

### Lines 1922-1948: Run test: tp determines moe tp raises
```python
    def test_tp_determines_moe_tp_raises(self) -> None:
        """TP4 + MOE_TP2 where tp_rank determines moe_tp_rank → ValueError."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="not orthogonal"):
            _validate_replicated_axes_orthogonal(
                explicit_replicated_axes=frozenset(
                    {ParallelAxis.TP, ParallelAxis.MOE_TP}
                ),
                parallel_infos=parallel_infos,
            )
```
**EN:** This test method exercises tp determines moe tp raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 tp determines moe tp raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 1950-1966: Run test: tp determines sp identical group raises
```python
    def test_tp_determines_sp_identical_group_raises(self) -> None:
        """TP2 + SP2 where sp_rank == tp_rank → ValueError."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.SP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="not orthogonal"):
            _validate_replicated_axes_orthogonal(
                explicit_replicated_axes=frozenset({ParallelAxis.TP, ParallelAxis.SP}),
                parallel_infos=parallel_infos,
            )
```
**EN:** This test method exercises tp determines sp identical group raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 tp determines sp identical group raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 1968-2001: Run test: three axes two overlapping pairs raises
```python
    def test_three_axes_two_overlapping_pairs_raises(self) -> None:
        """TP4 + ATTN_TP2 + MOE_TP2, TP determines both → error mentions two pairs."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=3, axis_size=4),
                ParallelAxis.ATTN_TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.MOE_TP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="not orthogonal") as exc_info:
            _validate_replicated_axes_orthogonal(
                explicit_replicated_axes=frozenset(
                    {ParallelAxis.TP, ParallelAxis.ATTN_TP, ParallelAxis.MOE_TP}
                ),
                parallel_infos=parallel_infos,
            )
        msg = str(exc_info.value)
        assert "attn_tp" in msg
        assert "moe_tp" in msg
```
**EN:** This test method exercises three axes two overlapping pairs raises and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three axes two overlapping pairs raises 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2003-2026: Run test: three axes one overlap one orthogonal raises
```python
    def test_three_axes_one_overlap_one_orthogonal_raises(self) -> None:
        """TP4 + MOE_TP2 (dependent) + CP2 (independent) → only tp/moe_tp pair errors."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(2):
            for tp_rank in range(4):
                parallel_infos.append(
                    {
                        ParallelAxis.TP: AxisInfo(axis_rank=tp_rank, axis_size=4),
                        ParallelAxis.MOE_TP: AxisInfo(
                            axis_rank=tp_rank % 2, axis_size=2
                        ),
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=2),
                    }
                )
        with pytest.raises(ValueError, match="not orthogonal") as exc_info:
            _validate_replicated_axes_orthogonal(
                explicit_replicated_axes=frozenset(
                    {ParallelAxis.TP, ParallelAxis.MOE_TP, ParallelAxis.CP}
                ),
                parallel_infos=parallel_infos,
            )
        msg = str(exc_info.value)
        assert "moe_tp" in msg
        assert "cp" not in msg
```
**EN:** This test method exercises three axes one overlap one orthogonal raises and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three axes one overlap one orthogonal raises 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2028-2037: Run test: single replicated axis no check
```python
    def test_single_replicated_axis_no_check(self) -> None:
        """Only one replicated axis → no orthogonality check needed, passes."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2)},
            {ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2)},
        ]
        _validate_replicated_axes_orthogonal(
            explicit_replicated_axes=frozenset({ParallelAxis.TP}),
            parallel_infos=parallel_infos,
        )
```
**EN:** This test method exercises single replicated axis no check and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 single replicated axis no check 场景，并验证观测到的行为是否符合预期契约。

### Lines 2039-2062: Run test: two independent axes ok
```python
    def test_two_independent_axes_ok(self) -> None:
        """TP2 + CP2 fully orthogonal → no error."""
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
            },
            {
                ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
                ParallelAxis.CP: AxisInfo(axis_rank=1, axis_size=2),
            },
        ]
        _validate_replicated_axes_orthogonal(
            explicit_replicated_axes=frozenset({ParallelAxis.TP, ParallelAxis.CP}),
            parallel_infos=parallel_infos,
        )
```
**EN:** This test method exercises two independent axes ok and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 two independent axes ok 场景，并验证观测到的行为是否符合预期契约。

### Lines 2065-2066: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.unsharder.planner`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
