# test_mixed_precision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/quantization/test_mixed_precision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Test quark-quantized {MXFP4, FP8} mixed precision models. / 该文件主要围绕 Mixed Precision 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-20)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Test quark-quantized {MXFP4, FP8} mixed precision models.

Run `pytest tests/quantization/test_mixed_precision.py`.

"""

import importlib
import importlib.metadata
import importlib.util
from dataclasses import dataclass

import lm_eval
import pytest
from packaging import version

QUARK_MXFP4_AVAILABLE = importlib.util.find_spec("quark") is not None and version.parse(
    importlib.metadata.version("amd-quark")
) >= version.parse("0.8.99")
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `importlib`, `importlib.metadata`, `lm_eval`, `pytest`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: ModelCase (lines 23-26)
```python
@dataclass
class ModelCase:
    model_id: str
    tp: int
```
**EN:** Groups related scenarios for Modelcase.
**CN:** 该类把与 Modelcase 相关的场景组织在一起。

### Class: EvaluationConfig (lines 29-37)
```python
@dataclass
class EvaluationConfig:
    model_name: str

    def get_model_args(self) -> str:
        return (
            f"pretrained={self.model_name},"
            "tensor_parallel_size=4,dtype=auto,gpu_memory_utilization=0.8,trust_remote_code=False"
        )
```
**EN:** Groups related scenarios for Evaluationconfig.
**CN:** 该类把与 Evaluationconfig 相关的场景组织在一起。

### Constants / assignments (lines 40-50)
```python
TEST_CONFIGS = {
    # Mixed-precision (AMP) model
    # - Demonstrates end-to-end pipeline functionality
    "amd/Qwen3-8B-WMXFP4FP8-AMXFP4FP8-AMP-KVFP8": {"arc_challenge": 0.52, "mmlu": 0.72},
    # Non-mixed-precision (PTQ) model
    # - Reference for pipeline compatibility verification -> No conflicts or breakings
    "amd/Llama-2-70b-chat-hf_FP8_MLPerf_V2": {
        "arc_challenge": 0.53,
        "mmlu": 0.61,
    },
}
```
**EN:** Defines shared constants or configuration objects like `TEST_CONFIGS`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `TEST_CONFIGS`），供后续测试重复使用。

### Test: test_mixed_precision_model_accuracies (lines 53-70)
```python
@pytest.mark.parametrize("model_name, accuracy_numbers", TEST_CONFIGS.items())
@pytest.mark.skipif(not QUARK_MXFP4_AVAILABLE, reason="amd-quark>=0.9 is not available")
def test_mixed_precision_model_accuracies(model_name: str, accuracy_numbers: dict):
    results = lm_eval.simple_evaluate(
        model="vllm",
        model_args=EvaluationConfig(model_name).get_model_args(),
        tasks=list(accuracy_numbers.keys()),
        batch_size=8,
    )

    rtol = 0.05

    for task, expect_accuracy in accuracy_numbers.items():
        measured_accuracy = results["results"][task]["acc,none"]
        assert (
            measured_accuracy - rtol < expect_accuracy
            and measured_accuracy + rtol > expect_accuracy
        ), f"Expected: {expect_accuracy} |  Measured: {measured_accuracy}"
```
**EN:** Checks Mixed Precision Model Accuracies under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `pytest.mark.skipif`, `lm_eval.simple_evaluate` before asserting the expected outcome.
**CN:** 该测试用例验证 Mixed Precision Model Accuracies 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `pytest.mark.skipif`, `lm_eval.simple_evaluate` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
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
- **Standard library / 标准库**: `importlib`, `importlib.metadata`, `importlib.util`, `dataclasses`
- **Third-party / 第三方依赖**: `lm_eval`, `pytest`, `packaging`
