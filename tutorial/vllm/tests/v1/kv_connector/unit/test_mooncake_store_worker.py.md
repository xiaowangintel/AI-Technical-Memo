# test_mooncake_store_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_mooncake_store_worker.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `mooncake store worker` behavior and regressions in the v1 stack. / 验证 v1 栈中 `mooncake store worker` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-27)
```python
import json
import logging
import math
import sys
import threading
import types
from types import SimpleNamespace
from unittest.mock import MagicMock, patch

import pytest
import torch

from vllm.distributed.kv_transfer.kv_connector.v1.mooncake import (
    rdma_utils,
)
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store import (
    worker,
)
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data import (  # noqa: E501
    ChunkedTokenDatabase,
    KeyMetadata,
    LoadSpec,
    ReqMeta,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.distributed.kv_transfer.kv_connector.v1.mooncake, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.distributed.kv_transfer.kv_connector.v1.mooncake, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data`。

### _make_store_sending_thread (lines 30-51)
```python
def _make_store_sending_thread(
    store: MagicMock,
    *,
    replicate_config: object | None = None,
) -> worker.KVCacheStoreSendingThread:
    token_database = ChunkedTokenDatabase(
        KeyMetadata("test-model", 0, 0, 0, 0), block_size=16
    )
    token_database.set_kv_caches_base_addr([0x1000])
    token_database.set_block_len([256])
    thread = worker.KVCacheStoreSendingThread(
        store=store,
        token_database=token_database,
        block_size=16,
        tp_rank=0,
        put_step=1,
        kv_role="kv_producer",
        ready_event=threading.Event(),
        replicate_config=replicate_config,
    )
    thread.request_queue.task_done = MagicMock()
    return thread
```
**EN:** Helper function `_make_store_sending_thread` encapsulates reusable logic for `store sending thread`. Inputs: `store`. Key calls include `ChunkedTokenDatabase, token_database.set_kv_caches_base_addr, token_database.set_block_len, worker.KVCacheStoreSendingThread, MagicMock, KeyMetadata`.
**CN:** 辅助函数 `_make_store_sending_thread` 封装了与 `store sending thread` 相关的可复用逻辑。 输入参数：`store`。 关键调用包括 `ChunkedTokenDatabase, token_database.set_kv_caches_base_addr, token_database.set_block_len, worker.KVCacheStoreSendingThread, MagicMock, KeyMetadata`。

### _make_store_recving_thread (lines 54-73)
```python
def _make_store_recving_thread(
    store: MagicMock,
    *,
    disk_offload_buffer_budget_bytes: int | None = None,
) -> worker.KVCacheStoreRecvingThread:
    token_database = ChunkedTokenDatabase(
        KeyMetadata("test-model", 0, 0, 0, 0), block_size=16
    )
    token_database.set_kv_caches_base_addr([0x1000])
    token_database.set_block_len([256])
    thread = worker.KVCacheStoreRecvingThread(
        store=store,
        token_database=token_database,
        block_size=16,
        tp_rank=0,
        ready_event=threading.Event(),
        disk_offload_buffer_budget_bytes=disk_offload_buffer_budget_bytes,
    )
    thread.request_queue.task_done = MagicMock()
    return thread
```
**EN:** Helper function `_make_store_recving_thread` encapsulates reusable logic for `store recving thread`. Inputs: `store`. Key calls include `ChunkedTokenDatabase, token_database.set_kv_caches_base_addr, token_database.set_block_len, worker.KVCacheStoreRecvingThread, MagicMock, KeyMetadata`.
**CN:** 辅助函数 `_make_store_recving_thread` 封装了与 `store recving thread` 相关的可复用逻辑。 输入参数：`store`。 关键调用包括 `ChunkedTokenDatabase, token_database.set_kv_caches_base_addr, token_database.set_block_len, worker.KVCacheStoreRecvingThread, MagicMock, KeyMetadata`。

### _make_load_req (lines 76-94)
```python
def _make_load_req(
    req_id: str,
    block_hashes: list[bytes],
    *,
    token_len: int,
    vllm_cached_tokens: int = 0,
) -> ReqMeta:
    return ReqMeta(
        req_id=req_id,
        token_len_chunk=token_len,
        block_ids=list(range(len(block_hashes))),
        block_hashes=block_hashes,
        load_spec=LoadSpec(
            vllm_cached_tokens=vllm_cached_tokens,
            kvpool_cached_tokens=token_len,
            can_load=True,
            token_len=token_len,
        ),
    )
```
**EN:** Helper function `_make_load_req` encapsulates reusable logic for `load req`. Inputs: `req_id, block_hashes`. Key calls include `ReqMeta, list, LoadSpec, range, len`.
**CN:** 辅助函数 `_make_load_req` 封装了与 `load req` 相关的可复用逻辑。 输入参数：`req_id, block_hashes`。 关键调用包括 `ReqMeta, list, LoadSpec, range, len`。

### _make_store_req (lines 97-105)
```python
def _make_store_req(req_id: str, block_hashes: list[bytes]) -> ReqMeta:
    return ReqMeta(
        req_id=req_id,
        token_len_chunk=32,
        block_ids=[0, 1],
        block_hashes=block_hashes,
        can_save=True,
        original_block_size=16,
    )
```
**EN:** Helper function `_make_store_req` encapsulates reusable logic for `store req`. Inputs: `req_id, block_hashes`. Key calls include `ReqMeta`.
**CN:** 辅助函数 `_make_store_req` 封装了与 `store req` 相关的可复用逻辑。 输入参数：`req_id, block_hashes`。 关键调用包括 `ReqMeta`。

### Module state / 模块级状态 (lines 108-116)
```python
_DISK_OFFLOAD_SINGLE_KEY_BYTES = worker._estimate_disk_offload_staging_bytes([256])
_DISK_OFFLOAD_USABLE_BUDGET_RATIO = 0.9
_DISK_OFFLOAD_BUDGET_FOR_THREE_KEYS = 4 * _DISK_OFFLOAD_SINGLE_KEY_BYTES
_DISK_OFFLOAD_BUDGET_FOR_SPLIT = math.ceil(
    2 * _DISK_OFFLOAD_SINGLE_KEY_BYTES / _DISK_OFFLOAD_USABLE_BUDGET_RATIO
)  # Allows two 256-byte chunks but not the third.
_DISK_OFFLOAD_BUDGET_TOO_SMALL = (
    _DISK_OFFLOAD_SINGLE_KEY_BYTES - 1
)  # Smaller than a single 256-byte chunk.
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_DISK_OFFLOAD_SINGLE_KEY_BYTES, _DISK_OFFLOAD_USABLE_BUDGET_RATIO, _DISK_OFFLOAD_BUDGET_FOR_THREE_KEYS, _DISK_OFFLOAD_BUDGET_FOR_SPLIT, _DISK_OFFLOAD_BUDGET_TOO_SMALL`. Shared setup calls include `worker._estimate_disk_offload_staging_bytes, math.ceil`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_DISK_OFFLOAD_SINGLE_KEY_BYTES, _DISK_OFFLOAD_USABLE_BUDGET_RATIO, _DISK_OFFLOAD_BUDGET_FOR_THREE_KEYS, _DISK_OFFLOAD_BUDGET_FOR_SPLIT, _DISK_OFFLOAD_BUDGET_TOO_SMALL`。 共享初始化调用包括 `worker._estimate_disk_offload_staging_bytes, math.ceil`。

### _FakeKVTransferConfig (lines 119-130)
```python
class _FakeKVTransferConfig:
    def __init__(
        self,
        *,
        kv_role: str = "kv_both",
        extra_config: dict[str, object] | None = None,
    ) -> None:
        self.kv_role = kv_role
        self.kv_connector_extra_config = extra_config or {}

    def get_from_extra_config(self, key: str, default: object) -> object:
        return self.kv_connector_extra_config.get(key, default)
```
**EN:** Class `_FakeKVTransferConfig` groups 0 test method(s) and 2 helper/fixture method(s).
**CN:** 类 `_FakeKVTransferConfig` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。

### _FakeModelConfig (lines 133-141)
```python
class _FakeModelConfig:
    model = "test-model"
    use_mla = False

    def get_num_layers(self, parallel_config) -> int:
        return 1

    def get_total_num_kv_heads(self) -> int:
        return 1
```
**EN:** Class `_FakeModelConfig` groups 0 test method(s) and 2 helper/fixture method(s).
**CN:** 类 `_FakeModelConfig` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。

### _make_vllm_config (lines 144-156)
```python
def _make_vllm_config(
    *, extra_config: dict[str, object] | None = None
) -> SimpleNamespace:
    return SimpleNamespace(
        model_config=_FakeModelConfig(),
        parallel_config=SimpleNamespace(
            pipeline_parallel_size=1,
            rank=0,
        ),
        kv_transfer_config=_FakeKVTransferConfig(extra_config=extra_config),
        cache_config=SimpleNamespace(block_size=16, num_gpu_blocks=10),
        kv_events_config=SimpleNamespace(enable_kv_cache_events=False),
    )
```
**EN:** Helper function `_make_vllm_config` encapsulates reusable logic for `vllm config`. Key calls include `SimpleNamespace, _FakeModelConfig, _FakeKVTransferConfig`.
**CN:** 辅助函数 `_make_vllm_config` 封装了与 `vllm config` 相关的可复用逻辑。 关键调用包括 `SimpleNamespace, _FakeModelConfig, _FakeKVTransferConfig`。

### _write_mooncake_config (lines 159-162)
```python
def _write_mooncake_config(tmp_path, config: dict[str, object]) -> str:
    config_path = tmp_path / "mooncake_config.json"
    config_path.write_text(json.dumps(config), encoding="utf-8")
    return str(config_path)
```
**EN:** Helper function `_write_mooncake_config` encapsulates reusable logic for `write mooncake config`. Inputs: `tmp_path, config`. Key calls include `config_path.write_text, str, json.dumps`.
**CN:** 辅助函数 `_write_mooncake_config` 封装了与 `write mooncake config` 相关的可复用逻辑。 输入参数：`tmp_path, config`。 关键调用包括 `config_path.write_text, str, json.dumps`。

### _install_fake_mooncake (lines 165-177)
```python
def _install_fake_mooncake(monkeypatch, store_instance: MagicMock):
    class FakeReplicateConfig:
        def __init__(self) -> None:
            self.preferred_segment = ""

    fake_store_module = types.ModuleType("mooncake.store")
    fake_store_module.MooncakeDistributedStore = lambda: store_instance  # type: ignore[attr-defined]
    fake_store_module.ReplicateConfig = FakeReplicateConfig  # type: ignore[attr-defined]
    fake_mooncake_module = types.ModuleType("mooncake")
    fake_mooncake_module.store = fake_store_module  # type: ignore[attr-defined]
    monkeypatch.setitem(sys.modules, "mooncake", fake_mooncake_module)
    monkeypatch.setitem(sys.modules, "mooncake.store", fake_store_module)
    return FakeReplicateConfig
```
**EN:** Helper function `_install_fake_mooncake` encapsulates reusable logic for `install fake mooncake`. Inputs: `monkeypatch, store_instance`. Key calls include `types.ModuleType, monkeypatch.setitem`.
**CN:** 辅助函数 `_install_fake_mooncake` 封装了与 `install fake mooncake` 相关的可复用逻辑。 输入参数：`monkeypatch, store_instance`。 关键调用包括 `types.ModuleType, monkeypatch.setitem`。

### _patch_worker_runtime (lines 180-187)
```python
def _patch_worker_runtime(monkeypatch, *, local_ip: str = "10.0.0.7") -> None:
    single_rank_group = SimpleNamespace(world_size=1, rank_in_group=0)
    monkeypatch.setattr(worker, "get_mooncake_dp_engine_index", lambda _: 0)
    monkeypatch.setattr(worker, "get_tensor_model_parallel_rank", lambda: 0)
    monkeypatch.setattr(worker, "get_tensor_model_parallel_world_size", lambda: 1)
    monkeypatch.setattr(worker, "get_pcp_group", lambda: single_rank_group)
    monkeypatch.setattr(worker, "get_dcp_group", lambda: single_rank_group)
    monkeypatch.setattr(worker, "get_ip", lambda: local_ip)
```
**EN:** Helper function `_patch_worker_runtime` encapsulates reusable logic for `patch worker runtime`. Inputs: `monkeypatch`. Key calls include `SimpleNamespace, monkeypatch.setattr`.
**CN:** 辅助函数 `_patch_worker_runtime` 封装了与 `patch worker runtime` 相关的可复用逻辑。 输入参数：`monkeypatch`。 关键调用包括 `SimpleNamespace, monkeypatch.setattr`。

### test_default_local_buffer_size_matches_pr40900 (lines 190-193)
```python
def test_default_local_buffer_size_matches_pr40900():
    """PR-40900 shipped a 4 GiB default for local_buffer_size; the dual-mode
    patch preserves it (and the JSON key) so unchanged PR-40900 configs work."""
    assert worker.DEFAULT_LOCAL_BUFFER_SIZE == 4 * 1024**3
```
**EN:** Test case covering `default local buffer size matches pr40900`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `default local buffer size matches pr40900` 的测试用例。 代码主体包含 1 个显式断言。

### test_get_requester_local_hostname_prefers_override (lines 196-199)
```python
def test_get_requester_local_hostname_prefers_override(monkeypatch):
    monkeypatch.setenv("MOONCAKE_REQUESTER_LOCAL_HOSTNAME", "worker-a:50053")

    assert rdma_utils.get_requester_local_hostname("10.0.0.7") == "worker-a:50053"
```
**EN:** Test case covering `get requester local hostname prefers override`. Inputs/fixtures: `monkeypatch`. It exercises `monkeypatch.setenv, rdma_utils.get_requester_local_hostname`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get requester local hostname prefers override` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `monkeypatch.setenv, rdma_utils.get_requester_local_hostname`。 代码主体包含 1 个显式断言。

### test_get_configured_preferred_segment_returns_explicit_override (lines 202-208)
```python
def test_get_configured_preferred_segment_returns_explicit_override():
    assert (
        rdma_utils.get_configured_preferred_segment(
            {"preferred_segment": "10.0.0.7:50053"}
        )
        == "10.0.0.7:50053"
    )
```
**EN:** Test case covering `get configured preferred segment returns explicit override`. It exercises `rdma_utils.get_configured_preferred_segment`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get configured preferred segment returns explicit override` 的测试用例。 该测试会调用 `rdma_utils.get_configured_preferred_segment`。 代码主体包含 1 个显式断言。

### test_get_configured_preferred_segment_prefers_explicit_over_env (lines 211-219)
```python
def test_get_configured_preferred_segment_prefers_explicit_over_env(monkeypatch):
    monkeypatch.setenv("MOONCAKE_PREFERRED_SEGMENT", "10.0.0.8:50053")

    assert (
        rdma_utils.get_configured_preferred_segment(
            {"preferred_segment": "10.0.0.7:50053"}
        )
        == "10.0.0.7:50053"
    )
```
**EN:** Test case covering `get configured preferred segment prefers explicit over env`. Inputs/fixtures: `monkeypatch`. It exercises `monkeypatch.setenv, rdma_utils.get_configured_preferred_segment`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get configured preferred segment prefers explicit over env` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `monkeypatch.setenv, rdma_utils.get_configured_preferred_segment`。 代码主体包含 1 个显式断言。

### test_get_configured_preferred_segment_returns_env_override (lines 222-225)
```python
def test_get_configured_preferred_segment_returns_env_override(monkeypatch):
    monkeypatch.setenv("MOONCAKE_PREFERRED_SEGMENT", "10.0.0.8:50053")

    assert rdma_utils.get_configured_preferred_segment({}) == "10.0.0.8:50053"
```
**EN:** Test case covering `get configured preferred segment returns env override`. Inputs/fixtures: `monkeypatch`. It exercises `monkeypatch.setenv, rdma_utils.get_configured_preferred_segment`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get configured preferred segment returns env override` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `monkeypatch.setenv, rdma_utils.get_configured_preferred_segment`。 代码主体包含 1 个显式断言。

### test_get_configured_preferred_segment_rejects_empty_override (lines 228-230)
```python
def test_get_configured_preferred_segment_rejects_empty_override():
    with pytest.raises(ValueError, match="preferred_segment"):
        rdma_utils.get_configured_preferred_segment({"preferred_segment": "  "})
```
**EN:** Test case covering `get configured preferred segment rejects empty override`. It exercises `pytest.raises, rdma_utils.get_configured_preferred_segment`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `get configured preferred segment rejects empty override` 的测试用例。 该测试会调用 `pytest.raises, rdma_utils.get_configured_preferred_segment`。 主要通过预期异常检查来完成验证。

### test_get_configured_worker_rnic_prefers_explicit_device_name (lines 233-248)
```python
def test_get_configured_worker_rnic_prefers_explicit_device_name(monkeypatch):
    store_config = worker.MooncakeStoreConfig(
        metadata_server="",
        local_buffer_size=1,
        protocol="rdma",
        device_name="rocep139s0",
        master_server_address="",
    )

    assert (
        rdma_utils.get_configured_worker_rnic(
            protocol=store_config.protocol,
            configured_device=store_config.device_name,
        )
        == "rocep139s0"
    )
```
**EN:** Test case covering `get configured worker rnic prefers explicit device name`. Inputs/fixtures: `monkeypatch`. It exercises `worker.MooncakeStoreConfig, rdma_utils.get_configured_worker_rnic`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get configured worker rnic prefers explicit device name` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `worker.MooncakeStoreConfig, rdma_utils.get_configured_worker_rnic`。 代码主体包含 1 个显式断言。

### test_get_configured_worker_rnic_selects_device_from_explicit_csv (lines 251-271)
```python
def test_get_configured_worker_rnic_selects_device_from_explicit_csv(monkeypatch):
    monkeypatch.setattr(
        rdma_utils,
        "get_current_physical_gpu_index",
        lambda: 1,
    )
    store_config = worker.MooncakeStoreConfig(
        metadata_server="",
        local_buffer_size=1,
        protocol="rdma",
        device_name="rocep139s0,rocep140s0",
        master_server_address="",
    )

    assert (
        rdma_utils.get_configured_worker_rnic(
            protocol=store_config.protocol,
            configured_device=store_config.device_name,
        )
        == "rocep140s0"
    )
```
**EN:** Test case covering `get configured worker rnic selects device from explicit csv`. Inputs/fixtures: `monkeypatch`. It exercises `monkeypatch.setattr, worker.MooncakeStoreConfig, rdma_utils.get_configured_worker_rnic`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get configured worker rnic selects device from explicit csv` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `monkeypatch.setattr, worker.MooncakeStoreConfig, rdma_utils.get_configured_worker_rnic`。 代码主体包含 1 个显式断言。

### test_get_configured_worker_rnic_warns_and_returns_empty_for_rdma_with_no_device (lines 274-289)
```python
def test_get_configured_worker_rnic_warns_and_returns_empty_for_rdma_with_no_device(
    caplog, monkeypatch
):
    """No device configured + protocol=rdma → emit a clear warning and return ""
    so the C++ side handles auto-selection. There is no Python-side fallback."""
    monkeypatch.setattr(logging.getLogger("vllm"), "propagate", True)
    with caplog.at_level(logging.WARNING):
        result = rdma_utils.get_configured_worker_rnic(
            protocol="rdma",
            configured_device="",
        )
    assert result == ""
    warnings = [r for r in caplog.records if r.levelno == logging.WARNING]
    assert any("No RDMA devices specified" in r.message for r in warnings), (
        f"expected fallback warning, got {[r.message for r in warnings]}"
    )
```
**EN:** Test case covering `get configured worker rnic warns and returns empty for rdma with no device`. Inputs/fixtures: `caplog, monkeypatch`. It exercises `monkeypatch.setattr, any, logging.getLogger, caplog.at_level, rdma_utils.get_configured_worker_rnic`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `get configured worker rnic warns and returns empty for rdma with no device` 的测试用例。 输入或 fixture：`caplog, monkeypatch`。 该测试会调用 `monkeypatch.setattr, any, logging.getLogger, caplog.at_level, rdma_utils.get_configured_worker_rnic`。 代码主体包含 2 个显式断言。

### test_get_configured_worker_rnic_silent_for_tcp_with_no_device (lines 292-305)
```python
def test_get_configured_worker_rnic_silent_for_tcp_with_no_device(caplog, monkeypatch):
    """protocol=tcp + no device → return "" silently (no RDMA, no warning)."""
    monkeypatch.setattr(logging.getLogger("vllm"), "propagate", True)
    with caplog.at_level(logging.WARNING):
        result = rdma_utils.get_configured_worker_rnic(
            protocol="tcp",
            configured_device="",
        )
    assert result == ""
    warnings = [r for r in caplog.records if r.levelno == logging.WARNING]
    assert not any("RDMA" in r.message for r in warnings), (
        "did not expect RDMA warning for tcp protocol, got "
        f"{[r.message for r in warnings]}"
    )
```
**EN:** Test case covering `get configured worker rnic silent for tcp with no device`. Inputs/fixtures: `caplog, monkeypatch`. It exercises `monkeypatch.setattr, logging.getLogger, caplog.at_level, rdma_utils.get_configured_worker_rnic, any`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `get configured worker rnic silent for tcp with no device` 的测试用例。 输入或 fixture：`caplog, monkeypatch`。 该测试会调用 `monkeypatch.setattr, logging.getLogger, caplog.at_level, rdma_utils.get_configured_worker_rnic, any`。 代码主体包含 2 个显式断言。

### test_get_configured_worker_rnic_rejects_short_explicit_csv (lines 308-318)
```python
def test_get_configured_worker_rnic_rejects_short_explicit_csv(monkeypatch):
    monkeypatch.setattr(
        rdma_utils,
        "get_current_physical_gpu_index",
        lambda: 2,
    )
    with pytest.raises(ValueError, match="does not cover local GPU 2"):
        rdma_utils.get_configured_worker_rnic(
            protocol="rdma",
            configured_device="rocep139s0,rocep140s0",
        )
```
**EN:** Test case covering `get configured worker rnic rejects short explicit csv`. Inputs/fixtures: `monkeypatch`. It exercises `monkeypatch.setattr, pytest.raises, rdma_utils.get_configured_worker_rnic`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `get configured worker rnic rejects short explicit csv` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `monkeypatch.setattr, pytest.raises, rdma_utils.get_configured_worker_rnic`。 主要通过预期异常检查来完成验证。

### _ReplicaDesc (lines 321-332)
```python
class _ReplicaDesc:
    def __init__(self, tier: str):
        self.tier = tier

    def is_memory_replica(self) -> bool:
        return self.tier == "memory"

    def is_disk_replica(self) -> bool:
        return self.tier == "disk"

    def is_local_disk_replica(self) -> bool:
        return self.tier == "disk"
```
**EN:** Class `_ReplicaDesc` groups 0 test method(s) and 4 helper/fixture method(s).
**CN:** 类 `_ReplicaDesc` 组织了 0 个测试方法，以及 4 个辅助或 fixture 方法。

### test_store_sending_thread_skips_request_during_cpu_pressure (lines 335-367)
```python
def test_store_sending_thread_skips_request_during_cpu_pressure():
    store = MagicMock()
    store.batch_is_exist.side_effect = lambda keys: [0] * len(keys)
    store.batch_put_from_multi_buffers.side_effect = [
        [-200, -200],
        [256, 256],
    ]
    thread = _make_store_sending_thread(store)

    thread.add_stored_request("req-a")
    thread._handle_request(_make_store_req("req-a", [b"a0", b"a1"]))
    assert thread._store_pressure_active is True
    assert "req-a" in thread._skip_store_requests
    assert store.batch_put_from_multi_buffers.call_count == 1
    # ... excerpt omitted for brevity ...
    assert thread._store_pressure_active is False
    assert "req-a" not in thread._skip_store_requests
    assert store.batch_put_from_multi_buffers.call_count == 2
    thread._handle_request(_make_store_req("req-a", [b"a4", b"a5"]))
    assert store.batch_put_from_multi_buffers.call_count == 3
```
**EN:** Test case covering `store sending thread skips request during CPU pressure`. It exercises `MagicMock, _make_store_sending_thread, thread.add_stored_request, thread._handle_request, _make_store_req, len`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `store sending thread skips request during cpu pressure` 的测试用例。 该测试会调用 `MagicMock, _make_store_sending_thread, thread.add_stored_request, thread._handle_request, _make_store_req, len`。 代码主体包含 8 个显式断言。

### test_store_sending_thread_only_skips_on_no_available_handle (lines 370-389)
```python
def test_store_sending_thread_only_skips_on_no_available_handle():
    store = MagicMock()
    store.batch_is_exist.side_effect = lambda keys: [0] * len(keys)
    store.batch_put_from_multi_buffers.side_effect = [
        [-500, -500],
        [256, 256],
    ]
    thread = _make_store_sending_thread(store)

    thread.add_stored_request("req-a")
    thread._handle_request(_make_store_req("req-a", [b"a0", b"a1"]))

    assert thread._store_pressure_active is False
    assert "req-a" not in thread._skip_store_requests
    assert store.batch_put_from_multi_buffers.call_count == 1

    thread.add_stored_request("req-a")
    thread._handle_request(_make_store_req("req-a", [b"a2", b"a3"]))

    assert store.batch_put_from_multi_buffers.call_count == 2
```
**EN:** Test case covering `store sending thread only skips on no available handle`. It exercises `MagicMock, _make_store_sending_thread, thread.add_stored_request, thread._handle_request, _make_store_req, len`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `store sending thread only skips on no available handle` 的测试用例。 该测试会调用 `MagicMock, _make_store_sending_thread, thread.add_stored_request, thread._handle_request, _make_store_req, len`。 代码主体包含 4 个显式断言。

### test_store_sending_thread_passes_replicate_config_when_preferred_segment_set (lines 392-405)
```python
def test_store_sending_thread_passes_replicate_config_when_preferred_segment_set():
    store = MagicMock()
    store.batch_is_exist.side_effect = lambda keys: [0] * len(keys)
    store.batch_put_from_multi_buffers.return_value = [256, 256]
    replicate_config = SimpleNamespace(preferred_segment="10.0.0.7:50053")
    thread = _make_store_sending_thread(store, replicate_config=replicate_config)

    thread.add_stored_request("req-a")
    thread._handle_request(_make_store_req("req-a", [b"a0", b"a1"]))

    assert store.batch_put_from_multi_buffers.call_count == 1
    call_args = store.batch_put_from_multi_buffers.call_args.args
    assert len(call_args) == 4
    assert call_args[3] is replicate_config
```
**EN:** Test case covering `store sending thread passes replicate config when preferred segment set`. It exercises `MagicMock, SimpleNamespace, _make_store_sending_thread, thread.add_stored_request, thread._handle_request, _make_store_req`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `store sending thread passes replicate config when preferred segment set` 的测试用例。 该测试会调用 `MagicMock, SimpleNamespace, _make_store_sending_thread, thread.add_stored_request, thread._handle_request, _make_store_req`。 代码主体包含 3 个显式断言。

### test_store_sending_thread_passes_default_replicate_config_when_no_preferred_segment (lines 408-424)
```python
def test_store_sending_thread_passes_default_replicate_config_when_no_preferred_segment():  # noqa: E501
    """Without a preferred_segment the SendingThread still forwards a
    (default-constructed) ReplicateConfig so the C++ side always sees a
    well-defined config object."""
    store = MagicMock()
    store.batch_is_exist.side_effect = lambda keys: [0] * len(keys)
    store.batch_put_from_multi_buffers.return_value = [256, 256]
    replicate_config = SimpleNamespace()
    thread = _make_store_sending_thread(store, replicate_config=replicate_config)

    thread.add_stored_request("req-a")
    thread._handle_request(_make_store_req("req-a", [b"a0", b"a1"]))

    assert store.batch_put_from_multi_buffers.call_count == 1
    call_args = store.batch_put_from_multi_buffers.call_args.args
    assert len(call_args) == 4
    assert call_args[3] is replicate_config
```
**EN:** Test case covering `store sending thread passes default replicate config when no preferred segment`. It exercises `MagicMock, SimpleNamespace, _make_store_sending_thread, thread.add_stored_request, thread._handle_request, _make_store_req`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `store sending thread passes default replicate config when no preferred segment` 的测试用例。 该测试会调用 `MagicMock, SimpleNamespace, _make_store_sending_thread, thread.add_stored_request, thread._handle_request, _make_store_req`。 代码主体包含 3 个显式断言。

### test_estimate_disk_offload_staging_bytes_sums_multi_segment_sizes (lines 427-428)
```python
def test_estimate_disk_offload_staging_bytes_sums_multi_segment_sizes():
    assert worker._estimate_disk_offload_staging_bytes([256, 512]) == 12288
```
**EN:** Test case covering `estimate disk offload staging bytes sums multi segment sizes`. It exercises `worker._estimate_disk_offload_staging_bytes`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `estimate disk offload staging bytes sums multi segment sizes` 的测试用例。 该测试会调用 `worker._estimate_disk_offload_staging_bytes`。 代码主体包含 1 个显式断言。

### test_recv_thread_uses_single_batch_when_no_disk_offload_budget (lines 431-453)
```python
def test_recv_thread_uses_single_batch_when_no_disk_offload_budget(monkeypatch):
    monkeypatch.delenv("VLLM_MOONCAKE_STORE_TIER_LOG", raising=False)
    store = MagicMock()
    store.batch_get_into_multi_buffers.return_value = [256, 256, 256]
    thread = _make_store_recving_thread(store, disk_offload_buffer_budget_bytes=None)

    req = _make_load_req(
        "req-a",
        [b"a0", b"a1", b"a2"],
        token_len=48,
    )

    thread._handle_request(req)

    assert store.batch_get_into_multi_buffers.call_count == 1
    keys, addrs, sizes = store.batch_get_into_multi_buffers.call_args.args
    assert keys == [
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6130",
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6131",
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6132",
    ]
    assert sizes == [[256], [256], [256]]
    store.batch_get_replica_desc.assert_not_called()
```
**EN:** Test case covering `recv thread uses single batch when no disk offload budget`. Inputs/fixtures: `monkeypatch`. It exercises `monkeypatch.delenv, MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request, batch_get_replica_desc.assert_not_called`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `recv thread uses single 批处理 when no disk offload budget` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `monkeypatch.delenv, MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request, batch_get_replica_desc.assert_not_called`。 代码主体包含 3 个显式断言。

### test_recv_thread_logs_tier_summary_when_enabled (lines 456-498)
```python
def test_recv_thread_logs_tier_summary_when_enabled(monkeypatch, caplog_vllm):
    monkeypatch.setenv("VLLM_MOONCAKE_STORE_TIER_LOG", "1")
    caplog_vllm.set_level(logging.INFO, logger=worker.logger.name)

    store = MagicMock()
    store.batch_get_into_multi_buffers.return_value = [256, 256, -10]
    thread = _make_store_recving_thread(store, disk_offload_buffer_budget_bytes=None)
    req = _make_load_req(
        "req-a",
        [b"a0", b"a1", b"a2"],
        token_len=48,
    )
    expected_keys = [
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6130",
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6131",
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6132",
    ]
    # ... excerpt omitted for brevity ...
        expected_keys[0]: [_ReplicaDesc("memory")],
        expected_keys[1]: [_ReplicaDesc("disk")],
        expected_keys[2]: [],
    assert store.batch_get_replica_desc.call_args.args == (expected_keys,)
    assert store.method_calls[0][0] == "batch_get_replica_desc"
    assert store.method_calls[1][0] == "batch_get_into_multi_buffers"
        and "unknown_keys=1" in message
        and "success_keys=2" in message
        and "failed_keys=1" in message
        and "bytes_by_tier={'memory': 256, 'disk': 256, 'unknown': 0}" in message
        for message in messages
```
**EN:** Test case covering `recv thread logs tier summary when enabled`. Inputs/fixtures: `monkeypatch, caplog_vllm`. It exercises `monkeypatch.setenv, caplog_vllm.set_level, MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `recv thread logs tier summary when enabled` 的测试用例。 输入或 fixture：`monkeypatch, caplog_vllm`。 该测试会调用 `monkeypatch.setenv, caplog_vllm.set_level, MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`。 代码主体包含 4 个显式断言。

### test_recv_thread_uses_ratio_scaled_budget_for_first_pass_split (lines 501-528)
```python
def test_recv_thread_uses_ratio_scaled_budget_for_first_pass_split():
    store = MagicMock()
    store.batch_get_into_multi_buffers.side_effect = [
        [256],
        [256],
    ]
    thread = _make_store_recving_thread(
        store,
        disk_offload_buffer_budget_bytes=2 * _DISK_OFFLOAD_SINGLE_KEY_BYTES,
    )

    req = _make_load_req(
        "req-a",
        [b"a0", b"a1"],
        token_len=32,
    )

    thread._handle_request(req)

    assert store.batch_get_into_multi_buffers.call_count == 2
    first_keys = store.batch_get_into_multi_buffers.call_args_list[0].args[0]
    second_keys = store.batch_get_into_multi_buffers.call_args_list[1].args[0]
    assert first_keys == [
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6130",
    ]
    assert second_keys == [
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6131",
    ]
```
**EN:** Test case covering `recv thread uses ratio scaled budget for first pass split`. It exercises `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `recv thread uses ratio scaled budget for first pass split` 的测试用例。 该测试会调用 `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`。 代码主体包含 3 个显式断言。

### test_recv_thread_splits_disk_offload_loads_by_budget (lines 531-571)
```python
def test_recv_thread_splits_disk_offload_loads_by_budget():
    store = MagicMock()
    store.batch_get_into_multi_buffers.side_effect = [
        [256, 256],
        [256],
    ]
    thread = _make_store_recving_thread(
        store,
        disk_offload_buffer_budget_bytes=_DISK_OFFLOAD_BUDGET_FOR_SPLIT,
    )

    req = _make_load_req(
        "req-a",
        [b"a0", b"a1", b"a2"],
        token_len=48,
    thread._handle_request(req)
    # ... excerpt omitted for brevity ...
    assert store.batch_get_into_multi_buffers.call_count == 2
    assert first_keys == [
    assert second_keys == [
    block_len = thread.token_database.block_len[0]
    assert first_addrs == [[base_addr], [base_addr + block_len]]
    assert second_addrs == [[base_addr + 2 * block_len]]
    expected_size = block_len
    assert first_sizes == [[expected_size], [expected_size]]
    assert second_sizes == [[expected_size]]
```
**EN:** Test case covering `recv thread splits disk offload loads by budget`. It exercises `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `recv thread splits disk offload loads by budget` 的测试用例。 该测试会调用 `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`。 代码主体包含 7 个显式断言。

### test_recv_thread_stops_after_first_failing_disk_offload_sub_batch (lines 574-590)
```python
def test_recv_thread_stops_after_first_failing_disk_offload_sub_batch():
    store = MagicMock()
    store.batch_get_into_multi_buffers.return_value = [-10, -10]
    thread = _make_store_recving_thread(
        store,
        disk_offload_buffer_budget_bytes=_DISK_OFFLOAD_BUDGET_FOR_SPLIT,
    )

    req = _make_load_req(
        "req-a",
        [b"a0", b"a1", b"a2"],
        token_len=48,
    )

    thread._handle_request(req)

    assert store.batch_get_into_multi_buffers.call_count == 1
```
**EN:** Test case covering `recv thread stops after first failing disk offload sub batch`. It exercises `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `recv thread stops after first failing disk offload sub 批处理` 的测试用例。 该测试会调用 `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`。 代码主体包含 1 个显式断言。

### test_recv_thread_skips_split_when_budget_holds_all_keys (lines 593-616)
```python
def test_recv_thread_skips_split_when_budget_holds_all_keys():
    """PR-36 removed the count-based split trigger; with budget for 3 keys,
    all three should be requested in a single call."""
    store = MagicMock()
    store.batch_get_into_multi_buffers.return_value = [256, 256, 256]
    thread = _make_store_recving_thread(
        store,
        disk_offload_buffer_budget_bytes=_DISK_OFFLOAD_BUDGET_FOR_THREE_KEYS,
    )

    req = _make_load_req(
        "req-a",
        [b"a0", b"a1", b"a2"],
        token_len=48,
    )

    thread._handle_request(req)

    assert store.batch_get_into_multi_buffers.call_count == 1
    assert store.batch_get_into_multi_buffers.call_args_list[0].args[0] == [
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6130",
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6131",
        "test-model@tp_rank:0@pcp0@dcp0@pp_rank:0@6132",
    ]
```
**EN:** Test case covering `recv thread skips split when budget holds all keys`. It exercises `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `recv thread skips split when budget holds all keys` 的测试用例。 该测试会调用 `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`。 代码主体包含 2 个显式断言。

### test_recv_thread_reports_unsplittable_key_larger_than_budget (lines 619-634)
```python
def test_recv_thread_reports_unsplittable_key_larger_than_budget():
    store = MagicMock()
    thread = _make_store_recving_thread(
        store,
        disk_offload_buffer_budget_bytes=_DISK_OFFLOAD_BUDGET_TOO_SMALL,
    )

    req = _make_load_req(
        "req-a",
        [b"a0"],
        token_len=16,
    )

    thread._handle_request(req)

    assert store.batch_get_into_multi_buffers.call_count == 0
```
**EN:** Test case covering `recv thread reports unsplittable key larger than budget`. It exercises `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `recv thread reports unsplittable key larger than budget` 的测试用例。 该测试会调用 `MagicMock, _make_store_recving_thread, _make_load_req, thread._handle_request`。 代码主体包含 1 个显式断言。

### test_requester_worker_init_uses_positional_setup (lines 637-668)
```python
def test_requester_worker_init_uses_positional_setup(tmp_path, monkeypatch):
    store = MagicMock()
    store.setup.return_value = 0
    _install_fake_mooncake(monkeypatch, store)
    _patch_worker_runtime(monkeypatch)
    monkeypatch.setenv(
        "MOONCAKE_CONFIG_PATH",
        _write_mooncake_config(
            tmp_path,
            {
                "metadata_server": "http://metadata/endpoint",
                "global_segment_size": "4gb",
                "local_buffer_size": "64mb",
                "protocol": "rdma",
                "device_name": "mlx5_0",
                "master_server_address": "10.0.0.7:50051",
                "enable_offload": True,
            },
        ),
    )
    w = worker.MooncakeStoreWorker(_make_vllm_config())

    assert not hasattr(w, "_isolate_offload_resources")
    assert store.setup.call_args.args == (
        "10.0.0.7",
        "http://metadata/endpoint",
        4 * 1024 * 1024 * 1024,  # global_segment_size: "4gb" honored
        64 * 1024 * 1024,
        "rdma",
        "mlx5_0",
        "10.0.0.7:50051",
    )
```
**EN:** Test case covering `requester worker init uses positional setup`. Inputs/fixtures: `tmp_path, monkeypatch`. It exercises `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, _write_mooncake_config`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `requester worker init uses positional setup` 的测试用例。 输入或 fixture：`tmp_path, monkeypatch`。 该测试会调用 `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, _write_mooncake_config`。 代码主体包含 2 个显式断言。

### test_requester_worker_init_prefers_local_hostname_override (lines 671-695)
```python
def test_requester_worker_init_prefers_local_hostname_override(
    tmp_path,
    monkeypatch,
):
    store = MagicMock()
    store.setup.return_value = 0
    _install_fake_mooncake(monkeypatch, store)
    _patch_worker_runtime(monkeypatch)
    monkeypatch.setenv("MOONCAKE_REQUESTER_LOCAL_HOSTNAME", "worker-a:50053")
    monkeypatch.setenv(
        "MOONCAKE_CONFIG_PATH",
        _write_mooncake_config(
            tmp_path,
            {
                "metadata_server": "http://metadata/endpoint",
                "local_buffer_size": "64mb",
                "protocol": "tcp",
                "device_name": "",
                "master_server_address": "10.0.0.7:50051",
            },
        ),
    )
    worker.MooncakeStoreWorker(_make_vllm_config())

    assert store.setup.call_args.args[0] == "worker-a:50053"
```
**EN:** Test case covering `requester worker init prefers local hostname override`. Inputs/fixtures: `tmp_path, monkeypatch`. It exercises `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, _write_mooncake_config`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `requester worker init prefers local hostname override` 的测试用例。 输入或 fixture：`tmp_path, monkeypatch`。 该测试会调用 `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, _write_mooncake_config`。 代码主体包含 1 个显式断言。

### test_requester_worker_init_skips_disk_budget_when_offload_disabled (lines 698-723)
```python
def test_requester_worker_init_skips_disk_budget_when_offload_disabled(
    tmp_path,
    monkeypatch,
):
    """enable_offload=False zeroes out the disk budget so we don't generate
    redundant owner GET-RPCs."""
    store = MagicMock()
    store.setup.return_value = 0
    _install_fake_mooncake(monkeypatch, store)
    _patch_worker_runtime(monkeypatch)
    monkeypatch.setenv(
        "MOONCAKE_CONFIG_PATH",
        _write_mooncake_config(
            tmp_path,
            {
                "metadata_server": "http://metadata/endpoint",
                "protocol": "tcp",
                "device_name": "",
                "master_server_address": "10.0.0.7:50051",
                "enable_offload": False,
            },
        ),
    )
    w = worker.MooncakeStoreWorker(_make_vllm_config())

    assert w.disk_offload_buffer_budget_bytes is None
```
**EN:** Test case covering `requester worker init skips disk budget when offload disabled`. Inputs/fixtures: `tmp_path, monkeypatch`. It exercises `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, _write_mooncake_config`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `requester worker init skips disk budget when offload disabled` 的测试用例。 输入或 fixture：`tmp_path, monkeypatch`。 该测试会调用 `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, _write_mooncake_config`。 代码主体包含 1 个显式断言。

### test_requester_worker_init_builds_replicate_config_for_preferred_segment (lines 726-755)
```python
def test_requester_worker_init_builds_replicate_config_for_preferred_segment(
    tmp_path,
    monkeypatch,
):
    store = MagicMock()
    store.setup.return_value = 0
    fake_replicate_config_cls = _install_fake_mooncake(monkeypatch, store)
    _patch_worker_runtime(monkeypatch)
    monkeypatch.setenv(
        "MOONCAKE_CONFIG_PATH",
        _write_mooncake_config(
            tmp_path,
            {
                "metadata_server": "http://metadata/endpoint",
                "protocol": "tcp",
                "device_name": "",
                "master_server_address": "10.0.0.7:50051",
            },
        ),
    )
    w = worker.MooncakeStoreWorker(
        _make_vllm_config(
            extra_config={
                "preferred_segment": "10.0.0.7:50053",
            }
        )
    )

    assert isinstance(w.store_replicate_config, fake_replicate_config_cls)
    assert w.store_replicate_config.preferred_segment == "10.0.0.7:50053"
```
**EN:** Test case covering `requester worker init builds replicate config for preferred segment`. Inputs/fixtures: `tmp_path, monkeypatch`. It exercises `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, isinstance`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `requester worker init builds replicate config for preferred segment` 的测试用例。 输入或 fixture：`tmp_path, monkeypatch`。 该测试会调用 `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, isinstance`。 代码主体包含 2 个显式断言。

### _auto_set_ready_event (lines 763-771)
```python
def _auto_set_ready_event(*args, **kwargs):
    """Side effect for mocked thread constructors that auto-sets ready_event."""
    for arg in args:
        if isinstance(arg, threading.Event):
            arg.set()
    for val in kwargs.values():
        if isinstance(val, threading.Event):
            val.set()
    return MagicMock()
```
**EN:** Helper function `_auto_set_ready_event` encapsulates reusable logic for `auto set ready event`. Key calls include `kwargs.values, MagicMock, isinstance, arg.set, val.set`.
**CN:** 辅助函数 `_auto_set_ready_event` 封装了与 `auto set ready event` 相关的可复用逻辑。 关键调用包括 `kwargs.values, MagicMock, isinstance, arg.set, val.set`。

### _make_bare_worker (lines 774-804)
```python
def _make_bare_worker(
    *,
    num_gpu_blocks: int = 10,
    block_size: int = 16,
    kv_role: str = "kv_both",
) -> worker.MooncakeStoreWorker:
    """Construct a MooncakeStoreWorker via __new__, bypassing __init__.

    Sets only the attributes that register_kv_caches() reads so we can
    test the stride-based layout detection without a real
    MooncakeDistributedStore.
    """
    w = object.__new__(worker.MooncakeStoreWorker)
    w.cache_config = MagicMock()
    w.cache_config.num_gpu_blocks = num_gpu_blocks
    w.store = MagicMock()
    w.store.register_buffer.return_value = 0
    w.use_mla = False
    w.token_database = ChunkedTokenDatabase(
        KeyMetadata("test-model", 0, 0, 0, 0), block_size=block_size
    )
    w.kv_role = kv_role
    w.block_size = block_size
    w.tp_rank = 0
    w.put_step = 1
    w.enable_kv_events = False
    w.disk_offload_buffer_budget_bytes = None
    w.kv_send_thread = None
    w.kv_recv_thread = None
    w.store_replicate_config = SimpleNamespace()
    return w
```
**EN:** Helper function `_make_bare_worker` encapsulates reusable logic for `bare worker`. Key calls include `object.__new__, MagicMock, ChunkedTokenDatabase, SimpleNamespace, KeyMetadata`.
**CN:** 辅助函数 `_make_bare_worker` 封装了与 `bare worker` 相关的可复用逻辑。 关键调用包括 `object.__new__, MagicMock, ChunkedTokenDatabase, SimpleNamespace, KeyMetadata`。

### test_register_kv_caches_blocks_first_single_segment (lines 812-845)
```python
def test_register_kv_caches_blocks_first_single_segment():
    """Blocks-first layout (FlashInfer/MLA): one segment per layer."""
    num_blocks = 10
    page_size_elements = 64  # elements per block
    w = _make_bare_worker(num_gpu_blocks=num_blocks)

    # Shape: (num_blocks, page_size_elements) — blocks outermost, no outer_dims
    tensor = torch.zeros(num_blocks, page_size_elements, dtype=torch.float16)
    with (
        patch(
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store."
            "worker.KVCacheStoreSendingThread",
            side_effect=_auto_set_ready_event,
        ),
            "worker.KVCacheStoreRecvingThread",
    # ... excerpt omitted for brevity ...
    assert len(w.kv_caches_base_addr) == 1
    assert w.kv_caches_base_addr[0] == tensor.untyped_storage().data_ptr()
    expected_block_len = tensor.untyped_storage().nbytes() // num_blocks
    assert len(w.block_len) == 1
    assert w.block_len[0] == expected_block_len
    w.store.register_buffer.assert_called_once_with(
        tensor.untyped_storage().data_ptr(),
        tensor.untyped_storage().nbytes(),
    )
```
**EN:** Test case covering `register KV caches blocks first single segment`. It exercises `_make_bare_worker, torch.zeros, register_buffer.assert_called_once_with, patch, w.register_kv_caches, len`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `register kv caches blocks first single segment` 的测试用例。 该测试会调用 `_make_bare_worker, torch.zeros, register_buffer.assert_called_once_with, patch, w.register_kv_caches, len`。 代码主体包含 4 个显式断言。

### test_register_kv_caches_kv_first_two_segments (lines 848-891)
```python
def test_register_kv_caches_kv_first_two_segments():
    """K/V-first layout (FlashAttn): two segments (K, V) per layer."""
    num_blocks = 10
    block_size_tokens = 16
    num_kv_heads = 4
    head_size = 8

    w = _make_bare_worker(num_gpu_blocks=num_blocks)
    # Shape: (2, num_blocks, block_size, num_kv_heads, head_size) — K/V outermost
    tensor = torch.zeros(
        2,
        num_blocks,
        block_size_tokens,
        num_kv_heads,
        head_size,
        dtype=torch.float16,
    )
    # ... excerpt omitted for brevity ...
    assert len(w.kv_caches_base_addr) == 2
    assert len(w.block_len) == 2
    seg_stride = tensor.stride(0) * el  # stride of the K/V dim in bytes
    base = tensor.untyped_storage().data_ptr()
    assert w.kv_caches_base_addr[0] == base
    assert w.kv_caches_base_addr[1] == base + seg_stride
    assert w.block_len[0] == seg_stride // num_blocks
    assert w.block_len[1] == seg_stride // num_blocks
```
**EN:** Test case covering `register KV caches KV first two segments`. It exercises `_make_bare_worker, torch.zeros, tensor.element_size, untyped_storage.data_ptr, patch, w.register_kv_caches`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `register kv caches kv first two segments` 的测试用例。 该测试会调用 `_make_bare_worker, torch.zeros, tensor.element_size, untyped_storage.data_ptr, patch, w.register_kv_caches`。 代码主体包含 6 个显式断言。

### test_register_kv_caches_cross_layer_single_segment (lines 894-952)
```python
def test_register_kv_caches_cross_layer_single_segment():
    """Cross-layer tensor: single segment with block_len = page_size * num_layers."""
    num_blocks = 10
    num_layers = 4
    per_layer_page_elements = 64  # elements per layer per block

    w = _make_bare_worker(num_gpu_blocks=num_blocks)
    # Cross-layer blocks-first tensor: all layers packed into a single
    # contiguous block.  Shape (num_blocks, num_layers * per_layer_page)
    # mimics the physical layout after stride reordering.
    total_page_elements = num_layers * per_layer_page_elements
    tensor = torch.zeros(num_blocks, total_page_elements, dtype=torch.float16)
    with (
        patch(
            "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store."
            "worker.KVCacheStoreSendingThread",
    # ... excerpt omitted for brevity ...
    assert len(w.kv_caches_base_addr) == 1
    assert w.kv_caches_base_addr[0] == tensor.untyped_storage().data_ptr()
    expected_block_len = tensor.untyped_storage().nbytes() // num_blocks
    assert (
        expected_block_len
    assert len(w.block_len) == 1
        ),
    ):
        w2.register_cross_layers_kv_caches(tensor)
    assert w2.kv_caches_base_addr == w.kv_caches_base_addr
    assert w2.block_len == w.block_len
```
**EN:** Test case covering `register KV caches cross layer single segment`. It exercises `_make_bare_worker, torch.zeros, patch, w.register_kv_caches, len, untyped_storage.data_ptr`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `register kv caches cross layer single segment` 的测试用例。 该测试会调用 `_make_bare_worker, torch.zeros, patch, w.register_kv_caches, len, untyped_storage.data_ptr`。 代码主体包含 7 个显式断言。

### _make_config (lines 960-973)
```python
def _make_config(**overrides):
    """Build a MooncakeStoreConfig with sensible defaults for validation tests.

    Required dataclass fields are populated; callers override only the field
    under test.
    """
    base = dict(
        metadata_server="http://metadata/endpoint",
        master_server_address="10.0.0.7:50051",
        protocol="rdma",
        device_name="mlx5_0",
    )
    base.update(overrides)
    return worker.MooncakeStoreConfig(**base)
```
**EN:** Helper function `_make_config` encapsulates reusable logic for `config`. Key calls include `dict, base.update, worker.MooncakeStoreConfig`.
**CN:** 辅助函数 `_make_config` 封装了与 `config` 相关的可复用逻辑。 关键调用包括 `dict, base.update, worker.MooncakeStoreConfig`。

### test_config_defaults_to_embedded (lines 976-982)
```python
def test_config_defaults_to_embedded():
    """A JSON without explicit mode parses as embedded with 4 GiB segment."""
    cfg = _make_config()
    assert cfg.mode == "embedded"
    assert cfg.global_segment_size == worker.DEFAULT_GLOBAL_SEGMENT_SIZE
    assert cfg.local_buffer_size == worker.DEFAULT_LOCAL_BUFFER_SIZE
    assert cfg.enable_offload is False
```
**EN:** Test case covering `config defaults to embedded`. It exercises `_make_config`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `config defaults to embedded` 的测试用例。 该测试会调用 `_make_config`。 代码主体包含 4 个显式断言。

### test_config_pr40900_unchanged (lines 985-1003)
```python
def test_config_pr40900_unchanged(tmp_path):
    """A literal PR-40900 config (no mode, no enable_offload, no preferred_segment)
    parses without raising and resolves to embedded mode."""
    config_path = _write_mooncake_config(
        tmp_path,
        {
            "metadata_server": "http://metadata/endpoint",
            "global_segment_size": "4GB",
            "local_buffer_size": "4GB",
            "protocol": "rdma",
            "device_name": "mlx5_0",
            "master_server_address": "10.0.0.7:50051",
        },
    )
    cfg = worker.MooncakeStoreConfig.from_file(config_path)
    assert cfg.mode == "embedded"
    assert cfg.global_segment_size == 4 * 1024**3
    assert cfg.local_buffer_size == 4 * 1024**3
    assert cfg.enable_offload is False
```
**EN:** Test case covering `config pr40900 unchanged`. Inputs/fixtures: `tmp_path`. It exercises `_write_mooncake_config, MooncakeStoreConfig.from_file`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `config pr40900 unchanged` 的测试用例。 输入或 fixture：`tmp_path`。 该测试会调用 `_write_mooncake_config, MooncakeStoreConfig.from_file`。 代码主体包含 4 个显式断言。

### test_config_embedded_rejects_zero_segment (lines 1006-1010)
```python
def test_config_embedded_rejects_zero_segment():
    with pytest.raises(
        ValueError, match=r"embedded mode requires global_segment_size > 0"
    ):
        _make_config(mode="embedded", global_segment_size=0)
```
**EN:** Test case covering `config embedded rejects zero segment`. It exercises `pytest.raises, _make_config`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `config embedded rejects zero segment` 的测试用例。 该测试会调用 `pytest.raises, _make_config`。 主要通过预期异常检查来完成验证。

### test_config_standalone_store_rejects_nonzero_segment (lines 1013-1018)
```python
def test_config_standalone_store_rejects_nonzero_segment():
    with pytest.raises(
        ValueError,
        match=r"standalone-store mode requires global_segment_size == 0",
    ):
        _make_config(mode="standalone-store", global_segment_size=4 * 1024**3)
```
**EN:** Test case covering `config standalone store rejects nonzero segment`. It exercises `pytest.raises, _make_config`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `config standalone store rejects nonzero segment` 的测试用例。 该测试会调用 `pytest.raises, _make_config`。 主要通过预期异常检查来完成验证。

### test_config_standalone_store_accepts_zero_segment (lines 1021-1024)
```python
def test_config_standalone_store_accepts_zero_segment():
    cfg = _make_config(mode="standalone-store", global_segment_size=0)
    assert cfg.mode == "standalone-store"
    assert cfg.global_segment_size == 0
```
**EN:** Test case covering `config standalone store accepts zero segment`. It exercises `_make_config`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `config standalone store accepts zero segment` 的测试用例。 该测试会调用 `_make_config`。 代码主体包含 2 个显式断言。

### test_config_unknown_mode (lines 1027-1029)
```python
def test_config_unknown_mode():
    with pytest.raises(ValueError, match=r"unknown Mooncake mode"):
        _make_config(mode="something-else")
```
**EN:** Test case covering `config unknown mode`. It exercises `pytest.raises, _make_config`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `config unknown mode` 的测试用例。 该测试会调用 `pytest.raises, _make_config`。 主要通过预期异常检查来完成验证。

### test_config_zero_local_buffer (lines 1032-1034)
```python
def test_config_zero_local_buffer():
    with pytest.raises(ValueError, match=r"local_buffer_size must be > 0"):
        _make_config(local_buffer_size=0)
```
**EN:** Test case covering `config zero local buffer`. It exercises `pytest.raises, _make_config`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `config zero local buffer` 的测试用例。 该测试会调用 `pytest.raises, _make_config`。 主要通过预期异常检查来完成验证。

### test_topology_standalone_store_with_disk_offload (lines 1047-1091)
```python
def test_topology_standalone_store_with_disk_offload(tmp_path, monkeypatch):
    """standalone-store + disk: global_segment_size=0, enable_offload=True,
    preferred_segment set. Assert setup() positional args, ReplicateConfig
    wiring, and that the disk-offload buffer budget is allocated."""
    store = MagicMock()
    store.setup.return_value = 0
    fake_replicate_config_cls = _install_fake_mooncake(monkeypatch, store)
    _patch_worker_runtime(monkeypatch)
    monkeypatch.setenv(
        "MOONCAKE_CONFIG_PATH",
        _write_mooncake_config(
            tmp_path,
            {
                "mode": "standalone-store",
                "metadata_server": "http://metadata/endpoint",
                "global_segment_size": 0,
                "local_buffer_size": "1GB",
                "protocol": "rdma",
    # ... excerpt omitted for brevity ...
    assert store.setup.call_args.args == (
    # ReplicateConfig is built and carries the preferred_segment.
    assert isinstance(w.store_replicate_config, fake_replicate_config_cls)
    assert w.store_replicate_config.preferred_segment == "10.0.0.7:50053"
    # Disk-offload staging budget is allocated (enable_offload=True).
    assert w.disk_offload_buffer_budget_bytes is not None
    assert w.disk_offload_buffer_budget_bytes > 0
```
**EN:** Test case covering `topology standalone store with disk offload`. Inputs/fixtures: `tmp_path, monkeypatch`. It exercises `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, isinstance`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `topology standalone store with disk offload` 的测试用例。 输入或 fixture：`tmp_path, monkeypatch`。 该测试会调用 `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, isinstance`。 代码主体包含 5 个显式断言。

### test_topology_embedded_cpu_only (lines 1094-1135)
```python
def test_topology_embedded_cpu_only(tmp_path, monkeypatch):
    """embedded + CPU-only: no mode key (defaults to embedded),
    global_segment_size>0, enable_offload absent, no preferred_segment.
    This is the PR-40900 baseline recipe."""
    store = MagicMock()
    store.setup.return_value = 0
    fake_replicate_config_cls = _install_fake_mooncake(monkeypatch, store)
    _patch_worker_runtime(monkeypatch)
    monkeypatch.setenv(
        "MOONCAKE_CONFIG_PATH",
        _write_mooncake_config(
            tmp_path,
            {
                "metadata_server": "http://metadata/endpoint",
                "global_segment_size": "4GB",
                "local_buffer_size": "4GB",
                "protocol": "rdma",
                "device_name": "mlx5_0",
    # ... excerpt omitted for brevity ...
    assert store.setup.call_args.args == (
    # preferred_segment field keeps its default value).
    assert w.preferred_segment is None
    assert isinstance(w.store_replicate_config, fake_replicate_config_cls)
    assert w.store_replicate_config.preferred_segment == ""
    # No disk budget — enable_offload was absent (defaults to False).
    assert w.disk_offload_buffer_budget_bytes is None
```
**EN:** Test case covering `topology embedded CPU only`. Inputs/fixtures: `tmp_path, monkeypatch`. It exercises `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, isinstance`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `topology embedded cpu only` 的测试用例。 输入或 fixture：`tmp_path, monkeypatch`。 该测试会调用 `MagicMock, _install_fake_mooncake, _patch_worker_runtime, monkeypatch.setenv, worker.MooncakeStoreWorker, isinstance`。 代码主体包含 5 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.distributed.kv_transfer.kv_connector.v1.mooncake, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data`.
- **CN:** 被测试的 vLLM 模块：`vllm.distributed.kv_transfer.kv_connector.v1.mooncake, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data`。
- **EN:** Standard-library support: `json, logging, math, sys, threading, types, unittest.mock`.
- **CN:** 标准库支持：`json, logging, math, sys, threading, types, unittest.mock`。
