# test_engine_child_pids.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/core/test_engine_child_pids.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on core engine child pids in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 core engine child pids 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Document the module
```python
"""
Unit tests for Engine.get_all_child_pids().

Verifies that launching an Engine exposes the PIDs of all child processes
(schedulers, detokenizer) and that those PIDs correspond to live processes.

Usage:
    python -m unittest test_engine_child_pids -v
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 11-21: Import dependencies
```python
import os
import unittest

import psutil

import sglang as sgl
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    CustomTestCase,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 23-24: Register CI metadata
```python
register_cuda_ci(est_time=77, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=77, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 27-27: Define class TestEngineChildPids
```python
class TestEngineChildPids(CustomTestCase):
```
**EN:** This declaration introduces the `TestEngineChildPids` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestEngineChildPids` 测试类，并说明它通过继承承担的职责。

### Lines 29-56: Run test: get all child pids returns live pids
```python
    def test_get_all_child_pids_returns_live_pids(self):
        engine = sgl.Engine(
            model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
            random_seed=42,
        )
        try:
            pids = engine.get_all_child_pids()

            self.assertIsInstance(pids, list)
            self.assertGreater(len(pids), 0, "Expected at least one child PID")

            for pid in pids:
                self.assertIsInstance(pid, int)
                self.assertTrue(
                    psutil.pid_exists(pid),
                    f"PID {pid} does not correspond to a running process",
                )

            current_proc = psutil.Process(os.getpid())
            child_pids = {c.pid for c in current_proc.children(recursive=True)}
            for pid in pids:
                self.assertIn(
                    pid,
                    child_pids,
                    f"PID {pid} is not a child of the current process",
                )
        finally:
            engine.shutdown()
```
**EN:** This test method exercises get all child pids returns live pids and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 get all child pids returns live pids 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 58-72: Run test: child pids include scheduler and detokenizer
```python
    def test_child_pids_include_scheduler_and_detokenizer(self):
        engine = sgl.Engine(
            model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
            random_seed=42,
        )
        try:
            pids = engine.get_all_child_pids()
            # dp_size=1 gives one scheduler + one detokenizer = at least 2 PIDs
            self.assertGreaterEqual(
                len(pids),
                2,
                "Expected at least 2 child PIDs (scheduler + detokenizer)",
            )
        finally:
            engine.shutdown()
```
**EN:** This test method exercises child pids include scheduler and detokenizer and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child pids include scheduler and detokenizer 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 74-87: Run test: child pids no duplicates
```python
    def test_child_pids_no_duplicates(self):
        engine = sgl.Engine(
            model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
            random_seed=42,
        )
        try:
            pids = engine.get_all_child_pids()
            self.assertEqual(
                len(pids),
                len(set(pids)),
                f"Duplicate PIDs found: {pids}",
            )
        finally:
            engine.shutdown()
```
**EN:** This test method exercises child pids no duplicates and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 child pids no duplicates 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 90-91: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `psutil`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `CustomTestCase`, `unittest.main`
