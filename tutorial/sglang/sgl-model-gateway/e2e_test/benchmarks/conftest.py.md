# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/benchmarks/conftest.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This pytest configuration module defines shared fixtures, hooks, and markers for the benchmark suite. It centralizes reusable test setup so individual test files stay focused on assertions. / 该 pytest 配置模块为 端到端基准测试 定义共享夹具、钩子与标记，将可复用的测试初始化集中管理，使各测试文件能专注于断言。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module docstring
```python
"""Benchmark-specific fixtures."""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 3-15: Imports and dependencies
```python
from __future__ import annotations

import logging
import os
import shutil
import subprocess
import time
from pathlib import Path

import pytest
from infra import GPUMonitor, should_monitor_gpu, terminate_process

from .results import BenchmarkResult
```
**EN:** This block imports `__future__`, `logging`, `os`, `shutil`, and 6 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 17-17: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 20-57: Helper function `_build_command`
```python
def _build_command(
    cli: str,
    router_url: str,
    model_path: str,
    experiment_folder: str,
    num_concurrency: int,
    traffic_scenario: str,
    max_requests: int,
) -> list[str]:
    """Build genai-bench command."""
    return [
        cli,
        "benchmark",
        "--api-backend",
        "openai",
        "--api-base",
        router_url,
        "--api-key",
        "dummy-token",
        "--api-model-name",
        model_path,
        "--model-tokenizer",
        model_path,
        "--task",
        "text-to-text",
        "--num-concurrency",
        str(num_concurrency),
        "--traffic-scenario",
        traffic_scenario,
        "--max-requests-per-run",
        str(max_requests),
        "--max-time-per-run",
        "3",
        "--experiment-folder-name",
        experiment_folder,
        "--experiment-base-dir",
        str(Path.cwd()),
    ]
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 60-86: Helper function `_find_results`
```python
def _find_results(experiment_folder: str, timeout: int = 10) -> list[Path]:
    """Find benchmark result JSON files."""
    base = Path.cwd()
    folder = base / experiment_folder

    if not folder.is_dir():
        # Search for folder
        for p in base.rglob(experiment_folder):
            if p.is_dir() and p.name == experiment_folder:
                folder = p
                break

    if not folder.is_dir():
        raise AssertionError(f"Experiment folder not found: {experiment_folder}")

    # Wait for JSON results
    for _ in range(timeout):
        files = [
            p
            for p in folder.rglob("*.json")
            if "experiment_metadata" not in p.name and "gpu_utilization" not in p.name
        ]
        if files:
            return files
        time.sleep(1)

    raise AssertionError(f"No JSON results found in {folder}")
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 89-102: Helper function `_cleanup_procs`
```python
def _cleanup_procs(procs: list, drain_delay: int) -> None:
    """Terminate processes gracefully."""
    if not procs:
        return
    if drain_delay > 0:
        time.sleep(drain_delay)
    for p in procs:
        try:
            proc = getattr(p, "proc", p) if hasattr(p, "proc") else p
            if isinstance(proc, subprocess.Popen):
                terminate_process(proc)
        except Exception:
            pass
    time.sleep(2)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 103-222: Fixture `genai_bench_runner`
```python


@pytest.fixture(scope="session")
def genai_bench_runner():
    """Run genai-bench and validate metrics.

    Usage:
        def test_perf(setup_backend, genai_bench_runner):
            backend, model_path, client, gateway = setup_backend
            genai_bench_runner(
                router_url=gateway.base_url,
                model_path=model_path,
                experiment_folder="benchmark_results",
                thresholds={"ttft_mean_max": 5, "gpu_util_p50_min": 99},
            )
    """

    def _run(
        *,
        router_url: str,
        model_path: str,
        experiment_folder: str,
        thresholds: dict | None = None,
        timeout_sec: int | None = None,
        num_concurrency: int = 32,
        traffic_scenario: str = "D(4000,100)",
        max_requests_per_run: int | None = None,
        kill_procs: list | None = None,
        drain_delay_sec: int = 6,
    ) -> None:
        cli = shutil.which("genai-bench")
        if not cli:
            pytest.fail("genai-bench CLI not found")

        # Clean previous results
        exp_dir = Path.cwd() / experiment_folder
        if exp_dir.exists():
            shutil.rmtree(exp_dir, ignore_errors=True)

        # Build and run command
        max_requests = max_requests_per_run or num_concurrency * 5
        cmd = _build_command(
            cli,
            router_url,
            model_path,
            experiment_folder,
            num_concurrency,
            traffic_scenario,
            max_requests,
        )
        timeout = timeout_sec or int(os.environ.get("GENAI_BENCH_TEST_TIMEOUT", "120"))

        try:
            proc = subprocess.Popen(
                cmd,
                env=os.environ.copy(),
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
            )
        except FileNotFoundError:
            pytest.fail(f"genai-bench executable not found at {cli}")
        except PermissionError:
            pytest.fail(f"Permission denied executing {cli}")
        except OSError as e:
            pytest.fail(f"Failed to start genai-bench: {e}")

        # Start GPU monitor if needed
        gpu_monitor: GPUMonitor | None = None
        if should_monitor_gpu(thresholds):
            interval = float(os.environ.get("GPU_UTIL_SAMPLE_INTERVAL", "2.0"))
            gpu_monitor = GPUMonitor(output_dir=exp_dir, interval=interval)
            gpu_monitor.start(target_pid=proc.pid)

        try:
            stdout, stderr = proc.communicate(timeout=timeout)
        except subprocess.TimeoutExpired:
            proc.kill()
            stdout, stderr = proc.communicate()
            logger.error("genai-bench timed out after %ds", timeout)

        # Log output if process failed or for debugging
        if proc.returncode != 0:
            logger.error(
                "genai-bench exited with code %d\nstdout:\n%s\nstderr:\n%s",
                proc.returncode,
                stdout or "(empty)",
                stderr or "(empty)",
            )

        try:
            # Parse and validate results
            for path in _find_results(experiment_folder):
                result = BenchmarkResult.from_json(path)
                result.log(experiment_folder, logger)
                if thresholds:
                    result.validate(thresholds)

            # Validate GPU utilization
            if gpu_monitor:
                gpu_monitor.stop()
                gpu_monitor.log_summary()
                gpu_monitor.assert_thresholds(thresholds)

        except AssertionError:
            # Log genai-bench output when results not found
            logger.error(
                "genai-bench output (returncode=%d):\nstdout:\n%s\nstderr:\n%s",
                proc.returncode,
                stdout or "(empty)",
                stderr or "(empty)",
            )
            raise

        finally:
            _cleanup_procs(kill_procs, drain_delay_sec)
            if gpu_monitor:
                gpu_monitor.stop(timeout=2)

    return _run
```
**EN:** This pytest fixture prepares `genai_bench_runner` so multiple tests can reuse the same setup or cleanup sequence.
**CN:** 这个 pytest 夹具为 `genai_bench_runner` 准备共享的初始化或清理流程，方便多个测试复用。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Performance measurement and comparison / 性能测量与对比
- Reusable functions: `_build_command`, `_find_results`, `_cleanup_procs`, `genai_bench_runner` / 可复用函数：`_build_command`, `_find_results`, `_cleanup_procs`, `genai_bench_runner`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `os`, `pathlib`, `shutil`, `subprocess`, `time`
- **Third-party / 第三方**: `pytest`
- **Internal / 内部模块**: `.results`, `infra`
