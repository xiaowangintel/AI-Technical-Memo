# test_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_offload/cpu/test_manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `manager` behavior and regressions in the v1 stack. / 验证 v1 栈中 `manager` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-19)
```python
from collections.abc import Iterable
from dataclasses import dataclass

import numpy as np
import pytest

from vllm.v1.kv_offload.base import (
    LoadStoreSpec,
    OffloadingEvent,
    OffloadKey,
    PrepareStoreOutput,
    ReqContext,
    make_offload_key,
)
from vllm.v1.kv_offload.cpu.common import CPULoadStoreSpec
from vllm.v1.kv_offload.cpu.manager import CPUOffloadingManager
from vllm.v1.kv_offload.cpu.policies.arc import ARCCachePolicy
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest`. vLLM modules under test include `vllm.v1.kv_offload.base, vllm.v1.kv_offload.cpu.common, vllm.v1.kv_offload.cpu.manager, vllm.v1.kv_offload.cpu.policies.arc`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest`。 被测试的 vLLM 模块包括 `vllm.v1.kv_offload.base, vllm.v1.kv_offload.cpu.common, vllm.v1.kv_offload.cpu.manager, vllm.v1.kv_offload.cpu.policies.arc`。

### make_req_context (lines 22-26)
```python
def make_req_context(
    req_id: str = "", kv_transfer_params: dict | None = None
) -> ReqContext:
    """Create a ReqContext as production code would, from a request's params."""
    return ReqContext(req_id=req_id, kv_transfer_params=kv_transfer_params)
```
**EN:** Helper function `make_req_context` encapsulates reusable logic for `req context`. Inputs: `req_id, kv_transfer_params`. Key calls include `ReqContext`.
**CN:** 辅助函数 `make_req_context` 封装了与 `req context` 相关的可复用逻辑。 输入参数：`req_id, kv_transfer_params`。 关键调用包括 `ReqContext`。

### Module state / 模块级状态 (line 29)
```python
_EMPTY_REQ_CTX = make_req_context()
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_EMPTY_REQ_CTX`. Shared setup calls include `make_req_context`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_EMPTY_REQ_CTX`。 共享初始化调用包括 `make_req_context`。

### ExpectedPrepareStoreOutput (lines 33-36)
```python
class ExpectedPrepareStoreOutput:
    keys_to_store: list[int]
    store_block_ids: list[int]
    evicted_keys: list[int]
```
**EN:** Class `ExpectedPrepareStoreOutput` groups 0 test method(s).
**CN:** 类 `ExpectedPrepareStoreOutput` 组织了 0 个测试方法。

### to_key (lines 39-40)
```python
def to_key(int_hash: int) -> OffloadKey:
    return make_offload_key(str(int_hash).encode(), 0)
```
**EN:** Helper function `to_key` encapsulates reusable logic for `to key`. Inputs: `int_hash`. Key calls include `make_offload_key, str.encode, str`.
**CN:** 辅助函数 `to_key` 封装了与 `to key` 相关的可复用逻辑。 输入参数：`int_hash`。 关键调用包括 `make_offload_key, str.encode, str`。

### to_keys (lines 43-44)
```python
def to_keys(int_hashes: list[int]) -> list[OffloadKey]:
    return [to_key(i) for i in int_hashes]
```
**EN:** Helper function `to_keys` encapsulates reusable logic for `to keys`. Inputs: `int_hashes`. Key calls include `to_key`.
**CN:** 辅助函数 `to_keys` 封装了与 `to keys` 相关的可复用逻辑。 输入参数：`int_hashes`。 关键调用包括 `to_key`。

### verify_store_output (lines 47-63)
```python
def verify_store_output(
    prepare_store_output: PrepareStoreOutput | None,
    expected_prepare_store_output: ExpectedPrepareStoreOutput,
):
    assert prepare_store_output is not None
    assert prepare_store_output.keys_to_store == to_keys(
        expected_prepare_store_output.keys_to_store
    )
    assert prepare_store_output.evicted_keys == to_keys(
        expected_prepare_store_output.evicted_keys
    )
    store_spec = prepare_store_output.store_spec
    assert isinstance(store_spec, CPULoadStoreSpec)
    expected_array = np.array(
        expected_prepare_store_output.store_block_ids, dtype=np.int64
    )
    assert np.array_equal(expected_array, store_spec.block_ids)
```
**EN:** Helper function `verify_store_output` encapsulates reusable logic for `verify store output`. Inputs: `prepare_store_output, expected_prepare_store_output`. Key calls include `isinstance, np.array, np.array_equal, to_keys`. It includes 5 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `verify_store_output` 封装了与 `verify store output` 相关的可复用逻辑。 输入参数：`prepare_store_output, expected_prepare_store_output`。 关键调用包括 `isinstance, np.array, np.array_equal, to_keys`。 其中包含 5 个内部断言，用于保护前置假设。

### verify_load_output (lines 66-71)
```python
def verify_load_output(
    prepare_load_output: LoadStoreSpec, expected_prepare_load_output: list[int]
):
    assert isinstance(prepare_load_output, CPULoadStoreSpec)
    expected_array = np.array(expected_prepare_load_output, dtype=np.int64)
    assert np.array_equal(expected_array, prepare_load_output.block_ids)
```
**EN:** Helper function `verify_load_output` encapsulates reusable logic for `verify load output`. Inputs: `prepare_load_output, expected_prepare_load_output`. Key calls include `isinstance, np.array, np.array_equal`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `verify_load_output` 封装了与 `verify load output` 相关的可复用逻辑。 输入参数：`prepare_load_output, expected_prepare_load_output`。 关键调用包括 `isinstance, np.array, np.array_equal`。 其中包含 2 个内部断言，用于保护前置假设。

### verify_events (lines 74-94)
```python
def verify_events(
    events: Iterable[OffloadingEvent],
    expected_stores: tuple[set[int], ...] = (),
    expected_evictions: tuple[set[int], ...] = (),
):
    stores: list[set[OffloadKey]] = []
    evictions: list[set[OffloadKey]] = []
    for event in events:
        assert event.medium == CPULoadStoreSpec.medium()
        if event.removed:
            evictions.append(set(event.keys))
        else:
            stores.append(set(event.keys))

    def to_key_sets(
        int_sets: tuple[set[int], ...],
    ) -> tuple[set[OffloadKey], ...]:
        return tuple([set(to_keys(list(int_set))) for int_set in int_sets])

    assert tuple(evictions) == to_key_sets(expected_evictions)
    assert tuple(stores) == to_key_sets(expected_stores)
```
**EN:** Helper function `verify_events` encapsulates reusable logic for `verify events`. Inputs: `events, expected_stores, expected_evictions`. Key calls include `tuple, to_key_sets, CPULoadStoreSpec.medium, evictions.append, stores.append, set`. It includes 3 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `verify_events` 封装了与 `verify events` 相关的可复用逻辑。 输入参数：`events, expected_stores, expected_evictions`。 关键调用包括 `tuple, to_key_sets, CPULoadStoreSpec.medium, evictions.append, stores.append, set`。 其中包含 3 个内部断言，用于保护前置假设。

### test_already_stored_block_not_evicted_during_prepare_store (lines 98-144)
```python
def test_already_stored_block_not_evicted_during_prepare_store(eviction_policy):
    """
    Regression test: a block that is already stored must not be evicted
    by prepare_store() when it needs to make room for new blocks.
    Applies to both lru and arc policies.

    Scenario:
        - Store blocks [1, 2] and complete.
        - touch([1]) makes block 2 the LRU candidate.
        - prepare_store([2, 3, 4, 5]):
            * block 2 is filtered out as "already stored"
            * but without the fix, block 2 would be evicted as the LRU
              candidate to make room for [3, 4, 5]
        - After complete_store([2, 3, 4, 5]), block 2 must still be present.
    manager = CPUOffloadingManager(
        num_blocks=4,
        cache_policy=eviction_policy,
    # ... excerpt omitted for brevity ...
    # complete_store must not silently drop block 2
    manager.complete_store(to_keys([2, 3, 4, 5]), _EMPTY_REQ_CTX)
    # block 2 must still be present in the cache
    assert manager.lookup(to_key(2), _EMPTY_REQ_CTX) is True
```
**EN:** Parameterized test covering `already stored block not evicted during prepare store`. Parameter axes: `eviction_policy`. Inputs/fixtures: `eviction_policy`. It exercises `mark.parametrize, CPUOffloadingManager, manager.prepare_store, manager.complete_store, manager.touch, verify_store_output`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `already stored block not evicted during prepare store` 的测试用例。 参数维度：`eviction_policy`。 输入或 fixture：`eviction_policy`。 该测试会调用 `mark.parametrize, CPUOffloadingManager, manager.prepare_store, manager.complete_store, manager.touch, verify_store_output`。 代码主体包含 1 个显式断言。

### test_cpu_manager (lines 147-262)
```python
def test_cpu_manager():
    """
    Tests CPUOffloadingManager with lru policy.
    # initialize a CPU manager with a capacity of 4 blocks
    cpu_manager = CPUOffloadingManager(
        num_blocks=4, cache_policy="lru", enable_events=True
    )

    # prepare store [1, 2]
    prepare_store_output = cpu_manager.prepare_store(to_keys([1, 2]), _EMPTY_REQ_CTX)
    verify_store_output(
        prepare_store_output,
        ExpectedPrepareStoreOutput(
            keys_to_store=[1, 2],
            store_block_ids=[0, 1],
            evicted_keys=[],
        ),
    # ... excerpt omitted for brevity ...
    assert cpu_manager.lookup(to_key(1), _EMPTY_REQ_CTX) is None
    assert cpu_manager.lookup(to_key(2), _EMPTY_REQ_CTX) is None
    assert list(cpu_manager.take_events()) == []
    verify_events(cpu_manager.take_events(), expected_stores=({1, 2},))
    assert cpu_manager.lookup(to_key(1), _EMPTY_REQ_CTX) is True
    assert cpu_manager.lookup(to_key(2), _EMPTY_REQ_CTX) is True
    verify_events(
        cpu_manager.take_events(),
        expected_stores=({3, 4, 5}, {6, 7, 8}),
        expected_evictions=({2, 3, 4}, {8}),
```
**EN:** Test case covering `CPU manager`. It exercises `CPUOffloadingManager, cpu_manager.prepare_store, verify_store_output, cpu_manager.complete_store, verify_events, cpu_manager.prepare_load`. The body contains 16 explicit assertion(s).
**CN:** 该代码块是覆盖 `cpu manager` 的测试用例。 该测试会调用 `CPUOffloadingManager, cpu_manager.prepare_store, verify_store_output, cpu_manager.complete_store, verify_events, cpu_manager.prepare_load`。 代码主体包含 16 个显式断言。

### test_prepare_load_preserves_key_order (lines 265-299)
```python
def test_prepare_load_preserves_key_order():
    """block_ids[i] must correspond to keys[i] (co-indexed invariant)."""
    manager = CPUOffloadingManager(num_blocks=4, cache_policy="lru")

    key_a, key_b, key_c = to_key(0), to_key(1), to_key(2)
    # Store all three keys and learn their block ID assignments
    store_output = manager.prepare_store([key_a, key_b, key_c], _EMPTY_REQ_CTX)
    assert store_output is not None
    assert isinstance(store_output.store_spec, CPULoadStoreSpec)
    key_to_block_id = {
        k: int(bid)
        for k, bid in zip(store_output.keys_to_store, store_output.store_spec.block_ids)
    }
    manager.complete_store([key_a, key_b, key_c], _EMPTY_REQ_CTX)
    # Forward order: [a, b, c]
    spec_fwd = manager.prepare_load([key_a, key_b, key_c], _EMPTY_REQ_CTX)
    # ... excerpt omitted for brevity ...
    assert isinstance(spec_fwd, CPULoadStoreSpec)
    assert [int(x) for x in spec_fwd.block_ids] == [
    assert isinstance(spec_perm, CPULoadStoreSpec)
    assert [int(x) for x in spec_perm.block_ids] == [
        key_to_block_id[key_b],
        key_to_block_id[key_c],
        key_to_block_id[key_a],
    ]
    manager.complete_load([key_a, key_b, key_c], _EMPTY_REQ_CTX)  # order irrelevant
```
**EN:** Test case covering `prepare load preserves key order`. It exercises `CPUOffloadingManager, manager.prepare_store, isinstance, manager.complete_store, manager.prepare_load, manager.complete_load`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `prepare load preserves key order` 的测试用例。 该测试会调用 `CPUOffloadingManager, manager.prepare_store, isinstance, manager.complete_store, manager.prepare_load, manager.complete_load`。 代码主体包含 6 个显式断言。

### TestARCPolicy (lines 302-601)
```python
class TestARCPolicy:
    """Unit tests for CPUOffloadingManager with ARC eviction policy."""

    def _make_manager(
        self, num_blocks: int = 4, enable_events: bool = True
    ) -> tuple[CPUOffloadingManager, ARCCachePolicy]:
        manager = CPUOffloadingManager(
            num_blocks=num_blocks,
            cache_policy="arc",
            enable_events=enable_events,
        )
        policy = manager._policy
        assert isinstance(policy, ARCCachePolicy)
        return manager, policy
    def test_basic(self):
        """
        Tests CPUOffloadingManager with arc policy.
    # ... excerpt omitted for brevity ...
        assert cpu_manager.lookup(to_key(1), _EMPTY_REQ_CTX) is None
        assert cpu_manager.lookup(to_key(2), _EMPTY_REQ_CTX) is None
        assert list(cpu_manager.take_events()) == []
        verify_events(cpu_manager.take_events(), expected_stores=({1, 2},))
        assert cpu_manager.lookup(to_key(1), _EMPTY_REQ_CTX) is True
        assert cpu_manager.lookup(to_key(2), _EMPTY_REQ_CTX) is True
        assert cpu_manager.lookup(to_key(3), _EMPTY_REQ_CTX) is True
        # verify events
        events = list(cpu_manager.take_events())
        assert len(events) > 0  # should have store and eviction events
```
**EN:** Class `TestARCPolicy` groups 9 test method(s) and 1 helper/fixture method(s). Representative scenarios: `test_basic, test_t1_to_t2_promotion, test_eviction_with_load, test_adaptive_target, test_t1_t2_eviction_policy, test_ghost_list_bounds, ...`.
**CN:** 类 `TestARCPolicy` 组织了 9 个测试方法，以及 1 个辅助或 fixture 方法。 代表性场景：`test_basic, test_t1_to_t2_promotion, test_eviction_with_load, test_adaptive_target, test_t1_t2_eviction_policy, test_ghost_list_bounds, ...`。

### test_filter_reused_manager (lines 604-650)
```python
def test_filter_reused_manager():
    """
    Tests CPUOffloadingManager reuse filtering (store_threshold=2).
    manager = CPUOffloadingManager(
        num_blocks=4,
        cache_policy="lru",
        enable_events=True,
        store_threshold=2,
        max_tracker_size=3,
    )

    # Lookup [1, 2] -> 1st time, added to tracker but not eligible for store yet
    assert manager.lookup(to_key(1), _EMPTY_REQ_CTX) is False
    assert manager.lookup(to_key(2), _EMPTY_REQ_CTX) is False
    # prepare store [1, 2] -> should be filtered
    prepare_store_output = manager.prepare_store(to_keys([1, 2]), _EMPTY_REQ_CTX)
    # ... excerpt omitted for brevity ...
    assert prepare_store_output is not None
    assert prepare_store_output.keys_to_store == []
    assert prepare_store_output.keys_to_store == to_keys([1])
    assert manager.lookup(to_key(3), _EMPTY_REQ_CTX) is False
    # prepare store [2] -> should still be filtered out since count was reset
    prepare_store_output = manager.prepare_store(to_keys([2]), _EMPTY_REQ_CTX)
    manager.complete_store(to_keys([1]), _EMPTY_REQ_CTX)
```
**EN:** Test case covering `filter reused manager`. It exercises `CPUOffloadingManager, manager.prepare_store, manager.complete_store, manager.lookup, to_keys, counts.get`. The body contains 14 explicit assertion(s).
**CN:** 该代码块是覆盖 `filter reused manager` 的测试用例。 该测试会调用 `CPUOffloadingManager, manager.prepare_store, manager.complete_store, manager.lookup, to_keys, counts.get`。 代码主体包含 14 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest`.
- **CN:** 外部库：`numpy, pytest`。
- **EN:** vLLM modules under test: `vllm.v1.kv_offload.base, vllm.v1.kv_offload.cpu.common, vllm.v1.kv_offload.cpu.manager, vllm.v1.kv_offload.cpu.policies.arc`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.kv_offload.base, vllm.v1.kv_offload.cpu.common, vllm.v1.kv_offload.cpu.manager, vllm.v1.kv_offload.cpu.policies.arc`。
- **EN:** Standard-library support: `collections.abc, dataclasses`.
- **CN:** 标准库支持：`collections.abc, dataclasses`。
