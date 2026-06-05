# test_accuracy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/nixl_integration/test_accuracy.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `accuracy` behavior and regressions in the v1 stack. / 验证 v1 栈中 `accuracy` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-6)
```python
import os

import lm_eval
import openai
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `lm_eval, openai`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `lm_eval, openai`。

### Module state / 模块级状态 (lines 8-32)
```python
BASE_URL = "http://localhost:8192/v1"
NUM_CONCURRENT = 100
TASK = "gsm8k"
FILTER = "exact_match,strict-match"
RTOL = 0.03

# Model-specific expected values
EXPECTED_VALUES = {
    "Qwen/Qwen3-0.6B": 0.41,
    "deepseek-ai/deepseek-vl2-small": 0.59,
    "deepseek-ai/deepseek-vl2-tiny": 0.19,
    "deepseek-ai/DeepSeek-V2-Lite-Chat": 0.65,
    "google/gemma-3-4b-it": 0.74,
    "nvidia/NVIDIA-Nemotron-3-Nano-30B-A3B-FP8": 0.84,
    "ibm-granite/granite-4.0-h-tiny": 0.80,
    "Qwen/Qwen3.5-0.8B": 0.33,
}

SIMPLE_PROMPT = (
    "The best part about working on vLLM is that I got to meet so many people across "
    "various different organizations like UCB, Google, and Meta which means",
)

# Get model name from environment variable
MODEL_NAME = os.environ.get("TEST_MODEL", "Qwen/Qwen3-0.6B")
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `BASE_URL, NUM_CONCURRENT, TASK, FILTER, RTOL, EXPECTED_VALUES, ...`. Shared setup calls include `environ.get`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`BASE_URL, NUM_CONCURRENT, TASK, FILTER, RTOL, EXPECTED_VALUES, ...`。 共享初始化调用包括 `environ.get`。

### run_simple_prompt (lines 35-42)
```python
def run_simple_prompt():
    client = openai.OpenAI(api_key="EMPTY", base_url=BASE_URL)
    completion = client.completions.create(model=MODEL_NAME, prompt=SIMPLE_PROMPT)

    print("-" * 50)
    print(f"Completion results for {MODEL_NAME}:")
    print(completion)
    print("-" * 50)
```
**EN:** Helper function `run_simple_prompt` encapsulates reusable logic for `run simple prompt`. Key calls include `openai.OpenAI, completions.create, print`.
**CN:** 辅助函数 `run_simple_prompt` 封装了与 `run simple prompt` 相关的可复用逻辑。 关键调用包括 `openai.OpenAI, completions.create, print`。

### test_accuracy (lines 45-75)
```python
def test_accuracy():
    """Run the end to end accuracy test."""
    run_simple_prompt()

    model_args = (
        f"model={MODEL_NAME},"
        f"base_url={BASE_URL}/completions,"
        f"num_concurrent={NUM_CONCURRENT},tokenized_requests=False"
    )

    results = lm_eval.simple_evaluate(
        model="local-completions",
        model_args=model_args,
        tasks=TASK,
    )

    measured_value = results["results"][TASK][FILTER]
    expected_value = EXPECTED_VALUES.get(MODEL_NAME)

    if expected_value is None:
        print(
            f"Warning: No expected value found for {MODEL_NAME}. "
            "Skipping accuracy check."
        )
        print(f"Measured value: {measured_value}")
        return

    assert (
        measured_value - RTOL < expected_value
        and measured_value + RTOL > expected_value
    ), f"Expected: {expected_value} | Measured: {measured_value}"
```
**EN:** Test case covering `accuracy`. It exercises `run_simple_prompt, lm_eval.simple_evaluate, EXPECTED_VALUES.get, print`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `accuracy` 的测试用例。 该测试会调用 `run_simple_prompt, lm_eval.simple_evaluate, EXPECTED_VALUES.get, print`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径

## Dependencies / 依赖关系
- **EN:** External libraries: `lm_eval, openai`.
- **CN:** 外部库：`lm_eval, openai`。
- **EN:** Standard-library support: `os`.
- **CN:** 标准库支持：`os`。
