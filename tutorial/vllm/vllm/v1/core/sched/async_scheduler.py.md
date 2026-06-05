# async_scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/sched/async_scheduler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AsyncScheduler` for the V1 `core/sched` subsystem. / 为 V1 的 `core/sched` 子系统实现 `AsyncScheduler`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from vllm.logger import init_logger
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.core.sched.scheduler import Scheduler
from vllm.v1.request import Request, RequestStatus

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `AsyncScheduler` class / `AsyncScheduler` 类
```python
class AsyncScheduler(Scheduler):
```
**EN:** Introduces the `AsyncScheduler` class on top of `Scheduler`. Core methods include `__init__`, `_update_after_schedule`, `_update_request_with_output`.
**CN:** 这里定义 `AsyncScheduler` 类，其基类包括 `Scheduler`。核心方法包括 `__init__`, `_update_after_schedule`, `_update_request_with_output`。

### `AsyncScheduler.__init__` method / `AsyncScheduler.__init__` 方法
```python
    def __init__(self, *args, **kwargs) -> None:
        super().__init__(*args, **kwargs)
        # reusable read-only placeholder list for speculative decoding.
        self._spec_token_placeholders: list[int] = [-1] * self.num_spec_tokens
```
**EN:** This method initializes the object state within `AsyncScheduler`. Key calls include `__init__`, `super`. It touches state such as `_spec_token_placeholders`.
**CN:** 该方法会初始化对象状态，其作用域位于`AsyncScheduler`。 关键调用包括 `__init__`, `super`。 它会读写 `_spec_token_placeholders` 等状态。

### `AsyncScheduler._update_after_schedule` method / `AsyncScheduler._update_after_schedule` 方法
```python
    def _update_after_schedule(self, scheduler_output: SchedulerOutput) -> None:
        super()._update_after_schedule(scheduler_output)
        spec_decode_tokens = scheduler_output.scheduled_spec_decode_tokens
        for req_id in scheduler_output.num_scheduled_tokens:
            request = self.requests[req_id]
            if request.is_prefill_chunk:
                continue

            scheduler_output.pending_structured_output_tokens |= (
                request.use_structured_output and request.num_output_placeholders > 0
            )
            # The request will generate a new token plus num_spec_tokens
            # in this scheduling step.
            cur_num_spec_tokens = len(spec_decode_tokens.get(req_id, ()))
            request.num_output_placeholders += 1 + cur_num_spec_tokens
            # Add placeholders for the new draft/spec tokens.
            # We will update the actual spec token ids in the worker process.
            request.spec_token_ids = self._spec_token_placeholders
```
**EN:** This method implements `_update_after_schedule` within `AsyncScheduler`. Key calls include `_update_after_schedule`, `len`, `super`, `get`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_update_after_schedule`，其作用域位于`AsyncScheduler`。 关键调用包括 `_update_after_schedule`, `len`, `super`, `get`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `AsyncScheduler._update_request_with_output` method / `AsyncScheduler._update_request_with_output` 方法
```python
    def _update_request_with_output(
        self, request: Request, new_token_ids: list[int]
    ) -> tuple[list[int], bool]:
        if request.discard_latest_async_tokens:
            # If the request is force preempted in reset_prefix_cache, we
            # should discard the latest async token.
            request.discard_latest_async_tokens = False
            return [], False

        status_before_update = request.status
        new_token_ids, stopped = super()._update_request_with_output(
            request, new_token_ids
        )

        # Update the number of output placeholders.
        request.num_output_placeholders -= len(new_token_ids)
        assert request.num_output_placeholders >= 0

        # Cache the new tokens. Preempted requests should be skipped.
        if status_before_update == RequestStatus.RUNNING:
            self.kv_cache_manager.cache_blocks(
                request, request.num_computed_tokens - request.num_output_placeholders
            )
        return new_token_ids, stopped
```
**EN:** This method implements `_update_request_with_output` within `AsyncScheduler`. Key calls include `_update_request_with_output`, `len`, `cache_blocks`, `super`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_update_request_with_output`，其作用域位于`AsyncScheduler`。 关键调用包括 `_update_request_with_output`, `len`, `cache_blocks`, `super`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `AsyncScheduler`: central class or interface in this module. / `AsyncScheduler`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.v1.core.sched.output`, `vllm.v1.core.sched.scheduler`, `vllm.v1.request`
