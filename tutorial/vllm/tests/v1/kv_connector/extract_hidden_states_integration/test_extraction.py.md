# test_extraction.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/extract_hidden_states_integration/test_extraction.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `extraction` behavior and regressions in the v1 stack. / 验证 v1 栈中 `extraction` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-11)
```python
import gc
import os

import pytest
import torch
from safetensors import safe_open

from vllm import LLM, ModelRegistry, SamplingParams
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, safetensors, transformers`. vLLM modules under test include `vllm`. Local helpers come from `tests.v1.kv_connector.extract_hidden_states_integration.predictable_llama`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, safetensors, transformers`。 被测试的 vLLM 模块包括 `vllm`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.extract_hidden_states_integration.predictable_llama`。

### get_and_check_output (lines 14-38)
```python
def get_and_check_output(output, expected_shape):
    assert output.kv_transfer_params is not None
    hidden_states_path = output.kv_transfer_params.get("hidden_states_path")
    assert hidden_states_path is not None
    assert os.path.exists(hidden_states_path)

    # Load and verify the saved tensors
    with safe_open(hidden_states_path, "pt") as f:
        # Check that token_ids and hidden_states are present
        tensor_names = f.keys()
        assert "token_ids" in tensor_names
        assert "hidden_states" in tensor_names

        token_ids = f.get_tensor("token_ids")
        hidden_states = f.get_tensor("hidden_states")

        prompt_token_ids = output.prompt_token_ids
        assert torch.equal(token_ids, torch.tensor(prompt_token_ids))

        assert hidden_states.shape == expected_shape

        # Verify hidden_states are not all zeros (i.e., they were actually computed)
        assert not torch.allclose(hidden_states, torch.zeros_like(hidden_states))

    return token_ids, hidden_states
```
**EN:** Helper function `get_and_check_output` encapsulates reusable logic for `and check output`. Inputs: `output, expected_shape`. Key calls include `kv_transfer_params.get, path.exists, safe_open, f.keys, f.get_tensor, torch.equal`. It includes 8 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `get_and_check_output` 封装了与 `and check output` 相关的可复用逻辑。 输入参数：`output, expected_shape`。 关键调用包括 `kv_transfer_params.get, path.exists, safe_open, f.keys, f.get_tensor, torch.equal`。 其中包含 8 个内部断言，用于保护前置假设。

### predictable_llama_config_path (lines 42-70)
```python
def predictable_llama_config_path(tmp_path_factory):
    """Create a minimal LlamaConfig for PredictableLlamaForCausalLM."""
    from transformers import LlamaConfig, LlamaTokenizerFast

    config_dir = tmp_path_factory.mktemp("predictable_llama")

    # Create a minimal Llama config with small dimensions
    config = LlamaConfig(
        vocab_size=1000,
        hidden_size=256,
        intermediate_size=512,
        num_hidden_layers=24,  # Enough layers to test various layer_ids
        num_attention_heads=4,
        num_key_value_heads=4,
        max_position_embeddings=128,
        architectures=["PredictableLlamaForCausalLM"],
    )

    # Save config
    config.save_pretrained(config_dir)

    # Create a simple tokenizer
    tokenizer = LlamaTokenizerFast.from_pretrained(
        "TinyLlama/TinyLlama-1.1B-Chat-v1.0",
        cache_dir=os.path.expanduser("~/.cache/huggingface"),
    )
    tokenizer.save_pretrained(config_dir)

    return str(config_dir)
```
**EN:** Fixture/helper `predictable_llama_config_path` prepares reusable state for downstream tests. Inputs: `tmp_path_factory`. Key calls include `pytest.fixture, tmp_path_factory.mktemp, LlamaConfig, config.save_pretrained, LlamaTokenizerFast.from_pretrained, tokenizer.save_pretrained`.
**CN:** `predictable_llama_config_path` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`tmp_path_factory`。 关键调用包括 `pytest.fixture, tmp_path_factory.mktemp, LlamaConfig, config.save_pretrained, LlamaTokenizerFast.from_pretrained, tokenizer.save_pretrained`。

### register_predictable_model (lines 74-82)
```python
def register_predictable_model():
    """Register the PredictableLlamaForCausalLM model."""
    from .predictable_llama import PredictableLlamaForCausalLM

    if "PredictableLlamaForCausalLM" not in ModelRegistry.get_supported_archs():
        ModelRegistry.register_model(
            "PredictableLlamaForCausalLM", PredictableLlamaForCausalLM
        )
    yield
```
**EN:** Fixture/helper `register_predictable_model` prepares reusable state for downstream tests. Key calls include `pytest.fixture, ModelRegistry.get_supported_archs, ModelRegistry.register_model`.
**CN:** `register_predictable_model` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `pytest.fixture, ModelRegistry.get_supported_archs, ModelRegistry.register_model`。

### test_extract_hidden_states_with_predictable_dummy_model (lines 85-161)
```python
def test_extract_hidden_states_with_predictable_dummy_model(
    predictable_llama_config_path, tmp_path, monkeypatch
):
    """Comprehensive test using a predictable dummy model with synthetic weights.

    The PredictableLlamaForCausalLM outputs deterministic hidden states where
    each layer produces values equal to (layer_index). This test verifies:
    1. Hidden states are correctly extracted from requested layers
    2. Values match the expected predictable pattern
    3. Layer ordering is preserved correctly (non-sequential layer IDs)
    4. Multiple prompts of different lengths produce consistent layer values
    """
    # Force fork so the engine worker inherits the autouse fixture's
    # ModelRegistry.register_model("PredictableLlamaForCausalLM", ...).
    # Spawn (the CI default) starts a fresh Python process that wouldn't
    # see the registration.
    monkeypatch.setenv("VLLM_WORKER_MULTIPROC_METHOD", "fork")
    # ... excerpt omitted for brevity ...
    assert len(outputs) == len(prompts)
        expected_shape = (
        _token_ids, hidden_states = get_and_check_output(output, expected_shape)
            assert torch.allclose(
                atol=1e-5,
            ), (
                f"Layer {layer_id} at position {idx} should output {float(layer_id)}, "
                f"but got mean={layer_hidden.mean():.3f}, "
                f"min={layer_hidden.min():.3f}, max={layer_hidden.max():.3f}"
            )
```
**EN:** Test case covering `extract hidden states with predictable dummy model`. Inputs/fixtures: `predictable_llama_config_path, tmp_path, monkeypatch`. It exercises `monkeypatch.setenv, len, LLM, SamplingParams, model_config.get_hidden_size, llm.generate`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `extract hidden states with predictable dummy model` 的测试用例。 输入或 fixture：`predictable_llama_config_path, tmp_path, monkeypatch`。 该测试会调用 `monkeypatch.setenv, len, LLM, SamplingParams, model_config.get_hidden_size, llm.generate`。 代码主体包含 2 个显式断言。

### test_extract_hidden_states_qwen35_hybrid_smoke (lines 164-213)
```python
def test_extract_hidden_states_qwen35_hybrid_smoke(tmp_path):
    """Smoke test for Qwen3.5 hybrid (mamba + full-attention) models.
    Uses load_format="dummy" to just check shape/plumbing.
    """
    layer_ids = [5, 11, 17]
    hidden_size = 1024  # Qwen/Qwen3.5-0.8B hidden_size

    llm = LLM(
        model="Qwen/Qwen3.5-0.8B",
        speculative_config={
            "method": "extract_hidden_states",
            "num_speculative_tokens": 1,
            "draft_model_config": {
                "hf_config": {"eagle_aux_hidden_state_layer_ids": layer_ids}
            },
        },
        kv_transfer_config={
            "kv_connector": "ExampleHiddenStatesConnector",
    # ... excerpt omitted for brevity ...
    assert len(outputs) == len(prompts)
        assert output.kv_transfer_params is not None
        assert hidden_states_path is not None
        assert os.path.exists(hidden_states_path)
        assert torch.equal(token_ids, torch.tensor(output.prompt_token_ids))
        assert hidden_states.shape == (
            len(output.prompt_token_ids),
            len(layer_ids),
            hidden_size,
        )
```
**EN:** Test case covering `extract hidden states qwen35 hybrid smoke`. Inputs/fixtures: `tmp_path`. It exercises `LLM, SamplingParams, llm.generate, gc.collect, len, kv_transfer_params.get`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `extract hidden states qwen35 hybrid smoke` 的测试用例。 输入或 fixture：`tmp_path`。 该测试会调用 `LLM, SamplingParams, llm.generate, gc.collect, len, kv_transfer_params.get`。 代码主体包含 6 个显式断言。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, safetensors, transformers`.
- **CN:** 外部库：`pytest, torch, safetensors, transformers`。
- **EN:** vLLM modules under test: `vllm`.
- **CN:** 被测试的 vLLM 模块：`vllm`。
- **EN:** Local test helpers: `tests.v1.kv_connector.extract_hidden_states_integration.predictable_llama`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.extract_hidden_states_integration.predictable_llama`。
- **EN:** Standard-library support: `gc, os`.
- **CN:** 标准库支持：`gc, os`。
