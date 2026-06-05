# test_code_completion_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/parser/test_code_completion_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates code completion parser behavior in SGLang's unit / parser area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 解析器 领域中与 code completion parser 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/parser/code_completion_parser.py"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-19: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import patch

from sglang.srt.entrypoints.openai.protocol import CompletionRequest
from sglang.srt.parser.code_completion_parser import (
    CompletionTemplate,
    FimPosition,
    completion_template_exists,
    completion_templates,
    generate_completion_prompt,
    generate_completion_prompt_from_request,
    is_completion_template_defined,
    register_completion_template,
    set_completion_template,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.parser.code_completion_parser`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.parser.code_completion_parser`。

### Lines 21-21: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestFimPosition declaration / 类 TestFimPosition 声明
```python
class TestFimPosition(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 25-27: test case middle and end are distinct / 测试用例 middle and end are distinct
```python
    def test_middle_and_end_are_distinct(self):
        """Test that MIDDLE and END are different enum values."""
        self.assertNotEqual(FimPosition.MIDDLE, FimPosition.END)
```
**EN:** Test that MIDDLE and END are different enum values. This test exercises `test_middle_and_end_are_distinct` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that MIDDLE and END are different enum values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_middle_and_end_are_distinct`。

### Lines 30-30: class TestCompletionTemplate declaration / 类 TestCompletionTemplate 声明
```python
class TestCompletionTemplate(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 31-42: test case dataclass fields / 测试用例 dataclass fields
```python
    def test_dataclass_fields(self):
        """Test creating a CompletionTemplate with all fields."""
        t = CompletionTemplate(
            name="test",
            fim_begin_token="<begin>",
            fim_middle_token="<middle>",
            fim_end_token="<end>",
            fim_position=FimPosition.MIDDLE,
        )
        self.assertEqual(t.name, "test")
        self.assertEqual(t.fim_begin_token, "<begin>")
        self.assertEqual(t.fim_position, FimPosition.MIDDLE)
```
**EN:** Test creating a CompletionTemplate with all fields. This test exercises `test_dataclass_fields` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test creating a CompletionTemplate with all fields. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dataclass_fields`。

### Lines 45-45: class TestRegisterCompletionTemplate declaration / 类 TestRegisterCompletionTemplate 声明
```python
class TestRegisterCompletionTemplate(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 46-50: test case builtin templates registered / 测试用例 builtin templates registered
```python
    def test_builtin_templates_registered(self):
        """Test that deepseek_coder, star_coder, qwen_coder are pre-registered."""
        self.assertTrue(completion_template_exists("deepseek_coder"))
        self.assertTrue(completion_template_exists("star_coder"))
        self.assertTrue(completion_template_exists("qwen_coder"))
```
**EN:** Test that deepseek_coder, star_coder, qwen_coder are pre-registered. This test exercises `test_builtin_templates_registered` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that deepseek_coder, star_coder, qwen_coder are pre-registered. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_builtin_templates_registered`。

### Lines 52-54: test case unregistered template not found / 测试用例 unregistered template not found
```python
    def test_unregistered_template_not_found(self):
        """Test that a non-existent template returns False."""
        self.assertFalse(completion_template_exists("nonexistent_template"))
```
**EN:** Test that a non-existent template returns False. This test exercises `test_unregistered_template_not_found` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a non-existent template returns False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unregistered_template_not_found`。

### Lines 56-68: test case register new template / 测试用例 register new template
```python
    def test_register_new_template(self):
        """Test registering a new template."""
        t = CompletionTemplate(
            name="_test_new_template",
            fim_begin_token="<b>",
            fim_middle_token="<m>",
            fim_end_token="<e>",
            fim_position=FimPosition.END,
        )
        register_completion_template(t)
        self.assertTrue(completion_template_exists("_test_new_template"))
        # Cleanup
        del completion_templates["_test_new_template"]
```
**EN:** Test registering a new template. This test exercises `test_register_new_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test registering a new template. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_new_template`。

### Lines 70-81: test case register duplicate raises / 测试用例 register duplicate raises
```python
    def test_register_duplicate_raises(self):
        """Test that registering a duplicate name without override raises."""
        with self.assertRaises(AssertionError):
            register_completion_template(
                CompletionTemplate(
                    name="deepseek_coder",
                    fim_begin_token="x",
                    fim_middle_token="y",
                    fim_end_token="z",
                    fim_position=FimPosition.MIDDLE,
                )
            )
```
**EN:** Test that registering a duplicate name without override raises. This test exercises `test_register_duplicate_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that registering a duplicate name without override raises. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_duplicate_raises`。

### Lines 83-102: test case register duplicate with override / 测试用例 register duplicate with override
```python
    def test_register_duplicate_with_override(self):
        """Test that override=True allows re-registration."""
        original = completion_templates["deepseek_coder"]
        try:
            register_completion_template(
                CompletionTemplate(
                    name="deepseek_coder",
                    fim_begin_token="<new>",
                    fim_middle_token="<new_m>",
                    fim_end_token="<new_e>",
                    fim_position=FimPosition.END,
                ),
                override=True,
            )
            self.assertEqual(
                completion_templates["deepseek_coder"].fim_begin_token, "<new>"
            )
        finally:
            # Restore original
            completion_templates["deepseek_coder"] = original
```
**EN:** Test that override=True allows re-registration. This test exercises `test_register_duplicate_with_override` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that override=True allows re-registration. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_duplicate_with_override`。

### Lines 105-105: class TestGenerateCompletionPrompt declaration / 类 TestGenerateCompletionPrompt 声明
```python
class TestGenerateCompletionPrompt(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 106-113: test case deepseek coder middle position / 测试用例 deepseek coder middle position
```python
    def test_deepseek_coder_middle_position(self):
        """Test FIM prompt with MIDDLE position (deepseek_coder style)."""
        result = generate_completion_prompt(
            "prefix_code", "suffix_code", "deepseek_coder"
        )
        t = completion_templates["deepseek_coder"]
        expected = f"{t.fim_begin_token}prefix_code{t.fim_middle_token}suffix_code{t.fim_end_token}"
        self.assertEqual(result, expected)
```
**EN:** Test FIM prompt with MIDDLE position (deepseek_coder style). This test exercises `test_deepseek_coder_middle_position` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test FIM prompt with MIDDLE position (deepseek_coder style). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_coder_middle_position`。

### Lines 115-120: test case star coder end position / 测试用例 star coder end position
```python
    def test_star_coder_end_position(self):
        """Test FIM prompt with END position (star_coder style)."""
        result = generate_completion_prompt("prefix_code", "suffix_code", "star_coder")
        t = completion_templates["star_coder"]
        expected = f"{t.fim_begin_token}prefix_code{t.fim_end_token}suffix_code{t.fim_middle_token}"
        self.assertEqual(result, expected)
```
**EN:** Test FIM prompt with END position (star_coder style). This test exercises `test_star_coder_end_position` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test FIM prompt with END position (star_coder style). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_star_coder_end_position`。

### Lines 122-129: test case qwen coder end position / 测试用例 qwen coder end position
```python
    def test_qwen_coder_end_position(self):
        """Test FIM prompt with END position (qwen_coder style)."""
        result = generate_completion_prompt("prefix", "suffix", "qwen_coder")
        t = completion_templates["qwen_coder"]
        expected = (
            f"{t.fim_begin_token}prefix{t.fim_end_token}suffix{t.fim_middle_token}"
        )
        self.assertEqual(result, expected)
```
**EN:** Test FIM prompt with END position (qwen_coder style). This test exercises `test_qwen_coder_end_position` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test FIM prompt with END position (qwen_coder style). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen_coder_end_position`。

### Lines 131-136: test case empty prompt and suffix / 测试用例 empty prompt and suffix
```python
    def test_empty_prompt_and_suffix(self):
        """Test FIM prompt generation with empty strings."""
        result = generate_completion_prompt("", "", "deepseek_coder")
        t = completion_templates["deepseek_coder"]
        expected = f"{t.fim_begin_token}{t.fim_middle_token}{t.fim_end_token}"
        self.assertEqual(result, expected)
```
**EN:** Test FIM prompt generation with empty strings. This test exercises `test_empty_prompt_and_suffix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test FIM prompt generation with empty strings. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_prompt_and_suffix`。

### Lines 139-139: class TestGenerateCompletionPromptFromRequest declaration / 类 TestGenerateCompletionPromptFromRequest 声明
```python
class TestGenerateCompletionPromptFromRequest(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 140-144: test case empty suffix returns prompt directly / 测试用例 empty suffix returns prompt directly
```python
    def test_empty_suffix_returns_prompt_directly(self):
        """Test that empty suffix bypasses FIM formatting."""
        request = CompletionRequest(prompt="just code", suffix="")
        result = generate_completion_prompt_from_request(request)
        self.assertEqual(result, "just code")
```
**EN:** Test that empty suffix bypasses FIM formatting. This test exercises `test_empty_suffix_returns_prompt_directly` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that empty suffix bypasses FIM formatting. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_suffix_returns_prompt_directly`。

### Lines 146-158: test case nonempty suffix uses fim template / 测试用例 nonempty suffix uses fim template
```python
    def test_nonempty_suffix_uses_fim_template(self):
        """Test that non-empty suffix triggers FIM formatting."""
        with patch(
            "sglang.srt.parser.code_completion_parser.completion_template_name",
            "deepseek_coder",
        ):
            request = CompletionRequest(prompt="prefix", suffix="suffix")
            result = generate_completion_prompt_from_request(request)
            t = completion_templates["deepseek_coder"]
            expected = (
                f"{t.fim_begin_token}prefix{t.fim_middle_token}suffix{t.fim_end_token}"
            )
            self.assertEqual(result, expected)
```
**EN:** Test that non-empty suffix triggers FIM formatting. This test exercises `test_nonempty_suffix_uses_fim_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that non-empty suffix triggers FIM formatting. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nonempty_suffix_uses_fim_template`。

### Lines 161-161: class TestSetCompletionTemplate declaration / 类 TestSetCompletionTemplate 声明
```python
class TestSetCompletionTemplate(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 162-171: test case set only once / 测试用例 set only once
```python
    def test_set_only_once(self):
        """Test that set_completion_template only sets the name once."""
        import sglang.srt.parser.code_completion_parser as module

        with patch.object(module, "completion_template_name", None):
            set_completion_template("star_coder")
            self.assertEqual(module.completion_template_name, "star_coder")
            # Second call should be ignored
            set_completion_template("qwen_coder")
            self.assertEqual(module.completion_template_name, "star_coder")
```
**EN:** Test that set_completion_template only sets the name once. This test exercises `test_set_only_once` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that set_completion_template only sets the name once. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_only_once`。

### Lines 173-184: test case is completion template defined / 测试用例 is completion template defined
```python
    def test_is_completion_template_defined(self):
        """Test the defined check before and after setting."""
        import sglang.srt.parser.code_completion_parser as module

        old_name = module.completion_template_name
        try:
            module.completion_template_name = None
            self.assertFalse(is_completion_template_defined())
            set_completion_template("deepseek_coder")
            self.assertTrue(is_completion_template_defined())
        finally:
            module.completion_template_name = old_name
```
**EN:** Test the defined check before and after setting. This test exercises `test_is_completion_template_defined` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the defined check before and after setting. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_completion_template_defined`。

### Lines 187-188: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestFimPosition`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCompletionTemplate`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRegisterCompletionTemplate`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGenerateCompletionPrompt`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGenerateCompletionPromptFromRequest`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSetCompletionTemplate`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFimPosition.test_middle_and_end_are_distinct`: Test that MIDDLE and END are different enum values. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_middle_and_end_are_distinct`。
- `TestCompletionTemplate.test_dataclass_fields`: Test creating a CompletionTemplate with all fields. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dataclass_fields`。
- `TestRegisterCompletionTemplate.test_builtin_templates_registered`: Test that deepseek_coder, star_coder, qwen_coder are pre-registered. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_builtin_templates_registered`。
- `TestRegisterCompletionTemplate.test_unregistered_template_not_found`: Test that a non-existent template returns False. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unregistered_template_not_found`。
- `TestRegisterCompletionTemplate.test_register_new_template`: Test registering a new template. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_new_template`。
- `TestRegisterCompletionTemplate.test_register_duplicate_raises`: Test that registering a duplicate name without override raises. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_duplicate_raises`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.parser.code_completion_parser`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 188
