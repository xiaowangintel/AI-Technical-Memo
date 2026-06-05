# test_eagle_quantization.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/model_executor/test_eagle_quantization.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Eagle Quantization behavior in the Model Executor test area through focused pytest scenarios. It focuses on scenarios such as Get Draft Quant Config With Draft Model, Get Draft Quant Config Without Draft Model, Fc Layer Quant Config Usage. / 该文件在 Model Executor 测试域中，通过有针对性的 pytest 场景验证 Eagle Quantization 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-18)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from unittest.mock import Mock, patch

import pytest
import torch

from vllm.config import LoadConfig, ModelConfig, SpeculativeConfig, VllmConfig
from vllm.model_executor.models.utils import get_draft_quant_config
from vllm.platforms import current_platform

DEVICE_TYPE = current_platform.device_type
DEVICES = (
    [f"{DEVICE_TYPE}:{i}" for i in range(min(torch.accelerator.device_count(), 2))]
    if not current_platform.is_cpu()
    else ["cpu"]
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `unittest.mock`, `pytest`, `torch`, `vllm.config`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_get_draft_quant_config_with_draft_model (lines 21-41)
```python
def test_get_draft_quant_config_with_draft_model():
    mock_draft_model_config = Mock(spec=ModelConfig)
    mock_load_config = Mock(spec=LoadConfig)
    mock_speculative_config = Mock(spec=SpeculativeConfig)
    mock_speculative_config.draft_model_config = mock_draft_model_config

    mock_vllm_config = Mock(spec=VllmConfig)
    mock_vllm_config.speculative_config = mock_speculative_config
    mock_vllm_config.load_config = mock_load_config

    mock_quant_config = Mock()
    with patch.object(
        VllmConfig, "get_quantization_config", return_value=mock_quant_config
    ):
        result = get_draft_quant_config(mock_vllm_config)

        # Verify the function calls get_quantization_config with draft model config
        VllmConfig.get_quantization_config.assert_called_once_with(
            mock_draft_model_config, mock_load_config
        )
        assert result == mock_quant_config
```
**EN:** Checks Get Draft Quant Config With Draft Model under a focused test scenario. The body exercises logic via `Mock`, `patch.object`, `get_draft_quant_config` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Draft Quant Config With Draft Model 在特定场景下的行为。 函数体会先通过 `Mock`, `patch.object`, `get_draft_quant_config` 驱动目标逻辑，再断言预期结果。

### Test: test_get_draft_quant_config_without_draft_model (lines 44-54)
```python
def test_get_draft_quant_config_without_draft_model():
    mock_speculative_config = Mock(spec=SpeculativeConfig)
    mock_speculative_config.draft_model_config = None

    mock_vllm_config = Mock(spec=VllmConfig)
    mock_vllm_config.speculative_config = mock_speculative_config
    mock_vllm_config.load_config = Mock(spec=LoadConfig)

    result = get_draft_quant_config(mock_vllm_config)

    assert result is None
```
**EN:** Checks Get Draft Quant Config Without Draft Model under a focused test scenario. The body exercises logic via `Mock`, `get_draft_quant_config` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Draft Quant Config Without Draft Model 在特定场景下的行为。 函数体会先通过 `Mock`, `get_draft_quant_config` 驱动目标逻辑，再断言预期结果。

### Test: test_fc_layer_quant_config_usage (lines 57-100)
```python
@torch.inference_mode()
@pytest.mark.parametrize("device", DEVICES)
def test_fc_layer_quant_config_usage(default_vllm_config, dist_init, device) -> None:
    import torch

    from vllm.model_executor.layers.linear import ReplicatedLinear

    if current_platform.is_cuda_alike():
        torch.accelerator.set_device_index(device)

    torch.set_default_device(device)

    input_size = 256
    output_size = 128

    fc_no_quant = ReplicatedLinear(
        input_size=input_size,
        output_size=output_size,
        bias=False,
# ... omitted for brevity ...
        input_size=input_size,
        output_size=output_size,
        bias=False,
        params_dtype=torch.float16,
        quant_config=mock_quant_config,
        prefix="fc",
    )

    assert fc_with_quant.quant_config == mock_quant_config

    # Check forward pass
    x = torch.randn(2, input_size, dtype=torch.float16)
    output, _ = fc_no_quant(x)
    assert output.shape == (2, output_size)
```
**EN:** Checks Fc Layer Quant Config Usage under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `torch.inference_mode`, `pytest.mark.parametrize`, `current_platform.is_cuda_alike` before asserting the expected outcome.
**CN:** 该测试用例验证 Fc Layer Quant Config Usage 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `torch.inference_mode`, `pytest.mark.parametrize`, `current_platform.is_cuda_alike` 驱动目标逻辑，再断言预期结果。

### Test: test_kv_cache_scale_name_handling (lines 103-114)
```python
def test_kv_cache_scale_name_handling():
    # Mock a quant config that supports cache scales
    mock_quant_config = Mock()
    mock_quant_config.get_cache_scale = Mock(return_value="layers.0.self_attn.kv_scale")

    # Condition check in load_weights
    name = "layers.0.self_attn.k_proj.weight"
    scale_name = mock_quant_config.get_cache_scale(name)

    # Check if get_cache_scale is called and returns expected value
    mock_quant_config.get_cache_scale.assert_called_once_with(name)
    assert scale_name == "layers.0.self_attn.kv_scale"
```
**EN:** Checks KV Cache Scale Name Handling under a focused test scenario. The body exercises logic via `Mock`, `mock_quant_config.get_cache_scale`, `mock_quant_config.get_cache_scale.assert_called_once_with` before asserting the expected outcome.
**CN:** 该测试用例验证 KV Cache Scale Name Handling 在特定场景下的行为。 函数体会先通过 `Mock`, `mock_quant_config.get_cache_scale`, `mock_quant_config.get_cache_scale.assert_called_once_with` 驱动目标逻辑，再断言预期结果。

### Test: test_kv_cache_scale_name_no_scale (lines 117-126)
```python
def test_kv_cache_scale_name_no_scale():
    # Mock a quant config that returns None for get_cache_scale
    mock_quant_config = Mock()
    mock_quant_config.get_cache_scale = Mock(return_value=None)

    name = "layers.0.mlp.gate_proj.weight"
    scale_name = mock_quant_config.get_cache_scale(name)

    # Should return None for weights that don't have cache scales
    assert scale_name is None
```
**EN:** Checks KV Cache Scale Name No Scale under a focused test scenario. The body exercises logic via `Mock`, `mock_quant_config.get_cache_scale` before asserting the expected outcome.
**CN:** 该测试用例验证 KV Cache Scale Name No Scale 在特定场景下的行为。 函数体会先通过 `Mock`, `mock_quant_config.get_cache_scale` 驱动目标逻辑，再断言预期结果。

### Test: test_maybe_remap_kv_scale_name (lines 129-140)
```python
def test_maybe_remap_kv_scale_name():
    from vllm.model_executor.model_loader.weight_utils import maybe_remap_kv_scale_name

    params_dict = {
        "layers.0.self_attn.kv_scale": Mock(),
        "layers.1.self_attn.kv_scale": Mock(),
    }

    name = "layers.0.self_attn.some_scale"
    remapped = maybe_remap_kv_scale_name(name, params_dict)

    assert remapped in params_dict or remapped == name or remapped is None
```
**EN:** Checks Maybe Remap KV Scale Name under a focused test scenario. The body exercises logic via `maybe_remap_kv_scale_name`, `Mock` before asserting the expected outcome.
**CN:** 该测试用例验证 Maybe Remap KV Scale Name 在特定场景下的行为。 函数体会先通过 `maybe_remap_kv_scale_name`, `Mock` 驱动目标逻辑，再断言预期结果。

### Test: test_eagle3_lm_head_receives_quant_config (lines 143-185)
```python
def test_eagle3_lm_head_receives_quant_config():
    """Eagle3LlamaForCausalLM must pass quant_config to ParallelLMHead.

    Without quant_config, quantized lm_head weights (e.g. INT8 per-channel)
    in Eagle3 drafter checkpoints fail to load because ParallelLMHead doesn't
    expect weight_packed tensors.
    """
    from vllm.model_executor.models.llama_eagle3 import Eagle3LlamaForCausalLM

    mock_quant_config = Mock()

    mock_hf_config = Mock()
    mock_hf_config.draft_vocab_size = 1000
    mock_hf_config.hidden_size = 256
    mock_hf_config.vocab_size = 32000
    mock_hf_config.logit_scale = 1.0

    mock_vllm_config = Mock()
    mock_vllm_config.speculative_config.draft_model_config.hf_config = mock_hf_config
# ... omitted for brevity ...
        ),
    ):
        MockModel.return_value.use_aux_hidden_state = True

        Eagle3LlamaForCausalLM(vllm_config=mock_vllm_config)

        MockLMHead.assert_called_once()
        call_kwargs = MockLMHead.call_args.kwargs
        assert "quant_config" in call_kwargs, (
            "ParallelLMHead must receive quant_config for quantized lm_head weights"
        )
        assert call_kwargs["quant_config"] is mock_quant_config, (
            "ParallelLMHead must receive the draft model's quant_config"
        )
```
**EN:** Eagle3LlamaForCausalLM must pass quant_config to ParallelLMHead. The body exercises logic via `Mock`, `patch`, `Eagle3LlamaForCausalLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Eagle3 Lm Head Receives Quant Config 在特定场景下的行为。 函数体会先通过 `Mock`, `patch`, `Eagle3LlamaForCausalLM` 驱动目标逻辑，再断言预期结果。

### Test: test_load_weights_kv_scale_handling (lines 188-215)
```python
def test_load_weights_kv_scale_handling():
    kv_scale_param = Mock()
    kv_scale_param.weight_loader = Mock()

    params_dict = {
        "layers.0.self_attn.kv_scale": kv_scale_param,
    }

    mock_quant_config = Mock()
    mock_quant_config.get_cache_scale = Mock(return_value="layers.0.self_attn.kv_scale")

    # Load_weights logic for KV cache scales
    name = "layers.0.self_attn.k_proj.weight"
    loaded_weight_tensor = torch.tensor([1.0, 2.0])

    if mock_quant_config is not None:
        scale_name = mock_quant_config.get_cache_scale(name)
        if scale_name:
            param = params_dict[scale_name]
            assert param is kv_scale_param
            weight_to_load = (
                loaded_weight_tensor
                if loaded_weight_tensor.dim() == 0
                else loaded_weight_tensor[0]
            )

            assert scale_name == "layers.0.self_attn.kv_scale"
            assert weight_to_load == loaded_weight_tensor[0]
```
**EN:** Checks Load Weights KV Scale Handling under a focused test scenario. The body exercises logic via `Mock`, `torch.tensor`, `mock_quant_config.get_cache_scale` before asserting the expected outcome.
**CN:** 该测试用例验证 Load Weights KV Scale Handling 在特定场景下的行为。 函数体会先通过 `Mock`, `torch.tensor`, `mock_quant_config.get_cache_scale` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Quantization / 量化**
  - **EN:** The file covers low-precision model paths, kernels, or configuration rules for quantized inference.
  - **CN:** 该文件覆盖低精度模型路径、相关内核或量化推理配置规则。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest.mock`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.config`, `vllm.model_executor.models.utils`, `vllm.platforms`, `vllm.model_executor.layers.linear`, `vllm.model_executor.model_loader.weight_utils`, `vllm.model_executor.models.llama_eagle3`
