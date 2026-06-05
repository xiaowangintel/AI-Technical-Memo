# test_scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/offloading_connector/test_scheduler.py`
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

### Imports and setup / 导入与设置 (lines 3-30)
```python
from collections.abc import Iterable
from unittest.mock import MagicMock

import pytest
import torch

from tests.v1.kv_connector.unit.offloading_connector.utils import (
    generate_store_output,
    to_keys,
)
from tests.v1.kv_connector.unit.utils import EOS_TOKEN_ID
from vllm.distributed.kv_events import BlockRemoved, BlockStored
from vllm.distributed.kv_transfer.kv_connector.v1.offloading.scheduler import (
    OffloadingConnectorScheduler,
)
from vllm.v1.core.kv_cache_utils import BlockHash
from vllm.v1.kv_cache_interface import (
    FullAttentionSpec,
    KVCacheGroupSpec,
    SlidingWindowSpec,
)
from vllm.v1.kv_offload.base import (
    OffloadingEvent,
    OffloadingManager,
    ReqContext,
    get_offload_block_hash,
)
from vllm.v1.request import RequestStatus
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.distributed.kv_events, vllm.distributed.kv_transfer.kv_connector.v1.offloading.scheduler, vllm.v1.core.kv_cache_utils, vllm.v1.kv_cache_interface, vllm.v1.kv_offload.base, ...`. Local helpers come from `tests.v1.kv_connector.unit.offloading_connector.utils, tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.distributed.kv_events, vllm.distributed.kv_transfer.kv_connector.v1.offloading.scheduler, vllm.v1.core.kv_cache_utils, vllm.v1.kv_cache_interface, vllm.v1.kv_offload.base, ...`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.offloading_connector.utils, tests.v1.kv_connector.unit.utils`。

### test_offloading_connector (lines 34-167)
```python
def test_offloading_connector(request_runner, async_scheduling: bool):
    block_size = 4
    block_size_factor = 3
    offloaded_block_size = block_size * block_size_factor
    num_gpu_blocks = 100

    runner = request_runner(
        block_size=block_size,
        num_gpu_blocks=num_gpu_blocks,
        async_scheduling=async_scheduling,
        block_size_factor=block_size_factor,
    )
    # 3 blocks, store just the middle block (skip first and last)
    # blocks = [0, 1, 2], [3, 4, 5], [6, 7, 8]
    runner.new_request(token_ids=[0] * offloaded_block_size * 3)
    runner.manager.prepare_store.side_effect = lambda keys, req_context: (
        generate_store_output(list(keys)[1:2])
    # ... excerpt omitted for brevity ...
        expected_stored=(3, 4, 5),
        expected_stored=(15, 16, 17),
    assert len(block_hashes1) == 6
    assert len(block_hashes2) == 6
    assert block_hashes1[:5] == block_hashes2[:5]
    assert block_hashes1[5] != block_hashes2[5]
    assert event.lora_id is None
    assert event.lora_name is None
    event = events[1]
    assert isinstance(event, BlockRemoved)
    assert event.block_hashes == to_hashes([4, 5, 6])
    assert event.medium == "B"
```
**EN:** Parameterized test covering `offloading connector`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, touch.assert_not_called, prepare_store.assert_called`. The body contains 16 explicit assertion(s).
**CN:** 该代码块是覆盖 `offloading connector` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, touch.assert_not_called, prepare_store.assert_called`。 代码主体包含 16 个显式断言。

### test_request_preemption (lines 171-239)
```python
def test_request_preemption(request_runner, async_scheduling: bool):
    block_size = 4
    block_size_factor = 3
    offloaded_block_size = block_size * block_size_factor
    num_gpu_blocks = 100

    runner = request_runner(
        block_size=block_size,
        num_gpu_blocks=num_gpu_blocks,
        async_scheduling=async_scheduling,
        block_size_factor=block_size_factor,
    )
    free_block_queue = runner.scheduler.kv_cache_manager.block_pool.free_block_queue
    num_free_blocks_empty = free_block_queue.num_free_blocks
    # 2 blocks, store all, without flushing
    # blocks = [0, 1, 2], [3, 4, 5]
    # ... excerpt omitted for brevity ...
        expected_flushed=(0, 1, 2, 3, 4, 5, 6, 7, 8),
        expected_stored=(0, 1, 2, 3, 4, 5, 6, 7, 8),
        expected_loaded=(0, 1, 2, 3, 4, 5, 6, 7, 8),
        decoded_tokens=[EOS_TOKEN_ID],
        expected_stored=(9, 10, 11),
    # All stores completed before request_finished -> fence index empty.
    assert runner.connector_scheduler._block_id_to_pending_jobs == {}
```
**EN:** Parameterized test covering `request preemption`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, generate_store_output`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `request preemption` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, generate_store_output`。 代码主体包含 1 个显式断言。

### test_concurrent_lookups_of_the_same_prefix (lines 243-306)
```python
def test_concurrent_lookups_of_the_same_prefix(request_runner, async_scheduling: bool):
    block_size = 4
    block_size_factor = 3
    offloaded_block_size = block_size * block_size_factor
    num_gpu_blocks = 100

    runner = request_runner(
        block_size=block_size,
        num_gpu_blocks=num_gpu_blocks,
        async_scheduling=async_scheduling,
        block_size_factor=block_size_factor,
    )
    # store 1 blocks
    runner.new_request(token_ids=[0] * offloaded_block_size)
    runner.manager.prepare_store.side_effect = lambda keys, req_context: (
        generate_store_output(keys)
    # ... excerpt omitted for brevity ...
        expected_stored=(0, 1, 2),
        expected_flushed=(0, 1, 2) if not async_scheduling else (),
    assert transfer_jobs
    assert transfer_jobs == list(runner.offloading_spec.handler.transfer_specs)
        expected_loaded=(0, 1, 2),
    # second request will use the GPU prefix cache
    # Fence index drained: stores completed before request_finished ran.
    assert runner.connector_scheduler._block_id_to_pending_jobs == {}
```
**EN:** Parameterized test covering `concurrent lookups of the same prefix`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, list`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `concurrent lookups of the same prefix` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, list`。 代码主体包含 4 个显式断言。

### test_abort_loading_requests (lines 310-362)
```python
def test_abort_loading_requests(request_runner, async_scheduling: bool):
    block_size = 4
    block_size_factor = 3
    offloaded_block_size = block_size * block_size_factor
    num_gpu_blocks = 100

    runner = request_runner(
        block_size=block_size,
        num_gpu_blocks=num_gpu_blocks,
        async_scheduling=async_scheduling,
        block_size_factor=block_size_factor,
    )
    # store 1 blocks
    runner.new_request(token_ids=[0] * offloaded_block_size)
    runner.manager.prepare_store.side_effect = lambda keys, req_context: (
        generate_store_output(keys)
    # ... excerpt omitted for brevity ...
        expected_stored=(0, 1, 2),
        expected_flushed=(0, 1, 2) if not async_scheduling else (),
    assert transfer_jobs
    assert req_id in runner.scheduler.requests
        expected_loaded=(0, 1, 2),
        expected_flushed=(0, 1, 2),
    # assert request is deleted
    assert req_id not in runner.scheduler.requests
```
**EN:** Parameterized test covering `abort loading requests`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, list`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort loading requests` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, list`。 代码主体包含 3 个显式断言。

### test_two_groups_full_and_sliding_window (lines 366-468)
```python
def test_two_groups_full_and_sliding_window(request_runner, async_scheduling: bool):
    block_size = 4
    num_gpu_blocks = 100
    # sliding_window=8 -> 2 offloaded blocks (block_size_factor=1)
    sliding_window = 8

    kv_cache_groups = [
        KVCacheGroupSpec(
            ["layer0"],
            FullAttentionSpec(
                block_size=block_size,
                num_kv_heads=1,
                head_size=1,
                dtype=torch.float32,
            ),
        ),
            ["layer1"],
    # ... excerpt omitted for brevity ...
    assert len(kv_group_configs) == 2
    assert kv_group_configs[0].sliding_window_size_in_blocks is None
    assert kv_group_configs[1].sliding_window_size_in_blocks == 2
    assert len(touch_calls) == 4
    assert len(touch_calls[0].args[0]) == 3
    assert len(touch_calls[1].args[0]) == 3
        decoded_tokens=[EOS_TOKEN_ID],
        # Group 0 (full attn): prefix lookup hits 3 → loads blocks 0,1,2
        # Group 1 (sliding window, window=2): only the last 2 blocks
        #   are within the window → loads blocks 1,2
        expected_loaded=((0, 3), (1, 3)),
    )
```
**EN:** Parameterized test covering `two groups full and sliding window`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, KVCacheGroupSpec`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `two groups full and sliding window` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, KVCacheGroupSpec`。 代码主体包含 12 个显式断言。

### test_two_groups_different_block_sizes (lines 472-613)
```python
def test_two_groups_different_block_sizes(request_runner, async_scheduling: bool):
    hash_block_size = 4
    num_gpu_blocks = 100

    # Group 0: block_size=12 (offloaded_block_size=12)
    # Group 1: block_size=16 (offloaded_block_size=16)
    kv_cache_groups = [
        KVCacheGroupSpec(
            ["layer0"],
            FullAttentionSpec(
                block_size=hash_block_size * 3,
                num_kv_heads=1,
                head_size=1,
                dtype=torch.float32,
            ),
        ),
            ["layer1"],
    # ... excerpt omitted for brevity ...
    assert len(kv_group_configs) == 2
    assert kv_group_configs[0].gpu_block_size == 12
    assert kv_group_configs[0].offloaded_block_size == 12
    assert kv_group_configs[1].gpu_block_size == 16
    assert kv_group_configs[1].offloaded_block_size == 16
    assert len(touch_calls) == 4
    )
    runner.run(
        decoded_tokens=[0],
        expected_loaded=((0, 4), (0, 5), (0, 6), (1, 3), (1, 4)),
    runner.run(decoded_tokens=[EOS_TOKEN_ID])
```
**EN:** Parameterized test covering `two groups different block sizes`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, KVCacheGroupSpec`. The body contains 19 explicit assertion(s).
**CN:** 该代码块是覆盖 `two groups different block sizes` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, KVCacheGroupSpec`。 代码主体包含 19 个显式断言。

### _make_scheduler_with_lookup (lines 621-632)
```python
def _make_scheduler_with_lookup(
    lookup_results: dict[int, bool | None],
) -> OffloadingConnectorScheduler:
    """Create an OffloadingConnectorScheduler with a mocked manager.lookup."""
    manager = MagicMock(spec=OffloadingManager)
    manager.lookup.side_effect = lambda key, req_context: lookup_results.get(
        int(get_offload_block_hash(key).decode()), False
    )

    scheduler = object.__new__(OffloadingConnectorScheduler)
    scheduler.manager = manager
    return scheduler
```
**EN:** Helper function `_make_scheduler_with_lookup` encapsulates reusable logic for `scheduler with lookup`. Inputs: `lookup_results`. Key calls include `MagicMock, object.__new__, lookup_results.get, int, get_offload_block_hash.decode, get_offload_block_hash`.
**CN:** 辅助函数 `_make_scheduler_with_lookup` 封装了与 `调度器 with lookup` 相关的可复用逻辑。 输入参数：`lookup_results`。 关键调用包括 `MagicMock, object.__new__, lookup_results.get, int, get_offload_block_hash.decode, get_offload_block_hash`。

### Module state / 模块级状态 (line 635)
```python
_EMPTY_REQ_CTX = ReqContext(req_id="")
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_EMPTY_REQ_CTX`. Shared setup calls include `ReqContext`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_EMPTY_REQ_CTX`。 共享初始化调用包括 `ReqContext`。

### TestMaximalPrefixLookup (lines 638-676)
```python
class TestMaximalPrefixLookup:
    def test_all_hit(self):
        sched = _make_scheduler_with_lookup({1: True, 2: True})
        assert sched._maximal_prefix_lookup(to_keys([1, 2]), _EMPTY_REQ_CTX) == 2

    def test_all_miss(self):
        sched = _make_scheduler_with_lookup({})
        assert sched._maximal_prefix_lookup(to_keys([1, 2]), _EMPTY_REQ_CTX) == 0
    def test_partial_prefix(self):
        assert sched._maximal_prefix_lookup(to_keys([1, 2, 3]), _EMPTY_REQ_CTX) == 2
    def test_miss_then_hit(self):
        sched = _make_scheduler_with_lookup({2: True})
    def test_single_hit(self):
    # ... excerpt omitted for brevity ...
        assert sched._maximal_prefix_lookup(to_keys([1]), _EMPTY_REQ_CTX) == 1
        assert sched._maximal_prefix_lookup([], _EMPTY_REQ_CTX) == 0
        assert sched._maximal_prefix_lookup(to_keys([1, 2]), _EMPTY_REQ_CTX) is None
    def test_none_stops_at_miss(self):
        """None is treated as hit for iteration, but miss stops the scan."""
        sched = _make_scheduler_with_lookup({1: None, 2: False, 3: True})
        assert sched._maximal_prefix_lookup(to_keys([1, 2, 3]), _EMPTY_REQ_CTX) is None
        # lookup should have been called for blocks 1 and 2 (stops at miss)
        assert sched.manager.lookup.call_count == 2
```
**EN:** Class `TestMaximalPrefixLookup` groups 9 test method(s). Representative scenarios: `test_all_hit, test_all_miss, test_partial_prefix, test_miss_then_hit, test_single_hit, test_empty, ...`.
**CN:** 类 `TestMaximalPrefixLookup` 组织了 9 个测试方法。 代表性场景：`test_all_hit, test_all_miss, test_partial_prefix, test_miss_then_hit, test_single_hit, test_empty, ...`。

### TestSlidingWindowLookup (lines 679-747)
```python
class TestSlidingWindowLookup:
    def test_all_hit_exact_window(self):
        sched = _make_scheduler_with_lookup({1: True, 2: True})
        assert sched._sliding_window_lookup(to_keys([1, 2]), 2, _EMPTY_REQ_CTX) == 2

    def test_all_miss(self):
        sched = _make_scheduler_with_lookup({})
        assert sched._sliding_window_lookup(to_keys([1, 2, 3]), 1, _EMPTY_REQ_CTX) == 0
    def test_window_at_end(self):
        sched = _make_scheduler_with_lookup({2: True, 3: True})
        assert sched._sliding_window_lookup(to_keys([1, 2, 3]), 2, _EMPTY_REQ_CTX) == 3
    def test_window_in_middle(self):
        assert (
            sched._sliding_window_lookup(to_keys([1, 2, 3, 4]), 2, _EMPTY_REQ_CTX) == 3
        )
    # ... excerpt omitted for brevity ...
        assert sched._sliding_window_lookup(to_keys([1, 2, 3]), 3, _EMPTY_REQ_CTX) == 2
        assert sched._sliding_window_lookup(to_keys([1, 2, 3]), 1, _EMPTY_REQ_CTX) == 3
        assert sched._sliding_window_lookup([], 1, _EMPTY_REQ_CTX) == 0
        # but block 3 was None so defer_lookup is set
        sched = _make_scheduler_with_lookup({1: True, 2: True, 3: None, 4: True})
            sched._sliding_window_lookup(to_keys([1, 2, 3, 4]), 2, _EMPTY_REQ_CTX)
            is None
```
**EN:** Class `TestSlidingWindowLookup` groups 12 test method(s). Representative scenarios: `test_all_hit_exact_window, test_all_miss, test_window_at_end, test_window_in_middle, test_no_full_window_falls_back_to_prefix, test_single_block_window, ...`.
**CN:** 类 `TestSlidingWindowLookup` 组织了 12 个测试方法。 代表性场景：`test_all_hit_exact_window, test_all_miss, test_window_at_end, test_window_in_middle, test_no_full_window_falls_back_to_prefix, test_single_block_window, ...`。

### test_do_remote_decode_stores_all_blocks (lines 751-802)
```python
def test_do_remote_decode_stores_all_blocks(request_runner, async_scheduling: bool):
    """With do_remote_decode=True, after loading prefix blocks from CPU,
    all blocks must be re-stored — not just the newly computed ones.

    This supports P/D disaggregation where the prefill instance offloads the
    complete KV cache so a remote decode node can consume it."""
    gpu_block_size = 4
    block_size_factor = 3
    offloaded_block_size = gpu_block_size * block_size_factor
    num_gpu_blocks = 100
    runner = request_runner(
        block_size_factor=block_size_factor,
        block_size=gpu_block_size,
        num_gpu_blocks=num_gpu_blocks,
        async_scheduling=async_scheduling,
    )
    # ... excerpt omitted for brevity ...
        expected_stored=(0, 1, 2),
        expected_flushed=(0, 1, 2) if not async_scheduling else (),
    runner.run(decoded_tokens=[0], expected_loaded=(0, 1, 2))
    # Store must include ALL 6 GPU blocks (both the loaded prefix and
    # the newly computed block), not just the 3 new ones.
    runner.run(decoded_tokens=[EOS_TOKEN_ID], expected_stored=(0, 1, 2, 3, 4, 5))
    # All stores completed before request_finished -> fence index empty.
    assert runner.connector_scheduler._block_id_to_pending_jobs == {}
```
**EN:** Parameterized test covering `do remote decode stores all blocks`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, generate_store_output`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `do remote decode stores all blocks` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, generate_store_output`。 代码主体包含 1 个显式断言。

### test_loads_do_not_populate_fence_index (lines 810-822)
```python
def test_loads_do_not_populate_fence_index(request_runner):
    """Loads don't populate _block_id_to_pending_jobs (protected by
    delay_free_blocks while in flight)."""
    runner = request_runner(
        block_size_factor=3,
        block_size=4,
        num_gpu_blocks=100,
        async_scheduling=False,
    )
    runner.new_request(token_ids=[0] * 12)
    runner.connector_scheduler._maximal_prefix_lookup = lambda key, req_context: 1
    runner.run(decoded_tokens=[], complete_transfers=False)
    assert runner.connector_scheduler._block_id_to_pending_jobs == {}
```
**EN:** Test case covering `loads do not populate fence index`. Inputs/fixtures: `request_runner`. It exercises `request_runner, runner.new_request, runner.run`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `loads do not populate fence index` 的测试用例。 输入或 fixture：`request_runner`。 该测试会调用 `request_runner, runner.new_request, runner.run`。 代码主体包含 1 个显式断言。

### test_fence_at_update_state_after_alloc (lines 825-861)
```python
def test_fence_at_update_state_after_alloc(request_runner):
    """A load reusing a finished request's pending-store block triggers
    a flush via update_state_after_alloc's fence.

    num_gpu_blocks=2 forces the BlockPool to give req2 the same block
    req1 just freed.
    """
    runner = request_runner(
        block_size_factor=1,
        block_size=4,
        num_gpu_blocks=2,
        async_scheduling=False,
    )
    runner.new_request(token_ids=[0] * 4)
    runner.manager.prepare_store.side_effect = lambda keys, req_context: (
        generate_store_output(keys)
    # ... excerpt omitted for brevity ...
        expected_stored=(0,),
        expected_flushed=(0,),
    assert runner.connector_scheduler._block_id_to_pending_jobs == {}
    runner.run(
        decoded_tokens=[],
        complete_transfers=False,
```
**EN:** Test case covering `fence at update state after alloc`. Inputs/fixtures: `request_runner`. It exercises `request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, generate_store_output`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `fence at update state after alloc` 的测试用例。 输入或 fixture：`request_runner`。 该测试会调用 `request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, generate_store_output`。 代码主体包含 2 个显式断言。

### test_fence_at_build_store_jobs (lines 864-896)
```python
def test_fence_at_build_store_jobs(request_runner):
    """A new prefill (no load -> update_state_after_alloc returns early)
    reusing a finished request's pending-store block is flushed by
    _build_store_jobs's fence."""
    runner = request_runner(
        block_size_factor=1,
        block_size=4,
        num_gpu_blocks=2,
        async_scheduling=False,
    )

    runner.new_request(token_ids=[0] * 4)
    runner.manager.prepare_store.side_effect = lambda keys, req_context: (
        generate_store_output(keys)
    runner.run(
        decoded_tokens=[EOS_TOKEN_ID],
        complete_transfers=False,
    # ... excerpt omitted for brevity ...
        expected_stored=(0,),
        expected_flushed=(0,),
    assert runner.connector_scheduler._block_id_to_pending_jobs == {}
        generate_store_output([])
```
**EN:** Test case covering `fence at build store jobs`. Inputs/fixtures: `request_runner`. It exercises `request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, generate_store_output`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `fence at build store jobs` 的测试用例。 输入或 fixture：`request_runner`。 该测试会调用 `request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, generate_store_output`。 代码主体包含 2 个显式断言。

### test_complete_store_called_per_job (lines 900-936)
```python
def test_complete_store_called_per_job(request_runner, async_scheduling: bool):
    """complete_store fires per-job, not deferred to request finish.
    Each call carries only that store's keys."""
    gpu_block_size = 4
    block_size_factor = 3
    offloaded_block_size = gpu_block_size * block_size_factor
    runner = request_runner(
        block_size_factor=block_size_factor,
        block_size=gpu_block_size,
        num_gpu_blocks=100,
        async_scheduling=async_scheduling,
    )
    runner.new_request(token_ids=[0] * offloaded_block_size)
    runner.manager.prepare_store.side_effect = lambda keys, req_context: (
        generate_store_output(keys)

    # First store: fires when block 0 is fully populated.
    # ... excerpt omitted for brevity ...
    runner.run(decoded_tokens=[0, 0], expected_stored=(0, 1, 2))
    assert runner.manager.complete_store.call_count == 1
    assert len(first_call_keys) == 1
        expected_stored=(3, 4, 5),
    assert first_call_keys != second_call_keys
    runner.manager.complete_store.reset_mock()
    # Finish: no store pending -> no further call.
    runner.run(decoded_tokens=[EOS_TOKEN_ID])
    assert runner.manager.complete_store.call_count == 0
```
**EN:** Parameterized test covering `complete store called per job`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, set, complete_store.reset_mock`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `complete store called per job` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, set, complete_store.reset_mock`。 代码主体包含 5 个显式断言。

### test_flush_all_jobs_when_no_requests_remain (lines 939-962)
```python
def test_flush_all_jobs_when_no_requests_remain(request_runner):
    """When all tracked requests are finished, build_connector_meta flushes
    all pending jobs since there will be no future step to complete them."""
    block_size = 4
    block_size_factor = 1
    offloaded_block_size = block_size * block_size_factor

    runner = request_runner(
        block_size=block_size,
        num_gpu_blocks=100,
        async_scheduling=False,
        block_size_factor=block_size_factor,
    )

    runner.new_request(token_ids=[0] * offloaded_block_size)
    runner.manager.prepare_store.side_effect = lambda keys, req_context: (
        generate_store_output(keys)
    )
    runner.run(
        decoded_tokens=[EOS_TOKEN_ID],
        complete_transfers=False,
        expected_stored=(0,),
        expected_flushed=(0,),
    )
```
**EN:** Test case covering `flush all jobs when no requests remain`. Inputs/fixtures: `request_runner`. It exercises `request_runner, runner.new_request, runner.run, generate_store_output`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `flush all jobs when no requests remain` 的测试用例。 输入或 fixture：`request_runner`。 该测试会调用 `request_runner, runner.new_request, runner.run, generate_store_output`。 主要通过 mock、回调或输出检查来完成验证。

### test_reset_cache (lines 966-1043)
```python
def test_reset_cache(request_runner, async_scheduling: bool):
    """reset_cache flushes in-flight loads, calls manager.reset_cache(), resets
    next_stored_block_idx for active requests and clears job tracking."""
    block_size = 4
    block_size_factor = 3
    offloaded_block_size = block_size * block_size_factor
    num_gpu_blocks = 100

    runner = request_runner(
        block_size=block_size,
        num_gpu_blocks=num_gpu_blocks,
        async_scheduling=async_scheduling,
        block_size_factor=block_size_factor,
    )
    # Store 1 offloaded block (3 GPU blocks) to CPU.
    runner.new_request(token_ids=[0] * offloaded_block_size)
    runner.manager.prepare_store.side_effect = lambda keys, req_context: (
    # ... excerpt omitted for brevity ...
        expected_stored=(0, 1, 2),
        expected_flushed=(0, 1, 2) if not async_scheduling else (),
    assert load_job_ids, "expected in-flight load jobs before reset"
            assert group_state.next_stored_block_idx > 0
    assert load_job_ids <= runner.connector_scheduler._current_batch_jobs_to_flush
    assert not runner.connector_scheduler._jobs
    # next_stored_block_idx must be reset to 0 for every active request so
    # that post-reset stores restart from block 0.
    for req_status in runner.connector_scheduler._req_status.values():
        for group_state in req_status.group_states:
            assert group_state.next_stored_block_idx == 0
```
**EN:** Parameterized test covering `reset cache`. Parameter axes: `async_scheduling`. Inputs/fixtures: `request_runner, async_scheduling`. It exercises `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, _req_status.values`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `reset 缓存` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`request_runner, async_scheduling`。 该测试会调用 `mark.parametrize, request_runner, runner.new_request, runner.run, scheduler.reset_prefix_cache, _req_status.values`。 代码主体包含 8 个显式断言。

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
- **EN:** vLLM modules under test: `vllm.distributed.kv_events, vllm.distributed.kv_transfer.kv_connector.v1.offloading.scheduler, vllm.v1.core.kv_cache_utils, vllm.v1.kv_cache_interface, vllm.v1.kv_offload.base, vllm.v1.request`.
- **CN:** 被测试的 vLLM 模块：`vllm.distributed.kv_events, vllm.distributed.kv_transfer.kv_connector.v1.offloading.scheduler, vllm.v1.core.kv_cache_utils, vllm.v1.kv_cache_interface, vllm.v1.kv_offload.base, vllm.v1.request`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.offloading_connector.utils, tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.offloading_connector.utils, tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `collections.abc, unittest.mock`.
- **CN:** 标准库支持：`collections.abc, unittest.mock`。
