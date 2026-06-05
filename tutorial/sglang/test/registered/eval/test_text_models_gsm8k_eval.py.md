# test_text_models_gsm8k_eval.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/eval/test_text_models_gsm8k_eval.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates text models gsm8k eval behavior in SGLang's eval area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 eval 领域中与 text models gsm8k eval 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest
import warnings
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP1,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP2,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2,
    DEFAULT_URL_FOR_TEST,
    ModelLaunchSettings,
    check_evaluation_test_results,
    parse_models,
    popen_launch_server,
    write_results_to_json,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `warnings`, `types`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `warnings`, `types`。

### Lines 21-23: supporting source context / 辅助源码上下文
```python

# Nightly eval tests run large models (up to 70B+ params) that may need
# downloading on cache miss. Use a longer timeout than the default 600s.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 24-48: CI registration and metadata / CI 注册与元数据
```python
NIGHTLY_EVAL_SERVER_TIMEOUT = 1800

register_cuda_ci(est_time=3600, suite="nightly-eval-text-2-gpu", nightly=True)

MODEL_SCORE_THRESHOLDS = {
    # Thresholds set at 5% below reported GSM8K (5-shot/CoT) scores
    "meta-llama/Llama-3.1-8B-Instruct": 0.80,  # 84.5% - 5%
    "mistralai/Mistral-7B-Instruct-v0.3": 0.47,  # 52.1% - 5%
    "deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct": 0.81,  # 86.4% - 5%
    "google/gemma-2-27b-it": 0.86,  # 90.7% - 5%
    "meta-llama/Llama-3.1-70B-Instruct": 0.89,  # 94.1% - 5%
    "mistralai/Mixtral-8x7B-Instruct-v0.1": 0.69,  # 74.4% - 5%
    "Qwen/Qwen2-57B-A14B-Instruct": 0.76,  # 80.7% - 5% (official A14B score; 88.2% was the 72B)
    "neuralmagic/Meta-Llama-3.1-8B-Instruct-FP8": 0.80,  # 84.5% - 5%
    "neuralmagic/Mistral-7B-Instruct-v0.3-FP8": 0.47,  # 52.1% - 5%
    "neuralmagic/DeepSeek-Coder-V2-Lite-Instruct-FP8": 0.81,  # 86.4% - 5%
    "zai-org/GLM-4.5-Air-FP8": 0.80,  # ~85%  - 5%
    # GSM8K baseline for gemma-2-2b is ~40-45%; threshold set at 5% below.
    # (Previously 0.50 based on MGSM-EN; tracked regression: https://github.com/sgl-project/sglang/issues/4324)
    "neuralmagic/gemma-2-2b-it-FP8": 0.38,  # ~43%  - 5%
    "neuralmagic/Meta-Llama-3.1-70B-Instruct-FP8": 0.89,  # 94.1% - 5%
    "neuralmagic/Mixtral-8x7B-Instruct-v0.1-FP8": 0.69,  # 74.4% - 5%
    "neuralmagic/Qwen2-72B-Instruct-FP8": 0.86,  # 91.1% - 5%
    "neuralmagic/Qwen2-57B-A14B-Instruct-FP8": 0.76,  # 80.7% - 5% (official A14B score)
}
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 49-51: supporting source context / 辅助源码上下文
```python


# Do not use `CustomTestCase` since `test_gsm8k_all_models` does not want retry
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 52-52: class TestNightlyGsm8KEval declaration / 类 TestNightlyGsm8KEval 声明
```python
class TestNightlyGsm8KEval(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 53-68: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.models = []
        models_tp1 = parse_models(
            DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1
        ) + parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP1)
        for model_path in models_tp1:
            cls.models.append(ModelLaunchSettings(model_path, tp_size=1))

        models_tp2 = parse_models(
            DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2
        ) + parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP2)
        for model_path in models_tp2:
            cls.models.append(ModelLaunchSettings(model_path, tp_size=2))

        cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 70-136: test case gsm8k all models / 测试用例 gsm8k all models
```python
    def test_gsm8k_all_models(self):
        warnings.filterwarnings(
            "ignore", category=ResourceWarning, message="unclosed.*socket"
        )
        is_first = True
        all_results = []
        for model_setup in self.models:
            with self.subTest(model=model_setup.model_path):
                other_args = list(model_setup.extra_args)
                process = None

                if model_setup.model_path == "meta-llama/Llama-3.1-70B-Instruct":
                    other_args.extend(["--mem-fraction-static", "0.9"])

                try:
                    process = popen_launch_server(
                        model=model_setup.model_path,
                        other_args=other_args,
                        base_url=self.base_url,
                        timeout=NIGHTLY_EVAL_SERVER_TIMEOUT,
                    )

                    args = SimpleNamespace(
                        base_url=self.base_url,
                        model=model_setup.model_path,
                        eval_name="gsm8k",
                        num_examples=None,
                        num_threads=1024,
                    )

                    metrics = run_eval(args)
                    print(
                        f"{'=' * 42}\n{model_setup.model_path} - metrics={metrics} score={metrics['score']}\n{'=' * 42}\n"
                    )

                    write_results_to_json(
                        model_setup.model_path, metrics, "w" if is_first else "a"
                    )
                    is_first = False

                    all_results.append(
                        (model_setup.model_path, metrics["score"], 0.0, None)
                    )
                except Exception as e:
                    error_message = str(e)
                    all_results.append(
                        (model_setup.model_path, None, None, error_message)
                    )
                    print(f"Error evaluating {model_setup.model_path}: {error_message}")
                finally:
                    if process is not None:
                        kill_process_tree(process.pid)

        try:
            with open("results.json", "r") as f:
                print("\nFinal Results from results.json:")
                print(json.dumps(json.load(f), indent=2))
        except Exception as e:
            print(f"Error reading results.json: {e}")

        # Check all scores after collecting all results
        check_evaluation_test_results(
            all_results,
            self.__class__.__name__,
            model_accuracy_thresholds=MODEL_SCORE_THRESHOLDS,
            model_count=len(self.models),
        )
```
**EN:** This test exercises `test_gsm8k_all_models` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_all_models`。

### Lines 139-140: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestNightlyGsm8KEval`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNightlyGsm8KEval.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestNightlyGsm8KEval.test_gsm8k_all_models`: This test exercises `test_gsm8k_all_models` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_all_models`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`, `warnings`, `types`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 140
