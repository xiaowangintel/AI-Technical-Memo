# backend_guidance.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/structured_output/backend_guidance.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_walk_json_for_additional_properties`, `has_guidance_unsupported_json_features`, `process_for_additional_properties` for the V1 `structured_output` subsystem. / 为 V1 的 `structured_output` 子系统实现 `_walk_json_for_additional_properties`, `has_guidance_unsupported_json_features`, `process_for_additional_properties`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import copy
import json
import os
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any

import torch

from vllm.logger import init_logger
from vllm.sampling_params import SamplingParams
from vllm.utils.import_utils import LazyLoader
from vllm.utils.mistral import is_mistral_tokenizer
from vllm.v1.structured_output.backend_types import (
    StructuredOutputBackend,
    StructuredOutputGrammar,
    StructuredOutputOptions,
)
from vllm.v1.structured_output.request import get_structured_output_key

if TYPE_CHECKING:
    import llguidance
    import llguidance.hf as llguidance_hf
    import llguidance.torch as llguidance_torch
else:
    llguidance = LazyLoader("llguidance", globals(), "llguidance")
    llguidance_hf = LazyLoader("llguidance.hf", globals(), "llguidance.hf")
    llguidance_torch = LazyLoader("llguidance.torch", globals(), "llguidance.torch")

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_walk_json_for_additional_properties` function / `_walk_json_for_additional_properties` 函数
```python
def _walk_json_for_additional_properties(data: object):
    if isinstance(data, dict):
        for value in data.values():
            _walk_json_for_additional_properties(value)
        if "additionalProperties" not in data and (
            "properties" in data or "patternProperties" in data
        ):
            data["additionalProperties"] = False
    elif isinstance(data, list):
        for item in data:
            _walk_json_for_additional_properties(item)
```
**EN:** This function implements `_walk_json_for_additional_properties` within the module. Key calls include `isinstance`, `values`, `_walk_json_for_additional_properties`. The control flow contains 3 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_walk_json_for_additional_properties`，其作用域位于the module。 关键调用包括 `isinstance`, `values`, `_walk_json_for_additional_properties`。 控制流包含 3 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `has_guidance_unsupported_json_features` function / `has_guidance_unsupported_json_features` 函数
```python
def has_guidance_unsupported_json_features(schema: dict[str, Any]) -> bool:
    """Check if JSON schema contains features unsupported by guidance/llguidance."""

    def check_object(obj: dict[str, Any]) -> bool:
        if not isinstance(obj, dict):
            return False

        # patternProperties is not supported by llguidance
        if "patternProperties" in obj:
            return True

        # Recursively check all nested objects and arrays
        for value in obj.values():
            if isinstance(value, dict):
                if check_object(value):
                    return True
            elif isinstance(value, list):
                for item in value:
                    if isinstance(item, dict) and check_object(item):
                        return True

        return False

    return check_object(schema)
```
**EN:** This function implements `has_guidance_unsupported_json_features` within the module. The docstring frames it as: Check if JSON schema contains features unsupported by guidance/llguidance. Key calls include `check_object`, `values`, `isinstance`. The control flow contains 6 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `has_guidance_unsupported_json_features`，其作用域位于the module。 关键调用包括 `check_object`, `values`, `isinstance`。 控制流包含 6 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `process_for_additional_properties` function / `process_for_additional_properties` 函数
```python
def process_for_additional_properties(
    guide_json: str | dict[str, Any],
) -> dict[str, Any]:
    if isinstance(guide_json, str):
        guide_json_obj = json.loads(guide_json)
    else:
        # copy for modifications
        guide_json_obj = copy.deepcopy(guide_json)
    _walk_json_for_additional_properties(guide_json_obj)
    return guide_json_obj
```
**EN:** This function implements `process_for_additional_properties` within the module. Key calls include `isinstance`, `_walk_json_for_additional_properties`, `loads`, `deepcopy`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `process_for_additional_properties`，其作用域位于the module。 关键调用包括 `isinstance`, `_walk_json_for_additional_properties`, `loads`, `deepcopy`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `GuidanceBackend` class / `GuidanceBackend` 类
```python
@dataclass
class GuidanceBackend(StructuredOutputBackend):
```
**EN:** Uses `@dataclass` to package related state for `GuidanceBackend`. Typical fields include configuration and runtime data.
**CN:** `GuidanceBackend` 使用 `@dataclass` 打包相关状态。典型字段包括 配置与运行期数据。

### `GuidanceBackend.__post_init__` method / `GuidanceBackend.__post_init__` 方法
```python
    def __post_init__(self):
        self.disable_any_whitespace = (
            self.vllm_config.structured_outputs_config.disable_any_whitespace
        )
        self.disable_additional_properties = (
            self.vllm_config.structured_outputs_config.disable_additional_properties
        )

        if is_mistral_tokenizer(self.tokenizer):
            self.ll_tokenizer = self.tokenizer.llg_tokenizer
        else:
            self.ll_tokenizer = llguidance_hf.from_tokenizer(
                self.tokenizer, max(self.vocab_size, len(self.tokenizer))
            )
```
**EN:** This method implements `__post_init__` within `GuidanceBackend`. Key calls include `is_mistral_tokenizer`, `from_tokenizer`, `max`, `len`. It touches state such as `disable_any_whitespace`, `disable_additional_properties`, `ll_tokenizer`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__post_init__`，其作用域位于`GuidanceBackend`。 关键调用包括 `is_mistral_tokenizer`, `from_tokenizer`, `max`, `len`。 它会读写 `disable_any_whitespace`, `disable_additional_properties`, `ll_tokenizer` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `GuidanceBackend.compile_grammar` method / `GuidanceBackend.compile_grammar` 方法
```python
    def compile_grammar(
        self, request_type: StructuredOutputOptions, grammar_spec: str
    ) -> StructuredOutputGrammar:
        self.serialized_grammar = serialize_guidance_grammar(
            request_type,
            grammar_spec,
            self.disable_any_whitespace,
            self.disable_additional_properties,
        )

        ll_matcher = llguidance.LLMatcher(
            self.ll_tokenizer,
            self.serialized_grammar,
            log_level=int(os.environ.get("LLGUIDANCE_LOG_LEVEL", "1")),
        )

        r = GuidanceGrammar(
            ll_matcher=ll_matcher,
            ll_tokenizer=self.ll_tokenizer,
            vocab_size=self.vocab_size,
        )

        r.check_error()
        return r
```
**EN:** This method implements `compile_grammar` within `GuidanceBackend`. Key calls include `serialize_guidance_grammar`, `LLMatcher`, `GuidanceGrammar`, `check_error`, `int`, `get`. It touches state such as `serialized_grammar`.
**CN:** 该方法会实现 `compile_grammar`，其作用域位于`GuidanceBackend`。 关键调用包括 `serialize_guidance_grammar`, `LLMatcher`, `GuidanceGrammar`, `check_error`, `int`, `get`。 它会读写 `serialized_grammar` 等状态。

### `GuidanceBackend.allocate_token_bitmask` method / `GuidanceBackend.allocate_token_bitmask` 方法
```python
    def allocate_token_bitmask(self, max_num_seqs: int):
        return llguidance_torch.allocate_token_bitmask(
            max_num_seqs, self.ll_tokenizer.vocab_size
        )
```
**EN:** This method implements `allocate_token_bitmask` within `GuidanceBackend`. Key calls include `allocate_token_bitmask`.
**CN:** 该方法会实现 `allocate_token_bitmask`，其作用域位于`GuidanceBackend`。 关键调用包括 `allocate_token_bitmask`。

### `GuidanceBackend.destroy` method / `GuidanceBackend.destroy` 方法
```python
    def destroy(self):
        pass
```
**EN:** This method implements `destroy` within `GuidanceBackend`.
**CN:** 该方法会实现 `destroy`，其作用域位于`GuidanceBackend`。

### `GuidanceGrammar` class / `GuidanceGrammar` 类
```python
@dataclass
class GuidanceGrammar(StructuredOutputGrammar):
    ll_matcher: llguidance.LLMatcher
    ll_tokenizer: llguidance.LLTokenizer
    vocab_size: int
    printed_error: bool = False
    terminated: bool = False
    rollback_lag: int = 0
```
**EN:** Uses `@dataclass` to package related state for `GuidanceGrammar`. Typical fields include `ll_matcher`, `ll_tokenizer`, `vocab_size`, `printed_error`, `terminated`, `rollback_lag`.
**CN:** `GuidanceGrammar` 使用 `@dataclass` 打包相关状态。典型字段包括 `ll_matcher`, `ll_tokenizer`, `vocab_size`, `printed_error`, `terminated`, `rollback_lag`。

### `GuidanceGrammar.check_error` method / `GuidanceGrammar.check_error` 方法
```python
    def check_error(self):
        if not self.printed_error:
            err = self.ll_matcher.get_error()
            if err:
                self.printed_error = True
                logger.warning("LLMatcher error: %s", err)
```
**EN:** This method validates assumptions or constraints within `GuidanceGrammar`. Key calls include `get_error`, `warning`. It touches state such as `printed_error`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`GuidanceGrammar`。 关键调用包括 `get_error`, `warning`。 它会读写 `printed_error` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `GuidanceGrammar.accept_tokens` method / `GuidanceGrammar.accept_tokens` 方法
```python
    def accept_tokens(self, request_id: str, tokens: list[int]) -> bool:
        """Accepts a list of tokens and advances the parser.

        Returns True if the parser was advanced successfully.
        Returns False if the parser failed to advance.
        """

        if self.ll_tokenizer.eos_token in tokens:
            if self.ll_matcher.is_stopped() and not self.terminated:
                self.rollback_lag = 1
            self.terminated = True

        if self.ll_matcher.is_stopped():
            return True

        # TODO - Add jump decoding support in the future:
        # self.ll_matcher.compute_ff_bytes() - this should always work
        # self.ll_matcher.compute_ff_tokens() - this only works for
        #   "canonical" tokenizers
        # For conversion between the two, see
        # https://github.com/guidance-ai/llguidance/blob/main/docs/fast_forward.md

        r = self.ll_matcher.consume_tokens(tokens)

        self.check_error()

        return r
```
**EN:** This method implements `accept_tokens` within `GuidanceGrammar`. The docstring frames it as: Accepts a list of tokens and advances the parser. Key calls include `is_stopped`, `consume_tokens`, `check_error`. It touches state such as `terminated`, `rollback_lag`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `accept_tokens`，其作用域位于`GuidanceGrammar`。 关键调用包括 `is_stopped`, `consume_tokens`, `check_error`。 它会读写 `terminated`, `rollback_lag` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `GuidanceGrammar.validate_tokens` method / `GuidanceGrammar.validate_tokens` 方法
```python
    def validate_tokens(self, tokens: list[int]) -> list[int]:
        """Checks if the list of tokens are accepted by the parser in sequence.
        Will not advance the parser.

        Returns the prefix list of tokens that are accepted by the parser.
        """
        if len(tokens) == 0:
            return []
        if self.ll_matcher.is_stopped():
            return []

        num_tokens = self.ll_matcher.validate_tokens(tokens)

        self.check_error()

        return tokens[:num_tokens]
```
**EN:** This method validates assumptions or constraints within `GuidanceGrammar`. The docstring frames it as: Checks if the list of tokens are accepted by the parser in sequence. Key calls include `is_stopped`, `validate_tokens`, `check_error`, `len`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`GuidanceGrammar`。 关键调用包括 `is_stopped`, `validate_tokens`, `check_error`, `len`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `GuidanceGrammar.rollback` method / `GuidanceGrammar.rollback` 方法
```python
    def rollback(self, num_tokens: int) -> None:
        if num_tokens > 0:
            self.ll_matcher.rollback(num_tokens - self.rollback_lag)
            self.terminated = False
            self.rollback_lag = 0
            self.check_error()
```
**EN:** This method implements `rollback` within `GuidanceGrammar`. Key calls include `rollback`, `check_error`. It touches state such as `terminated`, `rollback_lag`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `rollback`，其作用域位于`GuidanceGrammar`。 关键调用包括 `rollback`, `check_error`。 它会读写 `terminated`, `rollback_lag` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `GuidanceGrammar.fill_bitmask` method / `GuidanceGrammar.fill_bitmask` 方法
```python
    def fill_bitmask(self, bitmask: torch.Tensor, idx: int) -> None:
        # this will automatically return [EOS] mask if the matcher is stopped
        # or otherwise in an error state
        llguidance_torch.fill_next_token_bitmask(self.ll_matcher, bitmask, idx)
        self.check_error()
```
**EN:** This method implements `fill_bitmask` within `GuidanceGrammar`. Key calls include `fill_next_token_bitmask`, `check_error`.
**CN:** 该方法会实现 `fill_bitmask`，其作用域位于`GuidanceGrammar`。 关键调用包括 `fill_next_token_bitmask`, `check_error`。

### `GuidanceGrammar.is_terminated` method / `GuidanceGrammar.is_terminated` 方法
```python
    def is_terminated(self) -> bool:
        return self.terminated
```
**EN:** This method answers a boolean capability check within `GuidanceGrammar`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`GuidanceGrammar`。

### `GuidanceGrammar.reset` method / `GuidanceGrammar.reset` 方法
```python
    def reset(self):
        # This method may be not needed anymore? TODO
        self.ll_matcher.reset()
```
**EN:** This method implements `reset` within `GuidanceGrammar`. Key calls include `reset`.
**CN:** 该方法会实现 `reset`，其作用域位于`GuidanceGrammar`。 关键调用包括 `reset`。

### `serialize_guidance_grammar` function / `serialize_guidance_grammar` 函数
```python
def serialize_guidance_grammar(
    request_type: StructuredOutputOptions,
    grammar_spec: str | dict[str, Any],
    disable_any_whitespace: bool = False,
    disable_additional_properties: bool = False,
) -> str:
    def _process_schema(
        grammar_spec: str | dict[str, Any],
    ) -> str:
        if disable_additional_properties:
            grammar_spec = process_for_additional_properties(grammar_spec)
        return llguidance.LLMatcher.grammar_from_json_schema(
            grammar_spec,
            defaults={
                "whitespace_flexible": not disable_any_whitespace,
            },
        )

    if request_type == StructuredOutputOptions.JSON:
        return _process_schema(grammar_spec)
    elif request_type == StructuredOutputOptions.JSON_OBJECT:
        return llguidance.LLMatcher.grammar_from_json_schema(
            '{"type": "object"}',
            defaults={
                "whitespace_flexible": not disable_any_whitespace,
            },
        )
    else:
        if request_type == StructuredOutputOptions.REGEX:
            tp = "regex"
        elif request_type == StructuredOutputOptions.GRAMMAR:
            tp = "grammar"
        elif request_type == StructuredOutputOptions.CHOICE:
            tp = "choice"
        elif request_type == StructuredOutputOptions.STRUCTURAL_TAG:
            if isinstance(grammar_spec, str):
                s_tag = json.loads(grammar_spec)
            else:
                s_tag = grammar_spec
            triggers: list[str] = s_tag["triggers"]
            tags: list[llguidance.StructTag] = []
            for s in s_tag["structures"]:
                begin: str = s["begin"]
                trig = next((t for t in triggers if begin.startswith(t)), None)
                if trig is None:
                    raise ValueError(
                        f"Trigger {begin} not found in triggers {triggers}"
                    )
                tags.append(
                    llguidance.StructTag(
                        trigger=trig,
                        begin=s["begin"],
                        grammar=_process_schema(s["schema"]),
                        end=s["end"],
                    )
                )
            if not tags:
                raise ValueError("No structural tags found in the grammar spec.")
            return llguidance.StructTag.to_grammar(tags)
        else:
            logger.error(
                "Validation should have already occurred. Please file an issue."
            )
            raise ValueError(
                f"grammar is not of valid supported types. ({request_type!s})"
            )
        return llguidance.grammar_from(tp, grammar_spec)
```
**EN:** This function implements `serialize_guidance_grammar` within the module. Key calls include `grammar_from_json_schema`, `_process_schema`, `process_for_additional_properties`, `grammar_from`, `isinstance`, `to_grammar`. The control flow contains 10 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `serialize_guidance_grammar`，其作用域位于the module。 关键调用包括 `grammar_from_json_schema`, `_process_schema`, `process_for_additional_properties`, `grammar_from`, `isinstance`, `to_grammar`。 控制流包含 10 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `validate_guidance_grammar` function / `validate_guidance_grammar` 函数
```python
def validate_guidance_grammar(
    sampling_params: SamplingParams, tokenizer: llguidance.LLTokenizer | None = None
) -> None:
    # if structured output is not enabled, there is nothing to validate
    if sampling_params.structured_outputs is None:
        return
    tp, grm = get_structured_output_key(sampling_params.structured_outputs)
    guidance_grm = serialize_guidance_grammar(tp, grm)
    err = llguidance.LLMatcher.validate_grammar(guidance_grm, tokenizer)
    if err:
        raise ValueError(f"Grammar error: {err}")
```
**EN:** This function validates assumptions or constraints within the module. Key calls include `get_structured_output_key`, `serialize_guidance_grammar`, `validate_grammar`, `ValueError`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `get_structured_output_key`, `serialize_guidance_grammar`, `validate_grammar`, `ValueError`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_walk_json_for_additional_properties`: top-level helper or orchestration entry point. / `_walk_json_for_additional_properties`：顶层辅助函数或编排入口。
- `has_guidance_unsupported_json_features`: top-level helper or orchestration entry point. / `has_guidance_unsupported_json_features`：顶层辅助函数或编排入口。
- `process_for_additional_properties`: top-level helper or orchestration entry point. / `process_for_additional_properties`：顶层辅助函数或编排入口。
- `GuidanceBackend`: central class or interface in this module. / `GuidanceBackend`：本模块中的核心类或接口。
- `GuidanceGrammar`: central class or interface in this module. / `GuidanceGrammar`：本模块中的核心类或接口。
- `serialize_guidance_grammar`: top-level helper or orchestration entry point. / `serialize_guidance_grammar`：顶层辅助函数或编排入口。
- `validate_guidance_grammar`: top-level helper or orchestration entry point. / `validate_guidance_grammar`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `copy`, `json`, `os`, `dataclasses`, `typing`
- External / 外部依赖: `torch`, `llguidance`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.sampling_params`, `vllm.utils.import_utils`, `vllm.utils.mistral`, `vllm.v1.structured_output.backend_types`, `vllm.v1.structured_output.request`
