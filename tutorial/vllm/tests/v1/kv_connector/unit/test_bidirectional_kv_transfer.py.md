# test_bidirectional_kv_transfer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_bidirectional_kv_transfer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for bi-directional KV cache transfer between P and D nodes. / 该文件的文档字符串表明其用途：`unit tests for bi-directional KV 缓存 transfer between p and d nodes`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-26)
```python
"""Unit tests for bi-directional KV cache transfer between P and D nodes.

Tests cover the new behaviors added by the bi-directional KV transfer PR:
1. P-node scheduler lifecycle: P pulls KV from D using remote_block_ids,
   eliminating redundant prefill computation in multi-turn conversations.
2. P-node metadata: NixlConnectorMetadata correctly populates recv metadata
   when P pulls KV from D (do_remote_decode=True + remote_block_ids).
3. P-node worker: start_load_kv processes reqs_to_recv for KV pull from D.
4. D-node request_finished: returns kv_transfer_params with remote_block_ids
   and remote_num_tokens so P can pull KV in future turns.
5. Edge cases:
   - No double read after reschedule (_remote_blocks_processed flag)
   - remote_num_tokens bounded by block capacity (num_computed_tokens)
   - kv_recompute_threshold skips small transfers
   - P-node holds blocks for D after finishing
   - Cache MISS first turn falls back to local prefill
   - Partial remote coverage: P pulls partial, computes the rest
   - _remote_blocks_processed flag persists across reschedules

P-node flags: do_remote_prefill=False (prefill locally),
do_remote_decode=True (don't decode locally, send KV to D).
P pulls KV from D when remote_block_ids is not None and
external tokens > 0.
"""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for bi-directional KV cache transfer between P and D nodes.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for bi-directional KV 缓存 transfer between p and d nodes`。

### Imports and setup / 导入与设置 (lines 28-54)
```python
import copy
import time
from unittest.mock import patch

import pytest

from vllm.distributed.kv_transfer.kv_connector.v1.base import KVConnectorRole
from vllm.distributed.kv_transfer.kv_connector.v1.nixl.connector import (
    NixlConnector,
    NixlConnectorMetadata,
)
from vllm.forward_context import ForwardContext
from vllm.v1.outputs import (
    EMPTY_MODEL_RUNNER_OUTPUT,
    KVConnectorOutput,
)
from vllm.v1.request import RequestStatus

from .test_nixl_connector import FakeNixlConnectorWorker, FakeNixlWrapper
from .utils import (
    assert_scheduler_empty,
    create_model_runner_output,
    create_request,
    create_scheduler,
    create_vllm_config,
    make_kv_cache_config,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.nixl.connector, vllm.forward_context, vllm.v1.outputs, vllm.v1.request`. Local helpers come from `tests.v1.kv_connector.unit.test_nixl_connector, tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.nixl.connector, vllm.forward_context, vllm.v1.outputs, vllm.v1.request`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.test_nixl_connector, tests.v1.kv_connector.unit.utils`。

### Module state / 模块级状态 (lines 56-59)
```python
pytestmark = pytest.mark.cpu_test

# Common extra config for all bi-directional KV transfer tests.
BIDIR_KV_EXTRA_CONFIG = {"bidirectional_kv_xfer": True, "kv_recompute_threshold": 0}
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark, BIDIR_KV_EXTRA_CONFIG`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark, BIDIR_KV_EXTRA_CONFIG`。 该块还设置了作用于整个文件的 pytest 标记。

### _make_p_node_turn2_request (lines 65-83)
```python
def _make_p_node_turn2_request(
    request_id, block_size, num_tokens, num_remote_blocks=3, remote_num_tokens=None
):
    """Create a P-node Turn 2 request with remote_block_ids from D."""
    request = create_request(
        request_id=request_id,
        block_size=block_size,
        num_tokens=num_tokens,
        do_remote_decode=True,
    )
    if remote_num_tokens is None:
        remote_num_tokens = num_remote_blocks * block_size
    request.kv_transfer_params["remote_block_ids"] = [list(range(num_remote_blocks))]
    request.kv_transfer_params["remote_num_tokens"] = remote_num_tokens
    request.kv_transfer_params["remote_engine_id"] = "decode-engine"
    request.kv_transfer_params["remote_request_id"] = f"decode-{request_id}"
    request.kv_transfer_params["remote_host"] = "decode-host"
    request.kv_transfer_params["remote_port"] = 5678
    return request
```
**EN:** Helper function `_make_p_node_turn2_request` encapsulates reusable logic for `p node turn2 request`. Inputs: `request_id, block_size, num_tokens, num_remote_blocks, remote_num_tokens`. Key calls include `create_request, list, range`.
**CN:** 辅助函数 `_make_p_node_turn2_request` 封装了与 `p node turn2 request` 相关的可复用逻辑。 输入参数：`request_id, block_size, num_tokens, num_remote_blocks, remote_num_tokens`。 关键调用包括 `create_request, list, range`。

### _make_connector_with_fake_worker (lines 86-111)
```python
def _make_connector_with_fake_worker(
    hand_shake_latency=0, cycles_before_done=0, do_handshake=True
):
    """Create a NixlConnector with FakeNixlConnectorWorker."""
    vllm_config = create_vllm_config()
    kv_cache_config = make_kv_cache_config(block_size=16, num_blocks=2)
    connector = NixlConnector(vllm_config, KVConnectorRole.WORKER, kv_cache_config)
    connector.connector_worker = FakeNixlConnectorWorker(
        vllm_config,
        connector.engine_id,
        hand_shake_latency=hand_shake_latency,
        kv_cache_config=kv_cache_config,
    )
    worker = connector.connector_worker
    assert isinstance(worker.nixl_wrapper, FakeNixlWrapper)
    worker.nixl_wrapper.set_cycles_before_xfer_done(cycles_before_done)
    worker.kv_cache_layout = "HND"
    if do_handshake:
        remote_agents = worker._nixl_handshake(
            host="localhost",
            port=1234,
            remote_tp_size=1,
            expected_engine_id=FakeNixlConnectorWorker.REMOTE_ENGINE_ID,
        )
        worker._remote_agents[FakeNixlConnectorWorker.REMOTE_ENGINE_ID] = remote_agents
    return connector, worker
```
**EN:** Helper function `_make_connector_with_fake_worker` encapsulates reusable logic for `connector with fake worker`. Inputs: `hand_shake_latency, cycles_before_done, do_handshake`. Key calls include `create_vllm_config, make_kv_cache_config, NixlConnector, FakeNixlConnectorWorker, isinstance, nixl_wrapper.set_cycles_before_xfer_done`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_make_connector_with_fake_worker` 封装了与 `connector with fake worker` 相关的可复用逻辑。 输入参数：`hand_shake_latency, cycles_before_done, do_handshake`。 关键调用包括 `create_vllm_config, make_kv_cache_config, NixlConnector, FakeNixlConnectorWorker, isinstance, nixl_wrapper.set_cycles_before_xfer_done`。 其中包含 1 个内部断言，用于保护前置假设。

### _make_p_node_recv_metadata (lines 114-131)
```python
def _make_p_node_recv_metadata(request_id, local_blocks, remote_blocks):
    """Build NixlConnectorMetadata for P-node pulling KV from D."""
    meta = NixlConnectorMetadata()
    meta.add_new_req_to_recv(
        request_id=request_id,
        local_block_ids=(local_blocks,),
        kv_transfer_params={
            "do_remote_prefill": False,
            "do_remote_decode": True,
            "remote_block_ids": (remote_blocks,),
            "remote_engine_id": FakeNixlConnectorWorker.REMOTE_ENGINE_ID,
            "remote_request_id": f"decode-{request_id}",
            "remote_host": "localhost",
            "remote_port": 1234,
            "remote_tp_size": 1,
        },
    )
    return meta
```
**EN:** Helper function `_make_p_node_recv_metadata` encapsulates reusable logic for `p node recv metadata`. Inputs: `request_id, local_blocks, remote_blocks`. Key calls include `NixlConnectorMetadata, meta.add_new_req_to_recv`.
**CN:** 辅助函数 `_make_p_node_recv_metadata` 封装了与 `p node recv metadata` 相关的可复用逻辑。 输入参数：`request_id, local_blocks, remote_blocks`。 关键调用包括 `NixlConnectorMetadata, meta.add_new_req_to_recv`。

### _do_load_kv (lines 134-138)
```python
def _do_load_kv(connector, metadata):
    """Bind metadata and call start_load_kv."""
    connector.bind_connector_metadata(metadata)
    ctx = ForwardContext(no_compile_layers={}, attn_metadata={}, slot_mapping={})
    connector.start_load_kv(ctx)
```
**EN:** Helper function `_do_load_kv` encapsulates reusable logic for `do load KV`. Inputs: `connector, metadata`. Key calls include `connector.bind_connector_metadata, ForwardContext, connector.start_load_kv`.
**CN:** 辅助函数 `_do_load_kv` 封装了与 `do load kv` 相关的可复用逻辑。 输入参数：`connector, metadata`。 关键调用包括 `connector.bind_connector_metadata, ForwardContext, connector.start_load_kv`。

### test_multiturn_lifecycle (lines 144-191)
```python
def test_multiturn_lifecycle():
    """Full two-turn lifecycle on the P node:
    Turn 1: P prefills locally (do_remote_prefill=False), sends KV to D
    (do_remote_decode=True). Finishes LENGTH_CAPPED with remote_block_ids.
    Turn 2: P receives remote_block_ids from D. P pulls KV from D because
    remote_block_ids is not None and external tokens > 0. Computes only
    new tokens, finishes LENGTH_CAPPED."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size

    t1 = create_request(
        request_id=100, block_size=BS, num_tokens=int(BS * 2.5), do_remote_decode=True
    scheduler.add_request(t1)
    t1_id = t1.request_id
    # ... excerpt omitted for brevity ...
    assert t1.status == RequestStatus.FINISHED_LENGTH_CAPPED
    assert kv and sum(len(g) for g in kv["remote_block_ids"]) > 0
    assert t2.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    assert t2.status == RequestStatus.FINISHED_LENGTH_CAPPED
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={t1_id, t2_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `multiturn lifecycle`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `multiturn lifecycle` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`。 代码主体包含 4 个显式断言。

### test_first_turn_no_remote_blocks (lines 194-219)
```python
def test_first_turn_no_remote_blocks():
    """First turn: P has no remote_block_ids from D yet.
    Standard local prefill, returns kv_transfer_params for future turns."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=3, block_size=BS, num_tokens=int(BS * 2.5), do_remote_decode=True
    )
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    assert req.status != RequestStatus.WAITING_FOR_REMOTE_KVS
    mro = create_model_runner_output(reqs=[req])
    eco = scheduler.update_from_output(so, mro)
    assert req.status == RequestStatus.FINISHED_LENGTH_CAPPED
    assert eco[0].outputs[0].kv_transfer_params is not None
    so = scheduler.schedule()
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `first turn no remote blocks`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `first turn no remote blocks` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`。 代码主体包含 3 个显式断言。

### test_abort_p_side_during_send (lines 222-245)
```python
def test_abort_p_side_during_send():
    """P-side do_remote_decode=True: blocks held until finished_sending."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=42, block_size=BS, num_tokens=int(BS * 2.5), do_remote_decode=True
    )
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    mro = create_model_runner_output(reqs=[req])
    scheduler.update_from_output(so, mro)
    assert req_id in scheduler.requests
    so = scheduler.schedule()
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    assert req_id in scheduler.requests
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `abort p side during send`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort p side during send` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`。 代码主体包含 2 个显式断言。

### test_abort_p_side_non_length_capped (lines 248-271)
```python
def test_abort_p_side_non_length_capped():
    """P-side abort with non-LENGTH_CAPPED → immediate block free."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=44, block_size=BS, num_tokens=int(BS * 2.5), do_remote_decode=True
    )
    req.sampling_params.max_tokens = 100
    req.max_tokens = 100
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    mro = create_model_runner_output(reqs=[req])
    scheduler.update_from_output(so, mro)
    scheduler.finish_requests([req_id], RequestStatus.FINISHED_ABORTED)
    conn = scheduler.connector.connector_scheduler
    assert req_id in conn._reqs_not_processed
    assert req_id not in scheduler.requests
    so = scheduler.schedule()
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `abort p side non length capped`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `abort p side non length capped` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`。 代码主体包含 2 个显式断言。

### test_remote_blocks_exceed_prompt_tokens (lines 274-306)
```python
def test_remote_blocks_exceed_prompt_tokens():
    """D provides more remote tokens than P's prompt needs.
    P caps external tokens to prompt length."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    NUM_TOKENS = int(BS * 2.5)
    req = _make_p_node_turn2_request(
        300, BS, NUM_TOKENS, num_remote_blocks=5, remote_num_tokens=5 * BS
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    assert req.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    assert req.num_computed_tokens == NUM_TOKENS
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    # ... excerpt omitted for brevity ...
    assert req.status == RequestStatus.FINISHED_LENGTH_CAPPED
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `remote blocks exceed prompt tokens`. It exercises `create_vllm_config, create_scheduler, int, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `remote blocks exceed prompt tokens` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 3 个显式断言。

### test_p_node_pulls_partial_last_block_from_d (lines 309-349)
```python
def test_p_node_pulls_partial_last_block_from_d():
    """D sends remote_block_ids with partially filled last block.
    remote_num_tokens < len(remote_block_ids) * block_size.
    P pulls only remote_num_tokens worth of external tokens."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    num_remote_blocks = 3
    remote_num_tokens = int(BS * 2.5)
    assert remote_num_tokens < num_remote_blocks * BS
    NUM_TOKENS = int(BS * 3.5)
    req = _make_p_node_turn2_request(
        400,
        BS,
        NUM_TOKENS,
        num_remote_blocks=num_remote_blocks,
    # ... excerpt omitted for brevity ...
    assert req.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    assert len(scheduler.running) == 1
    assert req.status == RequestStatus.FINISHED_LENGTH_CAPPED
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `p node pulls partial last block from d`. It exercises `create_vllm_config, create_scheduler, int, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `p node pulls partial last block from d` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 4 个显式断言。

### test_add_new_req_to_recv_populates_remote_meta (lines 355-380)
```python
def test_add_new_req_to_recv_populates_remote_meta():
    """add_new_req_to_recv correctly populates RemoteMeta for P-node
    bi-directional KV pull from D."""
    meta = NixlConnectorMetadata()
    kv_params = {
        "remote_block_ids": [[0, 1, 2]],
        "remote_engine_id": "decode-engine",
        "remote_request_id": "decode-req-123",
        "remote_host": "decode-host",
        "remote_port": 5678,
    }
    local_block_ids = ([10, 11, 12],)
    meta.add_new_req_to_recv(
        request_id="test-req",
        local_block_ids=local_block_ids,
        kv_transfer_params=kv_params,
    )
    assert "test-req" in meta.reqs_to_recv
    rm = meta.reqs_to_recv["test-req"]
    assert rm.remote is not None
    assert rm.remote.block_ids == kv_params["remote_block_ids"]
    assert rm.remote.engine_id == "decode-engine"
    assert rm.remote.request_id == "decode-req-123"
    assert rm.remote.host == "decode-host"
    assert rm.remote.port == 5678
    assert rm.local_block_ids == local_block_ids
```
**EN:** Test case covering `add new req to recv populates remote meta`. It exercises `NixlConnectorMetadata, meta.add_new_req_to_recv`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `add new req to recv populates remote meta` 的测试用例。 该测试会调用 `NixlConnectorMetadata, meta.add_new_req_to_recv`。 代码主体包含 8 个显式断言。

### test_build_connector_meta_recv_entries (lines 383-401)
```python
def test_build_connector_meta_recv_entries():
    """P-node scheduler: do_remote_decode=True + remote_block_ids →
    _reqs_need_recv populated, build_connector_meta produces reqs_to_recv."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = _make_p_node_turn2_request(1, BS, int(BS * 2.5))
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    assert req.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    meta = so.kv_connector_metadata
    assert isinstance(meta, NixlConnectorMetadata)
    assert req_id in meta.reqs_to_recv
    rm = meta.reqs_to_recv[req_id]
    assert rm.remote is not None
    assert rm.remote.engine_id == "decode-engine"
```
**EN:** Test case covering `build connector meta recv entries`. It exercises `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, isinstance`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `build connector meta recv entries` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, isinstance`。 代码主体包含 5 个显式断言。

### test_build_connector_meta_clears_reqs_need_recv (lines 404-415)
```python
def test_build_connector_meta_clears_reqs_need_recv():
    """After build_connector_meta, _reqs_need_recv is cleared."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = _make_p_node_turn2_request(2, BS, int(BS * 2.5))
    scheduler.add_request(req)
    conn = scheduler.connector.connector_scheduler
    scheduler.schedule()
    assert len(conn._reqs_need_recv) == 0
```
**EN:** Test case covering `build connector meta clears reqs need recv`. It exercises `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, int`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `build connector meta clears reqs need recv` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, int`。 代码主体包含 1 个显式断言。

### test_build_connector_meta_multiple_requests (lines 418-433)
```python
def test_build_connector_meta_multiple_requests():
    """Multiple P-node requests all included in reqs_to_recv."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    reqs = [_make_p_node_turn2_request(10 + i, BS, int(BS * 2.5)) for i in range(3)]
    for r in reqs:
        scheduler.add_request(r)
    so = scheduler.schedule()
    meta = so.kv_connector_metadata
    assert isinstance(meta, NixlConnectorMetadata)
    assert len(meta.reqs_to_recv) == 3
    for r in reqs:
        assert r.request_id in meta.reqs_to_recv
```
**EN:** Test case covering `build connector meta multiple requests`. It exercises `create_vllm_config, create_scheduler, scheduler.schedule, isinstance, _make_p_node_turn2_request, scheduler.add_request`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `build connector meta multiple requests` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, scheduler.schedule, isinstance, _make_p_node_turn2_request, scheduler.add_request`。 代码主体包含 3 个显式断言。

### test_p_node_pull_kv_from_d (lines 443-450)
```python
def test_p_node_pull_kv_from_d(dist_init):
    """P node pulls KV from D via start_load_kv with reqs_to_recv."""
    connector, worker = _make_connector_with_fake_worker()
    meta = _make_p_node_recv_metadata("req-p1", [10, 11, 12], [20, 21, 22])
    _do_load_kv(connector, meta)
    assert "req-p1" in worker._recving_metadata
    _, done_recving = connector.get_finished(finished_req_ids=set())
    assert "req-p1" in done_recving
```
**EN:** Test case covering `p node pull KV from d`. Inputs/fixtures: `dist_init`. It exercises `patch, _make_connector_with_fake_worker, _make_p_node_recv_metadata, _do_load_kv, connector.get_finished, set`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `p node pull kv from d` 的测试用例。 输入或 fixture：`dist_init`。 该测试会调用 `patch, _make_connector_with_fake_worker, _make_p_node_recv_metadata, _do_load_kv, connector.get_finished, set`。 代码主体包含 2 个显式断言。

### test_p_node_pull_then_send_kv (lines 457-472)
```python
def test_p_node_pull_then_send_kv(dist_init):
    """Full P-node bi-directional: pull KV from D → prefill →
    send KV back to D via notification."""
    connector, worker = _make_connector_with_fake_worker()
    meta = _make_p_node_recv_metadata("req-p2", [10, 11], [20, 21])
    _do_load_kv(connector, meta)
    _, done_recving = connector.get_finished(finished_req_ids=set())
    assert "req-p2" in done_recving
    worker._reqs_to_send["req-p2"] = time.perf_counter() + 60
    worker._reqs_to_process.add("req-p2")
    notif = f"req-p2:{worker.world_size}".encode()
    orig = worker.nixl_wrapper.get_new_notifs
    worker.nixl_wrapper.get_new_notifs = lambda: {"agent": [notif]}
    done_sending, _ = connector.get_finished(finished_req_ids=set())
    assert "req-p2" in done_sending
    worker.nixl_wrapper.get_new_notifs = orig
```
**EN:** Test case covering `p node pull then send KV`. Inputs/fixtures: `dist_init`. It exercises `patch, _make_connector_with_fake_worker, _make_p_node_recv_metadata, _do_load_kv, connector.get_finished, _reqs_to_process.add`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `p node pull then send kv` 的测试用例。 输入或 fixture：`dist_init`。 该测试会调用 `patch, _make_connector_with_fake_worker, _make_p_node_recv_metadata, _do_load_kv, connector.get_finished, _reqs_to_process.add`。 代码主体包含 2 个显式断言。

### test_p_node_deferred_pull_on_no_handshake (lines 479-497)
```python
def test_p_node_deferred_pull_on_no_handshake(dist_init):
    """P defers KV pull when no prior handshake exists."""
    connector, worker = _make_connector_with_fake_worker(
        hand_shake_latency=0, do_handshake=False
    )
    meta = _make_p_node_recv_metadata("req-p3", [10, 11], [20, 21])
    _do_load_kv(connector, meta)
    assert "req-p3" in worker._recving_metadata
    timeout = 3.0
    start = time.perf_counter()
    while time.perf_counter() - start < timeout:
        connector.bind_connector_metadata(NixlConnectorMetadata())
        ctx = ForwardContext(no_compile_layers={}, attn_metadata={}, slot_mapping={})
        connector.start_load_kv(ctx)
        _, done = connector.get_finished(finished_req_ids=set())
        if "req-p3" in done:
            return
        time.sleep(0.2)
    raise AssertionError("Transfer did not complete after async handshake")
```
**EN:** Test case covering `p node deferred pull on no handshake`. Inputs/fixtures: `dist_init`. It exercises `patch, _make_connector_with_fake_worker, _make_p_node_recv_metadata, _do_load_kv, time.perf_counter, AssertionError`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `p node deferred pull on no handshake` 的测试用例。 输入或 fixture：`dist_init`。 该测试会调用 `patch, _make_connector_with_fake_worker, _make_p_node_recv_metadata, _do_load_kv, time.perf_counter, AssertionError`。 代码主体包含 1 个显式断言。

### test_d_node_request_finished_returns_kv_params (lines 503-532)
```python
def test_d_node_request_finished_returns_kv_params():
    """D-node request_finished returns kv_transfer_params with
    do_remote_decode=True, remote_block_ids, remote_num_tokens
    for P to pull. These params go directly to P node."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=1, block_size=BS, num_tokens=int(BS * 2.5), do_remote_prefill=True
    )
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    scheduler.update_from_output(
        so, create_model_runner_output(reqs=[], finished_recving={req_id})
    )
    so = scheduler.schedule()
    eco = scheduler.update_from_output(
        so, create_model_runner_output(reqs=[req], use_eos=True)
    )
    assert req.status == RequestStatus.FINISHED_STOPPED
    kv = eco[0].outputs[0].kv_transfer_params
    assert kv is not None
    assert kv["do_remote_decode"] is True
    assert kv["do_remote_prefill"] is False
    assert "remote_block_ids" in kv
    assert "remote_num_tokens" in kv
    assert kv["remote_num_tokens"] > 0
```
**EN:** Test case covering `d node request finished returns KV params`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `d node request finished returns kv params` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`。 代码主体包含 7 个显式断言。

### test_d_node_request_finished_delays_block_free (lines 535-557)
```python
def test_d_node_request_finished_delays_block_free():
    """D-node holds blocks (delay_free=True) until P reads them."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=2, block_size=BS, num_tokens=int(BS * 2.5), do_remote_prefill=True
    )
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    scheduler.update_from_output(
        so, create_model_runner_output(reqs=[], finished_recving={req_id})
    )
    so = scheduler.schedule()
    scheduler.update_from_output(
        so, create_model_runner_output(reqs=[req], use_eos=True)
    )
    assert req_id in scheduler.requests
    conn = scheduler.connector.connector_scheduler
    assert req_id in conn._reqs_need_send
```
**EN:** Test case covering `d node request finished delays block free`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `d node request finished delays block free` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`。 代码主体包含 2 个显式断言。

### test_d_node_request_finished_remote_num_tokens (lines 560-582)
```python
def test_d_node_request_finished_remote_num_tokens():
    """D-node kv_transfer_params includes correct remote_num_tokens."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=3, block_size=BS, num_tokens=int(BS * 2.5), do_remote_prefill=True
    )
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    scheduler.update_from_output(
        so, create_model_runner_output(reqs=[], finished_recving={req_id})
    )
    so = scheduler.schedule()
    eco = scheduler.update_from_output(
        so, create_model_runner_output(reqs=[req], use_eos=True)
    )
    kv = eco[0].outputs[0].kv_transfer_params
    assert kv["remote_num_tokens"] > 0
    assert sum(len(g) for g in kv["remote_block_ids"]) > 0
```
**EN:** Test case covering `d node request finished remote num tokens`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `d node request finished remote num tokens` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`。 代码主体包含 2 个显式断言。

### test_d_node_partial_last_block_remote_num_tokens (lines 585-610)
```python
def test_d_node_partial_last_block_remote_num_tokens():
    """D-node: remote_num_tokens < len(remote_block_ids) * block_size
    when last block is partially filled."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=5, block_size=BS, num_tokens=int(BS * 2.5), do_remote_prefill=True
    )
    scheduler.add_request(req)
    req_id = req.request_id
    so = scheduler.schedule()
    scheduler.update_from_output(
        so, create_model_runner_output(reqs=[], finished_recving={req_id})
    )
    so = scheduler.schedule()
    eco = scheduler.update_from_output(
        so, create_model_runner_output(reqs=[req], use_eos=True)
    )
    kv = eco[0].outputs[0].kv_transfer_params
    total_blocks = sum(len(g) for g in kv["remote_block_ids"])
    assert total_blocks == 3
    assert kv["remote_num_tokens"] < total_blocks * BS
    assert kv["remote_num_tokens"] > 0
```
**EN:** Test case covering `d node partial last block remote num tokens`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `d node partial last block remote num tokens` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`。 代码主体包含 3 个显式断言。

### test_no_double_read_blocks_after_reschedule (lines 616-667)
```python
def test_no_double_read_blocks_after_reschedule():
    """Edge case 1: update_state_after_alloc called twice for the same
    bidirectional request (once on initial schedule, once after
    WAITING_FOR_REMOTE_KVS → reschedule). The _remote_blocks_processed
    flag must prevent the request from being added to _reqs_need_recv
    twice, which would cause P to read D's blocks twice."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = _make_p_node_turn2_request(500, BS, int(BS * 2.5))
    scheduler.add_request(req)
    req_id = req.request_id
    conn = scheduler.connector.connector_scheduler

    # First schedule: request enters WAITING_FOR_REMOTE_KVS,
    # _reqs_need_recv populated then cleared by build_connector_meta.
    # ... excerpt omitted for brevity ...
    assert req.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    assert isinstance(meta, NixlConnectorMetadata)
    assert req_id in meta.reqs_to_recv
    assert len(conn._reqs_need_recv) == 0
    assert isinstance(meta2, NixlConnectorMetadata)
    assert req_id not in meta2.reqs_to_recv
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `no double read blocks after reschedule`. It exercises `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, isinstance`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `no double read blocks after reschedule` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, isinstance`。 代码主体包含 7 个显式断言。

### test_remote_num_tokens_bounded_by_blocks (lines 670-704)
```python
def test_remote_num_tokens_bounded_by_blocks():
    """Edge case 2: D-node request_finished must return
    remote_num_tokens <= len(remote_block_ids) * block_size.
    request.num_tokens includes the last sampled token which has no KV
    in the cache, so remote_num_tokens must use num_computed_tokens."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=501,
        block_size=BS,
        num_tokens=int(BS * 2.5),
        do_remote_prefill=True,
    scheduler.add_request(req)
    req_id = req.request_id
    # ... excerpt omitted for brevity ...
    assert kv is not None
    max_tokens_in_blocks = total_blocks * BS
    assert kv["remote_num_tokens"] <= max_tokens_in_blocks, (
        f"remote_num_tokens ({kv['remote_num_tokens']}) exceeds "
        f"block capacity ({max_tokens_in_blocks})"
    assert kv["remote_num_tokens"] > 0
```
**EN:** Test case covering `remote num tokens bounded by blocks`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `remote num tokens bounded by blocks` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`。 代码主体包含 3 个显式断言。

### test_kv_recompute_threshold_skips_small_transfer (lines 707-745)
```python
def test_kv_recompute_threshold_skips_small_transfer():
    """Edge case 3: When remote tokens are below kv_recompute_threshold,
    P should skip the remote pull and compute locally instead of
    entering WAITING_FOR_REMOTE_KVS."""
    threshold = 256
    vllm_config = create_vllm_config(
        kv_connector_extra_config={
            "bidirectional_kv_xfer": True,
            "kv_recompute_threshold": threshold,
        },
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size

    # Create request where remote tokens (48) < threshold (256)
    req = _make_p_node_turn2_request(
        502,
        BS,
    # ... excerpt omitted for brevity ...
    assert req.status != RequestStatus.WAITING_FOR_REMOTE_KVS
    assert req.status == RequestStatus.RUNNING
    assert req.status == RequestStatus.FINISHED_LENGTH_CAPPED
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req.request_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `KV recompute threshold skips small transfer`. It exercises `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `kv recompute threshold skips small transfer` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`。 代码主体包含 3 个显式断言。

### test_p_node_finished_holds_blocks_for_d (lines 748-787)
```python
def test_p_node_finished_holds_blocks_for_d():
    """Edge case 4: P-node finishes with FINISHED_LENGTH_CAPPED and
    do_remote_decode=True. P must hold blocks (delay_free=True) and
    return kv_transfer_params with do_remote_prefill=True so D can
    read P's blocks."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=503,
        block_size=BS,
        num_tokens=int(BS * 2.5),
        do_remote_decode=True,
    scheduler.add_request(req)
    req_id = req.request_id
    # ... excerpt omitted for brevity ...
    assert req.status == RequestStatus.FINISHED_LENGTH_CAPPED
    assert kv is not None
    assert kv["do_remote_prefill"] is True
    assert kv["do_remote_decode"] is False
    assert "remote_block_ids" in kv
    assert sum(len(g) for g in kv["remote_block_ids"]) > 0
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `p node finished holds blocks for d`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `p node finished holds blocks for d` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`。 代码主体包含 7 个显式断言。

### test_cache_miss_first_turn_no_remote_pull (lines 790-822)
```python
def test_cache_miss_first_turn_no_remote_pull():
    """Edge case 5: First turn with do_remote_decode=True but no
    remote_block_ids (cache MISS). P should prefill locally with
    num_external_tokens=0 and not enter WAITING_FOR_REMOTE_KVS."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = create_request(
        request_id=504,
        block_size=BS,
        num_tokens=int(BS * 2.5),
        do_remote_decode=True,
    # No remote_block_ids set — this is a cache MISS
    assert req.kv_transfer_params.get("remote_block_ids") is None
    scheduler.add_request(req)
    # ... excerpt omitted for brevity ...
    assert req.status != RequestStatus.WAITING_FOR_REMOTE_KVS
    assert req.status == RequestStatus.RUNNING
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req.request_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `cache miss first turn no remote pull`. It exercises `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `缓存 miss first turn no remote pull` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, create_request, scheduler.add_request, scheduler.schedule, create_model_runner_output`。 代码主体包含 3 个显式断言。

### test_partial_remote_tokens_less_than_prompt (lines 825-867)
```python
def test_partial_remote_tokens_less_than_prompt():
    """Edge case 6: D's remote_num_tokens covers only part of P's
    prompt. P should pull remote_num_tokens worth of external tokens
    and compute the rest locally."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    NUM_TOKENS = int(BS * 4.5)  # 72 tokens
    # D provides only 2 blocks (32 tokens) out of 72
    req = _make_p_node_turn2_request(
        505,
        BS,
        NUM_TOKENS,
        num_remote_blocks=2,
        remote_num_tokens=2 * BS,
    # ... excerpt omitted for brevity ...
    assert req.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    assert req.num_computed_tokens < NUM_TOKENS
    assert req.status == RequestStatus.FINISHED_LENGTH_CAPPED
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    so = scheduler.schedule()
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `partial remote tokens less than prompt`. It exercises `create_vllm_config, create_scheduler, int, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `partial remote tokens less than prompt` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, int, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule`。 代码主体包含 3 个显式断言。

### test_remote_blocks_processed_flag_persists (lines 870-915)
```python
def test_remote_blocks_processed_flag_persists():
    """Edge case 7: After recv completes and request is rescheduled,
    the _remote_blocks_processed flag in kv_transfer_params prevents
    the bidirectional path from re-entering _reqs_need_recv."""
    vllm_config = create_vllm_config(
        kv_connector_extra_config=BIDIR_KV_EXTRA_CONFIG,
    )
    scheduler = create_scheduler(vllm_config)
    BS = vllm_config.cache_config.block_size
    req = _make_p_node_turn2_request(506, BS, int(BS * 2.5))
    scheduler.add_request(req)
    req_id = req.request_id
    conn = scheduler.connector.connector_scheduler

    # First schedule → WAITING_FOR_REMOTE_KVS
    so = scheduler.schedule()
    assert req.status == RequestStatus.WAITING_FOR_REMOTE_KVS
    scheduler.update_from_output(so, EMPTY_MODEL_RUNNER_OUTPUT)
    # ... excerpt omitted for brevity ...
    assert req.kv_transfer_params.get("_remote_blocks_processed") is True
    assert req_id not in conn._reqs_need_recv
    assert isinstance(meta, NixlConnectorMetadata)
    assert req_id not in meta.reqs_to_recv
    mro = copy.deepcopy(EMPTY_MODEL_RUNNER_OUTPUT)
    mro.kv_connector_output = KVConnectorOutput(finished_sending={req_id})
    scheduler.update_from_output(so, mro)
    assert_scheduler_empty(scheduler)
```
**EN:** Test case covering `remote blocks processed flag persists`. It exercises `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `remote blocks processed flag persists` 的测试用例。 该测试会调用 `create_vllm_config, create_scheduler, _make_p_node_turn2_request, scheduler.add_request, scheduler.schedule, scheduler.update_from_output`。 代码主体包含 5 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.nixl.connector, vllm.forward_context, vllm.v1.outputs, vllm.v1.request`.
- **CN:** 被测试的 vLLM 模块：`vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.nixl.connector, vllm.forward_context, vllm.v1.outputs, vllm.v1.request`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.test_nixl_connector, tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.test_nixl_connector, tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `copy, time, unittest.mock`.
- **CN:** 标准库支持：`copy, time, unittest.mock`。
