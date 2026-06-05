# test_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/entrypoint/test_executor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on entrypoint executor in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 entrypoint executor 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Import dependencies
```python
import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.entrypoint.executor import (
    AlignerResult,
    StepPlansResult,
    SubPlansResult,
    _execute_step_plans,
    execute_aligner_plan,
    execute_sub_plan,
    execute_sub_plans,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.types import (
    AlignerPerStepPlan,
    AlignerPlan,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerPlan,
    TokenLocator,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
    ConcatParams,
    UnsharderPlan,
)
from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis, TokenLayout
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 31-31: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 34-34: Define class TestExecuteSubPlans
```python
class TestExecuteSubPlans:
```
**EN:** This declaration introduces the `TestExecuteSubPlans` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteSubPlans` 测试类，并说明它通过继承承担的职责。

### Lines 35-39: Run test: empty tensors returns none
```python
    def test_empty_tensors_returns_none(self) -> None:
        r: SubPlansResult = execute_sub_plans(tensors=[], plans=[])
        assert r.tensor is None
        assert r.checks == []
        assert r.snapshots == []
```
**EN:** This test method exercises empty tensors returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty tensors returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 41-47: Run test: no plans single tensor passthrough
```python
    def test_no_plans_single_tensor_passthrough(self) -> None:
        tensor: torch.Tensor = torch.tensor([1.0, 2.0, 3.0])
        r: SubPlansResult = execute_sub_plans(tensors=[tensor], plans=[])
        assert r.tensor is not None
        assert torch.equal(r.tensor, tensor)
        assert r.checks == []
        assert r.snapshots == []
```
**EN:** This test method exercises no plans single tensor passthrough and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no plans single tensor passthrough 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 49-57: Run test: no plans multiple tensors returns none
```python
    def test_no_plans_multiple_tensors_returns_none(self) -> None:
        tensors: list[torch.Tensor] = [
            torch.tensor([1.0]),
            torch.tensor([2.0]),
        ]
        r: SubPlansResult = execute_sub_plans(tensors=tensors, plans=[])
        assert r.tensor is None
        assert r.checks == []
        assert r.snapshots == []
```
**EN:** This test method exercises no plans multiple tensors returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no plans multiple tensors returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 59-75: Run test: with unsharder plan
```python
    def test_with_unsharder_plan(self) -> None:
        t0: torch.Tensor = torch.tensor([[1.0, 2.0]]).refine_names("b", "h")
        t1: torch.Tensor = torch.tensor([[3.0, 4.0]]).refine_names("b", "h")

        plan = UnsharderPlan(
            axis=ParallelAxis.TP,
            params=ConcatParams(dim_name="h"),
            groups=[[0, 1]],
        )

        r: SubPlansResult = execute_sub_plans(tensors=[t0, t1], plans=[plan])

        assert r.tensor is not None
        expected: torch.Tensor = torch.tensor([[1.0, 2.0, 3.0, 4.0]])
        assert torch.equal(r.tensor.rename(None), expected)
        assert r.checks == []
        assert len(r.snapshots) == 1
```
**EN:** This test method exercises with unsharder plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with unsharder plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 78-78: Define class TestExecuteSubPlan
```python
class TestExecuteSubPlan:
```
**EN:** This declaration introduces the `TestExecuteSubPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteSubPlan` 测试类，并说明它通过继承承担的职责。

### Lines 79-84: Run test: unknown plan type raises
```python
    def test_unknown_plan_type_raises(self) -> None:
        class _FakePlan:
            pass

        with pytest.raises(NotImplementedError, match="Unknown"):
            execute_sub_plan(tensors=[torch.tensor([1.0])], plan=_FakePlan())
```
**EN:** This test method exercises unknown plan type raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unknown plan type raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 87-87: Define class TestExecuteStepPlans
```python
class TestExecuteStepPlans:
```
**EN:** This declaration introduces the `TestExecuteStepPlans` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteStepPlans` 测试类，并说明它通过继承承担的职责。

### Lines 88-106: Run test: step with none result omitted
```python
    def test_step_with_none_result_omitted(self) -> None:
        tensors: list[torch.Tensor] = [
            torch.tensor([1.0]),
            torch.tensor([2.0]),
        ]

        step_plan = AlignerPerStepPlan(
            step=0,
            input_object_indices=[0, 1],
            sub_plans=[],
        )

        r: StepPlansResult = _execute_step_plans(
            tensors=tensors, step_plans=[step_plan]
        )

        assert r.tensors == {}
        assert r.checks == []
        assert len(r.traced_side.step_plans) == 1
```
**EN:** This test method exercises step with none result omitted and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 step with none result omitted 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 108-125: Run test: single step passthrough
```python
    def test_single_step_passthrough(self) -> None:
        tensor: torch.Tensor = torch.tensor([1.0, 2.0])

        step_plan = AlignerPerStepPlan(
            step=5,
            input_object_indices=[0],
            sub_plans=[],
        )

        r: StepPlansResult = _execute_step_plans(
            tensors=[tensor], step_plans=[step_plan]
        )

        assert 5 in r.tensors
        assert torch.equal(r.tensors[5], tensor)
        assert r.checks == []
        assert len(r.traced_side.step_plans) == 1
        assert r.traced_side.step_plans[0].step == 5
```
**EN:** This test method exercises single step passthrough and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single step passthrough 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 128-128: Define class TestExecuteAlignerPlan
```python
class TestExecuteAlignerPlan:
```
**EN:** This declaration introduces the `TestExecuteAlignerPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteAlignerPlan` 测试类，并说明它通过继承承担的职责。

### Lines 129-130: Define helper: make step plan
```python
    def _make_step_plan(self, *, step: int, indices: list[int]) -> AlignerPerStepPlan:
        return AlignerPerStepPlan(step=step, input_object_indices=indices, sub_plans=[])
```
**EN:** This helper function encapsulates reusable logic inside `TestExecuteAlignerPlan` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestExecuteAlignerPlan` 内部调用，从而让场景结构更清晰。

### Lines 132-151: Run test: x side empty returns failed x
```python
    def test_x_side_empty_returns_failed_x(self) -> None:
        plan = AlignerPlan(
            per_step_plans=Pair(
                x=[self._make_step_plan(step=0, indices=[0, 1])],
                y=[self._make_step_plan(step=0, indices=[0])],
            ),
            token_aligner_plan=None,
        )

        tensors_pair: Pair[list[torch.Tensor]] = Pair(
            x=[torch.tensor([1.0]), torch.tensor([2.0])],
            y=[torch.tensor([3.0])],
        )

        result: AlignerResult = execute_aligner_plan(
            tensors_pair=tensors_pair, plan=plan
        )

        assert result.tensors is None
        assert result.failed_side_xy == "x"
```
**EN:** This test method exercises x side empty returns failed x and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 x side empty returns failed x 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 153-172: Run test: y side empty returns failed y
```python
    def test_y_side_empty_returns_failed_y(self) -> None:
        plan = AlignerPlan(
            per_step_plans=Pair(
                x=[self._make_step_plan(step=0, indices=[0])],
                y=[self._make_step_plan(step=0, indices=[0, 1])],
            ),
            token_aligner_plan=None,
        )

        tensors_pair: Pair[list[torch.Tensor]] = Pair(
            x=[torch.tensor([1.0])],
            y=[torch.tensor([2.0]), torch.tensor([3.0])],
        )

        result: AlignerResult = execute_aligner_plan(
            tensors_pair=tensors_pair, plan=plan
        )

        assert result.tensors is None
        assert result.failed_side_xy == "y"
```
**EN:** This test method exercises y side empty returns failed y and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 y side empty returns failed y 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 174-194: Run test: no token aligner single step
```python
    def test_no_token_aligner_single_step(self) -> None:
        plan = AlignerPlan(
            per_step_plans=Pair(
                x=[self._make_step_plan(step=0, indices=[0])],
                y=[self._make_step_plan(step=0, indices=[0])],
            ),
            token_aligner_plan=None,
        )

        t_x: torch.Tensor = torch.tensor([1.0, 2.0])
        t_y: torch.Tensor = torch.tensor([3.0, 4.0])
        tensors_pair: Pair[list[torch.Tensor]] = Pair(x=[t_x], y=[t_y])

        result: AlignerResult = execute_aligner_plan(
            tensors_pair=tensors_pair, plan=plan
        )

        assert result.tensors is not None
        assert result.failed_side_xy is None
        assert torch.equal(result.tensors.x, t_x)
        assert torch.equal(result.tensors.y, t_y)
```
**EN:** This test method exercises no token aligner single step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no token aligner single step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 196-215: Run test: success returns none failed side
```python
    def test_success_returns_none_failed_side(self) -> None:
        plan = AlignerPlan(
            per_step_plans=Pair(
                x=[self._make_step_plan(step=0, indices=[0])],
                y=[self._make_step_plan(step=0, indices=[0])],
            ),
            token_aligner_plan=None,
        )

        tensors_pair: Pair[list[torch.Tensor]] = Pair(
            x=[torch.tensor([10.0])],
            y=[torch.tensor([20.0])],
        )

        result: AlignerResult = execute_aligner_plan(
            tensors_pair=tensors_pair, plan=plan
        )

        assert result.failed_side_xy is None
        assert result.tensors is not None
```
**EN:** This test method exercises success returns none failed side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 success returns none failed side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 218-218: Define class TestExecuteAlignerPlanWithTokenDim
```python
class TestExecuteAlignerPlanWithTokenDim:
```
**EN:** This declaration introduces the `TestExecuteAlignerPlanWithTokenDim` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteAlignerPlanWithTokenDim` 测试类，并说明它通过继承承担的职责。

### Lines 219-219: Document the class `TestExecuteAlignerPlanWithTokenDim`
```python
    """End-to-end tests for AlignerPlan with non-zero token_dim."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestExecuteAlignerPlanWithTokenDim`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestExecuteAlignerPlanWithTokenDim`的设计意图。

### Lines 221-222: Define helper: make step plan
```python
    def _make_step_plan(self, *, step: int, indices: list[int]) -> AlignerPerStepPlan:
        return AlignerPerStepPlan(step=step, input_object_indices=indices, sub_plans=[])
```
**EN:** This helper function encapsulates reusable logic inside `TestExecuteAlignerPlanWithTokenDim` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestExecuteAlignerPlanWithTokenDim` 内部调用，从而让场景结构更清晰。

### Lines 224-275: Run test: token dim nonzero e2e
```python
    def test_token_dim_nonzero_e2e(self) -> None:
        """AlignerPlan with token at dim 1 passes through to token aligner correctly."""
        torch.manual_seed(42)

        # shape [3, 4, 8]: dim0=a, dim1=token(4 tokens), dim2=hidden
        tensor_x: torch.Tensor = torch.randn(3, 4, 8).refine_names("a", "t", "h")
        tensor_y: torch.Tensor = torch.randn(3, 4, 8).refine_names("a", "t", "h")

        locator_x = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[0, 1, 2],
        )
        locator_y = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[0, 1, 2],
        )
        token_plan = TokenAlignerPlan(
            locators=Pair(x=locator_x, y=locator_y),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
        )

        plan = AlignerPlan(
            per_step_plans=Pair(
                x=[self._make_step_plan(step=0, indices=[0])],
                y=[self._make_step_plan(step=0, indices=[0])],
            ),
            token_aligner_mode="smart",
            token_aligner_plan=token_plan,
        )

        tensors_pair: Pair[list[torch.Tensor]] = Pair(x=[tensor_x], y=[tensor_y])
        result: AlignerResult = execute_aligner_plan(
            tensors_pair=tensors_pair, plan=plan
        )

        assert result.tensors is not None
        assert result.failed_side_xy is None
        # token dim stays at dim 1 -> shape [3, 3, 8] (3 tokens selected from 4)
        assert result.tensors.x.shape == (3, 3, 8)
        assert result.tensors.y.shape == (3, 3, 8)

        plain_x: torch.Tensor = tensor_x.rename(None)
        plain_y: torch.Tensor = tensor_y.rename(None)
        for i in range(3):
            assert torch.equal(
                result.tensors.x.select(dim=1, index=i),
                plain_x.select(dim=1, index=i),
            )
            assert torch.equal(
                result.tensors.y.select(dim=1, index=i),
                plain_y.select(dim=1, index=i),
            )
```
**EN:** This test method exercises token dim nonzero e2e and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 token dim nonzero e2e 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 277-324: Run test: bshd cross layout e2e
```python
    def test_bshd_cross_layout_e2e(self) -> None:
        """x=SGLang THD, y=Megatron BSHD: planner->executor full flow."""
        torch.manual_seed(42)

        # x side: THD layout, shape [6, 8] (6 tokens, hidden=8), pre-named
        tensor_x: torch.Tensor = torch.randn(6, 8).refine_names("t", "h")

        # y side: BSHD layout, shape [2, 3, 8] (B=2, S=3, H=8), pre-named
        tensor_y: torch.Tensor = torch.randn(2, 3, 8).refine_names("b", "s", "h")
        flat_y: torch.Tensor = tensor_y.rename(None).reshape(6, 8)

        locator = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[0, 2, 5],
        )
        token_plan = TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.BS),
        )

        plan = AlignerPlan(
            per_step_plans=Pair(
                x=[self._make_step_plan(step=0, indices=[0])],
                y=[self._make_step_plan(step=0, indices=[0])],
            ),
            token_aligner_mode="smart",
            token_aligner_plan=token_plan,
        )

        tensors_pair: Pair[list[torch.Tensor]] = Pair(x=[tensor_x], y=[tensor_y])
        result: AlignerResult = execute_aligner_plan(
            tensors_pair=tensors_pair, plan=plan
        )

        assert result.tensors is not None
        assert result.failed_side_xy is None

        assert result.tensors.x.shape == (3, 8)
        assert result.tensors.y.shape == (3, 8)

        plain_x: torch.Tensor = tensor_x.rename(None)
        assert torch.equal(result.tensors.x[0], plain_x[0])
        assert torch.equal(result.tensors.x[1], plain_x[2])
        assert torch.equal(result.tensors.x[2], plain_x[5])

        assert torch.equal(result.tensors.y[0], flat_y[0])
        assert torch.equal(result.tensors.y[1], flat_y[2])
        assert torch.equal(result.tensors.y[2], flat_y[5])
```
**EN:** This test method exercises bshd cross layout e2e and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd cross layout e2e 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 327-328: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.entrypoint.executor`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
