# poolside_v1_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/poolside_v1_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the poolside v1 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 poolside v1 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Docstring (lines 3-22)
```python
"""
Laguna reasoning parser.

``DeepSeekV3ReasoningParser.is_reasoning_end`` walks the entire
token sequence backwards and returns ``True`` on the first ``</think>`` it
sees. When called on ``prompt_token_ids`` that mistakes any stray
``</think>`` in conversation history, few-shot examples or tool descriptions
for a template-injected "thinking already ended" marker. In the streaming
path (see ``vllm/entrypoints/openai/chat_completion/serving.py``,
``prompt_is_reasoning_end_arr``) that false positive short-circuits the
reasoning parser for the whole response, so any ``<think>...</think>`` the
model emits itself ends up in the content field instead of the reasoning
field.

As we have more flexible templates, we instead scope
the backward search to the current assistant turn: the
walk terminates as soon as we hit the ``<assistant>`` start-of-message
token. A ``</think>`` in a prior user turn or few-shot example is no longer
visible.
"""
```
**EN:** Laguna reasoning parser.
**CN:** 模块文档字符串直接说明了该文件的用途。

### Imports and setup (lines 24-30)
```python
from collections.abc import Sequence

from transformers import PreTrainedTokenizerBase

from vllm.reasoning.deepseek_r1_reasoning_parser import DeepSeekR1ReasoningParser
from vllm.reasoning.deepseek_v3_reasoning_parser import DeepSeekV3ReasoningParser
from vllm.reasoning.identity_reasoning_parser import IdentityReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Class `PoolsideV1ReasoningParser` (lines 33-69)
```python
class PoolsideV1ReasoningParser(DeepSeekV3ReasoningParser):
    """Drop-in replacement for ``deepseek_v3`` that tolerates ``</think>``
    tokens appearing anywhere in the prompt other than the generation prefix.
    """
```
**EN:** Drop-in replacement for ``deepseek_v3`` that tolerates ``</think>`` tokens appearing anywhere in the prompt other than the generation prefix.
**CN:** 定义 `PoolsideV1ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `PoolsideV1ReasoningParser.__init__` (lines 40-49)
```python
    def __init__(self, tokenizer: PreTrainedTokenizerBase, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)

        if self._start_of_assistant_message not in self.vocab:
            raise ValueError(
                f"Tokenizer must contain {self._start_of_assistant_message!r} token"
            )
        self._start_of_assistant_message_token_id = self.vocab[
            self._start_of_assistant_message
        ]
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `PoolsideV1ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `PoolsideV1ReasoningParser.is_reasoning_end` (lines 51-69)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        # IdentityReasoningParser always returns True: no reasoning to parse.
        if isinstance(self._parser, IdentityReasoningParser):
            return True

        assert isinstance(self._parser, DeepSeekR1ReasoningParser)
        for tok_id in reversed(input_ids):
            # <think>: reasoning is not yet ended.
            if tok_id == self._parser.start_token_id:
                return False
            # </think>: reasoning has ended.
            if tok_id == self._parser.end_token_id:
                return True
            # <assistant>: reached the start of the current assistant turn
            # without seeing either marker. Anything further back belongs to
            # the prior conversation and should be ignored.
            if tok_id == self._start_of_assistant_message_token_id:
                return False
        return False
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `PoolsideV1ReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Module constants/configuration 1 (lines 72-72)
```python
__all__ = ["PoolsideV1ReasoningParser"]
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`
- **Third-party / 第三方**: `transformers`
- **Internal / 内部**: `vllm.reasoning.deepseek_r1_reasoning_parser`, `vllm.reasoning.deepseek_v3_reasoning_parser`, `vllm.reasoning.identity_reasoning_parser`
