# output_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/output_processor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `RequestOutputCollector`, `OutputProcessorOutput`, `StreamingUpdate` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `RequestOutputCollector`, `OutputProcessorOutput`, `StreamingUpdate`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import asyncio
from collections import defaultdict, deque
from collections.abc import Iterable
from dataclasses import dataclass
from typing import Any, cast

import numpy as np
import torch

from vllm.lora.request import LoRARequest
from vllm.outputs import (
    STREAM_FINISHED,
    CompletionOutput,
    PoolingOutput,
    PoolingRequestOutput,
    RequestOutput,
)
from vllm.sampling_params import RequestOutputKind
from vllm.tokenizers import TokenizerLike
from vllm.tracing import (
    SpanAttributes,
    SpanKind,
    extract_trace_context,
    instrument_manual,
)
from vllm.utils import length_from_prompt_token_ids_or_embeds
from vllm.v1.engine import EngineCoreOutput, EngineCoreRequest, FinishReason
from vllm.v1.engine.detokenizer import IncrementalDetokenizer
from vllm.v1.engine.logprobs import LogprobsProcessor
from vllm.v1.engine.parallel_sampling import ParentRequest
from vllm.v1.metrics.stats import (
    IterationStats,
    LoRARequestStates,
    RequestStateStats,
    SchedulerStats,
)

# shared empty CPU tensor used as a placeholder pooling output
EMPTY_CPU_TENSOR = torch.empty(0, device="cpu")
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `EMPTY_CPU_TENSOR`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `EMPTY_CPU_TENSOR`。

### `RequestOutputCollector` class / `RequestOutputCollector` 类
```python
class RequestOutputCollector:
    """
    Collects streamed RequestOutputs per individual request,
    for hand-off to the consuming asyncio generate task.

    When streaming deltas, RequestOutputs are merged if the
    producer gets ahead of the consumer.
    """
```
**EN:** Introduces the `RequestOutputCollector` class. Core methods include `__init__`, `put`, `get`, `get_nowait`, `close`, `__del__`. Docstring signal: Collects streamed RequestOutputs per individual request, for hand-off to the consuming asyncio generate task.
**CN:** 这里定义 `RequestOutputCollector` 类。核心方法包括 `__init__`, `put`, `get`, `get_nowait`, `close`, `__del__`。

### `RequestOutputCollector.__init__` method / `RequestOutputCollector.__init__` 方法
```python
    def __init__(self, output_kind: RequestOutputKind, request_id: str):
        self.aggregate = output_kind == RequestOutputKind.DELTA
        self.request_id = request_id
        self.output: RequestOutput | PoolingRequestOutput | Exception | None = None
        self.ready = asyncio.Event()

        self._input_stream_task: asyncio.Task | None = None
```
**EN:** This method initializes the object state within `RequestOutputCollector`. Key calls include `Event`. It touches state such as `aggregate`, `request_id`, `output`, `ready`, `_input_stream_task`.
**CN:** 该方法会初始化对象状态，其作用域位于`RequestOutputCollector`。 关键调用包括 `Event`。 它会读写 `aggregate`, `request_id`, `output`, `ready`, `_input_stream_task` 等状态。

### `RequestOutputCollector.put` method / `RequestOutputCollector.put` 方法
```python
    def put(self, output: RequestOutput | PoolingRequestOutput | Exception) -> None:
        """Non-blocking put operation."""
        if self.output is None or isinstance(output, Exception):
            self.output = output
            self.ready.set()
        elif isinstance(self.output, RequestOutput) and isinstance(
            output, RequestOutput
        ):
            # This ensures that request outputs with different request indexes
            # (if n > 1) do not override each other.
            self.output.add(output, aggregate=self.aggregate)
        elif isinstance(self.output, PoolingRequestOutput) and isinstance(
            output, PoolingRequestOutput
        ):
            self.output = output
```
**EN:** This method implements `put` within `RequestOutputCollector`. The docstring frames it as: Non-blocking put operation. Key calls include `isinstance`, `set`, `add`. It touches state such as `output`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `put`，其作用域位于`RequestOutputCollector`。 关键调用包括 `isinstance`, `set`, `add`。 它会读写 `output` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RequestOutputCollector.get` method / `RequestOutputCollector.get` 方法
```python
    async def get(self) -> RequestOutput | PoolingRequestOutput:
        """Get operation blocks on put event."""
        while (output := self.output) is None:
            await self.ready.wait()
        self.output = None
        self.ready.clear()
        if isinstance(output, Exception):
            raise output
        return output
```
**EN:** This method implements `get` within `RequestOutputCollector`. The docstring frames it as: Get operation blocks on put event. Key calls include `clear`, `isinstance`, `wait`. It touches state such as `output`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `get`，其作用域位于`RequestOutputCollector`。 关键调用包括 `clear`, `isinstance`, `wait`。 它会读写 `output` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `OutputProcessorOutput` class / `OutputProcessorOutput` 类
```python
@dataclass
class OutputProcessorOutput:
    request_outputs: list[RequestOutput | PoolingRequestOutput]
    reqs_to_abort: list[str]
```
**EN:** Uses `@dataclass` to package related state for `OutputProcessorOutput`. Typical fields include `request_outputs`, `reqs_to_abort`.
**CN:** `OutputProcessorOutput` 使用 `@dataclass` 打包相关状态。典型字段包括 `request_outputs`, `reqs_to_abort`。

### `StreamingUpdate` class / `StreamingUpdate` 类
```python
@dataclass
class StreamingUpdate:
    """Streaming input update data for output processor.

    Contains the incremental prompt data to be applied to a request state
    when the current sub-request completes.
    """

    prompt: str | None
    prompt_token_ids: list[int] | None
    arrival_time: float
    final: bool = False
```
**EN:** Uses `@dataclass` to package related state for `StreamingUpdate`. Typical fields include `prompt`, `prompt_token_ids`, `arrival_time`, `final`.
**CN:** `StreamingUpdate` 使用 `@dataclass` 打包相关状态。典型字段包括 `prompt`, `prompt_token_ids`, `arrival_time`, `final`。

### `RequestState` class / `RequestState` 类
```python
class RequestState:
```
**EN:** Introduces the `RequestState` class. Core methods include `__init__`, `apply_streaming_update`, `from_new_request`, `make_request_output`, `_new_request_output`, `_new_completion_output`.
**CN:** 这里定义 `RequestState` 类。核心方法包括 `__init__`, `apply_streaming_update`, `from_new_request`, `make_request_output`, `_new_request_output`, `_new_completion_output`。

### `RequestState.__init__` method / `RequestState.__init__` 方法
```python
    def __init__(
        self,
        request_id: str,
        external_req_id: str,
        parent_req: ParentRequest | None,
        request_index: int,
        lora_request: LoRARequest | None,
        output_kind: RequestOutputKind,
        prompt: str | None,
        prompt_token_ids: list[int] | None,
        prompt_embeds: torch.Tensor | None,
        logprobs_processor: LogprobsProcessor | None,
        detokenizer: IncrementalDetokenizer | None,
        max_tokens_param: int | None,
        arrival_time: float,
        queue: RequestOutputCollector | None,
        log_stats: bool,
        stream_interval: int,
        top_p: float | None = None,
        n: int | None = None,
        temperature: float | None = None,
        stream_input: bool = False,
    ):
        self.request_id = request_id
        self.external_req_id = external_req_id
        self.parent_req = parent_req
        self.request_index = request_index
        self.lora_request = lora_request
        self.lora_name = lora_request.lora_name if lora_request is not None else None
        self.output_kind = output_kind
        self.prompt = prompt
        self.prompt_token_ids = prompt_token_ids
        self.prompt_embeds = prompt_embeds
        self.prompt_len = length_from_prompt_token_ids_or_embeds(
            self.prompt_token_ids, self.prompt_embeds
        )
        self.logprobs_processor = logprobs_processor
        self.detokenizer = detokenizer
        self.max_tokens_param = max_tokens_param
        self.top_p = top_p
        self.n = n
        self.temperature = temperature
        self.is_prefilling = True
        self.queue = queue
        self.num_cached_tokens = 0

        self.stats = RequestStateStats(arrival_time=arrival_time) if log_stats else None

        # Routed experts accumulation (prompt + sample chunks)
        self.routed_experts_chunks: list[np.ndarray] = []

        # Stream Interval
        self.stream_interval = stream_interval
        self.sent_tokens_offset = 0  # Offset of sent tokens

        # Streaming input queue
        self.streaming_input = stream_input
        self.input_chunk_queue: deque[StreamingUpdate] | None = (
            deque() if stream_input else None
        )
```
**EN:** This method initializes the object state within `RequestState`. Key calls include `length_from_prompt_token_ids_or_embeds`, `RequestStateStats`, `deque`. It touches state such as `request_id`, `external_req_id`, `parent_req`, `request_index`, `lora_request`, `lora_name`, `output_kind`, `prompt`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`RequestState`。 关键调用包括 `length_from_prompt_token_ids_or_embeds`, `RequestStateStats`, `deque`。 它会读写 `request_id`, `external_req_id`, `parent_req`, `request_index`, `lora_request`, `lora_name`, `output_kind`, `prompt` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RequestState.apply_streaming_update` method / `RequestState.apply_streaming_update` 方法
```python
    def apply_streaming_update(self, update: StreamingUpdate) -> None:
        # Apply the update to the request state.
        self.streaming_input = not update.final
        # TODO also include relevant output tokens in new prompt here
        #     (match scheduler behavior).
        if update.prompt:
            self.prompt = (
                (self.prompt + update.prompt) if self.prompt else update.prompt
            )
        if self.prompt_token_ids:
            self.prompt_token_ids.extend(update.prompt_token_ids or ())
        else:
            self.prompt_token_ids = update.prompt_token_ids or []
        assert self.prompt_token_ids is not None
        self.prompt_len = len(self.prompt_token_ids)
        if self.stats is not None:
            self.stats.arrival_time = update.arrival_time
        self.is_prefilling = True
```
**EN:** This method implements `apply_streaming_update` within `RequestState`. Key calls include `len`, `extend`. It touches state such as `streaming_input`, `prompt_len`, `is_prefilling`, `prompt`, `prompt_token_ids`, `stats`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_streaming_update`，其作用域位于`RequestState`。 关键调用包括 `len`, `extend`。 它会读写 `streaming_input`, `prompt_len`, `is_prefilling`, `prompt`, `prompt_token_ids`, `stats` 等状态。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RequestState.from_new_request` method / `RequestState.from_new_request` 方法
```python
    @classmethod
    def from_new_request(
        cls,
        tokenizer: TokenizerLike | None,
        request: EngineCoreRequest,
        prompt: str | None,
        parent_req: ParentRequest | None,
        request_index: int,
        queue: RequestOutputCollector | None,
        log_stats: bool,
        stream_interval: int,
    ) -> "RequestState":
        if sampling_params := request.sampling_params:
            if not sampling_params.detokenize:
                tokenizer = None
            output_kind = sampling_params.output_kind
            logprobs_processor = LogprobsProcessor.from_new_request(
                tokenizer=tokenizer,
                request=request,
            )
            detokenizer = IncrementalDetokenizer.from_new_request(
                tokenizer=tokenizer,
                request=request,
            )
            max_tokens_param = sampling_params.max_tokens
            top_p = sampling_params.top_p
            n = sampling_params.n
            temperature = sampling_params.temperature
        else:
            logprobs_processor = None
            detokenizer = None
            max_tokens_param = None
            top_p = None
            n = None
            temperature = None
            assert request.pooling_params is not None
            output_kind = request.pooling_params.output_kind

        assert request.external_req_id is not None
        return cls(
            request_id=request.request_id,
            external_req_id=request.external_req_id,
            parent_req=parent_req,
            request_index=request_index,
            lora_request=request.lora_request,
            output_kind=output_kind,
            prompt=prompt,
            prompt_token_ids=request.prompt_token_ids,
            prompt_embeds=request.prompt_embeds,
            logprobs_processor=logprobs_processor,
            detokenizer=detokenizer,
            max_tokens_param=max_tokens_param,
            top_p=top_p,
            n=n,
            temperature=temperature,
            arrival_time=request.arrival_time,
            queue=queue,
            log_stats=log_stats,
            stream_interval=stream_interval,
            stream_input=request.resumable,
        )
```
**EN:** This method reconstructs data from another representation within `RequestState`. Key calls include `cls`, `from_new_request`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`RequestState`。 关键调用包括 `cls`, `from_new_request`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `OutputProcessor` class / `OutputProcessor` 类
```python
class OutputProcessor:
    """Process EngineCoreOutputs into RequestOutputs."""
```
**EN:** Introduces the `OutputProcessor` class. Core methods include `__init__`, `get_num_unfinished_requests`, `has_unfinished_requests`, `propagate_error`, `abort_requests`, `add_request`. Docstring signal: Process EngineCoreOutputs into RequestOutputs.
**CN:** 这里定义 `OutputProcessor` 类。核心方法包括 `__init__`, `get_num_unfinished_requests`, `has_unfinished_requests`, `propagate_error`, `abort_requests`, `add_request`。

### `OutputProcessor._update_streaming_request_state` method / `OutputProcessor._update_streaming_request_state` 方法
```python
    def _update_streaming_request_state(
        self, req_state: RequestState, request: EngineCoreRequest, prompt: str | None
    ) -> None:
        """Queue a streaming update instead of immediately applying it."""
        if not request.resumable:
            # Final request - just mark completion, don't add its dummy tokens.
            if req_state.input_chunk_queue is None:
                # Engine already finished - emit final output and clean up.
                self._finish_request(req_state)
                if req_state.queue is not None:
                    # Emit a final output with finished=True
                    # to unblock the generate() loop.
                    req_state.queue.put(STREAM_FINISHED)
            elif req_state.input_chunk_queue:
                req_state.input_chunk_queue[-1].final = True
            else:
                req_state.streaming_input = False
            return

        update = StreamingUpdate(
            prompt=prompt,
            prompt_token_ids=request.prompt_token_ids,
            arrival_time=request.arrival_time,
        )

        # Apply request updates now if the last input already completed.
        if req_state.input_chunk_queue is None:
            req_state.apply_streaming_update(update)
            req_state.input_chunk_queue = deque()
        else:
            # Queue the streaming update otherwise.
            req_state.input_chunk_queue.append(update)
```
**EN:** This method implements `_update_streaming_request_state` within `OutputProcessor`. The docstring frames it as: Queue a streaming update instead of immediately applying it. Key calls include `StreamingUpdate`, `apply_streaming_update`, `deque`, `append`, `_finish_request`, `put`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_update_streaming_request_state`，其作用域位于`OutputProcessor`。 关键调用包括 `StreamingUpdate`, `apply_streaming_update`, `deque`, `append`, `_finish_request`, `put`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `OutputProcessor.process_outputs` method / `OutputProcessor.process_outputs` 方法
```python
    def process_outputs(
        self,
        engine_core_outputs: list[EngineCoreOutput],
        engine_core_timestamp: float | None = None,
        iteration_stats: IterationStats | None = None,
    ) -> OutputProcessorOutput:
        """
        Process the EngineCoreOutputs:
        1) Compute stats for logging
        2) Detokenize
        3) Create and handle RequestOutput objects:
            * If there is a queue (for usage with AsyncLLM),
              put the RequestOutput objects into the queue for
              handling by the per-request generate() tasks.

            * If there is no queue (for usage with LLMEngine),
              return a list of RequestOutput objects.

        NOTE FOR DEVELOPERS

        vLLM V1 minimizes the number of python loops over the full
        batch to ensure system overheads are minimized. This is the
        only function that should loop over EngineCoreOutputs.

        If you need to touch every element of the batch, do it from
        within the loop below.
        """

        request_outputs: list[RequestOutput | PoolingRequestOutput] = []
        reqs_to_abort: list[str] = []
        for engine_core_output in engine_core_outputs:
            req_id = engine_core_output.request_id
            req_state = self.request_states.get(req_id)
            if req_state is None:
                # Ignore output for already-aborted request.
                continue

            # 1) Compute stats for this iteration.
            self._update_stats_from_output(
                req_state, engine_core_output, engine_core_timestamp, iteration_stats
            )

            new_token_ids = engine_core_output.new_token_ids
            pooling_output = engine_core_output.pooling_output
            finish_reason = engine_core_output.finish_reason
            stop_reason = engine_core_output.stop_reason
            kv_transfer_params = engine_core_output.kv_transfer_params
            if engine_core_output.routed_experts is not None:
                req_state.routed_experts_chunks.append(
                    engine_core_output.routed_experts
    # ... omitted for brevity ...
            if finish_reason is not None:
                if req_state.streaming_input:
                    if req_state.input_chunk_queue:
                        update = req_state.input_chunk_queue.popleft()
                        req_state.apply_streaming_update(update)
                    else:
                        req_state.input_chunk_queue = None
                else:
                    self._finish_request(req_state)
                    if not engine_core_output.finished:
                        # If req not finished in EngineCore, but Detokenizer
                        # detected stop string, abort needed in EngineCore.
                        reqs_to_abort.append(req_id)

                    # Track per-request stats
                    self._update_stats_from_finished(
                        req_state, finish_reason, iteration_stats
                    )
                    if self.tracing_enabled:
                        self.do_tracing(engine_core_output, req_state, iteration_stats)

        return OutputProcessorOutput(
            request_outputs=request_outputs,
            reqs_to_abort=reqs_to_abort,
        )
```
**EN:** This method implements `process_outputs` within `OutputProcessor`. The docstring frames it as: Process the EngineCoreOutputs: 1) Compute stats for logging 2) Detokenize 3) Create and handle RequestOutput objects: * If there is a queue (for usage with AsyncLLM), put the RequestOutput objects into the queue for handling by the per-request generate() tasks. Key calls include `OutputProcessorOutput`, `get`, `_update_stats_from_output`, `append`, `update`, `update_from_output`. The control flow contains 14 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `process_outputs`，其作用域位于`OutputProcessor`。 关键调用包括 `OutputProcessorOutput`, `get`, `_update_stats_from_output`, `append`, `update`, `update_from_output`。 控制流包含 14 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `OutputProcessor.update_scheduler_stats` method / `OutputProcessor.update_scheduler_stats` 方法
```python
    def update_scheduler_stats(self, scheduler_stats: SchedulerStats | None):
        self.lora_states.update_scheduler_stats(scheduler_stats)
```
**EN:** This method updates existing state within `OutputProcessor`. Key calls include `update_scheduler_stats`.
**CN:** 该方法会更新现有状态，其作用域位于`OutputProcessor`。 关键调用包括 `update_scheduler_stats`。

## Key Concepts / 关键概念
- `RequestOutputCollector`: central class or interface in this module. / `RequestOutputCollector`：本模块中的核心类或接口。
- `OutputProcessorOutput`: central class or interface in this module. / `OutputProcessorOutput`：本模块中的核心类或接口。
- `StreamingUpdate`: central class or interface in this module. / `StreamingUpdate`：本模块中的核心类或接口。
- `RequestState`: central class or interface in this module. / `RequestState`：本模块中的核心类或接口。
- `OutputProcessor`: central class or interface in this module. / `OutputProcessor`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `asyncio`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.lora.request`, `vllm.outputs`, `vllm.sampling_params`, `vllm.tokenizers`, `vllm.tracing`, `vllm.utils`, `vllm.v1.engine`, `vllm.v1.engine.detokenizer`, `vllm.v1.engine.logprobs`, `vllm.v1.engine.parallel_sampling`, `vllm.v1.metrics.stats`
