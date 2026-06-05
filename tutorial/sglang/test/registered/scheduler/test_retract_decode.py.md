# test_retract_decode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/scheduler/test_retract_decode.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates retract decode behavior in SGLang's scheduler area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 scheduler 领域中与 retract decode 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and dependencies / 模块导入与依赖
```python
import time
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
from sglang.utils import is_in_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `time`, `unittest`, `types`, `requests`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `time`, `unittest`, `types`, `requests`。

### Lines 20-21: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=353, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=600, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestRetractDecode declaration / 类 TestRetractDecode 声明
```python
class TestRetractDecode(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 25-27: class-level constants and configuration for `TestRetractDecode` / 类级常量与配置
```python
    """python -m unittest test_retract_decode.TestRetractDecode"""

    other_args = []
```
**EN:** This block defines shared names such as `other_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `other_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 29-43: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        launch_args = ["--chunked-prefill-size", "128"] + cls.other_args
        with (
            envs.SGLANG_TEST_RETRACT.override(True),
            envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_BUSY.override(1),
        ):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=launch_args,
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 45-47: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 49-62: test case mmlu / 测试用例 mmlu
```python
    def test_mmlu(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mmlu",
            num_examples=64,
            num_threads=32,
        )

        metrics = run_eval(args)
        self.assertGreaterEqual(metrics["score"], 0.65)
        time.sleep(1)  # wait for mem check

        assert self.process.poll() is None, "Server crashed during test"
```
**EN:** This test exercises `test_mmlu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu`。

### Lines 65-65: class TestRetractDecodePaged declaration / 类 TestRetractDecodePaged 声明
```python
class TestRetractDecodePaged(TestRetractDecode):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestRetractDecode`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestRetractDecode`。

### Lines 66-68: class-level constants and configuration for `TestRetractDecodePaged` / 类级常量与配置
```python
    """python -m unittest test_retract_decode.TestRetractDecodePaged"""

    other_args = ["--page-size", "16"]
```
**EN:** This block defines shared names such as `other_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `other_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 71-71: class TestRetractDecodeChunkCache declaration / 类 TestRetractDecodeChunkCache 声明
```python
class TestRetractDecodeChunkCache(TestRetractDecode):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestRetractDecode`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestRetractDecode`。

### Lines 72-74: class-level constants and configuration for `TestRetractDecodeChunkCache` / 类级常量与配置
```python
    """python -m unittest test_retract_decode.TestRetractDecodeChunkCache"""

    other_args = ["--disable-radix-cache"]
```
**EN:** This block defines shared names such as `other_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `other_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 77-77: class TestRetractDecodeChunkCachePaged declaration / 类 TestRetractDecodeChunkCachePaged 声明
```python
class TestRetractDecodeChunkCachePaged(TestRetractDecode):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestRetractDecode`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestRetractDecode`。

### Lines 78-80: class-level constants and configuration for `TestRetractDecodeChunkCachePaged` / 类级常量与配置
```python
    """python -m unittest test_retract_decode.TestRetractDecodeChunkCachePaged"""

    other_args = ["--disable-radix-cache", "--page-size", "16"]
```
**EN:** This block defines shared names such as `other_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `other_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 83-84: class TestRetractDecodeLongOutput declaration / 类 TestRetractDecodeLongOutput 声明
```python
@unittest.skipIf(is_in_ci(), "Skipped in CI due to long runtime")
class TestRetractDecodeLongOutput(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 85-87: class-level constants and configuration for `TestRetractDecodeLongOutput` / 类级常量与配置
```python
    """python -m unittest test_retract_decode.TestRetractDecodeLongOutput"""

    other_args = []
```
**EN:** This block defines shared names such as `other_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `other_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 89-104: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        launch_args = [
            "--chunked-prefill-size",
            "128",
            "--page-size",
            "16",
        ] + cls.other_args
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=launch_args,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 106-113: test case long output retract / 测试用例 long output retract
```python
    def test_long_output_retract(self):
        data = {
            "input_ids": [[233 + i] * 1234 for i in range(256)],
            "sampling_params": {"max_new_tokens": 90000, "ignore_eos": True},
        }
        res = requests.post(f"{self.base_url}/generate", json=data)
        assert res.status_code == 200, f"Request failed: {res.status_code}"
        assert self.process.poll() is None, "Server crashed during test"
```
**EN:** This test exercises `test_long_output_retract` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_long_output_retract`。

### Lines 116-117: class TestRetractDecodeLongOutputChunkCache declaration / 类 TestRetractDecodeLongOutputChunkCache 声明
```python
@unittest.skipIf(is_in_ci(), "Skipped in CI due to long runtime")
class TestRetractDecodeLongOutputChunkCache(TestRetractDecodeLongOutput):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestRetractDecodeLongOutput`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestRetractDecodeLongOutput`。

### Lines 118-120: class-level constants and configuration for `TestRetractDecodeLongOutputChunkCache` / 类级常量与配置
```python
    """python -m unittest test_retract_decode.TestRetractDecodeLongOutputChunkCache"""

    other_args = ["--disable-radix-cache"]
```
**EN:** This block defines shared names such as `other_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `other_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 123-124: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestRetractDecode`: python -m unittest test_retract_decode.TestRetractDecode / 用于组织相关测试、夹具或辅助方法。
- `TestRetractDecodePaged`: python -m unittest test_retract_decode.TestRetractDecodePaged / 用于组织相关测试、夹具或辅助方法。
- `TestRetractDecodeChunkCache`: python -m unittest test_retract_decode.TestRetractDecodeChunkCache / 用于组织相关测试、夹具或辅助方法。
- `TestRetractDecodeChunkCachePaged`: python -m unittest test_retract_decode.TestRetractDecodeChunkCachePaged / 用于组织相关测试、夹具或辅助方法。
- `TestRetractDecodeLongOutput`: python -m unittest test_retract_decode.TestRetractDecodeLongOutput / 用于组织相关测试、夹具或辅助方法。
- `TestRetractDecodeLongOutputChunkCache`: python -m unittest test_retract_decode.TestRetractDecodeLongOutputChunkCache / 用于组织相关测试、夹具或辅助方法。
- `TestRetractDecode.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestRetractDecode.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestRetractDecode.test_mmlu`: This test exercises `test_mmlu` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu`。
- `TestRetractDecodeLongOutput.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestRetractDecodeLongOutput.test_long_output_retract`: This test exercises `test_long_output_retract` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_long_output_retract`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `time`, `unittest`, `types`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`, `sglang.utils`

- **Total lines / 总行数**: 124
