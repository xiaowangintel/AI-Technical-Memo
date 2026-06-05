# test_remote_prefill_lifecycle.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_remote_prefill_lifecycle.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `remote prefill lifecycle` behavior and regressions in the v1 stack. / 验证 v1 栈中 `remote prefill lifecycle` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-22)
```python
import copy
from unittest.mock import patch

import pytest

from vllm.v1.outputs import (
    EMPTY_MODEL_RUNNER_OUTPUT,
    KVConnectorOutput,
    ModelRunnerOutput,
)
from vllm.v1.request import FinishReason, RequestStatus

from .utils import (
    assert_scheduler_empty,
    create_model_runner_output,
    create_request,
    create_scheduler,
    create_vllm_config,
    make_kv_cache_config,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.v1.outputs, vllm.v1.request`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.v1.outputs, vllm.v1.request`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### Module state / 模块级状态 (line 24)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### _num_waiting_requests (lines 27-28)
```python
def _num_waiting_requests(scheduler) -> int:
    return len(scheduler.waiting) + len(scheduler.skipped_waiting)
```
**EN:** Helper function `_num_waiting_requests` encapsulates reusable logic for `num waiting requests`. Inputs: `scheduler`. Key calls include `len`.
**CN:** 辅助函数 `_num_waiting_requests` 封装了与 `num waiting requests` 相关的可复用逻辑。 输入参数：`scheduler`。 关键调用包括 `len`。

### test_basic_lifecycle (lines 31-149)
```python
def test_basic_lifecycle():
    """Test lifecycle of a remote prefill."""

    vllm_config = create_vllm_config()
    scheduler = create_scheduler(vllm_config)
    # 2 Full Blocks and 1 Half Block.
    BLOCK_SIZE = vllm_config.cache_config.block_size
    NUM_EXTERNAL_FULL_BLOCKS = 2
    NUM_TOKENS = int(BLOCK_SIZE * (NUM_EXTERNAL_FULL_BLOCKS + 0.5))
    START_FREE_BLOCK_QUEUE_SIZE = (
        scheduler.kv_cache_manager.block_pool.free_block_queue.num_free_blocks
    )
    request = create_request(
        request_id=1,
        block_size=BLOCK_SIZE,
        num_tokens=NUM_TOKENS,
    # ... excerpt omitted for brevity ...
    assert len(scheduler.running) == 0
    assert len(scheduler_output.scheduled_new_reqs) == 0
    assert scheduler_output.scheduled_cached_reqs.num_reqs == 0
    assert len(scheduler_output.num_scheduled_tokens) == 0
    assert scheduler_output.total_num_scheduled_tokens == 0
    assert _num_waiting_requests(scheduler) == 1
    outputs = engine_core_outputs[0].outputs
    assert len(outputs) == 1
    output = outputs[0]
    assert output.finish_reason == FinishReason.STOP
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `basic lifecycle`. It exercises `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`. The body contains 23 explicit assertion(s).
**CN:** 该代码块是覆盖 `basic lifecycle` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 23 个显式断言。

### test_interleaved_lifecycle (lines 152-248)
```python
def test_interleaved_lifecycle():
    """Test Remote Prefills Work Well With Other Requests."""

    vllm_config = create_vllm_config()
    scheduler = create_scheduler(vllm_config)
    # 2 Full Blocks and 1 Half Block.
    BLOCK_SIZE = vllm_config.cache_config.block_size
    NUM_EXTERNAL_FULL_BLOCKS = 2
    NUM_TOKENS = int(BLOCK_SIZE * (NUM_EXTERNAL_FULL_BLOCKS + 0.5))
    request_remote = create_request(
        request_id=1,
        block_size=BLOCK_SIZE,
        num_tokens=NUM_TOKENS,
        do_remote_prefill=True,
    )
    request_local_a = create_request(
    # ... excerpt omitted for brevity ...
    assert len(scheduler.running) == 1
    assert len(scheduler.running) == 2
    assert _num_waiting_requests(scheduler) == 1
    assert len(scheduler_output.scheduled_new_reqs) == 1
    assert scheduler_output.scheduled_cached_reqs.num_reqs == 1
        [request_local_a, request_local_b, request_remote],
        use_eos=True,
    scheduler.update_from_output(scheduler_output, model_runner_output)
    scheduler.schedule()
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `interleaved lifecycle`. It exercises `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`. The body contains 21 explicit assertion(s).
**CN:** 该代码块是覆盖 `interleaved lifecycle` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 21 个显式断言。

### test_no_spurious_prefix_caching (lines 251-318)
```python
def test_no_spurious_prefix_caching():
    """
    With P/D, blocks can be allocated but uncomputed for
    multiple engine steps. This test confirms that we do
    not accidentally have cache hits against uncomputed
    blocks.

    vllm_config = create_vllm_config()
    scheduler = create_scheduler(vllm_config)
    # 2 and a half full external blocks.
    BLOCK_SIZE = vllm_config.cache_config.block_size
    NUM_EXTERNAL_FULL_BLOCKS = 2
    NUM_TOKENS = int(BLOCK_SIZE * (NUM_EXTERNAL_FULL_BLOCKS + 0.5))
    # ... excerpt omitted for brevity ...
    assert _num_waiting_requests(scheduler) == 1
    assert len(scheduler.running) == 1
        assert block.ref_cnt == 1
    assert num_hashed_blocks > 0
    # Remote blocks should not be cached.
    for block in remote_blocks:
        assert block._block_hash is None
```
**EN:** Test case covering `no spurious prefix caching`. It exercises `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `no spurious 前缀缓存` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 7 个显式断言。

### test_full_block_prompt (lines 321-393)
```python
def test_full_block_prompt():
    """Test that we handle a prompt that is the full block size."""

    vllm_config = create_vllm_config()
    scheduler = create_scheduler(vllm_config)
    # 2 Full Blocks and 1 Half Block.
    BLOCK_SIZE = vllm_config.cache_config.block_size
    NUM_EXTERNAL_FULL_BLOCKS = 2
    NUM_TOKENS = int(BLOCK_SIZE * NUM_EXTERNAL_FULL_BLOCKS)
    request = create_request(
        request_id=1,
        block_size=BLOCK_SIZE,
        num_tokens=NUM_TOKENS,
        do_remote_prefill=True,
    )
    # ... excerpt omitted for brevity ...
    assert num_blocks == NUM_EXTERNAL_FULL_BLOCKS
    assert _num_waiting_requests(scheduler) == 1
    assert request_id in scheduler.finished_recving_kv_req_ids
    assert scheduler_output.scheduled_new_reqs[0].num_computed_tokens == NUM_TOKENS - 1
    assert scheduler_output.num_scheduled_tokens[request_id] == 1
    outputs = engine_core_outputs[0].outputs
    assert len(outputs) == 1
    output = outputs[0]
    assert output.finish_reason == FinishReason.STOP
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `full block prompt`. It exercises `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `full block prompt` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 8 个显式断言。

### test_cannot_schedule_after_recv (lines 396-496)
```python
def test_cannot_schedule_after_recv():
    """
    Test that we can handle no schedule after recv due to not
    enough remaining KV blocks.

    # NOTE: the KVCacheManager will use 1 null block.
    # So there are 5 total working blocks.
    TOTAL_NUM_BLOCKS = 6
    vllm_config = create_vllm_config()
    scheduler = create_scheduler(vllm_config, num_blocks=TOTAL_NUM_BLOCKS)
    # Prime the KVCache.
    NUM_PROMPT_BLOCKS = 2
    BLOCK_SIZE = vllm_config.cache_config.block_size
    # Prompt will use 2 blocks + 1 block after we schedule.
    NUM_TOKENS_LOCAL = int(BLOCK_SIZE * NUM_PROMPT_BLOCKS)
    NUM_TOKENS_REMOTE = int(BLOCK_SIZE * NUM_PROMPT_BLOCKS)
    # ... excerpt omitted for brevity ...
    assert len(scheduler.running) == 1
    assert _num_waiting_requests(scheduler) == 0
    assert _num_waiting_requests(scheduler) == 1
    model_runner_output = create_model_runner_output(
        reqs=[request_remote], use_eos=True
    )
    scheduler.update_from_output(scheduler_output, model_runner_output)
    _ = scheduler.schedule()
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `cannot schedule after recv`. It exercises `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`. The body contains 15 explicit assertion(s).
**CN:** 该代码块是覆盖 `cannot schedule after recv` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 15 个显式断言。

### test_cannot_recv (lines 499-587)
```python
def test_cannot_recv():
    """
    Test that we can handle no schedule KV block transfer due to not
    enough remaining KV blocks.

    # NOTE: the KVCacheManager will use 1 null block.
    # So there are 5 total working blocks.
    TOTAL_NUM_BLOCKS = 6
    vllm_config = create_vllm_config()
    scheduler = create_scheduler(vllm_config, num_blocks=TOTAL_NUM_BLOCKS)
    # Prime the KVCache.
    NUM_PROMPT_BLOCKS = 2
    BLOCK_SIZE = vllm_config.cache_config.block_size
    # Prompt will use 2 blocks + 1 block after we schedule.
    NUM_TOKENS_LOCAL = int(BLOCK_SIZE * NUM_PROMPT_BLOCKS)
    NUM_TOKENS_REMOTE = int(BLOCK_SIZE * (NUM_PROMPT_BLOCKS + 0.5))
    # ... excerpt omitted for brevity ...
    assert len(scheduler.running) == 1
    assert _num_waiting_requests(scheduler) == 0
    assert _num_waiting_requests(scheduler) == 1
    assert request_remote.status != RequestStatus.WAITING_FOR_REMOTE_KVS
    assert len(scheduler.running) == 0
    model_runner_output = create_model_runner_output(
        reqs=[request_remote], use_eos=True
    )
    scheduler.update_from_output(scheduler_output, model_runner_output)
    _ = scheduler.schedule()
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `cannot recv`. It exercises `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`. The body contains 14 explicit assertion(s).
**CN:** 该代码块是覆盖 `cannot recv` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 14 个显式断言。

### test_p_side_chunked_prefill_mamba (lines 591-657)
```python
def test_p_side_chunked_prefill_mamba(mock_platform):
    """P-side integration: Mamba N-1 truncation + chunked prefill completes.

    A 64-token P-side request is truncated to 63 by the N-1 fix, then
    chunked into two prefill steps (32 + 31) and finishes with
    LENGTH_CAPPED because max_tokens is set to 1.
    """
    mock_platform.device_type = "cpu"
    BATCH_SIZE = 32
    NUM_TOKENS = 64
    BLOCK_SIZE = 16
    vllm_config = create_vllm_config(
        max_num_batched_tokens=BATCH_SIZE,
        block_size=BLOCK_SIZE,
    )
    vllm_config.scheduler_config.disable_hybrid_kv_cache_manager = False
    # ... excerpt omitted for brevity ...
    assert len(request.prompt_token_ids) == NUM_TOKENS - 1
    assert request.max_tokens == 1
    assert scheduler_output.num_scheduled_tokens[request_id] == BATCH_SIZE
    assert request.num_computed_tokens == BATCH_SIZE
    assert scheduler_output.num_scheduled_tokens[request_id] == remaining
    assert request.num_computed_tokens == NUM_TOKENS - 1
    engine_core_outputs = scheduler.update_from_output(scheduler_output, final_output)
    # max_tokens=1 → request finishes with LENGTH
    outputs = engine_core_outputs[0].outputs
    assert len(outputs) == 1
    assert outputs[0].finish_reason == FinishReason.LENGTH
```
**EN:** Test case covering `p side chunked prefillMamba`. Inputs/fixtures: `mock_platform`. It exercises `patch, create_vllm_config, make_kv_cache_config, create_scheduler, create_request, scheduler.add_request`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `p side chunked prefillmamba` 的测试用例。 输入或 fixture：`mock_platform`。 该测试会调用 `patch, create_vllm_config, make_kv_cache_config, create_scheduler, create_request, scheduler.add_request`。 代码主体包含 8 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.v1.outputs, vllm.v1.request`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.outputs, vllm.v1.request`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `copy, unittest.mock`.
- **CN:** 标准库支持：`copy, unittest.mock`。
