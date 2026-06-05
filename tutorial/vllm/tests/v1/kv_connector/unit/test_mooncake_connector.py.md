# test_mooncake_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_mooncake_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `mooncake connector` behavior and regressions in the v1 stack. / 验证 v1 栈中 `mooncake connector` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-32)
```python
import asyncio
import contextlib
import time
from unittest.mock import AsyncMock, MagicMock, patch

import pytest
import torch
import zmq.asyncio

from vllm.config import set_current_vllm_config
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector import (
    KVConnectorRole,
    MooncakeConnector,
    MooncakeConnectorMetadata,
    MooncakeXferMetadata,
    MooncakeXferResponse,
    MooncakeXferResponseStatus,
    PullReqMeta,
    SendBlockMeta,
)
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils import (
    MooncakeBootstrapServer,
)
from vllm.utils.network_utils import get_open_port
from vllm.v1.attention.backends.flash_attn import FlashAttentionBackend
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.request import RequestStatus

from .utils import create_request, create_scheduler, create_vllm_config
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, zmq.asyncio, httpx`. vLLM modules under test include `vllm.config, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils, vllm.utils.network_utils, vllm.v1.attention.backends.flash_attn, ...`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, zmq.asyncio, httpx`。 被测试的 vLLM 模块包括 `vllm.config, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils, vllm.utils.network_utils, vllm.v1.attention.backends.flash_attn, ...`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### _make_test_kv_cache_config (lines 35-36)
```python
def _make_test_kv_cache_config() -> KVCacheConfig:
    return KVCacheConfig(num_blocks=0, kv_cache_tensors=[], kv_cache_groups=[])
```
**EN:** Helper function `_make_test_kv_cache_config` encapsulates reusable logic for `test KV cache config`. Key calls include `KVCacheConfig`.
**CN:** 辅助函数 `_make_test_kv_cache_config` 封装了与 `test KV 缓存 config` 相关的可复用逻辑。 关键调用包括 `KVCacheConfig`。

### FakeMooncakeWrapper (lines 39-57)
```python
class FakeMooncakeWrapper:
    """Mock Mooncake TransferEngine for unit testing environments."""

    def __init__(self, *args, **kwargs):
        pass

    def initialize(self, local_hostname, metadata_server, protocol, device_name) -> int:
        return 0

    def get_rpc_port(self) -> int:
        return 12345

    def batch_transfer_sync_write(
        self, target_hostname, buffers, peer_buffer_addresses, lengths
    ) -> int:
        return 0

    def batch_register_memory(self, buffer_addresses, capacities) -> int:
        return 0
```
**EN:** Class `FakeMooncakeWrapper` groups 0 test method(s) and 5 helper/fixture method(s).
**CN:** 类 `FakeMooncakeWrapper` 组织了 0 个测试方法，以及 5 个辅助或 fixture 方法。

### test_basic_interface (lines 60-107)
```python
def test_basic_interface():
    """Unit test for basic MooncakeConnector interface functionality."""

    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_consumer"
    )
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
    # ... excerpt omitted for brevity ...
    assert kv_connector_metadata is not None
    assert isinstance(kv_connector_metadata, MooncakeConnectorMetadata)
    assert len(kv_connector_metadata.reqs_to_recv) == 1
    assert request_id in kv_connector_metadata.reqs_to_recv["my-engine-id"]
        all_block_ids,
        scheduler.kv_cache_manager.coordinator.single_type_managers[0].req_to_blocks[
            request_id
        ],
    ):
        assert block_id == block.block_id
```
**EN:** Test case covering `basic interface`. It exercises `create_vllm_config, create_scheduler, int, create_request, kv_transfer_params.update, scheduler.add_request`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `basic interface` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, kv_transfer_params.update, scheduler.add_request`。 代码主体包含 5 个显式断言。

### test_prompt_less_than_block_size (lines 110-145)
```python
def test_prompt_less_than_block_size():
    """Test that we can handle case where prompt is < block."""

    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_consumer"
    )
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
    # ... excerpt omitted for brevity ...
    # This request will read async.
    kv_connector_metadata = scheduler_output.kv_connector_metadata
    assert kv_connector_metadata is not None
    assert isinstance(kv_connector_metadata, MooncakeConnectorMetadata)
    assert len(kv_connector_metadata.reqs_to_recv["my-engine-id"]) == 1
    assert len(scheduler_output.scheduled_new_reqs) == 0
```
**EN:** Test case covering `prompt less than block size`. It exercises `create_vllm_config, create_scheduler, int, create_request, kv_transfer_params.update, scheduler.add_request`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt less than block size` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, create_request, kv_transfer_params.update, scheduler.add_request`。 代码主体包含 4 个显式断言。

### bootstrap_server (lines 149-156)
```python
def bootstrap_server():
    """Fixture to launch and cleanup a Mooncake Bootstrap HTTP Server."""

    port = get_open_port()
    server = MooncakeBootstrapServer("127.0.0.1", port)
    server.start()
    yield server
    server.shutdown()
```
**EN:** Fixture/helper `bootstrap_server` prepares reusable state for downstream tests. Key calls include `get_open_port, MooncakeBootstrapServer, server.start, server.shutdown`.
**CN:** `bootstrap_server` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `get_open_port, MooncakeBootstrapServer, server.start, server.shutdown`。

### test_bootstrap_server (lines 160-216)
```python
async def test_bootstrap_server(bootstrap_server: MooncakeBootstrapServer):
    """
    Tests the bootstrap server's api for worker registration and querying.

    Validates DP/TP/PP rank indexing and error handling for duplicate registrations.
    import httpx
    base_url = f"http://127.0.0.1:{bootstrap_server.port}"
    # Query when empty
    async with httpx.AsyncClient() as client:
        response = await client.get(f"{base_url}/query")
        assert response.status_code == 200
        assert response.json() == {}
    # Register a worker
    # ... excerpt omitted for brevity ...
        assert response.json() == {"status": "ok"}
        assert "0" in data
        assert data["0"]["engine_id"] == "eng-1"
        assert data["0"]["worker_addr"]["0"]["0"] == "tcp://1.1.1.1:1111"
        "addr": "tcp://3.3.3.3:3333",
    }
        response = await client.post(f"{base_url}/register", json=payload3_fail)
        assert response.status_code == 400
        assert "Engine ID mismatch" in response.text
```
**EN:** Async test covering `bootstrap server`. Inputs/fixtures: `bootstrap_server`. It exercises `httpx.AsyncClient, response.json, client.get, client.post`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `bootstrap server` 的测试用例。 输入或 fixture：`bootstrap_server`。 该测试会调用 `httpx.AsyncClient, response.json, client.get, client.post`。 代码主体包含 12 个显式断言。

### test_scheduler_request_finished (lines 219-249)
```python
def test_scheduler_request_finished():
    """
    Tests the scheduler-side logic when a request finishes.

    Differentiates between 'Finished' (requires transfer)
    and 'Aborted' (immediate free).
    """

    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_producer"
    )
    scheduler = create_scheduler(vllm_config)
    scheduler_connector = scheduler.get_kv_connector().connector_scheduler

    request = create_request(request_id=1, do_remote_decode=True)
    request.kv_transfer_params["transfer_id"] = request.request_id

    # Case: Capped length (Successful prefill, need to send to decoder)
    request.status = RequestStatus.FINISHED_LENGTH_CAPPED
    delay_free, _ = scheduler_connector.request_finished(request, block_ids=([10, 11],))
    assert delay_free is True
    assert "id-1" in scheduler_connector._reqs_need_send
    assert scheduler_connector._reqs_need_send["id-1"][1] == [[10, 11]]

    # Case: Aborted (No need to transfer, free blocks immediately)
    scheduler_connector._reqs_need_send.clear()
    request.status = RequestStatus.FINISHED_ABORTED
    delay_free, _ = scheduler_connector.request_finished(request, block_ids=([12],))
    assert delay_free is False
    assert len(scheduler_connector._reqs_need_send) == 0
    assert "id-1" in scheduler_connector._reqs_not_processed
```
**EN:** Test case covering `scheduler request finished`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler_connector.request_finished, _reqs_need_send.clear, scheduler.get_kv_connector`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `调度器 request finished` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler_connector.request_finished, _reqs_need_send.clear, scheduler.get_kv_connector`。 代码主体包含 6 个显式断言。

### patch_worker_dependencies (lines 253-307)
```python
def patch_worker_dependencies():
    """Helper to mock all distributed and network dependencies for Worker tests."""

    with (
        patch(
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector.TransferEngine",
            FakeMooncakeWrapper,
        ),
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector.get_ip",
            return_value="127.0.0.1",
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector.get_tensor_model_parallel_rank",
            return_value=0,
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector.get_tensor_model_parallel_world_size",
    # ... excerpt omitted for brevity ...
        mock_pp_group = MagicMock()
        mock_socket_object = AsyncMock()
        mock_socket_object.setsockopt = MagicMock()
        mock_socket_ctx = MagicMock()
        mock_http_client_instance = AsyncMock()
        yield {
            "mock_make_zmq": mock_make_zmq,
            "mock_socket_object": mock_socket_object,
            "mock_async_client": mock_async_client,
            "mock_http_client": mock_http_client_instance,
        }
```
**EN:** Helper function `patch_worker_dependencies` encapsulates reusable logic for `patch worker dependencies`. Key calls include `patch, MagicMock, AsyncMock`.
**CN:** 辅助函数 `patch_worker_dependencies` 封装了与 `patch worker dependencies` 相关的可复用逻辑。 关键调用包括 `patch, MagicMock, AsyncMock`。

### test_kv_producer (lines 315-469)
```python
async def test_kv_producer(monkeypatch):
    """
    Simulates a Producer Worker (Prefiller) receiving a transfer request
    from a Consumer (Decoder).

    Verifies memory offset calculation: ptr = base_addr + block_id * block_len.
    monkeypatch.setenv("VLLM_MOONCAKE_ABORT_REQUEST_TIMEOUT", "5")
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_producer"
    )
    with set_current_vllm_config(vllm_config), patch_worker_dependencies():
        prefill_connector = MooncakeConnector(
            vllm_config,
            KVConnectorRole.WORKER,
            _make_test_kv_cache_config(),
    # ... excerpt omitted for brevity ...
        mock_socket = AsyncMock(spec=zmq.asyncio.Socket)
        mock_socket.send_multipart = AsyncMock()
            assert sent_identity == identity
            assert response.status == MooncakeXferResponseStatus.FINISH
            assert response.ok_reqs == ["d-req-1"]
            assert transfer_id not in prefill_worker.reqs_need_send
            assert response.err_msg == "Mooncake transfer engine returned 123"
            assert response.err_reqs == ["d-req-1"]
        # Clean up
        prefill_worker.sender_loop = origin_sender_loop
        prefill_worker.shutdown()
```
**EN:** Async test covering `KV producer`. Inputs/fixtures: `monkeypatch`. It exercises `patch, monkeypatch.setenv, create_vllm_config, set_current_vllm_config, patch_worker_dependencies, MooncakeConnector`. The body contains 11 explicit assertion(s).
**CN:** 该代码块是覆盖 `kv producer` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `patch, monkeypatch.setenv, create_vllm_config, set_current_vllm_config, patch_worker_dependencies, MooncakeConnector`。 代码主体包含 11 个显式断言。

### test_kv_consumuer (lines 473-538)
```python
async def test_kv_consumuer(monkeypatch):
    """
    Simulates a Consumer Worker (Decoder) initiating a pull from a Producer.

    Verifies that MooncakeXferMetadata is correctly serialized and sent via ZMQ.
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_consumer"
    )
    with set_current_vllm_config(vllm_config), patch_worker_dependencies() as mocks:
        decode_connector = MooncakeConnector(
            vllm_config,
            KVConnectorRole.WORKER,
            _make_test_kv_cache_config(),
        )
        decode_worker = decode_connector.connector_worker
    # ... excerpt omitted for brevity ...
        assert sent_meta.remote_hostname == "127.0.0.1"
        assert sent_meta.remote_port == 54321
        assert sent_meta.req_blocks["d-req-1"] == ("xfer-req-1", [[100, 101]])
        # Verify internal state is updated correctly.
        assert "d-req-1" in decode_worker.finished_recving_reqs
        # Clean up
        decode_worker.shutdown()
```
**EN:** Async test covering `KV consumuer`. Inputs/fixtures: `monkeypatch`. It exercises `create_vllm_config, set_current_vllm_config, patch_worker_dependencies, MooncakeConnector, MooncakeXferResponse, _encoder.encode`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `kv consumuer` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `create_vllm_config, set_current_vllm_config, patch_worker_dependencies, MooncakeConnector, MooncakeXferResponse, _encoder.encode`。 代码主体包含 4 个显式断言。

### test_worker_get_finished_timeout (lines 542-579)
```python
async def test_worker_get_finished_timeout(monkeypatch):
    """Tests the cleanup mechanism for requests."""

    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_producer"
    )
    with set_current_vllm_config(vllm_config), patch_worker_dependencies():
        prefill_connector = MooncakeConnector(
            vllm_config,
            KVConnectorRole.WORKER,
            _make_test_kv_cache_config(),
        )
        prefill_worker = prefill_connector.connector_worker
        # Add an expired request (expire_time is in the past).
        prefill_worker.reqs_need_send["tx-expired"] = SendBlockMeta(
            p_req_id="p-req-expired",
            transfer_id="tx-expired",
    # ... excerpt omitted for brevity ...
            ready=MagicMock(),
        finished_reqs = await prefill_worker.fetch_finished_sending_reqs()
        assert "p-req-expired" in finished_reqs
        assert "p-req-active" not in finished_reqs
        assert "tx-expired" not in prefill_worker.reqs_need_send
        assert "tx-active" in prefill_worker.reqs_need_send
```
**EN:** Async test covering `worker get finished timeout`. Inputs/fixtures: `monkeypatch`. It exercises `create_vllm_config, set_current_vllm_config, patch_worker_dependencies, MooncakeConnector, SendBlockMeta, _make_test_kv_cache_config`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `worker get finished timeout` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `create_vllm_config, set_current_vllm_config, patch_worker_dependencies, MooncakeConnector, SendBlockMeta, _make_test_kv_cache_config`。 代码主体包含 4 个显式断言。

### test_register_kv_caches (lines 582-632)
```python
def test_register_kv_caches():
    """Tests the memory registration logic with the underlying Mooncake engine."""

    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_consumer"
    )
    with (
        set_current_vllm_config(vllm_config),
        patch_worker_dependencies(),
        patch(
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector.threading.Event"
        ),
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector.threading.Thread"
        ) as mock_thread,
    ):
        connector = MooncakeConnector(
    # ... excerpt omitted for brevity ...
            expected_ptrs = {
            assert set(registered_ptrs) == expected_ptrs
            assert set(registered_lens) == {tensor1[0].nbytes}
            # Verify block_len_per_layer is set correctly.
            assert len(worker.block_len_per_layer) == len(registered_ptrs)
            for bl in worker.block_len_per_layer:
                assert bl == tensor1[0].nbytes // tensor1.shape[1]
```
**EN:** Test case covering `register KV caches`. It exercises `create_vllm_config, set_current_vllm_config, patch_worker_dependencies, patch, MooncakeConnector, FlashAttentionBackend.get_kv_cache_shape`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `register kv caches` 的测试用例。 该测试会调用 `create_vllm_config, set_current_vllm_config, patch_worker_dependencies, patch, MooncakeConnector, FlashAttentionBackend.get_kv_cache_shape`。 代码主体包含 4 个显式断言。

### test_register_kv_caches_supports_mixed_mla_and_eagle_shapes (lines 635-681)
```python
def test_register_kv_caches_supports_mixed_mla_and_eagle_shapes():
    """Mixed MLA+Eagle caches should register by byte length, not shape."""

    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_consumer"
    )
    with (
        set_current_vllm_config(vllm_config),
        patch_worker_dependencies(),
        patch(
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector.threading.Event"
        ),
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector.threading.Thread"
        ) as mock_thread,
    ):
        connector = MooncakeConnector(
    # ... excerpt omitted for brevity ...
        assert registered_ptrs == [mla_cache.data_ptr(), eagle_cache.data_ptr()]
        assert registered_lens == [mla_cache.nbytes, eagle_cache.nbytes]
        assert worker.block_len_per_layer == [
            mla_cache.nbytes // mla_cache.shape[0],
            eagle_cache.nbytes // eagle_cache.shape[0],
        ]
```
**EN:** Test case covering `register KV caches supports mixed MLA and eagle shapes`. It exercises `create_vllm_config, set_current_vllm_config, patch_worker_dependencies, patch, MooncakeConnector, torch.zeros`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `register kv caches supports mixed mla and eagle shapes` 的测试用例。 该测试会调用 `create_vllm_config, set_current_vllm_config, patch_worker_dependencies, patch, MooncakeConnector, torch.zeros`。 代码主体包含 3 个显式断言。

### test_kv_producer_heterogeneous_tp (lines 691-834)
```python
async def test_kv_producer_heterogeneous_tp(monkeypatch, d_tp_size):
    """
    Tests heterogeneous TP support in the producer transfer path.

    Verifies correct pointer and offset calculation when producer TP=2
    sends to consumer with TP=1 (P>D) or TP=4 (P<D).
    Parametrized cases:
    - P TP=2 > D TP=1: one D rank receives; dst_offset based on P rank
    - P TP=2 < D TP=4: two D ranks receive; src_offset based on D rank
    P_TP_SIZE = 2
    P_TP_RANK = 0
    LOCAL_BLOCK_LEN = 4096
    local_block_len = LOCAL_BLOCK_LEN
    remote_block_len = LOCAL_BLOCK_LEN * P_TP_SIZE // d_tp_size
    # ... excerpt omitted for brevity ...
        mock_socket = AsyncMock(spec=zmq.asyncio.Socket)
        mock_socket.send_multipart = AsyncMock()
                assert len(src_ptrs) == len(flat_local)
                assert len(dst_ptrs) == len(flat_local)
                assert len(lengths) == len(flat_local)
                # Compute expected offsets based on TP ratio
        # After serving all D ranks, the request should be complete
        assert transfer_id not in prefill_worker.reqs_need_send
        assert "p-req-h1" in prefill_worker.finished_sending_reqs
        prefill_worker.sender_loop = origin_sender_loop
        prefill_worker.shutdown()
```
**EN:** Parameterized test covering `KV producer heterogeneous tp`. Parameter axes: `d_tp_size`. Inputs/fixtures: `monkeypatch, d_tp_size`. It exercises `patch, mark.parametrize, monkeypatch.setenv, create_vllm_config, set_current_vllm_config, patch_worker_dependencies`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `kv producer heterogeneous tp` 的测试用例。 参数维度：`d_tp_size`。 输入或 fixture：`monkeypatch, d_tp_size`。 该测试会调用 `patch, mark.parametrize, monkeypatch.setenv, create_vllm_config, set_current_vllm_config, patch_worker_dependencies`。 代码主体包含 10 个显式断言。

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
- **EN:** External libraries: `pytest, torch, zmq.asyncio, httpx`.
- **CN:** 外部库：`pytest, torch, zmq.asyncio, httpx`。
- **EN:** vLLM modules under test: `vllm.config, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils, vllm.utils.network_utils, vllm.v1.attention.backends.flash_attn, vllm.v1.kv_cache_interface, vllm.v1.request`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils, vllm.utils.network_utils, vllm.v1.attention.backends.flash_attn, vllm.v1.kv_cache_interface, vllm.v1.request`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `asyncio, contextlib, time, unittest.mock`.
- **CN:** 标准库支持：`asyncio, contextlib, time, unittest.mock`。
