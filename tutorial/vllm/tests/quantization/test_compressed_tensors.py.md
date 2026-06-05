# test_compressed_tensors.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/quantization/test_compressed_tensors.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Test model set-up and weight loading for llmcompressor-quantized models. / 该文件主要围绕 Compressed Tensors 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-62)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Test model set-up and weight loading for llmcompressor-quantized models.

Run `pytest tests/quantization/test_compressed_tensors.py`.
"""

from unittest.mock import Mock

import pytest
import torch
from compressed_tensors.quantization import (
    QuantizationArgs,
    QuantizationStrategy,
    QuantizationType,
)

from tests.models.utils import check_logprobs_close
from vllm.model_executor.kernels.linear import (
# ... omitted for brevity ...
# It does not support mix precision MM and mix quantization scheme.
ROCM_AITER_SUPPORTED_INT8_MODEL = [
    "neuralmagic/Llama-3.2-1B-quantized.w8a8",
    "nm-testing/tinyllama-oneshot-w8a8-channel-dynamic-token-v2",
]

# TritonInt8ScaledMMLinearKernel only supports symmetric quantization.
ROCM_TRITON_SCALED_MM_SUPPORTED_INT8_MODEL = [
    "nm-testing/tinyllama-oneshot-w8w8-test-static-shape-change",
    "nm-testing/tinyllama-oneshot-w8-channel-a8-tensor",
    "neuralmagic/Llama-3.2-1B-quantized.w8a8",
    "nm-testing/tinyllama-oneshot-w8a8-dynamic-token-v2",
    "nm-testing/tinyllama-oneshot-w8a8-channel-dynamic-token-v2",
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `unittest.mock`, `pytest`, `torch`, `vllm.model_executor.kernels.linear`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: enable_pickle (lines 65-68)
```python
@pytest.fixture(scope="function", autouse=True)
def enable_pickle(monkeypatch):
    """`LLM.apply_model` requires pickling a function."""
    monkeypatch.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
```
**EN:** `LLM.apply_model` requires pickling a function. The fixture mainly builds or returns values through `pytest.fixture`, `monkeypatch.setenv`.
**CN:** 该代码块定义 pytest 夹具 `enable_pickle`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `monkeypatch.setenv` 构造或返回测试所需的值。

### Test: test_compressed_tensors_w8a8_static_setup (lines 71-147)
```python
@pytest.mark.parametrize(
    "model_args",
    [
        (
            "nm-testing/tinyllama-oneshot-w8w8-test-static-shape-change",
            "tensor",
            QuantizationType.INT,
            2560,
            True,
        ),
        (
            "nm-testing/asym-w8w8-int8-static-per-tensor-tiny-llama",
            "tensor",
            QuantizationType.INT,
            2560,
            False,
        ),
    ],
)
# ... omitted for brevity ...

            if qkv_proj.scheme.strategy == "tensor":
                # Make sure it is a channelwise buffer
                # After running process_weights_after_loading
                assert len(qkv_proj.weight_scale.shape) == 2
                assert qkv_proj.weight_scale.shape[0] == shape_0
                assert qkv_proj.weight_scale.shape[1] == 1
            assert qkv_proj.weight_scale.dtype is torch.float32
            assert qkv_proj.input_scale.dtype is torch.float32

        llm.apply_model(check_model)

        output = llm.generate_greedy(["Hello my name is"], max_tokens=4)
        assert output
```
**EN:** Checks Compressed Tensors W8a8 Static Setup under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `current_platform.is_rocm`, `pytest.skip` before asserting the expected outcome.
**CN:** 该测试用例验证 Compressed Tensors W8a8 Static Setup 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `current_platform.is_rocm`, `pytest.skip` 驱动目标逻辑，再断言预期结果。

### Test: test_compressed_tensors_w8a8_logprobs (lines 150-210)
```python
@pytest.mark.parametrize(
    "model_path",
    [
        "neuralmagic/Llama-3.2-1B-quantized.w8a8",
    ],
)
@pytest.mark.parametrize("max_tokens", [4])
@pytest.mark.parametrize("num_logprobs", [10])
@pytest.mark.parametrize(
    "use_aiter", [True, False] if current_platform.is_rocm() else [False]
)
def test_compressed_tensors_w8a8_logprobs(
    hf_runner,
    vllm_runner,
    example_prompts,
    model_path,
    max_tokens,
    num_logprobs,
    use_aiter,
# ... omitted for brevity ...
    with vllm_runner(model_path, dtype=dtype, enforce_eager=True) as vllm_model:
        vllm_outputs = vllm_model.generate_greedy_logprobs(
            example_prompts, max_tokens, num_logprobs
        )

    check_logprobs_close(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )

    if current_platform.is_rocm():
        torch.accelerator.synchronize()
```
**EN:** Checks Compressed Tensors W8a8 Logprobs under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `check_logprobs_close`, `current_platform.is_rocm` before asserting the expected outcome.
**CN:** 该测试用例验证 Compressed Tensors W8a8 Logprobs 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `check_logprobs_close`, `current_platform.is_rocm` 驱动目标逻辑，再断言预期结果。

### Test: test_compressed_tensors_no_enforce_eager (lines 213-217)
```python
def test_compressed_tensors_no_enforce_eager(vllm_runner):
    model_path = "nm-testing/tinyllama-oneshot-w8w8-test-static-shape-change"
    with vllm_runner(model_path) as llm:
        output = llm.generate_greedy("Hello my name is", max_tokens=4)
        assert output
```
**EN:** Checks Compressed Tensors No Enforce Eager under a focused test scenario. The body exercises logic via `vllm_runner`, `llm.generate_greedy` before asserting the expected outcome.
**CN:** 该测试用例验证 Compressed Tensors No Enforce Eager 在特定场景下的行为。 函数体会先通过 `vllm_runner`, `llm.generate_greedy` 驱动目标逻辑，再断言预期结果。

### Test: test_compressed_tensors_w8a8_dynamic_per_token (lines 220-269)
```python
@pytest.mark.parametrize(
    "model_args",
    [
        ("nm-testing/tinyllama-oneshot-w8a8-dynamic-token-v2", "tensor"),
        (
            "nm-testing/tinyllama-oneshot-w8a8-channel-dynamic-token-v2",
            "channel",
        ),
    ],
)
@pytest.mark.parametrize(
    "use_aiter", [True, False] if current_platform.is_rocm() else [False]
)
def test_compressed_tensors_w8a8_dynamic_per_token(
    vllm_runner,
    model_args,
    use_aiter,
    monkeypatch,
):
# ... omitted for brevity ...
            layer = model.model.layers[0]

            qkv_proj = layer.self_attn.qkv_proj

            assert isinstance(qkv_proj.quant_method, CompressedTensorsLinearMethod)
            assert isinstance(qkv_proj.scheme, CompressedTensorsW8A8Int8)
            assert not qkv_proj.scheme.is_static_input_scheme
            assert qkv_proj.scheme.strategy == strategy
            assert qkv_proj.weight.dtype is torch.int8

        llm.apply_model(check_model)

        output = llm.generate_greedy(["Hello my name is"], max_tokens=4)
        assert output
```
**EN:** Checks Compressed Tensors W8a8 Dynamic Per Token under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `current_platform.is_rocm`, `pytest.skip` before asserting the expected outcome.
**CN:** 该测试用例验证 Compressed Tensors W8a8 Dynamic Per Token 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `current_platform.is_rocm`, `pytest.skip` 驱动目标逻辑，再断言预期结果。

### Test: test_compressed_tensors_wNa16 (lines 272-317)
```python
@pytest.mark.parametrize(
    "wNa16_args",
    [
        (
            "nm-testing/tinyllama-oneshot-w4a16-channel-v2",
            "channel",
            None,
            8,
            True,
            False,
        ),
        (
            "nm-testing/TinyLlama-1.1B-Chat-v1.0-W4A16-G128-Asym-Updated-ActOrder",
            "group",
            128,
            8,
            False,
            True,
        ),
# ... omitted for brevity ...
            assert isinstance(qkv_proj.quant_method, CompressedTensorsLinearMethod)
            assert isinstance(qkv_proj.scheme, CompressedTensorsWNA16)

            assert qkv_proj.scheme.strategy == strategy
            assert qkv_proj.scheme.group_size == (-1 if group is None else group)

            assert qkv_proj.scheme.pack_factor == pack_factor
            assert qkv_proj.scheme.symmetric == symmetric
            assert qkv_proj.scheme.has_g_idx == has_g_idx

        llm.apply_model(check_model)

        output = llm.generate_greedy("Hello my name is", max_tokens=4)
        assert output
```
**EN:** Checks Compressed Tensors Wna16 under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `pytest.mark.skipif`, `vllm_runner` before asserting the expected outcome.
**CN:** 该测试用例验证 Compressed Tensors Wna16 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `pytest.mark.skipif`, `vllm_runner` 驱动目标逻辑，再断言预期结果。

### Test: test_compressed_tensors_fp8 (lines 320-346)
```python
def test_compressed_tensors_fp8(vllm_runner):
    model_path = "nm-testing/Meta-Llama-3-8B-FP8-compressed-tensors-test"
    with vllm_runner(model_path, enforce_eager=True) as llm:

        def check_model(model):
            layer = model.model.layers[0]

            qkv_proj = layer.self_attn.qkv_proj

            assert isinstance(qkv_proj.quant_method, CompressedTensorsLinearMethod)
            assert isinstance(
                qkv_proj.scheme,
                (CompressedTensorsW8A8Fp8, CompressedTensorsW8A16Fp8),
            )

            assert qkv_proj.input_scale.dtype is torch.float32

            if isinstance(qkv_proj.scheme, CompressedTensorsW8A8Fp8):
                assert len(qkv_proj.input_scale.shape) == 0
                assert qkv_proj.weight.dtype is current_platform.fp8_dtype()
                assert qkv_proj.weight_scale.dtype is torch.float32
                assert len(qkv_proj.weight_scale.shape) == 0

        llm.apply_model(check_model)

        output = llm.generate_greedy("Hello my name is", max_tokens=4)
        assert output
```
**EN:** Checks Compressed Tensors FP8 under a focused test scenario. The body exercises logic via `vllm_runner`, `llm.apply_model`, `llm.generate_greedy` before asserting the expected outcome.
**CN:** 该测试用例验证 Compressed Tensors FP8 在特定场景下的行为。 函数体会先通过 `vllm_runner`, `llm.apply_model`, `llm.generate_greedy` 驱动目标逻辑，再断言预期结果。

### Helper: _make_ct_config (lines 573-594)
```python
def _make_ct_config(*, target: str = "Linear") -> CompressedTensorsConfig:
    """Build a minimal CompressedTensorsConfig with INT8 channel quant."""
    weight_quant = QuantizationArgs(
        num_bits=8,
        type=QuantizationType.INT,
        strategy=QuantizationStrategy.CHANNEL,
        symmetric=True,
        dynamic=False,
    )
    return CompressedTensorsConfig(
        target_scheme_map={
            target: {
                "weights": weight_quant,
                "input_activations": None,
                "format": "pack-quantized",
            }
        },
        ignore=[],
        quant_format="pack-quantized",
        sparsity_scheme_map={},
        sparsity_ignore_list=[],
    )
```
**EN:** Build a minimal CompressedTensorsConfig with INT8 channel quant. It coordinates operations such as `QuantizationArgs`, `CompressedTensorsConfig`.
**CN:** 该辅助函数为 Make Ct Config 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `QuantizationArgs`, `CompressedTensorsConfig` 等操作。

### Additional scenarios (summary)
```python
test_compressed_tensors_kv_cache_fp8_per_tensor
test_compressed_tensors_kv_cache_fp8_per_attn_head
test_compressed_tensors_nvfp4
test_compressed_tensors_w4a8_fp8
test_compressed_tensors_transforms_perplexity
test_compressed_tensors_fp8_block_enabled
test_compressed_tensors_moe_ignore_with_model
test_w4a16_moe_torch_compile
test_get_quant_method_returns_linear_method_for_parallel_lm_head
test_get_quant_method_returns_none_for_ignored_parallel_lm_head
test_get_quant_method_returns_none_for_unmatched_parallel_lm_head
test_find_matched_target_returns_none_on_no_match
test_get_scheme_dict_returns_none_on_no_match
test_compressed_tensors_mxfp8_moe_setup
test_compressed_tensors_mxfp4
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
- **Standard library / 标准库**: `unittest.mock`
- **Third-party / 第三方依赖**: `pytest`, `torch`, `compressed_tensors.quantization`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.quantization.compressed_tensors.compressed_tensors`, `vllm.model_executor.layers.quantization.compressed_tensors.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.nvfp4_utils`, `vllm.model_executor.layers.vocab_parallel_embedding`, `vllm.platforms`, `vllm.v1.attention.backends.fa_utils`, `vllm.model_executor.layers.quantization.compressed_tensors.compressed_tensors_moe`, ...
- **Local test utilities / 本地测试辅助**: `tests.models.utils`
