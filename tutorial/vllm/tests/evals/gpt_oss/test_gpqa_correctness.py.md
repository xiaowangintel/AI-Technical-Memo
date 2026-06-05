# test_gpqa_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/evals/gpt_oss/test_gpqa_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: GPQA evaluation using vLLM server and GPT-OSS evaluation package. / 该文件主要围绕 Gpqa Correctness 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-32)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
GPQA evaluation using vLLM server and GPT-OSS evaluation package.

Usage:
pytest -s -v tests/evals/gpt_oss/test_gpqa_correctness.py \
    --config-list-file=configs/models-h200.txt
"""

import os
import shlex
import subprocess
import sys
import urllib.request
from pathlib import Path

import regex as re
import yaml

from tests.utils import RemoteOpenAIServer

TOL = 0.05  # Absolute tolerance for accuracy comparison

# Path to tiktoken encoding files
TIKTOKEN_DATA_DIR = Path(__file__).parent / "data"

# Tiktoken encoding files to download
TIKTOKEN_FILES = {
    "cl100k_base.tiktoken": "https://openaipublic.blob.core.windows.net/encodings/cl100k_base.tiktoken",
    "o200k_base.tiktoken": "https://openaipublic.blob.core.windows.net/encodings/o200k_base.tiktoken",
}
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `os`, `shlex`, `regex`, `yaml`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: ensure_tiktoken_files (lines 35-46)
```python
def ensure_tiktoken_files():
    """Download tiktoken encoding files if they don't exist."""
    TIKTOKEN_DATA_DIR.mkdir(parents=True, exist_ok=True)

    for filename, url in TIKTOKEN_FILES.items():
        filepath = TIKTOKEN_DATA_DIR / filename
        if not filepath.exists():
            print(f"Downloading {filename} from {url}...")
            urllib.request.urlretrieve(url, filepath)
            print(f"  Downloaded to {filepath}")
        else:
            print(f"  {filename} already exists.")
```
**EN:** Download tiktoken encoding files if they don't exist. It coordinates operations such as `TIKTOKEN_DATA_DIR.mkdir`, `TIKTOKEN_FILES.items`, `filepath.exists`.
**CN:** 该辅助函数为 Ensure Tiktoken Files 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `TIKTOKEN_DATA_DIR.mkdir`, `TIKTOKEN_FILES.items`, `filepath.exists` 等操作。

### Helper: run_gpqa_eval (lines 49-105)
```python
def run_gpqa_eval(model_name: str, base_url: str, reasoning_effort: str) -> float:
    """Run GPQA evaluation using the gpt-oss evaluation package."""

    # Build the command to run the evaluation
    cmd = [
        sys.executable,
        "-m",
        "gpt_oss.evals",
        "--eval",
        "gpqa",
        "--model",
        model_name,
        "--reasoning-effort",
        reasoning_effort,
        "--base-url",
        base_url,
        "--n-threads",
        "200",
    ]
# ... omitted for brevity ...
            )

        # Parse the output to extract the score
        match = re.search(r"'metric':\s*([\d.]+)", result.stdout)
        if match:
            return float(match.group(1))

        # If we still can't find it, raise an error
        raise ValueError(
            f"Could not parse score from evaluation output:\n{result.stdout}"
        )

    except subprocess.TimeoutExpired as e:
        raise RuntimeError("Evaluation timed out") from e
```
**EN:** Run GPQA evaluation using the gpt-oss evaluation package. It coordinates operations such as `os.environ.copy`, `subprocess.run`, `print`.
**CN:** 该辅助函数为 Run Gpqa Eval 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `os.environ.copy`, `subprocess.run`, `print` 等操作。

### Test: test_gpqa_correctness (lines 108-172)
```python
def test_gpqa_correctness(config_filename):
    """Test GPQA correctness for a given model configuration."""
    # Ensure tiktoken files are downloaded
    ensure_tiktoken_files()

    # Verify tiktoken files exist
    for filename in TIKTOKEN_FILES:
        filepath = TIKTOKEN_DATA_DIR / filename
        assert filepath.exists(), f"Tiktoken file not found: {filepath}"

    eval_config = yaml.safe_load(config_filename.read_text(encoding="utf-8"))

    # Parse server arguments from config (use shlex to handle quoted strings)
    server_args_str = eval_config.get("server_args", "")
    server_args = shlex.split(server_args_str) if server_args_str else []

    # Add standard server arguments
    server_args.extend(
        [
# ... omitted for brevity ...
        expected_metric = eval_config["metric_threshold"]

        print(f"GPQA Results for {eval_config['model_name']}:")
        print(f"  Measured metric: {measured_metric:.4f}")
        print(f"  Expected metric: {expected_metric:.4f}")
        print(f"  Tolerance: {TOL:.4f}")

        # Verify metric is within tolerance
        assert measured_metric >= expected_metric - TOL, (
            f"GPQA metric too low: {measured_metric:.4f} < "
            f"{expected_metric:.4f} - {TOL:.4f} = {expected_metric - TOL:.4f}"
        )

        print(f"GPQA test passed for {eval_config['model_name']}")
```
**EN:** Test GPQA correctness for a given model configuration. The body exercises logic via `ensure_tiktoken_files`, `yaml.safe_load`, `eval_config.get` before asserting the expected outcome.
**CN:** 该测试用例验证 Gpqa Correctness 在特定场景下的行为。 函数体会先通过 `ensure_tiktoken_files`, `yaml.safe_load`, `eval_config.get` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。
- **Reasoning traces / 推理轨迹**
  - **EN:** The tests inspect reasoning-specific formats or parser behavior in intermediate outputs.
  - **CN:** 这些测试检查中间输出中的推理格式或解析器行为。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `shlex`, `subprocess`, `sys`, `urllib.request`, `pathlib`
- **Third-party / 第三方依赖**: `regex`, `yaml`
- **Local test utilities / 本地测试辅助**: `tests.utils`
