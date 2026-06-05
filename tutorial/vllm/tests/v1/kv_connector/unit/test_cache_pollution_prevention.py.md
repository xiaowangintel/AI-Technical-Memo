# test_cache_pollution_prevention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_cache_pollution_prevention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: test that invalid blocks are evicted from prefix cache to prevent pollution. / 该文件的文档字符串表明其用途：`test that invalid blocks are evicted from prefix 缓存 to prevent pollution`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 4-9)
```python
"""
test that invalid blocks are evicted from prefix cache to prevent pollution.

verifies that when sync-loading fails, invalid blocks are removed from the
prefix cache hash table so future requests cannot match and reuse corrupted data.
"""
```
**EN:** Module docstring that declares the scope of the file: test that invalid blocks are evicted from prefix cache to prevent pollution.
**CN:** 模块文档字符串直接说明了文件范围：`test that invalid blocks are evicted from prefix 缓存 to prevent pollution`。

### Imports and setup / 导入与设置 (lines 11-24)
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

### Module state / 模块级状态 (line 26)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### _make_get_num_new_matched_tokens (lines 29-37)
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

### fail_scheduler (lines 41-45)
```python
def fail_scheduler():
    """scheduler with kv_load_failure_policy='fail'"""
    vllm_config = create_vllm_config()
    vllm_config.kv_transfer_config.kv_load_failure_policy = "fail"
    return create_scheduler(vllm_config)
```
**EN:** Fixture/helper `fail_scheduler` prepares reusable state for downstream tests. Key calls include `create_vllm_config, create_scheduler`.
**CN:** `fail_scheduler` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `create_vllm_config, create_scheduler`。

### test_invalid_blocks_evicted_prevents_cache_pollution (lines 48-163)
```python
def test_invalid_blocks_evicted_prevents_cache_pollution(
    fail_scheduler: Scheduler,
):
    """
    verify invalid blocks are evicted to prevent future cache hits.

    scenario:
    1. request 1 loads externally-computed blocks (sync mode)
    2. some blocks fail to load and are marked invalid
    3. with fail policy, invalid blocks should be evicted from prefix cache
    4. request is marked as FINISHED_ERROR
    num_prompt_blocks = 100
    num_external_computed_blocks = 99
    invalid_block_idx = 50
    num_prompt_tokens = num_prompt_blocks * fail_scheduler.block_size
    num_external_computed_tokens = (
    # ... excerpt omitted for brevity ...
    fail_scheduler.connector = Mock()
    assert len(fail_scheduler.running) == 1
    assert request1.status == RequestStatus.RUNNING
    assert block.block_hash is not None, (
    assert request1.status == RequestStatus.FINISHED_ERROR
        assert block_obj.block_hash is None, (
    for idx in range(invalid_block_idx, len(req_block_ids)):
        block_id = req_block_ids[idx]
        assert block_id not in cached_block_ids, (
            f"invalid block {block_id} at index {idx} should not be in cache hash table"
        )
```
**EN:** Test case covering `invalid blocks evicted prevents cache pollution`. Inputs/fixtures: `fail_scheduler`. It exercises `create_request, fail_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, fail_scheduler.schedule, kv_cache_manager.cache_blocks`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `invalid blocks evicted prevents 缓存 pollution` 的测试用例。 输入或 fixture：`fail_scheduler`。 该测试会调用 `create_request, fail_scheduler.add_request, Mock, _make_get_num_new_matched_tokens, fail_scheduler.schedule, kv_cache_manager.cache_blocks`。 代码主体包含 7 个显式断言。

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
