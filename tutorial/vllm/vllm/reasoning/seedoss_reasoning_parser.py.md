# seedoss_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/seedoss_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the seedoss reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 seedoss 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 5-5)
```python
from vllm.reasoning.basic_parsers import BaseThinkingReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Class `SeedOSSReasoningParser` (lines 8-27)
```python
class SeedOSSReasoningParser(BaseThinkingReasoningParser):
    """
    Reasoning parser for SeedOSS model.

    The SeedOSS model uses <seed:think>...</seed:think> tokens to
    denote reasoning content text. This parser extracts
    the reasoning content from the model output.
    Similar to DeepSeek R1, it supports cases
    where the model doesn't generate the start token.
    """
```
**EN:** Reasoning parser for SeedOSS model.
**CN:** 定义 `SeedOSSReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `SeedOSSReasoningParser.start_token` (lines 20-22)
```python
    def start_token(self) -> str:
        """The token that starts reasoning content."""
        return "<seed:think>"
```
**EN:** The token that starts reasoning content.
**CN:** `SeedOSSReasoningParser.start_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `SeedOSSReasoningParser.end_token` (lines 25-27)
```python
    def end_token(self) -> str:
        """The token that ends reasoning content."""
        return "</seed:think>"
```
**EN:** The token that ends reasoning content.
**CN:** `SeedOSSReasoningParser.end_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.reasoning.basic_parsers`
