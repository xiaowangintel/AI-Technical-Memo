# interface.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/sched/interface.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `PauseState`, `SchedulerInterface` for the V1 `core/sched` subsystem. / 为 V1 的 `core/sched` 子系统实现 `PauseState`, `SchedulerInterface`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import enum
from abc import ABC, abstractmethod
from collections.abc import Iterable
from typing import TYPE_CHECKING

from vllm.multimodal import MULTIMODAL_REGISTRY, MultiModalRegistry

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.distributed.kv_transfer.kv_connector.v1 import KVConnectorBase_V1
    from vllm.v1.core.sched.output import GrammarOutput, SchedulerOutput
    from vllm.v1.engine import EngineCoreOutputs
    from vllm.v1.kv_cache_interface import KVCacheConfig
    from vllm.v1.metrics.stats import SchedulerStats
    from vllm.v1.outputs import DraftTokenIds, ModelRunnerOutput
    from vllm.v1.request import Request, RequestStatus
    from vllm.v1.structured_output import StructuredOutputManager
```
**EN:** Sets up the module namespace with standard-library helpers, and internal vLLM modules such as `vllm.multimodal`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1`, `vllm.v1.core.sched.output`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具，以及 `vllm.multimodal`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1`, `vllm.v1.core.sched.output` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `PauseState` class / `PauseState` 类
```python
class PauseState(enum.IntEnum):
    """Scheduler pause state.

    - UNPAUSED: Normal operation
    - PAUSE_NEW: No new requests are scheduled, requests already in
                 running state are scheduled.
    - PAUSE_ALL: No requests are scheduled
    """

    UNPAUSED = 0
    PAUSED_NEW = 1
    PAUSED_ALL = 2
```
**EN:** Defines the `PauseState` enum used to normalize modes or options across the subsystem. Representative members: `UNPAUSED`, `PAUSED_NEW`, `PAUSED_ALL`.
**CN:** `PauseState` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`UNPAUSED`, `PAUSED_NEW`, `PAUSED_ALL`。

### `SchedulerInterface` class / `SchedulerInterface` 类
```python
class SchedulerInterface(ABC):
```
**EN:** Declares the `SchedulerInterface` interface. Downstream implementations are expected to provide methods such as `__init__`, `schedule`, `get_grammar_bitmask`, `update_from_output`, `update_draft_token_ids`, `update_draft_token_ids_in_output`.
**CN:** `SchedulerInterface` 声明了一组接口约定。下游实现需要提供 `__init__`, `schedule`, `get_grammar_bitmask`, `update_from_output`, `update_draft_token_ids`, `update_draft_token_ids_in_output` 等方法。

### `SchedulerInterface.__init__` method / `SchedulerInterface.__init__` 方法
```python
    @abstractmethod
    def __init__(
        self,
        vllm_config: "VllmConfig",
        kv_cache_config: "KVCacheConfig",
        structured_output_manager: "StructuredOutputManager",
        block_size: int,
        hash_block_size: int,
        mm_registry: MultiModalRegistry = MULTIMODAL_REGISTRY,
        include_finished_set: bool = False,
        log_stats: bool = False,
    ) -> None:
        raise NotImplementedError
```
**EN:** This method initializes the object state within `SchedulerInterface`.
**CN:** 该方法会初始化对象状态，其作用域位于`SchedulerInterface`。

### `SchedulerInterface.schedule` method / `SchedulerInterface.schedule` 方法
```python
    @abstractmethod
    def schedule(self) -> "SchedulerOutput":
        """Schedule the requests to process in this scheduling step.

        The scheduling decision is made at the iteration level. Each scheduling
        step corresponds to a single forward pass of the model. Therefore, this
        method is called repeatedly by a busy loop in the engine.

        Essentially, the scheduler produces a dictionary of {req_id: num_tokens}
        that specifies how many tokens to process for each request in this
        scheduling step. For example, num_tokens can be as large as the number
        of prompt tokens for new requests, or it can be 1 for the requests that
        are auto-regressively generating new tokens one by one. Otherwise, it
        can be somewhere in between in case of chunked prefills, prefix caching,
        speculative decoding, etc.

        Additionally, the scheduler also returns useful data about each request
        or the batch as a whole. The model runner will use this information in
        preparing inputs to the model.

        Returns:
            A SchedulerOutput object containing information about the scheduled
            requests.
        """
        raise NotImplementedError
```
**EN:** This method coordinates scheduling decisions within `SchedulerInterface`. The docstring frames it as: Schedule the requests to process in this scheduling step.
**CN:** 该方法会协调调度决策，其作用域位于`SchedulerInterface`。

### `SchedulerInterface.update_from_output` method / `SchedulerInterface.update_from_output` 方法
```python
    @abstractmethod
    def update_from_output(
        self,
        scheduler_output: "SchedulerOutput",
        model_runner_output: "ModelRunnerOutput",
    ) -> dict[int, "EngineCoreOutputs"]:
        """Update the scheduler state based on the model runner output.

        This method is called after the model runner has processed the scheduled
        requests. The model runner output includes generated token ids, draft
        token ids for next step, etc. The scheduler uses this information to
        update its states, checks the finished requests, and returns the output
        for each request.

        Returns:
            A dict of client index to EngineCoreOutputs object containing the
            outputs for each request originating from that client.
        """
        raise NotImplementedError
```
**EN:** This method updates existing state within `SchedulerInterface`. The docstring frames it as: Update the scheduler state based on the model runner output.
**CN:** 该方法会更新现有状态，其作用域位于`SchedulerInterface`。

### `SchedulerInterface.update_draft_token_ids` method / `SchedulerInterface.update_draft_token_ids` 方法
```python
    @abstractmethod
    def update_draft_token_ids(self, draft_token_ids: "DraftTokenIds") -> None:
        """Update requests with newly generated draft token ids, applying
        structured output grammar validation if needed.

        Args:
            draft_token_ids: The input draft token ids for each request.
        """
        raise NotImplementedError
```
**EN:** This method updates existing state within `SchedulerInterface`. The docstring frames it as: Update requests with newly generated draft token ids, applying structured output grammar validation if needed.
**CN:** 该方法会更新现有状态，其作用域位于`SchedulerInterface`。

### `SchedulerInterface.update_draft_token_ids_in_output` method / `SchedulerInterface.update_draft_token_ids_in_output` 方法
```python
    @abstractmethod
    def update_draft_token_ids_in_output(
        self, draft_token_ids: "DraftTokenIds", scheduler_output: "SchedulerOutput"
    ) -> None:
        """Update scheduler output with newly generated draft token ids, applying
        structured output grammar validation if needed.

        Args:
            draft_token_ids: The input draft token ids for each request.
            scheduler_output: Update the given scheduler_output
                with the corresponding draft token ids.
        """
        raise NotImplementedError
```
**EN:** This method updates existing state within `SchedulerInterface`. The docstring frames it as: Update scheduler output with newly generated draft token ids, applying structured output grammar validation if needed.
**CN:** 该方法会更新现有状态，其作用域位于`SchedulerInterface`。

### `SchedulerInterface.finish_requests` method / `SchedulerInterface.finish_requests` 方法
```python
    @abstractmethod
    def finish_requests(
        self,
        request_ids: str | Iterable[str] | None,
        finished_status: "RequestStatus",
    ) -> list[tuple[str, int]]:
        """Finish the requests in the scheduler's internal queue. If the request
        is not in the queue, this method will do nothing for that request.

        This method is called in two cases:
        1. When the request is aborted by the client.
        2. When the frontend process detects a stop string of the request after
           de-tokenizing its generated tokens.

        Args:
            request_ids: A single or a list of request IDs, or None to finish all.
            finished_status: The finished status of the given requests.

        Returns:
            Tuple of (req_id, client_index) for requests that were aborted. Will not
            include any that were already finished.
        """
        raise NotImplementedError
```
**EN:** This method implements `finish_requests` within `SchedulerInterface`. The docstring frames it as: Finish the requests in the scheduler's internal queue.
**CN:** 该方法会实现 `finish_requests`，其作用域位于`SchedulerInterface`。

### `SchedulerInterface.has_finished_requests` method / `SchedulerInterface.has_finished_requests` 方法
```python
    @abstractmethod
    def has_finished_requests(self) -> bool:
        """Returns True if there are finished requests that need to be cleared.
        NOTE: This is different from `not self.has_unfinished_requests()`.

        The scheduler maintains an internal list of the requests finished in the
        previous step. This list is returned from the next call to schedule(),
        to be sent to the model runner in the next step to clear cached states
        for these finished requests.

        This method checks if this internal list of finished requests is
        non-empty. This information is useful for DP attention.
        """
        raise NotImplementedError
```
**EN:** This method implements `has_finished_requests` within `SchedulerInterface`. The docstring frames it as: Returns True if there are finished requests that need to be cleared.
**CN:** 该方法会实现 `has_finished_requests`，其作用域位于`SchedulerInterface`。

### `SchedulerInterface.reset_prefix_cache` method / `SchedulerInterface.reset_prefix_cache` 方法
```python
    @abstractmethod
    def reset_prefix_cache(
        self, reset_running_requests: bool = False, reset_connector: bool = False
    ) -> bool:
        """Reset the prefix cache for KV cache.

        This is particularly required when the model weights are live-updated.

        Args:
            reset_running_requests: If True, all the running requests will be
                preempted and moved to the waiting queue. Otherwise, this method
                will only reset the KV prefix cache when there is no running request
                taking KV cache.
        """
        raise NotImplementedError
```
**EN:** This method implements `reset_prefix_cache` within `SchedulerInterface`. The docstring frames it as: Reset the prefix cache for KV cache.
**CN:** 该方法会实现 `reset_prefix_cache`，其作用域位于`SchedulerInterface`。

## Key Concepts / 关键概念
- `PauseState`: central class or interface in this module. / `PauseState`：本模块中的核心类或接口。
- `SchedulerInterface`: central class or interface in this module. / `SchedulerInterface`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `enum`, `abc`, `collections`, `typing`
- Internal vLLM / 内部依赖: `vllm.multimodal`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1`, `vllm.v1.core.sched.output`, `vllm.v1.engine`, `vllm.v1.kv_cache_interface`, `vllm.v1.metrics.stats`, `vllm.v1.outputs`, `vllm.v1.request`, `vllm.v1.structured_output`
