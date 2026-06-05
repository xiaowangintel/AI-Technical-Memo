# test_quark.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/quantization/test_quark.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Test model set-up and weight loading for quark-quantized models. / 该文件主要围绕 Quark 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-39)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Test model set-up and weight loading for quark-quantized models.

Run `pytest tests/quantization/test_quark.py`.

See also `tests/kernels/moe/test_ocp_mx_moe.py`.
"""

import importlib.metadata
from dataclasses import dataclass
from importlib.util import find_spec

import huggingface_hub
import lm_eval
import pytest
import torch
from packaging import version

# ... omitted for brevity ...
    QuarkW8A8Int8MoEMethod,
)
from vllm.platforms import current_platform

from .reference_mxfp4 import dq_mxfp4_torch, qdq_mxfp4_torch

# Minimum amd-quark version for MXFP4/OCP_MX tests (single source of truth).
QUARK_MXFP4_MIN_VERSION = "0.8.99"

QUARK_MXFP4_AVAILABLE = find_spec("quark") is not None and version.parse(
    importlib.metadata.version("amd-quark")
) >= version.parse(QUARK_MXFP4_MIN_VERSION)

DEVICE_TYPE = current_platform.device_type
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `importlib.metadata`, `dataclasses`, `huggingface_hub`, `lm_eval`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: enable_pickle (lines 55-58)
```python
@pytest.fixture(scope="function", autouse=True)
def enable_pickle(monkeypatch):
    """`LLM.apply_model` requires pickling a function."""
    monkeypatch.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
```
**EN:** `LLM.apply_model` requires pickling a function. The fixture mainly builds or returns values through `pytest.fixture`, `monkeypatch.setenv`.
**CN:** 该代码块定义 pytest 夹具 `enable_pickle`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `monkeypatch.setenv` 构造或返回测试所需的值。

### Test: test_quark_fp8_w_per_tensor_a_per_tensor (lines 61-88)
```python
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8"])
@pytest.mark.parametrize("tp", [1])
def test_quark_fp8_w_per_tensor_a_per_tensor(vllm_runner, kv_cache_dtype, tp):
    model_path = "amd/Llama-3.1-8B-Instruct-FP8-KV-Quark-test"
    with vllm_runner(
        model_path,
        enforce_eager=True,
        kv_cache_dtype=kv_cache_dtype,
        tensor_parallel_size=tp,
    ) as llm:

        def check_model(model):
            layer = model.model.layers[0]

            qkv_proj = layer.self_attn.qkv_proj

            assert isinstance(qkv_proj.quant_method, QuarkLinearMethod)
            assert isinstance(qkv_proj.scheme, QuarkW8A8Fp8)

            if isinstance(qkv_proj.scheme, QuarkW8A8Fp8):
                assert len(qkv_proj.input_scale.shape) == 0
                assert qkv_proj.weight.dtype is current_platform.fp8_dtype()
                assert len(qkv_proj.weight_scale.shape) == 0

        llm.apply_model(check_model)

        output = llm.generate_greedy("Hello my name is", max_tokens=4)
        assert output
```
**EN:** Checks Quark FP8 W Per Tensor A Per Tensor under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `vllm_runner`, `llm.apply_model` before asserting the expected outcome.
**CN:** 该测试用例验证 Quark FP8 W Per Tensor A Per Tensor 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `vllm_runner`, `llm.apply_model` 驱动目标逻辑，再断言预期结果。

### Test: test_quark_fp8_w_per_channel_a_per_token (lines 91-112)
```python
@pytest.mark.parametrize("tp", [1])
def test_quark_fp8_w_per_channel_a_per_token(vllm_runner, tp):
    model_path = "amd/Qwen2.5-1.5B-Instruct-ptpc-Quark-ts"
    with vllm_runner(model_path, enforce_eager=True, tensor_parallel_size=tp) as llm:

        def check_model(model):
            layer = model.model.layers[0]

            qkv_proj = layer.self_attn.qkv_proj

            assert isinstance(qkv_proj.quant_method, QuarkLinearMethod)
            assert isinstance(qkv_proj.scheme, QuarkW8A8Fp8)

            if isinstance(qkv_proj.scheme, QuarkW8A8Fp8):
                assert qkv_proj.weight.dtype is current_platform.fp8_dtype()
                assert qkv_proj.weight_scale.shape[0] == qkv_proj.weight.shape[1]
                assert qkv_proj.weight_scale.shape[1] == 1

        llm.apply_model(check_model)

        output = llm.generate_greedy("Hello my name is", max_tokens=4)
        assert output
```
**EN:** Checks Quark FP8 W Per Channel A Per Token under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `vllm_runner`, `llm.apply_model` before asserting the expected outcome.
**CN:** 该测试用例验证 Quark FP8 W Per Channel A Per Token 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `vllm_runner`, `llm.apply_model` 驱动目标逻辑，再断言预期结果。

### Test: test_quark_int8_w_per_tensor_a_per_tensor (lines 115-131)
```python
@pytest.mark.parametrize("tp", [1])
def test_quark_int8_w_per_tensor_a_per_tensor(vllm_runner, tp):
    model_path = "amd/Llama-3.1-8B-Instruct-w-int8-a-int8-sym-test"
    with vllm_runner(model_path, enforce_eager=True, tensor_parallel_size=tp) as llm:

        def check_model(model):
            layer = model.model.layers[0]

            qkv_proj = layer.self_attn.qkv_proj

            assert isinstance(qkv_proj.quant_method, QuarkLinearMethod)
            assert isinstance(qkv_proj.scheme, QuarkW8A8Int8)

        llm.apply_model(check_model)

        output = llm.generate_greedy("Hello my name is", max_tokens=4)
        assert output
```
**EN:** Checks Quark INT8 W Per Tensor A Per Tensor under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `vllm_runner`, `llm.apply_model` before asserting the expected outcome.
**CN:** 该测试用例验证 Quark INT8 W Per Tensor A Per Tensor 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `vllm_runner`, `llm.apply_model` 驱动目标逻辑，再断言预期结果。

### Test: test_quark_int8_w8a8_moe (lines 134-159)
```python
@pytest.mark.parametrize("tp", [1])
def test_quark_int8_w8a8_moe(vllm_runner, tp):
    """Test W8A8 INT8 MoE quantization with a tiny Qwen3 MoE model."""
    model_path = "nameistoken/tiny-qwen3-moe-w8a8-int8-quark"
    with vllm_runner(
        model_path,
        enforce_eager=True,
        tensor_parallel_size=tp,
        gpu_memory_utilization=0.1,
    ) as llm:

        def check_model(model):
            layer = model.model.layers[0]
            # MoE experts should use QuarkW8A8Int8MoEMethod
            moe = layer.mlp.experts
            assert isinstance(moe.quant_method, QuarkW8A8Int8MoEMethod), (
                f"Expected QuarkW8A8Int8MoEMethod, got {type(moe.quant_method)}"
            )
            # Non-MoE linear layers should use QuarkW8A8Int8
            qkv_proj = layer.self_attn.qkv_proj
            assert isinstance(qkv_proj.scheme, QuarkW8A8Int8)

        llm.apply_model(check_model)

        output = llm.generate_greedy("Hello", max_tokens=4)
        assert output
```
**EN:** Test W8A8 INT8 MoE quantization with a tiny Qwen3 MoE model. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `vllm_runner`, `llm.apply_model` before asserting the expected outcome.
**CN:** 该测试用例验证 Quark INT8 W8a8 MoE 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `vllm_runner`, `llm.apply_model` 驱动目标逻辑，再断言预期结果。

### Test: test_quark_fp8_parity (lines 162-185)
```python
def test_quark_fp8_parity(vllm_runner):
    quark_model_id = "amd-quark/llama-tiny-fp8-quark-quant-method"
    fp8_model_id = "amd-quark/llama-tiny-fp8-quant-method"

    llm_kwargs = {
        "tensor_parallel_size": 1,
        "enforce_eager": True,
        "gpu_memory_utilization": 0.1,
    }
    with (
        vllm_runner(quark_model_id, **llm_kwargs) as quark_handle,
        vllm_runner(fp8_model_id, **llm_kwargs) as fp8_handle,
    ):

        def get_state_dict(model):
            return {k: v.cpu() for k, v in model.state_dict().items()}

        (quark_state_dict,) = quark_handle.apply_model(get_state_dict)
        (fp8_state_dict,) = fp8_handle.apply_model(get_state_dict)

    assert fp8_state_dict.keys() == quark_state_dict.keys()

    for key in fp8_state_dict:
        assert torch.equal(fp8_state_dict[key], quark_state_dict[key])
```
**EN:** Checks Quark FP8 Parity under a focused test scenario. The body exercises logic via `vllm_runner`, `quark_handle.apply_model`, `fp8_handle.apply_model` before asserting the expected outcome.
**CN:** 该测试用例验证 Quark FP8 Parity 在特定场景下的行为。 函数体会先通过 `vllm_runner`, `quark_handle.apply_model`, `fp8_handle.apply_model` 驱动目标逻辑，再断言预期结果。

### Class: AccuracyTestConfig (lines 188-213)
```python
@dataclass
class AccuracyTestConfig:
    model_name: str
    excepted_value: float

    def get_model_args(
        self,
        tp_size: int,
        model_max_len: int | None = None,
        kwargs: dict | None = None,
    ) -> dict:
        if kwargs is None:
            kwargs = {}

        model_args = {
            "pretrained": self.model_name,
            "dtype": "auto",
            "add_bos_token": True,
            "tensor_parallel_size": tp_size,
            "gpu_memory_utilization": 0.7,
            **kwargs,
        }
        if model_max_len is not None:
            model_args["max_model_len"] = model_max_len

        return model_args
```
**EN:** Groups related scenarios for Accuracytestconfig.
**CN:** 该类把与 Accuracytestconfig 相关的场景组织在一起。

### Constants / assignments (lines 216-222)
```python
GSM8K_ACCURACY_CONFIGS = [
    # Private model.
    AccuracyTestConfig(
        model_name="amd/DeepSeek-R1-WMXFP4-AMXFP4-Scale-UINT8-MoE-Quant",
        excepted_value=0.96,
    ),
]
```
**EN:** Defines shared constants or configuration objects like `GSM8K_ACCURACY_CONFIGS`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `GSM8K_ACCURACY_CONFIGS`），供后续测试重复使用。

### Additional scenarios (summary)
```python
Conditional block
Block
Constants / assignments
test_ocp_mx_wikitext_correctness
test_nvfp4_wikitext_correctness
test_mxfp4_gsm8k_correctness
test_mxfp4_fused_qdq_match_quark
test_mxfp4_dequant_kernel_match_quark
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
- **Standard library / 标准库**: `importlib.metadata`, `dataclasses`, `importlib.util`
- **Third-party / 第三方依赖**: `huggingface_hub`, `lm_eval`, `pytest`, `torch`, `packaging`, `quark.torch.export.nn.modules.realquantizer`, `quark.torch.kernel`, `quark.torch.quantization.config.config`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.quantization.quark.quark`, `vllm.model_executor.layers.quantization.quark.quark_moe`, `vllm.platforms`
- **Local test utilities / 本地测试辅助**: `.reference_mxfp4`
