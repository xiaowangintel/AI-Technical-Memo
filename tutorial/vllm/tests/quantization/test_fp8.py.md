# test_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/quantization/test_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests whether FP8 computation is enabled correctly. / 该文件主要围绕 FP8 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-37)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Tests whether FP8 computation is enabled correctly.

Run `pytest tests/quantization/test_fp8.py --forked`.
"""

import logging

import pytest
import regex as re
import torch

from tests.quantization.utils import is_quant_method_supported
from vllm import _custom_ops as ops
from vllm.config.model import ModelConfig
from vllm.model_executor.layers.fused_moe import FusedMoE
from vllm.model_executor.layers.quantization.fp8 import (
    Fp8Config,
# ... omitted for brevity ...
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.platforms import current_platform

DEVICE_TYPE = current_platform.device_type

MODELS = [
    "neuralmagic/Meta-Llama-3-8B-Instruct-FP8-KV",
    # The checkpoint below was removed from the HF.
    # TODO: add a small replacement checkpoint.
    pytest.param(
        "nm-testing/Qwen2-0.5B-Instruct-FP8-SkipQKV",
        marks=pytest.mark.skip(reason="Checkpoint removed from HF."),
    ),
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `logging`, `pytest`, `regex`, `vllm`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_model_load_and_run (lines 40-64)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("fp8"),
    reason="FP8 is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_id", MODELS)
@pytest.mark.parametrize(
    "force_marlin", [False] if current_platform.is_rocm() else [False, True]
)
@pytest.mark.parametrize(
    "use_rocm_aiter", [True, False] if current_platform.is_rocm() else [False]
)
def test_model_load_and_run(
    vllm_runner, model_id: str, force_marlin: bool, use_rocm_aiter: bool, monkeypatch
) -> None:
    if use_rocm_aiter:
        monkeypatch.setenv("VLLM_ROCM_USE_AITER", "1")

    if force_marlin:
        monkeypatch.setenv("VLLM_TEST_FORCE_FP8_MARLIN", "1")

    with vllm_runner(model_id, enforce_eager=True) as llm:
        # note: this does not test accuracy, just that we can run through
        # see lm-eval tests for accuracy
        outputs = llm.generate_greedy(["Hello my name is"], max_tokens=4)
        print(outputs[0][1])
```
**EN:** Checks Model Load And Run under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv` before asserting the expected outcome.
**CN:** 该测试用例验证 Model Load And Run 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv` 驱动目标逻辑，再断言预期结果。

### Constants / assignments (lines 67-81)
```python
KV_CACHE_MODELS = [
    # AutoFP8 format using separate .k_scale and .v_scale
    # The original checkpoint below was removed from the Hub. To unblock CI and
    # until a small replacement with split K/V scales is found, skip this case.
    # See PR #27717 for context.
    pytest.param(
        "nm-testing/Qwen2-1.5B-Instruct-FP8-K-V",
        marks=pytest.mark.skip(
            reason=(
                "Checkpoint removed from HF; temporarily disabling this "
                "AutoFP8 split K/V case (PR #27717)."
            )
        ),
    ),
]
```
**EN:** Defines shared constants or configuration objects like `KV_CACHE_MODELS`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `KV_CACHE_MODELS`），供后续测试重复使用。

### Test: test_kv_cache_model_load_and_run (lines 84-128)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("fp8"),
    reason="FP8 is not supported on this GPU type.",
)
@pytest.mark.parametrize("model_id", KV_CACHE_MODELS)
@pytest.mark.parametrize(
    "use_rocm_aiter", [True, False] if current_platform.is_rocm() else [False]
)
def test_kv_cache_model_load_and_run(
    vllm_runner, model_id: str, use_rocm_aiter: bool, monkeypatch
):
    if use_rocm_aiter:
        monkeypatch.setenv("VLLM_ROCM_USE_AITER", "1")

    # `LLM.apply_model` requires pickling a function.
    monkeypatch.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
    with vllm_runner(model_id, kv_cache_dtype="fp8", enforce_eager=True) as llm:

        def check_model(model):
# ... omitted for brevity ...
                # NOTE: it is valid for scales to be 1.0 (default value), but
                # we know these checkpoints have scales < 1.0
                # However on ROCm platform, the _k_scale and _v_scale will be
                # scaled by a factor of 2 as described in
                # vllm/model_executor/layers/quantization/kv_cache.py
                assert 0.0 < attn._k_scale < (1.0 * 2.0)
                assert 0.0 < attn._v_scale < (1.0 * 2.0)

        llm.apply_model(check_model)

        # note: this does not test accuracy, just that we can run through
        # see lm-eval tests for accuracy
        outputs = llm.generate_greedy(["Hello my name is"], max_tokens=4)
        print(outputs[0][1])
```
**EN:** Checks KV Cache Model Load And Run under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv` before asserting the expected outcome.
**CN:** 该测试用例验证 KV Cache Model Load And Run 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv` 驱动目标逻辑，再断言预期结果。

### Test: test_online_quantization (lines 131-201)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("fp8"),
    reason="FP8 is not supported on this GPU type.",
)
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8"])
@pytest.mark.parametrize(
    "force_marlin", [False] if current_platform.is_rocm() else [False, True]
)
@pytest.mark.parametrize(
    "use_rocm_aiter", [True, False] if current_platform.is_rocm() else [False]
)
def test_online_quantization(
    vllm_runner,
    kv_cache_dtype: str,
    force_marlin: bool,
    use_rocm_aiter: bool,
    monkeypatch,
) -> None:
    if use_rocm_aiter:
# ... omitted for brevity ...
                        "Skip `test_load_fp16_model`. "
                        "It only runs on ROCm platform with FP8 compute."
                        " e.g. MI300X and above."
                    )
            else:  # unsupported platform
                pytest.skip(
                    "Skip `test_load_fp16_model`. "
                    "It only runs on CUDA and ROCm platform."
                )

        llm.apply_model(check_model)

        outputs = llm.generate_greedy(["Hello my name is"], max_tokens=4)
        print(outputs[0][1])
```
**EN:** Checks Online Quantization under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv` before asserting the expected outcome.
**CN:** 该测试用例验证 Online Quantization 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv` 驱动目标逻辑，再断言预期结果。

### Test: test_online_quant_peak_mem (lines 204-275)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("fp8"),
    reason="FP8 is not supported on this GPU type.",
)
def test_online_quant_peak_mem(
    vllm_runner,
    caplog_mp_spawn,
    monkeypatch,
) -> None:
    # Note: `allenai/OLMoE-1B-7B-0125-Instruct` was selected because:
    # 1. it covers both Linear and MoE paths
    # 2. it is already used by other tests in CI, so adding it here
    #    does not increase disk space for CI runners
    # I really wanted to use `ibm-granite/granite-3.0-1b-a400m-base`
    # which I think is the smallest MoE model in vLLM (2.5 GiB bf16,
    # 1.3 GiB fp8), but could not as adding one more model makes CI
    # run out of disk space.
    model_name = "allenai/OLMoE-1B-7B-0125-Instruct"

# ... omitted for brevity ...
    expected_model_memory_gib = 6.7

    # for allenai/OLMoE-1B-7B-0125-Instruct the number we see today is 9.06
    # GiB, which is 1.36x above model_memory_gib. A slightly higher number is
    # expected as when we load and quantize weights in a streaming fashion we
    # need to have individual weights in bf16 + fp8 alive at the same time.
    expected_peak_memory_gib = expected_model_memory_gib * 1.4

    assert model_memory_gib < expected_model_memory_gib, (
        f"{model_memory_gib=} higher than {expected_model_memory_gib}"
    )
    assert peak_memory_gib < expected_peak_memory_gib, (
        f"{peak_memory_gib=} higher than {expected_peak_memory_gib}"
    )
```
**EN:** Checks Online Quant Peak Mem under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `monkeypatch.setenv`, `log_text.splitlines` before asserting the expected outcome.
**CN:** 该测试用例验证 Online Quant Peak Mem 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `monkeypatch.setenv`, `log_text.splitlines` 驱动目标逻辑，再断言预期结果。

### Test: test_online_quant_load_format_dummy (lines 278-294)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("fp8"),
    reason="FP8 is not supported on this GPU type.",
)
def test_online_quant_load_format_dummy(
    vllm_runner,
    monkeypatch,
    caplog,
) -> None:
    with vllm_runner(
        "ibm-granite/granite-3.0-1b-a400m-base",
        quantization="fp8",
        enforce_eager=True,
        load_format="dummy",
    ) as llm:
        outputs = llm.generate_greedy(["The future of AI is"], max_tokens=4)
        print(outputs[0][1])
```
**EN:** Checks Online Quant Load Format Dummy under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `vllm_runner`, `llm.generate_greedy` before asserting the expected outcome.
**CN:** 该测试用例验证 Online Quant Load Format Dummy 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `vllm_runner`, `llm.generate_greedy` 驱动目标逻辑，再断言预期结果。

### Test: test_scaled_fp8_quant (lines 297-375)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("fp8"),
    reason="FP8 is not supported on this GPU type.",
)
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
def test_scaled_fp8_quant(dtype) -> None:
    def quantize_ref(tensor, inv_scale):
        # The reference implementation that fully aligns to
        # the kernel being tested.
        finfo = torch.finfo(current_platform.fp8_dtype())
        scale = inv_scale.reciprocal()
        qweight = (tensor.to(torch.float32) * scale).clamp(min=finfo.min, max=finfo.max)
        qweight = qweight.to(current_platform.fp8_dtype())
        return qweight

    def per_tensor_dequantize(tensor, inv_scale, dtype):
        fake_qweight = tensor.to(dtype)
        dq_weight = fake_qweight * inv_scale
        return dq_weight
# ... omitted for brevity ...
    y_nc, _ = ops.scaled_fp8_quant(x_nc, inv_scale_nc)
    torch.testing.assert_close(
        ref_y_nc, per_tensor_dequantize(y_nc, inv_scale_nc, dtype)
    )

    # padding after non-contiguous input quantization
    y_nc_pad, _ = ops.scaled_fp8_quant(x_nc, inv_scale_nc, num_token_padding=m + 10)
    assert y_nc_pad.shape[0] == m + 10
    torch.testing.assert_close(
        ref_y_nc,
        per_tensor_dequantize(
            torch.narrow(y_nc_pad, 0, 0, x_nc.shape[0]), inv_scale_nc, dtype
        ),
    )
```
**EN:** Checks Scaled FP8 Quant under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skipif`, `pytest.mark.parametrize`, `(torch.randn(size=(11, 11), device=DEVICE_TYPE) * 13).to` before asserting the expected outcome.
**CN:** 该测试用例验证 Scaled FP8 Quant 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skipif`, `pytest.mark.parametrize`, `(torch.randn(size=(11, 11), device=DEVICE_TYPE) * 13).to` 驱动目标逻辑，再断言预期结果。

### Test: test_fp8_reloading (lines 378-475)
```python
@pytest.mark.skipif(
    current_platform.is_fp8_fnuz(),
    reason="FP8 e4m3fn weight reloading is not supported on e4m3fnuz platforms",
)
@pytest.mark.parametrize("method_cls", [Fp8LinearMethod, Fp8MoEMethod])
# FP8 weight reloading does not support online quantization
@pytest.mark.parametrize("is_checkpoint_fp8_serialized", [True])  # skip False
@pytest.mark.parametrize("weight_block_size", [None, [1, 1]])
# any postprocessing that is applied to the weights such as padding and repacking
# (excluding device sharding) must also be applied to the reloaded weights
#
# this is the case for marlin as well as per-tensor Fp8MoEMethod
@pytest.mark.parametrize("use_marlin", [False])  # skip True
def test_fp8_reloading(
    default_vllm_config,
    method_cls,
    is_checkpoint_fp8_serialized,
    weight_block_size,
    use_marlin,
# ... omitted for brevity ...
        weight_loader = getattr(param, "weight_loader", default_weight_loader)
        weight_loader(param, torch.zeros(shape))  # cannot use empty

    method.process_weights_after_loading(layer)

    # test reloading works after loading
    # assuming that no reshaping occurred
    for name, shape, original_weight_loader in original_metadata:
        param = getattr(layer, name)
        weight_loader = getattr(param, "weight_loader", default_weight_loader)
        assert weight_loader is original_weight_loader
        weight_loader(param, torch.zeros(shape))  # cannot use empty

    method.process_weights_after_loading(layer)
```
**EN:** Checks FP8 Reloading under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv` before asserting the expected outcome.
**CN:** 该测试用例验证 FP8 Reloading 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv` 驱动目标逻辑，再断言预期结果。

### Test: test_kv_cache_dtype_skip_layers (lines 478-498)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("fp8"),
    reason="FP8 is not supported on this GPU type.",
)
def test_kv_cache_dtype_skip_layers(vllm_runner, monkeypatch):
    """Test that kv_cache_dtype_skip_layers skips quantization for specified layers."""
    monkeypatch.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")

    with vllm_runner(
        "facebook/opt-125m",
        kv_cache_dtype="fp8",
        kv_cache_dtype_skip_layers=["0", "2"],
        enforce_eager=True,
    ) as llm:

        def check_layers(model):
            for i, layer in enumerate(model.model.decoder.layers):
                expected = "auto" if str(i) in ["0", "2"] else "fp8"
                assert layer.self_attn.attn.kv_cache_dtype == expected

        llm.apply_model(check_layers)
```
**EN:** Test that kv_cache_dtype_skip_layers skips quantization for specified layers. The body exercises logic via `pytest.mark.skipif`, `monkeypatch.setenv`, `vllm_runner` before asserting the expected outcome.
**CN:** 该测试用例验证 KV Cache Dtype Skip Layers 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `monkeypatch.setenv`, `vllm_runner` 驱动目标逻辑，再断言预期结果。

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
- **Standard library / 标准库**: `logging`
- **Third-party / 第三方依赖**: `pytest`, `regex`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.config.model`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.quantization.fp8`, `vllm.model_executor.model_loader.weight_utils`, `vllm.platforms`
- **Local test utilities / 本地测试辅助**: `tests.quantization.utils`
