# gpu_monitor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/infra/gpu_monitor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises gpu monitor behavior in the end-to-end infrastructure layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试基础设施 中与 gpu monitor 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module docstring
```python
"""GPU utilization monitoring for benchmarks.

This module provides a low-impact GPU monitor that runs in a separate process
and collects utilization samples using NVML.
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 7-15: Imports and dependencies
```python
from __future__ import annotations

import json
import logging
import os
import time
from multiprocessing import Process
from pathlib import Path
from typing import TYPE_CHECKING
```
**EN:** This block imports `__future__`, `json`, `logging`, `os`, and 4 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 17-18: If block
```python
if TYPE_CHECKING:
    from typing import Any
```
**EN:** This top-level block contributes supporting logic to the module and is executed or defined as part of file initialization.
**CN:** 这个顶层代码块为模块提供辅助逻辑，并在文件初始化阶段被执行或定义。

### Lines 20-20: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 23-34: Helper function `_percentile`
```python
def _percentile(samples: list[float], p: float) -> float:
    """Calculate percentile from sorted samples."""
    if not samples:
        return 0.0
    sorted_samples = sorted(samples)
    idx = max(
        0,
        min(
            len(sorted_samples) - 1, int(round((p / 100.0) * (len(sorted_samples) - 1)))
        ),
    )
    return float(sorted_samples[idx])
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 37-65: Helper function `_compute_stats`
```python
def _compute_stats(samples: list[float]) -> dict[str, float]:
    """Compute statistics for a list of samples."""
    if not samples:
        return {
            "mean": 0.0,
            "min": 0.0,
            "max": 0.0,
            "p5": 0.0,
            "p10": 0.0,
            "p25": 0.0,
            "p50": 0.0,
            "p75": 0.0,
            "p90": 0.0,
            "p95": 0.0,
            "count": 0,
        }
    return {
        "mean": sum(samples) / len(samples),
        "min": min(samples),
        "max": max(samples),
        "p5": _percentile(samples, 5),
        "p10": _percentile(samples, 10),
        "p25": _percentile(samples, 25),
        "p50": _percentile(samples, 50),
        "p75": _percentile(samples, 75),
        "p90": _percentile(samples, 90),
        "p95": _percentile(samples, 95),
        "count": len(samples),
    }
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 68-126: Helper function `_monitor_loop`
```python
def _monitor_loop(pid: int, output_path: str, interval: float) -> None:
    """Main monitoring loop - runs in separate process.

    Monitors GPU utilization until the target process exits, then writes
    results to output_path as JSON.
    """
    # Lower process priority to minimize impact on benchmark
    try:
        os.nice(10)
    except Exception:
        pass

    # Initialize NVML
    try:
        import pynvml

        pynvml.nvmlInit()
    except Exception as e:
        logger.warning("Failed to initialize NVML: %s", e)
        _write_empty_result(output_path)
        return

    # Get GPU handles
    try:
        device_count = pynvml.nvmlDeviceGetCount()
        handles = [pynvml.nvmlDeviceGetHandleByIndex(i) for i in range(device_count)]
    except Exception as e:
        logger.warning("Failed to get GPU handles: %s", e)
        _write_empty_result(output_path)
        _shutdown_nvml()
        return

    # Collect samples
    per_gpu_samples: dict[str, list[float]] = {str(i): [] for i in range(device_count)}
    overall_samples: list[float] = []

    try:
        while _process_alive(pid):
            try:
                gpu_utils = []
                for idx, handle in enumerate(handles):
                    try:
                        util = pynvml.nvmlDeviceGetUtilizationRates(handle).gpu
                        gpu_utils.append(float(util))
                        per_gpu_samples[str(idx)].append(float(util))
                    except Exception:
                        continue

                if gpu_utils:
                    avg = sum(gpu_utils) / len(gpu_utils)
                    overall_samples.append(avg)
            except Exception:
                pass

            time.sleep(interval)
    finally:
        # Write results
        _write_result(output_path, pid, interval, overall_samples, per_gpu_samples)
        _shutdown_nvml()
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 129-135: Helper function `_process_alive`
```python
def _process_alive(pid: int) -> bool:
    """Check if process is still running."""
    try:
        os.kill(pid, 0)
        return True
    except (OSError, ProcessLookupError):
        return False
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 138-153: Helper function `_write_empty_result`
```python
def _write_empty_result(path: str) -> None:
    """Write empty result file."""
    try:
        os.makedirs(os.path.dirname(path), exist_ok=True)
        with open(path, "w") as f:
            json.dump(
                {
                    "count": 0,
                    "overall": {"mean": 0.0},
                    "per_gpu": {},
                    "raw": {"overall": [], "per_gpu": {}},
                },
                f,
            )
    except Exception:
        pass
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 156-184: Helper function `_write_result`
```python
def _write_result(
    path: str,
    pid: int,
    interval: float,
    overall_samples: list[float],
    per_gpu_samples: dict[str, list[float]],
) -> None:
    """Write monitoring results to JSON file."""
    try:
        os.makedirs(os.path.dirname(path), exist_ok=True)
        with open(path, "w") as f:
            json.dump(
                {
                    "bench_pid": pid,
                    "interval_sec": interval,
                    "count": len(overall_samples),
                    "overall": _compute_stats(overall_samples),
                    "per_gpu": {
                        k: _compute_stats(v) for k, v in per_gpu_samples.items()
                    },
                    "raw": {
                        "overall": overall_samples,
                        "per_gpu": per_gpu_samples,
                    },
                },
                f,
            )
    except Exception as e:
        logger.warning("Failed to write GPU monitor results: %s", e)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 187-194: Helper function `_shutdown_nvml`
```python
def _shutdown_nvml() -> None:
    """Shutdown NVML."""
    try:
        import pynvml

        pynvml.nvmlShutdown()
    except Exception:
        pass
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 197-313: Class `GPUMonitor`
```python
class GPUMonitor:
    """GPU utilization monitor for benchmarks.

    Usage:
        monitor = GPUMonitor(output_dir="benchmark_results")
        monitor.start(target_pid=12345)
        # ... run benchmark ...
        result = monitor.stop()
        monitor.assert_thresholds({"gpu_util_p50_min": 99})
    """

    def __init__(
        self,
        output_dir: str | Path = ".",
        interval: float = 2.0,
    ):
        self.output_dir = Path(output_dir)
        self.interval = interval
        self._process: Process | None = None
        self._output_path: str | None = None
        self._result: dict[str, Any] | None = None

    @property
    def output_path(self) -> str | None:
        """Path to the GPU utilization JSON file."""
        return self._output_path

    def start(self, target_pid: int) -> None:
        """Start monitoring GPU utilization for the target process."""
        self._output_path = str(self.output_dir / "gpu_utilization.json")
        self._result = None

        self._process = Process(
            target=_monitor_loop,
            args=(target_pid, self._output_path, self.interval),
            daemon=True,
        )
        self._process.start()
        logger.debug("Started GPU monitor for PID %d", target_pid)

    def stop(self, timeout: float = 5.0) -> dict[str, Any] | None:
        """Stop monitoring and return results."""
        if self._process is None:
            return None

        try:
            self._process.join(timeout=timeout)
        except Exception:
            pass

        if self._process.is_alive():
            try:
                self._process.terminate()
            except Exception:
                pass

        self._process = None
        self._result = self._read_result()
        return self._result

    def _read_result(self) -> dict[str, Any] | None:
        """Read results from output file."""
        if not self._output_path or not os.path.exists(self._output_path):
            return None
        try:
            with open(self._output_path) as f:
                return json.load(f)
        except Exception as e:
            logger.warning("Failed to read GPU monitor result: %s", e)
            return None

    def log_summary(self) -> None:
        """Log a summary of GPU utilization."""
        result = self._result or self._read_result()
        if not result or result.get("count", 0) <= 0:
            logger.warning("GPU utilization monitor produced no samples")
            return

        overall = result.get("overall", {})
        logger.info(
            "GPU utilization: mean=%.2f%% p50=%.2f%% (samples=%d)",
            overall.get("mean", 0.0),
            overall.get("p50", 0.0),
            result.get("count", 0),
        )

    def assert_thresholds(self, thresholds: dict[str, float] | None) -> None:
        """Assert GPU utilization meets thresholds.

        Supported thresholds:
            - gpu_util_mean_min: Minimum mean GPU utilization %
            - gpu_util_p50_min: Minimum p50 GPU utilization %
        """
        if not thresholds:
            return

        result = self._result or self._read_result()
        if not result or result.get("count", 0) <= 0:
            logger.warning("GPU utilization monitor produced no samples")
            return

        overall = result.get("overall", {})

        mean_threshold = thresholds.get("gpu_util_mean_min")
        if mean_threshold is not None:
            mean_value = overall.get("mean", 0.0)
            assert (
                mean_value >= mean_threshold
            ), f"GPU utilization mean below threshold: {mean_value:.2f}% < {mean_threshold}%"

        p50_threshold = thresholds.get("gpu_util_p50_min")
        if p50_threshold is not None:
            p50_value = overall.get("p50")
            if p50_value is not None:
                assert (
                    p50_value >= p50_threshold
                ), f"GPU utilization p50 below threshold: {p50_value:.2f}% < {p50_threshold}%"
```
**EN:** Class `GPUMonitor` groups related state and behavior. It exposes 7 method(s) that implement the module's primary abstraction.
**CN:** 类 `GPUMonitor` 将相关状态与行为封装在一起，提供 7 个方法来实现本模块的核心抽象。

### Lines 316-329: Helper function `should_monitor`
```python
def should_monitor(thresholds: dict[str, Any] | None) -> bool:
    """Check if GPU monitoring should be enabled.

    Returns True if:
    - thresholds contains gpu_util_mean_min or gpu_util_p50_min, OR
    - GPU_UTIL_LOG environment variable is truthy
    """
    if thresholds:
        if thresholds.get("gpu_util_mean_min") is not None:
            return True
        if thresholds.get("gpu_util_p50_min") is not None:
            return True

    return os.environ.get("GPU_UTIL_LOG", "").lower() in ("1", "true", "yes")
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Test infrastructure orchestration / 测试基础设施编排
- Primary classes: `GPUMonitor` / 主要类：`GPUMonitor`
- Reusable functions: `_percentile`, `_compute_stats`, `_monitor_loop`, `_process_alive`, and 4 more / 可复用函数：`_percentile`, `_compute_stats`, `_monitor_loop`, `_process_alive`, and 4 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `json`, `logging`, `multiprocessing`, `os`, `pathlib`, `time`, `typing`
- **Third-party / 第三方**: `pynvml`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
