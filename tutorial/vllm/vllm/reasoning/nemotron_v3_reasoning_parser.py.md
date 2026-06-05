# nemotron_v3_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/nemotron_v3_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the nemotron V3 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 nemotron V3 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 3-9)
```python
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.entrypoints.openai.responses.protocol import (
    ResponsesRequest,
)
from vllm.reasoning.deepseek_r1_reasoning_parser import DeepSeekR1ReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Class `NemotronV3ReasoningParser` (lines 12-33)
```python
class NemotronV3ReasoningParser(DeepSeekR1ReasoningParser):
    """
    Reasoning parser for Nemotron V3 models.
    """
```
**EN:** Reasoning parser for Nemotron V3 models.
**CN:** 定义 `NemotronV3ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `NemotronV3ReasoningParser.extract_reasoning` (lines 17-33)
```python
    def extract_reasoning(
        self, model_output: str, request: ChatCompletionRequest | ResponsesRequest
    ) -> tuple[str | None, str | None]:
        reasoning, final_content = super().extract_reasoning(model_output, request)
        chat_template_kwargs = getattr(request, "chat_template_kwargs", None)

        if (
            chat_template_kwargs
            and (
                chat_template_kwargs.get("enable_thinking") is False
                or chat_template_kwargs.get("force_nonempty_content") is True
            )
            and (final_content is None or not final_content.strip())
        ):
            reasoning, final_content = final_content, reasoning

        return reasoning, final_content
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `NemotronV3ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.reasoning.deepseek_r1_reasoning_parser`
