# test_elastic_ep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/distributed/test_elastic_ep.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Elastic Ep behavior in the Distributed test area through focused pytest scenarios. It focuses on scenarios such as Cleanup Ray Between Tests, Send Scale Command, Run GSM8K Eval. / 该文件在 Distributed 测试域中，通过有针对性的 pytest 场景验证 Elastic Ep 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-12)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import os
import subprocess
import time

import pytest
import requests

from ..evals.gsm8k.gsm8k_eval import evaluate_gsm8k
from ..utils import RemoteOpenAIServer, multi_gpu_test
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `os`, `subprocess`, `pytest`, `requests`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: cleanup_ray_between_tests (lines 15-20)
```python
@pytest.fixture(autouse=True)
def cleanup_ray_between_tests():
    """Force-stop any lingering Ray processes between tests."""
    subprocess.run(["ray", "stop", "--force"], timeout=30, capture_output=True)
    time.sleep(5)
    yield
```
**EN:** Force-stop any lingering Ray processes between tests. The fixture mainly builds or returns values through `pytest.fixture`, `subprocess.run`, `time.sleep`.
**CN:** 该代码块定义 pytest 夹具 `cleanup_ray_between_tests`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `subprocess.run`, `time.sleep` 构造或返回测试所需的值。

### Constants / assignments (lines 23-23)
```python
MODEL_NAME = "deepseek-ai/DeepSeek-V2-Lite-Chat"
```
**EN:** Defines shared constants or configuration objects like `MODEL_NAME`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `MODEL_NAME`），供后续测试重复使用。

### Helper: _send_scale_command (lines 31-40)
```python
def _send_scale_command(server: RemoteOpenAIServer, new_dp_size: int) -> bool:
    url = server.url_for("scale_elastic_ep")
    payload = {"new_data_parallel_size": new_dp_size}
    headers = {"Content-Type": "application/json"}

    try:
        response = requests.post(url, json=payload, headers=headers, timeout=300)
        return response.status_code == 200
    except requests.exceptions.RequestException:
        return False
```
**EN:** Implements a reusable helper for Send Scale Command, reducing duplication across related tests. It coordinates operations such as `server.url_for`, `requests.post`.
**CN:** 该辅助函数为 Send Scale Command 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `server.url_for`, `requests.post` 等操作。

### Helper: _run_gsm8k_eval (lines 43-59)
```python
def _run_gsm8k_eval(server: RemoteOpenAIServer, stage: str) -> float:
    assert server.port is not None
    result = evaluate_gsm8k(
        num_questions=NUM_GSM8K_QUESTIONS,
        host=f"http://{server.host}",
        port=server.port,
    )
    accuracy = result["accuracy"]
    print(
        f"[{stage}] GSM8K accuracy: {accuracy:.3f} "
        f"({result['num_questions']} questions)"
    )
    assert accuracy >= EXPECTED_ACCURACY, (
        f"[{stage}] GSM8K accuracy {accuracy:.3f} is below "
        f"expected threshold {EXPECTED_ACCURACY}"
    )
    return accuracy
```
**EN:** Implements a reusable helper for Run GSM8K Eval, reducing duplication across related tests. It coordinates operations such as `evaluate_gsm8k`, `print`.
**CN:** 该辅助函数为 Run GSM8K Eval 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `evaluate_gsm8k`, `print` 等操作。

### Test: test_elastic_ep_scaling (lines 62-126)
```python
@multi_gpu_test(num_gpus=4)
def test_elastic_ep_scaling():
    vllm_serve_args = [
        "--trust-remote-code",
        "--tensor-parallel-size",
        "1",
        "--gpu-memory-utilization",
        "0.8",
        "--max-model-len",
        "4096",
        "--max-num-seqs",
        str(MAX_NUM_SEQS),
        "--enable-expert-parallel",
        "--all2all-backend",
        "allgather_reducescatter",
        "--enable-elastic-ep",
        "--enable-eplb",
        "--eplb-config.num_redundant_experts",
        "0",
# ... omitted for brevity ...
            f"{ACCURACY_TOL} below initial accuracy {initial_accuracy:.3f}"
        )

        print("\nAccuracy Summary:")
        print(f"  Initial:    {initial_accuracy:.3f}")
        print(
            f"  Scale up:   {scale_up_accuracy:.3f} "
            f"(diff: {scale_up_accuracy - initial_accuracy:+.3f})"
        )
        print(
            f"  Scale down: {scale_down_accuracy:.3f} "
            f"(diff: {scale_down_accuracy - initial_accuracy:+.3f})"
        )
        print(f"  Tolerance:  {ACCURACY_TOL:.3f}")
```
**EN:** Checks Elastic Ep Scaling under a focused test scenario. The body exercises logic via `multi_gpu_test`, `os.environ.get`, `str` before asserting the expected outcome.
**CN:** 该测试用例验证 Elastic Ep Scaling 在特定场景下的行为。 函数体会先通过 `multi_gpu_test`, `os.environ.get`, `str` 驱动目标逻辑，再断言预期结果。

### Test: test_elastic_ep_scaling_uneven (lines 129-202)
```python
@multi_gpu_test(num_gpus=4)
def test_elastic_ep_scaling_uneven():
    """Test scale up with uneven worker distribution.

    This tests the case where num_new_workers % old_dp_size != 0,
    specifically 2 -> 3 where remainder = 1 % 2 = 1.
    This exercises the remainder handling in sender-receiver pairing.
    """
    vllm_serve_args = [
        "--trust-remote-code",
        "--tensor-parallel-size",
        "1",
        "--gpu-memory-utilization",
        "0.8",
        "--max-model-len",
        "4096",
        "--max-num-seqs",
        str(MAX_NUM_SEQS),
        "--enable-expert-parallel",
# ... omitted for brevity ...
            f"{ACCURACY_TOL} below initial accuracy {initial_accuracy:.3f}"
        )

        print("\nAccuracy Summary (Uneven Scaling):")
        print(f"  Initial:    {initial_accuracy:.3f}")
        print(
            f"  Scale up:   {scale_up_accuracy:.3f} "
            f"(diff: {scale_up_accuracy - initial_accuracy:+.3f})"
        )
        print(
            f"  Scale down: {scale_down_accuracy:.3f} "
            f"(diff: {scale_down_accuracy - initial_accuracy:+.3f})"
        )
        print(f"  Tolerance:  {ACCURACY_TOL:.3f}")
```
**EN:** Test scale up with uneven worker distribution. The body exercises logic via `multi_gpu_test`, `os.environ.get`, `str` before asserting the expected outcome.
**CN:** 该测试用例验证 Elastic Ep Scaling Uneven 在特定场景下的行为。 函数体会先通过 `multi_gpu_test`, `os.environ.get`, `str` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `subprocess`, `time`
- **Third-party / 第三方依赖**: `pytest`, `requests`
- **Local test utilities / 本地测试辅助**: `..evals.gsm8k.gsm8k_eval`, `..utils`
