# test_gsm8k_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/evals/gsm8k/test_gsm8k_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: GSM8K evaluation using vLLM server and isolated GSM8K script. / 该文件主要围绕 GSM8K Correctness 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-20)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
GSM8K evaluation using vLLM server and isolated GSM8K script.
Replacement for lm-eval-harness with better performance and control.

Usage:
pytest -s -v tests/evals/gsm8k/test_gsm8k_correctness.py \
    --config-list-file=configs/models-small.txt
"""

import shlex

import pytest
import yaml

from tests.utils import RemoteOpenAIServer
from vllm.platforms import current_platform

from .gsm8k_eval import evaluate_gsm8k
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `shlex`, `pytest`, `yaml`, `vllm.platforms`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: run_gsm8k_eval (lines 23-49)
```python
def run_gsm8k_eval(eval_config: dict, server_url: str) -> dict:
    """Run GSM8K evaluation using our isolated script."""
    # Extract host and port from server URL
    if "://" in server_url:
        server_url = server_url.split("://")[1]

    host_port = server_url.split("/")[0]  # Remove path if present
    if ":" in host_port:
        host, p = host_port.split(":")
        port = int(p)
    else:
        host = host_port
        port = 8000

    # Add http:// prefix if not present
    if not host.startswith("http"):
        host = f"http://{host}"

    # Run GSM8K evaluation
    results = evaluate_gsm8k(
        num_questions=eval_config["num_questions"],
        num_shots=eval_config["num_fewshot"],
        host=host,
        port=port,
    )

    return results
```
**EN:** Run GSM8K evaluation using our isolated script. It coordinates operations such as `evaluate_gsm8k`, `server_url.split`, `host_port.split`.
**CN:** 该辅助函数为 Run GSM8K Eval 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `evaluate_gsm8k`, `server_url.split`, `host_port.split` 等操作。

### Test: test_gsm8k_correctness (lines 52-126)
```python
def test_gsm8k_correctness(config_filename):
    """Test GSM8K correctness for a given model configuration."""
    eval_config = yaml.safe_load(config_filename.read_text(encoding="utf-8"))

    if (
        not current_platform.is_cuda()
        and "Qwen3-30B-A3B-MXFP4A16" in eval_config["model_name"]
    ):
        pytest.skip(
            "Skipping Qwen3-30B-A3B-MXFP4A16 on non-CUDA platforms. "
            "Marlin kernels are not supported."
        )

    # TODO(akaratza): Enable DeepSeek-V3.2 and DeepSeek-R1 on ROCm platforms
    if current_platform.is_rocm() and (
        "deepseek-ai/DeepSeek-V3.2" in eval_config["model_name"]
        or "deepseek-ai/DeepSeek-R1" in eval_config["model_name"]
    ):
        pytest.skip(
# ... omitted for brevity ...
        print(f"  Measured metric: {measured_metric:.4f}")
        print(f"  Expected metric: {expected_metric:.4f}")
        print(f"  Tolerance: {tol:.4f}")
        print(f"  Questions: {results['num_questions']}")
        print(f"  Invalid rate: {results['invalid_rate']:.3f}")
        print(f"  Latency: {results['latency']:.1f}s")
        print(f"  QPS: {results['questions_per_second']:.1f}")

        assert measured_metric >= expected_metric - tol, (
            f"GSM8K metric too low: {measured_metric:.4f} < "
            f"{expected_metric:.4f} - {tol:.4f} = {expected_metric - tol:.4f}"
        )

        print(f"✅ GSM8K test passed for {eval_config['model_name']}")
```
**EN:** Test GSM8K correctness for a given model configuration. The body exercises logic via `yaml.safe_load`, `eval_config.get`, `server_args.extend` before asserting the expected outcome.
**CN:** 该测试用例验证 GSM8K Correctness 在特定场景下的行为。 函数体会先通过 `yaml.safe_load`, `eval_config.get`, `server_args.extend` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Benchmarking / 基准测试**
  - **EN:** The tests validate CLI arguments, datasets, or metrics used by benchmark workflows.
  - **CN:** 测试验证基准流程使用的命令行参数、数据集或指标。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `shlex`
- **Third-party / 第三方依赖**: `pytest`, `yaml`
- **vLLM internal / vLLM 内部依赖**: `vllm.platforms`
- **Local test utilities / 本地测试辅助**: `tests.utils`, `.gsm8k_eval`
