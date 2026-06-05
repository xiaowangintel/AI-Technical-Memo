# test_harmony_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/parser/test_harmony_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers OpenAI-compatible serving. The file defines 73 test(s), 1 fixture(s), and 11 helper/class block(s) to validate this area. / [CN] 该文件覆盖OpenAI 兼容服务。它定义了 73 个测试、1 个 fixture，以及 11 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L21)
```python
import pytest
from openai_harmony import Message, Role

from tests.entrypoints.openai.utils import verify_harmony_messages
from vllm.entrypoints.openai.parser.harmony_utils import (
    auto_drop_analysis_messages,
    extract_function_from_recipient,
    get_encoding,
    get_system_message,
    has_custom_tools,
    is_function_recipient,
    parse_chat_input_to_harmony_message,
    parse_chat_output,
)
from vllm.entrypoints.openai.responses.harmony import (
    response_input_to_harmony,
    response_previous_input_to_harmony,
)
```
**EN:** Imports third-party packages like `openai_harmony.Message`, `openai_harmony.Role`, `pytest`, project helpers such as `tests.entrypoints.openai.utils.verify_harmony_messages`, `vllm.entrypoints.openai.parser.harmony_utils.auto_drop_analysis_messages`, `vllm.entrypoints.openai.parser.harmony_utils.extract_function_from_recipient`.
**CN:** 导入第三方包（如 `openai_harmony.Message`、`openai_harmony.Role`、`pytest`）、项目内辅助模块（如 `tests.entrypoints.openai.utils.verify_harmony_messages`、`vllm.entrypoints.openai.parser.harmony_utils.auto_drop_analysis_messages`、`vllm.entrypoints.openai.parser.harmony_utils.extract_function_from_recipient`）。

### Class / 类: TestIsFunctionRecipient (L24-L112)
```python
class TestIsFunctionRecipient:
    @pytest.mark.parametrize(
        "recipient",
        [
            "functions.get_weather",
            "functions.search_web",
            "functions.math.sum",
        ],
    )
    def test_functions_prefix_accepted(self, recipient):
        assert is_function_recipient(recipient) is True

    @pytest.mark.parametrize(
        "recipient",
# ... 67 lines omitted for brevity ...
        [
            "<|start|>",
            "<|end|>",
            "<|channel|>",
        ],
    )
    def test_harmony_tokens_rejected(self, recipient):
        assert is_function_recipient(recipient) is False
```
**EN:** This class groups related scenarios in `TestIsFunctionRecipient`. It contains 8 test method(s) and 0 supporting method(s). Representative methods include `test_functions_prefix_accepted`, `test_bare_function_name_accepted`, `test_assistant_rejected`.
**CN:** 该类将与 `TestIsFunctionRecipient` 相关的场景组织在一起。 它包含 8 个测试方法和 0 个辅助方法。 代表性方法包括 `test_functions_prefix_accepted`、`test_bare_function_name_accepted`、`test_assistant_rejected`。

### Test method / 测试方法: TestIsFunctionRecipient.test_functions_prefix_accepted (L25-L34)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "functions.get_weather",
            "functions.search_web",
            "functions.math.sum",
        ],
    )
    def test_functions_prefix_accepted(self, recipient):
        assert is_function_recipient(recipient) is True
```
**EN:** This test validates `TestIsFunctionRecipient.test_functions_prefix_accepted`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `is_function_recipient(recipient) is True`.
**CN:** 这个测试验证 `TestIsFunctionRecipient.test_functions_prefix_accepted`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `is_function_recipient(recipient) is True`。

### Test method / 测试方法: TestIsFunctionRecipient.test_bare_function_name_accepted (L36-L46)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "get_weather",
            "search_web",
            "calculator",
            "my-tool",
        ],
    )
    def test_bare_function_name_accepted(self, recipient):
        assert is_function_recipient(recipient) is True
```
**EN:** This test validates `TestIsFunctionRecipient.test_bare_function_name_accepted`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `is_function_recipient(recipient) is True`.
**CN:** 这个测试验证 `TestIsFunctionRecipient.test_bare_function_name_accepted`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `is_function_recipient(recipient) is True`。

### Test method / 测试方法: TestIsFunctionRecipient.test_assistant_rejected (L48-L55)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "assistant",
        ],
    )
    def test_assistant_rejected(self, recipient):
        assert is_function_recipient(recipient) is False
```
**EN:** This test validates `TestIsFunctionRecipient.test_assistant_rejected`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `is_function_recipient(recipient) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipient.test_assistant_rejected`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `is_function_recipient(recipient) is False`。

### Test method / 测试方法: TestIsFunctionRecipient.test_dotted_function_names_accepted (L57-L67)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "math.sum",
            "code.run",
            "namespace.tool_name",
            "my.deeply.nested.tool",
        ],
    )
    def test_dotted_function_names_accepted(self, recipient):
        assert is_function_recipient(recipient) is True
```
**EN:** This test validates `TestIsFunctionRecipient.test_dotted_function_names_accepted`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `is_function_recipient(recipient) is True`.
**CN:** 这个测试验证 `TestIsFunctionRecipient.test_dotted_function_names_accepted`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `is_function_recipient(recipient) is True`。

### Test method / 测试方法: TestIsFunctionRecipient.test_builtin_tool_names_rejected (L69-L78)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "python",
            "browser",
            "container",
        ],
    )
    def test_builtin_tool_names_rejected(self, recipient):
        assert is_function_recipient(recipient) is False
```
**EN:** This test validates `TestIsFunctionRecipient.test_builtin_tool_names_rejected`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `is_function_recipient(recipient) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipient.test_builtin_tool_names_rejected`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `is_function_recipient(recipient) is False`。

### Test method / 测试方法: TestIsFunctionRecipient.test_builtin_dotted_variants_rejected (L80-L91)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "python.run",
            "python.execute",
            "browser.search",
            "browser.open",
            "container.exec",
        ],
    )
    def test_builtin_dotted_variants_rejected(self, recipient):
        assert is_function_recipient(recipient) is False
```
**EN:** This test validates `TestIsFunctionRecipient.test_builtin_dotted_variants_rejected`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `is_function_recipient(recipient) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipient.test_builtin_dotted_variants_rejected`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `is_function_recipient(recipient) is False`。

### Test method / 测试方法: TestIsFunctionRecipient.test_empty_recipients_rejected (L93-L101)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "",
            "functions.",
        ],
    )
    def test_empty_recipients_rejected(self, recipient):
        assert is_function_recipient(recipient) is False
```
**EN:** This test validates `TestIsFunctionRecipient.test_empty_recipients_rejected`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `is_function_recipient(recipient) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipient.test_empty_recipients_rejected`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `is_function_recipient(recipient) is False`。

### Test method / 测试方法: TestIsFunctionRecipient.test_harmony_tokens_rejected (L103-L112)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "<|start|>",
            "<|end|>",
            "<|channel|>",
        ],
    )
    def test_harmony_tokens_rejected(self, recipient):
        assert is_function_recipient(recipient) is False
```
**EN:** This test validates `TestIsFunctionRecipient.test_harmony_tokens_rejected`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `is_function_recipient(recipient) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipient.test_harmony_tokens_rejected`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `is_function_recipient(recipient) is False`。

### Class / 类: TestIsFunctionRecipientWithAllowedNames (L115-L162)
```python
class TestIsFunctionRecipientWithAllowedNames:
    """Tests for is_function_recipient with allowed_function_tool_names."""

    def test_prefixed_always_accepted(self):
        """functions. prefix is always accepted regardless of allowed names."""
        fn_names = frozenset({"other_tool"})
        assert is_function_recipient("functions.get_weather", fn_names) is True

    def test_bare_name_accepted_when_in_allowed_names(self):
        fn_names = frozenset({"get_weather", "search_web"})
        assert is_function_recipient("get_weather", fn_names) is True
        assert is_function_recipient("search_web", fn_names) is True

    def test_bare_name_rejected_when_not_in_allowed_names(self):
# ... 26 lines omitted for brevity ...
        assert is_function_recipient("python.run", fn_names) is False
        assert is_function_recipient("browser.search", fn_names) is False

    def test_none_allowed_names_uses_heuristic(self):
        """When allowed names is None (Chat Completions), use heuristic."""
        assert is_function_recipient("get_weather", None) is True
        assert is_function_recipient("math.sum", None) is True
        assert is_function_recipient("python", None) is False
```
**EN:** This class groups related scenarios in `TestIsFunctionRecipientWithAllowedNames`. It contains 9 test method(s) and 0 supporting method(s). Representative methods include `test_prefixed_always_accepted`, `test_bare_name_accepted_when_in_allowed_names`, `test_bare_name_rejected_when_not_in_allowed_names`.
**CN:** 该类将与 `TestIsFunctionRecipientWithAllowedNames` 相关的场景组织在一起。 它包含 9 个测试方法和 0 个辅助方法。 代表性方法包括 `test_prefixed_always_accepted`、`test_bare_name_accepted_when_in_allowed_names`、`test_bare_name_rejected_when_not_in_allowed_names`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_prefixed_always_accepted (L118-L121)
```python
    def test_prefixed_always_accepted(self):
        """functions. prefix is always accepted regardless of allowed names."""
        fn_names = frozenset({"other_tool"})
        assert is_function_recipient("functions.get_weather", fn_names) is True
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_prefixed_always_accepted`. The main assertion is `is_function_recipient('functions.get_weather', fn_names) is True`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_prefixed_always_accepted`。 核心断言是 `is_function_recipient('functions.get_weather', fn_names) is True`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_bare_name_accepted_when_in_allowed_names (L123-L126)
```python
    def test_bare_name_accepted_when_in_allowed_names(self):
        fn_names = frozenset({"get_weather", "search_web"})
        assert is_function_recipient("get_weather", fn_names) is True
        assert is_function_recipient("search_web", fn_names) is True
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_bare_name_accepted_when_in_allowed_names`. The main assertion is `is_function_recipient('get_weather', fn_names) is True` and `is_function_recipient('search_web', fn_names) is True`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_bare_name_accepted_when_in_allowed_names`。 核心断言是 `is_function_recipient('get_weather', fn_names) is True` and `is_function_recipient('search_web', fn_names) is True`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_bare_name_rejected_when_not_in_allowed_names (L128-L130)
```python
    def test_bare_name_rejected_when_not_in_allowed_names(self):
        fn_names = frozenset({"get_weather"})
        assert is_function_recipient("unknown_tool", fn_names) is False
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_bare_name_rejected_when_not_in_allowed_names`. The main assertion is `is_function_recipient('unknown_tool', fn_names) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_bare_name_rejected_when_not_in_allowed_names`。 核心断言是 `is_function_recipient('unknown_tool', fn_names) is False`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_dotted_name_accepted_when_in_allowed_names (L132-L135)
```python
    def test_dotted_name_accepted_when_in_allowed_names(self):
        fn_names = frozenset({"math.sum", "namespace.tool_name"})
        assert is_function_recipient("math.sum", fn_names) is True
        assert is_function_recipient("namespace.tool_name", fn_names) is True
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_dotted_name_accepted_when_in_allowed_names`. The main assertion is `is_function_recipient('math.sum', fn_names) is True` and `is_function_recipient('namespace.tool_name', fn_names) is True`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_dotted_name_accepted_when_in_allowed_names`。 核心断言是 `is_function_recipient('math.sum', fn_names) is True` and `is_function_recipient('namespace.tool_name', fn_names) is True`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_dotted_name_rejected_when_not_in_allowed_names (L137-L139)
```python
    def test_dotted_name_rejected_when_not_in_allowed_names(self):
        fn_names = frozenset({"get_weather"})
        assert is_function_recipient("custom_server.search", fn_names) is False
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_dotted_name_rejected_when_not_in_allowed_names`. The main assertion is `is_function_recipient('custom_server.search', fn_names) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_dotted_name_rejected_when_not_in_allowed_names`。 核心断言是 `is_function_recipient('custom_server.search', fn_names) is False`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_empty_allowed_names_rejects_bare_names (L141-L145)
```python
    def test_empty_allowed_names_rejects_bare_names(self):
        """Empty frozenset means no function tools — bare names are not functions."""
        fn_names: frozenset[str] = frozenset()
        assert is_function_recipient("get_weather", fn_names) is False
        assert is_function_recipient("math.sum", fn_names) is False
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_empty_allowed_names_rejects_bare_names`. The main assertion is `is_function_recipient('get_weather', fn_names) is False` and `is_function_recipient('math.sum', fn_names) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_empty_allowed_names_rejects_bare_names`。 核心断言是 `is_function_recipient('get_weather', fn_names) is False` and `is_function_recipient('math.sum', fn_names) is False`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_builtin_tools_always_rejected (L147-L151)
```python
    def test_builtin_tools_always_rejected(self):
        fn_names = frozenset({"python", "browser", "container"})
        assert is_function_recipient("python", fn_names) is False
        assert is_function_recipient("browser", fn_names) is False
        assert is_function_recipient("container", fn_names) is False
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_builtin_tools_always_rejected`. The main assertion is `is_function_recipient('python', fn_names) is False` and `is_function_recipient('browser', fn_names) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_builtin_tools_always_rejected`。 核心断言是 `is_function_recipient('python', fn_names) is False` and `is_function_recipient('browser', fn_names) is False`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_builtin_dotted_always_rejected (L153-L156)
```python
    def test_builtin_dotted_always_rejected(self):
        fn_names = frozenset({"python.run", "browser.search"})
        assert is_function_recipient("python.run", fn_names) is False
        assert is_function_recipient("browser.search", fn_names) is False
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_builtin_dotted_always_rejected`. The main assertion is `is_function_recipient('python.run', fn_names) is False` and `is_function_recipient('browser.search', fn_names) is False`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_builtin_dotted_always_rejected`。 核心断言是 `is_function_recipient('python.run', fn_names) is False` and `is_function_recipient('browser.search', fn_names) is False`。

### Test method / 测试方法: TestIsFunctionRecipientWithAllowedNames.test_none_allowed_names_uses_heuristic (L158-L162)
```python
    def test_none_allowed_names_uses_heuristic(self):
        """When allowed names is None (Chat Completions), use heuristic."""
        assert is_function_recipient("get_weather", None) is True
        assert is_function_recipient("math.sum", None) is True
        assert is_function_recipient("python", None) is False
```
**EN:** This test validates `TestIsFunctionRecipientWithAllowedNames.test_none_allowed_names_uses_heuristic`. The main assertion is `is_function_recipient('get_weather', None) is True` and `is_function_recipient('math.sum', None) is True`.
**CN:** 这个测试验证 `TestIsFunctionRecipientWithAllowedNames.test_none_allowed_names_uses_heuristic`。 核心断言是 `is_function_recipient('get_weather', None) is True` and `is_function_recipient('math.sum', None) is True`。

### Class / 类: TestExtractFunctionFromRecipient (L165-L197)
```python
class TestExtractFunctionFromRecipient:
    @pytest.mark.parametrize(
        "recipient,expected",
        [
            ("functions.get_weather", "get_weather"),
            ("functions.search_web", "search_web"),
            ("functions.", ""),
        ],
    )
    def test_strips_functions_prefix(self, recipient, expected):
        assert extract_function_from_recipient(recipient) == expected

    @pytest.mark.parametrize(
        "recipient",
# ... 11 lines omitted for brevity ...
        [
            ("functions.math.sum", "math.sum"),
            ("math.sum", "math.sum"),
            ("namespace.tool_name", "namespace.tool_name"),
        ],
    )
    def test_dotted_function_name_extraction(self, recipient, expected):
        assert extract_function_from_recipient(recipient) == expected
```
**EN:** This class groups related scenarios in `TestExtractFunctionFromRecipient`. It contains 3 test method(s) and 0 supporting method(s). Representative methods include `test_strips_functions_prefix`, `test_bare_name_returned_as_is`, `test_dotted_function_name_extraction`.
**CN:** 该类将与 `TestExtractFunctionFromRecipient` 相关的场景组织在一起。 它包含 3 个测试方法和 0 个辅助方法。 代表性方法包括 `test_strips_functions_prefix`、`test_bare_name_returned_as_is`、`test_dotted_function_name_extraction`。

### Test method / 测试方法: TestExtractFunctionFromRecipient.test_strips_functions_prefix (L166-L175)
```python
    @pytest.mark.parametrize(
        "recipient,expected",
        [
            ("functions.get_weather", "get_weather"),
            ("functions.search_web", "search_web"),
            ("functions.", ""),
        ],
    )
    def test_strips_functions_prefix(self, recipient, expected):
        assert extract_function_from_recipient(recipient) == expected
```
**EN:** This test validates `TestExtractFunctionFromRecipient.test_strips_functions_prefix`. It uses parameterization over `recipient`, `expected`. Key inputs are `recipient`, `expected`. The main assertion is `extract_function_from_recipient(recipient) == expected`.
**CN:** 这个测试验证 `TestExtractFunctionFromRecipient.test_strips_functions_prefix`。 它通过参数化组合 `recipient`、`expected`。 关键输入包括 `recipient`、`expected`。 核心断言是 `extract_function_from_recipient(recipient) == expected`。

### Test method / 测试方法: TestExtractFunctionFromRecipient.test_bare_name_returned_as_is (L177-L186)
```python
    @pytest.mark.parametrize(
        "recipient",
        [
            "get_weather",
            "calculator",
            "my-tool",
        ],
    )
    def test_bare_name_returned_as_is(self, recipient):
        assert extract_function_from_recipient(recipient) == recipient
```
**EN:** This test validates `TestExtractFunctionFromRecipient.test_bare_name_returned_as_is`. It uses parameterization over `recipient`. Key inputs are `recipient`. The main assertion is `extract_function_from_recipient(recipient) == recipient`.
**CN:** 这个测试验证 `TestExtractFunctionFromRecipient.test_bare_name_returned_as_is`。 它通过参数化组合 `recipient`。 关键输入包括 `recipient`。 核心断言是 `extract_function_from_recipient(recipient) == recipient`。

### Test method / 测试方法: TestExtractFunctionFromRecipient.test_dotted_function_name_extraction (L188-L197)
```python
    @pytest.mark.parametrize(
        "recipient,expected",
        [
            ("functions.math.sum", "math.sum"),
            ("math.sum", "math.sum"),
            ("namespace.tool_name", "namespace.tool_name"),
        ],
    )
    def test_dotted_function_name_extraction(self, recipient, expected):
        assert extract_function_from_recipient(recipient) == expected
```
**EN:** This test validates `TestExtractFunctionFromRecipient.test_dotted_function_name_extraction`. It uses parameterization over `recipient`, `expected`. Key inputs are `recipient`, `expected`. The main assertion is `extract_function_from_recipient(recipient) == expected`.
**CN:** 这个测试验证 `TestExtractFunctionFromRecipient.test_dotted_function_name_extraction`。 它通过参数化组合 `recipient`、`expected`。 关键输入包括 `recipient`、`expected`。 核心断言是 `extract_function_from_recipient(recipient) == expected`。

### Class / 类: TestCommonParseInputToHarmonyMessage (L200-L390)
```python
class TestCommonParseInputToHarmonyMessage:
    """
    Tests for scenarios that are common to both Chat Completion
    parse_chat_input_to_harmony_message and Responses API
    response_previous_input_to_harmony functions.
    """

    @pytest.fixture(
        params=[parse_chat_input_to_harmony_message, response_previous_input_to_harmony]
    )
    def parse_function(self, request):
        return request.param

    def test_assistant_message_with_tool_calls(self, parse_function):
# ... 169 lines omitted for brevity ...
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert len(messages[0].content) == 2
        assert messages[0].content[0].text == ""
        assert messages[0].content[1].text == "actual text"
```
**EN:** This class groups related scenarios in `TestCommonParseInputToHarmonyMessage`. It contains 10 test method(s) and 1 supporting method(s). Representative methods include `test_assistant_message_with_tool_calls`, `test_assistant_message_with_empty_tool_call_arguments`, `test_system_message`, `parse_function`.
**CN:** 该类将与 `TestCommonParseInputToHarmonyMessage` 相关的场景组织在一起。 它包含 10 个测试方法和 1 个辅助方法。 代表性方法包括 `test_assistant_message_with_tool_calls`、`test_assistant_message_with_empty_tool_call_arguments`、`test_system_message`、`parse_function`。

### Fixture method / 夹具方法: TestCommonParseInputToHarmonyMessage.parse_function (L207-L211)
```python
    @pytest.fixture(
        params=[parse_chat_input_to_harmony_message, response_previous_input_to_harmony]
    )
    def parse_function(self, request):
        return request.param
```
**EN:** This fixture prepares `TestCommonParseInputToHarmonyMessage.parse_function` for dependent tests. Key inputs are `request`.
**CN:** 这个 fixture 为依赖它的测试准备 `TestCommonParseInputToHarmonyMessage.parse_function`。 关键输入包括 `request`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_assistant_message_with_tool_calls (L213-L249)
```python
    def test_assistant_message_with_tool_calls(self, parse_function):
        """Test parsing assistant message with tool calls."""
        chat_msg = {
            "role": "assistant",
            "tool_calls": [
                {
                    "function": {
                        "name": "get_weather",
                        "arguments": '{"location": "San Francisco"}',
                    }
                },
                {
                    "function": {
                        "name": "search_web",
                        "arguments": '{"query": "latest news"}',
                    }
                },
            ],
# ... 11 lines omitted for brevity ...
        assert messages[0].content_type == "json"

        # Second tool call
        assert messages[1].author.role == Role.ASSISTANT
        assert messages[1].content[0].text == '{"query": "latest news"}'
        assert messages[1].channel == "commentary"
        assert messages[1].recipient == "functions.search_web"
        assert messages[1].content_type == "json"
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_assistant_message_with_tool_calls`. Key inputs are `parse_function`. The main assertion is `len(messages) == 2` and `messages[0].author.role == Role.ASSISTANT`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_assistant_message_with_tool_calls`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 2` and `messages[0].author.role == Role.ASSISTANT`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_assistant_message_with_empty_tool_call_arguments (L251-L269)
```python
    def test_assistant_message_with_empty_tool_call_arguments(self, parse_function):
        """Test parsing assistant message with tool call having None arguments."""
        chat_msg = {
            "role": "assistant",
            "tool_calls": [
                {
                    "function": {
                        "name": "get_current_time",
                        "arguments": None,
                    }
                }
            ],
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert messages[0].content[0].text == ""
        assert messages[0].recipient == "functions.get_current_time"
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_assistant_message_with_empty_tool_call_arguments`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `messages[0].content[0].text == ''`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_assistant_message_with_empty_tool_call_arguments`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `messages[0].content[0].text == ''`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_system_message (L271-L283)
```python
    def test_system_message(self, parse_function):
        """Test parsing system message."""
        chat_msg = {
            "role": "system",
            "content": "You are a helpful assistant",
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        # System messages are converted using Message.from_dict
        # which should preserve the role
        assert messages[0].author.role == Role.SYSTEM
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_system_message`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.SYSTEM`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_system_message`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.SYSTEM`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_developer_message (L285-L295)
```python
    def test_developer_message(self, parse_function):
        """Test parsing developer message."""
        chat_msg = {
            "role": "developer",
            "content": "Use concise language",
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.DEVELOPER
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_developer_message`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.DEVELOPER`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_developer_message`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.DEVELOPER`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_user_message_with_string_content (L297-L308)
```python
    def test_user_message_with_string_content(self, parse_function):
        """Test parsing user message with string content."""
        chat_msg = {
            "role": "user",
            "content": "What's the weather in San Francisco?",
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.USER
        assert messages[0].content[0].text == "What's the weather in San Francisco?"
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_user_message_with_string_content`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.USER`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_user_message_with_string_content`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.USER`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_user_message_with_array_content (L310-L326)
```python
    def test_user_message_with_array_content(self, parse_function):
        """Test parsing user message with array content."""
        chat_msg = {
            "role": "user",
            "content": [
                {"text": "What's in this image? "},
                {"text": "Please describe it."},
            ],
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.USER
        assert len(messages[0].content) == 2
        assert messages[0].content[0].text == "What's in this image? "
        assert messages[0].content[1].text == "Please describe it."
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_user_message_with_array_content`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.USER`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_user_message_with_array_content`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.USER`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_assistant_message_with_string_content (L328-L339)
```python
    def test_assistant_message_with_string_content(self, parse_function):
        """Test parsing assistant message with string content (no tool calls)."""
        chat_msg = {
            "role": "assistant",
            "content": "Hello! How can I help you today?",
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.ASSISTANT
        assert messages[0].content[0].text == "Hello! How can I help you today?"
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_assistant_message_with_string_content`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.ASSISTANT`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_assistant_message_with_string_content`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.ASSISTANT`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_pydantic_model_input (L341-L356)
```python
    def test_pydantic_model_input(self, parse_function):
        """Test parsing Pydantic model input (has model_dump method)."""

        class MockPydanticModel:
            def model_dump(self, exclude_none=True):
                return {
                    "role": "user",
                    "content": "Test message",
                }

        chat_msg = MockPydanticModel()
        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert messages[0].author.role == Role.USER
        assert messages[0].content[0].text == "Test message"
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_pydantic_model_input`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `messages[0].author.role == Role.USER`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_pydantic_model_input`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `messages[0].author.role == Role.USER`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_tool_call_with_missing_function_fields (L358-L373)
```python
    def test_tool_call_with_missing_function_fields(self, parse_function):
        """Test parsing tool call with missing name or arguments."""
        chat_msg = {
            "role": "assistant",
            "tool_calls": [
                {
                    "function": {}  # Missing both name and arguments
                }
            ],
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert messages[0].recipient == "functions."
        assert messages[0].content[0].text == ""
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_tool_call_with_missing_function_fields`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `messages[0].recipient == 'functions.'`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_tool_call_with_missing_function_fields`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `messages[0].recipient == 'functions.'`。

### Test method / 测试方法: TestCommonParseInputToHarmonyMessage.test_array_content_with_missing_text (L375-L390)
```python
    def test_array_content_with_missing_text(self, parse_function):
        """Test parsing array content where text field is missing."""
        chat_msg = {
            "role": "user",
            "content": [
                {},  # Missing text field
                {"text": "actual text"},
            ],
        }

        messages = parse_function(chat_msg)

        assert len(messages) == 1
        assert len(messages[0].content) == 2
        assert messages[0].content[0].text == ""
        assert messages[0].content[1].text == "actual text"
```
**EN:** This test validates `TestCommonParseInputToHarmonyMessage.test_array_content_with_missing_text`. Key inputs are `parse_function`. The main assertion is `len(messages) == 1` and `len(messages[0].content) == 2`.
**CN:** 这个测试验证 `TestCommonParseInputToHarmonyMessage.test_array_content_with_missing_text`。 关键输入包括 `parse_function`。 核心断言是 `len(messages) == 1` and `len(messages[0].content) == 2`。

### Class / 类: TestParseChatInputToHarmonyMessage (L393-L761)
```python
class TestParseChatInputToHarmonyMessage:
    """
    Tests for scenarios that are specific to the Chat Completion API
    parse_chat_input_to_harmony_message function.
    """

    def test_user_message_with_empty_content(self):
        chat_msg = {
            "role": "user",
            "content": "",
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

# ... 347 lines omitted for brevity ...
                {
                    "role": "tool",
                    "name": "functions.empty_tool",
                    "content": "",
                    "channel": "commentary",
                },
            ],
        )
```
**EN:** This class groups related scenarios in `TestParseChatInputToHarmonyMessage`. It contains 15 test method(s) and 0 supporting method(s). Representative methods include `test_user_message_with_empty_content`, `test_user_message_with_none_content`, `test_assistant_message_with_empty_content`.
**CN:** 该类将与 `TestParseChatInputToHarmonyMessage` 相关的场景组织在一起。 它包含 15 个测试方法和 0 个辅助方法。 代表性方法包括 `test_user_message_with_empty_content`、`test_user_message_with_none_content`、`test_assistant_message_with_empty_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_user_message_with_empty_content (L399-L415)
```python
    def test_user_message_with_empty_content(self):
        chat_msg = {
            "role": "user",
            "content": "",
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "user",
                    "content": "",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_user_message_with_empty_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_user_message_with_empty_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_user_message_with_none_content (L417-L433)
```python
    def test_user_message_with_none_content(self):
        chat_msg = {
            "role": "user",
            "content": None,
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "user",
                    "content": "",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_user_message_with_none_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_user_message_with_none_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_empty_content (L435-L443)
```python
    def test_assistant_message_with_empty_content(self):
        chat_msg = {
            "role": "assistant",
            "content": "",
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        assert len(messages) == 0
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_empty_content`. The main assertion is `len(messages) == 0`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_empty_content`。 核心断言是 `len(messages) == 0`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_none_content (L445-L453)
```python
    def test_assistant_message_with_none_content(self):
        chat_msg = {
            "role": "assistant",
            "content": None,
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        assert len(messages) == 0
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_none_content`. The main assertion is `len(messages) == 0`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_none_content`。 核心断言是 `len(messages) == 0`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_content_but_empty_reasoning (L455-L473)
```python
    def test_assistant_message_with_content_but_empty_reasoning(self):
        chat_msg = {
            "role": "assistant",
            "content": "The answer is 4.",
            "reasoning": "",
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "assistant",
                    "channel": "final",
                    "content": "The answer is 4.",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_content_but_empty_reasoning`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_content_but_empty_reasoning`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_reasoning_but_empty_content (L475-L493)
```python
    def test_assistant_message_with_reasoning_but_empty_content(self):
        chat_msg = {
            "role": "assistant",
            "reasoning": "I'm thinking about the user's question.",
            "content": "",
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "assistant",
                    "channel": "analysis",
                    "content": "I'm thinking about the user's question.",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_reasoning_but_empty_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_reasoning_but_empty_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_reasoning_but_none_content (L495-L513)
```python
    def test_assistant_message_with_reasoning_but_none_content(self):
        chat_msg = {
            "role": "assistant",
            "reasoning": "I'm thinking about the user's question.",
            "content": None,
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "assistant",
                    "channel": "analysis",
                    "content": "I'm thinking about the user's question.",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_reasoning_but_none_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_reasoning_but_none_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_but_no_content (L515-L541)
```python
    def test_assistant_message_with_tool_calls_but_no_content(self):
        chat_msg = {
            "role": "assistant",
            "tool_calls": [
                {
                    "function": {
                        "name": "get_weather",
                        "arguments": '{"location": "San Francisco"}',
                    }
                }
            ],
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "assistant",
                    "channel": "commentary",
                    "recipient": "functions.get_weather",
                    "content": '{"location": "San Francisco"}',
                    "content_type": "json",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_but_no_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_but_no_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_content (L543-L575)
```python
    def test_assistant_message_with_tool_calls_and_content(self):
        chat_msg = {
            "role": "assistant",
            "tool_calls": [
                {
                    "function": {
                        "name": "get_weather",
                        "arguments": '{"location": "San Francisco"}',
                    }
                }
            ],
            "content": "I'll call the tool.",
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
            messages,
# ... 7 lines omitted for brevity ...
                    "role": "assistant",
                    "channel": "commentary",
                    "recipient": "functions.get_weather",
                    "content": '{"location": "San Francisco"}',
                    "content_type": "json",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_reasoning (L577-L609)
```python
    def test_assistant_message_with_tool_calls_and_reasoning(self):
        chat_msg = {
            "role": "assistant",
            "tool_calls": [
                {
                    "function": {
                        "name": "get_weather",
                        "arguments": '{"location": "San Francisco"}',
                    }
                }
            ],
            "reasoning": "I should use the get_weather tool.",
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
            messages,
# ... 7 lines omitted for brevity ...
                    "role": "assistant",
                    "channel": "commentary",
                    "recipient": "functions.get_weather",
                    "content": '{"location": "San Francisco"}',
                    "content_type": "json",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_reasoning`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_reasoning`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_reasoning_and_content (L611-L649)
```python
    def test_assistant_message_with_tool_calls_and_reasoning_and_content(self):
        chat_msg = {
            "role": "assistant",
            "tool_calls": [
                {
                    "function": {
                        "name": "get_weather",
                        "arguments": '{"location": "San Francisco"}',
                    }
                }
            ],
            "reasoning": "I should use the get_weather tool.",
            "content": "I'll call the tool.",
        }

        messages = parse_chat_input_to_harmony_message(chat_msg)

        verify_harmony_messages(
# ... 13 lines omitted for brevity ...
                    "role": "assistant",
                    "channel": "commentary",
                    "recipient": "functions.get_weather",
                    "content": '{"location": "San Francisco"}',
                    "content_type": "json",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_reasoning_and_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_assistant_message_with_tool_calls_and_reasoning_and_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_tool_message_with_string_content (L651-L675)
```python
    def test_tool_message_with_string_content(self):
        tool_id_names = {
            "call_123": "get_weather",
        }
        chat_msg = {
            "role": "tool",
            "tool_call_id": "call_123",
            "content": "The weather in San Francisco is sunny, 72°F",
        }

        messages = parse_chat_input_to_harmony_message(
            chat_msg, tool_id_names=tool_id_names
        )

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "tool",
                    "name": "functions.get_weather",
                    "content": "The weather in San Francisco is sunny, 72°F",
                    "channel": "commentary",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_tool_message_with_string_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_tool_message_with_string_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_tool_message_with_array_content (L677-L709)
```python
    def test_tool_message_with_array_content(self):
        tool_id_names = {
            "call_123": "search_results",
        }
        chat_msg = {
            "role": "tool",
            "tool_call_id": "call_123",
            "content": [
                {"type": "text", "text": "Result 1: "},
                {"type": "text", "text": "Result 2: "},
                {
                    "type": "image",
                    "url": "http://example.com/img.png",
                },  # Should be ignored
                {"type": "text", "text": "Result 3"},
            ],
        }

# ... 7 lines omitted for brevity ...
                {
                    "role": "tool",
                    "name": "functions.search_results",
                    "content": "Result 1: Result 2: Result 3",
                    "channel": "commentary",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_tool_message_with_array_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_tool_message_with_array_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_tool_message_with_empty_content (L711-L735)
```python
    def test_tool_message_with_empty_content(self):
        tool_id_names = {
            "call_123": "empty_tool",
        }
        chat_msg = {
            "role": "tool",
            "tool_call_id": "call_123",
            "content": "",
        }

        messages = parse_chat_input_to_harmony_message(
            chat_msg, tool_id_names=tool_id_names
        )

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "tool",
                    "name": "functions.empty_tool",
                    "content": "",
                    "channel": "commentary",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_tool_message_with_empty_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_tool_message_with_empty_content`。

### Test method / 测试方法: TestParseChatInputToHarmonyMessage.test_tool_message_with_none_content (L737-L761)
```python
    def test_tool_message_with_none_content(self):
        tool_id_names = {
            "call_123": "empty_tool",
        }
        chat_msg = {
            "role": "tool",
            "tool_call_id": "call_123",
            "content": None,
        }

        messages = parse_chat_input_to_harmony_message(
            chat_msg, tool_id_names=tool_id_names
        )

        verify_harmony_messages(
            messages,
            [
                {
                    "role": "tool",
                    "name": "functions.empty_tool",
                    "content": "",
                    "channel": "commentary",
                },
            ],
        )
```
**EN:** This test validates `TestParseChatInputToHarmonyMessage.test_tool_message_with_none_content`.
**CN:** 这个测试验证 `TestParseChatInputToHarmonyMessage.test_tool_message_with_none_content`。

### Class / 类: TestAutoDropAnalysisMessages (L764-L883)
```python
class TestAutoDropAnalysisMessages:
    def test_no_analysis_messages(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 4."
            ).with_channel("final"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        assert cleaned_messages == messages

    def test_only_analysis_message(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking about the user's question."
# ... 98 lines omitted for brevity ...
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 4."
            ).with_channel("final"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        # Should have dropped the analysis message
        assert cleaned_messages == messages[1:]
```
**EN:** This class groups related scenarios in `TestAutoDropAnalysisMessages`. It contains 8 test method(s) and 0 supporting method(s). Representative methods include `test_no_analysis_messages`, `test_only_analysis_message`, `test_multiple_analysis_messages_without_final_message`.
**CN:** 该类将与 `TestAutoDropAnalysisMessages` 相关的场景组织在一起。 它包含 8 个测试方法和 0 个辅助方法。 代表性方法包括 `test_no_analysis_messages`、`test_only_analysis_message`、`test_multiple_analysis_messages_without_final_message`。

### Test method / 测试方法: TestAutoDropAnalysisMessages.test_no_analysis_messages (L765-L772)
```python
    def test_no_analysis_messages(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 4."
            ).with_channel("final"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        assert cleaned_messages == messages
```
**EN:** This test validates `TestAutoDropAnalysisMessages.test_no_analysis_messages`. The main assertion is `cleaned_messages == messages`.
**CN:** 这个测试验证 `TestAutoDropAnalysisMessages.test_no_analysis_messages`。 核心断言是 `cleaned_messages == messages`。

### Test method / 测试方法: TestAutoDropAnalysisMessages.test_only_analysis_message (L774-L781)
```python
    def test_only_analysis_message(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking about the user's question."
            ).with_channel("analysis"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        assert cleaned_messages == messages
```
**EN:** This test validates `TestAutoDropAnalysisMessages.test_only_analysis_message`. The main assertion is `cleaned_messages == messages`.
**CN:** 这个测试验证 `TestAutoDropAnalysisMessages.test_only_analysis_message`。 核心断言是 `cleaned_messages == messages`。

### Test method / 测试方法: TestAutoDropAnalysisMessages.test_multiple_analysis_messages_without_final_message (L783-L796)
```python
    def test_multiple_analysis_messages_without_final_message(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking about the user's question."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking more."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking even more."
            ).with_channel("analysis"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        assert cleaned_messages == messages
```
**EN:** This test validates `TestAutoDropAnalysisMessages.test_multiple_analysis_messages_without_final_message`. The main assertion is `cleaned_messages == messages`.
**CN:** 这个测试验证 `TestAutoDropAnalysisMessages.test_multiple_analysis_messages_without_final_message`。 核心断言是 `cleaned_messages == messages`。

### Test method / 测试方法: TestAutoDropAnalysisMessages.test_only_final_message (L798-L805)
```python
    def test_only_final_message(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 4."
            ).with_channel("final"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        assert cleaned_messages == messages
```
**EN:** This test validates `TestAutoDropAnalysisMessages.test_only_final_message`. The main assertion is `cleaned_messages == messages`.
**CN:** 这个测试验证 `TestAutoDropAnalysisMessages.test_only_final_message`。 核心断言是 `cleaned_messages == messages`。

### Test method / 测试方法: TestAutoDropAnalysisMessages.test_drops_one_analysis_messages_before_final_message (L807-L821)
```python
    def test_drops_one_analysis_messages_before_final_message(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking about the user's question."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 4."
            ).with_channel("final"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I should think harder."
            ).with_channel("analysis"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        # Should have dropped the first analysis message
        assert cleaned_messages == messages[1:]
```
**EN:** This test validates `TestAutoDropAnalysisMessages.test_drops_one_analysis_messages_before_final_message`. The main assertion is `cleaned_messages == messages[1:]`.
**CN:** 这个测试验证 `TestAutoDropAnalysisMessages.test_drops_one_analysis_messages_before_final_message`。 核心断言是 `cleaned_messages == messages[1:]`。

### Test method / 测试方法: TestAutoDropAnalysisMessages.test_drops_all_analysis_messages_before_final_message (L823-L843)
```python
    def test_drops_all_analysis_messages_before_final_message(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking about the user's question."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking more."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking even more."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 4."
            ).with_channel("final"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I should think harder."
            ).with_channel("analysis"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        # Should have dropped the first 3 analysis messages
        assert cleaned_messages == messages[3:]
```
**EN:** This test validates `TestAutoDropAnalysisMessages.test_drops_all_analysis_messages_before_final_message`. The main assertion is `cleaned_messages == messages[3:]`.
**CN:** 这个测试验证 `TestAutoDropAnalysisMessages.test_drops_all_analysis_messages_before_final_message`。 核心断言是 `cleaned_messages == messages[3:]`。

### Test method / 测试方法: TestAutoDropAnalysisMessages.test_multiple_analysis_messages_with_multiple_final_messages (L845-L870)
```python
    def test_multiple_analysis_messages_with_multiple_final_messages(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking about the user's question."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking more."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I'm thinking even more."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 4."
            ).with_channel("final"),
            Message.from_role_and_content(
                Role.ASSISTANT, "I should think harder."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 5."
            ).with_channel("final"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        # Should have dropped all those analysis messages
        assert len(cleaned_messages) == 2
        assert cleaned_messages[0].content[0].text == "The answer is 4."
        assert cleaned_messages[1].content[0].text == "The answer is 5."
```
**EN:** This test validates `TestAutoDropAnalysisMessages.test_multiple_analysis_messages_with_multiple_final_messages`. The main assertion is `len(cleaned_messages) == 2` and `cleaned_messages[0].content[0].text == 'The answer is 4.'`.
**CN:** 这个测试验证 `TestAutoDropAnalysisMessages.test_multiple_analysis_messages_with_multiple_final_messages`。 核心断言是 `len(cleaned_messages) == 2` and `cleaned_messages[0].content[0].text == 'The answer is 4.'`。

### Test method / 测试方法: TestAutoDropAnalysisMessages.test_drops_non_assistant_analysis_messages (L872-L883)
```python
    def test_drops_non_assistant_analysis_messages(self) -> None:
        messages = [
            Message.from_role_and_content(
                Role.TOOL, "The tool thinks we should think harder."
            ).with_channel("analysis"),
            Message.from_role_and_content(
                Role.ASSISTANT, "The answer is 4."
            ).with_channel("final"),
        ]
        cleaned_messages = auto_drop_analysis_messages(messages)
        # Should have dropped the analysis message
        assert cleaned_messages == messages[1:]
```
**EN:** This test validates `TestAutoDropAnalysisMessages.test_drops_non_assistant_analysis_messages`. The main assertion is `cleaned_messages == messages[1:]`.
**CN:** 这个测试验证 `TestAutoDropAnalysisMessages.test_drops_non_assistant_analysis_messages`。 核心断言是 `cleaned_messages == messages[1:]`。

### Class / 类: TestParseChatOutput (L886-L987)
```python
class TestParseChatOutput:
    def test_parse_chat_output_interrupted_first_message(self) -> None:
        harmony_str = "<|channel|>final<|message|>I'm in the middle of answering"
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning is None
        assert final_content == "I'm in the middle of answering"

    def test_parse_chat_output_interrupted_reasoning_first_message(self) -> None:
        harmony_str = "<|channel|>analysis<|message|>I'm in the middle of thinking"
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning == "I'm in the middle of thinking"
        assert final_content is None
# ... 80 lines omitted for brevity ...
            "<|message|>Let me look that up.<|end|>"
            "<|start|>assistant<|channel|>final"
            "<|message|>The answer is 42.<|end|>"
        )
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning is None
        assert final_content == "Let me look that up.\nThe answer is 42."
```
**EN:** This class groups related scenarios in `TestParseChatOutput`. It contains 10 test method(s) and 0 supporting method(s). Representative methods include `test_parse_chat_output_interrupted_first_message`, `test_parse_chat_output_interrupted_reasoning_first_message`, `test_parse_chat_output_complete_reasoning_interrupted_content`.
**CN:** 该类将与 `TestParseChatOutput` 相关的场景组织在一起。 它包含 10 个测试方法和 0 个辅助方法。 代表性方法包括 `test_parse_chat_output_interrupted_first_message`、`test_parse_chat_output_interrupted_reasoning_first_message`、`test_parse_chat_output_complete_reasoning_interrupted_content`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_interrupted_first_message (L887-L892)
```python
    def test_parse_chat_output_interrupted_first_message(self) -> None:
        harmony_str = "<|channel|>final<|message|>I'm in the middle of answering"
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning is None
        assert final_content == "I'm in the middle of answering"
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_interrupted_first_message`. The main assertion is `reasoning is None` and `final_content == "I'm in the middle of answering"`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_interrupted_first_message`。 核心断言是 `reasoning is None` and `final_content == "I'm in the middle of answering"`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_interrupted_reasoning_first_message (L894-L899)
```python
    def test_parse_chat_output_interrupted_reasoning_first_message(self) -> None:
        harmony_str = "<|channel|>analysis<|message|>I'm in the middle of thinking"
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning == "I'm in the middle of thinking"
        assert final_content is None
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_interrupted_reasoning_first_message`. The main assertion is `reasoning == "I'm in the middle of thinking"` and `final_content is None`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_interrupted_reasoning_first_message`。 核心断言是 `reasoning == "I'm in the middle of thinking"` and `final_content is None`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_complete_reasoning_interrupted_content (L901-L910)
```python
    def test_parse_chat_output_complete_reasoning_interrupted_content(self) -> None:
        harmony_str = (
            "<|channel|>analysis<|message|>I'm thinking.<|end|>"
            "<|start|>assistant<|channel|>final"
            "<|message|>I'm in the middle of answering"
        )
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning == "I'm thinking."
        assert final_content == "I'm in the middle of answering"
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_complete_reasoning_interrupted_content`. The main assertion is `reasoning == "I'm thinking."` and `final_content == "I'm in the middle of answering"`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_complete_reasoning_interrupted_content`。 核心断言是 `reasoning == "I'm thinking."` and `final_content == "I'm in the middle of answering"`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_complete_content (L912-L917)
```python
    def test_parse_chat_output_complete_content(self) -> None:
        harmony_str = "<|channel|>final<|message|>The answer is 4.<|end|>"
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning is None
        assert final_content == "The answer is 4."
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_complete_content`. The main assertion is `reasoning is None` and `final_content == 'The answer is 4.'`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_complete_content`。 核心断言是 `reasoning is None` and `final_content == 'The answer is 4.'`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_complete_commentary (L919-L926)
```python
    def test_parse_chat_output_complete_commentary(self) -> None:
        harmony_str = (
            "<|channel|>commentary<|message|>I need to call some tools.<|end|>"
        )
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning is None
        assert final_content == "I need to call some tools."
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_complete_commentary`. The main assertion is `reasoning is None` and `final_content == 'I need to call some tools.'`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_complete_commentary`。 核心断言是 `reasoning is None` and `final_content == 'I need to call some tools.'`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_complete_reasoning (L928-L935)
```python
    def test_parse_chat_output_complete_reasoning(self) -> None:
        harmony_str = (
            "<|channel|>analysis<|message|>I've thought hard about this.<|end|>"
        )
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning == "I've thought hard about this."
        assert final_content is None
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_complete_reasoning`. The main assertion is `reasoning == "I've thought hard about this."` and `final_content is None`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_complete_reasoning`。 核心断言是 `reasoning == "I've thought hard about this."` and `final_content is None`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_complete_reasoning_and_content (L937-L945)
```python
    def test_parse_chat_output_complete_reasoning_and_content(self) -> None:
        harmony_str = (
            "<|channel|>analysis<|message|>I've thought hard about this.<|end|>"
            "<|start|>assistant<|channel|>final<|message|>The answer is 4.<|end|>"
        )
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning == "I've thought hard about this."
        assert final_content == "The answer is 4."
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_complete_reasoning_and_content`. The main assertion is `reasoning == "I've thought hard about this."` and `final_content == 'The answer is 4.'`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_complete_reasoning_and_content`。 核心断言是 `reasoning == "I've thought hard about this."` and `final_content == 'The answer is 4.'`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_commentary_with_recipient_excluded (L947-L964)
```python
    def test_parse_chat_output_commentary_with_recipient_excluded(self) -> None:
        """Commentary with a recipient (tool call) should not appear in
        final_content — those are handled separately by the tool parser.

        The first message is a preamble (visible), the second is a tool
        call (excluded). Only the preamble should appear in final_content.
        """
        harmony_str = (
            "<|channel|>commentary"
            "<|message|>Let me check the weather.<|end|>"
            "<|start|>assistant to=functions.get_weather"
            "<|channel|>commentary"
            '<|message|>{"location": "SF"}<|end|>'
        )
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning is None
        assert final_content == "Let me check the weather."
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_commentary_with_recipient_excluded`. The main assertion is `reasoning is None` and `final_content == 'Let me check the weather.'`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_commentary_with_recipient_excluded`。 核心断言是 `reasoning is None` and `final_content == 'Let me check the weather.'`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_interrupted_preamble (L966-L973)
```python
    def test_parse_chat_output_interrupted_preamble(self) -> None:
        """Partial/interrupted preamble (commentary without recipient) should
        appear in final_content, not reasoning."""
        harmony_str = "<|channel|>commentary<|message|>I'll search for that"
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning is None
        assert final_content == "I'll search for that"
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_interrupted_preamble`. The main assertion is `reasoning is None` and `final_content == "I'll search for that"`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_interrupted_preamble`。 核心断言是 `reasoning is None` and `final_content == "I'll search for that"`。

### Test method / 测试方法: TestParseChatOutput.test_parse_chat_output_preamble_then_final (L975-L987)
```python
    def test_parse_chat_output_preamble_then_final(self) -> None:
        """Preamble followed by a final message should both appear in
        final_content, joined by newline."""
        harmony_str = (
            "<|channel|>commentary"
            "<|message|>Let me look that up.<|end|>"
            "<|start|>assistant<|channel|>final"
            "<|message|>The answer is 42.<|end|>"
        )
        token_ids = get_encoding().encode(harmony_str, allowed_special="all")
        reasoning, final_content, _ = parse_chat_output(token_ids)
        assert reasoning is None
        assert final_content == "Let me look that up.\nThe answer is 42."
```
**EN:** This test validates `TestParseChatOutput.test_parse_chat_output_preamble_then_final`. The main assertion is `reasoning is None` and `final_content == 'Let me look that up.\nThe answer is 42.'`.
**CN:** 这个测试验证 `TestParseChatOutput.test_parse_chat_output_preamble_then_final`。 核心断言是 `reasoning is None` and `final_content == 'Let me look that up.\nThe answer is 42.'`。

### Test / 测试: test_has_custom_tools (L990-L996)
```python
def test_has_custom_tools() -> None:
    assert not has_custom_tools(set())
    assert not has_custom_tools({"web_search_preview", "code_interpreter", "container"})
    assert has_custom_tools({"others"})
    assert has_custom_tools(
        {"web_search_preview", "code_interpreter", "container", "others"}
    )
```
**EN:** This test validates `test_has_custom_tools`. The main assertion is `not has_custom_tools(set())` and `not has_custom_tools({'web_search_preview', 'code_interpreter', 'container'})`.
**CN:** 这个测试验证 `test_has_custom_tools`。 核心断言是 `not has_custom_tools(set())` and `not has_custom_tools({'web_search_preview', 'code_interpreter', 'container'})`。

### Class / 类: TestGetSystemMessage (L999-L1029)
```python
class TestGetSystemMessage:
    """Tests for get_system_message channel configuration."""

    def test_commentary_channel_present_without_custom_tools(self) -> None:
        """Commentary channel must be valid even without custom tools."""
        sys_msg = get_system_message(with_custom_tools=False)
        valid_channels = sys_msg.content[0].channel_config.valid_channels
        assert "commentary" in valid_channels

    def test_commentary_channel_present_with_custom_tools(self) -> None:
        """Commentary channel present when custom tools are enabled."""
        sys_msg = get_system_message(with_custom_tools=True)
        valid_channels = sys_msg.content[0].channel_config.valid_channels
        assert "commentary" in valid_channels
# ... 9 lines omitted for brevity ...
                )

    def test_unsupported_reasoning_effort_raises_clear_error(self) -> None:
        with pytest.raises(
            ValueError,
            match="reasoning_effort='max' is not supported by Harmony",
        ):
            get_system_message(reasoning_effort="max")
```
**EN:** This class groups related scenarios in `TestGetSystemMessage`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_commentary_channel_present_without_custom_tools`, `test_commentary_channel_present_with_custom_tools`, `test_all_standard_channels_present`.
**CN:** 该类将与 `TestGetSystemMessage` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_commentary_channel_present_without_custom_tools`、`test_commentary_channel_present_with_custom_tools`、`test_all_standard_channels_present`。

### Test method / 测试方法: TestGetSystemMessage.test_commentary_channel_present_without_custom_tools (L1002-L1006)
```python
    def test_commentary_channel_present_without_custom_tools(self) -> None:
        """Commentary channel must be valid even without custom tools."""
        sys_msg = get_system_message(with_custom_tools=False)
        valid_channels = sys_msg.content[0].channel_config.valid_channels
        assert "commentary" in valid_channels
```
**EN:** This test validates `TestGetSystemMessage.test_commentary_channel_present_without_custom_tools`. The main assertion is `'commentary' in valid_channels`.
**CN:** 这个测试验证 `TestGetSystemMessage.test_commentary_channel_present_without_custom_tools`。 核心断言是 `'commentary' in valid_channels`。

### Test method / 测试方法: TestGetSystemMessage.test_commentary_channel_present_with_custom_tools (L1008-L1012)
```python
    def test_commentary_channel_present_with_custom_tools(self) -> None:
        """Commentary channel present when custom tools are enabled."""
        sys_msg = get_system_message(with_custom_tools=True)
        valid_channels = sys_msg.content[0].channel_config.valid_channels
        assert "commentary" in valid_channels
```
**EN:** This test validates `TestGetSystemMessage.test_commentary_channel_present_with_custom_tools`. The main assertion is `'commentary' in valid_channels`.
**CN:** 这个测试验证 `TestGetSystemMessage.test_commentary_channel_present_with_custom_tools`。 核心断言是 `'commentary' in valid_channels`。

### Test method / 测试方法: TestGetSystemMessage.test_all_standard_channels_present (L1014-L1022)
```python
    def test_all_standard_channels_present(self) -> None:
        """All three standard Harmony channels should always be valid."""
        for with_tools in (True, False):
            sys_msg = get_system_message(with_custom_tools=with_tools)
            valid_channels = sys_msg.content[0].channel_config.valid_channels
            for channel in ("analysis", "commentary", "final"):
                assert channel in valid_channels, (
                    f"{channel} missing when with_custom_tools={with_tools}"
                )
```
**EN:** This test validates `TestGetSystemMessage.test_all_standard_channels_present`. The main assertion is `channel in valid_channels`.
**CN:** 这个测试验证 `TestGetSystemMessage.test_all_standard_channels_present`。 核心断言是 `channel in valid_channels`。

### Test method / 测试方法: TestGetSystemMessage.test_unsupported_reasoning_effort_raises_clear_error (L1024-L1029)
```python
    def test_unsupported_reasoning_effort_raises_clear_error(self) -> None:
        with pytest.raises(
            ValueError,
            match="reasoning_effort='max' is not supported by Harmony",
        ):
            get_system_message(reasoning_effort="max")
```
**EN:** This test validates `TestGetSystemMessage.test_unsupported_reasoning_effort_raises_clear_error`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `TestGetSystemMessage.test_unsupported_reasoning_effort_raises_clear_error`。 它使用 `pytest.raises` 检查预期失败路径。

### Class / 类: TestResponseInputToHarmonyReasoningItem (L1032-L1115)
```python
class TestResponseInputToHarmonyReasoningItem:
    """Tests for response_input_to_harmony handling of reasoning input items.

    Per the OpenAI spec, ResponseReasoningItem.content is
    Optional[List[Content]] = None. Clients like langchain-openai may omit
    this field when constructing multi-turn input from previous responses.

    Reasoning items with content are converted to Harmony messages on the
    'analysis' channel. All content items are concatenated. Items without
    content return None (skipped by the caller).
    """

    def test_reasoning_with_single_content(self):
        """Test reasoning item with a single content entry."""
# ... 62 lines omitted for brevity ...
            "type": "reasoning",
            "id": "rs_123",
            "content": [],
        }

        msg = response_input_to_harmony(item, prev_responses=[])

        assert msg is None
```
**EN:** This class groups related scenarios in `TestResponseInputToHarmonyReasoningItem`. It contains 5 test method(s) and 0 supporting method(s). Representative methods include `test_reasoning_with_single_content`, `test_reasoning_with_multiple_content_items`, `test_reasoning_without_content_returns_none`.
**CN:** 该类将与 `TestResponseInputToHarmonyReasoningItem` 相关的场景组织在一起。 它包含 5 个测试方法和 0 个辅助方法。 代表性方法包括 `test_reasoning_with_single_content`、`test_reasoning_with_multiple_content_items`、`test_reasoning_without_content_returns_none`。

### Test method / 测试方法: TestResponseInputToHarmonyReasoningItem.test_reasoning_with_single_content (L1044-L1057)
```python
    def test_reasoning_with_single_content(self):
        """Test reasoning item with a single content entry."""
        item = {
            "type": "reasoning",
            "id": "rs_123",
            "content": [{"type": "reasoning_text", "text": "Thinking step by step"}],
        }

        msg = response_input_to_harmony(item, prev_responses=[])

        assert msg is not None
        assert msg.author.role == Role.ASSISTANT
        assert msg.content[0].text == "Thinking step by step"
        assert msg.channel == "analysis"
```
**EN:** This test validates `TestResponseInputToHarmonyReasoningItem.test_reasoning_with_single_content`. The main assertion is `msg is not None` and `msg.author.role == Role.ASSISTANT`.
**CN:** 这个测试验证 `TestResponseInputToHarmonyReasoningItem.test_reasoning_with_single_content`。 核心断言是 `msg is not None` and `msg.author.role == Role.ASSISTANT`。

### Test method / 测试方法: TestResponseInputToHarmonyReasoningItem.test_reasoning_with_multiple_content_items (L1059-L1078)
```python
    def test_reasoning_with_multiple_content_items(self):
        """Test reasoning item with multiple content entries concatenated."""
        item = {
            "type": "reasoning",
            "id": "rs_123",
            "content": [
                {"type": "reasoning_text", "text": "First, let me analyze"},
                {"type": "reasoning_text", "text": "Second, I should consider"},
                {"type": "reasoning_text", "text": "Finally, the answer is"},
            ],
        }

        msg = response_input_to_harmony(item, prev_responses=[])

        assert msg is not None
        assert msg.author.role == Role.ASSISTANT
        assert msg.content[0].text == (
            "First, let me analyze\nSecond, I should consider\nFinally, the answer is"
        )
        assert msg.channel == "analysis"
```
**EN:** This test validates `TestResponseInputToHarmonyReasoningItem.test_reasoning_with_multiple_content_items`. The main assertion is `msg is not None` and `msg.author.role == Role.ASSISTANT`.
**CN:** 这个测试验证 `TestResponseInputToHarmonyReasoningItem.test_reasoning_with_multiple_content_items`。 核心断言是 `msg is not None` and `msg.author.role == Role.ASSISTANT`。

### Test method / 测试方法: TestResponseInputToHarmonyReasoningItem.test_reasoning_without_content_returns_none (L1080-L1090)
```python
    def test_reasoning_without_content_returns_none(self):
        """Test reasoning item without content field returns None."""
        item = {
            "type": "reasoning",
            "id": "rs_123",
            "summary": [{"type": "summary_text", "text": "Thinking about math"}],
        }

        msg = response_input_to_harmony(item, prev_responses=[])

        assert msg is None
```
**EN:** This test validates `TestResponseInputToHarmonyReasoningItem.test_reasoning_without_content_returns_none`. The main assertion is `msg is None`.
**CN:** 这个测试验证 `TestResponseInputToHarmonyReasoningItem.test_reasoning_without_content_returns_none`。 核心断言是 `msg is None`。

### Test method / 测试方法: TestResponseInputToHarmonyReasoningItem.test_reasoning_with_none_content_returns_none (L1092-L1103)
```python
    def test_reasoning_with_none_content_returns_none(self):
        """Test reasoning item with content=None returns None."""
        item = {
            "type": "reasoning",
            "id": "rs_123",
            "content": None,
            "summary": [{"type": "summary_text", "text": "Thinking about math"}],
        }

        msg = response_input_to_harmony(item, prev_responses=[])

        assert msg is None
```
**EN:** This test validates `TestResponseInputToHarmonyReasoningItem.test_reasoning_with_none_content_returns_none`. The main assertion is `msg is None`.
**CN:** 这个测试验证 `TestResponseInputToHarmonyReasoningItem.test_reasoning_with_none_content_returns_none`。 核心断言是 `msg is None`。

### Test method / 测试方法: TestResponseInputToHarmonyReasoningItem.test_reasoning_with_empty_content_returns_none (L1105-L1115)
```python
    def test_reasoning_with_empty_content_returns_none(self):
        """Test reasoning item with empty content list returns None."""
        item = {
            "type": "reasoning",
            "id": "rs_123",
            "content": [],
        }

        msg = response_input_to_harmony(item, prev_responses=[])

        assert msg is None
```
**EN:** This test validates `TestResponseInputToHarmonyReasoningItem.test_reasoning_with_empty_content_returns_none`. The main assertion is `msg is None`.
**CN:** 这个测试验证 `TestResponseInputToHarmonyReasoningItem.test_reasoning_with_empty_content_returns_none`。 核心断言是 `msg is None`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `openai_harmony.Message`, `openai_harmony.Role`, `pytest`
- **Project / 项目内**: `tests.entrypoints.openai.utils.verify_harmony_messages`, `vllm.entrypoints.openai.parser.harmony_utils.auto_drop_analysis_messages`, `vllm.entrypoints.openai.parser.harmony_utils.extract_function_from_recipient`, `vllm.entrypoints.openai.parser.harmony_utils.get_encoding`, `vllm.entrypoints.openai.parser.harmony_utils.get_system_message`, `vllm.entrypoints.openai.parser.harmony_utils.has_custom_tools`, `vllm.entrypoints.openai.parser.harmony_utils.is_function_recipient`, `vllm.entrypoints.openai.parser.harmony_utils.parse_chat_input_to_harmony_message`, `vllm.entrypoints.openai.parser.harmony_utils.parse_chat_output`, `vllm.entrypoints.openai.responses.harmony.response_input_to_harmony`, `vllm.entrypoints.openai.responses.harmony.response_previous_input_to_harmony`
