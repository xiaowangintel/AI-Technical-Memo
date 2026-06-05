# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/conftest.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This pytest configuration module defines shared fixtures, hooks, and markers for the end-to-end test suite. It centralizes reusable test setup so individual test files stay focused on assertions. / 该 pytest 配置模块为 端到端测试 定义共享夹具、钩子与标记，将可复用的测试初始化集中管理，使各测试文件能专注于断言。

## Line-by-Line Analysis / 逐行分析
### Lines 1-107: Module docstring
```python
"""Pytest configuration for E2E tests.

Tests run serially under plain pytest. ModelPool / GPUAllocator stay
thread-safe so re-introducing parallelism (e.g. via pytest-xdist) is
still a tractable option; pytest-parallel was previously used but its
thread dispatch leaked fixture references and caused model_pool
deadlocks. Tests marked ``@pytest.mark.thread_unsafe`` would be
auto-skipped in any future parallel mode.

Markers
-------
This module defines several pytest markers for configuring E2E tests:

@pytest.mark.model(name)
    Specify which model to use for the test.

    Args:
        name: Model ID from MODEL_SPECS (e.g., "llama-8b", "qwen-7b")

    GPU Resource Management:
        When GPUs are limited (e.g., 4 GPUs, 6 models), the model pool uses
        MRU (Most Recently Used) eviction:
        1. Models are pre-launched until GPUs are full
        2. When a test needs a model that isn't running, MRU model is evicted
           (models just used are likely done, models not yet used are waiting)
        3. The needed model is then launched on-demand

    Examples:
        @pytest.mark.model("llama-8b")
        @pytest.mark.model("qwen-72b")

@pytest.mark.workers(count=1, prefill=None, decode=None)
    Configure worker topology for the test.

    Args:
        count: Number of regular workers (default: 1)
        prefill: Number of prefill workers for PD disaggregation
        decode: Number of decode workers for PD disaggregation

    Examples:
        @pytest.mark.workers(count=3)  # 3 regular workers
        @pytest.mark.workers(prefill=2, decode=2)  # PD mode

@pytest.mark.gateway(policy="round_robin", timeout=None, extra_args=None)
    Configure the gateway/router.

    Args:
        policy: Routing policy ("round_robin", "random", etc.)
        timeout: Startup timeout in seconds
        extra_args: Additional CLI arguments for the router

    Examples:
        @pytest.mark.gateway(policy="random")
        @pytest.mark.gateway(extra_args=["--cache-routing"])

@pytest.mark.e2e
    Mark test as an end-to-end test requiring GPU workers.

@pytest.mark.slow
    Mark test as slow-running.

@pytest.mark.thread_unsafe(reason=None)
    Mark test as incompatible with parallel thread execution.
    Tests with this marker are automatically skipped when running
    with --tests-per-worker > 1.

    Args:
        reason: Optional explanation of why the test is thread-unsafe.

    Examples:
        @pytest.mark.thread_unsafe
        @pytest.mark.thread_unsafe(reason="Modifies global state")

Fixtures
--------
model_pool: Session-scoped fixture managing SGLang worker processes.
setup_backend: Class-scoped fixture that launches gateway + provides client.

Usage Examples
--------------
Basic test with default model:

    @pytest.mark.e2e
    @pytest.mark.parametrize("setup_backend", ["http"], indirect=True)
    class TestBasic:
        def test_chat(self, setup_backend):
            backend, model, client, gateway = setup_backend
            response = client.chat.completions.create(...)

Test with specific model and multiple backends:

    @pytest.mark.e2e
    @pytest.mark.model("qwen-7b")
    @pytest.mark.parametrize("setup_backend", ["grpc", "http"], indirect=True)
    class TestQwen:
        def test_generate(self, setup_backend):
            ...

PD disaggregation mode:

    @pytest.mark.e2e
    @pytest.mark.workers(prefill=1, decode=1)
    @pytest.mark.parametrize("setup_backend", ["pd"], indirect=True)
    class TestPD:
        def test_pd_inference(self, setup_backend):
            ...
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 109-114: Imports and dependencies
```python
from __future__ import annotations

import logging
import sys
from importlib.util import find_spec
from pathlib import Path
```
**EN:** This block imports `__future__`, `logging`, `sys`, `importlib.util`, and 1 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 115-122: Module constants and configuration
```python

# ---------------------------------------------------------------------------
# Path setup (must happen before other imports)
# ---------------------------------------------------------------------------

_ROOT = Path(__file__).resolve().parents[1]  # sgl-model-gateway/
_E2E_TEST = Path(__file__).resolve().parent  # e2e_test/
_SRC = _ROOT / "bindings" / "python"
```
**EN:** This section defines module-level names such as `_ROOT`, `_E2E_TEST`, `_SRC`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 123-126: If block
```python

# Add e2e_test to path so "from infra import ..." works
if str(_E2E_TEST) not in sys.path:
    sys.path.insert(0, str(_E2E_TEST))
```
**EN:** This top-level block contributes supporting logic to the module and is executed or defined as part of file initialization.
**CN:** 这个顶层代码块为模块提供辅助逻辑，并在文件初始化阶段被执行或定义。

### Lines 127-134: Guarded setup and fallback handling
```python

# Add bindings/python to path if the wheel is not installed (for local development).
# find_spec raises ModuleNotFoundError when the parent package itself is absent,
# which is the case in CI jobs that don't install the sglang_router wheel.
try:
    _wheel_installed = find_spec("sglang_router.sglang_router_rs") is not None
except ModuleNotFoundError:
    _wheel_installed = False
```
**EN:** This block wraps initialization in exception handling so optional functionality can fail gracefully or expose clearer diagnostics.
**CN:** 该代码块使用异常处理包裹初始化流程，使可选功能在失败时能够优雅回退或提供更清晰的诊断信息。

### Lines 136-137: If block
```python
if not _wheel_installed and str(_SRC) not in sys.path:
    sys.path.insert(0, str(_SRC))
```
**EN:** This top-level block contributes supporting logic to the module and is executed or defined as part of file initialization.
**CN:** 这个顶层代码块为模块提供辅助逻辑，并在文件初始化阶段被执行或定义。

### Lines 138-167: Helper function `_setup_logging`
```python


# ---------------------------------------------------------------------------
# Logging setup (clean output without pytest's "---- live log ----" dividers)
# ---------------------------------------------------------------------------


def _setup_logging() -> None:
    """Configure clean logging to stdout with timestamps and thread info.

    In parallel mode (--tests-per-worker > 1), logs from different threads
    would be interleaved. Including thread name helps identify which test
    produced each log line.
    """
    # Include thread name for parallel execution readability
    # MainThread for sequential, Thread-N for parallel workers
    fmt = "%(asctime)s.%(msecs)03d [%(threadName)s] [%(name)s] %(message)s"
    datefmt = "%H:%M:%S"

    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(logging.Formatter(fmt, datefmt))

    for logger_name in ("e2e_test", "infra", "fixtures"):
        log = logging.getLogger(logger_name)
        log.setLevel(logging.INFO)
        log.addHandler(handler)
        log.propagate = False

    for logger_name in ("openai", "httpx", "httpcore", "numexpr"):
        logging.getLogger(logger_name).setLevel(logging.WARNING)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 170-170: Expr block
```python
_setup_logging()
```
**EN:** This top-level block contributes supporting logic to the module and is executed or defined as part of file initialization.
**CN:** 这个顶层代码块为模块提供辅助逻辑，并在文件初始化阶段被执行或定义。

### Lines 172-172: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 173-190: Helper function `pytest_runtest_logstart`
```python


# ---------------------------------------------------------------------------
# Test visibility hooks
# ---------------------------------------------------------------------------


def pytest_runtest_logstart(nodeid: str, location: tuple) -> None:
    """Print clear test header at start of each test."""
    import threading

    from infra import LOG_SEPARATOR_WIDTH

    test_name = nodeid.split("::")[-1] if "::" in nodeid else nodeid
    thread_name = threading.current_thread().name
    print(f"\n{'=' * LOG_SEPARATOR_WIDTH}")
    print(f"[{thread_name}] TEST: {test_name}")
    print(f"{'=' * LOG_SEPARATOR_WIDTH}")
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 191-209: Imports and dependencies
```python


# ---------------------------------------------------------------------------
# Import pytest hooks and fixtures from fixtures/ package
# ---------------------------------------------------------------------------

# Import fixtures - pytest discovers these by name
# Import hooks - pytest discovers these by name
from fixtures import (
    backend_router,
    model_base_url,
    model_client,
    model_pool,
    pytest_collection_finish,
    pytest_collection_modifyitems,
    pytest_configure,
    pytest_runtest_setup,
    setup_backend,
)
```
**EN:** This block imports `fixtures`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 210-225: Module constants and configuration
```python

# Re-export for pytest discovery
__all__ = [
    # Hooks
    "pytest_runtest_logstart",
    "pytest_collection_modifyitems",
    "pytest_collection_finish",
    "pytest_configure",
    "pytest_runtest_setup",
    # Fixtures
    "model_pool",
    "model_client",
    "model_base_url",
    "setup_backend",
    "backend_router",
]
```
**EN:** This section defines module-level names such as `__all__`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Reusable functions: `_setup_logging`, `pytest_runtest_logstart` / 可复用函数：`_setup_logging`, `pytest_runtest_logstart`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `importlib.util`, `logging`, `pathlib`, `sys`, `threading`
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: `fixtures`, `infra`
