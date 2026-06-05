# test_output_types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_output_types.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator output types in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator output types 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-56: Import dependencies
```python
import sys
from io import StringIO

import pytest
from registered.debug_utils.comparator.testing_helpers import (
    assert_rich_tags_balanced,
)
from registered.debug_utils.comparator.testing_helpers import make_diff as _make_diff
from registered.debug_utils.comparator.testing_helpers import make_stats as _make_stats
from registered.debug_utils.comparator.testing_helpers import (
    make_tensor_info as _make_tensor_info,
)
from rich.console import Console, Group
from rich.panel import Panel

from sglang.srt.debug_utils.comparator.aligner.axis_aligner import AxisAlignerPlan
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
from sglang.srt.debug_utils.comparator.aligner.reorderer.types import (
    ReordererPlan,
    ZigzagToNaturalParams,
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
from sglang.srt.debug_utils.comparator.output_types import (
    BundleFileInfo,
    BundleSideInfo,
    ComparisonNonTensorRecord,
    ComparisonSkipRecord,
    ComparisonTensorRecord,
    ConfigRecord,
    ErrorLog,
    InfoLog,
    LogRecord,
    RecordLocation,
    SummaryRecord,
    _format_aligner_plan,
    _split_logs,
)
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also checks concrete expectations with assertions.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会通过断言检查明确的预期。

### Lines 58-58: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 61-64: Define helper: render rich
```python
def _render_rich(renderable: object) -> str:
    buf: StringIO = StringIO()
    Console(file=buf, force_terminal=False, width=120).print(renderable)
    return buf.getvalue().rstrip("\n")
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 72-85: Run test: split logs mixed list
```python
def test_split_logs_mixed_list() -> None:
    """_split_logs correctly partitions a mixed list of ErrorLog and InfoLog."""
    errors, infos = _split_logs(
        [
            ErrorLog(category="a", message="err"),
            InfoLog(category="b", message="info"),
            ErrorLog(category="c", message="err2"),
        ]
    )
    assert len(errors) == 2
    assert len(infos) == 1
    assert errors[0].message == "err"
    assert errors[1].message == "err2"
    assert infos[0].message == "info"
```
**EN:** This test method exercises split logs mixed list and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 split logs mixed list 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 88-96: Run test: log record to text format
```python
def test_log_record_to_text_format() -> None:
    """LogRecord.to_text() renders errors with ✗ and infos with ℹ markers."""
    record = LogRecord(
        errors=[ErrorLog(category="a", message="bad thing")],
        infos=[InfoLog(category="b", message="fyi")],
    )
    text: str = record.to_text()
    assert "✗ bad thing" in text
    assert "ℹ fyi" in text
```
**EN:** This test method exercises log record to text format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 log record to text format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 99-99: Define class TestLogRecord
```python
class TestLogRecord:
```
**EN:** This declaration introduces the `TestLogRecord` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLogRecord` 测试类，并说明它通过继承承担的职责。

### Lines 100-102: Run test: format body returns empty
```python
    def test_format_body_returns_empty(self) -> None:
        record: LogRecord = LogRecord()
        assert record._format_body() == ""
```
**EN:** This test method exercises format body returns empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format body returns empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 104-106: Run test: format rich body returns empty
```python
    def test_format_rich_body_returns_empty(self) -> None:
        record: LogRecord = LogRecord()
        assert record._format_rich_body() == ""
```
**EN:** This test method exercises format rich body returns empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format rich body returns empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 108-110: Run test: to text empty no logs
```python
    def test_to_text_empty_no_logs(self) -> None:
        record: LogRecord = LogRecord()
        assert record.to_text() == ""
```
**EN:** This test method exercises to text empty no logs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to text empty no logs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 112-118: Run test: to text with errors and infos
```python
    def test_to_text_with_errors_and_infos(self) -> None:
        record: LogRecord = LogRecord(
            errors=[ErrorLog(category="a", message="bad thing")],
            infos=[InfoLog(category="b", message="fyi")],
        )
        text: str = record.to_text()
        assert text == "\n  ✗ bad thing\n  ℹ fyi"
```
**EN:** This test method exercises to text with errors and infos and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to text with errors and infos 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 126-126: Define class TestConfigRecord
```python
class TestConfigRecord:
```
**EN:** This declaration introduces the `TestConfigRecord` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestConfigRecord` 测试类，并说明它通过继承承担的职责。

### Lines 127-129: Run test: format body
```python
    def test_format_body(self) -> None:
        record: ConfigRecord = ConfigRecord(config={"a": 1, "b": "two"})
        assert record._format_body() == "Config: {'a': 1, 'b': 'two'}"
```
**EN:** This test method exercises format body and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format body 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 131-146: Run test: format rich body
```python
    def test_format_rich_body(self) -> None:
        record: ConfigRecord = ConfigRecord(config={"threshold": 0.001, "mode": "fast"})
        body = record._format_rich_body()

        assert isinstance(body, Panel)
        rendered: str = _render_rich(body)
        assert rendered == (
            "╭───────────────────────────────────────────────── Comparator Config "
            "──────────────────────────────────────────────────╮\n"
            "│   threshold : 0.001"
            "                                                                                                  │\n"
            "│   mode : fast"
            "                                                                                                        │\n"
            "╰──────────────────────────────────────────────────────────────────────"
            "────────────────────────────────────────────────╯"
        )
```
**EN:** This test method exercises format rich body and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format rich body 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 148-155: Run test: to text with errors
```python
    def test_to_text_with_errors(self) -> None:
        record: ConfigRecord = ConfigRecord(
            config={"x": 1},
            errors=[ErrorLog(category="cfg", message="bad config")],
        )
        text: str = record.to_text()
        assert text.startswith("Config: {'x': 1}")
        assert "✗ bad config" in text
```
**EN:** This test method exercises to text with errors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to text with errors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 163-163: Define class TestComparisonSkipRecord
```python
class TestComparisonSkipRecord:
```
**EN:** This declaration introduces the `TestComparisonSkipRecord` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComparisonSkipRecord` 测试类，并说明它通过继承承担的职责。

### Lines 164-169: Run test: format body no step
```python
    def test_format_body_no_step(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="layer.weight",
            reason="zero-dim tensor",
        )
        assert record._format_body() == "Skip: layer.weight (zero-dim tensor)"
```
**EN:** This test method exercises format body no step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format body no step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 171-177: Run test: format body with step
```python
    def test_format_body_with_step(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="layer.weight",
            reason="scalar",
            location=RecordLocation(step=3),
        )
        assert record._format_body() == "Skip: layer.weight (step=3) (scalar)"
```
**EN:** This test method exercises format body with step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format body with step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 179-185: Run test: format rich body
```python
    def test_format_rich_body(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="attn.qkv",
            reason="no baseline",
        )
        body: str = record._format_rich_body()
        assert body == "[dim]⊘ attn.qkv ── skipped (no baseline)[/]"
```
**EN:** This test method exercises format rich body and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format rich body 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 187-192: Run test: category skipped
```python
    def test_category_skipped(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="x",
            reason="r",
        )
        assert record.category == "skipped"
```
**EN:** This test method exercises category skipped and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 category skipped 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 194-200: Run test: category failed
```python
    def test_category_failed(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="x",
            reason="r",
            errors=[ErrorLog(category="e", message="boom")],
        )
        assert record.category == "failed"
```
**EN:** This test method exercises category failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 category failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 202-218: Run test: format body with available side
```python
    def test_format_body_with_available_side(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="layer.weight",
            reason="baseline_load_failed",
            available_side="target",
            available_tensor_info=_make_tensor_info(
                shape=[4, 8],
                dtype="torch.float32",
                stats=_make_stats(mean=0.5, std=1.2, min=-2.0, max=3.0),
                sample="tensor([0.1, 0.2, ...])",
            ),
        )
        body: str = record._format_body()
        assert "baseline_load_failed" in body
        assert "target: shape=[4, 8]" in body
        assert "mean=0.5000" in body
        assert "sample: tensor([0.1, 0.2, ...])" in body
```
**EN:** This test method exercises format body with available side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format body with available side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 220-245: Run test: format rich body with available side
```python
    def test_format_rich_body_with_available_side(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="attn.qkv",
            reason="baseline_load_failed",
            available_side="target",
            available_tensor_info=_make_tensor_info(
                shape=[4, 8],
                dtype="torch.float32",
                stats=_make_stats(mean=0.5, std=1.2, min=-2.0, max=3.0),
                sample="tensor([0.1, 0.2, ...])",
            ),
            available_bundle_info=BundleSideInfo(
                num_files=2,
                files=[
                    BundleFileInfo(shape=[4, 8], dtype="torch.float32"),
                    BundleFileInfo(shape=[4, 8], dtype="torch.float32"),
                ],
            ),
        )
        body: str = record._format_rich_body()
        assert "skipped (baseline_load_failed)" in body
        assert "target" in body
        assert "2 files" in body
        assert "mean=0.5000" in body
        assert "tensor(" in body
        assert_rich_tags_balanced(body)
```
**EN:** This test method exercises format rich body with available side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format rich body with available side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 247-257: Run test: format rich body minimal hides available side
```python
    def test_format_rich_body_minimal_hides_available_side(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="x",
            reason="target_load_failed",
            available_side="baseline",
            available_tensor_info=_make_tensor_info(),
        )
        body: str = record._format_rich_body(verbosity="minimal")
        assert "skipped" in body
        assert "stats" not in body
        assert_rich_tags_balanced(body)
```
**EN:** This test method exercises format rich body minimal hides available side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format rich body minimal hides available side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 265-265: Define class TestComparisonNonTensorRecord
```python
class TestComparisonNonTensorRecord:
```
**EN:** This declaration introduces the `TestComparisonNonTensorRecord` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComparisonNonTensorRecord` 测试类，并说明它通过继承承担的职责。

### Lines 266-275: Run test: format body equal
```python
    def test_format_body_equal(self) -> None:
        record: ComparisonNonTensorRecord = ComparisonNonTensorRecord(
            name="config.lr",
            baseline_value="0.001",
            target_value="0.001",
            baseline_type="float",
            target_type="float",
            values_equal=True,
        )
        assert record._format_body() == "NonTensor: config.lr = 0.001 (float) [equal]"
```
**EN:** This test method exercises format body equal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format body equal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 277-290: Run test: format body not equal
```python
    def test_format_body_not_equal(self) -> None:
        record: ComparisonNonTensorRecord = ComparisonNonTensorRecord(
            name="config.lr",
            baseline_value="0.001",
            target_value="0.01",
            baseline_type="float",
            target_type="float",
            values_equal=False,
        )
        assert record._format_body() == (
            "NonTensor: config.lr\n"
            "  baseline = 0.001 (float)\n"
            "  target   = 0.01 (float)"
        )
```
**EN:** This test method exercises format body not equal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format body not equal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 292-301: Run test: format rich body equal
```python
    def test_format_rich_body_equal(self) -> None:
        record: ComparisonNonTensorRecord = ComparisonNonTensorRecord(
            name="config.lr",
            baseline_value="0.001",
            target_value="0.001",
            baseline_type="float",
            target_type="float",
            values_equal=True,
        )
        assert record._format_rich_body() == ("═ config.lr = 0.001 (float) [green]✓[/]")
```
**EN:** This test method exercises format rich body equal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format rich body equal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 303-316: Run test: format rich body not equal
```python
    def test_format_rich_body_not_equal(self) -> None:
        record: ComparisonNonTensorRecord = ComparisonNonTensorRecord(
            name="config.lr",
            baseline_value="0.001",
            target_value="0.01",
            baseline_type="float",
            target_type="float",
            values_equal=False,
        )
        assert record._format_rich_body() == (
            "═ [bold red]config.lr[/]\n"
            "  baseline = 0.001 (float)\n"
            "  target   = 0.01 (float)"
        )
```
**EN:** This test method exercises format rich body not equal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format rich body not equal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 318-328: Run test: with step
```python
    def test_with_step(self) -> None:
        record: ComparisonNonTensorRecord = ComparisonNonTensorRecord(
            name="bias",
            baseline_value="True",
            target_value="True",
            baseline_type="bool",
            target_type="bool",
            values_equal=True,
            location=RecordLocation(step=5),
        )
        assert "(step=5)" in record._format_body()
```
**EN:** This test method exercises with step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 330-348: Run test: category
```python
    def test_category(self) -> None:
        passed: ComparisonNonTensorRecord = ComparisonNonTensorRecord(
            name="x",
            baseline_value="1",
            target_value="1",
            baseline_type="int",
            target_type="int",
            values_equal=True,
        )
        failed: ComparisonNonTensorRecord = ComparisonNonTensorRecord(
            name="x",
            baseline_value="1",
            target_value="2",
            baseline_type="int",
            target_type="int",
            values_equal=False,
        )
        assert passed.category == "passed"
        assert failed.category == "failed"
```
**EN:** This test method exercises category and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 category 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 356-356: Define class TestSummaryRecord
```python
class TestSummaryRecord:
```
**EN:** This declaration introduces the `TestSummaryRecord` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSummaryRecord` 测试类，并说明它通过继承承担的职责。

### Lines 357-366: Run test: format body
```python
    def test_format_body(self) -> None:
        record: SummaryRecord = SummaryRecord(
            total=10,
            passed=7,
            failed=2,
            skipped=1,
        )
        assert record._format_body() == (
            "Summary: 7 passed, 2 failed, 1 skipped (total 10)"
        )
```
**EN:** This test method exercises format body and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format body 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 368-386: Run test: format rich body
```python
    def test_format_rich_body(self) -> None:
        record: SummaryRecord = SummaryRecord(
            total=10,
            passed=7,
            failed=2,
            skipped=1,
        )
        body = record._format_rich_body()
        assert isinstance(body, Panel)

        rendered: str = _render_rich(body)
        assert rendered == (
            "╭────────────────────────────────────────────────────── SUMMARY "
            "───────────────────────────────────────────────────────╮\n"
            "│ 7 passed │ 2 failed │ 1 skipped │ 10 total"
            "                                                                           │\n"
            "╰──────────────────────────────────────────────────────────────────────"
            "────────────────────────────────────────────────╯"
        )
```
**EN:** This test method exercises format rich body and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format rich body 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 388-390: Run test: validation error
```python
    def test_validation_error(self) -> None:
        with pytest.raises(ValueError, match="total=5 !="):
            SummaryRecord(total=5, passed=1, failed=1, skipped=1)
```
**EN:** This test method exercises validation error and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 validation error 场景，并验证观测到的行为是否符合预期契约。

### Lines 398-398: Define class TestComparisonTensorRecordFormatBody
```python
class TestComparisonTensorRecordFormatBody:
```
**EN:** This declaration introduces the `TestComparisonTensorRecordFormatBody` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComparisonTensorRecordFormatBody` 测试类，并说明它通过继承承担的职责。

### Lines 399-426: Run test: basic
```python
    def test_basic(self) -> None:
        record: ComparisonTensorRecord = ComparisonTensorRecord(
            name="hidden",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
        )
        body: str = record._format_body()

        assert body == (
            "Raw [shape] [4, 8] vs [4, 8]\t[dtype] torch.float32 vs torch.float32\n"
            "After unify [shape] [4, 8] vs [4, 8]\t[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005"
        )
```
**EN:** This test method exercises basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 428-473: Run test: with replicated checks
```python
    def test_with_replicated_checks(self) -> None:
        from sglang.srt.debug_utils.comparator.output_types import ReplicatedCheckResult

        record: ComparisonTensorRecord = ComparisonTensorRecord(
            name="hidden",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
            replicated_checks=[
                ReplicatedCheckResult(
                    axis="tp",
                    group_index=0,
                    compared_index=1,
                    baseline_index=0,
                    passed=True,
                    atol=1e-3,
                    diff=_make_diff(
                        rel_diff=1e-6, max_abs_diff=1e-5, mean_abs_diff=1e-6
                    ),
                ),
            ],
        )
        body: str = record._format_body()

        assert body == (
            "Raw [shape] [4, 8] vs [4, 8]\t[dtype] torch.float32 vs torch.float32\n"
            "After unify [shape] [4, 8] vs [4, 8]\t[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005\n"
            "Replicated checks:\n"
            "  ✅ axis=tp group=0 idx=1 vs 0: "
            "rel_diff=1.000000e-06 max_abs_diff=1.000000e-05 mean_abs_diff=1.000000e-06"
        )
```
**EN:** This test method exercises with replicated checks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with replicated checks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 475-516: Run test: with aligner plan
```python
    def test_with_aligner_plan(self) -> None:
        plan: AlignerPlan = AlignerPlan(
            per_step_plans=Pair(x=[], y=[]),
        )
        traced: TracedAlignerPlan = TracedAlignerPlan(
            plan=plan,
            per_side=Pair(
                x=TracedSidePlan(step_plans=[]),
                y=TracedSidePlan(step_plans=[]),
            ),
        )
        record: ComparisonTensorRecord = ComparisonTensorRecord(
            name="hidden",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
            traced_plan=traced,
        )
        body: str = record._format_body()

        assert body == (
            "Raw [shape] [4, 8] vs [4, 8]\t[dtype] torch.float32 vs torch.float32\n"
            "After unify [shape] [4, 8] vs [4, 8]\t[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005\n"
            "Aligner Plan:\n"
            "  baseline: (no steps)\n"
            "  target: (no steps)"
        )
```
**EN:** This test method exercises with aligner plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with aligner plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 518-547: Run test: with step
```python
    def test_with_step(self) -> None:
        record: ComparisonTensorRecord = ComparisonTensorRecord(
            name="hidden",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
            location=RecordLocation(step=2),
        )
        body: str = record._format_body()

        assert body == (
            "[step=2] "
            "Raw [shape] [4, 8] vs [4, 8]\t[dtype] torch.float32 vs torch.float32\n"
            "After unify [shape] [4, 8] vs [4, 8]\t[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005"
        )
```
**EN:** This test method exercises with step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 555-579: Define helper: wrap plan
```python
def _wrap_plan(plan: AlignerPlan) -> TracedAlignerPlan:
    """Wrap an AlignerPlan into a TracedAlignerPlan with no snapshots."""
    baseline_traced_steps: list[TracedStepPlan] = [
        TracedStepPlan(
            step=sp.step,
            input_object_indices=sp.input_object_indices,
            sub_plans=[TracedSubPlan(plan=sub) for sub in sp.sub_plans],
        )
        for sp in plan.per_step_plans.x
    ]
    target_traced_steps: list[TracedStepPlan] = [
        TracedStepPlan(
            step=sp.step,
            input_object_indices=sp.input_object_indices,
            sub_plans=[TracedSubPlan(plan=sub) for sub in sp.sub_plans],
        )
        for sp in plan.per_step_plans.y
    ]
    return TracedAlignerPlan(
        plan=plan,
        per_side=Pair(
            x=TracedSidePlan(step_plans=baseline_traced_steps),
            y=TracedSidePlan(step_plans=target_traced_steps),
        ),
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 582-582: Define class TestFormatAlignerPlan
```python
class TestFormatAlignerPlan:
```
**EN:** This declaration introduces the `TestFormatAlignerPlan` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatAlignerPlan` 测试类，并说明它通过继承承担的职责。

### Lines 583-591: Run test: passthrough
```python
    def test_passthrough(self) -> None:
        plan: AlignerPlan = AlignerPlan(
            per_step_plans=Pair(x=[], y=[]),
        )
        result: str = _format_aligner_plan(_wrap_plan(plan))

        assert result == (
            "Aligner Plan:\n" "  baseline: (no steps)\n" "  target: (no steps)"
        )
```
**EN:** This test method exercises passthrough and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 passthrough 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 593-615: Run test: unsharder
```python
    def test_unsharder(self) -> None:
        unsharder: UnsharderPlan = UnsharderPlan(
            axis=ParallelAxis.TP,
            params=ConcatParams(dim_name="h"),
            groups=[[0, 1]],
        )
        plan: AlignerPlan = AlignerPlan(
            per_step_plans=Pair(
                x=[],
                y=[
                    AlignerPerStepPlan(
                        step=0, input_object_indices=[0, 1], sub_plans=[unsharder]
                    )
                ],
            ),
        )
        result: str = _format_aligner_plan(_wrap_plan(plan))

        assert result == (
            "Aligner Plan:\n"
            "  baseline: (no steps)\n"
            "  target: [step=0: unsharder(tp)]"
        )
```
**EN:** This test method exercises unsharder and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 unsharder 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 617-637: Run test: reorderer
```python
    def test_reorderer(self) -> None:
        reorderer: ReordererPlan = ReordererPlan(
            params=ZigzagToNaturalParams(dim_name="s", cp_size=2),
        )
        plan: AlignerPlan = AlignerPlan(
            per_step_plans=Pair(
                x=[],
                y=[
                    AlignerPerStepPlan(
                        step=0, input_object_indices=[0], sub_plans=[reorderer]
                    )
                ],
            ),
        )
        result: str = _format_aligner_plan(_wrap_plan(plan))

        assert result == (
            "Aligner Plan:\n"
            "  baseline: (no steps)\n"
            "  target: [step=0: reorderer(zigzag_to_natural)]"
        )
```
**EN:** This test method exercises reorderer and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reorderer 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 639-667: Run test: multi step
```python
    def test_multi_step(self) -> None:
        unsharder: UnsharderPlan = UnsharderPlan(
            axis=ParallelAxis.TP,
            params=ConcatParams(dim_name="h"),
            groups=[[0, 1]],
        )
        reorderer: ReordererPlan = ReordererPlan(
            params=ZigzagToNaturalParams(dim_name="s", cp_size=2),
        )
        plan: AlignerPlan = AlignerPlan(
            per_step_plans=Pair(
                x=[],
                y=[
                    AlignerPerStepPlan(
                        step=0, input_object_indices=[0, 1], sub_plans=[unsharder]
                    ),
                    AlignerPerStepPlan(
                        step=1, input_object_indices=[0], sub_plans=[reorderer]
                    ),
                ],
            ),
        )
        result: str = _format_aligner_plan(_wrap_plan(plan))

        assert result == (
            "Aligner Plan:\n"
            "  baseline: (no steps)\n"
            "  target: [step=0: unsharder(tp); step=1: reorderer(zigzag_to_natural)]"
        )
```
**EN:** This test method exercises multi step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 669-688: Run test: with token aligner
```python
    def test_with_token_aligner(self) -> None:
        ta_plan: TokenAlignerPlan = TokenAlignerPlan(
            locators=Pair(
                x=TokenLocator(steps=[0, 0, 0], token_index_in_step=[0, 1, 2]),
                y=TokenLocator(steps=[0, 0, 0], token_index_in_step=[0, 1, 2]),
            ),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
        )
        plan: AlignerPlan = AlignerPlan(
            per_step_plans=Pair(x=[], y=[]),
            token_aligner_plan=ta_plan,
        )
        result: str = _format_aligner_plan(_wrap_plan(plan))

        assert result == (
            "Aligner Plan:\n"
            "  baseline: (no steps)\n"
            "  target: (no steps)\n"
            "  token_aligner: 3 tokens aligned"
        )
```
**EN:** This test method exercises with token aligner and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with token aligner 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 690-705: Run test: with axis aligner
```python
    def test_with_axis_aligner(self) -> None:
        aa_plan: AxisAlignerPlan = AxisAlignerPlan(
            pattern=Pair(x="b s d -> s b d", y=None),
        )
        plan: AlignerPlan = AlignerPlan(
            per_step_plans=Pair(x=[], y=[]),
            axis_aligner_plan=aa_plan,
        )
        result: str = _format_aligner_plan(_wrap_plan(plan))

        assert result == (
            "Aligner Plan:\n"
            "  baseline: (no steps)\n"
            "  target: (no steps)\n"
            "  axis_aligner: x: b s d -> s b d"
        )
```
**EN:** This test method exercises with axis aligner and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with axis aligner 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 713-713: Define class TestOutputRecordLogAttachment
```python
class TestOutputRecordLogAttachment:
```
**EN:** This declaration introduces the `TestOutputRecordLogAttachment` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOutputRecordLogAttachment` 测试类，并说明它通过继承承担的职责。

### Lines 714-718: Run test: to text no logs
```python
    def test_to_text_no_logs(self) -> None:
        record: ConfigRecord = ConfigRecord(config={"a": 1})
        text: str = record.to_text()

        assert text == "Config: {'a': 1}"
```
**EN:** This test method exercises to text no logs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to text no logs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 720-727: Run test: to text errors only
```python
    def test_to_text_errors_only(self) -> None:
        record: ConfigRecord = ConfigRecord(
            config={"a": 1},
            errors=[ErrorLog(category="x", message="err1")],
        )
        text: str = record.to_text()

        assert text == "Config: {'a': 1}\n  ✗ err1"
```
**EN:** This test method exercises to text errors only and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to text errors only 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 729-736: Run test: to text infos only
```python
    def test_to_text_infos_only(self) -> None:
        record: ConfigRecord = ConfigRecord(
            config={"a": 1},
            infos=[InfoLog(category="x", message="note1")],
        )
        text: str = record.to_text()

        assert text == "Config: {'a': 1}\n  ℹ note1"
```
**EN:** This test method exercises to text infos only and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to text infos only 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 738-746: Run test: to text mixed
```python
    def test_to_text_mixed(self) -> None:
        record: ConfigRecord = ConfigRecord(
            config={"a": 1},
            errors=[ErrorLog(category="x", message="err1")],
            infos=[InfoLog(category="y", message="note1")],
        )
        text: str = record.to_text()

        assert text == "Config: {'a': 1}\n  ✗ err1\n  ℹ note1"
```
**EN:** This test method exercises to text mixed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to text mixed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 748-757: Run test: to rich string body
```python
    def test_to_rich_string_body(self) -> None:
        record: ComparisonSkipRecord = ComparisonSkipRecord(
            name="x",
            reason="r",
            errors=[ErrorLog(category="e", message="oops")],
        )
        body = record.to_rich()

        assert isinstance(body, str)
        assert body == "[dim]⊘ x ── skipped (r)[/]\n  [red]✗ oops[/]\n"
```
**EN:** This test method exercises to rich string body and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to rich string body 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 759-767: Run test: to rich group body
```python
    def test_to_rich_group_body(self) -> None:
        record: ConfigRecord = ConfigRecord(
            config={"a": 1},
            errors=[ErrorLog(category="e", message="oops")],
        )
        body = record.to_rich()

        # Panel body + log block → Group
        assert isinstance(body, Group)
```
**EN:** This test method exercises to rich group body and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 to rich group body 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 770-771: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.axis_aligner`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `io`, `pytest`, `registered.debug_utils.comparator.testing_helpers`, `rich.console`, `rich.panel`, `sys`
- Notable symbols / 关键符号: None / 无
