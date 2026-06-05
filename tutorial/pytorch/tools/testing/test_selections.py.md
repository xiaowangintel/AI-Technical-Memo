# test_selections.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/test_selections.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides shared testing infrastructure, test-selection helpers, or execution utilities for PyTorch CI.
- **Purpose (CN)**: 提供 PyTorch CI 使用的共享测试基础设施、测试选择辅助逻辑或执行工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```python
from __future__ import annotations

import math
import os
import subprocess
from pathlib import Path
from typing import TYPE_CHECKING

from tools.stats.import_test_stats import get_disabled_tests
from tools.testing.test_run import ShardedTest, TestRun
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.import_test_stats, tools.testing.test_run; Python standard-library modules such as __future__, math, os, and 3 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.import_test_stats、tools.testing.test_run；Python 标准库模块，如 __future__、math、os 等共 6 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 12-20
```python

try:
    from torch.testing._internal.common_cuda import SM80OrLater
    from torch.testing._internal.common_utils import TEST_CUDA
except ImportError:
    TEST_CUDA = False
    SM80OrLater = False
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as torch.testing._internal.common_cuda, torch.testing._internal.common_utils. Configuration constants such as TEST_CUDA centralize defaults so later functions share the same policy knobs. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 torch.testing._internal.common_cuda、torch.testing._internal.common_utils。 TEST_CUDA 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 21-29
```python
if TYPE_CHECKING:
    from collections.abc import Callable, Sequence


REPO_ROOT = Path(__file__).resolve().parents[2]

IS_MEM_LEAK_CHECK = os.getenv("PYTORCH_TEST_CUDA_MEM_LEAK_CHECK", "0") == "1"
BUILD_ENVIRONMENT = os.getenv("BUILD_ENVIRONMENT", "")
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as collections.abc. Configuration constants such as REPO_ROOT, IS_MEM_LEAK_CHECK, BUILD_ENVIRONMENT centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 collections.abc。 REPO_ROOT、IS_MEM_LEAK_CHECK、BUILD_ENVIRONMENT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 30-38
```python
# NUM_PROCS_FOR_SHARDING_CALC must remain consistent across all shards of a job
# to ensure that sharding is consistent, NUM_PROCS is the actual number of procs
# used to run tests.  If they are not equal, the only consequence should be
# unequal shards.
IS_ROCM = os.path.exists("/opt/rocm")
NUM_PROCS = 1 if IS_MEM_LEAK_CHECK else 3 if not TEST_CUDA or SM80OrLater else 2
NUM_PROCS_FOR_SHARDING_CALC = NUM_PROCS if not IS_ROCM or IS_MEM_LEAK_CHECK else 2
THRESHOLD = 60 * 10  # 10 minutes
```
- **EN**: Configuration constants such as IS_ROCM, NUM_PROCS, NUM_PROCS_FOR_SHARDING_CALC, and 1 more centralize defaults so later functions share the same policy knobs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: IS_ROCM、NUM_PROCS、NUM_PROCS_FOR_SHARDING_CALC 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 39-50
```python
# See Note [ROCm parallel CI testing]
# Special logic for ROCm GHA runners to query number of GPUs available.
# torch.version.hip was not available to check if this was a ROCm self-hosted runner.
# Must check for ROCm runner in another way. We look for /opt/rocm directory.
if IS_ROCM and not IS_MEM_LEAK_CHECK:
    try:
        # This is the same logic used in GHA health check, see .github/templates/common.yml.j2
        lines = (
            subprocess.check_output(["rocminfo"], encoding="ascii").strip().split("\n")
        )
        count = 0
        for line in lines:
```
- **EN**: Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 51-60
```python
            if " gfx" in line:
                count += 1
        if count == 0:
            raise AssertionError("There must be at least 1 GPU")
        # Limiting to 8 GPUs(PROCS)
        NUM_PROCS = min(count, 8)
    except subprocess.CalledProcessError:
        # The safe default for ROCm GHA runners is to run tests serially.
        NUM_PROCS = 1
```
- **EN**: Configuration constants such as NUM_PROCS, NUM_PROCS centralize defaults so later functions share the same policy knobs. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: NUM_PROCS、NUM_PROCS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 61-70
```python

class ShardJob:
    def __init__(self) -> None:
        self.serial: list[ShardedTest] = []
        self.parallel: list[ShardedTest] = []

    def get_total_time(self) -> float:
        """Default is the value for which to substitute if a test has no time"""
        procs = [0.0 for _ in range(NUM_PROCS_FOR_SHARDING_CALC)]
        for test in self.parallel:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as ShardJob, which package state and behavior for this tooling task. This chunk defines `get_total_time`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 ShardJob 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `get_total_time`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 71-79
```python
            min_index = procs.index(min(procs))
            procs[min_index] += test.get_time()
        time = max(procs) + sum(test.get_time() for test in self.serial)
        return time

    def convert_to_tuple(self) -> tuple[float, list[ShardedTest]]:
        return (self.get_total_time(), self.serial + self.parallel)
```
- **EN**: This chunk defines `convert_to_tuple`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `convert_to_tuple`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 80-89
```python
def get_with_pytest_shard(
    tests: Sequence[TestRun],
    test_file_times: dict[str, float],
    test_class_times: dict[str, dict[str, float]] | None,
) -> list[ShardedTest]:
    sharded_tests: list[ShardedTest] = []

    for test in tests:
        duration = get_duration(test, test_file_times, test_class_times or {})
```
- **EN**: This chunk defines `get_with_pytest_shard`, which narrows a larger candidate set down to the items relevant for the current workflow. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `get_with_pytest_shard`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 90-99
```python
        if duration and duration > THRESHOLD:
            num_shards = math.ceil(duration / THRESHOLD)
            for i in range(num_shards):
                sharded_tests.append(
                    ShardedTest(test, i + 1, num_shards, duration / num_shards)
                )
        else:
            sharded_tests.append(ShardedTest(test, 1, 1, duration))
    return sharded_tests
```
- **EN**: This chunk continues `get_with_pytest_shard` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_with_pytest_shard`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 100-109
```python

def get_duration(
    test: TestRun,
    test_file_times: dict[str, float],
    test_class_times: dict[str, dict[str, float]],
) -> float | None:
    """Calculate the time for a TestRun based on the given test_file_times and
    test_class_times.  Returns None if the time is unknown."""
    file_duration = test_file_times.get(test.test_file, None)
    if test.is_full_file():
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_duration`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_duration`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 110-119
```python
        return file_duration

    def get_duration_for_classes(
        test_file: str, test_classes: frozenset[str]
    ) -> float | None:
        duration: float = 0

        for test_class in test_classes:
            class_duration = test_class_times.get(test_file, {}).get(test_class, None)
            if class_duration is None:
```
- **EN**: This chunk defines `get_duration_for_classes`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_duration_for_classes`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 120-128
```python
                return None
            duration += class_duration
        return duration

    included = test.included()
    excluded = test.excluded()
    included_classes_duration = get_duration_for_classes(test.test_file, included)
    excluded_classes_duration = get_duration_for_classes(test.test_file, excluded)
```
- **EN**: This chunk continues `get_duration_for_classes` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_duration_for_classes`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 129-139
```python
    if included_classes_duration is None or excluded_classes_duration is None:
        # Didn't get the time for all classes, so time is unknown
        return None

    if included:
        return included_classes_duration
    if not excluded:
        raise AssertionError(
            f"TestRun {test} is not full file but doesn't have included or excluded classes"
        )
    if file_duration is None:
```
- **EN**: This chunk continues `get_duration_for_classes` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_duration_for_classes`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 140-151
```python
        return None
    return file_duration - excluded_classes_duration


def shard(
    sharded_jobs: list[ShardJob],
    pytest_sharded_tests: Sequence[ShardedTest],
    estimated_time_limit: float | None = None,
    serial: bool = False,
) -> None:
    # Modifies sharded_jobs in place
    if len(sharded_jobs) == 0:
```
- **EN**: This chunk defines `shard`, which narrows a larger candidate set down to the items relevant for the current workflow. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `shard`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 152-161
```python
        if len(pytest_sharded_tests) != 0:
            raise AssertionError("No shards provided but there are tests to shard")
        return

    round_robin_index = 0

    def _get_min_sharded_job(
        sharded_jobs: list[ShardJob], test: ShardedTest
    ) -> ShardJob:
        if test.time is None:
```
- **EN**: This chunk defines `_get_min_sharded_job`, which narrows a larger candidate set down to the items relevant for the current workflow. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_get_min_sharded_job`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 162-171
```python
            nonlocal round_robin_index
            job = sharded_jobs[round_robin_index % len(sharded_jobs)]
            round_robin_index += 1
            return job
        return min(sharded_jobs, key=lambda j: j.get_total_time())

    def _shard_serial(
        tests: Sequence[ShardedTest], sharded_jobs: list[ShardJob]
    ) -> None:
        if estimated_time_limit is None:
```
- **EN**: This chunk defines `_shard_serial`, which narrows a larger candidate set down to the items relevant for the current workflow. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_shard_serial`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 172-182
```python
            raise AssertionError("Estimated time limit must be provided")
        new_sharded_jobs = sharded_jobs
        for test in tests:
            if (
                len(sharded_jobs) > 1
                and sharded_jobs[-1].get_total_time() > estimated_time_limit
            ):
                new_sharded_jobs = sharded_jobs[:-1]
            min_sharded_job = _get_min_sharded_job(new_sharded_jobs, test)
            min_sharded_job.serial.append(test)
```
- **EN**: This chunk continues `_shard_serial` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `_shard_serial`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 183-194
```python
    def _shard_parallel(
        tests: Sequence[ShardedTest], sharded_jobs: list[ShardJob]
    ) -> None:
        for test in tests:
            min_sharded_job = _get_min_sharded_job(sharded_jobs, test)
            min_sharded_job.parallel.append(test)

    if serial:
        _shard_serial(pytest_sharded_tests, sharded_jobs)
    else:
        _shard_parallel(pytest_sharded_tests, sharded_jobs)
```
- **EN**: This chunk defines `_shard_parallel`, which narrows a larger candidate set down to the items relevant for the current workflow. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `_shard_parallel`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 195-208
```python
    return


def calculate_shards(
    num_shards: int,
    tests: Sequence[TestRun],
    test_file_times: dict[str, float],
    test_class_times: dict[str, dict[str, float]] | None,
    must_serial: Callable[[str], bool] | None = None,
    sort_by_time: bool = True,
) -> list[tuple[float, list[ShardedTest]]]:
    must_serial = must_serial or (lambda x: True)
    test_class_times = test_class_times or {}
```
- **EN**: This chunk defines `calculate_shards`, which narrows a larger candidate set down to the items relevant for the current workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `calculate_shards`，其作用是从更大的候选集中筛选出当前工作流相关的项目。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 209-217
```python
    # Divide tests into pytest shards
    if sort_by_time:
        known_tests = [
            x
            for x in tests
            if get_duration(x, test_file_times, test_class_times) is not None
        ]
        unknown_tests = [x for x in tests if x not in known_tests]
```
- **EN**: This chunk continues `calculate_shards` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `calculate_shards`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 218-228
```python
        pytest_sharded_tests = sorted(
            get_with_pytest_shard(known_tests, test_file_times, test_class_times),
            key=lambda j: j.get_time(),
            reverse=True,
        ) + get_with_pytest_shard(unknown_tests, test_file_times, test_class_times)
    else:
        pytest_sharded_tests = get_with_pytest_shard(
            tests, test_file_times, test_class_times
        )
    del tests
```
- **EN**: This chunk continues `calculate_shards` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `calculate_shards`，进一步展开其内部控制流或数据流转。

### Lines 229-246
```python
    serial_tests = [test for test in pytest_sharded_tests if must_serial(test.name)]
    parallel_tests = [test for test in pytest_sharded_tests if test not in serial_tests]

    serial_time = sum(test.get_time() for test in serial_tests)
    parallel_time = sum(test.get_time() for test in parallel_tests)
    total_time = serial_time + parallel_time / NUM_PROCS_FOR_SHARDING_CALC
    estimated_time_per_shard = total_time / num_shards
    # Separate serial tests from parallel tests as much as possible to maximize
    # parallelism by putting all the serial tests on the first num_serial_shards
    # shards. The estimated_time_limit is the estimated time it should take for
    # the least filled serial shard. Ex if we have 8 min of serial tests, 20 min
    # of parallel tests, 6 shards, and 2 procs per machine, we would expect each
    # machine to take 3 min and should aim for 3 serial shards, with shards 1
    # and 2 taking 3 min and shard 3 taking 2 min.  The estimated time limit
    # would be 2 min. This ensures that the first few shard contains as many
    # serial tests as possible and as few parallel tests as possible. The least
    # filled/last (in the example, the 3rd) shard may contain a lot of both
    # serial and parallel tests.
```
- **EN**: This chunk continues `calculate_shards` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `calculate_shards`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 247-256
```python
    estimated_time_limit = 0.0
    if estimated_time_per_shard != 0:
        estimated_time_limit = serial_time % estimated_time_per_shard
    if estimated_time_limit <= 0.01:
        estimated_time_limit = estimated_time_per_shard
    if total_time == 0:
        num_serial_shards = num_shards
    else:
        num_serial_shards = max(math.ceil(serial_time / total_time * num_shards), 1)
```
- **EN**: This chunk continues `calculate_shards` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `calculate_shards`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 257-269
```python
    sharded_jobs = [ShardJob() for _ in range(num_shards)]
    shard(
        sharded_jobs=sharded_jobs[:num_serial_shards],
        pytest_sharded_tests=serial_tests,
        estimated_time_limit=estimated_time_limit,
        serial=True,
    )
    shard(
        sharded_jobs=sharded_jobs,
        pytest_sharded_tests=parallel_tests,
        serial=False,
    )
```
- **EN**: This chunk continues `calculate_shards` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `calculate_shards`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 270-274
```python
    return [job.convert_to_tuple() for job in sharded_jobs]


def get_test_case_configs(dirpath: str) -> None:
    get_disabled_tests(dirpath=dirpath)
```
- **EN**: This chunk defines `get_test_case_configs`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_test_case_configs`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **Template expansion**
  - EN: The implementation relies on placeholders or structured text expansion to generate source artifacts.
  - CN: 该实现依赖占位符或结构化文本展开来生成源码产物。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **Pytest integration**
  - EN: The file plugs into pytest collection, execution, or reporting behavior.
  - CN: 该文件接入 pytest 的收集、执行或报告行为。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.import_test_stats`, `tools.testing.test_run`, `torch.testing._internal.common_cuda`, `torch.testing._internal.common_utils`
- **Python standard library / Python 标准库**: `__future__`, `math`, `os`, `subprocess`, `pathlib`, `typing`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `IS_MEM_LEAK_CHECK`, `BUILD_ENVIRONMENT`, `IS_ROCM`, `NUM_PROCS`, `NUM_PROCS_FOR_SHARDING_CALC`, `THRESHOLD`, `ShardJob`, `get_with_pytest_shard`, `get_duration`, `shard`, `calculate_shards`
