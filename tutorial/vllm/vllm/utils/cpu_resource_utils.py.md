# cpu_resource_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/cpu_resource_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-14)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json
import os
import subprocess
import sys
from dataclasses import dataclass
from functools import cache

import psutil
import regex as re

DEVICE_CONTROL_ENV_VAR = "CPU_VISIBLE_MEMORY_NODES"
```
**EN:** Sets up the module with standard-library support such as `json`, `os`, `subprocess`, external packages such as `psutil`, `regex`. It prepares the symbols later used by `LogicalCPUInfo`, `MemoryNodeInfo`, `get_memory_affinity`, `parse_id_list`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `LogicalCPUInfo`, `MemoryNodeInfo`, `get_memory_affinity`, `parse_id_list` 提供上下文。

### LogicalCPUInfo (lines 18-44)
```python
class LogicalCPUInfo:
    id: int = -1
    physical_core: int = -1
    numa_node: int = -1

    @classmethod
    def _int(cls, value: str) -> int:
        try:
            int_value = int(value)
        except Exception:
            int_value = -1
        return int_value

    @staticmethod
    def json_decoder(obj_dict: dict):
        id = obj_dict.get("cpu")
        physical_core = obj_dict.get("core")
        numa_node = obj_dict.get("node")

        if not (id is None or physical_core is None or numa_node is None):
            return LogicalCPUInfo(
                id=LogicalCPUInfo._int(id),
                physical_core=LogicalCPUInfo._int(physical_core),
                numa_node=LogicalCPUInfo._int(numa_node),
            )
        else:
            return obj_dict
```
**EN:** Defines the `LogicalCPUInfo` class used by this module. Key methods include `json_decoder`.
**CN:** `LogicalCPUInfo` 是该文件中的核心类，用于封装与 `LogicalCPUInfo` 相关的状态和行为。 关键方法包括 `json_decoder`。

### MemoryNodeInfo (lines 48-50)
```python
class MemoryNodeInfo:
    total_memory: int = -1
    available_memory: int = -1
```
**EN:** Defines the `MemoryNodeInfo` class used by this module.
**CN:** `MemoryNodeInfo` 是该文件中的核心类，用于封装与 `MemoryNodeInfo` 相关的状态和行为。

### get_memory_affinity (lines 53-62)
```python
def get_memory_affinity(pid: int = 0) -> list[int]:
    pid = os.getpid() if pid == 0 else pid
    path = f"/proc/{pid}/status"
    with open(path) as f:
        for line in f:
            if line.startswith("Mems_allowed_list:"):
                # Extract the string part (e.g., "0-1,3")
                raw_list = line.split(":")[1].strip()
                return parse_id_list(raw_list)
    return []
```
**EN:** `get_memory_affinity` retrieves data or state needed by the pipeline. It mainly works with `pid`. Inside the body, it relies on `os.getpid`, `open`, `line.startswith` to complete the main steps.
**CN:** `get_memory_affinity` 负责获取流水线所需的数据或状态。 它主要处理 `pid` 等参数。 实现过程中会调用 `os.getpid`, `open`, `line.startswith` 等函数完成关键步骤。

### parse_id_list (lines 65-77)
```python
def parse_id_list(raw_str: str) -> list[int]:
    """Parses strings like '0-2,4,7-8' into [0, 1, 2, 4, 7, 8]"""
    result: list[int] = []
    if not raw_str:
        return result

    for part in raw_str.split(","):
        if "-" in part:
            start, end = map(int, part.split("-"))
            result.extend(range(start, end + 1))
        else:
            result.append(int(part))
    return sorted(list(set(result)))
```
**EN:** `parse_id_list`: Parses strings like '0-2,4,7-8' into [0, 1, 2, 4, 7, 8]. It mainly works with `raw_str`. Inside the body, it relies on `raw_str.split`, `sorted`, `map` to complete the main steps.
**CN:** `parse_id_list` 负责把原始输入解析为结构化对象。 它主要处理 `raw_str` 等参数。 实现过程中会调用 `raw_str.split`, `sorted`, `map` 等函数完成关键步骤。

### get_memory_node_info (lines 80-120)
```python
def get_memory_node_info(node_id: int = 0) -> MemoryNodeInfo:
    if sys.platform == "darwin":
        # MacOS has no memory node
        return MemoryNodeInfo(
            total_memory=psutil.virtual_memory().total,
            available_memory=psutil.virtual_memory().available,
        )

    meminfo_path = f"/sys/devices/system/node/node{node_id}/meminfo"
    if not os.path.exists(meminfo_path):
        # Non-NUMA systems (e.g. many RISC-V boards) don't expose per-node
        # meminfo. Fall back to system-wide numbers from psutil.
        vm = psutil.virtual_memory()
        return MemoryNodeInfo(
            total_memory=vm.total,
            available_memory=vm.available,
        )

    meminfo = {}
    with open(meminfo_path) as f:
        for line in f:
            # Each line looks like: "Node 0 MemTotal: 97421888 kB"
            parts = line.split()
            key = parts[2].rstrip(":")
    # ...
    )

    return MemoryNodeInfo(
        total_memory=total_memory,
        available_memory=available_memory,
    )
```
**EN:** `get_memory_node_info` retrieves data or state needed by the pipeline. It mainly works with `node_id`. Inside the body, it relies on `MemoryNodeInfo`, `os.path.exists`, `psutil.virtual_memory` to complete the main steps.
**CN:** `get_memory_node_info` 负责获取流水线所需的数据或状态。 它主要处理 `node_id` 等参数。 实现过程中会调用 `MemoryNodeInfo`, `os.path.exists`, `psutil.virtual_memory` 等函数完成关键步骤。

### get_allowed_cpu_list (lines 123-128)
```python
def get_allowed_cpu_list() -> list[LogicalCPUInfo]:
    cpu_list = _get_cpu_list()
    if sys.platform == "linux":
        allowed = os.sched_getaffinity(0)
        return [x for x in cpu_list if x.id in allowed]
    return cpu_list
```
**EN:** `get_allowed_cpu_list` retrieves data or state needed by the pipeline. Inside the body, it relies on `_get_cpu_list`, `os.sched_getaffinity` to complete the main steps.
**CN:** `get_allowed_cpu_list` 负责获取流水线所需的数据或状态。 实现过程中会调用 `_get_cpu_list`, `os.sched_getaffinity` 等函数完成关键步骤。

### get_visible_memory_node (lines 131-149)
```python
def get_visible_memory_node() -> list[int]:
    if sys.platform == "darwin":
        return [0]

    allowed_memory_node_list = get_memory_affinity()

    env_key = DEVICE_CONTROL_ENV_VAR
    if (
        ("VLLM_CPU_SIM_MULTI_NUMA" not in os.environ)
        and env_key in os.environ
        and os.environ[env_key] != ""
    ):
        visible_nodes = [int(s) for s in os.environ[env_key].split(",")]
        visible_nodes = [
            node for node in visible_nodes if node in allowed_memory_node_list
        ]
        return visible_nodes

    return allowed_memory_node_list
```
**EN:** `get_visible_memory_node` retrieves data or state needed by the pipeline. Inside the body, it relies on `get_memory_affinity`, `os.environ.split` to complete the main steps.
**CN:** `get_visible_memory_node` 负责获取流水线所需的数据或状态。 实现过程中会调用 `get_memory_affinity`, `os.environ.split` 等函数完成关键步骤。

### _synthesize_cpu_list (lines 153-159)
```python
def _synthesize_cpu_list() -> list[LogicalCPUInfo]:
    """Synthesize a flat CPU list: each logical CPU is its own core on
    NUMA node 0.  Used when lscpu output is unavailable or unparsable
    (e.g. macOS, RISC-V)."""
    cpu_count = os.cpu_count()
    assert cpu_count
    return [LogicalCPUInfo(i, i, 0) for i in range(cpu_count)]
```
**EN:** `_synthesize_cpu_list`: Synthesize a flat CPU list: each logical CPU is its own core on NUMA node 0. Inside the body, it relies on `os.cpu_count`, `LogicalCPUInfo` to complete the main steps.
**CN:** `_synthesize_cpu_list` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `os.cpu_count`, `LogicalCPUInfo` 等函数完成关键步骤。

### _get_cpu_list (lines 162-198)
```python
def _get_cpu_list() -> list[LogicalCPUInfo]:
    if sys.platform == "darwin":
        # For MacOS, no user-level CPU affinity and SMT, return all CPUs
        return _synthesize_cpu_list()

    lscpu_output = subprocess.check_output(
        "lscpu --json --extended=CPU,CORE,NODE --online", shell=True, text=True
    )

    # For platforms without NUMA, map bare `-` node to 0 so non-NUMA
    # systems keep the existing behavior from #39781.
    lscpu_output = re.sub(r'"node":\s*-\s*(,|\n|\})', r'"node": 0\1', lscpu_output)

    # On some architectures (notably RISC-V), lscpu also emits bare `-`
    # for cpu/core.  Quote them so the JSON parses; they will decode to
    # -1 and be filtered out below, triggering the synthesized fallback.
    lscpu_output = re.sub(
        r'("(?:cpu|core)":\s*)-\s*(,|\n|\})',
        r'\1"-"\2',
        lscpu_output,
    )

    logical_cpu_list: list[LogicalCPUInfo] = json.loads(
        lscpu_output, object_hook=LogicalCPUInfo.json_decoder
    # ...
    # If lscpu returned no valid entries (e.g. RISC-V where all fields
    # are bare `-`), fall back to synthesized topology.
    if not logical_cpu_list:
        logical_cpu_list = _synthesize_cpu_list()

    return logical_cpu_list
```
**EN:** `_get_cpu_list` retrieves data or state needed by the pipeline. Inside the body, it relies on `subprocess.check_output`, `re.sub`, `_synthesize_cpu_list` to complete the main steps.
**CN:** `_get_cpu_list` 负责获取流水线所需的数据或状态。 实现过程中会调用 `subprocess.check_output`, `re.sub`, `_synthesize_cpu_list` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`LogicalCPUInfo`**: Core class that organizes module behavior. / **`LogicalCPUInfo`**：组织模块行为的核心类。
- **`MemoryNodeInfo`**: Core class that organizes module behavior. / **`MemoryNodeInfo`**：组织模块行为的核心类。
- **`get_memory_affinity`**: Key helper or entry point in this file. / **`get_memory_affinity`**：本文件中的关键辅助函数或入口。
- **`parse_id_list`**: Key helper or entry point in this file. / **`parse_id_list`**：本文件中的关键辅助函数或入口。
- **`get_memory_node_info`**: Key helper or entry point in this file. / **`get_memory_node_info`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: json, os, subprocess, sys, dataclasses, functools
- **Third-party / 第三方**: psutil, regex
- **Internal vLLM / vLLM 内部依赖**: None / 无
