# test_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator utils in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator utils 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Import dependencies
```python
import sys
from pathlib import Path

import pytest
import torch

from sglang.srt.debug_utils.comparator.output_types import SummaryRecord
from sglang.srt.debug_utils.comparator.utils import (
    Pair,
    argmax_coord,
    auto_descend_dir,
    calc_per_token_rel_diff,
    calc_rel_diff,
    compute_exit_code,
    compute_smaller_dtype,
    try_unify_shape,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 20-20: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 23-23: Define class TestCalcRelDiff
```python
class TestCalcRelDiff:
```
**EN:** This declaration introduces the `TestCalcRelDiff` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCalcRelDiff` 测试类，并说明它通过继承承担的职责。

### Lines 24-26: Run test: identical tensors
```python
    def test_identical_tensors(self):
        x = torch.randn(10, 10)
        assert calc_rel_diff(x, x).item() == pytest.approx(0.0, abs=1e-5)
```
**EN:** This test method exercises identical tensors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 identical tensors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 28-32: Run test: orthogonal tensors
```python
    def test_orthogonal_tensors(self):
        result = calc_rel_diff(
            torch.tensor([1.0, 0.0]), torch.tensor([0.0, 1.0])
        ).item()
        assert result == pytest.approx(1.0, abs=1e-5)
```
**EN:** This test method exercises orthogonal tensors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 orthogonal tensors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 34-38: Run test: similar tensors
```python
    def test_similar_tensors(self):
        x = torch.tensor([1.0, 2.0, 3.0])
        y = torch.tensor([1.01, 2.01, 3.01])
        result = calc_rel_diff(x, y).item()
        assert 0.0 < result < 0.01
```
**EN:** This test method exercises similar tensors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 similar tensors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 40-43: Run test: negated tensors
```python
    def test_negated_tensors(self):
        x = torch.tensor([1.0, 2.0])
        result = calc_rel_diff(x, -x).item()
        assert result == pytest.approx(2.0, abs=1e-5)
```
**EN:** This test method exercises negated tensors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 negated tensors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 46-46: Define class TestCalcPerTokenRelDiff
```python
class TestCalcPerTokenRelDiff:
```
**EN:** This declaration introduces the `TestCalcPerTokenRelDiff` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCalcPerTokenRelDiff` 测试类，并说明它通过继承承担的职责。

### Lines 47-53: Run test: identical tensors
```python
    def test_identical_tensors(self) -> None:
        """Identical tensors → per-token diff all zero."""
        x: torch.Tensor = torch.randn(8, 16)
        result: torch.Tensor = calc_per_token_rel_diff(x, x, seq_dim=0)

        assert result.shape == (8,)
        assert torch.allclose(result, torch.zeros(8), atol=1e-6)
```
**EN:** This test method exercises identical tensors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 identical tensors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 55-68: Run test: different tensors
```python
    def test_different_tensors(self) -> None:
        """Single token position differs → that position has higher diff."""
        torch.manual_seed(42)
        x: torch.Tensor = torch.randn(8, 16)
        y: torch.Tensor = x.clone()
        y[3, :] += 10.0

        result: torch.Tensor = calc_per_token_rel_diff(x, y, seq_dim=0)

        assert result.shape == (8,)
        assert result[3] > result[0]
        assert result[3] > result[7]
        for i in [0, 1, 2, 4, 5, 6, 7]:
            assert result[i] < 1e-6
```
**EN:** This test method exercises different tensors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 different tensors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 70-77: Run test: seq dim selection
```python
    def test_seq_dim_selection(self) -> None:
        """Different seq_dim values produce correct output shapes."""
        x: torch.Tensor = torch.randn(4, 8, 16)
        y: torch.Tensor = x + torch.randn_like(x) * 0.01

        assert calc_per_token_rel_diff(x, y, seq_dim=0).shape == (4,)
        assert calc_per_token_rel_diff(x, y, seq_dim=1).shape == (8,)
        assert calc_per_token_rel_diff(x, y, seq_dim=2).shape == (16,)
```
**EN:** This test method exercises seq dim selection and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 seq dim selection 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 79-89: Run test: 1d tensor
```python
    def test_1d_tensor(self) -> None:
        """1D tensor with seq_dim=0 returns per-element diff."""
        x: torch.Tensor = torch.tensor([1.0, 2.0, 3.0])
        y: torch.Tensor = torch.tensor([1.0, 2.0, 4.0])

        result: torch.Tensor = calc_per_token_rel_diff(x, y, seq_dim=0)

        assert result.shape == (3,)
        assert result[0] < 1e-6
        assert result[1] < 1e-6
        assert result[2] > 0.01
```
**EN:** This test method exercises 1d tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 1d tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 92-92: Define class TestArgmaxCoord
```python
class TestArgmaxCoord:
```
**EN:** This declaration introduces the `TestArgmaxCoord` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestArgmaxCoord` 测试类，并说明它通过继承承担的职责。

### Lines 93-95: Run test: 1d tensor
```python
    def test_1d_tensor(self):
        x = torch.tensor([0.0, 0.0, 5.0, 0.0])
        assert argmax_coord(x) == (2,)
```
**EN:** This test method exercises 1d tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 1d tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 97-100: Run test: 2d tensor
```python
    def test_2d_tensor(self):
        x = torch.zeros(3, 4)
        x[1, 2] = 10.0
        assert argmax_coord(x) == (1, 2)
```
**EN:** This test method exercises 2d tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 2d tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 102-105: Run test: 3d tensor
```python
    def test_3d_tensor(self):
        x = torch.zeros(2, 3, 4)
        x[1, 2, 3] = 10.0
        assert argmax_coord(x) == (1, 2, 3)
```
**EN:** This test method exercises 3d tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 3d tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 108-108: Define class TestTryUnifyShape
```python
class TestTryUnifyShape:
```
**EN:** This declaration introduces the `TestTryUnifyShape` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTryUnifyShape` 测试类，并说明它通过继承承担的职责。

### Lines 109-111: Run test: squeeze leading ones
```python
    def test_squeeze_leading_ones(self):
        target = torch.Size([3, 4])
        assert try_unify_shape(torch.randn(1, 1, 3, 4), target).shape == target
```
**EN:** This test method exercises squeeze leading ones and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 squeeze leading ones 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 113-115: Run test: no squeeze when leading dim not one
```python
    def test_no_squeeze_when_leading_dim_not_one(self):
        target = torch.Size([3, 4])
        assert try_unify_shape(torch.randn(2, 3, 4), target).shape == (2, 3, 4)
```
**EN:** This test method exercises no squeeze when leading dim not one and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no squeeze when leading dim not one 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 117-122: Run test: same shape noop
```python
    def test_same_shape_noop(self):
        target = torch.Size([3, 4])
        x = torch.randn(3, 4)
        result = try_unify_shape(x, target)
        assert result.shape == target
        assert result.data_ptr() == x.data_ptr()
```
**EN:** This test method exercises same shape noop and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 same shape noop 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 124-128: Run test: trailing dims mismatch
```python
    def test_trailing_dims_mismatch(self):
        target = torch.Size([5, 6])
        x = torch.randn(1, 3, 4)
        result = try_unify_shape(x, target)
        assert result.shape == (1, 3, 4)
```
**EN:** This test method exercises trailing dims mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 trailing dims mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 131-131: Define class TestComputeSmallerDtype
```python
class TestComputeSmallerDtype:
```
**EN:** This declaration introduces the `TestComputeSmallerDtype` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeSmallerDtype` 测试类，并说明它通过继承承担的职责。

### Lines 132-136: Run test: float32 bfloat16
```python
    def test_float32_bfloat16(self):
        assert (
            compute_smaller_dtype(Pair(x=torch.float32, y=torch.bfloat16))
            == torch.bfloat16
        )
```
**EN:** This test method exercises float32 bfloat16 and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 float32 bfloat16 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 138-142: Run test: reverse order
```python
    def test_reverse_order(self):
        assert (
            compute_smaller_dtype(Pair(x=torch.bfloat16, y=torch.float32))
            == torch.bfloat16
        )
```
**EN:** This test method exercises reverse order and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reverse order 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 144-145: Run test: same dtype returns none
```python
    def test_same_dtype_returns_none(self):
        assert compute_smaller_dtype(Pair(x=torch.float32, y=torch.float32)) is None
```
**EN:** This test method exercises same dtype returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 same dtype returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 147-148: Run test: unknown pair returns none
```python
    def test_unknown_pair_returns_none(self):
        assert compute_smaller_dtype(Pair(x=torch.int32, y=torch.int64)) is None
```
**EN:** This test method exercises unknown pair returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 unknown pair returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 151-151: Define class TestPairMap
```python
class TestPairMap:
```
**EN:** This declaration introduces the `TestPairMap` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPairMap` 测试类，并说明它通过继承承担的职责。

### Lines 152-156: Run test: map basic
```python
    def test_map_basic(self):
        pair = Pair(x=[1, 2, 3], y=[4, 5, 6])
        result = pair.map(lambda lst: sum(lst))
        assert result.x == 6
        assert result.y == 15
```
**EN:** This test method exercises map basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 map basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 158-162: Run test: map type change
```python
    def test_map_type_change(self):
        pair = Pair(x=[1, 2, 3], y=[10, 20])
        result = pair.map(len)
        assert result.x == 3
        assert result.y == 2
```
**EN:** This test method exercises map type change and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 map type change 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 164-169: Run test: map returns new pair
```python
    def test_map_returns_new_pair(self):
        pair = Pair(x="hello", y="world")
        result = pair.map(str.upper)
        assert result.x == "HELLO"
        assert result.y == "WORLD"
        assert result is not pair
```
**EN:** This test method exercises map returns new pair and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 map returns new pair 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 172-172: Define class TestComputeExitCode
```python
class TestComputeExitCode:
```
**EN:** This declaration introduces the `TestComputeExitCode` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeExitCode` 测试类，并说明它通过继承承担的职责。

### Lines 173-173: Document the class `TestComputeExitCode`
```python
    """Unit tests for compute_exit_code logic."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestComputeExitCode`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestComputeExitCode`的设计意图。

### Lines 175-187: Run test: all passed
```python
    def test_all_passed(self):
        """All passed → exit 0."""
        summary = SummaryRecord(total=3, passed=3, failed=0, skipped=0)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern=None,
                failed_names=[],
            )
            == 0
        )
```
**EN:** This test method exercises all passed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all passed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 189-201: Run test: has failed and passed
```python
    def test_has_failed_and_passed(self):
        """Has failed and passed → exit 1."""
        summary = SummaryRecord(total=4, passed=2, failed=2, skipped=0)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern=None,
                failed_names=["a", "b"],
            )
            == 1
        )
```
**EN:** This test method exercises has failed and passed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 has failed and passed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 203-215: Run test: all failed
```python
    def test_all_failed(self):
        """All failed (0 passed) → exit 1."""
        summary = SummaryRecord(total=3, passed=0, failed=3, skipped=0)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern=None,
                failed_names=["a", "b", "c"],
            )
            == 1
        )
```
**EN:** This test method exercises all failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 217-229: Run test: all skipped allow all
```python
    def test_all_skipped_allow_all(self):
        """All skipped + allow_skipped_pattern='.*' → exit 1 (nothing passed)."""
        summary = SummaryRecord(total=2, passed=0, failed=0, skipped=2)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=["a", "b"],
                allow_failed_pattern=None,
                failed_names=[],
            )
            == 1
        )
```
**EN:** This test method exercises all skipped allow all and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all skipped allow all 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 231-243: Run test: all skipped forbid all
```python
    def test_all_skipped_forbid_all(self):
        """All skipped + allow_skipped_pattern='^$' → exit 1."""
        summary = SummaryRecord(total=2, passed=0, failed=0, skipped=2)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern="^$",
                skipped_names=["a", "b"],
                allow_failed_pattern=None,
                failed_names=[],
            )
            == 1
        )
```
**EN:** This test method exercises all skipped forbid all and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all skipped forbid all 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 245-257: Run test: passed and skipped allow all
```python
    def test_passed_and_skipped_allow_all(self):
        """Passed + skipped, allow all → exit 0."""
        summary = SummaryRecord(total=3, passed=2, failed=0, skipped=1)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=["a"],
                allow_failed_pattern=None,
                failed_names=[],
            )
            == 0
        )
```
**EN:** This test method exercises passed and skipped allow all and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 passed and skipped allow all 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 259-271: Run test: passed and skipped forbid all
```python
    def test_passed_and_skipped_forbid_all(self):
        """Passed + skipped + forbid all → exit 1."""
        summary = SummaryRecord(total=3, passed=2, failed=0, skipped=1)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern="^$",
                skipped_names=["a"],
                allow_failed_pattern=None,
                failed_names=[],
            )
            == 1
        )
```
**EN:** This test method exercises passed and skipped forbid all and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 passed and skipped forbid all 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 273-285: Run test: skip pattern matches specific name
```python
    def test_skip_pattern_matches_specific_name(self):
        """Pattern matching specific name allows that skip, forbids others."""
        summary = SummaryRecord(total=4, passed=2, failed=0, skipped=2)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern="positions|seq_lens",
                skipped_names=["positions", "seq_lens"],
                allow_failed_pattern=None,
                failed_names=[],
            )
            == 0
        )
```
**EN:** This test method exercises skip pattern matches specific name and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 skip pattern matches specific name 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 287-299: Run test: skip pattern partial match forbidden
```python
    def test_skip_pattern_partial_match_forbidden(self):
        """Pattern matches some skips but not all → exit 1."""
        summary = SummaryRecord(total=4, passed=1, failed=0, skipped=3)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern="positions|seq_lens",
                skipped_names=["positions", "seq_lens", "hidden_states"],
                allow_failed_pattern=None,
                failed_names=[],
            )
            == 1
        )
```
**EN:** This test method exercises skip pattern partial match forbidden and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 skip pattern partial match forbidden 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 301-313: Run test: allow failed pattern matches all
```python
    def test_allow_failed_pattern_matches_all(self):
        """allow_failed_pattern='.*' tolerates all failures → exit 0."""
        summary = SummaryRecord(total=3, passed=1, failed=2, skipped=0)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern=".*",
                failed_names=["a", "b"],
            )
            == 0
        )
```
**EN:** This test method exercises allow failed pattern matches all and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 allow failed pattern matches all 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 315-327: Run test: allow failed pattern matches specific
```python
    def test_allow_failed_pattern_matches_specific(self):
        """Pattern matches all failed names → exit 0."""
        summary = SummaryRecord(total=3, passed=1, failed=2, skipped=0)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern="hidden_states|logits",
                failed_names=["hidden_states", "logits"],
            )
            == 0
        )
```
**EN:** This test method exercises allow failed pattern matches specific and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 allow failed pattern matches specific 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 329-341: Run test: allow failed pattern partial match
```python
    def test_allow_failed_pattern_partial_match(self):
        """Pattern matches some but not all failures → exit 1."""
        summary = SummaryRecord(total=3, passed=0, failed=3, skipped=0)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern="hidden_states",
                failed_names=["hidden_states", "logits", "attn"],
            )
            == 1
        )
```
**EN:** This test method exercises allow failed pattern partial match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 allow failed pattern partial match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 343-355: Run test: allow failed pattern no failures
```python
    def test_allow_failed_pattern_no_failures(self):
        """Pattern set but no failures → exit 0."""
        summary = SummaryRecord(total=2, passed=2, failed=0, skipped=0)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern=".*",
                failed_names=[],
            )
            == 0
        )
```
**EN:** This test method exercises allow failed pattern no failures and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 allow failed pattern no failures 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 357-369: Run test: both failed and skipped patterns
```python
    def test_both_failed_and_skipped_patterns(self):
        """Both patterns set, both satisfied → exit 0."""
        summary = SummaryRecord(total=4, passed=1, failed=1, skipped=2)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern="positions|seq_lens",
                skipped_names=["positions", "seq_lens"],
                allow_failed_pattern="logits",
                failed_names=["logits"],
            )
            == 0
        )
```
**EN:** This test method exercises both failed and skipped patterns and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 both failed and skipped patterns 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 371-383: Run test: failed pattern satisfied but skipped not
```python
    def test_failed_pattern_satisfied_but_skipped_not(self):
        """Failed pattern OK but skipped pattern fails → exit 1."""
        summary = SummaryRecord(total=3, passed=1, failed=1, skipped=1)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern="^$",
                skipped_names=["a"],
                allow_failed_pattern=".*",
                failed_names=["b"],
            )
            == 1
        )
```
**EN:** This test method exercises failed pattern satisfied but skipped not and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 failed pattern satisfied but skipped not 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 385-397: Run test: zero passed exits one
```python
    def test_zero_passed_exits_one(self):
        """No tensors passed → exit 1, even when all failures are allowed."""
        summary = SummaryRecord(total=2, passed=0, failed=2, skipped=0)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern=".*",
                failed_names=["a", "b"],
            )
            == 1
        )
```
**EN:** This test method exercises zero passed exits one and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zero passed exits one 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 399-411: Run test: zero passed all skipped exits one
```python
    def test_zero_passed_all_skipped_exits_one(self):
        """All skipped, nothing passed → exit 1."""
        summary = SummaryRecord(total=3, passed=0, failed=0, skipped=3)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=["a", "b", "c"],
                allow_failed_pattern=None,
                failed_names=[],
            )
            == 1
        )
```
**EN:** This test method exercises zero passed all skipped exits one and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zero passed all skipped exits one 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 413-426: Run test: errored with passed exits one
```python
    def test_errored_with_passed_exits_one(self):
        """Has errored bundle even with passed → exit 1."""
        summary = SummaryRecord(total=3, passed=2, failed=0, skipped=0, errored=1)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern=None,
                failed_names=[],
                errored_names=["broken_tensor"],
            )
            == 1
        )
```
**EN:** This test method exercises errored with passed exits one and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 errored with passed exits one 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 428-441: Run test: errored only exits one
```python
    def test_errored_only_exits_one(self):
        """All errored → exit 1 (passed==0 already exits 1, but errored also independently triggers)."""
        summary = SummaryRecord(total=1, passed=0, failed=0, skipped=0, errored=1)
        assert (
            compute_exit_code(
                summary,
                allow_skipped_pattern=".*",
                skipped_names=[],
                allow_failed_pattern=None,
                failed_names=[],
                errored_names=["broken_tensor"],
            )
            == 1
        )
```
**EN:** This test method exercises errored only exits one and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 errored only exits one 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 444-446: Define helper: make pt
```python
def _make_pt(directory: Path) -> None:
    directory.mkdir(parents=True, exist_ok=True)
    torch.save(torch.tensor([1.0]), directory / "dummy.pt")
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 449-449: Define class TestAutoDescendDir
```python
class TestAutoDescendDir:
```
**EN:** This declaration introduces the `TestAutoDescendDir` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAutoDescendDir` 测试类，并说明它通过继承承担的职责。

### Lines 450-454: Run test: no descend when pt at root
```python
    def test_no_descend_when_pt_at_root(self, tmp_path: Path) -> None:
        """Directory with .pt files directly is returned as-is."""
        _make_pt(tmp_path)
        _make_pt(tmp_path / "child_a")
        assert auto_descend_dir(tmp_path, label="test") == tmp_path
```
**EN:** This test method exercises no descend when pt at root and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no descend when pt at root 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 456-460: Run test: descend into single child
```python
    def test_descend_into_single_child(self, tmp_path: Path) -> None:
        """Single child with .pt triggers descend."""
        child: Path = tmp_path / "engine_0"
        _make_pt(child)
        assert auto_descend_dir(tmp_path, label="test") == child
```
**EN:** This test method exercises descend into single child and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 descend into single child 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 462-467: Run test: descend single nonempty child among empty
```python
    def test_descend_single_nonempty_child_among_empty(self, tmp_path: Path) -> None:
        """Two subdirs but only one has .pt — descend into that one."""
        nonempty: Path = tmp_path / "engine_0"
        _make_pt(nonempty)
        (tmp_path / "empty_child").mkdir()
        assert auto_descend_dir(tmp_path, label="test") == nonempty
```
**EN:** This test method exercises descend single nonempty child among empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 descend single nonempty child among empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 469-474: Run test: error with multiple nonempty children
```python
    def test_error_with_multiple_nonempty_children(self, tmp_path: Path) -> None:
        """Two children with .pt files — ambiguous, raises ValueError."""
        _make_pt(tmp_path / "engine_0")
        _make_pt(tmp_path / "engine_1")
        with pytest.raises(ValueError, match="multiple subdirectories contain data"):
            auto_descend_dir(tmp_path, label="test")
```
**EN:** This test method exercises error with multiple nonempty children and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 error with multiple nonempty children 场景，并验证观测到的行为是否符合预期契约。

### Lines 476-480: Run test: error when no data found
```python
    def test_error_when_no_data_found(self, tmp_path: Path) -> None:
        """No .pt files anywhere — raises ValueError."""
        (tmp_path / "empty_child").mkdir()
        with pytest.raises(ValueError, match="no .pt files found"):
            auto_descend_dir(tmp_path, label="test")
```
**EN:** This test method exercises error when no data found and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 error when no data found 场景，并验证观测到的行为是否符合预期契约。

### Lines 483-484: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pathlib`, `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
