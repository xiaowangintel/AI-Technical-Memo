# perf_logger.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/utils/perf_logger.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for perf logger in the multimodal generation stack. Key symbols include `MemorySnapshot`, `RequestMetrics`, `get_diffusion_perf_log_dir`. / 该模块包含多模态生成体系中与 perf logger 相关的运行时支持代码。 关键符号包括 `MemorySnapshot`, `RequestMetrics`, `get_diffusion_perf_log_dir`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo
import dataclasses
import json
import logging
import os
import subprocess
import sys
import time
from datetime import datetime
from functools import lru_cache
from pathlib import Path
from typing import Any, Dict, Optional

import torch
# ...
    init_logger,
)

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 31-44: Class `MemorySnapshot` / 类 `MemorySnapshot`
```python
@dataclasses.dataclass
class MemorySnapshot:
    allocated_mb: float  # current allocated memory
    reserved_mb: float  # current reserved memory (actual VRAM)
    peak_allocated_mb: float  # peak allocated since last reset
    peak_reserved_mb: float  # peak reserved since last reset

    def to_dict(self) -> Dict[str, Any]:
        return {
            "allocated_mb": round(self.allocated_mb, 2),
            "reserved_mb": round(self.reserved_mb, 2),
            "peak_allocated_mb": round(self.peak_allocated_mb, 2),
            "peak_reserved_mb": round(self.peak_reserved_mb, 2),
        }
```
**EN:** This class models `MemorySnapshot`. Important methods include `to_dict`.
**CN:** 该类实现 `MemorySnapshot`。 其中较重要的方法包括 `to_dict`。

### Lines 47-85: Class `RequestMetrics` / 类 `RequestMetrics`
```python
@dataclasses.dataclass
class RequestMetrics:
    """Performance metrics for a single request, including timings and memory snapshots."""

    def __init__(self, request_id: str):
        self.request_id = request_id
        self.stages: Dict[str, float] = {}
        self.steps: list[float] = []
        self.total_duration_ms: float = 0.0
        # memory tracking: {checkpoint_name: MemorySnapshot}
        self.memory_snapshots: Dict[str, MemorySnapshot] = {}

    @property
    def total_duration_s(self) -> float:
# ...
                name: snapshot.to_dict()
                for name, snapshot in self.memory_snapshots.items()
            },
        }
```
**EN:** This class models `RequestMetrics`. Performance metrics for a single request, including timings and memory snapshots. Important methods include `__init__`, `total_duration_s`, `record_stage`, `record_step`.
**CN:** 该类实现 `RequestMetrics`。 文档字符串指出：Performance metrics for a single request, including timings and memory snapshots. 其中较重要的方法包括 `__init__`, `total_duration_s`, `record_stage`, `record_step`。

### Lines 88-99: Function `get_diffusion_perf_log_dir` / 函数 `get_diffusion_perf_log_dir`
```python
def get_diffusion_perf_log_dir() -> str:
    """
    Determines the directory for performance logs.
    """
    log_dir = os.environ.get("SGLANG_PERF_LOG_DIR")
    if log_dir:
        return os.path.abspath(log_dir)
    if log_dir is None:
        sglang_path = Path(sglang.__file__).resolve()
        target_path = (sglang_path.parent / "../../.cache/logs").resolve()
        return str(target_path)
    return ""
```
**EN:** This function drives `get_diffusion_perf_log_dir`. Determines the directory for performance logs.
**CN:** 这个函数负责 `get_diffusion_perf_log_dir`。 文档字符串说明：Determines the directory for performance logs.

### Lines 102-118: Function `get_git_commit_hash` / 函数 `get_git_commit_hash`
```python
@lru_cache(maxsize=1)
def get_git_commit_hash() -> str:
    try:
        commit_hash = os.environ.get("SGLANG_GIT_COMMIT")
        if not commit_hash:
            commit_hash = (
                subprocess.check_output(
                    ["git", "rev-parse", "HEAD"], stderr=subprocess.DEVNULL
                )
                .strip()
                .decode("utf-8")
            )
        _CACHED_COMMIT_HASH = commit_hash
        return commit_hash
    except (subprocess.CalledProcessError, FileNotFoundError):
        _CACHED_COMMIT_HASH = "N/A"
        return "N/A"
```
**EN:** This function drives `get_git_commit_hash`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_git_commit_hash`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 121-140: Function `capture_memory_snapshot` / 函数 `capture_memory_snapshot`
```python
def capture_memory_snapshot() -> MemorySnapshot:
    if not torch.get_device_module().is_available():
        return MemorySnapshot(
            allocated_mb=0.0,
            reserved_mb=0.0,
            peak_allocated_mb=0.0,
            peak_reserved_mb=0.0,
        )

    allocated = torch.get_device_module().memory_allocated()
    reserved = torch.get_device_module().memory_reserved()
    peak_allocated = torch.get_device_module().max_memory_allocated()
    peak_reserved = torch.get_device_module().max_memory_reserved()

# ...
        reserved_mb=reserved / (1024**2),
        peak_allocated_mb=peak_allocated / (1024**2),
        peak_reserved_mb=peak_reserved / (1024**2),
    )
```
**EN:** This function drives `capture_memory_snapshot`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `capture_memory_snapshot`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 143-178: Class `RequestPerfRecord` / 类 `RequestPerfRecord`
```python
@dataclasses.dataclass
class RequestPerfRecord:
    request_id: str

    timestamp: str
    commit_hash: str
    tag: str

    stages: list[dict]
    steps: list[float]
    total_duration_ms: float
    memory_snapshots: dict[str, dict] = dataclasses.field(default_factory=dict)

    def __init__(
# ...
        self.stages = stages
        self.steps = steps
        self.total_duration_ms = total_duration_ms
        self.memory_snapshots = memory_snapshots or {}
```
**EN:** This class models `RequestPerfRecord`. Important methods include `__init__`.
**CN:** 该类实现 `RequestPerfRecord`。 其中较重要的方法包括 `__init__`。

### Lines 181-272: Class `StageProfiler` / 类 `StageProfiler`
```python
class StageProfiler:
    """
    A unified context manager, records performance metrics (usually of a single Stage or a step) into a provided RequestMetrics object (usually from a Req).
    """

    def __init__(
        self,
        stage_name: str,
        logger: _SGLDiffusionLogger,
        metrics: Optional["RequestMetrics"],
        log_stage_start_end: bool = False,
        perf_dump_path_provided: bool = False,
        capture_memory: bool = False,
        record_as_step: bool = False,
# ...
                    f"after_{self.stage_name}", snapshot
                )

        return False
```
**EN:** This class models `StageProfiler`. A unified context manager, records performance metrics (usually of a single Stage or a step) into a provided RequestMetrics object (usually from a Req). Important methods include `__init__`, `_should_record_as_step`, `__enter__`, `__exit__`.
**CN:** 该类实现 `StageProfiler`。 文档字符串指出：A unified context manager, records performance metrics (usually of a single Stage or a step) into a provided RequestMetrics object (usually from a Req). 其中较重要的方法包括 `__init__`, `_should_record_as_step`, `__enter__`, `__exit__`。

### Lines 275-375: Class `PerformanceLogger` / 类 `PerformanceLogger`
```python
class PerformanceLogger:
    """
    A global utility class for logging performance metrics for all request, categorized by request-id.

    Serves both as a runtime logger (stream to file) and a dump utility.

    Notice that RequestMetrics stores the performance metrics of a single request
    """

    @classmethod
    def dump_benchmark_report(
        cls,
        file_path: str,
        metrics: "RequestMetrics",
# ...
                    f.write(json.dumps(dataclasses.asdict(record)) + "\n")

        except (OSError, PermissionError) as e:
            print(f"WARNING: Failed to log performance record: {e}", file=sys.stderr)
```
**EN:** This class models `PerformanceLogger`. A global utility class for logging performance metrics for all request, categorized by request-id. Important methods include `dump_benchmark_report`, `log_request_summary`.
**CN:** 该类实现 `PerformanceLogger`。 文档字符串指出：A global utility class for logging performance metrics for all request, categorized by request-id. 其中较重要的方法包括 `dump_benchmark_report`, `log_request_summary`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略
- Command-line interface / 命令行接口
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang`, `sglang.multimodal_gen.envs`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `dateutil.tz`
- **Stdlib / 标准库**: `dataclasses`, `json`, `logging`, `os`, `subprocess`, `sys`, `time`, `datetime`
