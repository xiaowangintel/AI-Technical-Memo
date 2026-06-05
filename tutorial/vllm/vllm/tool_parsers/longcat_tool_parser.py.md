# longcat_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/longcat_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the longcat tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 longcat 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-8)
```python
import regex as re

from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import Tool
from vllm.tool_parsers.hermes_tool_parser import Hermes2ProToolParser
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Class `LongcatFlashToolParser` (lines 11-22)
```python
class LongcatFlashToolParser(Hermes2ProToolParser):
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        self.tool_call_start_token: str = "<longcat_tool_call>"
        self.tool_call_end_token: str = "</longcat_tool_call>"

        self.tool_call_regex = re.compile(
            r"<longcat_tool_call>(.*?)</longcat_tool_call>"
            r"|<longcat_tool_call>(.*)",
            re.DOTALL,
        )
```
**EN:** Declares `LongcatFlashToolParser`, the main parser class in this module. It extends Hermes2ProToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `LongcatFlashToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `LongcatFlashToolParser.__init__` (lines 12-22)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        self.tool_call_start_token: str = "<longcat_tool_call>"
        self.tool_call_end_token: str = "</longcat_tool_call>"

        self.tool_call_regex = re.compile(
            r"<longcat_tool_call>(.*?)</longcat_tool_call>"
            r"|<longcat_tool_call>(.*)",
            re.DOTALL,
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `LongcatFlashToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: None / 无
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.hermes_tool_parser`
