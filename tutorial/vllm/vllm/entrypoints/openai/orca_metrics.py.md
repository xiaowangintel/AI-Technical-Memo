# orca_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/orca_metrics.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Tracks ORCA-related metrics exposed by the API layer. Scope: OpenAI-compatible. / 跟踪 API 层暴露的 ORCA 相关指标。 作用域：OpenAI 兼容。

## Line-by-Line Analysis / 逐行分析
### Lines 3-5 — Module overview
```python
"""
Utility functions that create ORCA endpoint load report response headers.
"""
```
**EN:** The module header documents the main contract: Utility functions that create ORCA endpoint load report response headers.
**CN:** 模块开头的文档字符串说明了该文件的主要职责、使用边界或运行约束。

### Lines 7-11 — Imports and shared dependencies
```python
import json
from collections.abc import Mapping

from vllm.logger import init_logger
from vllm.v1.metrics.reader import Gauge, get_metrics_snapshot
```
**EN:** This import block pulls in standard-library modules such as `json`, `collections`, depends on internal helpers such as `vllm.logger`, `vllm.v1.metrics.reader`.
**CN:** 该导入块引入 `json`, `collections` 等标准库模块，依赖 `vllm.logger`, `vllm.v1.metrics.reader` 等 vLLM 内部模块。

### Lines 13-13 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 16-70 — Function `create_orca_header`
```python
def create_orca_header(
    metrics_format: str, named_metrics: list[tuple[str, float]]
) -> Mapping[str, str] | None:
    """
    Creates ORCA headers named 'endpoint-load-metrics' in the specified format
    and adds custom metrics to named_metrics.
    ORCA headers format description: https://docs.google.com/document/d/1C1ybMmDKJIVlrbOLbywhu9iRYo4rilR-cT50OTtOFTs/edit?tab=t.0
    ORCA proto https://github.com/cncf/xds/blob/main/xds/data/orca/v3/orca_load_report.proto

    Parameters:
    - metrics_format (str): The format of the header ('TEXT', 'JSON').
    - named_metrics (List[Tuple[str, float]]): List of tuples with metric names
    and their corresponding double values.

    Returns:
    - Optional[Mapping[str,str]]: A dictionary with header key as
    'endpoint-load-metrics' and values as the ORCA header strings with
    format prefix and data in  with named_metrics in.
    """

    if metrics_format.lower() not in ["text", "json"]:
        logger.warning(
            "Warning: `%s` format is not supported in the ORCA response header",
            format,
        )
        return None

    header = {}
...
    # output example:
    # endpoint-load-metrics: JSON “named_metrics”: {“custom-metric-util”: 0.4}
    elif metrics_format.lower() == "json":
        header["endpoint-load-metrics"] = f"JSON {json.dumps(orca_report)}"

    logger.info("Created ORCA header %s", header)

    return header
```
**EN:** This function `create_orca_header` is documented as: Creates ORCA headers named 'endpoint-load-metrics' in the specified format and adds custom metrics to named_metrics.
**CN:** 这里定义函数 `create_orca_header`，其文档字符串说明了主要职责与调用约定。

### Lines 73-97 — Function `get_named_metrics_from_prometheus`
```python
def get_named_metrics_from_prometheus() -> list[tuple[str, float]]:
    """
    Collects current metrics from Prometheus and returns some of them
    in the form of the `named_metrics` list for `create_orca_header()`.

    Parameters:
    - None

    Returns:
    - list[tuple[str, float]]: List of tuples of metric names and their values.
    """
    named_metrics: list[tuple[str, float]] = []
    # Map from prometheus metric names to ORCA named metrics.
    prometheus_to_orca_metrics = {
        "vllm:kv_cache_usage_perc": "kv_cache_usage_perc",
        "vllm:num_requests_waiting": "num_requests_waiting",
    }
    metrics = get_metrics_snapshot()
    for metric in metrics:
        orca_name = prometheus_to_orca_metrics.get(metric.name)
        # If this metric is mapped into ORCA, then add it to the report.
        # Note: Only Gauge metrics are currently supported.
        if orca_name is not None and isinstance(metric, Gauge):
            named_metrics.append((str(orca_name), float(metric.value)))
    return named_metrics
```
**EN:** This function `get_named_metrics_from_prometheus` is documented as: Collects current metrics from Prometheus and returns some of them in the form of the `named_metrics` list for `create_orca_header()`.
**CN:** 这里定义函数 `get_named_metrics_from_prometheus`，其文档字符串说明了主要职责与调用约定。

### Lines 100-120 — Function `metrics_header`
```python
def metrics_header(metrics_format: str) -> Mapping[str, str] | None:
    """
    Creates ORCA headers named 'endpoint-load-metrics' in the specified format.
    Metrics are collected from Prometheus using `get_named_metrics_from_prometheus()`.

    ORCA headers format description: https://docs.google.com/document/d/1C1ybMmDKJIVlrbOLbywhu9iRYo4rilR-cT50OTtOFTs/edit?tab=t.0
    ORCA proto https://github.com/cncf/xds/blob/main/xds/data/orca/v3/orca_load_report.proto

    Parameters:
    - metrics_format (str): The format of the header ('TEXT', 'JSON').

    Returns:
    - Optional[Mapping[str,str]]: A dictionary with header key as
    'endpoint-load-metrics' and values as the ORCA header strings with
    format prefix and data in  with named_metrics in.
    """
    if not metrics_format:
        return None
    # Get named metrics from prometheus.
    named_metrics = get_named_metrics_from_prometheus()
    return create_orca_header(metrics_format, named_metrics)
```
**EN:** This function `metrics_header` is documented as: Creates ORCA headers named 'endpoint-load-metrics' in the specified format.
**CN:** 这里定义函数 `metrics_header`，其文档字符串说明了主要职责与调用约定。

## Key Concepts / 关键概念
- Metrics and observability / 指标与可观测性
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections`
- **vLLM Internal / vLLM 内部**: `vllm.logger`, `vllm.v1.metrics.reader`
