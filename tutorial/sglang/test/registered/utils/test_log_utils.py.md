# test_log_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/utils/test_log_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates log utils behavior in SGLang's utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 工具 领域中与 log utils 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and dependencies / 模块导入与依赖
```python
import io
import json
import re
import tempfile
import unittest
import uuid
from contextlib import redirect_stdout
from pathlib import Path

from sglang.srt.utils.log_utils import create_log_targets, log_json
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `io`, `json`, `re`, `tempfile`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `io`, `json`, `re`, `tempfile`。

### Lines 13-15: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")

_LOG_PREFIX_RE = re.compile(r"^\[\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\] ")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci, compile.
**CN:** 该代码块通过 register_cpu_ci, compile 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 18-18: class TestLogUtils declaration / 类 TestLogUtils 声明
```python
class TestLogUtils(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 19-32: test case stdout / 测试用例 stdout
```python
    def test_stdout(self):
        for targets in [["stdout"], None]:
            with self.subTest(targets=targets):
                buf = io.StringIO()
                with redirect_stdout(buf):
                    loggers = create_log_targets(
                        targets=targets, name_prefix=f"test_stdout_{uuid.uuid4()}"
                    )
                    self.assertEqual(len(loggers), 1)
                    log_json(loggers[0], "test.event", {"key": "value"})
                data = _parse_log_json(buf.getvalue().strip())
                self.assertIn("timestamp", data)
                self.assertEqual(data["event"], "test.event")
                self.assertEqual(data["key"], "value")
```
**EN:** This test exercises `test_stdout` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stdout`。

### Lines 34-45: test case file / 测试用例 file
```python
    def test_file(self):
        with tempfile.TemporaryDirectory() as temp_dir:
            loggers = create_log_targets(
                targets=[temp_dir], name_prefix=f"test_file_{uuid.uuid4()}"
            )
            self.assertEqual(len(loggers), 1)
            log_json(loggers, "file.event", {"data": 123})
            _flush_all(loggers)
            data = _read_log_file(temp_dir)
            self.assertIn("timestamp", data)
            self.assertEqual(data["event"], "file.event")
            self.assertEqual(data["data"], 123)
```
**EN:** This test exercises `test_file` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_file`。

### Lines 47-62: test case multiple targets / 测试用例 multiple targets
```python
    def test_multiple_targets(self):
        with tempfile.TemporaryDirectory() as temp_dir:
            buf = io.StringIO()
            with redirect_stdout(buf):
                loggers = create_log_targets(
                    targets=["stdout", temp_dir],
                    name_prefix=f"test_multi_{uuid.uuid4()}",
                )
                self.assertEqual(len(loggers), 2)
                log_json(loggers, "multi.event", {"x": 1})
            _flush_all(loggers)
            stdout_data = _parse_log_json(buf.getvalue().strip())
            file_data = _read_log_file(temp_dir)
            self.assertEqual(stdout_data["event"], "multi.event")
            self.assertEqual(file_data["event"], "multi.event")
            self.assertEqual(stdout_data["x"], file_data["x"])
```
**EN:** This test exercises `test_multiple_targets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_targets`。

### Lines 65-67: function parse log json / 函数 parse log json
```python
def _parse_log_json(line: str) -> dict:
    """Strip the ``[YYYY-MM-DD HH:MM:SS] `` prefix added by the formatter."""
    return json.loads(_LOG_PREFIX_RE.sub("", line))
```
**EN:** Strip the ``[YYYY-MM-DD HH:MM:SS] `` prefix added by the formatter. This block implements `_parse_log_json` and captures one focused piece of the module's behavior.
**CN:** Strip the ``[YYYY-MM-DD HH:MM:SS] `` prefix added by the formatter. 该代码块实现 `_parse_log_json`，承担模块行为中的一个聚焦逻辑片段。

### Lines 70-73: function flush all / 函数 flush all
```python
def _flush_all(loggers: list) -> None:
    for logger in loggers:
        for handler in logger.handlers:
            handler.flush()
```
**EN:** This block implements `_flush_all` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_flush_all`，承担模块行为中的一个聚焦逻辑片段。

### Lines 76-79: function read log file / 函数 read log file
```python
def _read_log_file(temp_dir: str) -> dict:
    log_files = list(Path(temp_dir).glob("*.log"))
    assert len(log_files) == 1
    return _parse_log_json(log_files[0].read_text().strip())
```
**EN:** This block implements `_read_log_file` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_read_log_file`，承担模块行为中的一个聚焦逻辑片段。

### Lines 82-83: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestLogUtils`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_parse_log_json`: Strip the ``[YYYY-MM-DD HH:MM:SS] `` prefix added by the formatter. / 该代码块实现 `_parse_log_json`，承担模块行为中的一个聚焦逻辑片段。
- `_flush_all`: This block implements `_flush_all` and captures one focused piece of the module's behavior. / 该代码块实现 `_flush_all`，承担模块行为中的一个聚焦逻辑片段。
- `_read_log_file`: This block implements `_read_log_file` and captures one focused piece of the module's behavior. / 该代码块实现 `_read_log_file`，承担模块行为中的一个聚焦逻辑片段。
- `TestLogUtils.test_stdout`: This test exercises `test_stdout` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stdout`。
- `TestLogUtils.test_file`: This test exercises `test_file` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_file`。
- `TestLogUtils.test_multiple_targets`: This test exercises `test_multiple_targets` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_targets`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `io`, `json`, `re`, `tempfile`, `unittest`, `uuid`, `contextlib`, `pathlib`
- **Internal modules / 内部模块**: `sglang.srt.utils.log_utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 83
