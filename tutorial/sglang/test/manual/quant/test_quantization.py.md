# test_quantization.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/quant/test_quantization.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `quantization` scenario in `test/manual/quant`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/quant` 中的 `quantization` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Constants and scenario settings / 常量与场景配置
```python
import json
import unittest
import warnings
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_QUANT_TP1,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
    write_results_to_json,
)

MODEL_SCORE_THRESHOLDS = {
    # Baselines observed with gsm8k 5-shot concatenated format via chat API,
    # which scores lower than reported benchmarks using proper CoT format.
    # Thresholds set 5% below observed to catch catastrophic regressions.
    "hugging-quants/Meta-Llama-3.1-8B-Instruct-AWQ-INT4": 0.74,  # observed: 0.781
    "hugging-quants/Meta-Llama-3.1-8B-Instruct-GPTQ-INT4": 0.74,  # observed: 0.785
    "hugging-quants/Mixtral-8x7B-Instruct-v0.1-AWQ-INT4": 0.36,  # observed: 0.380
}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 28-29: Helper routines around parse_models / 辅助例程
```python
def parse_models(model_string):
    return [model.strip() for model in model_string.split(",") if model.strip()]
```
**EN:** This range implements helper routine(s) `parse_models` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `strip` and `split`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 32-54: Helper routines around popen_launch_server_wrapper / 辅助例程
```python
def popen_launch_server_wrapper(base_url, model, is_fp8, is_tp2):
    other_args = ["--log-level-http", "warning", "--trust-remote-code"]
    if is_fp8:
        if "Llama-3" in model or "gemma-2" in model:
            other_args.extend(["--kv-cache-dtype", "fp8_e5m2"])
        elif "Qwen2-72B-Instruct-FP8" in model:
            other_args.extend(["--quantization", "fp8"])
        elif "neuralmagic/Mixtral-8x7B-Instruct-v0.1-FP8" in model:
            other_args.extend([])
        else:
            other_args.extend(["--quantization", "fp8", "--kv-cache-dtype", "fp8_e5m2"])
    if is_tp2:
        other_args.extend(["--tp", "2"])
    if "DeepSeek" in model:
        other_args.extend(["--mem-frac", "0.85"])

    process = popen_launch_server(
        model,
        base_url,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        other_args=other_args,
    )
    return process
```
**EN:** This range implements helper routine(s) `popen_launch_server_wrapper` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `extend` and `popen_launch_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 57-81: Helper routines around check_model_scores / 辅助例程
```python
def check_model_scores(results):
    failed_models = []
    summary = " | model | score | threshold |\n"
    summary += "| ----- | ----- | --------- |\n"

    for model, score in results:
        threshold = MODEL_SCORE_THRESHOLDS.get(model)
        if threshold is None:
            print(f"Warning: No threshold defined for model {model}")
            continue

        if score < threshold:
            failed_models.append(
                f"\nScore Check Failed: {model}\n"
                f"Model {model} score ({score:.4f}) is below threshold ({threshold:.4f})"
            )

        line = f"| {model} | {score} | {threshold} |\n"
        summary += line

    print(summary)

    if is_in_ci():
        write_github_step_summary(
            f"### TestNightlyGsm8KEval for awq, gptq, gguf\n{summary}"
```
**EN:** This range implements helper routine(s) `check_model_scores` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`, `append`, `score` and `threshold`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-85: Assertions and result checks / 断言与结果检查
```python
        )

    if failed_models:
        raise AssertionError("\n".join(failed_models))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `AssertionError` and `join`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 88-89: Class definition for TestNightlyGsm8KEval / 类定义
```python
class TestNightlyGsm8KEval(unittest.TestCase):
    @classmethod
```
**EN:** This range declares `TestNightlyGsm8KEval`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 90-94: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model_groups = [
            (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_QUANT_TP1), False, False),
        ]
        cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `parse_models`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 96-112: Test routines around test_gsm8k_all_models / 测试例程
```python
    def test_gsm8k_all_models(self):
        warnings.filterwarnings(
            "ignore", category=ResourceWarning, message="unclosed.*socket"
        )
        is_first = True
        all_results = []

        for model_group, is_fp8, is_tp2 in self.model_groups:
            for model in model_group:
                with self.subTest(model=model):
                    process = popen_launch_server_wrapper(
                        self.base_url, model, is_fp8, is_tp2
                    )

                    args = SimpleNamespace(
                        base_url=self.base_url,
                        model=model,
```
**EN:** This range defines concrete test routine(s) `test_gsm8k_all_models`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `filterwarnings`, `subTest`, `popen_launch_server_wrapper` and `SimpleNamespace`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 113-120: Scenario logic / 场景逻辑
```python
                        eval_name="gsm8k",
                        num_examples=None,
                        num_threads=1024,
                    )

                    metrics = run_eval(args)
                    print(
                        f"{'=' * 42}\n{model} - metrics={metrics} score={metrics['score']}\n{'=' * 42}\n"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `run_eval`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 121-137: Process control logic / 进程控制逻辑
```python
                    )

                    write_results_to_json(model, metrics, "w" if is_first else "a")
                    is_first = False

                    all_results.append((model, metrics["score"]))
                    kill_process_tree(process.pid)

        try:
            with open("results.json", "r") as f:
                print("\nFinal Results from results.json:")
                print(json.dumps(json.load(f), indent=2))
        except Exception as e:
            print(f"Error reading results.json: {e}")

        # Check all scores after collecting all results
        check_model_scores(all_results)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `write_results_to_json`, `append`, `kill_process_tree` and `dumps`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 138-141: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `types`, `unittest`, `warnings`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
