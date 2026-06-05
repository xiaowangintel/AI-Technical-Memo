# test_scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/core/test_scheduler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `scheduler` behavior and regressions in the v1 stack. / 验证 v1 栈中 `调度器` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-39)
```python
import dataclasses
from unittest.mock import Mock

import pytest
import torch
from vllm.config import (
    CacheConfig,
    ECTransferConfig,
    KVTransferConfig,
    ModelConfig,
    SchedulerConfig,
    SpeculativeConfig,
    VllmConfig,
)
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalKwargsItem,
# ... excerpt omitted for brevity ...
from vllm.v1.outputs import DraftTokenIds, KVConnectorOutput, ModelRunnerOutput
from vllm.v1.request import Request, RequestStatus
from vllm.v1.structured_output import StructuredOutputManager
from .utils import EOS_TOKEN_ID, create_requests, create_scheduler, mock_kv
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.encoder_cache_manager, ...`. Local helpers come from `tests.v1.core.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.encoder_cache_manager, ...`。 本地测试辅助逻辑来自 `tests.v1.core.utils`。

### Module state / 模块级状态 (line 41)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### test_add_requests (lines 44-51)
```python
def test_add_requests():
    scheduler = create_scheduler()
    requests = create_requests(num_requests=10)

    for i, request in enumerate(requests):
        scheduler.add_request(request)
        assert request.request_id in scheduler.requests
        assert len(scheduler.waiting) == i + 1
```
**EN:** Test case covering `add requests`. It exercises `create_scheduler, create_requests, enumerate, scheduler.add_request, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `add requests` 的测试用例。 该测试会调用 `create_scheduler, create_requests, enumerate, scheduler.add_request, len`。 代码主体包含 2 个显式断言。

### test_finish_request (lines 54-63)
```python
def test_finish_request():
    scheduler = create_scheduler()
    requests = create_requests(num_requests=10)
    for request in requests:
        scheduler.add_request(request)

    for i, request in enumerate(requests):
        scheduler.finish_requests(request.request_id, RequestStatus.FINISHED_ABORTED)
        assert request.request_id not in scheduler.requests
        assert len(scheduler.waiting) == 9 - i
```
**EN:** Test case covering `finish request`. It exercises `create_scheduler, create_requests, enumerate, scheduler.add_request, scheduler.finish_requests, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `finish request` 的测试用例。 该测试会调用 `create_scheduler, create_requests, enumerate, scheduler.add_request, scheduler.finish_requests, len`。 代码主体包含 2 个显式断言。

### test_get_num_unfinished_requests (lines 66-74)
```python
def test_get_num_unfinished_requests():
    scheduler = create_scheduler()
    requests = create_requests(num_requests=10)
    for request in requests:
        scheduler.add_request(request)

    for i, request in enumerate(requests):
        scheduler.finish_requests(request.request_id, RequestStatus.FINISHED_STOPPED)
        assert scheduler.get_num_unfinished_requests() == len(requests) - i - 1
```
**EN:** Test case covering `get num unfinished requests`. It exercises `create_scheduler, create_requests, enumerate, scheduler.add_request, scheduler.finish_requests, scheduler.get_num_unfinished_requests`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get num unfinished requests` 的测试用例。 该测试会调用 `create_scheduler, create_requests, enumerate, scheduler.add_request, scheduler.finish_requests, scheduler.get_num_unfinished_requests`。 代码主体包含 1 个显式断言。

### test_schedule (lines 84-106)
```python
def test_schedule(enable_prefix_caching: bool, prompt_logprobs: int | None):
    """Test scheduling.
    Two cases: default APC/no prompt logprobs; APC=True + prompt logprobs
    """
    scheduler = create_scheduler(enable_prefix_caching=enable_prefix_caching)
    requests = create_requests(num_requests=10, prompt_logprobs=prompt_logprobs)
    for request in requests:
        scheduler.add_request(request)

    # Test initial scheduling
    output = scheduler.schedule()
    assert len(output.scheduled_new_reqs) == len(requests)
    assert output.scheduled_cached_reqs.num_reqs == 0
    assert len(output.finished_req_ids) == 0
    # Verify all requests are scheduled.
    for req_id, num_tokens in output.num_scheduled_tokens.items():
        assert num_tokens == len(requests[int(req_id)].prompt_token_ids)

    # Verify requests moved from waiting to running
    assert len(scheduler.waiting) == 0
    assert len(scheduler.running) == len(requests)
    for i, request in enumerate(requests):
        assert scheduler.running[i] == request
```
**EN:** Parameterized test covering `schedule`. Parameter axes: `enable_prefix_caching, prompt_logprobs`. Inputs/fixtures: `enable_prefix_caching, prompt_logprobs`. It exercises `mark.parametrize, create_scheduler, create_requests, scheduler.schedule, num_scheduled_tokens.items, enumerate`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule` 的测试用例。 参数维度：`enable_prefix_caching, prompt_logprobs`。 输入或 fixture：`enable_prefix_caching, prompt_logprobs`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, scheduler.schedule, num_scheduled_tokens.items, enumerate`。 代码主体包含 7 个显式断言。

### test_schedule_multimodal_requests (lines 109-128)
```python
def test_schedule_multimodal_requests():
    scheduler = create_scheduler(model="llava-hf/llava-1.5-7b-hf")
    mm_positions = [[PlaceholderRange(offset=i, length=100)] for i in range(10)]
    requests = create_requests(
        num_requests=10,
        num_tokens=200,
        mm_positions=mm_positions,
    )
    for request in requests:
        scheduler.add_request(request)

    output = scheduler.schedule()
    assert len(output.scheduled_new_reqs) == len(requests)
    assert output.scheduled_cached_reqs.num_reqs == 0
    assert len(output.finished_req_ids) == 0
    for req_id, num_tokens in output.num_scheduled_tokens.items():
        assert num_tokens == len(requests[int(req_id)].prompt_token_ids)
    assert len(output.scheduled_encoder_inputs) == 10
    for req_id, encoder_input in output.scheduled_encoder_inputs.items():
        assert len(encoder_input) == 1
```
**EN:** Test case covering `schedule multimodal requests`. It exercises `create_scheduler, create_requests, scheduler.schedule, num_scheduled_tokens.items, scheduled_encoder_inputs.items, scheduler.add_request`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule multimodal requests` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.schedule, num_scheduled_tokens.items, scheduled_encoder_inputs.items, scheduler.add_request`。 代码主体包含 6 个显式断言。

### test_async_scheduling_pp_allows_rescheduling_with_output_placeholders (lines 131-143)
```python
def test_async_scheduling_pp_allows_rescheduling_with_output_placeholders():
    """Async scheduling + PP: allow multi-step in-flight scheduling per request"""
    scheduler = create_scheduler(async_scheduling=True, pipeline_parallel_size=2)
    (req,) = create_requests(num_requests=1, num_tokens=8)
    scheduler.add_request(req)

    _ = scheduler.schedule()
    assert req.num_output_placeholders > 0

    # before any update_from_output, we still expect the request can be
    # scheduled again (multi-step in-flight).
    output = scheduler.schedule()
    assert req.request_id in output.num_scheduled_tokens
```
**EN:** Test case covering `async scheduling pp allows rescheduling with output placeholders`. It exercises `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `async scheduling pp allows rescheduling with output placeholders` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule`。 代码主体包含 2 个显式断言。

### test_schedule_partial_requests (lines 146-206)
```python
def test_schedule_partial_requests():
    """Test scheduling behavior with partial requests.

    This test verifies that:
    1. The scheduler can handle multiple partial requests in a single step when
       constrained by encoder budget.
    2. A request in RUNNING state may be unscheduled in subsequent steps if
       there is insufficient encoder budget.
    """
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        max_num_batched_tokens=1024,
    )
    mm_positions = [[PlaceholderRange(offset=100, length=600)] for _ in range(3)]
    requests = create_requests(
        num_requests=3,
        num_tokens=800,
        mm_positions=mm_positions,
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 3
    assert output.scheduled_cached_reqs.num_reqs == 0
    assert len(output.finished_req_ids) == 0
    assert scheduler.max_num_encoder_input_tokens == 1024
    assert output.num_scheduled_tokens[requests[0].request_id] == 800
    assert output.num_scheduled_tokens[requests[1].request_id] == 100
    assert len(output.scheduled_new_reqs) == 0
    assert output.scheduled_cached_reqs.num_reqs == 2
    assert output.num_scheduled_tokens[requests[0].request_id] == 1
    assert output.num_scheduled_tokens[requests[1].request_id] == 700
    assert requests[2].request_id not in output.num_scheduled_tokens
```
**EN:** Test case covering `schedule partial requests`. It exercises `create_scheduler, create_requests, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output, scheduler.add_request`. The body contains 14 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule partial requests` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output, scheduler.add_request`。 代码主体包含 14 个显式断言。

### test_no_mm_input_chunking (lines 209-256)
```python
def test_no_mm_input_chunking():
    # Disable multimodal input chunking.
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        max_num_batched_tokens=1024,
        disable_chunked_mm_input=True,
        max_model_len=2048,
    )
    mm_positions = [[PlaceholderRange(offset=400, length=800)]]
    requests = create_requests(
        num_requests=1, num_tokens=1200, mm_positions=mm_positions
    for request in requests:
        scheduler.add_request(request)

    output = scheduler.schedule()
    assert len(output.scheduled_new_reqs) == 1
    assert output.scheduled_cached_reqs.num_reqs == 0
    # ... excerpt omitted for brevity ...
    assert len(output.finished_req_ids) == 0
    assert output.num_scheduled_tokens[requests[0].request_id] == 400
    assert len(scheduler.running) == 1
    assert len(output.scheduled_new_reqs) == 0
    assert output.scheduled_cached_reqs.num_reqs == 1
    with pytest.raises(ValueError):
        _ = create_scheduler(
            model="llava-hf/llava-1.5-7b-hf",
            max_num_batched_tokens=100,
            disable_chunked_mm_input=True,
        )
```
**EN:** Test case covering `no mm input chunking`. It exercises `create_scheduler, create_requests, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output, scheduler.add_request`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `no mm input chunking` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output, scheduler.add_request`。 代码主体包含 9 个显式断言。

### test_schedule_concurrent_partial_requests (lines 260-332)
```python
def test_schedule_concurrent_partial_requests(enable_prefix_caching: bool):
    """Test scheduling behavior with concurrent partial requests.

    This test verifies that: there are multiple long prefill requests in the
    RUNNING state, and we can schedule them together.
    """
    scheduler = create_scheduler(
        model="facebook/opt-125m",
        max_num_batched_tokens=1024,
        long_prefill_token_threshold=400,
        enable_prefix_caching=enable_prefix_caching,
    )
    requests = create_requests(
        num_requests=3,
        num_tokens=800,
    for request in requests:
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 3
    assert output.scheduled_cached_reqs.num_reqs == 0
    assert len(output.finished_req_ids) == 0
    assert output.num_scheduled_tokens[requests[0].request_id] == 400
    assert output.num_scheduled_tokens[requests[1].request_id] == 400
    assert output.num_scheduled_tokens[requests[2].request_id] == 224
    assert len(output2.scheduled_new_reqs) == 0
    assert output2.scheduled_cached_reqs.num_reqs == 3
    assert len(output2.finished_req_ids) == 0
    assert output2.num_scheduled_tokens[requests[0].request_id] == 1
    assert output2.num_scheduled_tokens[requests[1].request_id] == 1
    assert output2.num_scheduled_tokens[requests[2].request_id] == 800 - 224 - 224
```
**EN:** Parameterized test covering `schedule concurrent partial requests`. Parameter axes: `enable_prefix_caching`. Inputs/fixtures: `enable_prefix_caching`. It exercises `mark.parametrize, create_scheduler, create_requests, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`. The body contains 20 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule concurrent partial requests` 的测试用例。 参数维度：`enable_prefix_caching`。 输入或 fixture：`enable_prefix_caching`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`。 代码主体包含 20 个显式断言。

### test_stop_via_update_from_output (lines 335-504)
```python
def test_stop_via_update_from_output():
    """Test stopping behavior through update_from_output"""
    scheduler = create_scheduler(num_speculative_tokens=1)

    # Test case 1: Stop on EOS token
    requests = create_requests(num_requests=2, max_tokens=10)
    for req in requests:
        req.num_computed_tokens = req.num_tokens
        scheduler.requests[req.request_id] = req
        scheduler.running.append(req)
        req.status = RequestStatus.RUNNING
    scheduler_output = SchedulerOutput(
        scheduled_new_reqs=[],
        scheduled_cached_reqs=CachedRequestData.make_empty(),
        num_scheduled_tokens={requests[0].request_id: 1, requests[1].request_id: 2},
        total_num_scheduled_tokens=3,
        scheduled_encoder_inputs={},
    # ... excerpt omitted for brevity ...
    assert len(scheduler.running) == 1
    assert scheduler.running[0].request_id == requests[1].request_id
    assert requests[0].status == RequestStatus.FINISHED_STOPPED
    assert requests[0].request_id in scheduler.finished_req_ids
    assert list(requests[0].output_token_ids) == [EOS_TOKEN_ID]
    assert list(requests[1].output_token_ids) == [10, 11]
    scheduler.update_from_output(scheduler_output, model_output)
    # Verify request continues past EOS
    assert not requests[0].is_finished()
    assert list(requests[0].output_token_ids) == [EOS_TOKEN_ID, 10, 11]
```
**EN:** Test case covering `stop via update from output`. It exercises `create_scheduler, create_requests, SchedulerOutput, ModelRunnerOutput, scheduler.update_from_output, running.append`. The body contains 22 explicit assertion(s).
**CN:** 该代码块是覆盖 `stop via update from output` 的测试用例。 该测试会调用 `create_scheduler, create_requests, SchedulerOutput, ModelRunnerOutput, scheduler.update_from_output, running.append`。 代码主体包含 22 个显式断言。

### test_check_stop_min_tokens (lines 507-594)
```python
def test_check_stop_min_tokens():
    """Test that requests don't stop when min_tokens requirement isn't met."""
    from vllm.v1.core.sched.utils import check_stop

    # Test case 1: num_output_tokens < min_tokens
    # Should return False (don't stop)
    sampling_params = SamplingParams(
        ignore_eos=False,
        max_tokens=20,
        min_tokens=5,
    )
    sampling_params.update_from_generation_config({}, EOS_TOKEN_ID)
    request = Request(
        request_id="0",
        prompt_token_ids=[0, 1, 2],
        sampling_params=sampling_params,
        pooling_params=None,
    # ... excerpt omitted for brevity ...
    assert result is False, "Should not stop when num_output_tokens<min_tokens"
    assert result is True, "Should stop on EOS when min_tokens met"
    assert request.status == RequestStatus.FINISHED_STOPPED
    assert result is True, "Should stop on EOS when min_tokens=0"
    assert request_no_min.status == RequestStatus.FINISHED_STOPPED
    )  # 6 tokens >= min_tokens=5
    result = check_stop(request_stop, max_model_len=100)
    assert result is True, "Should stop on stop token when min_tokens met"
    assert request_stop.status == RequestStatus.FINISHED_STOPPED
    assert request_stop.stop_reason == 42
```
**EN:** Test case covering `check stop min tokens`. It exercises `SamplingParams, sampling_params.update_from_generation_config, Request, request.append_output_token_ids, check_stop, sampling_params_no_min.update_from_generation_config`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `check stop min tokens` 的测试用例。 该测试会调用 `SamplingParams, sampling_params.update_from_generation_config, Request, request.append_output_token_ids, check_stop, sampling_params_no_min.update_from_generation_config`。 代码主体包含 9 个显式断言。

### test_schedule_concurrent_batches (lines 604-656)
```python
def test_schedule_concurrent_batches(
    enable_prefix_caching: bool, prompt_logprobs: int | None
):
    scheduler = create_scheduler(
        max_num_batched_tokens=1024,
        max_num_seqs=2,
        enable_prefix_caching=enable_prefix_caching,
    )
    requests = create_requests(
        num_requests=2,
        num_tokens=512,
        prompt_logprobs=prompt_logprobs,

    # Schedule the first request.
    scheduler.add_request(requests[0])
    scheduler_output0 = scheduler.schedule()
    assert len(scheduler_output0.scheduled_new_reqs) == 1
    # ... excerpt omitted for brevity ...
    assert scheduler_output0.num_scheduled_tokens[requests[0].request_id] == 512
    assert len(scheduler_output1.scheduled_new_reqs) == 1
    assert scheduler_output1.num_scheduled_tokens[requests[1].request_id] == 512
    assert scheduler_output2.num_scheduled_tokens[requests[0].request_id] == 1
        sampled_token_ids=[[0]],
        logprobs=None,
        prompt_logprobs_dict={},
        pooler_output=[],
    scheduler.update_from_output(scheduler_output1, model_runner_output)
```
**EN:** Parameterized test covering `schedule concurrent batches`. Parameter axes: `enable_prefix_caching, prompt_logprobs`. Inputs/fixtures: `enable_prefix_caching, prompt_logprobs`. It exercises `mark.parametrize, create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, ModelRunnerOutput`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule concurrent batches` 的测试用例。 参数维度：`enable_prefix_caching, prompt_logprobs`。 输入或 fixture：`enable_prefix_caching, prompt_logprobs`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, ModelRunnerOutput`。 代码主体包含 5 个显式断言。

### test_schedule_order (lines 660-684)
```python
def test_schedule_order(enable_chunked_prefill: bool):
    scheduler = create_scheduler(
        max_num_batched_tokens=1024,
        max_num_seqs=3,
        enable_chunked_prefill=enable_chunked_prefill,
    )

    # long requests
    requests = create_requests(num_requests=2, num_tokens=800, req_ids=["1", "2"])
    # short requests
    requests += create_requests(num_requests=2, num_tokens=10, req_ids=["3", "4"])

    for request in requests:
        scheduler.add_request(request)

    scheduler_output1 = scheduler.schedule()

    if enable_chunked_prefill:
        # When enable chunked prefill, long requests will be chunked.
        assert len(scheduler_output1.scheduled_new_reqs) == 2
    else:
        # When disable chunked prefill, should not skip the long requests,
        # and scheduling subsequent short requests in advance,
        # even though there is still token budgets remaining.
        assert len(scheduler_output1.scheduled_new_reqs) == 1
```
**EN:** Parameterized test covering `schedule order`. Parameter axes: `enable_chunked_prefill`. Inputs/fixtures: `enable_chunked_prefill`. It exercises `mark.parametrize, create_scheduler, create_requests, scheduler.schedule, scheduler.add_request, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule order` 的测试用例。 参数维度：`enable_chunked_prefill`。 输入或 fixture：`enable_chunked_prefill`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, scheduler.schedule, scheduler.add_request, len`。 代码主体包含 2 个显式断言。

### test_preempt_during_execution (lines 687-743)
```python
def test_preempt_during_execution():
    # NOTE(woosuk): The actual number of available blocks is 10 instead of 11
    # because block 0 is reserved as the null block.
    scheduler = create_scheduler(
        max_num_batched_tokens=100,
        block_size=16,
        num_blocks=11,
        enable_prefix_caching=False,
    )
    requests = create_requests(num_requests=2, num_tokens=80, block_size=16)

    # Schedule the first request.
    scheduler.add_request(requests[0])
    scheduler_output0 = scheduler.schedule()
    assert len(scheduler_output0.num_scheduled_tokens) == 1
    assert len(scheduler_output0.scheduled_new_reqs[0].block_ids[0]) == 5
    # Schedule the second request while the first request is still running.
    # ... excerpt omitted for brevity ...
    assert len(scheduler_output1.num_scheduled_tokens) == 1
    assert len(scheduler_output1.scheduled_new_reqs[0].block_ids[0]) == 5
    assert len(scheduler.running) == 1
    assert scheduler.running[0] == requests[0]
    assert requests[1].status == RequestStatus.PREEMPTED
    scheduler.update_from_output(scheduler_output1, model_runner_output1)
    # The second request (that is preempted) should be updated with the
    # sampled token id.
    assert len(requests[1].output_token_ids) == 1
    assert requests[1].output_token_ids[0] == 42
```
**EN:** Test case covering `preempt during execution`. It exercises `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `preempt during execution` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`。 代码主体包含 9 个显式断言。

### test_scheduler_reset_prefix_cache (lines 746-774)
```python
def test_scheduler_reset_prefix_cache():
    scheduler = create_scheduler(enable_prefix_caching=True)
    requests = create_requests(num_requests=10)
    for request in requests:
        scheduler.add_request(request)

    # Initial scheduling, requests should be at the running state now
    _ = scheduler.schedule()

    # Verify requests moved from waiting to running
    assert len(scheduler.waiting) == 0
    assert len(scheduler.running) == len(requests)
    for i, request in enumerate(requests):
        assert scheduler.running[i] == request

    # Reset prefix cache should fail since there are still running requests
    # and they are taking KV cache
    assert not scheduler.reset_prefix_cache()

    # Reset prefix cache with reset_running_requests=True. All running requests
    # Should be pushed back to the waiting queue and kv cache should be freed
    assert scheduler.reset_prefix_cache(reset_running_requests=True)

    # Verify requests moved from running to waiting
    assert len(scheduler.waiting) == len(requests)
    assert len(scheduler.running) == 0

    for i, request in enumerate(requests):
        assert scheduler.waiting[i] == request
```
**EN:** Test case covering `scheduler reset prefix cache`. It exercises `create_scheduler, create_requests, scheduler.schedule, enumerate, scheduler.reset_prefix_cache, scheduler.add_request`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `调度器 reset prefix 缓存` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.schedule, enumerate, scheduler.reset_prefix_cache, scheduler.add_request`。 代码主体包含 8 个显式断言。

### test_schedule_spec_decoding_stats (lines 793-885)
```python
def test_schedule_spec_decoding_stats(spec_tokens, output_tokens, expected):
    """Test scheduling behavior with speculative decoding.

    This test verifies that:
    1. Speculated tokens get scheduled correctly
    2. Spec decoding stats properly count number of draft and accepted tokens
    """
    num_spec_tokens = max(1, max(len(t) for t in spec_tokens))
    scheduler = create_scheduler(num_speculative_tokens=num_spec_tokens)
    requests = create_requests(num_requests=len(spec_tokens), num_tokens=1)
    req_ids = []
    req_to_index = {}
    for i, request in enumerate(requests):
        scheduler.add_request(request)
        req_ids.append(request.request_id)
        req_to_index[request.request_id] = i
    # Schedule a decode, which will also draft speculative tokens
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == len(requests)
    assert output.total_num_scheduled_tokens == len(requests)
        assert output.num_scheduled_tokens[req_id] == 1
        assert req_id not in output.scheduled_spec_decode_tokens
        assert running_req.num_computed_tokens == 1
        assert running_req.num_tokens == 2
        assert scheduler_stats.spec_decoding_stats is not None
        stats = scheduler_stats.spec_decoding_stats
        assert stats.num_drafts == expected[0]
        assert stats.num_draft_tokens == expected[1]
        assert stats.num_accepted_tokens == expected[2]
        assert stats.num_accepted_tokens_per_pos == expected[3]
```
**EN:** Parameterized test covering `schedule spec decoding stats`. Parameter axes: `spec_tokens, output_tokens, expected`. Inputs/fixtures: `spec_tokens, output_tokens, expected`. It exercises `mark.parametrize, max, create_scheduler, create_requests, enumerate, scheduler.schedule`. The body contains 21 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule spec decoding stats` 的测试用例。 参数维度：`spec_tokens, output_tokens, expected`。 输入或 fixture：`spec_tokens, output_tokens, expected`。 该测试会调用 `mark.parametrize, max, create_scheduler, create_requests, enumerate, scheduler.schedule`。 代码主体包含 21 个显式断言。

### test_spec_decoding_stats_empty_output (lines 888-945)
```python
def test_spec_decoding_stats_empty_output():
    """Test that spec decoding stats handle empty output tokens gracefully.

    This is a regression test for a bug where empty sampled_token_ids
    would cause num_accepted = len([]) - 1 = -1, leading to a
    ValueError when incrementing a Prometheus counter with a negative value.
    """
    num_spec_tokens = 3
    scheduler = create_scheduler(num_speculative_tokens=num_spec_tokens)
    requests = create_requests(num_requests=1, num_tokens=1)
    request = requests[0]
    req_id = request.request_id
    scheduler.add_request(request)
    # Initial schedule (prefill)
    output = scheduler.schedule()
    assert len(output.scheduled_new_reqs) == 1
    # ... excerpt omitted for brevity ...
    assert req_id in output.scheduled_spec_decode_tokens
    assert len(output.scheduled_spec_decode_tokens[req_id]) == 3
    # Spec decoding stats should be None since no tokens were generated
    scheduler_stats = (
        engine_core_outputs[0].scheduler_stats if engine_core_outputs else None
    )
    assert scheduler_stats is None or scheduler_stats.spec_decoding_stats is None
```
**EN:** Test case covering `spec decoding stats empty output`. It exercises `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `spec decoding stats empty output` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`。 代码主体包含 4 个显式断言。

### test_no_spec_tokens_scheduled_for_prefill_chunks (lines 948-1039)
```python
def test_no_spec_tokens_scheduled_for_prefill_chunks():
    """Test that draft tokens are ignored for prefill chunk requests.

    When a request is being prefilled in chunks (chunked prefill), draft tokens
    from `update_draft_token_ids` should be ignored until the prefill is complete.
    The bug manifests when:
    - A prefill chunk is scheduled
    - Draft tokens are provided via update_draft_token_ids
    - The next schedule has enough budget to include spec tokens
    Without the fix, spec tokens would incorrectly be scheduled with the
    remaining prefill tokens. With the fix, draft tokens are ignored for
    prefill chunks.
    """
    num_spec_tokens = 3
    # Use budget of 50, with 80 token prompt:
    # - First chunk: 50 tokens
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 1
    assert output.num_scheduled_tokens[req.request_id] == 50
    assert output.num_scheduled_tokens[req.request_id] == 30, (
    assert req.request_id not in output.scheduled_spec_decode_tokens, (
    assert req.spec_token_ids == [1, 2, 3], (
    # Third schedule - decode phase with spec tokens
    output = scheduler.schedule()
    # 1 new token + 3 spec tokens = 4
    assert output.num_scheduled_tokens[req.request_id] == 4
    assert req.request_id in output.scheduled_spec_decode_tokens
    assert len(output.scheduled_spec_decode_tokens[req.request_id]) == num_spec_tokens
```
**EN:** Test case covering `no spec tokens scheduled for prefill chunks`. It exercises `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `no spec tokens scheduled for prefill chunks` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`。 代码主体包含 8 个显式断言。

### test_scheduler_stats_waiting_queues (lines 1042-1087)
```python
def test_scheduler_stats_waiting_queues():
    """Test that scheduler stats correctly report waiting and skipped_waiting queues."""
    # Create scheduler with limited capacity so we can have waiting requests
    scheduler = create_scheduler(max_num_batched_tokens=100)

    # Create requests: some will be scheduled, some will wait on capacity,
    # and some will be blocked by constraints
    all_requests = create_requests(num_requests=5, num_tokens=50)
    # Add 3 requests - only 2 can be scheduled (2 * 50 = 100 tokens)
    # The 3rd will remain in waiting queue (capacity constraint)
    for request in all_requests[:3]:
        scheduler.add_request(request)
    # Manually add 2 more to skipped_waiting to simulate constraint-blocked
    for request in all_requests[3:]:
        request.status = RequestStatus.WAITING_FOR_REMOTE_KVS
        scheduler.skipped_waiting.add_request(request)
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 2
    assert len(scheduler.waiting) == 1
    assert len(scheduler.skipped_waiting) == 2
    assert engine_core_outputs and len(engine_core_outputs) > 0
    assert stats is not None
    # Verify stats match queue lengths after scheduling
    assert stats.num_running_reqs == 2  # 2 were scheduled
    assert stats.num_waiting_reqs == 1  # 1 waiting on capacity
    assert stats.num_skipped_waiting_reqs == 2  # 2 blocked by constraints
```
**EN:** Test case covering `scheduler stats waiting queues`. It exercises `create_scheduler, create_requests, scheduler.schedule, list, ModelRunnerOutput, scheduler.update_from_output`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `调度器 stats waiting queues` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.schedule, list, ModelRunnerOutput, scheduler.update_from_output`。 代码主体包含 8 个显式断言。

### _assert_right_scheduler_output (lines 1090-1102)
```python
def _assert_right_scheduler_output(
    output: SchedulerOutput,
    num_requests: int,
    expected_num_scheduled_tokens: int,
):
    """Check if SchedulerOutput is correct after remote KV cache hit."""

    # We should inject the kv_connector_metadata.
    assert len(output.kv_connector_metadata.requests) == num_requests

    # Only num_tokens - matched_num_new_tokens should be scheduled.
    for _, num_scheduled_tokens in output.num_scheduled_tokens.items():
        assert num_scheduled_tokens == expected_num_scheduled_tokens
```
**EN:** Helper function `_assert_right_scheduler_output` encapsulates reusable logic for `assert right scheduler output`. Inputs: `output, num_requests, expected_num_scheduled_tokens`. Key calls include `num_scheduled_tokens.items, len`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_assert_right_scheduler_output` 封装了与 `assert right 调度器 output` 相关的可复用逻辑。 输入参数：`output, num_requests, expected_num_scheduled_tokens`。 关键调用包括 `num_scheduled_tokens.items, len`。 其中包含 2 个内部断言，用于保护前置假设。

### _assert_right_kv_cache_manager (lines 1105-1136)
```python
def _assert_right_kv_cache_manager(
    scheduler: Scheduler,
    requests: list[Request],
    num_tokens: int,
    block_size: int,
    num_requests: int,
    num_total_blocks: int,
):
    """Check whether KVCacheManager is correct after allocate."""

    # Make sure the request stats are right.
    EXPECTED_TOTAL_BLOCKS = num_tokens // block_size
    for req in requests:
        blocks = scheduler.kv_cache_manager.coordinator.single_type_managers[
            0
        ].req_to_blocks[req.request_id]
        hashes = req.block_hashes
        assert (
            scheduler.kv_cache_manager.coordinator.single_type_managers[
                0
            ].num_cached_block[req.request_id]
            == EXPECTED_TOTAL_BLOCKS
        )
        assert len(blocks) == EXPECTED_TOTAL_BLOCKS
        assert len(hashes) == EXPECTED_TOTAL_BLOCKS

    # Make sure we actually touched all the blocks.
    BLOCKS_PER_REQ = num_tokens / block_size
    assert (
        scheduler.kv_cache_manager.block_pool.get_num_free_blocks()
        == num_total_blocks - num_requests * BLOCKS_PER_REQ
    )
```
**EN:** Helper function `_assert_right_kv_cache_manager` encapsulates reusable logic for `assert right KV cache manager`. Inputs: `scheduler, requests, num_tokens, block_size, num_requests, num_total_blocks`. Key calls include `block_pool.get_num_free_blocks, len`. It includes 4 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_assert_right_kv_cache_manager` 封装了与 `assert right KV 缓存 manager` 相关的可复用逻辑。 输入参数：`scheduler, requests, num_tokens, block_size, num_requests, num_total_blocks`。 关键调用包括 `block_pool.get_num_free_blocks, len`。 其中包含 4 个内部断言，用于保护前置假设。

### _step_until_done (lines 1139-1164)
```python
def _step_until_done(
    scheduler: Scheduler,
    output: SchedulerOutput,
    model_runner_output: ModelRunnerOutput,
):
    """Loop over schedule(), update_from_output() until finished."""

    all_finished = False
    _ = scheduler.update_from_output(output, model_runner_output)
    while not all_finished:
        # Schedule + a few iterations until stopping.
        output = scheduler.schedule()
        assert len(scheduler.running)
        for _, num_scheduled_tokens in output.num_scheduled_tokens.items():
            # We should be in the decode phase now.
            assert num_scheduled_tokens == 1
        if scheduler.connector is not None:
            assert len(output.kv_connector_metadata.requests) == 0
        if scheduler.ec_connector is not None:
            assert len(output.ec_connector_metadata.mm_datas) == 0
        ecos = scheduler.update_from_output(output, model_runner_output)[0]
        all_done = True
        for eco in ecos.outputs:
            if eco.finish_reason is None:
                all_done = False
        all_finished = all_done
```
**EN:** Helper function `_step_until_done` encapsulates reusable logic for `step until done`. Inputs: `scheduler, output, model_runner_output`. Key calls include `scheduler.update_from_output, scheduler.schedule, len, num_scheduled_tokens.items`. It includes 4 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_step_until_done` 封装了与 `step until done` 相关的可复用逻辑。 输入参数：`scheduler, output, model_runner_output`。 关键调用包括 `scheduler.update_from_output, scheduler.schedule, len, num_scheduled_tokens.items`。 其中包含 4 个内部断言，用于保护前置假设。

### _num_waiting_requests (lines 1167-1168)
```python
def _num_waiting_requests(scheduler: Scheduler) -> int:
    return len(scheduler.waiting) + len(scheduler.skipped_waiting)
```
**EN:** Helper function `_num_waiting_requests` encapsulates reusable logic for `num waiting requests`. Inputs: `scheduler`. Key calls include `len`.
**CN:** 辅助函数 `_num_waiting_requests` 封装了与 `num waiting requests` 相关的可复用逻辑。 输入参数：`scheduler`。 关键调用包括 `len`。

### _step_until_kv_transfer_finished (lines 1171-1211)
```python
def _step_until_kv_transfer_finished(scheduler: Scheduler, req_ids: list[str]):
    """Cycle requests through a KV transfer cycle."""

    # Requests should first transition to WAITING_FOR_REMOTE_KVS
    output = scheduler.schedule()
    assert _num_waiting_requests(scheduler) == len(req_ids)
    assert len(scheduler.running) == 0
    assert len(output.scheduled_new_reqs) == 0
    for req in scheduler.requests.values():
        assert req.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    # No model execution yet
    EMPTY_OUTPUT = ModelRunnerOutput(
        req_ids=[],
        req_id_to_index={},
        sampled_token_ids=[],
        logprobs=None,
        prompt_logprobs_dict={},
    # ... excerpt omitted for brevity ...
    )
    scheduler.update_from_output(output, MODEL_RUNNER_OUTPUT)
    for req_id in req_ids:
        assert req_id in scheduler.finished_recving_kv_req_ids
    return initial_ecos
```
**EN:** Helper function `_step_until_kv_transfer_finished` encapsulates reusable logic for `step until KV transfer finished`. Inputs: `scheduler, req_ids`. Key calls include `scheduler.schedule, requests.values, ModelRunnerOutput, scheduler.update_from_output, _num_waiting_requests, len`. It includes 7 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_step_until_kv_transfer_finished` 封装了与 `step until kv transfer finished` 相关的可复用逻辑。 输入参数：`scheduler, req_ids`。 关键调用包括 `scheduler.schedule, requests.values, ModelRunnerOutput, scheduler.update_from_output, _num_waiting_requests, len`。 其中包含 7 个内部断言，用于保护前置假设。

### test_kv_connector_basic (lines 1215-1339)
```python
def test_kv_connector_basic(is_async: bool):
    """
    Test whether Scheduler with KVConnector schedules tokens, allocates
    memory, and cleans up requests as expected under normal operation.

    # Setup Scheduler.
    BLOCK_SIZE = 16
    NUM_MATCHED_NEW_TOKENS = BLOCK_SIZE * 2
    scheduler = create_scheduler(
        enable_prefix_caching=True,
        use_kv_connector=mock_kv(
            matched_tokens=NUM_MATCHED_NEW_TOKENS, is_async=is_async
        ),
        block_size=BLOCK_SIZE,
    )
    NUM_TOTAL_BLOCKS = scheduler.kv_cache_manager.block_pool.get_num_free_blocks()
    # ... excerpt omitted for brevity ...
        expected_num_scheduled_tokens=NUM_TOKENS - NUM_MATCHED_NEW_TOKENS,
    assert (
        expected_num_scheduled_tokens=(
    _step_until_done(scheduler, output, MODEL_RUNNER_OUTPUT)
    _ = scheduler.schedule()
    # Confirm we clean up the memory properly.
        scheduler.kv_cache_manager.block_pool.get_num_free_blocks() == NUM_TOTAL_BLOCKS
```
**EN:** Parameterized test covering `KV connector basic`. Parameter axes: `is_async`. Inputs/fixtures: `is_async`. It exercises `mark.parametrize, create_scheduler, block_pool.get_num_free_blocks, create_requests, enumerate, ModelRunnerOutput`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 连接器 basic` 的测试用例。 参数维度：`is_async`。 输入或 fixture：`is_async`。 该测试会调用 `mark.parametrize, create_scheduler, block_pool.get_num_free_blocks, create_requests, enumerate, ModelRunnerOutput`。 代码主体包含 2 个显式断言。

### test_external_prefix_cache_metrics (lines 1344-1458)
```python
def test_external_prefix_cache_metrics(is_async: bool, local_cache_hits: bool):
    """
    Verify connector prefix cache metrics are updated
    correctly when the scheduler processes requests with KV connector hits.

    BLOCK_SIZE = 16
    if local_cache_hits:
        NUM_MATCHED_NEW_TOKENS = BLOCK_SIZE * 2  # 32 tokens
        NUM_LOCAL_HITS = NUM_MATCHED_NEW_TOKENS * 2  # 64 tokens
        NUM_REQUESTS = 1
        NUM_TOKENS = NUM_LOCAL_HITS * 2  # 128 tokens
    else:
        NUM_MATCHED_NEW_TOKENS = 4
        NUM_LOCAL_HITS = 0
        NUM_REQUESTS = 2
        NUM_TOKENS = 8  # 8 tokens
    # ... excerpt omitted for brevity ...
    assert ecos is not None and len(ecos) > 0
    assert ecos[0].scheduler_stats is not None
        assert local_stats is not None
        assert local_stats.queries == NUM_TOKENS * NUM_REQUESTS
        assert local_stats.hits == NUM_LOCAL_HITS * NUM_REQUESTS
    assert external_stats is not None
    assert external_stats.queries == (NUM_TOKENS - NUM_LOCAL_HITS) * NUM_REQUESTS
    assert external_stats.hits == NUM_MATCHED_NEW_TOKENS * NUM_REQUESTS
    assert external_stats.requests == NUM_REQUESTS
    assert external_stats.preempted_requests == 0
```
**EN:** Parameterized test covering `external prefix cache metrics`. Parameter axes: `is_async, local_cache_hits`. Inputs/fixtures: `is_async, local_cache_hits`. It exercises `mark.parametrize, create_scheduler, create_requests, enumerate, scheduler.schedule, ModelRunnerOutput`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `external prefix 缓存 metrics` 的测试用例。 参数维度：`is_async, local_cache_hits`。 输入或 fixture：`is_async, local_cache_hits`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, enumerate, scheduler.schedule, ModelRunnerOutput`。 代码主体包含 10 个显式断言。

### test_kv_connector_unable_to_allocate (lines 1464-1541)
```python
def test_kv_connector_unable_to_allocate(use_ec_connector, ec_role):
    """
    Test whether scheduler with KVConnector is able to handle
    unable to allocate (run out of blocks in allocate_slots().

    # Setup Scheduler With Mock External Cache Hit.
    BLOCK_SIZE = 4
    NUM_BLOCKS = 10
    NUM_MATCHED_NEW_TOKENS = BLOCK_SIZE * 2
    scheduler = create_scheduler(
        enable_prefix_caching=True,
        use_kv_connector=mock_kv(matched_tokens=NUM_MATCHED_NEW_TOKENS, is_async=False),
        block_size=BLOCK_SIZE,
        num_blocks=NUM_BLOCKS,
        # encoder connector should not affect test results
        use_ec_connector=use_ec_connector,
        ec_role=ec_role,
    # ... excerpt omitted for brevity ...
        expected_num_scheduled_tokens=NUM_TOKENS - NUM_MATCHED_NEW_TOKENS,
    assert len(scheduler.running) == 1
    assert len(scheduler.waiting) == 1
    assert scheduler.kv_cache_manager.block_pool.get_num_free_blocks() == NUM_BLOCKS - 1
    assert len(scheduler.running) == 0
    # All memory should be freed, with no requests waiting / running.
    _step_until_done(scheduler, output, MODEL_RUNNER_OUTPUT)
    assert len(scheduler.waiting) == 0
```
**EN:** Parameterized test covering `KV connector unable to allocate`. Parameter axes: `use_ec_connector, ec_role`. Inputs/fixtures: `use_ec_connector, ec_role`. It exercises `mark.parametrize, create_scheduler, create_requests, enumerate, ModelRunnerOutput, scheduler.schedule`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 连接器 unable to allocate` 的测试用例。 参数维度：`use_ec_connector, ec_role`。 输入或 fixture：`use_ec_connector, ec_role`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, enumerate, ModelRunnerOutput, scheduler.schedule`。 代码主体包含 10 个显式断言。

### test_kv_connector_handles_preemption (lines 1548-1699)
```python
def test_kv_connector_handles_preemption(is_async, use_ec_connector, ec_role):
    """
    Test whether scheduler with KVConnector is able to handle
    unable to allocate (run out of blocks in allocate_slots().

    # Setup Scheduler With Mock External Cache Hit.
    BLOCK_SIZE = 2
    # NOTE: there is 1 null block, so this is 6 blocks.
    NUM_BLOCKS = 7
    NUM_MATCHED_NEW_TOKENS = BLOCK_SIZE
    scheduler = create_scheduler(
        enable_prefix_caching=True,
        use_kv_connector=mock_kv(
            matched_tokens=NUM_MATCHED_NEW_TOKENS, is_async=is_async
        ),
        block_size=BLOCK_SIZE,
        num_blocks=NUM_BLOCKS,
    # ... excerpt omitted for brevity ...
        assert _num_waiting_requests(scheduler) == 2
        assert scheduler.running == []
        expected_num_scheduled_tokens=NUM_TOKENS - NUM_MATCHED_NEW_TOKENS,
    assert len(scheduler.running) == 2
        expected_num_scheduled_tokens=1,
    assert output.scheduled_new_reqs == []
    assert len(scheduler.running) == 1
    _ = scheduler.update_from_output(output, MODEL_RUNNER_OUTPUT)
    assert len(scheduler.running) == 0
    # All memory should be freed since nothing is running.
    assert scheduler.kv_cache_manager.block_pool.get_num_free_blocks() == NUM_BLOCKS - 1
```
**EN:** Parameterized test covering `KV connector handles preemption`. Parameter axes: `is_async, use_ec_connector, ec_role`. Inputs/fixtures: `is_async, use_ec_connector, ec_role`. It exercises `mark.parametrize, create_scheduler, create_requests, enumerate, ModelRunnerOutput, scheduler.schedule`. The body contains 24 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 连接器 handles preemption` 的测试用例。 参数维度：`is_async, use_ec_connector, ec_role`。 输入或 fixture：`is_async, use_ec_connector, ec_role`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, enumerate, ModelRunnerOutput, scheduler.schedule`。 代码主体包含 24 个显式断言。

### make_output (lines 1702-1710)
```python
def make_output(scheduler: Scheduler):
    return ModelRunnerOutput(
        req_ids=[req.request_id for req in scheduler.running],
        req_id_to_index={req.request_id: i for i, req in enumerate(scheduler.running)},
        sampled_token_ids=[[1000]] * len(scheduler.running),
        logprobs=None,
        prompt_logprobs_dict={},
        pooler_output=[],
    )
```
**EN:** Helper function `make_output` encapsulates reusable logic for `output`. Inputs: `scheduler`. Key calls include `ModelRunnerOutput, len, enumerate`.
**CN:** 辅助函数 `make_output` 封装了与 `output` 相关的可复用逻辑。 输入参数：`scheduler`。 关键调用包括 `ModelRunnerOutput, len, enumerate`。

### assert_scheduler_empty (lines 1713-1748)
```python
def assert_scheduler_empty(scheduler: Scheduler):
    """Confirm the scheduler is "empty" - i.e. no leaks."""
    # Scheduler Metadata.
    assert len(scheduler.requests) == 0
    assert len(scheduler.waiting) == 0
    assert len(scheduler.running) == 0
    assert len(scheduler.finished_req_ids) == 0

    # EncoderCacheManager.
    assert len(scheduler.encoder_cache_manager.freed) == 0
    assert len(scheduler.encoder_cache_manager.cached) == 0
    # KVCache Manager.
    assert (
        len(
            scheduler.kv_cache_manager.coordinator.single_type_managers[0].req_to_blocks
        )
        == 0
    # ... excerpt omitted for brevity ...
    assert num_free_blocks == (scheduler.kv_cache_manager.block_pool.num_gpu_blocks - 1)
    # NOTE(rob): just the ref count on blocks will be 0. The hash
    # value, etc will remain since we lazily evict for prefix cache.
    for block in scheduler.kv_cache_manager.block_pool.blocks:
        assert block.ref_cnt == 0
```
**EN:** Helper function `assert_scheduler_empty` encapsulates reusable logic for `assert scheduler empty`. Inputs: `scheduler`. Key calls include `len`. It includes 10 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `assert_scheduler_empty` 封装了与 `assert 调度器 empty` 相关的可复用逻辑。 输入参数：`scheduler`。 关键调用包括 `len`。 其中包含 10 个内部断言，用于保护前置假设。

### test_memory_leak (lines 1755-1783)
```python
def test_memory_leak():
    """Test that we do not have a memory leak."""

    scheduler = create_scheduler(enable_prefix_caching=True)

    NUM_REQUESTS = 5
    NUM_TOKENS = 10
    MAX_TOKENS = 10
    requests = create_requests(
        num_requests=NUM_REQUESTS, num_tokens=NUM_TOKENS, max_tokens=MAX_TOKENS
    )

    # Add each request.
    for request in requests:
        scheduler.add_request(request)
        scheduler_output = scheduler.schedule()
        model_runner_output = make_output(scheduler)
        scheduler.update_from_output(scheduler_output, model_runner_output)

    # Iterate until done.
    while True:
        scheduler_output = scheduler.schedule()
        if len(scheduler.running) == 0:
            break
        model_runner_output = make_output(scheduler)
        scheduler.update_from_output(scheduler_output, model_runner_output)

    # Confirm no memory leak.
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `memory leak`. It exercises `create_scheduler, create_requests, assert_scheduler_empty, scheduler.add_request, scheduler.schedule, make_output`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `memory leak` 的测试用例。 该测试会调用 `create_scheduler, create_requests, assert_scheduler_empty, scheduler.add_request, scheduler.schedule, make_output`。 主要通过 mock、回调或输出检查来完成验证。

### create_scheduler_with_priority (lines 1786-1896)
```python
def create_scheduler_with_priority(
    model: str = "facebook/opt-125m",
    max_num_seqs: int = 16,
    max_num_batched_tokens: int = 8192,
    enable_prefix_caching: bool = False,
    long_prefill_token_threshold: int = 0,
    disable_chunked_mm_input: bool = False,
    use_kv_connector: bool = False,
    num_blocks: int = 10000,
    block_size: int = 16,
    max_model_len: int | None = None,
    num_speculative_tokens: int | None = None,
    use_ec_connector: bool = False,
    ec_role: str | None = None,
) -> Scheduler:
    """Create scheduler with priority policy enabled.

    Args:
    # ... excerpt omitted for brevity ...
    return Scheduler(
        kv_cache_config=kv_cache_config,
        log_stats=True,
        structured_output_manager=StructuredOutputManager(vllm_config),
        block_size=block_size,
        hash_block_size=block_size,
    )
```
**EN:** Helper function `create_scheduler_with_priority` encapsulates reusable logic for `scheduler with priority`. Inputs: `model, max_num_seqs, max_num_batched_tokens, enable_prefix_caching, long_prefill_token_threshold, disable_chunked_mm_input, use_kv_connector, num_blocks, ...`. Key calls include `ModelConfig, SchedulerConfig, CacheConfig, VllmConfig, KVCacheConfig, Scheduler`.
**CN:** 辅助函数 `create_scheduler_with_priority` 封装了与 `调度器 with priority` 相关的可复用逻辑。 输入参数：`model, max_num_seqs, max_num_batched_tokens, enable_prefix_caching, long_prefill_token_threshold, disable_chunked_mm_input, use_kv_connector, num_blocks, ...`。 关键调用包括 `ModelConfig, SchedulerConfig, CacheConfig, VllmConfig, KVCacheConfig, Scheduler`。

### Module state / 模块级状态 (line 1899)
```python
_none_hash_initialized = False
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_none_hash_initialized`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_none_hash_initialized`。

### create_requests_with_priority (lines 1902-2003)
```python
def create_requests_with_priority(
    num_requests: int,
    priorities: list[int],
    arrival_times: list[float] | None = None,
    num_tokens: int = 10,
    mm_hashes_list: list[list[str]] | None = None,
    mm_positions: list[list[PlaceholderRange]] | None = None,
    max_tokens: int = 16,
    stop_token_ids: list[int] | None = None,
    prompt_logprobs: int | None = None,
    starting_idx: int = 0,
    same_prompt: bool = False,
    block_size: int = 16,
    req_ids: list[str] | None = None,
):
    """Create requests with specified priorities and arrival times."""
    assert len(priorities) == num_requests
    if arrival_times is not None:
    # ... excerpt omitted for brevity ...
        assert len(arrival_times) == num_requests
        assert mm_positions is not None, (
        assert len(mm_hashes_list) == len(mm_positions) == num_requests
        assert [len(h) for h in mm_hashes_list] == [len(p) for p in mm_positions]
        assert len(req_ids) == num_requests
                    assert seen_hashes[identifier] == position_length, (
            arrival_time=arrival_times[i],
            priority=priorities[i],
            block_hasher=block_hasher,
        )
        requests.append(request)
    return requests
```
**EN:** Helper function `create_requests_with_priority` encapsulates reusable logic for `requests with priority`. Inputs: `num_requests, priorities, arrival_times, num_tokens, mm_hashes_list, mm_positions, max_tokens, stop_token_ids, ...`. Key calls include `get_request_block_hasher, SamplingParams, sampling_params.update_from_generation_config, range, len, init_none_hash`. It includes 7 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `create_requests_with_priority` 封装了与 `requests with priority` 相关的可复用逻辑。 输入参数：`num_requests, priorities, arrival_times, num_tokens, mm_hashes_list, mm_positions, max_tokens, stop_token_ids, ...`。 关键调用包括 `get_request_block_hasher, SamplingParams, sampling_params.update_from_generation_config, range, len, init_none_hash`。 其中包含 7 个内部断言，用于保护前置假设。

### test_priority_scheduling_basic_ordering (lines 2006-2032)
```python
def test_priority_scheduling_basic_ordering():
    """Test that requests are scheduled in priority order
    (lower value = higher priority)."""
    scheduler = create_scheduler_with_priority()

    # Create requests with different priorities
    # Priority 0 (highest), 1, 2 (lowest)
    priorities = [2, 0, 1]  # Add in non-priority order
    arrival_times = [1.0, 2.0, 3.0]  # All different arrival times
    requests = create_requests_with_priority(
        num_requests=3, priorities=priorities, arrival_times=arrival_times
    )

    # Add requests in non-priority order
    for request in requests:
        scheduler.add_request(request)

    # Schedule and verify priority order
    output = scheduler.schedule()

    # Should schedule all requests since they fit in budget
    assert len(output.scheduled_new_reqs) == 3

    # Verify they are scheduled in priority order:
    # req_1 (priority 0), req_2 (priority 1), req_0 (priority 2)
    scheduled_req_ids = [req.req_id for req in output.scheduled_new_reqs]
    assert scheduled_req_ids == ["1", "2", "0"]
```
**EN:** Test case covering `priority scheduling basic ordering`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, scheduler.add_request, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling basic ordering` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, scheduler.add_request, len`。 代码主体包含 2 个显式断言。

### test_priority_scheduling_arrival_time_tiebreaker (lines 2035-2060)
```python
def test_priority_scheduling_arrival_time_tiebreaker():
    """Test that arrival time is used
    as tiebreaker when priorities are equal."""
    scheduler = create_scheduler_with_priority()

    # Create requests with same priority but different arrival times
    priorities = [1, 1, 1]  # All same priority
    arrival_times = [3.0, 1.0, 2.0]  # Different arrival times
    requests = create_requests_with_priority(
        num_requests=3, priorities=priorities, arrival_times=arrival_times
    )

    # Add requests in non-arrival order
    for request in requests:
        scheduler.add_request(request)

    # Schedule and verify arrival time order
    output = scheduler.schedule()

    # Should schedule all requests since they fit in budget
    assert len(output.scheduled_new_reqs) == 3

    # Verify they are scheduled in arrival time order:
    # req_1 (1.0), req_2 (2.0), req_0 (3.0)
    scheduled_req_ids = [req.req_id for req in output.scheduled_new_reqs]
    assert scheduled_req_ids == ["1", "2", "0"]
```
**EN:** Test case covering `priority scheduling arrival time tiebreaker`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, scheduler.add_request, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling arrival time tiebreaker` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, scheduler.add_request, len`。 代码主体包含 2 个显式断言。

### test_priority_scheduling_mixed_priority_and_arrival (lines 2063-2090)
```python
def test_priority_scheduling_mixed_priority_and_arrival():
    """Test priority scheduling with mixed priorities and arrival times."""
    scheduler = create_scheduler_with_priority()

    # Create requests with mixed priorities and arrival times
    priorities = [2, 1, 1, 0]  # Mixed priorities
    arrival_times = [1.0, 3.0, 2.0, 4.0]  # Mixed arrival times
    requests = create_requests_with_priority(
        num_requests=4, priorities=priorities, arrival_times=arrival_times
    )

    # Add requests
    for request in requests:
        scheduler.add_request(request)

    # Schedule and verify order
    output = scheduler.schedule()

    # Should schedule all requests since they fit in budget
    assert len(output.scheduled_new_reqs) == 4

    # Expected order:
    # 1. req_3 (priority 0, arrival 4.0)
    # 2. req_2 (priority 1, arrival 2.0) - earlier arrival than req_1
    # 3. req_1 (priority 1, arrival 3.0)
    # 4. req_0 (priority 2, arrival 1.0)
    scheduled_req_ids = [req.req_id for req in output.scheduled_new_reqs]
    assert scheduled_req_ids == ["3", "2", "1", "0"]
```
**EN:** Test case covering `priority scheduling mixed priority and arrival`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, scheduler.add_request, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling mixed priority and arrival` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, scheduler.add_request, len`。 代码主体包含 2 个显式断言。

### test_priority_scheduling_preemption (lines 2093-2189)
```python
def test_priority_scheduling_preemption():
    """Test that under KV block pressure the scheduler preempts the
    lowest-priority *running* request, not the highest-priority one.

    A low-priority request starts running first. Then a high-priority
    request arrives and is admitted to running.  When block pressure
    builds, the scheduler preempts the low-priority running request
    while keeping the high-priority one.
    Block math
    ----------
    block_size = 16, num_blocks = 6 (1 null → 5 usable).
    Phase 1: lo1 (priority 5, 32 tokens) → 2 blocks.  3 free.
             Decode → lo1 has 33 tokens (needs 3rd block on next schedule).
    Phase 2: hi1 (priority 0, 32 tokens) arrives.
             schedule() allocates lo1's 3rd block (3 used) and admits
             hi1 (2 blocks) → 5 used, 0 free. Both running.
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 1
    assert any(r.req_id == "hi1" for r in output.scheduled_new_reqs)
    assert len(scheduler.running) == 2
    assert lo1_req.status == RequestStatus.PREEMPTED, (
        "Expected low-priority 'lo1' to be preempted"
    )
    assert any(req.request_id == "hi1" for req in scheduler.running), (
        "High-priority 'hi1' should still be running"
```
**EN:** Test case covering `priority scheduling preemption`. It exercises `create_scheduler_with_priority, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output, any`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling preemption` 的测试用例。 该测试会调用 `create_scheduler_with_priority, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output, any`。 代码主体包含 5 个显式断言。

### test_priority_scheduling_no_preemption_when_space_available (lines 2192-2242)
```python
def test_priority_scheduling_no_preemption_when_space_available():
    """Test that preemption doesn't happen
    when there's space for new requests."""
    scheduler = create_scheduler_with_priority(
        max_num_seqs=3,  # Allow 3 concurrent requests
        max_num_batched_tokens=200,  # Sufficient token budget
    )

    # Add two low-priority running requests
    low_priority_requests = create_requests_with_priority(
        num_requests=2,
        priorities=[5, 5],
        arrival_times=[1.0, 2.0],
        num_tokens=30,
        req_ids=["lo1", "lo2"],
    for request in low_priority_requests:
    # ... excerpt omitted for brevity ...
    output = scheduler.schedule()
    # Should schedule the new request without preemption
    assert len(output.scheduled_new_reqs) == 1
    assert len(scheduler.running) == 3  # All three requests running
    assert len(scheduler.waiting) == 0  # No requests waiting
```
**EN:** Test case covering `priority scheduling no preemption when space available`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output, scheduler.add_request`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling no preemption when space available` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output, scheduler.add_request`。 代码主体包含 3 个显式断言。

### test_priority_scheduling_preemption_victim_selection (lines 2245-2284)
```python
def test_priority_scheduling_preemption_victim_selection():
    """Test that the correct victim is selected for
    preemption based on priority and arrival time."""
    # This test verifies the priority-based victim selection logic
    # by checking the waiting queue order after adding requests with different
    # priorities
    scheduler = create_scheduler_with_priority(
        max_num_seqs=1,  # Force sequential processing to test priority order
    )

    # Create requests with different priorities
    requests = create_requests_with_priority(
        num_requests=3,
        priorities=[3, 2, 0],  # Different priorities: low, medium, high
        arrival_times=[1.0, 2.0, 3.0],
        num_tokens=10,
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 1
    assert output.scheduled_new_reqs[0].req_id == "2"  # Highest priority
    assert len(scheduler.waiting) == 2
    waiting_priorities = [req.priority for req in waiting_requests]
    waiting_req_ids = [req.request_id for req in waiting_requests]
    # Should be req_1 (priority 2) then req_0 (priority 3)
    assert waiting_priorities == [2, 3]
    assert waiting_req_ids == ["1", "0"]
```
**EN:** Test case covering `priority scheduling preemption victim selection`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, list, scheduler.add_request, len`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling preemption victim selection` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, list, scheduler.add_request, len`。 代码主体包含 5 个显式断言。

### test_priority_scheduling_equal_priority_preemption (lines 2287-2323)
```python
def test_priority_scheduling_equal_priority_preemption():
    """Test arrival time tiebreaker when requests have equal priority."""
    # This test verifies that arrival time is used as a tiebreaker for equal
    # priorities
    scheduler = create_scheduler_with_priority(
        max_num_seqs=1,  # Force sequential processing
    )

    # Create requests with same priority but different arrival times
    requests = create_requests_with_priority(
        num_requests=3,
        priorities=[2, 2, 2],  # Same priority
        arrival_times=[3.0, 1.0, 2.0],  # Different arrival times
        num_tokens=10,
    # Add all requests
    for request in requests:
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 1
    assert output.scheduled_new_reqs[0].req_id == "1"  # Earliest arrival (1.0)
    assert len(scheduler.waiting) == 2
    waiting_arrival_times = [req.arrival_time for req in waiting_requests]
    waiting_req_ids = [req.request_id for req in waiting_requests]
    # Should be req_2 (arrival 2.0) then req_0 (arrival 3.0)
    assert waiting_arrival_times == [2.0, 3.0]
    assert waiting_req_ids == ["2", "0"]
```
**EN:** Test case covering `priority scheduling equal priority preemption`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, list, scheduler.add_request, len`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling equal priority preemption` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, list, scheduler.add_request, len`。 代码主体包含 5 个显式断言。

### test_priority_scheduling_waiting_queue_order (lines 2326-2362)
```python
def test_priority_scheduling_waiting_queue_order():
    """Test that the waiting queue maintains priority order."""
    scheduler = create_scheduler_with_priority(
        max_num_seqs=1,  # Only one request can run at a time
    )

    # Create multiple requests with different priorities
    requests = create_requests_with_priority(
        num_requests=4,
        priorities=[3, 1, 2, 0],  # Mixed priorities
        arrival_times=[1.0, 2.0, 3.0, 4.0],
        num_tokens=10,
    # Add all requests
    for request in requests:
        scheduler.add_request(request)
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 1
    assert output.scheduled_new_reqs[0].req_id == "3"
    assert len(scheduler.waiting) == 3
    waiting_priorities = [req.priority for req in waiting_requests]
    waiting_req_ids = [req.request_id for req in waiting_requests]
    # Should be ordered by priority: req_1 (1), req_2 (2), req_0 (3)
    assert waiting_req_ids == ["1", "2", "0"]
    assert waiting_priorities == [1, 2, 3]
```
**EN:** Test case covering `priority scheduling waiting queue order`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, list, scheduler.add_request, len`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling waiting queue order` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, list, scheduler.add_request, len`。 代码主体包含 5 个显式断言。

### test_priority_scheduling_fcfs_fallback (lines 2365-2390)
```python
def test_priority_scheduling_fcfs_fallback():
    """Test that FCFS behavior is maintained when all
    requests have same priority."""
    scheduler = create_scheduler_with_priority()

    # Create requests with same priority but different arrival times
    priorities = [1, 1, 1, 1]  # All same priority
    arrival_times = [4.0, 1.0, 3.0, 2.0]  # Different arrival times
    requests = create_requests_with_priority(
        num_requests=4, priorities=priorities, arrival_times=arrival_times
    )

    # Add requests
    for request in requests:
        scheduler.add_request(request)

    # Schedule
    output = scheduler.schedule()

    # Should schedule all requests in arrival time order
    assert len(output.scheduled_new_reqs) == 4
    scheduled_req_ids = [req.req_id for req in output.scheduled_new_reqs]

    # Expected order by arrival time:
    # req_1 (1.0), req_3 (2.0), req_2 (3.0), req_0 (4.0)
    assert scheduled_req_ids == ["1", "3", "2", "0"]
```
**EN:** Test case covering `priority scheduling fcfs fallback`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, scheduler.add_request, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling fcfs fallback` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, scheduler.add_request, len`。 代码主体包含 2 个显式断言。

### test_priority_scheduling_with_limited_slots (lines 2393-2431)
```python
def test_priority_scheduling_with_limited_slots():
    """Test priority scheduling when max_num_seqs limits concurrent requests."""
    scheduler = create_scheduler_with_priority(
        max_num_seqs=2,  # Only allow 2 concurrent requests
        max_num_batched_tokens=1000,  # Plenty of token budget
    )

    # Create requests with different priorities
    requests = create_requests_with_priority(
        num_requests=4,
        priorities=[3, 1, 2, 0],  # Mixed priorities
        arrival_times=[1.0, 2.0, 3.0, 4.0],
        num_tokens=10,
    # Add all requests
    for request in requests:
        scheduler.add_request(request)
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 2
    assert "3" in scheduled_req_ids  # Priority 0
    assert "1" in scheduled_req_ids  # Priority 1
    assert len(scheduler.waiting) == 2
    waiting_priorities = [req.priority for req in waiting_requests]
    waiting_req_ids = [req.request_id for req in waiting_requests]
    # Should be req_2 (priority 2) then req_0 (priority 3)
    assert waiting_priorities == [2, 3]
    assert waiting_req_ids == ["2", "0"]
```
**EN:** Test case covering `priority scheduling with limited slots`. It exercises `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, list, scheduler.add_request, len`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling with limited slots` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, scheduler.schedule, list, scheduler.add_request, len`。 代码主体包含 6 个显式断言。

### test_priority_scheduling_heap_property (lines 2434-2480)
```python
def test_priority_scheduling_heap_property():
    """Test that the waiting queue maintains heap
    property for priority scheduling."""
    scheduler = create_scheduler_with_priority(
        max_num_seqs=1,  # Only one request can run at a time
    )

    # Add requests in random priority order
    priorities = [5, 1, 8, 3, 2, 7, 4, 6]
    arrival_times = [float(i) for i in range(len(priorities))]
    requests = create_requests_with_priority(
        num_requests=len(priorities),
        priorities=priorities,
        arrival_times=arrival_times,
        num_tokens=10,
    # Add all requests
    # ... excerpt omitted for brevity ...
            # Finish the request to make room for the next one
            scheduler.finish_requests(req.req_id, RequestStatus.FINISHED_STOPPED)
    # Verify requests were scheduled in priority order (lowest value first)
    expected_priorities = sorted(priorities)
    assert scheduled_priorities == expected_priorities
```
**EN:** Test case covering `priority scheduling heap property`. It exercises `create_scheduler_with_priority, create_requests_with_priority, sorted, float, scheduler.add_request, scheduler.schedule`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling heap property` 的测试用例。 该测试会调用 `create_scheduler_with_priority, create_requests_with_priority, sorted, float, scheduler.add_request, scheduler.schedule`。 代码主体包含 1 个显式断言。

### test_schedule_skip_tokenizer_init (lines 2483-2489)
```python
def test_schedule_skip_tokenizer_init():
    scheduler = create_scheduler(skip_tokenizer_init=True)
    requests = create_requests(num_requests=5)
    for request in requests:
        scheduler.add_request(request)
    output = scheduler.schedule()
    assert len(output.scheduled_new_reqs) == len(requests)
```
**EN:** Test case covering `schedule skip tokenizer init`. It exercises `create_scheduler, create_requests, scheduler.schedule, scheduler.add_request, len`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule skip 分词器 init` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.schedule, scheduler.add_request, len`。 代码主体包含 1 个显式断言。

### test_schedule_skip_tokenizer_init_structured_output_request (lines 2492-2513)
```python
def test_schedule_skip_tokenizer_init_structured_output_request():
    scheduler = create_scheduler(skip_tokenizer_init=True)
    structured_outputs_params = StructuredOutputsParams(regex="[0-9]+")
    sampling_params = SamplingParams(
        ignore_eos=False,
        max_tokens=16,
        structured_outputs=structured_outputs_params,
    )
    sampling_params.update_from_generation_config({}, EOS_TOKEN_ID)
    request = Request(
        request_id="0",
        prompt_token_ids=[0, 1],
        mm_features=None,
        sampling_params=sampling_params,
        pooling_params=None,
    )
    scheduler.add_request(request)
    output = scheduler.schedule()
    assert len(output.scheduled_new_reqs) == 0
    assert len(scheduler.running) == 0
    assert len(scheduler.waiting) == 0
    assert len(scheduler.skipped_waiting) == 1
```
**EN:** Test case covering `schedule skip tokenizer init structured output request`. It exercises `create_scheduler, StructuredOutputsParams, SamplingParams, sampling_params.update_from_generation_config, Request, scheduler.add_request`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule skip 分词器 init structured output request` 的测试用例。 该测试会调用 `create_scheduler, StructuredOutputsParams, SamplingParams, sampling_params.update_from_generation_config, Request, scheduler.add_request`。 代码主体包含 4 个显式断言。

### test_abort_request_when_structured_output_fsm_cannot_advance (lines 2516-2594)
```python
def test_abort_request_when_structured_output_fsm_cannot_advance():
    scheduler = object.__new__(Scheduler)
    sampling_params = SamplingParams(ignore_eos=True, max_tokens=4)
    sampling_params.update_from_generation_config({}, EOS_TOKEN_ID)

    request = Request(
        request_id="0",
        prompt_token_ids=[0, 1],
        mm_features=None,
        sampling_params=sampling_params,
        pooling_params=None,
    )
    request.structured_output_request = Mock()
    request.structured_output_request.grammar = Mock()
    request.structured_output_request.grammar.accept_tokens.return_value = False
    request.status = RequestStatus.RUNNING
    request.num_computed_tokens = request.num_tokens
    # ... excerpt omitted for brevity ...
    scheduler.structured_output_manager = Mock()
    scheduler.waiting = Mock()
    scheduler.kv_cache_manager = Mock()
    scheduler.kv_event_publisher = Mock()
    scheduler.vllm_config = Mock()
    scheduler.make_stats = Mock(return_value=None)
    scheduler._free_request.assert_called_once_with(request)
    assert len(engine_core_outputs[0].outputs) == 1
    engine_core_output = engine_core_outputs[0].outputs[0]
    assert engine_core_output.request_id == request.request_id
    assert engine_core_output.new_token_ids == [123]
    assert engine_core_output.finish_reason == FinishReason.ERROR
```
**EN:** Test case covering `abort request when structured output fsm cannot advance`. It exercises `object.__new__, SamplingParams, sampling_params.update_from_generation_config, Request, Mock, set`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort request when structured output fsm cannot advance` 的测试用例。 该测试会调用 `object.__new__, SamplingParams, sampling_params.update_from_generation_config, Request, Mock, set`。 代码主体包含 8 个显式断言。

### test_priority_scheduling_preemption_and_resumption_when_out_of_kv (lines 2600-2720)
```python
def test_priority_scheduling_preemption_and_resumption_when_out_of_kv(
    use_ec_connector, ec_role
):
    """Test that priority scheduling preempts lower priority requests
    when out of KV cache space."""
    # Create scheduler with very limited memory to force preemption
    scheduler = create_scheduler_with_priority(
        max_num_seqs=2,  # Allow multiple requests
        max_num_batched_tokens=200,
        num_blocks=5,  # Can hold 64 tokens (first block is null)
        block_size=16,  # Standard block size
        use_kv_connector=True,
        # encoder connector should not affect test results
        use_ec_connector=use_ec_connector,
        ec_role=ec_role,
    )

    # Create a request and schedule it
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 1
    assert len(scheduler.waiting) == 0
    assert len(scheduler.running) == 1
    assert scheduler.kv_cache_manager.block_pool.get_num_free_blocks() == 0
    assert output.scheduled_cached_reqs.num_reqs == 1
    assert scheduled_cached_reqs.req_ids[0] == request_low.request_id
    assert request_low.request_id in scheduled_cached_reqs.resumed_req_ids
    assert request_low.request_id in scheduled_cached_reqs.all_token_ids
    # Resumed tokens include 30 prompt tokens and 2 decoded tokens
    assert len(scheduled_cached_reqs.all_token_ids[request_low.request_id]) == 32
    assert scheduled_cached_reqs.all_token_ids[request_low.request_id][31] == 100
```
**EN:** Parameterized test covering `priority scheduling preemption and resumption when out of KV`. Parameter axes: `use_ec_connector, ec_role`. Inputs/fixtures: `use_ec_connector, ec_role`. It exercises `mark.parametrize, create_scheduler_with_priority, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`. The body contains 25 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling preemption and resumption when out of kv` 的测试用例。 参数维度：`use_ec_connector, ec_role`。 输入或 fixture：`use_ec_connector, ec_role`。 该测试会调用 `mark.parametrize, create_scheduler_with_priority, scheduler.add_request, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`。 代码主体包含 25 个显式断言。

### test_chunked_prefill_disabled_for_encoder_decoder (lines 2733-2759)
```python
def test_chunked_prefill_disabled_for_encoder_decoder(
    enable_chunked_prefill: bool, is_encoder_decoder: bool, expect_enabled: bool
) -> None:
    """Validate that chunked prefill is appropriately disabled for
    encoder-decoder models."""
    scheduler_config = SchedulerConfig(
        enable_chunked_prefill=enable_chunked_prefill,
        is_encoder_decoder=is_encoder_decoder,
        # Must <= max_num_batched_tokens if chunked prefill is disabled
        max_model_len=SchedulerConfig.DEFAULT_MAX_NUM_BATCHED_TOKENS,
    )

    # `is_encoder_decoder` should only be used during construction
    # of the config, and otherwise stored in the model config.
    assert "is_encoder_decoder" not in vars(scheduler_config)
    assert "is_encoder_decoder" not in [
        f.name for f in dataclasses.fields(scheduler_config)
    ]
    _validate_chunked_prefill_settings_for_encoder_decoder(
        scheduler_config, is_encoder_decoder, expect_enabled
    )

    # Ensure it is retained in VllmConfig, even after its post-init.
    vllm_config = VllmConfig(scheduler_config=scheduler_config)
    _validate_chunked_prefill_settings_for_encoder_decoder(
        vllm_config.scheduler_config, is_encoder_decoder, expect_enabled
    )
```
**EN:** Test case covering `chunked prefill disabled for encoder decoder`. Inputs/fixtures: `enable_chunked_prefill, is_encoder_decoder, expect_enabled`. It exercises `mark.parametrize, SchedulerConfig, _validate_chunked_prefill_settings_for_encoder_decoder, VllmConfig, vars, dataclasses.fields`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `chunked prefill disabled for encoder decoder` 的测试用例。 输入或 fixture：`enable_chunked_prefill, is_encoder_decoder, expect_enabled`。 该测试会调用 `mark.parametrize, SchedulerConfig, _validate_chunked_prefill_settings_for_encoder_decoder, VllmConfig, vars, dataclasses.fields`。 代码主体包含 2 个显式断言。

### _validate_chunked_prefill_settings_for_encoder_decoder (lines 2762-2773)
```python
def _validate_chunked_prefill_settings_for_encoder_decoder(
    scheduler_config: SchedulerConfig, is_encoder_decoder: bool, expect_enabled: bool
) -> None:
    """Validate chunked prefill settings in the scheduler config for
    encoder-decoder models."""
    assert scheduler_config.enable_chunked_prefill is expect_enabled
    if is_encoder_decoder:
        # Encoder-decoder models should automatically disable chunked multimodal
        # inputs as well
        assert scheduler_config.disable_chunked_mm_input is not expect_enabled
    if is_encoder_decoder and not expect_enabled:
        assert scheduler_config.long_prefill_token_threshold == 0
```
**EN:** Helper function `_validate_chunked_prefill_settings_for_encoder_decoder` encapsulates reusable logic for `validate chunked prefill settings for encoder decoder`. Inputs: `scheduler_config, is_encoder_decoder, expect_enabled`. It includes 3 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_validate_chunked_prefill_settings_for_encoder_decoder` 封装了与 `validate chunked prefill settings for encoder decoder` 相关的可复用逻辑。 输入参数：`scheduler_config, is_encoder_decoder, expect_enabled`。 其中包含 3 个内部断言，用于保护前置假设。

### _assert_right_encoder_cache_allocated (lines 2783-2819)
```python
def _assert_right_encoder_cache_allocated(
    scheduler: Scheduler,
    hashes_to_check: list[str] | None = None,
    requests: list[Request] | None = None,
    expected_total_allocated: int | None = None,
):
    """Check whether encoder cache is allocated correctly."""
    encoder_cache_manager = scheduler.encoder_cache_manager

    # Verify encoder cache manager exists
    assert encoder_cache_manager is not None, "Encoder cache manager should exist"
    # Verify number of cache
    if expected_total_allocated is not None:
        assert len(encoder_cache_manager.cached) == expected_total_allocated
        if expected_total_allocated == 0:
            return
    # ... excerpt omitted for brevity ...
        assert not missed_hashes, (
            req_hashes = set(mm_hashes)  # unique hashes set
            missed_hashes = req_hashes - cached_hashes
            assert not missed_hashes, (
                f"Miss hashes in cache for request {req.request_id}: {missed_hashes} "
                f"Existing encoder cache: {encoder_cache_manager.cached}"
            )
```
**EN:** Helper function `_assert_right_encoder_cache_allocated` encapsulates reusable logic for `assert right encoder cache allocated`. Inputs: `scheduler, hashes_to_check, requests, expected_total_allocated`. Key calls include `set, cached.keys, len`. It includes 4 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_assert_right_encoder_cache_allocated` 封装了与 `assert right encoder 缓存 allocated` 相关的可复用逻辑。 输入参数：`scheduler, hashes_to_check, requests, expected_total_allocated`。 关键调用包括 `set, cached.keys, len`。 其中包含 4 个内部断言，用于保护前置假设。

### _assert_right_ec_connector_metadata (lines 2822-2844)
```python
def _assert_right_ec_connector_metadata(
    output: SchedulerOutput,
    mm_features_list: list[MultiModalFeatureSpec],
):
    """Verify that ECConnector metadata EXACTLY matches the input MM data"""
    # Get the connector metadata
    metadata = output.ec_connector_metadata

    # Create lookup dictionaries for efficient access
    metadata_dict = {mm_data.mm_hash: mm_data for mm_data in metadata.mm_datas}

    # Check all required identifiers exist in metadata; and no extra
    # In ECExampleConnector format
    # NOTE: even having same identifier, the mm_features can be different
    # since their mm_position can be in different offsets, etc
    identifiers_dict = {f.identifier for f in mm_features_list}
    assert set(metadata_dict.keys()) == identifiers_dict

    # Verify the info matches
    for i, mm_feature in enumerate(mm_features_list):
        identifier = mm_feature.identifier
        assert metadata_dict[identifier].mm_hash == identifier
        assert metadata_dict[identifier].num_token == mm_feature.mm_position.length
```
**EN:** Helper function `_assert_right_ec_connector_metadata` encapsulates reusable logic for `assert right ec connector metadata`. Inputs: `output, mm_features_list`. Key calls include `enumerate, set, metadata_dict.keys`. It includes 3 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_assert_right_ec_connector_metadata` 封装了与 `assert right ec connector metadata` 相关的可复用逻辑。 输入参数：`output, mm_features_list`。 关键调用包括 `enumerate, set, metadata_dict.keys`。 其中包含 3 个内部断言，用于保护前置假设。

### _assert_right_encoder_inputs (lines 2847-2881)
```python
def _assert_right_encoder_inputs(
    output: SchedulerOutput,
    check_exist: bool | None = True,
    requests: list[Request] | None = None,
    expected_encoder_inputs: list[list[int]] | None = None,
    expected_total_reqs: int | None = None,
):
    """Verify that requests/mm_hashes should (not) in scheduled encoder input
    If check_exist is False, this function returns True
    if requests are NOT in encoder inputs"""

    # Get the scheduled encoder inputs
    # NOTE: scheduled_encoder_inputs is a dictionary with request id as key
    scheduled_encoder_inputs = output.scheduled_encoder_inputs
    # Check if scheduled_encoder_inputs is empty as expected
    if expected_total_reqs is not None:
        assert len(scheduled_encoder_inputs) == expected_total_reqs
    # ... excerpt omitted for brevity ...
        if expected_total_reqs == 0:
    # Number of expected encoder inputs should match number of requests
    if expected_encoder_inputs:
        assert check_exist and requests is not None  # only support expect input exist
        assert len(requests) == len(expected_encoder_inputs)
    # Check request (not) exist as expected
            f"Request {request.id} presence mismatch: expected {check_exist}, "
            f"got {request.id in scheduled_encoder_inputs}"
        )
        if expected_encoder_inputs:
            scheduled_encoder_input = scheduled_encoder_inputs[request.request_id]
            assert scheduled_encoder_input == expected_encoder_inputs[i]
```
**EN:** Helper function `_assert_right_encoder_inputs` encapsulates reusable logic for `assert right encoder inputs`. Inputs: `output, check_exist, requests, expected_encoder_inputs, expected_total_reqs`. Key calls include `enumerate, len`. It includes 5 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_assert_right_encoder_inputs` 封装了与 `assert right encoder inputs` 相关的可复用逻辑。 输入参数：`output, check_exist, requests, expected_encoder_inputs, expected_total_reqs`。 关键调用包括 `enumerate, len`。 其中包含 5 个内部断言，用于保护前置假设。

### test_scheduler_no_ec_connector_by_default (lines 2884-2887)
```python
def test_scheduler_no_ec_connector_by_default():
    """Test scheduler doesn't have EC connector by default."""
    scheduler = create_scheduler()
    assert scheduler.ec_connector is None
```
**EN:** Test case covering `scheduler no ec connector by default`. It exercises `create_scheduler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `调度器 no ec connector by default` 的测试用例。 该测试会调用 `create_scheduler`。 代码主体包含 1 个显式断言。

### test_ec_connector_text_only_request (lines 2891-2928)
```python
def test_ec_connector_text_only_request(use_kv_connector):
    """Test text-only requests don't allocate encoder cache."""
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        use_kv_connector=use_kv_connector,
        use_ec_connector=True,
        ec_role="ec_consumer",
    )

    NUM_PROMPT_TOKENS = 100
    # Create text-only request (no mm_positions)
    requests = create_requests(
        num_requests=1,
        num_tokens=NUM_PROMPT_TOKENS,
    assert not requests[0].mm_features  # No MM data
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 1
    assert scheduled == NUM_PROMPT_TOKENS, (
    _assert_right_encoder_cache_allocated(scheduler, expected_total_allocated=0)
    # ECConnector should carry no metadata
    _assert_right_ec_connector_metadata(output, mm_features_list=[])
    # Scheduled encoder input should be empty; no mm to compute
    _assert_right_encoder_inputs(output, expected_total_reqs=0)
```
**EN:** Parameterized test covering `ec connector text only request`. Parameter axes: `use_kv_connector`. Inputs/fixtures: `use_kv_connector`. It exercises `mark.parametrize, create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, _assert_right_encoder_cache_allocated`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `ec connector text only request` 的测试用例。 参数维度：`use_kv_connector`。 输入或 fixture：`use_kv_connector`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, _assert_right_encoder_cache_allocated`。 代码主体包含 3 个显式断言。

### test_ec_connector_cache_hit_external_load (lines 2932-2979)
```python
def test_ec_connector_cache_hit_external_load(use_kv_connector):
    """Test ec_consumer loads from external cache when hit.
    A normal basic operation for EPD disaggrgation"""
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        enable_prefix_caching=True,
        # kv connector should not effect test results
        use_kv_connector=use_kv_connector,
        use_ec_connector=True,
        ec_role="ec_consumer",
    )

    # Create MM request
    NUM_TOKENS = 200  # NOTE: includes mm tokens
    NUM_ENCODER_TOKENS = 100
    mm_hashes_list = [["hash_test1"]]
    mm_positions = [[PlaceholderRange(offset=0, length=NUM_ENCODER_TOKENS)]]
    # ... excerpt omitted for brevity ...
    scheduler.ec_connector.has_cache_item = Mock(return_value=True)
    scheduler.ec_connector.update_state_after_alloc = Mock(
    assert scheduled_tokens == NUM_TOKENS
    # ECConnector should carry metadata of request
    _assert_right_ec_connector_metadata(output, mm_features_list=request.mm_features)
    # Scheduled encoder input should be empty; no mm to compute
    _assert_right_encoder_inputs(output, expected_total_reqs=0)
```
**EN:** Parameterized test covering `ec connector cache hit external load`. Parameter axes: `use_kv_connector`. Inputs/fixtures: `use_kv_connector`. It exercises `mark.parametrize, create_scheduler, Mock, scheduler.add_request, scheduler.schedule, update_state_after_alloc.assert_called_with`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `ec connector 缓存 hit external load` 的测试用例。 参数维度：`use_kv_connector`。 输入或 fixture：`use_kv_connector`。 该测试会调用 `mark.parametrize, create_scheduler, Mock, scheduler.add_request, scheduler.schedule, update_state_after_alloc.assert_called_with`。 代码主体包含 1 个显式断言。

### test_ec_connector_cache_miss_computes_locally (lines 2983-3033)
```python
def test_ec_connector_cache_miss_computes_locally(use_kv_connector):
    """Test consumer can compute encoder locally when cache miss (fallback)."""
    # encoder cache itself if it doesn't receive it from external storage

    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        enable_prefix_caching=True,
        use_kv_connector=use_kv_connector,
        use_ec_connector=True,
        ec_role="ec_consumer",
    )
    # Verify consumer role
    assert scheduler.ec_connector is not None
    assert not scheduler.ec_connector.is_producer
    # Create MM request
    request_mm_missed = create_requests(
    # ... excerpt omitted for brevity ...
    scheduler.ec_connector.has_cache_item = Mock(return_value=False)
    assert len(output.scheduled_new_reqs) == 1
    assert scheduled_tokens == 200, (
    _assert_right_encoder_inputs(
        output,
        requests=[request_mm_missed],
        expected_encoder_inputs=[[0]],  # index 0 of the mm item
        expected_total_reqs=1,
```
**EN:** Parameterized test covering `ec connector cache miss computes locally`. Parameter axes: `use_kv_connector`. Inputs/fixtures: `use_kv_connector`. It exercises `mark.parametrize, create_scheduler, Mock, scheduler.add_request, scheduler.schedule, _assert_right_encoder_cache_allocated`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `ec connector 缓存 miss computes locally` 的测试用例。 参数维度：`use_kv_connector`。 输入或 fixture：`use_kv_connector`。 该测试会调用 `mark.parametrize, create_scheduler, Mock, scheduler.add_request, scheduler.schedule, _assert_right_encoder_cache_allocated`。 代码主体包含 4 个显式断言。

### test_ec_connector_with_partial_cache_hit_multi_round (lines 3039-3188)
```python
def test_ec_connector_with_partial_cache_hit_multi_round(use_kv_connector):
    """Test consumer with partial cache hit (local & connector) with 2 requests."""
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        enable_prefix_caching=True,
        use_kv_connector=use_kv_connector,
        use_ec_connector=True,
        ec_role="ec_consumer",
    )

    # Create MM request
    NUM_TOKENS_1 = 300  # NOTE: includes mm tokens
    NUM_ENCODER_TOKENS_1 = 50
    mm_hashes_list_1 = [["hash1_A", "hash1_B", "hash1_A", "hash1_F"]]
    mm_positions_1 = [
        [
            PlaceholderRange(offset=0, length=NUM_ENCODER_TOKENS_1),
            PlaceholderRange(offset=100, length=NUM_ENCODER_TOKENS_1),
    # ... excerpt omitted for brevity ...
    scheduler.ec_connector.has_cache_item = Mock(
    scheduler.ec_connector.update_state_after_alloc = Mock(
    assert scheduled_tokens == NUM_TOKENS_1
        expected_encoder_inputs=[[0]],  # index 0 of the mm item ONLY
        expected_total_reqs=1,
    _assert_right_encoder_inputs(
        output,
        requests=[request2],
        expected_encoder_inputs=[[1, 2]],
```
**EN:** Parameterized test covering `ec connector with partial cache hit multi round`. Parameter axes: `use_kv_connector`. Inputs/fixtures: `use_kv_connector`. It exercises `mark.parametrize, create_scheduler, Mock, scheduler.add_request, scheduler.schedule, _assert_right_encoder_cache_allocated`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `ec connector with partial 缓存 hit multi round` 的测试用例。 参数维度：`use_kv_connector`。 输入或 fixture：`use_kv_connector`。 该测试会调用 `mark.parametrize, create_scheduler, Mock, scheduler.add_request, scheduler.schedule, _assert_right_encoder_cache_allocated`。 代码主体包含 2 个显式断言。

### test_ec_connector_schedule_multiple_requests (lines 3193-3277)
```python
def test_ec_connector_schedule_multiple_requests(cache_exist, use_kv_connector):
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        max_num_seqs=10,  # allow multiple requests
        max_num_batched_tokens=2048,
        enable_prefix_caching=True,
        use_kv_connector=use_kv_connector,
        use_ec_connector=True,
        ec_role="ec_consumer",
    )
    mm_hashes_list = [[f"hash_{i}"] for i in range(10)]
    mm_positions = [[PlaceholderRange(offset=i, length=100)] for i in range(10)]
    requests = create_requests(
        num_requests=10,
        num_tokens=200,
        mm_hashes_list=mm_hashes_list,
        mm_positions=mm_positions,
    # ... excerpt omitted for brevity ...
    scheduler.ec_connector.update_state_after_alloc = Mock(
        scheduler.ec_connector.has_cache_item = Mock(return_value=True)
        scheduler.ec_connector.has_cache_item = Mock(return_value=False)
    assert len(output.scheduled_new_reqs) == len(requests)
    assert output.scheduled_cached_reqs.num_reqs == 0
    assert len(output.finished_req_ids) == 0
            requests=requests,
            expected_encoder_inputs=[[0] for _ in range(10)],
            expected_total_reqs=10,
        )
    else:
        _assert_right_encoder_inputs(output, expected_total_reqs=0)
```
**EN:** Parameterized test covering `ec connector schedule multiple requests`. Parameter axes: `cache_exist, use_kv_connector`. Inputs/fixtures: `cache_exist, use_kv_connector`. It exercises `mark.parametrize, create_scheduler, create_requests, Mock, scheduler.schedule, num_scheduled_tokens.items`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `ec connector schedule multiple requests` 的测试用例。 参数维度：`cache_exist, use_kv_connector`。 输入或 fixture：`cache_exist, use_kv_connector`。 该测试会调用 `mark.parametrize, create_scheduler, create_requests, Mock, scheduler.schedule, num_scheduled_tokens.items`。 代码主体包含 5 个显式断言。

### test_ec_connector_unable_to_allocate (lines 3281-3374)
```python
def test_ec_connector_unable_to_allocate(use_kv_connector):
    """
    Test whether scheduler with ECConnector is able to handle
    unable to allocate (run out of blocks).

    # Setup Scheduler With Mock External Cache Hit.
    BLOCK_SIZE = 4
    NUM_BLOCKS = 10
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        enable_prefix_caching=True,
        use_kv_connector=use_kv_connector,
        block_size=BLOCK_SIZE,
        num_blocks=NUM_BLOCKS,
        use_ec_connector=True,
        ec_role="ec_consumer",
    )
    # ... excerpt omitted for brevity ...
    scheduler.ec_connector.has_cache_item = Mock(return_value=True)
    scheduler.ec_connector.update_state_after_alloc = Mock(
    assert scheduled_tokens == NUM_TOKENS
    assert len(scheduler.running) == 1
    assert len(scheduler.waiting) == 1
    assert scheduler.kv_cache_manager.block_pool.get_num_free_blocks() == NUM_BLOCKS - 1
    # All memory should be freed, with no requests waiting / running.
    _step_until_done(scheduler, output, MODEL_RUNNER_OUTPUT)
    assert len(scheduler.running) == 0
    assert len(scheduler.waiting) == 0
```
**EN:** Parameterized test covering `ec connector unable to allocate`. Parameter axes: `use_kv_connector`. Inputs/fixtures: `use_kv_connector`. It exercises `mark.parametrize, create_scheduler, Mock, create_requests, enumerate, ModelRunnerOutput`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `ec connector unable to allocate` 的测试用例。 参数维度：`use_kv_connector`。 输入或 fixture：`use_kv_connector`。 该测试会调用 `mark.parametrize, create_scheduler, Mock, create_requests, enumerate, ModelRunnerOutput`。 代码主体包含 12 个显式断言。

### test_priority_scheduling_ec_connector_preemption_and_resumption (lines 3379-3610)
```python
def test_priority_scheduling_ec_connector_preemption_and_resumption(
    cache_exist, use_kv_connector
):
    """Test that priority scheduling preempts lower priority requests
    when out of KV cache space."""
    # Create scheduler with very limited memory to force preemption
    scheduler = create_scheduler_with_priority(
        model="llava-hf/llava-1.5-7b-hf",
        enable_prefix_caching=True,
        max_num_seqs=2,  # allow multiple requests
        # kv connector should not effect test results
        use_kv_connector=use_kv_connector,
        num_blocks=15,  # can hold 244 tokens with 14 blocks (first block is null)
        block_size=16,  # standard block size
        use_ec_connector=True,
        ec_role="ec_consumer",
    )

    # ... excerpt omitted for brevity ...
    scheduler.ec_connector.has_cache_item = Mock(return_value=True)
    scheduler.ec_connector.update_state_after_alloc = Mock(
    assert len(output.scheduled_new_reqs) == 1
    assert scheduled_tokens == 94
    assert len(scheduler.waiting) == 0
    assert len(scheduler.running) == 1
            requests=[request_low],
            expected_encoder_inputs=[[0]],
            expected_total_reqs=1,
        )
    else:
        _assert_right_encoder_inputs(output, expected_total_reqs=0)
```
**EN:** Parameterized test covering `priority scheduling ec connector preemption and resumption`. Parameter axes: `cache_exist, use_kv_connector`. Inputs/fixtures: `cache_exist, use_kv_connector`. It exercises `mark.parametrize, create_scheduler_with_priority, Mock, scheduler.add_request, scheduler.schedule, _assert_right_encoder_cache_allocated`. The body contains 29 explicit assertion(s).
**CN:** 该代码块是覆盖 `priority scheduling ec connector preemption and resumption` 的测试用例。 参数维度：`cache_exist, use_kv_connector`。 输入或 fixture：`cache_exist, use_kv_connector`。 该测试会调用 `mark.parametrize, create_scheduler_with_priority, Mock, scheduler.add_request, scheduler.schedule, _assert_right_encoder_cache_allocated`。 代码主体包含 29 个显式断言。

### test_ec_connector_allocate_encoder_tokens_with_external_load (lines 3614-3761)
```python
def test_ec_connector_allocate_encoder_tokens_with_external_load(use_kv_connector):
    """
    Scenario:
      - Encoder cache size: 32
      - Request A: 1 feature (12 tokens) → NOT cached remotely.
      - Request B: 3 features (3 x 10 tokens) → ALL cached remotely.

    Steps:
      1. Schedule Request A (locally uses 12 tokens).
      2. Schedule Request B (remote cache) - only schedule 1st and 2nd
      3. Free A's cache, then schedule B again (continuation) - schedule 3rd image
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        max_num_batched_tokens=1024,
        enable_prefix_caching=True,
        use_kv_connector=use_kv_connector,
        block_size=16,
    # ... excerpt omitted for brevity ...
    scheduler.ec_connector.has_cache_item = Mock(
    scheduler.ec_connector.update_state_after_alloc = Mock(
    assert scheduled_tokens == NUM_TOKENS_1
    assert scheduler.get_num_unfinished_requests() == 2
        expected_encoder_inputs=[[0]],  # index 0 of the mm item of request1
        expected_total_reqs=1,
    # Should schedule no encoder input
    _assert_right_encoder_inputs(
        output,
        expected_total_reqs=0,
    )
```
**EN:** Parameterized test covering `ec connector allocate encoder tokens with external load`. Parameter axes: `use_kv_connector`. Inputs/fixtures: `use_kv_connector`. It exercises `mark.parametrize, create_scheduler, EncoderCacheManager, Mock, scheduler.add_request, scheduler.schedule`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `ec connector allocate encoder tokens with external load` 的测试用例。 参数维度：`use_kv_connector`。 输入或 fixture：`use_kv_connector`。 该测试会调用 `mark.parametrize, create_scheduler, EncoderCacheManager, Mock, scheduler.add_request, scheduler.schedule`。 代码主体包含 3 个显式断言。

### test_prepend_skipped_requests_order (lines 3769-3795)
```python
def test_prepend_skipped_requests_order():
    scheduler = create_scheduler(max_num_seqs=1, use_kv_connector=True)
    requests = create_requests(num_requests=4)
    for request in requests:
        scheduler.add_request(request)

    # 4 requests waiting, capture their order
    expected_waiting_reqs = list(scheduler.waiting)

    # simulate first 2 waiting requests are waiting for remote KVs
    for req in expected_waiting_reqs[:2]:
        req.status = RequestStatus.WAITING_FOR_REMOTE_KVS
    scheduler.waiting.remove_requests(expected_waiting_reqs[:2])
    for req in expected_waiting_reqs[:2]:
        scheduler.skipped_waiting.add_request(req)

    # schedule step
    # expect the first 2 waiting to be skipped, the third running,
    # and the fourth waiting
    scheduler.schedule()

    # pop the third request which is expected to be running
    expected_waiting_reqs.pop(2)

    # verify waiting order is preserved
    waiting_reqs = list(scheduler.skipped_waiting) + list(scheduler.waiting)
    assert waiting_reqs == expected_waiting_reqs
```
**EN:** Test case covering `prepend skipped requests order`. It exercises `create_scheduler, create_requests, list, waiting.remove_requests, scheduler.schedule, expected_waiting_reqs.pop`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `prepend skipped requests order` 的测试用例。 该测试会调用 `create_scheduler, create_requests, list, waiting.remove_requests, scheduler.schedule, expected_waiting_reqs.pop`。 代码主体包含 1 个显式断言。

### test_remote_kv_promotion_keeps_fcfs_with_grammar_prefix (lines 3798-3837)
```python
def test_remote_kv_promotion_keeps_fcfs_with_grammar_prefix():
    scheduler = create_scheduler(max_num_seqs=1)
    scheduler.connector = Mock()
    scheduler.connector.get_num_new_matched_tokens.return_value = (0, False)

    requests = create_requests(num_requests=4)
    for request in requests:
        scheduler.add_request(request)
    req_grammar_1, req_grammar_2, req_remote, req_tail = list(scheduler.waiting)
    # simulate two structured-output grammar requests at the waiting head
    # that become ready now.
    req_grammar_1.status = RequestStatus.WAITING_FOR_STRUCTURED_OUTPUT_GRAMMAR
    req_grammar_1.structured_output_request = Mock(grammar=object())
    req_grammar_2.status = RequestStatus.WAITING_FOR_STRUCTURED_OUTPUT_GRAMMAR
    req_grammar_2.structured_output_request = Mock(grammar=object())
    # ... excerpt omitted for brevity ...
    scheduler._update_waiting_for_remote_kv = Mock()
    assert output.scheduled_new_reqs
    assert output.scheduled_new_reqs[0].req_id == req_grammar_1.request_id
    ]
    assert waiting_req_ids == [
        req_grammar_2.request_id,
        req_remote.request_id,
        req_tail.request_id,
```
**EN:** Test case covering `remote KV promotion keeps fcfs with grammar prefix`. It exercises `create_scheduler, Mock, create_requests, list, waiting.remove_requests, skipped_waiting.add_request`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `remote kv promotion keeps fcfs with grammar prefix` 的测试用例。 该测试会调用 `create_scheduler, Mock, create_requests, list, waiting.remove_requests, skipped_waiting.add_request`。 代码主体包含 3 个显式断言。

### test_fcfs_mixed_skipped_waiting_types_keep_order (lines 3840-3879)
```python
def test_fcfs_mixed_skipped_waiting_types_keep_order():
    scheduler = create_scheduler(max_num_batched_tokens=20)
    scheduler._update_waiting_for_remote_kv = Mock()

    mk_req = lambda req_id, num_tokens=1: create_requests(  # noqa: E731
        num_requests=1, num_tokens=num_tokens, req_ids=[req_id]
    )[0]
    req_grammar, req_remote, req_stream = (
        mk_req("grammar"),
        mk_req("remote"),
        mk_req("stream"),
    )
    req_regular, req_tail = mk_req("regular", 20), mk_req("tail")
    req_grammar.status = RequestStatus.WAITING_FOR_STRUCTURED_OUTPUT_GRAMMAR
    req_grammar.structured_output_request = Mock(grammar=None)
    req_remote.status = RequestStatus.WAITING_FOR_REMOTE_KVS
    req_stream.status = RequestStatus.WAITING_FOR_STREAMING_REQ
    # ... excerpt omitted for brevity ...
    assert list(scheduler.skipped_waiting) == [req_grammar, req_remote, req_stream]
    assert not scheduler.running
    req_grammar.structured_output_request = Mock(grammar=object())
    expected_order = [
        req_stream.request_id,
        req_tail.request_id,
    ]
    assert [req.req_id for req in second_output.scheduled_new_reqs] == expected_order
    assert [req.request_id for req in scheduler.running] == expected_order
    scheduler._update_waiting_for_remote_kv.assert_called_once_with(req_remote)
```
**EN:** Test case covering `fcfs mixed skipped waiting types keep order`. It exercises `create_scheduler, Mock, scheduler.schedule, scheduler.finish_requests, finished_recving_kv_req_ids.add, _update_waiting_for_remote_kv.assert_called_once_with`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `fcfs mixed skipped waiting types keep order` 的测试用例。 该测试会调用 `create_scheduler, Mock, scheduler.schedule, scheduler.finish_requests, finished_recving_kv_req_ids.add, _update_waiting_for_remote_kv.assert_called_once_with`。 代码主体包含 4 个显式断言。

### test_abort_request_waiting_for_remote_kvs (lines 3882-3908)
```python
def test_abort_request_waiting_for_remote_kvs():
    scheduler = create_scheduler(use_kv_connector=True)

    # add a single request
    request = create_requests(num_requests=1)[0]
    scheduler.add_request(request)

    # set request to waiting for remote KVs, and abort it
    request.status = RequestStatus.WAITING_FOR_REMOTE_KVS
    scheduler.finish_requests((request.request_id,), RequestStatus.FINISHED_ABORTED)
    assert request.status == RequestStatus.FINISHED_ABORTED

    # verify request is not deleted
    assert request.request_id in scheduler.requests

    # finish recving request
    scheduler_output = scheduler.schedule()
    model_runner_output = ModelRunnerOutput(
        req_ids=[],
        req_id_to_index={},
        kv_connector_output=KVConnectorOutput(finished_recving={request.request_id}),
    )
    scheduler.update_from_output(scheduler_output, model_runner_output)

    # assert request is deleted
    assert request.request_id not in scheduler.requests
    assert not scheduler.finished_recving_kv_req_ids
```
**EN:** Test case covering `abort request waiting for remote kvs`. It exercises `create_scheduler, scheduler.add_request, scheduler.finish_requests, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort request waiting for remote kvs` 的测试用例。 该测试会调用 `create_scheduler, scheduler.add_request, scheduler.finish_requests, scheduler.schedule, ModelRunnerOutput, scheduler.update_from_output`。 代码主体包含 4 个显式断言。

### test_abort_request_finished_recving (lines 3911-3928)
```python
def test_abort_request_finished_recving():
    scheduler = create_scheduler(use_kv_connector=True)

    # add a single request
    request = create_requests(num_requests=1)[0]
    scheduler.add_request(request)

    # set request to waiting for remote KVs, finished but not yet updated
    request.status = RequestStatus.WAITING_FOR_REMOTE_KVS
    scheduler.finished_recving_kv_req_ids.add(request.request_id)

    # abort request
    scheduler.finish_requests((request.request_id,), RequestStatus.FINISHED_ABORTED)
    assert request.status == RequestStatus.FINISHED_ABORTED

    # verify request is deleted
    assert request.request_id not in scheduler.requests
    assert not scheduler.finished_recving_kv_req_ids
```
**EN:** Test case covering `abort request finished recving`. It exercises `create_scheduler, scheduler.add_request, finished_recving_kv_req_ids.add, scheduler.finish_requests, create_requests`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort request finished recving` 的测试用例。 该测试会调用 `create_scheduler, scheduler.add_request, finished_recving_kv_req_ids.add, scheduler.finish_requests, create_requests`。 代码主体包含 3 个显式断言。

### _create_encoder_decoder_scheduler (lines 3936-4024)
```python
def _create_encoder_decoder_scheduler(
    block_size: int = 16,
    num_blocks: int = 10000,
    max_num_batched_tokens: int = 8192,
    max_num_seqs: int = 16,
) -> Scheduler:
    """Create a scheduler configured for encoder-decoder cross-attention
    block allocation testing.

    Constructs a scheduler with both FullAttentionSpec (self-attention) and
    CrossAttentionSpec (cross-attention) KV cache groups, then patches it
    to behave as an encoder-decoder model.
    """
    from vllm.v1.core.encoder_cache_manager import EncoderDecoderCacheManager
    from vllm.v1.kv_cache_interface import CrossAttentionSpec
    model_config = ModelConfig(
        model="facebook/opt-125m",
    # ... excerpt omitted for brevity ...
    scheduler.max_num_encoder_input_tokens = max_num_batched_tokens
    scheduler.encoder_cache_manager = EncoderDecoderCacheManager(
        cache_size=max_num_batched_tokens
    )
    return scheduler
```
**EN:** Helper function `_create_encoder_decoder_scheduler` encapsulates reusable logic for `encoder decoder scheduler`. Inputs: `block_size, num_blocks, max_num_batched_tokens, max_num_seqs`. Key calls include `ModelConfig, SchedulerConfig, CacheConfig, VllmConfig, KVCacheConfig, Scheduler`.
**CN:** 辅助函数 `_create_encoder_decoder_scheduler` 封装了与 `encoder decoder 调度器` 相关的可复用逻辑。 输入参数：`block_size, num_blocks, max_num_batched_tokens, max_num_seqs`。 关键调用包括 `ModelConfig, SchedulerConfig, CacheConfig, VllmConfig, KVCacheConfig, Scheduler`。

### _get_num_cross_attn_blocks (lines 4027-4036)
```python
def _get_num_cross_attn_blocks(scheduler: Scheduler, request_id: str) -> int:
    """Get the number of cross-attention blocks allocated for a request."""
    from vllm.v1.core.single_type_kv_cache_manager import CrossAttentionManager

    coordinator = scheduler.kv_cache_manager.coordinator
    for manager in coordinator.single_type_managers:
        if isinstance(manager, CrossAttentionManager):
            blocks = manager.req_to_blocks.get(request_id, [])
            return len(blocks)
    raise AssertionError("No CrossAttentionManager found in coordinator")
```
**EN:** Helper function `_get_num_cross_attn_blocks` encapsulates reusable logic for `num cross attn blocks`. Inputs: `scheduler, request_id`. Key calls include `AssertionError, isinstance, req_to_blocks.get, len`.
**CN:** 辅助函数 `_get_num_cross_attn_blocks` 封装了与 `num cross attn blocks` 相关的可复用逻辑。 输入参数：`scheduler, request_id`。 关键调用包括 `AssertionError, isinstance, req_to_blocks.get, len`。

### test_variable_length_cross_attn_block_allocation (lines 4039-4098)
```python
def test_variable_length_cross_attn_block_allocation():
    """Test that cross-attention blocks are allocated per-request based on
    actual encoder input length, not a fixed maximum.

    Fixed max-encoder-length allocation would assign
    `ceil(max_encoder_tokens / block_size)` blocks to
    every request whereas with dynamic allocation, exactly
    `ceil(actual_encoder_tokens / block_size)` blocks are assigned
    to each request.
    """
    block_size = 16
    scheduler = _create_encoder_decoder_scheduler(block_size=block_size)
    # Create requests with distinctly different encoder input lengths,
    # simulating variable-length audio inputs to a model like Whisper.
    encoder_lengths = [500, 1000, 200]
    num_prompt_tokens = 100  # Decoder prompt tokens
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == len(requests)
        expected_blocks = ceil(enc_len / block_size)
        assert actual_blocks == expected_blocks, (
            f"expected {expected_blocks} cross-attn blocks, "
        _get_num_cross_attn_blocks(scheduler, req.request_id) for req in requests
    ]
    assert len(set(block_counts)) > 1, (
        "All requests have the same number of cross-attn blocks, "
        "suggesting static max-based allocation instead of per-request"
    )
```
**EN:** Test case covering `variable length cross attn block allocation`. It exercises `_create_encoder_decoder_scheduler, enumerate, scheduler.schedule, zip, requests.append, scheduler.add_request`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `variable length cross attn block allocation` 的测试用例。 该测试会调用 `_create_encoder_decoder_scheduler, enumerate, scheduler.schedule, zip, requests.append, scheduler.add_request`。 代码主体包含 3 个显式断言。

### test_cross_attn_blocks_not_over_allocated (lines 4101-4139)
```python
def test_cross_attn_blocks_not_over_allocated():
    """Test that cross-attention blocks are not over-allocated compared to
    what each request actually needs."""
    from math import ceil

    block_size = 16
    max_encoder_tokens = 1500  # e.g., Whisper's max mel-spectrogram length
    scheduler = _create_encoder_decoder_scheduler(block_size=block_size)
    # Request with a small encoder input (much less than the max).
    small_enc_len = 200
    request = create_requests(
        num_requests=1,
        num_tokens=100,
        mm_hashes_list=[["enc_small"]],
        mm_positions=[[PlaceholderRange(offset=0, length=small_enc_len)]],
        req_ids=["req_small"],
    )[0]
    # ... excerpt omitted for brevity ...
    assert len(output.scheduled_new_reqs) == 1
    expected_blocks = ceil(small_enc_len / block_size)
    assert actual_blocks == expected_blocks, (
        f"Expected {expected_blocks} blocks for {small_enc_len} encoder tokens, "
    # Blocks should be strictly less than what max-based allocation would give.
    assert actual_blocks < max_blocks, (
        f"Cross-attn blocks ({actual_blocks}) should be less than max "
        f"({max_blocks}), indicating no over-allocation"
    )
```
**EN:** Test case covering `cross attn blocks not over allocated`. It exercises `_create_encoder_decoder_scheduler, scheduler.add_request, scheduler.schedule, _get_num_cross_attn_blocks, ceil, create_requests`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `cross attn blocks not over allocated` 的测试用例。 该测试会调用 `_create_encoder_decoder_scheduler, scheduler.add_request, scheduler.schedule, _get_num_cross_attn_blocks, ceil, create_requests`。 代码主体包含 3 个显式断言。

### test_cross_attn_blocks_not_under_allocated (lines 4142-4194)
```python
def test_cross_attn_blocks_not_under_allocated():
    """Test that cross-attention blocks are sufficient for each request's
    actual encoder input length. Every encoder token must have a slot.

    Tests various edge cases including exact block boundaries, off-by-one,
    and the minimum/maximum encoder input sizes.
    """
    from math import ceil
    block_size = 16
    # Test various encoder lengths including edge cases around block boundaries.
    test_cases = [
        1,  # Minimum: single encoder token
        block_size - 1,  # Just under one full block
        block_size,  # Exactly one full block
        block_size + 1,  # Just over one block (needs 2 blocks)
        block_size * 10,  # Exact multiple of block size
    # ... excerpt omitted for brevity ...
        assert len(output.scheduled_new_reqs) == 1
        expected_blocks = ceil(enc_len / block_size)
        assert actual_blocks == expected_blocks, (
            f"Encoder length {enc_len}: expected {expected_blocks} blocks, "
        # Total available slots must be >= encoder tokens (no under-allocation).
        total_slots = actual_blocks * block_size
        assert total_slots >= enc_len, (
            f"Encoder length {enc_len}: total slots {total_slots} < "
            f"needed {enc_len} (under-allocation)"
        )
```
**EN:** Test case covering `cross attn blocks not under allocated`. It exercises `_create_encoder_decoder_scheduler, scheduler.add_request, scheduler.schedule, _get_num_cross_attn_blocks, ceil, create_requests`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `cross attn blocks not under allocated` 的测试用例。 该测试会调用 `_create_encoder_decoder_scheduler, scheduler.add_request, scheduler.schedule, _get_num_cross_attn_blocks, ceil, create_requests`。 代码主体包含 3 个显式断言。

### test_cross_attn_zero_blocks_without_encoder_inputs (lines 4197-4222)
```python
def test_cross_attn_zero_blocks_without_encoder_inputs():
    """Test that requests without encoder inputs get zero cross-attention
    blocks, even when the scheduler is configured for encoder-decoder."""
    block_size = 16
    scheduler = _create_encoder_decoder_scheduler(block_size=block_size)

    # Create a text-only request (no mm_features).
    request = create_requests(
        num_requests=1,
        num_tokens=100,
        req_ids=["req_text_only"],
    )[0]

    # Text-only request has no encoder inputs.
    assert not request.has_encoder_inputs

    scheduler.add_request(request)
    output = scheduler.schedule()

    assert len(output.scheduled_new_reqs) == 1

    # No cross-attention blocks should be allocated.
    actual_blocks = _get_num_cross_attn_blocks(scheduler, request.request_id)
    assert actual_blocks == 0, (
        f"Text-only request should have 0 cross-attn blocks, got {actual_blocks}"
    )
```
**EN:** Test case covering `cross attn zero blocks without encoder inputs`. It exercises `_create_encoder_decoder_scheduler, scheduler.add_request, scheduler.schedule, _get_num_cross_attn_blocks, create_requests, len`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `cross attn zero blocks without encoder inputs` 的测试用例。 该测试会调用 `_create_encoder_decoder_scheduler, scheduler.add_request, scheduler.schedule, _get_num_cross_attn_blocks, create_requests, len`。 代码主体包含 3 个显式断言。

### test_eagle3_mm_encoder_cache_with_shift (lines 4225-4291)
```python
def test_eagle3_mm_encoder_cache_with_shift():
    """Test EAGLE3 encoder scheduling accounts for shift_computed_tokens.

    Regression test for issue #32469: When EAGLE3 is enabled with
    disable_chunked_mm_input=True, ensure encoder inputs are scheduled
    when tokens overlap the MM range, properly accounting for
    shift_computed_tokens in the boundary calculation.
    Without the fix, the scheduler would fail to schedule encoder inputs
    at the boundary, causing "Encoder cache miss" errors.
    """
    scheduler = create_scheduler(
        model="llava-hf/llava-1.5-7b-hf",
        max_num_batched_tokens=1024,
        disable_chunked_mm_input=True,
        max_model_len=2048,
        num_speculative_tokens=4,  # This enables EAGLE with shift=1
    )
    # ... excerpt omitted for brevity ...
    assert output is not None
    assert req_id in output.num_scheduled_tokens
    assert scheduled_end_with_shift > start_pos, (
        f"Test setup error: expected to schedule into MM range. "
    assert req_id in output.scheduled_encoder_inputs, (
        f"Encoder input missing: scheduled {num_scheduled} tokens "
        f"(computed={request.num_computed_tokens}, end={tokens_end}, "
        f"shifted_end={scheduled_end_with_shift}) overlapping MM at "
        f"{start_pos}. The fix must schedule encoder inputs."
```
**EN:** Test case covering `eagle3 mm encoder cache with shift`. It exercises `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, PlaceholderRange`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `eagle3 mm encoder 缓存 with shift` 的测试用例。 该测试会调用 `create_scheduler, create_requests, scheduler.add_request, scheduler.schedule, PlaceholderRange`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.encoder_cache_manager, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.output, vllm.v1.core.sched.scheduler, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.encoder_cache_manager, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.output, vllm.v1.core.sched.scheduler, ...`。
- **EN:** Local test helpers: `tests.v1.core.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.core.utils`。
- **EN:** Standard-library support: `dataclasses, unittest.mock, math`.
- **CN:** 标准库支持：`dataclasses, unittest.mock, math`。
