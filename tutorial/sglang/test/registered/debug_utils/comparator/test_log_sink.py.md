# test_log_sink.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_log_sink.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator log sink in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator log sink 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Import dependencies
```python
import json
import sys

import pytest

from sglang.srt.debug_utils.comparator.log_sink import LogSink
from sglang.srt.debug_utils.comparator.output_types import (
    ErrorLog,
    InfoLog,
)
from sglang.srt.debug_utils.comparator.report_sink import report_sink
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 14-14: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 17-23: Define helper: make error log
```python
def _make_error_log(**overrides) -> ErrorLog:
    defaults: dict = dict(
        category="test",
        message="test warning",
    )
    defaults.update(overrides)
    return ErrorLog(**defaults)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 26-26: Define class TestLogSink
```python
class TestLogSink:
```
**EN:** This declaration introduces the `TestLogSink` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLogSink` 测试类，并说明它通过继承承担的职责。

### Lines 27-35: Run test: basic collection
```python
    def test_basic_collection(self) -> None:
        sink = LogSink()
        log = _make_error_log()

        with sink.context() as collected:
            sink.add(log)

        assert len(collected) == 1
        assert collected[0] is log
```
**EN:** This test method exercises basic collection and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic collection 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 37-50: Run test: nested contexts
```python
    def test_nested_contexts(self) -> None:
        sink = LogSink()
        outer_log = _make_error_log(message="outer")
        inner_log = _make_error_log(message="inner")

        with sink.context() as outer:
            sink.add(outer_log)
            with sink.context() as inner:
                sink.add(inner_log)
            assert len(inner) == 1
            assert inner[0] is inner_log

        assert len(outer) == 1
        assert outer[0] is outer_log
```
**EN:** This test method exercises nested contexts and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 nested contexts 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 52-56: Run test: empty context
```python
    def test_empty_context(self) -> None:
        sink = LogSink()
        with sink.context() as collected:
            pass
        assert collected == []
```
**EN:** This test method exercises empty context and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty context 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 58-65: Run test: add outside context prints
```python
    def test_add_outside_context_prints(self, capsys) -> None:
        sink = LogSink()
        report_sink.configure(output_format="text")

        sink.add(_make_error_log())

        captured = capsys.readouterr()
        assert "test warning" in captured.out
```
**EN:** This test method exercises add outside context prints and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 add outside context prints 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 67-76: Run test: context captures instead of printing
```python
    def test_context_captures_instead_of_printing(self, capsys) -> None:
        sink = LogSink()
        report_sink.configure(output_format="text")

        with sink.context() as collected:
            sink.add(_make_error_log())

        assert len(collected) == 1
        captured = capsys.readouterr()
        assert captured.out == ""
```
**EN:** This test method exercises context captures instead of printing and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 context captures instead of printing 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 78-87: Run test: json output outside context
```python
    def test_json_output_outside_context(self, capsys) -> None:
        sink = LogSink()
        report_sink.configure(output_format="json")

        sink.add(_make_error_log())

        captured = capsys.readouterr()
        parsed: dict = json.loads(captured.out.strip())
        assert "errors" in parsed
        assert len(parsed["errors"]) == 1
```
**EN:** This test method exercises json output outside context and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 json output outside context 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 89-98: Run test: info log outside context routes to infos
```python
    def test_info_log_outside_context_routes_to_infos(self, capsys) -> None:
        """InfoLog added outside context populates LogRecord.infos, not errors."""
        sink = LogSink()
        report_sink.configure(output_format="json")

        sink.add(InfoLog(category="test", message="info msg"))

        parsed: dict = json.loads(capsys.readouterr().out.strip())
        assert len(parsed["infos"]) == 1
        assert len(parsed["errors"]) == 0
```
**EN:** This test method exercises info log outside context routes to infos and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 info log outside context routes to infos 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 100-113: Run test: exception in context cleans stack
```python
    def test_exception_in_context_cleans_stack(self, capsys) -> None:
        sink = LogSink()
        report_sink.configure(output_format="text")

        with pytest.raises(RuntimeError):
            with sink.context() as collected:
                sink.add(_make_error_log())
                raise RuntimeError("boom")

        assert len(collected) == 1

        sink.add(_make_error_log(message="after exception"))
        captured = capsys.readouterr()
        assert "after exception" in captured.out
```
**EN:** This test method exercises exception in context cleans stack and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 exception in context cleans stack 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 116-117: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.report_sink`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `json`, `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
