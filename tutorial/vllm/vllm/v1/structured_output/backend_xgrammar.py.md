# backend_xgrammar.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/structured_output/backend_xgrammar.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `XgrammarBackend`, `XgrammarGrammar`, `has_xgrammar_unsupported_json_features` for the V1 `structured_output` subsystem. / 为 V1 的 `structured_output` 子系统实现 `XgrammarBackend`, `XgrammarGrammar`, `has_xgrammar_unsupported_json_features`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import json
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any

import torch

import vllm.envs
from vllm.logger import init_logger
from vllm.sampling_params import SamplingParams
from vllm.utils.import_utils import LazyLoader
from vllm.utils.mistral import is_mistral_tokenizer
from vllm.v1.structured_output.backend_types import (
    StructuredOutputBackend,
    StructuredOutputGrammar,
    StructuredOutputOptions,
)
from vllm.v1.structured_output.utils import (
    choice_as_grammar,
    convert_lark_to_ebnf,
    grammar_is_likely_lark,
)

if TYPE_CHECKING:
    import xgrammar as xgr
else:
    xgr = LazyLoader("xgr", globals(), "xgrammar")

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `XgrammarBackend` class / `XgrammarBackend` 类
```python
@dataclass
class XgrammarBackend(StructuredOutputBackend):
```
**EN:** Uses `@dataclass` to package related state for `XgrammarBackend`. Typical fields include configuration and runtime data.
**CN:** `XgrammarBackend` 使用 `@dataclass` 打包相关状态。典型字段包括 配置与运行期数据。

### `XgrammarBackend.__post_init__` method / `XgrammarBackend.__post_init__` 方法
```python
    def __post_init__(self):
        self.disable_any_whitespace = (
            self.vllm_config.structured_outputs_config.disable_any_whitespace
        )

        if is_mistral_tokenizer(self.tokenizer):
            # NOTE: ideally, xgrammar should handle this accordingly.
            # refer to https://github.com/mlc-ai/xgrammar/blob/d77c0a0173ef14779c918e3be7966ba852f7910f/python/xgrammar/tokenizer_info.py#L98
            stop_token_ids = [self.tokenizer.eos_token_id]

            # not self.tokenizer.vocab_size as self.tokenizer.vocab
            # collapses all decoded errors into a single token.
            self.vocab_size = len(self.tokenizer.vocab)
            tokenizer_info = xgr.TokenizerInfo(  # type: ignore
                encoded_vocab=self.tokenizer.vocab,
                # NOTE: https://github.com/mlc-ai/xgrammar/blob/5e141f6ff1ca02bc31f9e512e68b61f2a8ae88e5/tests/python/test_tokenizer_info.py#L43 # noqa: E501
                vocab_type=xgr.VocabType.RAW
                if self.tokenizer.is_tekken
                else xgr.VocabType.BYTE_FALLBACK,
                vocab_size=self.vocab_size,
                stop_token_ids=stop_token_ids,
                add_prefix_space=True,
            )
        else:
            tokenizer_info = xgr.TokenizerInfo.from_huggingface(
                self.tokenizer,
                vocab_size=self.vocab_size,
            )
        self.compiler = xgr.GrammarCompiler(
            tokenizer_info,
            max_threads=8,
            cache_enabled=True,
            cache_limit_bytes=vllm.envs.VLLM_XGRAMMAR_CACHE_MB * 1024 * 1024,
        )

        self.num_speculative_tokens = 0
        if self.vllm_config.speculative_config is not None:
            self.num_speculative_tokens = (
                self.vllm_config.speculative_config.num_speculative_tokens
            )
```
**EN:** This method implements `__post_init__` within `XgrammarBackend`. Key calls include `is_mistral_tokenizer`, `GrammarCompiler`, `len`, `TokenizerInfo`, `from_huggingface`. It touches state such as `disable_any_whitespace`, `compiler`, `num_speculative_tokens`, `vocab_size`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__post_init__`，其作用域位于`XgrammarBackend`。 关键调用包括 `is_mistral_tokenizer`, `GrammarCompiler`, `len`, `TokenizerInfo`, `from_huggingface`。 它会读写 `disable_any_whitespace`, `compiler`, `num_speculative_tokens`, `vocab_size` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `XgrammarBackend.compile_grammar` method / `XgrammarBackend.compile_grammar` 方法
```python
    def compile_grammar(
        self, request_type: StructuredOutputOptions, grammar_spec: str
    ) -> StructuredOutputGrammar:
        if request_type == StructuredOutputOptions.JSON:
            ctx = self.compiler.compile_json_schema(
                grammar_spec, any_whitespace=not self.disable_any_whitespace
            )
        elif request_type == StructuredOutputOptions.JSON_OBJECT:
            ctx = self.compiler.compile_json_schema(
                '{"type": "object"}', any_whitespace=not self.disable_any_whitespace
            )
        elif request_type == StructuredOutputOptions.GRAMMAR:
            ctx = self.compiler.compile_grammar(grammar_spec)
        elif request_type == StructuredOutputOptions.REGEX:
            ctx = self.compiler.compile_regex(grammar_spec)
        elif request_type == StructuredOutputOptions.STRUCTURAL_TAG:
            s_tag = json.loads(grammar_spec)
            if "structures" in s_tag:
                # Falling back to deprecated method of compiling structural tag
                tags = [
                    xgr.StructuralTagItem(
                        begin=s["begin"],
                        schema=json.dumps(s["schema"]),
                        end=s["end"],
                    )
                    for s in s_tag["structures"]
                ]
                ctx = self.compiler.compile_structural_tag(tags, s_tag["triggers"])
            else:
                ctx = self.compiler.compile_structural_tag(grammar_spec)
        else:
            logger.error(
                "Validation should have already occurred. Please file an issue."
            )
            raise ValueError(
                f"grammar is not of valid supported types. ({request_type!s})"
            )

        return XgrammarGrammar(
            matcher=xgr.GrammarMatcher(
                ctx,
                max_rollback_tokens=self.num_speculative_tokens,
            ),
            vocab_size=self.vocab_size,
            ctx=ctx,
        )
```
**EN:** This method implements `compile_grammar` within `XgrammarBackend`. Key calls include `XgrammarGrammar`, `compile_json_schema`, `GrammarMatcher`, `compile_grammar`, `compile_regex`, `loads`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `compile_grammar`，其作用域位于`XgrammarBackend`。 关键调用包括 `XgrammarGrammar`, `compile_json_schema`, `GrammarMatcher`, `compile_grammar`, `compile_regex`, `loads`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `XgrammarBackend.allocate_token_bitmask` method / `XgrammarBackend.allocate_token_bitmask` 方法
```python
    def allocate_token_bitmask(self, max_num_seqs: int):
        return xgr.allocate_token_bitmask(max_num_seqs, self.vocab_size)
```
**EN:** This method implements `allocate_token_bitmask` within `XgrammarBackend`. Key calls include `allocate_token_bitmask`.
**CN:** 该方法会实现 `allocate_token_bitmask`，其作用域位于`XgrammarBackend`。 关键调用包括 `allocate_token_bitmask`。

### `XgrammarBackend.destroy` method / `XgrammarBackend.destroy` 方法
```python
    def destroy(self):
        del self.compiler
```
**EN:** This method implements `destroy` within `XgrammarBackend`.
**CN:** 该方法会实现 `destroy`，其作用域位于`XgrammarBackend`。

### `XgrammarGrammar` class / `XgrammarGrammar` 类
```python
@dataclass
class XgrammarGrammar(StructuredOutputGrammar):
    # NOTE: This would be a generic-enough class for
    # supporting different backends, in the future.
    # For now, just xgrammar.
    #
    # https://xgrammar.mlc.ai/docs/api/python/index.html#xgrammar.GrammarMatcher.find_jump_forward_string
    # for jump-forward decoding

    vocab_size: int
    matcher: xgr.GrammarMatcher = field(hash=False)
    ctx: xgr.CompiledGrammar = field(hash=False)
    num_processed_tokens: int = field(
        default_factory=lambda: 0, repr=False, hash=False, init=False
    )
    _is_terminated: bool = field(default=False, repr=False, hash=False)
```
**EN:** Uses `@dataclass` to package related state for `XgrammarGrammar`. Typical fields include `vocab_size`, `matcher`, `ctx`, `num_processed_tokens`, `_is_terminated`.
**CN:** `XgrammarGrammar` 使用 `@dataclass` 打包相关状态。典型字段包括 `vocab_size`, `matcher`, `ctx`, `num_processed_tokens`, `_is_terminated`。

### `XgrammarGrammar.accept_tokens` method / `XgrammarGrammar.accept_tokens` 方法
```python
    def accept_tokens(self, request_id: str, tokens: list[int]) -> bool:
        """Accepts a list of tokens and advances the FSM.

        Returns True if the FSM was advanced successfully.
        Returns False if the FSM failed to advance.
        """
        if self._is_terminated:
            return False
        for token in tokens:
            if not self.matcher.accept_token(token):
                logger.error(
                    "Failed to advance FSM for request %s "
                    "for tokens %s. Please file an issue.",
                    request_id,
                    token,
                )
                return False
            self.num_processed_tokens += 1
        self._is_terminated = self.matcher.is_terminated()
        return True
```
**EN:** This method implements `accept_tokens` within `XgrammarGrammar`. The docstring frames it as: Accepts a list of tokens and advances the FSM. Key calls include `is_terminated`, `accept_token`, `error`. It touches state such as `_is_terminated`, `num_processed_tokens`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `accept_tokens`，其作用域位于`XgrammarGrammar`。 关键调用包括 `is_terminated`, `accept_token`, `error`。 它会读写 `_is_terminated`, `num_processed_tokens` 等状态。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `XgrammarGrammar.validate_tokens` method / `XgrammarGrammar.validate_tokens` 方法
```python
    def validate_tokens(self, tokens: list[int]) -> list[int]:
        """Checks if the list of tokens are accepted by the FSM in sequence.
        Will not advance the FSM.

        Returns the prefix list of tokens that are accepted by the FSM.
        """
        accepted_tokens = []
        for token in tokens:
            if self.matcher.accept_token(token):
                accepted_tokens.append(token)
            else:
                break
        if len(accepted_tokens) > 0:
            # Rollback the FSM to the initial state
            self.matcher.rollback(len(accepted_tokens))
        return accepted_tokens
```
**EN:** This method validates assumptions or constraints within `XgrammarGrammar`. The docstring frames it as: Checks if the list of tokens are accepted by the FSM in sequence. Key calls include `accept_token`, `len`, `rollback`, `append`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`XgrammarGrammar`。 关键调用包括 `accept_token`, `len`, `rollback`, `append`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `XgrammarGrammar.rollback` method / `XgrammarGrammar.rollback` 方法
```python
    def rollback(self, num_tokens: int) -> None:
        self.matcher.rollback(num_tokens)
        self.num_processed_tokens -= num_tokens
        self._is_terminated = self.matcher.is_terminated()
```
**EN:** This method implements `rollback` within `XgrammarGrammar`. Key calls include `rollback`, `is_terminated`. It touches state such as `num_processed_tokens`, `_is_terminated`.
**CN:** 该方法会实现 `rollback`，其作用域位于`XgrammarGrammar`。 关键调用包括 `rollback`, `is_terminated`。 它会读写 `num_processed_tokens`, `_is_terminated` 等状态。

### `XgrammarGrammar.fill_bitmask` method / `XgrammarGrammar.fill_bitmask` 方法
```python
    def fill_bitmask(self, bitmask: torch.Tensor, idx: int) -> None:
        self.matcher.fill_next_token_bitmask(bitmask, idx)
```
**EN:** This method implements `fill_bitmask` within `XgrammarGrammar`. Key calls include `fill_next_token_bitmask`.
**CN:** 该方法会实现 `fill_bitmask`，其作用域位于`XgrammarGrammar`。 关键调用包括 `fill_next_token_bitmask`。

### `XgrammarGrammar.is_terminated` method / `XgrammarGrammar.is_terminated` 方法
```python
    def is_terminated(self) -> bool:
        return self._is_terminated
```
**EN:** This method answers a boolean capability check within `XgrammarGrammar`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`XgrammarGrammar`。

### `XgrammarGrammar.reset` method / `XgrammarGrammar.reset` 方法
```python
    def reset(self):
        self.num_processed_tokens = 0
        self.matcher.reset()
```
**EN:** This method implements `reset` within `XgrammarGrammar`. Key calls include `reset`. It touches state such as `num_processed_tokens`.
**CN:** 该方法会实现 `reset`，其作用域位于`XgrammarGrammar`。 关键调用包括 `reset`。 它会读写 `num_processed_tokens` 等状态。

### Module constants / 模块常量
```python
STRING_SUPPORTED_FORMATS = {
    "email",
    "date",
    "time",
    "date-time",
    "duration",
    "ipv4",
    "ipv6",
    "hostname",
    "uuid",
    "uri",
    "uri-reference",
    "uri-template",
    "json-pointer",
    "relative-json-pointer",
}
```
**EN:** Defines module-level constants or aliases such as `STRING_SUPPORTED_FORMATS`, which are reused by later definitions.
**CN:** 定义 `STRING_SUPPORTED_FORMATS` 等模块级常量或别名，供后续定义复用。

### `has_xgrammar_unsupported_json_features` function / `has_xgrammar_unsupported_json_features` 函数
```python
def has_xgrammar_unsupported_json_features(schema: dict[str, Any]) -> bool:
    """Check if JSON schema contains features unsupported by xgrammar."""

    def check_object(obj: dict[str, Any]) -> bool:
        if not isinstance(obj, dict):
            return False

        # Check for numeric ranges
        if obj.get("type") in ("integer", "number") and ("multipleOf" in obj):
            return True

        # Check for array unsupported keywords
        if obj.get("type") == "array" and any(
            key in obj
            for key in ("uniqueItems", "contains", "minContains", "maxContains")
        ):
            return True

        # Unsupported keywords for strings
        if (
            obj.get("type") == "string"
            and "format" in obj
            and obj["format"] not in STRING_SUPPORTED_FORMATS
        ):
            return True

        # Unsupported keywords for objects
        if obj.get("type") == "object" and any(
            key in obj for key in ("patternProperties", "propertyNames")
        ):
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
**EN:** This function implements `has_xgrammar_unsupported_json_features` within the module. The docstring frames it as: Check if JSON schema contains features unsupported by xgrammar. Key calls include `check_object`, `values`, `isinstance`, `any`, `get`. The control flow contains 9 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `has_xgrammar_unsupported_json_features`，其作用域位于the module。 关键调用包括 `check_object`, `values`, `isinstance`, `any`, `get`。 控制流包含 9 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `validate_xgrammar_grammar` function / `validate_xgrammar_grammar` 函数
```python
def validate_xgrammar_grammar(sampling_params: SamplingParams) -> None:
    """Validate that the request is supported by structured output.

    Raises ValueError if the request is not supported.
    """
    if sampling_params.structured_outputs is None:
        return

    so_params = sampling_params.structured_outputs

    if so_params.regex:
        try:
            xgr.Grammar.from_regex(so_params.regex)
        except Exception as err:
            raise ValueError(
                f"Failed to transform regex into a grammar: {err}"
            ) from err

    if so_params.choice:
        choice_grammar = choice_as_grammar(so_params.choice)
        try:
            xgr.Grammar.from_ebnf(choice_grammar)
        except Exception as err:
            raise ValueError(
                f"Failed to transform choices into a grammar: {err}"
            ) from err
        so_params.choice = None
        so_params.grammar = choice_grammar
        return

    if so_params.json:
        if isinstance(so_params.json, str):
            try:
                schema = json.loads(so_params.json)
            except json.JSONDecodeError as e:
                raise ValueError("Invalid JSON grammar specification.") from e
        else:
            schema = so_params.json

        if has_xgrammar_unsupported_json_features(schema):
            raise ValueError(
                "The provided JSON schema contains features not supported by xgrammar."
            )

        try:
            xgr.Grammar.from_json_schema(schema)
        except Exception as err:
            raise ValueError(
                f"Failed to transform json schema into a grammar: {err}"
            ) from err
        return

    if so_params.grammar:
        if grammar_is_likely_lark(so_params.grammar):
            # xgrammar supports EBNF grammars only
            try:
                so_params.grammar = convert_lark_to_ebnf(so_params.grammar)
            except ValueError as e:
                raise ValueError(
                    "Failed to convert the grammar from Lark to EBNF. "
                ) from e

        # Test parsing EBNF grammar, possibly already converted from Lark
        try:
            # parse the grammar, but we aren't compiling it.
            xgr.Grammar.from_ebnf(so_params.grammar)
        except Exception as e:
            raise ValueError("Invalid grammar specification.") from e
        return

    if so_params.structural_tag:
        try:
            s_tag = json.loads(so_params.structural_tag)

            # Using the deprecated method of compiling structural tag
            if "structures" in s_tag:
                tags = [
                    xgr.StructuralTagItem(
                        begin=s["begin"],
                        schema=json.dumps(s["schema"]),
                        end=s["end"],
                    )
                    for s in s_tag["structures"]
                ]
                xgr.Grammar.from_structural_tag(tags, s_tag["triggers"])
            else:
                xgr.Grammar.from_structural_tag(so_params.structural_tag)
        except Exception as e:
            raise ValueError("Invalid structural tag specification.") from e
```
**EN:** This function validates assumptions or constraints within the module. The docstring frames it as: Validate that the request is supported by structured output. Key calls include `choice_as_grammar`, `isinstance`, `has_xgrammar_unsupported_json_features`, `grammar_is_likely_lark`, `from_regex`, `from_ebnf`. The control flow contains 17 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `choice_as_grammar`, `isinstance`, `has_xgrammar_unsupported_json_features`, `grammar_is_likely_lark`, `from_regex`, `from_ebnf`。 控制流包含 17 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `XgrammarBackend`: central class or interface in this module. / `XgrammarBackend`：本模块中的核心类或接口。
- `XgrammarGrammar`: central class or interface in this module. / `XgrammarGrammar`：本模块中的核心类或接口。
- `has_xgrammar_unsupported_json_features`: top-level helper or orchestration entry point. / `has_xgrammar_unsupported_json_features`：顶层辅助函数或编排入口。
- `validate_xgrammar_grammar`: top-level helper or orchestration entry point. / `validate_xgrammar_grammar`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `json`, `dataclasses`, `typing`
- External / 外部依赖: `torch`, `xgrammar`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.logger`, `vllm.sampling_params`, `vllm.utils.import_utils`, `vllm.utils.mistral`, `vllm.v1.structured_output.backend_types`, `vllm.v1.structured_output.utils`
