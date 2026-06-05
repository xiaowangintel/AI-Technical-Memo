# utilization_stats_lib.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/utilization_stats_lib.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```python
from dataclasses import dataclass, field
from datetime import datetime

#  pyrefly: ignore [missing-import]
from dataclasses_json import DataClassJsonMixin  # type: ignore[import-not-found]
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as dataclasses, datetime; external packages such as dataclasses_json. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 dataclasses、datetime；外部依赖包，如 dataclasses_json。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 7-12
```python

_DATA_MODEL_VERSION = 1.5


# data model for test log usage
@dataclass
```
- **EN**: Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 13-18
```python
class UtilizationStats:
    avg: float | None = None
    max: float | None = None
    raw: list[float] | None = None
```
- **EN**: It introduces classes such as UtilizationStats, which package state and behavior for this tooling task. This chunk continues `UtilizationStats` and expands its internal control flow or data movement.
- **CN**: 它引入了 UtilizationStats 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `UtilizationStats`，进一步展开其内部控制流或数据流转。

### Lines 19-30
```python
@dataclass
class UtilizationMetadata(DataClassJsonMixin):  # type: ignore[misc, no-any-unimported]
    level: str
    workflow_id: str
    job_id: str
    workflow_name: str
    job_name: str
    usage_collect_interval: float
    data_model_version: float
    start_at: int
    gpu_count: int | None = None
    cpu_count: int | None = None
```
- **EN**: It introduces classes such as UtilizationMetadata, which package state and behavior for this tooling task. This chunk continues `UtilizationMetadata` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 它引入了 UtilizationMetadata 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `UtilizationMetadata`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 31-36
```python
    gpu_type: str | None = None
    error: str | None = None


@dataclass
class GpuUsage(DataClassJsonMixin):  # type: ignore[misc, no-any-unimported]
```
- **EN**: It introduces classes such as GpuUsage, which package state and behavior for this tooling task. This chunk continues `GpuUsage` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 它引入了 GpuUsage 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `GpuUsage`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 37-43
```python
    uuid: str | None = None
    util_percent: UtilizationStats | None = None
    mem_util_percent: UtilizationStats | None = None
    allocated_mem_percent: UtilizationStats | None = None
    allocated_mem_value: UtilizationStats | None = None
    total_mem_value: float | None = None
```
- **EN**: This chunk continues `GpuUsage` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `GpuUsage`，进一步展开其内部控制流或数据流转。

### Lines 44-50
```python

@dataclass
class RecordData(DataClassJsonMixin):  # type: ignore[misc, no-any-unimported]
    cpu: UtilizationStats | None = None
    memory: UtilizationStats | None = None
    gpu_usage: list[GpuUsage] | None = None
```
- **EN**: It introduces classes such as RecordData, which package state and behavior for this tooling task. This chunk continues `RecordData` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 它引入了 RecordData 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `RecordData`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 51-61
```python

@dataclass
class UtilizationRecord(DataClassJsonMixin):  # type: ignore[misc, no-any-unimported]
    level: str
    timestamp: int
    data: RecordData | None = None
    cmd_names: list[str] | None = None
    error: str | None = None
    log_duration: str | None = None
    logs: list[str] | None = None
```
- **EN**: It introduces classes such as UtilizationRecord, which package state and behavior for this tooling task. This chunk continues `UtilizationRecord` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 它引入了 UtilizationRecord 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `UtilizationRecord`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 62-72
```python

# the db schema related to this is:
# https://github.com/pytorch/test-infra/blob/main/clickhouse_db_schema/oss_ci_utilization/oss_ci_utilization_metadata_schema.sql
@dataclass
class OssCiSegmentV1(DataClassJsonMixin):  # type: ignore[misc, no-any-unimported]
    level: str
    name: str
    start_at: int
    end_at: int
    extra_info: dict[str, str]
```
- **EN**: It introduces classes such as OssCiSegmentV1, which package state and behavior for this tooling task. This chunk continues `OssCiSegmentV1` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 它引入了 OssCiSegmentV1 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `OssCiSegmentV1`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 73-84
```python

@dataclass
class OssCiUtilizationMetadataV1:
    created_at: int
    repo: str
    workflow_id: int
    run_attempt: int
    job_id: int
    workflow_name: str
    job_name: str
    usage_collect_interval: float
    data_model_version: str
```
- **EN**: It introduces classes such as OssCiUtilizationMetadataV1, which package state and behavior for this tooling task. This chunk continues `OssCiUtilizationMetadataV1` and expands its internal control flow or data movement.
- **CN**: 它引入了 OssCiUtilizationMetadataV1 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `OssCiUtilizationMetadataV1`，进一步展开其内部控制流或数据流转。

### Lines 85-92
```python
    gpu_count: int
    cpu_count: int
    gpu_type: str
    start_at: int
    end_at: int
    segments: list[OssCiSegmentV1]
    tags: list[str] = field(default_factory=list)
```
- **EN**: This chunk continues `OssCiUtilizationMetadataV1` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `OssCiUtilizationMetadataV1`，进一步展开其内部控制流或数据流转。

### Lines 93-104
```python

# this data model is for the time series data:
# https://github.com/pytorch/test-infra/blob/main/clickhouse_db_schema/oss_ci_utilization/oss_ci_time_series_schema.sql
@dataclass
class OssCiUtilizationTimeSeriesV1:
    created_at: int
    type: str
    tags: list[str]
    time_stamp: int
    repo: str
    workflow_id: int
    run_attempt: int
```
- **EN**: It introduces classes such as OssCiUtilizationTimeSeriesV1, which package state and behavior for this tooling task. This chunk continues `OssCiUtilizationTimeSeriesV1` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 它引入了 OssCiUtilizationTimeSeriesV1 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `OssCiUtilizationTimeSeriesV1`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 105-110
```python
    job_id: int
    workflow_name: str
    job_name: str
    json_data: str
```
- **EN**: This chunk continues `OssCiUtilizationTimeSeriesV1` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `OssCiUtilizationTimeSeriesV1`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 111-118
```python
def getDataModelVersion() -> float:
    return _DATA_MODEL_VERSION


def getTsNow() -> int:
    ts = datetime.now().timestamp()
    return int(ts)
```
- **EN**: This chunk defines `getTsNow`, which implements a focused step inside the ci statistics pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `getTsNow`，其作用是实现CI 统计流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 119-127
```python

@dataclass
class WorkflowInfo:
    workflow_run_id: int
    workflow_name: str
    job_id: int
    run_attempt: int
    job_name: str
    repo: str = "pytorch/pytorch"
```
- **EN**: It introduces classes such as WorkflowInfo, which package state and behavior for this tooling task. This chunk continues `WorkflowInfo` and expands its internal control flow or data movement.
- **CN**: 它引入了 WorkflowInfo 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `WorkflowInfo`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **CI statistics**
  - EN: This file belongs to the ci statistics layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于CI 统计层，应结合同一子目录中的相邻脚本一起理解。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **_DATA_MODEL_VERSION**
  - EN: `_DATA_MODEL_VERSION` is one of the main local symbols exposed or implemented here.
  - CN: `_DATA_MODEL_VERSION` 是此处暴露或实现的主要局部符号之一。
- **UtilizationStats**
  - EN: `UtilizationStats` is one of the main local symbols exposed or implemented here.
  - CN: `UtilizationStats` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `dataclasses`, `datetime`
- **External packages / 外部依赖包**: `dataclasses_json`
- **Primary symbols in this file / 本文件核心符号**: `_DATA_MODEL_VERSION`, `UtilizationStats`, `UtilizationMetadata`, `GpuUsage`, `RecordData`, `UtilizationRecord`, `OssCiSegmentV1`, `OssCiUtilizationMetadataV1`, `OssCiUtilizationTimeSeriesV1`, `getDataModelVersion`, `getTsNow`, `WorkflowInfo`
