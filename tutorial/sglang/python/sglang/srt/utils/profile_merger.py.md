# profile_merger.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/profile_merger.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for profiling and performance inspection helpers. / 为 SGLang 运行时提供面向性能剖析与性能观测辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Module setup and shared state / 模块设置与共享状态
```python
"""Merge Chrome trace files from multiple ranks (TP, DP, PP, EP) into a single trace."""

import glob
import gzip
import json
import logging
import os
import re
from typing import Any, Dict, List, Optional, Tuple

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `glob`, `gzip`, `json`, `logging`, `os`, `re`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `glob`, `gzip`, `json`, `logging`, `os`, `re`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 14-15: Class `ProfileMerger` declaration / 类 `ProfileMerger` 声明
```python
class ProfileMerger:
    """Merge profile traces from all parallelism types: TP, DP, PP, EP."""
```
**EN:** This class establishes `ProfileMerger` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `merge_chrome_traces`, `_discover_trace_files`, `_extract_rank_info`, `_create_rank_label`, `_handle_file`.
**CN:** 该类将 `ProfileMerger` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `merge_chrome_traces`, `_discover_trace_files`, `_extract_rank_info`, `_create_rank_label`, `_handle_file` 等方法。

### Lines 17-37: Method `ProfileMerger.__init__` / 方法 `ProfileMerger.__init__`
```python
    def __init__(self, output_dir: str, profile_id: str):
        self.output_dir = output_dir
        self.profile_id = profile_id
        self.merged_trace_path = os.path.join(
            output_dir, f"merged-{profile_id}.trace.json.gz"
        )

        # Rank types in priority order (used for sorting and labeling)
        self.rank_types = ["tp", "dp", "pp", "ep"]

        # Sort index multipliers: DP (highest) > EP > PP > TP (lowest)
        # These ensure proper visual ordering in trace viewer
        self.sort_index_multipliers = {
            "dp_rank": 100_000_000,
            "ep_rank": 1_000_000,
            "pp_rank": 10_000,
            "tp_rank": 100,
        }

        # PID threshold for sort_index updates (only update for system PIDs < 1000)
        self.pid_sort_index_threshold = 1000
```
**EN:** This method implements `__init__` on `ProfileMerger`. It primarily calls `os.path.join` to complete its work. State updates are written into `self.output_dir`, `self.profile_id`, `self.merged_trace_path`, `self.rank_types`, `self.sort_index_multipliers`, `self.pid_sort_index_threshold`.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `__init__`。 它主要通过调用 `os.path.join` 来完成任务。 状态更新主要写入 `self.output_dir`, `self.profile_id`, `self.merged_trace_path`, `self.rank_types`, `self.sort_index_multipliers`, `self.pid_sort_index_threshold`。

### Lines 39-82: Method `ProfileMerger.merge_chrome_traces` / 方法 `ProfileMerger.merge_chrome_traces`
```python
    def merge_chrome_traces(self) -> str:
        """Merge Chrome traces from all ranks into a single trace.

        Returns:
            Path to merged trace file.

        Raises:
            ValueError: If no trace files found.
        """
        trace_files = self._discover_trace_files()
        if not trace_files:
            raise ValueError(f"No trace files found for profile_id: {self.profile_id}")

        logger.info(f"Found {len(trace_files)} trace files to merge")

        merged_trace = {"traceEvents": []}
        all_device_properties = []

        for trace_file in sorted(trace_files, key=self._get_rank_sort_key):
            rank_info = self._extract_rank_info(trace_file)
            logger.info(f"Processing {trace_file} with rank info: {rank_info}")

            output = self._handle_file(trace_file, rank_info)

            merged_trace["traceEvents"].extend(output["traceEvents"])

            if "deviceProperties" in output:
                all_device_properties.extend(output["deviceProperties"])
# ... omitted for brevity ...
            f.write(json.dumps(merged_trace).encode("utf-8"))

        logger.info(f"Merged profile saved to: {self.merged_trace_path}")
        logger.info(f"Total events merged: {len(merged_trace['traceEvents'])}")

        return self.merged_trace_path
```
**EN:** This method implements `merge_chrome_traces` on `ProfileMerger`. It primarily calls `self._discover_trace_files`, `logger.info`, `sorted`, `ValueError`, `self._extract_rank_info`, `self._handle_file` to complete its work. State updates are written into `trace_files`, `merged_trace`, `all_device_properties`, `rank_info`, `output`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `merge_chrome_traces`。 它主要通过调用 `self._discover_trace_files`, `logger.info`, `sorted`, `ValueError`, `self._extract_rank_info`, `self._handle_file` 来完成任务。 状态更新主要写入 `trace_files`, `merged_trace`, `all_device_properties`, `rank_info`, `output`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 84-101: Method `ProfileMerger._discover_trace_files` / 方法 `ProfileMerger._discover_trace_files`
```python
    def _discover_trace_files(self) -> List[str]:
        """Discover trace files matching profile_id (supports TP/DP/PP/EP formats)."""
        patterns = [f"{self.profile_id}*.trace.json.gz"]

        trace_files = []
        for pattern in patterns:
            search_pattern = os.path.join(self.output_dir, pattern)
            trace_files.extend(glob.glob(search_pattern))

        trace_files = [
            f
            for f in trace_files
            if not f.endswith(f"merged-{self.profile_id}.trace.json.gz")
            and not f.endswith("-memory.pickle")
            and "TP-" in f
        ]
        trace_files = list(set(trace_files))
        return trace_files
```
**EN:** This method implements `_discover_trace_files` on `ProfileMerger`. It primarily calls `list`, `os.path.join`, `trace_files.extend`, `set`, `glob.glob`, `f.endswith` to complete its work. State updates are written into `patterns`, `trace_files`, `search_pattern`. The implementation relies on iteration.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `_discover_trace_files`。 它主要通过调用 `list`, `os.path.join`, `trace_files.extend`, `set`, `glob.glob`, `f.endswith` 来完成任务。 状态更新主要写入 `patterns`, `trace_files`, `search_pattern`。 实现中使用了迭代逻辑。

### Lines 103-113: Method `ProfileMerger._extract_rank_info` / 方法 `ProfileMerger._extract_rank_info`
```python
    def _extract_rank_info(self, filename: str) -> Dict[str, int]:
        """Extract rank info (TP/DP/PP/EP) from filename."""
        basename = os.path.basename(filename)
        rank_info = {}

        for rank_type in self.rank_types:
            match = re.search(rf"{rank_type.upper()}-(\d+)", basename)
            if match:
                rank_info[f"{rank_type}_rank"] = int(match.group(1))

        return rank_info
```
**EN:** This method implements `_extract_rank_info` on `ProfileMerger`. It primarily calls `os.path.basename`, `re.search`, `int`, `match.group`, `rank_type.upper` to complete its work. State updates are written into `basename`, `rank_info`, `match`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `_extract_rank_info`。 它主要通过调用 `os.path.basename`, `re.search`, `int`, `match.group`, `rank_type.upper` 来完成任务。 状态更新主要写入 `basename`, `rank_info`, `match`。 实现中使用了条件分支、迭代逻辑。

### Lines 115-122: Method `ProfileMerger._create_rank_label` / 方法 `ProfileMerger._create_rank_label`
```python
    def _create_rank_label(self, rank_info: Dict[str, int]) -> str:
        parts = []
        for rank_type in self.rank_types:
            rank_key = f"{rank_type}_rank"
            if rank_key in rank_info:
                parts.append(f"{rank_type.upper()}{rank_info[rank_key]:02d}")

        return f"[{'-'.join(parts)}]" if parts else "[Unknown]"
```
**EN:** This method implements `_create_rank_label` on `ProfileMerger`. It primarily calls `parts.append`, `join`, `rank_type.upper` to complete its work. State updates are written into `parts`, `rank_key`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `_create_rank_label`。 它主要通过调用 `parts.append`, `join`, `rank_type.upper` 来完成任务。 状态更新主要写入 `parts`, `rank_key`。 实现中使用了条件分支、迭代逻辑。

### Lines 124-141: Method `ProfileMerger._handle_file` / 方法 `ProfileMerger._handle_file`
```python
    def _handle_file(self, path: str, rank_info: Dict[str, int]) -> Dict[str, Any]:
        logger.info(f"Processing file: {path}")

        try:
            with gzip.open(path, "rt", encoding="utf-8") as f:
                trace = json.load(f)

            output = {
                key: value for key, value in trace.items() if key != "traceEvents"
            }
            output["traceEvents"] = self._process_events(
                trace.get("traceEvents", []), rank_info
            )
            return output

        except Exception as e:
            logger.error(f"Failed to process trace file {path}: {e}")
            return {"traceEvents": []}
```
**EN:** This method implements `_handle_file` on `ProfileMerger`. It primarily calls `logger.info`, `self._process_events`, `gzip.open`, `json.load`, `trace.get`, `logger.error` to complete its work. State updates are written into `output`, `trace`. The implementation relies on context-managed resources, error handling.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `_handle_file`。 它主要通过调用 `logger.info`, `self._process_events`, `gzip.open`, `json.load`, `trace.get`, `logger.error` 来完成任务。 状态更新主要写入 `output`, `trace`。 实现中使用了上下文管理资源、错误处理。

### Lines 143-159: Method `ProfileMerger._process_events` / 方法 `ProfileMerger._process_events`
```python
    def _process_events(
        self, events: List[Dict], rank_info: Dict[str, int]
    ) -> List[Dict]:
        """Process events: update sort_index and add rank labels to PIDs."""
        rank_label = self._create_rank_label(rank_info)

        for event in events:
            if event.get("name") == "process_sort_index":
                pid = self._maybe_cast_int(event.get("pid"))
                if pid is not None and pid < self.pid_sort_index_threshold:
                    event["args"]["sort_index"] = self._calculate_sort_index(
                        rank_info, pid
                    )

            event["pid"] = f"{rank_label} {event['pid']}"

        return events
```
**EN:** This method implements `_process_events` on `ProfileMerger`. It primarily calls `self._create_rank_label`, `event.get`, `self._maybe_cast_int`, `self._calculate_sort_index` to complete its work. State updates are written into `rank_label`, `event`, `pid`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `_process_events`。 它主要通过调用 `self._create_rank_label`, `event.get`, `self._maybe_cast_int`, `self._calculate_sort_index` 来完成任务。 状态更新主要写入 `rank_label`, `event`, `pid`。 实现中使用了条件分支、迭代逻辑。

### Lines 161-165: Method `ProfileMerger._calculate_sort_index` / 方法 `ProfileMerger._calculate_sort_index`
```python
    def _calculate_sort_index(self, rank_info: Dict[str, int], pid: int) -> int:
        sort_index = pid
        for rank_type, multiplier in self.sort_index_multipliers.items():
            sort_index += rank_info.get(rank_type, 0) * multiplier
        return sort_index
```
**EN:** This method implements `_calculate_sort_index` on `ProfileMerger`. It primarily calls `self.sort_index_multipliers.items`, `rank_info.get` to complete its work. State updates are written into `sort_index`. The implementation relies on iteration.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `_calculate_sort_index`。 它主要通过调用 `self.sort_index_multipliers.items`, `rank_info.get` 来完成任务。 状态更新主要写入 `sort_index`。 实现中使用了迭代逻辑。

### Lines 167-172: Method `ProfileMerger._get_rank_sort_key` / 方法 `ProfileMerger._get_rank_sort_key`
```python
    def _get_rank_sort_key(self, path: str) -> Tuple[int, int, int, int]:
        rank_info = self._extract_rank_info(path)
        return tuple(
            rank_info.get(f"{rank_type}_rank", 0)
            for rank_type in ["dp", "ep", "pp", "tp"]
        )
```
**EN:** This method implements `_get_rank_sort_key` on `ProfileMerger`. It primarily calls `self._extract_rank_info`, `tuple`, `rank_info.get` to complete its work. State updates are written into `rank_info`.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `_get_rank_sort_key`。 它主要通过调用 `self._extract_rank_info`, `tuple`, `rank_info.get` 来完成任务。 状态更新主要写入 `rank_info`。

### Lines 174-178: Method `ProfileMerger._maybe_cast_int` / 方法 `ProfileMerger._maybe_cast_int`
```python
    def _maybe_cast_int(self, x) -> Optional[int]:
        try:
            return int(x)
        except (ValueError, TypeError):
            return None
```
**EN:** This method implements `_maybe_cast_int` on `ProfileMerger`. It primarily calls `int` to complete its work. The implementation relies on error handling.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `_maybe_cast_int`。 它主要通过调用 `int` 来完成任务。 实现中使用了错误处理。

### Lines 180-199: Method `ProfileMerger.get_merge_summary` / 方法 `ProfileMerger.get_merge_summary`
```python
    def get_merge_summary(self) -> Dict[str, Any]:
        if not os.path.exists(self.merged_trace_path):
            return {"error": "Merged trace file not found"}

        try:
            with gzip.open(self.merged_trace_path, "rt") as f:
                merged_data = json.load(f)

            trace_files = self._discover_trace_files()

            return {
                "merged_file": self.merged_trace_path,
                "total_events": len(merged_data.get("traceEvents", [])),
                "total_files": len(trace_files),
                "source_files": [os.path.basename(f) for f in trace_files],
                "profile_id": self.profile_id,
                "device_properties_count": len(merged_data.get("deviceProperties", [])),
            }
        except Exception as e:
            return {"error": f"Failed to read merged trace: {str(e)}"}
```
**EN:** This method implements `get_merge_summary` on `ProfileMerger`. It primarily calls `os.path.exists`, `self._discover_trace_files`, `gzip.open`, `json.load`, `len`, `merged_data.get` to complete its work. State updates are written into `trace_files`, `merged_data`. The implementation relies on conditional branches, context-managed resources, error handling.
**CN:** 该方法（属于 `ProfileMerger`）实现了 `get_merge_summary`。 它主要通过调用 `os.path.exists`, `self._discover_trace_files`, `gzip.open`, `json.load`, `len`, `merged_data.get` 来完成任务。 状态更新主要写入 `trace_files`, `merged_data`。 实现中使用了条件分支、上下文管理资源、错误处理。

## Key Concepts / 关键概念
- **Classes / 类**: `ProfileMerger`
- **Functions / 函数**: `__init__`, `merge_chrome_traces`, `_discover_trace_files`, `_extract_rank_info`, `_create_rank_label`, `_handle_file`, `_process_events`, `_calculate_sort_index`
- **Themes / 主题**: `profile`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: None / 无
- **Standard library / 标准库**: `glob`, `gzip`, `json`, `logging`, `os`, `re`, `typing`
