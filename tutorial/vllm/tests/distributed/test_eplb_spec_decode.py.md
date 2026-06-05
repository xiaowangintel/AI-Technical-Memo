# test_eplb_spec_decode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/distributed/test_eplb_spec_decode.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Eplb Spec Decode behavior in the Distributed test area through focused pytest scenarios. It focuses on scenarios such as Get Model Args, Eplb Spec Decode, Eplb Spec Decode Qwen3 Next Mtp Async. / 该文件在 Distributed 测试域中，通过有针对性的 pytest 场景验证 Eplb Spec Decode 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-9)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from __future__ import annotations

import lm_eval
import pytest

from tests.utils import large_gpu_mark
from vllm.platforms import current_platform
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `__future__`, `lm_eval`, `pytest`, `vllm.platforms`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: get_model_args (lines 12-46)
```python
def get_model_args(
    model_name: str,
    spec_model_name: str | None,
    spec_method: str,
    tp_size: int,
    model_max_len: int,
    use_async: bool = False,
) -> dict:
    speculative_config = {
        "method": spec_method,
        "model": spec_model_name,
        "num_speculative_tokens": 1,
        "max_model_len": model_max_len,
    }
    eplb_config = {
        "num_redundant_experts": tp_size,
        "window_size": 128,
        "step_interval": 1024,
        "log_balancedness": False,
# ... omitted for brevity ...
        eplb_config["use_async"] = True
    model_args = {
        "pretrained": model_name,
        "dtype": "auto",
        "add_bos_token": True,
        "tensor_parallel_size": tp_size,
        "gpu_memory_utilization": 0.7,
        "speculative_config": speculative_config,
        "enable_expert_parallel": True,
        "eplb_config": eplb_config,
        "enable_eplb": True,
        "max_model_len": model_max_len,
    }
    return model_args
```
**EN:** Implements a reusable helper for Get Model Args, reducing duplication across related tests.
**CN:** 该辅助函数为 Get Model Args 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Constants / assignments (lines 49-52)
```python
pytestmark = pytest.mark.skipif(
    current_platform.is_rocm(),
    reason="EPLB with Spec Decode is a work in progress on ROCm.",
)
```
**EN:** Defines shared constants or configuration objects like `pytestmark`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `pytestmark`），供后续测试重复使用。

### Test: test_eplb_spec_decode (lines 55-108)
```python
@pytest.mark.parametrize(
    "model_setup",
    [
        pytest.param(
            ("mtp", "Qwen/Qwen3-Next-80B-A3B-Instruct", None, 4, 0.86),
            marks=large_gpu_mark(min_gb=80),
        ),
        pytest.param(
            (
                "eagle",
                "meta-llama/Llama-4-Scout-17B-16E-Instruct",
                "morgendave/EAGLE-Llama-4-Scout-17B-16E-Instruct",
                4,
                0.92,
            ),
            marks=pytest.mark.skip(reason="Skipping due to CI OOM issues"),
        ),
    ],
    ids=["qwen3_next_mtp", "llama4_eagle"],
# ... omitted for brevity ...
    )

    results = lm_eval.simple_evaluate(
        model="vllm",
        model_args=model_args,
        tasks=TASK,
        batch_size=64,
        num_fewshot=8,
    )
    measured_value = results["results"][TASK][FILTER]
    assert (
        measured_value - RTOL < expected_gsm8k_value
        and measured_value + RTOL > expected_gsm8k_value
    ), f"Expected: {expected_gsm8k_value} |  Measured: {measured_value}"
```
**EN:** Test the correctness of EPLB speculative decoding with GSM8K dataset. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `get_model_args`, `lm_eval.simple_evaluate` before asserting the expected outcome.
**CN:** 该测试用例验证 Eplb Spec Decode 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `get_model_args`, `lm_eval.simple_evaluate` 驱动目标逻辑，再断言预期结果。

### Test: test_eplb_spec_decode_qwen3_next_mtp_async (lines 111-142)
```python
@large_gpu_mark(min_gb=80)
def test_eplb_spec_decode_qwen3_next_mtp_async() -> None:
    """
    Ensure async EPLB works with MTP speculative decoding for Qwen3-Next.
    """

    TASK = "gsm8k"
    FILTER = "exact_match,strict-match"
    RTOL = 0.03
    expected_gsm8k_value = 0.86

    model_args = get_model_args(
        model_name="Qwen/Qwen3-Next-80B-A3B-Instruct",
        spec_model_name=None,
        spec_method="mtp",
        tp_size=4,
        model_max_len=4096,
        use_async=True,
    )

    results = lm_eval.simple_evaluate(
        model="vllm",
        model_args=model_args,
        tasks=TASK,
        batch_size=64,
        num_fewshot=8,
    )
    measured_value = results["results"][TASK][FILTER]
    assert (
        measured_value - RTOL < expected_gsm8k_value
        and measured_value + RTOL > expected_gsm8k_value
    ), f"Expected: {expected_gsm8k_value} |  Measured: {measured_value}"
```
**EN:** Ensure async EPLB works with MTP speculative decoding for Qwen3-Next. The body exercises logic via `large_gpu_mark`, `get_model_args`, `lm_eval.simple_evaluate` before asserting the expected outcome.
**CN:** 该测试用例验证 Eplb Spec Decode Qwen3 Next Mtp Async 在特定场景下的行为。 函数体会先通过 `large_gpu_mark`, `get_model_args`, `lm_eval.simple_evaluate` 驱动目标逻辑，再断言预期结果。

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
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`
- **Third-party / 第三方依赖**: `lm_eval`, `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.platforms`
- **Local test utilities / 本地测试辅助**: `tests.utils`
