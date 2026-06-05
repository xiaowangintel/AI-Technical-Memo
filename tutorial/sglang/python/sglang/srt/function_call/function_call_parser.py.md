# function_call_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/function_call_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file coordinates detector selection for different model families and exposes the main parsing APIs. It bridges raw model text, tool schemas, and the normalized tool-call results consumed by the serving layer. / 该文件负责为不同模型族选择检测器并提供主要解析 API，连接原始模型文本、工具模式以及服务层消费的统一工具调用结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42: Module-level constants and helper logic
```python
import logging
from typing import Dict, List, Literal, Optional, Set, Tuple, Type, Union

from sglang.srt.entrypoints.openai.protocol import (
    LegacyStructuralTagResponseFormat,
    StructuralTagResponseFormat,
    StructuresResponseFormat,
    Tool,
    ToolCallConstraint,
    ToolChoice,
)
from sglang.srt.environ import ToolStrictLevel, envs
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import ToolCallItem
from sglang.srt.function_call.deepseekv3_detector import DeepSeekV3Detector
from sglang.srt.function_call.deepseekv4_detector import DeepSeekV4Detector
from sglang.srt.function_call.deepseekv31_detector import DeepSeekV31Detector
from sglang.srt.function_call.deepseekv32_detector import DeepSeekV32Detector
from sglang.srt.function_call.gemma4_detector import Gemma4Detector
from sglang.srt.function_call.gigachat3_detector import GigaChat3Detector
from sglang.srt.function_call.glm4_moe_detector import Glm4MoeDetector
from sglang.srt.function_call.glm47_moe_detector import Glm47MoeDetector
from sglang.srt.function_call.gpt_oss_detector import GptOssDetector
from sglang.srt.function_call.hermes_detector import HermesDetector
from sglang.srt.function_call.hunyuan_detector import HunyuanDetector
from sglang.srt.function_call.internlm_detector import InternlmDetector
from sglang.srt.function_call.kimik2_detector import KimiK2Detector
from sglang.srt.function_call.lfm2_detector import Lfm2Detector
from sglang.srt.function_call.llama32_detector import Llama32Detector
from sglang.srt.function_call.mimo_detector import MiMoDetector
from sglang.srt.function_call.minimax_m2 import MinimaxM2Detector
from sglang.srt.function_call.mistral_detector import MistralDetector
from sglang.srt.function_call.poolside_v1_detector import PoolsideV1Detector
from sglang.srt.function_call.pythonic_detector import PythonicDetector
from sglang.srt.function_call.qwen3_coder_detector import Qwen3CoderDetector
from sglang.srt.function_call.qwen25_detector import Qwen25Detector
from sglang.srt.function_call.step3_detector import Step3Detector
from sglang.srt.function_call.trinity_detector import TrinityDetector
from sglang.srt.function_call.utils import (
    _get_tool_schema_defs,
    get_json_schema_constraint,
)
```
**EN:** This block contains module-level constants, helpers, or documentation for detector selection and tool-call parsing. It prepares shared state that later classes and functions build on. Notable operations include `import`.
**CN:** 这一段包含与检测器选择与工具调用解析相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `import`。

### Lines 43-46: Module-level constants and helper logic
```python

logger = logging.getLogger(__name__)


```
**EN:** This block contains module-level constants, helpers, or documentation for detector selection and tool-call parsing. It prepares shared state that later classes and functions build on. Notable operations include `getLogger`.
**CN:** 这一段包含与检测器选择与工具调用解析相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `getLogger`。

### Lines 47-47: Class `FunctionCallParser` declaration
```python
class FunctionCallParser:
```
**EN:** This block declares the class `FunctionCallParser` and establishes its responsibility inside detector selection and tool-call parsing. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `FunctionCallParser`.
**CN:** 这一段声明了类 `FunctionCallParser`，并说明它在检测器选择与工具调用解析中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `FunctionCallParser`。

### Lines 48-85: Supporting state inside `FunctionCallParser`
```python
    """
    Parser for function/tool calls in model outputs.

    This class handles both streaming and non-streaming parsing of function calls using a detector.
    In streaming scenarios, each time new_text is received, it calls detector.parse_streaming_increment
    and returns the resulting normal_text and calls to the upper layer (or SSE).
    """

    ToolCallParserEnum: Dict[str, Type[BaseFormatDetector]] = {
        "deepseekv3": DeepSeekV3Detector,
        "deepseekv31": DeepSeekV31Detector,
        "deepseekv32": DeepSeekV32Detector,
        "deepseekv4": DeepSeekV4Detector,
        "glm": Glm4MoeDetector,
        "glm45": Glm4MoeDetector,
        "glm47": Glm47MoeDetector,
        "gpt-oss": GptOssDetector,
        "kimi_k2": KimiK2Detector,
        "lfm2": Lfm2Detector,
        "llama3": Llama32Detector,
        "mimo": MiMoDetector,
        "mistral": MistralDetector,
        "poolside_v1": PoolsideV1Detector,
        "pythonic": PythonicDetector,
        "qwen": Qwen25Detector,
        "qwen25": Qwen25Detector,
        "qwen3_coder": Qwen3CoderDetector,
        "step3": Step3Detector,
        "step3p5": Qwen3CoderDetector,
        "minimax-m2": MinimaxM2Detector,
        "trinity": TrinityDetector,
        "interns1": InternlmDetector,
        "hermes": HermesDetector,
        "hunyuan": HunyuanDetector,
        "gigachat3": GigaChat3Detector,
        "gemma4": Gemma4Detector,
    }

```
**EN:** This block adds supporting state or helper logic inside `FunctionCallParser`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `layer`.
**CN:** 这一段为 `FunctionCallParser` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `layer`。

### Lines 86-95: Method `__init__`
```python
    def __init__(self, tools: List[Tool], tool_call_parser: str):
        detector_class = self.ToolCallParserEnum.get(tool_call_parser)
        if detector_class:
            detector = detector_class()
        else:
            raise ValueError(f"Unsupported tool_call_parser: {tool_call_parser}")

        self.detector = detector
        self.tools = tools
        self.tool_strict_level = envs.SGLANG_TOOL_STRICT_LEVEL.get()
```
**EN:** This block defines the method `__init__` on `FunctionCallParser`. It introduces the parameters, setup steps, and the main entry point for this piece of detector selection and tool-call parsing. Definitions introduced here include `__init__`. Notable operations include `get`, `detector_class`, `ValueError`.
**CN:** 这一段定义了method `__init__`（属于 `FunctionCallParser`），介绍了参数、初始化步骤，以及这部分检测器选择与工具调用解析逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `get`、`detector_class`、`ValueError`。

### Lines 97-110: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """
        Check if the given text contains a tool call in the format supported by this parser.
        This delegates to the detector's implementation.

        Args:
            text: The text to check for tool calls

        Returns:
            True if the text contains a tool call, False otherwise
        """
        if not self.tools:
            return False
        return self.detector.has_tool_call(text)
```
**EN:** This block defines the method `has_tool_call` on `FunctionCallParser`. It introduces the parameters, setup steps, and the main entry point for this piece of detector selection and tool-call parsing. Definitions introduced here include `has_tool_call`. Notable operations include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `FunctionCallParser`），介绍了参数、初始化步骤，以及这部分检测器选择与工具调用解析逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。 值得注意的操作包括 `has_tool_call`。

### Lines 112-131: Method `parse_non_stream`
```python
    def parse_non_stream(self, full_text: str) -> Tuple[str, list[ToolCallItem]]:
        """
        One-time parsing of the full text to extract tool calls.

        Args:
            full_text: The complete text to parse

        Returns:
            A tuple containing:
            - The remaining text after parsing that was not consumed by the detector (can be treated as normal text)
            - A list of tool calls parsed from the text
        """
        if not self.tools:
            return full_text, []
        parsed_result = self.detector.detect_and_parse(full_text, self.tools)
        tool_call_list = parsed_result.calls
        if tool_call_list:
            return parsed_result.normal_text, tool_call_list
        else:
            return full_text, []
```
**EN:** This block defines the method `parse_non_stream` on `FunctionCallParser`. It introduces the parameters, setup steps, and the main entry point for this piece of detector selection and tool-call parsing. Definitions introduced here include `parse_non_stream`. Notable operations include `detector`, `detect_and_parse`.
**CN:** 这一段定义了method `parse_non_stream`（属于 `FunctionCallParser`），介绍了参数、初始化步骤，以及这部分检测器选择与工具调用解析逻辑的主要入口。 此处引入的定义包括 `parse_non_stream`。 值得注意的操作包括 `detector`、`detect_and_parse`。

### Lines 133-144: Method `parse_stream_chunk` signature and setup
```python
    def parse_stream_chunk(self, chunk_text: str) -> Tuple[str, list[ToolCallItem]]:
        """
        Streaming incremental parsing of chunks of text as they arrive.

        Args:
            chunk_text: The new chunk of text to parse

        Returns:
            A tuple containing:
            - The normal text that should be displayed to the user
            - A list of tool calls parsed from the chunk
        """
```
**EN:** This block defines the method `parse_stream_chunk` on `FunctionCallParser`. It introduces the parameters, setup steps, and the main entry point for this piece of detector selection and tool-call parsing. Definitions introduced here include `parse_stream_chunk`.
**CN:** 这一段定义了method `parse_stream_chunk`（属于 `FunctionCallParser`），介绍了参数、初始化步骤，以及这部分检测器选择与工具调用解析逻辑的主要入口。 此处引入的定义包括 `parse_stream_chunk`。

### Lines 145-157: Method `parse_stream_chunk` logic (part 1)
```python
        if not self.tools:
            return chunk_text, []
        final_normal_text = ""
        final_calls = []

        sp_result = self.detector.parse_streaming_increment(chunk_text, self.tools)
        if sp_result.normal_text:
            final_normal_text = sp_result.normal_text
        if sp_result.calls:
            final_calls.extend(sp_result.calls)
            final_normal_text = sp_result.normal_text

        return final_normal_text, final_calls
```
**EN:** This block continues `parse_stream_chunk` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding detector selection and tool-call parsing workflow. Notable operations include `parse_streaming_increment`, `extend`.
**CN:** 这一段延续了 `parse_stream_chunk` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的检测器选择与工具调用解析工作流。 值得注意的操作包括 `parse_streaming_increment`、`extend`。

### Lines 159-173: Method `get_legacy_structural_tag` signature and setup
```python
    def get_legacy_structural_tag(
        self, at_least_one: bool = False
    ) -> StructuralTagResponseFormat:
        """
        Generate a structural tag response format for all available tools.

        This creates the necessary structural tags that guide the model's output format.

        Args:
            at_least_one: If True, the grammar forces at least one tool call
                (no free text allowed). Used for required/named tool_choice.

        Raises:
            ValueError: If tools have conflicting $defs schemas.
        """
```
**EN:** This block defines the method `get_legacy_structural_tag` on `FunctionCallParser`. It introduces the parameters, setup steps, and the main entry point for this piece of detector selection and tool-call parsing. Definitions introduced here include `get_legacy_structural_tag`. Notable operations include `call`.
**CN:** 这一段定义了method `get_legacy_structural_tag`（属于 `FunctionCallParser`），介绍了参数、初始化步骤，以及这部分检测器选择与工具调用解析逻辑的主要入口。 此处引入的定义包括 `get_legacy_structural_tag`。 值得注意的操作包括 `call`。

### Lines 174-200: Method `get_legacy_structural_tag` logic (part 1)
```python
        # Validate $defs consistency before building structural tags
        _get_tool_schema_defs(self.tools)

        tool_structures: List[StructuresResponseFormat] = list()
        tool_trigger_set: Set[str] = set()

        get_structure_info = self.detector.structure_info()
        for tool in self.tools:
            function = tool.function
            name = function.name
            assert name is not None
            info = get_structure_info(name)

            # accept all if not strict, otherwise only accept the schema
            is_strict = (
                function.strict or self.tool_strict_level >= ToolStrictLevel.PARAMETER
            )
            schema = function.parameters if is_strict else {}

            tool_structures.append(
                StructuresResponseFormat(
                    begin=info.begin,
                    schema=schema or {},  # type: ignore
                    end=info.end,
                )
            )
            tool_trigger_set.add(info.trigger)
```
**EN:** This block continues `get_legacy_structural_tag` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding detector selection and tool-call parsing workflow. Notable operations include `_get_tool_schema_defs`, `structure_info`, `get_structure_info`, `append`.
**CN:** 这一段延续了 `get_legacy_structural_tag` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的检测器选择与工具调用解析工作流。 值得注意的操作包括 `_get_tool_schema_defs`、`structure_info`、`get_structure_info`、`append`。

### Lines 201-209: Method `get_legacy_structural_tag` logic (part 2)
```python

        # TODO(dark): move this into new structural tag format
        # This requires all grammar backend support the new format
        return LegacyStructuralTagResponseFormat(
            type="structural_tag",
            structures=tool_structures,
            triggers=list(tool_trigger_set),
            at_least_one=at_least_one,
        )
```
**EN:** This block continues `get_legacy_structural_tag` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding detector selection and tool-call parsing workflow. Notable operations include `TODO`, `LegacyStructuralTagResponseFormat`.
**CN:** 这一段延续了 `get_legacy_structural_tag` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的检测器选择与工具调用解析工作流。 值得注意的操作包括 `TODO`、`LegacyStructuralTagResponseFormat`。

### Lines 211-227: Method `get_structure_constraint` signature and setup
```python
    def get_structure_constraint(
        self,
        tool_choice: Union[ToolChoice, Literal["auto", "required"]],
        parallel_tool_calls: bool = True,
        thinking_mode: bool = False,
    ) -> Optional[ToolCallConstraint]:
        """
        Returns the appropriate structure constraint for tool calls based on the tool_choice.
        The constraint is used to guide the model's output format.

        Args:
            tool_choice: The tool choice setting from the request

        Returns:
            A tuple of (constraint_type, constraint_value) to be added to sampling parameters,
            or None if no constraint applies.
        """
```
**EN:** This block defines the method `get_structure_constraint` on `FunctionCallParser`. It introduces the parameters, setup steps, and the main entry point for this piece of detector selection and tool-call parsing. Definitions introduced here include `get_structure_constraint`. Notable operations include `of`.
**CN:** 这一段定义了method `get_structure_constraint`（属于 `FunctionCallParser`），介绍了参数、初始化步骤，以及这部分检测器选择与工具调用解析逻辑的主要入口。 此处引入的定义包括 `get_structure_constraint`。 值得注意的操作包括 `of`。

### Lines 228-252: Method `get_structure_constraint` logic (part 1)
```python
        is_required = tool_choice == "required" or isinstance(tool_choice, ToolChoice)
        should_constrain_auto = tool_choice == "auto" and (
            any(tool.function.strict for tool in self.tools)
            or self.tool_strict_level >= ToolStrictLevel.FUNCTION
        )

        # Highest priority: model-native structural_tag when available.
        try:
            if is_required or should_constrain_auto:
                structural_tag = self.detector.get_structural_tag(
                    tools=self.tools,
                    thinking_mode=thinking_mode,
                    tool_choice=tool_choice,
                )
                if structural_tag is not None:
                    return ("structural_tag", structural_tag)

                # Fallback to legacy structural tag if model-native tag is not supported.
                if self.detector.supports_structural_tag():
                    # For "required"/named: always use structural_tag to preserve the
                    # model's native tool call format. Schema is only included when
                    # strict=True, per OpenAI protocol semantics.
                    # For "auto": only constrain when strict is enabled.
                    tag = self.get_legacy_structural_tag(at_least_one=is_required)
                    return ("structural_tag", tag)
```
**EN:** This block continues `get_structure_constraint` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding detector selection and tool-call parsing workflow. Notable operations include `and`, `get_structural_tag`, `supports_structural_tag`, `get_legacy_structural_tag`.
**CN:** 这一段延续了 `get_structure_constraint` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的检测器选择与工具调用解析工作流。 值得注意的操作包括 `and`、`get_structural_tag`、`supports_structural_tag`、`get_legacy_structural_tag`。

### Lines 253-261: Method `get_structure_constraint` logic (part 2)
```python

            if tool_choice == "required" or isinstance(tool_choice, ToolChoice):
                json_schema = get_json_schema_constraint(
                    self.tools, tool_choice, parallel_tool_calls=parallel_tool_calls
                )
                return ("json_schema", json_schema)
        except Exception as e:
            logger.error(f"Error getting structure constraint: {e}")
            return None
```
**EN:** This block continues `get_structure_constraint` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding detector selection and tool-call parsing workflow. Notable operations include `get_json_schema_constraint`, `error`.
**CN:** 这一段延续了 `get_structure_constraint` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的检测器选择与工具调用解析工作流。 值得注意的操作包括 `get_json_schema_constraint`、`error`。

## Key Concepts / 关键概念
- `FunctionCallParser`: Class that encapsulates function call parser behavior in this module. / `FunctionCallParser`：封装与“函数调用解析器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`, `sglang.srt.function_call.deepseekv3_detector`, `sglang.srt.function_call.deepseekv4_detector`, `sglang.srt.function_call.deepseekv31_detector`, `sglang.srt.function_call.deepseekv32_detector`, `sglang.srt.function_call.gemma4_detector`, `sglang.srt.function_call.gigachat3_detector`, `sglang.srt.function_call.glm4_moe_detector`, `sglang.srt.function_call.glm47_moe_detector`, `sglang.srt.function_call.gpt_oss_detector`, `sglang.srt.function_call.hermes_detector`, `sglang.srt.function_call.hunyuan_detector`
- **More internal imports / 更多内部导入**: 14 additional module paths omitted for brevity. / 其余 14 个内部模块路径因篇幅原因未展开。
