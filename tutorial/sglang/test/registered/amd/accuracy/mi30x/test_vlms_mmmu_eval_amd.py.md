# test_vlms_mmmu_eval_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi30x/test_vlms_mmmu_eval_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on mi30x vlms mmmu eval amd in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 mi30x vlms mmmu eval amd 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Document the module
```python
"""
AMD VLM MMMU Evaluation Test - MI30x Only

This test evaluates Vision-Language Models (VLMs) on the MMMU benchmark on AMD GPUs.
Models are selected based on compatibility with AMD/ROCm platform.

VLMs tested here:
- Qwen VL series (Qwen2-VL-7B, Qwen2.5-VL-7B, Qwen3-VL-30B)
- InternVL2 series (InternVL2_5-2B)
- MiniCPM series (MiniCPM-v-2_6, MiniCPM-o-2_6)
- DeepSeek VL series (deepseek-vl2-small, Janus-Pro-7B, DeepSeek-OCR-2)
- Kimi VL (Kimi-VL-A3B-Instruct)
- MiMo VL (MiMo-VL-7B-RL)
- GLM VL (GLM-4.1V-9B-Thinking)

Note: NVILA models are excluded (NVIDIA-specific).
Note: This test runs only on MI30x runners (linux-mi325-gpu-2), not on MI35x.

Registry: nightly-amd-accuracy-2-gpu-vlm suite (2-GPU VLM tests)
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 22-38: Import dependencies
```python
import os
import time
import unittest
import warnings
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
    write_results_to_json,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 41-41: Register CI metadata
```python
register_amd_ci(est_time=7200, suite="nightly-amd-accuracy-2-gpu-vlm", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 45-104: Define module constants (part 1)
```python
AMD_VLM_MODELS = [
    # Qwen VL series - well supported on AMD
    {
        "model_path": "Qwen/Qwen2-VL-7B-Instruct",
        "tp_size": 1,
        "accuracy_threshold": 0.30,
        "extra_args": ["--trust-remote-code"],
    },
    {
        "model_path": "Qwen/Qwen2.5-VL-7B-Instruct",
        "tp_size": 1,
        "accuracy_threshold": 0.33,
        "extra_args": ["--trust-remote-code"],
    },
    {
        "model_path": "Qwen/Qwen3-VL-30B-A3B-Instruct",
        "tp_size": 2,
        "accuracy_threshold": 0.29,
        "extra_args": ["--trust-remote-code"],
    },
    # InternVL2 - smaller model, good for testing
    {
        "model_path": "OpenGVLab/InternVL2_5-2B",
        "tp_size": 1,
        "accuracy_threshold": 0.29,
        "extra_args": ["--trust-remote-code"],
    },
    # MiniCPM series
    {
        "model_path": "openbmb/MiniCPM-v-2_6",
        "tp_size": 1,
        "accuracy_threshold": 0.25,
        "extra_args": ["--trust-remote-code"],
    },
    {
        "model_path": "openbmb/MiniCPM-o-2_6",
        "tp_size": 1,
        "accuracy_threshold": 0.32,
        "extra_args": ["--trust-remote-code"],
    },
    # DeepSeek VL series
    {
        "model_path": "deepseek-ai/deepseek-vl2-small",
        "tp_size": 1,
        "accuracy_threshold": 0.31,
        "extra_args": ["--trust-remote-code"],
    },
    {
        "model_path": "deepseek-ai/Janus-Pro-7B",
        "tp_size": 1,
        "accuracy_threshold": 0.28,
        "extra_args": ["--trust-remote-code"],
    },
    # Kimi VL - MoE
    {
        "model_path": "moonshotai/Kimi-VL-A3B-Instruct",
        "tp_size": 1,
        "accuracy_threshold": 0.26,
        "extra_args": ["--trust-remote-code"],
    },
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 105-133: Define module constants (part 2)
```python
    # MiMo VL
    {
        "model_path": "XiaomiMiMo/MiMo-VL-7B-RL",
        "tp_size": 1,
        "accuracy_threshold": 0.27,
        "extra_args": ["--trust-remote-code"],
    },
    # GLM VL
    {
        "model_path": "zai-org/GLM-4.1V-9B-Thinking",
        "tp_size": 1,
        "accuracy_threshold": 0.27,
        "extra_args": ["--trust-remote-code"],
    },
    # DeepSeek-OCR-2 - last to avoid memory pressure on subsequent models
    {
        "model_path": "deepseek-ai/DeepSeek-OCR-2",
        "tp_size": 1,
        "accuracy_threshold": 0.25,
        "extra_args": [
            "--trust-remote-code",
            "--disable-cuda-graph",
            "--mem-fraction-static",
            "0.70",
            "--max-total-tokens",
            "16384",
        ],
    },
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 136-141: Define module constants
```python
TRITON_ATTENTION_MODELS = {
    # "deepseek-ai/deepseek-vl2-small",
    # "Qwen/Qwen3-VL-30B-A3B-Instruct",
    # "moonshotai/Kimi-VL-A3B-Instruct",
    "deepseek-ai/DeepSeek-OCR-2",
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 144-147: Define module constants
```python
AMD_FAILING_VLM_MODELS = {
    # GLM-4.1V processor not registered yet (Glm4vForConditionalGeneration)
    "zai-org/GLM-4.1V-9B-Thinking",
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 150-152: Define helper: get active models
```python
def get_active_models():
    """Get list of models to test, excluding known failures."""
    return [m for m in AMD_VLM_MODELS if m["model_path"] not in AMD_FAILING_VLM_MODELS]
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 155-155: Define class TestNightlyVLMMmmuEvalAMD
```python
class TestNightlyVLMMmmuEvalAMD(unittest.TestCase):
```
**EN:** This declaration introduces the `TestNightlyVLMMmmuEvalAMD` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNightlyVLMMmmuEvalAMD` 测试类，并说明它通过继承承担的职责。

### Lines 156-159: Document the class `TestNightlyVLMMmmuEvalAMD`
```python
    """AMD VLM MMMU Evaluation Test.

    Tests Vision-Language Models on MMMU benchmark using AMD GPUs.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNightlyVLMMmmuEvalAMD`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNightlyVLMMmmuEvalAMD`的设计意图。

### Lines 161-164: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.models = get_active_models()
        cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 166-223: Run test: mmmu vlm models (part 1)
```python
    def test_mmmu_vlm_models(self):
        """Test all configured VLM models on MMMU benchmark."""
        warnings.filterwarnings(
            "ignore", category=ResourceWarning, message="unclosed.*socket"
        )
        is_first = True
        all_results = []
        total_test_start = time.time()

        print(f"\n{'='*60}")
        print("AMD VLM MMMU Evaluation Test")
        print(f"{'='*60}")
        print(f"Benchmark: MMMU (100 samples)")
        print(f"Models to test: {len(self.models)}")
        for m in self.models:
            print(f"  - {m['model_path']} (TP={m['tp_size']})")
        print(f"{'='*60}\n")

        for model_config in self.models:
            model_path = model_config["model_path"]
            tp_size = model_config["tp_size"]
            accuracy_threshold = model_config["accuracy_threshold"]
            extra_args = model_config.get("extra_args", [])
            error_message = None

            with self.subTest(model=model_path):
                print(f"\n{'='*60}")
                print(f"Testing: {model_path} (TP={tp_size})")
                print(f"{'='*60}")

                model_start = time.time()
                startup_time = None
                eval_time = None
                score = None

                # Set AMD-specific environment variables
                if model_path in TRITON_ATTENTION_MODELS:
                    os.environ["SGLANG_USE_AITER"] = "0"
                else:
                    os.environ["SGLANG_USE_AITER"] = "1"

                # Build launch args
                other_args = list(extra_args)
                other_args.extend(["--log-level-http", "warning"])
                if tp_size > 1:
                    other_args.extend(["--tp", str(tp_size)])

                # Launch server with timing
                print(f"🚀 Launching server...")
                server_start = time.time()
                process = popen_launch_server(
                    model=model_path,
                    base_url=self.base_url,
                    other_args=other_args,
                    timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                )
                startup_time = time.time() - server_start
                print(f"⏱️  Server startup: {startup_time:.1f}s")
```
**EN:** This test method exercises mmmu vlm models and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario.
**CN:** 该测试方法会执行 mmmu vlm models 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务。

### Lines 225-272: Run test: mmmu vlm models (part 2)
```python
                try:
                    args = SimpleNamespace(
                        base_url=self.base_url,
                        model=model_path,
                        eval_name="mmmu",
                        num_examples=100,
                        num_threads=64,
                        max_tokens=30,
                    )

                    # Run evaluation with timing
                    print(f"📊 Running MMMU evaluation (100 samples)...")
                    eval_start = time.time()

                    # Retry up to 3 times
                    metrics = None
                    for attempt in range(3):
                        try:
                            metrics = run_eval(args)
                            score = metrics["score"]
                            if score >= accuracy_threshold:
                                break
                        except Exception as e:
                            print(f"   Attempt {attempt + 1} failed with error: {e}")
                            if attempt == 2:
                                raise

                    eval_time = time.time() - eval_start
                    total_time = time.time() - model_start

                    # Print results
                    print(f"\n📈 Results for {model_path}:")
                    print(
                        f"   Score: {score:.3f} (threshold: {accuracy_threshold:.2f})"
                    )
                    print(f"\n⏱️  Runtime breakdown:")
                    print(f"   Server startup: {startup_time:.1f}s")
                    print(f"   Evaluation: {eval_time:.1f}s")
                    print(f"   Total: {total_time:.1f}s")

                    passed = score >= accuracy_threshold
                    if passed:
                        print(f"\n   Status: ✅ PASSED")
                    else:
                        print(f"\n   Status: ❌ FAILED")

                    write_results_to_json(model_path, metrics, "w" if is_first else "a")
                    is_first = False
```
**EN:** This test method exercises mmmu vlm models and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and packages launch or evaluation arguments into a lightweight namespace.
**CN:** 该测试方法会执行 mmmu vlm models 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并把启动或评测参数封装到轻量级命名空间中。

### Lines 274-311: Run test: mmmu vlm models (part 3)
```python
                    all_results.append(
                        {
                            "model": model_path,
                            "tp_size": tp_size,
                            "score": score,
                            "threshold": accuracy_threshold,
                            "startup_time": startup_time,
                            "eval_time": eval_time,
                            "total_time": total_time,
                            "passed": passed,
                            "error": None,
                        }
                    )

                except Exception as e:
                    error_message = str(e)
                    total_time = time.time() - model_start
                    print(f"\n❌ Error evaluating {model_path}: {error_message}")
                    all_results.append(
                        {
                            "model": model_path,
                            "tp_size": tp_size,
                            "score": None,
                            "threshold": accuracy_threshold,
                            "startup_time": startup_time,
                            "eval_time": None,
                            "total_time": total_time,
                            "passed": False,
                            "error": error_message,
                        }
                    )

                finally:
                    print(f"\n🛑 Stopping server...")
                    kill_process_tree(process.pid)

        # Calculate total test runtime
        total_test_time = time.time() - total_test_start
```
**EN:** This test method exercises mmmu vlm models and verifies that the observed behavior matches the expected contract. It also releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 mmmu vlm models 场景，并验证观测到的行为是否符合预期契约。 其中还会在检查完成后释放已启动的进程。

### Lines 313-314: Run test: mmmu vlm models (part 4)
```python
        # Generate summary
        self._check_results(all_results, total_test_time)
```
**EN:** This test method exercises mmmu vlm models and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 mmmu vlm models 场景，并验证观测到的行为是否符合预期契约。

### Lines 316-370: Define helper: check results (part 1)
```python
    def _check_results(self, results, total_test_time):
        """Check results and generate summary."""
        failed_models = []
        passed_count = 0
        failed_count = 0

        summary = (
            "| Model | TP | Score | Threshold | Startup | Eval | Total | Status |\n"
        )
        summary += (
            "| ----- | -- | ----- | --------- | ------- | ---- | ----- | ------ |\n"
        )

        for result in results:
            model = result["model"]
            score = result["score"]
            tp_size = result["tp_size"]
            threshold = result["threshold"]
            startup_time = result.get("startup_time")
            eval_time = result.get("eval_time")
            total_time = result.get("total_time")
            error = result.get("error")

            if error:
                status = "❌ ERROR"
                failed_count += 1
                failed_models.append(f"- {model}: ERROR - {error[:100]}")
            elif result["passed"]:
                status = "✅ PASS"
                passed_count += 1
            else:
                status = "❌ FAIL"
                failed_count += 1
                failed_models.append(
                    f"- {model}: score={score:.4f}, threshold={threshold:.4f}"
                )

            # Format values
            score_str = f"{score:.3f}" if score is not None else "N/A"
            startup_str = f"{startup_time:.0f}s" if startup_time is not None else "N/A"
            eval_str = f"{eval_time:.0f}s" if eval_time is not None else "N/A"
            total_str = f"{total_time:.0f}s" if total_time is not None else "N/A"

            summary += f"| {model} | {tp_size} | {score_str} | {threshold:.2f} | {startup_str} | {eval_str} | {total_str} | {status} |\n"

        print(f"\n{'='*60}")
        print("SUMMARY - AMD VLM MMMU Evaluation")
        print(f"{'='*60}")
        print(summary)
        print(f"\n📊 Final Statistics:")
        print(f"   Passed: {passed_count}")
        print(f"   Failed: {failed_count}")
        print(
            f"\n⏱️  Total test runtime: {total_test_time:.1f}s ({total_test_time/60:.1f} min)"
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestNightlyVLMMmmuEvalAMD` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNightlyVLMMmmuEvalAMD` 内部调用，从而让场景结构更清晰。

### Lines 372-376: Define helper: check results (part 2)
```python
        if is_in_ci():
            write_github_step_summary(
                f"### TestNightlyVLMMmmuEvalAMD\n{summary}\n\n"
                f"**Total Runtime:** {total_test_time:.1f}s ({total_test_time/60:.1f} min)"
            )
```
**EN:** This helper function encapsulates reusable logic inside `TestNightlyVLMMmmuEvalAMD` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNightlyVLMMmmuEvalAMD` 内部调用，从而让场景结构更清晰。

### Lines 378-380: Define helper: check results (part 3)
```python
        if failed_models:
            failure_msg = "\n".join(failed_models)
            raise AssertionError(f"The following models failed:\n{failure_msg}")
```
**EN:** This helper function encapsulates reusable logic inside `TestNightlyVLMMmmuEvalAMD` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNightlyVLMMmmuEvalAMD` 内部调用，从而让场景结构更清晰。

### Lines 383-384: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `os`, `time`, `types`, `unittest`, `warnings`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
