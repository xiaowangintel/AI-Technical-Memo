# v1_loads.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/v1_loads.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements v1 loads logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 v1 loads 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Provide supporting module logic / 提供辅助模块逻辑
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""
/v1/loads API endpoint for comprehensive load metrics.

This module provides the /v1/loads endpoint which returns detailed scheduler
metrics for load balancing, monitoring, and capacity planning.
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 21-24: Import runtime dependencies / 导入运行时依赖
```python
import dataclasses
import time
from datetime import datetime, timezone
from typing import Optional
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 26-27: Import runtime dependencies / 导入运行时依赖
```python
from fastapi import APIRouter, Depends, HTTPException
from fastapi.responses import Response
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 29-37: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.managers.io_struct import (
    DisaggregationMetrics,
    GetLoadsReqOutput,
    LoRAMetrics,
    MemoryMetrics,
    QueueMetrics,
    SpeculativeMetrics,
)
from sglang.version import __version__
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 39-39: Provide supporting module logic / 提供辅助模块逻辑
```python
router = APIRouter()
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 41-47: Provide supporting module logic / 提供辅助模块逻辑
```python
_OPTIONAL_METRIC_SECTIONS = {
    "memory": ("memory", MemoryMetrics),
    "speculative": ("spec", SpeculativeMetrics),
    "lora": ("lora", LoRAMetrics),
    "disaggregation": ("disagg", DisaggregationMetrics),
    "queues": ("queues", QueueMetrics),
}
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 50-54: Implement get tokenizer manager / 实现get 分词器 管理器
```python
def _get_tokenizer_manager():
    """Dependency to get tokenizer_manager from global state."""
    from sglang.srt.entrypoints.http_server import get_global_state

    return get_global_state().tokenizer_manager
```
**EN:** This block implements the function `_get_tokenizer_manager()`. It focuses on Dependency to get tokenizer_manager from global state., providing reusable behavior for the v1 loads pipeline.
**CN:** 该代码块实现函数 `_get_tokenizer_manager()`。它围绕 `_get_tokenizer_manager` 所承担的 v1 loads 相关职责展开，为对应处理链路提供可复用能力。

### Lines 57-59: Implement loads dict factory / 实现loads dict factory
```python
def _loads_dict_factory(items):
    """Factory for dataclasses.asdict() that excludes None values and timestamp."""
    return {k: v for k, v in items if v is not None and k != "timestamp"}
```
**EN:** This block implements the function `_loads_dict_factory(items)`. It focuses on Factory for dataclasses.asdict() that excludes None values and timestamp., providing reusable behavior for the v1 loads pipeline.
**CN:** 该代码块实现函数 `_loads_dict_factory(items)`。它围绕 `_loads_dict_factory` 所承担的 v1 loads 相关职责展开，为对应处理链路提供可复用能力。

### Lines 62-88: Implement compute aggregate / 实现compute aggregate
```python
def _compute_aggregate(load_dicts: list) -> dict:
    """Compute aggregate metrics from load dicts."""
    if not load_dicts:
        return {
            "total_running_reqs": 0,
            "total_waiting_reqs": 0,
            "total_reqs": 0,
            "total_used_tokens": 0,
            "total_tokens": 0,
            "avg_token_usage": 0.0,
            "avg_throughput": 0.0,
            "avg_utilization": 0.0,
        }

    n = len(load_dicts)
    return {
        "total_running_reqs": sum(d["num_running_reqs"] for d in load_dicts),
        "total_waiting_reqs": sum(d["num_waiting_reqs"] for d in load_dicts),
        "total_reqs": sum(
            d["num_running_reqs"] + d["num_waiting_reqs"] for d in load_dicts
        ),
        "total_used_tokens": sum(d["num_used_tokens"] for d in load_dicts),
        "total_tokens": sum(d["num_total_tokens"] for d in load_dicts),
        "avg_token_usage": round(sum(d["token_usage"] for d in load_dicts) / n, 4),
        "avg_throughput": round(sum(d["gen_throughput"] for d in load_dicts) / n, 2),
        "avg_utilization": round(sum(d["utilization"] for d in load_dicts) / n, 4),
    }
```
**EN:** This block implements the function `_compute_aggregate(load_dicts)`. It focuses on Compute aggregate metrics from load dicts., providing reusable behavior for the v1 loads pipeline.
**CN:** 该代码块实现函数 `_compute_aggregate(load_dicts)`。它围绕 `_compute_aggregate` 所承担的 v1 loads 相关职责展开，为对应处理链路提供可复用能力。

### Lines 91-132: Implement format loads prometheus / 实现format loads prometheus
```python
def _format_loads_prometheus(load_results) -> Response:
    """Format load metrics in Prometheus text exposition format.

    Metrics are derived from dataclass field metadata, providing a single source of truth.
    """
    lines = []

    for f in dataclasses.fields(GetLoadsReqOutput):
        if "metric" not in f.metadata:
            continue
        metric_type, description = f.metadata["metric"]
        metric_name = f"sglang_{f.name}"
        lines.append(f"# HELP {metric_name} {description}")
        lines.append(f"# TYPE {metric_name} {metric_type}")
        for load in load_results:
            value = getattr(load, f.name, None)
            if value is not None:
                lines.append(f'{metric_name}{{dp_rank="{load.dp_rank}"}} {value}')

    for attr_name, (prefix, dataclass_type) in _OPTIONAL_METRIC_SECTIONS.items():
        if not any(getattr(load, attr_name, None) for load in load_results):
            continue
        for f in dataclasses.fields(dataclass_type):
            if "metric" not in f.metadata:
                continue
            metric_type, description = f.metadata["metric"]
            metric_name = f"sglang_{prefix}_{f.name}"
            lines.append(f"# HELP {metric_name} {description}")
            lines.append(f"# TYPE {metric_name} {metric_type}")
            for load in load_results:
                section = getattr(load, attr_name, None)
                if section:
                    value = getattr(section, f.name, None)
                    if value is not None:
                        lines.append(
                            f'{metric_name}{{dp_rank="{load.dp_rank}"}} {value}'
                        )

    return Response(
        content="\n".join(lines) + "\n",
        media_type="text/plain; version=0.0.4; charset=utf-8",
    )
```
**EN:** This block implements the function `_format_loads_prometheus(load_results)`. It focuses on Format load metrics in Prometheus text exposition format., providing reusable behavior for the v1 loads pipeline.
**CN:** 该代码块实现函数 `_format_loads_prometheus(load_results)`。它围绕 `_format_loads_prometheus` 所承担的 v1 loads 相关职责展开，为对应处理链路提供可复用能力。

### Lines 133-135: Provide supporting module logic / 提供辅助模块逻辑
```python


@router.get("/v1/loads")
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 136-171: Implement async get loads / 实现异步get loads
```python
async def get_loads(
    dp_rank: Optional[int] = None,
    include: Optional[str] = None,
    format: Optional[str] = None,
    tokenizer_manager=Depends(_get_tokenizer_manager),
):
    """
    Get comprehensive load metrics for all DP ranks.

    Query Parameters:
        dp_rank: Filter to specific DP rank (optional)
        include: Comma-separated sections to include (optional)
                 Options: core, memory, spec, lora, disagg, queues, all
                 Default: all
        format: Response format - 'json' (default) or 'prometheus'

    Returns:
        JSON response with timestamp, version, dp_rank_count, per-DP-rank loads, and aggregates
    """
    include_list = [s.strip() for s in include.split(",")] if include else None

    start = time.perf_counter()
    try:
        load_results = await tokenizer_manager.get_loads(
            include=include_list,
            dp_rank=dp_rank,
        )
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
    finally:
        mc = getattr(tokenizer_manager, "metrics_collector", None)
        if mc is not None:
            mc.get_loads_duration_seconds.labels(**mc.labels).observe(
                time.perf_counter() - start
            )
```
**EN:** This block implements the async function `get_loads(dp_rank, include, format, tokenizer_manager)`. It focuses on Get comprehensive load metrics for all DP ranks., providing reusable behavior for the v1 loads pipeline.
**CN:** 该代码块实现异步函数 `get_loads(dp_rank, include, format, tokenizer_manager)`。它围绕 `get_loads` 所承担的 v1 loads 相关职责展开，为对应处理链路提供可复用能力。

### Lines 172-187: Continue async get loads / 继续说明异步get loads
```python
    if format == "prometheus":
        return _format_loads_prometheus(load_results)

    loads = []
    for load in load_results:
        d = dataclasses.asdict(load, dict_factory=_loads_dict_factory)
        d["num_total_reqs"] = d["num_running_reqs"] + d["num_waiting_reqs"]
        loads.append(d)

    return {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "version": __version__,
        "dp_rank_count": len(loads),
        "loads": loads,
        "aggregate": _compute_aggregate(loads),
    }
```
**EN:** This block implements the async function `get_loads(dp_rank, include, format, tokenizer_manager)`. It focuses on Get comprehensive load metrics for all DP ranks., providing reusable behavior for the v1 loads pipeline.
**CN:** 该代码块实现异步函数 `get_loads(dp_rank, include, format, tokenizer_manager)`。它围绕 `get_loads` 所承担的 v1 loads 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Main callables / 主要可调用对象**: _get_tokenizer_manager, _loads_dict_factory, _compute_aggregate, _format_loads_prometheus, get_loads
- **Domain focus / 领域焦点**: v1 loads / v1 loads
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, time, typing
- **Third-party / 第三方库**: datetime, fastapi, fastapi.responses
- **Local Modules / 本地模块**: sglang.srt.entrypoints.http_server, sglang.srt.managers.io_struct, sglang.version
