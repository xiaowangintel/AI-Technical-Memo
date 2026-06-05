# test_gsm8k_eval_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi30x/test_gsm8k_eval_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on mi30x gsm8k eval amd in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 mi30x gsm8k eval amd 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Document the module
```python
"""
AMD GSM8K Evaluation Test (Migrated from test/srt/nightly/)

This test evaluates instruction-tuned models on the gsm8k benchmark using chat completions.
Models are tested with various TP configurations on AMD GPUs.

Registry: nightly-amd suite (2-GPU tests)
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 10-32: Import dependencies
```python
import json
import os
import time
import unittest
import warnings
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP1,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP2,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    is_in_ci,
    parse_models,
    popen_launch_server,
    write_github_step_summary,
    write_results_to_json,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 35-35: Register CI metadata
```python
register_amd_ci(est_time=3600, suite="nightly-amd", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 37-74: Define module constants
```python
MODEL_SCORE_THRESHOLDS = {
    # Thresholds set at 5% below reported GSM8K (5-shot/CoT) scores
    # Llama 3.1 series
    "meta-llama/Llama-3.1-8B-Instruct": 0.80,  # 84.5% - 5%
    "meta-llama/Llama-3.1-70B-Instruct": 0.89,  # 94.1% - 5%
    # Llama 3.2 series (smaller models)
    "meta-llama/Llama-3.2-3B-Instruct": 0.43,  # 48.2% - 5%
    # Mistral series
    "mistralai/Mistral-7B-Instruct-v0.3": 0.47,  # 52.1% - 5%
    "mistralai/Mixtral-8x7B-Instruct-v0.1": 0.69,  # 74.4% - 5% (lower if AMD scores differently)
    # DeepSeek series
    "deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct": 0.81,  # 86.4% - 5%
    # Qwen2 series
    "Qwen/Qwen2-57B-A14B-Instruct": 0.76,  # 80.7% - 5% (official A14B score; 88.2% was the 72B)
    "Qwen/Qwen2.5-7B-Instruct": 0.82,  # 86.3% - 5%
    # Qwen3 series
    "Qwen/Qwen3-30B-A3B-Thinking-2507": 0.86,  # 91.4% - 5% (full attention mode; ensure sufficient max_tokens)
    "Qwen/Qwen3-8B": 0.76,  # ~81%  - 5%
    # Google Gemma
    "google/gemma-2-27b-it": 0.86,  # 90.7% - 5%
    "google/gemma-2-9b-it": 0.74,  # 78.5% - 5%
    # "neuralmagic/gemma-2-2b-it-FP8": 0.4,  # Small 2B model - OOM on single GPU
    # FP8 quantized models
    "neuralmagic/Meta-Llama-3.1-8B-Instruct-FP8": 0.80,  # 84.5% - 5%
    "neuralmagic/Mistral-7B-Instruct-v0.3-FP8": 0.46,  # ~51%  - 5%
    "neuralmagic/Meta-Llama-3.1-70B-Instruct-FP8": 0.89,  # 94.1% - 5%
    "neuralmagic/Qwen2-72B-Instruct-FP8": 0.86,  # 91.1% - 5%
    "neuralmagic/Qwen2-57B-A14B-Instruct-FP8": 0.76,  # 80.7% - 5% (official A14B score)
    "neuralmagic/Mixtral-8x7B-Instruct-v0.1-FP8": 0.69,  # 74.4% - 5%
    "neuralmagic/DeepSeek-Coder-V2-Lite-Instruct-FP8": 0.81,  # 86.4% - 5%
}

failing_models = {
    "neuralmagic/DeepSeek-Coder-V2-Lite-Instruct-FP8",  # RuntimeError: This GEMM is not supported!
    "zai-org/GLM-4.5-Air-FP8",  # TypeError: cannot unpack non-iterable ForwardMetadata object
    "google/gemma-2-9b-it",  # OOM on single GPU (exit code -9)
    "neuralmagic/gemma-2-2b-it-FP8",  # OOM on single GPU (exit code -9)
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 77-80: Define helper: remove failing models
```python
def remove_failing_models(model_str):
    models = model_str.split(",")
    filtered = [m for m in models if m not in failing_models]
    return ",".join(filtered)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 83-94: Define module constants
```python
DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1 = remove_failing_models(
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1
)
DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2 = remove_failing_models(
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2
)
DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP1 = remove_failing_models(
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP1
)
DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP2 = remove_failing_models(
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP2
)
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 98-116: Define module constants
```python
AMD_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1 = remove_failing_models(
    "meta-llama/Llama-3.2-3B-Instruct,Qwen/Qwen2.5-7B-Instruct,Qwen/Qwen3-8B,google/gemma-2-9b-it"
)
# TP2 models - larger models requiring 2 GPUs
AMD_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2 = remove_failing_models(
    "Qwen/Qwen3-30B-A3B-Thinking-2507"
)

NO_MOE_PADDING_MODELS = {"neuralmagic/Mixtral-8x7B-Instruct-v0.1-FP8"}
DISABLE_HF_XET_MODELS = {
    "Qwen/Qwen2-57B-A14B-Instruct",
    "neuralmagic/Qwen2-57B-A14B-Instruct-FP8",
}
TRITON_MOE_MODELS = {
    # "neuralmagic/Mixtral-8x7B-Instruct-v0.1-FP8",
    "neuralmagic/DeepSeek-Coder-V2-Lite-Instruct-FP8",
    # "mistralai/Mixtral-8x7B-Instruct-v0.1",
    # "mistralai/Mistral-7B-Instruct-v0.3",
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 123-145: Define helper: popen launch server wrapper
```python
def popen_launch_server_wrapper(base_url, model, is_tp2):
    other_args = ["--log-level-http", "warning", "--trust-remote-code"]
    if is_tp2:
        other_args.extend(["--tp", "2"])

    # Use same config as sanity_check.py for AMD-specific models (scaled for tp=2)
    # Original tp=8: chunked-prefill-size=130172, max-running-requests=128
    # Scaled tp=2:   chunked-prefill-size=32543,  max-running-requests=32
    # if model in AMD_SPECIAL_CONFIG_MODELS:
    #     other_args.extend([
    #         "--chunked-prefill-size", "32543",
    #         "--max-running-requests", "32",
    #         "--mem-fraction-static", "0.85",
    #         "--attention-backend", "aiter",
    #     ])

    process = popen_launch_server(
        model,
        base_url,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        other_args=other_args,
    )
    return process
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also launches a model server for the scenario.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会启动场景所需的模型服务。

### Lines 148-201: Define helper: check model scores
```python
def check_model_scores(results):
    """Check model scores and generate summary table with pass/fail status."""
    failed_models = []
    passed_count = 0
    failed_count = 0

    summary = "| Model | TP | Score | Threshold | Startup | Eval | Total | Status |\n"
    summary += "| ----- | -- | ----- | --------- | ------- | ---- | ----- | ------ |\n"

    for result in results:
        model = result["model"]
        score = result["score"]
        tp_size = result.get("tp_size", 2)
        startup_time = result.get("startup_time")
        eval_time = result.get("eval_time")
        total_time = result.get("total_time")

        threshold = MODEL_SCORE_THRESHOLDS.get(model)
        if threshold is None:
            print(f"Warning: No threshold defined for model {model}")
            status = "⚠️ NO THRESHOLD"
        elif score >= threshold:
            status = "✅ PASS"
            passed_count += 1
        else:
            status = "❌ FAIL"
            failed_count += 1
            failed_models.append(
                f"- {model}: score={score:.4f}, threshold={threshold:.4f}"
            )

        # Format times
        startup_str = f"{startup_time:.0f}s" if startup_time is not None else "N/A"
        eval_str = f"{eval_time:.0f}s" if eval_time is not None else "N/A"
        total_str = f"{total_time:.0f}s" if total_time is not None else "N/A"
        threshold_str = f"{threshold:.2f}" if threshold is not None else "N/A"

        line = f"| {model} | {tp_size} | {score:.3f} | {threshold_str} | {startup_str} | {eval_str} | {total_str} | {status} |\n"
        summary += line

    print(f"\n{'='*60}")
    print("SUMMARY - TP=2 Instruction Models (gsm8k)")
    print(f"{'='*60}")
    print(summary)
    print(f"\n📊 Final Statistics:")
    print(f"   Passed: {passed_count}")
    print(f"   Failed: {failed_count}")

    if is_in_ci():
        write_github_step_summary(f"### TestNightlyGsm8KEval (TP=2)\n{summary}")

    if failed_models:
        failure_msg = "\n".join(failed_models)
        raise AssertionError(f"The following models failed:\n{failure_msg}")
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 205-205: Define class TestNightlyGsm8KEval
```python
class TestNightlyGsm8KEval(unittest.TestCase):
```
**EN:** This declaration introduces the `TestNightlyGsm8KEval` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNightlyGsm8KEval` 测试类，并说明它通过继承承担的职责。

### Lines 206-217: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model_groups = [
            (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1), False, False),
            (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2), False, True),
            (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP1), True, False),
            (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP2), True, True),
            # AMD-specific models verified on MI300X
            (parse_models(AMD_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1), False, False),
            (parse_models(AMD_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2), False, True),
        ]
        cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 219-268: Run test: gsm8k all models (part 1)
```python
    def test_gsm8k_all_models(self):
        warnings.filterwarnings(
            "ignore", category=ResourceWarning, message="unclosed.*socket"
        )
        is_first = True
        all_results = []
        total_test_start = time.time()

        print(f"\n{'='*60}")
        print("AMD GSM8K Evaluation Test (TP=2 Instruction Models)")
        print(f"{'='*60}")
        print(f"Benchmark: gsm8k (chat completions)")
        print(f"{'='*60}\n")

        for model_group, is_fp8, is_tp2 in self.model_groups:
            for model in model_group:
                with self.subTest(model=model):
                    tp_size = 2 if is_tp2 else 1
                    print(f"\n{'='*60}")
                    print(f"Testing: {model} (TP={tp_size}, FP8={is_fp8})")
                    print(f"{'='*60}")

                    model_start = time.time()
                    startup_time = None
                    eval_time = None

                    os.environ["SGLANG_MOE_PADDING"] = (
                        "0" if model in NO_MOE_PADDING_MODELS else "1"
                    )
                    os.environ["HF_HUB_DISABLE_XET"] = (
                        "1" if model in DISABLE_HF_XET_MODELS else "0"
                    )
                    os.environ["SGLANG_USE_AITER"] = (
                        "0" if model in TRITON_MOE_MODELS else "1"
                    )

                    # Launch server with timing
                    print(f"🚀 Launching server...")
                    server_start = time.time()
                    process = popen_launch_server_wrapper(self.base_url, model, is_tp2)
                    startup_time = time.time() - server_start
                    print(f"⏱️  Server startup: {startup_time:.1f}s")

                    args = SimpleNamespace(
                        base_url=self.base_url,
                        model=model,
                        eval_name="gsm8k",
                        num_examples=None,
                        num_threads=1024,
                    )
```
**EN:** This test method exercises gsm8k all models and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and packages launch or evaluation arguments into a lightweight namespace.
**CN:** 该测试方法会执行 gsm8k all models 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并把启动或评测参数封装到轻量级命名空间中。

### Lines 270-322: Run test: gsm8k all models (part 2)
```python
                    # Run eval with timing and retries
                    print(f"📊 Running gsm8k evaluation...")
                    eval_start = time.time()
                    threshold = MODEL_SCORE_THRESHOLDS.get(model)
                    metrics = None
                    for attempt in range(3):
                        try:
                            metrics = run_eval(args)
                            score = metrics["score"]
                            if threshold and score >= threshold:
                                break
                        except Exception as e:
                            print(f"   Attempt {attempt + 1} failed with error: {e}")
                    eval_time = time.time() - eval_start
                    total_time = time.time() - model_start

                    # Print results
                    score = metrics["score"] if metrics else 0.0
                    threshold_str = f"{threshold:.2f}" if threshold else "N/A"
                    passed = threshold and score >= threshold

                    print(f"\n📈 Results for {model}:")
                    print(f"   Score: {score:.3f} (threshold: {threshold_str})")
                    print(f"\n⏱️  Runtime breakdown:")
                    print(f"   Server startup: {startup_time:.1f}s")
                    print(f"   Evaluation: {eval_time:.1f}s")
                    print(f"   Total: {total_time:.1f}s")

                    if passed:
                        print(f"\n   Status: ✅ PASSED")
                    else:
                        print(f"\n   Status: ❌ FAILED")

                    write_results_to_json(model, metrics, "w" if is_first else "a")
                    is_first = False

                    all_results.append(
                        {
                            "model": model,
                            "score": score,
                            "tp_size": tp_size,
                            "is_fp8": is_fp8,
                            "startup_time": startup_time,
                            "eval_time": eval_time,
                            "total_time": total_time,
                        }
                    )

                    print(f"\n🛑 Stopping server...")
                    kill_process_tree(process.pid)

        # Calculate total test runtime
        total_test_time = time.time() - total_test_start
```
**EN:** This test method exercises gsm8k all models and verifies that the observed behavior matches the expected contract. It also releases spawned processes after the checks finish and runs an evaluation workflow to measure model quality.
**CN:** 该测试方法会执行 gsm8k all models 场景，并验证观测到的行为是否符合预期契约。 其中还会在检查完成后释放已启动的进程，并运行评测流程以衡量模型质量。

### Lines 324-329: Run test: gsm8k all models (part 3)
```python
        try:
            with open("results.json", "r") as f:
                print("\nFinal Results from results.json:")
                print(json.dumps(json.load(f), indent=2))
        except Exception as e:
            print(f"Error reading results.json: {e}")
```
**EN:** This test method exercises gsm8k all models and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 gsm8k all models 场景，并验证观测到的行为是否符合预期契约。

### Lines 331-335: Run test: gsm8k all models (part 4)
```python
        # Check all scores after collecting all results
        check_model_scores(all_results)
        print(
            f"\n⏱️  Total test runtime: {total_test_time:.1f}s ({total_test_time/60:.1f} min)"
        )
```
**EN:** This test method exercises gsm8k all models and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 gsm8k all models 场景，并验证观测到的行为是否符合预期契约。

### Lines 338-339: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `json`, `os`, `time`, `types`, `unittest`, `warnings`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
