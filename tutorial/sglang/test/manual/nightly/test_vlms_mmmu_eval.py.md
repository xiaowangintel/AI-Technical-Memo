# test_vlms_mmmu_eval.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/nightly/test_vlms_mmmu_eval.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `vlms mmmu eval` scenario in `test/manual/nightly`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/nightly` 中的 `vlms mmmu eval` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-51: Constants and scenario settings / 常量与场景配置
```python
import json
import unittest
import warnings
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    ModelEvalMetrics,
    ModelLaunchSettings,
    check_evaluation_test_results,
    popen_launch_server,
    write_results_to_json,
)

MODEL_THRESHOLDS = {
    # Conservative thresholds on 100 MMMU samples, especially for latency thresholds
    ModelLaunchSettings("deepseek-ai/deepseek-vl2-small"): ModelEvalMetrics(
        0.330, 56.1
    ),
    ModelLaunchSettings("deepseek-ai/Janus-Pro-7B"): ModelEvalMetrics(0.285, 40.3),
    ModelLaunchSettings("Efficient-Large-Model/NVILA-8B-hf"): ModelEvalMetrics(
        0.270, 56.7
    ),
    ModelLaunchSettings("Efficient-Large-Model/NVILA-Lite-2B-hf"): ModelEvalMetrics(
        0.270, 23.8
    ),
    ModelLaunchSettings("google/gemma-3-4b-it"): ModelEvalMetrics(0.360, 10.9),
    ModelLaunchSettings("google/gemma-3n-E4B-it"): ModelEvalMetrics(0.360, 17.7),
    ModelLaunchSettings("mistral-community/pixtral-12b"): ModelEvalMetrics(0.360, 16.6),
    ModelLaunchSettings("moonshotai/Kimi-VL-A3B-Instruct"): ModelEvalMetrics(
        0.330, 22.3
    ),
    ModelLaunchSettings("openbmb/MiniCPM-o-2_6"): ModelEvalMetrics(0.330, 29.3),
    ModelLaunchSettings("openbmb/MiniCPM-v-2_6"): ModelEvalMetrics(0.259, 36.3),
    ModelLaunchSettings("OpenGVLab/InternVL2_5-2B"): ModelEvalMetrics(0.300, 17.0),
    ModelLaunchSettings("Qwen/Qwen2-VL-7B-Instruct"): ModelEvalMetrics(0.310, 83.3),
    ModelLaunchSettings("Qwen/Qwen2.5-VL-7B-Instruct"): ModelEvalMetrics(0.340, 31.9),
    ModelLaunchSettings(
        "Qwen/Qwen3-VL-30B-A3B-Instruct", extra_args=["--tp=2"]
    ): ModelEvalMetrics(0.29, 37.0),
    ModelLaunchSettings(
        "unsloth/Mistral-Small-3.1-24B-Instruct-2503"
    ): ModelEvalMetrics(0.310, 16.7),
    ModelLaunchSettings("XiaomiMiMo/MiMo-VL-7B-RL"): ModelEvalMetrics(0.28, 32.0),
    ModelLaunchSettings("zai-org/GLM-4.1V-9B-Thinking"): ModelEvalMetrics(0.280, 30.4),
}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `ModelLaunchSettings` and `ModelEvalMetrics`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 52-53: Class definition for TestNightlyVLMMmmuEval / 类定义
```python
class TestNightlyVLMMmmuEval(unittest.TestCase):
    @classmethod
```
**EN:** This range declares `TestNightlyVLMMmmuEval`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 54-56: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.models = list(MODEL_THRESHOLDS.keys())
        cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `keys`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 58-76: Test routines around test_mmmu_vlm_models / 测试例程
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
                process = popen_launch_server(
                    model=model_path,
                    base_url=self.base_url,
                    other_args=model.extra_args,
                    timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                )
                try:
                    args = SimpleNamespace(
                        base_url=self.base_url,
```
**EN:** This range defines concrete test routine(s) `test_mmmu_vlm_models`. The logic drives the target scenario and encodes the expected acceptance criteria. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `filterwarnings`, `subTest`, `popen_launch_server` and `SimpleNamespace`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 77-82: Scenario logic / 场景逻辑
```python
                        model=model_path,
                        eval_name="mmmu",
                        num_examples=100,
                        num_threads=64,
                        max_tokens=30,
                    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 83-101: Process control logic / 进程控制逻辑
```python

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
                        (model_path, metrics["score"], metrics["latency"])
                    )
                finally:
                    kill_process_tree(process.pid)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `run_eval`, `round`, `write_results_to_json` and `append`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 102-107: Scenario logic / 场景逻辑
```python

        try:
            with open("results.json", "r") as f:
                print("\nFinal Results from results.json:")
                print(json.dumps(json.load(f), indent=2))
        except Exception as e:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `dumps` and `load`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 108-123: Scenario logic / 场景逻辑
```python
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
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `items` and `check_evaluation_test_results`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 124-127: Script entry point / 脚本入口
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
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `types`, `unittest`, `warnings`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
