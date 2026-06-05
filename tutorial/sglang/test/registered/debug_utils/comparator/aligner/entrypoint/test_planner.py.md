# test_planner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/entrypoint/test_planner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on entrypoint planner in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 entrypoint planner 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Import dependencies
```python
import sys
from typing import Any, Optional

import pytest

from sglang.srt.debug_utils.comparator.aligner.entrypoint.planner import (
    _compute_per_step_plans,
    compute_aligner_plan,
    compute_per_step_sub_plans,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.types import (
    AlignerPerStepPlan,
    AlignerPerStepSubPlan,
    AlignerPlan,
)
from sglang.srt.debug_utils.comparator.aligner.reorderer.types import (
    ReordererPlan,
    ZigzagToNaturalThdParams,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
    CpThdConcatParams,
    UnsharderPlan,
)
from sglang.srt.debug_utils.comparator.dims_spec import TokenLayout
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 28-28: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 31-53: Define helper: make meta
```python
def _make_meta(
    *,
    step: int = 0,
    dims: Optional[str] = None,
    tp_rank: int = 0,
    tp_size: int = 1,
    cp_rank: int = 0,
    cp_size: int = 1,
    extra_parallel_info: Optional[dict[str, int]] = None,
) -> dict[str, Any]:
    meta: dict[str, Any] = {"step": step}
    if dims is not None:
        meta["dims"] = dims
    parallel_info: dict[str, int] = {
        "tp_rank": tp_rank,
        "tp_size": tp_size,
        "cp_rank": cp_rank,
        "cp_size": cp_size,
    }
    if extra_parallel_info is not None:
        parallel_info.update(extra_parallel_info)
    meta["sglang_parallel_info"] = parallel_info
    return meta
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 56-56: Define class TestComputePerStepSubPlans
```python
class TestComputePerStepSubPlans:
```
**EN:** This declaration introduces the `TestComputePerStepSubPlans` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputePerStepSubPlans` 测试类，并说明它通过继承承担的职责。

### Lines 57-59: Run test: empty metas
```python
    def test_empty_metas(self) -> None:
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(metas=[])
        assert result == []
```
**EN:** This test method exercises empty metas and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty metas 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 61-65: Run test: single meta
```python
    def test_single_meta(self) -> None:
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=[_make_meta(dims="b h[tp]", tp_size=2)]
        )
        assert result == []
```
**EN:** This test method exercises single meta and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single meta 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 67-74: Run test: dims none
```python
    def test_dims_none(self) -> None:
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=[
                _make_meta(tp_rank=0, tp_size=2),
                _make_meta(tp_rank=1, tp_size=2),
            ]
        )
        assert result == []
```
**EN:** This test method exercises dims none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dims none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 76-87: Run test: tp sharded returns unsharder plan
```python
    def test_tp_sharded_returns_unsharder_plan(self) -> None:
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=[
                _make_meta(dims="b h[tp]", tp_rank=0, tp_size=2),
                _make_meta(dims="b h[tp]", tp_rank=1, tp_size=2),
            ]
        )
        assert len(result) >= 1
        unsharder_plans: list[UnsharderPlan] = [
            p for p in result if isinstance(p, UnsharderPlan)
        ]
        assert len(unsharder_plans) >= 1
```
**EN:** This test method exercises tp sharded returns unsharder plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tp sharded returns unsharder plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 89-103: Run test: zigzag returns both plans
```python
    def test_zigzag_returns_both_plans(self) -> None:
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=[
                _make_meta(dims="b s[cp:zigzag] h", cp_rank=0, cp_size=2),
                _make_meta(dims="b s[cp:zigzag] h", cp_rank=1, cp_size=2),
            ]
        )
        unsharder_plans: list[UnsharderPlan] = [
            p for p in result if isinstance(p, UnsharderPlan)
        ]
        reorderer_plans: list[ReordererPlan] = [
            p for p in result if isinstance(p, ReordererPlan)
        ]
        assert len(unsharder_plans) >= 1
        assert len(reorderer_plans) >= 1
```
**EN:** This test method exercises zigzag returns both plans and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zigzag returns both plans 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 106-106: Define class TestComputePerStepPlans
```python
class TestComputePerStepPlans:
```
**EN:** This declaration introduces the `TestComputePerStepPlans` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputePerStepPlans` 测试类，并说明它通过继承承担的职责。

### Lines 107-119: Run test: groups by step
```python
    def test_groups_by_step(self) -> None:
        metas: list[dict[str, Any]] = [
            _make_meta(step=0, tp_rank=0, tp_size=2),
            _make_meta(step=0, tp_rank=1, tp_size=2),
            _make_meta(step=1, tp_rank=0, tp_size=1),
        ]
        result: list[AlignerPerStepPlan] = _compute_per_step_plans(metas=metas)

        assert len(result) == 2
        assert result[0].step == 0
        assert result[0].input_object_indices == [0, 1]
        assert result[1].step == 1
        assert result[1].input_object_indices == [2]
```
**EN:** This test method exercises groups by step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 groups by step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 121-130: Run test: sorted by step
```python
    def test_sorted_by_step(self) -> None:
        metas: list[dict[str, Any]] = [
            _make_meta(step=2),
            _make_meta(step=0),
            _make_meta(step=1),
        ]
        result: list[AlignerPerStepPlan] = _compute_per_step_plans(metas=metas)

        steps: list[int] = [p.step for p in result]
        assert steps == [0, 1, 2]
```
**EN:** This test method exercises sorted by step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sorted by step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 132-140: Run test: single meta per step empty sub plans
```python
    def test_single_meta_per_step_empty_sub_plans(self) -> None:
        metas: list[dict[str, Any]] = [
            _make_meta(step=0),
            _make_meta(step=1),
        ]
        result: list[AlignerPerStepPlan] = _compute_per_step_plans(metas=metas)

        assert len(result) == 2
        assert all(plan.sub_plans == [] for plan in result)
```
**EN:** This test method exercises single meta per step empty sub plans and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single meta per step empty sub plans 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 143-143: Define class TestComputeAlignerPlan
```python
class TestComputeAlignerPlan:
```
**EN:** This declaration introduces the `TestComputeAlignerPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeAlignerPlan` 测试类，并说明它通过继承承担的职责。

### Lines 144-156: Run test: wraps both sides
```python
    def test_wraps_both_sides(self) -> None:
        metas_x: list[dict[str, Any]] = [_make_meta(step=0)]
        metas_y: list[dict[str, Any]] = [_make_meta(step=0)]

        plan: AlignerPlan = compute_aligner_plan(
            metas_pair=Pair(x=metas_x, y=metas_y),
            token_aligner_mode=None,
            token_aligner_plan=None,
        )

        assert len(plan.per_step_plans.x) == 1
        assert len(plan.per_step_plans.y) == 1
        assert plan.token_aligner_plan is None
```
**EN:** This test method exercises wraps both sides and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 wraps both sides 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 158-179: Run test: preserves token aligner plan
```python
    def test_preserves_token_aligner_plan(self) -> None:
        from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
            TokenAlignerPlan,
            TokenLocator,
        )

        ta_plan = TokenAlignerPlan(
            locators=Pair(
                x=TokenLocator(steps=[0], token_index_in_step=[0]),
                y=TokenLocator(steps=[0], token_index_in_step=[0]),
            ),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
        )

        plan: AlignerPlan = compute_aligner_plan(
            metas_pair=Pair(x=[_make_meta()], y=[_make_meta()]),
            token_aligner_mode="smart",
            token_aligner_plan=ta_plan,
        )

        assert plan.token_aligner_plan is ta_plan
        assert plan.token_aligner_mode == "smart"
```
**EN:** This test method exercises preserves token aligner plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 preserves token aligner plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 182-182: Define class TestComputePerStepSubPlansThd
```python
class TestComputePerStepSubPlansThd:
```
**EN:** This declaration introduces the `TestComputePerStepSubPlansThd` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputePerStepSubPlansThd` 测试类，并说明它通过继承承担的职责。

### Lines 183-239: Run test: thd zigzag returns thd plans
```python
    def test_thd_zigzag_returns_thd_plans(self) -> None:
        """t[cp:zigzag] h[tp] generates THD-typed unsharder + reorderer plans."""
        thd_global_seq_lens: list[int] = [100, 64, 92]
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=[
                _make_meta(
                    dims="t[cp:zigzag] h[tp]",
                    cp_rank=0,
                    cp_size=2,
                    tp_rank=0,
                    tp_size=2,
                ),
                _make_meta(
                    dims="t[cp:zigzag] h[tp]",
                    cp_rank=0,
                    cp_size=2,
                    tp_rank=1,
                    tp_size=2,
                ),
                _make_meta(
                    dims="t[cp:zigzag] h[tp]",
                    cp_rank=1,
                    cp_size=2,
                    tp_rank=0,
                    tp_size=2,
                ),
                _make_meta(
                    dims="t[cp:zigzag] h[tp]",
                    cp_rank=1,
                    cp_size=2,
                    tp_rank=1,
                    tp_size=2,
                ),
            ],
            thd_global_seq_lens=thd_global_seq_lens,
        )

        unsharder_plans: list[UnsharderPlan] = [
            p for p in result if isinstance(p, UnsharderPlan)
        ]
        reorderer_plans: list[ReordererPlan] = [
            p for p in result if isinstance(p, ReordererPlan)
        ]

        # Should have at least one THD concat plan for CP axis
        thd_concat_plans: list[UnsharderPlan] = [
            p for p in unsharder_plans if isinstance(p.params, CpThdConcatParams)
        ]
        assert len(thd_concat_plans) == 1
        assert thd_concat_plans[0].params.seq_lens_per_rank == [50, 32, 46]

        # Should have exactly one THD reorder plan
        assert len(reorderer_plans) == 1
        assert isinstance(reorderer_plans[0].params, ZigzagToNaturalThdParams)
        assert reorderer_plans[0].params.cp_size == 2
        # Reorder seq_lens = global seq_lens (reorder happens after unshard)
        assert reorderer_plans[0].params.seq_lens == [100, 64, 92]
```
**EN:** This test method exercises thd zigzag returns thd plans and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 thd zigzag returns thd plans 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 242-242: Define class TestComputePerStepSubPlansDpFiltered
```python
class TestComputePerStepSubPlansDpFiltered:
```
**EN:** This declaration introduces the `TestComputePerStepSubPlansDpFiltered` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputePerStepSubPlansDpFiltered` 测试类，并说明它通过继承承担的职责。

### Lines 243-245: Document the class `TestComputePerStepSubPlansDpFiltered`
```python
    """Tests that compute_per_step_sub_plans passes dp_filtered_axis to unsharder,
    so DP axes already handled by the upstream DP filter don't cause validation errors.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestComputePerStepSubPlansDpFiltered`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestComputePerStepSubPlansDpFiltered`的设计意图。

### Lines 247-269: Run test: dp2 tp2 does not raise
```python
    def test_dp2_tp2_does_not_raise(self) -> None:
        """DP2 + TP2, dims='t h[tp]' → should not raise despite DP being active."""
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=[
                _make_meta(
                    dims="t h[tp]",
                    tp_rank=0,
                    tp_size=2,
                    extra_parallel_info={"dp_rank": 0, "dp_size": 2},
                ),
                _make_meta(
                    dims="t h[tp]",
                    tp_rank=1,
                    tp_size=2,
                    extra_parallel_info={"dp_rank": 0, "dp_size": 2},
                ),
            ]
        )
        unsharder_plans: list[UnsharderPlan] = [
            p for p in result if isinstance(p, UnsharderPlan)
        ]
        assert len(unsharder_plans) == 1
        assert unsharder_plans[0].axis.value == "tp"
```
**EN:** This test method exercises dp2 tp2 does not raise and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 tp2 does not raise 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 271-285: Run test: dp2 only no sharding does not raise
```python
    def test_dp2_only_no_sharding_does_not_raise(self) -> None:
        """DP2 only, dims='t h' → should not raise, no plans produced."""
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=[
                _make_meta(
                    dims="t h",
                    extra_parallel_info={"dp_rank": 0, "dp_size": 2},
                ),
                _make_meta(
                    dims="t h",
                    extra_parallel_info={"dp_rank": 0, "dp_size": 2},
                ),
            ]
        )
        assert result == []
```
**EN:** This test method exercises dp2 only no sharding does not raise and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 only no sharding does not raise 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 287-309: Run test: dp alias passes correct filtered axis
```python
    def test_dp_alias_passes_correct_filtered_axis(self) -> None:
        """dims with '# dp:=moe_dp', metas have moe_dp → should not raise."""
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(
            metas=[
                _make_meta(
                    dims="t h[tp] # dp:=moe_dp",
                    tp_rank=0,
                    tp_size=2,
                    extra_parallel_info={"moe_dp_rank": 0, "moe_dp_size": 2},
                ),
                _make_meta(
                    dims="t h[tp] # dp:=moe_dp",
                    tp_rank=1,
                    tp_size=2,
                    extra_parallel_info={"moe_dp_rank": 0, "moe_dp_size": 2},
                ),
            ]
        )
        unsharder_plans: list[UnsharderPlan] = [
            p for p in result if isinstance(p, UnsharderPlan)
        ]
        assert len(unsharder_plans) == 1
        assert unsharder_plans[0].axis.value == "tp"
```
**EN:** This test method exercises dp alias passes correct filtered axis and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp alias passes correct filtered axis 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 311-331: Run test: dp2 tp2 cp2 does not raise
```python
    def test_dp2_tp2_cp2_does_not_raise(self) -> None:
        """DP2 + TP2 + CP2, dims='s[cp] h[tp]' → should not raise."""
        metas = []
        for cp_rank in range(2):
            for tp_rank in range(2):
                metas.append(
                    _make_meta(
                        dims="s[cp] h[tp]",
                        tp_rank=tp_rank,
                        tp_size=2,
                        cp_rank=cp_rank,
                        cp_size=2,
                        extra_parallel_info={"dp_rank": 0, "dp_size": 2},
                    )
                )
        result: list[AlignerPerStepSubPlan] = compute_per_step_sub_plans(metas=metas)
        unsharder_plans: list[UnsharderPlan] = [
            p for p in result if isinstance(p, UnsharderPlan)
        ]
        axes = {p.axis.value for p in unsharder_plans}
        assert axes == {"cp", "tp"}
```
**EN:** This test method exercises dp2 tp2 cp2 does not raise and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 tp2 cp2 does not raise 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 334-335: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.entrypoint.planner`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `typing`
- Notable symbols / 关键符号: None / 无
