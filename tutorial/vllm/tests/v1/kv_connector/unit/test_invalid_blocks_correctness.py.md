# test_invalid_blocks_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_invalid_blocks_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for correctness in invalid block handling. / 该文件的文档字符串表明其用途：`tests for correctness in invalid block handling`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 4-12)
```python
"""
Tests for correctness in invalid block handling.

These tests verify correct behavior in three scenarios:
1. Sync recompute case: Blocks should not be freed for running requests
   that need to recompute invalid blocks
2. Sync fail case: Invalid blocks must be evicted from cache when request fails
3. Async recompute case: Invalid blocks should not be cached after transfer
"""
```
**EN:** Module docstring that declares the scope of the file: Tests for correctness in invalid block handling.
**CN:** 模块文档字符串直接说明了文件范围：`tests for correctness in invalid block handling`。

### Imports and setup / 导入与设置 (lines 14-27)
```python
from collections.abc import Callable
from unittest.mock import Mock

import pytest

from vllm.v1.core.sched.scheduler import Scheduler
from vllm.v1.request import FinishReason, Request, RequestStatus

from .utils import (
    create_model_runner_output,
    create_request,
    create_scheduler,
    create_vllm_config,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.v1.core.sched.scheduler, vllm.v1.request`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.v1.core.sched.scheduler, vllm.v1.request`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### Module state / 模块级状态 (line 29)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### _make_get_num_new_matched_tokens (lines 32-40)
```python
def _make_get_num_new_matched_tokens(
    req_num_new_matched_tokens: dict[str, int],
    async_load: bool,
) -> Callable[[Request, int], tuple[int, bool]]:
    def get_num_new_matched_tokens(request: Request, _: int) -> tuple[int, bool]:
        value = req_num_new_matched_tokens.get(request.request_id, 0)
        return value, async_load

    return get_num_new_matched_tokens
```
**EN:** Helper function `_make_get_num_new_matched_tokens` encapsulates reusable logic for `get num new matched tokens`. Inputs: `req_num_new_matched_tokens, async_load`. Key calls include `req_num_new_matched_tokens.get`.
**CN:** 辅助函数 `_make_get_num_new_matched_tokens` 封装了与 `get num new matched tokens` 相关的可复用逻辑。 输入参数：`req_num_new_matched_tokens, async_load`。 关键调用包括 `req_num_new_matched_tokens.get`。

### fail_scheduler (lines 44-48)
```python
def fail_scheduler():
    """scheduler with kv_load_failure_policy='fail'"""
    vllm_config = create_vllm_config()
    vllm_config.kv_transfer_config.kv_load_failure_policy = "fail"
    return create_scheduler(vllm_config)
```
**EN:** Fixture/helper `fail_scheduler` prepares reusable state for downstream tests. Key calls include `create_vllm_config, create_scheduler`.
**CN:** `fail_scheduler` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `create_vllm_config, create_scheduler`。

### recompute_scheduler (lines 52-56)
```python
def recompute_scheduler():
    """scheduler with kv_load_failure_policy='recompute'"""
    vllm_config = create_vllm_config()
    vllm_config.kv_transfer_config.kv_load_failure_policy = "recompute"
    return create_scheduler(vllm_config)
```
**EN:** Fixture/helper `recompute_scheduler` prepares reusable state for downstream tests. Key calls include `create_vllm_config, create_scheduler`.
**CN:** `recompute_scheduler` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `create_vllm_config, create_scheduler`。

### test_sync_recompute_blocks_not_freed_for_running_requests (lines 59-179)
```python
def test_sync_recompute_blocks_not_freed_for_running_requests(
    recompute_scheduler: Scheduler,
):
    """
    Test sync recompute case - blocks must not be freed for running requests.

    When a running request has invalid blocks and retry_policy is 'recompute':
    1. Request should remain in RUNNING state
    2. num_computed_tokens should be truncated to invalid block boundary
    3. Blocks should NOT be freed (request still needs them for recomputation)
    4. Request should remain in scheduler.requests and scheduler.running
    num_prompt_blocks = 100
    num_external_computed_blocks = 99
    invalid_block_idx = 50
    num_prompt_tokens = num_prompt_blocks * recompute_scheduler.block_size
    num_external_computed_tokens = (
    # ... excerpt omitted for brevity ...
    recompute_scheduler.connector = Mock()
    assert len(recompute_scheduler.running) == 1
    assert len(scheduler_output.scheduled_new_reqs) == 1
    assert request.status == RequestStatus.RUNNING
    assert request.status == RequestStatus.RUNNING, (
    expected_truncated_tokens = invalid_block_idx * recompute_scheduler.block_size
    if scheduler_output_2.num_scheduled_tokens:
        scheduled_req_ids.extend(scheduler_output_2.num_scheduled_tokens.keys())
    assert (
        request.request_id in scheduled_req_ids or len(recompute_scheduler.running) > 0
    ), "Request should be reschedulable for recomputation"
```
**EN:** Test case covering `sync recompute blocks not freed for running requests`. Inputs/fixtures: `recompute_scheduler`. It exercises `create_request, recompute_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, recompute_scheduler.schedule, create_model_runner_output`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `sync recompute blocks not freed for running requests` 的测试用例。 输入或 fixture：`recompute_scheduler`。 该测试会调用 `create_request, recompute_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, recompute_scheduler.schedule, create_model_runner_output`。 代码主体包含 12 个显式断言。

### test_sync_fail_invalid_blocks_evicted (lines 182-293)
```python
def test_sync_fail_invalid_blocks_evicted(fail_scheduler: Scheduler):
    """
    Test sync fail case - invalid blocks must be evicted from cache.

    When a request fails with policy='fail' and has invalid blocks from sync loading:
    1. Request should be finished with FINISHED_ERROR
    2. Invalid blocks should be evicted from the KV cache
    3. Valid blocks (if shared) should remain in cache
    4. Future requests should not reuse the invalid blocks
    This test verifies that invalid blocks are properly evicted to prevent
    cache corruption and reuse of invalid data.
    num_prompt_blocks = 100
    num_external_computed_blocks = 99
    invalid_block_idx = 50
    num_prompt_tokens = num_prompt_blocks * fail_scheduler.block_size
    # ... excerpt omitted for brevity ...
    fail_scheduler.connector = Mock()
    assert len(fail_scheduler.running) == 1
    assert request.status == RequestStatus.RUNNING
    assert block is not None
    assert request.status == RequestStatus.FINISHED_ERROR
    assert request.get_finished_reason() == FinishReason.ERROR
    stats = engine_outputs.scheduler_stats
    assert stats.connector_prefix_cache_stats is not None
    conn_stats = stats.connector_prefix_cache_stats
    assert conn_stats.requests == 1
    assert conn_stats.queries == num_prompt_tokens
    assert conn_stats.hits == num_external_computed_tokens
```
**EN:** Test case covering `sync fail invalid blocks evicted`. Inputs/fixtures: `fail_scheduler`. It exercises `create_request, fail_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, fail_scheduler.schedule, create_model_runner_output`. The body contains 17 explicit assertion(s).
**CN:** 该代码块是覆盖 `sync fail invalid blocks evicted` 的测试用例。 输入或 fixture：`fail_scheduler`。 该测试会调用 `create_request, fail_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, fail_scheduler.schedule, create_model_runner_output`。 代码主体包含 17 个显式断言。

### test_async_recompute_blocks_not_cached_when_invalid (lines 296-480)
```python
def test_async_recompute_blocks_not_cached_when_invalid(
    recompute_scheduler: Scheduler,
):
    """
    Test async recompute case - invalid blocks not cached after transfer.

    When async KV loading has invalid blocks and retry_policy is 'recompute':
    1. Blocks are allocated but not cached yet
    2. When async transfer completes, only valid blocks should be cached
    3. Invalid blocks should never enter the prefix cache
    This test verifies correctness, the failed_recving_kv_req_ids protection
    ensures only valid blocks are cached when the transfer completes, and we
    only evict blocks from cache that are already hashed in the block table.
    from unittest.mock import patch
    num_prompt_blocks = 100
    # ... excerpt omitted for brevity ...
    recompute_scheduler.connector = Mock()
    assert len(recompute_scheduler.skipped_waiting) == 1
    assert request.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    assert request.num_computed_tokens == num_external_computed_tokens
    assert block.block_hash is None, (
        return original_evict_blocks(block_ids)
    # request should no longer be in the failed/finished receiving sets
    assert request.request_id not in recompute_scheduler.failed_recving_kv_req_ids
    assert request.request_id not in recompute_scheduler.finished_recving_kv_req_ids
    # request should be in the running queue
    assert request in recompute_scheduler.running
```
**EN:** Test case covering `async recompute blocks not cached when invalid`. Inputs/fixtures: `recompute_scheduler`. It exercises `create_request, recompute_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, recompute_scheduler.schedule, kv_cache_manager.get_block_ids`. The body contains 26 explicit assertion(s).
**CN:** 该代码块是覆盖 `async recompute blocks not cached when invalid` 的测试用例。 输入或 fixture：`recompute_scheduler`。 该测试会调用 `create_request, recompute_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, recompute_scheduler.schedule, kv_cache_manager.get_block_ids`。 代码主体包含 26 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
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
