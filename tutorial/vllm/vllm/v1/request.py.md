# request.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/request.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `StreamingUpdate`, `Request`, `RequestStatus` for the V1 `v1` subsystem. / 为 V1 的 `v1` 子系统实现 `StreamingUpdate`, `Request`, `RequestStatus`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import enum
import time
from collections import deque
from collections.abc import Callable, Mapping
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any

import torch

from vllm.multimodal.inputs import MultiModalFeatureSpec
from vllm.pooling_params import PoolingParams
from vllm.sampling_params import SamplingParams
from vllm.utils import length_from_prompt_token_ids_or_embeds
from vllm.v1.engine import (
    EngineCoreEvent,
    EngineCoreEventType,
    EngineCoreRequest,
    FinishReason,
)
from vllm.v1.metrics.stats import PrefillStats
from vllm.v1.structured_output.request import StructuredOutputRequest
from vllm.v1.utils import ConstantList

if TYPE_CHECKING:
    from vllm.lora.request import LoRARequest
    from vllm.v1.core.kv_cache_utils import BlockHash
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.multimodal.inputs`, `vllm.pooling_params`, `vllm.sampling_params`, `vllm.utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.multimodal.inputs`, `vllm.pooling_params`, `vllm.sampling_params`, `vllm.utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `StreamingUpdate` class / `StreamingUpdate` 类
```python
@dataclass
class StreamingUpdate:
    """Lightweight data for streaming session continuation.

    Contains only the fields needed to update an existing streaming session
    with new input data.
    """

    mm_features: list[MultiModalFeatureSpec] | None
    prompt_token_ids: list[int] | None
    max_tokens: int
    arrival_time: float
    sampling_params: SamplingParams | None
```
**EN:** Uses `@dataclass` to package related state for `StreamingUpdate`. Typical fields include `mm_features`, `prompt_token_ids`, `max_tokens`, `arrival_time`, `sampling_params`.
**CN:** `StreamingUpdate` 使用 `@dataclass` 打包相关状态。典型字段包括 `mm_features`, `prompt_token_ids`, `max_tokens`, `arrival_time`, `sampling_params`。

### `StreamingUpdate.from_request` method / `StreamingUpdate.from_request` 方法
```python
    @classmethod
    def from_request(cls, request: "Request") -> "StreamingUpdate | None":
        if not request.resumable:
            return None
        return cls(
            mm_features=request.mm_features,
            prompt_token_ids=request.prompt_token_ids,
            max_tokens=request.max_tokens,
            arrival_time=request.arrival_time,
            sampling_params=request.sampling_params,
        )
```
**EN:** This method reconstructs data from another representation within `StreamingUpdate`. Key calls include `cls`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`StreamingUpdate`。 关键调用包括 `cls`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Request` class / `Request` 类
```python
class Request:
```
**EN:** Introduces the `Request` class. Core methods include `__init__`, `from_engine_core_request`, `append_output_token_ids`, `update_block_hashes`, `use_structured_output`, `num_tokens`.
**CN:** 这里定义 `Request` 类。核心方法包括 `__init__`, `from_engine_core_request`, `append_output_token_ids`, `update_block_hashes`, `use_structured_output`, `num_tokens`。

### `Request.__init__` method / `Request.__init__` 方法
```python
    def __init__(
        self,
        request_id: str,
        prompt_token_ids: list[int] | None,
        sampling_params: SamplingParams | None,
        pooling_params: PoolingParams | None,
        client_index: int = 0,
        arrival_time: float | None = None,
        prompt_embeds: torch.Tensor | None = None,
        prompt_is_token_ids: list[bool] | None = None,
        mm_features: list[MultiModalFeatureSpec] | None = None,
        lora_request: "LoRARequest | None" = None,
        cache_salt: str | None = None,
        priority: int = 0,
        trace_headers: Mapping[str, str] | None = None,
        block_hasher: Callable[["Request"], list["BlockHash"]] | None = None,
        resumable: bool = False,
        reasoning_ended: bool | None = None,
        reasoning_parser_kwargs: dict[str, Any] | None = None,
        abort_immediately: bool = False,
    ) -> None:
        self.request_id = request_id
        self.client_index = client_index
        self.priority = priority
        self.sampling_params = sampling_params
        self.pooling_params = pooling_params
        self.lora_request = lora_request
        self.structured_output_request = StructuredOutputRequest.from_sampling_params(
            sampling_params
        )
        if self.structured_output_request is not None:
            self.structured_output_request.reasoning_ended = reasoning_ended
            self.structured_output_request.reasoning_parser_kwargs = (
                reasoning_parser_kwargs
            )
        self.arrival_time = arrival_time if arrival_time is not None else time.time()

        self.status = RequestStatus.WAITING
        self.events: list[EngineCoreEvent] = []
        self.stop_reason: int | str | None = None

        # P/D: Connector-specific KV transfer parameters.
        self.kv_transfer_params: dict[str, Any] | None = None

        if pooling_params is not None:
            # Pooling models.
            self.max_tokens = 1
        elif sampling_params is not None:
            # Generative models.
            assert sampling_params.max_tokens is not None
    # ... omitted for brevity ...
        # indicates that the output is corrupted
        self.num_nans_in_logits = 0

        # The number of times this request has been preempted by the scheduler.
        self.num_preemptions = 0

        self.prefill_stats: PrefillStats | None = PrefillStats()

        self.block_hashes: list[BlockHash] = []
        # Store the block hasher without binding self to avoid creating a
        # reference cycle (Request -> partial -> Request) that prevents
        # immediate garbage collection via reference counting.
        self._block_hasher: Callable[[Request], list[BlockHash]] | None = block_hasher
        self.update_block_hashes()

        self.skip_reading_prefix_cache = self.get_skip_reading_prefix_cache()

        # Used for streaming
        self.resumable = resumable
        # None entry in the queue means finished.
        self.streaming_queue: deque[StreamingUpdate | None] | None = None

        # If True, request should be aborted immediately after being added to
        # the scheduler so the connector's request_finished hook runs.
        self.abort_immediately = abort_immediately
```
**EN:** This method initializes the object state within `Request`. Key calls include `from_sampling_params`, `length_from_prompt_token_ids_or_embeds`, `ConstantList`, `PrefillStats`, `update_block_hashes`, `get_skip_reading_prefix_cache`. It touches state such as `request_id`, `client_index`, `priority`, `sampling_params`, `pooling_params`, `lora_request`, `structured_output_request`, `arrival_time`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`Request`。 关键调用包括 `from_sampling_params`, `length_from_prompt_token_ids_or_embeds`, `ConstantList`, `PrefillStats`, `update_block_hashes`, `get_skip_reading_prefix_cache`。 它会读写 `request_id`, `client_index`, `priority`, `sampling_params`, `pooling_params`, `lora_request`, `structured_output_request`, `arrival_time` 等状态。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `Request.from_engine_core_request` method / `Request.from_engine_core_request` 方法
```python
    @classmethod
    def from_engine_core_request(
        cls,
        request: EngineCoreRequest,
        block_hasher: Callable[["Request"], list["BlockHash"]] | None,
    ) -> "Request":
        return cls(
            request_id=request.request_id,
            client_index=request.client_index,
            prompt_token_ids=request.prompt_token_ids,
            prompt_embeds=request.prompt_embeds,
            prompt_is_token_ids=request.prompt_is_token_ids,
            mm_features=request.mm_features,
            sampling_params=request.sampling_params,
            pooling_params=request.pooling_params,
            arrival_time=request.arrival_time,
            lora_request=request.lora_request,
            cache_salt=request.cache_salt,
            priority=request.priority,
            trace_headers=request.trace_headers,
            block_hasher=block_hasher,
            resumable=request.resumable,
            reasoning_ended=request.reasoning_ended,
            reasoning_parser_kwargs=request.reasoning_parser_kwargs,
            abort_immediately=request.abort_immediately,
        )
```
**EN:** This method reconstructs data from another representation within `Request`. Key calls include `cls`.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`Request`。 关键调用包括 `cls`。

### `Request.append_output_token_ids` method / `Request.append_output_token_ids` 方法
```python
    def append_output_token_ids(
        self,
        token_ids: int | list[int],
    ) -> None:
        if isinstance(token_ids, int):
            self._output_token_ids.append(token_ids)
            self._all_token_ids.append(token_ids)
        else:
            self._output_token_ids.extend(token_ids)
            self._all_token_ids.extend(token_ids)

        self.update_block_hashes()
```
**EN:** This method implements `append_output_token_ids` within `Request`. Key calls include `isinstance`, `update_block_hashes`, `append`, `extend`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `append_output_token_ids`，其作用域位于`Request`。 关键调用包括 `isinstance`, `update_block_hashes`, `append`, `extend`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Request.update_block_hashes` method / `Request.update_block_hashes` 方法
```python
    def update_block_hashes(self) -> None:
        """Compute block hashes for any new full blocks and append them."""
        if self._block_hasher is not None:
            self.block_hashes.extend(self._block_hasher(self))
```
**EN:** This method updates existing state within `Request`. The docstring frames it as: Compute block hashes for any new full blocks and append them. Key calls include `extend`, `_block_hasher`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`Request`。 关键调用包括 `extend`, `_block_hasher`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Request.get_skip_reading_prefix_cache` method / `Request.get_skip_reading_prefix_cache` 方法
```python
    def get_skip_reading_prefix_cache(self) -> bool:
        if (
            self.sampling_params is not None
            and self.sampling_params.skip_reading_prefix_cache is not None
        ):
            return self.sampling_params.skip_reading_prefix_cache
        elif (
            self.pooling_params is not None
            and self.pooling_params.skip_reading_prefix_cache is not None
        ):
            return self.pooling_params.skip_reading_prefix_cache
        return False
```
**EN:** This method returns or derives a value within `Request`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`Request`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Request.record_event` method / `Request.record_event` 方法
```python
    def record_event(
        self,
        event_type: EngineCoreEventType,
        timestamp: float | None = None,
    ) -> None:
        self.events.append(EngineCoreEvent.new_event(event_type, timestamp))
```
**EN:** This method implements `record_event` within `Request`. Key calls include `append`, `new_event`.
**CN:** 该方法会实现 `record_event`，其作用域位于`Request`。 关键调用包括 `append`, `new_event`。

### `Request.take_events` method / `Request.take_events` 方法
```python
    def take_events(self) -> list[EngineCoreEvent] | None:
        if not self.events:
            return None
        events, self.events = self.events, []
        return events
```
**EN:** This method implements `take_events` within `Request`. It touches state such as `events`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `take_events`，其作用域位于`Request`。 它会读写 `events` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Request.take_prefill_stats` method / `Request.take_prefill_stats` 方法
```python
    def take_prefill_stats(self) -> PrefillStats | None:
        if self.prefill_stats is None:
            return None
        prefill_stats = self.prefill_stats
        self.prefill_stats = None
        return prefill_stats
```
**EN:** This method implements `take_prefill_stats` within `Request`. It touches state such as `prefill_stats`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `take_prefill_stats`，其作用域位于`Request`。 它会读写 `prefill_stats` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RequestStatus` class / `RequestStatus` 类
```python
class RequestStatus(enum.IntEnum):
    """Status of a request."""

    WAITING = enum.auto()
    WAITING_FOR_STRUCTURED_OUTPUT_GRAMMAR = enum.auto()
    WAITING_FOR_REMOTE_KVS = enum.auto()
    WAITING_FOR_STREAMING_REQ = enum.auto()
    RUNNING = enum.auto()
    PREEMPTED = enum.auto()
    # Note: anything after PREEMPTED will be considered
    # as a finished status.
    FINISHED_STOPPED = enum.auto()
    FINISHED_LENGTH_CAPPED = enum.auto()
    FINISHED_ABORTED = enum.auto()
    FINISHED_IGNORED = enum.auto()
    FINISHED_ERROR = enum.auto()
    FINISHED_REPETITION = enum.auto()
```
**EN:** Defines the `RequestStatus` enum used to normalize modes or options across the subsystem. Representative members: `WAITING`, `WAITING_FOR_STRUCTURED_OUTPUT_GRAMMAR`, `WAITING_FOR_REMOTE_KVS`, `WAITING_FOR_STREAMING_REQ`, `RUNNING`, `PREEMPTED`.
**CN:** `RequestStatus` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`WAITING`, `WAITING_FOR_STRUCTURED_OUTPUT_GRAMMAR`, `WAITING_FOR_REMOTE_KVS`, `WAITING_FOR_STREAMING_REQ`, `RUNNING`, `PREEMPTED`。

### `RequestStatus.__str__` method / `RequestStatus.__str__` 方法
```python
    def __str__(self) -> str:
        return self.name
```
**EN:** This method implements `__str__` within `RequestStatus`.
**CN:** 该方法会实现 `__str__`，其作用域位于`RequestStatus`。

### `RequestStatus.is_finished` method / `RequestStatus.is_finished` 方法
```python
    @staticmethod
    def is_finished(status: "RequestStatus") -> bool:
        return status > RequestStatus.PREEMPTED
```
**EN:** This method answers a boolean capability check within `RequestStatus`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`RequestStatus`。

### `RequestStatus.get_finished_reason` method / `RequestStatus.get_finished_reason` 方法
```python
    @staticmethod
    def get_finished_reason(status: "RequestStatus") -> FinishReason | None:
        return _FINISHED_REASON_MAP.get(status)
```
**EN:** This method returns or derives a value within `RequestStatus`. Key calls include `get`.
**CN:** 该方法会返回或推导一个值，其作用域位于`RequestStatus`。 关键调用包括 `get`。

### Module constants / 模块常量
```python
_FINISHED_REASON_MAP = {
    RequestStatus.FINISHED_STOPPED: FinishReason.STOP,
    RequestStatus.FINISHED_LENGTH_CAPPED: FinishReason.LENGTH,
    RequestStatus.FINISHED_ABORTED: FinishReason.ABORT,
    RequestStatus.FINISHED_IGNORED: FinishReason.LENGTH,
    RequestStatus.FINISHED_ERROR: FinishReason.ERROR,
    RequestStatus.WAITING_FOR_STREAMING_REQ: FinishReason.STOP,
    RequestStatus.FINISHED_REPETITION: FinishReason.REPETITION,
}
```
**EN:** Defines module-level constants or aliases such as `_FINISHED_REASON_MAP`, which are reused by later definitions.
**CN:** 定义 `_FINISHED_REASON_MAP` 等模块级常量或别名，供后续定义复用。

## Key Concepts / 关键概念
- `StreamingUpdate`: central class or interface in this module. / `StreamingUpdate`：本模块中的核心类或接口。
- `Request`: central class or interface in this module. / `Request`：本模块中的核心类或接口。
- `RequestStatus`: central class or interface in this module. / `RequestStatus`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `enum`, `time`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.multimodal.inputs`, `vllm.pooling_params`, `vllm.sampling_params`, `vllm.utils`, `vllm.v1.engine`, `vllm.v1.metrics.stats`, `vllm.v1.structured_output.request`, `vllm.v1.utils`, `vllm.lora.request`, `vllm.v1.core.kv_cache_utils`
