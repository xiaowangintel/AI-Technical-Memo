# test_radix_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/radix_cache/test_radix_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates radix attention behavior in SGLang's radix cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 Radix 缓存 领域中与 radix attention 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.environ import envs
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.radix_cache_server_kit import run_radix_attention_test
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    kill_process_tree,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.environ`, `sglang.test.ci.ci_register`, `sglang.test.kits.radix_cache_server_kit`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.environ`, `sglang.test.ci.ci_register`, `sglang.test.kits.radix_cache_server_kit`。

### Lines 15-16: supporting source context / 辅助源码上下文
```python

# RadixAttention server integration tests
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 17-18: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=100, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=100, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class TestRadixCacheFCFS declaration / 类 TestRadixCacheFCFS 声明
```python
class TestRadixCacheFCFS(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 22-38: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--chunked-prefill-size",
                "128",
                "--max-total-tokens",
                "20000",
                "--schedule-policy",
                "fcfs",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 40-42: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 44-45: test case radix attention / 测试用例 radix attention
```python
    def test_radix_attention(self):
        run_radix_attention_test(self.base_url)
```
**EN:** This test exercises `test_radix_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_radix_attention`。

### Lines 48-49: class TestRadixCacheLPM declaration / 类 TestRadixCacheLPM 声明
```python
@unittest.skipIf(is_in_ci(), "To reduce the CI execution time.")
class TestRadixCacheLPM(TestRadixCacheFCFS):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestRadixCacheFCFS`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestRadixCacheFCFS`。

### Lines 50-66: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--chunked-prefill-size",
                "128",
                "--max-total-tokens",
                "20000",
                "--schedule-policy",
                "lpm",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 69-69: class TestRadixCacheNonOverlapLPM declaration / 类 TestRadixCacheNonOverlapLPM 声明
```python
class TestRadixCacheNonOverlapLPM(TestRadixCacheFCFS):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestRadixCacheFCFS`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestRadixCacheFCFS`。

### Lines 70-87: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--disable-overlap-schedule",
                "--chunked-prefill-size",
                "128",
                "--max-total-tokens",
                "20000",
                "--schedule-policy",
                "lpm",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 90-93: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    envs.SGLANG_TEST_RETRACT.set(True)
    envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_BUSY.set(1)
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestRadixCacheFCFS`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRadixCacheLPM`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRadixCacheNonOverlapLPM`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRadixCacheFCFS.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestRadixCacheFCFS.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestRadixCacheFCFS.test_radix_attention`: This test exercises `test_radix_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_radix_attention`。
- `TestRadixCacheLPM.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestRadixCacheNonOverlapLPM.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.test.ci.ci_register`, `sglang.test.kits.radix_cache_server_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 93
