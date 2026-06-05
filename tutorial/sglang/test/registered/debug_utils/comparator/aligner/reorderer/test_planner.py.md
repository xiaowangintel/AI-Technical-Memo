# test_planner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/reorderer/test_planner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on reorderer planner in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 reorderer planner 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Import dependencies
```python
import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.reorderer.executor import (
    execute_reorderer_plan,
)
from sglang.srt.debug_utils.comparator.aligner.reorderer.planner import (
    compute_reorderer_plans,
)
from sglang.srt.debug_utils.comparator.aligner.reorderer.types import ReordererPlan
from sglang.srt.debug_utils.comparator.aligner.unsharder.executor import (
    execute_unsharder_plan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.planner import (
    compute_unsharder_plan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import AxisInfo
from sglang.srt.debug_utils.comparator.dims_spec import (
    DimSpec,
    ParallelAxis,
    parse_dims,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 27-27: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 30-30: Define class TestComputeReordererPlans
```python
class TestComputeReordererPlans:
```
**EN:** This declaration introduces the `TestComputeReordererPlans` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeReordererPlans` 测试类，并说明它通过继承承担的职责。

### Lines 31-47: Run test: compute reorderer plans zigzag
```python
    def test_compute_reorderer_plans_zigzag(self) -> None:
        """s[cp:zigzag] produces a ReordererPlan."""
        dim_specs = parse_dims("b s[cp:zigzag] h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
        ]
        plans = compute_reorderer_plans(
            dim_specs=dim_specs, parallel_infos=parallel_infos
        )

        assert len(plans) == 1
        assert plans[0].params.op == "zigzag_to_natural"
        assert plans[0].params.dim_name == "s"
        assert plans[0].params.cp_size == 2
```
**EN:** This test method exercises compute reorderer plans zigzag and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 compute reorderer plans zigzag 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 49-68: Run test: compute reorderer plans thd zigzag
```python
    def test_compute_reorderer_plans_thd_zigzag(self) -> None:
        """t[cp:zigzag] produces a ZigzagToNaturalThdParams plan."""
        dim_specs = parse_dims("t[cp:zigzag] h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
        ]
        thd_global_seq_lens: list[int] = [100, 64, 92]
        plans = compute_reorderer_plans(
            dim_specs=dim_specs,
            parallel_infos=parallel_infos,
            thd_global_seq_lens=thd_global_seq_lens,
        )

        assert len(plans) == 1
        assert plans[0].params.op == "zigzag_to_natural_thd"
        assert plans[0].params.cp_size == 2
        assert plans[0].params.seq_lens == [100, 64, 92]
```
**EN:** This test method exercises compute reorderer plans thd zigzag and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 compute reorderer plans thd zigzag 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 70-77: Run test: non seq dim still raises
```python
    def test_non_seq_dim_still_raises(self) -> None:
        """Zigzag on non-sequence/non-token dim (e.g. h[cp:zigzag]) raises ValueError."""
        dim_specs = parse_dims("h[cp:zigzag] d").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2)},
        ]
        with pytest.raises(ValueError, match="only supported on sequence dims"):
            compute_reorderer_plans(dim_specs=dim_specs, parallel_infos=parallel_infos)
```
**EN:** This test method exercises non seq dim still raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 non seq dim still raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 79-89: Run test: thd zigzag without seq lens raises
```python
    def test_thd_zigzag_without_seq_lens_raises(self) -> None:
        """t[cp:zigzag] without thd_global_seq_lens raises ValueError."""
        dim_specs = parse_dims("t[cp:zigzag] h[tp]").dims
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
            {
                ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
            },
        ]
        with pytest.raises(ValueError, match="thd_global_seq_lens is required"):
            compute_reorderer_plans(dim_specs=dim_specs, parallel_infos=parallel_infos)
```
**EN:** This test method exercises thd zigzag without seq lens raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 thd zigzag without seq lens raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 91-104: Run test: thd natural no reorder
```python
    def test_thd_natural_no_reorder(self) -> None:
        """t[cp:natural] and t[cp] produce no reorder plans."""
        for dims_str in ["t[cp:natural] h[tp]", "t[cp] h[tp]"]:
            dim_specs = parse_dims(dims_str).dims
            parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
                {
                    ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                    ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                },
            ]
            plans = compute_reorderer_plans(
                dim_specs=dim_specs, parallel_infos=parallel_infos
            )
            assert plans == []
```
**EN:** This test method exercises thd natural no reorder and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 thd natural no reorder 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 106-119: Run test: compute reorderer plans natural
```python
    def test_compute_reorderer_plans_natural(self) -> None:
        """s[cp] and s[cp:natural] produce no reorder plans."""
        for dims_str in ["b s[cp] h[tp]", "b s[cp:natural] h[tp]"]:
            dim_specs = parse_dims(dims_str).dims
            parallel_infos: list[dict[ParallelAxis, AxisInfo]] = [
                {
                    ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=2),
                    ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
                },
            ]
            plans = compute_reorderer_plans(
                dim_specs=dim_specs, parallel_infos=parallel_infos
            )
            assert plans == []
```
**EN:** This test method exercises compute reorderer plans natural and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 compute reorderer plans natural 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 122-122: Define class TestCpZigzagTpE2E
```python
class TestCpZigzagTpE2E:
```
**EN:** This declaration introduces the `TestCpZigzagTpE2E` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCpZigzagTpE2E` 测试类，并说明它通过继承承担的职责。

### Lines 123-170: Run test: cp zigzag tp e2e
```python
    def test_cp_zigzag_tp_e2e(self) -> None:
        """CP=2 zigzag + TP=2: full pipeline round-trip."""
        torch.manual_seed(42)
        full_tensor = torch.randn(4, 8, 16)

        # Shard: first split seq dim (dim=1) into CP=2 with zigzag ordering,
        # then split hidden dim (dim=2) into TP=2.
        natural_cp_chunks = list(full_tensor.chunk(4, dim=1))
        zigzag_order: list[int] = [0, 3, 1, 2]
        zigzagged = torch.cat([natural_cp_chunks[i] for i in zigzag_order], dim=1)

        cp_chunks = list(zigzagged.chunk(2, dim=1))
        tensors: list[torch.Tensor] = []
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
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

        dim_specs: list[DimSpec] = parse_dims("b s[cp:zigzag] h[tp]").dims
        dim_names: list[str] = [s.name for s in dim_specs]

        unsharder_plans = compute_unsharder_plan(
            dim_specs=dim_specs, parallel_infos=parallel_infos
        )
        reorderer_plans = compute_reorderer_plans(
            dim_specs=dim_specs, parallel_infos=parallel_infos
        )
        all_plans = [*unsharder_plans, *reorderer_plans]

        assert len(unsharder_plans) == 2
        assert len(reorderer_plans) == 1

        current: list[torch.Tensor] = [t.refine_names(*dim_names) for t in tensors]
        for plan in all_plans:
            if isinstance(plan, ReordererPlan):
                current = execute_reorderer_plan(plan, current)
            else:
                current = execute_unsharder_plan(plan, current).tensors

        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises cp zigzag tp e2e and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp zigzag tp e2e 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 173-173: Define class TestCpZigzagSpSameDimE2E
```python
class TestCpZigzagSpSameDimE2E:
```
**EN:** This declaration introduces the `TestCpZigzagSpSameDimE2E` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCpZigzagSpSameDimE2E` 测试类，并说明它通过继承承担的职责。

### Lines 174-174: Document the class `TestCpZigzagSpSameDimE2E`
```python
    """E2E test for t[cp:zigzag,sp] — two axes sharding the same token dim."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestCpZigzagSpSameDimE2E`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestCpZigzagSpSameDimE2E`的设计意图。

### Lines 176-233: Run test: cp2 sp2 zigzag e2e (part 1)
```python
    def test_cp2_sp2_zigzag_e2e(self) -> None:
        """CP=2 zigzag + SP=2 on same token dim: full unshard + reorder round-trip.

        Shard order (outer to inner, matching left-to-right in dims annotation):
          1. CP zigzag splits token dim into 2 CP chunks (zigzag order)
          2. SP splits each CP chunk into 2 SP chunks

        Unshard order (inner to outer, right-to-left):
          1. SP concat (inner): merge SP chunks back
          2. CP concat (outer): merge CP chunks back
          3. Zigzag reorder: restore natural token order
        """
        torch.manual_seed(42)
        total_tokens: int = 16
        hidden: int = 8
        full_tensor: torch.Tensor = torch.randn(total_tokens, hidden)

        # Step 1: CP zigzag split — split into 2*cp_size=4 natural chunks, reorder by zigzag
        cp_size: int = 2
        sp_size: int = 2
        n_natural_chunks: int = cp_size * 2
        natural_chunks: list[torch.Tensor] = list(
            full_tensor.chunk(n_natural_chunks, dim=0)
        )
        zigzag_order: list[int] = [0, 3, 1, 2]
        zigzagged: torch.Tensor = torch.cat(
            [natural_chunks[i] for i in zigzag_order], dim=0
        )
        cp_chunks: list[torch.Tensor] = list(zigzagged.chunk(cp_size, dim=0))

        # Step 2: SP split within each CP chunk
        tensors: list[torch.Tensor] = []
        parallel_infos: list[dict[ParallelAxis, AxisInfo]] = []
        for cp_rank in range(cp_size):
            sp_chunks: list[torch.Tensor] = list(
                cp_chunks[cp_rank].chunk(sp_size, dim=0)
            )
            for sp_rank in range(sp_size):
                tensors.append(sp_chunks[sp_rank])
                parallel_infos.append(
                    {
                        ParallelAxis.CP: AxisInfo(axis_rank=cp_rank, axis_size=cp_size),
                        ParallelAxis.SP: AxisInfo(axis_rank=sp_rank, axis_size=sp_size),
                    }
                )

        dim_specs: list[DimSpec] = parse_dims("t[cp:zigzag,sp] h").dims
        dim_names: list[str] = [s.name for s in dim_specs]

        unsharder_plans = compute_unsharder_plan(
            dim_specs=dim_specs, parallel_infos=parallel_infos
        )
        reorderer_plans = compute_reorderer_plans(
            dim_specs=dim_specs,
            parallel_infos=parallel_infos,
            thd_global_seq_lens=[total_tokens],
        )
        all_plans = [*unsharder_plans, *reorderer_plans]
```
**EN:** This test method exercises cp2 sp2 zigzag e2e and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 cp2 sp2 zigzag e2e 场景，并验证观测到的行为是否符合预期契约。

### Lines 235-245: Run test: cp2 sp2 zigzag e2e (part 2)
```python
        assert len(unsharder_plans) == 2  # SP concat, CP concat
        assert unsharder_plans[0].axis == ParallelAxis.SP
        assert unsharder_plans[1].axis == ParallelAxis.CP
        assert len(reorderer_plans) == 1  # zigzag reorder

        current: list[torch.Tensor] = [t.refine_names(*dim_names) for t in tensors]
        for plan in all_plans:
            if isinstance(plan, ReordererPlan):
                current = execute_reorderer_plan(plan, current)
            else:
                current = execute_unsharder_plan(plan, current).tensors
```
**EN:** This test method exercises cp2 sp2 zigzag e2e and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp2 sp2 zigzag e2e 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 247-248: Run test: cp2 sp2 zigzag e2e (part 3)
```python
        assert len(current) == 1
        assert torch.allclose(current[0].rename(None), full_tensor)
```
**EN:** This test method exercises cp2 sp2 zigzag e2e and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp2 sp2 zigzag e2e 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 251-252: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.reorderer.executor`, `sglang.srt.debug_utils.comparator.aligner.reorderer.planner`, `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.executor`, `sglang.srt.debug_utils.comparator.aligner.unsharder.planner`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
