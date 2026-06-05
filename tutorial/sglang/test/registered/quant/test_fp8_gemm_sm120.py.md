# test_fp8_gemm_sm120.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_fp8_gemm_sm120.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fp8 gemm sm120 behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 fp8 gemm sm120 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace
from urllib.parse import urlparse

from sglang.srt.utils import get_device_sm, kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    popen_launch_server,
    try_cached_model,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `urllib.parse`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `urllib.parse`, `sglang.srt.utils`。

### Lines 15-18: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=146, stage="extra-a", runner_config="1-gpu-small")

PERTENSOR_MODEL_PATH = "nvidia/Llama-3.1-8B-Instruct-FP8"
BLOCKWISE_MODEL_PATH = "Qwen/Qwen3-4B-Instruct-2507-FP8"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class FP8GemmSM120Base declaration / 类 FP8GemmSM120Base 声明
```python
class FP8GemmSM120Base:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 22-24: class-level constants and configuration for `FP8GemmSM120Base` / 类级常量与配置
```python
    model_path = None
    backend = None
    quantization = None
```
**EN:** This block defines shared names such as `model_path`, `backend`, `quantization`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `backend`, `quantization` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 26-45: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if cls.backend is None:
            raise NotImplementedError("Subclass must set 'backend' attribute")
        cls.model = try_cached_model(cls.model_path)
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--fp8-gemm-backend",
            cls.backend,
            "--disable-piecewise-cuda-graph",
        ]
        if cls.quantization:
            other_args += ["--quantization", cls.quantization]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 47-50: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process"):
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 52-65: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        parsed_url = urlparse(self.base_url)
        args = SimpleNamespace(
            num_shots=self.num_shots,
            data_path=None,
            num_questions=1319,
            max_new_tokens=512,
            parallel=200,
            host=parsed_url.hostname,
            port=parsed_url.port,
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")
        self.assertGreaterEqual(metrics["accuracy"], self.accuracy_threshold)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 68-69: class TestFP8PerTensorGemmSM120Auto declaration / 类 TestFP8PerTensorGemmSM120Auto 声明
```python
@unittest.skipIf(get_device_sm() < 100, "Test requires CUDA SM 100 or higher")
class TestFP8PerTensorGemmSM120Auto(FP8GemmSM120Base, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `FP8GemmSM120Base`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `FP8GemmSM120Base`, `unittest.TestCase`。

### Lines 70-74: class-level constants and configuration for `TestFP8PerTensorGemmSM120Auto` / 类级常量与配置
```python
    model_path = PERTENSOR_MODEL_PATH
    backend = "auto"
    quantization = "modelopt_fp8"
    num_shots = 5
    accuracy_threshold = 0.73
```
**EN:** This block defines shared names such as `model_path`, `backend`, `quantization`, `num_shots`, `accuracy_threshold`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `backend`, `quantization`, `num_shots`, `accuracy_threshold` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 77-78: class TestFP8BlockwiseGemmSM120Auto declaration / 类 TestFP8BlockwiseGemmSM120Auto 声明
```python
@unittest.skipIf(get_device_sm() < 100, "Test requires CUDA SM 100 or higher")
class TestFP8BlockwiseGemmSM120Auto(FP8GemmSM120Base, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `FP8GemmSM120Base`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `FP8GemmSM120Base`, `unittest.TestCase`。

### Lines 79-82: class-level constants and configuration for `TestFP8BlockwiseGemmSM120Auto` / 类级常量与配置
```python
    model_path = BLOCKWISE_MODEL_PATH
    backend = "auto"
    num_shots = 8
    accuracy_threshold = 0.87
```
**EN:** This block defines shared names such as `model_path`, `backend`, `num_shots`, `accuracy_threshold`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `backend`, `num_shots`, `accuracy_threshold` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 85-86: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `FP8GemmSM120Base`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFP8PerTensorGemmSM120Auto`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFP8BlockwiseGemmSM120Auto`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `FP8GemmSM120Base.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `FP8GemmSM120Base.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `FP8GemmSM120Base.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`, `urllib.parse`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`

- **Total lines / 总行数**: 86
