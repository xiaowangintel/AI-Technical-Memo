# test_ascend_deepseek_mtp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/ascend/test_ascend_deepseek_mtp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `ascend deepseek mtp` scenario in `test/manual/ascend`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/ascend` 中的 `ascend deepseek mtp` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Constants and scenario settings / 常量与场景配置
```python
import unittest
from types import SimpleNamespace
from urllib.parse import urlparse

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

TEST_MODEL_MATRIX = {
    "/root/.cache/modelscope/hub/models/vllm-ascend/DeepSeek-R1-0528-W8A8": {
        "accuracy": 0.95,
        "latency": 1000,
        "output_throughput": 6,
    },
}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 23-25: Class definition for TestAscendDeepSeekMTP / 类定义
```python
class TestAscendDeepSeekMTP(CustomTestCase):

    @classmethod
```
**EN:** This range declares `TestAscendDeepSeekMTP`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 26-47: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.models = TEST_MODEL_MATRIX.keys()
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.url = urlparse(DEFAULT_URL_FOR_TEST)

        cls.common_args = [
            "--trust-remote-code",
            "--attention-backend",
            "ascend",
            "--mem-fraction-static",
            0.8,
            "--disable-radix-cache",
            "--chunked-prefill-size",
            32768,
            "--tp-size",
            16,
            "--dp-size",
            2,
            "--enable-dp-attention",
            "--speculative-algorithm",
            "NEXTN",
            "--speculative-num-steps",
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `keys` and `urlparse`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 48-50: Scenario logic / 场景逻辑
```python
            1,
            "--speculative-eagle-topk",
            1,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 51-56: Environment-driven configuration / 环境驱动配置
```python
            "--speculative-num-draft-tokens",
            2,
        ]

        envs.SGLANG_NPU_USE_MLAPO.set(True)
        envs.SGLANG_ENABLE_OVERLAP_PLAN_STREAM.set(True)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 58-72: Test routines around test_a_gsm8k / 测试例程
```python
    def test_a_gsm8k(self):
        for model in self.models:
            with self.subTest(model=model):
                print(f"##=== Testing accuracy: {model} ===##")

                process = popen_launch_server(
                    model,
                    self.base_url,
                    timeout=2400,
                    other_args=[
                        *self.common_args,
                    ],
                )

                try:
```
**EN:** This range defines concrete test routine(s) `test_a_gsm8k`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `subTest` and `popen_launch_server`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 73-82: Scenario logic / 场景逻辑
```python
                    args = SimpleNamespace(
                        num_shots=5,
                        data_path=None,
                        num_questions=1319,
                        max_new_tokens=512,
                        parallel=128,
                        host=f"http://{self.url.hostname}",
                        port=int(self.url.port),
                    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `SimpleNamespace`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 83-89: Assertions and result checks / 断言与结果检查
```python
                    metrics = run_eval_few_shot_gsm8k(args)
                    self.assertGreaterEqual(
                        metrics["accuracy"],
                        TEST_MODEL_MATRIX[model]["accuracy"],
                    )
                finally:
                    kill_process_tree(process.pid)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Assertions in this block enforce the intended outcome. Representative call sites include `run_eval_few_shot_gsm8k`, `assertGreaterEqual` and `kill_process_tree`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-93: Script entry point / 脚本入口
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
- **Standard Library / 标准库**: `types`, `unittest`, `urllib.parse`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`
