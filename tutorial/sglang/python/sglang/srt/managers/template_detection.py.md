# template_detection.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/template_detection.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements template detection logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 template detection 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Provide supporting module logic / 提供辅助模块逻辑
```python
# Copyright 2026 SGLang Team
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
Template detection utilities for auto-detecting reasoning and tool-call parsers.

Provides rule-based detection of reasoning mode, reasoning parser, and tool-call
parser from chat templates and tokenizer vocabularies.
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 21-24: Import runtime dependencies / 导入运行时依赖
```python
import logging
import re
from dataclasses import dataclass
from typing import Callable, Optional, Tuple
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 26-29: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


@dataclass(frozen=True)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 30-43: Define class TemplateDetectionContext / 定义类 TemplateDetectionContext
```python
class TemplateDetectionContext:
    template: str
    reasoning_config: Optional["ReasoningToggleConfig"]
    force_reasoning: bool
    vocab: set[str]

    def has_text(self, needle: str) -> bool:
        return needle in self.template

    def has_vocab(self, token: str) -> bool:
        return token in self.vocab

    def has_pattern(self, pattern: str, flags: int = 0) -> bool:
        return re.search(pattern, self.template, flags) is not None
```
**EN:** This block declares the class `TemplateDetectionContext`. It centers on coordinating template detection behavior, with methods such as has_text, has_vocab, has_pattern.
**CN:** 该代码块声明类 `TemplateDetectionContext`。它负责承载与 template detection 相关的核心状态与行为，并通过 has_text, has_vocab, has_pattern 等方法组织实现。

### Lines 44-46: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass(frozen=True)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 47-50: Define class DetectionRule / 定义类 DetectionRule
```python
class DetectionRule:
    name: str
    value: object
    predicate: Callable[[TemplateDetectionContext], bool]
```
**EN:** This block declares the class `DetectionRule`. It centers on coordinating template detection behavior.
**CN:** 该代码块声明类 `DetectionRule`。它负责承载与 template detection 相关的核心状态与行为。

### Lines 51-53: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass(frozen=True)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 54-61: Define class ReasoningToggleConfig / 定义类 ReasoningToggleConfig
```python
class ReasoningToggleConfig:
    toggle_param: Optional[str] = None
    default_enabled: Optional[bool] = None
    special_case: Optional[str] = None

    @property
    def always_on(self) -> bool:
        return self.special_case == "always"
```
**EN:** This block declares the class `ReasoningToggleConfig`. It centers on coordinating template detection behavior, with methods such as always_on.
**CN:** 该代码块声明类 `ReasoningToggleConfig`。它负责承载与 template detection 相关的核心状态与行为，并通过 always_on 等方法组织实现。

### Lines 62-67: Import runtime dependencies / 导入运行时依赖
```python


# ---------------------------------------------------------------------------
# Reasoning mode rules (detect toggle config from template)
# ---------------------------------------------------------------------------
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 68-145: Provide supporting module logic / 提供辅助模块逻辑
```python
REASONING_MODE_RULES = (
    DetectionRule(
        name="gpt_oss_channel_markers",
        value=ReasoningToggleConfig(special_case="always"),
        predicate=lambda ctx: ctx.has_text("<|channel|>"),
    ),
    DetectionRule(
        name="force_reasoning_pattern",
        value=ReasoningToggleConfig(special_case="always"),
        predicate=lambda ctx: ctx.has_pattern(r"<\|im_start\|>assistant\\n<think>\\n")
        and not ctx.has_text("enable_thinking")
        and not ctx.has_text("thinking"),
    ),
    DetectionRule(
        name="mistral_reasoning_effort",
        value=ReasoningToggleConfig(special_case="mistral"),
        predicate=lambda ctx: ctx.has_text("reasoning_effort")
        and ctx.has_text("[THINK]"),
    ),
    DetectionRule(
        name="explicit_enable_thinking_default_false",
        value=ReasoningToggleConfig(
            toggle_param="enable_thinking", default_enabled=False
        ),
        predicate=lambda ctx: ctx.has_pattern(
            r"{%\s*if\s+not\s+enable_thinking\s+is\s+defined\s*%}.*?"
            r"{%\s*set\s+enable_thinking\s*=\s*(?:false|False)\s*%}",
            re.DOTALL,
        ),
    ),
    DetectionRule(
        name="enable_thinking_default_true",
        value=ReasoningToggleConfig(
            toggle_param="enable_thinking", default_enabled=True
        ),
        predicate=lambda ctx: ctx.has_pattern(
            r"{%\s*if\s+not\s+enable_thinking\s+is\s+defined\s*%}.*?"
            r"{%\s*set\s+enable_thinking\s*=\s*(?:true|True)\s*%}",
            re.DOTALL,
        )
        or ctx.has_pattern(
            r"set\s+enable_thinking\s*=\s*enable_thinking\s+if\s+enable_thinking\s+is\s+defined\s+else\s+(?:true|True)"
        )
        or ctx.has_pattern(
            r"enable_thinking\s+is\s+defined\s+and\s+(?:enable_thinking\s+is\s+false|not\s+enable_thinking)"
        )
        or ctx.has_pattern(
            r"enable_thinking\s+is\s+not\s+defined\s+or\s+enable_thinking"
        )
        or ctx.has_pattern(r"namespace\([^)]*enable_thinking\s*=\s*true"),
    ),
    DetectionRule(
        name="explicit_thinking_default_false",
        value=ReasoningToggleConfig(toggle_param="thinking", default_enabled=False),
        predicate=lambda ctx: ctx.has_pattern(
            r"{%\s*if\s+not\s+thinking\s+is\s+defined\s*%}.*?"
            r"{%\s*set\s+thinking\s*=\s*(?:false|False)\s*%}",
            re.DOTALL,
        ),
    ),
    DetectionRule(
        name="thinking_default_true",
        value=ReasoningToggleConfig(toggle_param="thinking", default_enabled=True),
        predicate=lambda ctx: ctx.has_pattern(
            r"{%\s*if\s+not\s+thinking\s+is\s+defined\s*%}.*?"
            r"{%\s*set\s+thinking\s*=\s*(?:true|True)\s*%}",
            re.DOTALL,
        )
        or ctx.has_pattern(
            r"set\s+thinking\s*=\s*thinking\s+if\s+thinking\s+is\s+defined\s+else\s+(?:true|True)"
        )
        or ctx.has_pattern(
            r"thinking\s+is\s+defined\s+and\s+(?:thinking\s+is\s+false|not\s+thinking)"
        )
        or ctx.has_pattern(r"thinking\s+is\s+not\s+defined\s+or\s+thinking")
        or ctx.has_pattern(r"namespace\([^)]*thinking\s*=\s*true"),
    ),
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 146-152: Import runtime dependencies / 导入运行时依赖
```python


# ---------------------------------------------------------------------------
# Shared predicates for model-family detection
# ---------------------------------------------------------------------------
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 153-154: Implement is gemma4 / 实现is gemma4
```python
def _is_gemma4(ctx):
    return ctx.has_text("<|channel>")
```
**EN:** This block implements the function `_is_gemma4(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_gemma4`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_gemma4(ctx)`。它围绕 `_is_gemma4` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 157-158: Implement is kimi / 实现is kimi
```python
def _is_kimi(ctx):
    return ctx.has_text("◁think▷")
```
**EN:** This block implements the function `_is_kimi(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_kimi`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_kimi(ctx)`。它围绕 `_is_kimi` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 161-164: Implement is interns1 / 实现is interns1
```python
def _is_interns1(ctx):
    return ctx.has_text("default_thinking_sys") and ctx.reasoning_config == (
        ReasoningToggleConfig(toggle_param="enable_thinking", default_enabled=True)
    )
```
**EN:** This block implements the function `_is_interns1(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_interns1`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_interns1(ctx)`。它围绕 `_is_interns1` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 167-171: Implement is mistral / 实现is mistral
```python
def _is_mistral(ctx):
    return (
        ctx.reasoning_config is not None
        and ctx.reasoning_config.special_case == "mistral"
    )
```
**EN:** This block implements the function `_is_mistral(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_mistral`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_mistral(ctx)`。它围绕 `_is_mistral` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 174-175: Implement is gpt oss / 实现is gpt oss
```python
def _is_gpt_oss(ctx):
    return ctx.has_text("<|channel|>")
```
**EN:** This block implements the function `_is_gpt_oss(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_gpt_oss`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_gpt_oss(ctx)`。它围绕 `_is_gpt_oss` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 178-179: Implement is kimi k2 / 实现is kimi k2
```python
def _is_kimi_k2(ctx):
    return ctx.has_vocab("<|tool_calls_section_begin|>")
```
**EN:** This block implements the function `_is_kimi_k2(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_kimi_k2`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_kimi_k2(ctx)`。它围绕 `_is_kimi_k2` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 182-185: Implement is nemotron 3 / 实现is nemotron 3
```python
def _is_nemotron_3(ctx):
    return ctx.has_text("truncate_history_thinking") and ctx.reasoning_config == (
        ReasoningToggleConfig(toggle_param="enable_thinking", default_enabled=True)
    )
```
**EN:** This block implements the function `_is_nemotron_3(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_nemotron_3`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_nemotron_3(ctx)`。它围绕 `_is_nemotron_3` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 188-199: Implement is glm45 / 实现is glm45
```python
def _is_glm45(ctx):
    return (
        (
            ctx.has_text("[gMASK]<sop>")
            or ctx.has_pattern(r"(?<!<)/nothink")
            or ctx.has_pattern(r"(?<!<)/think")
        )
        and ctx.has_vocab("<tool_call>")
        and ctx.reasoning_config
        == ReasoningToggleConfig(toggle_param="enable_thinking", default_enabled=True)
        and (ctx.has_vocab("<|user|>") or ctx.has_vocab("<|endoftext|>"))
    )
```
**EN:** This block implements the function `_is_glm45(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_glm45`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_glm45(ctx)`。它围绕 `_is_glm45` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 202-205: Implement is mimo / 实现is mimo
```python
def _is_mimo(ctx):
    return ctx.reasoning_config == ReasoningToggleConfig(
        toggle_param="enable_thinking", default_enabled=False
    )
```
**EN:** This block implements the function `_is_mimo(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_mimo`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_mimo(ctx)`。它围绕 `_is_mimo` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 208-209: Implement is minimax / 实现is minimax
```python
def _is_minimax(ctx):
    return ctx.has_text("<minimax:tool_call>")
```
**EN:** This block implements the function `_is_minimax(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_minimax`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_minimax(ctx)`。它围绕 `_is_minimax` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 212-215: Implement is qwen3 / 实现is qwen3
```python
def _is_qwen3(ctx):
    return ctx.reasoning_config == ReasoningToggleConfig(
        toggle_param="enable_thinking", default_enabled=True
    )
```
**EN:** This block implements the function `_is_qwen3(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_qwen3`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_qwen3(ctx)`。它围绕 `_is_qwen3` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 218-221: Implement is deepseek v3 / 实现is deepseek v3
```python
def _is_deepseek_v3(ctx):
    return ctx.reasoning_config == ReasoningToggleConfig(
        toggle_param="thinking", default_enabled=False
    )
```
**EN:** This block implements the function `_is_deepseek_v3(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_deepseek_v3`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_deepseek_v3(ctx)`。它围绕 `_is_deepseek_v3` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 224-225: Implement is deepseek r1 / 实现is deepseek r1
```python
def _is_deepseek_r1(ctx):
    return ctx.force_reasoning
```
**EN:** This block implements the function `_is_deepseek_r1(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_deepseek_r1`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_deepseek_r1(ctx)`。它围绕 `_is_deepseek_r1` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 228-229: Implement is deepseek r1 think tags / 实现is deepseek r1 think tags
```python
def _is_deepseek_r1_think_tags(ctx):
    return ctx.has_text("<think>") or ctx.has_text("</think>")
```
**EN:** This block implements the function `_is_deepseek_r1_think_tags(ctx)`. It focuses on handling the template detection responsibilities represented by `_is_deepseek_r1_think_tags`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_is_deepseek_r1_think_tags(ctx)`。它围绕 `_is_deepseek_r1_think_tags` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 230-282: Provide supporting module logic / 提供辅助模块逻辑
```python


# ---------------------------------------------------------------------------
# Reasoning parser rules
# ---------------------------------------------------------------------------

REASONING_PARSER_RULES = (
    DetectionRule(name="gemma4", value="gemma4", predicate=_is_gemma4),
    DetectionRule(name="kimi", value="kimi", predicate=_is_kimi),
    DetectionRule(name="interns1", value="interns1", predicate=_is_interns1),
    DetectionRule(name="mistral", value="mistral", predicate=_is_mistral),
    DetectionRule(name="gpt_oss", value="gpt-oss", predicate=_is_gpt_oss),
    DetectionRule(name="kimi_k2", value="kimi_k2", predicate=_is_kimi_k2),
    DetectionRule(name="nemotron_3", value="nemotron_3", predicate=_is_nemotron_3),
    DetectionRule(name="glm45", value="glm45", predicate=_is_glm45),
    DetectionRule(name="mimo", value="mimo", predicate=_is_mimo),
    DetectionRule(name="minimax", value="minimax", predicate=_is_minimax),
    DetectionRule(name="qwen3", value="qwen3", predicate=_is_qwen3),
    DetectionRule(name="deepseek_v3", value="deepseek-v3", predicate=_is_deepseek_v3),
    DetectionRule(
        name="deepseek_r1_force", value="deepseek-r1", predicate=_is_deepseek_r1
    ),
    DetectionRule(
        name="deepseek_r1_think_tags",
        value="deepseek-r1",
        predicate=_is_deepseek_r1_think_tags,
    ),
)

# ---------------------------------------------------------------------------
# Tool-call parser rules (reuse shared predicates, different values)
# ---------------------------------------------------------------------------

TOOL_CALL_PARSER_RULES = (
    DetectionRule(name="gemma4", value="gemma4", predicate=_is_gemma4),
    DetectionRule(name="gpt_oss", value="gpt-oss", predicate=_is_gpt_oss),
    DetectionRule(name="kimi_k2", value="kimi_k2", predicate=_is_kimi_k2),
    DetectionRule(name="minimax", value="minimax-m2", predicate=_is_minimax),
    DetectionRule(name="interns1", value="interns1", predicate=_is_interns1),
    DetectionRule(name="mistral", value="mistral", predicate=_is_mistral),
    DetectionRule(name="glm45", value="glm45", predicate=_is_glm45),
    DetectionRule(name="mimo", value="mimo", predicate=_is_mimo),
    DetectionRule(name="qwen", value="qwen", predicate=_is_qwen3),
    DetectionRule(name="deepseek_v3", value="deepseekv3", predicate=_is_deepseek_v3),
    DetectionRule(name="deepseek_r1", value="deepseekv3", predicate=_is_deepseek_r1),
)


# ---------------------------------------------------------------------------
# Detection functions
# ---------------------------------------------------------------------------
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 283-306: Implement build detection context / 实现build detection context
```python
def build_detection_context(
    template: Optional[str],
    tokenizer,
    reasoning_config: Optional[ReasoningToggleConfig] = None,
    force_reasoning: bool = False,
) -> Optional[TemplateDetectionContext]:
    if template is None:
        return None
    vocab = set()
    if tokenizer is not None:
        try:
            vocab = set(tokenizer.get_vocab().keys())
        except Exception as e:
            logger.warning(
                "Failed to load tokenizer vocab for template detection: %s. "
                "Vocab-dependent detection rules will be skipped.",
                e,
            )
    return TemplateDetectionContext(
        template=template,
        reasoning_config=reasoning_config,
        force_reasoning=force_reasoning,
        vocab=vocab,
    )
```
**EN:** This block implements the function `build_detection_context(template, tokenizer, reasoning_config, force_reasoning)`. It focuses on handling the template detection responsibilities represented by `build_detection_context`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `build_detection_context(template, tokenizer, reasoning_config, force_reasoning)`。它围绕 `build_detection_context` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 309-332: Implement match rules / 实现match rules
```python
def match_rules(
    ctx: TemplateDetectionContext,
    rules: Tuple[DetectionRule, ...],
    label: str,
) -> Optional[str]:
    for rule in rules:
        try:
            if rule.predicate(ctx):
                logger.info(
                    "Detected %s '%s' from template rule '%s'.",
                    label,
                    rule.value,
                    rule.name,
                )
                return rule.value
        except Exception as e:
            logger.warning(
                "Detection rule '%s' for %s raised an exception: %s. Skipping.",
                rule.name,
                label,
                e,
                exc_info=True,
            )
    return None
```
**EN:** This block implements the function `match_rules(ctx, rules, label)`. It focuses on handling the template detection responsibilities represented by `match_rules`, providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `match_rules(ctx, rules, label)`。它围绕 `match_rules` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 335-357: Implement detect reasoning pattern / 实现detect reasoning pattern
```python
def detect_reasoning_pattern(
    template: Optional[str],
) -> Tuple[bool, Optional[ReasoningToggleConfig]]:
    """Detect if the chat template contains reasoning/thinking patterns."""
    if template is None:
        return False, None

    ctx = TemplateDetectionContext(
        template=template,
        reasoning_config=None,
        force_reasoning=False,
        vocab=set(),
    )
    for rule in REASONING_MODE_RULES:
        if rule.predicate(ctx):
            logger.info(
                "Detected reasoning config '%s' from template rule '%s'.",
                rule.value,
                rule.name,
            )
            return rule.value.always_on, rule.value

    return False, None
```
**EN:** This block implements the function `detect_reasoning_pattern(template)`. It focuses on Detect if the chat template contains reasoning/thinking patterns., providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `detect_reasoning_pattern(template)`。它围绕 `detect_reasoning_pattern` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 360-372: Implement detect reasoning parser / 实现detect reasoning parser
```python
def detect_reasoning_parser(
    template: Optional[str],
    tokenizer,
    reasoning_config: Optional[ReasoningToggleConfig] = None,
    force_reasoning: bool = False,
) -> Optional[str]:
    """Auto-detect which reasoning parser to use from the chat template."""
    ctx = build_detection_context(
        template, tokenizer, reasoning_config, force_reasoning
    )
    if ctx is None:
        return None
    return match_rules(ctx, REASONING_PARSER_RULES, "reasoning parser")
```
**EN:** This block implements the function `detect_reasoning_parser(template, tokenizer, reasoning_config, force_reasoning)`. It focuses on Auto-detect which reasoning parser to use from the chat template., providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `detect_reasoning_parser(template, tokenizer, reasoning_config, force_reasoning)`。它围绕 `detect_reasoning_parser` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 375-387: Implement detect tool call parser / 实现detect 工具 call parser
```python
def detect_tool_call_parser(
    template: Optional[str],
    tokenizer,
    reasoning_config: Optional[ReasoningToggleConfig] = None,
    force_reasoning: bool = False,
) -> Optional[str]:
    """Auto-detect which tool-call parser to use from the chat template."""
    ctx = build_detection_context(
        template, tokenizer, reasoning_config, force_reasoning
    )
    if ctx is None:
        return None
    return match_rules(ctx, TOOL_CALL_PARSER_RULES, "tool-call parser")
```
**EN:** This block implements the function `detect_tool_call_parser(template, tokenizer, reasoning_config, force_reasoning)`. It focuses on Auto-detect which tool-call parser to use from the chat template., providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `detect_tool_call_parser(template, tokenizer, reasoning_config, force_reasoning)`。它围绕 `detect_tool_call_parser` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 390-409: Implement resolve auto parser / 实现resolve auto parser
```python
def _resolve_auto_parser(
    server_args,
    attr: str,
    ctx: TemplateDetectionContext,
    rules: Tuple[DetectionRule, ...],
    label: str,
) -> None:
    """Resolve a single auto parser, updating server_args in place."""
    detected = match_rules(ctx, rules, label)
    if detected:
        setattr(server_args, attr, detected)
        logger.info(
            f"Auto-detected --{attr.replace('_', '-')} as '{detected}' from chat template"
        )
    else:
        logger.warning(
            f"--{attr.replace('_', '-')}=auto specified but could not detect "
            f"{label} from chat template. Disabling {label}."
        )
        setattr(server_args, attr, None)
```
**EN:** This block implements the function `_resolve_auto_parser(server_args, attr, ctx, rules, label)`. It focuses on Resolve a single auto parser, updating server_args in place., providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `_resolve_auto_parser(server_args, attr, ctx, rules, label)`。它围绕 `_resolve_auto_parser` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 412-447: Implement resolve auto parsers / 实现resolve auto parsers
```python
def resolve_auto_parsers(server_args) -> None:
    """Resolve --reasoning-parser=auto and --tool-call-parser=auto before scheduler.

    This performs a lightweight tokenizer load to detect parsers from the chat
    template. Called early in engine init before scheduler subprocesses are spawned.
    """
    needs_reasoning = server_args.reasoning_parser == "auto"
    needs_tool_call = server_args.tool_call_parser == "auto"

    if not needs_reasoning and not needs_tool_call:
        return

    from sglang.srt.utils.hf_transformers_utils import get_tokenizer

    try:
        tokenizer = get_tokenizer(
            server_args.model_path,
            trust_remote_code=server_args.trust_remote_code,
        )
        template = getattr(tokenizer, "chat_template", None)
    except Exception as e:
        logger.warning(f"Failed to load tokenizer for auto-detection: {e}")
        if needs_reasoning:
            logger.warning(
                "--reasoning-parser=auto specified but could not detect "
                "reasoning parser from chat template. Disabling reasoning parser."
            )
            server_args.reasoning_parser = None
        if needs_tool_call:
            logger.warning(
                "--tool-call-parser=auto specified but could not detect "
                "tool-call parser from chat template. Disabling tool-call parser."
            )
            server_args.tool_call_parser = None
        return
```
**EN:** This block implements the function `resolve_auto_parsers(server_args)`. It focuses on Resolve --reasoning-parser=auto and --tool-call-parser=auto before scheduler., providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `resolve_auto_parsers(server_args)`。它围绕 `resolve_auto_parsers` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

### Lines 448-471: Continue resolve auto parsers / 继续说明resolve auto parsers
```python
    force_reasoning, reasoning_config = detect_reasoning_pattern(template)
    ctx = build_detection_context(
        template, tokenizer, reasoning_config, force_reasoning
    )
    if ctx is None:
        return

    if needs_reasoning:
        _resolve_auto_parser(
            server_args,
            "reasoning_parser",
            ctx,
            REASONING_PARSER_RULES,
            "reasoning parser",
        )

    if needs_tool_call:
        _resolve_auto_parser(
            server_args,
            "tool_call_parser",
            ctx,
            TOOL_CALL_PARSER_RULES,
            "tool-call parser",
        )
```
**EN:** This block implements the function `resolve_auto_parsers(server_args)`. It focuses on Resolve --reasoning-parser=auto and --tool-call-parser=auto before scheduler., providing reusable behavior for the template detection pipeline.
**CN:** 该代码块实现函数 `resolve_auto_parsers(server_args)`。它围绕 `resolve_auto_parsers` 所承担的 template detection 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Core types / 核心类型**: TemplateDetectionContext, DetectionRule, ReasoningToggleConfig
- **Main callables / 主要可调用对象**: _is_gemma4, _is_kimi, _is_interns1, _is_mistral, _is_gpt_oss, _is_kimi_k2, _is_nemotron_3, _is_glm45, ...
- **Domain focus / 领域焦点**: template detection / template detection
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, re, typing
- **Third-party / 第三方库**: None / 无
- **Local Modules / 本地模块**: sglang.srt.utils.hf_transformers_utils
