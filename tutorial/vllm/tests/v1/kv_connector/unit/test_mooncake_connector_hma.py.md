# test_mooncake_connector_hma.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_mooncake_connector_hma.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for MooncakeConnector HMA (Hybrid Memory Architecture) support. / 该文件的文档字符串表明其用途：`unit tests for mooncakeconnector hma (hybrid memory architecture) support`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-7)
```python
"""Unit tests for MooncakeConnector HMA (Hybrid Memory Architecture) support.

Covers sliding-window clipping, multi-group metadata shape, multi-group
send trimming, and group-count invariant checking in _build_transfer_params.
"""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for MooncakeConnector HMA (Hybrid Memory Architecture) support.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for mooncakeconnector hma (hybrid memory architecture) support`。

### Imports and setup / 导入与设置 (lines 9-26)
```python
import asyncio
from unittest.mock import patch

import pytest

from vllm.config import set_current_vllm_config
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector import (
    KVConnectorRole,
    MooncakeConnector,
    MooncakeConnectorMetadata,
    MooncakeConnectorScheduler,
    MooncakeXferMetadata,
    SendBlockMeta,
    TransferRegion,
)

from .test_mooncake_connector import FakeMooncakeWrapper, patch_worker_dependencies
from .utils import create_request, create_vllm_config, make_kv_cache_config
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.config, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.v1.request`. Local helpers come from `tests.v1.kv_connector.unit.test_mooncake_connector, tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.config, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.v1.request`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.test_mooncake_connector, tests.v1.kv_connector.unit.utils`。

### test_sw_sizes (lines 42-61)
```python
def test_sw_sizes(swa_enabled, expected_blocks_per_sw):
    """blocks_per_sw is correctly computed based on SWA enabled/disabled."""
    block_size = 16
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector",
        kv_role="kv_both",
        block_size=block_size,
    )
    # Override so HMA detection works
    vllm_config.scheduler_config.disable_hybrid_kv_cache_manager = False
    kv_cache_config = make_kv_cache_config(
        block_size=block_size, swa_enabled=swa_enabled, sw_size=2048
    )

    scheduler = MooncakeConnectorScheduler(
        vllm_config=vllm_config,
        engine_id="test-engine",
        kv_cache_config=kv_cache_config,
    )
    assert scheduler.blocks_per_sw == expected_blocks_per_sw
```
**EN:** Parameterized test covering `sw sizes`. Parameter axes: `swa_enabled, expected_blocks_per_sw`. Inputs/fixtures: `swa_enabled, expected_blocks_per_sw`. It exercises `mark.parametrize, create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `sw sizes` 的测试用例。 参数维度：`swa_enabled, expected_blocks_per_sw`。 输入或 fixture：`swa_enabled, expected_blocks_per_sw`。 该测试会调用 `mark.parametrize, create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler`。 代码主体包含 1 个显式断言。

### test_is_hma_required (lines 76-94)
```python
def test_is_hma_required(swa_enabled, disable_hma, expected_is_hma):
    """_is_hma_required is correctly derived from kv_cache_config."""
    block_size = 16
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector",
        kv_role="kv_both",
        block_size=block_size,
    )
    vllm_config.scheduler_config.disable_hybrid_kv_cache_manager = disable_hma
    kv_cache_config = make_kv_cache_config(
        block_size=block_size, swa_enabled=swa_enabled
    )

    scheduler = MooncakeConnectorScheduler(
        vllm_config=vllm_config,
        engine_id="test-engine",
        kv_cache_config=kv_cache_config,
    )
    assert scheduler._is_hma_required is expected_is_hma
```
**EN:** Parameterized test covering `is hma required`. Parameter axes: `swa_enabled, disable_hma, expected_is_hma`. Inputs/fixtures: `swa_enabled, disable_hma, expected_is_hma`. It exercises `mark.parametrize, create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `is hma required` 的测试用例。 参数维度：`swa_enabled, disable_hma, expected_is_hma`。 输入或 fixture：`swa_enabled, disable_hma, expected_is_hma`。 该测试会调用 `mark.parametrize, create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler`。 代码主体包含 1 个显式断言。

### test_get_sw_clipped_blocks (lines 101-133)
```python
def test_get_sw_clipped_blocks():
    """get_sw_clipped_blocks clips SWA group but keeps FA group intact."""
    block_size = 16
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector",
        kv_role="kv_both",
        block_size=block_size,
    )
    vllm_config.scheduler_config.disable_hybrid_kv_cache_manager = False
    # SW=128 tokens → 128/16 = 8 blocks + 1 = 9 blocks_per_sw
    kv_cache_config = make_kv_cache_config(
        block_size=block_size, swa_enabled=True, sw_size=128

    scheduler = MooncakeConnectorScheduler(
        vllm_config=vllm_config,
        engine_id="test-engine",
        kv_cache_config=kv_cache_config,
    # ... excerpt omitted for brevity ...
    assert scheduler.blocks_per_sw == [0, 9]
    # FA: untouched (blocks_per_sw[0] = 0)
    assert clipped[0] == fa_blocks
    # SW: clipped to last 9 blocks
    assert clipped[1] == sw_blocks[-9:]
    assert len(clipped[1]) == 9
```
**EN:** Test case covering `get sw clipped blocks`. It exercises `create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler, list, scheduler.get_sw_clipped_blocks, range`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `get sw clipped blocks` 的测试用例。 该测试会调用 `create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler, list, scheduler.get_sw_clipped_blocks, range`。 代码主体包含 4 个显式断言。

### test_get_sw_clipped_blocks_noop_no_hma (lines 137-157)
```python
def test_get_sw_clipped_blocks_noop_no_hma():
    """get_sw_clipped_blocks is a no-op when HMA is not required."""
    block_size = 16
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector",
        kv_role="kv_both",
        block_size=block_size,
    )
    # FA only → _is_hma_required = False
    kv_cache_config = make_kv_cache_config(block_size=block_size, swa_enabled=False)

    scheduler = MooncakeConnectorScheduler(
        vllm_config=vllm_config,
        engine_id="test-engine",
        kv_cache_config=kv_cache_config,
    )
    assert scheduler._is_hma_required is False

    block_ids = ([1, 2, 3],)
    clipped = scheduler.get_sw_clipped_blocks(block_ids)
    assert clipped == [[1, 2, 3]]
```
**EN:** Test case covering `get sw clipped blocks noop no hma`. It exercises `create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler, scheduler.get_sw_clipped_blocks`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `get sw clipped blocks noop no hma` 的测试用例。 该测试会调用 `create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler, scheduler.get_sw_clipped_blocks`。 代码主体包含 2 个显式断言。

### test_metadata_hma_block_ids (lines 164-205)
```python
def test_metadata_hma_block_ids():
    """MooncakeConnectorMetadata.add_new_req stores per-group block IDs."""
    metadata = MooncakeConnectorMetadata()

    # FA group: 6 blocks, SW group: 3 blocks (clipped)
    fa_blocks = [0, 1, 2, 3, 4, 5]
    sw_blocks = [10, 11, 12]
    # Test recv path
    metadata.add_new_req(
        request_id="recv-req",
        local_block_ids=[fa_blocks, sw_blocks],
        kv_transfer_params={
            "transfer_id": "recv-req",
            "remote_engine_id": "remote-engine",
            "remote_bootstrap_addr": "http://bootstrap:33333",
        },
        load_remote_cache=True,
    # ... excerpt omitted for brevity ...
    assert "recv-req" in metadata.reqs_to_recv["remote-engine"]
    assert len(req_meta.local_block_ids) == 2
    assert req_meta.local_block_ids[0] == fa_blocks
    assert req_meta.local_block_ids[1] == sw_blocks
    assert "send-req" in metadata.reqs_to_send
    transfer_id, stored_blocks = metadata.reqs_to_send["send-req"]
    assert transfer_id == "send-req"
    assert len(stored_blocks) == 2
    assert stored_blocks[0] == fa_blocks
    assert stored_blocks[1] == sw_blocks
```
**EN:** Test case covering `metadata hma block ids`. It exercises `MooncakeConnectorMetadata, metadata.add_new_req, len`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `metadata hma block ids` 的测试用例。 该测试会调用 `MooncakeConnectorMetadata, metadata.add_new_req, len`。 代码主体包含 9 个显式断言。

### test_build_transfer_params_multi_group_trimming (lines 217-293)
```python
async def test_build_transfer_params_multi_group_trimming(monkeypatch):
    """_build_transfer_params trims per-group blocks when local > remote."""

    monkeypatch.setenv("VLLM_MOONCAKE_ABORT_REQUEST_TIMEOUT", "5")
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_producer"
    )
    kv_cache_config = make_kv_cache_config(
        block_size=vllm_config.cache_config.block_size, swa_enabled=True
    with set_current_vllm_config(vllm_config), patch_worker_dependencies():
        connector = MooncakeConnector(
            vllm_config, KVConnectorRole.WORKER, kv_cache_config
        )
        worker = connector.connector_worker
        block_len = 4096
    # ... excerpt omitted for brevity ...
        assert err_reqs == []
        assert err_msg is None
        # Flattened: [12,13,21,22] = 4 blocks → coalesced into some transfers
        assert len(src_ptrs) > 0
        assert len(dst_ptrs) == len(src_ptrs)
        assert len(lengths) == len(src_ptrs)
        worker.shutdown()
```
**EN:** Async test covering `build transfer params multi group trimming`. Inputs/fixtures: `monkeypatch`. It exercises `patch, monkeypatch.setenv, create_vllm_config, make_kv_cache_config, set_current_vllm_config, patch_worker_dependencies`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `build transfer params multi group trimming` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `patch, monkeypatch.setenv, create_vllm_config, make_kv_cache_config, set_current_vllm_config, patch_worker_dependencies`。 代码主体包含 5 个显式断言。

### test_build_transfer_params_group_count_mismatch (lines 305-374)
```python
async def test_build_transfer_params_group_count_mismatch(monkeypatch):
    """_build_transfer_params reports an error when group counts differ."""

    monkeypatch.setenv("VLLM_MOONCAKE_ABORT_REQUEST_TIMEOUT", "5")
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector", kv_role="kv_producer"
    )
    kv_cache_config = make_kv_cache_config(
        block_size=vllm_config.cache_config.block_size, swa_enabled=True
    with set_current_vllm_config(vllm_config), patch_worker_dependencies():
        connector = MooncakeConnector(
            vllm_config, KVConnectorRole.WORKER, kv_cache_config
        )
        worker = connector.connector_worker
        block_len = 4096
    # ... excerpt omitted for brevity ...
        assert err_reqs == ["d-mismatch"]
        assert err_msg == "KV group count mismatch"
        assert src_ptrs == []
        assert dst_ptrs == []
        assert lengths == []
        worker.shutdown()
```
**EN:** Async test covering `build transfer params group count mismatch`. Inputs/fixtures: `monkeypatch`. It exercises `patch, monkeypatch.setenv, create_vllm_config, make_kv_cache_config, set_current_vllm_config, patch_worker_dependencies`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `build transfer params group count mismatch` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `patch, monkeypatch.setenv, create_vllm_config, make_kv_cache_config, set_current_vllm_config, patch_worker_dependencies`。 代码主体包含 5 个显式断言。

### test_request_finished_with_hma_groups (lines 381-420)
```python
def test_request_finished_with_hma_groups():
    """request_finished correctly handles per-group block_ids."""
    block_size = 16
    vllm_config = create_vllm_config(
        kv_connector="MooncakeConnector",
        kv_role="kv_producer",
        block_size=block_size,
    )
    vllm_config.scheduler_config.disable_hybrid_kv_cache_manager = False
    kv_cache_config = make_kv_cache_config(
        block_size=block_size, swa_enabled=True, sw_size=128

    scheduler = MooncakeConnectorScheduler(
        vllm_config=vllm_config,
        engine_id="test-engine",
        kv_cache_config=kv_cache_config,
    # ... excerpt omitted for brevity ...
    assert delay_free is True
    assert request.request_id in scheduler._reqs_need_send
    _, stored_blocks = scheduler._reqs_need_send[request.request_id]
    # FA: untouched
    assert stored_blocks[0] == fa_blocks
    # SW: clipped to last 9 blocks (sw_size=128, block_size=16 → 8+1=9)
    assert stored_blocks[1] == sw_blocks[-9:]
```
**EN:** Test case covering `request finished with hma groups`. It exercises `create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler, create_request, list, scheduler.request_finished`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `request finished with hma groups` 的测试用例。 该测试会调用 `create_vllm_config, make_kv_cache_config, MooncakeConnectorScheduler, create_request, list, scheduler.request_finished`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.config, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.v1.request`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector, vllm.v1.request`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.test_mooncake_connector, tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.test_mooncake_connector, tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `asyncio, unittest.mock`.
- **CN:** 标准库支持：`asyncio, unittest.mock`。
