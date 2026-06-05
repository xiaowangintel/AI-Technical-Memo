# test_scheduler_kv_connector_override.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_scheduler_kv_connector_override.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `scheduler KV connector override` behavior and regressions in the v1 stack. / 验证 v1 栈中 `调度器 KV 连接器 override` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-21)
```python
from unittest.mock import MagicMock, patch

import pytest

import vllm.plugins as plugins_module
from tests.v1.core.utils import create_requests, create_scheduler
from vllm.distributed.kv_transfer.kv_connector.factory import (
    KVConnectorFactory,
)
from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorBase_V1,
    KVConnectorMetadata,
)
from vllm.v1.core.kv_cache_manager import KVCacheBlocks
from vllm.v1.core.kv_cache_utils import BlockHash
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.core.sched.scheduler import Scheduler
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.request import Request
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.plugins, vllm.distributed.kv_transfer.kv_connector.factory, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, ...`. Local helpers come from `tests.v1.core.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.plugins, vllm.distributed.kv_transfer.kv_connector.factory, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, ...`。 本地测试辅助逻辑来自 `tests.v1.core.utils`。

### DummyConnectorMetadata (lines 24-26)
```python
class DummyConnectorMetadata(KVConnectorMetadata):
    def __init__(self, block_hashes_by_req: dict[str, list[BlockHash]]):
        self.block_hashes_by_req = block_hashes_by_req
```
**EN:** Class `DummyConnectorMetadata` groups 0 test method(s) and 1 helper/fixture method(s). Bases: `KVConnectorMetadata`.
**CN:** 类 `DummyConnectorMetadata` 组织了 0 个测试方法，以及 1 个辅助或 fixture 方法。 基类：`KVConnectorMetadata`。

### DummyKVConnector (lines 29-64)
```python
class DummyKVConnector(KVConnectorBase_V1):
    def __init__(self, vllm_config, role, kv_cache_config: KVCacheConfig):
        super().__init__(vllm_config, role, kv_cache_config)

    def get_num_new_matched_tokens(
        self, request: Request, num_computed_tokens: int
    ) -> tuple[int | None, bool]:
        return (0, False)
    def update_state_after_alloc(
        self, request: Request, blocks: KVCacheBlocks, num_external_tokens: int
    ):
        pass
    def build_connector_meta(
        self, scheduler_output: SchedulerOutput
    ) -> KVConnectorMetadata:
        block_hashes_by_req = getattr(scheduler_output, "block_hashes_by_req", None)
    # ... excerpt omitted for brevity ...
        assert block_hashes_by_req is not None, (
            "DummyKVConnector expected 'block_hashes_by_req' on scheduler_output"
        return DummyConnectorMetadata(
    def save_kv_layer(self, layer_name, kv_layer, attn_metadata, **kwargs):
    def wait_for_save(self):
```
**EN:** Class `DummyKVConnector` groups 0 test method(s) and 8 helper/fixture method(s). Bases: `KVConnectorBase_V1`.
**CN:** 类 `DummyKVConnector` 组织了 0 个测试方法，以及 8 个辅助或 fixture 方法。 基类：`KVConnectorBase_V1`。

### _my_plugin (lines 67-86)
```python
def _my_plugin():
    """Registers the dummy KV connector and overrides _build_kv_connector_meta"""
    KVConnectorFactory.register_connector(
        "DummyKVConnector",
        __name__,
        DummyKVConnector.__name__,
    )

    def _custom_build_kv_connector_meta(
        self, connector: KVConnectorBase_V1, scheduler_output: SchedulerOutput
    ) -> KVConnectorMetadata:
        block_hashes_by_req: dict[str, list[BlockHash]] = {}
        for req_id in scheduler_output.num_scheduled_tokens:
            request = self.requests[req_id]
            block_hashes_by_req[req_id] = request.block_hashes

        scheduler_output.block_hashes_by_req = block_hashes_by_req  # type: ignore[attr-defined]
        return connector.build_connector_meta(scheduler_output)

    Scheduler._build_kv_connector_meta = _custom_build_kv_connector_meta
```
**EN:** Helper function `_my_plugin` encapsulates reusable logic for `my plugin`. Key calls include `KVConnectorFactory.register_connector, connector.build_connector_meta`.
**CN:** 辅助函数 `_my_plugin` 封装了与 `my plugin` 相关的可复用逻辑。 关键调用包括 `KVConnectorFactory.register_connector, connector.build_connector_meta`。

### _load_plugin (lines 90-103)
```python
def _load_plugin():
    """Load the fake plugin through the real load_general_plugins() path."""
    ep = MagicMock()
    ep.name = "dummy_kv_connector_plugin"
    ep.value = f"{__name__}:_my_plugin"
    ep.load.return_value = _my_plugin

    # Reset the global guard so load_general_plugins() actually runs.
    plugins_module.plugins_loaded = False
    with patch("importlib.metadata.entry_points", return_value=[ep]):
        plugins_module.load_general_plugins()
        yield
    # Reset again so other tests are not affected.
    plugins_module.plugins_loaded = False
```
**EN:** Fixture/helper `_load_plugin` prepares reusable state for downstream tests. Key calls include `MagicMock, patch, plugins_module.load_general_plugins`.
**CN:** `_load_plugin` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `MagicMock, patch, plugins_module.load_general_plugins`。

### test_connector_receives_block_hashes (lines 106-131)
```python
def test_connector_receives_block_hashes(_load_plugin):
    block_size = 16
    num_tokens = 48  # 3 full blocks worth of tokens
    scheduler = create_scheduler(
        use_kv_connector="DummyKVConnector", block_size=block_size
    )
    requests = create_requests(
        num_requests=3, num_tokens=num_tokens, block_size=block_size
    )
    for req in requests:
        scheduler.add_request(req)

    output = scheduler.schedule()

    # Verify the connector metadata was built with block hashes.
    meta = output.kv_connector_metadata
    assert isinstance(meta, DummyConnectorMetadata)
    assert len(meta.block_hashes_by_req) == 3

    for req in requests:
        assert req.request_id in meta.block_hashes_by_req
        # Each request has num_tokens / block_size = 3 full block hashes.
        assert len(meta.block_hashes_by_req[req.request_id]) == (
            num_tokens // block_size
        )
        assert meta.block_hashes_by_req[req.request_id] == req.block_hashes
```
**EN:** Test case covering `connector receives block hashes`. Inputs/fixtures: `_load_plugin`. It exercises `create_scheduler, create_requests, scheduler.schedule, isinstance, scheduler.add_request, len`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `connector receives block hashes` 的测试用例。 输入或 fixture：`_load_plugin`。 该测试会调用 `create_scheduler, create_requests, scheduler.schedule, isinstance, scheduler.add_request, len`。 代码主体包含 5 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.plugins, vllm.distributed.kv_transfer.kv_connector.factory, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.output, vllm.v1.core.sched.scheduler, vllm.v1.kv_cache_interface, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.plugins, vllm.distributed.kv_transfer.kv_connector.factory, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.output, vllm.v1.core.sched.scheduler, vllm.v1.kv_cache_interface, ...`。
- **EN:** Local test helpers: `tests.v1.core.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.core.utils`。
- **EN:** Standard-library support: `unittest.mock`.
- **CN:** 标准库支持：`unittest.mock`。
