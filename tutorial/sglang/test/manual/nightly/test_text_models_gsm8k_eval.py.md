# test_text_models_gsm8k_eval.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/nightly/test_text_models_gsm8k_eval.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `text models gsm8k eval` scenario in `test/manual/nightly`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/nightly` 中的 `text models gsm8k eval` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: Constants and scenario settings / 常量与场景配置
```python
import json
import unittest
import warnings
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP1,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP2,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1,
    DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    ModelLaunchSettings,
    check_evaluation_test_results,
    parse_models,
    popen_launch_server,
    write_results_to_json,
)

MODEL_SCORE_THRESHOLDS = {
    "meta-llama/Llama-3.1-8B-Instruct": 0.82,
    "mistralai/Mistral-7B-Instruct-v0.3": 0.58,
    "deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct": 0.85,
    "google/gemma-2-27b-it": 0.91,
    "meta-llama/Llama-3.1-70B-Instruct": 0.95,
    "mistralai/Mixtral-8x7B-Instruct-v0.1": 0.616,
    "Qwen/Qwen2-57B-A14B-Instruct": 0.86,
    "neuralmagic/Meta-Llama-3.1-8B-Instruct-FP8": 0.83,
    "neuralmagic/Mistral-7B-Instruct-v0.3-FP8": 0.54,
    "neuralmagic/DeepSeek-Coder-V2-Lite-Instruct-FP8": 0.835,
    "zai-org/GLM-4.5-Air-FP8": 0.75,
    # The threshold of neuralmagic/gemma-2-2b-it-FP8 should be 0.6, but this model has some accuracy regression.
    # The fix is tracked at https://github.com/sgl-project/sglang/issues/4324, we set it to 0.50, for now, to make CI green.
    "neuralmagic/gemma-2-2b-it-FP8": 0.50,
    "neuralmagic/Meta-Llama-3.1-70B-Instruct-FP8": 0.94,
    "neuralmagic/Mixtral-8x7B-Instruct-v0.1-FP8": 0.65,
    "neuralmagic/Qwen2-72B-Instruct-FP8": 0.94,
    "neuralmagic/Qwen2-57B-A14B-Instruct-FP8": 0.82,
}


# Do not use `CustomTestCase` since `test_mgsm_en_all_models` does not want retry
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 45-46: Class definition for TestNightlyGsm8KEval / 类定义
```python
class TestNightlyGsm8KEval(unittest.TestCase):
    @classmethod
```
**EN:** This range declares `TestNightlyGsm8KEval`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 47-61: Lifecycle helpers / 生命周期辅助逻辑
```python
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
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `parse_models`, `append` and `ModelLaunchSettings`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 63-69: Test routines around test_mgsm_en_all_models / 测试例程
```python
    def test_mgsm_en_all_models(self):
        warnings.filterwarnings(
            "ignore", category=ResourceWarning, message="unclosed.*socket"
        )
        is_first = True
        all_results = []
        for model_setup in self.models:
```
**EN:** This range defines concrete test routine(s) `test_mgsm_en_all_models`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `filterwarnings`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 70-87: Process control logic / 进程控制逻辑
```python
            with self.subTest(model=model_setup.model_path):
                other_args = list(model_setup.extra_args)

                if model_setup.model_path == "meta-llama/Llama-3.1-70B-Instruct":
                    other_args.extend(["--mem-fraction-static", "0.9"])

                process = popen_launch_server(
                    model=model_setup.model_path,
                    other_args=other_args,
                    base_url=self.base_url,
                    timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                )

                try:
                    args = SimpleNamespace(
                        base_url=self.base_url,
                        model=model_setup.model_path,
                        eval_name="mgsm_en",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `subTest`, `extend`, `popen_launch_server` and `SimpleNamespace`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 88-94: Scenario logic / 场景逻辑
```python
                        num_examples=None,
                        num_threads=1024,
                    )

                    metrics = run_eval(args)
                    print(
                        f"{'=' * 42}\n{model_setup.model_path} - metrics={metrics} score={metrics['score']}\n{'=' * 42}\n"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `run_eval`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 95-112: Process control logic / 进程控制逻辑
```python
                    )

                    write_results_to_json(
                        model_setup.model_path, metrics, "w" if is_first else "a"
                    )
                    is_first = False

                    # 0.0 for empty latency
                    all_results.append((model_setup.model_path, metrics["score"], 0.0))
                finally:
                    kill_process_tree(process.pid)

        try:
            with open("results.json", "r") as f:
                print("\nFinal Results from results.json:")
                print(json.dumps(json.load(f), indent=2))
        except Exception as e:
            print(f"Error reading results.json: {e}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `write_results_to_json`, `append`, `kill_process_tree` and `dumps`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 113-119: Scenario logic / 场景逻辑
```python

        # Check all scores after collecting all results
        check_evaluation_test_results(
            all_results,
            self.__class__.__name__,
            model_accuracy_thresholds=MODEL_SCORE_THRESHOLDS,
            model_count=len(self.models),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `check_evaluation_test_results`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 120-120: Scenario logic / 场景逻辑
```python
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 121-124: Script entry point / 脚本入口
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
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `types`, `unittest`, `warnings`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
