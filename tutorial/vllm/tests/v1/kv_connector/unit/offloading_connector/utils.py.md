# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/offloading_connector/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helpers reused by the v1 offloading connector tests. / 为 v1 `offloading connector` 测试提供可复用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-55)
```python
from collections.abc import Iterable, Iterator
from dataclasses import dataclass
from typing import Any
from unittest.mock import MagicMock, patch

import pytest
import torch
from tests.v1.kv_connector.unit.utils import (
    EOS_TOKEN_ID,
    create_model_runner_output,
    create_vllm_config,
)
from vllm import SamplingParams
from vllm.config import KVTransferConfig, VllmConfig, set_current_vllm_config
from vllm.distributed.kv_transfer.kv_connector.v1 import KVConnectorRole
from vllm.distributed.kv_transfer.kv_connector.v1.offloading.common import (
    OffloadingConnectorMetadata,
# ... excerpt omitted for brevity ...
    OffloadingHandler,
    TransferResult,
    TransferSpec,
from vllm.v1.request import Request
from vllm.v1.structured_output import StructuredOutputManager
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.v1, vllm.distributed.kv_transfer.kv_connector.v1.offloading.common, vllm.distributed.kv_transfer.kv_connector.v1.offloading_connector, ...`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.v1, vllm.distributed.kv_transfer.kv_connector.v1.offloading.common, vllm.distributed.kv_transfer.kv_connector.v1.offloading_connector, ...`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### to_key (lines 58-59)
```python
def to_key(int_hash: int) -> OffloadKey:
    return make_offload_key(str(int_hash).encode(), 0)
```
**EN:** Helper function `to_key` encapsulates reusable logic for `to key`. Inputs: `int_hash`. Key calls include `make_offload_key, str.encode, str`.
**CN:** 辅助函数 `to_key` 封装了与 `to key` 相关的可复用逻辑。 输入参数：`int_hash`。 关键调用包括 `make_offload_key, str.encode, str`。

### to_keys (lines 62-63)
```python
def to_keys(int_hashes: list[int]) -> list[OffloadKey]:
    return [to_key(i) for i in int_hashes]
```
**EN:** Helper function `to_keys` encapsulates reusable logic for `to keys`. Inputs: `int_hashes`. Key calls include `to_key`.
**CN:** 辅助函数 `to_keys` 封装了与 `to keys` 相关的可复用逻辑。 输入参数：`int_hashes`。 关键调用包括 `to_key`。

### MockLoadStoreSpec (lines 66-75)
```python
class MockLoadStoreSpec(LoadStoreSpec):
    def __init__(self, offload_keys: Iterable[OffloadKey]):
        self.offload_keys: list[OffloadKey] = list(offload_keys)

    @staticmethod
    def medium() -> str:
        return "Mock"

    def __repr__(self) -> str:
        return repr(self.offload_keys)
```
**EN:** Class `MockLoadStoreSpec` groups 0 test method(s) and 3 helper/fixture method(s). Bases: `LoadStoreSpec`.
**CN:** 类 `MockLoadStoreSpec` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。 基类：`LoadStoreSpec`。

### MockOffloadingHandler (lines 78-112)
```python
class MockOffloadingHandler(OffloadingHandler):
    def __init__(self):
        self.transfer_specs: dict[int, TransferSpec] = {}
        self.completed_transfers: list[TransferResult] = []
        self.waiting_jobs: set[int] = set()
        self.completed_jobs: list[int] = []
        self.flushed_jobs: set[int] = set()

    def get_finished(self) -> list[TransferResult]:
        finished = self.completed_transfers
        self.completed_transfers = []
        return finished
    def transfer_async(self, job_id: int, spec: TransferSpec) -> bool:
        self.transfer_specs[job_id] = spec
        self.waiting_jobs.add(job_id)
        return True
    # ... excerpt omitted for brevity ...
                )
                self.completed_transfers.append(result)
    def wait(self, job_ids: set[int]) -> None:
        self.flushed_jobs |= job_ids
        self.complete_jobs(job_ids)
```
**EN:** Class `MockOffloadingHandler` groups 0 test method(s) and 5 helper/fixture method(s). Bases: `OffloadingHandler`.
**CN:** 类 `MockOffloadingHandler` 组织了 0 个测试方法，以及 5 个辅助或 fixture 方法。 基类：`OffloadingHandler`。

### MockOffloadingSpec (lines 115-150)
```python
class MockOffloadingSpec(OffloadingSpec):
    def __init__(self, vllm_config: VllmConfig, kv_cache_config: KVCacheConfig):
        super().__init__(vllm_config, kv_cache_config)

        self.manager = MagicMock(spec=OffloadingManager)
        self.manager.lookup.return_value = 0
        self.manager.prepare_load = lambda keys, req_context: MockLoadStoreSpec(keys)
        self.manager.lookup.return_value = False
        self.handler = MockOffloadingHandler()
    def get_manager(self) -> OffloadingManager:
        return self.manager
    def get_handlers(
        self, _
    ) -> Iterator[tuple[type[LoadStoreSpec], type[LoadStoreSpec], OffloadingHandler]]:
        yield GPULoadStoreSpec, MockLoadStoreSpec, self.handler
        yield MockLoadStoreSpec, GPULoadStoreSpec, self.handler
    # ... excerpt omitted for brevity ...
        return specs
    def get_flushed_transfers(self):
        specs = [
            self.handler.transfer_specs[job_id] for job_id in self.handler.flushed_jobs
        ]
        self.handler.flushed_jobs.clear()
```
**EN:** Class `MockOffloadingSpec` groups 0 test method(s) and 6 helper/fixture method(s). Bases: `OffloadingSpec`.
**CN:** 类 `MockOffloadingSpec` 组织了 0 个测试方法，以及 6 个辅助或 fixture 方法。 基类：`OffloadingSpec`。

### GPUBlock (lines 154-156)
```python
class GPUBlock:
    group_idx: int
    request_block_offset: int
```
**EN:** Class `GPUBlock` groups 0 test method(s).
**CN:** 类 `GPUBlock` 组织了 0 个测试方法。

### TransferSummary (lines 160-162)
```python
class TransferSummary:
    gpu_blocks: list[GPUBlock]
    offload_addresses: list[Any]
```
**EN:** Class `TransferSummary` groups 0 test method(s).
**CN:** 类 `TransferSummary` 组织了 0 个测试方法。

### RequestRunner (lines 165-597)
```python
class RequestRunner:
    def __init__(
        self,
        block_size: int,
        num_gpu_blocks: int,
        block_size_factor: int = 1,
        async_scheduling: bool = True,
        kv_cache_groups: list[KVCacheGroupSpec] | None = None,
    ):
        assert block_size_factor == 1 or kv_cache_groups is None, (
            "block_size_factor > 1 requires all groups to have the same "
            "block size, so kv_cache_groups must be None (use default group)"
        )

        self.block_size_factor: int = block_size_factor
        self.block_size: int = block_size
        self.num_gpu_blocks: int = num_gpu_blocks
        self.async_scheduling: bool = async_scheduling
    # ... excerpt omitted for brevity ...
            mock_layer = MagicMock()
            return {name: mock_layer for name in layer_names}
        assert scheduler_connector is not None
        assert isinstance(scheduler_connector, OffloadingConnector)
        assert self.connector_scheduler is not None
        assert isinstance(manager, MagicMock)
        assert set(expected_stored_gpu_blocks) == stored_gpu_blocks
        self.completed_stores.clear()
        assert set(expected_flushed_gpu_blocks) == self.flushed_gpu_blocks
        self.flushed_gpu_blocks.clear()
```
**EN:** Class `RequestRunner` groups 0 test method(s) and 7 helper/fixture method(s).
**CN:** 类 `RequestRunner` 组织了 0 个测试方法，以及 7 个辅助或 fixture 方法。

### request_runner (lines 601-621)
```python
def request_runner():
    runners = []

    def runner_factory(
        block_size,
        num_gpu_blocks,
        async_scheduling,
        block_size_factor=1,
        kv_cache_groups=None,
    ):
        runner = RequestRunner(
            block_size=block_size,
            num_gpu_blocks=num_gpu_blocks,
            block_size_factor=block_size_factor,
            async_scheduling=async_scheduling,
            kv_cache_groups=kv_cache_groups,
        )
        runners.append(runner)
        return runner

    yield runner_factory  # pass factory to the test
```
**EN:** Fixture/helper `request_runner` prepares reusable state for downstream tests. Key calls include `RequestRunner, runners.append`.
**CN:** `request_runner` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `RequestRunner, runners.append`。

### generate_store_output (lines 624-630)
```python
def generate_store_output(keys: Iterable[OffloadKey]):
    keys = list(keys)
    return PrepareStoreOutput(
        keys_to_store=list(keys),
        store_spec=MockLoadStoreSpec(keys),
        evicted_keys=[],
    )
```
**EN:** Helper function `generate_store_output` encapsulates reusable logic for `generate store output`. Inputs: `keys`. Key calls include `list, PrepareStoreOutput, MockLoadStoreSpec`.
**CN:** 辅助函数 `generate_store_output` 封装了与 `generate store output` 相关的可复用逻辑。 输入参数：`keys`。 关键调用包括 `list, PrepareStoreOutput, MockLoadStoreSpec`。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.v1, vllm.distributed.kv_transfer.kv_connector.v1.offloading.common, vllm.distributed.kv_transfer.kv_connector.v1.offloading_connector, vllm.forward_context, vllm.utils.hashing, vllm.v1.attention.backends.flash_attn, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.v1, vllm.distributed.kv_transfer.kv_connector.v1.offloading.common, vllm.distributed.kv_transfer.kv_connector.v1.offloading_connector, vllm.forward_context, vllm.utils.hashing, vllm.v1.attention.backends.flash_attn, ...`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `collections.abc, dataclasses, typing, unittest.mock`.
- **CN:** 标准库支持：`collections.abc, dataclasses, typing, unittest.mock`。
