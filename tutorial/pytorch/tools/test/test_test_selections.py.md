# test_test_selections.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/test/test_test_selections.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides test fixtures, sample inputs, and validation helpers for the tooling layer itself.
- **Purpose (CN)**: 为工具层本身提供测试夹具、示例输入和验证辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
from __future__ import annotations

import functools
import random
import sys
import unittest
from collections import defaultdict
from pathlib import Path


REPO_ROOT = Path(__file__).resolve().parents[2]
try:
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, functools, random, and 4 more. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、functools、random 等共 7 项。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 13-24
```python
    # using tools/ to optimize test run.
    sys.path.append(str(REPO_ROOT))
    from tools.testing.test_run import ShardedTest, TestRun
    from tools.testing.test_selections import calculate_shards, THRESHOLD
except ModuleNotFoundError:
    print("Can't import required modules, exiting")
    sys.exit(1)


def gen_class_times(test_times: dict[str, float]) -> dict[str, dict[str, float]]:
    return {k: {"class1": v} for k, v in test_times.items()}
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.test_run, tools.testing.test_selections. This chunk defines `gen_class_times`, which generates derived source text, templates, or metadata outputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.test_run、tools.testing.test_selections。 这一段定义了 `gen_class_times`，其作用是生成派生源码文本、模板或元数据输出。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 25-40
```python

class TestCalculateShards(unittest.TestCase):
    tests: list[TestRun] = [
        TestRun("super_long_test"),
        TestRun("long_test1"),
        TestRun("long_test2"),
        TestRun("normal_test1"),
        TestRun("normal_test2"),
        TestRun("normal_test3"),
        TestRun("short_test1"),
        TestRun("short_test2"),
        TestRun("short_test3"),
        TestRun("short_test4"),
        TestRun("short_test5"),
    ]
```
- **EN**: It introduces classes such as TestCalculateShards, which package state and behavior for this tooling task. This chunk continues `TestCalculateShards` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 它引入了 TestCalculateShards 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `TestCalculateShards`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 41-54
```python
    test_times: dict[str, float] = {
        "super_long_test": 55,
        "long_test1": 22,
        "long_test2": 18,
        "normal_test1": 9,
        "normal_test2": 7,
        "normal_test3": 5,
        "short_test1": 1,
        "short_test2": 0.6,
        "short_test3": 0.4,
        "short_test4": 0.3,
        "short_test5": 0.01,
    }
```
- **EN**: This chunk continues `TestCalculateShards` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TestCalculateShards`，进一步展开其内部控制流或数据流转。

### Lines 55-68
```python
    test_class_times: dict[str, dict[str, float]] = {
        "super_long_test": {"class1": 55},
        "long_test1": {"class1": 1, "class2": 21},
        "long_test2": {"class1": 10, "class2": 8},
        "normal_test1": {"class1": 9},
        "normal_test2": {"class1": 7},
        "normal_test3": {"class1": 5},
        "short_test1": {"class1": 1},
        "short_test2": {"class1": 0.6},
        "short_test3": {"class1": 0.4},
        "short_test4": {"class1": 0.3},
        "short_test5": {"class1": 0.01},
    }
```
- **EN**: This chunk continues `TestCalculateShards` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TestCalculateShards`，进一步展开其内部控制流或数据流转。

### Lines 69-92
```python
    def assert_shards_equal(
        self,
        expected_shards: list[tuple[float, list[ShardedTest]]],
        actual_shards: list[tuple[float, list[ShardedTest]]],
    ) -> None:
        for expected, actual in zip(expected_shards, actual_shards):
            self.assertAlmostEqual(expected[0], actual[0])
            self.assertListEqual(expected[1], actual[1])

    def test_no_times(self) -> None:
        # Check that round robin sharding is used when no times are provided
        expected_shards = [
            (
                0.0,
                [
                    ShardedTest(
                        test="super_long_test", shard=1, num_shards=1, time=None
                    ),
                    ShardedTest(test="long_test2", shard=1, num_shards=1, time=None),
                    ShardedTest(test="normal_test2", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test1", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test3", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test5", shard=1, num_shards=1, time=None),
                ],
```
- **EN**: This chunk defines `test_no_times`, which implements a focused step inside the tooling tests pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `test_no_times`，其作用是实现工具测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 93-109
```python
            ),
            (
                0.0,
                [
                    ShardedTest(test="long_test1", shard=1, num_shards=1, time=None),
                    ShardedTest(test="normal_test1", shard=1, num_shards=1, time=None),
                    ShardedTest(test="normal_test3", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test2", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test4", shard=1, num_shards=1, time=None),
                ],
            ),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(2, self.tests, {}, {}, sort_by_time=False),
        )
```
- **EN**: This chunk continues `test_no_times` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_no_times`，进一步展开其内部控制流或数据流转。

### Lines 110-133
```python
    def test_some_times_with_not_sort_by_time(self) -> None:
        expected_shards = [
            (
                400.0,
                [
                    ShardedTest(test="test_1", shard=1, num_shards=1, time=None),
                    ShardedTest(test="test_2", shard=1, num_shards=1, time=400),
                    ShardedTest(test="test_5", shard=1, num_shards=1, time=None),
                ],
            ),
            (
                300.0,
                [
                    ShardedTest(test="test_3", shard=1, num_shards=1, time=300),
                    ShardedTest(test="test_4", shard=1, num_shards=1, time=None),
                ],
            ),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(
                2,
                [
                    TestRun("test_1"),
```
- **EN**: This chunk defines `test_some_times_with_not_sort_by_time`, which implements a focused step inside the tooling tests pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段定义了 `test_some_times_with_not_sort_by_time`，其作用是实现工具测试流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 134-145
```python
                    TestRun("test_2"),
                    TestRun("test_3"),
                    TestRun("test_4"),
                    TestRun("test_5"),
                ],
                {"test_2": 400, "test_3": 300},
                {},
                sort_by_time=False,
            ),
        )

    def test_serial_parallel_interleaving(self) -> None:
```
- **EN**: This chunk defines `test_serial_parallel_interleaving`, which implements a focused step inside the tooling tests pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段定义了 `test_serial_parallel_interleaving`，其作用是实现工具测试流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 146-169
```python
        expected_shards = [
            (
                300.0,
                [
                    ShardedTest(test="test_1", shard=1, num_shards=1, time=None),
                    ShardedTest(test="test_3", shard=1, num_shards=1, time=300),
                    ShardedTest(test="test_4", shard=1, num_shards=1, time=None),
                ],
            ),
            (
                400.0,
                [
                    ShardedTest(test="test_2", shard=1, num_shards=1, time=400),
                    ShardedTest(test="test_5", shard=1, num_shards=1, time=None),
                ],
            ),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(
                2,
                [
                    TestRun("test_1"),
                    TestRun("test_2"),
```
- **EN**: This chunk continues `test_serial_parallel_interleaving` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段延续了 `test_serial_parallel_interleaving`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 170-181
```python
                    TestRun("test_3"),
                    TestRun("test_4"),
                    TestRun("test_5"),
                ],
                {"test_2": 400, "test_3": 300},
                {},
                must_serial=lambda x: x in ["test_1", "test_3"],
                sort_by_time=False,
            ),
        )

    def test_calculate_2_shards_with_complete_test_times(self) -> None:
```
- **EN**: This chunk defines `test_calculate_2_shards_with_complete_test_times`, which narrows a larger candidate set down to the items relevant for the current workflow. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段定义了 `test_calculate_2_shards_with_complete_test_times`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 182-205
```python
        expected_shards = [
            (
                60.0,
                [
                    ShardedTest(test="super_long_test", shard=1, num_shards=1, time=55),
                    ShardedTest(test="normal_test3", shard=1, num_shards=1, time=5),
                ],
            ),
            (
                58.31,
                [
                    ShardedTest(test="long_test1", shard=1, num_shards=1, time=22),
                    ShardedTest(test="long_test2", shard=1, num_shards=1, time=18),
                    ShardedTest(test="normal_test1", shard=1, num_shards=1, time=9),
                    ShardedTest(test="normal_test2", shard=1, num_shards=1, time=7),
                    ShardedTest(test="short_test1", shard=1, num_shards=1, time=1),
                    ShardedTest(test="short_test2", shard=1, num_shards=1, time=0.6),
                    ShardedTest(test="short_test3", shard=1, num_shards=1, time=0.4),
                    ShardedTest(test="short_test4", shard=1, num_shards=1, time=0.3),
                    ShardedTest(test="short_test5", shard=1, num_shards=1, time=0.01),
                ],
            ),
        ]
        self.assert_shards_equal(
```
- **EN**: This chunk continues `test_calculate_2_shards_with_complete_test_times` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_calculate_2_shards_with_complete_test_times`，进一步展开其内部控制流或数据流转。

### Lines 206-229
```python
            expected_shards,
            calculate_shards(2, self.tests, self.test_times, self.test_class_times),
        )

    def test_calculate_1_shard_with_complete_test_times(self) -> None:
        tests = self.tests.copy()
        class_test1 = TestRun("long_test1", excluded=["class2"])
        class_test2 = TestRun("long_test1", included=["class2"])
        tests.append(class_test1)
        tests.append(class_test2)

        expected_shards = [
            (
                140.31,
                [
                    ShardedTest(test="super_long_test", shard=1, num_shards=1, time=55),
                    ShardedTest(test="long_test1", shard=1, num_shards=1, time=22),
                    ShardedTest(class_test2, shard=1, num_shards=1, time=21),
                    ShardedTest(test="long_test2", shard=1, num_shards=1, time=18),
                    ShardedTest(test="normal_test1", shard=1, num_shards=1, time=9),
                    ShardedTest(test="normal_test2", shard=1, num_shards=1, time=7),
                    ShardedTest(test="normal_test3", shard=1, num_shards=1, time=5),
                    ShardedTest(test="short_test1", shard=1, num_shards=1, time=1),
                    ShardedTest(class_test1, shard=1, num_shards=1, time=1),
```
- **EN**: This chunk defines `test_calculate_1_shard_with_complete_test_times`, which narrows a larger candidate set down to the items relevant for the current workflow. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段定义了 `test_calculate_1_shard_with_complete_test_times`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 230-241
```python
                    ShardedTest(test="short_test2", shard=1, num_shards=1, time=0.6),
                    ShardedTest(test="short_test3", shard=1, num_shards=1, time=0.4),
                    ShardedTest(test="short_test4", shard=1, num_shards=1, time=0.3),
                    ShardedTest(test="short_test5", shard=1, num_shards=1, time=0.01),
                ],
            )
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(1, tests, self.test_times, self.test_class_times),
        )
```
- **EN**: This chunk continues `test_calculate_1_shard_with_complete_test_times` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_calculate_1_shard_with_complete_test_times`，进一步展开其内部控制流或数据流转。

### Lines 242-265
```python
    def test_calculate_5_shards_with_complete_test_times(self) -> None:
        expected_shards = [
            (
                55.0,
                [ShardedTest(test="super_long_test", shard=1, num_shards=1, time=55)],
            ),
            (22.0, [ShardedTest(test="long_test1", shard=1, num_shards=1, time=22)]),
            (18.0, [ShardedTest(test="long_test2", shard=1, num_shards=1, time=18)]),
            (
                11.31,
                [
                    ShardedTest(test="normal_test1", shard=1, num_shards=1, time=9),
                    ShardedTest(test="short_test1", shard=1, num_shards=1, time=1),
                    ShardedTest(test="short_test2", shard=1, num_shards=1, time=0.6),
                    ShardedTest(test="short_test3", shard=1, num_shards=1, time=0.4),
                    ShardedTest(test="short_test4", shard=1, num_shards=1, time=0.3),
                    ShardedTest(test="short_test5", shard=1, num_shards=1, time=0.01),
                ],
            ),
            (
                12.0,
                [
                    ShardedTest(test="normal_test2", shard=1, num_shards=1, time=7),
                    ShardedTest(test="normal_test3", shard=1, num_shards=1, time=5),
```
- **EN**: This chunk defines `test_calculate_5_shards_with_complete_test_times`, which narrows a larger candidate set down to the items relevant for the current workflow.
- **CN**: 这一段定义了 `test_calculate_5_shards_with_complete_test_times`，其作用是从更大的候选集中筛选出当前工作流相关的项目。

### Lines 266-289
```python
                ],
            ),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(5, self.tests, self.test_times, self.test_class_times),
        )

    def test_calculate_2_shards_with_incomplete_test_times(self) -> None:
        incomplete_test_times = {
            k: v for k, v in self.test_times.items() if "test1" in k
        }
        expected_shards = [
            (
                22.0,
                [
                    ShardedTest(test="long_test1", shard=1, num_shards=1, time=22),
                    ShardedTest(
                        test="super_long_test", shard=1, num_shards=1, time=None
                    ),
                    ShardedTest(test="normal_test2", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test2", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test4", shard=1, num_shards=1, time=None),
                ],
```
- **EN**: This chunk defines `test_calculate_2_shards_with_incomplete_test_times`, which narrows a larger candidate set down to the items relevant for the current workflow. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `test_calculate_2_shards_with_incomplete_test_times`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 290-312
```python
            ),
            (
                10.0,
                [
                    ShardedTest(test="normal_test1", shard=1, num_shards=1, time=9),
                    ShardedTest(test="short_test1", shard=1, num_shards=1, time=1),
                    ShardedTest(test="long_test2", shard=1, num_shards=1, time=None),
                    ShardedTest(test="normal_test3", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test3", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test5", shard=1, num_shards=1, time=None),
                ],
            ),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(
                2,
                self.tests,
                incomplete_test_times,
                gen_class_times(incomplete_test_times),
            ),
        )
```
- **EN**: This chunk continues `test_calculate_2_shards_with_incomplete_test_times` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_calculate_2_shards_with_incomplete_test_times`，进一步展开其内部控制流或数据流转。

### Lines 313-336
```python
    def test_calculate_5_shards_with_incomplete_test_times(self) -> None:
        incomplete_test_times = {
            k: v for k, v in self.test_times.items() if "test1" in k
        }
        expected_shards = [
            (
                22.0,
                [
                    ShardedTest(test="long_test1", shard=1, num_shards=1, time=22),
                    ShardedTest(
                        test="super_long_test", shard=1, num_shards=1, time=None
                    ),
                    ShardedTest(test="short_test3", shard=1, num_shards=1, time=None),
                ],
            ),
            (
                9.0,
                [
                    ShardedTest(test="normal_test1", shard=1, num_shards=1, time=9),
                    ShardedTest(test="long_test2", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test4", shard=1, num_shards=1, time=None),
                ],
            ),
            (
```
- **EN**: This chunk defines `test_calculate_5_shards_with_incomplete_test_times`, which narrows a larger candidate set down to the items relevant for the current workflow. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `test_calculate_5_shards_with_incomplete_test_times`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 337-360
```python
                1.0,
                [
                    ShardedTest(test="short_test1", shard=1, num_shards=1, time=1),
                    ShardedTest(test="normal_test2", shard=1, num_shards=1, time=None),
                    ShardedTest(test="short_test5", shard=1, num_shards=1, time=None),
                ],
            ),
            (
                0.0,
                [
                    ShardedTest(test="normal_test3", shard=1, num_shards=1, time=None),
                ],
            ),
            (
                0.0,
                [
                    ShardedTest(test="short_test2", shard=1, num_shards=1, time=None),
                ],
            ),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(
                5,
```
- **EN**: This chunk continues `test_calculate_5_shards_with_incomplete_test_times` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_calculate_5_shards_with_incomplete_test_times`，进一步展开其内部控制流或数据流转。

### Lines 361-382
```python
                self.tests,
                incomplete_test_times,
                gen_class_times(incomplete_test_times),
            ),
        )

    def test_split_shards(self) -> None:
        test_times: dict[str, float] = {"test1": THRESHOLD, "test2": THRESHOLD}
        expected_shards = [
            (600.0, [ShardedTest(test="test1", shard=1, num_shards=1, time=THRESHOLD)]),
            (600.0, [ShardedTest(test="test2", shard=1, num_shards=1, time=THRESHOLD)]),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(
                2,
                [TestRun(t) for t in test_times],
                test_times,
                gen_class_times(test_times),
            ),
        )
```
- **EN**: This chunk defines `test_split_shards`, which narrows a larger candidate set down to the items relevant for the current workflow. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `test_split_shards`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 383-406
```python
        test_times = {"test1": THRESHOLD * 4, "test2": THRESHOLD * 2.5}
        expected_shards = [
            (
                2200.0,
                [
                    ShardedTest(test="test1", shard=1, num_shards=4, time=600.0),
                    ShardedTest(test="test1", shard=3, num_shards=4, time=600.0),
                    ShardedTest(test="test2", shard=1, num_shards=3, time=500.0),
                    ShardedTest(test="test2", shard=3, num_shards=3, time=500.0),
                ],
            ),
            (
                1700.0,
                [
                    ShardedTest(test="test1", shard=2, num_shards=4, time=600.0),
                    ShardedTest(test="test1", shard=4, num_shards=4, time=600.0),
                    ShardedTest(test="test2", shard=2, num_shards=3, time=500.0),
                ],
            ),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(
                2,
```
- **EN**: This chunk continues `test_split_shards` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_split_shards`，进一步展开其内部控制流或数据流转。

### Lines 407-430
```python
                [TestRun(t) for t in test_times],
                test_times,
                gen_class_times(test_times),
            ),
        )

        test_times = {"test1": THRESHOLD / 2, "test2": THRESHOLD}
        expected_shards = [
            (600.0, [ShardedTest(test="test2", shard=1, num_shards=1, time=THRESHOLD)]),
            (
                300.0,
                [ShardedTest(test="test1", shard=1, num_shards=1, time=THRESHOLD / 2)],
            ),
        ]
        self.assert_shards_equal(
            expected_shards,
            calculate_shards(
                2,
                [TestRun(t) for t in test_times],
                test_times,
                gen_class_times(test_times),
            ),
        )
```
- **EN**: This chunk continues `test_split_shards` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `test_split_shards`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 431-447
```python
    def test_zero_tests(self) -> None:
        self.assertListEqual([(0.0, []), (0.0, [])], calculate_shards(2, [], {}, None))

    def test_split_shards_random(self) -> None:
        random.seed(120)
        for _ in range(100):
            num_shards = random.randint(1, 10)
            num_tests = random.randint(1, 100)
            test_names = [str(i) for i in range(num_tests)]
            tests = [TestRun(x) for x in test_names]
            serial = [x for x in test_names if random.randint(0, 1) == 0]
            has_times = [x for x in test_names if random.randint(0, 1) == 0]
            random_times: dict[str, float] = {
                i: random.randint(0, THRESHOLD * 10) for i in has_times
            }
            sort_by_time = random.randint(0, 1) == 0
```
- **EN**: This chunk defines `test_split_shards_random`, which narrows a larger candidate set down to the items relevant for the current workflow. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `test_split_shards_random`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 448-460
```python
            shards = calculate_shards(
                num_shards,
                tests,
                random_times,
                None,
                must_serial=lambda x: x in serial,
                sort_by_time=sort_by_time,
            )

            times = [x[0] for x in shards]
            max_diff = max(times) - min(times)
            self.assertTrue(max_diff <= THRESHOLD + (num_tests - len(has_times)) * 60)
```
- **EN**: This chunk continues `test_split_shards_random` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `test_split_shards_random`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 461-484
```python
            all_sharded_tests: dict[str, list[ShardedTest]] = defaultdict(list)
            for _, sharded_tests in shards:
                for sharded_test in sharded_tests:
                    all_sharded_tests[sharded_test.name].append(sharded_test)

            # Check that all test files are represented in the shards
            self.assertListEqual(sorted(test_names), sorted(all_sharded_tests.keys()))
            # Check that for each test file, the pytest shards' times adds up to
            # original and all shards are present
            for test, sharded_tests in all_sharded_tests.items():
                if random_times.get(test) is None:
                    self.assertTrue(len(sharded_tests) == 1)
                    self.assertTrue(sharded_tests[0].time is None)
                else:
                    # x.time is not None because of the above check
                    self.assertAlmostEqual(
                        random_times[test],
                        sum(x.time for x in sharded_tests),  # type: ignore[misc]
                    )
                self.assertListEqual(
                    list(range(sharded_tests[0].num_shards)),
                    sorted(x.shard - 1 for x in sharded_tests),
                )
            # Check that sort_by_time is respected
```
- **EN**: This chunk continues `test_split_shards_random` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `test_split_shards_random`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 485-496
```python
            if sort_by_time:

                def comparator(a: ShardedTest, b: ShardedTest) -> int:
                    # serial comes first
                    if a.name in serial and b.name not in serial:
                        return -1
                    if a.name not in serial and b.name in serial:
                        return 1
                    # known test times come first
                    if a.time is not None and b.time is None:
                        return -1
                    if a.time is None and b.time is not None:
```
- **EN**: This chunk defines `comparator`, which implements a focused step inside the tooling tests pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `comparator`，其作用是实现工具测试流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 497-509
```python
                        return 1
                    if a.time == b.time:
                        return 0
                    # not None due to the above checks
                    return -1 if a.time > b.time else 1  # type: ignore[operator]

            else:

                def comparator(a: ShardedTest, b: ShardedTest) -> int:
                    # serial comes first
                    if a.name in serial and b.name not in serial:
                        return -1
                    if a.name not in serial and b.name in serial:
```
- **EN**: This chunk defines `comparator`, which implements a focused step inside the tooling tests pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `comparator`，其作用是实现工具测试流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 510-521
```python
                        return 1
                    return test_names.index(a.name) - test_names.index(b.name)

            for _, sharded_tests in shards:
                self.assertListEqual(
                    sorted(sharded_tests, key=functools.cmp_to_key(comparator)),
                    sharded_tests,
                )

    def test_calculate_2_shards_against_optimal_shards(self) -> None:
        random.seed(120)
        for _ in range(100):
```
- **EN**: This chunk defines `test_calculate_2_shards_against_optimal_shards`, which narrows a larger candidate set down to the items relevant for the current workflow. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `test_calculate_2_shards_against_optimal_shards`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 522-541
```python
            random_times = {k.test_file: random.random() * 10 for k in self.tests}
            # all test times except first two
            rest_of_tests = [
                i
                for k, i in random_times.items()
                if k != "super_long_test" and k != "long_test1"
            ]
            sum_of_rest = sum(rest_of_tests)
            random_times["super_long_test"] = max(sum_of_rest / 2, *rest_of_tests)
            random_times["long_test1"] = sum_of_rest - random_times["super_long_test"]
            # An optimal sharding would look like the below, but we don't need to compute this for the test:
            # optimal_shards = [
            #     (sum_of_rest, ['super_long_test', 'long_test1']),
            #     (sum_of_rest, [i for i in self.tests if i != 'super_long_test' and i != 'long_test1']),
            # ]
            calculated_shards = calculate_shards(
                2, self.tests, random_times, gen_class_times(random_times)
            )
            max_shard_time = max(calculated_shards[0][0], calculated_shards[1][0])
            if sum_of_rest != 0:
```
- **EN**: This chunk continues `test_calculate_2_shards_against_optimal_shards` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `test_calculate_2_shards_against_optimal_shards`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 542-553
```python
                # The calculated shard should not have a ratio worse than 7/6 for num_shards = 2
                self.assertGreaterEqual(7.0 / 6.0, max_shard_time / sum_of_rest)
                sorted_tests = sorted([t.test_file for t in self.tests])
                sorted_shard_tests = sorted(
                    calculated_shards[0][1] + calculated_shards[1][1]
                )
                # All the tests should be represented by some shard
                self.assertEqual(sorted_tests, [x.name for x in sorted_shard_tests])


if __name__ == "__main__":
    unittest.main()
```
- **EN**: This chunk continues `test_calculate_2_shards_against_optimal_shards` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `test_calculate_2_shards_against_optimal_shards`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Tooling tests**
  - EN: This file belongs to the tooling tests layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于工具测试层，应结合同一子目录中的相邻脚本一起理解。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **REPO_ROOT**
  - EN: `REPO_ROOT` is one of the main local symbols exposed or implemented here.
  - CN: `REPO_ROOT` 是此处暴露或实现的主要局部符号之一。
- **gen_class_times**
  - EN: `gen_class_times` is one of the main local symbols exposed or implemented here.
  - CN: `gen_class_times` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.testing.test_run`, `tools.testing.test_selections`
- **Python standard library / Python 标准库**: `__future__`, `functools`, `random`, `sys`, `unittest`, `collections`, `pathlib`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `gen_class_times`, `TestCalculateShards`
