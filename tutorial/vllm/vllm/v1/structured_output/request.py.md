# request.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/structured_output/request.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `StructuredOutputRequest`, `get_structured_output_key` for the V1 `structured_output` subsystem. / 为 V1 的 `structured_output` 子系统实现 `StructuredOutputRequest`, `get_structured_output_key`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import dataclasses
import functools
import json
from concurrent.futures import Future
from concurrent.futures._base import TimeoutError
from typing import TYPE_CHECKING, Any, cast

from vllm.sampling_params import SamplingParams, StructuredOutputsParams
from vllm.v1.structured_output.backend_types import (
    StructuredOutputGrammar,
    StructuredOutputKey,
    StructuredOutputOptions,
)

if TYPE_CHECKING:
    from vllm.reasoning import ReasoningParser
```
**EN:** Sets up the module namespace with standard-library helpers, and internal vLLM modules such as `vllm.sampling_params`, `vllm.v1.structured_output.backend_types`, `vllm.reasoning`, `vllm.v1.request`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具，以及 `vllm.sampling_params`, `vllm.v1.structured_output.backend_types`, `vllm.reasoning`, `vllm.v1.request` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `StructuredOutputRequest` class / `StructuredOutputRequest` 类
```python
@dataclasses.dataclass
class StructuredOutputRequest:
    params: StructuredOutputsParams
    _grammar: Future[StructuredOutputGrammar] | StructuredOutputGrammar | None = None
    reasoning_ended: bool | None = None
    reasoning_parser_kwargs: dict[str, Any] | None = None
    # Cached per request; do not share reasoning parsers across requests because
    # their behavior can depend on reasoning_parser_kwargs.
    reasoner: "ReasoningParser | None" = None
```
**EN:** Introduces the `StructuredOutputRequest` class. Core methods include `from_sampling_params`, `_check_grammar_completion`, `is_grammar_ready`, `grammar`, `grammar`, `structured_output_key`.
**CN:** 这里定义 `StructuredOutputRequest` 类。核心方法包括 `from_sampling_params`, `_check_grammar_completion`, `is_grammar_ready`, `grammar`, `grammar`, `structured_output_key`。

### `StructuredOutputRequest.from_sampling_params` method / `StructuredOutputRequest.from_sampling_params` 方法
```python
    @staticmethod
    def from_sampling_params(
        sampling_params: SamplingParams | None,
    ) -> "StructuredOutputRequest | None":
        if sampling_params is None:
            return None
        params = sampling_params.structured_outputs
        if not params or params.all_constraints_none():
            return None
        return StructuredOutputRequest(params=params)
```
**EN:** This method reconstructs data from another representation within `StructuredOutputRequest`. Key calls include `StructuredOutputRequest`, `all_constraints_none`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`StructuredOutputRequest`。 关键调用包括 `StructuredOutputRequest`, `all_constraints_none`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `StructuredOutputRequest._check_grammar_completion` method / `StructuredOutputRequest._check_grammar_completion` 方法
```python
    def _check_grammar_completion(self) -> bool:
        # NOTE: We have to lazy import to gate circular imports
        from vllm.v1.request import RequestStatus

        if isinstance(self._grammar, Future):
            try:
                # We will check whether the future is ready within 100 us
                self._grammar = self._grammar.result(timeout=0.0001)
                self.status = RequestStatus.WAITING
            except TimeoutError:
                return False
        return True
```
**EN:** This method implements `_check_grammar_completion` within `StructuredOutputRequest`. Key calls include `isinstance`, `result`. It touches state such as `_grammar`, `status`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_check_grammar_completion`，其作用域位于`StructuredOutputRequest`。 关键调用包括 `isinstance`, `result`。 它会读写 `_grammar`, `status` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `StructuredOutputRequest.is_grammar_ready` method / `StructuredOutputRequest.is_grammar_ready` 方法
```python
    @property
    def is_grammar_ready(self) -> bool:
        return self._check_grammar_completion()
```
**EN:** This method answers a boolean capability check within `StructuredOutputRequest`. Key calls include `_check_grammar_completion`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`StructuredOutputRequest`。 关键调用包括 `_check_grammar_completion`。

### `StructuredOutputRequest.grammar` method / `StructuredOutputRequest.grammar` 方法
```python
    @property
    def grammar(self) -> StructuredOutputGrammar | None:
        completed = self._check_grammar_completion()
        return (
            cast(StructuredOutputGrammar | None, self._grammar) if completed else None
        )
```
**EN:** This method implements `grammar` within `StructuredOutputRequest`. Key calls include `_check_grammar_completion`, `cast`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `grammar`，其作用域位于`StructuredOutputRequest`。 关键调用包括 `_check_grammar_completion`, `cast`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `StructuredOutputRequest.grammar` method / `StructuredOutputRequest.grammar` 方法
```python
    @grammar.setter
    def grammar(
        self, grammar: StructuredOutputGrammar | Future[StructuredOutputGrammar]
    ) -> None:
        self._grammar = grammar
```
**EN:** This method implements `grammar` within `StructuredOutputRequest`. It touches state such as `_grammar`.
**CN:** 该方法会实现 `grammar`，其作用域位于`StructuredOutputRequest`。 它会读写 `_grammar` 等状态。

### `StructuredOutputRequest.structured_output_key` method / `StructuredOutputRequest.structured_output_key` 方法
```python
    @functools.cached_property
    def structured_output_key(self) -> StructuredOutputKey:
        return get_structured_output_key(self.params)
```
**EN:** This method implements `structured_output_key` within `StructuredOutputRequest`. Key calls include `get_structured_output_key`.
**CN:** 该方法会实现 `structured_output_key`，其作用域位于`StructuredOutputRequest`。 关键调用包括 `get_structured_output_key`。

### `get_structured_output_key` function / `get_structured_output_key` 函数
```python
def get_structured_output_key(params: StructuredOutputsParams) -> StructuredOutputKey:
    if params.json is not None:
        if not isinstance(params.json, str):
            json_str = json.dumps(params.json)
        else:
            json_str = params.json
        return StructuredOutputOptions.JSON, json_str
    if params.json_object:
        return StructuredOutputOptions.JSON_OBJECT, ""
    if params.regex is not None:
        return StructuredOutputOptions.REGEX, params.regex
    if params.choice is not None:
        if not isinstance(params.choice, str):
            json_str = json.dumps(params.choice)
        else:
            json_str = params.choice
        return StructuredOutputOptions.CHOICE, json_str
    if params.grammar is not None:
        return StructuredOutputOptions.GRAMMAR, params.grammar
    if params.structural_tag is not None:
        return StructuredOutputOptions.STRUCTURAL_TAG, params.structural_tag
    raise ValueError("No valid structured output parameter found")
```
**EN:** This function returns or derives a value within the module. Key calls include `ValueError`, `isinstance`, `dumps`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `ValueError`, `isinstance`, `dumps`。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `StructuredOutputRequest`: central class or interface in this module. / `StructuredOutputRequest`：本模块中的核心类或接口。
- `get_structured_output_key`: top-level helper or orchestration entry point. / `get_structured_output_key`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `functools`, `json`, `concurrent`, `typing`
- Internal vLLM / 内部依赖: `vllm.sampling_params`, `vllm.v1.structured_output.backend_types`, `vllm.reasoning`, `vllm.v1.request`
