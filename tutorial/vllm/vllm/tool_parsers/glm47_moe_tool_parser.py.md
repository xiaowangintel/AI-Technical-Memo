# glm47_moe_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/glm47_moe_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the glm47 moe tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 glm47 moe 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Docstring (lines 3-13)
```python
"""
GLM-4.7 Tool Call Parser.

GLM-4.7 uses a slightly different tool call format compared to GLM-4.5:
  - The function name may appear on the same line as ``<tool_call>`` without
    a newline separator before the first ``<arg_key>``.
  - Tool calls may have zero arguments
    (e.g. ``<tool_call>func</tool_call>``).

This parser overrides the parent regex patterns to handle both formats.
"""
```
**EN:** GLM-4.7 Tool Call Parser.
**CN:** 模块文档字符串直接说明了该文件的用途。

### Imports and setup (lines 15-20)
```python
import regex as re

from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import Tool
from vllm.tool_parsers.glm4_moe_tool_parser import Glm4MoeModelToolParser
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 22-22)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Glm47MoeModelToolParser` (lines 25-40)
```python
class Glm47MoeModelToolParser(Glm4MoeModelToolParser):
    supports_required_and_named = False
```
**EN:** Declares `Glm47MoeModelToolParser`, the main parser class in this module. It extends Glm4MoeModelToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `Glm47MoeModelToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Glm47MoeModelToolParser.__init__` (lines 28-40)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)
        # GLM-4.7 format: <tool_call>func_name[<arg_key>...]*</tool_call>
        # The function name can be followed by a newline, whitespace, or
        # directly by <arg_key> tags (no separator).  The arg section is
        # optional so that zero-argument calls are supported.
        self.func_detail_regex = re.compile(
            r"<tool_call>\s*(\S+?)\s*(<arg_key>.*)?</tool_call>", re.DOTALL
        )
        self.func_arg_regex = re.compile(
            r"<arg_key>(.*?)</arg_key>\s*<arg_value>(.*?)</arg_value>",
            re.DOTALL,
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Glm47MoeModelToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: None / 无
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.glm4_moe_tool_parser`
