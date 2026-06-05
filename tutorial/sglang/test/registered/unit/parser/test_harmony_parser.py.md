# test_harmony_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/parser/test_harmony_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates harmony parser behavior in SGLang's unit / parser area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 解析器 领域中与 harmony parser 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/parser/harmony_parser.py"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-15: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.parser.harmony_parser import (
    CanonicalStrategy,
    Event,
    HarmonyParser,
    TextStrategy,
    Token,
    iter_tokens,
    prefix_hold,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.parser.harmony_parser`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.parser.harmony_parser`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 17-17: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-20: class TestEvent declaration / 类 TestEvent 声明
```python
class TestEvent(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 21-25: test case init / 测试用例 init
```python
    def test_init(self):
        """Test Event dataclass initialization."""
        event = Event("reasoning", "content")
        self.assertEqual(event.event_type, "reasoning")
        self.assertEqual(event.content, "content")
```
**EN:** Test Event dataclass initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Event dataclass initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 28-28: class TestToken declaration / 类 TestToken 声明
```python
class TestToken(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 29-34: test case init / 测试用例 init
```python
    def test_init(self):
        """Test Token dataclass initialization."""
        token = Token("START", 0, 7)
        self.assertEqual(token.type, "START")
        self.assertEqual(token.start, 0)
        self.assertEqual(token.end, 7)
```
**EN:** Test Token dataclass initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Token dataclass initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 37-37: class TestPrefixHold declaration / 类 TestPrefixHold 声明
```python
class TestPrefixHold(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 38-42: test case empty text / 测试用例 empty text
```python
    def test_empty_text(self):
        """Test prefix_hold with empty text."""
        emit, hold = prefix_hold("", ["<|start|>"])
        self.assertEqual(emit, "")
        self.assertEqual(hold, "")
```
**EN:** Test prefix_hold with empty text. This test exercises `test_empty_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prefix_hold with empty text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_text`。

### Lines 44-48: test case no matching prefixes / 测试用例 no matching prefixes
```python
    def test_no_matching_prefixes(self):
        """Test prefix_hold with no matching prefixes."""
        emit, hold = prefix_hold("hello world", ["<|start|>", "<|end|>"])
        self.assertEqual(emit, "hello world")
        self.assertEqual(hold, "")
```
**EN:** Test prefix_hold with no matching prefixes. This test exercises `test_no_matching_prefixes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prefix_hold with no matching prefixes. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_matching_prefixes`。

### Lines 50-54: test case partial token suffix / 测试用例 partial token suffix
```python
    def test_partial_token_suffix(self):
        """Test prefix_hold with partial token at end."""
        emit, hold = prefix_hold("hello <|ret", ["<|return|>"])
        self.assertEqual(emit, "hello ")
        self.assertEqual(hold, "<|ret")
```
**EN:** Test prefix_hold with partial token at end. This test exercises `test_partial_token_suffix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prefix_hold with partial token at end. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_token_suffix`。

### Lines 56-60: test case multiple potential matches / 测试用例 multiple potential matches
```python
    def test_multiple_potential_matches(self):
        """Test prefix_hold with multiple potential matches."""
        emit, hold = prefix_hold("text <|", ["<|start|>", "<|end|>"])
        self.assertEqual(emit, "text ")
        self.assertEqual(hold, "<|")
```
**EN:** Test prefix_hold with multiple potential matches. This test exercises `test_multiple_potential_matches` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prefix_hold with multiple potential matches. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_potential_matches`。

### Lines 62-66: test case exact token match / 测试用例 exact token match
```python
    def test_exact_token_match(self):
        """Test prefix_hold with exact token match."""
        emit, hold = prefix_hold("text <|start|>", ["<|start|>"])
        self.assertEqual(emit, "text <|start|>")
        self.assertEqual(hold, "")
```
**EN:** Test prefix_hold with exact token match. This test exercises `test_exact_token_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prefix_hold with exact token match. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exact_token_match`。

### Lines 69-69: class TestIterTokens declaration / 类 TestIterTokens 声明
```python
class TestIterTokens(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 70-73: test case empty text / 测试用例 empty text
```python
    def test_empty_text(self):
        """Test iter_tokens with empty text."""
        tokens = list(iter_tokens(""))
        self.assertEqual(tokens, [])
```
**EN:** Test iter_tokens with empty text. This test exercises `test_empty_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test iter_tokens with empty text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_text`。

### Lines 75-81: test case plain text / 测试用例 plain text
```python
    def test_plain_text(self):
        """Test iter_tokens with plain text."""
        tokens = list(iter_tokens("hello world"))
        self.assertEqual(len(tokens), 1)
        self.assertEqual(tokens[0].type, "TEXT")
        self.assertEqual(tokens[0].start, 0)
        self.assertEqual(tokens[0].end, 11)
```
**EN:** Test iter_tokens with plain text. This test exercises `test_plain_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test iter_tokens with plain text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plain_text`。

### Lines 83-89: test case single token / 测试用例 single token
```python
    def test_single_token(self):
        """Test iter_tokens with single structural token."""
        tokens = list(iter_tokens("<|start|>"))
        self.assertEqual(len(tokens), 1)
        self.assertEqual(tokens[0].type, "START")
        self.assertEqual(tokens[0].start, 0)
        self.assertEqual(tokens[0].end, 9)
```
**EN:** Test iter_tokens with single structural token. This test exercises `test_single_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test iter_tokens with single structural token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_token`。

### Lines 91-106: test case mixed content / 测试用例 mixed content
```python
    def test_mixed_content(self):
        """Test iter_tokens with mixed text and tokens."""
        tokens = list(iter_tokens("text<|start|>more text"))
        self.assertEqual(len(tokens), 3)

        self.assertEqual(tokens[0].type, "TEXT")
        self.assertEqual(tokens[0].start, 0)
        self.assertEqual(tokens[0].end, 4)

        self.assertEqual(tokens[1].type, "START")
        self.assertEqual(tokens[1].start, 4)
        self.assertEqual(tokens[1].end, 13)

        self.assertEqual(tokens[2].type, "TEXT")
        self.assertEqual(tokens[2].start, 13)
        self.assertEqual(tokens[2].end, 22)
```
**EN:** Test iter_tokens with mixed text and tokens. This test exercises `test_mixed_content` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test iter_tokens with mixed text and tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_content`。

### Lines 108-119: test case unknown token partial suffix / 测试用例 unknown token partial suffix
```python
    def test_unknown_token_partial_suffix(self):
        """Test iter_tokens with unknown token that could be partial."""
        tokens = list(iter_tokens("text <|ret"))
        self.assertEqual(len(tokens), 2)

        self.assertEqual(tokens[0].type, "TEXT")
        self.assertEqual(tokens[0].start, 0)
        self.assertEqual(tokens[0].end, 5)

        self.assertEqual(tokens[1].type, "TEXT")
        self.assertEqual(tokens[1].start, 5)
        self.assertEqual(tokens[1].end, 10)
```
**EN:** Test iter_tokens with unknown token that could be partial. This test exercises `test_unknown_token_partial_suffix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test iter_tokens with unknown token that could be partial. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_token_partial_suffix`。

### Lines 121-129: test case unknown token middle / 测试用例 unknown token middle
```python
    def test_unknown_token_middle(self):
        """Test iter_tokens with unknown token in middle."""
        tokens = list(iter_tokens("text <|weird|> more <|start|>"))
        self.assertEqual(len(tokens), 5)

        self.assertEqual(tokens[0].type, "TEXT")
        self.assertEqual(tokens[1].type, "TEXT")  # "<|"
        self.assertEqual(tokens[2].type, "TEXT")  # "weird|> more "
        self.assertEqual(tokens[3].type, "START")
```
**EN:** Test iter_tokens with unknown token in middle. This test exercises `test_unknown_token_middle` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test iter_tokens with unknown token in middle. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_token_middle`。

### Lines 130-131: supporting source context / 辅助源码上下文
```python
        # No trailing text token since it ends with a known token

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 132-149: test case all structural tokens / 测试用例 all structural tokens
```python
    def test_all_structural_tokens(self):
        """Test iter_tokens recognizes all structural tokens."""
        text = "<|start|><|channel|><|message|><|constrain|><|end|><|call|><|return|>"
        tokens = list(iter_tokens(text))

        expected_types = [
            "START",
            "CHANNEL",
            "MESSAGE",
            "CONSTRAIN",
            "END",
            "CALL",
            "RETURN",
        ]
        self.assertEqual(len(tokens), len(expected_types))

        for token, expected_type in zip(tokens, expected_types):
            self.assertEqual(token.type, expected_type)
```
**EN:** Test iter_tokens recognizes all structural tokens. This test exercises `test_all_structural_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test iter_tokens recognizes all structural tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_structural_tokens`。

### Lines 152-152: class TestCanonicalStrategy declaration / 类 TestCanonicalStrategy 声明
```python
class TestCanonicalStrategy(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 153-154: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = CanonicalStrategy()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 156-159: test case init / 测试用例 init
```python
    def test_init(self):
        """Test CanonicalStrategy initialization."""
        self.assertIn("<|start|>", self.strategy.guard_tokens)
        self.assertIn("<|constrain|>", self.strategy.guard_tokens)
```
**EN:** Test CanonicalStrategy initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test CanonicalStrategy initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 161-170: test case extract channel type / 测试用例 extract channel type
```python
    def test_extract_channel_type(self):
        """Test _extract_channel_type method."""
        self.assertEqual(self.strategy._extract_channel_type("analysis"), "analysis")
        self.assertEqual(
            self.strategy._extract_channel_type("commentary to=functions.tool"),
            "commentary",
        )
        self.assertEqual(self.strategy._extract_channel_type("final to=user"), "final")
        self.assertEqual(self.strategy._extract_channel_type("ANALYSIS"), "analysis")
        self.assertIsNone(self.strategy._extract_channel_type("unknown"))
```
**EN:** Test _extract_channel_type method. This test exercises `test_extract_channel_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test _extract_channel_type method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extract_channel_type`。

### Lines 172-180: test case parse single analysis block / 测试用例 parse single analysis block
```python
    def test_parse_single_analysis_block(self):
        """Test parsing single analysis block."""
        text = "<|channel|>analysis<|message|>Let me think about this<|end|>"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[0].content, "Let me think about this")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing single analysis block. This test exercises `test_parse_single_analysis_block` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing single analysis block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_single_analysis_block`。

### Lines 182-190: test case parse single commentary block / 测试用例 parse single commentary block
```python
    def test_parse_single_commentary_block(self):
        """Test parsing single commentary block."""
        text = "<|channel|>commentary<|message|>User-visible message<|end|>"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, "User-visible message")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing single commentary block. This test exercises `test_parse_single_commentary_block` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing single commentary block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_single_commentary_block`。

### Lines 192-200: test case parse single final block / 测试用例 parse single final block
```python
    def test_parse_single_final_block(self):
        """Test parsing single final block."""
        text = "<|start|>assistant<|channel|>final<|message|>The answer is 42<|return|>"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, "The answer is 42")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing single final block. This test exercises `test_parse_single_final_block` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing single final block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_single_final_block`。

### Lines 202-210: test case parse tool call commentary / 测试用例 parse tool call commentary
```python
    def test_parse_tool_call_commentary(self):
        """Test parsing tool call on commentary channel."""
        text = '<|channel|>commentary to=functions.get_weather<|message|>{"location": "SF"}<|call|>'
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "tool_call")
        self.assertEqual(events[0].content, '{"location": "SF"}')
        self.assertEqual(remaining, "")
```
**EN:** Test parsing tool call on commentary channel. This test exercises `test_parse_tool_call_commentary` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing tool call on commentary channel. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_tool_call_commentary`。

### Lines 212-220: test case parse tool call analysis / 测试用例 parse tool call analysis
```python
    def test_parse_tool_call_analysis(self):
        """Test parsing built-in tool call on analysis channel."""
        text = '<|channel|>analysis to=browser.search<|message|>{"query": "SGLang"}<|call|>'
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "tool_call")
        self.assertEqual(events[0].content, '{"query": "SGLang"}')
        self.assertEqual(remaining, "")
```
**EN:** Test parsing built-in tool call on analysis channel. This test exercises `test_parse_tool_call_analysis` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing built-in tool call on analysis channel. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_tool_call_analysis`。

### Lines 222-236: test case parse complex sequence / 测试用例 parse complex sequence
```python
    def test_parse_complex_sequence(self):
        """Test parsing complex sequence with multiple blocks."""
        text = (
            "<|channel|>analysis<|message|>Need to use function get_weather.<|end|>"
            "<|start|>assistant<|channel|>commentary to=functions.get_weather<|message|>"
            '{"location":"San Francisco"}<|call|>'
        )
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[0].content, "Need to use function get_weather.")
        self.assertEqual(events[1].event_type, "tool_call")
        self.assertEqual(events[1].content, '{"location":"San Francisco"}')
        self.assertEqual(remaining, "")
```
**EN:** Test parsing complex sequence with multiple blocks. This test exercises `test_parse_complex_sequence` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing complex sequence with multiple blocks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_complex_sequence`。

### Lines 238-258: test case parse with interspersed text / 测试用例 parse with interspersed text
```python
    def test_parse_with_interspersed_text(self):
        """Test parsing with plain text between blocks."""
        text = (
            "Some text "
            "<|channel|>analysis<|message|>reasoning<|end|>"
            " more text "
            "<|start|>assistant<|channel|>final<|message|>answer<|return|>"
            " trailing text"
        )
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 4)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, "Some text ")
        self.assertEqual(events[1].event_type, "reasoning")
        self.assertEqual(events[1].content, "reasoning")
        self.assertEqual(events[2].event_type, "normal")
        self.assertEqual(events[2].content, " more text ")
        self.assertEqual(events[3].event_type, "normal")
        self.assertEqual(events[3].content, "answer trailing text")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing with plain text between blocks. This test exercises `test_parse_with_interspersed_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with plain text between blocks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_with_interspersed_text`。

### Lines 260-268: test case parse incomplete block / 测试用例 parse incomplete block
```python
    def test_parse_incomplete_block(self):
        """Test parsing incomplete block (streaming scenario)."""
        text = "<|channel|>analysis<|message|>partial content"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[0].content, "partial content")
        self.assertEqual(remaining, "<|channel|>analysis<|message|>")
```
**EN:** Test parsing incomplete block (streaming scenario). This test exercises `test_parse_incomplete_block` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing incomplete block (streaming scenario). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_incomplete_block`。

### Lines 270-278: test case parse partial token suffix / 测试用例 parse partial token suffix
```python
    def test_parse_partial_token_suffix(self):
        """Test parsing with partial token at end."""
        text = "complete text <|ret"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, "complete text ")
        self.assertEqual(remaining, "<|ret")
```
**EN:** Test parsing with partial token at end. This test exercises `test_parse_partial_token_suffix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with partial token at end. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_partial_token_suffix`。

### Lines 280-288: test case parse tool response message / 测试用例 parse tool response message
```python
    def test_parse_tool_response_message(self):
        """Test parsing tool response message (no channel)."""
        text = '<|start|>functions.get_weather to=assistant<|message|>{"sunny": true}<|end|>'
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, '{"sunny": true}')
        self.assertEqual(remaining, "")
```
**EN:** Test parsing tool response message (no channel). This test exercises `test_parse_tool_response_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing tool response message (no channel). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_tool_response_message`。

### Lines 290-298: test case parse empty content blocks / 测试用例 parse empty content blocks
```python
    def test_parse_empty_content_blocks(self):
        """Test parsing blocks with empty content."""
        text = "<|channel|>analysis<|message|><|end|>"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[0].content, "")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing blocks with empty content. This test exercises `test_parse_empty_content_blocks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing blocks with empty content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_empty_content_blocks`。

### Lines 300-325: test case parse commentary filler between blocks / 测试用例 parse commentary filler between blocks
```python
    def test_parse_commentary_filler_between_blocks(self):
        """Test that 'commentary' filler between <|call|> and <|channel|> is filtered out."""
        # This pattern occurs when the model generates malformed output
        text = (
            '<|channel|>commentary to=functions.get_weather<|message|>{"location":"SF"}<|call|>'
            "commentary"  # This should be filtered out
            '<|channel|>commentary to=functions.get_temp<|message|>{"location":"NYC"}<|call|>'
        )
        events, remaining = self.strategy.parse(text)

        # Should have 2 tool calls, no "commentary" normal text
        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "tool_call")
        self.assertEqual(events[0].content, '{"location":"SF"}')
        self.assertEqual(events[1].event_type, "tool_call")
        self.assertEqual(events[1].content, '{"location":"NYC"}')
        self.assertEqual(remaining, "")

        # Verify no "commentary" text was emitted as normal content
        normal_events = [e for e in events if e.event_type == "normal"]
        commentary_events = [
            e for e in normal_events if "commentary" in e.content.lower()
        ]
        self.assertEqual(
            len(commentary_events), 0, "Commentary filler should be filtered out"
        )
```
**EN:** Test that 'commentary' filler between <|call|> and <|channel|> is filtered out. This test exercises `test_parse_commentary_filler_between_blocks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that 'commentary' filler between <|call|> and <|channel|> is filtered out. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_commentary_filler_between_blocks`。

### Lines 328-328: class TestTextStrategy declaration / 类 TestTextStrategy 声明
```python
class TestTextStrategy(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 329-330: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = TextStrategy()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 332-334: test case init / 测试用例 init
```python
    def test_init(self):
        """Test TextStrategy initialization."""
        self.assertIn("analysis_then_final", self.strategy.patterns)
```
**EN:** Test TextStrategy initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test TextStrategy initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 336-346: test case parse analysis then final / 测试用例 parse analysis then final
```python
    def test_parse_analysis_then_final(self):
        """Test parsing analysis then final format."""
        text = "analysis I need to think about this. assistantfinal The answer is 42."
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[0].content, "I need to think about this.")
        self.assertEqual(events[1].event_type, "normal")
        self.assertEqual(events[1].content, "The answer is 42.")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing analysis then final format. This test exercises `test_parse_analysis_then_final` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing analysis then final format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_analysis_then_final`。

### Lines 348-358: test case parse commentary then final / 测试用例 parse commentary then final
```python
    def test_parse_commentary_then_final(self):
        """Test parsing commentary then final format."""
        text = "commentary User-visible preamble. assistantfinal The answer is 42."
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, "User-visible preamble.")
        self.assertEqual(events[1].event_type, "normal")
        self.assertEqual(events[1].content, "The answer is 42.")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing commentary then final format. This test exercises `test_parse_commentary_then_final` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing commentary then final format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_commentary_then_final`。

### Lines 360-368: test case parse final only / 测试用例 parse final only
```python
    def test_parse_final_only(self):
        """Test parsing final-only format."""
        text = "assistantfinal The direct answer."
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, "The direct answer.")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing final-only format. This test exercises `test_parse_final_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing final-only format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_final_only`。

### Lines 370-379: test case parse analysis only / 测试用例 parse analysis only
```python
    def test_parse_analysis_only(self):
        """Test parsing analysis-only format."""
        text = "analysis This is reasoning content."
        events, remaining = self.strategy.parse(text)

        # For analysis-only, streaming parse should keep header and emit with leading space
        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[0].content, " This is reasoning content.")
        self.assertEqual(remaining, "analysis")
```
**EN:** Test parsing analysis-only format. This test exercises `test_parse_analysis_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing analysis-only format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_analysis_only`。

### Lines 381-387: test case parse incomplete assistantfinal / 测试用例 parse incomplete assistantfinal
```python
    def test_parse_incomplete_assistantfinal(self):
        """Test parsing with incomplete assistantfinal."""
        text = "analysis reasoning content assistantfin"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 0)
        self.assertEqual(remaining, text)  # Hold entire buffer
```
**EN:** Test parsing with incomplete assistantfinal. This test exercises `test_parse_incomplete_assistantfinal` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with incomplete assistantfinal. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_incomplete_assistantfinal`。

### Lines 389-397: test case parse partial analysis streaming / 测试用例 parse partial analysis streaming
```python
    def test_parse_partial_analysis_streaming(self):
        """Test streaming partial analysis content."""
        text = "analysis partial content"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[0].content, " partial content")  # Space preserved
        self.assertEqual(remaining, "analysis")  # Hold header
```
**EN:** Test streaming partial analysis content. This test exercises `test_parse_partial_analysis_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming partial analysis content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_partial_analysis_streaming`。

### Lines 399-406: test case parse case insensitive / 测试用例 parse case insensitive
```python
    def test_parse_case_insensitive(self):
        """Test case insensitive parsing."""
        text = "ANALYSIS reasoning ASSISTANTFINAL answer"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[1].event_type, "normal")
```
**EN:** Test case insensitive parsing. This test exercises `test_parse_case_insensitive` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test case insensitive parsing. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_case_insensitive`。

### Lines 408-416: test case parse plain text fallback / 测试用例 parse plain text fallback
```python
    def test_parse_plain_text_fallback(self):
        """Test parsing plain text without harmony markers."""
        text = "Just plain text without any markers."
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, "Just plain text without any markers.")
        self.assertEqual(remaining, "")
```
**EN:** Test parsing plain text without harmony markers. This test exercises `test_parse_plain_text_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing plain text without harmony markers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_plain_text_fallback`。

### Lines 418-432: test case parse analysis no space after header / 测试用例 parse analysis no space after header
```python
    def test_parse_analysis_no_space_after_header(self):
        """Test parsing analysis format without space after header (real gpt-oss output)."""
        text = "analysisThe user typed random strings. We should respond politely.assistantfinalIt looks like you're testing. How can I help?"
        events, remaining = self.strategy.parse(text)

        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(
            events[0].content,
            "The user typed random strings. We should respond politely.",
        )
        self.assertEqual(events[1].event_type, "normal")
        self.assertEqual(
            events[1].content, "It looks like you're testing. How can I help?"
        )
```
**EN:** Test parsing analysis format without space after header (real gpt-oss output). This test exercises `test_parse_analysis_no_space_after_header` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing analysis format without space after header (real gpt-oss output). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_analysis_no_space_after_header`。

### Lines 435-435: class TestHarmonyParser declaration / 类 TestHarmonyParser 声明
```python
class TestHarmonyParser(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 436-437: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.parser = HarmonyParser()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 439-442: test case init / 测试用例 init
```python
    def test_init(self):
        """Test HarmonyParser initialization."""
        self.assertIsNone(self.parser.strategy)
        self.assertEqual(self.parser._buffer, "")
```
**EN:** Test HarmonyParser initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test HarmonyParser initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 444-450: test case strategy selection canonical / 测试用例 strategy selection canonical
```python
    def test_strategy_selection_canonical(self):
        """Test automatic strategy selection for canonical format."""
        events = self.parser.parse("<|channel|>analysis<|message|>test<|end|>")

        self.assertIsInstance(self.parser.strategy, CanonicalStrategy)
        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "reasoning")
```
**EN:** Test automatic strategy selection for canonical format. This test exercises `test_strategy_selection_canonical` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test automatic strategy selection for canonical format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strategy_selection_canonical`。

### Lines 452-458: test case strategy selection text / 测试用例 strategy selection text
```python
    def test_strategy_selection_text(self):
        """Test automatic strategy selection for text format."""
        events = self.parser.parse("analysis test content")

        self.assertIsInstance(self.parser.strategy, TextStrategy)
        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "reasoning")
```
**EN:** Test automatic strategy selection for text format. This test exercises `test_strategy_selection_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test automatic strategy selection for text format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strategy_selection_text`。

### Lines 460-470: test case strategy selection delayed / 测试用例 strategy selection delayed
```python
    def test_strategy_selection_delayed(self):
        """Test strategy selection with insufficient initial content."""
        # First chunk doesn't have enough info
        events1 = self.parser.parse("some")
        self.assertEqual(len(events1), 0)
        self.assertIsNone(self.parser.strategy)

        # Second chunk triggers strategy selection
        events2 = self.parser.parse(" analysis content")
        self.assertIsInstance(self.parser.strategy, TextStrategy)
        self.assertEqual(len(events2), 1)
```
**EN:** Test strategy selection with insufficient initial content. This test exercises `test_strategy_selection_delayed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test strategy selection with insufficient initial content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strategy_selection_delayed`。

### Lines 472-504: test case streaming canonical format / 测试用例 streaming canonical format
```python
    def test_streaming_canonical_format(self):
        """Test streaming with canonical format."""
        chunks = [
            "<|channel|>analysis<|message|>",
            "reasoning content",
            "<|end|>",
            "<|start|>assistant<|channel|>final<|message|>",
            "final answer",
            "<|return|>",
        ]

        all_events = []
        for chunk in chunks:
            events = self.parser.parse(chunk)
            all_events.extend(events)

        # Verify we get both reasoning and normal events
        reasoning_events = [e for e in all_events if e.event_type == "reasoning"]
        self.assertGreater(len(reasoning_events), 0)

        normal_events = [e for e in all_events if e.event_type == "normal"]
        self.assertGreater(len(normal_events), 0)

        # Verify content is eventually parsed correctly
        combined_reasoning = "".join(e.content for e in reasoning_events)
        combined_normal = "".join(
            e.content
            for e in normal_events
            if e.content and "<|return|>" not in e.content
        )

        self.assertIn("reasoning content", combined_reasoning)
        self.assertIn("final answer", combined_normal)
```
**EN:** Test streaming with canonical format. This test exercises `test_streaming_canonical_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with canonical format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_canonical_format`。

### Lines 506-520: test case streaming text format / 测试用例 streaming text format
```python
    def test_streaming_text_format(self):
        """Test streaming with text format."""
        chunks = ["analysis reasoning", " content assistantfinal", " the answer"]

        all_events = []
        for chunk in chunks:
            events = self.parser.parse(chunk)
            all_events.extend(events)

        # Should have reasoning and normal events
        reasoning_events = [e for e in all_events if e.event_type == "reasoning"]
        normal_events = [e for e in all_events if e.event_type == "normal"]

        self.assertGreater(len(reasoning_events), 0)
        self.assertGreater(len(normal_events), 0)
```
**EN:** Test streaming with text format. This test exercises `test_streaming_text_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with text format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_text_format`。

### Lines 522-568: test case streaming commentary filler / 测试用例 streaming commentary filler
```python
    def test_streaming_commentary_filler(self):
        """Test that 'commentary' filler is filtered in streaming case."""
        # Test when commentary arrives as a separate chunk after <|call|>
        chunks = [
            "<|channel|>commentary to=functions.get_weather",
            "<|message|>",
            '{"location":"SF"}',
            "<|call|>",
            "comment",  # This arrives as separate chunk - should be filtered
            "ary",  # Continuation of the filler - should be filtered
            "<|channel|>commentary to=functions.get_temp",
            "<|message|>",
            '{"location":"NYC"}',
            "<|call|>",
            "comment",  # Another separate chunk - should be filtered
            "ary",  # Continuation of the filler - should be filtered
            "<|start|>assistant<|channel|>final",
            "<|message|>Done<|return|>",
        ]

        all_events = []
        for chunk in chunks:
            events = self.parser.parse(chunk)
            all_events.extend(events)

        # Count event types
        tool_events = [e for e in all_events if e.event_type == "tool_call"]
        normal_events = [e for e in all_events if e.event_type == "normal"]

        # Should have 2 tool calls and 1 final message
        self.assertEqual(len(tool_events), 2, "Should have 2 tool calls")
        self.assertEqual(
            len(normal_events), 1, "Should have 1 normal event (final message)"
        )

        # Verify no "commentary" in normal events
        for event in normal_events:
            self.assertNotEqual(
                event.content.strip().lower(),
                "commentary",
                "Commentary filler should not appear as normal content in streaming",
            )

        # Verify content
        self.assertEqual(tool_events[0].content, '{"location":"SF"}')
        self.assertEqual(tool_events[1].content, '{"location":"NYC"}')
        self.assertEqual(normal_events[0].content, "Done")
```
**EN:** Test that 'commentary' filler is filtered in streaming case. This test exercises `test_streaming_commentary_filler` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that 'commentary' filler is filtered in streaming case. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_commentary_filler`。

### Lines 570-609: test case repetitive tool calls with commentary filler / 测试用例 repetitive tool calls with commentary filler
```python
    def test_repetitive_tool_calls_with_commentary_filler(self):
        """Test handling of repetitive tool calls with 'commentary' filler text."""
        # This simulates malformed output with repeated tool calls and commentary filler
        text = (
            "<|channel|>analysis<|message|>Need to get weather<|end|>"
            '<|start|>assistant<|channel|>commentary to=functions.get_weather<|message|>{"city":"Boston"}<|call|>'
            "commentary"  # Filler that should be filtered
            '<|channel|>commentary to=functions.get_weather<|message|>{"city":"Boston"}<|call|>'
            "commentary"  # Another filler
            '<|channel|>commentary to=functions.get_weather<|message|>{"city":"Boston"}<|call|>'
            "<|channel|>analysis<|message|>Tool not responding<|end|>"
            "<|start|>assistant<|channel|>final<|message|>Unable to fetch weather data<|return|>"
        )

        events = self.parser.parse(text)

        # Count event types
        reasoning_events = [e for e in events if e.event_type == "reasoning"]
        tool_events = [e for e in events if e.event_type == "tool_call"]
        normal_events = [e for e in events if e.event_type == "normal"]

        # Verify correct number of each type
        self.assertEqual(len(reasoning_events), 2, "Should have 2 reasoning events")
        self.assertEqual(len(tool_events), 3, "Should have 3 tool calls")
        self.assertEqual(
            len(normal_events), 1, "Should have 1 normal event (final message)"
        )

        # Verify no "commentary" filler in normal events
        for event in normal_events:
            self.assertNotEqual(
                event.content.strip().lower(),
                "commentary",
                "Commentary filler should not appear as normal content",
            )

        # Verify content is correct
        self.assertEqual(reasoning_events[0].content, "Need to get weather")
        self.assertEqual(reasoning_events[1].content, "Tool not responding")
        self.assertEqual(normal_events[0].content, "Unable to fetch weather data")
```
**EN:** Test handling of repetitive tool calls with 'commentary' filler text. This test exercises `test_repetitive_tool_calls_with_commentary_filler` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of repetitive tool calls with 'commentary' filler text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repetitive_tool_calls_with_commentary_filler`。

### Lines 612-612: class TestIntegrationScenarios declaration / 类 TestIntegrationScenarios 声明
```python
class TestIntegrationScenarios(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 613-613: supporting statements / 辅助语句
```python
    """Integration tests for realistic Harmony parsing scenarios."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 615-630: test case complete reasoning flow / 测试用例 complete reasoning flow
```python
    def test_complete_reasoning_flow(self):
        """Test complete reasoning flow from HARMONY_DOCS.md examples."""
        parser = HarmonyParser()

        text = (
            '<|channel|>analysis<|message|>User asks: "What is 2 + 2?" Simple arithmetic. Provide answer.<|end|>'
            "<|start|>assistant<|channel|>final<|message|>2 + 2 = 4.<|return|>"
        )

        events = parser.parse(text)

        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertIn("Simple arithmetic", events[0].content)
        self.assertEqual(events[1].event_type, "normal")
        self.assertEqual(events[1].content, "2 + 2 = 4.")
```
**EN:** Test complete reasoning flow from HARMONY_DOCS.md examples. This test exercises `test_complete_reasoning_flow` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test complete reasoning flow from HARMONY_DOCS.md examples. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_complete_reasoning_flow`。

### Lines 632-648: test case tool call sequence / 测试用例 tool call sequence
```python
    def test_tool_call_sequence(self):
        """Test tool call sequence from HARMONY_DOCS.md examples."""
        parser = HarmonyParser()

        text = (
            "<|channel|>analysis<|message|>Need to use function get_weather.<|end|>"
            "<|start|>assistant<|channel|>commentary to=functions.get_weather <|constrain|>json<|message|>"
            '{"location":"San Francisco"}<|call|>'
        )

        events = parser.parse(text)

        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[0].content, "Need to use function get_weather.")
        self.assertEqual(events[1].event_type, "tool_call")
        self.assertEqual(events[1].content, '{"location":"San Francisco"}')
```
**EN:** Test tool call sequence from HARMONY_DOCS.md examples. This test exercises `test_tool_call_sequence` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tool call sequence from HARMONY_DOCS.md examples. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_sequence`。

### Lines 650-667: test case preamble sequence / 测试用例 preamble sequence
```python
    def test_preamble_sequence(self):
        """Test preamble sequence with multiple commentary blocks."""
        parser = HarmonyParser()

        text = (
            "<|channel|>analysis<|message|>Long chain of thought<|end|>"
            "<|start|>assistant<|channel|>commentary<|message|>**Action plan**: 1. Generate file 2. Start server<|end|>"
            "<|start|>assistant<|channel|>commentary to=functions.generate_file<|message|>"
            '{"template": "basic_html"}<|call|>'
        )

        events = parser.parse(text)

        self.assertEqual(len(events), 3)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[1].event_type, "normal")
        self.assertIn("Action plan", events[1].content)
        self.assertEqual(events[2].event_type, "tool_call")
```
**EN:** Test preamble sequence with multiple commentary blocks. This test exercises `test_preamble_sequence` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test preamble sequence with multiple commentary blocks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preamble_sequence`。

### Lines 669-679: test case built in tool call / 测试用例 built in tool call
```python
    def test_built_in_tool_call(self):
        """Test built-in tool call on analysis channel."""
        parser = HarmonyParser()

        text = '<|channel|>analysis to=browser.search<|message|>{"query": "SGLang"}<|call|>'

        events = parser.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "tool_call")
        self.assertEqual(events[0].content, '{"query": "SGLang"}')
```
**EN:** Test built-in tool call on analysis channel. This test exercises `test_built_in_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test built-in tool call on analysis channel. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_built_in_tool_call`。

### Lines 681-691: test case tool response handling / 测试用例 tool response handling
```python
    def test_tool_response_handling(self):
        """Test tool response message handling."""
        parser = HarmonyParser()

        text = '<|start|>functions.get_weather to=assistant<|channel|>commentary<|message|>{"sunny": true, "temperature": 20}<|end|>'

        events = parser.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].event_type, "normal")
        self.assertEqual(events[0].content, '{"sunny": true, "temperature": 20}')
```
**EN:** Test tool response message handling. This test exercises `test_tool_response_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tool response message handling. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_response_handling`。

### Lines 693-708: test case text fallback formats / 测试用例 text fallback formats
```python
    def test_text_fallback_formats(self):
        """Test various text fallback formats."""
        parser = HarmonyParser()

        # Test analysis then final
        events1 = parser.parse("analysis thinking assistantfinal answer")
        self.assertEqual(len([e for e in events1 if e.event_type == "reasoning"]), 1)
        self.assertEqual(len([e for e in events1 if e.event_type == "normal"]), 1)

        # Reset parser for next test
        parser = HarmonyParser()

        # Test final only
        events2 = parser.parse("assistantfinal direct answer")
        self.assertEqual(len(events2), 1)
        self.assertEqual(events2[0].event_type, "normal")
```
**EN:** Test various text fallback formats. This test exercises `test_text_fallback_formats` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test various text fallback formats. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_text_fallback_formats`。

### Lines 710-756: test case streaming property canonical / 测试用例 streaming property canonical
```python
    def test_streaming_property_canonical(self):
        """Test streaming property: chunked parsing produces same semantic content as one-shot parsing."""
        full_text = (
            "<|channel|>analysis<|message|>reasoning content<|end|>"
            "<|start|>assistant<|channel|>final<|message|>final content"
        )

        # One-shot parsing
        parser1 = HarmonyParser()
        events_oneshot = parser1.parse(full_text)
        events_oneshot += parser1.parse("")

        # Chunked parsing
        parser2 = HarmonyParser()
        chunks = [
            "<|channel|>",
            "analysis",
            "<|message|>",
            "reasoning content",
            "<|end|>",
            "<|start|>assistant",
            "<|channel|>final",
            "<|message|>",
            "final ",
            "content",
        ]
        events_chunked = []
        for chunk in chunks:
            events_chunked.extend(parser2.parse(chunk))

        # Compare semantic content rather than exact event structure
        reasoning_oneshot = "".join(
            e.content for e in events_oneshot if e.event_type == "reasoning"
        )
        normal_oneshot = "".join(
            e.content for e in events_oneshot if e.event_type == "normal"
        )

        reasoning_chunked = "".join(
            e.content for e in events_chunked if e.event_type == "reasoning"
        )
        normal_chunked = "".join(
            e.content for e in events_chunked if e.event_type == "normal"
        )

        self.assertEqual(reasoning_chunked, reasoning_oneshot)
        self.assertEqual(normal_chunked, normal_oneshot)
```
**EN:** Test streaming property: chunked parsing produces same semantic content as one-shot parsing. This test exercises `test_streaming_property_canonical` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming property: chunked parsing produces same semantic content as one-shot parsing. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_property_canonical`。

### Lines 758-790: test case streaming property text / 测试用例 streaming property text
```python
    def test_streaming_property_text(self):
        """Test streaming property for text format."""
        full_text = "analysis reasoning content assistantfinal final answer"

        # One-shot parsing
        parser1 = HarmonyParser()
        events_oneshot = parser1.parse(full_text)

        # Chunked parsing
        parser2 = HarmonyParser()
        chunks = ["analysis reason", "ing content assistant", "final final answer"]
        events_chunked = []
        for chunk in chunks:
            events_chunked.extend(parser2.parse(chunk))

        # Combine content by type for comparison
        reasoning_oneshot = "".join(
            e.content for e in events_oneshot if e.event_type == "reasoning"
        )
        normal_oneshot = "".join(
            e.content for e in events_oneshot if e.event_type == "normal"
        )

        reasoning_chunked = "".join(
            e.content for e in events_chunked if e.event_type == "reasoning"
        )
        normal_chunked = "".join(
            e.content for e in events_chunked if e.event_type == "normal"
        )

        # Account for whitespace differences due to streaming - compare trimmed content
        self.assertEqual(reasoning_oneshot.strip(), reasoning_chunked.strip())
        self.assertEqual(normal_oneshot.strip(), normal_chunked.strip())
```
**EN:** Test streaming property for text format. This test exercises `test_streaming_property_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming property for text format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_property_text`。

### Lines 793-793: class TestEdgeCases declaration / 类 TestEdgeCases 声明
```python
class TestEdgeCases(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 794-794: supporting statements / 辅助语句
```python
    """Test edge cases and error conditions."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 796-805: test case malformed channel headers / 测试用例 malformed channel headers
```python
    def test_malformed_channel_headers(self):
        """Test handling of malformed channel headers."""
        parser = HarmonyParser()

        # Unknown channel type
        text = "<|channel|>unknown<|message|>content<|end|>"
        events = parser.parse(text)

        # Should be held as incomplete since channel is unknown
        self.assertEqual(len(events), 0)
```
**EN:** Test handling of malformed channel headers. This test exercises `test_malformed_channel_headers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of malformed channel headers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_channel_headers`。

### Lines 807-819: test case mixed unknown tokens / 测试用例 mixed unknown tokens
```python
    def test_mixed_unknown_tokens(self):
        """Test handling of mixed unknown tokens."""
        parser = HarmonyParser()

        text = "text <|weird|> more text <|channel|>analysis<|message|>content<|end|>"
        events = parser.parse(text)

        # Should parse the valid parts
        reasoning_events = [e for e in events if e.event_type == "reasoning"]
        normal_events = [e for e in events if e.event_type == "normal"]

        self.assertEqual(len(reasoning_events), 1)
        self.assertGreater(len(normal_events), 0)
```
**EN:** Test handling of mixed unknown tokens. This test exercises `test_mixed_unknown_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of mixed unknown tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_unknown_tokens`。

### Lines 821-825: test case empty input / 测试用例 empty input
```python
    def test_empty_input(self):
        """Test handling of empty input."""
        parser = HarmonyParser()
        events = parser.parse("")
        self.assertEqual(len(events), 0)
```
**EN:** Test handling of empty input. This test exercises `test_empty_input` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of empty input. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_input`。

### Lines 827-835: test case whitespace preservation / 测试用例 whitespace preservation
```python
    def test_whitespace_preservation(self):
        """Test that whitespace is preserved correctly."""
        parser = HarmonyParser()

        text = "<|channel|>analysis<|message|>  content with spaces  <|end|>"
        events = parser.parse(text)

        self.assertEqual(len(events), 1)
        self.assertEqual(events[0].content, "  content with spaces  ")
```
**EN:** Test that whitespace is preserved correctly. This test exercises `test_whitespace_preservation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that whitespace is preserved correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_whitespace_preservation`。

### Lines 837-858: test case streaming whitespace preservation / 测试用例 streaming whitespace preservation
```python
    def test_streaming_whitespace_preservation(self):
        """Test that streaming preserves whitespace between chunks."""
        parser = HarmonyParser()

        # Simulate streaming where space is at chunk boundary
        chunks = ["analysis The user typed ", '"wapppa". Not a question.']

        all_events = []
        for chunk in chunks:
            events = parser.parse(chunk)
            all_events.extend(events)

        # Combine all reasoning content
        reasoning_content = "".join(
            e.content for e in all_events if e.event_type == "reasoning"
        )

        # Should preserve the space before the quote
        self.assertIn('typed "wapppa"', reasoning_content)
        self.assertNotIn(
            'typed"wapppa"', reasoning_content
        )  # Should not be mashed together
```
**EN:** Test that streaming preserves whitespace between chunks. This test exercises `test_streaming_whitespace_preservation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that streaming preserves whitespace between chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_whitespace_preservation`。

### Lines 860-874: test case consecutive blocks same type / 测试用例 consecutive blocks same type
```python
    def test_consecutive_blocks_same_type(self):
        """Test consecutive blocks of the same type."""
        parser = HarmonyParser()

        text = (
            "<|channel|>analysis<|message|>first reasoning<|end|>"
            "<|channel|>analysis<|message|>second reasoning<|end|>"
        )
        events = parser.parse(text)

        self.assertEqual(len(events), 2)
        self.assertEqual(events[0].event_type, "reasoning")
        self.assertEqual(events[1].event_type, "reasoning")
        self.assertEqual(events[0].content, "first reasoning")
        self.assertEqual(events[1].content, "second reasoning")
```
**EN:** Test consecutive blocks of the same type. This test exercises `test_consecutive_blocks_same_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test consecutive blocks of the same type. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_consecutive_blocks_same_type`。

### Lines 877-877: class TestAdditionalEdgeCases declaration / 类 TestAdditionalEdgeCases 声明
```python
class TestAdditionalEdgeCases(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 878-878: supporting statements / 辅助语句
```python
    """Additional tests to cover remaining edge cases."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 880-886: test case prefix hold with empty token in list / 测试用例 prefix hold with empty token in list
```python
    def test_prefix_hold_with_empty_token_in_list(self):
        """Test that empty string token in the list is skipped."""
        from sglang.srt.parser.harmony_parser import prefix_hold

        emit, hold = prefix_hold("hello", ["", "world"])
        self.assertEqual(emit, "hello")
        self.assertEqual(hold, "")
```
**EN:** Test that empty string token in the list is skipped. This test exercises `test_prefix_hold_with_empty_token_in_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that empty string token in the list is skipped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prefix_hold_with_empty_token_in_list`。

### Lines 888-894: test case iter tokens unknown token no closing / 测试用例 iter tokens unknown token no closing
```python
    def test_iter_tokens_unknown_token_no_closing(self):
        """Test iter_tokens with <| that has no closing |>."""
        from sglang.srt.parser.harmony_parser import iter_tokens

        tokens = list(iter_tokens("<|broken text without close", 0))
        # Should emit TEXT tokens for the content after <|
        self.assertTrue(any(t.type == "TEXT" for t in tokens))
```
**EN:** Test iter_tokens with <| that has no closing |>. This test exercises `test_iter_tokens_unknown_token_no_closing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test iter_tokens with <| that has no closing |>. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_iter_tokens_unknown_token_no_closing`。

### Lines 896-905: test case canonical commentary filler after call / 测试用例 canonical commentary filler after call
```python
    def test_canonical_commentary_filler_after_call(self):
        """Test that MESSAGE token after CALL is filtered as commentary filler."""
        from sglang.srt.parser.harmony_parser import CanonicalStrategy

        strategy = CanonicalStrategy()
        text = "<|start|><|channel|>analysis<|message|>thinking<|end|><|call|><|message|>noise<|return|><|channel|>final<|message|>answer<|end|>"
        events, remainder = strategy.parse(text)
        # The MESSAGE after CALL should be filtered, final answer should appear
        answers = [e.content for e in events if e.event_type == "normal"]
        self.assertTrue(any("answer" in a for a in answers))
```
**EN:** Test that MESSAGE token after CALL is filtered as commentary filler. This test exercises `test_canonical_commentary_filler_after_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that MESSAGE token after CALL is filtered as commentary filler. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_canonical_commentary_filler_after_call`。

### Lines 907-916: test case canonical standalone structural token filtered / 测试用例 canonical standalone structural token filtered
```python
    def test_canonical_standalone_structural_token_filtered(self):
        """Test that standalone structural tokens like <|end|> in TEXT position are filtered."""
        from sglang.srt.parser.harmony_parser import CanonicalStrategy

        strategy = CanonicalStrategy()
        # A malformed sequence where an END token appears in an unexpected position
        text = "<|start|><|channel|>analysis<|message|>content<|end|>"
        events, remainder = strategy.parse(text)
        # Should parse without error
        self.assertTrue(len(events) >= 0)
```
**EN:** Test that standalone structural tokens like <|end|> in TEXT position are filtered. This test exercises `test_canonical_standalone_structural_token_filtered` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that standalone structural tokens like <|end|> in TEXT position are filtered. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_canonical_standalone_structural_token_filtered`。

### Lines 918-929: test case canonical incomplete block returns partial / 测试用例 canonical incomplete block returns partial
```python
    def test_canonical_incomplete_block_returns_partial(self):
        """Test parsing an incomplete channel block (no END token)."""
        from sglang.srt.parser.harmony_parser import CanonicalStrategy

        strategy = CanonicalStrategy()
        text = "<|start|><|channel|>analysis<|message|>partial content"
        events, remainder = strategy.parse(text)
        # Incomplete block: should hold content as remainder or emit partial
        reasoning_events = [e for e in events if e.event_type == "reasoning"]
        # The partial content may be in events or remainder
        total = "".join(e.content for e in reasoning_events) + remainder
        self.assertIn("partial", total)
```
**EN:** Test parsing an incomplete channel block (no END token). This test exercises `test_canonical_incomplete_block_returns_partial` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing an incomplete channel block (no END token). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_canonical_incomplete_block_returns_partial`。

### Lines 931-939: test case text strategy commentary channel / 测试用例 text strategy commentary channel
```python
    def test_text_strategy_commentary_channel(self):
        """Test TextStrategy parsing commentary channel."""
        from sglang.srt.parser.harmony_parser import TextStrategy

        strategy = TextStrategy()
        text = "commentary: some discussion\nassistantfinal: the answer"
        events, remainder = strategy.parse(text)
        normal = [e for e in events if e.event_type == "normal"]
        self.assertTrue(any("the answer" in e.content for e in normal))
```
**EN:** Test TextStrategy parsing commentary channel. This test exercises `test_text_strategy_commentary_channel` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test TextStrategy parsing commentary channel. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_text_strategy_commentary_channel`。

### Lines 941-949: test case canonical call with text commentary after / 测试用例 canonical call with text commentary after
```python
    def test_canonical_call_with_text_commentary_after(self):
        """Test filtering of 'commentary' text after CALL token."""
        from sglang.srt.parser.harmony_parser import CanonicalStrategy

        strategy = CanonicalStrategy()
        text = "<|start|><|channel|>analysis<|message|>think<|end|><|call|>commentary<|return|><|channel|>final<|message|>result<|end|>"
        events, remainder = strategy.parse(text)
        normal = [e for e in events if e.event_type == "normal"]
        self.assertTrue(any("result" in e.content for e in normal))
```
**EN:** Test filtering of 'commentary' text after CALL token. This test exercises `test_canonical_call_with_text_commentary_after` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test filtering of 'commentary' text after CALL token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_canonical_call_with_text_commentary_after`。

### Lines 951-960: test case canonical return without final / 测试用例 canonical return without final
```python
    def test_canonical_return_without_final(self):
        """Test that _parse_block returns None for block without proper end."""
        from sglang.srt.parser.harmony_parser import CanonicalStrategy

        strategy = CanonicalStrategy()
        # Channel block that has no message content before end
        text = "<|start|><|channel|>final<|end|>"
        events, remainder = strategy.parse(text)
        # Should handle gracefully
        self.assertIsInstance(events, list)
```
**EN:** Test that _parse_block returns None for block without proper end. This test exercises `test_canonical_return_without_final` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that _parse_block returns None for block without proper end. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_canonical_return_without_final`。

### Lines 962-972: test case iter tokens unknown at end no next marker / 测试用例 iter tokens unknown at end no next marker
```python
    def test_iter_tokens_unknown_at_end_no_next_marker(self):
        """Test unknown token with |> close but no next <| marker after it."""
        from sglang.srt.parser.harmony_parser import iter_tokens

        # <|weird|> is unknown, has closing |>, but nothing after it
        tokens = list(iter_tokens("<|weird|>trailing", 0))
        # Should emit TEXT tokens covering the content
        all_text = "".join(
            "<|weird|>trailing"[t.start : t.end] for t in tokens if t.type == "TEXT"
        )
        self.assertIn("weird|>trailing", all_text)
```
**EN:** Test unknown token with |> close but no next <| marker after it. This test exercises `test_iter_tokens_unknown_at_end_no_next_marker` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test unknown token with |> close but no next <| marker after it. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_iter_tokens_unknown_at_end_no_next_marker`。

### Lines 974-984: test case canonical standalone end token filtered / 测试用例 canonical standalone end token filtered
```python
    def test_canonical_standalone_end_token_filtered(self):
        """Test that standalone <|end|> in TEXT position is filtered out."""
        from sglang.srt.parser.harmony_parser import CanonicalStrategy

        strategy = CanonicalStrategy()
        # Malformed: <|end|> appears before any channel/message structure
        text = "<|end|><|start|><|channel|>final<|message|>answer<|end|>"
        events, remainder = strategy.parse(text)
        # The standalone <|end|> should be filtered, answer should appear
        normal = [e.content for e in events if e.event_type == "normal"]
        self.assertTrue(any("answer" in c for c in normal))
```
**EN:** Test that standalone <|end|> in TEXT position is filtered out. This test exercises `test_canonical_standalone_end_token_filtered` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that standalone <|end|> in TEXT position is filtered out. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_canonical_standalone_end_token_filtered`。

### Lines 986-996: test case canonical incomplete parse block no end / 测试用例 canonical incomplete parse block no end
```python
    def test_canonical_incomplete_parse_block_no_end(self):
        """Test that a channel block without END/CALL/RETURN returns None (incomplete)."""
        from sglang.srt.parser.harmony_parser import CanonicalStrategy

        strategy = CanonicalStrategy()
        # Channel with message but no end token
        text = "<|start|><|channel|>final<|message|>partial"
        events, remainder = strategy.parse(text)
        # Should be treated as incomplete
        total = "".join(e.content for e in events) + remainder
        self.assertIn("partial", total)
```
**EN:** Test that a channel block without END/CALL/RETURN returns None (incomplete). This test exercises `test_canonical_incomplete_parse_block_no_end` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a channel block without END/CALL/RETURN returns None (incomplete). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_canonical_incomplete_parse_block_no_end`。

### Lines 998-1008: test case text strategy commentary only / 测试用例 text strategy commentary only
```python
    def test_text_strategy_commentary_only(self):
        """Test TextStrategy with commentary-only pattern (no 'assistantfinal')."""
        from sglang.srt.parser.harmony_parser import TextStrategy

        strategy = TextStrategy()
        text = "commentary: just a comment here"
        events, remainder = strategy.parse(text)
        normal = [e for e in events if e.event_type == "normal"]
        # Commentary content should appear as normal text
        combined = "".join(e.content for e in normal) + remainder
        self.assertIn("comment", combined)
```
**EN:** Test TextStrategy with commentary-only pattern (no 'assistantfinal'). This test exercises `test_text_strategy_commentary_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test TextStrategy with commentary-only pattern (no 'assistantfinal'). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_text_strategy_commentary_only`。

### Lines 1010-1019: test case text strategy commentary with hold / 测试用例 text strategy commentary with hold
```python
    def test_text_strategy_commentary_with_hold(self):
        """Test TextStrategy commentary channel with prefix that could be 'assistantfinal'."""
        from sglang.srt.parser.harmony_parser import TextStrategy

        strategy = TextStrategy()
        # Content ends with "assistant" which is a prefix of "assistantfinal"
        text = "commentary: discussion assistant"
        events, remainder = strategy.parse(text)
        # "assistant" at end should be held back
        self.assertIn("assistant", remainder)
```
**EN:** Test TextStrategy commentary channel with prefix that could be 'assistantfinal'. This test exercises `test_text_strategy_commentary_with_hold` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test TextStrategy commentary channel with prefix that could be 'assistantfinal'. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_text_strategy_commentary_with_hold`。

### Lines 1022-1023: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEvent`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestToken`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPrefixHold`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestIterTokens`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCanonicalStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTextStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHarmonyParser`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestIntegrationScenarios`: Integration tests for realistic Harmony parsing scenarios. / 用于组织相关测试、夹具或辅助方法。
- `TestEvent.test_init`: Test Event dataclass initialization. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。
- `TestToken.test_init`: Test Token dataclass initialization. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。
- `TestPrefixHold.test_empty_text`: Test prefix_hold with empty text. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_text`。
- `TestPrefixHold.test_no_matching_prefixes`: Test prefix_hold with no matching prefixes. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_matching_prefixes`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.parser.harmony_parser`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 1023
