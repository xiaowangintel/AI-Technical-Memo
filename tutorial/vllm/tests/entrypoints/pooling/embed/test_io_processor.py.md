# test_io_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_io_processor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and process management. The file defines 28 test(s), 0 fixture(s), and 21 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与进程管理。它定义了 28 个测试、0 个 fixture，以及 21 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L14)
```python
import pytest

from vllm import PoolingParams
from vllm.entrypoints.pooling.embed.io_processor import EmbedIOProcessor
from vllm.entrypoints.pooling.embed.protocol import (
    CohereEmbedContent,
    CohereEmbedInput,
    CohereEmbedRequest,
)
from vllm.entrypoints.pooling.typing import PoolingServeContext
```
**EN:** Imports third-party packages like `pytest`, project helpers such as `vllm.PoolingParams`, `vllm.entrypoints.pooling.embed.io_processor.EmbedIOProcessor`, `vllm.entrypoints.pooling.embed.protocol.CohereEmbedContent`.
**CN:** 导入第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.PoolingParams`、`vllm.entrypoints.pooling.embed.io_processor.EmbedIOProcessor`、`vllm.entrypoints.pooling.embed.protocol.CohereEmbedContent`）。

### Class / 类: TestResolveTruncation (L17-L71)
```python
class TestResolveTruncation:
    """Unit tests for EmbedIOProcessor._resolve_cohere_truncation."""

    @staticmethod
    def _make_request(**kwargs) -> CohereEmbedRequest:
        defaults = {
            "model": "test",
            "input_type": "search_document",
            "texts": ["hello"],
        }
        return CohereEmbedRequest(**(defaults | kwargs))

    def test_truncate_end_default(self):
        req = self._make_request()
# ... 33 lines omitted for brevity ...
        assert tokens == -1
        assert side == "left"

    def test_truncate_start_with_max_tokens(self):
        req = self._make_request(truncate="START", max_tokens=64)
        tokens, side = EmbedIOProcessor._resolve_cohere_truncation(req)
        assert tokens == 64
        assert side == "left"
```
**EN:** This class groups related scenarios in `TestResolveTruncation`. It contains 7 test method(s) and 1 supporting method(s). Representative methods include `test_truncate_end_default`, `test_truncate_end_explicit`, `test_truncate_end_with_max_tokens`, `_make_request`.
**CN:** 该类将与 `TestResolveTruncation` 相关的场景组织在一起。 它包含 7 个测试方法和 1 个辅助方法。 代表性方法包括 `test_truncate_end_default`、`test_truncate_end_explicit`、`test_truncate_end_with_max_tokens`、`_make_request`。

### Helper method / 辅助方法: TestResolveTruncation._make_request (L20-L27)
```python
    @staticmethod
    def _make_request(**kwargs) -> CohereEmbedRequest:
        defaults = {
            "model": "test",
            "input_type": "search_document",
            "texts": ["hello"],
        }
        return CohereEmbedRequest(**(defaults | kwargs))
```
**EN:** This helper encapsulates reusable logic in `TestResolveTruncation._make_request`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestResolveTruncation._make_request` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestResolveTruncation.test_truncate_end_default (L29-L33)
```python
    def test_truncate_end_default(self):
        req = self._make_request()
        tokens, side = EmbedIOProcessor._resolve_cohere_truncation(req)
        assert tokens == -1
        assert side is None
```
**EN:** This test validates `TestResolveTruncation.test_truncate_end_default`. The main assertion is `tokens == -1` and `side is None`.
**CN:** 这个测试验证 `TestResolveTruncation.test_truncate_end_default`。 核心断言是 `tokens == -1` and `side is None`。

### Test method / 测试方法: TestResolveTruncation.test_truncate_end_explicit (L35-L39)
```python
    def test_truncate_end_explicit(self):
        req = self._make_request(truncate="END")
        tokens, side = EmbedIOProcessor._resolve_cohere_truncation(req)
        assert tokens == -1
        assert side is None
```
**EN:** This test validates `TestResolveTruncation.test_truncate_end_explicit`. The main assertion is `tokens == -1` and `side is None`.
**CN:** 这个测试验证 `TestResolveTruncation.test_truncate_end_explicit`。 核心断言是 `tokens == -1` and `side is None`。

### Test method / 测试方法: TestResolveTruncation.test_truncate_end_with_max_tokens (L41-L45)
```python
    def test_truncate_end_with_max_tokens(self):
        req = self._make_request(truncate="END", max_tokens=128)
        tokens, side = EmbedIOProcessor._resolve_cohere_truncation(req)
        assert tokens == 128
        assert side is None
```
**EN:** This test validates `TestResolveTruncation.test_truncate_end_with_max_tokens`. The main assertion is `tokens == 128` and `side is None`.
**CN:** 这个测试验证 `TestResolveTruncation.test_truncate_end_with_max_tokens`。 核心断言是 `tokens == 128` and `side is None`。

### Test method / 测试方法: TestResolveTruncation.test_truncate_none (L47-L51)
```python
    def test_truncate_none(self):
        req = self._make_request(truncate="NONE")
        tokens, side = EmbedIOProcessor._resolve_cohere_truncation(req)
        assert tokens is None
        assert side is None
```
**EN:** This test validates `TestResolveTruncation.test_truncate_none`. The main assertion is `tokens is None` and `side is None`.
**CN:** 这个测试验证 `TestResolveTruncation.test_truncate_none`。 核心断言是 `tokens is None` and `side is None`。

### Test method / 测试方法: TestResolveTruncation.test_truncate_none_with_max_tokens (L53-L59)
```python
    def test_truncate_none_with_max_tokens(self):
        """truncate=NONE should NOT set truncate_prompt_tokens; the
        max_tokens limit is enforced separately via _check_max_tokens."""
        req = self._make_request(truncate="NONE", max_tokens=10)
        tokens, side = EmbedIOProcessor._resolve_cohere_truncation(req)
        assert tokens is None
        assert side is None
```
**EN:** This test validates `TestResolveTruncation.test_truncate_none_with_max_tokens`. The main assertion is `tokens is None` and `side is None`.
**CN:** 这个测试验证 `TestResolveTruncation.test_truncate_none_with_max_tokens`。 核心断言是 `tokens is None` and `side is None`。

### Test method / 测试方法: TestResolveTruncation.test_truncate_start (L61-L65)
```python
    def test_truncate_start(self):
        req = self._make_request(truncate="START")
        tokens, side = EmbedIOProcessor._resolve_cohere_truncation(req)
        assert tokens == -1
        assert side == "left"
```
**EN:** This test validates `TestResolveTruncation.test_truncate_start`. The main assertion is `tokens == -1` and `side == 'left'`.
**CN:** 这个测试验证 `TestResolveTruncation.test_truncate_start`。 核心断言是 `tokens == -1` and `side == 'left'`。

### Test method / 测试方法: TestResolveTruncation.test_truncate_start_with_max_tokens (L67-L71)
```python
    def test_truncate_start_with_max_tokens(self):
        req = self._make_request(truncate="START", max_tokens=64)
        tokens, side = EmbedIOProcessor._resolve_cohere_truncation(req)
        assert tokens == 64
        assert side == "left"
```
**EN:** This test validates `TestResolveTruncation.test_truncate_start_with_max_tokens`. The main assertion is `tokens == 64` and `side == 'left'`.
**CN:** 这个测试验证 `TestResolveTruncation.test_truncate_start_with_max_tokens`。 核心断言是 `tokens == 64` and `side == 'left'`。

### Class / 类: TestApplyStPrompt (L74-L112)
```python
class TestApplyStPrompt:
    """Unit tests for EmbedIOProcessor._apply_task_instruction."""

    @staticmethod
    def _make_handler(task_instructions: dict[str, str] | None):
        handler = object.__new__(EmbedIOProcessor)
        handler.task_instructions = task_instructions
        return handler

    def test_no_prompts_configured(self):
        handler = self._make_handler(None)
        texts = ["hello", "world"]
        assert handler._apply_task_instruction(texts, "query") is texts

# ... 17 lines omitted for brevity ...
            "Represent this sentence for searching: b",
            "Represent this sentence for searching: c",
        ]

    def test_empty_prefix_returns_unchanged(self):
        handler = self._make_handler({"passage": ""})
        texts = ["hello"]
        assert handler._apply_task_instruction(texts, "passage") is texts
```
**EN:** This class groups related scenarios in `TestApplyStPrompt`. It contains 5 test method(s) and 1 supporting method(s). Representative methods include `test_no_prompts_configured`, `test_matching_input_type`, `test_non_matching_input_type`, `_make_handler`.
**CN:** 该类将与 `TestApplyStPrompt` 相关的场景组织在一起。 它包含 5 个测试方法和 1 个辅助方法。 代表性方法包括 `test_no_prompts_configured`、`test_matching_input_type`、`test_non_matching_input_type`、`_make_handler`。

### Helper method / 辅助方法: TestApplyStPrompt._make_handler (L77-L81)
```python
    @staticmethod
    def _make_handler(task_instructions: dict[str, str] | None):
        handler = object.__new__(EmbedIOProcessor)
        handler.task_instructions = task_instructions
        return handler
```
**EN:** This helper encapsulates reusable logic in `TestApplyStPrompt._make_handler`. Key inputs are `task_instructions`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestApplyStPrompt._make_handler` 中。 关键输入包括 `task_instructions`。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestApplyStPrompt.test_no_prompts_configured (L83-L86)
```python
    def test_no_prompts_configured(self):
        handler = self._make_handler(None)
        texts = ["hello", "world"]
        assert handler._apply_task_instruction(texts, "query") is texts
```
**EN:** This test validates `TestApplyStPrompt.test_no_prompts_configured`. The main assertion is `handler._apply_task_instruction(texts, 'query') is texts`.
**CN:** 这个测试验证 `TestApplyStPrompt.test_no_prompts_configured`。 核心断言是 `handler._apply_task_instruction(texts, 'query') is texts`。

### Test method / 测试方法: TestApplyStPrompt.test_matching_input_type (L88-L91)
```python
    def test_matching_input_type(self):
        handler = self._make_handler({"query": "search_query: "})
        result = handler._apply_task_instruction(["hello"], "query")
        assert result == ["search_query: hello"]
```
**EN:** This test validates `TestApplyStPrompt.test_matching_input_type`. The main assertion is `result == ['search_query: hello']`.
**CN:** 这个测试验证 `TestApplyStPrompt.test_matching_input_type`。 核心断言是 `result == ['search_query: hello']`。

### Test method / 测试方法: TestApplyStPrompt.test_non_matching_input_type (L93-L96)
```python
    def test_non_matching_input_type(self):
        handler = self._make_handler({"query": "search_query: "})
        texts = ["hello"]
        assert handler._apply_task_instruction(texts, "document") is texts
```
**EN:** This test validates `TestApplyStPrompt.test_non_matching_input_type`. The main assertion is `handler._apply_task_instruction(texts, 'document') is texts`.
**CN:** 这个测试验证 `TestApplyStPrompt.test_non_matching_input_type`。 核心断言是 `handler._apply_task_instruction(texts, 'document') is texts`。

### Test method / 测试方法: TestApplyStPrompt.test_multiple_texts (L98-L107)
```python
    def test_multiple_texts(self):
        handler = self._make_handler(
            {"query": "Represent this sentence for searching: "}
        )
        result = handler._apply_task_instruction(["a", "b", "c"], "query")
        assert result == [
            "Represent this sentence for searching: a",
            "Represent this sentence for searching: b",
            "Represent this sentence for searching: c",
        ]
```
**EN:** This test validates `TestApplyStPrompt.test_multiple_texts`. The main assertion is `result == ['Represent this sentence for searching: a', 'Represent this sentence for searching: b', 'Represent this sentence for searching...`.
**CN:** 这个测试验证 `TestApplyStPrompt.test_multiple_texts`。 核心断言是 `result == ['Represent this sentence for searching: a', 'Represent this sentence for searching: b', 'Represent this sentence for searching...`。

### Test method / 测试方法: TestApplyStPrompt.test_empty_prefix_returns_unchanged (L109-L112)
```python
    def test_empty_prefix_returns_unchanged(self):
        handler = self._make_handler({"passage": ""})
        texts = ["hello"]
        assert handler._apply_task_instruction(texts, "passage") is texts
```
**EN:** This test validates `TestApplyStPrompt.test_empty_prefix_returns_unchanged`. The main assertion is `handler._apply_task_instruction(texts, 'passage') is texts`.
**CN:** 这个测试验证 `TestApplyStPrompt.test_empty_prefix_returns_unchanged`。 核心断言是 `handler._apply_task_instruction(texts, 'passage') is texts`。

### Class / 类: TestLoadTaskInstructions (L115-L147)
```python
class TestLoadTaskInstructions:
    """Unit tests for EmbedIOProcessor._load_task_instructions."""

    def test_no_attribute(self):
        class FakeConfig:
            pass

        assert EmbedIOProcessor._load_task_instructions(FakeConfig()) is None

    def test_with_task_instructions(self):
        class FakeConfig:
            task_instructions = {
                "retrieval.query": "Represent the query: ",
                "retrieval.passage": "",
# ... 11 lines omitted for brevity ...

        assert EmbedIOProcessor._load_task_instructions(FakeConfig()) is None

    def test_non_dict(self):
        class FakeConfig:
            task_instructions = "not a dict"

        assert EmbedIOProcessor._load_task_instructions(FakeConfig()) is None
```
**EN:** This class groups related scenarios in `TestLoadTaskInstructions`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_no_attribute`, `test_with_task_instructions`, `test_empty_dict`.
**CN:** 该类将与 `TestLoadTaskInstructions` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_no_attribute`、`test_with_task_instructions`、`test_empty_dict`。

### Test method / 测试方法: TestLoadTaskInstructions.test_no_attribute (L118-L122)
```python
    def test_no_attribute(self):
        class FakeConfig:
            pass

        assert EmbedIOProcessor._load_task_instructions(FakeConfig()) is None
```
**EN:** This test validates `TestLoadTaskInstructions.test_no_attribute`. The main assertion is `EmbedIOProcessor._load_task_instructions(FakeConfig()) is None`.
**CN:** 这个测试验证 `TestLoadTaskInstructions.test_no_attribute`。 核心断言是 `EmbedIOProcessor._load_task_instructions(FakeConfig()) is None`。

### Test method / 测试方法: TestLoadTaskInstructions.test_with_task_instructions (L124-L135)
```python
    def test_with_task_instructions(self):
        class FakeConfig:
            task_instructions = {
                "retrieval.query": "Represent the query: ",
                "retrieval.passage": "",
            }

        result = EmbedIOProcessor._load_task_instructions(FakeConfig())
        assert result == {
            "retrieval.query": "Represent the query: ",
            "retrieval.passage": "",
        }
```
**EN:** This test validates `TestLoadTaskInstructions.test_with_task_instructions`. The main assertion is `result == {'retrieval.query': 'Represent the query: ', 'retrieval.passage': ''}`.
**CN:** 这个测试验证 `TestLoadTaskInstructions.test_with_task_instructions`。 核心断言是 `result == {'retrieval.query': 'Represent the query: ', 'retrieval.passage': ''}`。

### Test method / 测试方法: TestLoadTaskInstructions.test_empty_dict (L137-L141)
```python
    def test_empty_dict(self):
        class FakeConfig:
            task_instructions = {}

        assert EmbedIOProcessor._load_task_instructions(FakeConfig()) is None
```
**EN:** This test validates `TestLoadTaskInstructions.test_empty_dict`. The main assertion is `EmbedIOProcessor._load_task_instructions(FakeConfig()) is None`.
**CN:** 这个测试验证 `TestLoadTaskInstructions.test_empty_dict`。 核心断言是 `EmbedIOProcessor._load_task_instructions(FakeConfig()) is None`。

### Test method / 测试方法: TestLoadTaskInstructions.test_non_dict (L143-L147)
```python
    def test_non_dict(self):
        class FakeConfig:
            task_instructions = "not a dict"

        assert EmbedIOProcessor._load_task_instructions(FakeConfig()) is None
```
**EN:** This test validates `TestLoadTaskInstructions.test_non_dict`. The main assertion is `EmbedIOProcessor._load_task_instructions(FakeConfig()) is None`.
**CN:** 这个测试验证 `TestLoadTaskInstructions.test_non_dict`。 核心断言是 `EmbedIOProcessor._load_task_instructions(FakeConfig()) is None`。

### Class / 类: TestCheckMaxTokens (L150-L176)
```python
class TestCheckMaxTokens:
    """Unit tests for EmbedIOProcessor._check_cohere_max_tokens."""

    @staticmethod
    def _fake_output(n_tokens: int):
        class _Out:
            def __init__(self, n: int):
                self.prompt_token_ids = list(range(n))

        return _Out(n_tokens)

    def test_none_check_is_noop(self):
        outs = [self._fake_output(100)]
        EmbedIOProcessor._check_cohere_max_tokens(outs, None)
# ... 5 lines omitted for brevity ...
    def test_exceeds_limit(self):
        outs = [self._fake_output(3), self._fake_output(10)]
        with pytest.raises(ValueError, match="exceeds max_tokens=5"):
            EmbedIOProcessor._check_cohere_max_tokens(outs, 5)

    def test_exact_limit(self):
        outs = [self._fake_output(5)]
        EmbedIOProcessor._check_cohere_max_tokens(outs, 5)
```
**EN:** This class groups related scenarios in `TestCheckMaxTokens`. It contains 4 test method(s) and 1 supporting method(s). Representative methods include `test_none_check_is_noop`, `test_within_limit`, `test_exceeds_limit`, `_fake_output`.
**CN:** 该类将与 `TestCheckMaxTokens` 相关的场景组织在一起。 它包含 4 个测试方法和 1 个辅助方法。 代表性方法包括 `test_none_check_is_noop`、`test_within_limit`、`test_exceeds_limit`、`_fake_output`。

### Helper method / 辅助方法: TestCheckMaxTokens._fake_output (L153-L159)
```python
    @staticmethod
    def _fake_output(n_tokens: int):
        class _Out:
            def __init__(self, n: int):
                self.prompt_token_ids = list(range(n))

        return _Out(n_tokens)
```
**EN:** This helper encapsulates reusable logic in `TestCheckMaxTokens._fake_output`. Key inputs are `n_tokens`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestCheckMaxTokens._fake_output` 中。 关键输入包括 `n_tokens`。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestCheckMaxTokens.test_none_check_is_noop (L161-L163)
```python
    def test_none_check_is_noop(self):
        outs = [self._fake_output(100)]
        EmbedIOProcessor._check_cohere_max_tokens(outs, None)
```
**EN:** This test validates `TestCheckMaxTokens.test_none_check_is_noop`.
**CN:** 这个测试验证 `TestCheckMaxTokens.test_none_check_is_noop`。

### Test method / 测试方法: TestCheckMaxTokens.test_within_limit (L165-L167)
```python
    def test_within_limit(self):
        outs = [self._fake_output(5), self._fake_output(3)]
        EmbedIOProcessor._check_cohere_max_tokens(outs, 5)
```
**EN:** This test validates `TestCheckMaxTokens.test_within_limit`.
**CN:** 这个测试验证 `TestCheckMaxTokens.test_within_limit`。

### Test method / 测试方法: TestCheckMaxTokens.test_exceeds_limit (L169-L172)
```python
    def test_exceeds_limit(self):
        outs = [self._fake_output(3), self._fake_output(10)]
        with pytest.raises(ValueError, match="exceeds max_tokens=5"):
            EmbedIOProcessor._check_cohere_max_tokens(outs, 5)
```
**EN:** This test validates `TestCheckMaxTokens.test_exceeds_limit`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `TestCheckMaxTokens.test_exceeds_limit`。 它使用 `pytest.raises` 检查预期失败路径。

### Test method / 测试方法: TestCheckMaxTokens.test_exact_limit (L174-L176)
```python
    def test_exact_limit(self):
        outs = [self._fake_output(5)]
        EmbedIOProcessor._check_cohere_max_tokens(outs, 5)
```
**EN:** This test validates `TestCheckMaxTokens.test_exact_limit`.
**CN:** 这个测试验证 `TestCheckMaxTokens.test_exact_limit`。

### Class / 类: TestValidateInputType (L179-L212)
```python
class TestValidateInputType:
    """Unit tests for EmbedIOProcessor._validate_input_type."""

    @staticmethod
    def _make_handler(task_instructions: dict[str, str] | None):
        handler = object.__new__(EmbedIOProcessor)
        handler.task_instructions = task_instructions
        return handler

    def test_none_input_type_always_accepted(self):
        handler = self._make_handler(None)
        handler._validate_input_type(None)
        handler_with = self._make_handler({"query": "q: "})
        handler_with._validate_input_type(None)
# ... 12 lines omitted for brevity ...
        handler = self._make_handler({"query": "q: ", "document": "d: "})
        with pytest.raises(ValueError, match="Unsupported input_type 'other'"):
            handler._validate_input_type("other")

    def test_error_lists_supported(self):
        handler = self._make_handler({"a": "", "b": ""})
        with pytest.raises(ValueError, match="Supported values: a, b"):
            handler._validate_input_type("z")
```
**EN:** This class groups related scenarios in `TestValidateInputType`. It contains 5 test method(s) and 1 supporting method(s). Representative methods include `test_none_input_type_always_accepted`, `test_no_prompts_rejects`, `test_known_type_accepted`, `_make_handler`.
**CN:** 该类将与 `TestValidateInputType` 相关的场景组织在一起。 它包含 5 个测试方法和 1 个辅助方法。 代表性方法包括 `test_none_input_type_always_accepted`、`test_no_prompts_rejects`、`test_known_type_accepted`、`_make_handler`。

### Helper method / 辅助方法: TestValidateInputType._make_handler (L182-L186)
```python
    @staticmethod
    def _make_handler(task_instructions: dict[str, str] | None):
        handler = object.__new__(EmbedIOProcessor)
        handler.task_instructions = task_instructions
        return handler
```
**EN:** This helper encapsulates reusable logic in `TestValidateInputType._make_handler`. Key inputs are `task_instructions`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestValidateInputType._make_handler` 中。 关键输入包括 `task_instructions`。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestValidateInputType.test_none_input_type_always_accepted (L188-L192)
```python
    def test_none_input_type_always_accepted(self):
        handler = self._make_handler(None)
        handler._validate_input_type(None)
        handler_with = self._make_handler({"query": "q: "})
        handler_with._validate_input_type(None)
```
**EN:** This test validates `TestValidateInputType.test_none_input_type_always_accepted`.
**CN:** 这个测试验证 `TestValidateInputType.test_none_input_type_always_accepted`。

### Test method / 测试方法: TestValidateInputType.test_no_prompts_rejects (L194-L197)
```python
    def test_no_prompts_rejects(self):
        handler = self._make_handler(None)
        with pytest.raises(ValueError, match="does not define any input_type"):
            handler._validate_input_type("anything")
```
**EN:** This test validates `TestValidateInputType.test_no_prompts_rejects`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `TestValidateInputType.test_no_prompts_rejects`。 它使用 `pytest.raises` 检查预期失败路径。

### Test method / 测试方法: TestValidateInputType.test_known_type_accepted (L199-L202)
```python
    def test_known_type_accepted(self):
        handler = self._make_handler({"query": "q: ", "document": "d: "})
        handler._validate_input_type("query")
        handler._validate_input_type("document")
```
**EN:** This test validates `TestValidateInputType.test_known_type_accepted`.
**CN:** 这个测试验证 `TestValidateInputType.test_known_type_accepted`。

### Test method / 测试方法: TestValidateInputType.test_unknown_type_rejected (L204-L207)
```python
    def test_unknown_type_rejected(self):
        handler = self._make_handler({"query": "q: ", "document": "d: "})
        with pytest.raises(ValueError, match="Unsupported input_type 'other'"):
            handler._validate_input_type("other")
```
**EN:** This test validates `TestValidateInputType.test_unknown_type_rejected`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `TestValidateInputType.test_unknown_type_rejected`。 它使用 `pytest.raises` 检查预期失败路径。

### Test method / 测试方法: TestValidateInputType.test_error_lists_supported (L209-L212)
```python
    def test_error_lists_supported(self):
        handler = self._make_handler({"a": "", "b": ""})
        with pytest.raises(ValueError, match="Supported values: a, b"):
            handler._validate_input_type("z")
```
**EN:** This test validates `TestValidateInputType.test_error_lists_supported`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `TestValidateInputType.test_error_lists_supported`。 它使用 `pytest.raises` 检查预期失败路径。

### Class / 类: TestPreProcessCohereOnline (L215-L326)
```python
class TestPreProcessCohereOnline:
    """Unit tests for EmbedIOProcessor._pre_process_cohere_online."""

    @staticmethod
    def _make_context(**request_kwargs) -> PoolingServeContext[CohereEmbedRequest]:
        return PoolingServeContext(
            request=CohereEmbedRequest(model="test", **request_kwargs),
            pooling_params=PoolingParams(),
            model_name="test",
            request_id="embd-test",
        )

    @staticmethod
    def _make_handler():
# ... 90 lines omitted for brevity ...
                            task_prefix="query: ",
                        )
                    ],
                    "truncate_prompt_tokens": -1,
                    "truncation_side": None,
                },
            )
        ]
```
**EN:** This class groups related scenarios in `TestPreProcessCohereOnline`. It contains 3 test method(s) and 2 supporting method(s). Representative methods include `test_text_only_without_task_prefix_uses_completion_path`, `test_text_only_falls_back_to_prefixed_completion_without_template`, `test_text_only_with_template_uses_chat_path`, `_make_context`, `_make_handler`.
**CN:** 该类将与 `TestPreProcessCohereOnline` 相关的场景组织在一起。 它包含 3 个测试方法和 2 个辅助方法。 代表性方法包括 `test_text_only_without_task_prefix_uses_completion_path`、`test_text_only_falls_back_to_prefixed_completion_without_template`、`test_text_only_with_template_uses_chat_path`、`_make_context`、`_make_handler`。

### Helper method / 辅助方法: TestPreProcessCohereOnline._make_context (L218-L225)
```python
    @staticmethod
    def _make_context(**request_kwargs) -> PoolingServeContext[CohereEmbedRequest]:
        return PoolingServeContext(
            request=CohereEmbedRequest(model="test", **request_kwargs),
            pooling_params=PoolingParams(),
            model_name="test",
            request_id="embd-test",
        )
```
**EN:** This helper encapsulates reusable logic in `TestPreProcessCohereOnline._make_context`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestPreProcessCohereOnline._make_context` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: TestPreProcessCohereOnline._make_handler (L227-L231)
```python
    @staticmethod
    def _make_handler():
        handler = object.__new__(EmbedIOProcessor)
        handler._validate_input_type = lambda _input_type: None
        return handler
```
**EN:** This helper encapsulates reusable logic in `TestPreProcessCohereOnline._make_handler`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestPreProcessCohereOnline._make_handler` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestPreProcessCohereOnline.test_text_only_without_task_prefix_uses_completion_path (L233-L252)
```python
    def test_text_only_without_task_prefix_uses_completion_path(self):
        handler = self._make_handler()
        ctx = self._make_context(texts=["hello"])
        calls: list[tuple[str, object]] = []

        def preprocess_cmpl_online(request, prompt_input, prompt_embeds):
            calls.append(("completion", prompt_input))
            return ["completion"]

        handler._get_task_instruction_prefix = lambda _input_type: None
        handler._has_chat_template = lambda: False
        handler._preprocess_cmpl_online = preprocess_cmpl_online
        handler._batch_render_chat = lambda *_args, **_kwargs: (
            pytest.fail("text-only request should not require chat rendering")
        )

        handler._pre_process_cohere_online(ctx)

        assert ctx.engine_inputs == ["completion"]
        assert calls == [("completion", ["hello"])]
```
**EN:** This test validates `TestPreProcessCohereOnline.test_text_only_without_task_prefix_uses_completion_path`. The main assertion is `ctx.engine_inputs == ['completion']` and `calls == [('completion', ['hello'])]`.
**CN:** 这个测试验证 `TestPreProcessCohereOnline.test_text_only_without_task_prefix_uses_completion_path`。 核心断言是 `ctx.engine_inputs == ['completion']` and `calls == [('completion', ['hello'])]`。

### Test method / 测试方法: TestPreProcessCohereOnline.test_text_only_falls_back_to_prefixed_completion_without_template (L254-L273)
```python
    def test_text_only_falls_back_to_prefixed_completion_without_template(self):
        handler = self._make_handler()
        ctx = self._make_context(texts=["hello"], input_type="query")
        calls: list[tuple[str, object]] = []

        def preprocess_cmpl(request, prompt_input, prompt_embeds):
            calls.append(("completion", prompt_input))
            return ["fallback"]

        handler._get_task_instruction_prefix = lambda _input_type: "query: "
        handler._has_chat_template = lambda: False
        handler._batch_render_chat = lambda *_args, **_kwargs: (
            pytest.fail("chat rendering should be skipped without a template")
        )
        handler._preprocess_cmpl_online = preprocess_cmpl

        handler._pre_process_cohere_online(ctx)

        assert ctx.engine_inputs == ["fallback"]
        assert calls == [("completion", ["query: hello"])]
```
**EN:** This test validates `TestPreProcessCohereOnline.test_text_only_falls_back_to_prefixed_completion_without_template`. The main assertion is `ctx.engine_inputs == ['fallback']` and `calls == [('completion', ['query: hello'])]`.
**CN:** 这个测试验证 `TestPreProcessCohereOnline.test_text_only_falls_back_to_prefixed_completion_without_template`。 核心断言是 `ctx.engine_inputs == ['fallback']` and `calls == [('completion', ['query: hello'])]`。

### Test method / 测试方法: TestPreProcessCohereOnline.test_text_only_with_template_uses_chat_path (L275-L326)
```python
    def test_text_only_with_template_uses_chat_path(self):
        handler = self._make_handler()
        ctx = self._make_context(texts=["hello"], input_type="query")
        calls: list[tuple[str, object]] = []

        def batch_render_chat(
            request,
            all_messages,
            truncate_prompt_tokens,
            truncation_side,
        ):
            calls.append(
                (
                    "chat",
                    {
                        "request": request,
                        "all_messages": all_messages,
                        "truncate_prompt_tokens": truncate_prompt_tokens,
# ... 26 lines omitted for brevity ...
                            task_prefix="query: ",
                        )
                    ],
                    "truncate_prompt_tokens": -1,
                    "truncation_side": None,
                },
            )
        ]
```
**EN:** This test validates `TestPreProcessCohereOnline.test_text_only_with_template_uses_chat_path`. The main assertion is `ctx.engine_inputs == ['chat']` and `calls == [('chat', {'request': ctx.request, 'all_messages': [handler._mixed_input_to_messages(CohereEmbedInput(content=[CohereEmbedConten...`.
**CN:** 这个测试验证 `TestPreProcessCohereOnline.test_text_only_with_template_uses_chat_path`。 核心断言是 `ctx.engine_inputs == ['chat']` and `calls == [('chat', {'request': ctx.request, 'all_messages': [handler._mixed_input_to_messages(CohereEmbedInput(content=[CohereEmbedConten...`。

## Key Concepts / 关键概念
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.PoolingParams`, `vllm.entrypoints.pooling.embed.io_processor.EmbedIOProcessor`, `vllm.entrypoints.pooling.embed.protocol.CohereEmbedContent`, `vllm.entrypoints.pooling.embed.protocol.CohereEmbedInput`, `vllm.entrypoints.pooling.embed.protocol.CohereEmbedRequest`, `vllm.entrypoints.pooling.typing.PoolingServeContext`
