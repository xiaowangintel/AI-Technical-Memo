# test_kv_load_failure_recovery.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_kv_load_failure_recovery.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `KV load failure recovery` behavior and regressions in the v1 stack. / 验证 v1 栈中 `kv load failure recovery` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-17)
```python
from collections.abc import Callable
from unittest.mock import Mock

import pytest

from vllm.v1.core.sched.scheduler import Scheduler
from vllm.v1.request import Request, RequestStatus

from .utils import (
    create_model_runner_output,
    create_request,
    create_scheduler,
    create_vllm_config,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.v1.core.sched.scheduler, vllm.v1.request`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.v1.core.sched.scheduler, vllm.v1.request`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### _make_get_num_new_matched_tokens (lines 20-28)
```python
def _make_get_num_new_matched_tokens(
    req_num_new_matched_tokens: dict[str, int],
    async_load,
) -> Callable[[Request, int], tuple[int, bool]]:
    def get_num_new_matched_tokens(request: Request, _: int) -> tuple[int, bool]:
        value = req_num_new_matched_tokens.get(request.request_id, 0)
        return value, async_load

    return get_num_new_matched_tokens
```
**EN:** Helper function `_make_get_num_new_matched_tokens` encapsulates reusable logic for `get num new matched tokens`. Inputs: `req_num_new_matched_tokens, async_load`. Key calls include `req_num_new_matched_tokens.get`.
**CN:** 辅助函数 `_make_get_num_new_matched_tokens` 封装了与 `get num new matched tokens` 相关的可复用逻辑。 输入参数：`req_num_new_matched_tokens, async_load`。 关键调用包括 `req_num_new_matched_tokens.get`。

### scheduler (lines 32-34)
```python
def scheduler():
    vllm_config = create_vllm_config(kv_load_failure_policy="recompute")
    return create_scheduler(vllm_config)
```
**EN:** Fixture/helper `scheduler` prepares reusable state for downstream tests. Key calls include `create_vllm_config, create_scheduler`.
**CN:** `scheduler` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `create_vllm_config, create_scheduler`。

### test_async_load_failure (lines 45-111)
```python
def test_async_load_failure(
    scheduler: Scheduler,
    num_prompt_blocks: int,
    num_external_computed_blocks: int,
    invalid_block_idxs: set[int],
):
    assert num_prompt_blocks >= num_external_computed_blocks

    num_prompt_tokens = num_prompt_blocks * scheduler.block_size
    num_external_computed_tokens = num_external_computed_blocks * scheduler.block_size
    request1 = create_request(num_tokens=num_prompt_tokens)
    scheduler.add_request(request=request1)
    request2 = create_request(num_tokens=num_prompt_tokens)
    scheduler.add_request(request=request2)
    request3 = create_request(num_tokens=num_prompt_tokens)
    scheduler.add_request(request=request3)
    # ... excerpt omitted for brevity ...
    scheduler.connector = Mock()
    assert len(scheduler.waiting) == 0
    assert len(scheduler.skipped_waiting) == 3
        assert request.num_computed_tokens == num_external_computed_tokens
        assert request.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    assert scheduler.connector.get_num_new_matched_tokens.call_count == 3
            )
        else:
            assert request.num_computed_tokens == num_external_computed_tokens
    assert scheduler.failed_recving_kv_req_ids == {request2.request_id}
```
**EN:** Parameterized test covering `async load failure`. Parameter axes: `num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`. Inputs/fixtures: `scheduler, num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`. It exercises `mark.parametrize, create_request, scheduler.add_request, Mock, _make_get_num_new_matched_tokens, scheduler.schedule`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `async load failure` 的测试用例。 参数维度：`num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`。 输入或 fixture：`scheduler, num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`。 该测试会调用 `mark.parametrize, create_request, scheduler.add_request, Mock, _make_get_num_new_matched_tokens, scheduler.schedule`。 代码主体包含 13 个显式断言。

### test_sync_load_failure (lines 122-187)
```python
def test_sync_load_failure(
    scheduler: Scheduler,
    num_prompt_blocks: int,
    num_external_computed_blocks: int,
    invalid_block_idxs: set[int],
):
    assert num_prompt_blocks >= num_external_computed_blocks

    num_prompt_tokens = num_prompt_blocks * scheduler.block_size
    num_external_computed_tokens = num_external_computed_blocks * scheduler.block_size
    request1 = create_request(num_tokens=num_prompt_tokens)
    scheduler.add_request(request=request1)
    request2 = create_request(num_tokens=num_prompt_tokens)
    scheduler.add_request(request=request2)
    request3 = create_request(num_tokens=num_prompt_tokens)
    scheduler.add_request(request=request3)
    # ... excerpt omitted for brevity ...
    scheduler.connector = Mock()
    expected_computed_tokens = {
    assert len(scheduler.running) == 3
    assert len(scheduler_output.scheduled_new_reqs) == 3
        assert request.num_computed_tokens == expected_computed_tokens[request.req_id]
    assert scheduler.connector.get_num_new_matched_tokens.call_count == 3
    assert scheduler.running[0].request_id == request2.request_id
    assert scheduler.running[0].num_computed_tokens == (
        min(invalid_block_idxs) * scheduler.block_size
    )
    assert scheduler.connector.request_finished.call_count == 2
```
**EN:** Parameterized test covering `sync load failure`. Parameter axes: `num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`. Inputs/fixtures: `scheduler, num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`. It exercises `mark.parametrize, create_request, scheduler.add_request, Mock, _make_get_num_new_matched_tokens, scheduler.schedule`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `sync load failure` 的测试用例。 参数维度：`num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`。 输入或 fixture：`scheduler, num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`。 该测试会调用 `mark.parametrize, create_request, scheduler.add_request, Mock, _make_get_num_new_matched_tokens, scheduler.schedule`。 代码主体包含 10 个显式断言。

### test_sync_load_failure_with_shared_blocks (lines 201-270)
```python
def test_sync_load_failure_with_shared_blocks(
    scheduler: Scheduler,
    num_prompt_blocks: int,
    num_external_computed_blocks: int,
    num_common_prefix_blocks: int,
    invalid_block_idxs: set[int],
):
    assert num_prompt_blocks >= num_external_computed_blocks >= num_common_prefix_blocks

    num_prompt_tokens = num_prompt_blocks * scheduler.block_size
    num_external_computed_tokens = num_external_computed_blocks * scheduler.block_size
    common_prefix_len = num_common_prefix_blocks * scheduler.block_size
    request1 = create_request(
        num_tokens=num_prompt_tokens, common_prefix_len=common_prefix_len
    )
    scheduler.add_request(request=request1)
    request2 = create_request(
    # ... excerpt omitted for brevity ...
    scheduler.connector = Mock()
    expected_computed_tokens = {
    assert len(scheduler.running) == 2
    assert len(scheduler_output.scheduled_new_reqs) == 2
        assert request.num_computed_tokens == expected_computed_tokens[request.req_id]
    assert scheduler.connector.get_num_new_matched_tokens.call_count == 2
    for request in scheduler.running:
        assert (
            request.num_computed_tokens == expected_computed_tokens[request.request_id]
        )
```
**EN:** Parameterized test covering `sync load failure with shared blocks`. Parameter axes: `num_prompt_blocks, num_external_computed_blocks, num_common_prefix_blocks, invalid_block_idxs`. Inputs/fixtures: `scheduler, num_prompt_blocks, num_external_computed_blocks, num_common_prefix_blocks, invalid_block_idxs`. It exercises `mark.parametrize, create_request, scheduler.add_request, Mock, _make_get_num_new_matched_tokens, scheduler.schedule`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `sync load failure with shared blocks` 的测试用例。 参数维度：`num_prompt_blocks, num_external_computed_blocks, num_common_prefix_blocks, invalid_block_idxs`。 输入或 fixture：`scheduler, num_prompt_blocks, num_external_computed_blocks, num_common_prefix_blocks, invalid_block_idxs`。 该测试会调用 `mark.parametrize, create_request, scheduler.add_request, Mock, _make_get_num_new_matched_tokens, scheduler.schedule`。 代码主体包含 8 个显式断言。

### test_async_progressive_load_failure (lines 280-339)
```python
def test_async_progressive_load_failure(
    scheduler: Scheduler,
    num_prompt_blocks: int,
    num_external_computed_blocks: int,
    invalid_block_idxs: set[int],
):
    assert num_prompt_blocks >= num_external_computed_blocks

    num_prompt_tokens = num_prompt_blocks * scheduler.block_size
    num_external_computed_tokens = num_external_computed_blocks * scheduler.block_size
    request = create_request(num_tokens=num_prompt_tokens)
    scheduler.add_request(request=request)
    # Mock KV connector method.
    # req_id -> num_external_computed_tokens
    req_num_new_matched_tokens = {
        request.request_id: num_external_computed_tokens,
    # ... excerpt omitted for brevity ...
    scheduler.connector = Mock()
    assert len(scheduler.waiting) == 0
    assert len(scheduler.skipped_waiting) == 1
    assert scheduler.skipped_waiting.peek_request().request_id == request.request_id
    assert request.num_computed_tokens == num_external_computed_tokens
    assert request.status == RequestStatus.WAITING_FOR_REMOTE_KVS
        assert request.num_computed_tokens == (
            min_invalid_block_idx * scheduler.block_size
        )
        assert request.status == RequestStatus.WAITING_FOR_REMOTE_KVS
        assert scheduler.failed_recving_kv_req_ids == {request.request_id}
        assert scheduler.connector.get_num_new_matched_tokens.call_count == 1
```
**EN:** Parameterized test covering `async progressive load failure`. Parameter axes: `num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`. Inputs/fixtures: `scheduler, num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`. It exercises `mark.parametrize, create_request, scheduler.add_request, Mock, _make_get_num_new_matched_tokens, scheduler.schedule`. The body contains 14 explicit assertion(s).
**CN:** 该代码块是覆盖 `async progressive load failure` 的测试用例。 参数维度：`num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`。 输入或 fixture：`scheduler, num_prompt_blocks, num_external_computed_blocks, invalid_block_idxs`。 该测试会调用 `mark.parametrize, create_request, scheduler.add_request, Mock, _make_get_num_new_matched_tokens, scheduler.schedule`。 代码主体包含 14 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.v1.core.sched.scheduler, vllm.v1.request`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.core.sched.scheduler, vllm.v1.request`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `collections.abc, unittest.mock`.
- **CN:** 标准库支持：`collections.abc, unittest.mock`。
