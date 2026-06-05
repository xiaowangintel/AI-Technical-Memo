# test_gpu_model_runner_streaming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/streaming_input/test_gpu_model_runner_streaming.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for GPUModelRunner._update_streaming_request function. / 该文件的文档字符串表明其用途：`unit tests for gpumodelrunner._update_streaming_request function`。

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
"""Unit tests for GPUModelRunner._update_streaming_request function."""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for GPUModelRunner._update_streaming_request function.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for gpumodelrunner._update_streaming_request function`。

### Imports and setup / 导入与设置 (lines 6-17)
```python
from unittest.mock import Mock

import pytest

from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalKwargsItem,
    PlaceholderRange,
)
from vllm.sampling_params import SamplingParams
from vllm.v1.worker.gpu_input_batch import CachedRequestState, InputBatch
from vllm.v1.worker.gpu_model_runner import GPUModelRunner
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.multimodal.inputs, vllm.sampling_params, vllm.v1.worker.gpu_input_batch, vllm.v1.worker.gpu_model_runner`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.multimodal.inputs, vllm.sampling_params, vllm.v1.worker.gpu_input_batch, vllm.v1.worker.gpu_model_runner`。

### Module state / 模块级状态 (line 19)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### mock_model_runner_with_input_batch (lines 23-45)
```python
def mock_model_runner_with_input_batch():
    """Create a mock GPUModelRunner with a real InputBatch for e2e testing."""

    runner = Mock(spec=GPUModelRunner)
    runner.uses_mrope = False
    runner.requests = {}
    runner.max_num_reqs = 10
    runner.max_model_len = 1024

    # Create a real InputBatch for e2e testing
    runner.input_batch = InputBatch(
        max_num_reqs=10,
        max_model_len=1024,
        max_num_batched_tokens=1024,
        device="cpu",
        pin_memory=False,
        vocab_size=32000,
        block_sizes=[16],
        kernel_block_sizes=[16],
        logitsprocs=None,
        is_pooling_model=False,
    )
    return runner
```
**EN:** Fixture/helper `mock_model_runner_with_input_batch` prepares reusable state for downstream tests. Key calls include `Mock, InputBatch`.
**CN:** `mock_model_runner_with_input_batch` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `Mock, InputBatch`。

### test_e2e_streaming_request_update_basic_flow (lines 48-116)
```python
def test_e2e_streaming_request_update_basic_flow(mock_model_runner_with_input_batch):
    """Test that streaming session are updated correctly.

    This test validates that when a streaming session is updated with new prompt tokens:
    1. The request is removed from InputBatch before updating (avoids duplication)
    2. Request state fields are updated correctly
    3. output_token_ids is cleared (intermediate outputs are now in prompt_token_ids)
    """
    runner = mock_model_runner_with_input_batch
    req_id = "streaming_req_0"
    # Step 1: Create initial request state with some computed tokens
    initial_req_state = CachedRequestState(
        req_id=req_id,
        prompt_token_ids=[1, 2, 3],
        mm_features=[],
        sampling_params=SamplingParams(temperature=0.5),
        pooling_params=None,
    # ... excerpt omitted for brevity ...
    assert req_id in runner.input_batch.req_id_to_index
    new_req_data = Mock()
    assert updated_req_state.prompt_token_ids == [1, 2, 3, 10, 4, 5]
    assert updated_req_state.num_computed_tokens == 4
    assert updated_req_state.sampling_params.temperature == 0.8
    assert updated_req_state.sampling_params.max_tokens == 50
    # Verify the same object is returned
    assert runner.requests[req_id] is updated_req_state
    # Verify request was removed from InputBatch during update (avoids duplication)
    assert req_id not in runner.input_batch.req_id_to_index
```
**EN:** Test case covering `end-to-end streaming request update basic flow`. Inputs/fixtures: `mock_model_runner_with_input_batch`. It exercises `CachedRequestState, input_batch.add_request, Mock, SamplingParams, GPUModelRunner._update_streaming_request`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `end-to-end streaming request update basic flow` 的测试用例。 输入或 fixture：`mock_model_runner_with_input_batch`。 该测试会调用 `CachedRequestState, input_batch.add_request, Mock, SamplingParams, GPUModelRunner._update_streaming_request`。 代码主体包含 9 个显式断言。

### test_e2e_streaming_with_multimodal_features (lines 119-209)
```python
def test_e2e_streaming_with_multimodal_features(mock_model_runner_with_input_batch):
    """Test that streaming session with multimodal features are updated correctly.

    This test validates that when a streaming session with mm features is updated:
    1. The request is removed from InputBatch before updating (avoids duplication)
    2. Multimodal features from both requests are preserved and merged correctly
    3. New prompt tokens (including intermediate outputs) are appended correctly
    4. output_token_ids is cleared (intermediate outputs are now in prompt_token_ids)
    """
    runner = mock_model_runner_with_input_batch
    req_id = "streaming_mm_req_0"
    # Step 1: Create initial request state with one multimodal feature
    mm_feature_1 = MultiModalFeatureSpec(
        data=MultiModalKwargsItem.dummy(),
        modality="audio",
        identifier="audio_1",
        mm_position=PlaceholderRange(offset=2, length=10),
    # ... excerpt omitted for brevity ...
    assert req_id in runner.input_batch.req_id_to_index
    new_req_data = Mock()
    assert len(updated_req_state.mm_features) == 2
    assert updated_req_state.mm_features[0] == mm_feature_1
    assert updated_req_state.mm_features[1] == mm_feature_2
    assert len(updated_req_state.prompt_token_ids) == 21
    # Verify the same object is returned
    assert runner.requests[req_id] is updated_req_state
    # Verify request was removed from InputBatch during update (avoids duplication)
    assert req_id not in runner.input_batch.req_id_to_index
```
**EN:** Test case covering `end-to-end streaming with multimodal features`. Inputs/fixtures: `mock_model_runner_with_input_batch`. It exercises `MultiModalFeatureSpec, CachedRequestState, input_batch.add_request, Mock, SamplingParams, GPUModelRunner._update_streaming_request`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `end-to-end streaming with multimodal features` 的测试用例。 输入或 fixture：`mock_model_runner_with_input_batch`。 该测试会调用 `MultiModalFeatureSpec, CachedRequestState, input_batch.add_request, Mock, SamplingParams, GPUModelRunner._update_streaming_request`。 代码主体包含 13 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设
- **EN:** Multimodal input bookkeeping
- **CN:** 多模态输入管理
- **EN:** End-to-end integration rather than isolated units
- **CN:** 端到端集成而非孤立单元

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.multimodal.inputs, vllm.sampling_params, vllm.v1.worker.gpu_input_batch, vllm.v1.worker.gpu_model_runner`.
- **CN:** 被测试的 vLLM 模块：`vllm.multimodal.inputs, vllm.sampling_params, vllm.v1.worker.gpu_input_batch, vllm.v1.worker.gpu_model_runner`。
- **EN:** Standard-library support: `unittest.mock`.
- **CN:** 标准库支持：`unittest.mock`。
