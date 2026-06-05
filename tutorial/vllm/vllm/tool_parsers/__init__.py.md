# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/__init__.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Registers tool parser implementations through a lazy-loading map keyed by model format names. / 通过以模型格式名为键的延迟加载映射注册工具解析器实现。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-7)
```python
from vllm.tool_parsers.abstract_tool_parser import (
    ToolParser,
    ToolParserManager,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 9-9)
```python
__all__ = ["ToolParser", "ToolParserManager"]
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Module constants/configuration 2 (lines 24-189)
```python
_TOOL_PARSERS_TO_REGISTER = {
    "deepseek_v3": (  # name
        "deepseekv3_tool_parser",  # filename
        "DeepSeekV3ToolParser",  # class_name
    ),
    "deepseek_v31": (
        "deepseekv31_tool_parser",
        "DeepSeekV31ToolParser",
    ),
    "deepseek_v32": (
        "deepseekv32_tool_parser",
        "DeepSeekV32ToolParser",
    ),
    "deepseek_v4": (
        "deepseekv4_tool_parser",
        "DeepSeekV4ToolParser",
    ),
    "cohere_command3": (
        "cohere_command_tool_parser",
        "CohereCommand3ToolParser",
    ),
    "cohere_command4": (
        "cohere_command_tool_parser",
        "CohereCommand4ToolParser",
    ),
    "ernie45": (
        "ernie45_tool_parser",
        "Ernie45ToolParser",
    ),
    "glm45": (
# ... omitted for brevity / 为简洁省略 ...
        "gigachat3_tool_parser",
        "GigaChat3ToolParser",
    ),
    "functiongemma": (
        "functiongemma_tool_parser",
        "FunctionGemmaToolParser",
    ),
    "gemma4": (
        "gemma4_tool_parser",
        "Gemma4ToolParser",
    ),
}
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `register_lazy_tool_parsers` (lines 192-195)
```python
def register_lazy_tool_parsers():
    for name, (file_name, class_name) in _TOOL_PARSERS_TO_REGISTER.items():
        module_path = f"vllm.tool_parsers.{file_name}"
        ToolParserManager.register_lazy_module(name, module_path, class_name)
```
**EN:** Registers an implementation in a shared registry so the serving stack can look it up by model or format name.
**CN:** `register_lazy_tool_parsers` 函数把实现注册到共享注册表中，便于服务层按模型或格式名称查找。

### Module bootstrap (lines 198-198)
```python
register_lazy_tool_parsers()
```
**EN:** Executes top-level bootstrap logic so the file registers or activates its functionality when imported.
**CN:** 执行顶层启动逻辑，使该文件在被导入时完成注册或激活自身功能。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.tool_parsers.abstract_tool_parser`
