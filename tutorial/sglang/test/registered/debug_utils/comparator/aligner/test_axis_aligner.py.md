# test_axis_aligner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/test_axis_aligner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on axis aligner in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 axis aligner 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import sys
from typing import Optional

import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.axis_aligner import (
    AxisAlignerPlan,
    compute_axis_aligner_plan,
    execute_axis_aligner_plan,
)
from sglang.srt.debug_utils.comparator.log_sink import log_sink
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 16-16: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 19-19: Define class TestComputeAxisAlignerPlan
```python
class TestComputeAxisAlignerPlan:
```
**EN:** This declaration introduces the `TestComputeAxisAlignerPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeAxisAlignerPlan` 测试类，并说明它通过继承承担的职责。

### Lines 20-23: Run test: no dims returns none
```python
    def test_no_dims_returns_none(self) -> None:
        assert compute_axis_aligner_plan(Pair(x=None, y=None)) is None
        assert compute_axis_aligner_plan(Pair(x="t h d", y=None)) is None
        assert compute_axis_aligner_plan(Pair(x=None, y="t h d")) is None
```
**EN:** This test method exercises no dims returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no dims returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 25-29: Run test: same order returns none
```python
    def test_same_order_returns_none(self) -> None:
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t h d", y="t h d")
        )
        assert result is None
```
**EN:** This test method exercises same order returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 same order returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 31-37: Run test: different order
```python
    def test_different_order(self) -> None:
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t h d", y="t d h")
        )
        assert result is not None
        assert result.pattern.x == "t h d -> t d h"
        assert result.pattern.y is None
```
**EN:** This test method exercises different order and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 different order 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 39-48: Run test: name mismatch returns none with warning
```python
    def test_name_mismatch_returns_none_with_warning(self) -> None:
        with log_sink.context() as warnings:
            result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
                Pair(x="t h d", y="t h e")
            )

        assert result is None
        assert len(warnings) == 1
        assert warnings[0].category == "axis_aligner_dim_mismatch"
        assert "dim name sets differ" in warnings[0].message
```
**EN:** This test method exercises name mismatch returns none with warning and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 name mismatch returns none with warning 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 50-55: Run test: modifiers ignored for name extraction
```python
    def test_modifiers_ignored_for_name_extraction(self) -> None:
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t h[tp] d", y="t d h[tp]")
        )
        assert result is not None
        assert result.pattern.x == "t h d -> t d h"
```
**EN:** This test method exercises modifiers ignored for name extraction and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 modifiers ignored for name extraction 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 57-63: Run test: squeeze only no swap
```python
    def test_squeeze_only_no_swap(self) -> None:
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t 1 h", y="t h")
        )
        assert result is not None
        assert result.pattern.x == "t 1 h -> t h"
        assert result.pattern.y is None
```
**EN:** This test method exercises squeeze only no swap and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 squeeze only no swap 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 65-71: Run test: squeeze both sides
```python
    def test_squeeze_both_sides(self) -> None:
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t 1 h", y="1 t h")
        )
        assert result is not None
        assert result.pattern.x == "t 1 h -> t h"
        assert result.pattern.y == "1 t h -> t h"
```
**EN:** This test method exercises squeeze both sides and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 squeeze both sides 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 73-79: Run test: squeeze plus swap
```python
    def test_squeeze_plus_swap(self) -> None:
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t 1 h d", y="t d h")
        )
        assert result is not None
        assert result.pattern.x == "t 1 h d -> t d h"
        assert result.pattern.y is None
```
**EN:** This test method exercises squeeze plus swap and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 squeeze plus swap 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 81-87: Run test: squeeze y only
```python
    def test_squeeze_y_only(self) -> None:
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t h", y="t 1 h")
        )
        assert result is not None
        assert result.pattern.x is None
        assert result.pattern.y == "t 1 h -> t h"
```
**EN:** This test method exercises squeeze y only and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 squeeze y only 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 89-96: Run test: multiple squeeze one side
```python
    def test_multiple_squeeze_one_side(self) -> None:
        """Two squeeze dims on x, none on y."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="1 t 1 h", y="t h")
        )
        assert result is not None
        assert result.pattern.x == "1 t 1 h -> t h"
        assert result.pattern.y is None
```
**EN:** This test method exercises multiple squeeze one side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple squeeze one side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 98-105: Run test: multiple squeeze asymmetric
```python
    def test_multiple_squeeze_asymmetric(self) -> None:
        """Different numbers of squeeze dims on each side."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="1 t 1 h", y="1 t h")
        )
        assert result is not None
        assert result.pattern.x == "1 t 1 h -> t h"
        assert result.pattern.y == "1 t h -> t h"
```
**EN:** This test method exercises multiple squeeze asymmetric and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple squeeze asymmetric 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 107-114: Run test: four dim full reversal
```python
    def test_four_dim_full_reversal(self) -> None:
        """4-dim permutation: full reversal."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="a b c d", y="d c b a")
        )
        assert result is not None
        assert result.pattern.x == "a b c d -> d c b a"
        assert result.pattern.y is None
```
**EN:** This test method exercises four dim full reversal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 four dim full reversal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 117-117: Define class TestComputeAxisAlignerPlanFused
```python
class TestComputeAxisAlignerPlanFused:
```
**EN:** This declaration introduces the `TestComputeAxisAlignerPlanFused` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeAxisAlignerPlanFused` 测试类，并说明它通过继承承担的职责。

### Lines 118-125: Run test: fused vs separate
```python
    def test_fused_vs_separate(self) -> None:
        """x=fused 2D, y=separate 3D: y flattens to match x's fused form."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t (num_heads*head_dim)[tp]", y="t num_heads[tp] head_dim")
        )
        assert result is not None
        assert result.pattern.x is None
        assert result.pattern.y == "t num_heads head_dim -> t (num_heads head_dim)"
```
**EN:** This test method exercises fused vs separate and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused vs separate 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 127-134: Run test: separate vs fused
```python
    def test_separate_vs_fused(self) -> None:
        """x=separate 3D, y=fused 2D: x flattens to match y's fused form."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t num_heads[tp] head_dim", y="t (num_heads*head_dim)[tp]")
        )
        assert result is not None
        assert result.pattern.x == "t num_heads head_dim -> t (num_heads head_dim)"
        assert result.pattern.y is None
```
**EN:** This test method exercises separate vs fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 separate vs fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 136-141: Run test: both fused same no plan
```python
    def test_both_fused_same_no_plan(self) -> None:
        """Both sides fused, same order → None (no-op)."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t (a*b)", y="t (a*b)")
        )
        assert result is None
```
**EN:** This test method exercises both fused same no plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 both fused same no plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 143-150: Run test: fused name mismatch returns none
```python
    def test_fused_name_mismatch_returns_none(self) -> None:
        """Fused vs separate with mismatched names → None."""
        with log_sink.context() as warnings:
            result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
                Pair(x="t (a*b)", y="t c d")
            )
        assert result is None
        assert len(warnings) == 1
```
**EN:** This test method exercises fused name mismatch returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused name mismatch returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 152-159: Run test: partial fused and regular
```python
    def test_partial_fused_and_regular(self) -> None:
        """x has "(a*b) c", y has "a b c": y flattens a,b to match x's fused form."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="(a*b) c", y="a b c")
        )
        assert result is not None
        assert result.pattern.x is None
        assert result.pattern.y == "a b c -> (a b) c"
```
**EN:** This test method exercises partial fused and regular and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 partial fused and regular 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 161-168: Run test: fused vs reordered separate
```python
    def test_fused_vs_reordered_separate(self) -> None:
        """x=fused "(a*b) c", y=reordered separate "b a c": y flattens+reorders."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="(a*b) c", y="b a c")
        )
        assert result is not None
        assert result.pattern.x is None
        assert result.pattern.y == "b a c -> (a b) c"
```
**EN:** This test method exercises fused vs reordered separate and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused vs reordered separate 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 170-177: Run test: fused reorder both sides
```python
    def test_fused_reorder_both_sides(self) -> None:
        """x=fused "c (a*b)", y=separate "a b c": x reorders fused, y flattens."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="c (a*b)", y="a b c")
        )
        assert result is not None
        assert result.pattern.x == "c a___b -> a___b c"
        assert result.pattern.y == "a b c -> (a b) c"
```
**EN:** This test method exercises fused reorder both sides and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused reorder both sides 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 179-186: Run test: fused with squeeze
```python
    def test_fused_with_squeeze(self) -> None:
        """Fused + squeeze on one side, separate on other."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t 1 (a*b)", y="t a b")
        )
        assert result is not None
        assert result.pattern.x == "t 1 a___b -> t a___b"
        assert result.pattern.y == "t a b -> t (a b)"
```
**EN:** This test method exercises fused with squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused with squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 188-195: Run test: three way fused vs separate
```python
    def test_three_way_fused_vs_separate(self) -> None:
        """3-way fused on x, separate on y."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t (a*b*c)", y="t a b c")
        )
        assert result is not None
        assert result.pattern.x is None
        assert result.pattern.y == "t a b c -> t (a b c)"
```
**EN:** This test method exercises three way fused vs separate and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three way fused vs separate 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 197-204: Run test: separate vs three way fused
```python
    def test_separate_vs_three_way_fused(self) -> None:
        """Separate on x, 3-way fused on y."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t a b c", y="t (a*b*c)")
        )
        assert result is not None
        assert result.pattern.x == "t a b c -> t (a b c)"
        assert result.pattern.y is None
```
**EN:** This test method exercises separate vs three way fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 separate vs three way fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 206-213: Run test: both fused different order
```python
    def test_both_fused_different_order(self) -> None:
        """Both sides fused same group but dims in different order."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="c (a*b)", y="(a*b) c")
        )
        assert result is not None
        assert result.pattern.x == "c a___b -> a___b c"
        assert result.pattern.y is None
```
**EN:** This test method exercises both fused different order and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 both fused different order 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 215-224: Run test: overlapping fused groups returns none
```python
    def test_overlapping_fused_groups_returns_none(self) -> None:
        """x fuses (a*b), y fuses (b*c): incompatible overlap → None with warning."""
        with log_sink.context() as warnings:
            result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
                Pair(x="(a*b) c", y="a (b*c)")
            )
        assert result is None
        assert len(warnings) == 1
        assert warnings[0].category == "axis_aligner_fused_conflict"
        assert "overlapping fused groups" in warnings[0].message
```
**EN:** This test method exercises overlapping fused groups returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 overlapping fused groups returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 227-227: Define class TestExecuteAxisAlignerPlan
```python
class TestExecuteAxisAlignerPlan:
```
**EN:** This declaration introduces the `TestExecuteAxisAlignerPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteAxisAlignerPlan` 测试类，并说明它通过继承承担的职责。

### Lines 228-239: Run test: rearrange
```python
    def test_rearrange(self) -> None:
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(4, 8, 16).refine_names("t", "h", "d")
        plan = AxisAlignerPlan(pattern=Pair(x="t h d -> t d h", y=None))

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor, plan=plan, side="x"
        )

        assert result.shape == (4, 16, 8)
        for i in range(4):
            assert torch.equal(result[i], tensor.rename(None)[i].T)
```
**EN:** This test method exercises rearrange and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 rearrange 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 241-250: Run test: execute squeeze
```python
    def test_execute_squeeze(self) -> None:
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(4, 1, 8).refine_names("t", "singleton0", "h")
        plan = AxisAlignerPlan(pattern=Pair(x="t 1 h -> t h", y=None))

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor, plan=plan, side="x"
        )

        assert result.shape == (4, 8)
```
**EN:** This test method exercises execute squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 execute squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 252-263: Run test: execute squeeze then swap
```python
    def test_execute_squeeze_then_swap(self) -> None:
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(4, 1, 8, 16).refine_names(
            "t", "singleton0", "h", "d"
        )
        plan = AxisAlignerPlan(pattern=Pair(x="t 1 h d -> t d h", y=None))

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor, plan=plan, side="x"
        )

        assert result.shape == (4, 16, 8)
```
**EN:** This test method exercises execute squeeze then swap and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 execute squeeze then swap 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 265-274: Run test: execute y side
```python
    def test_execute_y_side(self) -> None:
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(4, 1, 8).refine_names("t", "singleton0", "h")
        plan = AxisAlignerPlan(pattern=Pair(x=None, y="t 1 h -> t h"))

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor, plan=plan, side="y"
        )

        assert result.shape == (4, 8)
```
**EN:** This test method exercises execute y side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 execute y side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 276-285: Run test: noop side
```python
    def test_noop_side(self) -> None:
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(4, 8, 16).refine_names("t", "h", "d")
        plan = AxisAlignerPlan(pattern=Pair(x="t h d -> t d h", y=None))

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor, plan=plan, side="y"
        )

        assert result.shape == (4, 8, 16)
```
**EN:** This test method exercises noop side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 noop side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 287-294: Run test: invalid side raises
```python
    def test_invalid_side_raises(self) -> None:
        """Invalid side value should raise ValueError."""
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(4, 8, 16)
        plan = AxisAlignerPlan(pattern=Pair(x="t h d -> t d h", y=None))

        with pytest.raises(ValueError, match="side must be"):
            execute_axis_aligner_plan(tensor=tensor, plan=plan, side="z")
```
**EN:** This test method exercises invalid side raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 invalid side raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 297-297: Define class TestExecuteAxisAlignerPlanFlatten
```python
class TestExecuteAxisAlignerPlanFlatten:
```
**EN:** This declaration introduces the `TestExecuteAxisAlignerPlanFlatten` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteAxisAlignerPlanFlatten` 测试类，并说明它通过继承承担的职责。

### Lines 298-311: Run test: flatten separate to match fused
```python
    def test_flatten_separate_to_match_fused(self) -> None:
        """3D (t=4, nh=8, hd=16) → 2D (t=4, nh*hd=128) via einops flatten."""
        torch.manual_seed(42)
        tensor_3d: torch.Tensor = torch.randn(4, 8, 16)
        plan = AxisAlignerPlan(
            pattern=Pair(x=None, y="t nh hd -> t (nh hd)"),
        )

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor_3d, plan=plan, side="y"
        )

        assert result.shape == (4, 128)
        assert torch.equal(result, tensor_3d.reshape(4, 128))
```
**EN:** This test method exercises flatten separate to match fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 flatten separate to match fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 313-326: Run test: flatten preserves data
```python
    def test_flatten_preserves_data(self) -> None:
        """Flatten should be equivalent to reshape — verify element equality."""
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(2, 3, 4, 5)
        plan = AxisAlignerPlan(
            pattern=Pair(x="a b c d -> a (b c) d", y=None),
        )

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor, plan=plan, side="x"
        )

        assert result.shape == (2, 12, 5)
        assert torch.equal(result, tensor.reshape(2, 12, 5))
```
**EN:** This test method exercises flatten preserves data and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 flatten preserves data 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 328-340: Run test: flatten then rearrange
```python
    def test_flatten_then_rearrange(self) -> None:
        """Flatten + reorder in a single einops pattern."""
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(4, 8, 16, 32)
        plan = AxisAlignerPlan(
            pattern=Pair(x="t a b d -> t d (a b)", y=None),
        )

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor, plan=plan, side="x"
        )

        assert result.shape == (4, 32, 128)
```
**EN:** This test method exercises flatten then rearrange and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 flatten then rearrange 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 343-343: Define class TestEndToEndFusedAlignment
```python
class TestEndToEndFusedAlignment:
```
**EN:** This declaration introduces the `TestEndToEndFusedAlignment` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestEndToEndFusedAlignment` 测试类，并说明它通过继承承担的职责。

### Lines 344-363: Run test: fused vs separate full pipeline
```python
    def test_fused_vs_separate_full_pipeline(self) -> None:
        """Full pipeline: x=fused 2D "t nh*hd", y=separate 3D "t nh hd"."""
        torch.manual_seed(42)
        num_heads: int = 8
        head_dim: int = 16

        x_tensor: torch.Tensor = torch.randn(4, num_heads * head_dim)
        y_tensor: torch.Tensor = x_tensor.reshape(4, num_heads, head_dim)

        plan: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t (num_heads*head_dim)", y="t num_heads head_dim")
        )
        assert plan is not None

        y_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=y_tensor, plan=plan, side="y"
        )

        assert y_aligned.shape == x_tensor.shape
        assert torch.equal(y_aligned, x_tensor)
```
**EN:** This test method exercises fused vs separate full pipeline and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused vs separate full pipeline 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 365-384: Run test: separate vs fused full pipeline
```python
    def test_separate_vs_fused_full_pipeline(self) -> None:
        """Full pipeline: x=separate 3D "t nh hd", y=fused 2D "t nh*hd"."""
        torch.manual_seed(42)
        num_heads: int = 8
        head_dim: int = 16

        x_tensor: torch.Tensor = torch.randn(4, num_heads, head_dim)
        y_tensor: torch.Tensor = x_tensor.reshape(4, num_heads * head_dim)

        plan: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t num_heads head_dim", y="t (num_heads*head_dim)")
        )
        assert plan is not None

        x_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=x_tensor, plan=plan, side="x"
        )

        assert x_aligned.shape == y_tensor.shape
        assert torch.equal(x_aligned, y_tensor)
```
**EN:** This test method exercises separate vs fused full pipeline and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 separate vs fused full pipeline 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 386-410: Run test: fused with reorder
```python
    def test_fused_with_reorder(self) -> None:
        """Fused x + reordered separate y: both need alignment."""
        torch.manual_seed(42)
        a_size: int = 3
        b_size: int = 5

        # x: fused "c a*b" shape (7, 15)
        x_tensor: torch.Tensor = torch.randn(7, a_size * b_size)
        # y: separate "a b c" shape (3, 5, 7)
        y_tensor: torch.Tensor = x_tensor.reshape(7, a_size, b_size).permute(1, 2, 0)

        plan: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="c (a*b)", y="a b c")
        )
        assert plan is not None

        x_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=x_tensor, plan=plan, side="x"
        )
        y_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=y_tensor, plan=plan, side="y"
        )

        assert x_aligned.shape == y_aligned.shape
        assert torch.allclose(x_aligned, y_aligned)
```
**EN:** This test method exercises fused with reorder and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused with reorder 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 413-413: Define class TestEndToEndThreeWayFused
```python
class TestEndToEndThreeWayFused:
```
**EN:** This declaration introduces the `TestEndToEndThreeWayFused` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestEndToEndThreeWayFused` 测试类，并说明它通过继承承担的职责。

### Lines 414-432: Run test: three way fused vs separate
```python
    def test_three_way_fused_vs_separate(self) -> None:
        """Full pipeline: x=3-way fused "t (a*b*c)", y=separate "t a b c"."""
        torch.manual_seed(42)
        a_size, b_size, c_size = 2, 3, 4

        x_tensor: torch.Tensor = torch.randn(5, a_size * b_size * c_size)
        y_tensor: torch.Tensor = x_tensor.reshape(5, a_size, b_size, c_size)

        plan: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t (a*b*c)", y="t a b c")
        )
        assert plan is not None

        y_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=y_tensor, plan=plan, side="y"
        )

        assert y_aligned.shape == x_tensor.shape
        assert torch.equal(y_aligned, x_tensor)
```
**EN:** This test method exercises three way fused vs separate and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three way fused vs separate 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 435-435: Define class TestSeqTokenEquivalencePlan
```python
class TestSeqTokenEquivalencePlan:
```
**EN:** This declaration introduces the `TestSeqTokenEquivalencePlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSeqTokenEquivalencePlan` 测试类，并说明它通过继承承担的职责。

### Lines 436-436: Document the class `TestSeqTokenEquivalencePlan`
```python
    """Tests for s≡t dimension name equivalence in compute_axis_aligner_plan."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestSeqTokenEquivalencePlan`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestSeqTokenEquivalencePlan`的设计意图。

### Lines 438-445: Run test: s t equivalence squeeze
```python
    def test_s_t_equivalence_squeeze(self) -> None:
        """sglang 't h' vs megatron 's 1 h': plan squeezes y-side singleton, x-side no-op."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t h", y="s 1 h")
        )
        assert result is not None
        assert result.pattern.x is None
        assert result.pattern.y == "s 1 h -> s h"
```
**EN:** This test method exercises s t equivalence squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 s t equivalence squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 447-452: Run test: s t equivalence same shape
```python
    def test_s_t_equivalence_same_shape(self) -> None:
        """'t h d' vs 's h d': same order after normalization → no plan needed."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t h d", y="s h d")
        )
        assert result is None
```
**EN:** This test method exercises s t equivalence same shape and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 s t equivalence same shape 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 454-460: Run test: s t equivalence with swap
```python
    def test_s_t_equivalence_with_swap(self) -> None:
        """'t d h' vs 's h d': plan not None, x-pattern reorders."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t d h", y="s h d")
        )
        assert result is not None
        assert result.pattern.x is not None
```
**EN:** This test method exercises s t equivalence with swap and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 s t equivalence with swap 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 462-468: Run test: s t equivalence with fused
```python
    def test_s_t_equivalence_with_fused(self) -> None:
        """'t (a*b)' vs 's a b': plan not None, y-pattern flattens."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t (a*b)", y="s a b")
        )
        assert result is not None
        assert result.pattern.y is not None
```
**EN:** This test method exercises s t equivalence with fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 s t equivalence with fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 470-475: Run test: s t equivalence with squeeze and fused
```python
    def test_s_t_equivalence_with_squeeze_and_fused(self) -> None:
        """'t (num_heads*head_dim)' vs 's 1 num_heads head_dim': plan squeezes + flattens."""
        result: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t (num_heads*head_dim)", y="s 1 num_heads head_dim")
        )
        assert result is not None
```
**EN:** This test method exercises s t equivalence with squeeze and fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 s t equivalence with squeeze and fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 478-478: Define class TestSeqTokenEquivalenceExecute
```python
class TestSeqTokenEquivalenceExecute:
```
**EN:** This declaration introduces the `TestSeqTokenEquivalenceExecute` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSeqTokenEquivalenceExecute` 测试类，并说明它通过继承承担的职责。

### Lines 479-479: Document the class `TestSeqTokenEquivalenceExecute`
```python
    """Tests for s≡t dimension name equivalence in execute_axis_aligner_plan."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestSeqTokenEquivalenceExecute`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestSeqTokenEquivalenceExecute`的设计意图。

### Lines 481-492: Run test: execute s t squeeze
```python
    def test_execute_s_t_squeeze(self) -> None:
        """Tensor [4,1,8] with pattern 's 1 h -> s h' → shape [4,8]."""
        torch.manual_seed(42)
        tensor: torch.Tensor = torch.randn(4, 1, 8)
        plan = AxisAlignerPlan(pattern=Pair(x=None, y="s 1 h -> s h"))

        result: torch.Tensor = execute_axis_aligner_plan(
            tensor=tensor, plan=plan, side="y"
        )

        assert result.shape == (4, 8)
        assert torch.equal(result, tensor.squeeze(1))
```
**EN:** This test method exercises execute s t squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 execute s t squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 495-495: Define class TestEndToEndSeqTokenEquivalence
```python
class TestEndToEndSeqTokenEquivalence:
```
**EN:** This declaration introduces the `TestEndToEndSeqTokenEquivalence` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestEndToEndSeqTokenEquivalence` 测试类，并说明它通过继承承担的职责。

### Lines 496-496: Document the class `TestEndToEndSeqTokenEquivalence`
```python
    """End-to-end tests for s≡t equivalence through compute + execute pipeline."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestEndToEndSeqTokenEquivalence`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestEndToEndSeqTokenEquivalence`的设计意图。

### Lines 498-518: Run test: s t squeeze full pipeline
```python
    def test_s_t_squeeze_full_pipeline(self) -> None:
        """x=tensor(4,8) dims='t h', y=tensor(4,1,8) dims='s 1 h' → both aligned to (4,8)."""
        torch.manual_seed(42)
        data: torch.Tensor = torch.randn(4, 8)
        x_tensor: torch.Tensor = data.clone()
        y_tensor: torch.Tensor = data.unsqueeze(1)

        plan: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t h", y="s 1 h")
        )
        assert plan is not None

        x_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=x_tensor, plan=plan, side="x"
        )
        y_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=y_tensor, plan=plan, side="y"
        )

        assert x_aligned.shape == y_aligned.shape == (4, 8)
        assert torch.equal(x_aligned, y_aligned)
```
**EN:** This test method exercises s t squeeze full pipeline and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 s t squeeze full pipeline 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 520-542: Run test: s t fused full pipeline
```python
    def test_s_t_fused_full_pipeline(self) -> None:
        """x=tensor(4,128) dims='t (nh*hd)', y=tensor(4,1,8,16) dims='s 1 nh hd' → both (4,128)."""
        torch.manual_seed(42)
        num_heads: int = 8
        head_dim: int = 16
        data: torch.Tensor = torch.randn(4, num_heads * head_dim)
        x_tensor: torch.Tensor = data.clone()
        y_tensor: torch.Tensor = data.reshape(4, num_heads, head_dim).unsqueeze(1)

        plan: Optional[AxisAlignerPlan] = compute_axis_aligner_plan(
            Pair(x="t (nh*hd)", y="s 1 nh hd")
        )
        assert plan is not None

        x_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=x_tensor, plan=plan, side="x"
        )
        y_aligned: torch.Tensor = execute_axis_aligner_plan(
            tensor=y_tensor, plan=plan, side="y"
        )

        assert x_aligned.shape == y_aligned.shape == (4, num_heads * head_dim)
        assert torch.equal(x_aligned, y_aligned)
```
**EN:** This test method exercises s t fused full pipeline and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 s t fused full pipeline 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 545-546: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.axis_aligner`, `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`, `typing`
- Notable symbols / 关键符号: None / 无
