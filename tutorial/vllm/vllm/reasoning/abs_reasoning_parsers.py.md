# abs_reasoning_parsers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/abs_reasoning_parsers.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines the abstract reasoning parser API and the registry that loads parser implementations eagerly or lazily. / 定义推理解析器的抽象接口，以及用于立即或延迟加载具体解析器实现的注册表。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-14)
```python
import importlib
import os
from abc import abstractmethod
from collections.abc import Callable, Iterable, Sequence
from functools import cached_property
from typing import TYPE_CHECKING, cast

from vllm.entrypoints.mcp.tool_server import ToolServer
from vllm.logger import init_logger
from vllm.utils.collection_utils import is_list_of
from vllm.utils.import_utils import import_from_path
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 23-23)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `ReasoningParser` (lines 26-186)
```python
class ReasoningParser:
    """
    Abstract reasoning parser class that should not be used directly.
    Provided and methods should be used in derived classes.

    It is used to extract reasoning content from the model output.
    """
```
**EN:** Abstract reasoning parser class that should not be used directly.
**CN:** 定义 `ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `ReasoningParser.__init__` (lines 34-38)
```python
    def __init__(self, tokenizer: "TokenizerLike", *args, **kwargs):
        self.model_tokenizer = tokenizer
        # Optional vLLM ModelConfig from the server. Use get (not pop) so composite
        # parsers can forward **kwargs to nested parsers.
        self._model_config: ModelConfig | None = kwargs.get("model_config")
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `ReasoningParser.reasoning_start_str` (lines 47-51)
```python
    def reasoning_start_str(self) -> str | None:
        """Set `reasoning_start_str` to the strings that delimit
        the reasoning block (e.g. `""<seed:think>""` and `"<think>"`).
        """
        return None
```
**EN:** Set `reasoning_start_str` to the strings that delimit the reasoning block (e.g.
**CN:** `ReasoningParser.reasoning_start_str` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `ReasoningParser.is_reasoning_end` (lines 61-75)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        """
        Check if the reasoning content ends in the input_ids.

        It is used in structured engines like `xgrammar` to check if the
        reasoning content ends in the model output.

        Parameters:
        input_ids: list[int]
            The input_ids of the model output.

        Returns:
        bool
            True if the reasoning content ends in the input_ids.
        """
```
**EN:** Check if the reasoning content ends in the input_ids.
**CN:** `ReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `ReasoningParser.is_reasoning_end_streaming` (lines 77-100)
```python
    def is_reasoning_end_streaming(
        self, input_ids: Sequence[int], delta_ids: Iterable[int]
    ) -> bool:
        """
        Check if the reasoning content ends in the input_ids on a
        decode step.

        It is used in structured engines like `xgrammar` to check if the
        reasoning content ends in the model output during a decode step.
        `input_ids` the entire model output and `delta_ids` are the last few
        computed tokens of the model output (like during a decode step).

        Parameters:
        input_ids: list[int]
            The entire model output.
        delta_ids: list[int]
            The last few computed tokens of the model output at the current decode step.

        Returns:
        bool
            True if the reasoning content ends in the `delta_ids` on a
            decode step.
        """
        return self.is_reasoning_end(input_ids)
```
**EN:** Check if the reasoning content ends in the input_ids on a decode step.
**CN:** `ReasoningParser.is_reasoning_end_streaming` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `ReasoningParser.extract_content_ids` (lines 103-112)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        """
        Extract content token ids from the input_ids.
        Parameters:
        input_ids: list[int]
            The input_ids of the model output.
        Returns:
        list[int]
            The extracted content from the input_ids.
        """
```
**EN:** Extract content token ids from the input_ids.
**CN:** `ReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `ReasoningParser.count_reasoning_tokens` (lines 114-131)
```python
    def count_reasoning_tokens(self, token_ids: Sequence[int]) -> int:
        """Count the number of reasoning tokens in a sequence.

        Text-based reasoning models typically wrap their chain-of-thought
        between special start/end tokens (e.g., ``<think> ... </think>``).
        Implementations that support reasoning token counting should override
        this method. The default implementation returns ``0`` so existing
        parsers remain unchanged unless they explicitly opt in.

        Args:
            token_ids: Sequence of generated token ids (excluding prompt).

        Returns:
            int: Number of tokens that belong to reasoning content.
        """

        # By default, assume the parser cannot detect reasoning spans.
        return 0
```
**EN:** Count the number of reasoning tokens in a sequence.
**CN:** `ReasoningParser.count_reasoning_tokens` 方法统计与解析相关的片段或 token 数量，便于上层服务进行计数和报告。

### Method `ReasoningParser.extract_reasoning` (lines 134-151)
```python
    def extract_reasoning(
        self,
        model_output: str,
        request: "ChatCompletionRequest | ResponsesRequest",
    ) -> tuple[str | None, str | None]:
        """
        Extract reasoning content from a complete model-generated string.

        Used for non-streaming responses where we have the entire model response
        available before sending to the client.

        Parameters:
            model_output: The model-generated string to extract reasoning content from.
            request: The request object that was used to generate the model_output.

        Returns:
            A tuple containing the reasoning content and the content.
        """
```
**EN:** Extract reasoning content from a complete model-generated string.
**CN:** `ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `ReasoningParser.extract_reasoning_streaming` (lines 154-169)
```python
    def extract_reasoning_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],
        current_token_ids: Sequence[int],
        delta_token_ids: Sequence[int],
    ) -> "DeltaMessage | None":
        """
        Instance method that should be implemented for extracting reasoning
        from an incomplete response; for use when handling reasoning calls and
        streaming. Has to be an instance method because  it requires state -
        the current tokens/diffs, but also the information about what has
        previously been parsed and extracted (see constructor)
        """
```
**EN:** Instance method that should be implemented for extracting reasoning from an incomplete response; for use when handling reasoning calls and streaming.
**CN:** `ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `ReasoningParser.adjust_request` (lines 171-175)
```python
    def adjust_request(
        self, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> "ChatCompletionRequest | ResponsesRequest":
        """Adjust request parameters; override in subclasses as needed."""
        return request
```
**EN:** Adjust request parameters; override in subclasses as needed.
**CN:** `ReasoningParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `ReasoningParser.prepare_structured_tag` (lines 177-186)
```python
    def prepare_structured_tag(
        self,
        original_tag: str | None,
        tool_server: ToolServer | None,
    ) -> str | None:
        """
        Instance method that is implemented for preparing the structured tag
        Otherwise, None is returned
        """
        return None
```
**EN:** Instance method that is implemented for preparing the structured tag Otherwise, None is returned
**CN:** `ReasoningParser.prepare_structured_tag` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Class `ReasoningParserManager` (lines 189-350)
```python
class ReasoningParserManager:
    """
    Central registry for ReasoningParser implementations.

    Supports two registration modes:
      - Eager registration via `register_module`
      - Lazy registration via `register_lazy_module`

    Each reasoning parser must inherit from `ReasoningParser`.
    """
```
**EN:** Central registry for ReasoningParser implementations.
**CN:** 定义 `ReasoningParserManager` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `ReasoningParserManager.get_reasoning_parser` (lines 204-223)
```python
    def get_reasoning_parser(cls, name: str) -> type[ReasoningParser]:
        """
        Retrieve a registered or lazily registered ReasoningParser class.

        If the parser is lazily registered, it will be imported and cached
        on first access.

        Raises:
            KeyError: if no parser is found under the given name.
        """
        if name in cls.reasoning_parsers:
            return cls.reasoning_parsers[name]

        if name in cls.lazy_parsers:
            return cls._load_lazy_parser(name)

        registered = ", ".join(cls.list_registered())
        raise KeyError(
            f"Reasoning parser '{name}' not found. Available parsers: {registered}"
        )
```
**EN:** Retrieve a registered or lazily registered ReasoningParser class.
**CN:** `ReasoningParserManager.get_reasoning_parser` 方法构建或返回解析器与上层服务需要的派生元数据。

### Method `ReasoningParserManager.list_registered` (lines 226-228)
```python
    def list_registered(cls) -> list[str]:
        """Return names of all eagerly and lazily registered reasoning parsers."""
        return sorted(set(cls.reasoning_parsers.keys()) | set(cls.lazy_parsers.keys()))
```
**EN:** Return names of all eagerly and lazily registered reasoning parsers.
**CN:** `ReasoningParserManager.list_registered` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `ReasoningParserManager._load_lazy_parser` (lines 231-251)
```python
    def _load_lazy_parser(cls, name: str) -> type[ReasoningParser]:
        """Import and register a lazily loaded reasoning parser."""
        module_path, class_name = cls.lazy_parsers[name]
        try:
            mod = importlib.import_module(module_path)
            parser_cls = getattr(mod, class_name)
            if not issubclass(parser_cls, ReasoningParser):
                raise TypeError(
                    f"{class_name} in {module_path} is not a ReasoningParser subclass."
                )

            cls.reasoning_parsers[name] = parser_cls  # cache
            return parser_cls
        except Exception as e:
            logger.exception(
                "Failed to import lazy reasoning parser '%s' from %s: %s",
                name,
                module_path,
                e,
            )
            raise
```
**EN:** Import and register a lazily loaded reasoning parser.
**CN:** `ReasoningParserManager._load_lazy_parser` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `ReasoningParserManager._register_module` (lines 254-279)
```python
    def _register_module(
        cls,
        module: type[ReasoningParser],
        module_name: str | list[str] | None = None,
        force: bool = True,
    ) -> None:
        """Register a ReasoningParser class immediately."""
        if not issubclass(module, ReasoningParser):
            raise TypeError(
                f"module must be subclass of ReasoningParser, but got {type(module)}"
            )

        if module_name is None:
            module_names = [module.__name__]
        elif isinstance(module_name, str):
            module_names = [module_name]
        elif is_list_of(module_name, str):
            module_names = module_name
        else:
            raise TypeError("module_name must be str, list[str], or None.")

        for name in module_names:
            if not force and name in cls.reasoning_parsers:
                existed = cls.reasoning_parsers[name]
                raise KeyError(f"{name} is already registered at {existed.__module__}")
            cls.reasoning_parsers[name] = module
```
**EN:** Register a ReasoningParser class immediately.
**CN:** `ReasoningParserManager._register_module` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `ReasoningParserManager.register_lazy_module` (lines 282-293)
```python
    def register_lazy_module(cls, name: str, module_path: str, class_name: str) -> None:
        """
        Register a lazy module mapping for delayed import.

        Example:
            ReasoningParserManager.register_lazy_module(
                name="qwen3",
                module_path="vllm.reasoning.parsers.qwen3_reasoning_parser",
                class_name="Qwen3ReasoningParser",
            )
        """
        cls.lazy_parsers[name] = (module_path, class_name)
```
**EN:** Register a lazy module mapping for delayed import.
**CN:** `ReasoningParserManager.register_lazy_module` 方法把实现注册到共享注册表中，便于服务层按模型或格式名称查找。

### Method `ReasoningParserManager.register_module` (lines 296-334)
```python
    def register_module(
        cls,
        name: str | list[str] | None = None,
        force: bool = True,
        module: type[ReasoningParser] | None = None,
    ) -> (
        type[ReasoningParser] | Callable[[type[ReasoningParser]], type[ReasoningParser]]
    ):
        """
        Register module with the given name or name list. it can be used as a
        decoder(with module as None) or normal function(with module as not
        None).
        """
        if not isinstance(force, bool):
            raise TypeError(f"force must be a boolean, but got {type(force)}")

        # Immediate registration (explicit call)
        if module is not None:
            cls._register_module(module=module, module_name=name, force=force)
            return module

        # Decorator usage
        def _decorator(obj: type[ReasoningParser]) -> type[ReasoningParser]:
            module_path = obj.__module__
            class_name = obj.__name__

            if isinstance(name, str):
                names = [name]
            elif is_list_of(name, str):
                names = cast(list[str], name)
            else:
                names = [class_name]

            for n in names:
                cls.lazy_parsers[n] = (module_path, class_name)

            return obj

        return _decorator
```
**EN:** Register module with the given name or name list.
**CN:** `ReasoningParserManager.register_module` 方法把实现注册到共享注册表中，便于服务层按模型或格式名称查找。

### Method `ReasoningParserManager.import_reasoning_parser` (lines 337-350)
```python
    def import_reasoning_parser(cls, plugin_path: str) -> None:
        """
        Import a user-defined reasoning parser by the path
        of the reasoning parser define file.
        """
        module_name = os.path.splitext(os.path.basename(plugin_path))[0]

        try:
            import_from_path(module_name, plugin_path)
        except Exception:
            logger.exception(
                "Failed to load module '%s' from %s.", module_name, plugin_path
            )
            return
```
**EN:** Import a user-defined reasoning parser by the path of the reasoning parser define file.
**CN:** `ReasoningParserManager.import_reasoning_parser` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。
- **Lazy parser registration**: Lazy parser registration. / 通过延迟注册降低导入开销，并按名称解析具体解析器。
- **Tokenizer-aware parsing**: Tokenizer-aware parsing. / 依赖 tokenizer 词表和 token ID 来提高边界检测效率。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `importlib`, `os`, `abc`, `collections.abc`, `functools`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.mcp.tool_server`, `vllm.logger`, `vllm.utils.collection_utils`, `vllm.utils.import_utils`
