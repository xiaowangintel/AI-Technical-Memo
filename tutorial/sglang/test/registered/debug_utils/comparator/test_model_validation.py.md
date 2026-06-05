# test_model_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_model_validation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator model validation in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator model validation 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-45: Import dependencies
```python
import json
import sys

import pytest
from pydantic import ValidationError

from sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types import (
    TracedAlignerPlan,
    TracedSidePlan,
    TracedStepPlan,
    TracedSubPlan,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.types import (
    AlignerPerStepPlan,
    AlignerPlan,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    PositionalSeqId,
    TokenAlignerPlan,
    TokenAlignerSeqInfo,
    TokenAlignerStepAux,
    TokenLocator,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
    AxisInfo,
    ConcatParams,
    UnsharderPlan,
)
from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis, TokenLayout
from sglang.srt.debug_utils.comparator.output_types import (
    ComparisonErrorRecord,
    ComparisonNonTensorRecord,
    ComparisonSkipRecord,
    ComparisonTensorRecord,
    ErrorLog,
    SummaryRecord,
    parse_record_json,
)
from sglang.srt.debug_utils.comparator.tensor_comparator.types import (
    DiffInfo,
    TensorInfo,
    TensorStats,
)
from sglang.srt.debug_utils.comparator.utils import Pair, _check_equal_lengths
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 47-47: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 50-50: Define class TestCheckEqualLengths
```python
class TestCheckEqualLengths:
```
**EN:** This declaration introduces the `TestCheckEqualLengths` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCheckEqualLengths` 测试类，并说明它通过继承承担的职责。

### Lines 51-52: Run test: all equal
```python
    def test_all_equal(self):
        _check_equal_lengths(a=[1, 2], b=[3, 4])
```
**EN:** This test method exercises all equal and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 all equal 场景，并验证观测到的行为是否符合预期契约。

### Lines 54-55: Run test: empty lists
```python
    def test_empty_lists(self):
        _check_equal_lengths(a=[], b=[])
```
**EN:** This test method exercises empty lists and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 empty lists 场景，并验证观测到的行为是否符合预期契约。

### Lines 57-59: Run test: mismatch raises
```python
    def test_mismatch_raises(self):
        with pytest.raises(ValueError, match="Length mismatch"):
            _check_equal_lengths(a=[1, 2], b=[3])
```
**EN:** This test method exercises mismatch raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 mismatch raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 62-62: Define class TestTokenAlignerStepAux
```python
class TestTokenAlignerStepAux:
```
**EN:** This declaration introduces the `TestTokenAlignerStepAux` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTokenAlignerStepAux` 测试类，并说明它通过继承承担的职责。

### Lines 63-73: Run test: valid
```python
    def test_valid(self):
        aux = TokenAlignerStepAux(
            input_ids=[10, 20, 30],
            positions=[0, 1, 2],
            seq_lens=[2, 1],
            seq_ids=[
                PositionalSeqId(step=0, seq_index=0),
                PositionalSeqId(step=0, seq_index=1),
            ],
        )
        assert len(aux.input_ids) == 3
```
**EN:** This test method exercises valid and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 valid 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 75-85: Run test: token length mismatch
```python
    def test_token_length_mismatch(self):
        with pytest.raises(ValueError, match="Length mismatch"):
            TokenAlignerStepAux(
                input_ids=[10, 20, 30],
                positions=[0, 1],
                seq_lens=[2, 1],
                seq_ids=[
                    PositionalSeqId(step=0, seq_index=0),
                    PositionalSeqId(step=0, seq_index=1),
                ],
            )
```
**EN:** This test method exercises token length mismatch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 token length mismatch 场景，并验证观测到的行为是否符合预期契约。

### Lines 87-94: Run test: seq length mismatch
```python
    def test_seq_length_mismatch(self):
        with pytest.raises(ValueError, match="Length mismatch"):
            TokenAlignerStepAux(
                input_ids=[10, 20, 30],
                positions=[0, 1, 2],
                seq_lens=[2, 1],
                seq_ids=[PositionalSeqId(step=0, seq_index=0)],
            )
```
**EN:** This test method exercises seq length mismatch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seq length mismatch 场景，并验证观测到的行为是否符合预期契约。

### Lines 96-106: Run test: sum seq lens mismatch
```python
    def test_sum_seq_lens_mismatch(self):
        with pytest.raises(ValueError, match="sum\\(seq_lens\\)"):
            TokenAlignerStepAux(
                input_ids=[10, 20, 30],
                positions=[0, 1, 2],
                seq_lens=[1, 1],
                seq_ids=[
                    PositionalSeqId(step=0, seq_index=0),
                    PositionalSeqId(step=0, seq_index=1),
                ],
            )
```
**EN:** This test method exercises sum seq lens mismatch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 sum seq lens mismatch 场景，并验证观测到的行为是否符合预期契约。

### Lines 109-109: Define class TestTokenAlignerSeqInfo
```python
class TestTokenAlignerSeqInfo:
```
**EN:** This declaration introduces the `TestTokenAlignerSeqInfo` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTokenAlignerSeqInfo` 测试类，并说明它通过继承承担的职责。

### Lines 110-116: Run test: valid
```python
    def test_valid(self):
        info = TokenAlignerSeqInfo(
            input_ids=[10, 20, 30],
            positions=[0, 1, 2],
            locator=TokenLocator(steps=[0, 0, 1], token_index_in_step=[0, 1, 0]),
        )
        assert len(info.input_ids) == 3
```
**EN:** This test method exercises valid and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 valid 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 118-124: Run test: length mismatch
```python
    def test_length_mismatch(self):
        with pytest.raises(ValidationError):
            TokenAlignerSeqInfo(
                input_ids=[10, 20, 30],
                positions=[0, 1, 2],
                locator=TokenLocator(steps=[0, 0], token_index_in_step=[0, 1, 0]),
            )
```
**EN:** This test method exercises length mismatch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 length mismatch 场景，并验证观测到的行为是否符合预期契约。

### Lines 126-132: Run test: positions not sequential
```python
    def test_positions_not_sequential(self):
        with pytest.raises(ValidationError, match="positions must be"):
            TokenAlignerSeqInfo(
                input_ids=[10, 20, 30],
                positions=[0, 2, 1],
                locator=TokenLocator(steps=[0, 0, 1], token_index_in_step=[0, 1, 0]),
            )
```
**EN:** This test method exercises positions not sequential and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 positions not sequential 场景，并验证观测到的行为是否符合预期契约。

### Lines 135-135: Define class TestTokenAlignerPlan
```python
class TestTokenAlignerPlan:
```
**EN:** This declaration introduces the `TestTokenAlignerPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTokenAlignerPlan` 测试类，并说明它通过继承承担的职责。

### Lines 136-144: Run test: valid
```python
    def test_valid(self):
        plan = TokenAlignerPlan(
            locators=Pair(
                x=TokenLocator(steps=[0, 0, 1], token_index_in_step=[0, 1, 0]),
                y=TokenLocator(steps=[0, 1, 1], token_index_in_step=[0, 0, 1]),
            ),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
        )
        assert len(plan.locators.x.steps) == 3
```
**EN:** This test method exercises valid and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 valid 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 146-154: Run test: length mismatch
```python
    def test_length_mismatch(self):
        with pytest.raises(ValidationError, match="Length mismatch"):
            TokenAlignerPlan(
                locators=Pair(
                    x=TokenLocator(steps=[0, 0], token_index_in_step=[0, 1]),
                    y=TokenLocator(steps=[0, 1, 1], token_index_in_step=[0, 0, 1]),
                ),
                layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
            )
```
**EN:** This test method exercises length mismatch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 length mismatch 场景，并验证观测到的行为是否符合预期契约。

### Lines 157-157: Define class TestSummaryRecord
```python
class TestSummaryRecord:
```
**EN:** This declaration introduces the `TestSummaryRecord` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSummaryRecord` 测试类，并说明它通过继承承担的职责。

### Lines 158-160: Run test: valid
```python
    def test_valid(self):
        record = SummaryRecord(total=10, passed=7, failed=2, skipped=1)
        assert record.total == 10
```
**EN:** This test method exercises valid and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 valid 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 162-164: Run test: total mismatch
```python
    def test_total_mismatch(self):
        with pytest.raises(ValidationError, match="total=10"):
            SummaryRecord(total=10, passed=5, failed=2, skipped=1)
```
**EN:** This test method exercises total mismatch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 total mismatch 场景，并验证观测到的行为是否符合预期契约。

### Lines 166-168: Run test: valid with errored
```python
    def test_valid_with_errored(self):
        record = SummaryRecord(total=10, passed=6, failed=2, skipped=1, errored=1)
        assert record.errored == 1
```
**EN:** This test method exercises valid with errored and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 valid with errored 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 170-172: Run test: total mismatch with errored
```python
    def test_total_mismatch_with_errored(self):
        with pytest.raises(ValidationError, match="total=10"):
            SummaryRecord(total=10, passed=6, failed=2, skipped=1, errored=0)
```
**EN:** This test method exercises total mismatch with errored and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 total mismatch with errored 场景，并验证观测到的行为是否符合预期契约。

### Lines 175-175: Define class TestAxisInfo
```python
class TestAxisInfo:
```
**EN:** This declaration introduces the `TestAxisInfo` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAxisInfo` 测试类，并说明它通过继承承担的职责。

### Lines 176-178: Run test: valid
```python
    def test_valid(self):
        info = AxisInfo(axis_rank=0, axis_size=4)
        assert info.axis_rank == 0
```
**EN:** This test method exercises valid and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 valid 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 180-182: Run test: axis size zero
```python
    def test_axis_size_zero(self):
        with pytest.raises(ValidationError, match="axis_size must be > 0"):
            AxisInfo(axis_rank=0, axis_size=0)
```
**EN:** This test method exercises axis size zero and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 axis size zero 场景，并验证观测到的行为是否符合预期契约。

### Lines 184-186: Run test: axis size negative
```python
    def test_axis_size_negative(self):
        with pytest.raises(ValidationError, match="axis_size must be > 0"):
            AxisInfo(axis_rank=0, axis_size=-1)
```
**EN:** This test method exercises axis size negative and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 axis size negative 场景，并验证观测到的行为是否符合预期契约。

### Lines 188-190: Run test: axis rank negative
```python
    def test_axis_rank_negative(self):
        with pytest.raises(ValidationError, match="axis_rank must be in"):
            AxisInfo(axis_rank=-1, axis_size=4)
```
**EN:** This test method exercises axis rank negative and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 axis rank negative 场景，并验证观测到的行为是否符合预期契约。

### Lines 192-194: Run test: axis rank too large
```python
    def test_axis_rank_too_large(self):
        with pytest.raises(ValidationError, match="axis_rank must be in"):
            AxisInfo(axis_rank=4, axis_size=4)
```
**EN:** This test method exercises axis rank too large and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 axis rank too large 场景，并验证观测到的行为是否符合预期契约。

### Lines 196-198: Run test: axis rank equals size minus one
```python
    def test_axis_rank_equals_size_minus_one(self):
        info = AxisInfo(axis_rank=3, axis_size=4)
        assert info.axis_rank == 3
```
**EN:** This test method exercises axis rank equals size minus one and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 axis rank equals size minus one 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 201-206: Define helper: make tensor info
```python
def _make_tensor_info() -> TensorInfo:
    return TensorInfo(
        shape=[4, 4],
        dtype="float32",
        stats=TensorStats(mean=0.0, abs_mean=0.8, std=1.0, min=-2.0, max=2.0),
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 209-219: Define helper: make diff info
```python
def _make_diff_info(*, passed: bool) -> DiffInfo:
    return DiffInfo(
        rel_diff=0.001,
        max_abs_diff=0.01,
        mean_abs_diff=0.005,
        max_diff_coord=[0, 0],
        baseline_at_max=1.0,
        target_at_max=1.01,
        diff_threshold=1e-3,
        passed=passed,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 222-236: Define helper: make comparison record
```python
def _make_comparison_record(
    *,
    diff: DiffInfo | None,
    errors: list | None = None,
) -> ComparisonTensorRecord:
    ti: TensorInfo = _make_tensor_info()
    return ComparisonTensorRecord(
        name="t",
        baseline=ti,
        target=ti,
        unified_shape=[4, 4],
        shape_mismatch=False,
        diff=diff,
        errors=errors or [],
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 239-239: Define class TestOutputRecordCategories
```python
class TestOutputRecordCategories:
```
**EN:** This declaration introduces the `TestOutputRecordCategories` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOutputRecordCategories` 测试类，并说明它通过继承承担的职责。

### Lines 240-246: Run test: skip record with errors is failed
```python
    def test_skip_record_with_errors_is_failed(self) -> None:
        record = ComparisonSkipRecord(
            name="t",
            reason="test",
            errors=[ErrorLog(category="c", message="m")],
        )
        assert record.category == "failed"
```
**EN:** This test method exercises skip record with errors is failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 skip record with errors is failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 248-250: Run test: skip record no warnings is skipped
```python
    def test_skip_record_no_warnings_is_skipped(self) -> None:
        record = ComparisonSkipRecord(name="t", reason="test")
        assert record.category == "skipped"
```
**EN:** This test method exercises skip record no warnings is skipped and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 skip record no warnings is skipped 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 252-254: Run test: comparison record diff none is failed
```python
    def test_comparison_record_diff_none_is_failed(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(diff=None)
        assert record.category == "failed"
```
**EN:** This test method exercises comparison record diff none is failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 comparison record diff none is failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 256-261: Run test: comparison record passed with errors is failed
```python
    def test_comparison_record_passed_with_errors_is_failed(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff_info(passed=True),
            errors=[ErrorLog(category="c", message="m")],
        )
        assert record.category == "failed"
```
**EN:** This test method exercises comparison record passed with errors is failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 comparison record passed with errors is failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 263-267: Run test: comparison record passed no warnings is passed
```python
    def test_comparison_record_passed_no_warnings_is_passed(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff_info(passed=True),
        )
        assert record.category == "passed"
```
**EN:** This test method exercises comparison record passed no warnings is passed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 comparison record passed no warnings is passed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 269-278: Run test: non tensor record equal is passed
```python
    def test_non_tensor_record_equal_is_passed(self) -> None:
        record = ComparisonNonTensorRecord(
            name="sm_scale",
            baseline_value="0.125",
            target_value="0.125",
            baseline_type="float",
            target_type="float",
            values_equal=True,
        )
        assert record.category == "passed"
```
**EN:** This test method exercises non tensor record equal is passed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor record equal is passed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 280-289: Run test: non tensor record different is failed
```python
    def test_non_tensor_record_different_is_failed(self) -> None:
        record = ComparisonNonTensorRecord(
            name="sm_scale",
            baseline_value="0.125",
            target_value="0.25",
            baseline_type="float",
            target_type="float",
            values_equal=False,
        )
        assert record.category == "failed"
```
**EN:** This test method exercises non tensor record different is failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor record different is failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 291-301: Run test: non tensor record with errors is failed
```python
    def test_non_tensor_record_with_errors_is_failed(self) -> None:
        record = ComparisonNonTensorRecord(
            name="sm_scale",
            baseline_value="0.125",
            target_value="0.125",
            baseline_type="float",
            target_type="float",
            values_equal=True,
            errors=[ErrorLog(category="c", message="m")],
        )
        assert record.category == "failed"
```
**EN:** This test method exercises non tensor record with errors is failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor record with errors is failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 303-318: Run test: non tensor record json roundtrip
```python
    def test_non_tensor_record_json_roundtrip(self) -> None:
        record = ComparisonNonTensorRecord(
            name="sm_scale",
            baseline_value="0.125",
            target_value="0.25",
            baseline_type="float",
            target_type="float",
            values_equal=False,
        )
        json_str: str = record.model_dump_json()
        roundtripped = parse_record_json(json_str)
        assert isinstance(roundtripped, ComparisonNonTensorRecord)
        assert roundtripped.name == "sm_scale"
        assert roundtripped.values_equal is False
        assert roundtripped.baseline_value == "0.125"
        assert roundtripped.target_value == "0.25"
```
**EN:** This test method exercises non tensor record json roundtrip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor record json roundtrip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 320-331: Run test: non tensor record text format equal
```python
    def test_non_tensor_record_text_format_equal(self) -> None:
        record = ComparisonNonTensorRecord(
            name="sm_scale",
            baseline_value="0.125",
            target_value="0.125",
            baseline_type="float",
            target_type="float",
            values_equal=True,
        )
        text: str = record.to_text()
        assert "sm_scale" in text
        assert "[equal]" in text
```
**EN:** This test method exercises non tensor record text format equal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor record text format equal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 333-344: Run test: non tensor record text format different
```python
    def test_non_tensor_record_text_format_different(self) -> None:
        record = ComparisonNonTensorRecord(
            name="sm_scale",
            baseline_value="0.125",
            target_value="0.25",
            baseline_type="float",
            target_type="float",
            values_equal=False,
        )
        text: str = record.to_text()
        assert "baseline" in text
        assert "target" in text
```
**EN:** This test method exercises non tensor record text format different and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor record text format different 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 346-353: Run test: error record category is errored
```python
    def test_error_record_category_is_errored(self) -> None:
        record = ComparisonErrorRecord(
            name="t",
            exception_type="ValueError",
            exception_message="bad",
            traceback_str="...",
        )
        assert record.category == "errored"
```
**EN:** This test method exercises error record category is errored and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 error record category is errored 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 355-367: Run test: error record json roundtrip
```python
    def test_error_record_json_roundtrip(self) -> None:
        record = ComparisonErrorRecord(
            name="t",
            exception_type="ValueError",
            exception_message="bad",
            traceback_str="traceback...",
        )
        json_str: str = record.model_dump_json()
        roundtripped = parse_record_json(json_str)
        assert isinstance(roundtripped, ComparisonErrorRecord)
        assert roundtripped.name == "t"
        assert roundtripped.exception_type == "ValueError"
        assert roundtripped.exception_message == "bad"
```
**EN:** This test method exercises error record json roundtrip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 error record json roundtrip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 369-379: Run test: error record text format
```python
    def test_error_record_text_format(self) -> None:
        record = ComparisonErrorRecord(
            name="t",
            exception_type="RuntimeError",
            exception_message="oops",
            traceback_str="Traceback...",
        )
        text: str = record.to_text()
        assert "RuntimeError" in text
        assert "oops" in text
        assert "Traceback" in text
```
**EN:** This test method exercises error record text format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 error record text format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 382-412: Define helper: make traced aligner plan
```python
def _make_traced_aligner_plan() -> TracedAlignerPlan:
    unsharder = UnsharderPlan(
        axis=ParallelAxis.TP,
        params=ConcatParams(dim_name="h"),
        groups=[[0, 1]],
    )
    plan = AlignerPlan(
        per_step_plans=Pair(
            x=[
                AlignerPerStepPlan(
                    step=0, input_object_indices=[0, 1], sub_plans=[unsharder]
                )
            ],
            y=[
                AlignerPerStepPlan(
                    step=0, input_object_indices=[0, 1], sub_plans=[unsharder]
                )
            ],
        ),
    )
    traced_sub = TracedSubPlan(plan=unsharder, snapshot=None)
    traced_step = TracedStepPlan(
        step=0, input_object_indices=[0, 1], sub_plans=[traced_sub]
    )
    return TracedAlignerPlan(
        plan=plan,
        per_side=Pair(
            x=TracedSidePlan(step_plans=[traced_step]),
            y=TracedSidePlan(step_plans=[traced_step]),
        ),
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 415-415: Define class TestAlignerPlanInComparisonTensorRecord
```python
class TestAlignerPlanInComparisonTensorRecord:
```
**EN:** This declaration introduces the `TestAlignerPlanInComparisonTensorRecord` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAlignerPlanInComparisonTensorRecord` 测试类，并说明它通过继承承担的职责。

### Lines 416-423: Run test: comparison record with traced plan
```python
    def test_comparison_record_with_traced_plan(self) -> None:
        traced_plan: TracedAlignerPlan = _make_traced_aligner_plan()
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff_info(passed=True),
        )
        record_with_plan = record.model_copy(update={"traced_plan": traced_plan})
        assert record_with_plan.traced_plan is not None
        assert record_with_plan.traced_plan.per_side.x.step_plans[0].step == 0
```
**EN:** This test method exercises comparison record with traced plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 comparison record with traced plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 425-447: Run test: traced plan json roundtrip
```python
    def test_traced_plan_json_roundtrip(self) -> None:
        traced_plan: TracedAlignerPlan = _make_traced_aligner_plan()
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff_info(passed=True),
        )
        record_with_plan = record.model_copy(update={"traced_plan": traced_plan})

        json_str: str = record_with_plan.model_dump_json()
        parsed = json.loads(json_str)
        assert "traced_plan" in parsed
        assert (
            parsed["traced_plan"]["per_side"]["x"]["step_plans"][0]["sub_plans"][0][
                "plan"
            ]["type"]
            == "unsharder"
        )

        roundtripped: ComparisonTensorRecord = parse_record_json(json_str)
        assert roundtripped.traced_plan is not None
        assert (
            roundtripped.traced_plan.per_side.x.step_plans[0].sub_plans[0].plan.type
            == "unsharder"
        )
```
**EN:** This test method exercises traced plan json roundtrip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 traced plan json roundtrip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 449-455: Run test: comparison record without traced plan
```python
    def test_comparison_record_without_traced_plan(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff_info(passed=True),
        )
        json_str: str = record.model_dump_json()
        roundtripped: ComparisonTensorRecord = parse_record_json(json_str)
        assert roundtripped.traced_plan is None
```
**EN:** This test method exercises comparison record without traced plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 comparison record without traced plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 457-466: Run test: traced plan text format
```python
    def test_traced_plan_text_format(self) -> None:
        traced_plan: TracedAlignerPlan = _make_traced_aligner_plan()
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff_info(passed=True),
        )
        record_with_plan = record.model_copy(update={"traced_plan": traced_plan})

        text: str = record_with_plan.to_text()
        assert "Aligner Plan:" in text
        assert "unsharder" in text
```
**EN:** This test method exercises traced plan text format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 traced plan text format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 469-470: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.tensor_comparator.types`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `json`, `pydantic`, `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
