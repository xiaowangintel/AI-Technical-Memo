# test_mooncake_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_mooncake_stats.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `mooncake stats` behavior and regressions in the v1 stack. / 验证 v1 栈中 `mooncake stats` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-14)
```python
import threading
from unittest.mock import MagicMock

from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector import (
    MooncakeConnector,
    MooncakeConnectorWorker,
    SendBlockMeta,
)
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.stats import (
    MooncakeKVConnectorStats,
)
```
**EN:** Imports the libraries needed to build the test harness. vLLM modules under test include `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.stats`.
**CN:** 该代码块导入构建测试环境所需的库。 被测试的 vLLM 模块包括 `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.stats`。

### test_is_empty_on_fresh_stats (lines 17-20)
```python
def test_is_empty_on_fresh_stats():
    stats = MooncakeKVConnectorStats()
    assert stats.is_empty()
    assert stats.num_successful_transfers == 0
```
**EN:** Test case covering `is empty on fresh stats`. It exercises `MooncakeKVConnectorStats, stats.is_empty`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `is empty on fresh stats` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, stats.is_empty`。 代码主体包含 2 个显式断言。

### test_record_transfer_and_reduce (lines 23-42)
```python
def test_record_transfer_and_reduce():
    stats = MooncakeKVConnectorStats()
    # 1 MB transfer in 1 ms -> 1000 MB/s throughput
    stats.record_transfer(duration_s=0.001, total_bytes=1 * 2**20, num_descs=4)
    # 2 MB transfer in 2 ms
    stats.record_transfer(duration_s=0.002, total_bytes=2 * 2**20, num_descs=6)
    assert not stats.is_empty()
    assert stats.num_successful_transfers == 2

    reduced = stats.reduce()
    assert reduced["Num successful transfers"] == 2
    # avg = (1 + 2) / 2 = 1.5 ms
    assert reduced["Avg xfer time (ms)"] == 1.5
    assert reduced["Avg MB per transfer"] == 1.5
    # 3 MB total / 3 ms total = 1000 MB/s
    assert reduced["Throughput (MB/s)"] == 1000.0
    assert reduced["Avg number of descriptors"] == 5.0
    assert reduced["Num failed transfers"] == 0
    assert reduced["Num failed recvs"] == 0
    assert reduced["Num KV expired reqs"] == 0
```
**EN:** Test case covering `record transfer and reduce`. It exercises `MooncakeKVConnectorStats, stats.record_transfer, stats.reduce, stats.is_empty`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `record transfer and reduce` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, stats.record_transfer, stats.reduce, stats.is_empty`。 代码主体包含 10 个显式断言。

### test_record_failures_keeps_stats_non_empty (lines 45-58)
```python
def test_record_failures_keeps_stats_non_empty():
    stats = MooncakeKVConnectorStats()
    stats.record_failed_transfer()
    stats.record_failed_recv()
    stats.record_kv_expired_req()
    assert not stats.is_empty()

    reduced = stats.reduce()
    # No successful transfers -> latency/throughput all zero, but failure
    # counters still surface.
    assert reduced["Num successful transfers"] == 0
    assert reduced["Num failed transfers"] == 1
    assert reduced["Num failed recvs"] == 1
    assert reduced["Num KV expired reqs"] == 1
```
**EN:** Test case covering `record failures keeps stats non empty`. It exercises `MooncakeKVConnectorStats, stats.record_failed_transfer, stats.record_failed_recv, stats.record_kv_expired_req, stats.reduce, stats.is_empty`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `record failures keeps stats non empty` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, stats.record_failed_transfer, stats.record_failed_recv, stats.record_kv_expired_req, stats.reduce, stats.is_empty`。 代码主体包含 5 个显式断言。

### test_aggregate_sums_observations (lines 61-73)
```python
def test_aggregate_sums_observations():
    a = MooncakeKVConnectorStats()
    b = MooncakeKVConnectorStats()
    a.record_transfer(duration_s=0.001, total_bytes=1 * 2**20, num_descs=1)
    b.record_transfer(duration_s=0.002, total_bytes=2 * 2**20, num_descs=2)
    b.record_failed_transfer()

    a.aggregate(b)

    assert a.num_successful_transfers == 2
    reduced = a.reduce()
    assert reduced["Num successful transfers"] == 2
    assert reduced["Num failed transfers"] == 1
```
**EN:** Test case covering `aggregate sums observations`. It exercises `MooncakeKVConnectorStats, a.record_transfer, b.record_transfer, b.record_failed_transfer, a.aggregate, a.reduce`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `aggregate sums observations` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, a.record_transfer, b.record_transfer, b.record_failed_transfer, a.aggregate, a.reduce`。 代码主体包含 3 个显式断言。

### test_aggregate_with_empty_other_is_noop (lines 76-83)
```python
def test_aggregate_with_empty_other_is_noop():
    a = MooncakeKVConnectorStats()
    a.record_transfer(duration_s=0.001, total_bytes=1, num_descs=1)
    b = MooncakeKVConnectorStats()

    a.aggregate(b)

    assert a.num_successful_transfers == 1
```
**EN:** Test case covering `aggregate with empty other is noop`. It exercises `MooncakeKVConnectorStats, a.record_transfer, a.aggregate`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `aggregate with empty other is noop` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, a.record_transfer, a.aggregate`。 代码主体包含 1 个显式断言。

### test_getstate_drops_lock_and_setstate_recreates_it (lines 86-101)
```python
def test_getstate_drops_lock_and_setstate_recreates_it():
    # KVConnectorStats subclasses must be picklable (worker→scheduler IPC),
    # but threading.Lock isn't — so __getstate__ strips it and __setstate__
    # rebuilds a fresh per-process lock.
    original = MooncakeKVConnectorStats()
    original.record_transfer(duration_s=0.01, total_bytes=2048, num_descs=3)

    state = original.__getstate__()
    assert "_lock" not in state

    rebuilt = MooncakeKVConnectorStats.__new__(MooncakeKVConnectorStats)
    rebuilt.__setstate__(state)
    assert rebuilt.data == original.data
    # Lock works on the receiver side.
    rebuilt.record_transfer(duration_s=0.02, total_bytes=4096, num_descs=5)
    assert rebuilt.num_successful_transfers == 2
```
**EN:** Test case covering `getstate drops lock and setstate recreates it`. It exercises `MooncakeKVConnectorStats, original.record_transfer, original.__getstate__, MooncakeKVConnectorStats.__new__, rebuilt.__setstate__, rebuilt.record_transfer`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `getstate drops lock and setstate recreates it` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, original.record_transfer, original.__getstate__, MooncakeKVConnectorStats.__new__, rebuilt.__setstate__, rebuilt.record_transfer`。 代码主体包含 3 个显式断言。

### test_concurrent_writers_keep_row_lengths_aligned (lines 104-158)
```python
def test_concurrent_writers_keep_row_lengths_aligned():
    # Multiple writers + a snapshot reader must never produce a snapshot
    # with mismatched column lengths — reduce()'s
    # len(descs) == num_successful_transfers assertion would fire.
    stats = MooncakeKVConnectorStats()
    stop = threading.Event()
    writer_count = 4
    snapshots: list[MooncakeKVConnectorStats] = []

    def writer():
        i = 0
        while not stop.is_set():
            stats.record_transfer(
                duration_s=0.001 + i * 1e-9,
                total_bytes=1024 + i,
                num_descs=1 + (i % 8),
            )
            i += 1
    # ... excerpt omitted for brevity ...
    for snap in snapshots:
        n = len(snap.data["transfer_duration"])
        assert len(snap.data["bytes_transferred"]) == n
        assert len(snap.data["num_descriptors"]) == n
        total_rows += n
    assert total_rows > 0
```
**EN:** Test case covering `concurrent writers keep row lengths aligned`. It exercises `MooncakeKVConnectorStats, threading.Event, threading.Thread, snapshotter.start, Event.wait, stop.set`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `concurrent writers keep row lengths aligned` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, threading.Event, threading.Thread, snapshotter.start, Event.wait, stop.set`。 代码主体包含 3 个显式断言。

### test_clone_and_reset_hands_off_old_data (lines 161-175)
```python
def test_clone_and_reset_hands_off_old_data():
    stats = MooncakeKVConnectorStats()
    stats.record_transfer(duration_s=0.001, total_bytes=1, num_descs=1)
    stats.record_failed_recv()

    snapshot = stats.clone_and_reset()

    assert snapshot.num_successful_transfers == 1
    assert not snapshot.is_empty()
    # Original is now empty.
    assert stats.is_empty()
    assert stats.num_successful_transfers == 0
    # Recording on the original does not mutate the snapshot.
    stats.record_transfer(duration_s=0.005, total_bytes=2, num_descs=2)
    assert snapshot.num_successful_transfers == 1
```
**EN:** Test case covering `clone and reset hands off old data`. It exercises `MooncakeKVConnectorStats, stats.record_transfer, stats.record_failed_recv, stats.clone_and_reset, stats.is_empty, snapshot.is_empty`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `clone and reset hands off old data` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, stats.record_transfer, stats.record_failed_recv, stats.clone_and_reset, stats.is_empty, snapshot.is_empty`。 代码主体包含 5 个显式断言。

### test_build_kv_connector_stats_none_returns_empty_instance (lines 178-181)
```python
def test_build_kv_connector_stats_none_returns_empty_instance():
    out = MooncakeConnector.build_kv_connector_stats()
    assert isinstance(out, MooncakeKVConnectorStats)
    assert out.is_empty()
```
**EN:** Test case covering `build KV connector stats none returns empty instance`. It exercises `MooncakeConnector.build_kv_connector_stats, isinstance, out.is_empty`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `build KV 连接器 stats none returns empty instance` 的测试用例。 该测试会调用 `MooncakeConnector.build_kv_connector_stats, isinstance, out.is_empty`。 代码主体包含 2 个显式断言。

### test_build_kv_connector_stats_with_data_round_trips (lines 184-195)
```python
def test_build_kv_connector_stats_with_data_round_trips():
    original = MooncakeKVConnectorStats()
    original.record_transfer(duration_s=0.01, total_bytes=1024, num_descs=3)
    original.record_failed_transfer()

    # Serialized form is the .data dict; build should reconstruct an instance
    # that behaves the same.
    rebuilt = MooncakeConnector.build_kv_connector_stats(data=original.data)

    assert isinstance(rebuilt, MooncakeKVConnectorStats)
    assert rebuilt.num_successful_transfers == 1
    assert rebuilt.reduce()["Num failed transfers"] == 1
```
**EN:** Test case covering `build KV connector stats with data round trips`. It exercises `MooncakeKVConnectorStats, original.record_transfer, original.record_failed_transfer, MooncakeConnector.build_kv_connector_stats, isinstance, rebuilt.reduce`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `build KV 连接器 stats with data round trips` 的测试用例。 该测试会调用 `MooncakeKVConnectorStats, original.record_transfer, original.record_failed_transfer, MooncakeConnector.build_kv_connector_stats, isinstance, rebuilt.reduce`。 代码主体包含 3 个显式断言。

### _bare_worker (lines 198-209)
```python
def _bare_worker() -> MooncakeConnectorWorker:
    """Construct a MooncakeConnectorWorker skipping __init__ (full init requires
    a live TransferEngine). Only the attributes touched by the methods under
    test are populated; role flags and async_zmq_ctx keep __del__'s shutdown
    path a no-op."""
    worker = MooncakeConnectorWorker.__new__(MooncakeConnectorWorker)
    worker.xfer_stats = MooncakeKVConnectorStats()
    worker.engine = MagicMock()
    worker.async_zmq_ctx = MagicMock()
    worker.is_kv_consumer = True
    worker.is_kv_producer = True
    return worker
```
**EN:** Helper function `_bare_worker` encapsulates reusable logic for `bare worker`. Key calls include `MooncakeConnectorWorker.__new__, MooncakeKVConnectorStats, MagicMock`.
**CN:** 辅助函数 `_bare_worker` 封装了与 `bare worker` 相关的可复用逻辑。 关键调用包括 `MooncakeConnectorWorker.__new__, MooncakeKVConnectorStats, MagicMock`。

### test_send_blocks_records_success (lines 212-228)
```python
def test_send_blocks_records_success():
    worker = _bare_worker()
    worker.engine.batch_transfer_sync_write.return_value = 0

    ret = worker._send_blocks(
        "host:1234",
        src_ptrs=[0x1000, 0x2000],
        dst_ptrs=[0x3000, 0x4000],
        lengths=[1024, 2048],
    )

    assert ret == 0
    assert worker.xfer_stats.num_successful_transfers == 1
    data = worker.xfer_stats.data
    assert data["bytes_transferred"] == [1024 + 2048]
    assert data["num_descriptors"] == [2]
    assert data["num_failed_transfers"] == []
```
**EN:** Test case covering `send blocks records success`. It exercises `_bare_worker, worker._send_blocks`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `send blocks records success` 的测试用例。 该测试会调用 `_bare_worker, worker._send_blocks`。 代码主体包含 5 个显式断言。

### test_send_blocks_records_failure (lines 231-239)
```python
def test_send_blocks_records_failure():
    worker = _bare_worker()
    worker.engine.batch_transfer_sync_write.return_value = 1  # non-zero = fail

    ret = worker._send_blocks("host:1234", [0x1000], [0x2000], [4096])

    assert ret == 1
    assert worker.xfer_stats.num_successful_transfers == 0
    assert worker.xfer_stats.data["num_failed_transfers"] == [1]
```
**EN:** Test case covering `send blocks records failure`. It exercises `_bare_worker, worker._send_blocks`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `send blocks records failure` 的测试用例。 该测试会调用 `_bare_worker, worker._send_blocks`。 代码主体包含 3 个显式断言。

### test_get_kv_connector_stats_returns_none_when_empty (lines 242-245)
```python
def test_get_kv_connector_stats_returns_none_when_empty():
    worker = _bare_worker()

    assert worker.get_kv_connector_stats() is None
```
**EN:** Test case covering `get KV connector stats returns none when empty`. It exercises `_bare_worker, worker.get_kv_connector_stats`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 连接器 stats returns none when empty` 的测试用例。 该测试会调用 `_bare_worker, worker.get_kv_connector_stats`。 代码主体包含 1 个显式断言。

### test_get_kv_connector_stats_returns_and_resets (lines 248-258)
```python
def test_get_kv_connector_stats_returns_and_resets():
    worker = _bare_worker()
    worker.engine.batch_transfer_sync_write.return_value = 0
    worker._send_blocks("host:1234", [0x1000], [0x2000], [4096])

    snapshot = worker.get_kv_connector_stats()
    assert isinstance(snapshot, MooncakeKVConnectorStats)
    assert snapshot.num_successful_transfers == 1

    # Second call returns None because the worker's stats were reset.
    assert worker.get_kv_connector_stats() is None
```
**EN:** Test case covering `get KV connector stats returns and resets`. It exercises `_bare_worker, worker._send_blocks, worker.get_kv_connector_stats, isinstance`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 连接器 stats returns and resets` 的测试用例。 该测试会调用 `_bare_worker, worker._send_blocks, worker.get_kv_connector_stats, isinstance`。 代码主体包含 3 个显式断言。

### test_expired_request_bumps_counter (lines 261-281)
```python
def test_expired_request_bumps_counter():
    import asyncio

    worker = _bare_worker()
    worker.reqs_need_send = {
        "tid1": SendBlockMeta(
            p_req_id="req1",
            transfer_id="tid1",
            local_block_ids=[0, 1],
            ready=asyncio.Event(),
            expire_time=-1.0,  # Already expired.
            sending=0,
        ),
    }
    worker.finished_sending_reqs = set()

    asyncio.run(worker.fetch_finished_sending_reqs())

    assert worker.xfer_stats.data["num_kv_expired_reqs"] == [1]
    # Expired transfer also cleaned out of reqs_need_send.
    assert "tid1" not in worker.reqs_need_send
```
**EN:** Test case covering `expired request bumps counter`. It exercises `_bare_worker, set, asyncio.run, SendBlockMeta, worker.fetch_finished_sending_reqs, asyncio.Event`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `expired request bumps counter` 的测试用例。 该测试会调用 `_bare_worker, set, asyncio.run, SendBlockMeta, worker.fetch_finished_sending_reqs, asyncio.Event`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同

## Dependencies / 依赖关系
- **EN:** vLLM modules under test: `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.stats`.
- **CN:** 被测试的 vLLM 模块：`vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.stats`。
- **EN:** Standard-library support: `threading, unittest.mock, asyncio`.
- **CN:** 标准库支持：`threading, unittest.mock, asyncio`。
