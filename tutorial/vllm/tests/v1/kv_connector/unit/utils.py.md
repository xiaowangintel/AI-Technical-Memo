# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helpers reused by the v1 unit tests. / 为 v1 `unit` 测试提供可复用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-46)
```python
import tempfile
from collections import defaultdict
from collections.abc import Callable
from dataclasses import dataclass
from itertools import chain, count
from typing import Any, Literal

import torch
from vllm import SamplingParams
from vllm.config import (
    AttentionConfig,
    CacheConfig,
    DeviceConfig,
    KVTransferConfig,
    ModelConfig,
    SchedulerConfig,
    VllmConfig,
# ... excerpt omitted for brevity ...
    MambaSpec,
    SlidingWindowSpec,
)
from vllm.v1.outputs import KVConnectorOutput, ModelRunnerOutput
from vllm.v1.request import Request
from vllm.v1.structured_output import StructuredOutputManager
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `torch`. vLLM modules under test include `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.factory, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.example_connector, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `torch`。 被测试的 vLLM 模块包括 `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.factory, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.example_connector, ...`。

### Module state / 模块级状态 (line 48)
```python
EOS_TOKEN_ID = 50256
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `EOS_TOKEN_ID`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`EOS_TOKEN_ID`。

### assert_scheduler_empty (lines 51-87)
```python
def assert_scheduler_empty(scheduler: Scheduler):
    """Confirm the scheduler is "empty" - i.e. no leaks."""
    # Scheduler Metadata.
    assert len(scheduler.requests) == 0
    assert len(scheduler.waiting) == 0
    assert len(scheduler.running) == 0
    assert len(scheduler.finished_req_ids) == 0
    assert len(scheduler.finished_recving_kv_req_ids) == 0

    # EncoderCacheManager.
    assert len(scheduler.encoder_cache_manager.freed) == 0
    assert len(scheduler.encoder_cache_manager.cached) == 0
    # KVCache Manager.
    assert (
        len(
            scheduler.kv_cache_manager.coordinator.single_type_managers[0].req_to_blocks
        )
    # ... excerpt omitted for brevity ...
    assert num_free_blocks == (scheduler.kv_cache_manager.block_pool.num_gpu_blocks - 1)
    # NOTE(rob): just the ref count on blocks will be 0. The hash
    # value, etc will remain since we lazily evict for prefix cache.
    for block in scheduler.kv_cache_manager.block_pool.blocks:
        assert block.ref_cnt == 0
```
**EN:** Helper function `assert_scheduler_empty` encapsulates reusable logic for `assert scheduler empty`. Inputs: `scheduler`. Key calls include `len`. It includes 11 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `assert_scheduler_empty` 封装了与 `assert 调度器 empty` 相关的可复用逻辑。 输入参数：`scheduler`。 关键调用包括 `len`。 其中包含 11 个内部断言，用于保护前置假设。

### create_vllm_config (lines 90-148)
```python
def create_vllm_config(
    model: str = "facebook/opt-125m",
    max_num_seqs: int = 16,
    max_num_batched_tokens: int = 64,
    block_size: int = 16,
    max_model_len: int = 10000,
    enable_chunked_prefill: bool = True,
    enable_permute_local_kv: bool = False,
    kv_connector_extra_config: dict[str, Any] | None = None,
    dtype: str = "float16",
    cache_dtype: str = "auto",
    hf_overrides: dict[str, Any] | None = None,
    attention_backend: str | None = None,
    kv_load_failure_policy: Literal["recompute", "fail"] = "fail",
    kv_connector: str = "NixlConnector",
    kv_connector_module_path: str | None = None,
    kv_role: str = "kv_both",
    disable_hybrid_kv_cache_manager: bool | None = None,
    # ... excerpt omitted for brevity ...
    return VllmConfig(
        model_config=model_config,
        cache_config=cache_config,
        kv_transfer_config=kv_transfer_config,
        device_config=DeviceConfig("cpu"),
        attention_config=attention_config,
    )
```
**EN:** Helper function `create_vllm_config` encapsulates reusable logic for `vllm config`. Inputs: `model, max_num_seqs, max_num_batched_tokens, block_size, max_model_len, enable_chunked_prefill, enable_permute_local_kv, kv_connector_extra_config, ...`. Key calls include `ModelConfig, SchedulerConfig, CacheConfig, KVTransferConfig, AttentionConfig, VllmConfig`.
**CN:** 辅助函数 `create_vllm_config` 封装了与 `vllm config` 相关的可复用逻辑。 输入参数：`model, max_num_seqs, max_num_batched_tokens, block_size, max_model_len, enable_chunked_prefill, enable_permute_local_kv, kv_connector_extra_config, ...`。 关键调用包括 `ModelConfig, SchedulerConfig, CacheConfig, KVTransferConfig, AttentionConfig, VllmConfig`。

### create_scheduler (lines 151-185)
```python
def create_scheduler(
    vllm_config: VllmConfig,
    num_blocks: int = 10000,
    kv_cache_config: KVCacheConfig | None = None,
) -> Scheduler | AsyncScheduler:
    """Initialize Scheduler For Testing."""
    block_size = vllm_config.cache_config.block_size
    if kv_cache_config is None:
        kv_cache_config = KVCacheConfig(
            num_blocks=num_blocks,  # A large number of blocks to hold all requests
            kv_cache_tensors=[],
            kv_cache_groups=[
                KVCacheGroupSpec(
                    ["layer"],
                    FullAttentionSpec(
                        block_size=block_size,
                        num_kv_heads=1,
                        head_size=1,
    # ... excerpt omitted for brevity ...
    return scheduler_cls(
        vllm_config=vllm_config,
        kv_cache_config=kv_cache_config,
        log_stats=True,
        structured_output_manager=StructuredOutputManager(vllm_config),
        block_size=block_size,
    )
```
**EN:** Helper function `create_scheduler` encapsulates reusable logic for `scheduler`. Inputs: `vllm_config, num_blocks, kv_cache_config`. Key calls include `scheduler_cls, KVCacheConfig, StructuredOutputManager, KVCacheGroupSpec, FullAttentionSpec`.
**CN:** 辅助函数 `create_scheduler` 封装了与 `调度器` 相关的可复用逻辑。 输入参数：`vllm_config, num_blocks, kv_cache_config`。 关键调用包括 `scheduler_cls, KVCacheConfig, StructuredOutputManager, KVCacheGroupSpec, FullAttentionSpec`。

### Module state / 模块级状态 (lines 188-189)
```python
_request_count = count(1)
_none_hash_initialized = False
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_request_count, _none_hash_initialized`. Shared setup calls include `count`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_request_count, _none_hash_initialized`。 共享初始化调用包括 `count`。

### create_request (lines 192-248)
```python
def create_request(
    request_id: int | None = None,
    num_tokens: int = 10,
    common_prefix_len=0,
    max_tokens: int = 16,
    do_remote_decode: bool = False,
    do_remote_prefill: bool = False,
    num_remote_blocks: int = 3,
    block_size: int = 16,
    hash_fn: Callable = sha256,
) -> Request:
    """Make dummy request for testing."""
    assert num_tokens >= common_prefix_len >= 0

    if request_id is None:
        request_id = next(_request_count)
    global _none_hash_initialized
    # ... excerpt omitted for brevity ...
        assert not do_remote_prefill
        pooling_params=None,
        mm_features=None,
        block_hasher=get_request_block_hasher(block_size, hash_fn),
    )
    req.kv_transfer_params = kv_transfer_params
    return req
```
**EN:** Helper function `create_request` encapsulates reusable logic for `request`. Inputs: `request_id, num_tokens, common_prefix_len, max_tokens, do_remote_decode, do_remote_prefill, num_remote_blocks, block_size, ...`. Key calls include `SamplingParams, sampling_params.update_from_generation_config, Request, next, init_none_hash, dict`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `create_request` 封装了与 `request` 相关的可复用逻辑。 输入参数：`request_id, num_tokens, common_prefix_len, max_tokens, do_remote_decode, do_remote_prefill, num_remote_blocks, block_size, ...`。 关键调用包括 `SamplingParams, sampling_params.update_from_generation_config, Request, next, init_none_hash, dict`。 其中包含 2 个内部断言，用于保护前置假设。

### create_model_runner_output (lines 251-295)
```python
def create_model_runner_output(
    reqs: list[Request],
    finished_sending: set[str] | None = None,
    finished_recving: set[str] | None = None,
    invalid_block_ids: set[int] | None = None,
    use_eos: bool = False,
    token_id: int = 0,
    kv_connector_worker_meta: KVConnectorWorkerMetadata | None = None,
) -> ModelRunnerOutput:
    """Make dummy model runner output for testing."""

    # Make request data.
    req_ids = [req.request_id for req in reqs]
    req_id_to_index = {req_id: idx for idx, req_id in enumerate(req_ids)}
    # Make sampled tokens.
    sampled_token = EOS_TOKEN_ID if use_eos else token_id
    sampled_token_ids = [[sampled_token] for _ in req_ids]
    # ... excerpt omitted for brevity ...
    return ModelRunnerOutput(
        sampled_token_ids=sampled_token_ids,
        logprobs=None,
        prompt_logprobs_dict={},
        pooler_output=None,
        kv_connector_output=kv_connector_output,
    )
```
**EN:** Helper function `create_model_runner_output` encapsulates reusable logic for `model runner output`. Inputs: `reqs, finished_sending, finished_recving, invalid_block_ids, use_eos, token_id, kv_connector_worker_meta`. Key calls include `ModelRunnerOutput, KVConnectorOutput, enumerate, set`.
**CN:** 辅助函数 `create_model_runner_output` 封装了与 `模型执行器 output` 相关的可复用逻辑。 输入参数：`reqs, finished_sending, finished_recving, invalid_block_ids, use_eos, token_id, kv_connector_worker_meta`。 关键调用包括 `ModelRunnerOutput, KVConnectorOutput, enumerate, set`。

### TestExampleConnector (lines 298-357)
```python
class TestExampleConnector(ExampleConnector):
    def __init__(
        self,
        config: VllmConfig,
        role: KVConnectorRole,
        kv_cache_config: KVCacheConfig,
    ):
        self.name = config.kv_transfer_config.kv_connector_extra_config["name"]
        self._connector = ExampleConnector(config, role, kv_cache_config)
        self.call_record: dict[str, int] = defaultdict(int)
        # Use a unique temp file per connector
        self._event_file = (
            tempfile.gettempdir()
            + f"/connector_{self.name}-{self.role.name}_events.log"
        )
        # Start with an empty file
        with open(self._event_file, "w") as _:
            pass
    # ... excerpt omitted for brevity ...
            return object.__getattribute__(self, name)
                except Exception as e:
                    print(f"[ERROR] Could not log event {name} for {self.name}: {e}")
                return attr(*args, **kwargs)

            return wrapper
        return attr
```
**EN:** Class `TestExampleConnector` groups 0 test method(s) and 2 helper/fixture method(s). Bases: `ExampleConnector`.
**CN:** 类 `TestExampleConnector` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。 基类：`ExampleConnector`。

### MockKVConfig (lines 361-363)
```python
class MockKVConfig:
    matched_tokens: int = 0
    is_async: bool = False
```
**EN:** Class `MockKVConfig` groups 0 test method(s).
**CN:** 类 `MockKVConfig` 组织了 0 个测试方法。

### MockKVConnectorMetadata (lines 366-369)
```python
class MockKVConnectorMetadata(KVConnectorMetadata):
    def __init__(self):
        # Scheduler tests check metadata.requests
        self.requests: list = []
```
**EN:** Class `MockKVConnectorMetadata` groups 0 test method(s) and 1 helper/fixture method(s). Bases: `KVConnectorMetadata`.
**CN:** 类 `MockKVConnectorMetadata` 组织了 0 个测试方法，以及 1 个辅助或 fixture 方法。 基类：`KVConnectorMetadata`。

### MockKVConnector (lines 372-429)
```python
class MockKVConnector(KVConnectorBase_V1):
    """Mock KV connector for scheduler tests, supporting both sync and async mode."""

    def __init__(
        self,
        vllm_config: VllmConfig,
        role: KVConnectorRole,
        kv_cache_config: KVCacheConfig,
    ):
        super().__init__(vllm_config, role, kv_cache_config)
        extra_config = self._kv_transfer_config.kv_connector_extra_config
        self.config = MockKVConfig(
            matched_tokens=extra_config["matched_tokens"],
            is_async=extra_config["is_async"],
        )
    def get_num_new_matched_tokens(
    # ... excerpt omitted for brevity ...
        return (self.config.matched_tokens, self.config.is_async)
        return metadata
    def save_kv_layer(self, layer_name, kv_layer, attn_metadata, **kwargs):
        pass
    def wait_for_save(self):
```
**EN:** Class `MockKVConnector` groups 0 test method(s) and 8 helper/fixture method(s). Bases: `KVConnectorBase_V1`.
**CN:** 类 `MockKVConnector` 组织了 0 个测试方法，以及 8 个辅助或 fixture 方法。 基类：`KVConnectorBase_V1`。

### Module state / 模块级状态 (lines 432-438)
```python
KVConnectorFactory.register_connector(
    "TestExampleConnector", __name__, TestExampleConnector.__name__
)

KVConnectorFactory.register_connector(
    "MockKVConnector", __name__, MockKVConnector.__name__
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Shared setup calls include `KVConnectorFactory.register_connector`.
**CN:** 定义模块级常量、特性开关或共享状态。 共享初始化调用包括 `KVConnectorFactory.register_connector`。

### make_kv_cache_config (lines 441-485)
```python
def make_kv_cache_config(
    block_size: int,
    swa_enabled: bool = False,
    mamba_enabled: bool = False,
    sw_size: int = 128,
    num_blocks: int = 100,
) -> KVCacheConfig:
    kv_cache_groups = [
        KVCacheGroupSpec(
            ["layer0", "layer2"],
            FullAttentionSpec(
                block_size=block_size,
                num_kv_heads=4,
                head_size=16,
                dtype=torch.float16,
            ),
        )
    ]
    # ... excerpt omitted for brevity ...
                ),
            )
    return KVCacheConfig(
        num_blocks=num_blocks, kv_cache_tensors=[], kv_cache_groups=kv_cache_groups
    )
```
**EN:** Helper function `make_kv_cache_config` encapsulates reusable logic for `KV cache config`. Inputs: `block_size, swa_enabled, mamba_enabled, sw_size, num_blocks`. Key calls include `KVCacheConfig, KVCacheGroupSpec, kv_cache_groups.append, FullAttentionSpec, SlidingWindowSpec, MambaSpec`.
**CN:** 辅助函数 `make_kv_cache_config` 封装了与 `KV 缓存 config` 相关的可复用逻辑。 输入参数：`block_size, swa_enabled, mamba_enabled, sw_size, num_blocks`。 关键调用包括 `KVCacheConfig, KVCacheGroupSpec, kv_cache_groups.append, FullAttentionSpec, SlidingWindowSpec, MambaSpec`。

### make_nixl_scheduler (lines 488-525)
```python
def make_nixl_scheduler(
    has_mamba: bool = False,
    is_hma_required: bool = False,
    heartbeat: bool = False,
    kv_lease_duration: int = 30,
):
    """Create a NixlConnectorScheduler via __new__ (skipping __init__).

    Only sets the flags needed by the tests.  When *heartbeat=True* the
    scheduler-side heartbeat bookkeeping fields are also initialised.
    """
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.scheduler import (
        NixlConnectorScheduler,
    )
    sched = object.__new__(NixlConnectorScheduler)
    sched._has_mamba = has_mamba
    sched._is_hma_required = is_hma_required
    # ... excerpt omitted for brevity ...
        sched.engine_id = "test-engine"
        sched.side_channel_host = "localhost"
        sched.side_channel_port = 5555
        sched.blocks_per_sw = []
        sched.is_bidirectional_kv_xfer_enabled = False
    return sched
```
**EN:** Helper function `make_nixl_scheduler` encapsulates reusable logic for `NIXL scheduler`. Inputs: `has_mamba, is_hma_required, heartbeat, kv_lease_duration`. Key calls include `object.__new__, set`.
**CN:** 辅助函数 `make_nixl_scheduler` 封装了与 `nixl 调度器` 相关的可复用逻辑。 输入参数：`has_mamba, is_hma_required, heartbeat, kv_lease_duration`。 关键调用包括 `object.__new__, set`。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径

## Dependencies / 依赖关系
- **EN:** External libraries: `torch`.
- **CN:** 外部库：`torch`。
- **EN:** vLLM modules under test: `vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.factory, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.example_connector, vllm.utils.hashing, vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config, vllm.distributed.kv_transfer.kv_connector.factory, vllm.distributed.kv_transfer.kv_connector.v1.base, vllm.distributed.kv_transfer.kv_connector.v1.example_connector, vllm.utils.hashing, vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, ...`。
- **EN:** Standard-library support: `tempfile, collections, collections.abc, dataclasses, itertools, typing`.
- **CN:** 标准库支持：`tempfile, collections, collections.abc, dataclasses, itertools, typing`。
