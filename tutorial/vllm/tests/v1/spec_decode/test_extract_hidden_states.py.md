# test_extract_hidden_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/spec_decode/test_extract_hidden_states.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `extract hidden states` behavior and regressions in the v1 stack. / 验证 v1 栈中 `extract hidden states` 相关行为与回归。

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
import json
from unittest import mock

import numpy as np
import pytest
import torch
from transformers import CLIPVisionConfig, LlamaConfig, LlavaConfig, PretrainedConfig

from tests.v1.attention.utils import (
    BatchSpec,
    create_common_attn_metadata,
)
from vllm.config import (
    AttentionConfig,
    CacheConfig,
    DeviceConfig,
    ModelConfig,
    ParallelConfig,
    SchedulerConfig,
    SpeculativeConfig,
    VllmConfig,
)
from vllm.config.load import LoadConfig
from vllm.platforms import current_platform
from vllm.transformers_utils.config import get_hf_text_config
from vllm.transformers_utils.configs.extract_hidden_states import (
    ExtractHiddenStatesConfig,
)
from vllm.v1.spec_decode.extract_hidden_states import ExtractHiddenStatesProposer
from vllm.v1.worker.gpu_input_batch import CachedRequestState, InputBatch
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest, torch, transformers`. vLLM modules under test include `vllm.config, vllm.config.load, vllm.platforms, vllm.transformers_utils.config, vllm.transformers_utils.configs.extract_hidden_states, ...`. Local helpers come from `tests.v1.attention.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest, torch, transformers`。 被测试的 vLLM 模块包括 `vllm.config, vllm.config.load, vllm.platforms, vllm.transformers_utils.config, vllm.transformers_utils.configs.extract_hidden_states, ...`。 本地测试辅助逻辑来自 `tests.v1.attention.utils`。

### Module state / 模块级状态 (lines 35-36)
```python
model_dir = "TinyLlama/TinyLlama-1.1B-Chat-v1.0"
DEVICE_TYPE = current_platform.device_type
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `model_dir, DEVICE_TYPE`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`model_dir, DEVICE_TYPE`。

### _create_proposer (lines 39-76)
```python
def _create_proposer(
    num_speculative_tokens: int = 1,
    layer_ids: list[int] | None = None,
) -> ExtractHiddenStatesProposer:
    """Create an ExtractHiddenStatesProposer for testing."""
    if layer_ids is None:
        layer_ids = [1, 2, 3, 4]

    model_config = ModelConfig(model=model_dir, runner="generate", max_model_len=100)
    speculative_config = SpeculativeConfig(
        target_model_config=model_config,
        target_parallel_config=ParallelConfig(),
        method="extract_hidden_states",
        num_speculative_tokens=num_speculative_tokens,
        draft_model_config={
            "hf_config": {
                "eagle_aux_hidden_state_layer_ids": layer_ids,
    # ... excerpt omitted for brevity ...
            is_encoder_decoder=model_config.is_encoder_decoder,
        ),
        attention_config=AttentionConfig(),
    )
    return ExtractHiddenStatesProposer(vllm_config=vllm_config, device=device)
```
**EN:** Helper function `_create_proposer` encapsulates reusable logic for `proposer`. Inputs: `num_speculative_tokens, layer_ids`. Key calls include `ModelConfig, SpeculativeConfig, VllmConfig, ExtractHiddenStatesProposer, ParallelConfig, CacheConfig`.
**CN:** 辅助函数 `_create_proposer` 封装了与 `proposer` 相关的可复用逻辑。 输入参数：`num_speculative_tokens, layer_ids`。 关键调用包括 `ModelConfig, SpeculativeConfig, VllmConfig, ExtractHiddenStatesProposer, ParallelConfig, CacheConfig`。

### test_proposer_initialization (lines 79-94)
```python
def test_proposer_initialization():
    """Test that the proposer initializes correctly with the right parameters."""
    layer_ids = [1, 2, 3, 4]
    proposer = _create_proposer(num_speculative_tokens=1, layer_ids=layer_ids)

    assert proposer.num_hidden_states == len(layer_ids)
    assert proposer.vllm_config.speculative_config is not None
    assert proposer.vllm_config.speculative_config.num_speculative_tokens == 1

    # Verify the hidden states buffer is correctly shaped
    expected_shape = (
        proposer.max_num_tokens,
        len(layer_ids),
        proposer.hidden_size,
    )
    assert proposer.hidden_states.shape == expected_shape
```
**EN:** Test case covering `proposer initialization`. It exercises `_create_proposer, len`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `proposer initialization` 的测试用例。 该测试会调用 `_create_proposer, len`。 代码主体包含 4 个显式断言。

### test_proposer_initialization_missing_layer_ids (lines 97-129)
```python
def test_proposer_initialization_missing_layer_ids():
    """Test that initialization fails when layer_ids are not provided."""
    model_config = ModelConfig(model=model_dir, runner="generate", max_model_len=100)

    speculative_config = SpeculativeConfig(
        target_model_config=model_config,
        target_parallel_config=ParallelConfig(),
        method="extract_hidden_states",
        num_speculative_tokens=1,
        draft_model_config={
            "hf_config": {}  # Missing eagle_aux_hidden_state_layer_ids
        },
    )
    device = DEVICE_TYPE
    vllm_config = VllmConfig(
        model_config=model_config,
        cache_config=CacheConfig(),
    # ... excerpt omitted for brevity ...
    with pytest.raises(
        ValueError, match="eagle_aux_hidden_state_layer_ids must be set"
    ):
        ExtractHiddenStatesProposer(vllm_config=vllm_config, device=device)
```
**EN:** Test case covering `proposer initialization missing layer ids`. It exercises `ModelConfig, SpeculativeConfig, VllmConfig, pytest.raises, ExtractHiddenStatesProposer, ParallelConfig`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `proposer initialization missing layer ids` 的测试用例。 该测试会调用 `ModelConfig, SpeculativeConfig, VllmConfig, pytest.raises, ExtractHiddenStatesProposer, ParallelConfig`。 主要通过预期异常检查来完成验证。

### test_prepare_next_token_ids_padded (lines 132-195)
```python
def test_prepare_next_token_ids_padded():
    """
    Test for prepare_next_token_ids_padded with extract_hidden_states.

    Since num_speculative_tokens == 1, sampled_token_ids has shape (batch_size, 1).
    For each request we either use the sampled token (if valid and not discarded)
    or a backup token from the request state.
    device = torch.device(DEVICE_TYPE)
    num_requests = 4
    req_ids = [f"req_{i + 1}" for i in range(num_requests)]
    mock_input_batch = mock.MagicMock(spec=InputBatch)
    mock_input_batch.req_ids = req_ids
    mock_input_batch.num_reqs = num_requests
    mock_input_batch.vocab_size = 100
    mock_input_batch.num_tokens_no_spec = np.array([5] * num_requests)
    # ... excerpt omitted for brevity ...
        mock_request = mock.MagicMock(spec=CachedRequestState)
    expected_next_token_ids_cpu = [1, 4, 30, 40]
    expected_next_token_ids_tensor = torch.tensor(
        expected_next_token_ids_cpu, dtype=torch.int32, device=device
    expected_valid_sampled_tokens_count = torch.tensor(
        mock_input_batch,
        discarded_req_mask,
    )
    assert torch.equal(next_token_ids, expected_next_token_ids_tensor)
    assert torch.equal(valid_sampled_tokens_count, expected_valid_sampled_tokens_count)
```
**EN:** Test case covering `prepare next token ids padded`. It exercises `torch.device, mock.MagicMock, np.array, torch.tensor, _create_proposer, proposer.prepare_next_token_ids_padded`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `prepare next token ids padded` 的测试用例。 该测试会调用 `torch.device, mock.MagicMock, np.array, torch.tensor, _create_proposer, proposer.prepare_next_token_ids_padded`。 代码主体包含 2 个显式断言。

### test_propose (lines 198-277)
```python
def test_propose():
    """
    Test the propose() method of ExtractHiddenStatesProposer.

    This should:
    1. Accept target hidden states and sampled token IDs
    2. Return the sampled tokens as "draft" tokens (shape [batch_size, 1])
    3. Cache the hidden states in the model's KV cache
    device = torch.device(DEVICE_TYPE)
    # Setup test parameters
    batch_size = 2
    num_tokens = 5
    num_hidden_layers = 4
    proposer = _create_proposer(
        num_speculative_tokens=1, layer_ids=list(range(num_hidden_layers))
    # ... excerpt omitted for brevity ...
    model_mock = mock.MagicMock()
    mock_attn_metadata = mock.MagicMock()
    mock_attn_metadata_builder = mock.MagicMock()
    assert draft_tokens.shape == (batch_size, 1)
    assert torch.equal(draft_tokens, sampled_token_ids)
    # Verify hidden states were copied to the buffer The stacked hidden states
    # should have shape [num_tokens, num_hidden_layers, hidden_size]
    expected_stacked = torch.stack(target_hidden_states, dim=1)
    assert torch.allclose(
        proposer.hidden_states[:num_tokens], expected_stacked, atol=1e-6
    )
```
**EN:** Test case covering `propose`. It exercises `torch.device, _create_proposer, mock.MagicMock, BatchSpec, create_common_attn_metadata, tensor.unsqueeze`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `propose` 的测试用例。 该测试会调用 `torch.device, _create_proposer, mock.MagicMock, BatchSpec, create_common_attn_metadata, tensor.unsqueeze`。 代码主体包含 3 个显式断言。

### test_propose_different_layer_counts (lines 281-331)
```python
def test_propose_different_layer_counts(num_hidden_layers):
    """Test that propose works correctly with different numbers of hidden layers."""
    device = torch.device(DEVICE_TYPE)

    batch_size = 2
    num_tokens = 5
    proposer = _create_proposer(
        num_speculative_tokens=1, layer_ids=list(range(num_hidden_layers))
    )
    hidden_size = proposer.hidden_size
    # Setup mocks
    model_mock = mock.MagicMock()
    proposer.model = model_mock
    proposer.attn_layer_names = ["cache_only_layers.28"]
    mock_attn_metadata_builder = mock.MagicMock()
    # ... excerpt omitted for brevity ...
    mock_attn_metadata_builder.build_for_drafting.return_value = mock.MagicMock()
        common_attn_metadata=common_attn_metadata,
        slot_mappings=None,
    assert draft_tokens.shape == (batch_size, 1)
    assert torch.equal(draft_tokens, sampled_token_ids)
```
**EN:** Parameterized test covering `propose different layer counts`. Parameter axes: `num_hidden_layers`. Inputs/fixtures: `num_hidden_layers`. It exercises `mark.parametrize, torch.device, _create_proposer, mock.MagicMock, BatchSpec, create_common_attn_metadata`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `propose different layer counts` 的测试用例。 参数维度：`num_hidden_layers`。 输入或 fixture：`num_hidden_layers`。 该测试会调用 `mark.parametrize, torch.device, _create_proposer, mock.MagicMock, BatchSpec, create_common_attn_metadata`。 代码主体包含 2 个显式断言。

### _DummyVLMConfig (lines 339-354)
```python
class _DummyVLMConfig(PretrainedConfig):
    """Minimal composite config that mimics VLMs like Kimi-K2.5 or LLaVA.

    The text model's parameters (hidden_size, num_attention_heads, …) live
    exclusively under ``text_config``; the top-level config has none of them.
    """

    model_type = "test_vlm"

    def __init__(self, text_config: PretrainedConfig, **kwargs):
        self.text_config = text_config
        super().__init__(architectures=["LlamaForCausalLM"], **kwargs)

    def get_text_config(self, decoder: bool = False) -> PretrainedConfig:
        del decoder
        return self.text_config
```
**EN:** Class `_DummyVLMConfig` groups 0 test method(s) and 2 helper/fixture method(s). Bases: `PretrainedConfig`.
**CN:** 类 `_DummyVLMConfig` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。 基类：`PretrainedConfig`。

### test_extract_hidden_states_text_only_config_regression (lines 357-381)
```python
def test_extract_hidden_states_text_only_config_regression():
    """Text-only models (no nested text_config) must keep working."""
    model_config = ModelConfig(model=model_dir, runner="generate", max_model_len=100)

    speculative_config = SpeculativeConfig(
        target_model_config=model_config,
        target_parallel_config=ParallelConfig(),
        method="extract_hidden_states",
        num_speculative_tokens=1,
        draft_model_config={
            "hf_config": {
                "eagle_aux_hidden_state_layer_ids": [1, 2, 3, 4],
            }
        },
    )

    assert speculative_config.draft_model_config is not None
    # For text-only models, hf_text_config should be the config itself.
    assert speculative_config.draft_model_config.hf_text_config is (
        speculative_config.draft_model_config.hf_config
    )
    assert (
        speculative_config.draft_model_config.hf_text_config.num_attention_heads
        == model_config.hf_text_config.num_attention_heads
    )
```
**EN:** Test case covering `extract hidden states text only config regression`. It exercises `ModelConfig, SpeculativeConfig, ParallelConfig`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `extract hidden states text only config regression` 的测试用例。 该测试会调用 `ModelConfig, SpeculativeConfig, ParallelConfig`。 代码主体包含 3 个显式断言。

### test_extract_hidden_states_config_preserves_vlm_text_config (lines 384-424)
```python
def test_extract_hidden_states_config_preserves_vlm_text_config():
    """A real VLM config (LLaVA) with nested text_config must be preserved."""
    text_config = LlamaConfig(
        vocab_size=32000,
        hidden_size=128,
        intermediate_size=256,
        num_hidden_layers=2,
        num_attention_heads=8,
    )
    vlm_config = LlavaConfig(
        vision_config=CLIPVisionConfig(),
        text_config=text_config,

    # Precondition: to_dict() flattens the nested config to a plain dict.
    assert isinstance(vlm_config.to_dict()["text_config"], dict)
    extract_config = ExtractHiddenStatesConfig(
    # ... excerpt omitted for brevity ...
    assert isinstance(extract_config.text_config, LlamaConfig)
    assert extracted is extract_config.text_config
    assert extracted.num_attention_heads == text_config.num_attention_heads
    assert extracted.hidden_size == text_config.hidden_size
    assert isinstance(serialized["text_config"], dict)
    assert serialized["text_config"]["num_attention_heads"] == (
    json_str = json.loads(extract_config.to_json_string())
    assert json_str["text_config"]["num_attention_heads"] == (
        text_config.num_attention_heads
```
**EN:** Test case covering `extract hidden states config preserves vlm text config`. It exercises `LlamaConfig, LlavaConfig, isinstance, ExtractHiddenStatesConfig, get_hf_text_config, extract_config.to_dict`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `extract hidden states config preserves vlm text config` 的测试用例。 该测试会调用 `LlamaConfig, LlavaConfig, isinstance, ExtractHiddenStatesConfig, get_hf_text_config, extract_config.to_dict`。 代码主体包含 8 个显式断言。

### test_extract_hidden_states_speculative_config_vlm (lines 427-471)
```python
def test_extract_hidden_states_speculative_config_vlm():
    """SpeculativeConfig with a VLM target must build without errors."""
    nested_text_config = LlamaConfig(
        vocab_size=32000,
        hidden_size=128,
        intermediate_size=256,
        num_hidden_layers=2,
        num_attention_heads=8,
    )

    target_model_config = ModelConfig(
        model=model_dir,
        runner="generate",
        max_model_len=100,
    # Replace the real text-only config with our composite VLM config.
    target_model_config.hf_config = _DummyVLMConfig(
        text_config=nested_text_config,
    # ... excerpt omitted for brevity ...
    assert speculative_config.draft_model_config is not None
    assert isinstance(
    assert speculative_config.draft_model_config.hf_text_config is (
        speculative_config.draft_model_config.hf_config.text_config
    assert (
        speculative_config.draft_model_config.hf_text_config.num_attention_heads
        == nested_text_config.num_attention_heads
```
**EN:** Test case covering `extract hidden states speculative config vlm`. It exercises `LlamaConfig, ModelConfig, _DummyVLMConfig, SpeculativeConfig, isinstance, ParallelConfig`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `extract hidden states speculative config vlm` 的测试用例。 该测试会调用 `LlamaConfig, ModelConfig, _DummyVLMConfig, SpeculativeConfig, isinstance, ParallelConfig`。 代码主体包含 4 个显式断言。

### test_extract_hidden_states_config_invalid_text_config (lines 474-488)
```python
def test_extract_hidden_states_config_invalid_text_config():
    """A nested text_config missing required attrs must still be rejected."""
    broken_text_config = PretrainedConfig(hidden_size=128)
    vlm_config = _DummyVLMConfig(text_config=broken_text_config)

    extract_config = ExtractHiddenStatesConfig(
        vlm_config,
        eagle_aux_hidden_state_layer_ids=[1],
    )

    # The object is preserved (not flattened), …
    assert extract_config.text_config is broken_text_config
    # … but validation still rejects the missing attribute.
    with pytest.raises(ValueError, match="num_attention_heads"):
        get_hf_text_config(extract_config)
```
**EN:** Test case covering `extract hidden states config invalid text config`. It exercises `PretrainedConfig, _DummyVLMConfig, ExtractHiddenStatesConfig, pytest.raises, get_hf_text_config`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `extract hidden states config invalid text config` 的测试用例。 该测试会调用 `PretrainedConfig, _DummyVLMConfig, ExtractHiddenStatesConfig, pytest.raises, get_hf_text_config`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest, torch, transformers`.
- **CN:** 外部库：`numpy, pytest, torch, transformers`。
- **EN:** vLLM modules under test: `vllm.config, vllm.config.load, vllm.platforms, vllm.transformers_utils.config, vllm.transformers_utils.configs.extract_hidden_states, vllm.v1.spec_decode.extract_hidden_states, vllm.v1.worker.gpu_input_batch`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.config.load, vllm.platforms, vllm.transformers_utils.config, vllm.transformers_utils.configs.extract_hidden_states, vllm.v1.spec_decode.extract_hidden_states, vllm.v1.worker.gpu_input_batch`。
- **EN:** Local test helpers: `tests.v1.attention.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.attention.utils`。
- **EN:** Standard-library support: `json, unittest`.
- **CN:** 标准库支持：`json, unittest`。
