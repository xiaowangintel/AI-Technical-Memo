# test_torchao.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/quantization/test_torchao.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Torchao behavior in the Quantization test area through focused pytest scenarios. It focuses on scenarios such as Pre Quantized Model, Opt 125m Int8wo Model Loading With Params, Qwenvl Int8wo Model Loading With Params. / 该文件在 Quantization 测试域中，通过有针对性的 pytest 场景验证 Torchao 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-14)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import importlib.util

import pytest
import torch

from vllm.model_executor.model_loader import get_model_loader
from vllm.platforms import current_platform

DEVICE_TYPE = current_platform.device_type
DTYPE = ["bfloat16"]

TORCHAO_AVAILABLE = importlib.util.find_spec("torchao") is not None
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `importlib.util`, `json`, `pytest`, `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_pre_quantized_model (lines 17-30)
```python
@pytest.mark.skipif(
    current_platform.is_rocm() and current_platform.is_fp8_fnuz(),
    reason="Only fp8_fnuz supported on CDNA3 architecture",
)
@pytest.mark.skipif(not TORCHAO_AVAILABLE, reason="torchao is not available")
def test_pre_quantized_model(vllm_runner):
    with vllm_runner(
        "torchao-testing/opt-125m-Float8WeightOnlyConfig-v2-0.15.0",
        quantization="torchao",
        dtype="bfloat16",
        enforce_eager=True,
    ) as llm:
        output = llm.generate_greedy(["The capital of France is"], max_tokens=4)
    assert output
```
**EN:** Checks Pre Quantized Model under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `vllm_runner`, `llm.generate_greedy` before asserting the expected outcome.
**CN:** 该测试用例验证 Pre Quantized Model 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `vllm_runner`, `llm.generate_greedy` 驱动目标逻辑，再断言预期结果。

### Test: test_opt_125m_int8wo_model_loading_with_params (lines 33-53)
```python
@pytest.mark.skipif(not TORCHAO_AVAILABLE, reason="torchao is not available")
@pytest.mark.parametrize(
    "pt_load_map_location",
    [
        f"{DEVICE_TYPE}:0",
        # {"": "cuda"},
    ],
)
def test_opt_125m_int8wo_model_loading_with_params(vllm_runner, pt_load_map_location):
    torch._dynamo.reset()
    model_name = "jerryzh168/opt-125m-int8wo-partial-quant"
    with vllm_runner(
        model_name=model_name,
        quantization="torchao",
        dtype="bfloat16",
        pt_load_map_location=pt_load_map_location,
        enforce_eager=True,
    ) as llm:
        output = llm.generate_greedy(["The capital of France is"], max_tokens=4)

        assert output
```
**EN:** Checks Opt 125m Int8wo Model Loading With Params under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch._dynamo.reset` before asserting the expected outcome.
**CN:** 该测试用例验证 Opt 125m Int8wo Model Loading With Params 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch._dynamo.reset` 驱动目标逻辑，再断言预期结果。

### Test: test_qwenvl_int8wo_model_loading_with_params (lines 56-69)
```python
@pytest.mark.skipif(not TORCHAO_AVAILABLE, reason="torchao is not available")
def test_qwenvl_int8wo_model_loading_with_params(vllm_runner):
    torch._dynamo.reset()
    model_name = "mobicham/Qwen2.5-VL-3B-Instruct_int8wo_ao"
    with vllm_runner(
        model_name=model_name,
        quantization="torchao",
        dtype="bfloat16",
        pt_load_map_location=f"{DEVICE_TYPE}:0",
        enforce_eager=True,
    ) as llm:
        output = llm.generate_greedy(["The capital of France is"], max_tokens=4)

        assert output
```
**EN:** Checks Qwenvl Int8wo Model Loading With Params under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `torch._dynamo.reset`, `vllm_runner` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwenvl Int8wo Model Loading With Params 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `torch._dynamo.reset`, `vllm_runner` 驱动目标逻辑，再断言预期结果。

### Test: test_opt_125m_awq_int4wo_model_loading_with_params (lines 72-89)
```python
@pytest.mark.skipif(not TORCHAO_AVAILABLE, reason="torchao is not available")
@pytest.mark.skip(
    reason="since torchao nightly is only compatible with torch nightly"
    "currently https://github.com/pytorch/ao/issues/2919, we'll have to skip "
    "torchao tests that requires newer versions (0.14.0.dev+) for now"
)
def test_opt_125m_awq_int4wo_model_loading_with_params(vllm_runner):
    torch._dynamo.reset()
    model_name = "torchao-testing/opt-125m-AWQConfig-Int4WeightOnlyConfig-v2-0.14.0.dev"
    with vllm_runner(
        model_name=model_name,
        quantization="torchao",
        dtype="bfloat16",
        pt_load_map_location=f"{DEVICE_TYPE}:0",
    ) as llm:
        output = llm.generate_greedy(["The capital of France is"], max_tokens=4)

        assert output
```
**EN:** Checks Opt 125m Awq Int4wo Model Loading With Params under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `pytest.mark.skip`, `torch._dynamo.reset` before asserting the expected outcome.
**CN:** 该测试用例验证 Opt 125m Awq Int4wo Model Loading With Params 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `pytest.mark.skip`, `torch._dynamo.reset` 驱动目标逻辑，再断言预期结果。

### Test: test_online_quant_config_dict_json (lines 92-134)
```python
@pytest.mark.skipif(not TORCHAO_AVAILABLE, reason="torchao is not available")
def test_online_quant_config_dict_json(vllm_runner, enable_pickle):
    """Testing online quantization, load_weights integration point,
    with config dict serialized to json string
    """
    torch._dynamo.reset()
    model_name = "facebook/opt-125m"

    import json

    from torchao.core.config import config_to_dict
    from torchao.quantization import Float8DynamicActivationFloat8WeightConfig, PerRow

    torchao_quant_config = Float8DynamicActivationFloat8WeightConfig(
        granularity=PerRow()
    )
    hf_overrides = {
        "quantization_config_dict_json": json.dumps(
            config_to_dict(torchao_quant_config)
# ... omitted for brevity ...
        output = llm.generate_greedy(["The capital of France is"], max_tokens=4)

        load_config = llm.llm.llm_engine.vllm_config.load_config
        model_config = llm.llm.llm_engine.vllm_config.model_config

        def load_weights(model):
            model_loader = get_model_loader(load_config)
            weights_iterator = model_loader.get_all_weights(model_config, model)
            model.load_weights(weights_iterator)

        llm.apply_model(load_weights)

        reload_output = llm.generate_greedy(["The capital of France is"], max_tokens=4)
        assert output[0][0] == reload_output[0][0]
```
**EN:** Testing online quantization, load_weights integration point, with config dict serialized to json string The body exercises logic via `pytest.mark.skipif`, `torch._dynamo.reset`, `Float8DynamicActivationFloat8WeightConfig` before asserting the expected outcome.
**CN:** 该测试用例验证 Online Quant Config Dict JSON 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `torch._dynamo.reset`, `Float8DynamicActivationFloat8WeightConfig` 驱动目标逻辑，再断言预期结果。

### Test: test_online_quant_config_file (lines 137-169)
```python
@pytest.mark.skipif(not TORCHAO_AVAILABLE, reason="torchao is not available")
def test_online_quant_config_file(vllm_runner):
    """Testing on the fly quantization, load_weights integration point,
    with config file
    """
    torch._dynamo.reset()
    model_name = "facebook/opt-125m"
    import json
    from tempfile import NamedTemporaryFile

    from torchao.core.config import config_to_dict
    from torchao.quantization import Float8DynamicActivationFloat8WeightConfig, PerRow

    config = Float8DynamicActivationFloat8WeightConfig(granularity=PerRow())

    with NamedTemporaryFile(mode="w", delete=False) as f:
        f.write(json.dumps(config_to_dict(config)))
        # close the file to save it
        f.close()
        config_file_name = str(f.name)

        hf_overrides = {"quantization_config_file": config_file_name}
        with vllm_runner(
            model_name=model_name,
            dtype="bfloat16",
            pt_load_map_location=f"{DEVICE_TYPE}:0",
            quantization="torchao",
            hf_overrides=hf_overrides,
            enforce_eager=True,
        ) as llm:
            output = llm.generate_greedy(["The capital of France is"], max_tokens=4)

            assert output
```
**EN:** Testing on the fly quantization, load_weights integration point, with config file The body exercises logic via `pytest.mark.skipif`, `torch._dynamo.reset`, `Float8DynamicActivationFloat8WeightConfig` before asserting the expected outcome.
**CN:** 该测试用例验证 Online Quant Config File 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `torch._dynamo.reset`, `Float8DynamicActivationFloat8WeightConfig` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
test_reload_weights
test_safetensors_model_loading_with_params
test_opt_125m_module_fqn_to_config_regex_model
test_opt_125m_int4wo_model_running_preshuffled_kernel
test_opt_125m_int4wo_model_running_preshuffled_kernel_online_quant
Conditional block
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

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
- **Standard library / 标准库**: `importlib.util`, `json`, `tempfile`
- **Third-party / 第三方依赖**: `pytest`, `torch`, `torchao.core.config`, `torchao.quantization`, `torchao.utils`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.model_loader`, `vllm.platforms`, `vllm`
