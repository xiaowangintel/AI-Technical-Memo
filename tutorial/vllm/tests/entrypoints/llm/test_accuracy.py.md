# test_accuracy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/llm/test_accuracy.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers LLM entrypoint behavior. The file defines 2 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖LLM 入口行为。它定义了 2 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L12-L15)
```python
import lm_eval
import pytest

from vllm.platforms import current_platform
```
**EN:** Imports third-party packages like `lm_eval`, `pytest`, project helpers such as `vllm.platforms.current_platform`.
**CN:** 导入第三方包（如 `lm_eval`、`pytest`）、项目内辅助模块（如 `vllm.platforms.current_platform`）。

### Module setup / 模块级配置: MODEL_NAMES, FP8_KV_MODEL_NAMES, NUM_CONCURRENT (L17-L31)
```python
MODEL_NAMES = [
    "Qwen/Qwen3-1.7B",
    "google/gemma-3-1b-it",
]
FP8_KV_MODEL_NAMES = [
    "Qwen/Qwen3-1.7B",
]
NUM_CONCURRENT = 500
TASK = "gsm8k"
FILTER = "exact_match,strict-match"
RTOL = 0.03
EXPECTED_VALUES = {
    "Qwen/Qwen3-1.7B": 0.68,
    "google/gemma-3-1b-it": 0.25,
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAMES`, `FP8_KV_MODEL_NAMES`, `NUM_CONCURRENT`, `TASK`, `FILTER`, `RTOL`, `EXPECTED_VALUES`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAMES`、`FP8_KV_MODEL_NAMES`、`NUM_CONCURRENT`、`TASK`、`FILTER`、`RTOL`、`EXPECTED_VALUES`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: run_test (L34-L57)
```python
def run_test(model_name, more_args=None):
    """Run the end to end accuracy test."""

    model_args = f"pretrained={model_name},max_model_len=4096"

    if more_args is not None:
        model_args = "{},{}".format(model_args, more_args)

    results = lm_eval.simple_evaluate(
        model="vllm",
        model_args=model_args,
        tasks="gsm8k",
        batch_size="auto",
    )

    measured_value = results["results"][TASK][FILTER]
    assert model_name in EXPECTED_VALUES, (
        f"Cannot find the expected value for the model {model_name=}"
    )
    expected_value = EXPECTED_VALUES[model_name]
    assert (
        measured_value - RTOL < expected_value
        and measured_value + RTOL > expected_value
    ), f"Expected: {expected_value} |  Measured: {measured_value}"
```
**EN:** This helper encapsulates reusable logic in `run_test`. Key inputs are `model_name`, `more_args`. The main assertion is `model_name in EXPECTED_VALUES` and `measured_value - RTOL < expected_value and measured_value + RTOL > expected_value`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_test` 中。 关键输入包括 `model_name`、`more_args`。 核心断言是 `model_name in EXPECTED_VALUES` and `measured_value - RTOL < expected_value and measured_value + RTOL > expected_value`。

### Module setup / 模块级配置: TPU_TP_TEST_STR (L61-L61)
```python
TPU_TP_TEST_STR = ""  # "tensor_parallel_size=4"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `TPU_TP_TEST_STR`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `TPU_TP_TEST_STR`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_lm_eval_accuracy_v1_engine (L64-L78)
```python
@pytest.mark.parametrize("model", MODEL_NAMES)
def test_lm_eval_accuracy_v1_engine(model):
    """Run with the V1 Engine."""

    more_args = None
    if current_platform.is_tpu():
        # Limit compilation time for TPU V1

        more_args = "max_model_len=2048,max_num_seqs=64"

        # Add TP test (if provided)
        if TPU_TP_TEST_STR:
            more_args += ",{}".format(TPU_TP_TEST_STR)

    run_test(model, more_args)
```
**EN:** This test validates `test_lm_eval_accuracy_v1_engine`. It uses parameterization over `model`. Key inputs are `model`.
**CN:** 这个测试验证 `test_lm_eval_accuracy_v1_engine`。 它通过参数化组合 `model`。 关键输入包括 `model`。

### Test / 测试: test_lm_eval_accuracy_v1_engine_fp8_kv_cache (L81-L94)
```python
@pytest.mark.parametrize("model", FP8_KV_MODEL_NAMES)
def test_lm_eval_accuracy_v1_engine_fp8_kv_cache(model):
    """Run with the V1 Engine."""

    more_args = None
    if current_platform.is_tpu():
        # Limit compilation time for TPU V1
        more_args = "max_model_len=2048,max_num_seqs=128,kv_cache_dtype=fp8"

        # Add TP test (if provided)
        if TPU_TP_TEST_STR:
            more_args += ",{}".format(TPU_TP_TEST_STR)

    run_test(model, more_args)
```
**EN:** This test validates `test_lm_eval_accuracy_v1_engine_fp8_kv_cache`. It uses parameterization over `model`. Key inputs are `model`.
**CN:** 这个测试验证 `test_lm_eval_accuracy_v1_engine_fp8_kv_cache`。 它通过参数化组合 `model`。 关键输入包括 `model`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `lm_eval`, `pytest`
- **Project / 项目内**: `vllm.platforms.current_platform`
