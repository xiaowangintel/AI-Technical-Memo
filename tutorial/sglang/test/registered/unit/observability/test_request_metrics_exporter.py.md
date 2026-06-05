# test_request_metrics_exporter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/observability/test_request_metrics_exporter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates request metrics exporter behavior in SGLang's unit / observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / observability 领域中与 request metrics exporter 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for request_metrics_exporter.py — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-6: module imports and dependencies / 模块导入与依赖
```python
from dataclasses import dataclass
from typing import Any, Dict, List, Optional

from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `dataclasses`, `typing`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `dataclasses`, `typing`, `sglang.test.ci.ci_register`。

### Lines 8-8: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 10-19: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import json
import os
import shutil
import tempfile
import types
import unittest
from unittest.mock import MagicMock, patch

from sglang.srt.constants import HEALTH_CHECK_RID_PREFIX
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `json`, `os`, `shutil`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `json`, `os`, `shutil`。

### Lines 20-23: supporting source context / 辅助源码上下文
```python

# ── Test helper classes (local only, never injected into sys.modules) ──


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 24-25: class _GenerateReqInput declaration / 类 _GenerateReqInput 声明
```python
@dataclass
class _GenerateReqInput:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 26-29: class-level constants and configuration for `_GenerateReqInput` / 类级常量与配置
```python
    rid: Optional[str] = None
    text: Optional[str] = None
    image_data: Optional[Any] = None
    sampling_params: Optional[Dict] = None
```
**EN:** This block defines shared names such as `rid`, `text`, `image_data`, `sampling_params`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `rid`, `text`, `image_data`, `sampling_params` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 32-33: class _EmbeddingReqInput declaration / 类 _EmbeddingReqInput 声明
```python
@dataclass
class _EmbeddingReqInput:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 34-37: class-level constants and configuration for `_EmbeddingReqInput` / 类级常量与配置
```python
    rid: Optional[str] = None
    text: Optional[str] = None
    image_data: Optional[Any] = None
    input_ids: Optional[List[int]] = None
```
**EN:** This block defines shared names such as `rid`, `text`, `image_data`, `input_ids`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `rid`, `text`, `image_data`, `input_ids` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 40-40: class _ServerArgs declaration / 类 _ServerArgs 声明
```python
class _ServerArgs:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 41-43: method init / 方法 init
```python
    def __init__(self, **kwargs):
        for k, v in kwargs.items():
            setattr(self, k, v)
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 44-52: supporting source context / 辅助源码上下文
```python


# ── Deferred import of the module-under-test ──
# request_metrics_exporter.py imports io_struct and server_args at module level.
# We use patch.dict to temporarily provide lightweight stubs so the import
# succeeds without pulling in heavy transitive deps (torch, triton, …).
# The patch is started in setUpModule and stopped in tearDownModule,
# so sys.modules is never modified during pytest collection.

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 53-60: module-level constants and configuration / 模块级常量与配置
```python
_patcher = None

# Module-under-test symbols, populated by setUpModule
FileRequestMetricsExporter = None
RequestMetricsExporter = None
RequestMetricsExporterManager = None
create_request_metrics_exporters = None
_ConcreteExporter = None
```
**EN:** This block defines shared names such as `_patcher`, `FileRequestMetricsExporter`, `RequestMetricsExporter`, `RequestMetricsExporterManager`, `create_request_metrics_exporters`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_patcher`, `FileRequestMetricsExporter`, `RequestMetricsExporter`, `RequestMetricsExporterManager`, `create_request_metrics_exporters` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 63-105: function set Up Module / 函数 set Up Module
```python
def setUpModule():
    global _patcher
    global FileRequestMetricsExporter, RequestMetricsExporter
    global RequestMetricsExporterManager, create_request_metrics_exporters
    global _ConcreteExporter

    stub_modules = {}
    for name in (
        "sglang.srt.managers",
        "sglang.srt.managers.io_struct",
        "sglang.srt.server_args",
    ):
        if name not in __import__("sys").modules:
            stub_modules[name] = types.ModuleType(name)

    if stub_modules:
        if "sglang.srt.managers.io_struct" in stub_modules:
            stub_modules["sglang.srt.managers.io_struct"].GenerateReqInput = (
                _GenerateReqInput
            )
            stub_modules["sglang.srt.managers.io_struct"].EmbeddingReqInput = (
                _EmbeddingReqInput
            )
        if "sglang.srt.server_args" in stub_modules:
            stub_modules["sglang.srt.server_args"].ServerArgs = _ServerArgs

        _patcher = patch.dict("sys.modules", stub_modules)
        _patcher.start()

    import sglang.srt.observability.request_metrics_exporter as _mod

    FileRequestMetricsExporter = _mod.FileRequestMetricsExporter
    RequestMetricsExporter = _mod.RequestMetricsExporter
    RequestMetricsExporterManager = _mod.RequestMetricsExporterManager
    create_request_metrics_exporters = _mod.create_request_metrics_exporters

    class ConcreteExporter(RequestMetricsExporter):
        """Minimal concrete subclass for testing base class methods."""

        async def write_record(self, obj, out_dict):
            pass

    _ConcreteExporter = ConcreteExporter
```
**EN:** This block implements `setUpModule` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `setUpModule`，承担模块行为中的一个聚焦逻辑片段。

### Lines 108-110: function tear Down Module / 函数 tear Down Module
```python
def tearDownModule():
    if _patcher is not None:
        _patcher.stop()
```
**EN:** This block implements `tearDownModule` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `tearDownModule`，承担模块行为中的一个聚焦逻辑片段。

### Lines 111-115: supporting source context / 辅助源码上下文
```python


# ── Helpers ──


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 116-120: function make server args / 函数 make server args
```python
def _make_server_args(tmp_dir, enabled=True):
    return _ServerArgs(
        export_metrics_to_file=enabled,
        export_metrics_to_file_dir=tmp_dir,
    )
```
**EN:** This block implements `_make_server_args` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 123-123: class TestFormatOutputData declaration / 类 TestFormatOutputData 声明
```python
class TestFormatOutputData(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 124-141: test case basic formatting / 测试用例 basic formatting
```python
    def test_basic_formatting(self):
        server_args = _make_server_args("/tmp/unused")
        exporter = _ConcreteExporter(
            server_args, obj_skip_names=None, out_skip_names=None
        )

        obj = _GenerateReqInput(
            rid="req-1", text="hello", sampling_params={"temp": 0.5}
        )
        out_dict = {"meta_info": {"latency": 1.5, "tokens": 10}}

        result = exporter._format_output_data(obj, out_dict)

        params = json.loads(result["request_parameters"])
        self.assertEqual(params["rid"], "req-1")
        self.assertEqual(params["text"], "hello")
        self.assertIn("latency", result)
        self.assertIn("tokens", result)
```
**EN:** This test exercises `test_basic_formatting` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_formatting`。

### Lines 143-153: test case excludes always exclude fields / 测试用例 excludes always exclude fields
```python
    def test_excludes_always_exclude_fields(self):
        server_args = _make_server_args("/tmp/unused")
        exporter = _ConcreteExporter(
            server_args, obj_skip_names=None, out_skip_names=None
        )

        obj = _GenerateReqInput(rid="req-1", image_data="should_be_excluded")
        result = exporter._format_output_data(obj, {})

        params = json.loads(result["request_parameters"])
        self.assertNotIn("image_data", params)
```
**EN:** This test exercises `test_excludes_always_exclude_fields` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_excludes_always_exclude_fields`。

### Lines 155-166: test case excludes obj skip names / 测试用例 excludes obj skip names
```python
    def test_excludes_obj_skip_names(self):
        server_args = _make_server_args("/tmp/unused")
        exporter = _ConcreteExporter(
            server_args, obj_skip_names={"text"}, out_skip_names=None
        )

        obj = _GenerateReqInput(rid="req-1", text="skip_me")
        result = exporter._format_output_data(obj, {})

        params = json.loads(result["request_parameters"])
        self.assertNotIn("text", params)
        self.assertIn("rid", params)
```
**EN:** This test exercises `test_excludes_obj_skip_names` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_excludes_obj_skip_names`。

### Lines 168-178: test case excludes none values / 测试用例 excludes none values
```python
    def test_excludes_none_values(self):
        server_args = _make_server_args("/tmp/unused")
        exporter = _ConcreteExporter(
            server_args, obj_skip_names=None, out_skip_names=None
        )

        obj = _GenerateReqInput(rid="req-1", text=None)
        result = exporter._format_output_data(obj, {})

        params = json.loads(result["request_parameters"])
        self.assertNotIn("text", params)
```
**EN:** This test exercises `test_excludes_none_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_excludes_none_values`。

### Lines 180-191: test case filters out skip names / 测试用例 filters out skip names
```python
    def test_filters_out_skip_names(self):
        server_args = _make_server_args("/tmp/unused")
        exporter = _ConcreteExporter(
            server_args, obj_skip_names=None, out_skip_names={"secret"}
        )

        obj = _GenerateReqInput(rid="req-1")
        out_dict = {"meta_info": {"latency": 1.5, "secret": "hidden"}}
        result = exporter._format_output_data(obj, out_dict)

        self.assertIn("latency", result)
        self.assertNotIn("secret", result)
```
**EN:** This test exercises `test_filters_out_skip_names` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filters_out_skip_names`。

### Lines 194-194: class TestFileRequestMetricsExporter declaration / 类 TestFileRequestMetricsExporter 声明
```python
class TestFileRequestMetricsExporter(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 195-196: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tmp_dir = tempfile.mkdtemp()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 198-199: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        shutil.rmtree(self.tmp_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 201-202: method make exporter / 方法 make exporter
```python
    def _make_exporter(self):
        return FileRequestMetricsExporter(_make_server_args(self.tmp_dir), None, None)
```
**EN:** This block implements `_make_exporter` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_exporter`，承担模块行为中的一个聚焦逻辑片段。

### Lines 204-207: test case init creates directory / 测试用例 init creates directory
```python
    def test_init_creates_directory(self):
        sub_dir = os.path.join(self.tmp_dir, "nested", "dir")
        FileRequestMetricsExporter(_make_server_args(sub_dir), None, None)
        self.assertTrue(os.path.isdir(sub_dir))
```
**EN:** This test exercises `test_init_creates_directory` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_creates_directory`。

### Lines 209-214: test case ensure file handler opens file / 测试用例 ensure file handler opens file
```python
    def test_ensure_file_handler_opens_file(self):
        exporter = self._make_exporter()
        exporter._ensure_file_handler("20240101_12")
        self.assertIsNotNone(exporter._current_file_handler)
        self.assertEqual(exporter._current_hour_suffix, "20240101_12")
        exporter.close()
```
**EN:** This test exercises `test_ensure_file_handler_opens_file` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ensure_file_handler_opens_file`。

### Lines 216-223: test case ensure file handler rotates / 测试用例 ensure file handler rotates
```python
    def test_ensure_file_handler_rotates(self):
        exporter = self._make_exporter()
        exporter._ensure_file_handler("20240101_12")
        first_handler = exporter._current_file_handler
        exporter._ensure_file_handler("20240101_13")
        self.assertTrue(first_handler.closed)
        self.assertEqual(exporter._current_hour_suffix, "20240101_13")
        exporter.close()
```
**EN:** This test exercises `test_ensure_file_handler_rotates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ensure_file_handler_rotates`。

### Lines 225-235: test case ensure file handler close error / 测试用例 ensure file handler close error
```python
    def test_ensure_file_handler_close_error(self):
        """Previous handler close failure is logged but doesn't prevent rotation."""
        exporter = self._make_exporter()
        mock_handler = MagicMock()
        mock_handler.close.side_effect = OSError("disk error")
        exporter._current_file_handler = mock_handler
        exporter._current_hour_suffix = "old"

        exporter._ensure_file_handler("new")
        self.assertEqual(exporter._current_hour_suffix, "new")
        exporter.close()
```
**EN:** Previous handler close failure is logged but doesn't prevent rotation. This test exercises `test_ensure_file_handler_close_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Previous handler close failure is logged but doesn't prevent rotation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ensure_file_handler_close_error`。

### Lines 237-243: test case ensure file handler open error / 测试用例 ensure file handler open error
```python
    def test_ensure_file_handler_open_error(self):
        exporter = self._make_exporter()
        with patch("builtins.open", side_effect=OSError("permission denied")):
            with self.assertRaises(OSError):
                exporter._ensure_file_handler("20240101_12")
        self.assertIsNone(exporter._current_file_handler)
        self.assertIsNone(exporter._current_hour_suffix)
```
**EN:** This test exercises `test_ensure_file_handler_open_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ensure_file_handler_open_error`。

### Lines 245-250: test case close / 测试用例 close
```python
    def test_close(self):
        exporter = self._make_exporter()
        exporter._ensure_file_handler("20240101_12")
        exporter.close()
        self.assertIsNone(exporter._current_file_handler)
        self.assertIsNone(exporter._current_hour_suffix)
```
**EN:** This test exercises `test_close` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_close`。

### Lines 252-254: test case close noop when no handler / 测试用例 close noop when no handler
```python
    def test_close_noop_when_no_handler(self):
        exporter = self._make_exporter()
        exporter.close()  # should not raise
```
**EN:** This test exercises `test_close_noop_when_no_handler` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_close_noop_when_no_handler`。

### Lines 256-266: test case close error / 测试用例 close error
```python
    def test_close_error(self):
        """Close failure is logged but state is still reset."""
        exporter = self._make_exporter()
        mock_handler = MagicMock()
        mock_handler.close.side_effect = OSError("disk error")
        exporter._current_file_handler = mock_handler
        exporter._current_hour_suffix = "old"

        exporter.close()
        self.assertIsNone(exporter._current_file_handler)
        self.assertIsNone(exporter._current_hour_suffix)
```
**EN:** Close failure is logged but state is still reset. This test exercises `test_close_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Close failure is logged but state is still reset. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_close_error`。

### Lines 268-282: test case write record / 测试用例 write record
```python
    def test_write_record(self):
        exporter = self._make_exporter()
        obj = _GenerateReqInput(rid="req-1", text="hello")
        out_dict = {"meta_info": {"latency": 1.5}}

        asyncio.run(exporter.write_record(obj, out_dict))

        # Find the written file
        files = os.listdir(self.tmp_dir)
        self.assertEqual(len(files), 1)
        with open(os.path.join(self.tmp_dir, files[0])) as f:
            record = json.loads(f.readline())
        self.assertIn("request_parameters", record)
        self.assertAlmostEqual(record["latency"], 1.5)
        exporter.close()
```
**EN:** This test exercises `test_write_record` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_write_record`。

### Lines 284-290: test case write record skips health check / 测试用例 write record skips health check
```python
    def test_write_record_skips_health_check(self):
        exporter = self._make_exporter()
        obj = _GenerateReqInput(rid=f"{HEALTH_CHECK_RID_PREFIX}_123", text="ping")
        asyncio.run(exporter.write_record(obj, {}))

        files = os.listdir(self.tmp_dir)
        self.assertEqual(len(files), 0)
```
**EN:** This test exercises `test_write_record_skips_health_check` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_write_record_skips_health_check`。

### Lines 292-299: test case write record handler none / 测试用例 write record handler none
```python
    def test_write_record_handler_none(self):
        """If file handler is None after ensure, write_record returns early."""
        exporter = self._make_exporter()
        obj = _GenerateReqInput(rid="req-1")

        with patch.object(exporter, "_ensure_file_handler"):
            exporter._current_file_handler = None
            asyncio.run(exporter.write_record(obj, {}))
```
**EN:** If file handler is None after ensure, write_record returns early. This test exercises `test_write_record_handler_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** If file handler is None after ensure, write_record returns early. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_write_record_handler_none`。

### Lines 300-301: supporting source context / 辅助源码上下文
```python
        # No crash, no file written

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 302-310: test case write record exception / 测试用例 write record exception
```python
    def test_write_record_exception(self):
        """Exceptions during write are caught and logged."""
        exporter = self._make_exporter()
        obj = _GenerateReqInput(rid="req-1")

        with patch.object(
            exporter, "_ensure_file_handler", side_effect=RuntimeError("boom")
        ):
            asyncio.run(exporter.write_record(obj, {}))
```
**EN:** Exceptions during write are caught and logged. This test exercises `test_write_record_exception` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Exceptions during write are caught and logged. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_write_record_exception`。

### Lines 311-313: supporting source context / 辅助源码上下文
```python
        # Should not raise


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 314-314: class TestRequestMetricsExporterManager declaration / 类 TestRequestMetricsExporterManager 声明
```python
class TestRequestMetricsExporterManager(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 315-316: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tmp_dir = tempfile.mkdtemp()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 318-319: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        shutil.rmtree(self.tmp_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 321-324: test case no exporters / 测试用例 no exporters
```python
    def test_no_exporters(self):
        server_args = _make_server_args(self.tmp_dir, enabled=False)
        manager = RequestMetricsExporterManager(server_args)
        self.assertFalse(manager.exporter_enabled())
```
**EN:** This test exercises `test_no_exporters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_exporters`。

### Lines 326-329: test case with file exporter / 测试用例 with file exporter
```python
    def test_with_file_exporter(self):
        server_args = _make_server_args(self.tmp_dir, enabled=True)
        manager = RequestMetricsExporterManager(server_args)
        self.assertTrue(manager.exporter_enabled())
```
**EN:** This test exercises `test_with_file_exporter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_file_exporter`。

### Lines 331-340: test case write record delegates / 测试用例 write record delegates
```python
    def test_write_record_delegates(self):
        server_args = _make_server_args(self.tmp_dir, enabled=True)
        manager = RequestMetricsExporterManager(server_args)

        obj = _GenerateReqInput(rid="req-1", text="hello")
        out_dict = {"meta_info": {"latency": 1.0}}
        asyncio.run(manager.write_record(obj, out_dict))

        files = os.listdir(self.tmp_dir)
        self.assertEqual(len(files), 1)
```
**EN:** This test exercises `test_write_record_delegates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_write_record_delegates`。

### Lines 343-343: class TestCreateExporters declaration / 类 TestCreateExporters 声明
```python
class TestCreateExporters(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 344-345: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tmp_dir = tempfile.mkdtemp()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 347-348: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        shutil.rmtree(self.tmp_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 350-353: test case disabled / 测试用例 disabled
```python
    def test_disabled(self):
        server_args = _make_server_args(self.tmp_dir, enabled=False)
        exporters = create_request_metrics_exporters(server_args)
        self.assertEqual(len(exporters), 0)
```
**EN:** This test exercises `test_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disabled`。

### Lines 355-359: test case enabled / 测试用例 enabled
```python
    def test_enabled(self):
        server_args = _make_server_args(self.tmp_dir, enabled=True)
        exporters = create_request_metrics_exporters(server_args)
        self.assertEqual(len(exporters), 1)
        self.assertIsInstance(exporters[0], FileRequestMetricsExporter)
```
**EN:** This test exercises `test_enabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enabled`。

### Lines 362-363: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_GenerateReqInput`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_EmbeddingReqInput`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_ServerArgs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `setUpModule`: This block implements `setUpModule` and captures one focused piece of the module's behavior. / 该代码块实现 `setUpModule`，承担模块行为中的一个聚焦逻辑片段。
- `tearDownModule`: This block implements `tearDownModule` and captures one focused piece of the module's behavior. / 该代码块实现 `tearDownModule`，承担模块行为中的一个聚焦逻辑片段。
- `_make_server_args`: This block implements `_make_server_args` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。
- `TestFormatOutputData`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFileRequestMetricsExporter`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_ServerArgs.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `TestFormatOutputData.test_basic_formatting`: This test exercises `test_basic_formatting` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_formatting`。
- `TestFormatOutputData.test_excludes_always_exclude_fields`: This test exercises `test_excludes_always_exclude_fields` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_excludes_always_exclude_fields`。
- `TestFormatOutputData.test_excludes_obj_skip_names`: This test exercises `test_excludes_obj_skip_names` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_excludes_obj_skip_names`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`, `asyncio`, `json`, `os`, `shutil`, `tempfile`, `types`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.constants`

- **Total lines / 总行数**: 363
