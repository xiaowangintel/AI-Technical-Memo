# test_poolside_v1_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_poolside_v1_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates poolside v1 detector behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 poolside v1 detector 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for PoolsideV1Detector — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-9: module imports and dependencies / 模块导入与依赖
```python
import json

from sglang.srt.entrypoints.openai.protocol import Function, Tool
from sglang.srt.function_call.function_call_parser import FunctionCallParser
from sglang.srt.function_call.poolside_v1_detector import PoolsideV1Detector
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.function_call_parser`, `sglang.srt.function_call.poolside_v1_detector`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.function_call_parser`, `sglang.srt.function_call.poolside_v1_detector`。

### Lines 11-11: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(1.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 14-14: class TestPoolsideV1Detector declaration / 类 TestPoolsideV1Detector 声明
```python
class TestPoolsideV1Detector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 15-54: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather",
                    parameters={
                        "type": "object",
                        "properties": {
                            "location": {"type": "string"},
                            "count": {"type": "integer"},
                            "options": {"type": "object"},
                        },
                        "required": ["location"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="search",
                    description="Search",
                    parameters={
                        "type": "object",
                        "properties": {"query": {"type": "string"}},
                        "required": ["query"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="now",
                    description="Current time",
                    parameters={"type": "object", "properties": {}},
                ),
            ),
        ]
        self.detector = PoolsideV1Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 55-57: supporting source context / 辅助源码上下文
```python

    # ==================== has_tool_call ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 58-63: test case has tool call true / 测试用例 has tool call true
```python
    def test_has_tool_call_true(self):
        text = (
            "<tool_call>get_weather\n<arg_key>location</arg_key>\n"
            "<arg_value>SF</arg_value>\n</tool_call>"
        )
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_true` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_true`。

### Lines 65-66: test case has tool call false / 测试用例 has tool call false
```python
    def test_has_tool_call_false(self):
        self.assertFalse(self.detector.has_tool_call("just a sentence."))
```
**EN:** This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。

### Lines 67-69: supporting source context / 辅助源码上下文
```python

    # ==================== detect_and_parse ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 70-79: test case single tool call string arg / 测试用例 single tool call string arg
```python
    def test_single_tool_call_string_arg(self):
        text = (
            "<tool_call>get_weather\n<arg_key>location</arg_key>\n"
            "<arg_value>San Francisco</arg_value>\n</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args, {"location": "San Francisco"})
```
**EN:** This test exercises `test_single_tool_call_string_arg` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_string_arg`。

### Lines 81-94: test case single tool call mixed types / 测试用例 single tool call mixed types
```python
    def test_single_tool_call_mixed_types(self):
        text = (
            "<tool_call>get_weather\n"
            "<arg_key>location</arg_key>\n<arg_value>London</arg_value>\n"
            "<arg_key>count</arg_key>\n<arg_value>3</arg_value>\n"
            '<arg_key>options</arg_key>\n<arg_value>{"verbose": true}</arg_value>\n'
            "</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["location"], "London")
        self.assertEqual(args["count"], 3)
        self.assertEqual(args["options"], {"verbose": True})
```
**EN:** This test exercises `test_single_tool_call_mixed_types` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_mixed_types`。

### Lines 96-107: test case multiple tool calls / 测试用例 multiple tool calls
```python
    def test_multiple_tool_calls(self):
        text = (
            "<tool_call>get_weather\n<arg_key>location</arg_key>\n"
            "<arg_value>NYC</arg_value>\n</tool_call>\n"
            "<tool_call>search\n<arg_key>query</arg_key>\n"
            "<arg_value>pizza</arg_value>\n</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "search")
        self.assertEqual(json.loads(result.calls[1].parameters), {"query": "pizza"})
```
**EN:** This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。

### Lines 109-117: test case leading text extracted as normal / 测试用例 leading text extracted as normal
```python
    def test_leading_text_extracted_as_normal(self):
        text = (
            "Sure, checking now. "
            "<tool_call>search\n<arg_key>query</arg_key>\n"
            "<arg_value>tacos</arg_value>\n</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(result.normal_text, "Sure, checking now. ")
        self.assertEqual(len(result.calls), 1)
```
**EN:** This test exercises `test_leading_text_extracted_as_normal` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_leading_text_extracted_as_normal`。

### Lines 119-125: test case unknown tool dropped / 测试用例 unknown tool dropped
```python
    def test_unknown_tool_dropped(self):
        text = (
            "<tool_call>nonexistent_fn\n<arg_key>x</arg_key>\n"
            "<arg_value>1</arg_value>\n</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
```
**EN:** This test exercises `test_unknown_tool_dropped` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_tool_dropped`。

### Lines 127-135: test case malformed value falls back to string / 测试用例 malformed value falls back to string
```python
    def test_malformed_value_falls_back_to_string(self):
        text = (
            "<tool_call>get_weather\n<arg_key>options</arg_key>\n"
            "<arg_value>not_json</arg_value>\n</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["options"], "not_json")
```
**EN:** This test exercises `test_malformed_value_falls_back_to_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_value_falls_back_to_string`。

### Lines 137-142: test case zero arg call / 测试用例 zero arg call
```python
    def test_zero_arg_call(self):
        text = "<tool_call>now\n</tool_call>"
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "now")
        self.assertEqual(json.loads(result.calls[0].parameters), {})
```
**EN:** This test exercises `test_zero_arg_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_arg_call`。

### Lines 144-150: test case zero arg call no newline / 测试用例 zero arg call no newline
```python
    def test_zero_arg_call_no_newline(self):
        """`<tool_call>now</tool_call>` (no `\\n` between name and close tag)."""
        text = "<tool_call>now</tool_call>"
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "now")
        self.assertEqual(json.loads(result.calls[0].parameters), {})
```
**EN:** `<tool_call>now</tool_call>` (no `\n` between name and close tag). This test exercises `test_zero_arg_call_no_newline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `<tool_call>now</tool_call>` (no `\n` between name and close tag). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_arg_call_no_newline`。

### Lines 152-163: test case truncated pre value emits no calls / 测试用例 truncated pre value emits no calls
```python
    def test_truncated_pre_value_emits_no_calls(self):
        """Regression: max-tokens cutoff mid-`<arg_value>` must drop the
        in-flight call, matching the old closing-tag-anchored regex behavior.
        Without the truncated-call filter in detect_and_parse, streaming-as-
        primitive surfaced a tool call with parameters="{}" on this input."""
        text = (
            "<tool_call>get_weather\n<arg_key>location</arg_key>\n" "<arg_value>San Fr"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(
            len(result.calls), 0, "truncated mid-arg_value must yield 0 calls"
        )
```
**EN:** Regression: max-tokens cutoff mid-`<arg_value>` must drop the in-flight call, matching the old closing-tag-anchored regex behavior. This test exercises `test_truncated_pre_value_emits_no_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression: max-tokens cutoff mid-`<arg_value>` must drop the in-flight call, matching the old closing-tag-anchored regex behavior. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_truncated_pre_value_emits_no_calls`。

### Lines 165-178: test case truncated post value emits no calls / 测试用例 truncated post value emits no calls
```python
    def test_truncated_post_value_emits_no_calls(self):
        """Regression: cutoff after `</arg_value>` but before `</tool_call>`
        used to surface a tool call with non-JSON parameters
        ('{"location": "SF"' with no closing brace). The filter must drop it."""
        text = (
            "<tool_call>get_weather\n<arg_key>location</arg_key>\n"
            "<arg_value>SF</arg_value>\n"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(
            len(result.calls),
            0,
            "truncated after arg_value but before </tool_call> must yield 0 calls",
        )
```
**EN:** Regression: cutoff after `</arg_value>` but before `</tool_call>` used to surface a tool call with non-JSON parameters ('{"location": "SF"' with no closing brace). This test exercises `test_truncated_post_value_emits_no_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression: cutoff after `</arg_value>` but before `</tool_call>` used to surface a tool call with non-JSON parameters ('{"location": "SF"' with no closing brace). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_truncated_post_value_emits_no_calls`。

### Lines 180-210: test case set literal falls back to raw string / 测试用例 set literal falls back to raw string
```python
    def test_set_literal_falls_back_to_raw_string(self):
        """Regression: ast.literal_eval('{1,2,3}') returns a set, which
        json.dumps cannot serialize. Without the round-trip guard in
        _convert_param_value, the parse_streaming_increment loop would
        TypeError downstream. The guard rejects sets and falls back to the
        raw string (which then matches the underlying schema-string-typed
        treatment)."""
        tools_with_obj = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather",
                    parameters={
                        "type": "object",
                        "properties": {"options": {"type": "object"}},
                    },
                ),
            )
        ]
        detector = PoolsideV1Detector()
        text = (
            "<tool_call>get_weather\n<arg_key>options</arg_key>\n"
            "<arg_value>{1, 2, 3}</arg_value>\n</tool_call>"
        )
        result = detector.detect_and_parse(text, tools_with_obj)
        self.assertEqual(len(result.calls), 1)
        args = json.loads(result.calls[0].parameters)
        # set literal couldn't round-trip, so it's preserved as the raw
        # string (the only sane fallback).
        self.assertEqual(args["options"], "{1, 2, 3}")
```
**EN:** Regression: ast.literal_eval('{1,2,3}') returns a set, which json.dumps cannot serialize. This test exercises `test_set_literal_falls_back_to_raw_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression: ast.literal_eval('{1,2,3}') returns a set, which json.dumps cannot serialize. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_literal_falls_back_to_raw_string`。

### Lines 212-224: test case truncated after complete call keeps complete / 测试用例 truncated after complete call keeps complete
```python
    def test_truncated_after_complete_call_keeps_complete(self):
        """A complete tool_call followed by a truncated second one must keep
        the complete one and drop only the truncated tail — matching the old
        regex behavior on the same input."""
        text = (
            "<tool_call>get_weather\n<arg_key>location</arg_key>\n"
            "<arg_value>NYC</arg_value>\n</tool_call>\n"
            "<tool_call>search\n<arg_key>q"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(json.loads(result.calls[0].parameters), {"location": "NYC"})
```
**EN:** A complete tool_call followed by a truncated second one must keep the complete one and drop only the truncated tail — matching the old regex behavior on the same input. This test exercises `test_truncated_after_complete_call_keeps_complete` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A complete tool_call followed by a truncated second one must keep the complete one and drop only the truncated tail — matching the old regex behavior on the same input. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_truncated_after_complete_call_keeps_complete`。

### Lines 226-235: test case arg key without value emits empty call / 测试用例 arg key without value emits empty call
```python
    def test_arg_key_without_value_emits_empty_call(self):
        """Non-streaming: malformed `<arg_key>K</arg_key></tool_call>` (no
        `<arg_value>`) yields a tool call with empty params — the orphan
        `<arg_key>` is dropped because the regex looks for key/value pairs.
        Locks in the contract the streaming FSM must match."""
        text = "<tool_call>get_weather\n<arg_key>location</arg_key></tool_call>"
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(json.loads(result.calls[0].parameters), {})
```
**EN:** Non-streaming: malformed `<arg_key>K</arg_key></tool_call>` (no `<arg_value>`) yields a tool call with empty params — the orphan `<arg_key>` is dropped because the regex looks for key/value pairs. This test exercises `test_arg_key_without_value_emits_empty_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-streaming: malformed `<arg_key>K</arg_key></tool_call>` (no `<arg_value>`) yields a tool call with empty params — the orphan `<arg_key>` is dropped because the regex looks for key/value pairs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_arg_key_without_value_emits_empty_call`。

### Lines 237-269: test case streaming arg key without value closes call / 测试用例 streaming arg key without value closes call
```python
    def test_streaming_arg_key_without_value_closes_call(self):
        """Regression: malformed `<arg_key>K</arg_key></tool_call>` (no
        `<arg_value>`) used to leave the streaming FSM stuck in READING_VALUE
        — the bare-`<` discard ate `</tool_call>` byte-by-byte instead of
        recognizing it as a close. Worse: a *subsequent* tool call's
        `<arg_value>` would mis-attribute its content to the orphan
        `current_pending_key`, silently swallowing the second call's name.
        Both calls must be emitted with the orphan key dropped."""
        detector = PoolsideV1Detector()
        wire = (
            "<tool_call>get_weather\n<arg_key>location</arg_key></tool_call>"
            "<tool_call>search\n<arg_key>query</arg_key>\n"
            "<arg_value>tacos</arg_value>\n</tool_call>"
        )
        all_calls = []
        for chunk in [wire[i : i + 8] for i in range(0, len(wire), 8)]:
            r = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(r.calls)
        names = [c.name for c in all_calls if c.name]
        self.assertEqual(
            names,
            ["get_weather", "search"],
            "second call must not be swallowed when first is malformed",
        )
        per_tool: dict = {}
        for c in all_calls:
            if c.parameters:
                per_tool.setdefault(c.tool_index, "")
                per_tool[c.tool_index] += c.parameters
        # Orphan `location` key dropped — first call has empty params.
        self.assertEqual(json.loads(per_tool[0]), {})
        # Second call's value must NOT leak into first call's stale key.
        self.assertEqual(json.loads(per_tool[1]), {"query": "tacos"})
```
**EN:** Regression: malformed `<arg_key>K</arg_key></tool_call>` (no `<arg_value>`) used to leave the streaming FSM stuck in READING_VALUE — the bare-`<` discard ate `</tool_call>` byte-by-byte instead of recognizing it as a close. This test exercises `test_streaming_arg_key_without_value_closes_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression: malformed `<arg_key>K</arg_key></tool_call>` (no `<arg_value>`) used to leave the streaming FSM stuck in READING_VALUE — the bare-`<` discard ate `</tool_call>` byte-by-byte instead of recognizing it as a close. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_arg_key_without_value_closes_call`。

### Lines 271-292: test case orphan key followed by new key uses new key / 测试用例 orphan key followed by new key uses new key
```python
    def test_orphan_key_followed_by_new_key_uses_new_key(self):
        """Non-streaming: malformed `<arg_key>K1</arg_key><arg_key>K2</arg_key>
        <arg_value>V</arg_value>` (model emitted a key, then re-emitted a new
        key without a value for the first) yields `{K2: V}` — the orphan K1
        is dropped. Without the `[^<]` constraint in arg_pair_regex, the
        non-greedy `.*?` backtracks across the `</arg_key>` boundary and
        produces a junk key spanning both <arg_key> tags."""
        text = (
            "<tool_call>get_weather\n"
            "<arg_key>location</arg_key>"
            "<arg_key>count</arg_key><arg_value>3</arg_value>"
            "\n</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(
            args,
            {"count": 3},
            f"orphan key 'location' should be dropped, count=3 should win, got {args}",
        )
```
**EN:** Non-streaming: malformed `<arg_key>K1</arg_key><arg_key>K2</arg_key> <arg_value>V</arg_value>` (model emitted a key, then re-emitted a new key without a value for the first) yields `{K2: V}` — the orphan K1 is dropped. This test exercises `test_orphan_key_followed_by_new_key_uses_new_key` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-streaming: malformed `<arg_key>K1</arg_key><arg_key>K2</arg_key> <arg_value>V</arg_value>` (model emitted a key, then re-emitted a new key without a value for the first) yields `{K2: V}` — the orphan K1 is dropped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_orphan_key_followed_by_new_key_uses_new_key`。

### Lines 294-319: test case streaming orphan key followed by new key uses new key / 测试用例 streaming orphan key followed by new key uses new key
```python
    def test_streaming_orphan_key_followed_by_new_key_uses_new_key(self):
        """Regression: streaming on `<arg_key>K1</arg_key><arg_key>K2</arg_key>
        <arg_value>V</arg_value>` used to mis-attribute V to K1 — the bare-`<`
        discard ate the second `<arg_key>` as garbage and the value bound to
        the stale `current_pending_key`. With the orphan-key-replace branch
        in READING_VALUE, the new key wins and streaming matches the
        non-streaming regex path."""
        detector = PoolsideV1Detector()
        wire = (
            "<tool_call>get_weather\n"
            "<arg_key>location</arg_key>"
            "<arg_key>count</arg_key><arg_value>3</arg_value>"
            "\n</tool_call>"
        )
        all_calls = []
        for chunk in [wire[i : i + 8] for i in range(0, len(wire), 8)]:
            r = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(r.calls)
        names = [c.name for c in all_calls if c.name]
        self.assertEqual(names, ["get_weather"])
        params = "".join(c.parameters for c in all_calls if c.parameters)
        self.assertEqual(
            json.loads(params),
            {"count": 3},
            "orphan key 'location' should be dropped; count=3 must win",
        )
```
**EN:** Regression: streaming on `<arg_key>K1</arg_key><arg_key>K2</arg_key> <arg_value>V</arg_value>` used to mis-attribute V to K1 — the bare-`<` discard ate the second `<arg_key>` as garbage and the value bound to the stale `current_pending_key`. This test exercises `test_streaming_orphan_key_followed_by_new_key_uses_new_key` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression: streaming on `<arg_key>K1</arg_key><arg_key>K2</arg_key> <arg_value>V</arg_value>` used to mis-attribute V to K1 — the bare-`<` discard ate the second `<arg_key>` as garbage and the value bound to the stale `current_pending_key`. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_orphan_key_followed_by_new_key_uses_new_key`。

### Lines 321-327: test case streaming malformed no name does not hang / 测试用例 streaming malformed no name does not hang
```python
    def test_streaming_malformed_no_name_does_not_hang(self):
        """Regression: malformed `<tool_call><arg_key>...` (no name, no \\n)
        used to spin in branch 2 with consume=0. Must drain to </tool_call>."""
        detector = PoolsideV1Detector()
        wire = "<tool_call><arg_key>k</arg_key><arg_value>v</arg_value></tool_call>"
        result = detector.parse_streaming_increment(wire, self.tools)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Regression: malformed `<tool_call><arg_key>...` (no name, no \n) used to spin in branch 2 with consume=0. This test exercises `test_streaming_malformed_no_name_does_not_hang` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression: malformed `<tool_call><arg_key>...` (no name, no \n) used to spin in branch 2 with consume=0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_malformed_no_name_does_not_hang`。

### Lines 329-339: test case streaming arg tags without tool call wrapper / 测试用例 streaming arg tags without tool call wrapper
```python
    def test_streaming_arg_tags_without_tool_call_wrapper(self):
        """Regression: stray `<arg_key>...</arg_key><arg_value>...</arg_value>`
        with no preceding `<tool_call>` used to crash with IndexError on
        `streamed_args_for_tool[-1]` (masked by the old broad except). The
        FSM's READING_VALUE state is unreachable from OUTSIDE, so this returns
        0 calls without raising — and now that the broad except is gone, any
        regression here would propagate as a real test failure."""
        detector = PoolsideV1Detector()
        wire = "<arg_key>k</arg_key><arg_value>v</arg_value>"
        result = detector.parse_streaming_increment(wire, self.tools)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Regression: stray `<arg_key>...</arg_key><arg_value>...</arg_value>` with no preceding `<tool_call>` used to crash with IndexError on `streamed_args_for_tool[-1]` (masked by the old broad except). This test exercises `test_streaming_arg_tags_without_tool_call_wrapper` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression: stray `<arg_key>...</arg_key><arg_value>...</arg_value>` with no preceding `<tool_call>` used to crash with IndexError on `streamed_args_for_tool[-1]` (masked by the old broad except). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_arg_tags_without_tool_call_wrapper`。

### Lines 340-342: supporting source context / 辅助源码上下文
```python

    # ==================== structure_info ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 343-348: test case structure info / 测试用例 structure info
```python
    def test_structure_info(self):
        info_func = self.detector.structure_info()
        info = info_func("get_weather")
        self.assertEqual(info.trigger, "<tool_call>")
        self.assertIn("get_weather", info.begin)
        self.assertIn("</tool_call>", info.end)
```
**EN:** This test exercises `test_structure_info` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structure_info`。

### Lines 349-351: supporting source context / 辅助源码上下文
```python

    # ==================== Streaming ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 352-362: test case streaming single call chunked / 测试用例 streaming single call chunked
```python
    def test_streaming_single_call_chunked(self):
        detector = PoolsideV1Detector()
        chunks = [
            "<tool_",
            "call>get_weather\n<arg_key>",
            "location</arg_key>\n<arg_value>San Fr",
            "ancisco</arg_value>\n</tool_call>",
        ]
        names, params = self._collect(detector, chunks)
        self.assertEqual(names, ["get_weather"])
        self.assertEqual(json.loads(params), {"location": "San Francisco"})
```
**EN:** This test exercises `test_streaming_single_call_chunked` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_single_call_chunked`。

### Lines 364-382: test case streaming char by char robustness / 测试用例 streaming char by char robustness
```python
    def test_streaming_char_by_char_robustness(self):
        """Per-arg streaming under one-byte chunks. Values are emitted as a
        single `"key": value` fragment when `</arg_value>` arrives; this test
        proves the FSM doesn't leak trailing `<` / `</arg_v...` into the
        parameter delta as bytes arrive (partial-tag holdback in branch 6),
        and that the final reconstruction matches."""
        detector = PoolsideV1Detector()
        wire = (
            "<tool_call>get_weather\n<arg_key>location</arg_key>\n"
            "<arg_value>hello world</arg_value>\n</tool_call>"
        )
        chunks = list(wire)
        names, params = self._collect(detector, chunks)
        self.assertEqual(names, ["get_weather"])
        decoded = json.loads(params)
        self.assertEqual(decoded, {"location": "hello world"})
        # And the emitted parameter delta itself contains no stray tag bytes.
        self.assertNotIn("<", params)
        self.assertNotIn(">", params)
```
**EN:** Per-arg streaming under one-byte chunks. This test exercises `test_streaming_char_by_char_robustness` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Per-arg streaming under one-byte chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_char_by_char_robustness`。

### Lines 384-410: test case streaming index is sequential not tools slot / 测试用例 streaming index is sequential not tools slot
```python
    def test_streaming_index_is_sequential_not_tools_slot(self):
        """Regression: streaming emissions must use a per-response sequential
        index. If we emit the name with `tools_indices[name]` and the params
        with `current_tool_id`, OpenAI clients group chunks by `index` and
        split a `search`-only call (slot 1) into two broken calls."""
        detector = PoolsideV1Detector()
        # `search` is at tools-list slot 1, NOT 0.
        wire = (
            "<tool_call>search\n<arg_key>query</arg_key>\n"
            "<arg_value>tacos</arg_value>\n</tool_call>"
        )
        all_calls = []
        for c in list(wire):
            r = detector.parse_streaming_increment(c, self.tools)
            all_calls.extend(r.calls)
        # All chunks for this call must share the same index.
        indices = {c.tool_index for c in all_calls}
        self.assertEqual(
            indices,
            {0},
            f"streaming emitted mixed indices {indices}; OpenAI clients would "
            "split this into multiple broken calls",
        )
        names = [c.name for c in all_calls if c.name]
        params = "".join(c.parameters for c in all_calls if c.parameters)
        self.assertEqual(names, ["search"])
        self.assertEqual(json.loads(params), {"query": "tacos"})
```
**EN:** Regression: streaming emissions must use a per-response sequential index. This test exercises `test_streaming_index_is_sequential_not_tools_slot` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression: streaming emissions must use a per-response sequential index. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_index_is_sequential_not_tools_slot`。

### Lines 412-433: test case streaming multiple calls / 测试用例 streaming multiple calls
```python
    def test_streaming_multiple_calls(self):
        detector = PoolsideV1Detector()
        wire = (
            "<tool_call>get_weather\n<arg_key>location</arg_key>\n"
            "<arg_value>NYC</arg_value>\n</tool_call>\n"
            "<tool_call>search\n<arg_key>query</arg_key>\n"
            "<arg_value>pizza</arg_value>\n</tool_call>"
        )
        all_calls = []
        for chunk in [wire[i : i + 16] for i in range(0, len(wire), 16)]:
            r = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(r.calls)
        names = [c.name for c in all_calls if c.name]
        self.assertEqual(names, ["get_weather", "search"])
        # Each tool's argument deltas concatenate to a complete JSON object
        per_tool: dict = {}
        for c in all_calls:
            if c.parameters:
                per_tool.setdefault(c.tool_index, "")
                per_tool[c.tool_index] += c.parameters
        self.assertEqual(json.loads(per_tool[0]), {"location": "NYC"})
        self.assertEqual(json.loads(per_tool[1]), {"query": "pizza"})
```
**EN:** This test exercises `test_streaming_multiple_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_calls`。

### Lines 435-441: test case streaming zero arg call / 测试用例 streaming zero arg call
```python
    def test_streaming_zero_arg_call(self):
        detector = PoolsideV1Detector()
        wire = "<tool_call>now\n</tool_call>"
        names, params = self._collect(detector, list(wire))
        self.assertEqual(names, ["now"])
        # Either a single "{}" emission or a sequence whose join parses to {}
        self.assertEqual(json.loads(params or "{}"), {})
```
**EN:** This test exercises `test_streaming_zero_arg_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_zero_arg_call`。

### Lines 443-458: test case streaming text before tool call / 测试用例 streaming text before tool call
```python
    def test_streaming_text_before_tool_call(self):
        detector = PoolsideV1Detector()
        chunks = [
            "Let me check. ",
            "<tool_call>search\n<arg_key>query</arg_key>\n",
            "<arg_value>foo</arg_value>\n</tool_call>",
        ]
        all_calls = []
        normal = ""
        for chunk in chunks:
            r = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(r.calls)
            normal += r.normal_text
        self.assertEqual(normal, "Let me check. ")
        names = [c.name for c in all_calls if c.name]
        self.assertEqual(names, ["search"])
```
**EN:** This test exercises `test_streaming_text_before_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_text_before_tool_call`。

### Lines 459-461: supporting source context / 辅助源码上下文
```python

    # ==================== Registry ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 462-466: test case registered in function call parser / 测试用例 registered in function call parser
```python
    def test_registered_in_function_call_parser(self):
        self.assertIn("poolside_v1", FunctionCallParser.ToolCallParserEnum)
        self.assertIs(
            FunctionCallParser.ToolCallParserEnum["poolside_v1"], PoolsideV1Detector
        )
```
**EN:** This test exercises `test_registered_in_function_call_parser` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_registered_in_function_call_parser`。

### Lines 467-469: supporting source context / 辅助源码上下文
```python

    # ==================== Helpers ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 470-477: method collect / 方法 collect
```python
    def _collect(self, detector, chunks):
        all_calls = []
        for chunk in chunks:
            r = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(r.calls)
        names = [c.name for c in all_calls if c.name]
        params = "".join(c.parameters for c in all_calls if c.parameters)
        return names, params
```
**EN:** This block implements `_collect` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_collect`，承担模块行为中的一个聚焦逻辑片段。

### Lines 480-483: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import unittest

    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPoolsideV1Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPoolsideV1Detector.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestPoolsideV1Detector.test_has_tool_call_true`: This test exercises `test_has_tool_call_true` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_true`。
- `TestPoolsideV1Detector.test_has_tool_call_false`: This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。
- `TestPoolsideV1Detector.test_single_tool_call_string_arg`: This test exercises `test_single_tool_call_string_arg` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_string_arg`。
- `TestPoolsideV1Detector.test_single_tool_call_mixed_types`: This test exercises `test_single_tool_call_mixed_types` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_mixed_types`。
- `TestPoolsideV1Detector.test_multiple_tool_calls`: This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。
- `TestPoolsideV1Detector.test_leading_text_extracted_as_normal`: This test exercises `test_leading_text_extracted_as_normal` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_leading_text_extracted_as_normal`。
- `TestPoolsideV1Detector.test_unknown_tool_dropped`: This test exercises `test_unknown_tool_dropped` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_tool_dropped`。
- `TestPoolsideV1Detector.test_malformed_value_falls_back_to_string`: This test exercises `test_malformed_value_falls_back_to_string` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_value_falls_back_to_string`。
- `TestPoolsideV1Detector.test_zero_arg_call`: This test exercises `test_zero_arg_call` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_arg_call`。
- `TestPoolsideV1Detector.test_zero_arg_call_no_newline`: `<tool_call>now</tool_call>` (no `\n` between name and close tag). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_arg_call_no_newline`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.function_call_parser`, `sglang.srt.function_call.poolside_v1_detector`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 483
