# abstract_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/abstract_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines the abstract tool parser contract plus the registry that discovers model-specific tool parsers. / 定义工具解析器的抽象契约，以及用于发现模型专用工具解析器的注册表。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-36)
```python
import importlib
import json
import os
from collections.abc import Callable, Sequence
from functools import cached_property

from openai.types.responses import (
    ResponseFormatTextJSONSchemaConfig,
    ResponseTextConfig,
)
from openai.types.responses.function_tool import FunctionTool

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionNamedToolChoiceParam,
    ChatCompletionRequest,
    ChatCompletionToolsParam,
)
from vllm.entrypoints.openai.engine.protocol import (
    DeltaMessage,
    ExtractedToolCallInformation,
)
from vllm.entrypoints.openai.responses.protocol import (
    ResponsesRequest,
)
from vllm.envs import VLLM_ENFORCE_STRICT_TOOL_CALLING
from vllm.logger import init_logger
from vllm.sampling_params import (
    StructuredOutputsParams,
)
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.utils import Tool, get_json_schema_from_tools
from vllm.utils.collection_utils import is_list_of
from vllm.utils.import_utils import import_from_path
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 38-40)
```python
__all__ = ["Tool"]

logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `ToolParser` (lines 43-189)
```python
class ToolParser:
    """
    Abstract ToolParser class that should not be used directly. Provided
    properties and methods should be used in
    derived classes.
    """
```
**EN:** Abstract ToolParser class that should not be used directly.
**CN:** 定义 `ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `ToolParser.__init__` (lines 61-80)
```python
    def __init__(
        self,
        tokenizer: TokenizerLike,
        tools: list[Tool] | None = None,
    ):
        self.prev_tool_call_arr: list[dict] = []
        # the index of the tool call that is currently being parsed
        self.current_tool_id: int = -1
        self.current_tool_name_sent: bool = False
        self.streamed_args_for_tool: list[str] = []

        self.model_tokenizer = tokenizer
        if tools:
            self.tools: list[ChatCompletionToolsParam | FunctionTool] = [
                tool
                for tool in tools
                if isinstance(tool, (ChatCompletionToolsParam, FunctionTool))
            ]
        else:
            self.tools = []
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `ToolParser.vocab` (lines 83-86)
```python
    def vocab(self) -> dict[str, int]:
        # NOTE: Only PreTrainedTokenizerFast is guaranteed to have .vocab
        # whereas all tokenizers have .get_vocab()
        return self.model_tokenizer.get_vocab()
```
**EN:** Provides the `vocab` helper used by this module to keep parsing logic modular and reusable.
**CN:** `ToolParser.vocab` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `ToolParser.adjust_request` (lines 88-151)
```python
    def adjust_request(
        self,
        request: ChatCompletionRequest | ResponsesRequest,
    ) -> ChatCompletionRequest | ResponsesRequest:
        # If there are no tools, return the request as is.
        if not request.tools:
            return request

        # Step 1 (highest priority for ChatCompletionRequest): apply
        # vLLM-owned structural tag support for model-specific tool formats.
        if (
            isinstance(request, ChatCompletionRequest)
            and VLLM_ENFORCE_STRICT_TOOL_CALLING
        ):
            need_tool_calling = (
                request.tool_choice == "auto"
                or request.tool_choice == "required"
                or isinstance(request.tool_choice, ChatCompletionNamedToolChoiceParam)
            )
            if need_tool_calling:
                structure_tag = self.get_structural_tag(request)
                if structure_tag is not None:
                    if request.structured_outputs is None:
                        request.structured_outputs = StructuredOutputsParams(
                            structural_tag=json.dumps(structure_tag.model_dump()),
                        )
                    else:
                        request.structured_outputs.structural_tag = json.dumps(
                            structure_tag.model_dump()
                        )
                    return request

        # Step 2: set structured output params when tool constraints are
        # derived from the tool schema.
        json_schema_from_tool = get_json_schema_from_tools(
            tool_choice=request.tool_choice, tools=request.tools
        )
        # Set structured output params for tool calling
        if json_schema_from_tool is not None:
            if isinstance(request, ChatCompletionRequest):
                # tool_choice: "Forced Function" or "required" will override
                # structured output json settings to make tool calling work correctly
                request.structured_outputs = StructuredOutputsParams(
                    json=json_schema_from_tool  # type: ignore[call-arg]
                )
                request.response_format = None
            if isinstance(request, ResponsesRequest):
                # Single-shot construction so Pydantic v2 tracks `format`
                # in __fields_set__ — assigning to `.format` after the bare
                # `ResponseTextConfig()` constructor does not, which can
                # drop the nested config from `model_dump`. Also drop the
                # `description` kwarg: it is not a field on
                # ResponseFormatTextJSONSchemaConfig and was being silently
                # passed through as extra.
                request.text = ResponseTextConfig(
                    format=ResponseFormatTextJSONSchemaConfig(
                        type="json_schema",
                        name="tool_calling_response",
                        schema=json_schema_from_tool,
                        strict=True,
                    )
                )

        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `ToolParser.get_structural_tag` (lines 153-154)
```python
    def get_structural_tag(self, request: ChatCompletionRequest):
        return None
```
**EN:** Builds or returns derived metadata used by the parser and by higher-level serving code.
**CN:** `ToolParser.get_structural_tag` 方法构建或返回解析器与上层服务需要的派生元数据。

### Method `ToolParser.extract_tool_calls` (lines 156-168)
```python
    def extract_tool_calls(
        self, model_output: str, request: ChatCompletionRequest
    ) -> ExtractedToolCallInformation:
        """
        Static method that should be implemented for extracting tool calls from
        a complete model-generated string.
        Used for non-streaming responses where we have the entire model response
        available before sending to the client.
        Static because it's stateless.
        """
        raise NotImplementedError(
            "AbstractToolParser.extract_tool_calls has not been implemented!"
        )
```
**EN:** Static method that should be implemented for extracting tool calls from a complete model-generated string.
**CN:** `ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `ToolParser.extract_tool_calls_streaming` (lines 170-189)
```python
    def extract_tool_calls_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],
        current_token_ids: Sequence[int],
        delta_token_ids: Sequence[int],
        request: ChatCompletionRequest,
    ) -> DeltaMessage | None:
        """
        Instance method that should be implemented for extracting tool calls
        from an incomplete response; for use when handling tool calls and
        streaming. Has to be an instance method because  it requires state -
        the current tokens/diffs, but also the information about what has
        previously been parsed and extracted (see constructor)
        """
        raise NotImplementedError(
            "AbstractToolParser.extract_tool_calls_streaming has not been implemented!"
        )
```
**EN:** Instance method that should be implemented for extracting tool calls from an incomplete response; for use when handling tool calls and streaming.
**CN:** `ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Class `ToolParserManager` (lines 192-347)
```python
class ToolParserManager:
    """
    Central registry for ToolParser implementations.

    Supports two modes:
      - Eager (immediate) registration via `register_module`
      - Lazy registration via `register_lazy_module`
    """
```
**EN:** Central registry for ToolParser implementations.
**CN:** 定义 `ToolParserManager` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `ToolParserManager.get_tool_parser` (lines 205-219)
```python
    def get_tool_parser(cls, name: str) -> type[ToolParser]:
        """
        Retrieve a registered or lazily registered ToolParser class.

        If the parser is lazily registered,
        it will be imported and cached on first access.
        Raises KeyError if not found.
        """
        if name in cls.tool_parsers:
            return cls.tool_parsers[name]

        if name in cls.lazy_parsers:
            return cls._load_lazy_parser(name)

        raise KeyError(f"Tool parser '{name}' not found.")
```
**EN:** Retrieve a registered or lazily registered ToolParser class.
**CN:** `ToolParserManager.get_tool_parser` 方法构建或返回解析器与上层服务需要的派生元数据。

### Method `ToolParserManager._load_lazy_parser` (lines 222-241)
```python
    def _load_lazy_parser(cls, name: str) -> type[ToolParser]:
        """Import and register a lazily loaded parser."""
        module_path, class_name = cls.lazy_parsers[name]
        try:
            mod = importlib.import_module(module_path)
            parser_cls = getattr(mod, class_name)
            if not issubclass(parser_cls, ToolParser):
                raise TypeError(
                    f"{class_name} in {module_path} is not a ToolParser subclass."
                )
            cls.tool_parsers[name] = parser_cls  # cache
            return parser_cls
        except Exception as e:
            logger.exception(
                "Failed to import lazy tool parser '%s' from %s: %s",
                name,
                module_path,
                e,
            )
            raise
```
**EN:** Import and register a lazily loaded parser.
**CN:** `ToolParserManager._load_lazy_parser` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `ToolParserManager._register_module` (lines 244-270)
```python
    def _register_module(
        cls,
        module: type[ToolParser],
        module_name: str | list[str] | None = None,
        force: bool = True,
    ) -> None:
        """Register a ToolParser class immediately."""
        if not issubclass(module, ToolParser):
            raise TypeError(
                f"module must be subclass of ToolParser, but got {type(module)}"
            )

        if module_name is None:
            module_name = module.__name__

        if isinstance(module_name, str):
            module_names = [module_name]
        elif is_list_of(module_name, str):
            module_names = module_name
        else:
            raise TypeError("module_name must be str, list[str], or None.")

        for name in module_names:
            if not force and name in cls.tool_parsers:
                existed = cls.tool_parsers[name]
                raise KeyError(f"{name} is already registered at {existed.__module__}")
            cls.tool_parsers[name] = module
```
**EN:** Register a ToolParser class immediately.
**CN:** `ToolParserManager._register_module` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `ToolParserManager.register_lazy_module` (lines 273-284)
```python
    def register_lazy_module(cls, name: str, module_path: str, class_name: str) -> None:
        """
        Register a lazy module mapping.

        Example:
            ToolParserManager.register_lazy_module(
                name="kimi_k2",
                module_path="vllm.tool_parsers.kimi_k2_parser",
                class_name="KimiK2ToolParser",
            )
        """
        cls.lazy_parsers[name] = (module_path, class_name)
```
**EN:** Register a lazy module mapping.
**CN:** `ToolParserManager.register_lazy_module` 方法把实现注册到共享注册表中，便于服务层按模型或格式名称查找。

### Method `ToolParserManager.register_module` (lines 287-330)
```python
    def register_module(
        cls,
        name: str | list[str] | None = None,
        force: bool = True,
        module: type[ToolParser] | None = None,
    ) -> type[ToolParser] | Callable[[type[ToolParser]], type[ToolParser]]:
        """
        Register module immediately or lazily (as a decorator).

        Usage:
            @ToolParserManager.register_module("kimi_k2")
            class KimiK2ToolParser(ToolParser):
                ...

        Or:
            ToolParserManager.register_module(module=SomeToolParser)
        """
        if not isinstance(force, bool):
            raise TypeError(f"force must be a boolean, but got {type(force)}")

        # Immediate registration
        if module is not None:
            cls._register_module(module=module, module_name=name, force=force)
            return module

        # Decorator usage
        def _decorator(obj: type[ToolParser]) -> type[ToolParser]:
            module_path = obj.__module__
            class_name = obj.__name__

            if isinstance(name, str):
                names = [name]
            elif name is not None and is_list_of(name, str):
                names = name
            else:
                names = [class_name]

            for n in names:
                # Lazy mapping only: do not import now
                cls.lazy_parsers[n] = (module_path, class_name)

            return obj

        return _decorator
```
**EN:** Register module immediately or lazily (as a decorator).
**CN:** `ToolParserManager.register_module` 方法把实现注册到共享注册表中，便于服务层按模型或格式名称查找。

### Method `ToolParserManager.list_registered` (lines 333-335)
```python
    def list_registered(cls) -> list[str]:
        """Return names of all eagerly and lazily registered tool parsers."""
        return sorted(set(cls.tool_parsers.keys()) | set(cls.lazy_parsers.keys()))
```
**EN:** Return names of all eagerly and lazily registered tool parsers.
**CN:** `ToolParserManager.list_registered` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `ToolParserManager.import_tool_parser` (lines 338-347)
```python
    def import_tool_parser(cls, plugin_path: str) -> None:
        """Import a user-defined parser file from arbitrary path."""

        module_name = os.path.splitext(os.path.basename(plugin_path))[0]
        try:
            import_from_path(module_name, plugin_path)
        except Exception:
            logger.exception(
                "Failed to load module '%s' from %s.", module_name, plugin_path
            )
```
**EN:** Import a user-defined parser file from arbitrary path.
**CN:** `ToolParserManager.import_tool_parser` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Structured decoding constraints**: Structured decoding constraints. / 用结构标签约束模型输出，减少无效工具调用格式。
- **Tokenizer-aware parsing**: Tokenizer-aware parsing. / 依赖 tokenizer 词表和 token ID 来提高边界检测效率。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `importlib`, `json`, `os`, `collections.abc`, `functools`
- **Third-party / 第三方**: `openai.types.responses`, `openai.types.responses.function_tool`
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.envs`, `vllm.logger`, `vllm.sampling_params`, `vllm.tokenizers`, `vllm.tool_parsers.utils`, `vllm.utils.collection_utils`, `vllm.utils.import_utils`
