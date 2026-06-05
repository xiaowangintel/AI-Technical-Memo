# parallel_sampling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/parallel_sampling.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `ParentRequest` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `ParentRequest`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from copy import copy
from typing import cast

from vllm.outputs import CompletionOutput
from vllm.sampling_params import RequestOutputKind, SamplingParams
from vllm.v1.engine import EngineCoreRequest
from vllm.v1.metrics.stats import IterationStats
```
**EN:** Sets up the module namespace with standard-library helpers, and internal vLLM modules such as `vllm.outputs`, `vllm.sampling_params`, `vllm.v1.engine`, `vllm.v1.metrics.stats`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具，以及 `vllm.outputs`, `vllm.sampling_params`, `vllm.v1.engine`, `vllm.v1.metrics.stats` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `ParentRequest` class / `ParentRequest` 类
```python
class ParentRequest:
    """Info, state & processing for parallel sampling request.

    Store parent request ID and sampling params.
    Facilitate generating child request sampling params.
    """

    request_id: str
    external_req_id: str
    sampling_params: SamplingParams

    # To track the completion of child requests
    child_requests: set[str]

    # To aggregate child completions when not streaming
    output_aggregator: list[CompletionOutput]

    # To find the max number of generated tokens across all children
    max_num_generation_tokens: int

    # To efficiently obtain child sampling params
    cached_child_sampling_params: SamplingParams | None
```
**EN:** Introduces the `ParentRequest` class. Core methods include `__init__`, `_get_child_sampling_params`, `get_child_info`, `n`, `get_outputs`, `observe_num_generation_tokens`. Docstring signal: Info, state & processing for parallel sampling request.
**CN:** 这里定义 `ParentRequest` 类。核心方法包括 `__init__`, `_get_child_sampling_params`, `get_child_info`, `n`, `get_outputs`, `observe_num_generation_tokens`。

### `ParentRequest.__init__` method / `ParentRequest.__init__` 方法
```python
    def __init__(self, request: EngineCoreRequest) -> None:
        assert request.external_req_id is not None
        sampling_params = request.params
        self.request_id = request.request_id
        self.external_req_id = request.external_req_id
        self.sampling_params = sampling_params

        self.child_requests = set()
        self.output_aggregator = (
            [cast(CompletionOutput, None)] * sampling_params.n
            if (sampling_params.output_kind == RequestOutputKind.FINAL_ONLY)
            else []
        )
        self.max_num_generation_tokens = 0
        self.cached_child_sampling_params = None
```
**EN:** This method initializes the object state within `ParentRequest`. Key calls include `set`, `cast`. It touches state such as `request_id`, `external_req_id`, `sampling_params`, `child_requests`, `output_aggregator`, `max_num_generation_tokens`, `cached_child_sampling_params`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`ParentRequest`。 关键调用包括 `set`, `cast`。 它会读写 `request_id`, `external_req_id`, `sampling_params`, `child_requests`, `output_aggregator`, `max_num_generation_tokens`, `cached_child_sampling_params` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ParentRequest._get_child_sampling_params` method / `ParentRequest._get_child_sampling_params` 方法
```python
    def _get_child_sampling_params(
        self,
        index: int,
    ) -> SamplingParams:
        """Efficiently obtain child `sampling_params`

        If `sampling_params.seed` is not `None` then
        each child request requires a unique clone of
        parent `sampling_params` with a unique seed.

        Args:
          index: index within `n` child requests

        Returns:
          Child `sampling_params` instance.
        """
        seed = self.sampling_params.seed
        if self.cached_child_sampling_params:
            # Reuse child sampling_params data structure
            return self.cached_child_sampling_params
        # Build child sampling_params
        child_sampling_params = copy(self.sampling_params)
        child_sampling_params.n = 1
        if seed is None:
            # Cache child sampling_params for later reuse
            self.cached_child_sampling_params = child_sampling_params
        else:
            # Each child gets a clone with a unique seed
            child_sampling_params.seed = seed + index
        return child_sampling_params
```
**EN:** This method implements `_get_child_sampling_params` within `ParentRequest`. The docstring frames it as: Efficiently obtain child `sampling_params` If `sampling_params.seed` is not `None` then each child request requires a unique clone of parent `sampling_params` with a unique seed. Key calls include `copy`. It touches state such as `cached_child_sampling_params`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_get_child_sampling_params`，其作用域位于`ParentRequest`。 关键调用包括 `copy`。 它会读写 `cached_child_sampling_params` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ParentRequest.get_child_info` method / `ParentRequest.get_child_info` 方法
```python
    def get_child_info(self, index: int) -> tuple[str, SamplingParams]:
        """Get child request ID and sampling params.

        Args:
          index: index within `n` child requests.

        Returns:
          (request ID, sampling_params) tuple
        """
        child_req_id = f"{index}_{self.request_id}"
        self.child_requests.add(child_req_id)
        return child_req_id, self._get_child_sampling_params(index)
```
**EN:** This method returns or derives a value within `ParentRequest`. The docstring frames it as: Get child request ID and sampling params. Key calls include `add`, `_get_child_sampling_params`.
**CN:** 该方法会返回或推导一个值，其作用域位于`ParentRequest`。 关键调用包括 `add`, `_get_child_sampling_params`。

### `ParentRequest.n` method / `ParentRequest.n` 方法
```python
    @property
    def n(self) -> int:
        return self.sampling_params.n
```
**EN:** This method implements `n` within `ParentRequest`.
**CN:** 该方法会实现 `n`，其作用域位于`ParentRequest`。

### `ParentRequest.get_outputs` method / `ParentRequest.get_outputs` 方法
```python
    def get_outputs(
        self,
        child_request_id: str,
        completion_output: CompletionOutput,
    ) -> tuple[list[CompletionOutput], bool]:
        already_finished_and_returned: bool = False
        if completion_output.finished():
            if child_request_id in self.child_requests:
                self.child_requests.remove(child_request_id)
            else:
                # child request ID is not available in child_requests
                # which means the request had finished in previous
                # batch step and returned to the client earlier
                already_finished_and_returned = True

        if self.sampling_params.output_kind != RequestOutputKind.FINAL_ONLY:
            # If streaming, just return the current output
            #
            # DO NOT output finished and already returned child request to client again
            outputs = [] if already_finished_and_returned else [completion_output]
        else:
            # If not streaming, aggregate the n final outputs.
            self.output_aggregator[completion_output.index] = completion_output
            outputs = [] if self.child_requests else self.output_aggregator

        finished = not self.child_requests
        return outputs, finished
```
**EN:** This method returns or derives a value within `ParentRequest`. Key calls include `finished`, `remove`. It touches state such as `output_aggregator`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`ParentRequest`。 关键调用包括 `finished`, `remove`。 它会读写 `output_aggregator` 等状态。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ParentRequest.observe_num_generation_tokens` method / `ParentRequest.observe_num_generation_tokens` 方法
```python
    def observe_num_generation_tokens(self, num_generation_tokens: int):
        self.max_num_generation_tokens = max(
            num_generation_tokens, self.max_num_generation_tokens
        )
        return self.max_num_generation_tokens
```
**EN:** This method implements `observe_num_generation_tokens` within `ParentRequest`. Key calls include `max`. It touches state such as `max_num_generation_tokens`.
**CN:** 该方法会实现 `observe_num_generation_tokens`，其作用域位于`ParentRequest`。 关键调用包括 `max`。 它会读写 `max_num_generation_tokens` 等状态。

### `ParentRequest.observe_finished_request` method / `ParentRequest.observe_finished_request` 方法
```python
    @staticmethod
    def observe_finished_request(
        parent_req: "ParentRequest | None",
        iteration_stats: IterationStats,
        num_generation_tokens: int,
    ):
        n_param = parent_req.n if parent_req is not None else 1

        if parent_req is not None:
            num_generation_tokens = parent_req.observe_num_generation_tokens(
                num_generation_tokens
            )

        # Child requests finished, we can now record to iteration stats
        if parent_req is None or not parent_req.child_requests:
            iteration_stats.max_num_generation_tokens_iter.append(num_generation_tokens)
            iteration_stats.n_params_iter.append(n_param)
```
**EN:** This method implements `observe_finished_request` within `ParentRequest`. Key calls include `observe_num_generation_tokens`, `append`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `observe_finished_request`，其作用域位于`ParentRequest`。 关键调用包括 `observe_num_generation_tokens`, `append`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `ParentRequest`: central class or interface in this module. / `ParentRequest`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `copy`, `typing`
- Internal vLLM / 内部依赖: `vllm.outputs`, `vllm.sampling_params`, `vllm.v1.engine`, `vllm.v1.metrics.stats`
