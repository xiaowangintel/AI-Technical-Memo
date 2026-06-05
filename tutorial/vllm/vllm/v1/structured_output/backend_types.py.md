# backend_types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/structured_output/backend_types.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `StructuredOutputOptions`, `StructuredOutputGrammar`, `StructuredOutputBackend` for the V1 `structured_output` subsystem. / 为 V1 的 `structured_output` 子系统实现 `StructuredOutputOptions`, `StructuredOutputGrammar`, `StructuredOutputBackend`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import enum
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    import torch

    from vllm.config import VllmConfig
    from vllm.tokenizers import TokenizerLike
else:
    VllmConfig = object
    TokenizerLike = object
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.tokenizers`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.tokenizers` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `StructuredOutputOptions` class / `StructuredOutputOptions` 类
```python
class StructuredOutputOptions(enum.Enum):
    JSON = enum.auto()
    JSON_OBJECT = enum.auto()
    REGEX = enum.auto()
    GRAMMAR = enum.auto()
    CHOICE = enum.auto()
    STRUCTURAL_TAG = enum.auto()
```
**EN:** Defines the `StructuredOutputOptions` enum used to normalize modes or options across the subsystem. Representative members: `JSON`, `JSON_OBJECT`, `REGEX`, `GRAMMAR`, `CHOICE`, `STRUCTURAL_TAG`.
**CN:** `StructuredOutputOptions` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`JSON`, `JSON_OBJECT`, `REGEX`, `GRAMMAR`, `CHOICE`, `STRUCTURAL_TAG`。

### Module constants / 模块常量
```python
StructuredOutputKey = tuple[StructuredOutputOptions, str]
```
**EN:** Defines module-level constants or aliases such as `StructuredOutputKey`, which are reused by later definitions.
**CN:** 定义 `StructuredOutputKey` 等模块级常量或别名，供后续定义复用。

### `StructuredOutputGrammar` class / `StructuredOutputGrammar` 类
```python
class StructuredOutputGrammar(ABC):
    """Request-level backend for structured output requests."""
```
**EN:** Declares the `StructuredOutputGrammar` interface. Downstream implementations are expected to provide methods such as `accept_tokens`, `validate_tokens`, `rollback`, `fill_bitmask`, `is_terminated`, `reset`.
**CN:** `StructuredOutputGrammar` 声明了一组接口约定。下游实现需要提供 `accept_tokens`, `validate_tokens`, `rollback`, `fill_bitmask`, `is_terminated`, `reset` 等方法。

### `StructuredOutputGrammar.accept_tokens` method / `StructuredOutputGrammar.accept_tokens` 方法
```python
    @abstractmethod
    def accept_tokens(self, request_id: str, tokens: list[int]) -> bool:
        """
        Determines whether the provided tokens are accepted for the
        given request.

        Args:
            request_id (str): The unique identifier for the request.
            tokens (list[int]): A list of token IDs to evaluate.

        Returns:
            bool: True if the tokens are accepted, False otherwise.
        """
```
**EN:** This method implements `accept_tokens` within `StructuredOutputGrammar`. The docstring frames it as: Determines whether the provided tokens are accepted for the given request.
**CN:** 该方法会实现 `accept_tokens`，其作用域位于`StructuredOutputGrammar`。

### `StructuredOutputGrammar.validate_tokens` method / `StructuredOutputGrammar.validate_tokens` 方法
```python
    @abstractmethod
    def validate_tokens(self, tokens: list[int]) -> list[int]:
        """
        Validates the provided tokens against the grammar.
        Will not advance the FSM.

        Args:
            tokens (list[int]): A list of token IDs to validate.

        Returns:
            list[int]: A list of accepted token IDs. Will be a prefix
                of the input tokens, and empty if none are accepted.
        """
```
**EN:** This method validates assumptions or constraints within `StructuredOutputGrammar`. The docstring frames it as: Validates the provided tokens against the grammar.
**CN:** 该方法会校验前提与约束，其作用域位于`StructuredOutputGrammar`。

### `StructuredOutputGrammar.rollback` method / `StructuredOutputGrammar.rollback` 方法
```python
    @abstractmethod
    def rollback(self, num_tokens: int) -> None:
        """
        Rolls back the state of the grammar by a specified number of tokens.
        Will also revert counters for the number of processed tokens.

        Args:
            num_tokens (int): The number of tokens to roll back.
        """
```
**EN:** This method implements `rollback` within `StructuredOutputGrammar`. The docstring frames it as: Rolls back the state of the grammar by a specified number of tokens.
**CN:** 该方法会实现 `rollback`，其作用域位于`StructuredOutputGrammar`。

### `StructuredOutputGrammar.fill_bitmask` method / `StructuredOutputGrammar.fill_bitmask` 方法
```python
    @abstractmethod
    def fill_bitmask(self, bitmask: "torch.Tensor", batch_index: int) -> None:
        """
        Fills the bitmask for a specific batch index.

        Args:
            bitmask (torch.Tensor): The bitmask to fill
            batch_index (int): The index in the bitmask to fill
        """
```
**EN:** This method implements `fill_bitmask` within `StructuredOutputGrammar`. The docstring frames it as: Fills the bitmask for a specific batch index.
**CN:** 该方法会实现 `fill_bitmask`，其作用域位于`StructuredOutputGrammar`。

### `StructuredOutputGrammar.is_terminated` method / `StructuredOutputGrammar.is_terminated` 方法
```python
    @abstractmethod
    def is_terminated(self) -> bool:
        """
        Checks whether the structured output process has terminated.

        Returns:
            bool: True if the process is terminated, False otherwise.
        """
```
**EN:** This method answers a boolean capability check within `StructuredOutputGrammar`. The docstring frames it as: Checks whether the structured output process has terminated.
**CN:** 该方法会回答布尔能力判断，其作用域位于`StructuredOutputGrammar`。

### `StructuredOutputGrammar.reset` method / `StructuredOutputGrammar.reset` 方法
```python
    @abstractmethod
    def reset(self):
        """
        Resets the state of the structured output grammar.
        """
```
**EN:** This method implements `reset` within `StructuredOutputGrammar`. The docstring frames it as: Resets the state of the structured output grammar.
**CN:** 该方法会实现 `reset`，其作用域位于`StructuredOutputGrammar`。

### `StructuredOutputBackend` class / `StructuredOutputBackend` 类
```python
@dataclass
class StructuredOutputBackend(ABC):
    """Engine-level backend for structured output requests."""

    vllm_config: VllmConfig
    tokenizer: TokenizerLike
    vocab_size: int
```
**EN:** Declares the `StructuredOutputBackend` interface. Downstream implementations are expected to provide methods such as `compile_grammar`, `allocate_token_bitmask`, `destroy`.
**CN:** `StructuredOutputBackend` 声明了一组接口约定。下游实现需要提供 `compile_grammar`, `allocate_token_bitmask`, `destroy` 等方法。

### `StructuredOutputBackend.compile_grammar` method / `StructuredOutputBackend.compile_grammar` 方法
```python
    @abstractmethod
    def compile_grammar(
        self, request_type: StructuredOutputOptions, grammar_spec: str
    ) -> StructuredOutputGrammar:
        """
        Compiles a grammar specification into a structured output grammar.

        Args:
            request_type (StructuredOutputOptions): The type of structured
                output request.
            grammar_spec (str): The grammar specification to compile.

        Returns:
            StructuredOutputGrammar: The compiled structured output grammar.
        """
```
**EN:** This method implements `compile_grammar` within `StructuredOutputBackend`. The docstring frames it as: Compiles a grammar specification into a structured output grammar.
**CN:** 该方法会实现 `compile_grammar`，其作用域位于`StructuredOutputBackend`。

### `StructuredOutputBackend.allocate_token_bitmask` method / `StructuredOutputBackend.allocate_token_bitmask` 方法
```python
    @abstractmethod
    def allocate_token_bitmask(self, max_num_seqs: int) -> "torch.Tensor":
        """
        Allocates a token bitmask for the specified maximum number of sequences.

        Args:
            max_num_seqs (int): The maximum number of sequences for which
                to allocate the bitmask.
        """
```
**EN:** This method implements `allocate_token_bitmask` within `StructuredOutputBackend`. The docstring frames it as: Allocates a token bitmask for the specified maximum number of sequences.
**CN:** 该方法会实现 `allocate_token_bitmask`，其作用域位于`StructuredOutputBackend`。

### `StructuredOutputBackend.destroy` method / `StructuredOutputBackend.destroy` 方法
```python
    @abstractmethod
    def destroy(self):
        """
        Backend-specific cleanup.
        """
```
**EN:** This method implements `destroy` within `StructuredOutputBackend`. The docstring frames it as: Backend-specific cleanup.
**CN:** 该方法会实现 `destroy`，其作用域位于`StructuredOutputBackend`。

## Key Concepts / 关键概念
- `StructuredOutputOptions`: central class or interface in this module. / `StructuredOutputOptions`：本模块中的核心类或接口。
- `StructuredOutputGrammar`: central class or interface in this module. / `StructuredOutputGrammar`：本模块中的核心类或接口。
- `StructuredOutputBackend`: central class or interface in this module. / `StructuredOutputBackend`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `enum`, `abc`, `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.tokenizers`
