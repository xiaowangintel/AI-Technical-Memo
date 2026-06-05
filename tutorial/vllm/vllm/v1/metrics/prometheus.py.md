# prometheus.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/metrics/prometheus.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `setup_multiprocess_prometheus`, `get_prometheus_registry`, `unregister_vllm_metrics` for the V1 `metrics` subsystem. / 为 V1 的 `metrics` 子系统实现 `setup_multiprocess_prometheus`, `get_prometheus_registry`, `unregister_vllm_metrics`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import os
import tempfile

from prometheus_client import REGISTRY, CollectorRegistry, multiprocess

from vllm.logger import init_logger

logger = init_logger(__name__)

# Global temporary directory for prometheus multiprocessing
_prometheus_multiproc_dir: tempfile.TemporaryDirectory | None = None
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `_prometheus_multiproc_dir`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `_prometheus_multiproc_dir`。

### `setup_multiprocess_prometheus` function / `setup_multiprocess_prometheus` 函数
```python
def setup_multiprocess_prometheus():
    """Set up prometheus multiprocessing directory if not already configured."""
    global _prometheus_multiproc_dir

    if "PROMETHEUS_MULTIPROC_DIR" not in os.environ:
        # Make TemporaryDirectory for prometheus multiprocessing
        # Note: global TemporaryDirectory will be automatically
        # cleaned up upon exit.
        _prometheus_multiproc_dir = tempfile.TemporaryDirectory()
        os.environ["PROMETHEUS_MULTIPROC_DIR"] = _prometheus_multiproc_dir.name
        logger.debug(
            "Created PROMETHEUS_MULTIPROC_DIR at %s", _prometheus_multiproc_dir.name
        )
    else:
        logger.warning(
            "Found PROMETHEUS_MULTIPROC_DIR was set by user. "
            "This directory must be wiped between vLLM runs or "
            "you will find inaccurate metrics. Unset the variable "
            "and vLLM will properly handle cleanup."
        )
```
**EN:** This function implements `setup_multiprocess_prometheus` within the module. The docstring frames it as: Set up prometheus multiprocessing directory if not already configured. Key calls include `TemporaryDirectory`, `debug`, `warning`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `setup_multiprocess_prometheus`，其作用域位于the module。 关键调用包括 `TemporaryDirectory`, `debug`, `warning`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_prometheus_registry` function / `get_prometheus_registry` 函数
```python
def get_prometheus_registry() -> CollectorRegistry:
    """Get the appropriate prometheus registry based on multiprocessing
    configuration.

    Returns:
        Registry: A prometheus registry
    """
    if os.getenv("PROMETHEUS_MULTIPROC_DIR") is not None:
        logger.debug("Using multiprocess registry for prometheus metrics")
        registry = CollectorRegistry()
        multiprocess.MultiProcessCollector(registry)
        return registry

    return REGISTRY
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Get the appropriate prometheus registry based on multiprocessing configuration. Key calls include `getenv`, `debug`, `CollectorRegistry`, `MultiProcessCollector`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `getenv`, `debug`, `CollectorRegistry`, `MultiProcessCollector`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `unregister_vllm_metrics` function / `unregister_vllm_metrics` 函数
```python
def unregister_vllm_metrics():
    """Unregister any existing vLLM collectors from the prometheus registry.

    This is useful for testing and CI/CD where metrics may be registered
    multiple times across test runs.

    Also, in case of multiprocess, we need to unregister the metrics from the
    global registry.
    """
    registry = REGISTRY
    # Unregister any existing vLLM collectors
    for collector in list(registry._collector_to_names):
        if hasattr(collector, "_name") and "vllm" in collector._name:
            registry.unregister(collector)
```
**EN:** This function implements `unregister_vllm_metrics` within the module. The docstring frames it as: Unregister any existing vLLM collectors from the prometheus registry. Key calls include `list`, `hasattr`, `unregister`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `unregister_vllm_metrics`，其作用域位于the module。 关键调用包括 `list`, `hasattr`, `unregister`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `shutdown_prometheus` function / `shutdown_prometheus` 函数
```python
def shutdown_prometheus():
    """Shutdown prometheus metrics."""

    path = _prometheus_multiproc_dir
    if path is None:
        return
    try:
        pid = os.getpid()
        multiprocess.mark_process_dead(pid, path)
        logger.debug("Marked Prometheus metrics for process %d as dead", pid)
    except Exception as e:
        logger.error("Error during metrics cleanup: %s", str(e))
```
**EN:** This function implements `shutdown_prometheus` within the module. The docstring frames it as: Shutdown prometheus metrics. Key calls include `getpid`, `mark_process_dead`, `debug`, `error`, `str`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `shutdown_prometheus`，其作用域位于the module。 关键调用包括 `getpid`, `mark_process_dead`, `debug`, `error`, `str`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `setup_multiprocess_prometheus`: top-level helper or orchestration entry point. / `setup_multiprocess_prometheus`：顶层辅助函数或编排入口。
- `get_prometheus_registry`: top-level helper or orchestration entry point. / `get_prometheus_registry`：顶层辅助函数或编排入口。
- `unregister_vllm_metrics`: top-level helper or orchestration entry point. / `unregister_vllm_metrics`：顶层辅助函数或编排入口。
- `shutdown_prometheus`: top-level helper or orchestration entry point. / `shutdown_prometheus`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `os`, `tempfile`
- External / 外部依赖: `prometheus_client`
- Internal vLLM / 内部依赖: `vllm.logger`
