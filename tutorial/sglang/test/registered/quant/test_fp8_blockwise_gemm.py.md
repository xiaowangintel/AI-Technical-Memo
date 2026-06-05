# test_fp8_blockwise_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_fp8_blockwise_gemm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fp8 blockwise gemm behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 fp8 blockwise gemm 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace
from urllib.parse import urlparse

from sglang.srt.utils import get_device_sm, kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
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
register_cuda_ci(est_time=430, stage="extra-b", runner_config="4-gpu-b200")

MODEL_PATH = "Qwen/Qwen3-4B-Instruct-2507-FP8"
MXFP8_MODEL_PATH = "zianglih/Qwen3-4B-Instruct-2507-MXFP8"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class FP8BlockwiseGemmBase declaration / 类 FP8BlockwiseGemmBase 声明
```python
class FP8BlockwiseGemmBase:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 22-22: class-level constants and configuration for `FP8BlockwiseGemmBase` / 类级常量与配置
```python
    backend = None
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 24-40: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if cls.backend is None:
            raise NotImplementedError("Subclass must set 'backend' attribute")
        cls.model = try_cached_model(MODEL_PATH)
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--fp8-gemm-backend",
            cls.backend,
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 42-44: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 46-61: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        parsed_url = urlparse(self.base_url)
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=1319,
            num_threads=200,
            num_shots=8,
        )
        metrics = run_eval(args)
        print(metrics)

        self.assertGreaterEqual(metrics["score"], 0.8)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 64-64: class MXFP8GemmBase declaration / 类 MXFP8GemmBase 声明
```python
class MXFP8GemmBase:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 65-65: class-level constants and configuration for `MXFP8GemmBase` / 类级常量与配置
```python
    backend = None
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 67-83: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if cls.backend is None:
            raise NotImplementedError("Subclass must set 'backend' attribute")
        cls.model = try_cached_model(MXFP8_MODEL_PATH)
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--fp8-gemm-backend",
            cls.backend,
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 85-87: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 89-104: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        parsed_url = urlparse(self.base_url)
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=1319,
            num_threads=200,
            num_shots=8,
        )
        metrics = run_eval(args)
        print(metrics)

        self.assertGreaterEqual(metrics["score"], 0.8)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 107-107: class TestFP8BlockwiseGemmTriton declaration / 类 TestFP8BlockwiseGemmTriton 声明
```python
class TestFP8BlockwiseGemmTriton(FP8BlockwiseGemmBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `FP8BlockwiseGemmBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `FP8BlockwiseGemmBase`, `unittest.TestCase`。

### Lines 108-108: class-level constants and configuration for `TestFP8BlockwiseGemmTriton` / 类级常量与配置
```python
    backend = "triton"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 111-111: class TestFP8BlockwiseGemmDeepGemm declaration / 类 TestFP8BlockwiseGemmDeepGemm 声明
```python
class TestFP8BlockwiseGemmDeepGemm(FP8BlockwiseGemmBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `FP8BlockwiseGemmBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `FP8BlockwiseGemmBase`, `unittest.TestCase`。

### Lines 112-112: class-level constants and configuration for `TestFP8BlockwiseGemmDeepGemm` / 类级常量与配置
```python
    backend = "deep_gemm"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 115-116: class TestFP8BlockwiseGemmFlashinferTrtllm declaration / 类 TestFP8BlockwiseGemmFlashinferTrtllm 声明
```python
@unittest.skipIf(get_device_sm() < 100, "Test requires CUDA SM 100 or higher")
class TestFP8BlockwiseGemmFlashinferTrtllm(FP8BlockwiseGemmBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `FP8BlockwiseGemmBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `FP8BlockwiseGemmBase`, `unittest.TestCase`。

### Lines 117-117: class-level constants and configuration for `TestFP8BlockwiseGemmFlashinferTrtllm` / 类级常量与配置
```python
    backend = "flashinfer_trtllm"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 120-121: class TestFP8BlockwiseGemmFlashinferDeepGemm declaration / 类 TestFP8BlockwiseGemmFlashinferDeepGemm 声明
```python
@unittest.skipIf(get_device_sm() != 90, "Test requires CUDA SM 90")
class TestFP8BlockwiseGemmFlashinferDeepGemm(FP8BlockwiseGemmBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `FP8BlockwiseGemmBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `FP8BlockwiseGemmBase`, `unittest.TestCase`。

### Lines 122-122: class-level constants and configuration for `TestFP8BlockwiseGemmFlashinferDeepGemm` / 类级常量与配置
```python
    backend = "flashinfer_deepgemm"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 125-127: class TestMXFP8GemmTriton declaration / 类 TestMXFP8GemmTriton 声明
```python
@unittest.skip("Currently PCG capture takes too long to complete, disable until fixed")
@unittest.skipIf(get_device_sm() < 100, "Test requires CUDA SM 100 or higher")
class TestMXFP8GemmTriton(MXFP8GemmBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `MXFP8GemmBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `MXFP8GemmBase`, `unittest.TestCase`。

### Lines 128-128: class-level constants and configuration for `TestMXFP8GemmTriton` / 类级常量与配置
```python
    backend = "triton"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 131-132: class TestMXFP8GemmFlashinferTrtllm declaration / 类 TestMXFP8GemmFlashinferTrtllm 声明
```python
@unittest.skipIf(get_device_sm() < 100, "Test requires CUDA SM 100 or higher")
class TestMXFP8GemmFlashinferTrtllm(MXFP8GemmBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `MXFP8GemmBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `MXFP8GemmBase`, `unittest.TestCase`。

### Lines 133-133: class-level constants and configuration for `TestMXFP8GemmFlashinferTrtllm` / 类级常量与配置
```python
    backend = "flashinfer_trtllm"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 136-137: class TestMXFP8GemmFlashinferCutlass declaration / 类 TestMXFP8GemmFlashinferCutlass 声明
```python
@unittest.skipIf(get_device_sm() < 100, "Test requires CUDA SM 100 or higher")
class TestMXFP8GemmFlashinferCutlass(MXFP8GemmBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `MXFP8GemmBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `MXFP8GemmBase`, `unittest.TestCase`。

### Lines 138-138: class-level constants and configuration for `TestMXFP8GemmFlashinferCutlass` / 类级常量与配置
```python
    backend = "flashinfer_cutlass"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 141-142: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `FP8BlockwiseGemmBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `MXFP8GemmBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFP8BlockwiseGemmTriton`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFP8BlockwiseGemmDeepGemm`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFP8BlockwiseGemmFlashinferTrtllm`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFP8BlockwiseGemmFlashinferDeepGemm`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMXFP8GemmTriton`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMXFP8GemmFlashinferTrtllm`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `FP8BlockwiseGemmBase.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `FP8BlockwiseGemmBase.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `FP8BlockwiseGemmBase.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `MXFP8GemmBase.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`, `urllib.parse`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 142
