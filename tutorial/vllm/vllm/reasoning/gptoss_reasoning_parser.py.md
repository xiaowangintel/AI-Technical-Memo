# gptoss_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/gptoss_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the gptoss reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 gptoss 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 3-13)
```python
import json
from collections.abc import Iterable, Sequence
from typing import TYPE_CHECKING

from transformers import PreTrainedTokenizerBase

from vllm.entrypoints.mcp.tool_server import ToolServer
from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.entrypoints.openai.parser.harmony_utils import parse_chat_output
from vllm.logger import init_logger
from vllm.reasoning import ReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 19-35)
```python
logger = init_logger(__name__)

no_func_reasoning_tag = {
    "type": "structural_tag",
    "format": {
        "type": "triggered_tags",
        "tags": [
            {
                "begin": "<|channel|>analysis<|message|>",
                "content": {"type": "any_text"},
                "end": "<|end|>",
            }
        ],
        "triggers": ["<|channel|>analysis"],
        "stop_after_first": False,
    },
}
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `from_builtin_tool_to_tag` (lines 38-51)
```python
def from_builtin_tool_to_tag(tool: str) -> list[dict]:
    tag = [
        {
            "begin": f"<|channel|>commentary to={tool}",
            "content": {"type": "any_text"},
            "end": "<|end|>",
        },
        {
            "begin": f"<|channel|>analysis to={tool}",
            "content": {"type": "any_text"},
            "end": "<|end|>",
        },
    ]
    return tag
```
**EN:** Provides the `from_builtin_tool_to_tag` helper used by this module to keep parsing logic modular and reusable.
**CN:** `from_builtin_tool_to_tag` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `tag_with_builtin_funcs` (lines 54-62)
```python
def tag_with_builtin_funcs(no_func_reasoning_tag, builtin_tool_list: list[str]) -> dict:
    import copy

    new_tag = copy.deepcopy(no_func_reasoning_tag)
    new_tag["format"]["triggers"].append("<|channel|>commentary to=")

    for tool in builtin_tool_list:
        new_tag["format"]["tags"].extend(from_builtin_tool_to_tag(tool))
    return new_tag
```
**EN:** Provides the `tag_with_builtin_funcs` helper used by this module to keep parsing logic modular and reusable.
**CN:** `tag_with_builtin_funcs` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Class `GptOssReasoningParser` (lines 65-206)
```python
class GptOssReasoningParser(ReasoningParser):
    """
    Reasoning parser for GptOss model.

    The GptOss model uses harmony to extract reasoning content and this parser
    is only used for detecting the end of the reasoning content.
    """
```
**EN:** Reasoning parser for GptOss model.
**CN:** 定义 `GptOssReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `GptOssReasoningParser.__init__` (lines 73-84)
```python
    def __init__(self, tokenizer: PreTrainedTokenizerBase, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)
        # The model can output some special tokens between "final" and "<|message|>"
        # So we need to look for both sequences to determine the end of reasoning.
        self.reasoning_end_token_ids_prefix = self.model_tokenizer.encode(
            "<|channel|>final"
        )
        self.reasoning_end_token_ids_suffix = self.model_tokenizer.encode("<|message|>")
        # We also need to check for the <|end|> token to avoid false positives from
        # previous messages in multi-turn conversations.
        self.eom_token_id = self.vocab["<|end|>"]
        self.reasoning_max_num_between_tokens = 20
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `GptOssReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `GptOssReasoningParser.is_reasoning_end` (lines 86-113)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        end_token_ids_prefix = self.reasoning_end_token_ids_prefix
        end_token_ids_suffix = self.reasoning_end_token_ids_suffix
        assert len(end_token_ids_prefix) > 0, "reasoning_end_token_ids_prefix is empty"
        assert len(end_token_ids_suffix) > 0, "reasoning_end_token_ids_suffix is empty"
        # Check if the end sequence is present in the input_ids.
        # We search from the end of input_ids to find the last match.
        for i in range(len(input_ids) - len(end_token_ids_prefix), -1, -1):
            if input_ids[i] == self.eom_token_id:
                # We looped backwards far enough to find the end of a previous message,
                # which means we have searched the entirety of the current message
                # and can exit early without searching further back into prior
                # messages of the conversation.
                return False
            if input_ids[i : i + len(end_token_ids_prefix)] == end_token_ids_prefix:
                # We have found the prefix, now we look for the suffix after the prefix.
                suffix_start = i + len(end_token_ids_prefix)
                for j in range(
                    suffix_start, len(input_ids) - len(end_token_ids_suffix) + 1
                ):
                    if j - suffix_start >= self.reasoning_max_num_between_tokens:
                        break
                    if (
                        input_ids[j : j + len(end_token_ids_suffix)]
                        == end_token_ids_suffix
                    ):
                        return True
        return False
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `GptOssReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `GptOssReasoningParser.is_reasoning_end_streaming` (lines 115-132)
```python
    def is_reasoning_end_streaming(
        self, input_ids: Sequence[int], delta_ids: Iterable[int]
    ) -> bool:
        # The pattern window covers the end-of-reasoning marker itself.
        # We add len(delta_ids) so that under speculative decoding (where
        # a single step can accept many tokens) the entire accepted chunk
        # is always inside the scan region.
        delta_ids = tuple(delta_ids)
        pattern_len = (
            len(self.reasoning_end_token_ids_prefix)
            + self.reasoning_max_num_between_tokens
            + len(self.reasoning_end_token_ids_suffix)
        )
        window = pattern_len + len(delta_ids)
        n = len(input_ids)
        if n <= window:
            return self.is_reasoning_end(input_ids)
        return self.is_reasoning_end(input_ids[n - window :])
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `GptOssReasoningParser.is_reasoning_end_streaming` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `GptOssReasoningParser.extract_content_ids` (lines 134-138)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        _, content, _ = parse_chat_output(input_ids)
        if content is None:
            return []
        return self.model_tokenizer.encode(content)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `GptOssReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `GptOssReasoningParser.extract_reasoning_streaming` (lines 140-167)
```python
    def extract_reasoning_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],
        current_token_ids: Sequence[int],
        delta_token_ids: Sequence[int],
    ) -> DeltaMessage | None:
        prev_reasoning, prev_content, _ = parse_chat_output(list(previous_token_ids))
        cur_reasoning, cur_content, _ = parse_chat_output(list(current_token_ids))
        reasoning_delta = None
        content_delta = None
        if cur_reasoning is not None:
            prev_r = prev_reasoning or ""
            if cur_reasoning.startswith(prev_r):
                reasoning_delta = cur_reasoning[len(prev_r) :] or None
            else:
                reasoning_delta = cur_reasoning
        if cur_content is not None:
            prev_c = prev_content or ""
            if cur_content.startswith(prev_c):
                content_delta = cur_content[len(prev_c) :] or None
            else:
                content_delta = cur_content
        if reasoning_delta is None and content_delta is None:
            return None
        return DeltaMessage(reasoning=reasoning_delta, content=content_delta)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `GptOssReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `GptOssReasoningParser.extract_reasoning` (lines 169-176)
```python
    def extract_reasoning(
        self,
        model_output: str,
        request: "ChatCompletionRequest | ResponsesRequest",
    ) -> tuple[str | None, str | None]:
        raise NotImplementedError(
            "gpt-oss has a special branch for parsing reasoning in non-streaming mode. This method shouldn't be used."  # noqa: E501
        )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `GptOssReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `GptOssReasoningParser.prepare_structured_tag` (lines 179-206)
```python
    def prepare_structured_tag(
        self, original_tag: str | None, tool_server: ToolServer | None
    ) -> str | None:
        if original_tag is None:
            if tool_server is None:
                return json.dumps(no_func_reasoning_tag)
            else:
                builtin_tool_list: list[str] = []
                if tool_server.has_tool("browser"):
                    builtin_tool_list.append("browser")
                if tool_server.has_tool("python"):
                    builtin_tool_list.append("python")
                if tool_server.has_tool("container"):
                    builtin_tool_list.append("container")

                if len(builtin_tool_list) > 0:
                    logger.info("Builtin_tool_list: %s", builtin_tool_list)
                    func_tag = json.dumps(
                        tag_with_builtin_funcs(no_func_reasoning_tag, builtin_tool_list)
                    )
                else:
                    logger.info("Builtin_tool_list is empty")
                    func_tag = json.dumps(no_func_reasoning_tag)

                return func_tag
        else:
            # There is potential risk for appending the tag to the original tag
            return original_tag
```
**EN:** Provides the `prepare_structured_tag` helper used by this module to keep parsing logic modular and reusable.
**CN:** `GptOssReasoningParser.prepare_structured_tag` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。
- **Structured decoding constraints**: Structured decoding constraints. / 用结构标签约束模型输出，减少无效工具调用格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`, `typing`
- **Third-party / 第三方**: `transformers`
- **Internal / 内部**: `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.logger`, `vllm.reasoning`
