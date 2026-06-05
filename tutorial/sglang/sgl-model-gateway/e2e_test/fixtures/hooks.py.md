# hooks.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/fixtures/hooks.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises hooks behavior in the end-to-end fixture layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试夹具 中与 hooks 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module docstring
```python
"""Pytest hooks for E2E test collection and validation.

This module handles:
- Test collection: Scanning markers to determine required workers
- GPU validation: Ensuring sufficient GPUs for test requirements
- Marker registration: Defining custom pytest markers
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 9-15: Imports and dependencies
```python
from __future__ import annotations

import logging
import os
from typing import TYPE_CHECKING

import pytest
```
**EN:** This block imports `__future__`, `logging`, `os`, `typing`, and 1 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 17-18: If block
```python
if TYPE_CHECKING:
    from infra import ConnectionMode, WorkerIdentity, WorkerType
```
**EN:** This top-level block contributes supporting logic to the module and is executed or defined as part of file initialization.
**CN:** 这个顶层代码块为模块提供辅助逻辑，并在文件初始化阶段被执行或定义。

### Lines 20-36: Module constants and configuration
```python
logger = logging.getLogger(__name__)

# ---------------------------------------------------------------------------
# Test collection state
# ---------------------------------------------------------------------------

# Track max worker counts: (model_id, mode, worker_type) -> max_count
_worker_counts: dict[tuple["ConnectionMode", "WorkerType"], int] = {}

# Track first-seen order to preserve test collection order
_first_seen_order: list[tuple[str, "ConnectionMode", "WorkerType"]] = []

# Track max GPU requirement for any single test (for validation)
_max_test_gpu_requirement: int = 0
_max_test_name: str = ""

_needs_default_model: bool = False
```
**EN:** This section defines module-level names such as `logger`, `_worker_counts`, `_first_seen_order`, `_max_test_gpu_requirement`, and 2 more, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 39-47: Helper function `reset_collection_state`
```python
def reset_collection_state() -> None:
    """Reset collection state (useful for testing)."""
    global _worker_counts, _first_seen_order
    global _max_test_gpu_requirement, _max_test_name, _needs_default_model
    _worker_counts = {}
    _first_seen_order = []
    _max_test_gpu_requirement = 0
    _max_test_name = ""
    _needs_default_model = False
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 50-52: Helper function `get_worker_counts`
```python
def get_worker_counts() -> dict:
    """Get the worker counts dictionary."""
    return _worker_counts
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 55-57: Helper function `get_first_seen_order`
```python
def get_first_seen_order() -> list:
    """Get the first-seen order list."""
    return _first_seen_order
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 60-62: Helper function `get_max_gpu_requirement`
```python
def get_max_gpu_requirement() -> tuple[int, str]:
    """Get the max GPU requirement and test name."""
    return _max_test_gpu_requirement, _max_test_name
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 65-67: Helper function `needs_default_model`
```python
def needs_default_model() -> bool:
    """Check if any test needs the default model."""
    return _needs_default_model
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 68-256: Helper function `pytest_collection_modifyitems`
```python


# ---------------------------------------------------------------------------
# Test collection hook
# ---------------------------------------------------------------------------


def pytest_collection_modifyitems(
    session: pytest.Session,
    config: pytest.Config,
    items: list[pytest.Item],
) -> None:
    """Scan collected tests to determine required workers.

    This runs after test collection but before tests execute.
    It extracts worker requirements from markers in test collection order,
    tracking the max count needed for each (model, mode, worker_type) combination.
    """
    global _worker_counts, _first_seen_order, _needs_default_model
    global _max_test_gpu_requirement, _max_test_name

    from infra import (
        DEFAULT_MODEL,
        MODEL_SPECS,
        PARAM_MODEL,
        PARAM_SETUP_BACKEND,
        ConnectionMode,
        WorkerType,
    )

    available_gpus = _count_gpus_without_cuda()

    def track_worker(
        model_id: str, mode: ConnectionMode, worker_type: WorkerType, count: int
    ) -> None:
        """Track a worker requirement, updating max count if needed."""
        key = (model_id, mode, worker_type)
        if key not in _worker_counts:
            _first_seen_order.append(key)
            _worker_counts[key] = count
        else:
            _worker_counts[key] = max(_worker_counts[key], count)

    def calculate_test_gpus(
        model_id: str, prefill: int, decode: int, regular: int
    ) -> int:
        """Calculate GPU requirement for a single test."""
        if model_id not in MODEL_SPECS:
            return 0
        tp = MODEL_SPECS[model_id].get("tp", 1)
        return tp * (prefill + decode + regular)

    for item in items:
        # Extract model from marker or use default
        # First check the class directly (handles inheritance correctly)
        model_id = None
        if hasattr(item, "cls") and item.cls is not None:
            for marker in (
                item.cls.pytestmark if hasattr(item.cls, "pytestmark") else []
            ):
                if marker.name == PARAM_MODEL and marker.args:
                    model_id = marker.args[0]
                    break
        # Fall back to get_closest_marker for method-level markers
        if model_id is None:
            model_marker = item.get_closest_marker(PARAM_MODEL)
            model_id = (
                model_marker.args[0] if model_marker and model_marker.args else None
            )

        # Check parametrize for model
        if model_id is None:
            for marker in item.iter_markers("parametrize"):
                if marker.args and len(marker.args) >= 2:
                    param_name = marker.args[0]
                    if param_name == PARAM_MODEL or PARAM_MODEL in param_name:
                        param_values = marker.args[1]
                        if isinstance(param_values, (list, tuple)) and param_values:
                            model_id = param_values[0]
                        break

        # Extract backends from parametrize
        backends: list[str] = []
        for marker in item.iter_markers("parametrize"):
            if marker.args and len(marker.args) >= 2:
                param_name = marker.args[0]
                param_values = marker.args[1]
                if param_name == PARAM_SETUP_BACKEND:
                    if isinstance(param_values, (list, tuple)):
                        backends.extend(param_values)

        # Check for workers marker
        workers_marker = item.get_closest_marker("workers")
        prefill_count = 0
        decode_count = 0
        regular_count = 1
        if workers_marker:
            prefill_count = workers_marker.kwargs.get("prefill") or 0
            decode_count = workers_marker.kwargs.get("decode") or 0
            regular_count = workers_marker.kwargs.get("count") or 1

        # Track if this test needs default model
        is_e2e = item.get_closest_marker("e2e") is not None
        if model_id is None and is_e2e:
            _needs_default_model = True
            model_id = DEFAULT_MODEL

        # Track worker requirements
        test_gpus = 0
        if model_id and backends:
            for backend in backends:
                if backend == "pd":
                    mode = ConnectionMode.HTTP
                    p_count = prefill_count if prefill_count > 0 else 1
                    d_count = decode_count if decode_count > 0 else 1
                    track_worker(model_id, mode, WorkerType.PREFILL, p_count)
                    track_worker(model_id, mode, WorkerType.DECODE, d_count)
                    test_gpus = max(
                        test_gpus, calculate_test_gpus(model_id, p_count, d_count, 0)
                    )
                else:
                    try:
                        mode = ConnectionMode(backend)
                    except ValueError:
                        continue

                    if prefill_count > 0 or decode_count > 0:
                        track_worker(model_id, mode, WorkerType.PREFILL, prefill_count)
                        track_worker(model_id, mode, WorkerType.DECODE, decode_count)
                        test_gpus = max(
                            test_gpus,
                            calculate_test_gpus(
                                model_id, prefill_count, decode_count, 0
                            ),
                        )
                    else:
                        track_worker(model_id, mode, WorkerType.REGULAR, regular_count)
                        test_gpus = max(
                            test_gpus,
                            calculate_test_gpus(model_id, 0, 0, regular_count),
                        )

        elif model_id and is_e2e:
            track_worker(model_id, ConnectionMode.HTTP, WorkerType.REGULAR, 1)
            test_gpus = calculate_test_gpus(model_id, 0, 0, 1)

        if test_gpus > _max_test_gpu_requirement:
            _max_test_gpu_requirement = test_gpus
            _max_test_name = item.nodeid

        # Mark over-capacity tests as skipped (including when available_gpus
        # is 0) so pytest_collection_finish can detect the all-skipped case
        # and fail loudly instead of passing green with zero tests run.
        if test_gpus > available_gpus:
            item.add_marker(
                pytest.mark.skip(
                    reason=(
                        f"requires {test_gpus} GPUs (model={model_id}, "
                        f"tp={MODEL_SPECS.get(model_id, {}).get('tp', 1)}); "
                        f"only {available_gpus} available on this runner"
                    )
                )
            )

    # Prune workers that can never launch on this runner.
    for key in list(_worker_counts.keys()):
        spec = MODEL_SPECS.get(key[0], {})
        if spec.get("tp", 1) > available_gpus:
            del _worker_counts[key]
    _first_seen_order[:] = [k for k in _first_seen_order if k in _worker_counts]

    # Log results
    if _worker_counts:
        summary = []
        for key in _first_seen_order:
            model_id, mode, worker_type = key
            count = _worker_counts[key]
            if worker_type == WorkerType.REGULAR:
                summary.append(f"{model_id}:{mode.value}x{count}")
            else:
                summary.append(f"{model_id}:{mode.value}:{worker_type.value}x{count}")
        logger.info("Scanned worker requirements (in test order): %s", summary)
        logger.info(
            "Max GPU requirement for single test: %d (%s)",
            _max_test_gpu_requirement,
            _max_test_name,
        )
    else:
        logger.info("Scanned worker requirements: (none)")
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 257-299: Helper function `get_pool_requirements`
```python


# ---------------------------------------------------------------------------
# Pool requirements
# ---------------------------------------------------------------------------


def get_pool_requirements() -> list["WorkerIdentity"]:
    """Build pool requirements from scanned test markers.

    Returns:
        List of WorkerIdentity objects to pre-launch.
    """
    from infra import DEFAULT_MODEL, ConnectionMode, WorkerIdentity, WorkerType

    # Track which models have PD workers as their first requirement
    models_with_pd_first: set[str] = set()
    first_worker_type_per_model: dict[str, WorkerType] = {}

    for model_id, mode, worker_type in _first_seen_order:
        if model_id not in first_worker_type_per_model:
            first_worker_type_per_model[model_id] = worker_type
            if worker_type in (WorkerType.PREFILL, WorkerType.DECODE):
                models_with_pd_first.add(model_id)
                logger.info(
                    "Model %s has PD test first - skipping regular worker pre-launch",
                    model_id,
                )

    # Generate individual WorkerIdentity objects in first-seen order
    requirements: list[WorkerIdentity] = []
    for model_id, mode, worker_type in _first_seen_order:
        if model_id in models_with_pd_first and worker_type == WorkerType.REGULAR:
            continue

        count = _worker_counts.get((model_id, mode, worker_type), 1)
        for i in range(count):
            requirements.append(WorkerIdentity(model_id, mode, worker_type, i))

    if not requirements:
        requirements.append(WorkerIdentity(DEFAULT_MODEL, ConnectionMode.HTTP))

    return requirements
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 300-357: Helper function `_count_gpus_without_cuda`
```python


# ---------------------------------------------------------------------------
# GPU validation
# ---------------------------------------------------------------------------


def _count_gpus_without_cuda() -> int:
    """Count available GPUs without initializing CUDA.

    Must avoid CUDA initialization because pytest_collection_modifyitems
    runs before pytest-parallel forks workers, and CUDA cannot be
    re-initialized after fork.

    Honors CUDA_VISIBLE_DEVICES first — container runners commonly expose
    all host GPUs to the container (e.g. NVIDIA_VISIBLE_DEVICES=all) and
    gate per-process visibility via CUDA_VISIBLE_DEVICES, so nvidia-smi
    would over-report. Falls back to nvidia-smi only when the env var is
    unset, and logs (rather than swallows) any nvidia-smi failure so a
    misconfigured runner is debuggable from CI logs.
    """
    cvd = os.environ.get("CUDA_VISIBLE_DEVICES")
    if cvd is not None:
        # CUDA treats "-1" as "no devices"; don't count it as one.
        return len([d for d in cvd.split(",") if d.strip() and d.strip() != "-1"])

    import subprocess

    try:
        result = subprocess.run(
            ["nvidia-smi", "--query-gpu=name", "--format=csv,noheader"],
            capture_output=True,
            text=True,
            timeout=10,
        )
    except FileNotFoundError:
        logger.error(
            "nvidia-smi not found and CUDA_VISIBLE_DEVICES is unset; "
            "cannot determine GPU count, treating as 0"
        )
        return 0
    except (subprocess.SubprocessError, OSError) as e:
        logger.error(
            "nvidia-smi failed (%s); cannot determine GPU count, treating as 0",
            e,
            exc_info=True,
        )
        return 0

    if result.returncode != 0:
        logger.error(
            "nvidia-smi exited with code %d; treating as 0 GPUs. stderr=%r stdout=%r",
            result.returncode,
            result.stderr,
            result.stdout,
        )
        return 0
    return len([line for line in result.stdout.strip().split("\n") if line])
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 360-370: Helper function `validate_gpu_requirements`
```python
def validate_gpu_requirements() -> tuple[int, int]:
    """Check if there are enough GPUs for any single test.

    Uses nvidia-smi instead of torch.cuda to avoid CUDA initialization,
    which would break pytest-parallel (CUDA cannot be re-initialized after fork).

    Returns:
        Tuple of (max_required_gpus, available_gpus).
    """
    available_gpus = _count_gpus_without_cuda()
    return _max_test_gpu_requirement, available_gpus
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 373-419: Helper function `pytest_collection_finish`
```python
def pytest_collection_finish(session: pytest.Session) -> None:
    """Validate GPU requirements after test collection."""
    from infra import ENV_SKIP_MODEL_POOL

    # _max_test_gpu_requirement survives pruning; _worker_counts may be
    # emptied above when no test fits, and we still want the loud-fail.
    if _max_test_gpu_requirement == 0:
        return

    if os.environ.get(ENV_SKIP_MODEL_POOL, "").lower() in ("1", "true", "yes"):
        return

    max_required, available_gpus = validate_gpu_requirements()

    if max_required > available_gpus:
        # Tests whose individual GPU need exceeds capacity are already skipped
        # in pytest_collection_modifyitems. If literally every collected test
        # was skipped this way, refuse to pass green — that's the runner-
        # mismatch case that should fail loud (e.g. wrong matrix entry,
        # nvidia-smi returning 0 on a healthy host).
        non_skipped = [
            it
            for it in session.items
            if not any(m.name == "skip" for m in it.iter_markers())
        ]
        if not non_skipped:
            raise pytest.UsageError(
                f"Runner has {available_gpus} GPU(s); every collected test "
                f"requires more (largest: {_max_test_name} needs {max_required}). "
                f"Zero tests would run — refusing to pass silently."
            )
        # Otherwise: surface the gap so it's obvious in logs that this runner
        # only ran the fitting subset.
        logger.warning(
            "Runner has %d GPU(s); skipped tests requiring up to %d (largest: %s)",
            available_gpus,
            max_required,
            _max_test_name,
        )
        return

    logger.info(
        "GPU validation passed: max %d required (by %s), %d available",
        max_required,
        _max_test_name,
        available_gpus,
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 420-464: Helper function `pytest_configure`
```python


# ---------------------------------------------------------------------------
# Marker registration
# ---------------------------------------------------------------------------


def pytest_configure(config: pytest.Config) -> None:
    """Register custom markers."""
    config.addinivalue_line(
        "markers",
        "model(name): mark test to use a specific model from MODEL_SPECS",
    )
    config.addinivalue_line(
        "markers",
        "backend(name): mark test to use a specific backend (grpc, http, openai, etc.)",
    )
    config.addinivalue_line(
        "markers",
        "workers(count=1, prefill=None, decode=None): "
        "worker configuration - use count for regular workers, "
        "or prefill/decode for PD disaggregation mode",
    )
    config.addinivalue_line(
        "markers",
        "gateway(policy='round_robin', timeout=None, extra_args=None): "
        "gateway/router configuration",
    )
    config.addinivalue_line(
        "markers",
        "e2e: mark test as an end-to-end test requiring GPU workers",
    )
    config.addinivalue_line(
        "markers",
        "slow: mark test as slow-running",
    )
    config.addinivalue_line(
        "markers",
        "thread_unsafe: mark test as incompatible with parallel thread execution",
    )
    config.addinivalue_line(
        "markers",
        "storage(backend): mark test to use a specific history storage backend "
        "(memory, oracle). Default is memory.",
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 465-488: Helper function `is_parallel_execution`
```python


# ---------------------------------------------------------------------------
# Parallel execution support
# ---------------------------------------------------------------------------


def is_parallel_execution(config: pytest.Config) -> bool:
    """Check if tests are running in parallel mode (pytest-parallel).

    Returns True if --tests-per-worker > 1, indicating concurrent thread execution.
    """
    # pytest-parallel adds the 'tests_per_worker' option
    tests_per_worker = getattr(config.option, "tests_per_worker", None)
    if tests_per_worker is None:
        return False

    if tests_per_worker == "auto":
        return True

    try:
        return int(tests_per_worker) > 1
    except (ValueError, TypeError):
        return False
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 491-497: Helper function `pytest_runtest_setup`
```python
def pytest_runtest_setup(item: pytest.Item) -> None:
    """Skip thread_unsafe tests when running in parallel mode."""
    if is_parallel_execution(item.config):
        marker = item.get_closest_marker("thread_unsafe")
        if marker:
            reason = marker.kwargs.get("reason", "Test is not thread-safe")
            pytest.skip(f"Skipping in parallel mode: {reason}")
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Reusable functions: `reset_collection_state`, `get_worker_counts`, `get_first_seen_order`, `get_max_gpu_requirement`, and 9 more / 可复用函数：`reset_collection_state`, `get_worker_counts`, `get_first_seen_order`, `get_max_gpu_requirement`, and 9 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `os`, `subprocess`, `typing`
- **Third-party / 第三方**: `pytest`
- **Internal / 内部模块**: `infra`
