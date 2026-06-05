# test_heuristics.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/test/heuristics/test_heuristics.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides test fixtures, sample inputs, and validation helpers for the tooling layer itself.
- **Purpose (CN)**: 为工具层本身提供测试夹具、示例输入和验证辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```python
# For testing specific heuristics
from __future__ import annotations

import io
import json
import sys
import unittest
from pathlib import Path
from typing import Any
from unittest import mock
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, io, json, and 4 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、io、json 等共 7 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 12-23
```python

REPO_ROOT = Path(__file__).resolve().parents[3]
sys.path.append(str(REPO_ROOT))

from tools.test.heuristics.test_interface import TestTD
from tools.testing.target_determination.heuristics.filepath import (
    file_matches_keyword,
    get_keywords,
)
from tools.testing.target_determination.heuristics.historical_class_failure_correlation import (
    HistoricalClassFailurCorrelation,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.test.heuristics.test_interface, tools.testing.target_determination.heuristics.filepath, tools.testing.target_determination.heuristics.historical_class_failure_correlation. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.test.heuristics.test_interface、tools.testing.target_determination.heuristics.filepath、tools.testing.target_determination.heuristics.historical_class_failure_correlation。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 24-29
```python
from tools.testing.target_determination.heuristics.interface import TestPrioritizations
from tools.testing.target_determination.heuristics.previously_failed_in_pr import (
    get_previous_failures,
)
from tools.testing.test_run import TestRun
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.target_determination.heuristics.interface, tools.testing.target_determination.heuristics.previously_failed_in_pr, tools.testing.test_run.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.target_determination.heuristics.interface、tools.testing.target_determination.heuristics.previously_failed_in_pr、tools.testing.test_run。

### Lines 30-35
```python

sys.path.remove(str(REPO_ROOT))

HEURISTIC_CLASS = "tools.testing.target_determination.heuristics.historical_class_failure_correlation."
```
- **EN**: Configuration constants such as HEURISTIC_CLASS centralize defaults so later functions share the same policy knobs.
- **CN**: HEURISTIC_CLASS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 36-41
```python
def mocked_file(contents: dict[Any, Any]) -> io.IOBase:
    file_object = io.StringIO()
    json.dump(contents, file_object)
    file_object.seek(0)
    return file_object
```
- **EN**: This chunk defines `mocked_file`, which implements a focused step inside the tooling tests pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `mocked_file`，其作用是实现工具测试流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 42-53
```python

def gen_historical_class_failures() -> dict[str, dict[str, float]]:
    return {
        "file1": {
            "test1::classA": 0.5,
            "test2::classA": 0.2,
            "test5::classB": 0.1,
        },
        "file2": {
            "test1::classB": 0.3,
            "test3::classA": 0.2,
            "test5::classA": 1.5,
```
- **EN**: This chunk defines `gen_historical_class_failures`, which generates derived source text, templates, or metadata outputs. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `gen_historical_class_failures`，其作用是生成派生源码文本、模板或元数据输出。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 54-63
```python
            "test7::classC": 0.1,
        },
        "file3": {
            "test1::classC": 0.4,
            "test4::classA": 0.2,
            "test7::classC": 1.5,
            "test8::classC": 0.1,
        },
    }
```
- **EN**: This chunk continues `gen_historical_class_failures` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `gen_historical_class_failures`，进一步展开其内部控制流或数据流转。

### Lines 64-75
```python

ALL_TESTS = [
    "test1",
    "test2",
    "test3",
    "test4",
    "test5",
    "test6",
    "test7",
    "test8",
]
```
- **EN**: This chunk continues `gen_historical_class_failures` and expands its internal control flow or data movement. Configuration constants such as ALL_TESTS centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段延续了 `gen_historical_class_failures`，进一步展开其内部控制流或数据流转。 ALL_TESTS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 76-82
```python

class TestHistoricalClassFailureCorrelation(TestTD):
    @mock.patch(
        HEURISTIC_CLASS + "_get_historical_test_class_correlations",
        return_value=gen_historical_class_failures(),
    )
    @mock.patch(
```
- **EN**: It introduces classes such as TestHistoricalClassFailureCorrelation, which package state and behavior for this tooling task. This chunk continues `TestHistoricalClassFailureCorrelation` and expands its internal control flow or data movement.
- **CN**: 它引入了 TestHistoricalClassFailureCorrelation 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `TestHistoricalClassFailureCorrelation`，进一步展开其内部控制流或数据流转。

### Lines 83-92
```python
        HEURISTIC_CLASS + "query_changed_files",
        return_value=["file1"],
    )
    def test_get_prediction_confidence(
        self,
        historical_class_failures: dict[str, dict[str, float]],
        changed_files: list[str],
    ) -> None:
        tests_to_prioritize = ALL_TESTS
```
- **EN**: This chunk defines `test_get_prediction_confidence`, which implements a focused step inside the tooling tests pipeline.
- **CN**: 这一段定义了 `test_get_prediction_confidence`，其作用是实现工具测试流水线中的一个关键步骤。

### Lines 93-104
```python
        heuristic = HistoricalClassFailurCorrelation()
        test_prioritizations = heuristic.get_prediction_confidence(tests_to_prioritize)

        expected = TestPrioritizations(
            tests_to_prioritize,
            {
                TestRun("test1::classA"): 0.25,
                TestRun("test2::classA"): 0.1,
                TestRun("test5::classB"): 0.05,
                TestRun("test1", excluded=["classA"]): 0.0,
                TestRun("test2", excluded=["classA"]): 0.0,
                TestRun("test3"): 0.0,
```
- **EN**: This chunk continues `test_get_prediction_confidence` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段延续了 `test_get_prediction_confidence`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 105-112
```python
                TestRun("test4"): 0.0,
                TestRun("test5", excluded=["classB"]): 0.0,
                TestRun("test6"): 0.0,
                TestRun("test7"): 0.0,
                TestRun("test8"): 0.0,
            },
        )
```
- **EN**: This chunk continues `test_get_prediction_confidence` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段延续了 `test_get_prediction_confidence`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 113-118
```python
        self.assert_test_scores_almost_equal(
            test_prioritizations._test_scores, expected._test_scores
        )


class TestParsePrevTests(TestTD):
```
- **EN**: It introduces classes such as TestParsePrevTests, which package state and behavior for this tooling task. This chunk continues `TestParsePrevTests` and expands its internal control flow or data movement.
- **CN**: 它引入了 TestParsePrevTests 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `TestParsePrevTests`，进一步展开其内部控制流或数据流转。

### Lines 119-124
```python
    @mock.patch("os.path.exists", return_value=False)
    def test_cache_does_not_exist(self, mock_exists: Any) -> None:
        expected_failing_test_files: set[str] = set()

        found_tests = get_previous_failures()
```
- **EN**: This chunk defines `test_cache_does_not_exist`, which implements a focused step inside the tooling tests pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段定义了 `test_cache_does_not_exist`，其作用是实现工具测试流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 125-131
```python
        self.assertSetEqual(expected_failing_test_files, found_tests)

    @mock.patch("os.path.exists", return_value=True)
    @mock.patch("builtins.open", return_value=mocked_file({"": True}))
    def test_empty_cache(self, mock_exists: Any, mock_open: Any) -> None:
        expected_failing_test_files: set[str] = set()
```
- **EN**: This chunk defines `test_empty_cache`, which implements a focused step inside the tooling tests pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段定义了 `test_empty_cache`，其作用是实现工具测试流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 132-143
```python
        found_tests = get_previous_failures()

        self.assertSetEqual(expected_failing_test_files, found_tests)
        mock_open.assert_called()

    lastfailed_with_multiple_tests_per_file = {
        "test/test_car.py::TestCar::test_num[17]": True,
        "test/test_car.py::TestBar::test_num[25]": True,
        "test/test_far.py::TestFar::test_fun_copy[17]": True,
        "test/test_bar.py::TestBar::test_fun_copy[25]": True,
    }
```
- **EN**: This chunk continues `test_empty_cache` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_empty_cache`，进一步展开其内部控制流或数据流转。

### Lines 144-149
```python
    @mock.patch("os.path.exists", return_value=True)
    @mock.patch(
        "builtins.open",
        return_value=mocked_file(lastfailed_with_multiple_tests_per_file),
    )
    def test_dedupes_failing_test_files(self, mock_exists: Any, mock_open: Any) -> None:
```
- **EN**: This chunk defines `test_dedupes_failing_test_files`, which implements a focused step inside the tooling tests pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段定义了 `test_dedupes_failing_test_files`，其作用是实现工具测试流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 150-155
```python
        expected_failing_test_files = {"test_car", "test_bar", "test_far"}
        found_tests = get_previous_failures()

        self.assertSetEqual(expected_failing_test_files, found_tests)
```
- **EN**: This chunk continues `test_dedupes_failing_test_files` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_dedupes_failing_test_files`，进一步展开其内部控制流或数据流转。

### Lines 156-165
```python
class TestFilePath(TestTD):
    def test_get_keywords(self) -> None:
        self.assertEqual(get_keywords("test/test_car.py"), ["car"])
        self.assertEqual(get_keywords("test/nn/test_amp.py"), ["nn", "amp"])
        self.assertEqual(get_keywords("torch/nn/test_amp.py"), ["nn", "amp"])
        self.assertEqual(
            get_keywords("torch/nn/mixed_precision/test_something.py"),
            ["nn", "amp", "something"],
        )
```
- **EN**: It introduces classes such as TestFilePath, which package state and behavior for this tooling task. This chunk defines `test_get_keywords`, which implements a focused step inside the tooling tests pipeline.
- **CN**: 它引入了 TestFilePath 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `test_get_keywords`，其作用是实现工具测试流水线中的一个关键步骤。

### Lines 166-173
```python
    def test_match_keywords(self) -> None:
        self.assertTrue(file_matches_keyword("test/quantization/test_car.py", "quant"))
        self.assertTrue(file_matches_keyword("test/test_quantization.py", "quant"))
        self.assertTrue(file_matches_keyword("test/nn/test_amp.py", "nn"))
        self.assertTrue(file_matches_keyword("test/nn/test_amp.py", "amp"))
        self.assertTrue(file_matches_keyword("test/test_onnx.py", "onnx"))
        self.assertFalse(file_matches_keyword("test/test_onnx.py", "nn"))
```
- **EN**: This chunk defines `test_match_keywords`, which implements a focused step inside the tooling tests pipeline.
- **CN**: 这一段定义了 `test_match_keywords`，其作用是实现工具测试流水线中的一个关键步骤。

### Lines 174-179
```python
    def test_get_keywords_match(self) -> None:
        def helper(test_file: str, changed_file: str) -> bool:
            return any(
                file_matches_keyword(test_file, x) for x in get_keywords(changed_file)
            )
```
- **EN**: This chunk defines `helper`, which implements a focused step inside the tooling tests pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `helper`，其作用是实现工具测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 180-187
```python
        self.assertTrue(helper("test/quantization/test_car.py", "quantize/t.py"))
        self.assertFalse(helper("test/onnx/test_car.py", "nn/t.py"))
        self.assertTrue(helper("test/nn/test_car.py", "nn/t.py"))
        self.assertFalse(helper("test/nn/test_car.py", "test/b.py"))
        self.assertTrue(helper("test/test_mixed_precision.py", "torch/amp/t.py"))
        self.assertTrue(helper("test/test_amp.py", "torch/mixed_precision/t.py"))
        self.assertTrue(helper("test/idk/other/random.py", "torch/idk/t.py"))
```
- **EN**: This chunk continues `helper` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `helper`，进一步展开其内部控制流或数据流转。

### Lines 188-190
```python

if __name__ == "__main__":
    unittest.main()
```
- **EN**: This chunk continues `helper` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `helper`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Tooling tests**
  - EN: This file belongs to the tooling tests layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于工具测试层，应结合同一子目录中的相邻脚本一起理解。
- **Heuristic decision-making**
  - EN: The code uses lightweight rules to prioritize tests, files, or workflow actions.
  - CN: 代码使用轻量规则来排序测试、文件或工作流动作。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **REPO_ROOT**
  - EN: `REPO_ROOT` is one of the main local symbols exposed or implemented here.
  - CN: `REPO_ROOT` 是此处暴露或实现的主要局部符号之一。
- **HEURISTIC_CLASS**
  - EN: `HEURISTIC_CLASS` is one of the main local symbols exposed or implemented here.
  - CN: `HEURISTIC_CLASS` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.test.heuristics.test_interface`, `tools.testing.target_determination.heuristics.filepath`, `tools.testing.target_determination.heuristics.historical_class_failure_correlation`, `tools.testing.target_determination.heuristics.interface`, `tools.testing.target_determination.heuristics.previously_failed_in_pr`, `tools.testing.test_run`
- **Python standard library / Python 标准库**: `__future__`, `io`, `json`, `sys`, `unittest`, `pathlib`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `HEURISTIC_CLASS`, `mocked_file`, `gen_historical_class_failures`, `ALL_TESTS`, `TestHistoricalClassFailureCorrelation`, `TestParsePrevTests`, `TestFilePath`
