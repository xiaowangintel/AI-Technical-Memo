# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/quantization/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Utils behavior in the Quantization test area through focused pytest scenarios. It focuses on scenarios such as Is Quant Method Supported, Test Online Quant Peak Mem Impl. / 该文件在 Quantization 测试域中，通过有针对性的 pytest 场景验证 Utils 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-9)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import logging

import regex as re

from vllm.model_executor.layers.quantization import get_quantization_config
from vllm.platforms import current_platform
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `logging`, `regex`, `vllm.model_executor.layers.quantization`, `vllm.platforms`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: is_quant_method_supported (lines 12-27)
```python
def is_quant_method_supported(quant_method: str) -> bool:
    # Currently, all quantization methods require Nvidia or AMD GPUs
    if not (current_platform.is_cuda() or current_platform.is_rocm()):
        return False

    try:
        current_platform.verify_quantization(quant_method)
    except ValueError:
        return False

    capability = current_platform.get_device_capability()
    assert capability is not None

    min_capability = get_quantization_config(quant_method).get_min_capability()

    return capability.to_int() >= min_capability
```
**EN:** Implements a reusable helper for Is Quant Method Supported, reducing duplication across related tests. It coordinates operations such as `current_platform.get_device_capability`, `get_quantization_config(quant_method).get_min_capability`, `current_platform.verify_quantization`.
**CN:** 该辅助函数为 Is Quant Method Supported 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `current_platform.get_device_capability`, `get_quantization_config(quant_method).get_min_capability`, `current_platform.verify_quantization` 等操作。

### Helper: _test_online_quant_peak_mem_impl (lines 30-97)
```python
def _test_online_quant_peak_mem_impl(
    quantization_arg_value,
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

    # Force spawn to ensure caplog_mp_spawn works consistently
    # (it relies on VLLM_LOGGING_CONFIG_PATH which spawn reads but fork ignores)
    monkeypatch.setenv("VLLM_WORKER_MULTIPROC_METHOD", "spawn")
# ... omitted for brevity ...

    # for allenai/OLMoE-1B-7B-0125-Instruct the number we see today is 9.06
    # GiB on CUDA, which is 1.36x above model_memory_gib. A slightly higher
    # number is expected as when we load and quantize weights in a streaming
    # fashion we need to have individual weights in bf16 + fp8 alive at the
    # same time.
    expected_peak_memory_gib = expected_model_memory_gib * 1.4

    assert model_memory_gib < expected_model_memory_gib, (
        f"{model_memory_gib=} higher than {expected_model_memory_gib}"
    )
    assert peak_memory_gib < expected_peak_memory_gib, (
        f"{peak_memory_gib=} higher than {expected_peak_memory_gib}"
    )
```
**EN:** Implements a reusable helper for Test Online Quant Peak Mem Impl, reducing duplication across related tests. It coordinates operations such as `monkeypatch.setenv`, `log_text.splitlines`, `print`.
**CN:** 该辅助函数为 Test Online Quant Peak Mem Impl 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `monkeypatch.setenv`, `log_text.splitlines`, `print` 等操作。

## Key Concepts / 关键概念
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Quantization / 量化**
  - **EN:** The file covers low-precision model paths, kernels, or configuration rules for quantized inference.
  - **CN:** 该文件覆盖低精度模型路径、相关内核或量化推理配置规则。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`
- **Third-party / 第三方依赖**: `regex`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.quantization`, `vllm.platforms`
