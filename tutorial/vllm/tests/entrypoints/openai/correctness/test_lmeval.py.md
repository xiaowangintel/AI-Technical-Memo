# test_lmeval.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/correctness/test_lmeval.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers OpenAI-compatible serving. The file defines 1 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖OpenAI 兼容服务。它定义了 1 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L12-L16)
```python
import lm_eval

from vllm.platforms import current_platform

from ....utils import RemoteOpenAIServer
```
**EN:** Imports third-party packages like `lm_eval`, project helpers such as `vllm.platforms.current_platform`, `....utils.RemoteOpenAIServer`.
**CN:** 导入第三方包（如 `lm_eval`）、项目内辅助模块（如 `vllm.platforms.current_platform`、`....utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME, NUM_CONCURRENT, TASK (L18-L35)
```python
MODEL_NAME = "Qwen/Qwen2-1.5B-Instruct"
NUM_CONCURRENT = 500
TASK = "gsm8k"
FILTER = "exact_match,strict-match"
RTOL = 0.03
EXPECTED_VALUE = 0.54
DEFAULT_ARGS = ["--max-model-len", "4096"]
MORE_ARGS_LIST = [
    [],  # Default
    ["--enable-chunked-prefill"],  # Chunked
]
MAX_WAIT_SECONDS = None

if current_platform.is_tpu():
    MORE_ARGS_LIST = [
        [],  # Default
    ]
    MAX_WAIT_SECONDS = 600
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `NUM_CONCURRENT`, `TASK`, `FILTER`, `RTOL`, `EXPECTED_VALUE`, `DEFAULT_ARGS`, `MORE_ARGS_LIST`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`NUM_CONCURRENT`、`TASK`、`FILTER`、`RTOL`、`EXPECTED_VALUE`、`DEFAULT_ARGS`、`MORE_ARGS_LIST`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: run_test (L38-L66)
```python
def run_test(more_args):
    """Run the end to end accuracy test."""

    args = list(DEFAULT_ARGS)
    args.extend(more_args)
    print(f"Running with: {args}")

    with RemoteOpenAIServer(
        MODEL_NAME, args, max_wait_seconds=MAX_WAIT_SECONDS
    ) as remote_server:
        url = f"{remote_server.url_for('v1')}/completions"

        model_args = (
            f"model={MODEL_NAME},"
            f"base_url={url},"
            f"num_concurrent={NUM_CONCURRENT},tokenized_requests=False"
        )

        results = lm_eval.simple_evaluate(
            model="local-completions",
            model_args=model_args,
            tasks=TASK,
        )

        measured_value = results["results"][TASK][FILTER]
        assert (
            measured_value - RTOL < EXPECTED_VALUE
            and measured_value + RTOL > EXPECTED_VALUE
        ), f"Expected: {EXPECTED_VALUE} |  Measured: {measured_value}"
```
**EN:** This helper encapsulates reusable logic in `run_test`. Key inputs are `more_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The main assertion is `measured_value - RTOL < EXPECTED_VALUE and measured_value + RTOL > EXPECTED_VALUE`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_test` 中。 关键输入包括 `more_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 核心断言是 `measured_value - RTOL < EXPECTED_VALUE and measured_value + RTOL > EXPECTED_VALUE`。

### Test / 测试: test_lm_eval_accuracy_v1_engine (L69-L78)
```python
def test_lm_eval_accuracy_v1_engine():
    """Run with the V1 Engine."""

    more_args = []

    # Limit compilation time for V1
    if current_platform.is_tpu():
        more_args = ["--max-num-seqs", "64"]

    run_test(more_args)
```
**EN:** This test validates `test_lm_eval_accuracy_v1_engine`.
**CN:** 这个测试验证 `test_lm_eval_accuracy_v1_engine`。

## Key Concepts / 关键概念
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `lm_eval`
- **Project / 项目内**: `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `....utils.RemoteOpenAIServer`
