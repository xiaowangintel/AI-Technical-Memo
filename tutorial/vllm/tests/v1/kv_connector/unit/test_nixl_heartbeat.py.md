# test_nixl_heartbeat.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_nixl_heartbeat.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for the scheduler-driven heartbeat / lease-renewal system. / 该文件的文档字符串表明其用途：`unit tests for the scheduler-driven heartbeat / lease-renewal system`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 3)
```python
"""Unit tests for the scheduler-driven heartbeat / lease-renewal system."""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for the scheduler-driven heartbeat / lease-renewal system.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for the scheduler-driven heartbeat / lease-renewal system`。

### Imports and setup / 导入与设置 (lines 5-12)
```python
import time
from unittest.mock import MagicMock

import pytest

from vllm.v1.outputs import KVConnectorOutput

from .utils import create_request, make_nixl_scheduler
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.v1.outputs, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.v1.outputs, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### Module state / 模块级状态 (line 14)
```python
_ENGINE_A = "my-engine-id"
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_ENGINE_A`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_ENGINE_A`。

### _sched (lines 17-18)
```python
def _sched(kv_lease_duration: int = 30):
    return make_nixl_scheduler(heartbeat=True, kv_lease_duration=kv_lease_duration)
```
**EN:** Helper function `_sched` encapsulates reusable logic for `sched`. Inputs: `kv_lease_duration`. Key calls include `make_nixl_scheduler`.
**CN:** 辅助函数 `_sched` 封装了与 `sched` 相关的可复用逻辑。 输入参数：`kv_lease_duration`。 关键调用包括 `make_nixl_scheduler`。

### _req (lines 21-22)
```python
def _req(request_id: int = 1):
    return create_request(request_id=request_id, do_remote_prefill=True)
```
**EN:** Helper function `_req` encapsulates reusable logic for `req`. Inputs: `request_id`. Key calls include `create_request`.
**CN:** 辅助函数 `_req` 封装了与 `req` 相关的可复用逻辑。 输入参数：`request_id`。 关键调用包括 `create_request`。

### _worker_stub (lines 25-33)
```python
def _worker_stub():
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker import (
        NixlConnectorWorker,
    )

    w = object.__new__(NixlConnectorWorker)
    w._reqs_to_send = {}
    w._lease_extension = 20
    return w
```
**EN:** Helper function `_worker_stub` encapsulates reusable logic for `worker stub`. Key calls include `object.__new__`.
**CN:** 辅助函数 `_worker_stub` 封装了与 `worker stub` 相关的可复用逻辑。 关键调用包括 `object.__new__`。

### test_on_new_request_tracks_and_groups (lines 41-56)
```python
def test_on_new_request_tracks_and_groups():
    """Add two reqs to same engine, one to another; verify grouping."""
    s = _sched()
    s.on_new_request(_req(1))
    s.on_new_request(_req(2))

    assert s._heartbeat_by_engine[_ENGINE_A].req_ids == {"prefill-1", "prefill-2"}
    info = s._heartbeat_by_engine[_ENGINE_A]
    assert (info.host, info.port, info.tp_size) == ("my-host", 1234, 1)
    assert s._heartbeat_req_engine["id-1"] == (_ENGINE_A, "prefill-1")

    # Different engine.
    r3 = _req(3)
    r3.kv_transfer_params["remote_engine_id"] = "engine-b"
    s.on_new_request(r3)
    assert len(s._heartbeat_by_engine) == 2
```
**EN:** Test case covering `on new request tracks and groups`. It exercises `_sched, s.on_new_request, _req, len`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `on new request tracks and groups` 的测试用例。 该测试会调用 `_sched, s.on_new_request, _req, len`。 代码主体包含 4 个显式断言。

### test_on_new_request_ignores_non_prefill (lines 67-70)
```python
def test_on_new_request_ignores_non_prefill(make_req):
    s = _sched()
    s.on_new_request(make_req())
    assert len(s._heartbeat_by_engine) == 0
```
**EN:** Parameterized test covering `on new request ignores non prefill`. Parameter axes: `make_req`. Inputs/fixtures: `make_req`. It exercises `mark.parametrize, _sched, s.on_new_request, make_req, len, create_request`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `on new request ignores non prefill` 的测试用例。 参数维度：`make_req`。 输入或 fixture：`make_req`。 该测试会调用 `mark.parametrize, _sched, s.on_new_request, make_req, len, create_request`。 代码主体包含 1 个显式断言。

### test_stop_heartbeat_partial_and_full (lines 78-90)
```python
def test_stop_heartbeat_partial_and_full():
    """Stop one of two reqs on same engine, then stop the other."""
    s = _sched()
    s.on_new_request(_req(1))
    s.on_new_request(_req(2))

    s._stop_heartbeat("id-1")
    assert s._heartbeat_by_engine[_ENGINE_A].req_ids == {"prefill-2"}
    assert "id-1" not in s._heartbeat_req_engine

    s._stop_heartbeat("id-2")
    assert len(s._heartbeat_by_engine) == 0
    assert len(s._heartbeat_req_engine) == 0
```
**EN:** Test case covering `stop heartbeat partial and full`. It exercises `_sched, s.on_new_request, s._stop_heartbeat, _req, len`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `stop heartbeat partial and full` 的测试用例。 该测试会调用 `_sched, s.on_new_request, s._stop_heartbeat, _req, len`。 代码主体包含 4 个显式断言。

### test_build_connector_meta_heartbeat_throttling (lines 98-110)
```python
def test_build_connector_meta_heartbeat_throttling():
    # kv_lease_duration=30 => _heartbeat_interval = 30 // 6 = 5
    s = _sched(kv_lease_duration=30)
    s.on_new_request(_req(1))

    # Ensure the first call triggers by placing last_heartbeat far in the past.
    s._last_heartbeat_time = time.perf_counter() - 10
    meta1 = s.build_connector_meta(MagicMock())
    assert _ENGINE_A in meta1.heartbeat_by_engine

    # Immediate second call is throttled (< 5s since last).
    meta2 = s.build_connector_meta(MagicMock())
    assert len(meta2.heartbeat_by_engine) == 0
```
**EN:** Test case covering `build connector meta heartbeat throttling`. It exercises `_sched, s.on_new_request, s.build_connector_meta, _req, time.perf_counter, MagicMock`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `build connector meta heartbeat throttling` 的测试用例。 该测试会调用 `_sched, s.on_new_request, s.build_connector_meta, _req, time.perf_counter, MagicMock`。 代码主体包含 2 个显式断言。

### test_update_connector_output_stops_heartbeat (lines 118-131)
```python
def test_update_connector_output_stops_heartbeat():
    s = _sched()
    s.on_new_request(_req(1))

    s.update_connector_output(
        KVConnectorOutput(
            finished_sending=None,
            finished_recving={"id-1"},
            invalid_block_ids=set(),
        )
    )

    assert len(s._heartbeat_by_engine) == 0
    assert len(s._heartbeat_req_engine) == 0
```
**EN:** Test case covering `update connector output stops heartbeat`. It exercises `_sched, s.on_new_request, s.update_connector_output, _req, KVConnectorOutput, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `update connector output stops heartbeat` 的测试用例。 该测试会调用 `_sched, s.on_new_request, s.update_connector_output, _req, KVConnectorOutput, len`。 代码主体包含 2 个显式断言。

### test_request_finished_stops_heartbeat (lines 134-144)
```python
def test_request_finished_stops_heartbeat():
    s = _sched()
    r = _req(1)
    s.on_new_request(r)

    # Simulate update_state_after_alloc having consumed do_remote_prefill.
    r.kv_transfer_params["do_remote_prefill"] = False
    s.request_finished(r, block_ids=())

    assert len(s._heartbeat_by_engine) == 0
    assert len(s._heartbeat_req_engine) == 0
```
**EN:** Test case covering `request finished stops heartbeat`. It exercises `_sched, _req, s.on_new_request, s.request_finished, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `request finished stops heartbeat` 的测试用例。 该测试会调用 `_sched, _req, s.on_new_request, s.request_finished, len`。 代码主体包含 2 个显式断言。

### test_handle_heartbeat (lines 152-165)
```python
def test_handle_heartbeat():
    w = _worker_stub()
    far_future = time.perf_counter() + 99999
    w._reqs_to_send = {"req-a": 100.0, "req-b": far_future}

    before = time.perf_counter()
    w._handle_heartbeat("req-a,req-b,req-unknown")

    # req-a: pushed forward to ~now+20.
    assert w._reqs_to_send["req-a"] >= before + 20
    # req-b: already far out, max() keeps it.
    assert w._reqs_to_send["req-b"] >= far_future
    # req-unknown: not added.
    assert "req-unknown" not in w._reqs_to_send
```
**EN:** Test case covering `handle heartbeat`. It exercises `_worker_stub, time.perf_counter, w._handle_heartbeat`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `handle heartbeat` 的测试用例。 该测试会调用 `_worker_stub, time.perf_counter, w._handle_heartbeat`。 代码主体包含 3 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.v1.outputs, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.outputs, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `time, unittest.mock`.
- **CN:** 标准库支持：`time, unittest.mock`。
