# test_scheduler_streaming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/streaming_input/test_scheduler_streaming.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `scheduler streaming` behavior and regressions in the v1 stack. / 验证 v1 栈中 `调度器 streaming` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-25)
```python
import unittest
from unittest.mock import MagicMock

import torch

from vllm.config import DeviceConfig, VllmConfig
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalKwargsItem,
    PlaceholderRange,
)
from vllm.sampling_params import SamplingParams
from vllm.v1.core.sched.scheduler import Scheduler
from vllm.v1.engine import FinishReason
from vllm.v1.kv_cache_interface import (
    FullAttentionSpec,
    KVCacheConfig,
    KVCacheGroupSpec,
)
from vllm.v1.outputs import ModelRunnerOutput
from vllm.v1.request import Request, RequestStatus, StreamingUpdate
from vllm.v1.structured_output import StructuredOutputManager
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `torch`. vLLM modules under test include `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.v1.core.sched.scheduler, vllm.v1.engine, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `torch`。 被测试的 vLLM 模块包括 `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.v1.core.sched.scheduler, vllm.v1.engine, ...`。

### Module state / 模块级状态 (line 27)
```python
STOP_TOKEN = 128001
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `STOP_TOKEN`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`STOP_TOKEN`。

### DummyRequest (lines 30-48)
```python
class DummyRequest(Request):
    def __init__(
        self,
        request_id,
        resumable=True,
        prompt_token_ids=None,
        mm_features: list[MultiModalFeatureSpec] | None = None,
        max_tokens: int | None = 16,
    ):
        super().__init__(
            request_id=request_id,
            prompt_token_ids=prompt_token_ids if prompt_token_ids is not None else [],
            sampling_params=SamplingParams(
                stop_token_ids=[STOP_TOKEN], max_tokens=max_tokens
            ),
            pooling_params=None,
            mm_features=mm_features,
            resumable=resumable,
        )
```
**EN:** Class `DummyRequest` groups 0 test method(s) and 1 helper/fixture method(s). Bases: `Request`.
**CN:** 类 `DummyRequest` 组织了 0 个测试方法，以及 1 个辅助或 fixture 方法。 基类：`Request`。

### create_scheduler (lines 51-80)
```python
def create_scheduler() -> Scheduler:
    vllm_config = VllmConfig(device_config=DeviceConfig("cpu"))
    vllm_config.model_config = MagicMock()
    vllm_config.model_config.skip_tokenizer_init = True
    vllm_config.model_config.is_multimodal_model = False
    vllm_config.model_config.max_model_len = 1024
    vllm_config.model_config.enable_return_routed_experts = False
    vllm_config.cache_config = MagicMock()
    vllm_config.cache_config.num_gpu_blocks = 1000
    vllm_config.cache_config.enable_prefix_caching = False
    kv_cache_config = KVCacheConfig(
        num_blocks=1000,
        kv_cache_tensors=[],
        kv_cache_groups=[
            KVCacheGroupSpec(
                ["layer"],
                FullAttentionSpec(
                    block_size=16, num_kv_heads=1, head_size=1, dtype=torch.float32
                ),
            )
        ],
    )
    return Scheduler(
        vllm_config=vllm_config,
        kv_cache_config=kv_cache_config,
        log_stats=True,
        structured_output_manager=StructuredOutputManager(vllm_config),
        block_size=16,
        hash_block_size=16,
    )
```
**EN:** Helper function `create_scheduler` encapsulates reusable logic for `scheduler`. Key calls include `VllmConfig, MagicMock, KVCacheConfig, Scheduler, DeviceConfig, StructuredOutputManager`.
**CN:** 辅助函数 `create_scheduler` 封装了与 `调度器` 相关的可复用逻辑。 关键调用包括 `VllmConfig, MagicMock, KVCacheConfig, Scheduler, DeviceConfig, StructuredOutputManager`。

### TestStreamingScheduler (lines 83-575)
```python
class TestStreamingScheduler(unittest.TestCase):
    def test_add_request(self):
        scheduler = create_scheduler()

        request = DummyRequest(
            request_id="test_request",
            resumable=True,
        )
        scheduler.add_request(request)
        assert "test_request" in scheduler.requests
        assert request.status == RequestStatus.WAITING
        assert len(scheduler.waiting) == 1
        next_request = DummyRequest(
    # ... excerpt omitted for brevity ...
        assert next_request.status == RequestStatus.WAITING
        assert len(scheduler.requests["test_request"].streaming_queue) == 1
        assert session.sampling_params.max_tokens == 10
        assert session.max_tokens == 10
        # Both cached states must be independent of each other
        assert (
            cached_state_cycle1["prompt_token_ids"]
            is not cached_state_cycle3["prompt_token_ids"]
        ), "Cached states from different cycles should be independent objects."
```
**EN:** Class `TestStreamingScheduler` groups 8 test method(s). Bases: `unittest.TestCase`. Representative scenarios: `test_add_request, test_update_request_as_session_max_token, test_update_request_as_session, test_update_request_as_session_with_multimodal, test_process_streaming_requests_with_finish_session, test_streaming_request_session_update, ...`.
**CN:** 类 `TestStreamingScheduler` 组织了 8 个测试方法。 基类：`unittest.TestCase`。 代表性场景：`test_add_request, test_update_request_as_session_max_token, test_update_request_as_session, test_update_request_as_session_with_multimodal, test_process_streaming_requests_with_finish_session, test_streaming_request_session_update, ...`。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化

## Dependencies / 依赖关系
- **EN:** External libraries: `torch`.
- **CN:** 外部库：`torch`。
- **EN:** vLLM modules under test: `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.v1.core.sched.scheduler, vllm.v1.engine, vllm.v1.kv_cache_interface, vllm.v1.outputs, vllm.v1.request, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.v1.core.sched.scheduler, vllm.v1.engine, vllm.v1.kv_cache_interface, vllm.v1.outputs, vllm.v1.request, ...`。
- **EN:** Standard-library support: `unittest, unittest.mock`.
- **CN:** 标准库支持：`unittest, unittest.mock`。
