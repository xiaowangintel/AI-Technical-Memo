# test_deepseek_v4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tokenizers_/test_deepseek_v4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Deepseek V4 behavior in the Tokenizers test area through focused pytest scenarios. It focuses on scenarios such as Fakehftokenizer, Tokenizer, Model Config. / 该文件在 Tokenizers 测试域中，通过有针对性的 pytest 场景验证 Deepseek V4 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-15)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json
from pathlib import Path
from types import SimpleNamespace

import pytest

from vllm.entrypoints.chat_utils import parse_chat_messages
from vllm.renderers.registry import RENDERER_REGISTRY
from vllm.tokenizers.deepseek_v4 import get_deepseek_v4_tokenizer
from vllm.tokenizers.registry import TokenizerRegistry

FIXTURES_DIR = Path(__file__).parent / "fixtures" / "deepseek_v4"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `pathlib`, `pytest`, `vllm.entrypoints.chat_utils`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: FakeHfTokenizer (lines 18-31)
```python
class FakeHfTokenizer:
    vocab_size = 100

    def get_added_vocab(self) -> dict[str, int]:
        return {"</think>": 100}

    def encode(
        self,
        text: str,
        add_special_tokens: bool = False,
        **kwargs,
    ) -> list[int]:
        self.last_encode = (text, add_special_tokens, kwargs)
        return [len(text)]
```
**EN:** Groups related scenarios for Fakehftokenizer.
**CN:** 该类把与 Fakehftokenizer 相关的场景组织在一起。

### Helper: _tokenizer (lines 34-35)
```python
def _tokenizer():
    return get_deepseek_v4_tokenizer(FakeHfTokenizer())
```
**EN:** Implements a reusable helper for Tokenizer, reducing duplication across related tests. It coordinates operations such as `get_deepseek_v4_tokenizer`, `FakeHfTokenizer`.
**CN:** 该辅助函数为 Tokenizer 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `get_deepseek_v4_tokenizer`, `FakeHfTokenizer` 等操作。

### Helper: _model_config (lines 38-44)
```python
def _model_config():
    return SimpleNamespace(
        multimodal_config=None,
        allowed_local_media_path="",
        allowed_media_domains=None,
        enable_prompt_embeds=False,
    )
```
**EN:** Implements a reusable helper for Model Config, reducing duplication across related tests. It coordinates operations such as `SimpleNamespace`.
**CN:** 该辅助函数为 Model Config 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `SimpleNamespace` 等操作。

### Test: test_deepseek_v4_tokenizer_registered (lines 70-76)
```python
def test_deepseek_v4_tokenizer_registered():
    assert TokenizerRegistry.load_tokenizer_cls("deepseek_v4").__name__ == (
        "DeepseekV4Tokenizer"
    )
    assert RENDERER_REGISTRY.load_renderer_cls("deepseek_v4").__name__ == (
        "DeepseekV4Renderer"
    )
```
**EN:** Checks Deepseek V4 Tokenizer Registered under a focused test scenario. The body exercises logic via `TokenizerRegistry.load_tokenizer_cls`, `RENDERER_REGISTRY.load_renderer_cls` before asserting the expected outcome.
**CN:** 该测试用例验证 Deepseek V4 Tokenizer Registered 在特定场景下的行为。 函数体会先通过 `TokenizerRegistry.load_tokenizer_cls`, `RENDERER_REGISTRY.load_renderer_cls` 驱动目标逻辑，再断言预期结果。

### Test: test_deepseek_v4_defaults_to_chat_mode (lines 79-85)
```python
def test_deepseek_v4_defaults_to_chat_mode():
    prompt = _tokenizer().apply_chat_template(
        [{"role": "user", "content": "Hello"}],
        tokenize=False,
    )

    assert prompt == ("<｜begin▁of▁sentence｜><｜User｜>Hello<｜Assistant｜></think>")
```
**EN:** Checks Deepseek V4 Defaults To Chat Mode under a focused test scenario. The body exercises logic via `_tokenizer().apply_chat_template`, `_tokenizer` before asserting the expected outcome.
**CN:** 该测试用例验证 Deepseek V4 Defaults To Chat Mode 在特定场景下的行为。 函数体会先通过 `_tokenizer().apply_chat_template`, `_tokenizer` 驱动目标逻辑，再断言预期结果。

### Test: test_deepseek_v4_enables_thinking_with_compatible_kwargs (lines 88-96)
```python
@pytest.mark.parametrize("kwargs", [{"thinking": True}, {"enable_thinking": True}])
def test_deepseek_v4_enables_thinking_with_compatible_kwargs(kwargs):
    prompt = _tokenizer().apply_chat_template(
        [{"role": "user", "content": "Hello"}],
        tokenize=False,
        **kwargs,
    )

    assert prompt == ("<｜begin▁of▁sentence｜><｜User｜>Hello<｜Assistant｜><think>")
```
**EN:** Checks Deepseek V4 Enables Thinking With Compatible Kwargs under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `_tokenizer().apply_chat_template`, `_tokenizer` before asserting the expected outcome.
**CN:** 该测试用例验证 Deepseek V4 Enables Thinking With Compatible Kwargs 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `_tokenizer().apply_chat_template`, `_tokenizer` 驱动目标逻辑，再断言预期结果。

### Test: test_deepseek_v4_uses_v4_tool_prompt_from_request_tools (lines 99-126)
```python
def test_deepseek_v4_uses_v4_tool_prompt_from_request_tools():
    tools = [
        {
            "type": "function",
            "function": {
                "name": "get_weather",
                "description": "Get weather for a city",
                "parameters": {
                    "type": "object",
                    "properties": {"city": {"type": "string"}},
                    "required": ["city"],
                },
            },
        }
    ]

    prompt = _tokenizer().apply_chat_template(
        [{"role": "user", "content": "Weather?"}],
        tools=tools,
        tokenize=False,
    )

    assert "## Tools" in prompt
    assert "<｜DSML｜tool_calls>" in prompt
    assert "</｜DSML｜tool_calls>" in prompt
    assert "function_calls" not in prompt
    assert '"name": "get_weather"' in prompt
    assert prompt.endswith("<｜User｜>Weather?<｜Assistant｜></think>")
```
**EN:** Checks Deepseek V4 Uses V4 Tool Prompt From Request Tools under a focused test scenario. The body exercises logic via `_tokenizer().apply_chat_template`, `prompt.endswith`, `_tokenizer` before asserting the expected outcome.
**CN:** 该测试用例验证 Deepseek V4 Uses V4 Tool Prompt From Request Tools 在特定场景下的行为。 函数体会先通过 `_tokenizer().apply_chat_template`, `prompt.endswith`, `_tokenizer` 驱动目标逻辑，再断言预期结果。

### Test: test_deepseek_v4_renders_parsed_history_tool_arguments (lines 129-183)
```python
def test_deepseek_v4_renders_parsed_history_tool_arguments():
    messages = [
        {"role": "user", "content": "List the repo"},
        {
            "role": "assistant",
            "tool_calls": [
                {
                    "id": "call_1",
                    "type": "function",
                    "function": {
                        "name": "str_replace_editor",
                        "arguments": '{"command": "view", "path": "/testbed"}',
                    },
                }
            ],
        },
        {
            "role": "tool",
            "tool_call_id": "call_1",
# ... omitted for brevity ...
        _model_config(),
        content_format="string",
    )

    prompt = _tokenizer().apply_chat_template(
        conversation=conversation,
        messages=messages,
        tools=tools,
        tokenize=False,
    )

    assert '<｜DSML｜parameter name="command" string="true">view' in prompt
    assert '<｜DSML｜parameter name="path" string="true">/testbed' in prompt
    assert 'parameter name="arguments"' not in prompt
```
**EN:** Checks Deepseek V4 Renders Parsed History Tool Arguments under a focused test scenario. The body exercises logic via `parse_chat_messages`, `_tokenizer().apply_chat_template`, `_model_config` before asserting the expected outcome.
**CN:** 该测试用例验证 Deepseek V4 Renders Parsed History Tool Arguments 在特定场景下的行为。 函数体会先通过 `parse_chat_messages`, `_tokenizer().apply_chat_template`, `_model_config` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
_load_reference_case
_render_reference_case
test_deepseek_v4_accepts_openai_reasoning_effort_values
test_deepseek_v4_none_reasoning_effort_disables_thinking
test_deepseek_v4_maps_compatible_thinking_reasoning_effort_values
test_deepseek_v4_preserves_reference_max_reasoning_effort
test_deepseek_v4_maps_xhigh_to_reference_max_reasoning_effort
test_deepseek_v4_matches_reference_golden_fixtures
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Tool-call parsing / 工具调用解析**
  - **EN:** The scenarios verify extraction of structured tool calls from model outputs and streaming deltas.
  - **CN:** 这些场景验证如何从模型输出和流式增量中提取结构化工具调用。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `pathlib`, `types`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.chat_utils`, `vllm.renderers.registry`, `vllm.tokenizers.deepseek_v4`, `vllm.tokenizers.registry`
