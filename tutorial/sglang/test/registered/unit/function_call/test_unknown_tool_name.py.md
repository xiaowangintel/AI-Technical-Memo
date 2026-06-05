# test_unknown_tool_name.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_unknown_tool_name.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates unknown tool name behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 unknown tool name 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import json
import logging

import pytest

from sglang.srt.entrypoints.openai.protocol import Function, Tool
from sglang.srt.environ import envs
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import StreamingParseResult
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `logging`, `pytest`, `sglang.srt.entrypoints.openai.protocol`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `logging`, `pytest`, `sglang.srt.entrypoints.openai.protocol`。

### Lines 12-12: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(5, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 15-15: class DummyDetector declaration / 类 DummyDetector 声明
```python
class DummyDetector(BaseFormatDetector):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `BaseFormatDetector`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `BaseFormatDetector`。

### Lines 16-17: method has tool call / 方法 has tool call
```python
    def has_tool_call(self, text: str) -> bool:
        return True
```
**EN:** This block implements `has_tool_call` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `has_tool_call`，承担模块行为中的一个聚焦逻辑片段。

### Lines 19-23: method detect and parse / 方法 detect and parse
```python
    def detect_and_parse(self, text: str, tools):
        action = json.loads(text)
        return StreamingParseResult(
            normal_text="", calls=self.parse_base_json(action, tools)
        )
```
**EN:** This block implements `detect_and_parse` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `detect_and_parse`，承担模块行为中的一个聚焦逻辑片段。

### Lines 25-26: method structure info / 方法 structure info
```python
    def structure_info(self):
        pass
```
**EN:** This block implements `structure_info` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `structure_info`，承担模块行为中的一个聚焦逻辑片段。

### Lines 29-50: test case unknown tool name dropped default / 测试用例 unknown tool name dropped default
```python
def test_unknown_tool_name_dropped_default(caplog):
    """Test that unknown tools are dropped by default (legacy behavior)."""
    with envs.SGLANG_FORWARD_UNKNOWN_TOOLS.override(False):
        tools = [
            Tool(
                function=Function(
                    name="get_weather", parameters={"type": "object", "properties": {}}
                )
            )
        ]
        detector = DummyDetector()
        with caplog.at_level(
            logging.WARNING, logger="sglang.srt.function_call.base_format_detector"
        ):
            result = detector.detect_and_parse(
                '{"name":"unknown_tool","parameters":{"city":"Paris"}}', tools
            )
        assert any(
            "Model attempted to call undefined function: unknown_tool" in m
            for m in caplog.messages
        )
        assert len(result.calls) == 0  # dropped in default mode
```
**EN:** Test that unknown tools are dropped by default (legacy behavior). This test exercises `test_unknown_tool_name_dropped_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that unknown tools are dropped by default (legacy behavior). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_tool_name_dropped_default`。

### Lines 53-77: test case unknown tool name forwarded / 测试用例 unknown tool name forwarded
```python
def test_unknown_tool_name_forwarded(caplog):
    """Test that unknown tools are forwarded when env var is True."""
    with envs.SGLANG_FORWARD_UNKNOWN_TOOLS.override(True):
        tools = [
            Tool(
                function=Function(
                    name="get_weather", parameters={"type": "object", "properties": {}}
                )
            )
        ]
        detector = DummyDetector()
        with caplog.at_level(
            logging.WARNING, logger="sglang.srt.function_call.base_format_detector"
        ):
            result = detector.detect_and_parse(
                '{"name":"unknown_tool","parameters":{"city":"Paris"}}', tools
            )
        assert any(
            "Model attempted to call undefined function: unknown_tool" in m
            for m in caplog.messages
        )
        assert len(result.calls) == 1
        assert result.calls[0].name == "unknown_tool"
        assert result.calls[0].tool_index == -1
        assert json.loads(result.calls[0].parameters)["city"] == "Paris"
```
**EN:** Test that unknown tools are forwarded when env var is True. This test exercises `test_unknown_tool_name_forwarded` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that unknown tools are forwarded when env var is True. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_tool_name_forwarded`。

### Lines 80-83: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import sys

    sys.exit(pytest.main([__file__]))
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `DummyDetector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `test_unknown_tool_name_dropped_default`: Test that unknown tools are dropped by default (legacy behavior). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_tool_name_dropped_default`。
- `test_unknown_tool_name_forwarded`: Test that unknown tools are forwarded when env var is True. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_tool_name_forwarded`。
- `DummyDetector.has_tool_call`: This block implements `has_tool_call` and captures one focused piece of the module's behavior. / 该代码块实现 `has_tool_call`，承担模块行为中的一个聚焦逻辑片段。
- `DummyDetector.detect_and_parse`: This block implements `detect_and_parse` and captures one focused piece of the module's behavior. / 该代码块实现 `detect_and_parse`，承担模块行为中的一个聚焦逻辑片段。
- `DummyDetector.structure_info`: This block implements `structure_info` and captures one focused piece of the module's behavior. / 该代码块实现 `structure_info`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`
- **Third-party modules / 第三方模块**: `pytest`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 83
