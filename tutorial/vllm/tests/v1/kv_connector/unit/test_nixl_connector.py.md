# test_nixl_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_nixl_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `NIXL connector` behavior and regressions in the v1 stack. / 验证 v1 栈中 `nixl connector` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-75)
```python
import contextlib
import inspect
import os
import tempfile
import textwrap
import time
import uuid
from collections import defaultdict
from typing import Any, cast
from unittest.mock import MagicMock, patch

import msgspec
import pytest
import ray
import torch
from vllm import LLM
from vllm.config import KVTransferConfig, set_current_vllm_config
# ... excerpt omitted for brevity ...
from .utils import (
    create_request,
    create_scheduler,
    create_vllm_config,
    make_kv_cache_config,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `msgspec, pytest, ray, torch, zmq.error`. vLLM modules under test include `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.utils, vllm.distributed.kv_transfer.kv_connector.v1, vllm.distributed.kv_transfer.kv_connector.v1.base, ...`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `msgspec, pytest, ray, torch, zmq.error`。 被测试的 vLLM 模块包括 `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.utils, vllm.distributed.kv_transfer.kv_connector.v1, vllm.distributed.kv_transfer.kv_connector.v1.base, ...`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### clear_kv_transfer (lines 79-98)
```python
def clear_kv_transfer():
    """
    The test cases in this file use `VLLM_ENABLE_V1_MULTIPROCESSING=0`,
    causing the global variable `_KV_CONNECTOR_AGENT`
    to be assigned but never deleted.

    Since the current pytest process does not terminate and instead
    continues running tests from other files,
    this global variable remains in memory and interferes
    with test cases in other modules.

    So we use this fixture to ensure that the global variable
    `_KV_CONNECTOR_AGENT` is properly cleaned up after each test.
    """
    yield
    if has_kv_transfer_group():
        ensure_kv_transfer_shutdown()
    # Reset any KV cache layout override set during tests so it doesn't
    # leak into tests in other modules.
    set_kv_cache_layout(None)
```
**EN:** Fixture/helper `clear_kv_transfer` prepares reusable state for downstream tests. Key calls include `pytest.fixture, has_kv_transfer_group, set_kv_cache_layout, ensure_kv_transfer_shutdown`.
**CN:** `clear_kv_transfer` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `pytest.fixture, has_kv_transfer_group, set_kv_cache_layout, ensure_kv_transfer_shutdown`。

### get_default_xfer_telemetry (lines 101-119)
```python
def get_default_xfer_telemetry(
    xferDurationS: float = 1,
    postDurationS: float = 1,
    totalBytes: int = 1,
    descCount: int = 1,
) -> dict:
    class AttributeDict(dict):
        __slots__ = ()
        __getattr__ = dict.__getitem__
        __setattr__ = dict.__setitem__  # type: ignore[assignment]

    # We can't instantiate nixlXferTelemetry because it's read only and
    # ray env does not have NIXL, so we must fake it
    return AttributeDict(
        xferDuration=xferDurationS * 1e6,  # in us
        postDuration=postDurationS * 1e6,  # in us
        totalBytes=totalBytes,
        descCount=descCount,
    )
```
**EN:** Helper function `get_default_xfer_telemetry` encapsulates reusable logic for `default xfer telemetry`. Inputs: `xferDurationS, postDurationS, totalBytes, descCount`. Key calls include `AttributeDict`.
**CN:** 辅助函数 `get_default_xfer_telemetry` 封装了与 `default xfer telemetry` 相关的可复用逻辑。 输入参数：`xferDurationS, postDurationS, totalBytes, descCount`。 关键调用包括 `AttributeDict`。

### FakeNixlWrapper (lines 122-205)
```python
class FakeNixlWrapper:
    """Mock implementation of NixlWrapper for testing.

    We don't inherit from nixl._api.nixl_agent because nixl may not be
    installed.
    Note: The complete source of this class is also used in the
    `_make_fake_nixl_pkg` function to create a fake nixl package
    for Ray workers.
    """
    AGENT_METADATA = b"fake_agent_metadata"
    REMOTE_AGENT_NAME = "remote_agent"
    def __init__(self, agent_name: str, *args, **kwargs):
        self._cycles_before_xfer_done = 0
        self._check_xfer_state_cycles: defaultdict[int, int] = defaultdict(lambda: 0)
    # ... excerpt omitted for brevity ...
        return [str(uuid.uuid4()) for _ in caches_data]
        return [str(uuid.uuid4()) for _ in blocks_data]
        return uuid.uuid4().int
        return self.AGENT_METADATA
        return self.REMOTE_AGENT_NAME
        return {}
    ############################################################
    # Follow are for changing the behavior during testing.
    def set_cycles_before_xfer_done(self, cycles: int):
        """Set the number of cycles before a transfer is considered done."""
```
**EN:** Class `FakeNixlWrapper` groups 0 test method(s) and 18 helper/fixture method(s).
**CN:** 类 `FakeNixlWrapper` 组织了 0 个测试方法，以及 18 个辅助或 fixture 方法。

### _make_fake_nixl_pkg (lines 209-247)
```python
def _make_fake_nixl_pkg():
    """Context manager that creates a temporary package making
       `from nixl._api import nixl_agent` resolve to our FakeNixlWrapper.
       Also creates rixl package for ROCm compatibility.

    Automatically cleans up the temporary directory when done.
    """
    with tempfile.TemporaryDirectory() as td:
        # Create both nixl and rixl packages for cross-platform compatibility
        for pkg_name in ["nixl", "rixl"]:
            pkg_root = os.path.join(td, pkg_name, "_api")
            os.makedirs(pkg_root, exist_ok=True)
            # Get the source code of FakeNixlWrapper class and dedent it
            fake_nixl_source = inspect.getsource(FakeNixlWrapper)
            fake_nixl_source = textwrap.dedent(fake_nixl_source)
            stub = f"""\
    # ... excerpt omitted for brevity ...
            with open(os.path.join(pkg_root2, "__init__.py"), "w") as f:
                f.write("class nixlXferTelemetry: pass")
            # touch parent package
            open(os.path.join(td, pkg_name, "__init__.py"), "w").close()
        yield td
```
**EN:** Helper function `_make_fake_nixl_pkg` encapsulates reusable logic for `fakeNIXL pkg`. Key calls include `tempfile.TemporaryDirectory, path.join, os.makedirs, inspect.getsource, textwrap.dedent, open.close`.
**CN:** 辅助函数 `_make_fake_nixl_pkg` 封装了与 `fakenixl pkg` 相关的可复用逻辑。 关键调用包括 `tempfile.TemporaryDirectory, path.join, os.makedirs, inspect.getsource, textwrap.dedent, open.close`。

### test_basic_interface (lines 250-287)
```python
def test_basic_interface():
    """Unit test for basic NixlConnector interface functionality."""

    vllm_config = create_vllm_config()
    scheduler = create_scheduler(vllm_config)
    # 2 Full Blocks and 1 Half Block.
    BLOCK_SIZE = vllm_config.cache_config.block_size
    NUM_EXTERNAL_FULL_BLOCKS = 2
    NUM_TOKENS = int(BLOCK_SIZE * (NUM_EXTERNAL_FULL_BLOCKS + 0.5))
    request = create_request(
        request_id=1,
        block_size=BLOCK_SIZE,
        num_tokens=NUM_TOKENS,
        do_remote_prefill=True,
    )
    request_id = request.request_id
    # ... excerpt omitted for brevity ...
    assert kv_connector_metadata is not None
    assert isinstance(kv_connector_metadata, NixlConnectorMetadata)
    assert len(kv_connector_metadata.reqs_to_recv) == 1
    assert request_id in kv_connector_metadata.reqs_to_recv
        req_meta.local_block_ids[0],
        scheduler.kv_cache_manager.coordinator.single_type_managers[0].req_to_blocks[
            request_id
        ],
    ):
        assert block_id == block.block_id
```
**EN:** Test case covering `basic interface`. It exercises `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `basic interface` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 5 个显式断言。

### test_prompt_less_than_block_size (lines 290-321)
```python
def test_prompt_less_than_block_size():
    """
    Test that we can handle case where prompt is < block.

    In this case, the P worker will still send remote_block_ids of the
    partial block. The D worker should schedule an async read
    in this case.
    """
    vllm_config = create_vllm_config()
    scheduler = create_scheduler(vllm_config)

    # Half of a block.
    BLOCK_SIZE = vllm_config.cache_config.block_size
    NUM_TOKENS = int(BLOCK_SIZE * 0.5)

    # Request will have 1 partial remote block.
    request = create_request(
        request_id=1,
        block_size=BLOCK_SIZE,
        num_tokens=NUM_TOKENS,
        do_remote_prefill=True,
        num_remote_blocks=1,
    )
    scheduler.add_request(request)
    scheduler_output = scheduler.schedule()

    # This request will read async.
    kv_connector_metadata = scheduler_output.kv_connector_metadata
    assert kv_connector_metadata is not None
    assert isinstance(kv_connector_metadata, NixlConnectorMetadata)
    assert len(kv_connector_metadata.reqs_to_recv) == 1
    assert len(scheduler_output.scheduled_new_reqs) == 0
```
**EN:** Test case covering `prompt less than block size`. It exercises `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt less than block size` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 4 个显式断言。

### test_abort_immediately_remote_prefill_enqueues_empty_recv (lines 324-349)
```python
def test_abort_immediately_remote_prefill_enqueues_empty_recv():
    """A remote-prefill request added with abort_immediately=True should
    be added to the scheduler's waiting queue then immediately aborted, so the
    NIXL connector's request_finished hook enqueues an empty recv to notify
    the prefill instance to free its blocks."""
    from vllm.v1.request import RequestStatus

    scheduler = create_scheduler(create_vllm_config())

    request = create_request(request_id=42, num_tokens=10, do_remote_prefill=True)
    assert request.kv_transfer_params is not None
    assert request.kv_transfer_params["do_remote_prefill"] is True

    # Mimic the EngineCore.add_request path for an abort-immediately req.
    scheduler.add_request(request)
    scheduler.finish_requests([request.request_id], RequestStatus.FINISHED_ABORTED)

    scheduler_output = scheduler.schedule()
    meta = scheduler_output.kv_connector_metadata
    assert isinstance(meta, NixlConnectorMetadata)
    assert set(meta.reqs_to_recv) == {request.request_id}
    req_meta = meta.reqs_to_recv[request.request_id]
    assert req_meta.local_block_ids == []
    assert req_meta.remote.request_id == f"prefill-{42}"
    # do_remote_prefill is consumed by request_finished to prevent re-issuing.
    assert request.kv_transfer_params["do_remote_prefill"] is False
```
**EN:** Test case covering `abort immediately remote prefill enqueues empty recv`. It exercises `create_scheduler, create_request, scheduler.add_request, scheduler.finish_requests, scheduler.schedule, isinstance`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort immediately remote prefill enqueues empty recv` 的测试用例。 该测试会调用 `create_scheduler, create_request, scheduler.add_request, scheduler.finish_requests, scheduler.schedule, isinstance`。 代码主体包含 7 个显式断言。

### test_kv_transfer_handshake (lines 356-470)
```python
def test_kv_transfer_handshake(dist_init):
    """Unit test for basic NixlConnector interface functionality."""
    from vllm.config import set_current_vllm_config

    # Test setup, we creates a scheduler that contains a NixlConnector
    # of role SCHEDULER, and expect it to be serving NixlAgentMetadata from
    # all workers of the instance.
    vllm_config = create_vllm_config()
    # in case the test runs on non-GPU machine
    vllm_config.kv_transfer_config.kv_buffer_device = "cpu"
    scheduler = create_scheduler(vllm_config)
    with set_current_vllm_config(vllm_config):
        # Create two NixlConnector of role WORKER, one is the worker of
        # the scheduler (prefill), the other is a worker of decode instance.
        # Prefill connector will register KV cache to populate proper handshake
        # metadata.
    # ... excerpt omitted for brevity ...
        expected_agent_metadata = decoder.decode(metadata.agent_metadata_bytes)
        assert delay
            assert received_metadata[0] == expected_agent_metadata
            assert received_metadata[1] == 0  # remote_tp_rank
            assert received_metadata[2] == 1  # remote_tp_size
        # Need to shutdown the background thread to release NIXL side channel port
        scheduler_connector.shutdown()
```
**EN:** Test case covering `KV transfer handshake`. Inputs/fixtures: `dist_init`. It exercises `patch, create_vllm_config, create_scheduler, set_current_vllm_config, KVCacheConfig, NixlConnector`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `kv transfer handshake` 的测试用例。 输入或 fixture：`dist_init`。 该测试会调用 `patch, create_vllm_config, create_scheduler, set_current_vllm_config, KVCacheConfig, NixlConnector`。 代码主体包含 4 个显式断言。

### FakeNixlConnectorWorker (lines 473-564)
```python
class FakeNixlConnectorWorker(NixlConnectorWorker):
    REMOTE_ENGINE_ID = "remote_engine"

    def __init__(
        self,
        *args,
        hand_shake_latency: float = 1.8,
        kv_cache_layout="HND",
        kv_cache_config=None,
        **kwargs,
    ):
        if kv_cache_config is None:
            kv_cache_config = make_kv_cache_config(block_size=16)
        super().__init__(*args, kv_cache_config=kv_cache_config, **kwargs)
        self._hand_shake_latency = hand_shake_latency
        self.kv_cache_layout = kv_cache_layout
        # Mock register_kv_caches attribute needed for tests that do not call it.
        self.src_xfer_handles_by_block_size = {self.block_size: 1}
    # ... excerpt omitted for brevity ...
        self, host: str, port: int, remote_tp_size: int, expected_engine_id: str
        assert expected_engine_id == self.REMOTE_ENGINE_ID
                ),
                remote_tp_rank=remote_tp_rank,
                remote_tp_size=remote_tp_size,
            )
            remote_agents[remote_tp_rank] = remote_agent_name
        return remote_agents
```
**EN:** Class `FakeNixlConnectorWorker` groups 0 test method(s) and 2 helper/fixture method(s). Bases: `NixlConnectorWorker`.
**CN:** 类 `FakeNixlConnectorWorker` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。 基类：`NixlConnectorWorker`。

### TestNixlHandshake (lines 567-1074)
```python
class TestNixlHandshake:
    @patch(
        "vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker.NixlWrapper",
        FakeNixlWrapper,
    )
    def test_multi_xfer_one_engine(
        self,
        default_vllm_config,
        # dist_init is a fixture that initializes the distributed environment.
        dist_init,
    ):
        """Test case where multiple xfers are initiated to the same engine.

        This test triggers the connector to load remote KV for the same
        `request_id`. The transfer is not done immediately due to
        `set_cycles_before_xfer_done`, so there is a state where there are
        multiple transfer states for the same `request_id`, and `get_finished`
        should handle it correctly (wait for all transfers to be done).
    # ... excerpt omitted for brevity ...
        assert isinstance(connector.connector_worker.nixl_wrapper, FakeNixlWrapper)
            assert _after_load - _before_load < 0.1, (
                assert request_id in done_recving
    @pytest.mark.parametrize(
    @pytest.mark.parametrize("local_tp_size", [1, 2])
                physical_blocks_per_logical_kv_block=1,
            )
            # We don't check layout for homogeneous TP and MLA for now, as the
            # whole block is moved.
            worker.add_remote_agent(meta, remote_tp_size=1)
```
**EN:** Class `TestNixlHandshake` groups 7 test method(s). Representative scenarios: `test_multi_xfer_one_engine, test_async_load_kv, test_prefill_tp_size_greater_than_decode_tp_size, test_prefill_tp_size_greater_than_decode_tp_size_mla, test_concurrent_load_kv, test_handshake_fails_on_kv_cache_layout_mismatch, ...`.
**CN:** 类 `TestNixlHandshake` 组织了 7 个测试方法。 代表性场景：`test_multi_xfer_one_engine, test_async_load_kv, test_prefill_tp_size_greater_than_decode_tp_size, test_prefill_tp_size_greater_than_decode_tp_size_mla, test_concurrent_load_kv, test_handshake_fails_on_kv_cache_layout_mismatch, ...`。

### test_kv_connector_stats (lines 1084-1149)
```python
def test_kv_connector_stats(default_vllm_config, dist_init):
    """Test that KV transfer stats are properly recorded and retrieved."""
    vllm_config = create_vllm_config()

    # Test worker role in decode server.
    connector = NixlConnector(
        vllm_config, KVConnectorRole.WORKER, make_kv_cache_config(block_size=16)
    )
    connector.connector_worker = FakeNixlConnectorWorker(
        vllm_config, connector.engine_id, hand_shake_latency=0
    # Verify that xfer_stats starts empty
    initial_stats = connector.get_kv_connector_stats()
    assert initial_stats is None
    # Create transfer metadata
    request_id = "test_req_for_stats"
    # ... excerpt omitted for brevity ...
        assert "Transfer did not complete within expected iterations"
    assert isinstance(stats_after_transfer, NixlKVConnectorStats)
    assert not stats_after_transfer.is_empty()
    assert stats_after_transfer.num_successful_transfers == 1
    # Verify stats are reset after retrieval
    stats_after_reset = connector.get_kv_connector_stats()
    assert stats_after_reset is None
```
**EN:** Test case covering `KV connector stats`. Inputs/fixtures: `default_vllm_config, dist_init`. It exercises `patch, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, connector.get_kv_connector_stats, NixlConnectorMetadata`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 连接器 stats` 的测试用例。 输入或 fixture：`default_vllm_config, dist_init`。 该测试会调用 `patch, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, connector.get_kv_connector_stats, NixlConnectorMetadata`。 代码主体包含 6 个显式断言。

### test_kv_connector_stats_aggregation (lines 1152-1216)
```python
def test_kv_connector_stats_aggregation():
    """
    Test KV transfer stats aggregation across TP ranks using
    KVOutputAggregator (used by MultiprocExecutor).

    # Create KVOutputAggregator for 3 workers (simulating TP=3), same thing
    # done in MultiprocExecutor.execute_model
    aggregator = KVOutputAggregator(expected_finished_count=3)
    # Create stats for multiple workers with different transfer patterns
    worker1_stats = NixlKVConnectorStats()
    worker2_stats = NixlKVConnectorStats()
    worker3_stats = NixlKVConnectorStats()
    # Record different transfers on each worker
    # Worker 1: 2 transfers
    stats = get_default_xfer_telemetry()
    # ... excerpt omitted for brevity ...
    assert isinstance(kv_connector_stats, NixlKVConnectorStats)
    assert kv_connector_stats.num_successful_transfers == 6
    # Logging proc, call reduce() to get CLI-friendly stats.
    cli_stats = kv_connector_stats.reduce()
    assert cli_stats["Avg xfer time (ms)"] == 1500.0
    assert cli_stats["Avg post time (ms)"] == 1500.0
    assert cli_stats["Avg number of descriptors"] == 1.5
```
**EN:** Test case covering `KV connector stats aggregation`. It exercises `KVOutputAggregator, NixlKVConnectorStats, get_default_xfer_telemetry, worker1_stats.record_transfer, worker2_stats.record_transfer, worker3_stats.record_transfer`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 连接器 stats aggregation` 的测试用例。 该测试会调用 `KVOutputAggregator, NixlKVConnectorStats, get_default_xfer_telemetry, worker1_stats.record_transfer, worker2_stats.record_transfer, worker3_stats.record_transfer`。 代码主体包含 5 个显式断言。

### test_multi_kv_connector_stats_aggregation (lines 1219-1291)
```python
def test_multi_kv_connector_stats_aggregation():
    """
    Test MultiKVConnectorStats aggregation across TP ranks using
    KVOutputAggregator (used by MultiprocExecutor).

    aggregator = KVOutputAggregator(expected_finished_count=3)
    from dataclasses import dataclass
    # Mock a KVConnectorStats class for testing aggregation over connectors.
    @dataclass
    class FooKVConnectorStats(KVConnectorStats):
        def reset(self):
            self.data = {"num_foo_transfers": 0}
        def record_transfer(self):
            if "num_foo_transfers" not in self.data:
    # ... excerpt omitted for brevity ...
            return self.data["num_foo_transfers"] == 0
            return self
        return MultiKVConnectorStats(data=data)
    assert isinstance(kv_connector_stats, MultiKVConnectorStats)
    # Validate per-connector totals across workers
    assert isinstance(kv_connector_stats["NixlConnector"], NixlKVConnectorStats)
    assert kv_connector_stats["NixlConnector"].num_successful_transfers == 5
    assert isinstance(kv_connector_stats["FooConnector"], FooKVConnectorStats)
    assert kv_connector_stats["FooConnector"].data["num_foo_transfers"] == 6
```
**EN:** Test case covering `multi KV connector stats aggregation`. It exercises `KVOutputAggregator, enumerate, aggregator.aggregate, isinstance, MultiKVConnectorStats, make_multi_stats`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `multi KV 连接器 stats aggregation` 的测试用例。 该测试会调用 `KVOutputAggregator, enumerate, aggregator.aggregate, isinstance, MultiKVConnectorStats, make_multi_stats`。 代码主体包含 5 个显式断言。

### test_scheduler_kv_connector_stats_aggregation (lines 1298-1358)
```python
def test_scheduler_kv_connector_stats_aggregation():
    """Test scheduler and worker KV connector stats aggregation."""
    from vllm.v1.core.sched.output import SchedulerOutput

    scheduler = create_scheduler(create_vllm_config())
    # Worker stats with transfer metrics
    worker_stats = NixlKVConnectorStats()
    worker_stats.record_transfer(get_default_xfer_telemetry())
    worker_stats.data["remote_tokens"] = []
    # Scheduler stats with custom metric (needs dummy transfer to avoid being skipped)
    scheduler_stats = NixlKVConnectorStats()
    scheduler_stats.data.update(
        {  # dummy transfer just for testing, to bypass is_empty() check
            "transfer_duration": [0],
            "post_duration": [0],
            "bytes_transferred": [0],
    # ... excerpt omitted for brevity ...
    final_stats = next(
        iter(engine_core_outputs.values())
    ).scheduler_stats.kv_connector_stats
    nixl_stats = final_stats["NixlConnector"]
    assert nixl_stats.num_successful_transfers == 2
    assert nixl_stats.data["remote_tokens"] == [128]
```
**EN:** Test case covering `scheduler KV connector stats aggregation`. It exercises `patch, create_scheduler, NixlKVConnectorStats, worker_stats.record_transfer, data.update, ModelRunnerOutput`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `调度器 KV 连接器 stats aggregation` 的测试用例。 该测试会调用 `patch, create_scheduler, NixlKVConnectorStats, worker_stats.record_transfer, data.update, ModelRunnerOutput`。 代码主体包含 2 个显式断言。

### test_abort_timeout_on_prefiller (lines 1366-1421)
```python
def test_abort_timeout_on_prefiller(monkeypatch, distributed_executor_backend):
    """
    Test lifecycle of an aborted Remote Prefill request hitting the timeout.
    -----> P
            |  {process request}
     <-/--- |  {result is NOT delivered, eg proxy is down}
            |
            |  {eventually free blocks}
    model_name = "Qwen/Qwen3-0.6B"
    timeout = 6
    kv_transfer_config = KVTransferConfig(
        kv_connector="NixlConnector",
        kv_role="kv_both",
        kv_connector_extra_config={"kv_lease_duration": timeout},
    )
    llm_kwargs = {
    # ... excerpt omitted for brevity ...
            try:
                run_test_and_cleanup()
            finally:
                ray.shutdown()
    else:
        run_test_and_cleanup()
```
**EN:** Parameterized test covering `abort timeout on prefiller`. Parameter axes: `distributed_executor_backend`. Inputs/fixtures: `monkeypatch, distributed_executor_backend`. It exercises `mark.parametrize, patch, KVTransferConfig, monkeypatch.setenv, LLM, run_test_and_cleanup`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `abort timeout on prefiller` 的测试用例。 参数维度：`distributed_executor_backend`。 输入或 fixture：`monkeypatch, distributed_executor_backend`。 该测试会调用 `mark.parametrize, patch, KVTransferConfig, monkeypatch.setenv, LLM, run_test_and_cleanup`。 主要通过 mock、回调或输出检查来完成验证。

### RequestIdMapper (lines 1424-1437)
```python
class RequestIdMapper:
    """Helper class to map external request IDs to internal request IDs."""

    def __init__(self, output_processor: OutputProcessor):
        self.req_id_mapping: dict[str, str] = {}
        self.original_add_request = output_processor.add_request
        output_processor.add_request = self._add_request

    def _add_request(self, request: EngineCoreRequest, *args, **kwargs):
        self.req_id_mapping[request.external_req_id] = request.request_id
        return self.original_add_request(request, *args, **kwargs)

    def __call__(self, external_req_id: str) -> str:
        return self.req_id_mapping[external_req_id]
```
**EN:** Class `RequestIdMapper` groups 0 test method(s) and 3 helper/fixture method(s).
**CN:** 类 `RequestIdMapper` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。

### _run_abort_timeout_test (lines 1440-1490)
```python
def _run_abort_timeout_test(llm: LLM, timeout: int):
    """Helper function to run the abort timeout test logic."""
    remote_prefill_opts = {
        "do_remote_decode": True,
        "do_remote_prefill": False,
        "remote_engine_id": None,
        "remote_block_ids": None,
        "remote_host": None,
        "remote_port": None,
    }
    # Simulate sidecar request
    sampling_params = SamplingParams(
        temperature=0.0,
        max_tokens=1,
        extra_args={"kv_transfer_params": remote_prefill_opts},
    )
    scheduler = llm.llm_engine.engine_core.engine_core.scheduler
    req_to_blocks = scheduler.kv_cache_manager.coordinator.single_type_managers[
    # ... excerpt omitted for brevity ...
        assert len(outputs) == 1
        return id_mapper(outputs[0].request_id)
    assert req0_id in scheduler.finished_req_ids and req0_id in req_to_blocks
    assert req0_id in req_to_blocks
    assert req1_id in scheduler.finished_req_ids and req1_id in req_to_blocks
    time.sleep(timeout)
    _ = llm.generate([f"What is the capital of France? {padding}"], sampling_params)
    # Request-0 times out and is cleared!
    assert req0_id not in req_to_blocks
    # Need to shutdown the background thread to release NIXL side channel port
    llm.llm_engine.engine_core.shutdown()
```
**EN:** Helper function `_run_abort_timeout_test` encapsulates reusable logic for `run abort timeout test`. Inputs: `llm, timeout`. Key calls include `SamplingParams, RequestIdMapper, req_id, time.sleep, llm.generate, engine_core.shutdown`. It includes 5 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_run_abort_timeout_test` 封装了与 `run abort timeout test` 相关的可复用逻辑。 输入参数：`llm, timeout`。 关键调用包括 `SamplingParams, RequestIdMapper, req_id, time.sleep, llm.generate, engine_core.shutdown`。 其中包含 5 个内部断言，用于保护前置假设。

### test_register_kv_caches (lines 1514-1754)
```python
def test_register_kv_caches(
    default_vllm_config, dist_init, attn_backend, enable_cross_layers
):
    """
    Test that register_kv_caches() properly calls nixl_wrapper methods with
    correct data.

    This test verifies:
    1. nixl_wrapper.get_reg_descs() is called with caches_data containing
       tensor metadata
    2. nixl_wrapper.get_xfer_descs() is called with blocks_data containing
       block layout info
    vllm_config = create_vllm_config(attention_backend=attn_backend)
    # Enable cross layers blocks
    vllm_config.kv_transfer_config.kv_connector_extra_config[
    # ... excerpt omitted for brevity ...
        expected_tensor_size: int
        expected_base_addrs: list[int]
        expected_num_entries: int
            assert connector.prefer_cross_layer_blocks == (
            assert not connector.prefer_cross_layer_blocks
            expected_tensor_size = (
            assert block_len == expected_block_len, (
                f"Block entry {i}: Expected block len {expected_block_len}, "
                f"got {block_len}"
            )
        assert connector.connector_worker.block_size == 16
```
**EN:** Parameterized test covering `register KV caches`. Parameter axes: `enable_cross_layers, attn_backend`. Inputs/fixtures: `default_vllm_config, dist_init, attn_backend, enable_cross_layers`. It exercises `mark.parametrize, create_vllm_config, set_kv_cache_layout, patch, NixlConnector, FullAttentionSpec`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `register kv caches` 的测试用例。 参数维度：`enable_cross_layers, attn_backend`。 输入或 fixture：`default_vllm_config, dist_init, attn_backend, enable_cross_layers`。 该测试会调用 `mark.parametrize, create_vllm_config, set_kv_cache_layout, patch, NixlConnector, FullAttentionSpec`。 代码主体包含 10 个显式断言。

### FakePlatform (lines 1757-1773)
```python
class FakePlatform(Platform):
    device_type: str = "oot"

    @classmethod
    def get_nixl_supported_devices(cls) -> dict[str, tuple[str, ...]]:
        """
        Returns a mapping from device_type to a tuple of supported
        kv_buffer_device for nixl.
        """
        return {"oot": ("oot",)}

    @classmethod
    def get_nixl_memory_type(cls) -> str | None:
        """
        Returns the nixl memory type for the current platform.
        """
        return "VRAM"
```
**EN:** Class `FakePlatform` groups 0 test method(s) and 2 helper/fixture method(s). Bases: `Platform`.
**CN:** 类 `FakePlatform` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。 基类：`Platform`。

### test_kv_buffer_to_nixl_memory_types (lines 1782-1822)
```python
def test_kv_buffer_to_nixl_memory_types(
    default_vllm_config, dist_init, kv_buffer_device, nixl_memory_type
):
    """
    Test that register_kv_caches() passes the correct memory types from the
    config to the nixl_wrapper.
    vllm_config = create_vllm_config()
    # Override the default memory types in the config
    vllm_config.kv_transfer_config.kv_buffer_device = kv_buffer_device
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.utils import (
        _NIXL_SUPPORTED_DEVICE,
    )

    _NIXL_SUPPORTED_DEVICE.update(FakePlatform.get_nixl_supported_devices())
    with (
        patch("vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker.NixlWrapper"),
    # ... excerpt omitted for brevity ...
            vllm_config, KVConnectorRole.WORKER, make_kv_cache_config(block_size=16)
        )
        # Verify get_reg_descs was called with the correct memory_type
        assert connector.connector_worker.kv_buffer_device == kv_buffer_device
        assert connector.connector_worker.nixl_memory_type == nixl_memory_type
```
**EN:** Parameterized test covering `KV buffer toNIXL memory types`. Parameter axes: `kv_buffer_device, nixl_memory_type`. Inputs/fixtures: `default_vllm_config, dist_init, kv_buffer_device, nixl_memory_type`. It exercises `mark.parametrize, create_vllm_config, _NIXL_SUPPORTED_DEVICE.update, FakePlatform.get_nixl_supported_devices, patch, NixlConnector`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `kv buffer tonixl memory types` 的测试用例。 参数维度：`kv_buffer_device, nixl_memory_type`。 输入或 fixture：`default_vllm_config, dist_init, kv_buffer_device, nixl_memory_type`。 该测试会调用 `mark.parametrize, create_vllm_config, _NIXL_SUPPORTED_DEVICE.update, FakePlatform.get_nixl_supported_devices, patch, NixlConnector`。 代码主体包含 2 个显式断言。

### test_shutdown_cleans_up_resources (lines 1829-1887)
```python
def test_shutdown_cleans_up_resources(default_vllm_config, dist_init):
    """Test that shutdown() properly cleans up all resources."""
    vllm_config = create_vllm_config()

    scheduler = NixlConnectorScheduler(
        vllm_config,
        vllm_config.kv_transfer_config.engine_id,
        make_kv_cache_config(block_size=16),
    )
    worker = NixlConnectorWorker(
    nixl_wrapper = worker.nixl_wrapper
    with (
        patch.object(worker, "_handshake_initiation_executor") as mock_exec,
    # ... excerpt omitted for brevity ...
        assert mock_rel_dlist.call_count == 4
        mock_rel_dlist.assert_any_call(457)  # src handle (2nd chunk)
        mock_rel_dlist.assert_any_call(789)  # dst handle
        mock_rem_agent.assert_called_once_with("agent1")
        assert mock_dereg.call_count == 2
        mock_dereg.assert_any_call("desc1")
        mock_dereg.assert_any_call("desc2")
```
**EN:** Test case covering `shutdown cleans up resources`. Inputs/fixtures: `default_vllm_config, dist_init`. It exercises `patch, create_vllm_config, NixlConnectorScheduler, NixlConnectorWorker, make_kv_cache_config, patch.object`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `shutdown cleans up resources` 的测试用例。 输入或 fixture：`default_vllm_config, dist_init`。 该测试会调用 `patch, create_vllm_config, NixlConnectorScheduler, NixlConnectorWorker, make_kv_cache_config, patch.object`。 代码主体包含 2 个显式断言。

### test_aborted_request_removed_from_worker_in_batch (lines 1894-1954)
```python
def test_aborted_request_removed_from_worker_in_batch(default_vllm_config, dist_init):
    """
    Create and schedule a request so that P adds it to in-batch tracking via
    the real scheduler, then simulate an abort (request not in next scheduler
    iteration) and verify the worker no longer tracks it as in-batch.
    vllm_config = create_vllm_config()

    scheduler = create_scheduler(vllm_config)
    # KVConnector Worker in P
    connector = NixlConnector(
        vllm_config, KVConnectorRole.WORKER, make_kv_cache_config(block_size=16)
    )
    connector.connector_worker = FakeNixlConnectorWorker(
        vllm_config, connector.engine_id, hand_shake_latency=0
    # Create a request that triggers do_remote_decode so that
    # ... excerpt omitted for brevity ...
    assert kv_meta is not None
    assert isinstance(kv_meta, NixlConnectorMetadata)
    assert req.request_id in kv_meta.reqs_in_batch
    assert req.request_id in connector.connector_worker._reqs_to_process
    assert kv_meta2 is not None
    assert isinstance(kv_meta2, NixlConnectorMetadata)
    #### Model Runner start ####
    connector.bind_connector_metadata(kv_meta2)
    connector.start_load_kv(dummy_ctx)
    # After abort, the worker should not keep tracking it as "in-batch"
    assert req.request_id not in connector.connector_worker._reqs_to_process
```
**EN:** Test case covering `aborted request removed from worker in batch`. Inputs/fixtures: `default_vllm_config, dist_init`. It exercises `patch, create_vllm_config, create_scheduler, NixlConnector, FakeNixlConnectorWorker, create_request`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `aborted request removed from worker in 批处理` 的测试用例。 输入或 fixture：`default_vllm_config, dist_init`。 该测试会调用 `patch, create_vllm_config, create_scheduler, NixlConnector, FakeNixlConnectorWorker, create_request`。 代码主体包含 8 个显式断言。

### FailingNixlWrapper (lines 1958-2007)
```python
class FailingNixlWrapper(FakeNixlWrapper):
    """Mock NixlWrapper that fails on specific operations."""

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fail_handshake = False
        self.fail_transfer_setup = False
        self.fail_send_notif = False
        self.fail_transfer_state = False  # Returns "ERR" state
        self.fail_transfer_exception = False  # Raises exception in check_xfer_state
    def add_remote_agent(self, agent_metadata: bytes) -> str:
        if self.fail_handshake:
            from zmq.error import Again
            raise Again("Simulated timeout failure")
        return super().add_remote_agent(agent_metadata)
    # ... excerpt omitted for brevity ...
        return super().make_prepped_xfer(
        return super().send_notif(agent_name, notif_msg)
    def check_xfer_state(self, handle: int) -> str:
        if self.fail_transfer_exception:
            raise RuntimeError("Simulated check_xfer_state exception")
        if self.fail_transfer_state:
            return "ERR"  # Bad transfer state
        return super().check_xfer_state(handle)
```
**EN:** Class `FailingNixlWrapper` groups 0 test method(s) and 5 helper/fixture method(s). Bases: `FakeNixlWrapper`.
**CN:** 类 `FailingNixlWrapper` 组织了 0 个测试方法，以及 5 个辅助或 fixture 方法。 基类：`FakeNixlWrapper`。

### test_transfer_failure_logging (lines 2025-2162)
```python
def test_transfer_failure_logging(
    default_vllm_config,
    dist_init,
    failure_type,
    wrapper_config,
    needs_get_finished,
    enable_hma,
):
    """Test that transfer failures are logged with structured context.

    Run with `pytest -sv` to see the log output.
    Covers failure types:
    - transfer_setup_failed: make_prepped_xfer fails
    - handshake_failed: add_remote_agent fails during request handshake
    - notification_failed: send_notif fails
    - transfer_failed: check_xfer_state returns bad state (e.g., "ERR")
    - transfer_exception: check_xfer_state raises exception
    # ... excerpt omitted for brevity ...
    assert len(error_logs) >= 1, f"Expected at least one error log for {failure_type}"
    # Check that at least one log matches the expected format
    assert any("NIXL transfer failure" in msg for msg in all_messages), (
    assert any("failure_type" in msg for msg in all_messages), (
    assert any("Context:" in msg for msg in all_messages), (
    )
    # Check that the expected failure_type appears in at least one log
    # Note: handshake_failed also triggers handshake_setup_failed
    assert failure_type in combined_logs or (
        failure_type == "handshake_failed" and "handshake_setup_failed" in combined_logs
    ), f"Expected '{failure_type}' in logs. Got: {all_messages}"
```
**EN:** Parameterized test covering `transfer failure logging`. Parameter axes: `failure_type, wrapper_config, needs_get_finished, enable_hma`. Inputs/fixtures: `default_vllm_config, dist_init, failure_type, wrapper_config, needs_get_finished, enable_hma`. It exercises `patch, mark.parametrize, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, wrapper_config.items`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `transfer failure logging` 的测试用例。 参数维度：`failure_type, wrapper_config, needs_get_finished, enable_hma`。 输入或 fixture：`default_vllm_config, dist_init, failure_type, wrapper_config, needs_get_finished, enable_hma`。 该测试会调用 `patch, mark.parametrize, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, wrapper_config.items`。 代码主体包含 5 个显式断言。

### test_handshake_failure_returns_finished (lines 2169-2213)
```python
def test_handshake_failure_returns_finished(default_vllm_config, dist_init):
    """Test that handshake failures mark blocks invalid and return via get_finished."""
    vllm_config = create_vllm_config()

    connector = NixlConnector(
        vllm_config, KVConnectorRole.WORKER, make_kv_cache_config(block_size=16)
    )
    connector.connector_worker = FakeNixlConnectorWorker(
        vllm_config, connector.engine_id, hand_shake_latency=0.1
    connector.connector_worker.nixl_wrapper.fail_handshake = True
    request_id = "test_handshake_fail"
    metadata = NixlConnectorMetadata()
    metadata.add_new_req_to_recv(
        request_id=request_id,
        local_block_ids=([1, 2, 3],),
        kv_transfer_params={
    # ... excerpt omitted for brevity ...
    invalid_blocks = connector.get_block_ids_with_load_errors()
    assert invalid_blocks == {1, 2, 3}
    # Check that request appears in get_finished
    _, done_recving = connector.get_finished(finished_req_ids=set())
    assert request_id in done_recving
```
**EN:** Test case covering `handshake failure returns finished`. Inputs/fixtures: `default_vllm_config, dist_init`. It exercises `patch, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, NixlConnectorMetadata, metadata.add_new_req_to_recv`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `handshake failure returns finished` 的测试用例。 输入或 fixture：`default_vllm_config, dist_init`。 该测试会调用 `patch, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, NixlConnectorMetadata, metadata.add_new_req_to_recv`。 代码主体包含 2 个显式断言。

### test_transfer_setup_failure_returns_finished (lines 2220-2267)
```python
def test_transfer_setup_failure_returns_finished(default_vllm_config, dist_init):
    """Test that transfer setup failures mark blocks invalid
    and return via get_finished."""
    vllm_config = create_vllm_config()

    connector = NixlConnector(
        vllm_config, KVConnectorRole.WORKER, make_kv_cache_config(block_size=16)
    )
    connector.connector_worker = FakeNixlConnectorWorker(
        vllm_config, connector.engine_id, hand_shake_latency=0
    connector.connector_worker.nixl_wrapper.fail_transfer_setup = True
    request_id = "test_transfer_fail"
    metadata = NixlConnectorMetadata()
    metadata.add_new_req_to_recv(
        request_id=request_id,
        local_block_ids=([7, 8, 9],),
    # ... excerpt omitted for brevity ...
    invalid_blocks = connector.get_block_ids_with_load_errors()
    assert invalid_blocks == {7, 8, 9}
    # ensure request appears in get_finished
    _, done_recving = connector.get_finished(finished_req_ids=set())
    assert request_id in done_recving
```
**EN:** Test case covering `transfer setup failure returns finished`. Inputs/fixtures: `default_vllm_config, dist_init`. It exercises `patch, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, NixlConnectorMetadata, metadata.add_new_req_to_recv`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `transfer setup failure returns finished` 的测试用例。 输入或 fixture：`default_vllm_config, dist_init`。 该测试会调用 `patch, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, NixlConnectorMetadata, metadata.add_new_req_to_recv`。 代码主体包含 2 个显式断言。

### test_failed_request_skips_kv_postprocessing (lines 2283-2388)
```python
def test_failed_request_skips_kv_postprocessing(
    default_vllm_config, dist_init, failure_mode
):
    """Test that failed requests skip KV sync and post-processing in
    get_finished().

    This is the core safety behavior: when a KV transfer fails at any stage,
    the request must still appear in done_recving (so the scheduler can apply
    kv_load_failure_policy), but sync_recved_kv_to_device and post-processing
    must NOT be called since no valid KV data was received.
    Covers all failure paths that involve an actual (attempted) KV transfer:
    - handshake: add_remote_agent raises during async handshake
    - transfer_setup: make_prepped_xfer raises before handle is in transfers
    - transfer_failed: check_xfer_state returns bad state ("ERR") in
      _pop_done_transfers — this is the path that previously had the bug
      where post-processing was NOT skipped
    - transfer_exception: check_xfer_state raises in _pop_done_transfers
    # ... excerpt omitted for brevity ...
    assert request_id in done_recving
    # Metadata for the request should have been cleaned up.
    assert request_id not in worker._recving_metadata
    # Blocks should have been marked as invalid.
    invalid_blocks = connector.get_block_ids_with_load_errors()
    assert invalid_blocks == {1, 2, 3}
```
**EN:** Parameterized test covering `failed request skips KV postprocessing`. Parameter axes: `failure_mode`. Inputs/fixtures: `default_vllm_config, dist_init, failure_mode`. It exercises `patch, mark.parametrize, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, setattr`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `failed request skips kv postprocessing` 的测试用例。 参数维度：`failure_mode`。 输入或 fixture：`default_vllm_config, dist_init, failure_mode`。 该测试会调用 `patch, mark.parametrize, create_vllm_config, NixlConnector, FakeNixlConnectorWorker, setattr`。 代码主体包含 3 个显式断言。

### test_compatibility_hash_validation (lines 2417-2544)
```python
def test_compatibility_hash_validation(
    default_vllm_config,
    dist_init,
    mismatch_type,
    config_overrides,
    version_override,
    should_fail,
    enforce_handshake_compat,
):
    """
    Test NIXL compatibility hash validation during handshake.

    Parameters:
        mismatch_type: description of what is being tested
        config_overrides: dict of config to override for the remote instance
        version_override: version dict e.g. {"vllm_version": "0.6.1"}
        should_fail: whether the handshake should fail
        enforce_handshake_compat: whether to enforce compatibility checking
    # ... excerpt omitted for brevity ...
    mock_socket = MagicMock()
            with pytest.raises(RuntimeError, match="compatibility hash mismatch"):
                    expected_engine_id=FakeNixlConnectorWorker.REMOTE_ENGINE_ID,
                remote_tp_size=1,
                expected_engine_id=FakeNixlConnectorWorker.REMOTE_ENGINE_ID,
            )
            # Verify handshake returned agent mapping
            assert isinstance(result, dict)
            assert len(result) == 1
```
**EN:** Parameterized test covering `compatibility hash validation`. Parameter axes: `mismatch_type, config_overrides, version_override, should_fail, enforce_handshake_compat`. Inputs/fixtures: `default_vllm_config, dist_init, mismatch_type, config_overrides, version_override, should_fail, enforce_handshake_compat`. It exercises `mark.parametrize, patch, create_vllm_config, make_kv_cache_config, NixlConnector, cast`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `compatibility hash validation` 的测试用例。 参数维度：`mismatch_type, config_overrides, version_override, should_fail, enforce_handshake_compat`。 输入或 fixture：`default_vllm_config, dist_init, mismatch_type, config_overrides, version_override, should_fail, enforce_handshake_compat`。 该测试会调用 `mark.parametrize, patch, create_vllm_config, make_kv_cache_config, NixlConnector, cast`。 代码主体包含 2 个显式断言。

### test_handshake_decode_errors (lines 2560-2752)
```python
def test_handshake_decode_errors(default_vllm_config, dist_init, error_scenario):
    """
    Test that msgspec decode errors are properly handled during handshake.

    Tests both DecodeError and ValidationError for both decoders:
    - NixlHandshakePayload decoder
    - NixlAgentMetadata decoder
    local_vllm_config = create_vllm_config(
        model="facebook/opt-125m",
        block_size=16,
    )
    decode_connector = NixlConnector(
        local_vllm_config, KVConnectorRole.WORKER, make_kv_cache_config(block_size=16)
    decode_worker = decode_connector.connector_worker
    backend = get_current_attn_backend(local_vllm_config)
    # ... excerpt omitted for brevity ...
    mock_socket = MagicMock()
        with pytest.raises(RuntimeError):
                expected_engine_id=FakeNixlConnectorWorker.REMOTE_ENGINE_ID,
        assert worker.transfer_topo.tp_ratio(prefill_tp_size) == -prefill_tp_size
        assert decode_req_id != prefill_req_id
        worker._read_blocks = MagicMock()  # type: ignore[method-assign]
        for agent, notif in send_notif_calls:
            assert notif == expected_notif, (
                f"Broadcast notif to {agent!r} must use prefill_req_id; "
                f"got {notif!r} (expected {expected_notif!r}, "
                f"buggy form would be {bad_notif!r})"
            )
```
**EN:** Parameterized test covering `handshake decode errors`. Parameter axes: `error_scenario`. Inputs/fixtures: `default_vllm_config, dist_init, error_scenario`. It exercises `mark.parametrize, patch, create_vllm_config, NixlConnector, get_current_attn_backend, backend.get_kv_cache_shape`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `handshake decode errors` 的测试用例。 参数维度：`error_scenario`。 输入或 fixture：`default_vllm_config, dist_init, error_scenario`。 该测试会调用 `mark.parametrize, patch, create_vllm_config, NixlConnector, get_current_attn_backend, backend.get_kv_cache_shape`。 代码主体包含 6 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理

## Dependencies / 依赖关系
- **EN:** External libraries: `msgspec, pytest, ray, torch, zmq.error`.
- **CN:** 外部库：`msgspec, pytest, ray, torch, zmq.error`。
- **EN:** vLLM modules under test: `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.utils, vllm.distributed.kv_transfer.kv_connector.v1, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.metrics, vllm.distributed.kv_transfer.kv_connector.v1.multi_connector, vllm.distributed.kv_transfer.kv_connector.v1.nixl, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.utils, vllm.distributed.kv_transfer.kv_connector.v1, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.metrics, vllm.distributed.kv_transfer.kv_connector.v1.multi_connector, vllm.distributed.kv_transfer.kv_connector.v1.nixl, ...`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `contextlib, inspect, os, tempfile, textwrap, time, uuid, collections, ...`.
- **CN:** 标准库支持：`contextlib, inspect, os, tempfile, textwrap, time, uuid, collections, ...`。
