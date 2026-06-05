# monitor.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/monitor.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
#!/usr/bin/env python3
"""
A Python script that logging the system-level utilization usage in json format.
Data collected: CPU, memory, GPU memory utilization, and GPU utilization if available.

Usage:
- To run the script with default data collect time setting, use the following command:
    python3 monitor.py

- To run the script in the local machine with debug mode and customized data collect time, use the following command:
    python3 monitor.py --debug --log-interval 10 --data-collect-interval 2
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 13-24
```python
- To log the data to a file, use the following command:
    python3 monitor.py > usage_log.txt 2>&1

- To gracefully exit the script in the local machine, press ctrl+c, or kill the process using:
    kill <pid>
"""

from __future__ import annotations

import os
import sys
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, os, sys. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、os、sys。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。

### Lines 25-37
```python

# adding sys.path makes the monitor script able to import path tools.stats.utilization_stats_lib
sys.path.insert(0, os.path.join(os.path.dirname(__file__), "..", ".."))
import argparse
import copy
import dataclasses
import os
import signal
import threading
import time
from collections import defaultdict
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, copy, dataclasses, and 6 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、copy、dataclasses 等共 9 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 38-49
```python
import psutil  # type: ignore[import]

from tools.stats.utilization_stats_lib import (
    getDataModelVersion,
    getTsNow,
    GpuUsage,
    RecordData,
    UtilizationMetadata,
    UtilizationRecord,
    UtilizationStats,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.utilization_stats_lib; external packages such as psutil  # type: ignore[import].
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.utilization_stats_lib；外部依赖包，如 psutil  # type: ignore[import]。

### Lines 50-61
```python

_HAS_PYNVML = False
_HAS_AMDSMI = False

_job_name = os.environ.get("JOB_NAME", "")
_job_id = os.environ.get("JOB_ID", "")
_workflow_run_id = os.environ.get("WORKFLOW_RUN_ID", "")
_workflow_name = os.environ.get("WORKFLOW_NAME", "")


@dataclasses.dataclass
class UsageData:
```
- **EN**: It introduces classes such as UsageData, which package state and behavior for this tooling task. This chunk continues `UsageData` and expands its internal control flow or data movement.
- **CN**: 它引入了 UsageData 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `UsageData`，进一步展开其内部控制流或数据流转。

### Lines 62-73
```python
    """
    Dataclass for storing usage data. This is the data that will be logged to the usage_log file.
    """

    cpu_percent: float
    memory_percent: float
    processes: list[dict[str, Any]]
    gpu_list: list[GpuData]


@dataclasses.dataclass
class GpuData:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as GpuData, which package state and behavior for this tooling task. This chunk continues `GpuData` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 GpuData 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `GpuData`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 74-85
```python
    """
    Dataclass for storing gpu data. This is the data that will be logged to the usage_log file.
    """

    uuid: str
    utilization: float
    mem_utilization: float
    allocated_mem: float
    allocated_mem_value: float
    total_mem_value: float
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `GpuData` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `GpuData`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 86-99
```python
try:
    import pynvml  # type: ignore[import]

    _HAS_PYNVML = True
except ModuleNotFoundError:
    pass

try:
    import amdsmi  # type: ignore[import]

    _HAS_AMDSMI = True
except ModuleNotFoundError:
    pass
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as pynvml  # type: ignore[import], amdsmi  # type: ignore[import]. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 pynvml  # type: ignore[import]、amdsmi  # type: ignore[import]。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 100-113
```python

def parse_args() -> argparse.Namespace:
    """
    Parse command line arguments.

    Returns:
        argparse.Namespace: Parsed arguments.
    """
    parser = argparse.ArgumentParser(description=" System-level Usage Logger ")

    # debug mode used in local to gracefully exit the script when ctrl+c is
    # pressed,and print out the json output in a pretty format.
    parser.add_argument("--debug", action="store_true", help="Enable debug mode")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `parse_args`, which parses or loads structured input into tool-friendly data structures. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `parse_args`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 114-128
```python
    parser.add_argument(
        "--log-interval",
        type=float,
        default=5,
        help="set time interval for logging utilization data, default is 5 seconds",
    )
    parser.add_argument(
        "--data-collect-interval",
        type=float,
        default=1,
        help="set time interval to collect data, default is 1 second, this should not longer than log_interval",
    )
    args = parser.parse_args()
    return args
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 129-143
```python

class SharedResource:
    """
    thread-safe utils for shared resources used in both worker processor
    and main processor during UsageLogger.
    It collects the usage data or errors from the worker processor, and
    output the aggregated data or errors to the main processor for logging.
    """

    def __init__(self, is_debug_mode: bool = False) -> None:
        self._data_list: list[UsageData] = []
        self._data_errors: list[str] = []
        self._data_logs: list[str] = []
        self._lock = threading.Lock()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as SharedResource, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 SharedResource 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 144-155
```python
    def get_and_reset(self) -> tuple[list[UsageData], list[str], list[str]]:
        """
        get deepcopy of list of usageData and list of string errors
        """
        copy_data = []
        copy_errors = []
        copy_logs = []
        with self._lock:
            copy_data = copy.deepcopy(self._data_list)
            copy_errors = copy.deepcopy(self._data_errors)
            copy_logs = copy.deepcopy(self._data_logs)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get_and_reset`, which implements a focused step inside the ci statistics pipeline.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get_and_reset`，其作用是实现CI 统计流水线中的一个关键步骤。

### Lines 156-168
```python
            self._data_list.clear()
            self._data_errors.clear()
            self._data_logs.clear()
        return copy_data, copy_errors, copy_logs

    def add_data(self, data: UsageData) -> None:
        with self._lock:
            self._data_list.append(data)

    def add_error(self, error: Exception) -> None:
        with self._lock:
            self._data_errors.append(str(error))
```
- **EN**: This chunk defines `add_error`, which implements a focused step inside the ci statistics pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `add_error`，其作用是实现CI 统计流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 169-182
```python
    def add_log(self, log: str) -> None:
        with self._lock:
            print("here log")
            self._data_logs.append(log)


class UsageLogger:
    """
    Collect and display usage data, including:
    CPU, memory, GPU memory utilization, and GPU utilization.
    By default, data is collected every 1 seconds, and log
    the aggregated result every 5 seconds.
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as UsageLogger, which package state and behavior for this tooling task. This chunk defines `add_log`, which implements a focused step inside the ci statistics pipeline.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 UsageLogger 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `add_log`，其作用是实现CI 统计流水线中的一个关键步骤。

### Lines 183-206
```python
    def __init__(
        self,
        log_interval: float = 5,
        data_collect_interval: float = 1,
        is_debug_mode: bool = False,
        pynvml_enabled: bool = False,
        amdsmi_enabled: bool = False,
    ) -> None:
        """
        log_interval: Time interval in seconds for collecting usage data; default is 5 seconds.
        is_debug_mode:
            Useful if you're testing on a local machine and want to see the output
            in a pretty format with more information.
        """
        self._log_interval = log_interval
        self._data_collect_interval = data_collect_interval
        self._metadata = UtilizationMetadata(
            level="metadata",
            usage_collect_interval=self._data_collect_interval,
            data_model_version=getDataModelVersion(),
            job_id=_job_id,
            job_name=_job_name,
            workflow_id=_workflow_run_id,
            workflow_name=_workflow_name,
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `__init__`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `__init__`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 207-220
```python
            start_at=getTsNow(),
        )

        self._has_pynvml = pynvml_enabled
        self._has_amdsmi = amdsmi_enabled
        self._gpu_handles: list[Any] = []
        self._gpu_lib_detected: str = ""
        self._num_of_cpus = 0
        self._debug_mode = is_debug_mode
        self._initial_gpu_handler()

        self.shared_resource = SharedResource()
        self.exit_event = threading.Event()
```
- **EN**: This chunk continues `__init__` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `__init__`，进一步展开其内部控制流或数据流转。

### Lines 221-232
```python
    def _collect_data(self) -> None:
        """
        Collects the data every data_collect_interval (in seconds).
        """
        while not self.exit_event.is_set():
            try:
                # collect cpu, memory and gpu metrics
                memory = psutil.virtual_memory().percent
                cpu_percent = psutil.cpu_percent()
                processes = self._get_process_info()
                gpu_list = self._collect_gpu_data()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_collect_data`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_collect_data`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 233-245
```python
                data = UsageData(
                    cpu_percent=cpu_percent,
                    memory_percent=memory,
                    processes=processes,
                    gpu_list=gpu_list,
                )
                if self._debug_mode:
                    print(f"collecting data {data}")

                self.shared_resource.add_data(data)

            except Exception as e:
                if self._debug_mode:
```
- **EN**: This chunk continues `_collect_data` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `_collect_data`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 246-257
```python
                    print(f"error detected: {str(e)}")
                self.shared_resource.add_error(e)
            finally:
                time.sleep(self._data_collect_interval)

    def _generate_stats(self, data_list: list[float]) -> UtilizationStats:
        """
        Generate stats from the data list.
        """
        if len(data_list) == 0:
            return UtilizationStats()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_generate_stats`, which generates derived source text, templates, or metadata outputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_generate_stats`，其作用是生成派生源码文本、模板或元数据输出。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 258-274
```python
        total = sum(data_list)
        avg = total / len(data_list)
        maxi = max(data_list)

        return UtilizationStats(
            avg=round(avg, 2),
            max=round(maxi, 2),
            raw=data_list,
        )

    def _output_data(self) -> None:
        """
        output the data.
        """
        self._metadata.start_at = getTsNow()
        self.log_json(self._metadata.to_json())
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_output_data`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_output_data`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 275-290
```python
        while not self.exit_event.is_set():
            collecting_start_time = time.time()
            stats = UtilizationRecord(
                level="record",
                timestamp=getTsNow(),
            )

            try:
                data_list, error_list, log_list = self.shared_resource.get_and_reset()
                if self._debug_mode:
                    print(
                        f"collected data: {len(data_list)}, errors found: {len(error_list)}, logs {len(log_list)}"
                    )
                # records and clears found errors
                errors = list(set(error_list))
```
- **EN**: This chunk continues `_output_data` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `_output_data`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 291-306
```python
                # if has errors but data list is None, a bug may exist in the monitor code, log the errors
                if not data_list and len(errors) > 0:
                    raise ValueError(
                        f"no data is collected but detected errors during the interval: {errors}, logs: {log_list}"
                    )
                if not data_list:
                    # pass since no data is collected
                    continue

                cpu_stats = self._generate_stats(
                    [data.cpu_percent for data in data_list]
                )
                memory_stats = self._generate_stats(
                    [data.memory_percent for data in data_list]
                )
```
- **EN**: This chunk continues `_output_data` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `_output_data`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 307-318
```python
                # find all cmds during the interval
                cmds = {
                    process["cmd"] for data in data_list for process in data.processes
                }

                stats.cmd_names = list(cmds)
                record = RecordData()
                record.cpu = cpu_stats
                record.memory = memory_stats

                # collect gpu metrics
                if self._has_pynvml or self._has_amdsmi:
```
- **EN**: This chunk continues `_output_data` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_output_data`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 319-331
```python
                    gpu_list = self._calculate_gpu_utilization(data_list)
                    record.gpu_usage = gpu_list
                stats.data = record
                stats.logs = log_list
            except Exception as e:
                stats = UtilizationRecord(
                    level="record", timestamp=getTsNow(), error=str(e)
                )
            finally:
                collecting_end_time = time.time()
                time_diff = collecting_end_time - collecting_start_time
                # verify there is data
                if stats.level:
```
- **EN**: This chunk continues `_output_data` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `_output_data`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 332-348
```python
                    stats.log_duration = f"{time_diff * 1000:.2f} ms"
                    self.log_json(stats.to_json())
                time.sleep(self._log_interval)
        # shut down gpu connections when exiting
        self._shutdown_gpu_connections()

    def _calculate_gpu_utilization(self, data_list: list[UsageData]) -> list[GpuUsage]:
        """
        Calculates the GPU utilization.
        """
        calculate_gpu = []
        gpu_mem_utilization = defaultdict(list)
        gpu_utilization = defaultdict(list)
        gpu_allocated_mem = defaultdict(list)
        gpu_allocated_mem_values = defaultdict(list)
        gpu_total_mem_values = defaultdict(float)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_calculate_gpu_utilization`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_calculate_gpu_utilization`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 349-372
```python
        for data in data_list:
            for gpu in data.gpu_list:
                gpu_mem_utilization[gpu.uuid].append(gpu.mem_utilization)
                gpu_utilization[gpu.uuid].append(gpu.utilization)
                gpu_allocated_mem[gpu.uuid].append(gpu.allocated_mem)
                gpu_allocated_mem_values[gpu.uuid].append(gpu.allocated_mem_value)
                gpu_total_mem_values[gpu.uuid] = gpu.total_mem_value

        for gpu_uuid in gpu_utilization:
            gpu_util_stats = self._generate_stats(gpu_utilization[gpu_uuid])
            gpu_mem_util_stats = self._generate_stats(gpu_mem_utilization[gpu_uuid])
            gpu_allocated_mem_stats = self._generate_stats(gpu_allocated_mem[gpu_uuid])
            gpu_allocated_mem_value_stats = self._generate_stats(
                gpu_allocated_mem_values[gpu_uuid]
            )
            calculate_gpu.append(
                GpuUsage(
                    uuid=gpu_uuid,
                    util_percent=gpu_util_stats,
                    mem_util_percent=gpu_mem_util_stats,
                    allocated_mem_percent=gpu_allocated_mem_stats,
                    allocated_mem_value=gpu_allocated_mem_value_stats,
                    total_mem_value=gpu_total_mem_values[gpu_uuid],
                )
```
- **EN**: This chunk continues `_calculate_gpu_utilization` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `_calculate_gpu_utilization`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 373-387
```python
            )
        return calculate_gpu

    def start(self) -> None:
        collect_thread = threading.Thread(target=self._collect_data)
        collect_thread.start()
        self._output_data()
        collect_thread.join()

    def stop(self, *args: Any) -> None:
        """
        Exits the program gracefully. this shuts down the logging loop.
        """
        self.exit_event.set()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `stop`, which implements a focused step inside the ci statistics pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `stop`，其作用是实现CI 统计流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 388-404
```python
    def log_json(self, stats: Any) -> None:
        """
        Logs the stats in json format to stdout.
        """
        print(stats)

    def _collect_gpu_data(self) -> list[GpuData]:
        gpu_data_list = []
        if self._has_pynvml:
            # Iterate over the available GPUs
            for gpu_handle in self._gpu_handles:
                # see https://docs.nvidia.com/deploy/nvml-api/group__nvmlDeviceQueries.html
                gpu_utilization = pynvml.nvmlDeviceGetUtilizationRates(gpu_handle)
                gpu_uuid = pynvml.nvmlDeviceGetUUID(gpu_handle)
                gpu_memory_info = pynvml.nvmlDeviceGetMemoryInfo(gpu_handle)
                mem_utilization = gpu_utilization.memory
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_collect_gpu_data`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_collect_gpu_data`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 405-421
```python
                allocate_mem_MB = gpu_memory_info.used / 1024**2
                total_mem_MB = gpu_memory_info.total / 1024**2
                allocate_mem_percent = allocate_mem_MB / total_mem_MB * 100

                gpu_data_list.append(
                    GpuData(
                        uuid=gpu_uuid,
                        utilization=gpu_utilization.gpu,
                        mem_utilization=mem_utilization,
                        allocated_mem=allocate_mem_percent,
                        allocated_mem_value=allocate_mem_MB,
                        total_mem_value=total_mem_MB,
                    )
                )
        elif self._has_amdsmi:
            # Iterate over the available GPUs
            for handle in self._gpu_handles:
```
- **EN**: This chunk continues `_collect_gpu_data` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_collect_gpu_data`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 422-444
```python
                # see https://rocm.docs.amd.com/projects/amdsmi/en/latest/how-to/amdsmi-py-lib.html
                engine_usage = amdsmi.amdsmi_get_gpu_activity(handle)
                gpu_uuid = amdsmi.amdsmi_get_gpu_device_uuid(handle)
                gpu_utilization = engine_usage["gfx_activity"]
                gpu_mem_utilization = gpu_utilization["umc_activity"]
                mem_info = amdsmi.amdsmi_get_gpu_memory_usage(handle)

                allocate_mem_MB = mem_info["vram_usage"] / 1024**2
                total_mem_MB = mem_info["vram_total"] / 1024**2
                allocate_mem_percent = allocate_mem_MB / total_mem_MB * 100

                gpu_data_list.append(
                    GpuData(
                        uuid=gpu_uuid,
                        utilization=gpu_utilization,
                        mem_utilization=gpu_mem_utilization,
                        allocated_mem=allocate_mem_percent,
                        allocated_mem_value=allocate_mem_MB,
                        total_mem_value=total_mem_MB,
                    )
                )
        return gpu_data_list
```
- **EN**: This chunk continues `_collect_gpu_data` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_collect_gpu_data`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 445-456
```python
    def _initial_gpu_handler(self) -> None:
        """
        Initializes the GPU handlers if gpus are available, and updates the log summary info.
        """
        try:
            if self._has_pynvml:
                self._gpu_lib_detected = "pynvml"
                # Todo: investigate if we can use device uuid instead of index.
                # there is chance that the gpu index can change when the gpu is rebooted.
                self._gpu_handles = [
                    pynvml.nvmlDeviceGetHandleByIndex(i)
                    for i in range(pynvml.nvmlDeviceGetCount())
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_initial_gpu_handler`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_initial_gpu_handler`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 457-468
```python
                ]
            if self._has_amdsmi:
                self._gpu_lib_detected = "amdsmi"
                self._gpu_handles = amdsmi.amdsmi_get_processor_handles()

            self._num_of_cpus = psutil.cpu_count(logical=True)
            # update summary info
            self._metadata.gpu_count = len(self._gpu_handles)
            self._metadata.cpu_count = self._num_of_cpus

            if self._has_pynvml or self._has_amdsmi:
                if len(self._gpu_handles) == 0:
```
- **EN**: This chunk continues `_initial_gpu_handler` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_initial_gpu_handler`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 469-481
```python
                    self._metadata.gpu_type = ""
                else:
                    self._metadata.gpu_type = self._gpu_lib_detected
        except Exception as e:
            self._metadata.error = str(e)

    def _shutdown_gpu_connections(self) -> None:
        if self._has_amdsmi:
            try:
                amdsmi.amdsmi_shut_down()
            except amdsmi.AmdSmiException:
                pass
        if self._has_pynvml:
```
- **EN**: This chunk defines `_shutdown_gpu_connections`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `_shutdown_gpu_connections`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 482-495
```python
            try:
                pynvml.nvmlShutdown()
            except pynvml.NVMLError:
                pass

    def _pynvml_get_per_process_gpu_info(self, handle: Any) -> list[dict[str, Any]]:
        processes = pynvml.nvmlDeviceGetComputeRunningProcesses(handle)
        per_process_info = []

        for p in processes:
            mem = p.usedGpuMemory / (1024 * 1024)
            pid = p.pid
            info = {"pid": pid, "gpu_memory": mem}
            try:
```
- **EN**: This chunk defines `_pynvml_get_per_process_gpu_info`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `_pynvml_get_per_process_gpu_info`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 496-508
```python
                proc = psutil.Process(pid)
                cmdline = proc.cmdline()
                info.update({"cmd": " ".join(cmdline)})
            except Exception:
                pass
            finally:
                per_process_info.append(info)
        return per_process_info

    def _rocm_get_per_process_gpu_info(self, handle: Any) -> list[dict[str, Any]]:
        processes = amdsmi.amdsmi_get_gpu_process_list(handle)
        per_process_info = []
        for p in processes:
```
- **EN**: This chunk defines `_rocm_get_per_process_gpu_info`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_rocm_get_per_process_gpu_info`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 509-520
```python
            try:
                proc_info = amdsmi.amdsmi_get_gpu_process_info(handle, p)
            except AttributeError:
                # https://github.com/ROCm/amdsmi/commit/c551c3caedbd903ba828e7fdffa5b56d475a15e7
                # BC-breaking change that removes amdsmi_get_gpu_process_info API from amdsmi
                proc_info = p

            info = {
                "pid": proc_info["pid"],
                "gpu_memory": proc_info["memory_usage"]["vram_mem"] / (1024 * 1024),
            }
            try:
```
- **EN**: This chunk continues `_rocm_get_per_process_gpu_info` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `_rocm_get_per_process_gpu_info`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 521-533
```python
                proc = psutil.Process(proc_info["pid"])
                cmdline = proc.cmdline()
                info.update({"cmd": " ".join(cmdline)})
            except Exception:
                pass
            finally:
                per_process_info.append(info)
        return per_process_info

    def _get_process_info(self) -> list[dict[str, Any]]:
        def get_processes_running_python_tests() -> list[Any]:
            python_test_processes = []
            for process in psutil.process_iter():
```
- **EN**: This chunk defines `get_processes_running_python_tests`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_processes_running_python_tests`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 534-545
```python
                try:
                    cmd = " ".join(process.cmdline())
                    processName = process.name()
                    pid = process.pid
                    is_python = "python" in processName and "python" in cmd
                    is_pytest = "pytest" in cmd
                    if is_python or is_pytest:
                        python_test_processes.append({"pid": pid, "cmd": cmd})
                except Exception:
                    pass
            return python_test_processes
```
- **EN**: This chunk continues `get_processes_running_python_tests` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_processes_running_python_tests`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 546-558
```python
        processes = get_processes_running_python_tests()
        return processes


def main() -> None:
    """
    Main function of the program.
    """

    # initialize gpu management libraries
    pynvml_enabled = False
    amdsmi_enabled = False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 559-572
```python
    if _HAS_PYNVML:
        try:
            pynvml.nvmlInit()
            pynvml_enabled = True
        except pynvml.NVMLError:
            pass
    if _HAS_AMDSMI:
        try:
            amdsmi.amdsmi_init()
            amdsmi_enabled = True
        except amdsmi.AmdSmiException:
            pass
    args = parse_args()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 573-585
```python
    usagelogger = UsageLogger(
        log_interval=args.log_interval,
        data_collect_interval=args.data_collect_interval,
        is_debug_mode=args.debug,
        pynvml_enabled=pynvml_enabled,
        amdsmi_enabled=amdsmi_enabled,
    )

    # gracefully exit the script when pid is killed
    signal.signal(signal.SIGTERM, usagelogger.stop)
    # gracefully exit the script when keyboard ctrl+c is pressed.
    signal.signal(signal.SIGINT, usagelogger.stop)
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。

### Lines 586-591
```python
    # start the logging
    usagelogger.start()


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **CI statistics**
  - EN: This file belongs to the ci statistics layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于CI 统计层，应结合同一子目录中的相邻脚本一起理解。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **_HAS_PYNVML**
  - EN: `_HAS_PYNVML` is one of the main local symbols exposed or implemented here.
  - CN: `_HAS_PYNVML` 是此处暴露或实现的主要局部符号之一。
- **_HAS_AMDSMI**
  - EN: `_HAS_AMDSMI` is one of the main local symbols exposed or implemented here.
  - CN: `_HAS_AMDSMI` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.utilization_stats_lib`
- **Python standard library / Python 标准库**: `__future__`, `os`, `sys`, `argparse`, `copy`, `dataclasses`, `signal`, `threading`, `time`, `collections`, `typing`
- **External packages / 外部依赖包**: `psutil  # type: ignore[import]`, `pynvml  # type: ignore[import]`, `amdsmi  # type: ignore[import]`
- **Primary symbols in this file / 本文件核心符号**: `_HAS_PYNVML`, `_HAS_AMDSMI`, `UsageData`, `GpuData`, `parse_args`, `SharedResource`, `UsageLogger`, `main`
