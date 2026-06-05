# test_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/parser/test_reasoning_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates reasoning parser behavior in SGLang's unit / parser area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 解析器 领域中与 reasoning parser 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/parser/reasoning_parser.py"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-19: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.parser.reasoning_parser import (
    BaseReasoningFormatDetector,
    DeepSeekR1Detector,
    Gemma4Detector,
    Glm45Detector,
    HunyuanDetector,
    KimiDetector,
    KimiK2Detector,
    Nemotron3Detector,
    Qwen3Detector,
    ReasoningParser,
    StreamingParseResult,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.parser.reasoning_parser`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.parser.reasoning_parser`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 21-21: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestStreamingParseResult declaration / 类 TestStreamingParseResult 声明
```python
class TestStreamingParseResult(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 25-29: test case init default / 测试用例 init default
```python
    def test_init_default(self):
        """Test default initialization of StreamingParseResult."""
        result = StreamingParseResult()
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test default initialization of StreamingParseResult. This test exercises `test_init_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test default initialization of StreamingParseResult. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_default`。

### Lines 31-35: test case init with values / 测试用例 init with values
```python
    def test_init_with_values(self):
        """Test initialization with specific values."""
        result = StreamingParseResult("normal", "reasoning")
        self.assertEqual(result.normal_text, "normal")
        self.assertEqual(result.reasoning_text, "reasoning")
```
**EN:** Test initialization with specific values. This test exercises `test_init_with_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test initialization with specific values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_with_values`。

### Lines 38-38: class TestBaseReasoningFormatDetector declaration / 类 TestBaseReasoningFormatDetector 声明
```python
class TestBaseReasoningFormatDetector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 39-45: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = BaseReasoningFormatDetector(
            think_start_token="<think>",
            think_end_token="</think>",
            force_reasoning=False,
            stream_reasoning=True,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 47-54: test case init / 测试用例 init
```python
    def test_init(self):
        """Test initialization of BaseReasoningFormatDetector."""
        self.assertEqual(self.detector.think_start_token, "<think>")
        self.assertEqual(self.detector.think_end_token, "</think>")
        self.assertFalse(self.detector._in_reasoning)
        self.assertTrue(self.detector.stream_reasoning)
        self.assertEqual(self.detector._buffer, "")
        self.assertFalse(self.detector.stripped_think_start)
```
**EN:** Test initialization of BaseReasoningFormatDetector. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test initialization of BaseReasoningFormatDetector. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 56-61: test case detect and parse normal text / 测试用例 detect and parse normal text
```python
    def test_detect_and_parse_normal_text(self):
        """Test parsing normal text without reasoning."""
        text = "This is normal text"
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test parsing normal text without reasoning. This test exercises `test_detect_and_parse_normal_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing normal text without reasoning. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_normal_text`。

### Lines 63-68: test case detect and parse with start token / 测试用例 detect and parse with start token
```python
    def test_detect_and_parse_with_start_token(self):
        """Test parsing text starting with think token."""
        text = "<think>This is reasoning"
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "This is reasoning")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test parsing text starting with think token. This test exercises `test_detect_and_parse_with_start_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text starting with think token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_start_token`。

### Lines 70-75: test case detect and parse complete reasoning / 测试用例 detect and parse complete reasoning
```python
    def test_detect_and_parse_complete_reasoning(self):
        """Test parsing complete reasoning block."""
        text = "<think>This is reasoning</think>This is normal"
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "This is reasoning")
        self.assertEqual(result.normal_text, "This is normal")
```
**EN:** Test parsing complete reasoning block. This test exercises `test_detect_and_parse_complete_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing complete reasoning block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_complete_reasoning`。

### Lines 77-85: test case detect and parse force reasoning / 测试用例 detect and parse force reasoning
```python
    def test_detect_and_parse_force_reasoning(self):
        """Test forced reasoning mode."""
        detector = BaseReasoningFormatDetector(
            "<think>", "</think>", force_reasoning=True
        )
        text = "This should be reasoning"
        result = detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "This should be reasoning")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test forced reasoning mode. This test exercises `test_detect_and_parse_force_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test forced reasoning mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_force_reasoning`。

### Lines 87-91: test case parse streaming increment normal / 测试用例 parse streaming increment normal
```python
    def test_parse_streaming_increment_normal(self):
        """Test streaming parse of normal text."""
        result = self.detector.parse_streaming_increment("Hello world")
        self.assertEqual(result.normal_text, "Hello world")
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test streaming parse of normal text. This test exercises `test_parse_streaming_increment_normal` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse of normal text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_normal`。

### Lines 93-105: test case parse streaming increment partial token / 测试用例 parse streaming increment partial token
```python
    def test_parse_streaming_increment_partial_token(self):
        """Test streaming parse with partial token."""
        # Test partial start token
        result = self.detector.parse_streaming_increment("<thi")
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "")

        # Reset detector and test partial end token when in reasoning mode
        detector = BaseReasoningFormatDetector("<think>", "</think>")
        detector._in_reasoning = True
        result = detector.parse_streaming_increment("</thi")
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test streaming parse with partial token. This test exercises `test_parse_streaming_increment_partial_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse with partial token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_partial_token`。

### Lines 107-113: test case parse streaming increment complete start / 测试用例 parse streaming increment complete start
```python
    def test_parse_streaming_increment_complete_start(self):
        """Test streaming parse with complete start token."""
        result = self.detector.parse_streaming_increment("<think>")
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "")
        self.assertTrue(self.detector._in_reasoning)
        self.assertTrue(self.detector.stripped_think_start)
```
**EN:** Test streaming parse with complete start token. This test exercises `test_parse_streaming_increment_complete_start` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse with complete start token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_complete_start`。

### Lines 115-123: test case parse streaming increment reasoning content / 测试用例 parse streaming increment reasoning content
```python
    def test_parse_streaming_increment_reasoning_content(self):
        """Test streaming parse of reasoning content."""
        # First add start token
        self.detector.parse_streaming_increment("<think>")

        # Then add reasoning content
        result = self.detector.parse_streaming_increment("reasoning content")
        self.assertEqual(result.reasoning_text, "reasoning content")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test streaming parse of reasoning content. This test exercises `test_parse_streaming_increment_reasoning_content` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse of reasoning content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_reasoning_content`。

### Lines 125-135: test case parse streaming increment end token / 测试用例 parse streaming increment end token
```python
    def test_parse_streaming_increment_end_token(self):
        """Test streaming parse with end token."""
        # Start reasoning mode
        self.detector.parse_streaming_increment("<think>")
        self.detector.parse_streaming_increment("reasoning")

        # End reasoning - the reasoning content accumulated in previous calls is cleared when end token is found
        result = self.detector.parse_streaming_increment("</think>normal text")
        self.assertEqual(result.reasoning_text, "")  # Buffer cleared, returns empty
        self.assertEqual(result.normal_text, "normal text")
        self.assertFalse(self.detector._in_reasoning)
```
**EN:** Test streaming parse with end token. This test exercises `test_parse_streaming_increment_end_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse with end token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_end_token`。

### Lines 137-149: test case parse streaming increment no stream reasoning / 测试用例 parse streaming increment no stream reasoning
```python
    def test_parse_streaming_increment_no_stream_reasoning(self):
        """Test streaming parse without streaming reasoning."""
        detector = BaseReasoningFormatDetector(
            "<think>", "</think>", stream_reasoning=False
        )

        # Start reasoning mode
        detector.parse_streaming_increment("<think>")

        # Add reasoning content - should not return content
        result = detector.parse_streaming_increment("reasoning content")
        self.assertEqual(result.reasoning_text, "")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test streaming parse without streaming reasoning. This test exercises `test_parse_streaming_increment_no_stream_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse without streaming reasoning. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_no_stream_reasoning`。

### Lines 151-157: test case parse streaming increment mixed content / 测试用例 parse streaming increment mixed content
```python
    def test_parse_streaming_increment_mixed_content(self):
        """Test streaming parse with mixed content in one chunk."""
        result = self.detector.parse_streaming_increment(
            "<think>reasoning</think>normal"
        )
        self.assertEqual(result.reasoning_text, "reasoning")
        self.assertEqual(result.normal_text, "normal")
```
**EN:** Test streaming parse with mixed content in one chunk. This test exercises `test_parse_streaming_increment_mixed_content` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse with mixed content in one chunk. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_mixed_content`。

### Lines 160-160: class TestDeepSeekR1Detector declaration / 类 TestDeepSeekR1Detector 声明
```python
class TestDeepSeekR1Detector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 161-162: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = DeepSeekR1Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 164-169: test case init / 测试用例 init
```python
    def test_init(self):
        """Test DeepSeekR1Detector initialization."""
        self.assertEqual(self.detector.think_start_token, "<think>")
        self.assertEqual(self.detector.think_end_token, "</think>")
        self.assertTrue(self.detector._in_reasoning)  # force_reasoning=True
        self.assertTrue(self.detector.stream_reasoning)
```
**EN:** Test DeepSeekR1Detector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test DeepSeekR1Detector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 171-174: test case init no stream reasoning / 测试用例 init no stream reasoning
```python
    def test_init_no_stream_reasoning(self):
        """Test DeepSeekR1Detector with stream_reasoning=False."""
        detector = DeepSeekR1Detector(stream_reasoning=False)
        self.assertFalse(detector.stream_reasoning)
```
**EN:** Test DeepSeekR1Detector with stream_reasoning=False. This test exercises `test_init_no_stream_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test DeepSeekR1Detector with stream_reasoning=False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_no_stream_reasoning`。

### Lines 176-184: test case detect and parse r1 format / 测试用例 detect and parse r1 format
```python
    def test_detect_and_parse_r1_format(self):
        """Test parsing DeepSeek-R1 format."""
        text = "I need to think about this. The answer is 42."
        result = self.detector.detect_and_parse(text)
        # Should be treated as reasoning because force_reasoning=True
        self.assertEqual(
            result.reasoning_text, "I need to think about this. The answer is 42."
        )
        self.assertEqual(result.normal_text, "")
```
**EN:** Test parsing DeepSeek-R1 format. This test exercises `test_detect_and_parse_r1_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing DeepSeek-R1 format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_r1_format`。

### Lines 186-191: test case detect and parse with end token / 测试用例 detect and parse with end token
```python
    def test_detect_and_parse_with_end_token(self):
        """Test parsing with end token."""
        text = "I think this is the answer</think>The final answer is 42."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "I think this is the answer")
        self.assertEqual(result.normal_text, "The final answer is 42.")
```
**EN:** Test parsing with end token. This test exercises `test_detect_and_parse_with_end_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with end token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_end_token`。

### Lines 193-199: test case detect and parse with start token / 测试用例 detect and parse with start token
```python
    def test_detect_and_parse_with_start_token(self):
        """Test parsing deepseek-ai/DeepSeek-R1-0528 format, which generates the <think> token."""
        text = "<think>I need to think about this.</think>The answer is 42."
        result = self.detector.detect_and_parse(text)
        # Should be treated as reasoning because force_reasoning=True
        self.assertEqual(result.reasoning_text, "I need to think about this.")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing deepseek-ai/DeepSeek-R1-0528 format, which generates the <think> token. This test exercises `test_detect_and_parse_with_start_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing deepseek-ai/DeepSeek-R1-0528 format, which generates the <think> token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_start_token`。

### Lines 202-202: class TestQwen3Detector declaration / 类 TestQwen3Detector 声明
```python
class TestQwen3Detector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 203-204: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = Qwen3Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 206-211: test case init / 测试用例 init
```python
    def test_init(self):
        """Test Qwen3Detector initialization."""
        self.assertEqual(self.detector.think_start_token, "<think>")
        self.assertEqual(self.detector.think_end_token, "</think>")
        self.assertFalse(self.detector._in_reasoning)  # force_reasoning=False
        self.assertTrue(self.detector.stream_reasoning)
```
**EN:** Test Qwen3Detector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Qwen3Detector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 213-218: test case detect and parse qwen3 format / 测试用例 detect and parse qwen3 format
```python
    def test_detect_and_parse_qwen3_format(self):
        """Test parsing Qwen3 format."""
        text = "<think>Let me think about this problem</think>The answer is 42."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "Let me think about this problem")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing Qwen3 format. This test exercises `test_detect_and_parse_qwen3_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing Qwen3 format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_qwen3_format`。

### Lines 220-225: test case detect and parse without thinking / 测试用例 detect and parse without thinking
```python
    def test_detect_and_parse_without_thinking(self):
        """Test parsing without thinking (enable_thinking=False case)."""
        text = "Direct answer without thinking."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test parsing without thinking (enable_thinking=False case). This test exercises `test_detect_and_parse_without_thinking` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing without thinking (enable_thinking=False case). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_without_thinking`。

### Lines 228-228: class TestQwen3ForcedReasoningDetector declaration / 类 TestQwen3ForcedReasoningDetector 声明
```python
class TestQwen3ForcedReasoningDetector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 229-230: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = Qwen3Detector(force_reasoning=True)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 232-237: test case init / 测试用例 init
```python
    def test_init(self):
        """Test Qwen3ForcedReasoningDetector initialization."""
        self.assertEqual(self.detector.think_start_token, "<think>")
        self.assertEqual(self.detector.think_end_token, "</think>")
        self.assertTrue(self.detector._in_reasoning)  # force_reasoning=True
        self.assertTrue(self.detector.stream_reasoning)
```
**EN:** Test Qwen3ForcedReasoningDetector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Qwen3ForcedReasoningDetector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 239-246: test case detect and parse qwen3 forced reasoning format / 测试用例 detect and parse qwen3 forced reasoning format
```python
    def test_detect_and_parse_qwen3_forced_reasoning_format(self):
        """Test parsing Qwen3-ForcedReasoning format (no <think> start tag)."""
        text = "I need to think about this step by step.</think>The answer is 42."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(
            result.reasoning_text, "I need to think about this step by step."
        )
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing Qwen3-ForcedReasoning format (no <think> start tag). This test exercises `test_detect_and_parse_qwen3_forced_reasoning_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing Qwen3-ForcedReasoning format (no <think> start tag). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_qwen3_forced_reasoning_format`。

### Lines 248-254: test case detect and parse with start token / 测试用例 detect and parse with start token
```python
    def test_detect_and_parse_with_start_token(self):
        """Test parsing Qwen3-ForcedReasoning with optional <think> start tag."""
        text = "<think>I need to think about this.</think>The answer is 42."
        result = self.detector.detect_and_parse(text)
        # Should work because base class logic handles both force_reasoning=True OR start token
        self.assertEqual(result.reasoning_text, "I need to think about this.")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing Qwen3-ForcedReasoning with optional <think> start tag. This test exercises `test_detect_and_parse_with_start_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing Qwen3-ForcedReasoning with optional <think> start tag. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_start_token`。

### Lines 256-271: test case streaming qwen3 forced reasoning format / 测试用例 streaming qwen3 forced reasoning format
```python
    def test_streaming_qwen3_forced_reasoning_format(self):
        """Test streaming parse of Qwen3-ForcedReasoning format."""
        # First chunk without <think> start
        result = self.detector.parse_streaming_increment("I need to")
        self.assertEqual(result.reasoning_text, "I need to")
        self.assertEqual(result.normal_text, "")

        # More reasoning content
        result = self.detector.parse_streaming_increment(" think about this.")
        self.assertEqual(result.reasoning_text, " think about this.")
        self.assertEqual(result.normal_text, "")

        # End token with normal text
        result = self.detector.parse_streaming_increment("</think>The answer is 42.")
        self.assertEqual(result.reasoning_text, "")  # Buffer cleared
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test streaming parse of Qwen3-ForcedReasoning format. This test exercises `test_streaming_qwen3_forced_reasoning_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse of Qwen3-ForcedReasoning format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_qwen3_forced_reasoning_format`。

### Lines 274-274: class TestKimiDetector declaration / 类 TestKimiDetector 声明
```python
class TestKimiDetector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 275-276: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = KimiDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 278-283: test case init / 测试用例 init
```python
    def test_init(self):
        """Test KimiDetector initialization."""
        self.assertEqual(self.detector.think_start_token, "◁think▷")
        self.assertEqual(self.detector.think_end_token, "◁/think▷")
        self.assertFalse(self.detector._in_reasoning)
        self.assertTrue(self.detector.stream_reasoning)
```
**EN:** Test KimiDetector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test KimiDetector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 285-290: test case detect and parse kimi format / 测试用例 detect and parse kimi format
```python
    def test_detect_and_parse_kimi_format(self):
        """Test parsing Kimi format."""
        text = "◁think▷Let me consider this carefully◁/think▷The answer is 42."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "Let me consider this carefully")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing Kimi format. This test exercises `test_detect_and_parse_kimi_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing Kimi format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_kimi_format`。

### Lines 292-297: test case detect and parse kimi no thinking / 测试用例 detect and parse kimi no thinking
```python
    def test_detect_and_parse_kimi_no_thinking(self):
        """Test parsing Kimi format without thinking."""
        text = "Direct answer without thinking tokens."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test parsing Kimi format without thinking. This test exercises `test_detect_and_parse_kimi_no_thinking` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing Kimi format without thinking. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_kimi_no_thinking`。

### Lines 299-320: test case streaming kimi format / 测试用例 streaming kimi format
```python
    def test_streaming_kimi_format(self):
        """Test streaming parse of Kimi format."""
        # Test partial token
        result = self.detector.parse_streaming_increment("◁thi")
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "")

        # Complete start token
        result = self.detector.parse_streaming_increment("nk▷Start")
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "Start")
        self.assertTrue(self.detector._in_reasoning)

        # Add reasoning content
        result = self.detector.parse_streaming_increment("thinking...")
        self.assertEqual(result.reasoning_text, "thinking...")
        self.assertEqual(result.normal_text, "")

        # End token - reasoning content is cleared when end token is processed
        result = self.detector.parse_streaming_increment("◁/think▷answer")
        self.assertEqual(result.reasoning_text, "")  # Buffer cleared
        self.assertEqual(result.normal_text, "answer")
```
**EN:** Test streaming parse of Kimi format. This test exercises `test_streaming_kimi_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse of Kimi format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_kimi_format`。

### Lines 323-323: class TestKimiK2Detector declaration / 类 TestKimiK2Detector 声明
```python
class TestKimiK2Detector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 324-324: supporting statements / 辅助语句
```python
    """Test cases for KimiK2 detector with tool interruption support."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 326-327: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = KimiK2Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 329-335: test case init / 测试用例 init
```python
    def test_init(self):
        """Test KimiK2Detector initialization."""
        self.assertEqual(self.detector.think_start_token, "<think>")
        self.assertEqual(self.detector.think_end_token, "</think>")
        self.assertEqual(self.detector.tool_start_token, "<|tool_calls_section_begin|>")
        self.assertFalse(self.detector._in_reasoning)
        self.assertTrue(self.detector.stream_reasoning)
```
**EN:** Test KimiK2Detector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test KimiK2Detector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 337-344: test case detect and parse tool interrupt / 测试用例 detect and parse tool interrupt
```python
    def test_detect_and_parse_tool_interrupt(self):
        """Test parsing with Kimi-K2 tool-section interruption."""
        text = "<think>thinking<|tool_calls_section_begin|><|tool_call_begin|>"
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "thinking")
        self.assertEqual(
            result.normal_text, "<|tool_calls_section_begin|><|tool_call_begin|>"
        )
```
**EN:** Test parsing with Kimi-K2 tool-section interruption. This test exercises `test_detect_and_parse_tool_interrupt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with Kimi-K2 tool-section interruption. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_tool_interrupt`。

### Lines 346-357: test case streaming tool interrupt / 测试用例 streaming tool interrupt
```python
    def test_streaming_tool_interrupt(self):
        """Test streaming parse interrupted by tool section."""
        self.detector.parse_streaming_increment("<think>")
        result1 = self.detector.parse_streaming_increment("reasoning")
        self.assertEqual(result1.reasoning_text, "reasoning")
        self.assertEqual(result1.normal_text, "")

        result2 = self.detector.parse_streaming_increment(
            "<|tool_calls_section_begin|>"
        )
        self.assertEqual(result2.reasoning_text, "")
        self.assertEqual(result2.normal_text, "<|tool_calls_section_begin|>")
```
**EN:** Test streaming parse interrupted by tool section. This test exercises `test_streaming_tool_interrupt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse interrupted by tool section. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_interrupt`。

### Lines 359-365: test case streaming after interrupt is normal / 测试用例 streaming after interrupt is normal
```python
    def test_streaming_after_interrupt_is_normal(self):
        """After interruption, subsequent chunks should be normal text."""
        self.detector.parse_streaming_increment("<think>")
        self.detector.parse_streaming_increment("reasoning<|tool_calls_section_begin|>")
        result = self.detector.parse_streaming_increment("<|tool_call_begin|>")
        self.assertEqual(result.reasoning_text, "")
        self.assertEqual(result.normal_text, "<|tool_call_begin|>")
```
**EN:** After interruption, subsequent chunks should be normal text. This test exercises `test_streaming_after_interrupt_is_normal` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** After interruption, subsequent chunks should be normal text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_after_interrupt_is_normal`。

### Lines 368-368: class TestGlm45Detector declaration / 类 TestGlm45Detector 声明
```python
class TestGlm45Detector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 369-369: supporting statements / 辅助语句
```python
    """Test cases for GLM45 detector with tool interruption support."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 371-372: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = Glm45Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 374-380: test case init / 测试用例 init
```python
    def test_init(self):
        """Test Glm45Detector initialization."""
        self.assertEqual(self.detector.think_start_token, "<think>")
        self.assertEqual(self.detector.think_end_token, "</think>")
        self.assertEqual(self.detector.tool_start_token, "<tool_call>")
        self.assertFalse(self.detector._in_reasoning)
        self.assertTrue(self.detector.stream_reasoning)
```
**EN:** Test Glm45Detector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Glm45Detector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 382-387: test case detect and parse normal reasoning / 测试用例 detect and parse normal reasoning
```python
    def test_detect_and_parse_normal_reasoning(self):
        """Test parsing normal reasoning block without tool interruption."""
        text = "<think>Let me think about this step by step</think>The answer is 42."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "Let me think about this step by step")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing normal reasoning block without tool interruption. This test exercises `test_detect_and_parse_normal_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing normal reasoning block without tool interruption. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_normal_reasoning`。

### Lines 389-399: test case detect and parse tool interrupt / 测试用例 detect and parse tool interrupt
```python
    def test_detect_and_parse_tool_interrupt(self):
        """
        Test parsing with tool interruption.

        GLM45 can interrupt reasoning with tool token (<tool_call>) without closing </think>.
        Should split at the first occurrence of tool_start_token using find().
        """
        text = "<think>I need to think<tool_call>tool call data"
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "I need to think")
        self.assertEqual(result.normal_text, "<tool_call>tool call data")
```
**EN:** Test parsing with tool interruption. This test exercises `test_detect_and_parse_tool_interrupt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with tool interruption. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_tool_interrupt`。

### Lines 401-414: test case detect and parse multiple tool calls find / 测试用例 detect and parse multiple tool calls find
```python
    def test_detect_and_parse_multiple_tool_calls_find(self):
        """
        Test that find() finds the FIRST occurrence of tool_start_token.

        If multiple tool calls exist in buffer, should split at the first one.
        """
        text = "<think>thinking<tool_call>first tool<tool_call>second tool<tool_call>final tool"
        result = self.detector.detect_and_parse(text)
        # Should split at the first <tool_call>
        self.assertEqual(result.reasoning_text, "thinking")
        self.assertEqual(
            result.normal_text,
            "<tool_call>first tool<tool_call>second tool<tool_call>final tool",
        )
```
**EN:** Test that find() finds the FIRST occurrence of tool_start_token. This test exercises `test_detect_and_parse_multiple_tool_calls_find` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that find() finds the FIRST occurrence of tool_start_token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_multiple_tool_calls_find`。

### Lines 416-425: test case detect and parse truncated reasoning / 测试用例 detect and parse truncated reasoning
```python
    def test_detect_and_parse_truncated_reasoning(self):
        """
        Test truncated reasoning without tool or end tag.

        Should return all content as reasoning_text.
        """
        text = "<think>This is incomplete"
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "This is incomplete")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test truncated reasoning without tool or end tag. This test exercises `test_detect_and_parse_truncated_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test truncated reasoning without tool or end tag. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_truncated_reasoning`。

### Lines 427-432: test case detect and parse normal text only / 测试用例 detect and parse normal text only
```python
    def test_detect_and_parse_normal_text_only(self):
        """Test parsing text without reasoning block."""
        text = "Just the answer without any reasoning."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test parsing text without reasoning block. This test exercises `test_detect_and_parse_normal_text_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text without reasoning block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_normal_text_only`。

### Lines 434-451: test case streaming normal flow / 测试用例 streaming normal flow
```python
    def test_streaming_normal_flow(self):
        """Test streaming with normal reasoning flow."""
        # Start reasoning
        result1 = self.detector.parse_streaming_increment("<think>")
        self.assertEqual(result1.normal_text, "")
        self.assertEqual(result1.reasoning_text, "")
        self.assertTrue(self.detector._in_reasoning)

        # Reasoning content
        result2 = self.detector.parse_streaming_increment("thinking...")
        self.assertEqual(result2.normal_text, "")
        self.assertEqual(result2.reasoning_text, "thinking...")

        # End reasoning
        result3 = self.detector.parse_streaming_increment("</think>answer")
        self.assertEqual(result3.normal_text, "answer")
        self.assertEqual(result3.reasoning_text, "")
        self.assertFalse(self.detector._in_reasoning)
```
**EN:** Test streaming with normal reasoning flow. This test exercises `test_streaming_normal_flow` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with normal reasoning flow. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_normal_flow`。

### Lines 453-476: test case streaming tool interrupt split tokens / 测试用例 streaming tool interrupt split tokens
```python
    def test_streaming_tool_interrupt_split_tokens(self):
        """
        Test streaming with tool interruption where tool token is split across chunks.

        This tests the buffer prefix logic that prevents partial emission of tool token.
        """
        # Start reasoning
        self.detector.parse_streaming_increment("<think>")

        # Add reasoning
        result1 = self.detector.parse_streaming_increment("thinking")
        self.assertEqual(result1.reasoning_text, "thinking")

        # Send partial tool token (should be buffered, not emitted)
        result2 = self.detector.parse_streaming_increment("<tool_call>")
        # Tool token is in buffer, causing switch to normal mode
        self.assertEqual(result2.reasoning_text, "")
        self.assertEqual(result2.normal_text, "<tool_call>")
        self.assertFalse(self.detector._in_reasoning)

        # Send tool args
        result3 = self.detector.parse_streaming_increment("tool args")
        self.assertEqual(result3.reasoning_text, "")
        self.assertEqual(result3.normal_text, "tool args")
```
**EN:** Test streaming with tool interruption where tool token is split across chunks. This test exercises `test_streaming_tool_interrupt_split_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with tool interruption where tool token is split across chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_interrupt_split_tokens`。

### Lines 478-497: test case streaming no stream reasoning / 测试用例 streaming no stream reasoning
```python
    def test_streaming_no_stream_reasoning(self):
        """Test streaming without stream_reasoning enabled."""
        detector = Glm45Detector(stream_reasoning=False)

        # Start reasoning
        detector.parse_streaming_increment("<think>")

        # Reasoning content is buffered and not returned yet
        result = detector.parse_streaming_increment("thinking")
        self.assertEqual(result.reasoning_text, "")
        self.assertEqual(result.normal_text, "")

        # Tool interruption should still work - flushes buffered reasoning.
        # Note: when stream_reasoning=False, the <think> tag is stripped from the
        # local `current_text` variable but NOT from `self._buffer` (which is never
        # cleared in the non-streaming path). So the flushed reasoning content
        # includes the raw <think> tag.
        result = detector.parse_streaming_increment("<tool_call>tool call")
        self.assertEqual(result.reasoning_text, "<think>thinking")
        self.assertEqual(result.normal_text, "<tool_call>tool call")
```
**EN:** Test streaming without stream_reasoning enabled. This test exercises `test_streaming_no_stream_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming without stream_reasoning enabled. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_no_stream_reasoning`。

### Lines 499-504: test case streaming empty reasoning with tool / 测试用例 streaming empty reasoning with tool
```python
    def test_streaming_empty_reasoning_with_tool(self):
        """Test empty reasoning block followed by tool call."""
        result1 = self.detector.parse_streaming_increment("<think>")
        result2 = self.detector.parse_streaming_increment("<tool_call>tool call")
        self.assertEqual(result2.reasoning_text, "")
        self.assertEqual(result2.normal_text, "<tool_call>tool call")
```
**EN:** Test empty reasoning block followed by tool call. This test exercises `test_streaming_empty_reasoning_with_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test empty reasoning block followed by tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_empty_reasoning_with_tool`。

### Lines 506-520: test case forced reasoning mode / 测试用例 forced reasoning mode
```python
    def test_forced_reasoning_mode(self):
        """Test GLM45 with force_reasoning=True."""
        detector = Glm45Detector(force_reasoning=True)

        # Without start token, should still be in reasoning mode
        text = "This is reasoning"
        result = detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "This is reasoning")
        self.assertEqual(result.normal_text, "")

        # Tool interruption should work with forced reasoning
        text = "More reasoning<tool_call>tool call"
        result = detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "More reasoning")
        self.assertEqual(result.normal_text, "<tool_call>tool call")
```
**EN:** Test GLM45 with force_reasoning=True. This test exercises `test_forced_reasoning_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test GLM45 with force_reasoning=True. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_forced_reasoning_mode`。

### Lines 523-523: class TestHunyuanDetector declaration / 类 TestHunyuanDetector 声明
```python
class TestHunyuanDetector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 524-524: supporting statements / 辅助语句
```python
    """Test cases for Hunyuan detector with tool interruption support."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 526-527: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = HunyuanDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 529-535: test case init / 测试用例 init
```python
    def test_init(self):
        """Test HunyuanDetector initialization."""
        self.assertEqual(self.detector.think_start_token, "<think>")
        self.assertEqual(self.detector.think_end_token, "</think>")
        self.assertEqual(self.detector.tool_start_token, "<tool_calls>")
        self.assertFalse(self.detector._in_reasoning)
        self.assertTrue(self.detector.stream_reasoning)
```
**EN:** Test HunyuanDetector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test HunyuanDetector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 537-542: test case detect and parse normal reasoning / 测试用例 detect and parse normal reasoning
```python
    def test_detect_and_parse_normal_reasoning(self):
        """Test parsing normal reasoning block without tool interruption."""
        text = "<think>Let me think about this</think>The answer is 42."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "Let me think about this")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing normal reasoning block without tool interruption. This test exercises `test_detect_and_parse_normal_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing normal reasoning block without tool interruption. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_normal_reasoning`。

### Lines 544-549: test case detect and parse without thinking / 测试用例 detect and parse without thinking
```python
    def test_detect_and_parse_without_thinking(self):
        """Test parsing without thinking tokens (no_think mode)."""
        text = "Direct answer without thinking."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test parsing without thinking tokens (no_think mode). This test exercises `test_detect_and_parse_without_thinking` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing without thinking tokens (no_think mode). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_without_thinking`。

### Lines 551-556: test case detect and parse tool interrupt / 测试用例 detect and parse tool interrupt
```python
    def test_detect_and_parse_tool_interrupt(self):
        """Test parsing with tool call interruption during reasoning."""
        text = "<think>I need to check<tool_calls><tool_call>get_weather<tool_sep></tool_call></tool_calls>"
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "I need to check")
        self.assertIn("<tool_calls>", result.normal_text)
```
**EN:** Test parsing with tool call interruption during reasoning. This test exercises `test_detect_and_parse_tool_interrupt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with tool call interruption during reasoning. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_tool_interrupt`。

### Lines 558-566: test case streaming normal reasoning / 测试用例 streaming normal reasoning
```python
    def test_streaming_normal_reasoning(self):
        """Test streaming parse of normal reasoning block."""
        self.detector.parse_streaming_increment("<think>")
        result1 = self.detector.parse_streaming_increment("reasoning content")
        self.assertEqual(result1.reasoning_text, "reasoning content")

        result2 = self.detector.parse_streaming_increment("</think>answer")
        self.assertEqual(result2.normal_text, "answer")
        self.assertFalse(self.detector._in_reasoning)
```
**EN:** Test streaming parse of normal reasoning block. This test exercises `test_streaming_normal_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse of normal reasoning block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_normal_reasoning`。

### Lines 568-577: test case streaming tool interrupt / 测试用例 streaming tool interrupt
```python
    def test_streaming_tool_interrupt(self):
        """Test streaming parse interrupted by tool call section."""
        self.detector.parse_streaming_increment("<think>")
        result1 = self.detector.parse_streaming_increment("thinking")
        self.assertEqual(result1.reasoning_text, "thinking")

        result2 = self.detector.parse_streaming_increment("<tool_calls>")
        self.assertEqual(result2.reasoning_text, "")
        self.assertEqual(result2.normal_text, "<tool_calls>")
        self.assertFalse(self.detector._in_reasoning)
```
**EN:** Test streaming parse interrupted by tool call section. This test exercises `test_streaming_tool_interrupt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse interrupted by tool call section. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_interrupt`。

### Lines 579-585: test case streaming after interrupt is normal / 测试用例 streaming after interrupt is normal
```python
    def test_streaming_after_interrupt_is_normal(self):
        """After tool interruption, subsequent chunks should be normal text."""
        self.detector.parse_streaming_increment("<think>")
        self.detector.parse_streaming_increment("reasoning<tool_calls>")
        result = self.detector.parse_streaming_increment("<tool_call>data")
        self.assertEqual(result.reasoning_text, "")
        self.assertEqual(result.normal_text, "<tool_call>data")
```
**EN:** After tool interruption, subsequent chunks should be normal text. This test exercises `test_streaming_after_interrupt_is_normal` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** After tool interruption, subsequent chunks should be normal text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_after_interrupt_is_normal`。

### Lines 587-597: test case reasoning parser integration / 测试用例 reasoning parser integration
```python
    def test_reasoning_parser_integration(self):
        """Test Hunyuan through ReasoningParser API."""
        parser = ReasoningParser("hunyuan")
        self.assertIsInstance(parser.detector, HunyuanDetector)

        # Non-streaming
        reasoning, normal = parser.parse_non_stream(
            "<think>thinking<tool_calls><tool_call>func<tool_sep></tool_call></tool_calls>"
        )
        self.assertEqual(reasoning, "thinking")
        self.assertIn("<tool_calls>", normal)
```
**EN:** Test Hunyuan through ReasoningParser API. This test exercises `test_reasoning_parser_integration` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Hunyuan through ReasoningParser API. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoning_parser_integration`。

### Lines 599-613: test case reasoning parser streaming / 测试用例 reasoning parser streaming
```python
    def test_reasoning_parser_streaming(self):
        """Test Hunyuan streaming through ReasoningParser API."""
        parser = ReasoningParser("hunyuan")
        chunks = ["<think>", "reasoning", "<tool_calls>", "<tool_call>func"]
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            reasoning, normal = parser.parse_stream_chunk(chunk)
            if reasoning:
                all_reasoning += reasoning
            if normal:
                all_normal += normal

        self.assertEqual(all_reasoning, "reasoning")
        self.assertIn("<tool_calls>", all_normal)
```
**EN:** Test Hunyuan streaming through ReasoningParser API. This test exercises `test_reasoning_parser_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Hunyuan streaming through ReasoningParser API. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoning_parser_streaming`。

### Lines 616-616: class TestNemotron3Detector declaration / 类 TestNemotron3Detector 声明
```python
class TestNemotron3Detector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 617-618: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = Nemotron3Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 620-626: test case init / 测试用例 init
```python
    def test_init(self):
        """Test Nemotron3Detector initialization."""
        self.assertEqual(self.detector.think_start_token, "<think>")
        self.assertEqual(self.detector.think_end_token, "</think>")
        self.assertFalse(self.detector._in_reasoning)
        self.assertTrue(self.detector.stream_reasoning)
        self.assertFalse(self.detector._force_nonempty_content)
```
**EN:** Test Nemotron3Detector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Nemotron3Detector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 628-633: test case detect and parse complete reasoning / 测试用例 detect and parse complete reasoning
```python
    def test_detect_and_parse_complete_reasoning(self):
        """Test parsing complete reasoning block."""
        text = "<think>Let me think about this</think>The answer is 42."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "Let me think about this")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing complete reasoning block. This test exercises `test_detect_and_parse_complete_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing complete reasoning block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_complete_reasoning`。

### Lines 635-640: test case detect and parse no thinking / 测试用例 detect and parse no thinking
```python
    def test_detect_and_parse_no_thinking(self):
        """Test parsing without thinking tokens."""
        text = "Direct answer without thinking."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test parsing without thinking tokens. This test exercises `test_detect_and_parse_no_thinking` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing without thinking tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_no_thinking`。

### Lines 642-647: test case detect and parse reasoning only / 测试用例 detect and parse reasoning only
```python
    def test_detect_and_parse_reasoning_only(self):
        """Test parsing when output is all reasoning (no content after </think>)."""
        text = "<think>All reasoning, no answer</think>"
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "All reasoning, no answer")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test parsing when output is all reasoning (no content after </think>). This test exercises `test_detect_and_parse_reasoning_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing when output is all reasoning (no content after </think>). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_reasoning_only`。

### Lines 649-655: test case force nonempty content swaps when no normal text / 测试用例 force nonempty content swaps when no normal text
```python
    def test_force_nonempty_content_swaps_when_no_normal_text(self):
        """Test force_nonempty_content swaps reasoning to content when content is empty."""
        detector = Nemotron3Detector(force_nonempty_content=True)
        text = "<think>All reasoning, no answer</think>"
        result = detector.detect_and_parse(text)
        self.assertEqual(result.normal_text, "All reasoning, no answer")
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test force_nonempty_content swaps reasoning to content when content is empty. This test exercises `test_force_nonempty_content_swaps_when_no_normal_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test force_nonempty_content swaps reasoning to content when content is empty. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_force_nonempty_content_swaps_when_no_normal_text`。

### Lines 657-663: test case force nonempty content no swap when normal text exists / 测试用例 force nonempty content no swap when normal text exists
```python
    def test_force_nonempty_content_no_swap_when_normal_text_exists(self):
        """Test force_nonempty_content does not swap when content already exists."""
        detector = Nemotron3Detector(force_nonempty_content=True)
        text = "<think>Reasoning here</think>The answer is 42."
        result = detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "Reasoning here")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test force_nonempty_content does not swap when content already exists. This test exercises `test_force_nonempty_content_no_swap_when_normal_text_exists` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test force_nonempty_content does not swap when content already exists. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_force_nonempty_content_no_swap_when_normal_text_exists`。

### Lines 665-672: test case force nonempty content truncated reasoning / 测试用例 force nonempty content truncated reasoning
```python
    def test_force_nonempty_content_truncated_reasoning(self):
        """Test force_nonempty_content with truncated reasoning (no end token)."""
        detector = Nemotron3Detector(force_nonempty_content=True)
        text = "<think>Truncated reasoning without end token"
        result = detector.detect_and_parse(text)
        # Truncated reasoning has no normal_text, so swap should occur
        self.assertEqual(result.normal_text, "Truncated reasoning without end token")
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test force_nonempty_content with truncated reasoning (no end token). This test exercises `test_force_nonempty_content_truncated_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test force_nonempty_content with truncated reasoning (no end token). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_force_nonempty_content_truncated_reasoning`。

### Lines 674-681: test case force nonempty content no thinking tokens / 测试用例 force nonempty content no thinking tokens
```python
    def test_force_nonempty_content_no_thinking_tokens(self):
        """Test force_nonempty_content with plain text (no thinking tokens)."""
        detector = Nemotron3Detector(force_nonempty_content=True)
        text = "Plain text without any thinking."
        result = detector.detect_and_parse(text)
        # Normal text already exists, no swap needed
        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test force_nonempty_content with plain text (no thinking tokens). This test exercises `test_force_nonempty_content_no_thinking_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test force_nonempty_content with plain text (no thinking tokens). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_force_nonempty_content_no_thinking_tokens`。

### Lines 684-684: class TestGemma4Detector declaration / 类 TestGemma4Detector 声明
```python
class TestGemma4Detector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 685-686: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = Gemma4Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 688-694: test case init / 测试用例 init
```python
    def test_init(self):
        """Test Gemma4Detector initialization."""
        self.assertEqual(self.detector.think_start_token, "<|channel>")
        self.assertEqual(self.detector.think_end_token, "<channel|>")
        self.assertEqual(self.detector.think_start_self_label, "thought\n")
        self.assertFalse(self.detector._in_reasoning)
        self.assertTrue(self.detector.stream_reasoning)
```
**EN:** Test Gemma4Detector initialization. This test exercises `test_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Gemma4Detector initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

### Lines 696-701: test case detect and parse complete reasoning / 测试用例 detect and parse complete reasoning
```python
    def test_detect_and_parse_complete_reasoning(self):
        """Test parsing complete Gemma4 reasoning block (think_start_self_label is stripped)."""
        text = "<|channel>thought\nLet me think about this<channel|>The answer is 42."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "Let me think about this")
        self.assertEqual(result.normal_text, "The answer is 42.")
```
**EN:** Test parsing complete Gemma4 reasoning block (think_start_self_label is stripped). This test exercises `test_detect_and_parse_complete_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing complete Gemma4 reasoning block (think_start_self_label is stripped). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_complete_reasoning`。

### Lines 703-708: test case detect and parse without thinking / 测试用例 detect and parse without thinking
```python
    def test_detect_and_parse_without_thinking(self):
        """Test parsing without thinking (enable_thinking=False case)."""
        text = "Direct answer without thinking."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test parsing without thinking (enable_thinking=False case). This test exercises `test_detect_and_parse_without_thinking` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing without thinking (enable_thinking=False case). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_without_thinking`。

### Lines 710-715: test case detect and parse reasoning only / 测试用例 detect and parse reasoning only
```python
    def test_detect_and_parse_reasoning_only(self):
        """Test parsing when output is all reasoning (no end token yet)."""
        text = "<|channel>thought\nStill thinking..."
        result = self.detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "Still thinking...")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test parsing when output is all reasoning (no end token yet). This test exercises `test_detect_and_parse_reasoning_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing when output is all reasoning (no end token yet). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_reasoning_only`。

### Lines 717-732: test case streaming complete flow / 测试用例 streaming complete flow
```python
    def test_streaming_complete_flow(self):
        """Test streaming parse of Gemma4 reasoning flow."""
        chunks = [
            "<|channel>",
            "thought\nreasoning content",
            "<channel|>",
            "final answer",
        ]
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk)
            all_reasoning += result.reasoning_text
            all_normal += result.normal_text
        self.assertIn("reasoning content", all_reasoning)
        self.assertIn("final answer", all_normal)
```
**EN:** Test streaming parse of Gemma4 reasoning flow. This test exercises `test_streaming_complete_flow` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse of Gemma4 reasoning flow. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_complete_flow`。

### Lines 734-744: test case streaming full start sequence / 测试用例 streaming full start sequence
```python
    def test_streaming_full_start_sequence(self):
        """Test streaming with the full start sequence (token + self_label)."""
        # Gemma4 start sequence is "<|channel>thought\n", not just "<|channel>"
        result = self.detector.parse_streaming_increment("<|channel>thought\n")
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "")
        self.assertTrue(self.detector._in_reasoning)

        result = self.detector.parse_streaming_increment("reasoning content")
        self.assertEqual(result.reasoning_text, "reasoning content")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test streaming with the full start sequence (token + self_label). This test exercises `test_streaming_full_start_sequence` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with the full start sequence (token + self_label). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_full_start_sequence`。

### Lines 746-751: test case streaming partial start buffered / 测试用例 streaming partial start buffered
```python
    def test_streaming_partial_start_buffered(self):
        """Test that partial start sequence is buffered."""
        # "<|channel>" alone is a prefix of "<|channel>thought\n", so it's buffered
        result = self.detector.parse_streaming_increment("<|channel>")
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test that partial start sequence is buffered. This test exercises `test_streaming_partial_start_buffered` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that partial start sequence is buffered. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_partial_start_buffered`。

### Lines 753-761: test case streaming end token mid chunk / 测试用例 streaming end token mid chunk
```python
    def test_streaming_end_token_mid_chunk(self):
        """Test end token arriving in the same chunk as reasoning content."""
        self.detector.parse_streaming_increment("<|channel>thought\n")
        result = self.detector.parse_streaming_increment(
            "some reasoning<channel|>the answer"
        )
        self.assertEqual(result.reasoning_text, "some reasoning")
        self.assertEqual(result.normal_text, "the answer")
        self.assertFalse(self.detector._in_reasoning)
```
**EN:** Test end token arriving in the same chunk as reasoning content. This test exercises `test_streaming_end_token_mid_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test end token arriving in the same chunk as reasoning content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_end_token_mid_chunk`。

### Lines 763-773: test case streaming split end token / 测试用例 streaming split end token
```python
    def test_streaming_split_end_token(self):
        """Test end token split across two chunks."""
        self.detector.parse_streaming_increment("<|channel>thought\n")
        self.detector.parse_streaming_increment("reasoning content")

        result1 = self.detector.parse_streaming_increment("<chan")
        self.assertEqual(result1.normal_text, "")

        result2 = self.detector.parse_streaming_increment("nel|>final answer")
        self.assertFalse(self.detector._in_reasoning)
        self.assertIn("final answer", result2.normal_text)
```
**EN:** Test end token split across two chunks. This test exercises `test_streaming_split_end_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test end token split across two chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_split_end_token`。

### Lines 775-785: test case streaming self label split across chunks / 测试用例 streaming self label split across chunks
```python
    def test_streaming_self_label_split_across_chunks(self):
        """Test self_label ('thought\\n') arriving separately from start token."""
        result1 = self.detector.parse_streaming_increment("<|channel>")
        self.assertEqual(result1.reasoning_text, "")
        self.assertEqual(result1.normal_text, "")

        result2 = self.detector.parse_streaming_increment("thought\n")
        self.assertTrue(self.detector._in_reasoning)

        result3 = self.detector.parse_streaming_increment("reasoning here")
        self.assertEqual(result3.reasoning_text, "reasoning here")
```
**EN:** Test self_label ('thought\n') arriving separately from start token. This test exercises `test_streaming_self_label_split_across_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test self_label ('thought\n') arriving separately from start token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_self_label_split_across_chunks`。

### Lines 787-797: test case streaming force reasoning / 测试用例 streaming force reasoning
```python
    def test_streaming_force_reasoning(self):
        """Test streaming with force_reasoning=True (no start token needed)."""
        detector = Gemma4Detector(force_reasoning=True)

        result1 = detector.parse_streaming_increment("reasoning content")
        self.assertEqual(result1.reasoning_text, "reasoning content")
        self.assertEqual(result1.normal_text, "")

        result2 = detector.parse_streaming_increment("<channel|>the answer")
        self.assertFalse(detector._in_reasoning)
        self.assertIn("the answer", result2.normal_text)
```
**EN:** Test streaming with force_reasoning=True (no start token needed). This test exercises `test_streaming_force_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with force_reasoning=True (no start token needed). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_force_reasoning`。

### Lines 799-808: test case streaming multiple reasoning chunks / 测试用例 streaming multiple reasoning chunks
```python
    def test_streaming_multiple_reasoning_chunks(self):
        """Test reasoning content arriving in many small chunks."""
        self.detector.parse_streaming_increment("<|channel>thought\n")

        all_reasoning = ""
        for chunk in ["Think", "ing ", "step ", "by ", "step."]:
            result = self.detector.parse_streaming_increment(chunk)
            all_reasoning += result.reasoning_text
            self.assertEqual(result.normal_text, "")
        self.assertEqual(all_reasoning, "Thinking step by step.")
```
**EN:** Test reasoning content arriving in many small chunks. This test exercises `test_streaming_multiple_reasoning_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test reasoning content arriving in many small chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_reasoning_chunks`。

### Lines 810-816: test case force reasoning / 测试用例 force reasoning
```python
    def test_force_reasoning(self):
        """Test Gemma4Detector with force_reasoning=True."""
        detector = Gemma4Detector(force_reasoning=True)
        text = "This should be reasoning<channel|>The answer."
        result = detector.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "This should be reasoning")
        self.assertEqual(result.normal_text, "The answer.")
```
**EN:** Test Gemma4Detector with force_reasoning=True. This test exercises `test_force_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Gemma4Detector with force_reasoning=True. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_force_reasoning`。

### Lines 819-819: class TestReasoningParser declaration / 类 TestReasoningParser 声明
```python
class TestReasoningParser(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 820-841: test case init valid model / 测试用例 init valid model
```python
    def test_init_valid_model(self):
        """Test initialization with valid model types."""
        parser = ReasoningParser("deepseek-r1")
        self.assertIsInstance(parser.detector, DeepSeekR1Detector)

        parser = ReasoningParser("qwen3")
        self.assertIsInstance(parser.detector, Qwen3Detector)

        parser = ReasoningParser("kimi")
        self.assertIsInstance(parser.detector, KimiDetector)

        parser = ReasoningParser("kimi_k2")
        self.assertIsInstance(parser.detector, KimiK2Detector)

        parser = ReasoningParser("glm45")
        self.assertIsInstance(parser.detector, Glm45Detector)

        parser = ReasoningParser("hunyuan")
        self.assertIsInstance(parser.detector, HunyuanDetector)

        parser = ReasoningParser("gemma4")
        self.assertIsInstance(parser.detector, Gemma4Detector)
```
**EN:** Test initialization with valid model types. This test exercises `test_init_valid_model` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test initialization with valid model types. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_valid_model`。

### Lines 843-847: test case init invalid model / 测试用例 init invalid model
```python
    def test_init_invalid_model(self):
        """Test initialization with invalid model type."""
        with self.assertRaises(ValueError) as context:
            ReasoningParser("invalid-model")
        self.assertIn("Unsupported model type", str(context.exception))
```
**EN:** Test initialization with invalid model type. This test exercises `test_init_invalid_model` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test initialization with invalid model type. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_invalid_model`。

### Lines 849-853: test case init no model / 测试用例 init no model
```python
    def test_init_no_model(self):
        """Test initialization without model type."""
        with self.assertRaises(ValueError) as context:
            ReasoningParser(None)
        self.assertEqual(str(context.exception), "Model type must be specified")
```
**EN:** Test initialization without model type. This test exercises `test_init_no_model` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test initialization without model type. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_no_model`。

### Lines 855-862: test case parse non stream / 测试用例 parse non stream
```python
    def test_parse_non_stream(self):
        """Test non-streaming parsing."""
        parser = ReasoningParser("qwen3")
        reasoning, normal = parser.parse_non_stream(
            "<think>Let me think</think>The answer is 42."
        )
        self.assertEqual(reasoning, "Let me think")
        self.assertEqual(normal, "The answer is 42.")
```
**EN:** Test non-streaming parsing. This test exercises `test_parse_non_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test non-streaming parsing. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_non_stream`。

### Lines 864-881: test case parse stream chunk / 测试用例 parse stream chunk
```python
    def test_parse_stream_chunk(self):
        """Test streaming chunk parsing."""
        parser = ReasoningParser("qwen3")

        # First chunk with start token
        reasoning, normal = parser.parse_stream_chunk("<think>")
        self.assertEqual(reasoning, "")
        self.assertEqual(normal, "")

        # Second chunk with reasoning content
        reasoning, normal = parser.parse_stream_chunk("thinking...")
        self.assertEqual(reasoning, "thinking...")
        self.assertEqual(normal, "")

        # Third chunk with end token and normal text
        reasoning, normal = parser.parse_stream_chunk("</think>answer")
        self.assertEqual(reasoning, "")  # Buffer cleared when end token processed
        self.assertEqual(normal, "answer")
```
**EN:** Test streaming chunk parsing. This test exercises `test_parse_stream_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming chunk parsing. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_stream_chunk`。

### Lines 883-891: test case case insensitive model type / 测试用例 case insensitive model type
```python
    def test_case_insensitive_model_type(self):
        """Test case insensitive model type matching."""
        parser1 = ReasoningParser("DeepSeek-R1")
        parser2 = ReasoningParser("QWEN3")
        parser3 = ReasoningParser("Kimi")

        self.assertIsInstance(parser1.detector, DeepSeekR1Detector)
        self.assertIsInstance(parser2.detector, Qwen3Detector)
        self.assertIsInstance(parser3.detector, KimiDetector)
```
**EN:** Test case insensitive model type matching. This test exercises `test_case_insensitive_model_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test case insensitive model type matching. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_case_insensitive_model_type`。

### Lines 893-899: test case stream reasoning parameter / 测试用例 stream reasoning parameter
```python
    def test_stream_reasoning_parameter(self):
        """Test stream_reasoning parameter is passed correctly."""
        parser = ReasoningParser("qwen3", stream_reasoning=False)
        self.assertFalse(parser.detector.stream_reasoning)

        parser = ReasoningParser("qwen3", stream_reasoning=True)
        self.assertTrue(parser.detector.stream_reasoning)
```
**EN:** Test stream_reasoning parameter is passed correctly. This test exercises `test_stream_reasoning_parameter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test stream_reasoning parameter is passed correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream_reasoning_parameter`。

### Lines 901-925: test case glm45 tool interruption / 测试用例 glm45 tool interruption
```python
    def test_glm45_tool_interruption(self):
        """Test GLM45 tool interruption through ReasoningParser API."""
        parser = ReasoningParser("glm45")

        # Non-streaming: tool interrupt
        reasoning, normal = parser.parse_non_stream(
            "<think>thinking<tool_call>tool call"
        )
        self.assertEqual(reasoning, "thinking")
        self.assertEqual(normal, "<tool_call>tool call")

        # Streaming: tool interrupt
        parser = ReasoningParser("glm45")
        chunks = ["<think>", "reasoning", "<tool_call>", "tool args"]
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            reasoning, normal = parser.parse_stream_chunk(chunk)
            if reasoning:
                all_reasoning += reasoning
            if normal:
                all_normal += normal

        self.assertEqual(all_reasoning, "reasoning")
        self.assertEqual(all_normal, "<tool_call>tool args")
```
**EN:** Test GLM45 tool interruption through ReasoningParser API. This test exercises `test_glm45_tool_interruption` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test GLM45 tool interruption through ReasoningParser API. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_glm45_tool_interruption`。

### Lines 927-956: test case kimik2 tool interruption / 测试用例 kimik2 tool interruption
```python
    def test_kimik2_tool_interruption(self):
        """Test Kimi-K2 tool interruption through ReasoningParser API."""
        parser = ReasoningParser("kimi_k2")

        # Non-streaming: tool interrupt
        reasoning, normal = parser.parse_non_stream(
            "<think>thinking<|tool_calls_section_begin|><|tool_call_begin|>"
        )
        self.assertEqual(reasoning, "thinking")
        self.assertEqual(normal, "<|tool_calls_section_begin|><|tool_call_begin|>")

        # Streaming: tool interrupt
        parser = ReasoningParser("kimi_k2")
        chunks = [
            "<think>",
            "reasoning",
            "<|tool_calls_section_begin|>",
            "<|tool_call_begin|>",
        ]
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            reasoning, normal = parser.parse_stream_chunk(chunk)
            if reasoning:
                all_reasoning += reasoning
            if normal:
                all_normal += normal

        self.assertEqual(all_reasoning, "reasoning")
        self.assertEqual(all_normal, "<|tool_calls_section_begin|><|tool_call_begin|>")
```
**EN:** Test Kimi-K2 tool interruption through ReasoningParser API. This test exercises `test_kimik2_tool_interruption` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Kimi-K2 tool interruption through ReasoningParser API. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimik2_tool_interruption`。

### Lines 959-959: class TestIntegrationScenarios declaration / 类 TestIntegrationScenarios 声明
```python
class TestIntegrationScenarios(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 960-960: supporting statements / 辅助语句
```python
    """Integration tests for realistic usage scenarios."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 962-972: test case deepseek r1 complete response / 测试用例 deepseek r1 complete response
```python
    def test_deepseek_r1_complete_response(self):
        """Test complete DeepSeek-R1 response parsing."""
        parser = ReasoningParser("deepseek-r1")
        text = "I need to solve this step by step. First, I'll analyze the problem. The given equation is x + 2 = 5. To solve for x, I subtract 2 from both sides: x = 5 - 2 = 3.</think>The answer is x = 3."

        reasoning, normal = parser.parse_non_stream(text)
        self.assertIn("step by step", reasoning)
        self.assertIn(
            "= 3", reasoning
        )  # The reasoning contains "x = 5 - 2 = 3" which has "= 3"
        self.assertEqual(normal, "The answer is x = 3.")
```
**EN:** Test complete DeepSeek-R1 response parsing. This test exercises `test_deepseek_r1_complete_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test complete DeepSeek-R1 response parsing. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_r1_complete_response`。

### Lines 974-996: test case qwen3 streaming scenario / 测试用例 qwen3 streaming scenario
```python
    def test_qwen3_streaming_scenario(self):
        """Test Qwen3 streaming scenario."""
        parser = ReasoningParser("qwen3")

        chunks = [
            "<think>",
            "Let me analyze this problem.",
            " I need to consider multiple factors.",
            "</think>",
            "Based on my analysis, the solution is to use a different approach.",
        ]

        all_reasoning = ""
        all_normal = ""

        for chunk in chunks:
            reasoning, normal = parser.parse_stream_chunk(chunk)
            all_reasoning += reasoning
            all_normal += normal

        self.assertIn("analyze", all_reasoning)
        self.assertIn("multiple factors", all_reasoning)
        self.assertIn("different approach", all_normal)
```
**EN:** Test Qwen3 streaming scenario. This test exercises `test_qwen3_streaming_scenario` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Qwen3 streaming scenario. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_streaming_scenario`。

### Lines 998-1019: test case kimi streaming scenario / 测试用例 kimi streaming scenario
```python
    def test_kimi_streaming_scenario(self):
        """Test Kimi streaming scenario."""
        parser = ReasoningParser("kimi")
        chunks = [
            "◁thi",
            "nk▷",
            "Let me analyze this problem.",
            " I need to consider multiple factors.",
            "◁/th",
            "ink▷",
            "The answer is 42.",
        ]
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            reasoning, normal = parser.parse_stream_chunk(chunk)
            all_reasoning += reasoning
            all_normal += normal

        self.assertIn("analyze", all_reasoning)
        self.assertIn("multiple factors", all_reasoning)
        self.assertIn("42", all_normal)
```
**EN:** Test Kimi streaming scenario. This test exercises `test_kimi_streaming_scenario` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Kimi streaming scenario. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_streaming_scenario`。

### Lines 1021-1028: test case gemma4 complete response / 测试用例 gemma4 complete response
```python
    def test_gemma4_complete_response(self):
        """Test complete Gemma4 response parsing (think_start_self_label stripped)."""
        parser = ReasoningParser("gemma4")
        text = "<|channel>thought\nI need to solve x + 2 = 5. Subtracting 2: x = 3.<channel|>The answer is x = 3."
        reasoning, normal = parser.parse_non_stream(text)
        self.assertIn("x = 3", reasoning)
        self.assertNotIn("thought\n", reasoning)
        self.assertEqual(normal, "The answer is x = 3.")
```
**EN:** Test complete Gemma4 response parsing (think_start_self_label stripped). This test exercises `test_gemma4_complete_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test complete Gemma4 response parsing (think_start_self_label stripped). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemma4_complete_response`。

### Lines 1030-1048: test case gemma4 streaming scenario / 测试用例 gemma4 streaming scenario
```python
    def test_gemma4_streaming_scenario(self):
        """Test Gemma4 streaming scenario."""
        parser = ReasoningParser("gemma4")
        chunks = [
            "<|channel>",
            "thought\nLet me analyze.",
            " Multiple factors.",
            "<channel|>",
            "The solution is 42.",
        ]
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            reasoning, normal = parser.parse_stream_chunk(chunk)
            all_reasoning += reasoning
            all_normal += normal
        self.assertIn("analyze", all_reasoning)
        self.assertIn("Multiple factors", all_reasoning)
        self.assertIn("42", all_normal)
```
**EN:** Test Gemma4 streaming scenario. This test exercises `test_gemma4_streaming_scenario` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Gemma4 streaming scenario. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemma4_streaming_scenario`。

### Lines 1050-1057: test case empty reasoning blocks / 测试用例 empty reasoning blocks
```python
    def test_empty_reasoning_blocks(self):
        """Test handling of empty reasoning blocks."""
        parser = ReasoningParser("qwen3")
        text = "<think></think>Just the answer."

        reasoning, normal = parser.parse_non_stream(text)
        self.assertEqual(reasoning, "")
        self.assertEqual(normal, "Just the answer.")
```
**EN:** Test handling of empty reasoning blocks. This test exercises `test_empty_reasoning_blocks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of empty reasoning blocks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_reasoning_blocks`。

### Lines 1059-1067: test case qwen3 forced reasoning complete response / 测试用例 qwen3 forced reasoning complete response
```python
    def test_qwen3_forced_reasoning_complete_response(self):
        """Test complete Qwen3-ForcedReasoning response parsing."""
        parser = ReasoningParser("qwen3", force_reasoning=True)
        text = "Let me solve this step by step. The equation is x + 2 = 5. Subtracting 2 from both sides gives x = 3.</think>The solution is x = 3."

        reasoning, normal = parser.parse_non_stream(text)
        self.assertIn("step by step", reasoning)
        self.assertIn("x = 3", reasoning)
        self.assertEqual(normal, "The solution is x = 3.")
```
**EN:** Test complete Qwen3-ForcedReasoning response parsing. This test exercises `test_qwen3_forced_reasoning_complete_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test complete Qwen3-ForcedReasoning response parsing. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_forced_reasoning_complete_response`。

### Lines 1069-1091: test case qwen3 forced reasoning streaming scenario / 测试用例 qwen3 forced reasoning streaming scenario
```python
    def test_qwen3_forced_reasoning_streaming_scenario(self):
        """Test Qwen3-ForcedReasoning streaming scenario."""
        parser = ReasoningParser("qwen3", force_reasoning=True)

        chunks = [
            "I need to analyze",
            " this problem carefully.",
            " Let me break it down.",
            "</think>",
            "The final answer is 42.",
        ]

        all_reasoning = ""
        all_normal = ""

        for chunk in chunks:
            reasoning, normal = parser.parse_stream_chunk(chunk)
            all_reasoning += reasoning
            all_normal += normal

        self.assertIn("analyze", all_reasoning)
        self.assertIn("break it down", all_reasoning)
        self.assertIn("final answer", all_normal)
```
**EN:** Test Qwen3-ForcedReasoning streaming scenario. This test exercises `test_qwen3_forced_reasoning_streaming_scenario` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Qwen3-ForcedReasoning streaming scenario. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_forced_reasoning_streaming_scenario`。

### Lines 1094-1094: class TestBufferLossBugFix declaration / 类 TestBufferLossBugFix 声明
```python
class TestBufferLossBugFix(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1095-1095: supporting statements / 辅助语句
```python
    """Test cases for the buffer loss bug fix in parse_streaming_increment."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1097-1127: test case partial end tag buffer loss bug / 测试用例 partial end tag buffer loss bug
```python
    def test_partial_end_tag_buffer_loss_bug(self):
        """
        Test the bug where partial end tag fragments are lost when followed by normal text.

        Bug scenario:
        1. _in_reasoning is False
        2. new_text is "</" (part of closing thinking tag)
        3. Fragment is stored in buffer and empty string is returned
        4. Next step: new_text is "answer", _in_reasoning still False
        5. Buffer is cleared and "answer" is returned directly
        6. The "</" from previous step is lost

        This test verifies the fix where the return was changed from:
        return StreamingParseResult(normal_text=new_text)
        to:
        return StreamingParseResult(normal_text=current_text)
        """
        detector = BaseReasoningFormatDetector("<think>", "</think>")

        # Step 1: Send partial end tag when not in reasoning mode
        # This should be buffered since it could be start of "</think>"
        result1 = detector.parse_streaming_increment("</")
        self.assertEqual(result1.normal_text, "")
        self.assertEqual(result1.reasoning_text, "")

        # Step 2: Send normal text that doesn't complete the end tag
        # Before fix: would return only "answer", losing the "</"
        # After fix: should return the complete buffered content "</answer"
        result2 = detector.parse_streaming_increment("answer")
        self.assertEqual(result2.normal_text, "</answer")
        self.assertEqual(result2.reasoning_text, "")
```
**EN:** Test the bug where partial end tag fragments are lost when followed by normal text. This test exercises `test_partial_end_tag_buffer_loss_bug` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the bug where partial end tag fragments are lost when followed by normal text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_end_tag_buffer_loss_bug`。

### Lines 1129-1143: test case partial start tag buffer preservation / 测试用例 partial start tag buffer preservation
```python
    def test_partial_start_tag_buffer_preservation(self):
        """
        Test that partial start tag fragments are properly preserved.
        """
        detector = BaseReasoningFormatDetector("<think>", "</think>")

        # Send partial start tag
        result1 = detector.parse_streaming_increment("<th")
        self.assertEqual(result1.normal_text, "")
        self.assertEqual(result1.reasoning_text, "")

        # Complete with non-matching text
        result2 = detector.parse_streaming_increment("is is text")
        self.assertEqual(result2.normal_text, "<this is text")
        self.assertEqual(result2.reasoning_text, "")
```
**EN:** Test that partial start tag fragments are properly preserved. This test exercises `test_partial_start_tag_buffer_preservation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that partial start tag fragments are properly preserved. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_start_tag_buffer_preservation`。

### Lines 1145-1164: test case partial end tag in reasoning mode / 测试用例 partial end tag in reasoning mode
```python
    def test_partial_end_tag_in_reasoning_mode(self):
        """
        Test partial end tag handling when already in reasoning mode.
        """
        detector = BaseReasoningFormatDetector("<think>", "</think>")

        # Enter reasoning mode
        detector.parse_streaming_increment("<think>")
        detector.parse_streaming_increment("some reasoning")

        # Send partial end tag
        result1 = detector.parse_streaming_increment("</")
        self.assertEqual(result1.normal_text, "")
        self.assertEqual(result1.reasoning_text, "")

        # Complete the end tag with normal text
        result2 = detector.parse_streaming_increment("think>normal text")
        self.assertEqual(result2.normal_text, "normal text")
        # The reasoning text should be empty since buffer was cleared when end tag was processed
        self.assertEqual(result2.reasoning_text, "")
```
**EN:** Test partial end tag handling when already in reasoning mode. This test exercises `test_partial_end_tag_in_reasoning_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test partial end tag handling when already in reasoning mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_end_tag_in_reasoning_mode`。

### Lines 1166-1183: test case multiple partial fragments / 测试用例 multiple partial fragments
```python
    def test_multiple_partial_fragments(self):
        """
        Test handling of multiple partial fragments that don't match any tokens.
        """
        detector = BaseReasoningFormatDetector("<think>", "</think>")

        # Send multiple partial fragments
        result1 = detector.parse_streaming_increment("<")
        self.assertEqual(result1.normal_text, "")
        self.assertEqual(result1.reasoning_text, "")

        result2 = detector.parse_streaming_increment("/")
        self.assertEqual(result2.normal_text, "")
        self.assertEqual(result2.reasoning_text, "")

        result3 = detector.parse_streaming_increment("random>")
        self.assertEqual(result3.normal_text, "</random>")
        self.assertEqual(result3.reasoning_text, "")
```
**EN:** Test handling of multiple partial fragments that don't match any tokens. This test exercises `test_multiple_partial_fragments` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of multiple partial fragments that don't match any tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_partial_fragments`。

### Lines 1185-1203: test case edge case exact token match / 测试用例 edge case exact token match
```python
    def test_edge_case_exact_token_match(self):
        """
        Test edge case where buffer content exactly matches a token.
        """
        detector = BaseReasoningFormatDetector("<think>", "</think>")

        # Build up the exact start token character by character
        detector.parse_streaming_increment("<")
        detector.parse_streaming_increment("t")
        detector.parse_streaming_increment("h")
        detector.parse_streaming_increment("i")
        detector.parse_streaming_increment("n")
        result = detector.parse_streaming_increment("k>")

        # Should enter reasoning mode
        self.assertEqual(result.normal_text, "")
        self.assertEqual(result.reasoning_text, "")
        self.assertTrue(detector._in_reasoning)
        self.assertTrue(detector.stripped_think_start)
```
**EN:** Test edge case where buffer content exactly matches a token. This test exercises `test_edge_case_exact_token_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test edge case where buffer content exactly matches a token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_edge_case_exact_token_match`。

### Lines 1206-1206: class TestGptOssDetector declaration / 类 TestGptOssDetector 声明
```python
class TestGptOssDetector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1207-1207: supporting statements / 辅助语句
```python
    """Test cases for GptOssDetector which delegates to HarmonyParser."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1209-1212: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        from sglang.srt.parser.reasoning_parser import GptOssDetector

        self.detector = GptOssDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1214-1219: test case detect and parse with analysis and final / 测试用例 detect and parse with analysis and final
```python
    def test_detect_and_parse_with_analysis_and_final(self):
        """Test one-shot parsing with analysis (reasoning) and final (normal) blocks."""
        text = "<|start|><|channel|>analysis<|message|>thinking hard<|end|><|channel|>final<|message|>the answer<|end|>"
        result = self.detector.detect_and_parse(text)
        self.assertIn("thinking hard", result.reasoning_text)
        self.assertIn("the answer", result.normal_text)
```
**EN:** Test one-shot parsing with analysis (reasoning) and final (normal) blocks. This test exercises `test_detect_and_parse_with_analysis_and_final` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test one-shot parsing with analysis (reasoning) and final (normal) blocks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_analysis_and_final`。

### Lines 1221-1225: test case detect and parse normal only / 测试用例 detect and parse normal only
```python
    def test_detect_and_parse_normal_only(self):
        """Test one-shot parsing with only final block."""
        text = "<|start|><|channel|>final<|message|>just the answer<|end|>"
        result = self.detector.detect_and_parse(text)
        self.assertIn("just the answer", result.normal_text)
```
**EN:** Test one-shot parsing with only final block. This test exercises `test_detect_and_parse_normal_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test one-shot parsing with only final block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_normal_only`。

### Lines 1227-1243: test case streaming analysis then final / 测试用例 streaming analysis then final
```python
    def test_streaming_analysis_then_final(self):
        """Test streaming parse across multiple chunks."""
        chunks = [
            "<|start|><|channel|>analysis<|message|>",
            "reasoning part",
            "<|end|>",
            "<|channel|>final<|message|>answer",
            "<|end|>",
        ]
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk)
            all_reasoning += result.reasoning_text
            all_normal += result.normal_text
        self.assertIn("reasoning part", all_reasoning)
        self.assertIn("answer", all_normal)
```
**EN:** Test streaming parse across multiple chunks. This test exercises `test_streaming_analysis_then_final` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse across multiple chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_analysis_then_final`。

### Lines 1245-1250: test case streaming with tool call / 测试用例 streaming with tool call
```python
    def test_streaming_with_tool_call(self):
        """Test streaming parse with tool call events."""
        text = "<|start|><|channel|>analysis<|message|>think<|end|><|call|>tool_data<|return|><|channel|>final<|message|>result<|end|>"
        result = self.detector.detect_and_parse(text)
        self.assertIn("think", result.reasoning_text)
        self.assertIn("result", result.normal_text)
```
**EN:** Test streaming parse with tool call events. This test exercises `test_streaming_with_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse with tool call events. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_with_tool_call`。

### Lines 1253-1253: class TestMiniMaxAppendThinkDetector declaration / 类 TestMiniMaxAppendThinkDetector 声明
```python
class TestMiniMaxAppendThinkDetector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1254-1254: supporting statements / 辅助语句
```python
    """Test cases for MiniMaxAppendThinkDetector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1256-1259: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        from sglang.srt.parser.reasoning_parser import MiniMaxAppendThinkDetector

        self.detector = MiniMaxAppendThinkDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1261-1264: test case detect and parse prepends think / 测试用例 detect and parse prepends think
```python
    def test_detect_and_parse_prepends_think(self):
        """Test that detect_and_parse prepends <think> to the text."""
        result = self.detector.detect_and_parse("Hello world")
        self.assertEqual(result.normal_text, "<think>Hello world")
```
**EN:** Test that detect_and_parse prepends <think> to the text. This test exercises `test_detect_and_parse_prepends_think` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that detect_and_parse prepends <think> to the text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_prepends_think`。

### Lines 1266-1269: test case streaming first chunk prepends think / 测试用例 streaming first chunk prepends think
```python
    def test_streaming_first_chunk_prepends_think(self):
        """Test that first streaming chunk gets <think> prepended."""
        result = self.detector.parse_streaming_increment("First chunk")
        self.assertEqual(result.normal_text, "<think>First chunk")
```
**EN:** Test that first streaming chunk gets <think> prepended. This test exercises `test_streaming_first_chunk_prepends_think` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that first streaming chunk gets <think> prepended. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_first_chunk_prepends_think`。

### Lines 1271-1275: test case streaming second chunk no prepend / 测试用例 streaming second chunk no prepend
```python
    def test_streaming_second_chunk_no_prepend(self):
        """Test that subsequent streaming chunks are passed through."""
        self.detector.parse_streaming_increment("First")
        result = self.detector.parse_streaming_increment("Second")
        self.assertEqual(result.normal_text, "Second")
```
**EN:** Test that subsequent streaming chunks are passed through. This test exercises `test_streaming_second_chunk_no_prepend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that subsequent streaming chunks are passed through. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_second_chunk_no_prepend`。

### Lines 1278-1278: class TestReasoningParserAdvanced declaration / 类 TestReasoningParserAdvanced 声明
```python
class TestReasoningParserAdvanced(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1279-1279: supporting statements / 辅助语句
```python
    """Additional tests for ReasoningParser init edge cases."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1281-1286: test case gpt oss model type / 测试用例 gpt oss model type
```python
    def test_gpt_oss_model_type(self):
        """Test that gpt-oss model type creates GptOssDetector."""
        from sglang.srt.parser.reasoning_parser import GptOssDetector

        parser = ReasoningParser("gpt-oss")
        self.assertIsInstance(parser.detector, GptOssDetector)
```
**EN:** Test that gpt-oss model type creates GptOssDetector. This test exercises `test_gpt_oss_model_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that gpt-oss model type creates GptOssDetector. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gpt_oss_model_type`。

### Lines 1288-1293: test case minimax append think model type / 测试用例 minimax append think model type
```python
    def test_minimax_append_think_model_type(self):
        """Test that minimax-append-think creates MiniMaxAppendThinkDetector."""
        from sglang.srt.parser.reasoning_parser import MiniMaxAppendThinkDetector

        parser = ReasoningParser("minimax-append-think")
        self.assertIsInstance(parser.detector, MiniMaxAppendThinkDetector)
```
**EN:** Test that minimax-append-think creates MiniMaxAppendThinkDetector. This test exercises `test_minimax_append_think_model_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that minimax-append-think creates MiniMaxAppendThinkDetector. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_minimax_append_think_model_type`。

### Lines 1295-1298: test case qwen3 thinking forces reasoning / 测试用例 qwen3 thinking forces reasoning
```python
    def test_qwen3_thinking_forces_reasoning(self):
        """Test that qwen3-thinking model type forces reasoning mode."""
        parser = ReasoningParser("qwen3-thinking")
        self.assertTrue(parser.detector._in_reasoning)
```
**EN:** Test that qwen3-thinking model type forces reasoning mode. This test exercises `test_qwen3_thinking_forces_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that qwen3-thinking model type forces reasoning mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_thinking_forces_reasoning`。

### Lines 1300-1308: test case minimax forces reasoning / 测试用例 minimax forces reasoning
```python
    def test_minimax_forces_reasoning(self):
        """Test that minimax model type forces reasoning mode.

        minimax maps to Qwen3Detector but ReasoningParser overrides
        force_reasoning=True, unlike the default Qwen3Detector behavior.
        """
        parser = ReasoningParser("minimax")
        self.assertIsInstance(parser.detector, Qwen3Detector)
        self.assertTrue(parser.detector._in_reasoning)
```
**EN:** Test that minimax model type forces reasoning mode. This test exercises `test_minimax_forces_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that minimax model type forces reasoning mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_minimax_forces_reasoning`。

### Lines 1310-1325: test case detector map aliases / 测试用例 detector map aliases
```python
    def test_detector_map_aliases(self):
        """Test that all DetectorMap alias keys create the correct detector type."""
        # These are aliases that map to existing detector classes
        alias_tests = {
            "deepseek-v3": Qwen3Detector,
            "step3": DeepSeekR1Detector,
            "step3p5": DeepSeekR1Detector,
            "interns1": Qwen3Detector,
        }
        for model_type, expected_class in alias_tests.items():
            parser = ReasoningParser(model_type)
            self.assertIsInstance(
                parser.detector,
                expected_class,
                f"{model_type} should create {expected_class.__name__}",
            )
```
**EN:** Test that all DetectorMap alias keys create the correct detector type. This test exercises `test_detector_map_aliases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that all DetectorMap alias keys create the correct detector type. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detector_map_aliases`。

### Lines 1327-1346: test case continue final message with request / 测试用例 continue final message with request
```python
    def test_continue_final_message_with_request(self):
        """Test continue_final_message passes previous content to detector."""
        from sglang.srt.entrypoints.openai.protocol import (
            ChatCompletionMessageGenericParam,
            ChatCompletionMessageUserParam,
            ChatCompletionRequest,
        )

        request = ChatCompletionRequest(
            model="test",
            messages=[
                ChatCompletionMessageUserParam(role="user", content="Hi"),
                ChatCompletionMessageGenericParam(
                    role="assistant", content="Let me think..."
                ),
            ],
            continue_final_message=True,
        )
        parser = ReasoningParser("qwen3", request=request)
        self.assertTrue(parser.detector.continue_final_message)
```
**EN:** Test continue_final_message passes previous content to detector. This test exercises `test_continue_final_message_with_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test continue_final_message passes previous content to detector. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_continue_final_message_with_request`。

### Lines 1348-1363: test case force nonempty content via chat template kwargs / 测试用例 force nonempty content via chat template kwargs
```python
    def test_force_nonempty_content_via_chat_template_kwargs(self):
        """Test that force_nonempty_content is passed via chat_template_kwargs."""
        from sglang.srt.entrypoints.openai.protocol import (
            ChatCompletionMessageUserParam,
            ChatCompletionRequest,
        )

        request = ChatCompletionRequest(
            model="test",
            messages=[
                ChatCompletionMessageUserParam(role="user", content="Hi"),
            ],
            chat_template_kwargs={"force_nonempty_content": True},
        )
        parser = ReasoningParser("nemotron_3", request=request)
        self.assertTrue(parser.detector._force_nonempty_content)
```
**EN:** Test that force_nonempty_content is passed via chat_template_kwargs. This test exercises `test_force_nonempty_content_via_chat_template_kwargs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that force_nonempty_content is passed via chat_template_kwargs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_force_nonempty_content_via_chat_template_kwargs`。

### Lines 1366-1366: class TestContinueFinalMessage declaration / 类 TestContinueFinalMessage 声明
```python
class TestContinueFinalMessage(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1367-1367: supporting statements / 辅助语句
```python
    """Test continue_final_message mode for BaseReasoningFormatDetector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1369-1379: test case continue with think start in previous / 测试用例 continue with think start in previous
```python
    def test_continue_with_think_start_in_previous(self):
        """Test that previous_content with <think> sets _in_reasoning=True."""
        detector = BaseReasoningFormatDetector(
            "<think>",
            "</think>",
            force_reasoning=False,
            continue_final_message=True,
            previous_content="<think>some reasoning",
        )
        self.assertTrue(detector._in_reasoning)
        self.assertEqual(detector.previous_count, len("<think>some reasoning"))
```
**EN:** Test that previous_content with <think> sets _in_reasoning=True. This test exercises `test_continue_with_think_start_in_previous` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that previous_content with <think> sets _in_reasoning=True. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_continue_with_think_start_in_previous`。

### Lines 1381-1391: test case continue with think end in previous / 测试用例 continue with think end in previous
```python
    def test_continue_with_think_end_in_previous(self):
        """Test that previous_content with </think> sets _in_reasoning=False."""
        detector = BaseReasoningFormatDetector(
            "<think>",
            "</think>",
            force_reasoning=True,
            continue_final_message=True,
            previous_content="<think>done</think>normal",
        )
        # think_end_token in previous → _in_reasoning = False
        self.assertFalse(detector._in_reasoning)
```
**EN:** Test that previous_content with </think> sets _in_reasoning=False. This test exercises `test_continue_with_think_end_in_previous` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that previous_content with </think> sets _in_reasoning=False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_continue_with_think_end_in_previous`。

### Lines 1393-1410: test case continue detect parse with end in previous / 测试用例 continue detect parse with end in previous
```python
    def test_continue_detect_parse_with_end_in_previous(self):
        """Test detect_and_parse when think_end_token is in previous_content only.
        This covers the branch where think_end is NOT in current text
        but IS in previous_content, so output is treated as normal_text."""
        detector = BaseReasoningFormatDetector(
            "<think>",
            "</think>",
            force_reasoning=True,
            continue_final_message=True,
            previous_content="<think>reasoning</think>",
        )
        # _in_reasoning is False (think_end in previous)
        # But force_reasoning was True → detect_and_parse still enters the
        # reasoning path because think_start is in previous_content.
        # However, since _in_reasoning=False and no think_start in new text,
        # it returns normal_text directly.
        result = detector.detect_and_parse("new content here")
        self.assertEqual(result.normal_text, "new content here")
```
**EN:** Test detect_and_parse when think_end_token is in previous_content only. This test exercises `test_continue_detect_parse_with_end_in_previous` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test detect_and_parse when think_end_token is in previous_content only. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_continue_detect_parse_with_end_in_previous`。

### Lines 1412-1431: test case continue end in previous new text has start but no end / 测试用例 continue end in previous new text has start but no end
```python
    def test_continue_end_in_previous_new_text_has_start_but_no_end(self):
        """Test: think_end in previous, new text has think_start but no think_end.
        This produces: in_reasoning=True (from think_start in text),
        think_end NOT in processed_text, think_end IN previous_content,
        so it falls through to the else branch that returns normal_text."""
        detector = BaseReasoningFormatDetector(
            "<think>",
            "</think>",
            force_reasoning=False,
            continue_final_message=True,
            previous_content="earlier <think>old</think>old answer",
        )
        # _in_reasoning = False (think_end in previous overrides)
        self.assertFalse(detector._in_reasoning)
        # New text has <think> (triggers in_reasoning) but no </think>
        # think_end IS in previous_content → skips the truncated-reasoning branch
        # think_end NOT in processed_text → falls to else that returns normal_text
        result = detector.detect_and_parse("<think>continuing reasoning")
        self.assertEqual(result.normal_text, "continuing reasoning")
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Test: think_end in previous, new text has think_start but no think_end. This test exercises `test_continue_end_in_previous_new_text_has_start_but_no_end` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test: think_end in previous, new text has think_start but no think_end. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_continue_end_in_previous_new_text_has_start_but_no_end`。

### Lines 1433-1448: test case continue detect parse think start in prev but end also in prev / 测试用例 continue detect parse think start in prev but end also in prev
```python
    def test_continue_detect_parse_think_start_in_prev_but_end_also_in_prev(self):
        """Test detect_and_parse where both think tokens are in previous,
        and new text contains <think> to re-enter reasoning."""
        detector = BaseReasoningFormatDetector(
            "<think>",
            "</think>",
            force_reasoning=False,
            continue_final_message=True,
            previous_content="<think>old reasoning</think>old answer",
        )
        # _in_reasoning = False (end token in previous overrides start)
        self.assertFalse(detector._in_reasoning)
        # New text starts a fresh reasoning block
        result = detector.detect_and_parse("<think>new reasoning</think>new answer")
        self.assertEqual(result.reasoning_text, "new reasoning")
        self.assertEqual(result.normal_text, "new answer")
```
**EN:** Test detect_and_parse where both think tokens are in previous, and new text contains <think> to re-enter reasoning. This test exercises `test_continue_detect_parse_think_start_in_prev_but_end_also_in_prev` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test detect_and_parse where both think tokens are in previous, and new text contains <think> to re-enter reasoning. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_continue_detect_parse_think_start_in_prev_but_end_also_in_prev`。

### Lines 1450-1458: test case streaming returns empty when in reasoning and end buffered / 测试用例 streaming returns empty when in reasoning and end buffered
```python
    def test_streaming_returns_empty_when_in_reasoning_and_end_buffered(self):
        """Test that streaming returns empty when buffer could be partial end token."""
        detector = BaseReasoningFormatDetector(
            "<think>", "</think>", force_reasoning=True, stream_reasoning=True
        )
        # In reasoning mode, send partial end token
        result = detector.parse_streaming_increment("</")
        self.assertEqual(result.reasoning_text, "")
        self.assertEqual(result.normal_text, "")
```
**EN:** Test that streaming returns empty when buffer could be partial end token. This test exercises `test_streaming_returns_empty_when_in_reasoning_and_end_buffered` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that streaming returns empty when buffer could be partial end token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_returns_empty_when_in_reasoning_and_end_buffered`。

### Lines 1459-1462: supporting source context / 辅助源码上下文
```python
        # This goes through the path where _in_reasoning is True but buffer
        # is a prefix of think_end_token → returns empty


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 1463-1463: class TestGptOssDetectorToolCall declaration / 类 TestGptOssDetectorToolCall 声明
```python
class TestGptOssDetectorToolCall(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1464-1464: supporting statements / 辅助语句
```python
    """Test GptOssDetector tool_call raw_text handling."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1466-1480: test case detect and parse tool call raw text / 测试用例 detect and parse tool call raw text
```python
    def test_detect_and_parse_tool_call_raw_text(self):
        """Test that tool_call events use raw_text when available."""
        from sglang.srt.parser.reasoning_parser import GptOssDetector

        detector = GptOssDetector()
        # Sequence with CALL...RETURN that produces tool_call events with raw_text
        text = (
            "<|start|><|channel|>analysis<|message|>think<|end|>"
            "<|call|>function_data<|return|>"
            "<|channel|>final<|message|>result<|end|>"
        )
        result = detector.detect_and_parse(text)
        self.assertIn("think", result.reasoning_text)
        # Tool call raw_text and/or final result should be in normal_text
        self.assertIn("result", result.normal_text)
```
**EN:** Test that tool_call events use raw_text when available. This test exercises `test_detect_and_parse_tool_call_raw_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that tool_call events use raw_text when available. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_tool_call_raw_text`。

### Lines 1482-1499: test case streaming tool call raw text / 测试用例 streaming tool call raw text
```python
    def test_streaming_tool_call_raw_text(self):
        """Test streaming parse with tool_call events preserving raw_text."""
        from sglang.srt.parser.reasoning_parser import GptOssDetector

        detector = GptOssDetector()
        chunks = [
            "<|start|><|channel|>analysis<|message|>reason<|end|>",
            "<|call|>tool_payload<|return|>",
            "<|channel|>final<|message|>done<|end|>",
        ]
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk)
            all_reasoning += result.reasoning_text
            all_normal += result.normal_text
        self.assertIn("reason", all_reasoning)
        self.assertIn("done", all_normal)
```
**EN:** Test streaming parse with tool_call events preserving raw_text. This test exercises `test_streaming_tool_call_raw_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parse with tool_call events preserving raw_text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_call_raw_text`。

### Lines 1502-1502: class TestPoolsideV1Registered declaration / 类 TestPoolsideV1Registered 声明
```python
class TestPoolsideV1Registered(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1503-1506: supporting statements / 辅助语句
```python
    """poolside_v1 (Laguna-XS.2) reuses the Qwen3 `<think>...</think>` envelope.
    Request dispatch differs (Mimo-style explicit `enable_thinking=True`,
    asserted in test_serving_chat.py), driven by
    `reasoning_default = "explicit_enable_thinking"` on the detector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1508-1510: test case registered to qwen3 subclass / 测试用例 registered to qwen3 subclass
```python
    def test_registered_to_qwen3_subclass(self):
        cls = ReasoningParser.DetectorMap["poolside_v1"]
        self.assertTrue(issubclass(cls, Qwen3Detector))
```
**EN:** This test exercises `test_registered_to_qwen3_subclass` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_registered_to_qwen3_subclass`。

### Lines 1512-1515: test case explicit enable thinking default / 测试用例 explicit enable thinking default
```python
    def test_explicit_enable_thinking_default(self):
        rp = ReasoningParser("poolside_v1", stream_reasoning=True)
        self.assertEqual(rp.detector.reasoning_default, "explicit_enable_thinking")
        self.assertTrue(rp.detector.thinks_internally)
```
**EN:** This test exercises `test_explicit_enable_thinking_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_explicit_enable_thinking_default`。

### Lines 1518-1519: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStreamingParseResult`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBaseReasoningFormatDetector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDeepSeekR1Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwen3Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwen3ForcedReasoningDetector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiDetector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK2Detector`: Test cases for KimiK2 detector with tool interruption support. / 用于组织相关测试、夹具或辅助方法。
- `TestGlm45Detector`: Test cases for GLM45 detector with tool interruption support. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingParseResult.test_init_default`: Test default initialization of StreamingParseResult. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_default`。
- `TestStreamingParseResult.test_init_with_values`: Test initialization with specific values. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_with_values`。
- `TestBaseReasoningFormatDetector.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestBaseReasoningFormatDetector.test_init`: Test initialization of BaseReasoningFormatDetector. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.parser.reasoning_parser`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 1519
