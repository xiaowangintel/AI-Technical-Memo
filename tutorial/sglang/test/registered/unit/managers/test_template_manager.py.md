# test_template_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_template_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates template manager behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 template manager 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

from sglang.srt.managers.template_detection import (
    ReasoningToggleConfig,
    detect_reasoning_parser,
    detect_reasoning_pattern,
    detect_tool_call_parser,
    resolve_auto_parsers,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `sglang.srt.managers.template_detection`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `sglang.srt.managers.template_detection`, `sglang.test.ci.ci_register`。

### Lines 13-13: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(2.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 16-16: class _DummyTokenizer declaration / 类 _DummyTokenizer 声明
```python
class _DummyTokenizer:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 17-18: method init / 方法 init
```python
    def __init__(self, vocab):
        self._vocab = vocab
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 20-21: helper routine get vocab / 辅助流程 get vocab
```python
    def get_vocab(self):
        return {token: i for i, token in enumerate(self._vocab)}
```
**EN:** This helper encapsulates `get_vocab` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_vocab`，以便周围测试复用准备、执行或校验逻辑。

### Lines 24-25: class TestTemplateManagerReasoningDetection declaration / 类 TestTemplateManagerReasoningDetection 声明
```python
class TestTemplateManagerReasoningDetection(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 26-31: method detect / 方法 detect
```python
    def _detect(self, template, vocab):
        force, config = detect_reasoning_pattern(template)
        parser = detect_reasoning_parser(
            template, _DummyTokenizer(vocab), config, force
        )
        return force, config, parser
```
**EN:** This block implements `_detect` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_detect`，承担模块行为中的一个聚焦逻辑片段。

### Lines 33-47: test case qwen3 template not misclassified as glm45 / 测试用例 qwen3 template not misclassified as glm45
```python
    def test_qwen3_template_not_misclassified_as_glm45(self):
        template = """
        {% set enable_thinking = enable_thinking if enable_thinking is defined else true %}
        {% if '</think>' in content %}
        <tool_call>
        """
        _, config, parser = self._detect(
            template, ["<tool_call>", "<|endoftext|>", "</think>"]
        )

        self.assertEqual(
            config,
            ReasoningToggleConfig(toggle_param="enable_thinking", default_enabled=True),
        )
        self.assertEqual(parser, "qwen3")
```
**EN:** This test exercises `test_qwen3_template_not_misclassified_as_glm45` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_template_not_misclassified_as_glm45`。

### Lines 49-64: test case glm45 requires glm specific template markers / 测试用例 glm45 requires glm specific template markers
```python
    def test_glm45_requires_glm_specific_template_markers(self):
        template = """
        [gMASK]<sop>
        {% set enable_thinking = enable_thinking if enable_thinking is defined else true %}
        /nothink
        <tool_call>
        """
        _, config, parser = self._detect(
            template, ["<tool_call>", "<|endoftext|>", "<|user|>"]
        )

        self.assertEqual(
            config,
            ReasoningToggleConfig(toggle_param="enable_thinking", default_enabled=True),
        )
        self.assertEqual(parser, "glm45")
```
**EN:** This test exercises `test_glm45_requires_glm_specific_template_markers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_glm45_requires_glm_specific_template_markers`。

### Lines 66-77: test case interns1 detects enable thinking default true / 测试用例 interns1 detects enable thinking default true
```python
    def test_interns1_detects_enable_thinking_default_true(self):
        template = """
        {% set default_thinking_sys %}...<think>...</think>{% endset %}
        {% if enable_thinking is not defined or enable_thinking %}
        """
        _, config, parser = self._detect(template, ["<|endoftext|>"])

        self.assertEqual(
            config,
            ReasoningToggleConfig(toggle_param="enable_thinking", default_enabled=True),
        )
        self.assertEqual(parser, "interns1")
```
**EN:** This test exercises `test_interns1_detects_enable_thinking_default_true` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_interns1_detects_enable_thinking_default_true`。

### Lines 79-90: test case nemotron detects uppercase true assignment / 测试用例 nemotron detects uppercase true assignment
```python
    def test_nemotron_detects_uppercase_true_assignment(self):
        template = """
        {% set enable_thinking = enable_thinking if enable_thinking is defined else True %}
        {% set truncate_history_thinking = truncate_history_thinking if truncate_history_thinking is defined else True %}
        """
        _, config, parser = self._detect(template, ["<|endoftext|>"])

        self.assertEqual(
            config,
            ReasoningToggleConfig(toggle_param="enable_thinking", default_enabled=True),
        )
        self.assertEqual(parser, "nemotron_3")
```
**EN:** This test exercises `test_nemotron_detects_uppercase_true_assignment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nemotron_detects_uppercase_true_assignment`。

### Lines 92-99: test case minimax uses template signature without toggle config / 测试用例 minimax uses template signature without toggle config
```python
    def test_minimax_uses_template_signature_without_toggle_config(self):
        template = """
        {%- set toolcall_begin_token = '<minimax:tool_call>' -%}
        """
        _, config, parser = self._detect(template, ["<minimax:tool_call>"])

        self.assertIsNone(config)
        self.assertEqual(parser, "minimax")
```
**EN:** This test exercises `test_minimax_uses_template_signature_without_toggle_config` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_minimax_uses_template_signature_without_toggle_config`。

### Lines 102-102: class TestTemplateDetectionRuleMatrix declaration / 类 TestTemplateDetectionRuleMatrix 声明
```python
class TestTemplateDetectionRuleMatrix(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 103-103: supporting statements / 辅助语句
```python
    """Table-driven tests for REASONING_PARSER_RULES and REASONING_MODE_RULES."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 105-112: method detect / 方法 detect
```python
    def _detect(self, template, vocab=None):
        if vocab is None:
            vocab = []
        force, config = detect_reasoning_pattern(template)
        parser = detect_reasoning_parser(
            template, _DummyTokenizer(vocab), config, force
        )
        return force, config, parser
```
**EN:** This block implements `_detect` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_detect`，承担模块行为中的一个聚焦逻辑片段。

### Lines 114-174: class-level constants and configuration for `TestTemplateDetectionRuleMatrix` / 类级常量与配置
```python
    PARSER_RULES_MATRIX = [
        # (name, template_snippet, vocab, expected_parser, expected_toggle_param)
        (
            "deepseek_r1_think_tags",
            "<think>\nLet me reason about this\n</think>\nAnswer here",
            [],
            "deepseek-r1",
            None,  # matched by deepseek_r1_think_tags rule (has <think> text)
        ),
        (
            "deepseek_v3",
            "{% if not thinking is defined %}{% set thinking = false %}{% endif %}\n"
            "<think>",
            [],
            "deepseek-v3",
            "thinking",
        ),
        (
            "qwen3_enable_thinking_true",
            "{% set enable_thinking = enable_thinking if enable_thinking is defined else true %}\n",
            [],
            "qwen3",
            "enable_thinking",
        ),
        (
            "kimi_unicode_markers",
            "\u25c1think\u25b7some text\u25c1/think\u25b7",
            [],
            "kimi",
            None,
        ),
        (
            "mistral_reasoning_effort",
            "{% if reasoning_effort %}[THINK]{% endif %}",
            [],
            "mistral",
            None,  # special_case="mistral"
        ),
        (
            "gpt_oss_channel",
            "<|channel|>analysis<|message|>",
            [],
            "gpt-oss",
            None,  # special_case="always"
        ),
        (
            "kimi_k2_with_tool_vocab",
            "{% set thinking = thinking if thinking is defined else true %}\n<think>",
            ["<|tool_calls_section_begin|>", "<|tool_calls_section_end|>"],
            "kimi_k2",
            "thinking",
        ),
        (
            "mimo_enable_thinking_false",
            "{% if not enable_thinking is defined %}{% set enable_thinking = false %}{% endif %}\n"
            "enable_thinking",
            [],
            "mimo",
            "enable_thinking",
        ),
    ]
```
**EN:** This block defines shared names such as `PARSER_RULES_MATRIX`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `PARSER_RULES_MATRIX` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 176-200: test case parser rules matrix / 测试用例 parser rules matrix
```python
    def test_parser_rules_matrix(self):
        for (
            name,
            template,
            vocab,
            expected_parser,
            expected_toggle,
        ) in self.PARSER_RULES_MATRIX:
            with self.subTest(name=name):
                _, config, parser = self._detect(template, vocab)
                self.assertEqual(
                    parser,
                    expected_parser,
                    f"Rule '{name}': expected parser '{expected_parser}', got '{parser}'",
                )
                if expected_toggle is not None:
                    self.assertIsNotNone(
                        config, f"Rule '{name}': expected config, got None"
                    )
                    self.assertEqual(
                        config.toggle_param,
                        expected_toggle,
                        f"Rule '{name}': expected toggle '{expected_toggle}', "
                        f"got '{config.toggle_param}'",
                    )
```
**EN:** This test exercises `test_parser_rules_matrix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parser_rules_matrix`。

### Lines 202-207: test case unrecognized template returns none / 测试用例 unrecognized template returns none
```python
    def test_unrecognized_template_returns_none(self):
        template = "Hello {{ user_message }}, how can I help you?"
        _, config, parser = self._detect(template)

        self.assertIsNone(config)
        self.assertIsNone(parser)
```
**EN:** This test exercises `test_unrecognized_template_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unrecognized_template_returns_none`。

### Lines 209-213: test case empty template returns none / 测试用例 empty template returns none
```python
    def test_empty_template_returns_none(self):
        _, config, parser = self._detect("")

        self.assertIsNone(config)
        self.assertIsNone(parser)
```
**EN:** This test exercises `test_empty_template_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_template_returns_none`。

### Lines 215-222: test case qwen3 precedence over deepseek r1 / 测试用例 qwen3 precedence over deepseek r1
```python
    def test_qwen3_precedence_over_deepseek_r1(self):
        """Template with enable_thinking=true but no <think> tag should be qwen3, not deepseek_r1."""
        template = "{% set enable_thinking = enable_thinking if enable_thinking is defined else true %}"
        _, config, parser = self._detect(template)

        self.assertEqual(parser, "qwen3")
        self.assertEqual(config.toggle_param, "enable_thinking")
        self.assertTrue(config.default_enabled)
```
**EN:** Template with enable_thinking=true but no <think> tag should be qwen3, not deepseek_r1. This test exercises `test_qwen3_precedence_over_deepseek_r1` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Template with enable_thinking=true but no <think> tag should be qwen3, not deepseek_r1. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_precedence_over_deepseek_r1`。

### Lines 225-225: class TestToolCallParserDetection declaration / 类 TestToolCallParserDetection 声明
```python
class TestToolCallParserDetection(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 226-226: supporting statements / 辅助语句
```python
    """Tests for detect_tool_call_parser() using real model tokenizers."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 228-236: method detect all / 方法 detect all
```python
    def _detect_all(self, model_name):
        from transformers import AutoTokenizer

        tok = AutoTokenizer.from_pretrained(model_name, trust_remote_code=True)
        template = tok.chat_template
        force, config = detect_reasoning_pattern(template)
        rp = detect_reasoning_parser(template, tok, config, force)
        tcp = detect_tool_call_parser(template, tok, config, force)
        return rp, tcp
```
**EN:** This block implements `_detect_all` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_detect_all`，承担模块行为中的一个聚焦逻辑片段。

### Lines 238-241: test case qwen3 detects qwen tool call parser / 测试用例 qwen3 detects qwen tool call parser
```python
    def test_qwen3_detects_qwen_tool_call_parser(self):
        rp, tcp = self._detect_all("Qwen/Qwen3-0.6B")
        self.assertEqual(rp, "qwen3")
        self.assertEqual(tcp, "qwen")
```
**EN:** This test exercises `test_qwen3_detects_qwen_tool_call_parser` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_detects_qwen_tool_call_parser`。

### Lines 243-275: test case tool call parser rule values via snippets / 测试用例 tool call parser rule values via snippets
```python
    def test_tool_call_parser_rule_values_via_snippets(self):
        """Table-driven: verify tool-call rule values differ from reasoning where expected."""
        cases = [
            # (name, template, vocab, expected_tool_call)
            (
                "qwen_maps_from_qwen3_config",
                "{% set enable_thinking = enable_thinking if enable_thinking is defined else true %}",
                [],
                "qwen",
            ),
            ("gpt_oss", "<|channel|>analysis<|message|>", [], "gpt-oss"),
            ("gemma4", "<|channel>content", [], "gemma4"),
            ("minimax_maps_to_m2", "<minimax:tool_call>", [], "minimax-m2"),
            (
                "deepseekv3",
                "{% if not thinking is defined %}{% set thinking = false %}{% endif %}",
                [],
                "deepseekv3",
            ),
            (
                "kimi_k2",
                "{% set thinking = thinking if thinking is defined else true %}\n<think>",
                ["<|tool_calls_section_begin|>"],
                "kimi_k2",
            ),
        ]
        for name, template, vocab, expected in cases:
            with self.subTest(name=name):
                force, config = detect_reasoning_pattern(template)
                result = detect_tool_call_parser(
                    template, _DummyTokenizer(vocab), config, force
                )
                self.assertEqual(result, expected)
```
**EN:** Table-driven: verify tool-call rule values differ from reasoning where expected. This test exercises `test_tool_call_parser_rule_values_via_snippets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Table-driven: verify tool-call rule values differ from reasoning where expected. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_parser_rule_values_via_snippets`。

### Lines 277-278: test case none template returns none / 测试用例 none template returns none
```python
    def test_none_template_returns_none(self):
        self.assertIsNone(detect_tool_call_parser(None, None))
```
**EN:** This test exercises `test_none_template_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_template_returns_none`。

### Lines 280-283: test case unrecognized template returns none / 测试用例 unrecognized template returns none
```python
    def test_unrecognized_template_returns_none(self):
        force, config = detect_reasoning_pattern("Hello {{ user }}")
        result = detect_tool_call_parser("Hello {{ user }}", None, config, force)
        self.assertIsNone(result)
```
**EN:** This test exercises `test_unrecognized_template_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unrecognized_template_returns_none`。

### Lines 286-286: class TestResolveAutoParsers declaration / 类 TestResolveAutoParsers 声明
```python
class TestResolveAutoParsers(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 287-287: supporting statements / 辅助语句
```python
    """Tests for resolve_auto_parsers() using real model tokenizers."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 289-295: method make server args / 方法 make server args
```python
    def _make_server_args(self, reasoning_parser=None, tool_call_parser=None):
        return SimpleNamespace(
            reasoning_parser=reasoning_parser,
            tool_call_parser=tool_call_parser,
            model_path="Qwen/Qwen3-0.6B",
            trust_remote_code=False,
        )
```
**EN:** This block implements `_make_server_args` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 297-301: test case resolves both parsers with real model / 测试用例 resolves both parsers with real model
```python
    def test_resolves_both_parsers_with_real_model(self):
        args = self._make_server_args(reasoning_parser="auto", tool_call_parser="auto")
        resolve_auto_parsers(args)
        self.assertEqual(args.reasoning_parser, "qwen3")
        self.assertEqual(args.tool_call_parser, "qwen")
```
**EN:** This test exercises `test_resolves_both_parsers_with_real_model` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_resolves_both_parsers_with_real_model`。

### Lines 303-307: test case resolves reasoning parser only / 测试用例 resolves reasoning parser only
```python
    def test_resolves_reasoning_parser_only(self):
        args = self._make_server_args(reasoning_parser="auto", tool_call_parser=None)
        resolve_auto_parsers(args)
        self.assertEqual(args.reasoning_parser, "qwen3")
        self.assertIsNone(args.tool_call_parser)
```
**EN:** This test exercises `test_resolves_reasoning_parser_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_resolves_reasoning_parser_only`。

### Lines 309-313: test case resolves tool call parser only / 测试用例 resolves tool call parser only
```python
    def test_resolves_tool_call_parser_only(self):
        args = self._make_server_args(reasoning_parser="qwen3", tool_call_parser="auto")
        resolve_auto_parsers(args)
        self.assertEqual(args.reasoning_parser, "qwen3")
        self.assertEqual(args.tool_call_parser, "qwen")
```
**EN:** This test exercises `test_resolves_tool_call_parser_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_resolves_tool_call_parser_only`。

### Lines 315-319: test case neither auto is noop / 测试用例 neither auto is noop
```python
    def test_neither_auto_is_noop(self):
        args = self._make_server_args(reasoning_parser="qwen3", tool_call_parser="qwen")
        resolve_auto_parsers(args)
        self.assertEqual(args.reasoning_parser, "qwen3")
        self.assertEqual(args.tool_call_parser, "qwen")
```
**EN:** This test exercises `test_neither_auto_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_neither_auto_is_noop`。

### Lines 321-330: test case nonexistent model disables both parsers / 测试用例 nonexistent model disables both parsers
```python
    def test_nonexistent_model_disables_both_parsers(self):
        args = SimpleNamespace(
            reasoning_parser="auto",
            tool_call_parser="auto",
            model_path="nonexistent/model-does-not-exist-xyz",
            trust_remote_code=False,
        )
        resolve_auto_parsers(args)
        self.assertIsNone(args.reasoning_parser)
        self.assertIsNone(args.tool_call_parser)
```
**EN:** This test exercises `test_nonexistent_model_disables_both_parsers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nonexistent_model_disables_both_parsers`。

### Lines 333-334: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_DummyTokenizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTemplateManagerReasoningDetection`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTemplateDetectionRuleMatrix`: Table-driven tests for REASONING_PARSER_RULES and REASONING_MODE_RULES. / 用于组织相关测试、夹具或辅助方法。
- `TestToolCallParserDetection`: Tests for detect_tool_call_parser() using real model tokenizers. / 用于组织相关测试、夹具或辅助方法。
- `TestResolveAutoParsers`: Tests for resolve_auto_parsers() using real model tokenizers. / 用于组织相关测试、夹具或辅助方法。
- `_DummyTokenizer.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_DummyTokenizer.get_vocab`: This helper encapsulates `get_vocab` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_vocab`，以便周围测试复用准备、执行或校验逻辑。
- `TestTemplateManagerReasoningDetection._detect`: This block implements `_detect` and captures one focused piece of the module's behavior. / 该代码块实现 `_detect`，承担模块行为中的一个聚焦逻辑片段。
- `TestTemplateManagerReasoningDetection.test_qwen3_template_not_misclassified_as_glm45`: This test exercises `test_qwen3_template_not_misclassified_as_glm45` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_template_not_misclassified_as_glm45`。
- `TestTemplateManagerReasoningDetection.test_glm45_requires_glm_specific_template_markers`: This test exercises `test_glm45_requires_glm_specific_template_markers` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_glm45_requires_glm_specific_template_markers`。
- `TestTemplateManagerReasoningDetection.test_interns1_detects_enable_thinking_default_true`: This test exercises `test_interns1_detects_enable_thinking_default_true` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_interns1_detects_enable_thinking_default_true`。
- `TestTemplateManagerReasoningDetection.test_nemotron_detects_uppercase_true_assignment`: This test exercises `test_nemotron_detects_uppercase_true_assignment` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nemotron_detects_uppercase_true_assignment`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.srt.managers.template_detection`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 334
