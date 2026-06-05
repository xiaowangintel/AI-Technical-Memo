# deepseekv4_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/deepseekv4_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the deepseekV4 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 deepseekV4 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-11)
```python
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.tool_parsers.deepseekv32_tool_parser import DeepSeekV32ToolParser
from vllm.tool_parsers.structural_tag_registry import (
    get_enable_structured_outputs_in_reasoning,
    get_model_structural_tag,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Class `DeepSeekV4ToolParser` (lines 14-31)
```python
class DeepSeekV4ToolParser(DeepSeekV32ToolParser):
    """
    DeepSeek V4 DSML tool parser.

    V4 keeps the V3.2 DSML invoke/parameter grammar, but wraps tool calls in
    ``<｜DSML｜tool_calls>`` instead of ``<｜DSML｜function_calls>``.
    """
```
**EN:** DeepSeek V4 DSML tool parser.
**CN:** 定义 `DeepSeekV4ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `DeepSeekV4ToolParser.get_structural_tag` (lines 25-31)
```python
    def get_structural_tag(self, request: ChatCompletionRequest):
        return get_model_structural_tag(
            model="deepseek_v4",
            tools=request.tools,
            tool_choice=request.tool_choice,
            reasoning=get_enable_structured_outputs_in_reasoning(),
        )
```
**EN:** Builds or returns derived metadata used by the parser and by higher-level serving code.
**CN:** `DeepSeekV4ToolParser.get_structural_tag` 方法构建或返回解析器与上层服务需要的派生元数据。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.tool_parsers.deepseekv32_tool_parser`, `vllm.tool_parsers.structural_tag_registry`
