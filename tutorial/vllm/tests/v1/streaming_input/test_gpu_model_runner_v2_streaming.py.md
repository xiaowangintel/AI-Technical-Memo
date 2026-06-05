# test_gpu_model_runner_v2_streaming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/streaming_input/test_gpu_model_runner_v2_streaming.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for MRv2 GPUModelRunner.add_requests streaming input support. / 该文件的文档字符串表明其用途：`unit tests for mrv2 gpumodelrunner.add_requests streaming input support`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 4)
```python
"""Unit tests for MRv2 GPUModelRunner.add_requests streaming input support."""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for MRv2 GPUModelRunner.add_requests streaming input support.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for mrv2 gpumodelrunner.add_requests streaming input support`。

### Imports and setup / 导入与设置 (lines 6-17)
```python
from unittest.mock import Mock

import pytest
import torch

from vllm.v1.core.sched.output import (
    CachedRequestData,
    NewRequestData,
    SchedulerOutput,
)
from vllm.v1.worker.gpu.model_runner import GPUModelRunner
from vllm.v1.worker.gpu.states import RequestState
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.v1.core.sched.output, vllm.v1.worker.gpu.model_runner, vllm.v1.worker.gpu.states`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.v1.core.sched.output, vllm.v1.worker.gpu.model_runner, vllm.v1.worker.gpu.states`。

### Module state / 模块级状态 (line 19)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### mock_model_runner_with_req_states (lines 23-51)
```python
def mock_model_runner_with_req_states():
    """Create a mock MRv2 GPUModelRunner with a real RequestState."""

    runner = Mock(spec=GPUModelRunner)
    runner.req_states = RequestState(
        max_num_reqs=10,
        max_model_len=1024,
        max_num_batched_tokens=1024,
        num_speculative_steps=0,
        vocab_size=32000,
        device=torch.device("cpu"),
        model_dtype=torch.float32,
        cache_draft_logits=False,
    )
    runner.encoder_cache = None
    runner.model_state = Mock()
    runner.block_tables = Mock()
    runner.lora_state = Mock()
    runner.sampler = None
    runner.prompt_logprobs_worker = None
    runner.is_last_pp_rank = False

    # Mock staged writes — they use Triton kernels that require GPU
    runner.req_states.apply_staged_writes = Mock()

    # Bind the real methods to our mock
    runner._remove_request = GPUModelRunner._remove_request.__get__(runner)
    runner.add_requests = GPUModelRunner.add_requests.__get__(runner)
    return runner
```
**EN:** Fixture/helper `mock_model_runner_with_req_states` prepares reusable state for downstream tests. Key calls include `Mock, RequestState, _remove_request.__get__, add_requests.__get__, torch.device`.
**CN:** `mock_model_runner_with_req_states` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `Mock, RequestState, _remove_request.__get__, add_requests.__get__, torch.device`。

### _make_scheduler_output (lines 54-65)
```python
def _make_scheduler_output(new_reqs):
    return SchedulerOutput(
        scheduled_new_reqs=new_reqs,
        scheduled_cached_reqs=CachedRequestData.make_empty(),
        num_scheduled_tokens={},
        total_num_scheduled_tokens=0,
        scheduled_spec_decode_tokens={},
        scheduled_encoder_inputs={},
        num_common_prefix_blocks=[],
        finished_req_ids=set(),
        free_encoder_mm_hashes=[],
    )
```
**EN:** Helper function `_make_scheduler_output` encapsulates reusable logic for `scheduler output`. Inputs: `new_reqs`. Key calls include `SchedulerOutput, CachedRequestData.make_empty, set`.
**CN:** 辅助函数 `_make_scheduler_output` 封装了与 `调度器 output` 相关的可复用逻辑。 输入参数：`new_reqs`。 关键调用包括 `SchedulerOutput, CachedRequestData.make_empty, set`。

### test_e2e_streaming_request_update_basic_flow (lines 68-133)
```python
def test_e2e_streaming_request_update_basic_flow(
    mock_model_runner_with_req_states,
):
    """Test that streaming sessions are updated correctly.

    This test validates that when a streaming session is updated with new
    prompt tokens:
    1. The old request state is removed (no free_indices leak)
    2. The new state is written with updated prefill_token_ids
    3. model_state and block_tables are re-registered for the new state
    """
    runner = mock_model_runner_with_req_states
    req_states = runner.req_states
    req_id = "streaming_req_0"
    initial_free = len(req_states.free_indices)
    # Step 1: Add initial request with 3 prompt tokens, all computed
    initial_req_data = NewRequestData(
    # ... excerpt omitted for brevity ...
    assert req_id in req_states.req_id_to_index
    assert len(req_states.free_indices) == initial_free - 1
    assert sum(1 for v in req_states.index_to_req_id.values() if v == req_id) == 1
    assert req_states.prompt_len.np[new_idx] == 3
    # Verify model_state and block_tables were re-registered
    runner.model_state.add_request.assert_called_with(new_idx, updated_req_data)
    runner.block_tables.append_block_ids.assert_called_with(
        new_idx, ([0, 1],), overwrite=True
    )
```
**EN:** Test case covering `end-to-end streaming request update basic flow`. Inputs/fixtures: `mock_model_runner_with_req_states`. It exercises `len, NewRequestData, runner.add_requests, add_request.assert_called_with, append_block_ids.assert_called_with, _make_scheduler_output`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `end-to-end streaming request update basic flow` 的测试用例。 输入或 fixture：`mock_model_runner_with_req_states`。 该测试会调用 `len, NewRequestData, runner.add_requests, add_request.assert_called_with, append_block_ids.assert_called_with, _make_scheduler_output`。 代码主体包含 8 个显式断言。

### test_e2e_streaming_with_multimodal_features (lines 136-207)
```python
def test_e2e_streaming_with_multimodal_features(
    mock_model_runner_with_req_states,
):
    """Test that streaming sessions with multimodal features are updated.

    This test validates that when a streaming session with mm features
    is updated:
    1. The old request state is removed (no free_indices leak)
    2. encoder_cache is cleaned up and re-registered with new mm_features
    3. model_state is re-registered (recomputes M-RoPE positions etc.)
    """
    runner = mock_model_runner_with_req_states
    req_states = runner.req_states
    req_id = "streaming_mm_req_0"
    initial_free = len(req_states.free_indices)
    # Enable encoder_cache for multimodal
    runner.encoder_cache = Mock()
    # ... excerpt omitted for brevity ...
    mm_feature_1 = Mock()
    assert req_id in req_states.req_id_to_index
    mm_feature_2 = Mock()
    assert len(req_states.free_indices) == initial_free - 1
    assert sum(1 for v in req_states.index_to_req_id.values() if v == req_id) == 1
    # Verify model_state was re-registered with new data
    new_idx = req_states.req_id_to_index[req_id]
    runner.model_state.add_request.assert_called_once_with(new_idx, updated_req_data)
    # Verify updated prefill length
    assert req_states.prefill_len.np[new_idx] == 21
```
**EN:** Test case covering `end-to-end streaming with multimodal features`. Inputs/fixtures: `mock_model_runner_with_req_states`. It exercises `len, Mock, NewRequestData, runner.add_requests, encoder_cache.reset_mock, model_state.reset_mock`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `end-to-end streaming with multimodal features` 的测试用例。 输入或 fixture：`mock_model_runner_with_req_states`。 该测试会调用 `len, Mock, NewRequestData, runner.add_requests, encoder_cache.reset_mock, model_state.reset_mock`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.v1.core.sched.output, vllm.v1.worker.gpu.model_runner, vllm.v1.worker.gpu.states`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.core.sched.output, vllm.v1.worker.gpu.model_runner, vllm.v1.worker.gpu.states`。
- **EN:** Standard-library support: `unittest.mock`.
- **CN:** 标准库支持：`unittest.mock`。
