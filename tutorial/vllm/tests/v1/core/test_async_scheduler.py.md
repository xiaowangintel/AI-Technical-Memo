# test_async_scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/core/test_async_scheduler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `async scheduler` behavior and regressions in the v1 stack. / 验证 v1 栈中 `async 调度器` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-14)
```python
from collections import deque
from unittest.mock import Mock

import pytest

from vllm.v1.core.sched.async_scheduler import AsyncScheduler
from vllm.v1.core.sched.output import CachedRequestData, SchedulerOutput
from vllm.v1.outputs import ModelRunnerOutput
from vllm.v1.request import RequestStatus
from vllm.v1.utils import ConstantList

from .utils import create_requests, create_scheduler
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.v1.core.sched.async_scheduler, vllm.v1.core.sched.output, vllm.v1.outputs, vllm.v1.request, vllm.v1.utils`. Local helpers come from `tests.v1.core.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.v1.core.sched.async_scheduler, vllm.v1.core.sched.output, vllm.v1.outputs, vllm.v1.request, vllm.v1.utils`。 本地测试辅助逻辑来自 `tests.v1.core.utils`。

### Module state / 模块级状态 (line 16)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### _make_model_runner_output (lines 19-30)
```python
def _make_model_runner_output(
    scheduler_output: SchedulerOutput,
) -> ModelRunnerOutput:
    req_ids = list(scheduler_output.num_scheduled_tokens.keys())
    return ModelRunnerOutput(
        req_ids=req_ids,
        req_id_to_index={req_id: i for i, req_id in enumerate(req_ids)},
        sampled_token_ids=[[i] for i in range(len(req_ids))],
        logprobs=None,
        prompt_logprobs_dict={},
        pooler_output=[],
    )
```
**EN:** Helper function `_make_model_runner_output` encapsulates reusable logic for `model runner output`. Inputs: `scheduler_output`. Key calls include `list, ModelRunnerOutput, num_scheduled_tokens.keys, enumerate, range, len`.
**CN:** 辅助函数 `_make_model_runner_output` 封装了与 `模型执行器 output` 相关的可复用逻辑。 输入参数：`scheduler_output`。 关键调用包括 `list, ModelRunnerOutput, num_scheduled_tokens.keys, enumerate, range, len`。

### test_stop_by_max_tokens (lines 34-64)
```python
def test_stop_by_max_tokens(max_tokens: int):
    scheduler = create_scheduler(async_scheduling=True)
    requests = create_requests(num_requests=2, max_tokens=max_tokens)
    req0, req1 = requests

    expected_total_num_scheduled_tokens = 0
    sched_outputs: deque[SchedulerOutput] = deque()
    scheduler.add_request(req0)
    sched_outputs.append(scheduler.schedule())
    expected_total_num_scheduled_tokens += req0.num_prompt_tokens + max_tokens - 1

    scheduler.add_request(req1)
    sched_outputs.append(scheduler.schedule())
    expected_total_num_scheduled_tokens += req1.num_prompt_tokens + max_tokens - 1

    total_num_scheduled_tokens = 0
    while sched_outputs:
        sched_output = sched_outputs.popleft()
        total_num_scheduled_tokens += sched_output.total_num_scheduled_tokens
        model_runner_output = _make_model_runner_output(sched_output)
        scheduler.update_from_output(sched_output, model_runner_output)

        sched_output = scheduler.schedule()
        if sched_output.num_scheduled_tokens:
            sched_outputs.append(sched_output)

    assert scheduler.get_num_unfinished_requests() == 0
    assert req0.num_output_tokens == max_tokens
    assert req1.num_output_tokens == max_tokens
    # Ensure we aren't scheduling more tokens than necessary.
    assert total_num_scheduled_tokens == expected_total_num_scheduled_tokens
```
**EN:** Parameterized test covering `stop by max tokens`. Parameter axes: `max_tokens`. Inputs/fixtures: `max_tokens`. It exercises `mark.parametrize, create_scheduler, create_requests, deque, scheduler.add_request, sched_outputs.append`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `stop by max tokens` 的测试用例。 参数维度：`max_tokens`。 输入或 fixture：`max_tokens`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, deque, scheduler.add_request, sched_outputs.append`。 代码主体包含 4 个显式断言。

### test_abort (lines 67-100)
```python
def test_abort():
    scheduler = create_scheduler(async_scheduling=True)
    requests = create_requests(num_requests=10, max_tokens=20)

    for req in requests:
        scheduler.add_request(req)
    sched_outputs: deque[SchedulerOutput] = deque()
    sched_outputs.append(scheduler.schedule())
    abort_order = [0, 8, 3, 1, 6, 4, 2, 5, 7, 9]
    abort_order_copy = abort_order.copy()
    def abort_request():
        if not abort_order:
            return
        req = requests[abort_order.pop(0)]
    # ... excerpt omitted for brevity ...
        if sched_output.num_scheduled_tokens:
            sched_outputs.append(sched_output)
    for i, req in enumerate(requests):
        assert req.status == RequestStatus.FINISHED_ABORTED
        assert req.num_output_tokens == abort_order_copy.index(i)
```
**EN:** Test case covering `abort`. It exercises `create_scheduler, create_requests, deque, sched_outputs.append, abort_order.copy, enumerate`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort` 的测试用例。 该测试会调用 `create_scheduler, create_requests, deque, sched_outputs.append, abort_order.copy, enumerate`。 代码主体包含 2 个显式断言。

### test_preempt (lines 103-136)
```python
def test_preempt():
    scheduler = create_scheduler(async_scheduling=True)
    requests = create_requests(num_requests=10, max_tokens=20)

    for req in requests:
        scheduler.add_request(req)
    sched_outputs: deque[SchedulerOutput] = deque()
    sched_outputs.append(scheduler.schedule())
    abort_order = [0, 8, 3, 1, 6, 4, 2, 5, 7, 9]
    abort_order_copy = abort_order.copy()
    def abort_request():
        if not abort_order:
            return
        req = requests[abort_order.pop(0)]
    # ... excerpt omitted for brevity ...
        if sched_output.num_scheduled_tokens:
            sched_outputs.append(sched_output)
    for i, req in enumerate(requests):
        assert req.status == RequestStatus.FINISHED_ABORTED
        assert req.num_output_tokens == abort_order_copy.index(i)
```
**EN:** Test case covering `preempt`. It exercises `create_scheduler, create_requests, deque, sched_outputs.append, abort_order.copy, enumerate`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `preempt` 的测试用例。 该测试会调用 `create_scheduler, create_requests, deque, sched_outputs.append, abort_order.copy, enumerate`。 代码主体包含 2 个显式断言。

### test_prefix_caching_for_prefill_dedup (lines 139-193)
```python
def test_prefix_caching_for_prefill_dedup():
    CHUNK_SIZE = 1000
    BLOCK_SIZE = 16
    num_prompt_tokens = 100
    scheduler = create_scheduler(
        async_scheduling=True,
        max_num_batched_tokens=CHUNK_SIZE,
        enable_prefix_caching=True,
        block_size=BLOCK_SIZE,
    )
    requests = create_requests(
        num_requests=5,
        num_tokens=num_prompt_tokens,
        max_tokens=3,
        same_prompt=True,

    # ... excerpt omitted for brevity ...
    assert len(sched_output.num_scheduled_tokens) == 2
    assert sched_output.num_scheduled_tokens[req0.request_id] == num_prompt_tokens
    assert (
                assert (
                    sched_output.num_scheduled_tokens[added_req.request_id]
                    == num_prompt_tokens % BLOCK_SIZE
                )
    assert scheduler.get_num_unfinished_requests() == 0
```
**EN:** Test case covering `prefix caching for prefill dedup`. It exercises `create_scheduler, create_requests, requests.pop, scheduler.add_request, deque, scheduler.schedule`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `前缀缓存 for prefill dedup` 的测试用例。 该测试会调用 `create_scheduler, create_requests, requests.pop, scheduler.add_request, deque, scheduler.schedule`。 代码主体包含 5 个显式断言。

### test_prefix_caching_for_multi_turn (lines 196-258)
```python
def test_prefix_caching_for_multi_turn():
    CHUNK_SIZE = 1000
    BLOCK_SIZE = 16
    num_prompt_tokens = 100
    num_output_tokens = 200
    scheduler = create_scheduler(
        async_scheduling=True,
        max_num_batched_tokens=CHUNK_SIZE,
        enable_prefix_caching=True,
        block_size=BLOCK_SIZE,
    )
    requests = create_requests(
        num_requests=5,
        num_tokens=num_prompt_tokens,
        max_tokens=num_output_tokens,

    # ... excerpt omitted for brevity ...
    assert scheduler.get_num_unfinished_requests() == 0
    # Make sure the next-turn requests get prefix cache hit by the previous
    # requests.
    for req in next_turn_requests:
        assert sched_output.num_scheduled_tokens[req.request_id] == (
            req.num_prompt_tokens % BLOCK_SIZE
        )
```
**EN:** Test case covering `prefix caching for multi turn`. It exercises `create_scheduler, create_requests, deque, sched_outputs.append, enumerate, scheduler.schedule`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `前缀缓存 for multi turn` 的测试用例。 该测试会调用 `create_scheduler, create_requests, deque, sched_outputs.append, enumerate, scheduler.schedule`。 代码主体包含 2 个显式断言。

### test_abort_request_when_structured_output_fsm_cannot_advance (lines 261-322)
```python
def test_abort_request_when_structured_output_fsm_cannot_advance():
    scheduler = object.__new__(AsyncScheduler)
    request = create_requests(num_requests=1, num_tokens=1)[0]
    request.structured_output_request = Mock()
    request.structured_output_request.grammar = Mock()
    request.structured_output_request.grammar.accept_tokens.return_value = False
    request.status = RequestStatus.RUNNING
    request.num_computed_tokens = request.num_tokens
    request.num_output_placeholders = 1

    scheduler.perf_metrics = None
    scheduler.connector = None
    scheduler.structured_output_manager = Mock()
    scheduler.structured_output_manager.should_advance.return_value = True
    scheduler.requests = {request.request_id: request}
    scheduler.running = [request]
    scheduler.waiting = Mock()
    scheduler.kv_cache_manager = Mock()
    # ... excerpt omitted for brevity ...
    scheduler.kv_event_publisher = Mock()
    scheduler.vllm_config = Mock()
    scheduler.make_stats = Mock(return_value=None)
        return None
    scheduler._free_request = Mock(side_effect=free_request)
    scheduler.update_from_output(output, model_runner_output)
    assert request.resumable is False
    assert request.status == RequestStatus.FINISHED_ERROR
    assert request.request_id not in scheduler.requests
    assert not scheduler.running
```
**EN:** Test case covering `abort request when structured output fsm cannot advance`. It exercises `object.__new__, Mock, set, SchedulerOutput, ModelRunnerOutput, scheduler.update_from_output`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort request when structured output fsm cannot advance` 的测试用例。 该测试会调用 `object.__new__, Mock, set, SchedulerOutput, ModelRunnerOutput, scheduler.update_from_output`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.v1.core.sched.async_scheduler, vllm.v1.core.sched.output, vllm.v1.outputs, vllm.v1.request, vllm.v1.utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.core.sched.async_scheduler, vllm.v1.core.sched.output, vllm.v1.outputs, vllm.v1.request, vllm.v1.utils`。
- **EN:** Local test helpers: `tests.v1.core.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.core.utils`。
- **EN:** Standard-library support: `collections, unittest.mock`.
- **CN:** 标准库支持：`collections, unittest.mock`。
