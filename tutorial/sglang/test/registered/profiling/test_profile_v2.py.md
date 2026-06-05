# test_profile_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/profiling/test_profile_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates profile v2 behavior in SGLang's profiling area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 profiling 领域中与 profile v2 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and dependencies / 模块导入与依赖
```python
import os
import shutil
import tempfile
import unittest
from pathlib import Path

import requests

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `shutil`, `tempfile`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `shutil`, `tempfile`, `unittest`。

### Lines 20-25: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(
    est_time=120,
    stage="base-b",
    runner_config="1-gpu-small",
    disabled="Temporarily disabled",
)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 28-29: class TestStartProfile declaration / 类 TestStartProfile 声明
```python
class TestStartProfile(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 30-41: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.output_dir = tempfile.mkdtemp()
        envs.SGLANG_TORCH_PROFILER_DIR.set(cls.output_dir)
        envs.SGLANG_PROFILE_V2.set(True)
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 43-45: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 47-48: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self._clear_profile_dir()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 50-59: test case profile by stage / 测试用例 profile by stage
```python
    def test_profile_by_stage(self):
        self._start_profile(
            profile_by_stage=True,
            num_steps=10,
        )

        self._post_request()

        self._check_profile_output(pattern="*-prefill*", expect_existence=True)
        self._check_profile_output(pattern="*-decode*", expect_existence=True)
```
**EN:** This test exercises `test_profile_by_stage` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_by_stage`。

### Lines 61-71: test case decode only / 测试用例 decode only
```python
    def test_decode_only(self):
        self._start_profile(
            profile_by_stage=True,
            profile_stages=["decode"],
            num_steps=10,
        )

        self._post_request()

        self._check_profile_output(pattern="*-prefill*", expect_existence=False)  # NOTE
        self._check_profile_output(pattern="*-decode*", expect_existence=True)
```
**EN:** This test exercises `test_decode_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_only`。

### Lines 73-79: method start profile / 方法 start profile
```python
    def _start_profile(self, **kwargs):
        """Start profiling with optional parameters."""
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/start_profile",
            json=kwargs if kwargs else None,
        )
        self.assertEqual(response.status_code, 200)
```
**EN:** Start profiling with optional parameters. This block implements `_start_profile` and captures one focused piece of the module's behavior.
**CN:** Start profiling with optional parameters. 该代码块实现 `_start_profile`，承担模块行为中的一个聚焦逻辑片段。

### Lines 81-92: method post request / 方法 post request
```python
    def _post_request(self):
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 32,
                },
            },
        )
        self.assertEqual(response.status_code, 200)
```
**EN:** This block implements `_post_request` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_post_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 94-96: method clear profile dir / 方法 clear profile dir
```python
    def _clear_profile_dir(self):
        if os.path.isdir(self.output_dir):
            shutil.rmtree(self.output_dir)
```
**EN:** This block implements `_clear_profile_dir` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_clear_profile_dir`，承担模块行为中的一个聚焦逻辑片段。

### Lines 98-106: method check profile output / 方法 check profile output
```python
    def _check_profile_output(self, pattern: str, expect_existence: bool):
        self.assertTrue(
            os.path.isdir(self.output_dir), "Output directory does not exist."
        )
        self.assertEqual(
            len(list(Path(self.output_dir).glob(pattern))) > 0,
            expect_existence,
            f"Does not find {pattern=} ({list(Path(self.output_dir).glob('**/*'))=})",
        )
```
**EN:** This block implements `_check_profile_output` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_profile_output`，承担模块行为中的一个聚焦逻辑片段。

### Lines 109-110: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStartProfile`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStartProfile.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestStartProfile.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestStartProfile.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestStartProfile.test_profile_by_stage`: This test exercises `test_profile_by_stage` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_by_stage`。
- `TestStartProfile.test_decode_only`: This test exercises `test_decode_only` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_only`。
- `TestStartProfile._start_profile`: Start profiling with optional parameters. / 该代码块实现 `_start_profile`，承担模块行为中的一个聚焦逻辑片段。
- `TestStartProfile._post_request`: This block implements `_post_request` and captures one focused piece of the module's behavior. / 该代码块实现 `_post_request`，承担模块行为中的一个聚焦逻辑片段。
- `TestStartProfile._clear_profile_dir`: This block implements `_clear_profile_dir` and captures one focused piece of the module's behavior. / 该代码块实现 `_clear_profile_dir`，承担模块行为中的一个聚焦逻辑片段。
- `TestStartProfile._check_profile_output`: This block implements `_check_profile_output` and captures one focused piece of the module's behavior. / 该代码块实现 `_check_profile_output`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `shutil`, `tempfile`, `unittest`, `pathlib`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 110
