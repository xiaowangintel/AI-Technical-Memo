# test_error_propagation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_error_propagation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `error propagation` behavior and regressions in the v1 stack. / 验证 v1 栈中 `error propagation` 相关行为与回归。

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

### Module state / 模块级状态 (line 19)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### _make_get_num_new_matched_tokens (lines 22-30)
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

### fail_scheduler (lines 34-38)
```python
def fail_scheduler():
    """scheduler with kv_load_failure_policy='fail'"""
    vllm_config = create_vllm_config()
    vllm_config.kv_transfer_config.kv_load_failure_policy = "fail"
    return create_scheduler(vllm_config)
```
**EN:** Fixture/helper `fail_scheduler` prepares reusable state for downstream tests. Key calls include `create_vllm_config, create_scheduler`.
**CN:** `fail_scheduler` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `create_vllm_config, create_scheduler`。

### test_error_propagation_sync_load (lines 41-92)
```python
def test_error_propagation_sync_load(fail_scheduler: Scheduler):
    """test invalid_block_ids with fail policy -> FINISHED_ERROR (sync load)"""
    num_prompt_blocks = 100
    num_external_computed_blocks = 99
    invalid_block_idx = 50

    num_prompt_tokens = num_prompt_blocks * fail_scheduler.block_size
    num_external_computed_tokens = (
        num_external_computed_blocks * fail_scheduler.block_size
    )
    request = create_request(num_tokens=num_prompt_tokens)
    fail_scheduler.add_request(request=request)
    req_num_new_matched_tokens = {
        request.request_id: num_external_computed_tokens,
    }
    # ... excerpt omitted for brevity ...
    fail_scheduler.connector = Mock()
    assert len(fail_scheduler.running) == 1
    assert len(scheduler_output.scheduled_new_reqs) == 1
    assert fail_scheduler.connector.get_num_new_matched_tokens.call_count == 1
    assert request.status == RequestStatus.FINISHED_ERROR
    assert request.get_finished_reason() == FinishReason.ERROR
    assert len(engine_outputs.outputs) == 1
    output = engine_outputs.outputs[0]
    assert output.request_id == request.request_id
    assert output.finish_reason == FinishReason.ERROR
    assert len(fail_scheduler.running) == 0
```
**EN:** Test case covering `error propagation sync load`. Inputs/fixtures: `fail_scheduler`. It exercises `create_request, fail_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, fail_scheduler.schedule, create_model_runner_output`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `error propagation sync load` 的测试用例。 输入或 fixture：`fail_scheduler`。 该测试会调用 `create_request, fail_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, fail_scheduler.schedule, create_model_runner_output`。 代码主体包含 10 个显式断言。

### test_error_propagation_async_load (lines 95-148)
```python
def test_error_propagation_async_load(fail_scheduler: Scheduler):
    """test invalid_block_ids with fail policy -> FINISHED_ERROR (async load)"""
    num_prompt_blocks = 100
    num_external_computed_blocks = 99
    invalid_block_idx = 50

    num_prompt_tokens = num_prompt_blocks * fail_scheduler.block_size
    num_external_computed_tokens = (
        num_external_computed_blocks * fail_scheduler.block_size
    )
    request = create_request(num_tokens=num_prompt_tokens)
    fail_scheduler.add_request(request=request)
    req_num_new_matched_tokens = {
        request.request_id: num_external_computed_tokens,
    }
    # ... excerpt omitted for brevity ...
    fail_scheduler.connector = Mock()
    assert len(fail_scheduler.skipped_waiting) == 1
    assert request.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    assert request.num_computed_tokens == num_external_computed_tokens
    assert request.status == RequestStatus.FINISHED_ERROR
    assert request.get_finished_reason() == FinishReason.ERROR
    output = engine_outputs.outputs[0]
    assert output.request_id == request.request_id
    assert output.finish_reason == FinishReason.ERROR
    assert len(fail_scheduler.waiting) == 0
    assert len(fail_scheduler.skipped_waiting) == 0
```
**EN:** Test case covering `error propagation async load`. Inputs/fixtures: `fail_scheduler`. It exercises `create_request, fail_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, fail_scheduler.schedule, kv_cache_manager.get_block_ids`. The body contains 11 explicit assertion(s).
**CN:** 该代码块是覆盖 `error propagation async load` 的测试用例。 输入或 fixture：`fail_scheduler`。 该测试会调用 `create_request, fail_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, fail_scheduler.schedule, kv_cache_manager.get_block_ids`。 代码主体包含 11 个显式断言。

## Key Concepts / 关键概念
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
