# test_profile_merger.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/utils/test_profile_merger.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates profile merger behavior in SGLang's unit / utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 工具 领域中与 profile merger 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: supporting statements / 辅助语句
```python
"""
Unit tests for the ProfileMerger implementation.

Usage:
    python test_profile_merger.py
    python -m unittest test_profile_merger.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 9-18: module imports and dependencies / 模块导入与依赖
```python
import gzip
import json
import os
import shutil
import tempfile
import unittest

from sglang.srt.managers.io_struct import ProfileReq, ProfileReqInput, ProfileReqType
from sglang.srt.utils.profile_merger import ProfileMerger
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `gzip`, `json`, `os`, `shutil`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `gzip`, `json`, `os`, `shutil`。

### Lines 20-21: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=9, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=8, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestProfileMerger declaration / 类 TestProfileMerger 声明
```python
class TestProfileMerger(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 25-28: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.temp_dir = tempfile.mkdtemp()
        self.profile_id = "test_profile_123"
        self.merger = ProfileMerger(self.temp_dir, self.profile_id)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 30-31: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        shutil.rmtree(self.temp_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 33-62: test case rank extraction and labeling / 测试用例 rank extraction and labeling
```python
    def test_rank_extraction_and_labeling(self):
        # Test TP-only
        filename = f"{self.profile_id}-TP-0.trace.json.gz"
        rank_info = self.merger._extract_rank_info(filename)
        self.assertEqual(rank_info, {"tp_rank": 0})
        label = self.merger._create_rank_label(rank_info)
        self.assertEqual(label, "[TP00]")

        # Test all parallelism types
        filename = f"{self.profile_id}-TP-1-DP-2-PP-3-EP-4.trace.json.gz"
        rank_info = self.merger._extract_rank_info(filename)
        self.assertEqual(
            rank_info, {"tp_rank": 1, "dp_rank": 2, "pp_rank": 3, "ep_rank": 4}
        )
        label = self.merger._create_rank_label(rank_info)
        self.assertEqual(label, "[TP01-DP02-PP03-EP04]")

        # Test partial ranks
        filename = f"{self.profile_id}-TP-0-DP-1.trace.json.gz"
        rank_info = self.merger._extract_rank_info(filename)
        self.assertEqual(rank_info, {"tp_rank": 0, "dp_rank": 1})
        label = self.merger._create_rank_label(rank_info)
        self.assertEqual(label, "[TP00-DP01]")

        # Test no ranks
        filename = f"{self.profile_id}.trace.json.gz"
        rank_info = self.merger._extract_rank_info(filename)
        self.assertEqual(rank_info, {})
        label = self.merger._create_rank_label(rank_info)
        self.assertEqual(label, "[Unknown]")
```
**EN:** This test exercises `test_rank_extraction_and_labeling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank_extraction_and_labeling`。

### Lines 64-79: test case sort index calculation / 测试用例 sort index calculation
```python
    def test_sort_index_calculation(self):
        # Single rank
        rank_info = {"tp_rank": 0}
        sort_idx = self.merger._calculate_sort_index(rank_info, 83)
        self.assertEqual(sort_idx, 83)

        # Multiple ranks
        rank_info = {"tp_rank": 1, "dp_rank": 2, "pp_rank": 3, "ep_rank": 4}
        sort_idx = self.merger._calculate_sort_index(rank_info, 83)
        self.assertNotEqual(sort_idx, 83)
        self.assertGreater(sort_idx, 1000000)

        # Empty ranks
        rank_info = {}
        sort_idx = self.merger._calculate_sort_index(rank_info, 83)
        self.assertEqual(sort_idx, 83)
```
**EN:** This test exercises `test_sort_index_calculation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sort_index_calculation`。

### Lines 81-90: test case rank sort key / 测试用例 rank sort key
```python
    def test_rank_sort_key(self):
        # Full ranks: TP-1, DP-2, PP-3, EP-4 → sorted as (DP, EP, PP, TP)
        filename = f"{self.profile_id}-TP-1-DP-2-PP-3-EP-4.trace.json.gz"
        sort_key = self.merger._get_rank_sort_key(filename)
        self.assertEqual(sort_key, (2, 4, 3, 1))

        # Missing ranks: only TP-1 → sorted as (DP=0, EP=0, PP=0, TP=1)
        filename = f"{self.profile_id}-TP-1.trace.json.gz"
        sort_key = self.merger._get_rank_sort_key(filename)
        self.assertEqual(sort_key, (0, 0, 0, 1))
```
**EN:** This test exercises `test_rank_sort_key` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank_sort_key`。

### Lines 92-120: test case discover trace files / 测试用例 discover trace files
```python
    def test_discover_trace_files(self):
        # Create mock trace files
        trace_files = [
            f"{self.profile_id}-TP-0.trace.json.gz",  # Old format
            f"{self.profile_id}-TP-1.trace.json.gz",  # Old format
            f"{self.profile_id}-TP-0-DP-1.trace.json.gz",  # New format
        ]

        for filename in trace_files:
            filepath = os.path.join(self.temp_dir, filename)
            with gzip.open(filepath, "wt") as f:
                json.dump({"traceEvents": []}, f)

        discovered = self.merger._discover_trace_files()
        self.assertEqual(len(discovered), 3)

        # Check that all expected files are discovered
        discovered_basenames = {os.path.basename(f) for f in discovered}
        expected_basenames = {
            f"{self.profile_id}-TP-0.trace.json.gz",
            f"{self.profile_id}-TP-1.trace.json.gz",
            f"{self.profile_id}-TP-0-DP-1.trace.json.gz",
        }
        self.assertEqual(discovered_basenames, expected_basenames)

        # Test no matches
        empty_merger = ProfileMerger(self.temp_dir, "nonexistent")
        discovered = empty_merger._discover_trace_files()
        self.assertEqual(len(discovered), 0)
```
**EN:** This test exercises `test_discover_trace_files` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_discover_trace_files`。

### Lines 122-200: test case merge chrome traces / 测试用例 merge chrome traces
```python
    def test_merge_chrome_traces(self):
        # Create multiple trace files in random order
        trace_files = [
            {
                "filename": f"{self.profile_id}-TP-1-DP-1.trace.json.gz",
                "events": [
                    {"ph": "X", "name": "op1", "pid": 83, "ts": 1000.0, "dur": 10.0}
                ],
            },
            {
                "filename": f"{self.profile_id}-TP-0.trace.json.gz",
                "events": [
                    {"ph": "X", "name": "op2", "pid": 84, "ts": 2000.0, "dur": 15.0}
                ],
            },
            {
                "filename": f"{self.profile_id}-TP-0-DP-1.trace.json.gz",
                "events": [
                    {"ph": "X", "name": "op3", "pid": 85, "ts": 3000.0, "dur": 20.0}
                ],
            },
        ]

        for trace_data in trace_files:
            filepath = os.path.join(self.temp_dir, trace_data["filename"])
            trace_content = {
                "schemaVersion": 1,
                "deviceProperties": [{"device_id": 0, "name": "GPU-0"}],
                "traceEvents": trace_data["events"],
            }
            with gzip.open(filepath, "wt") as f:
                json.dump(trace_content, f)

        # Test file ordering by capturing log messages
        import logging

        logger = logging.getLogger("sglang.srt.utils.profile_merger")
        with self.assertLogs(logger, level="INFO") as log_capture:
            merged_path = self.merger.merge_chrome_traces()

        # Verify files were processed in rank order
        log_messages = [
            record.getMessage()
            for record in log_capture.records
            if "Processing file:" in record.getMessage()
        ]
        self.assertIn("TP-0.trace.json.gz", log_messages[0])  # (0,0,0,0) comes first
        self.assertIn(
            "TP-0-DP-1.trace.json.gz", log_messages[1]
        )  # (0,1,0,0) comes second
        self.assertIn(
            "TP-1-DP-1.trace.json.gz", log_messages[2]
        )  # (1,1,0,0) comes last

        # Verify merged content
        self.assertTrue(os.path.exists(merged_path))
        with gzip.open(merged_path, "rt") as f:
            merged_data = json.load(f)

        self.assertEqual(len(merged_data["traceEvents"]), 3)
        self.assertEqual(len(merged_data["deviceProperties"]), 3)

        # Check rank labels in events
        events = merged_data["traceEvents"]
        pids = [event["pid"] for event in events]
        self.assertIn("[TP00] 84", pids)
        self.assertIn("[TP00-DP01] 85", pids)
        self.assertIn("[TP01-DP01] 83", pids)

        # Test merge summary
        summary = self.merger.get_merge_summary()
        self.assertEqual(summary["total_files"], 3)
        self.assertEqual(summary["total_events"], 3)
        self.assertEqual(summary["profile_id"], self.profile_id)

        # Test no files error
        empty_merger = ProfileMerger(self.temp_dir, "nonexistent")
        with self.assertRaises(ValueError):
            empty_merger.merge_chrome_traces()
```
**EN:** This test exercises `test_merge_chrome_traces` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_chrome_traces`。

### Lines 203-204: class TestProfileMergerIntegration declaration / 类 TestProfileMergerIntegration 声明
```python
class TestProfileMergerIntegration(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 205-218: test case data structures merge profiles / 测试用例 data structures merge profiles
```python
    def test_data_structures_merge_profiles(self):
        # Test ProfileReqInput
        req_input = ProfileReqInput()
        self.assertFalse(req_input.merge_profiles)

        req_input = ProfileReqInput(merge_profiles=True)
        self.assertTrue(req_input.merge_profiles)

        # Test ProfileReq
        req = ProfileReq(type=ProfileReqType.START_PROFILE)
        self.assertFalse(req.merge_profiles)

        req = ProfileReq(type=ProfileReqType.START_PROFILE, merge_profiles=True)
        self.assertTrue(req.merge_profiles)
```
**EN:** This test exercises `test_data_structures_merge_profiles` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_data_structures_merge_profiles`。

### Lines 220-243: test case integration parameters / 测试用例 integration parameters
```python
    def test_integration_parameters(self):
        import inspect

        # Test TokenizerManager
        from sglang.srt.managers.tokenizer_control_mixin import (
            TokenizerControlMixin,
        )

        sig = inspect.signature(TokenizerControlMixin.start_profile)
        self.assertIn("merge_profiles", sig.parameters)

        # Test SchedulerProfilerMixin
        from sglang.srt.managers.scheduler_components.profiler_manager import (
            SchedulerProfilerManager,
        )

        sig = inspect.signature(SchedulerProfilerManager._init_profile)
        self.assertIn("merge_profiles", sig.parameters)

        # Test CLI profiler
        from sglang.profiler import run_profile

        sig = inspect.signature(run_profile)
        self.assertIn("merge_profiles", sig.parameters)
```
**EN:** This test exercises `test_integration_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_integration_parameters`。

### Lines 246-246: class TestProfileMergerEdgeCases declaration / 类 TestProfileMergerEdgeCases 声明
```python
class TestProfileMergerEdgeCases(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 247-250: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.temp_dir = tempfile.mkdtemp()
        self.profile_id = "test_edge_cases"
        self.merger = ProfileMerger(self.temp_dir, self.profile_id)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 252-255: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        import shutil

        shutil.rmtree(self.temp_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 257-290: test case error handling and edge cases / 测试用例 error handling and edge cases
```python
    def test_error_handling_and_edge_cases(self):
        # Test malformed trace file
        filename = f"{self.profile_id}-TP-0.trace.json.gz"
        filepath = os.path.join(self.temp_dir, filename)
        with gzip.open(filepath, "wt") as f:
            f.write("invalid json content")

        merged_path = self.merger.merge_chrome_traces()
        self.assertTrue(os.path.exists(merged_path))

        with gzip.open(merged_path, "rt") as f:
            merged_data = json.load(f)
        self.assertEqual(len(merged_data["traceEvents"]), 0)

        # Test empty trace file
        with gzip.open(filepath, "wt") as f:
            json.dump({}, f)
        merged_path = self.merger.merge_chrome_traces()
        self.assertTrue(os.path.exists(merged_path))

        # Test missing device properties
        trace_data = {
            "schemaVersion": 1,
            "traceEvents": [
                {"ph": "X", "name": "test", "pid": 83, "ts": 1000.0, "dur": 10.0}
            ],
        }
        with gzip.open(filepath, "wt") as f:
            json.dump(trace_data, f)

        merged_path = self.merger.merge_chrome_traces()
        with gzip.open(merged_path, "rt") as f:
            merged_data = json.load(f)
        self.assertNotIn("deviceProperties", merged_data)
```
**EN:** This test exercises `test_error_handling_and_edge_cases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_error_handling_and_edge_cases`。

### Lines 292-320: test case missing ranks and none handling / 测试用例 missing ranks and none handling
```python
    def test_missing_ranks_and_none_handling(self):
        # Test rank extraction with missing ranks
        filename = f"{self.profile_id}-TP-0.trace.json.gz"
        rank_info = self.merger._extract_rank_info(filename)
        self.assertEqual(rank_info, {"tp_rank": 0})

        # Test rank label creation with missing ranks
        label = self.merger._create_rank_label({"tp_rank": 0})
        self.assertEqual(label, "[TP00]")

        label = self.merger._create_rank_label({})
        self.assertEqual(label, "[Unknown]")

        # Test sort index calculation
        sort_idx = self.merger._calculate_sort_index({"tp_rank": 0}, 83)
        self.assertGreater(sort_idx, 0)

        sort_idx = self.merger._calculate_sort_index({}, 83)
        self.assertEqual(sort_idx, 83)

        # Test sort key generation
        sort_key = self.merger._get_rank_sort_key(filename)
        self.assertEqual(sort_key, (0, 0, 0, 0))

        # Test _maybe_cast_int with various inputs
        self.assertIsNone(self.merger._maybe_cast_int(None))
        self.assertIsNone(self.merger._maybe_cast_int("invalid"))
        self.assertEqual(self.merger._maybe_cast_int("123"), 123)
        self.assertEqual(self.merger._maybe_cast_int(456), 456)
```
**EN:** This test exercises `test_missing_ranks_and_none_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_ranks_and_none_handling`。

### Lines 322-365: test case mixed rank scenarios / 测试用例 mixed rank scenarios
```python
    def test_mixed_rank_scenarios(self):
        trace_scenarios = [
            {
                "filename": f"{self.profile_id}-TP-0.trace.json.gz",
                "events": [
                    {"ph": "X", "name": "op1", "pid": 83, "ts": 1000.0, "dur": 10.0}
                ],
            },
            {
                "filename": f"{self.profile_id}-TP-1-DP-0.trace.json.gz",
                "events": [
                    {"ph": "X", "name": "op2", "pid": 84, "ts": 2000.0, "dur": 15.0}
                ],
            },
            {
                "filename": f"{self.profile_id}-TP-0-DP-1-PP-0.trace.json.gz",
                "events": [
                    {"ph": "X", "name": "op3", "pid": 85, "ts": 3000.0, "dur": 20.0}
                ],
            },
        ]

        for scenario in trace_scenarios:
            filepath = os.path.join(self.temp_dir, scenario["filename"])
            trace_data = {
                "schemaVersion": 1,
                "deviceProperties": [{"device_id": 0, "name": "GPU-0"}],
                "traceEvents": scenario["events"],
            }
            with gzip.open(filepath, "wt") as f:
                json.dump(trace_data, f)

        merged_path = self.merger.merge_chrome_traces()
        self.assertTrue(os.path.exists(merged_path))

        with gzip.open(merged_path, "rt") as f:
            merged_data = json.load(f)

        self.assertEqual(len(merged_data["traceEvents"]), 3)
        events = merged_data["traceEvents"]
        pids = [event["pid"] for event in events]
        self.assertIn("[TP00] 83", pids)
        self.assertIn("[TP01-DP00] 84", pids)
        self.assertIn("[TP00-DP01-PP00] 85", pids)
```
**EN:** This test exercises `test_mixed_rank_scenarios` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_rank_scenarios`。

### Lines 368-369: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestProfileMerger`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestProfileMergerIntegration`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestProfileMergerEdgeCases`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestProfileMerger.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestProfileMerger.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestProfileMerger.test_rank_extraction_and_labeling`: This test exercises `test_rank_extraction_and_labeling` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank_extraction_and_labeling`。
- `TestProfileMerger.test_sort_index_calculation`: This test exercises `test_sort_index_calculation` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sort_index_calculation`。
- `TestProfileMerger.test_rank_sort_key`: This test exercises `test_rank_sort_key` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank_sort_key`。
- `TestProfileMerger.test_discover_trace_files`: This test exercises `test_discover_trace_files` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_discover_trace_files`。
- `TestProfileMerger.test_merge_chrome_traces`: This test exercises `test_merge_chrome_traces` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_chrome_traces`。
- `TestProfileMergerIntegration.test_data_structures_merge_profiles`: This test exercises `test_data_structures_merge_profiles` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_data_structures_merge_profiles`。
- `TestProfileMergerIntegration.test_integration_parameters`: This test exercises `test_integration_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_integration_parameters`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `gzip`, `json`, `os`, `shutil`, `tempfile`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.managers.io_struct`, `sglang.srt.utils.profile_merger`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 369
