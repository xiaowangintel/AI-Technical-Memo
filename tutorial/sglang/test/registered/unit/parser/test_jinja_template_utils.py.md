# test_jinja_template_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/parser/test_jinja_template_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates jinja template utils behavior in SGLang's unit / parser area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 解析器 领域中与 jinja template utils 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/parser/jinja_template_utils.py"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-10: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.parser.jinja_template_utils import (
    detect_jinja_template_content_format,
    process_content_for_template_format,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.parser.jinja_template_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.parser.jinja_template_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 12-12: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 15-15: class TestTemplateContentFormatDetection declaration / 类 TestTemplateContentFormatDetection 声明
```python
class TestTemplateContentFormatDetection(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 16-16: supporting statements / 辅助语句
```python
    """Test template content format detection functionality."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 18-37: test case detect llama4 openai format / 测试用例 detect llama4 openai format
```python
    def test_detect_llama4_openai_format(self):
        """Test detection of llama4-style template (should be 'openai' format)."""
        llama4_pattern = """
{%- for message in messages %}
    {%- if message['content'] is string %}
        {{- message['content'] }}
    {%- else %}
        {%- for content in message['content'] %}
            {%- if content['type'] == 'image' %}
                {{- '<|image|>' }}
            {%- elif content['type'] == 'text' %}
                {{- content['text'] | trim }}
            {%- endif %}
        {%- endfor %}
    {%- endif %}
{%- endfor %}
        """

        result = detect_jinja_template_content_format(llama4_pattern)
        self.assertEqual(result, "openai")
```
**EN:** Test detection of llama4-style template (should be 'openai' format). This test exercises `test_detect_llama4_openai_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test detection of llama4-style template (should be 'openai' format). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_llama4_openai_format`。

### Lines 39-50: test case detect deepseek string format / 测试用例 detect deepseek string format
```python
    def test_detect_deepseek_string_format(self):
        """Test detection of deepseek-style template (should be 'string' format)."""
        deepseek_pattern = """
{%- for message in messages %}
    {%- if message['role'] == 'user' %}
        {{- '<|User|>' + message['content'] + '<|Assistant|>' }}
    {%- endif %}
{%- endfor %}
        """

        result = detect_jinja_template_content_format(deepseek_pattern)
        self.assertEqual(result, "string")
```
**EN:** Test detection of deepseek-style template (should be 'string' format). This test exercises `test_detect_deepseek_string_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test detection of deepseek-style template (should be 'string' format). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_deepseek_string_format`。

### Lines 52-57: test case detect invalid template / 测试用例 detect invalid template
```python
    def test_detect_invalid_template(self):
        """Test handling of invalid template (should default to 'string')."""
        invalid_pattern = "{{{{ invalid jinja syntax }}}}"

        result = detect_jinja_template_content_format(invalid_pattern)
        self.assertEqual(result, "string")
```
**EN:** Test handling of invalid template (should default to 'string'). This test exercises `test_detect_invalid_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of invalid template (should default to 'string'). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_invalid_template`。

### Lines 59-62: test case detect empty template / 测试用例 detect empty template
```python
    def test_detect_empty_template(self):
        """Test handling of empty template (should default to 'string')."""
        result = detect_jinja_template_content_format("")
        self.assertEqual(result, "string")
```
**EN:** Test handling of empty template (should default to 'string'). This test exercises `test_detect_empty_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of empty template (should default to 'string'). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_empty_template`。

### Lines 64-102: test case detect msg content pattern / 测试用例 detect msg content pattern
```python
    def test_detect_msg_content_pattern(self):
        """Test detection of template with msg.content pattern (should be 'openai' format)."""
        msg_content_pattern = """
[gMASK]<sop>
{%- for msg in messages %}
    {%- if msg.role == 'system' %}
<|system|>
{{ msg.content }}
    {%- elif msg.role == 'user' %}
<|user|>{{ '\n' }}
        {%- if msg.content is string %}
{{ msg.content }}
        {%- else %}
            {%- for item in msg.content %}
                {%- if item.type == 'video' or 'video' in item %}
<|begin_of_video|><|video|><|end_of_video|>
                {%- elif item.type == 'image' or 'image' in item %}
<|begin_of_image|><|image|><|end_of_image|>
                {%- elif item.type == 'text' %}
{{ item.text }}
                {%- endif %}
            {%- endfor %}
        {%- endif %}
    {%- elif msg.role == 'assistant' %}
        {%- if msg.metadata %}
<|assistant|>{{ msg.metadata }}
{{ msg.content }}
        {%- else %}
<|assistant|>
{{ msg.content }}
        {%- endif %}
    {%- endif %}
{%- endfor %}
{% if add_generation_prompt %}<|assistant|>
{% endif %}
        """

        result = detect_jinja_template_content_format(msg_content_pattern)
        self.assertEqual(result, "openai")
```
**EN:** Test detection of template with msg.content pattern (should be 'openai' format). This test exercises `test_detect_msg_content_pattern` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test detection of template with msg.content pattern (should be 'openai' format). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_msg_content_pattern`。

### Lines 104-142: test case detect m content pattern / 测试用例 detect m content pattern
```python
    def test_detect_m_content_pattern(self):
        """Test detection of template with m.content pattern (should be 'openai' format)."""
        msg_content_pattern = """
[gMASK]<sop>
{%- for m in messages %}
    {%- if m.role == 'system' %}
<|system|>
{{ m.content }}
    {%- elif m.role == 'user' %}
<|user|>{{ '\n' }}
        {%- if m.content is string %}
{{ m.content }}
        {%- else %}
            {%- for item in m.content %}
                {%- if item.type == 'video' or 'video' in item %}
<|begin_of_video|><|video|><|end_of_video|>
                {%- elif item.type == 'image' or 'image' in item %}
<|begin_of_image|><|image|><|end_of_image|>
                {%- elif item.type == 'text' %}
{{ item.text }}
                {%- endif %}
            {%- endfor %}
        {%- endif %}
    {%- elif m.role == 'assistant' %}
        {%- if m.metadata %}
<|assistant|>{{ m.metadata }}
{{ m.content }}
        {%- else %}
<|assistant|>
{{ m.content }}
        {%- endif %}
    {%- endif %}
{%- endfor %}
{% if add_generation_prompt %}<|assistant|>
{% endif %}
        """

        result = detect_jinja_template_content_format(msg_content_pattern)
        self.assertEqual(result, "openai")
```
**EN:** Test detection of template with m.content pattern (should be 'openai' format). This test exercises `test_detect_m_content_pattern` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test detection of template with m.content pattern (should be 'openai' format). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_m_content_pattern`。

### Lines 144-178: test case process content openai format / 测试用例 process content openai format
```python
    def test_process_content_openai_format(self):
        """Test content processing for openai format."""
        msg_dict = {
            "role": "user",
            "content": [
                {"type": "text", "text": "Look at this image:"},
                {
                    "type": "image_url",
                    "image_url": {"url": "http://example.com/image.jpg"},
                },
                {"type": "text", "text": "What do you see?"},
            ],
        }

        image_data = []
        video_data = []
        audio_data = []
        modalities = []

        result = process_content_for_template_format(
            msg_dict, "openai", image_data, video_data, audio_data, modalities
        )

        # Check that image_data was extracted
        self.assertEqual(len(image_data), 1)
        self.assertEqual(image_data[0].url, "http://example.com/image.jpg")

        # Check that content was normalized
        expected_content = [
            {"type": "text", "text": "Look at this image:"},
            {"type": "image"},  # normalized from image_url
            {"type": "text", "text": "What do you see?"},
        ]
        self.assertEqual(result["content"], expected_content)
        self.assertEqual(result["role"], "user")
```
**EN:** Test content processing for openai format. This test exercises `test_process_content_openai_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test content processing for openai format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_openai_format`。

### Lines 180-208: test case process content string format / 测试用例 process content string format
```python
    def test_process_content_string_format(self):
        """Test content processing for string format."""
        msg_dict = {
            "role": "user",
            "content": [
                {"type": "text", "text": "Hello"},
                {
                    "type": "image_url",
                    "image_url": {"url": "http://example.com/image.jpg"},
                },
                {"type": "text", "text": "world"},
            ],
        }

        image_data = []
        video_data = []
        audio_data = []
        modalities = []

        result = process_content_for_template_format(
            msg_dict, "string", image_data, video_data, audio_data, modalities
        )

        # For string format, should flatten to text only
        self.assertEqual(result["content"], "Hello world")
        self.assertEqual(result["role"], "user")

        # Image data should not be extracted for string format
        self.assertEqual(len(image_data), 0)
```
**EN:** Test content processing for string format. This test exercises `test_process_content_string_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test content processing for string format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_string_format`。

### Lines 210-241: test case process content with audio / 测试用例 process content with audio
```python
    def test_process_content_with_audio(self):
        """Test content processing with audio content."""
        msg_dict = {
            "role": "user",
            "content": [
                {"type": "text", "text": "Listen to this:"},
                {
                    "type": "audio_url",
                    "audio_url": {"url": "http://example.com/audio.mp3"},
                },
            ],
        }

        image_data = []
        video_data = []
        audio_data = []
        modalities = []

        result = process_content_for_template_format(
            msg_dict, "openai", image_data, video_data, audio_data, modalities
        )

        # Check that audio_data was extracted
        self.assertEqual(len(audio_data), 1)
        self.assertEqual(audio_data[0], "http://example.com/audio.mp3")

        # Check that content was normalized
        expected_content = [
            {"type": "text", "text": "Listen to this:"},
            {"type": "audio"},  # normalized from audio_url
        ]
        self.assertEqual(result["content"], expected_content)
```
**EN:** Test content processing with audio content. This test exercises `test_process_content_with_audio` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test content processing with audio content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_with_audio`。

### Lines 243-259: test case process content already string / 测试用例 process content already string
```python
    def test_process_content_already_string(self):
        """Test processing content that's already a string."""
        msg_dict = {"role": "user", "content": "Hello world"}

        image_data = []
        video_data = []
        audio_data = []
        modalities = []

        result = process_content_for_template_format(
            msg_dict, "openai", image_data, video_data, audio_data, modalities
        )

        # Should pass through unchanged
        self.assertEqual(result["content"], "Hello world")
        self.assertEqual(result["role"], "user")
        self.assertEqual(len(image_data), 0)
```
**EN:** Test processing content that's already a string. This test exercises `test_process_content_already_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test processing content that's already a string. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_already_string`。

### Lines 261-285: test case process content with modalities / 测试用例 process content with modalities
```python
    def test_process_content_with_modalities(self):
        """Test content processing with modalities field."""
        msg_dict = {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {"url": "http://example.com/image.jpg"},
                    "modalities": ["vision"],
                }
            ],
        }

        image_data = []
        video_data = []
        audio_data = []
        modalities = []

        result = process_content_for_template_format(
            msg_dict, "openai", image_data, video_data, audio_data, modalities
        )

        # Check that modalities was extracted
        self.assertEqual(len(modalities), 1)
        self.assertEqual(modalities[0], ["vision"])
```
**EN:** Test content processing with modalities field. This test exercises `test_process_content_with_modalities` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test content processing with modalities field. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_with_modalities`。

### Lines 287-307: test case process content filter none values / 测试用例 process content filter none values
```python
    def test_process_content_filter_none_values(self):
        """Test that None values are filtered out of processed messages."""
        msg_dict = {
            "role": "user",
            "content": "Hello",
            "name": None,
            "tool_call_id": None,
        }

        image_data = []
        video_data = []
        audio_data = []
        modalities = []

        result = process_content_for_template_format(
            msg_dict, "string", image_data, video_data, audio_data, modalities
        )

        # None values should be filtered out
        expected_keys = {"role", "content"}
        self.assertEqual(set(result.keys()), expected_keys)
```
**EN:** Test that None values are filtered out of processed messages. This test exercises `test_process_content_filter_none_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None values are filtered out of processed messages. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_filter_none_values`。

### Lines 309-327: test case process content with video / 测试用例 process content with video
```python
    def test_process_content_with_video(self):
        """Test content processing with video_url content."""
        msg_dict = {
            "role": "user",
            "content": [
                {"type": "text", "text": "Watch this:"},
                {"type": "video_url", "video_url": {"url": "http://example.com/v.mp4"}},
            ],
        }
        image_data = []
        video_data = []
        audio_data = []
        modalities = []
        result = process_content_for_template_format(
            msg_dict, "openai", image_data, video_data, audio_data, modalities
        )
        self.assertEqual(len(video_data), 1)
        self.assertEqual(video_data[0], "http://example.com/v.mp4")
        self.assertEqual(result["content"][1], {"type": "video"})
```
**EN:** Test content processing with video_url content. This test exercises `test_process_content_with_video` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test content processing with video_url content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_with_video`。

### Lines 329-352: test case process content video with max dynamic patch / 测试用例 process content video with max dynamic patch
```python
    def test_process_content_video_with_max_dynamic_patch(self):
        """Test video_url with max_dynamic_patch stores structured dict."""
        msg_dict = {
            "role": "user",
            "content": [
                {
                    "type": "video_url",
                    "video_url": {
                        "url": "http://example.com/v.mp4",
                        "max_dynamic_patch": 4,
                    },
                },
            ],
        }
        image_data = []
        video_data = []
        audio_data = []
        modalities = []
        result = process_content_for_template_format(
            msg_dict, "openai", image_data, video_data, audio_data, modalities
        )
        self.assertEqual(len(video_data), 1)
        self.assertIsInstance(video_data[0], dict)
        self.assertEqual(video_data[0]["max_dynamic_patch"], 4)
```
**EN:** Test video_url with max_dynamic_patch stores structured dict. This test exercises `test_process_content_video_with_max_dynamic_patch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test video_url with max_dynamic_patch stores structured dict. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_video_with_max_dynamic_patch`。

### Lines 354-383: test case process content v32 encoding / 测试用例 process content v32 encoding
```python
    def test_process_content_v32_encoding(self):
        """Test v32 encoding mode flattens text and ignores structured content parts."""
        msg_dict = {
            "role": "user",
            "content": [
                {"type": "text", "text": "Hello"},
                {
                    "type": "image_url",
                    "image_url": {"url": "http://example.com/img.jpg"},
                },
                {"type": "text", "text": "World"},
            ],
        }
        image_data = []
        video_data = []
        audio_data = []
        modalities = []
        result = process_content_for_template_format(
            msg_dict,
            "openai",
            image_data,
            video_data,
            audio_data,
            modalities,
            use_dpsk_v32_encoding=True,
        )
        # v32 encoding: content is joined text, not list
        self.assertEqual(result["content"], "Hello World")
        # Image data is still extracted
        self.assertEqual(len(image_data), 1)
```
**EN:** Test v32 encoding mode flattens text and ignores structured content parts. This test exercises `test_process_content_v32_encoding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test v32 encoding mode flattens text and ignores structured content parts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_v32_encoding`。

### Lines 385-394: test case process content invalid format raises / 测试用例 process content invalid format raises
```python
    def test_process_content_invalid_format_raises(self):
        """Test that invalid content_format raises ValueError."""
        msg_dict = {
            "role": "user",
            "content": [{"type": "text", "text": "Hi"}],
        }
        with self.assertRaises(ValueError):
            process_content_for_template_format(
                msg_dict, "invalid_format", [], [], [], []
            )
```
**EN:** Test that invalid content_format raises ValueError. This test exercises `test_process_content_invalid_format_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that invalid content_format raises ValueError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_invalid_format_raises`。

### Lines 396-416: test case process content video with modalities / 测试用例 process content video with modalities
```python
    def test_process_content_video_with_modalities(self):
        """Test that video content with modalities field is extracted."""
        msg_dict = {
            "role": "user",
            "content": [
                {
                    "type": "video_url",
                    "video_url": {"url": "http://example.com/v.mp4"},
                    "modalities": ["video"],
                },
            ],
        }
        image_data = []
        video_data = []
        audio_data = []
        modalities = []
        result = process_content_for_template_format(
            msg_dict, "openai", image_data, video_data, audio_data, modalities
        )
        self.assertEqual(len(modalities), 1)
        self.assertEqual(modalities[0], ["video"])
```
**EN:** Test that video content with modalities field is extracted. This test exercises `test_process_content_video_with_modalities` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that video content with modalities field is extracted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_video_with_modalities`。

### Lines 418-429: test case detect template with filter / 测试用例 detect template with filter
```python
    def test_detect_template_with_filter(self):
        """Test that content access through a Jinja filter is detected as openai."""
        # Template with | trim filter on content iteration
        template = """
{%- for message in messages %}
    {%- for content in message['content'] | trim %}
        {{- content }}
    {%- endfor %}
{%- endfor %}
        """
        result = detect_jinja_template_content_format(template)
        self.assertEqual(result, "openai")
```
**EN:** Test that content access through a Jinja filter is detected as openai. This test exercises `test_detect_template_with_filter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that content access through a Jinja filter is detected as openai. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_template_with_filter`。

### Lines 431-446: test case detect template with is test / 测试用例 detect template with is test
```python
    def test_detect_template_with_is_test(self):
        """Test that 'is string' test on content triggers openai detection."""
        # Template with 'is string' test that also iterates content
        template = """
{%- for message in messages %}
    {%- if message['content'] is string %}
        {{- message['content'] }}
    {%- else %}
        {%- for item in message['content'] %}
            {{- item }}
        {%- endfor %}
    {%- endif %}
{%- endfor %}
        """
        result = detect_jinja_template_content_format(template)
        self.assertEqual(result, "openai")
```
**EN:** Test that 'is string' test on content triggers openai detection. This test exercises `test_detect_template_with_is_test` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that 'is string' test on content triggers openai detection. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_template_with_is_test`。

### Lines 448-458: test case detect template with slice / 测试用例 detect template with slice
```python
    def test_detect_template_with_slice(self):
        """Test that content access through slice is detected as openai."""
        template = """
{%- for message in messages %}
    {%- for item in message['content'][:5] %}
        {{- item }}
    {%- endfor %}
{%- endfor %}
        """
        result = detect_jinja_template_content_format(template)
        self.assertEqual(result, "openai")
```
**EN:** Test that content access through slice is detected as openai. This test exercises `test_detect_template_with_slice` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that content access through slice is detected as openai. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_template_with_slice`。

### Lines 460-469: test case detect template no content loop is string / 测试用例 detect template no content loop is string
```python
    def test_detect_template_no_content_loop_is_string(self):
        """Test that template without content iteration returns string format."""
        template = """
{%- for message in messages %}
    {{- message['role'] }}: {{ message['content'] }}
{%- endfor %}
        """
        # No "image"/"audio"/"video" keyword, no content loop → string
        result = detect_jinja_template_content_format(template)
        self.assertEqual(result, "string")
```
**EN:** Test that template without content iteration returns string format. This test exercises `test_detect_template_no_content_loop_is_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that template without content iteration returns string format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_template_no_content_loop_is_string`。

### Lines 471-487: test case detect msg content without multimodal keywords / 测试用例 detect msg content without multimodal keywords
```python
    def test_detect_msg_content_without_multimodal_keywords(self):
        """Test AST detection of 'for item in msg.content' without keyword shortcut.
        Templates that contain 'image'/'video'/'audio'/'vision' take a shortcut.
        This template deliberately avoids those keywords to test the AST path."""
        template = """
{%- for msg in messages %}
    {%- if msg.content is string %}
        {{- msg.content }}
    {%- else %}
        {%- for item in msg.content %}
            {{- item.text }}
        {%- endfor %}
    {%- endif %}
{%- endfor %}
        """
        result = detect_jinja_template_content_format(template)
        self.assertEqual(result, "openai")
```
**EN:** Test AST detection of 'for item in msg.content' without keyword shortcut. This test exercises `test_detect_msg_content_without_multimodal_keywords` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test AST detection of 'for item in msg.content' without keyword shortcut. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_msg_content_without_multimodal_keywords`。

### Lines 490-491: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestTemplateContentFormatDetection`: Test template content format detection functionality. / 用于组织相关测试、夹具或辅助方法。
- `TestTemplateContentFormatDetection.test_detect_llama4_openai_format`: Test detection of llama4-style template (should be 'openai' format). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_llama4_openai_format`。
- `TestTemplateContentFormatDetection.test_detect_deepseek_string_format`: Test detection of deepseek-style template (should be 'string' format). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_deepseek_string_format`。
- `TestTemplateContentFormatDetection.test_detect_invalid_template`: Test handling of invalid template (should default to 'string'). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_invalid_template`。
- `TestTemplateContentFormatDetection.test_detect_empty_template`: Test handling of empty template (should default to 'string'). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_empty_template`。
- `TestTemplateContentFormatDetection.test_detect_msg_content_pattern`: Test detection of template with msg.content pattern (should be 'openai' format). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_msg_content_pattern`。
- `TestTemplateContentFormatDetection.test_detect_m_content_pattern`: Test detection of template with m.content pattern (should be 'openai' format). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_m_content_pattern`。
- `TestTemplateContentFormatDetection.test_process_content_openai_format`: Test content processing for openai format. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_openai_format`。
- `TestTemplateContentFormatDetection.test_process_content_string_format`: Test content processing for string format. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_string_format`。
- `TestTemplateContentFormatDetection.test_process_content_with_audio`: Test content processing with audio content. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_with_audio`。
- `TestTemplateContentFormatDetection.test_process_content_already_string`: Test processing content that's already a string. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_already_string`。
- `TestTemplateContentFormatDetection.test_process_content_with_modalities`: Test content processing with modalities field. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_content_with_modalities`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.parser.jinja_template_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 491
