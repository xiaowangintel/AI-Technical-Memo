# test_vlms_mmmu_eval.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/eval/test_vlms_mmmu_eval.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates vlms mmmu eval behavior in SGLang's eval area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 eval 领域中与 vlms mmmu eval 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest
import warnings
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    ModelEvalMetrics,
    ModelLaunchSettings,
    check_evaluation_test_results,
    popen_launch_server,
    write_results_to_json,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `warnings`, `types`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `warnings`, `types`。

### Lines 17-19: supporting source context / 辅助源码上下文
```python

# Nightly eval tests run large models that may need downloading on cache miss.
# Use a longer timeout than the default 600s.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 20-63: CI registration and metadata / CI 注册与元数据
```python
NIGHTLY_EVAL_SERVER_TIMEOUT = 1800

register_cuda_ci(est_time=7200, suite="nightly-eval-vlm-2-gpu", nightly=True)

MODEL_THRESHOLDS = {
    # Conservative thresholds on 100 MMMU samples, especially for latency thresholds
    ModelLaunchSettings("deepseek-ai/deepseek-vl2-small"): ModelEvalMetrics(
        0.320, 56.1
    ),
    ModelLaunchSettings("deepseek-ai/Janus-Pro-7B"): ModelEvalMetrics(0.285, 40.3),
    ModelLaunchSettings("Efficient-Large-Model/NVILA-8B-hf"): ModelEvalMetrics(
        0.270, 56.7
    ),
    ModelLaunchSettings("Efficient-Large-Model/NVILA-Lite-2B-hf"): ModelEvalMetrics(
        0.270, 23.8
    ),
    ModelLaunchSettings("google/gemma-4-E4B-it"): ModelEvalMetrics(0.26, 15.0),
    ModelLaunchSettings(
        "google/gemma-4-26B-A4B-it", extra_args=["--tp=2"]
    ): ModelEvalMetrics(0.27, 22.3),
    ModelLaunchSettings(
        "google/gemma-4-31B-it", extra_args=["--tp=2"]
    ): ModelEvalMetrics(0.28, 25.5),
    ModelLaunchSettings("mistral-community/pixtral-12b"): ModelEvalMetrics(0.360, 16.6),
    ModelLaunchSettings("moonshotai/Kimi-VL-A3B-Instruct"): ModelEvalMetrics(
        0.330, 23.5
    ),
    ModelLaunchSettings("openbmb/MiniCPM-o-2_6"): ModelEvalMetrics(0.330, 29.5),
    ModelLaunchSettings("openbmb/MiniCPM-v-2_6"): ModelEvalMetrics(0.259, 36.3),
    ModelLaunchSettings("OpenGVLab/InternVL2_5-2B"): ModelEvalMetrics(0.300, 18.0),
    ModelLaunchSettings("Qwen/Qwen2-VL-7B-Instruct"): ModelEvalMetrics(0.310, 83.3),
    ModelLaunchSettings("Qwen/Qwen2.5-VL-7B-Instruct"): ModelEvalMetrics(0.330, 31.9),
    ModelLaunchSettings(
        "Qwen/Qwen3-VL-30B-A3B-Instruct", extra_args=["--tp=2"]
    ): ModelEvalMetrics(0.29, 37.0),
    ModelLaunchSettings(
        "unsloth/Mistral-Small-3.1-24B-Instruct-2503"
    ): ModelEvalMetrics(0.30, 16.7),
    ModelLaunchSettings("XiaomiMiMo/MiMo-VL-7B-RL"): ModelEvalMetrics(0.28, 40.0),
    ModelLaunchSettings("zai-org/GLM-4.1V-9B-Thinking"): ModelEvalMetrics(0.280, 30.4),
    ModelLaunchSettings(
        "zai-org/GLM-4.5V-FP8", extra_args=["--tp=2"]
    ): ModelEvalMetrics(0.26, 34.0),
}
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, ModelLaunchSettings, ModelEvalMetrics.
**CN:** 该代码块通过 register_cuda_ci, ModelLaunchSettings, ModelEvalMetrics 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 66-66: class TestNightlyVLMMmmuEval declaration / 类 TestNightlyVLMMmmuEval 声明
```python
class TestNightlyVLMMmmuEval(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 67-70: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.models = list(MODEL_THRESHOLDS.keys())
        cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 72-149: test case mmmu vlm models / 测试用例 mmmu vlm models
```python
    def test_mmmu_vlm_models(self):
        warnings.filterwarnings(
            "ignore", category=ResourceWarning, message="unclosed.*socket"
        )
        is_first = True
        all_results = []

        for model in self.models:
            model_path = model.model_path
            with self.subTest(model=model_path):
                process = None
                try:
                    process = popen_launch_server(
                        model=model_path,
                        base_url=self.base_url,
                        other_args=model.extra_args,
                        timeout=NIGHTLY_EVAL_SERVER_TIMEOUT,
                    )

                    args = SimpleNamespace(
                        base_url=self.base_url,
                        model=model_path,
                        eval_name="mmmu",
                        num_examples=100,
                        num_threads=64,
                        max_tokens=30,
                    )

                    args.return_latency = True

                    metrics, latency = run_eval(args)

                    metrics["score"] = round(metrics["score"], 4)
                    metrics["latency"] = round(latency, 4)
                    print(
                        f"{'=' * 42}\n{model_path} - metrics={metrics} score={metrics['score']}\n{'=' * 42}\n"
                    )

                    write_results_to_json(model_path, metrics, "w" if is_first else "a")
                    is_first = False

                    all_results.append(
                        (
                            model_path,
                            metrics["score"],
                            metrics["latency"],
                            None,
                        )
                    )
                except Exception as e:
                    error_message = str(e)
                    all_results.append((model_path, None, None, error_message))
                    print(f"Error evaluating {model_path}: {error_message}")
                finally:
                    if process is not None:
                        kill_process_tree(process.pid)

        try:
            with open("results.json", "r") as f:
                print("\nFinal Results from results.json:")
                print(json.dumps(json.load(f), indent=2))
        except Exception as e:
            print(f"Error reading results: {e}")

        model_accuracy_thresholds = {
            model.model_path: threshold.accuracy
            for model, threshold in MODEL_THRESHOLDS.items()
        }
        model_latency_thresholds = {
            model.model_path: threshold.eval_time
            for model, threshold in MODEL_THRESHOLDS.items()
        }
        check_evaluation_test_results(
            all_results,
            self.__class__.__name__,
            model_accuracy_thresholds=model_accuracy_thresholds,
            model_latency_thresholds=model_latency_thresholds,
        )
```
**EN:** This test exercises `test_mmmu_vlm_models` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmmu_vlm_models`。

### Lines 152-153: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestNightlyVLMMmmuEval`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNightlyVLMMmmuEval.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestNightlyVLMMmmuEval.test_mmmu_vlm_models`: This test exercises `test_mmmu_vlm_models` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmmu_vlm_models`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`, `warnings`, `types`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 153
