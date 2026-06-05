# test_dump_metric.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/debug_utils/test_dump_metric.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `dump metric` scenario in `test/manual/debug_utils`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/debug_utils` 中的 `dump metric` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and shared helpers / 导入与共享辅助项
```python
"""Unit tests for dump_metric() function."""

import json
import os
import tempfile
import unittest
from pathlib import Path

from sglang.test.test_utils import dump_metric
```
**EN:** This range imports `json`, `os`, `tempfile` and `unittest`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Representative call sites include `dump_metric`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 12-16: Class definition for TestDumpMetric / 类定义
```python
class TestDumpMetric(unittest.TestCase):
    """Test suite for dump_metric() function."""

    _ENV_KEYS_TO_CLEAN = ["SGLANG_TEST_METRICS_OUTPUT", "PYTEST_CURRENT_TEST"]
```
**EN:** This range declares `TestDumpMetric`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `dump_metric`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 17-20: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUp(self):
        """Clean up env vars before each test."""
        for key in self._ENV_KEYS_TO_CLEAN:
            os.environ.pop(key, None)
```
**EN:** This range implements lifecycle helper(s) `setUp` to prepare or clean up shared resources across the test flow. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `pop`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 22-25: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDown(self):
        """Clean up env vars after each test."""
        for key in self._ENV_KEYS_TO_CLEAN:
            os.environ.pop(key, None)
```
**EN:** This range implements lifecycle helper(s) `tearDown` to prepare or clean up shared resources across the test flow. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `pop`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 27-36: Test routines around test_writes_valid_jsonl / 测试例程
```python
    def test_writes_valid_jsonl(self):
        """Test that dump_metric writes one valid JSON line when env is set."""
        with tempfile.TemporaryDirectory() as tmpdir:
            base_path = os.path.join(tmpdir, "metrics")
            os.environ["SGLANG_TEST_METRICS_OUTPUT"] = base_path

            dump_metric("test_accuracy", 0.95, labels={"model": "llama"})

            # Check file exists with PID suffix
            pid = os.getpid()
```
**EN:** This range defines concrete test routine(s) `test_writes_valid_jsonl`. The logic drives the target scenario and encodes the expected acceptance criteria. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `TemporaryDirectory`, `join`, `dump_metric` and `getpid`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 37-51: Assertions and result checks / 断言与结果检查
```python
            jsonl_path = f"{base_path}.{pid}.jsonl"
            self.assertTrue(os.path.exists(jsonl_path))

            # Read and validate
            with open(jsonl_path, encoding="utf-8") as f:
                lines = f.readlines()

            self.assertEqual(len(lines), 1)
            record = json.loads(lines[0])

            # Validate required fields
            self.assertIn("filename", record)
            self.assertIn("test_case", record)
            self.assertEqual(record["metric_name"], "test_accuracy")
            self.assertEqual(record["value"], 0.95)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`, `exists`, `readlines` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 52-56: Assertions and result checks / 断言与结果检查
```python

            # Validate optional fields
            self.assertIn("ts", record)
            self.assertIsInstance(record["ts"], (int, float))
            self.assertEqual(record["labels"], {"model": "llama"})
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIn`, `assertIsInstance` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 58-61: Test routines around test_no_env_no_file / 测试例程
```python
    def test_no_env_no_file(self):
        """Test that dump_metric doesn't create file when env var not set."""
        with tempfile.TemporaryDirectory() as tmpdir:
            # Don't set env var
```
**EN:** This range defines concrete test routine(s) `test_no_env_no_file`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `TemporaryDirectory`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 62-66: Assertions and result checks / 断言与结果检查
```python
            dump_metric("test_metric", 42)

            # Verify no files created
            files = list(Path(tmpdir).glob("*.jsonl"))
            self.assertEqual(len(files), 0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `dump_metric`, `Path`, `glob` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 68-86: Class definition for NonSerializable / 类定义
```python
    def test_labels_not_serializable_stringified(self):
        """Test that non-serializable labels are stringified."""
        with tempfile.TemporaryDirectory() as tmpdir:
            base_path = os.path.join(tmpdir, "metrics")
            os.environ["SGLANG_TEST_METRICS_OUTPUT"] = base_path

            # Non-serializable label
            class NonSerializable:
                pass

            dump_metric("test_metric", 100, labels={"obj": NonSerializable()})

            pid = os.getpid()
            jsonl_path = f"{base_path}.{pid}.jsonl"
            with open(jsonl_path, encoding="utf-8") as f:
                lines = f.readlines()

            record = json.loads(lines[0])
            self.assertIn("labels", record)
```
**EN:** This range declares `NonSerializable`, which organizes the scenario as a reusable test-oriented class. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Assertions in this block enforce the intended outcome. Representative call sites include `TemporaryDirectory`, `join`, `dump_metric` and `getpid`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 87-87: Assertions and result checks / 断言与结果检查
```python
            self.assertIsInstance(record["labels"], str)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsInstance`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 89-108: Test routines around test_bool_to_int / 测试例程
```python
    def test_bool_to_int(self):
        """Test that bool values are converted to int."""
        with tempfile.TemporaryDirectory() as tmpdir:
            base_path = os.path.join(tmpdir, "metrics")
            os.environ["SGLANG_TEST_METRICS_OUTPUT"] = base_path

            dump_metric("bool_true", True)
            dump_metric("bool_false", False)

            pid = os.getpid()
            jsonl_path = f"{base_path}.{pid}.jsonl"
            with open(jsonl_path, encoding="utf-8") as f:
                lines = f.readlines()

            self.assertEqual(len(lines), 2)
            record1 = json.loads(lines[0])
            record2 = json.loads(lines[1])

            self.assertEqual(record1["value"], 1)  # True -> 1
            self.assertEqual(record2["value"], 0)  # False -> 0
```
**EN:** This range defines concrete test routine(s) `test_bool_to_int`. The logic drives the target scenario and encodes the expected acceptance criteria. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Assertions in this block enforce the intended outcome. Representative call sites include `TemporaryDirectory`, `join`, `dump_metric` and `getpid`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 110-111: Test routines around test_pytest_current_test_parsing / 测试例程
```python
    def test_pytest_current_test_parsing(self):
        """Test PYTEST_CURRENT_TEST parsing for test_case."""
```
**EN:** This range defines concrete test routine(s) `test_pytest_current_test_parsing`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 112-128: Assertions and result checks / 断言与结果检查
```python
        with tempfile.TemporaryDirectory() as tmpdir:
            base_path = os.path.join(tmpdir, "metrics")
            os.environ["SGLANG_TEST_METRICS_OUTPUT"] = base_path
            os.environ["PYTEST_CURRENT_TEST"] = (
                "test/srt/test_example.py::TestClass::test_method (call)"
            )

            dump_metric("pytest_metric", 123)

            pid = os.getpid()
            jsonl_path = f"{base_path}.{pid}.jsonl"
            with open(jsonl_path, encoding="utf-8") as f:
                lines = f.readlines()

            record = json.loads(lines[0])
            # Only assert test_case parsing, not filename
            self.assertEqual(record["test_case"], "TestClass.test_method")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Assertions in this block enforce the intended outcome. Representative call sites include `TemporaryDirectory`, `join`, `test_method` and `dump_metric`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 129-132: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Environment-aware configuration / 环境感知配置
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `os`, `pathlib`, `tempfile`, `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.test_utils`
