# backend_lm_format_enforcer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/structured_output/backend_lm_format_enforcer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_cached_build_vllm_token_enforcer_tokenizer_data`, `LMFormatEnforcerGrammar`, `LMFormatEnforcerBackend` for the V1 `structured_output` subsystem. / 为 V1 的 `structured_output` 子系统实现 `_cached_build_vllm_token_enforcer_tokenizer_data`, `LMFormatEnforcerGrammar`, `LMFormatEnforcerBackend`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import ast
import json
from dataclasses import dataclass, field
from functools import lru_cache
from typing import TYPE_CHECKING

import torch
from transformers import PreTrainedTokenizerBase

from vllm.sampling_params import SamplingParams
from vllm.utils.import_utils import LazyLoader
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.v1.structured_output.backend_types import (
    StructuredOutputBackend,
    StructuredOutputGrammar,
    StructuredOutputOptions,
)

if TYPE_CHECKING:
    import lmformatenforcer
    import lmformatenforcer.integrations.vllm as lmfe_vllm
else:
    lmformatenforcer = LazyLoader("lmformatenforcer", globals(), "lmformatenforcer")
    lmfe_vllm = LazyLoader(
        "lmformatenforcer.integrations.vllm",
        globals(),
        "lmformatenforcer.integrations.vllm",
    )
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, `transformers`, `lmformatenforcer`, and internal vLLM modules such as `vllm.sampling_params`, `vllm.utils.import_utils`, `vllm.utils.platform_utils`, `vllm.v1.structured_output.backend_types`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch`, `transformers`, `lmformatenforcer` 等外部依赖，以及 `vllm.sampling_params`, `vllm.utils.import_utils`, `vllm.utils.platform_utils`, `vllm.v1.structured_output.backend_types` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `_cached_build_vllm_token_enforcer_tokenizer_data` function / `_cached_build_vllm_token_enforcer_tokenizer_data` 函数
```python
@lru_cache
def _cached_build_vllm_token_enforcer_tokenizer_data(
    tokenizer: PreTrainedTokenizerBase, vocab_size: int
) -> "lmfe_vllm.TokenEnforcerTokenizerData":
    return lmfe_vllm.build_vllm_token_enforcer_tokenizer_data(
        tokenizer, use_bitmask=True, vocab_size=vocab_size
    )
```
**EN:** This function implements `_cached_build_vllm_token_enforcer_tokenizer_data` within the module. Key calls include `build_vllm_token_enforcer_tokenizer_data`.
**CN:** 该函数会实现 `_cached_build_vllm_token_enforcer_tokenizer_data`，其作用域位于the module。 关键调用包括 `build_vllm_token_enforcer_tokenizer_data`。

### `LMFormatEnforcerGrammar` class / `LMFormatEnforcerGrammar` 类
```python
@dataclass
class LMFormatEnforcerGrammar(StructuredOutputGrammar):
    token_enforcer: lmformatenforcer.TokenEnforcer
    current_tokens_prefix: list[int] = field(default_factory=list)
```
**EN:** Uses `@dataclass` to package related state for `LMFormatEnforcerGrammar`. Typical fields include `token_enforcer`, `current_tokens_prefix`.
**CN:** `LMFormatEnforcerGrammar` 使用 `@dataclass` 打包相关状态。典型字段包括 `token_enforcer`, `current_tokens_prefix`。

### `LMFormatEnforcerGrammar.accept_tokens` method / `LMFormatEnforcerGrammar.accept_tokens` 方法
```python
    def accept_tokens(self, request_id: str, tokens: list[int]) -> bool:
        original_len = len(self.current_tokens_prefix)
        for token in tokens:
            if not self.token_enforcer.get_allowed_tokens(
                self.current_tokens_prefix
            ).is_token_allowed(token):
                # Rollback partial updates to ensure atomicity.
                del self.current_tokens_prefix[original_len:]
                return False
            self.current_tokens_prefix.append(token)
        return True
```
**EN:** This method implements `accept_tokens` within `LMFormatEnforcerGrammar`. Key calls include `len`, `append`, `is_token_allowed`, `get_allowed_tokens`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `accept_tokens`，其作用域位于`LMFormatEnforcerGrammar`。 关键调用包括 `len`, `append`, `is_token_allowed`, `get_allowed_tokens`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `LMFormatEnforcerGrammar.validate_tokens` method / `LMFormatEnforcerGrammar.validate_tokens` 方法
```python
    def validate_tokens(self, tokens: list[int]) -> list[int]:
        for prefix_length in range(len(tokens)):
            prefix = tokens[:prefix_length]
            next_token = tokens[prefix_length]
            if not self.token_enforcer.get_allowed_tokens(
                self.current_tokens_prefix + prefix
            ).is_token_allowed(next_token):
                break
        else:
            return tokens

        return tokens[:prefix_length]
```
**EN:** This method validates assumptions or constraints within `LMFormatEnforcerGrammar`. Key calls include `range`, `len`, `is_token_allowed`, `get_allowed_tokens`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`LMFormatEnforcerGrammar`。 关键调用包括 `range`, `len`, `is_token_allowed`, `get_allowed_tokens`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `LMFormatEnforcerGrammar.rollback` method / `LMFormatEnforcerGrammar.rollback` 方法
```python
    def rollback(self, num_tokens: int) -> None:
        self.current_tokens_prefix = self.current_tokens_prefix[:-num_tokens]
```
**EN:** This method implements `rollback` within `LMFormatEnforcerGrammar`. It touches state such as `current_tokens_prefix`.
**CN:** 该方法会实现 `rollback`，其作用域位于`LMFormatEnforcerGrammar`。 它会读写 `current_tokens_prefix` 等状态。

### `LMFormatEnforcerGrammar.fill_bitmask` method / `LMFormatEnforcerGrammar.fill_bitmask` 方法
```python
    def fill_bitmask(self, bitmask: torch.Tensor, batch_index: int) -> None:
        allowed_tokens = self.token_enforcer.get_allowed_tokens(
            self.current_tokens_prefix
        )
        bitmask[batch_index] = allowed_tokens.allowed_tokens
```
**EN:** This method implements `fill_bitmask` within `LMFormatEnforcerGrammar`. Key calls include `get_allowed_tokens`.
**CN:** 该方法会实现 `fill_bitmask`，其作用域位于`LMFormatEnforcerGrammar`。 关键调用包括 `get_allowed_tokens`。

### `LMFormatEnforcerGrammar.is_terminated` method / `LMFormatEnforcerGrammar.is_terminated` 方法
```python
    def is_terminated(self) -> bool:
        # We are considered terminated if the prefix ends with eos_token_id
        return_value = (
            len(self.current_tokens_prefix) > 0
            and self.current_tokens_prefix[-1] == self.token_enforcer.eos_token_id
        )
        return return_value
```
**EN:** This method answers a boolean capability check within `LMFormatEnforcerGrammar`. Key calls include `len`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`LMFormatEnforcerGrammar`。 关键调用包括 `len`。

### `LMFormatEnforcerGrammar.reset` method / `LMFormatEnforcerGrammar.reset` 方法
```python
    def reset(self):
        self.current_tokens_prefix = []
```
**EN:** This method implements `reset` within `LMFormatEnforcerGrammar`. It touches state such as `current_tokens_prefix`.
**CN:** 该方法会实现 `reset`，其作用域位于`LMFormatEnforcerGrammar`。 它会读写 `current_tokens_prefix` 等状态。

### `LMFormatEnforcerBackend` class / `LMFormatEnforcerBackend` 类
```python
@dataclass
class LMFormatEnforcerBackend(StructuredOutputBackend):
```
**EN:** Uses `@dataclass` to package related state for `LMFormatEnforcerBackend`. Typical fields include configuration and runtime data.
**CN:** `LMFormatEnforcerBackend` 使用 `@dataclass` 打包相关状态。典型字段包括 配置与运行期数据。

### `LMFormatEnforcerBackend.__post_init__` method / `LMFormatEnforcerBackend.__post_init__` 方法
```python
    def __post_init__(self):
        self.tokenizer_data = _cached_build_vllm_token_enforcer_tokenizer_data(
            self.tokenizer, self.vocab_size
        )
```
**EN:** This method implements `__post_init__` within `LMFormatEnforcerBackend`. Key calls include `_cached_build_vllm_token_enforcer_tokenizer_data`. It touches state such as `tokenizer_data`.
**CN:** 该方法会实现 `__post_init__`，其作用域位于`LMFormatEnforcerBackend`。 关键调用包括 `_cached_build_vllm_token_enforcer_tokenizer_data`。 它会读写 `tokenizer_data` 等状态。

### `LMFormatEnforcerBackend.compile_grammar` method / `LMFormatEnforcerBackend.compile_grammar` 方法
```python
    def compile_grammar(
        self, request_type: StructuredOutputOptions, grammar_spec: str
    ) -> StructuredOutputGrammar:
        character_level_parser: lmformatenforcer.CharacterLevelParser
        if request_type == StructuredOutputOptions.JSON:
            spec_dict = json.loads(grammar_spec)
            character_level_parser = lmformatenforcer.JsonSchemaParser(spec_dict)
        elif request_type == StructuredOutputOptions.JSON_OBJECT:
            character_level_parser = lmformatenforcer.JsonSchemaParser(None)
        elif request_type == StructuredOutputOptions.REGEX:
            character_level_parser = lmformatenforcer.RegexParser(grammar_spec)
        elif request_type == StructuredOutputOptions.CHOICE:
            choices = ast.literal_eval(grammar_spec)
            character_level_parser = lmformatenforcer.UnionParser(
                [lmformatenforcer.StringParser(choice) for choice in choices]
            )
        else:
            raise ValueError(
                f"Invalid request type for LM Format Enforcer backend({request_type!s})"
            )
        max_rollback_tokens = (
            self.vllm_config.speculative_config.num_speculative_tokens
            if self.vllm_config.speculative_config is not None
            else 0
        )

        if max_rollback_tokens > 0:
            raise ValueError(
                "LM Format Enforcer backend does not support speculative tokens"
            )

        token_enforcer = lmformatenforcer.TokenEnforcer(
            tokenizer_data=self.tokenizer_data,
            parser=character_level_parser,
        )
        return LMFormatEnforcerGrammar(token_enforcer)
```
**EN:** This method implements `compile_grammar` within `LMFormatEnforcerBackend`. Key calls include `TokenEnforcer`, `LMFormatEnforcerGrammar`, `loads`, `JsonSchemaParser`, `ValueError`, `RegexParser`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `compile_grammar`，其作用域位于`LMFormatEnforcerBackend`。 关键调用包括 `TokenEnforcer`, `LMFormatEnforcerGrammar`, `loads`, `JsonSchemaParser`, `ValueError`, `RegexParser`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LMFormatEnforcerBackend.allocate_token_bitmask` method / `LMFormatEnforcerBackend.allocate_token_bitmask` 方法
```python
    def allocate_token_bitmask(self, max_num_seqs: int) -> torch.Tensor:
        return torch.full(
            (max_num_seqs, (self.vocab_size + 31) // 32),
            -1,
            dtype=torch.int32,
            pin_memory=is_pin_memory_available(),
        )
```
**EN:** This method implements `allocate_token_bitmask` within `LMFormatEnforcerBackend`. Key calls include `full`, `is_pin_memory_available`.
**CN:** 该方法会实现 `allocate_token_bitmask`，其作用域位于`LMFormatEnforcerBackend`。 关键调用包括 `full`, `is_pin_memory_available`。

### `LMFormatEnforcerBackend.destroy` method / `LMFormatEnforcerBackend.destroy` 方法
```python
    def destroy(self):
        pass
```
**EN:** This method implements `destroy` within `LMFormatEnforcerBackend`.
**CN:** 该方法会实现 `destroy`，其作用域位于`LMFormatEnforcerBackend`。

### `validate_structured_output_request_lm_format_enforcer` function / `validate_structured_output_request_lm_format_enforcer` 函数
```python
def validate_structured_output_request_lm_format_enforcer(params: SamplingParams):
    if params.structured_outputs is None:
        return

    so_params = params.structured_outputs

    if so_params.regex:
        return
    elif so_params.json:
        if isinstance(so_params.json, str):
            try:
                # make sure schema is valid json
                json.loads(so_params.json)
            except json.JSONDecodeError as e:
                raise ValueError("Invalid JSON grammar specification.") from e
        else:
            try:
                json.dumps(so_params.json)
            except Exception as e:
                raise ValueError(
                    f"Error serializing structured outputs jsonschema: {e}"
                ) from e
        return
    elif so_params.choice:
        return
    elif so_params.grammar:
        raise ValueError(
            "LM Format Enforcer structured outputs backend "
            "does not support grammar specifications"
        )
```
**EN:** This function validates assumptions or constraints within the module. Key calls include `isinstance`, `loads`, `dumps`, `ValueError`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `isinstance`, `loads`, `dumps`, `ValueError`。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_cached_build_vllm_token_enforcer_tokenizer_data`: top-level helper or orchestration entry point. / `_cached_build_vllm_token_enforcer_tokenizer_data`：顶层辅助函数或编排入口。
- `LMFormatEnforcerGrammar`: central class or interface in this module. / `LMFormatEnforcerGrammar`：本模块中的核心类或接口。
- `LMFormatEnforcerBackend`: central class or interface in this module. / `LMFormatEnforcerBackend`：本模块中的核心类或接口。
- `validate_structured_output_request_lm_format_enforcer`: top-level helper or orchestration entry point. / `validate_structured_output_request_lm_format_enforcer`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `ast`, `json`, `dataclasses`, `functools`, `typing`
- External / 外部依赖: `torch`, `transformers`, `lmformatenforcer`
- Internal vLLM / 内部依赖: `vllm.sampling_params`, `vllm.utils.import_utils`, `vllm.utils.platform_utils`, `vllm.v1.structured_output.backend_types`
