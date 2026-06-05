# test_types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/tensor_comparator/test_types.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on tensor comparator types in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 tensor comparator types 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Import dependencies
```python
import json
import sys

import pytest

from sglang.srt.debug_utils.comparator.output_types import (
    ComparisonSkipRecord,
    ComparisonTensorRecord,
    ConfigRecord,
    ErrorLog,
    InfoLog,
    LogRecord,
    ReplicatedCheckResult,
    SummaryRecord,
    parse_record_json,
)
from sglang.srt.debug_utils.comparator.tensor_comparator.types import (
    DiffInfo,
    TensorInfo,
    TensorStats,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 24-24: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 27-37: Define helper: make stats
```python
def _make_stats(**overrides) -> TensorStats:
    defaults: dict = dict(
        mean=0.5,
        abs_mean=1.2,
        std=1.0,
        min=-2.0,
        max=3.0,
        percentiles={1: -1.8, 5: -1.5, 50: 0.0, 95: 2.5, 99: 2.8},
    )
    defaults.update(overrides)
    return TensorStats(**defaults)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 40-52: Define helper: make diff
```python
def _make_diff(**overrides) -> DiffInfo:
    defaults = dict(
        rel_diff=1e-4,
        max_abs_diff=5e-4,
        mean_abs_diff=2e-4,
        max_diff_coord=[2, 3],
        baseline_at_max=1.0,
        target_at_max=1.0005,
        diff_threshold=1e-3,
        passed=True,
    )
    defaults.update(overrides)
    return DiffInfo(**defaults)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 55-62: Define helper: make tensor info
```python
def _make_tensor_info(**overrides) -> TensorInfo:
    defaults = dict(
        shape=[4, 8],
        dtype="torch.float32",
        stats=_make_stats(),
    )
    defaults.update(overrides)
    return TensorInfo(**defaults)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 65-65: Define class TestStrictBase
```python
class TestStrictBase:
```
**EN:** This declaration introduces the `TestStrictBase` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestStrictBase` 测试类，并说明它通过继承承担的职责。

### Lines 66-68: Run test: rejects extra fields
```python
    def test_rejects_extra_fields(self):
        with pytest.raises(Exception):
            TensorStats(mean=0.0, abs_mean=0.5, std=1.0, min=-1.0, max=1.0, bogus=42)
```
**EN:** This test method exercises rejects extra fields and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 rejects extra fields 场景，并验证观测到的行为是否符合预期契约。

### Lines 70-82: Run test: rejects extra fields on diff
```python
    def test_rejects_extra_fields_on_diff(self):
        with pytest.raises(Exception):
            DiffInfo(
                rel_diff=0.0,
                max_abs_diff=0.0,
                mean_abs_diff=0.0,
                max_diff_coord=[0],
                baseline_at_max=0.0,
                target_at_max=0.0,
                diff_threshold=1e-3,
                passed=True,
                extra_field=123,
            )
```
**EN:** This test method exercises rejects extra fields on diff and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 rejects extra fields on diff 场景，并验证观测到的行为是否符合预期契约。

### Lines 85-85: Define class TestRecordTypes
```python
class TestRecordTypes:
```
**EN:** This declaration introduces the `TestRecordTypes` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestRecordTypes` 测试类，并说明它通过继承承担的职责。

### Lines 86-99: Run test: comparison record inherits tensor fields
```python
    def test_comparison_record_inherits_tensor_fields(self):
        record = ComparisonTensorRecord(
            name="hidden_states",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
        )
        parsed = json.loads(record.model_dump_json())
        assert parsed["type"] == "comparison_tensor"
        assert parsed["name"] == "hidden_states"
        assert "baseline" in parsed
        assert "diff" in parsed
```
**EN:** This test method exercises comparison record inherits tensor fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 comparison record inherits tensor fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 101-127: Run test: discriminated union parsing
```python
    def test_discriminated_union_parsing(self):
        for record in [
            ConfigRecord(
                config={
                    "baseline_path": "/a",
                    "target_path": "/b",
                    "diff_threshold": 1e-3,
                    "start_step": 0,
                    "end_step": 100,
                }
            ),
            ComparisonSkipRecord(name="attn", reason="no_baseline"),
            ComparisonTensorRecord(
                name="mlp",
                baseline=_make_tensor_info(),
                target=_make_tensor_info(),
                unified_shape=[4, 8],
                shape_mismatch=False,
            ),
            SummaryRecord(total=10, passed=8, failed=1, skipped=1),
            LogRecord(
                errors=[ErrorLog(category="test", message="test warning")],
            ),
        ]:
            restored = parse_record_json(record.model_dump_json())
            assert type(restored) is type(record)
            assert restored == record
```
**EN:** This test method exercises discriminated union parsing and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 discriminated union parsing 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 130-147: Define helper: make replicated check
```python
def _make_replicated_check(**overrides) -> ReplicatedCheckResult:
    defaults: dict = dict(
        axis="tp",
        group_index=0,
        compared_index=1,
        baseline_index=0,
        passed=False,
        atol=1e-6,
        diff=_make_diff(
            rel_diff=0.1,
            max_abs_diff=0.1,
            mean_abs_diff=0.05,
            diff_threshold=1e-6,
            passed=False,
        ),
    )
    defaults.update(overrides)
    return ReplicatedCheckResult(**defaults)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 150-150: Define class TestWarnings
```python
class TestWarnings:
```
**EN:** This declaration introduces the `TestWarnings` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestWarnings` 测试类，并说明它通过继承承担的职责。

### Lines 151-162: Run test: comparison record failed when diff passed but errors
```python
    def test_comparison_record_failed_when_diff_passed_but_errors(self):
        """ComparisonTensorRecord with diff.passed=True but errors → category=='failed'."""
        record = ComparisonTensorRecord(
            name="hidden",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(passed=True),
            errors=[ErrorLog(category="test", message="some warning")],
        )
        assert record.category == "failed"
```
**EN:** This test method exercises comparison record failed when diff passed but errors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 comparison record failed when diff passed but errors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 164-171: Run test: skip record failed when errors
```python
    def test_skip_record_failed_when_errors(self):
        """ComparisonSkipRecord with errors → category=='failed' instead of 'skipped'."""
        record = ComparisonSkipRecord(
            name="x",
            reason="no_baseline",
            errors=[ErrorLog(category="test", message="some warning")],
        )
        assert record.category == "failed"
```
**EN:** This test method exercises skip record failed when errors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 skip record failed when errors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 173-184: Run test: replicated checks all passed
```python
    def test_replicated_checks_all_passed(self):
        """ComparisonTensorRecord with all replicated_checks passed → category=='passed'."""
        record = ComparisonTensorRecord(
            name="hidden",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(passed=True),
            replicated_checks=[_make_replicated_check(passed=True)],
        )
        assert record.category == "passed"
```
**EN:** This test method exercises replicated checks all passed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replicated checks all passed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 186-197: Run test: replicated checks failed means record failed
```python
    def test_replicated_checks_failed_means_record_failed(self):
        """ComparisonTensorRecord with any replicated_check.passed=False → category=='failed'."""
        record = ComparisonTensorRecord(
            name="hidden",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(passed=True),
            replicated_checks=[_make_replicated_check(passed=False)],
        )
        assert record.category == "failed"
```
**EN:** This test method exercises replicated checks failed means record failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replicated checks failed means record failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 199-227: Run test: replicated check json round trip
```python
    def test_replicated_check_json_round_trip(self):
        """ReplicatedCheckResult survives JSON round-trip via ComparisonTensorRecord."""
        check = _make_replicated_check(
            axis="cp",
            group_index=2,
            compared_index=3,
            baseline_index=0,
            passed=False,
        )
        record = ComparisonTensorRecord(
            name="mlp",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
            replicated_checks=[check],
        )

        restored = parse_record_json(record.model_dump_json())
        assert isinstance(restored, ComparisonTensorRecord)
        assert len(restored.replicated_checks) == 1

        restored_check: ReplicatedCheckResult = restored.replicated_checks[0]
        assert restored_check.axis == "cp"
        assert restored_check.group_index == 2
        assert restored_check.compared_index == 3
        assert restored_check.baseline_index == 0
        assert not restored_check.passed
```
**EN:** This test method exercises replicated check json round trip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replicated check json round trip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 229-257: Run test: any log discriminated union round trip
```python
    def test_any_log_discriminated_union_round_trip(self):
        """ErrorLog and InfoLog survive JSON round-trip via a LogRecord."""
        all_errors = [
            ErrorLog(
                category="rids_mismatch",
                message="rids mismatch across ranks: rank 0 has [1,2,3], "
                "rank 1 has [4,5,6]",
            ),
        ]
        all_infos = [
            InfoLog(
                category="aux_tensors_missing",
                message="Aux tensors missing, skipping token alignment",
            ),
        ]

        record = LogRecord(errors=all_errors, infos=all_infos)
        restored = parse_record_json(record.model_dump_json())
        assert isinstance(restored, LogRecord)
        assert len(restored.errors) == len(all_errors)
        assert len(restored.infos) == len(all_infos)

        for original, parsed in zip(all_errors, restored.errors):
            assert type(parsed) is type(original)
            assert parsed == original

        for original, parsed in zip(all_infos, restored.infos):
            assert type(parsed) is type(original)
            assert parsed == original
```
**EN:** This test method exercises any log discriminated union round trip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 any log discriminated union round trip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 260-261: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.tensor_comparator.types`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `json`, `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
