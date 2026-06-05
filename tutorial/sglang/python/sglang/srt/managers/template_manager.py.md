# template_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/template_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements template manager logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 template 管理器 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Provide supporting module logic / 提供辅助模块逻辑
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""
Centralized template management for chat templates and completion templates.

This module provides a unified interface for managing both chat conversation templates
and code completion templates, eliminating global state and improving modularity.
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 21-24: Import runtime dependencies / 导入运行时依赖
```python
import json
import logging
import os
from typing import Dict, Optional
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 26-49: Register helpers or handlers / 注册辅助逻辑或处理器
```python
from sglang.srt.managers.template_detection import (
    REASONING_PARSER_RULES,
    TOOL_CALL_PARSER_RULES,
    ReasoningToggleConfig,
    build_detection_context,
    detect_reasoning_pattern,
    match_rules,
)
from sglang.srt.managers.tokenizer_manager import TokenizerManager
from sglang.srt.parser.code_completion_parser import (
    CompletionTemplate,
    FimPosition,
    completion_template_exists,
    register_completion_template,
    set_completion_template,
)
from sglang.srt.parser.conversation import (
    Conversation,
    SeparatorStyle,
    chat_template_exists,
    get_conv_template_by_model_path,
    register_conv_template,
)
from sglang.srt.parser.jinja_template_utils import detect_jinja_template_content_format
```
**EN:** This block wires callbacks, registries, or handlers into the surrounding runtime.
**CN:** 该代码块把回调、注册表或处理器接入周边运行时流程。

### Lines 51-51: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 54-61: Provide supporting module logic / 提供辅助模块逻辑
```python
class TemplateManager:
    """
    Centralized manager for chat and completion templates.

    This class encapsulates all template-related state and operations,
    eliminating the need for global variables and providing a clean
    interface for template management.
    """
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 63-70: Initialize TemplateManager / 初始化 TemplateManager
```python
    def __init__(self):
        self._chat_template_name: Optional[str] = None
        self._completion_template_name: Optional[str] = None
        self._jinja_template_content_format: Optional[str] = "openai"
        self._force_reasoning: bool = False
        self._reasoning_config: Optional[ReasoningToggleConfig] = None
        self._suggested_reasoning_parser: Optional[str] = None
        self._suggested_tool_call_parser: Optional[str] = None
```
**EN:** This block implements the initializer `__init__()` for `TemplateManager`. It prepares the object state and connects the instance to the surrounding template manager workflow.
**CN:** 该代码块实现 `TemplateManager` 的初始化方法 `__init__()`。它负责准备对象状态，并把实例接入 template 管理器 相关的运行流程。

### Lines 71-72: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 73-75: Implement chat template name / 实现聊天 template name
```python
    def chat_template_name(self) -> Optional[str]:
        """Get the current chat template name."""
        return self._chat_template_name
```
**EN:** This block implements the method `chat_template_name()` on `TemplateManager`. It focuses on Get the current chat template name., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `chat_template_name()`。它围绕 `chat_template_name` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 76-77: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 78-80: Implement completion template name / 实现补全 template name
```python
    def completion_template_name(self) -> Optional[str]:
        """Get the current completion template name."""
        return self._completion_template_name
```
**EN:** This block implements the method `completion_template_name()` on `TemplateManager`. It focuses on Get the current completion template name., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `completion_template_name()`。它围绕 `completion_template_name` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 81-82: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 83-85: Implement jinja template content format / 实现jinja template content format
```python
    def jinja_template_content_format(self) -> Optional[str]:
        """Get the detected template content format ('string' or 'openai' or None)."""
        return self._jinja_template_content_format
```
**EN:** This block implements the method `jinja_template_content_format()` on `TemplateManager`. It focuses on Get the detected template content format ('string' or 'openai' or None)., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `jinja_template_content_format()`。它围绕 `jinja_template_content_format` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 86-87: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 88-95: Implement force reasoning / 实现force reasoning
```python
    def force_reasoning(self) -> bool:
        """
        Check if the current chat template enforces reasoning/thinking.

        Returns:
            True if the template contains reasoning patterns like <think> tags
        """
        return self._force_reasoning
```
**EN:** This block implements the method `force_reasoning()` on `TemplateManager`. It focuses on Check if the current chat template enforces reasoning/thinking., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `force_reasoning()`。它围绕 `force_reasoning` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 96-97: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 98-100: Implement reasoning config / 实现reasoning 配置
```python
    def reasoning_config(self) -> Optional[ReasoningToggleConfig]:
        """Get the reasoning toggle config inferred from chat template."""
        return self._reasoning_config
```
**EN:** This block implements the method `reasoning_config()` on `TemplateManager`. It focuses on Get the reasoning toggle config inferred from chat template., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `reasoning_config()`。它围绕 `reasoning_config` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 101-102: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 103-105: Implement suggested reasoning parser / 实现suggested reasoning parser
```python
    def suggested_reasoning_parser(self) -> Optional[str]:
        """Get the auto-detected reasoning parser name, or None."""
        return self._suggested_reasoning_parser
```
**EN:** This block implements the method `suggested_reasoning_parser()` on `TemplateManager`. It focuses on Get the auto-detected reasoning parser name, or None., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `suggested_reasoning_parser()`。它围绕 `suggested_reasoning_parser` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 106-107: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 108-110: Implement suggested tool call parser / 实现suggested 工具 call parser
```python
    def suggested_tool_call_parser(self) -> Optional[str]:
        """Get the auto-detected tool-call parser name, or None."""
        return self._suggested_tool_call_parser
```
**EN:** This block implements the method `suggested_tool_call_parser()` on `TemplateManager`. It focuses on Get the auto-detected tool-call parser name, or None., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `suggested_tool_call_parser()`。它围绕 `suggested_tool_call_parser` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 112-129: Implement run template detection / 实现run template detection
```python
    def _run_template_detection(self, template, tokenizer) -> None:
        """Run reasoning pattern and parser detection on a template."""
        self._force_reasoning, self._reasoning_config = detect_reasoning_pattern(
            template
        )
        # Build context once, reuse for both parser detections (avoids
        # duplicate tokenizer.get_vocab() calls).
        ctx = build_detection_context(
            template, tokenizer, self._reasoning_config, self._force_reasoning
        )
        if ctx is None:
            return
        self._suggested_reasoning_parser = match_rules(
            ctx, REASONING_PARSER_RULES, "reasoning parser"
        )
        self._suggested_tool_call_parser = match_rules(
            ctx, TOOL_CALL_PARSER_RULES, "tool-call parser"
        )
```
**EN:** This block implements the method `_run_template_detection(template, tokenizer)` on `TemplateManager`. It focuses on Run reasoning pattern and parser detection on a template., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `_run_template_detection(template, tokenizer)`。它围绕 `_run_template_detection` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 131-166: Implement load chat template / 实现load 聊天 template
```python
    def load_chat_template(
        self,
        tokenizer_manager: TokenizerManager,
        chat_template_arg: Optional[str],
        model_path: str,
    ) -> None:
        """
        Load a chat template from various sources.

        Args:
            tokenizer_manager: The tokenizer manager instance
            chat_template_arg: Template name, file path, or None to auto-detect
            model_path: Path to the model
        """
        if chat_template_arg:
            self._load_explicit_chat_template(tokenizer_manager, chat_template_arg)
        else:
            # Guess chat template from model path
            self.guess_chat_template_from_model_path(model_path)

            # If no pre-defined template was found, fallback to HuggingFace template
            if self._chat_template_name is None:
                # Try HuggingFace template first
                hf_template = self._resolve_hf_chat_template(tokenizer_manager)
                if hf_template:
                    # override the chat template
                    if tokenizer_manager.tokenizer:
                        tokenizer_manager.tokenizer.chat_template = hf_template
                    self._jinja_template_content_format = (
                        detect_jinja_template_content_format(hf_template)
                    )
                    logger.info(
                        f"Using default HuggingFace chat template with detected content format: {self._jinja_template_content_format}"
                    )
                else:
                    # Default to string content format if no template was found
```
**EN:** This block implements the method `load_chat_template(tokenizer_manager, chat_template_arg, model_path)` on `TemplateManager`. It focuses on Load a chat template from various sources., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `load_chat_template(tokenizer_manager, chat_template_arg, model_path)`。它围绕 `load_chat_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 167-183: Continue load chat template / 继续说明load 聊天 template
```python
                    self._jinja_template_content_format = "string"
                    logger.info(
                        "No chat template found, defaulting to 'string' content format"
                    )

        # Detect reasoning pattern and suggest parser from chat template
        if tokenizer_manager.tokenizer:
            template = tokenizer_manager.tokenizer.chat_template
            self._run_template_detection(template, tokenizer_manager.tokenizer)
            if self._suggested_reasoning_parser:
                logger.info(
                    f"Auto-detected reasoning parser: {self._suggested_reasoning_parser}"
                )
            if self._suggested_tool_call_parser:
                logger.info(
                    f"Auto-detected tool-call parser: {self._suggested_tool_call_parser}"
                )
```
**EN:** This block implements the method `load_chat_template(tokenizer_manager, chat_template_arg, model_path)` on `TemplateManager`. It focuses on Load a chat template from various sources., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `load_chat_template(tokenizer_manager, chat_template_arg, model_path)`。它围绕 `load_chat_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 185-204: Implement load explicit chat template / 实现load explicit 聊天 template
```python
    def _load_explicit_chat_template(
        self, tokenizer_manager: TokenizerManager, chat_template_arg: str
    ) -> None:
        """Load explicitly specified chat template."""
        logger.info(f"Loading chat template from argument: {chat_template_arg}")

        if chat_template_exists(chat_template_arg):
            self._chat_template_name = chat_template_arg
            return

        if not os.path.exists(chat_template_arg):
            raise RuntimeError(
                f"Chat template {chat_template_arg} is not a built-in template name "
                "or a valid chat template file path."
            )

        if chat_template_arg.endswith(".jinja"):
            self._load_jinja_template(tokenizer_manager, chat_template_arg)
        else:
            self._load_json_chat_template(chat_template_arg)
```
**EN:** This block implements the method `_load_explicit_chat_template(tokenizer_manager, chat_template_arg)` on `TemplateManager`. It focuses on Load explicitly specified chat template., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `_load_explicit_chat_template(tokenizer_manager, chat_template_arg)`。它围绕 `_load_explicit_chat_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 206-216: Implement guess chat template from model path / 实现guess 聊天 template from 模型 path
```python
    def guess_chat_template_from_model_path(self, model_path: str) -> None:
        """
        Infer chat template name from model path.

        Args:
            model_path: Path to the model
        """
        template_name = get_conv_template_by_model_path(model_path)
        if template_name is not None:
            logger.info(f"Inferred chat template from model path: {template_name}")
            self._chat_template_name = template_name
```
**EN:** This block implements the method `guess_chat_template_from_model_path(model_path)` on `TemplateManager`. It focuses on Infer chat template name from model path., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `guess_chat_template_from_model_path(model_path)`。它围绕 `guess_chat_template_from_model_path` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 218-238: Implement load completion template / 实现load 补全 template
```python
    def load_completion_template(self, completion_template_arg: str) -> None:
        """
        Load completion template for code completion.

        Args:
            completion_template_arg: Template name or file path
        """
        logger.info(f"Loading completion template: {completion_template_arg}")

        if not completion_template_exists(completion_template_arg):
            if not os.path.exists(completion_template_arg):
                raise RuntimeError(
                    f"Completion template {completion_template_arg} is not a built-in template name "
                    "or a valid completion template file path."
                )

            self._load_json_completion_template(completion_template_arg)
        else:
            self._completion_template_name = completion_template_arg

        set_completion_template(self._completion_template_name)
```
**EN:** This block implements the method `load_completion_template(completion_template_arg)` on `TemplateManager`. It focuses on Load completion template for code completion., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `load_completion_template(completion_template_arg)`。它围绕 `load_completion_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 240-261: Implement initialize templates / 实现initialize templates
```python
    def initialize_templates(
        self,
        tokenizer_manager: TokenizerManager,
        model_path: str,
        chat_template: Optional[str] = None,
        completion_template: Optional[str] = None,
    ) -> None:
        """
        Initialize all templates based on provided configuration.

        Args:
            tokenizer_manager: The tokenizer manager instance
            model_path: Path to the model
            chat_template: Optional chat template name/path
            completion_template: Optional completion template name/path
        """
        # Load chat template
        self.load_chat_template(tokenizer_manager, chat_template, model_path)

        # Load completion template
        if completion_template:
            self.load_completion_template(completion_template)
```
**EN:** This block implements the method `initialize_templates(tokenizer_manager, model_path, chat_template, completion_template)` on `TemplateManager`. It focuses on Initialize all templates based on provided configuration., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `initialize_templates(tokenizer_manager, model_path, chat_template, completion_template)`。它围绕 `initialize_templates` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 263-277: Implement load jinja template / 实现load jinja template
```python
    def _load_jinja_template(
        self, tokenizer_manager: TokenizerManager, template_path: str
    ) -> None:
        """Load a Jinja template file."""
        with open(template_path, "r") as f:
            chat_template = "".join(f.readlines()).strip("\n")
        tokenizer_manager.tokenizer.chat_template = chat_template.replace("\\n", "\n")
        self._chat_template_name = None
        # Detect content format from the loaded template
        self._jinja_template_content_format = detect_jinja_template_content_format(
            chat_template
        )
        logger.info(
            f"Detected user specified Jinja chat template with content format: {self._jinja_template_content_format}"
        )
```
**EN:** This block implements the method `_load_jinja_template(tokenizer_manager, template_path)` on `TemplateManager`. It focuses on Load a Jinja template file., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `_load_jinja_template(tokenizer_manager, template_path)`。它围绕 `_load_jinja_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 279-306: Implement load json chat template / 实现load JSON 聊天 template
```python
    def _load_json_chat_template(self, template_path: str) -> None:
        """Load a JSON chat template file."""
        assert template_path.endswith(
            ".json"
        ), "unrecognized format of chat template file"

        with open(template_path, "r") as filep:
            template = json.load(filep)
            try:
                sep_style = SeparatorStyle[template["sep_style"]]
            except KeyError:
                raise ValueError(
                    f"Unknown separator style: {template['sep_style']}"
                ) from None

            register_conv_template(
                Conversation(
                    name=template["name"],
                    system_template=template["system"] + "\n{system_message}",
                    system_message=template.get("system_message", ""),
                    roles=(template["user"], template["assistant"]),
                    sep_style=sep_style,
                    sep=template.get("sep", "\n"),
                    stop_str=template["stop_str"],
                ),
                override=True,
            )
        self._chat_template_name = template["name"]
```
**EN:** This block implements the method `_load_json_chat_template(template_path)` on `TemplateManager`. It focuses on Load a JSON chat template file., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `_load_json_chat_template(template_path)`。它围绕 `_load_json_chat_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 308-333: Implement load json completion template / 实现load JSON 补全 template
```python
    def _load_json_completion_template(self, template_path: str) -> None:
        """Load a JSON completion template file."""
        assert template_path.endswith(
            ".json"
        ), "unrecognized format of completion template file"

        with open(template_path, "r") as filep:
            template = json.load(filep)
            try:
                fim_position = FimPosition[template["fim_position"]]
            except KeyError:
                raise ValueError(
                    f"Unknown fim position: {template['fim_position']}"
                ) from None

            register_completion_template(
                CompletionTemplate(
                    name=template["name"],
                    fim_begin_token=template["fim_begin_token"],
                    fim_middle_token=template["fim_middle_token"],
                    fim_end_token=template["fim_end_token"],
                    fim_position=fim_position,
                ),
                override=True,
            )
        self._completion_template_name = template["name"]
```
**EN:** This block implements the method `_load_json_completion_template(template_path)` on `TemplateManager`. It focuses on Load a JSON completion template file., so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `_load_json_completion_template(template_path)`。它围绕 `_load_json_completion_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 335-363: Implement resolve hf chat template / 实现resolve hf 聊天 template
```python
    def _resolve_hf_chat_template(
        self, tokenizer_manager: TokenizerManager
    ) -> Optional[str]:
        try:
            # Try (mm-)processor first, then tokenizer
            template = (
                getattr(tokenizer_manager.processor, "chat_template", None)
                if tokenizer_manager.processor
                else None
            ) or (
                getattr(tokenizer_manager.tokenizer, "chat_template", None)
                if tokenizer_manager.tokenizer
                else None
            )

            if template is None:
                logger.warning("No HuggingFace chat template found")
                return None

            # Handle dict templates (multiple named templates)
            if isinstance(template, dict):
                return self._select_named_template(template, tokenizer_manager)

            # Single string template
            return template

        except Exception as e:
            logger.warning(f"Error getting chat template: {e}")
            return None
```
**EN:** This block implements the method `_resolve_hf_chat_template(tokenizer_manager)` on `TemplateManager`. It focuses on handling the template manager responsibilities represented by `_resolve_hf_chat_template`, so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `_resolve_hf_chat_template(tokenizer_manager)`。它围绕 `_resolve_hf_chat_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 365-387: Implement select named template / 实现select named template
```python
    def _select_named_template(
        self, templates: Dict[str, str], tokenizer_manager: TokenizerManager
    ) -> str:
        if not templates:
            raise ValueError("Empty templates dict provided")

        available_names = list(templates.keys())
        logger.info(f"Multiple HuggingFace chat templates available: {available_names}")

        # Use specified template if provided
        if preferred_name := tokenizer_manager.server_args.hf_chat_template_name:
            if preferred_name not in templates:
                raise ValueError(
                    f"Specified template '{preferred_name}' not found. "
                    f"Available templates: {available_names}"
                )
            logger.info(f"Using specified chat template: '{preferred_name}'")
            return templates[preferred_name]

        # Fallback: Use first available template
        first_name = available_names[0]
        logger.info(f"Using first available template: '{first_name}'")
        return templates[first_name]
```
**EN:** This block implements the method `_select_named_template(templates, tokenizer_manager)` on `TemplateManager`. It focuses on handling the template manager responsibilities represented by `_select_named_template`, so the class can advance the template manager workflow in a self-contained way.
**CN:** 该代码块实现 `TemplateManager` 上的方法 `_select_named_template(templates, tokenizer_manager)`。它围绕 `_select_named_template` 所承担的 template 管理器 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: TemplateManager
- **Domain focus / 领域焦点**: template manager / template 管理器
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: json, logging, os, typing
- **Third-party / 第三方库**: None / 无
- **Local Modules / 本地模块**: sglang.srt.managers.template_detection, sglang.srt.managers.tokenizer_manager, sglang.srt.parser.code_completion_parser, sglang.srt.parser.conversation, sglang.srt.parser.jinja_template_utils
