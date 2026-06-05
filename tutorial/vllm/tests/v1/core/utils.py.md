# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/core/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helpers reused by the v1 core tests. / 为 v1 `core` 测试提供可复用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-33)
```python
import torch

from tests.v1.kv_connector.unit.utils import MockKVConfig
from vllm.config import (
    CacheConfig,
    ECTransferConfig,
    KVTransferConfig,
    ModelConfig,
    ParallelConfig,
    SchedulerConfig,
    SpeculativeConfig,
    VllmConfig,
)
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalKwargsItem,
    PlaceholderRange,
)
from vllm.sampling_params import SamplingParams
from vllm.utils.hashing import sha256
from vllm.v1.core.kv_cache_utils import get_request_block_hasher, init_none_hash
from vllm.v1.core.sched.async_scheduler import AsyncScheduler
from vllm.v1.core.sched.scheduler import Scheduler
from vllm.v1.kv_cache_interface import (
    FullAttentionSpec,
    KVCacheConfig,
    KVCacheGroupSpec,
)
from vllm.v1.request import Request
from vllm.v1.structured_output import StructuredOutputManager
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `torch`. vLLM modules under test include `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.kv_cache_utils, ...`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `torch`。 被测试的 vLLM 模块包括 `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.kv_cache_utils, ...`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### Module state / 模块级状态 (line 35)
```python
EOS_TOKEN_ID = 50256
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `EOS_TOKEN_ID`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`EOS_TOKEN_ID`。

### mock_kv (lines 38-39)
```python
def mock_kv(matched_tokens: int, is_async: bool):
    return MockKVConfig(matched_tokens=matched_tokens, is_async=is_async)
```
**EN:** Helper function `mock_kv` encapsulates reusable logic for `KV`. Inputs: `matched_tokens, is_async`. Key calls include `MockKVConfig`.
**CN:** 辅助函数 `mock_kv` 封装了与 `kv` 相关的可复用逻辑。 输入参数：`matched_tokens, is_async`。 关键调用包括 `MockKVConfig`。

### create_scheduler (lines 42-170)
```python
def create_scheduler(
    model: str = "facebook/opt-125m",
    max_num_seqs: int = 16,
    max_num_batched_tokens: int = 8192,
    enable_chunked_prefill: bool = True,
    enable_prefix_caching: bool = False,
    long_prefill_token_threshold: int = 0,
    disable_chunked_mm_input: bool = False,
    use_kv_connector: None | bool | str | MockKVConfig = None,
    num_blocks: int = 10000,
    block_size: int = 16,
    max_model_len: int | None = None,
    num_speculative_tokens: int | None = None,
    skip_tokenizer_init: bool = False,
    async_scheduling: bool = False,
    pipeline_parallel_size: int = 1,
    use_ec_connector: bool = False,
    ec_role: str | None = None,
    # ... excerpt omitted for brevity ...
    return scheduler_cls(
        vllm_config=vllm_config,
        kv_cache_config=kv_cache_config,
        block_size=block_size,
        log_stats=True,
        structured_output_manager=StructuredOutputManager(vllm_config),
    )
```
**EN:** Helper function `create_scheduler` encapsulates reusable logic for `scheduler`. Inputs: `model, max_num_seqs, max_num_batched_tokens, enable_chunked_prefill, enable_prefix_caching, long_prefill_token_threshold, disable_chunked_mm_input, use_kv_connector, ...`. Key calls include `ModelConfig, SchedulerConfig, CacheConfig, isinstance, VllmConfig, KVCacheConfig`.
**CN:** 辅助函数 `create_scheduler` 封装了与 `调度器` 相关的可复用逻辑。 输入参数：`model, max_num_seqs, max_num_batched_tokens, enable_chunked_prefill, enable_prefix_caching, long_prefill_token_threshold, disable_chunked_mm_input, use_kv_connector, ...`。 关键调用包括 `ModelConfig, SchedulerConfig, CacheConfig, isinstance, VllmConfig, KVCacheConfig`。

### Module state / 模块级状态 (line 173)
```python
_none_hash_initialized = False
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_none_hash_initialized`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_none_hash_initialized`。

### create_requests (lines 176-262)
```python
def create_requests(
    num_requests: int,
    num_tokens: int = 10,
    mm_hashes_list: list[list[str]] | None = None,
    mm_positions: list[list[PlaceholderRange]] | None = None,
    ignore_eos: bool = False,
    max_tokens: int = 16,
    stop_token_ids: list[int] | None = None,
    prompt_logprobs: int | None = None,
    same_prompt: bool = False,
    block_size: int = 16,
    req_ids: list[str] | None = None,
) -> list[Request]:
    global _none_hash_initialized
    if not _none_hash_initialized:
        init_none_hash(sha256)
        _none_hash_initialized = True

    # ... excerpt omitted for brevity ...
        assert mm_positions is not None, (
        assert len(mm_hashes_list) == len(mm_positions) == num_requests
        assert [len(h) for h in mm_hashes_list] == [len(p) for p in mm_positions]
        assert len(req_ids) == num_requests
                    assert seen_hashes[identifier] == position_length, (
            pooling_params=None,
            mm_features=mm_features if mm_features else None,
            block_hasher=block_hasher,
        )
        requests.append(request)
    return requests
```
**EN:** Helper function `create_requests` encapsulates reusable logic for `requests`. Inputs: `num_requests, num_tokens, mm_hashes_list, mm_positions, ignore_eos, max_tokens, stop_token_ids, prompt_logprobs, ...`. Key calls include `get_request_block_hasher, SamplingParams, sampling_params.update_from_generation_config, range, init_none_hash, enumerate`. It includes 5 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `create_requests` 封装了与 `requests` 相关的可复用逻辑。 输入参数：`num_requests, num_tokens, mm_hashes_list, mm_positions, ignore_eos, max_tokens, stop_token_ids, prompt_logprobs, ...`。 关键调用包括 `get_request_block_hasher, SamplingParams, sampling_params.update_from_generation_config, range, init_none_hash, enumerate`。 其中包含 5 个内部断言，用于保护前置假设。

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
- **EN:** vLLM modules under test: `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.async_scheduler, vllm.v1.core.sched.scheduler, vllm.v1.kv_cache_interface, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.async_scheduler, vllm.v1.core.sched.scheduler, vllm.v1.kv_cache_interface, ...`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.utils`。
