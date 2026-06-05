# test_mixed_chunked_prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/scheduler/test_mixed_chunked_prefill.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mixed chunked prefill behavior in SGLang's scheduler area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 scheduler 领域中与 mixed chunked prefill 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`。

### Lines 15-16: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=167, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=180, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 19-19: class TestMixedChunkedPrefill declaration / 类 TestMixedChunkedPrefill 声明
```python
class TestMixedChunkedPrefill(GSM8KMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `GSM8KMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `GSM8KMixin`, `CustomTestCase`。

### Lines 20-28: class-level constants and configuration for `TestMixedChunkedPrefill` / 类级常量与配置
```python
    model = DEFAULT_MODEL_NAME_FOR_TEST
    base_url = DEFAULT_URL_FOR_TEST
    gsm8k_accuracy_thres = 0.62

    extra_args = [
        "--enable-mixed-chunk",
        "--chunked-prefill-size",
        "32",
    ]
```
**EN:** This block defines shared names such as `model`, `base_url`, `gsm8k_accuracy_thres`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `base_url`, `gsm8k_accuracy_thres`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 30-38: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        with envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_BUSY.override(2):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=cls.extra_args,
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

### Lines 45-45: class TestMixedChunkedPrefillNoRadixCache declaration / 类 TestMixedChunkedPrefillNoRadixCache 声明
```python
class TestMixedChunkedPrefillNoRadixCache(TestMixedChunkedPrefill):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestMixedChunkedPrefill`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestMixedChunkedPrefill`。

### Lines 46-51: class-level constants and configuration for `TestMixedChunkedPrefillNoRadixCache` / 类级常量与配置
```python
    extra_args = [
        "--enable-mixed-chunk",
        "--chunked-prefill-size",
        "32",
        "--disable-radix-cache",
    ]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 54-55: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMixedChunkedPrefill`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMixedChunkedPrefillNoRadixCache`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMixedChunkedPrefill.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestMixedChunkedPrefill.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 55
