# test_scheduler_status_logger.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/utils/test_scheduler_status_logger.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates scheduler status logger behavior in SGLang's utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 工具 领域中与 scheduler status logger 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import shutil
import tempfile
import time
import unittest
from pathlib import Path

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `shutil`, `tempfile`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `shutil`, `tempfile`。

### Lines 20-21: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=120, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=120, suite="nightly-amd-1-gpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestSchedulerStatusLogger declaration / 类 TestSchedulerStatusLogger 声明
```python
class TestSchedulerStatusLogger(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 25-39: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.temp_dir = tempfile.mkdtemp()
        cls.addClassCleanup(shutil.rmtree, cls.temp_dir)
        env = os.environ.copy()
        env["SGLANG_LOG_SCHEDULER_STATUS_TARGET"] = cls.temp_dir
        env["SGLANG_LOG_SCHEDULER_STATUS_INTERVAL"] = "1"
        cls.process = popen_launch_server(
            "Qwen/Qwen3-0.6B",
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--skip-server-warmup", "--enable-metrics"],
            env=env,
        )
        cls.addClassCleanup(kill_process_tree, cls.process.pid)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 41-61: test case scheduler status dump / 测试用例 scheduler status dump
```python
    def test_scheduler_status_dump(self):
        response = requests.post(
            DEFAULT_URL_FOR_TEST + "/generate",
            json={
                "text": "Hello",
                "sampling_params": {"max_new_tokens": 8, "temperature": 0},
            },
            timeout=30,
        )
        self.assertEqual(response.status_code, 200)

        time.sleep(2)

        events = list(_find_log_events(self.temp_dir, "scheduler.status"))
        print(f"{events=}")
        self.assertGreater(len(events), 0, "scheduler.status event not found")
        data = events[0]
        for field in ["timestamp", "rank", "running_rids", "queued_rids"]:
            self.assertIn(field, data)
        self.assertIsInstance(data["running_rids"], list)
        self.assertIsInstance(data["queued_rids"], list)
```
**EN:** This test exercises `test_scheduler_status_dump` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scheduler_status_dump`。

### Lines 64-75: function find log events / 函数 find log events
```python
def _find_log_events(log_dir: str, event_name: str):
    for f in Path(log_dir).glob("*.log"):
        for line in f.read_text().splitlines():
            idx = line.find("{")
            if idx == -1:
                continue
            try:
                data = json.loads(line[idx:])
            except json.JSONDecodeError:
                continue
            if data.get("event") == event_name:
                yield data
```
**EN:** This block implements `_find_log_events` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_find_log_events`，承担模块行为中的一个聚焦逻辑片段。

### Lines 78-79: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestSchedulerStatusLogger`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_find_log_events`: This block implements `_find_log_events` and captures one focused piece of the module's behavior. / 该代码块实现 `_find_log_events`，承担模块行为中的一个聚焦逻辑片段。
- `TestSchedulerStatusLogger.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSchedulerStatusLogger.test_scheduler_status_dump`: This test exercises `test_scheduler_status_dump` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scheduler_status_dump`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `shutil`, `tempfile`, `time`, `unittest`, `pathlib`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 79
