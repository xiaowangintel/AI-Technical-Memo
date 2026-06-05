# upload_utilization_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/upload_utilization_stats/upload_utilization_stats.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```python
#!/usr/bin/env python3

import os
import sys
from pathlib import Path


sys.path.insert(0, os.path.join(os.path.dirname(__file__), "..", "..", ".."))
import argparse
import json
import zipfile
from dataclasses import asdict
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as os, sys, pathlib, and 5 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 os、sys、pathlib 等共 8 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 15-27
```python
import pandas as pd  # type: ignore[import]
from tools.stats.upload_stats_lib import download_s3_artifacts, upload_to_s3
from tools.stats.utilization_stats_lib import (
    getDataModelVersion,
    getTsNow,
    OssCiSegmentV1,
    OssCiUtilizationMetadataV1,
    OssCiUtilizationTimeSeriesV1,
    UtilizationMetadata,
    UtilizationRecord,
    WorkflowInfo,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.upload_stats_lib, tools.stats.utilization_stats_lib; external packages such as pandas.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.upload_stats_lib、tools.stats.utilization_stats_lib；外部依赖包，如 pandas。

### Lines 28-41
```python

TEST_USAGE_LOG_FILENAME = "usage_log.txt"
CMD_PYTHON_LEVEL = "CMD_PYTHON"
UTILIZATION_BUCKET = "ossci-utilization"
PYTORCH_REPO = "pytorch/pytorch"
JOB_TEST_ARTIFACT_PREFIX = "logs-test"


class SegmentGenerator:
    """
    generates test segment from utilization records, currently it only generate segments on python commands level
    segment_delta_threshold is the threshold to determine if a segment is continuous or not, default is 60 seconds.
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as SegmentGenerator, which package state and behavior for this tooling task. This chunk continues `SegmentGenerator` and expands its internal control flow or data movement. Configuration constants such as TEST_USAGE_LOG_FILENAME, CMD_PYTHON_LEVEL, UTILIZATION_BUCKET, and 2 more centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 SegmentGenerator 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `SegmentGenerator`，进一步展开其内部控制流或数据流转。 TEST_USAGE_LOG_FILENAME、CMD_PYTHON_LEVEL、UTILIZATION_BUCKET 等共 5 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 42-55
```python
    def generate(
        self, records: list[UtilizationRecord], segment_delta_threshold: int = 60
    ) -> list[OssCiSegmentV1]:
        if len(records) == 0:
            return []

        cmd_col_name = "cmd"
        time_col_name = "time"

        # flatten time series with detected cmds
        df = pd.DataFrame(
            [
                {time_col_name: record.timestamp, cmd_col_name: process}
                for record in records
```
- **EN**: This chunk defines `generate`, which generates derived source text, templates, or metadata outputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `generate`，其作用是生成派生源码文本、模板或元数据输出。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 56-68
```python
                for process in (record.cmd_names or [])
            ]
        )
        df[time_col_name] = pd.to_datetime(df[time_col_name], unit="s", utc=True)

        # get unique cmd names
        unique_cmds_df = pd.DataFrame(df[cmd_col_name].unique(), columns=[cmd_col_name])

        # get all detected python cmds
        cmd_list = unique_cmds_df[
            unique_cmds_df[cmd_col_name].str.startswith("python")
        ][cmd_col_name].tolist()
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 69-90
```python
        # find segments by screening continuoues time series data
        segments: list[OssCiSegmentV1] = []
        for value in cmd_list:
            subset = df[df[cmd_col_name] == value].copy()

            continuous_segments = self._find_continuous_windows(
                segment_delta_threshold, time_col_name, subset
            )
            for row in continuous_segments:
                segment = OssCiSegmentV1(
                    level=CMD_PYTHON_LEVEL,
                    name=value,
                    start_at=int(row["start_time"].timestamp()),
                    end_at=int(row["end_time"].timestamp()),
                    extra_info={},
                )
                segments.append(segment)
        print(
            f"[Db Segments] detected pytest cmd: {len(cmd_list)}, generated segments: {len(segments)}"
        )
        return segments
```
- **EN**: This chunk continues `generate` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `generate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 91-110
```python
    def _find_continuous_windows(
        self,
        threshold: int,
        time_column_name: str,
        df: Any,  # the lintrunner keep complaining about the type of df, but it's not a problem
    ) -> list[dict[str, Any]]:
        time_threshold = pd.Timedelta(seconds=threshold)
        df = df.sort_values(by=time_column_name).reset_index(drop=True)
        df["time_diff"] = df[time_column_name].diff()
        df["segment"] = (df["time_diff"] > time_threshold).cumsum()
        segments = (
            df.groupby("segment")
            .agg(
                start_time=(time_column_name, "first"),
                end_time=(time_column_name, "last"),
            )
            .reset_index(drop=True)
        )
        return segments[["start_time", "end_time"]].to_dict(orient="records")  # type: ignore[no-any-return]
```
- **EN**: This chunk defines `_find_continuous_windows`, which implements a focused step inside the ci statistics pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_find_continuous_windows`，其作用是实现CI 统计流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 111-129
```python

class UtilizationDbConverter:
    """convert utilization log model to db model"""

    def __init__(
        self,
        info: WorkflowInfo,
        metadata: UtilizationMetadata,
        records: list[UtilizationRecord],
        segments: list[OssCiSegmentV1],
    ):
        self.metadata = metadata
        self.records = records
        self.segments = segments
        self.created_at = getTsNow()
        self.info = info
        end_time_stamp = max([record.timestamp for record in records])
        self.end_at = end_time_stamp
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as UtilizationDbConverter, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 UtilizationDbConverter 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 130-153
```python
    def convert(
        self,
    ) -> tuple[OssCiUtilizationMetadataV1, list[OssCiUtilizationTimeSeriesV1]]:
        db_metadata = self._to_oss_ci_metadata()
        timeseries = self._to_oss_ci_timeseries_list()
        return db_metadata, timeseries

    def _to_oss_ci_metadata(self) -> OssCiUtilizationMetadataV1:
        return OssCiUtilizationMetadataV1(
            repo=self.info.repo,
            workflow_id=self.info.workflow_run_id,
            run_attempt=self.info.run_attempt,
            job_id=self.info.job_id,
            workflow_name=self.info.workflow_name,
            job_name=self.info.job_name,
            usage_collect_interval=self.metadata.usage_collect_interval,
            data_model_version=str(self.metadata.data_model_version),
            created_at=self.created_at,
            gpu_count=self.metadata.gpu_count if self.metadata.gpu_count else 0,
            cpu_count=self.metadata.cpu_count if self.metadata.cpu_count else 0,
            gpu_type=self.metadata.gpu_type if self.metadata.gpu_type else "",
            start_at=self.metadata.start_at,
            end_at=self.end_at,
            segments=self.segments,
```
- **EN**: This chunk defines `_to_oss_ci_metadata`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_to_oss_ci_metadata`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 154-177
```python
            tags=[],
        )

    def _to_oss_ci_timeseries_list(self) -> list[OssCiUtilizationTimeSeriesV1]:
        return [
            self._to_oss_ci_time_series(record, type="utilization", tags=["record"])
            for record in self.records
        ]

    def _to_oss_ci_time_series(
        self, record: UtilizationRecord, type: str, tags: list[str]
    ) -> OssCiUtilizationTimeSeriesV1:
        return OssCiUtilizationTimeSeriesV1(
            created_at=self.created_at,
            type=type,
            tags=tags,
            time_stamp=record.timestamp,
            repo=self.info.repo,
            workflow_id=self.info.workflow_run_id,
            run_attempt=self.info.run_attempt,
            job_id=self.info.job_id,
            workflow_name=self.info.workflow_name,
            job_name=self.info.job_name,
            json_data=str(record.data.to_json() if record.data else {}),
```
- **EN**: This chunk defines `_to_oss_ci_time_series`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_to_oss_ci_time_series`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 178-201
```python
        )


class UploadUtilizationData:
    """
    main class to handle utilization data conversion and s3 upload
    fetches raw log data from s3, convert to log model, then convert to db model, and upload to s3
    """

    def __init__(
        self,
        artifact_prefix: str,
        info: WorkflowInfo,
        dry_run: bool = False,
        debug: bool = False,
        local_path: str = "",
    ):
        self.artifact_prefix = artifact_prefix
        self.info = info
        self.segment_generator = SegmentGenerator()
        self.debug_mode = debug
        self.dry_run = dry_run
        self.local_path = local_path
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as UploadUtilizationData, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the ci statistics pipeline.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 UploadUtilizationData 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现CI 统计流水线中的一个关键步骤。

### Lines 202-213
```python
    def start(self) -> None:
        if self.local_path:
            metadata, valid_records, _ = self.get_log_data_from_local(self.local_path)
        else:
            print(f"Search for test log in s3 bucket: {UTILIZATION_BUCKET}")
            metadata, valid_records, _ = self.get_log_data_from_s3(
                self.info.workflow_run_id,
                self.info.job_id,
                self.info.run_attempt,
                self.artifact_prefix,
            )
```
- **EN**: This chunk defines `start`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `start`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 214-226
```python
        if not metadata:
            print("[Log Model] Failed to process test log, metadata is None")
            return None

        if len(valid_records) == 0:
            print("[Log Model] Failed to process test log, no valid records")
            return None
        segments = self.segment_generator.generate(valid_records)

        db_metadata, db_records = UtilizationDbConverter(
            self.info, metadata, valid_records, segments
        ).convert()
```
- **EN**: This chunk continues `start` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `start`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 227-239
```python
        if len(db_records) > 0:
            print(
                f"[db model] Peek db timeseries \n:{json.dumps(asdict(db_records[0]), indent=4)}"
            )

        if self.dry_run:
            print("[dry-run-mode]: no upload in dry run mode")
            return

        version = f"v_{db_metadata.data_model_version}"
        metadata_collection = "util_metadata"
        ts_collection = "util_timeseries"
        if self.debug_mode:
```
- **EN**: This chunk continues `start` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `start`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 240-253
```python
            metadata_collection = f"debug_{metadata_collection}"
            ts_collection = f"debug_{ts_collection}"

        self._upload_utilization_data_to_s3(
            collection=metadata_collection,
            version=version,
            repo=self.info.repo,
            workflow_run_id=self.info.workflow_run_id,
            workflow_run_attempt=self.info.run_attempt,
            job_id=self.info.job_id,
            file_name="metadata",
            docs=[asdict(db_metadata)],
        )
```
- **EN**: This chunk continues `start` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `start`，进一步展开其内部控制流或数据流转。

### Lines 254-265
```python
        self._upload_utilization_data_to_s3(
            collection=ts_collection,
            version=version,
            repo=self.info.repo,
            workflow_run_id=self.info.workflow_run_id,
            workflow_run_attempt=self.info.run_attempt,
            job_id=self.info.job_id,
            file_name="time_series",
            docs=[asdict(record) for record in db_records],
        )

    def _upload_utilization_data_to_s3(
```
- **EN**: This chunk defines `_upload_utilization_data_to_s3`, which parses or loads structured input into tool-friendly data structures. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `_upload_utilization_data_to_s3`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 266-279
```python
        self,
        collection: str,
        version: str,
        repo: str,
        workflow_run_id: int,
        workflow_run_attempt: int,
        job_id: int,
        file_name: str,
        docs: list[dict[str, Any]],
    ) -> None:
        bucket_name = UTILIZATION_BUCKET
        key = f"{collection}/{version}/{repo}/{workflow_run_id}/{workflow_run_attempt}/{job_id}/{file_name}"
        upload_to_s3(bucket_name, key, docs)
```
- **EN**: This chunk continues `_upload_utilization_data_to_s3` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `_upload_utilization_data_to_s3`，进一步展开其内部控制流或数据流转。

### Lines 280-291
```python
    def get_log_data_from_local(
        self,
        file_path: str,
        artifact_prefix: str = "",
    ) -> tuple[
        UtilizationMetadata | None, list[UtilizationRecord], list[UtilizationRecord]
    ]:
        test_log_content = read_file(file_path)
        if not test_log_content:
            return None, [], []
        metadata, records, error_records = self.convert_to_log_models(test_log_content)
        if metadata is None:
```
- **EN**: This chunk defines `get_log_data_from_local`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_log_data_from_local`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 292-308
```python
            return None, [], []
        print(f"Converted Log Model: UtilizationMetadata:\n {metadata}")
        return metadata, records, error_records

    def get_log_data_from_s3(
        self,
        workflow_run_id: int,
        job_id: int,
        workflow_run_attempt: int,
        artifact_prefix: str = JOB_TEST_ARTIFACT_PREFIX,
    ) -> tuple[
        UtilizationMetadata | None, list[UtilizationRecord], list[UtilizationRecord]
    ]:
        artifact_paths = download_s3_artifacts(
            artifact_prefix, workflow_run_id, workflow_run_attempt, job_id
        )
        if len(artifact_paths) == 0:
```
- **EN**: This chunk defines `get_log_data_from_s3`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_log_data_from_s3`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 309-320
```python
            print(
                f"Failed to download artifacts for workflow {workflow_run_id} and job {job_id}"
            )
            return None, [], []
        elif len(artifact_paths) > 1:
            print(
                f"Found more than one artifact for workflow {workflow_run_id} and job {job_id}, {artifact_paths}"
            )
            return None, [], []

        p = artifact_paths[0]
```
- **EN**: This chunk continues `get_log_data_from_s3` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_log_data_from_s3`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 321-332
```python
        test_log_content = handle_file(p)
        if not test_log_content:
            return None, [], []

        metadata, records, error_records = self.convert_to_log_models(test_log_content)
        if metadata is None:
            return None, [], []

        print(f"Converted Log Model: UtilizationMetadata:\n {metadata}")
        return metadata, records, error_records

    def _process_raw_record(self, line: str) -> tuple[UtilizationRecord | None, bool]:
```
- **EN**: This chunk defines `_process_raw_record`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_process_raw_record`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 333-354
```python
        try:
            record = UtilizationRecord.from_json(line)
            if record.error:
                return record, False
            return record, True
        except Exception as e:
            print(f"Failed to parse JSON line: {e}")
            return None, False

    def _process_utilization_records(
        self,
        lines: list[str],
    ) -> tuple[list[UtilizationRecord], list[UtilizationRecord]]:
        results = [self._process_raw_record(line) for line in lines[1:]]
        valid_records = [
            record for record, valid in results if valid and record is not None
        ]
        invalid_records = [
            record for record, valid in results if not valid and record is not None
        ]
        return valid_records, invalid_records
```
- **EN**: This chunk defines `_process_utilization_records`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_process_utilization_records`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 355-368
```python
    def convert_to_log_models(
        self,
        content: str,
    ) -> tuple[
        UtilizationMetadata | None, list[UtilizationRecord], list[UtilizationRecord]
    ]:
        if not content:
            return None, [], []
        lines = content.splitlines()
        metadata = None
        if len(lines) < 2:
            print("Expected at least two records from log file")
            return None, [], []
```
- **EN**: This chunk defines `convert_to_log_models`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `convert_to_log_models`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 369-380
```python
        try:
            metadata = UtilizationMetadata.from_json(lines[0])
        except Exception as e:
            print(f":: warning Failed to parse metadata: {e} for data: {lines[0]}")
            return None, [], []

        if metadata.data_model_version != getDataModelVersion():
            print(
                f":: warning Data model version mismatch: {metadata.data_model_version} != {getDataModelVersion()}"
            )
            return None, [], []
```
- **EN**: This chunk continues `convert_to_log_models` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `convert_to_log_models`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 381-394
```python
        result_logs, error_logs = self._process_utilization_records(lines)
        return metadata, result_logs, error_logs


def handle_file(file_path: Path) -> str:
    if file_path.match("*.zip"):
        print(f"extracting {TEST_USAGE_LOG_FILENAME} from zip file {file_path}")
        return unzip_file(file_path, TEST_USAGE_LOG_FILENAME)
    elif file_path.match("*.txt"):
        print(f"extracting {file_path}")
        return read_file(file_path)
    print(f"{file_path} is not a supported file type")
    return ""
```
- **EN**: This chunk defines `handle_file`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `handle_file`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 395-415
```python

def read_file(file_path: str | Path) -> str:
    try:
        if isinstance(file_path, Path):
            if file_path.is_file():
                with file_path.open("r") as f:
                    return f.read()
            else:
                print(f"::warning file {file_path} does not exist.")
        elif isinstance(file_path, str):
            if os.path.isfile(file_path):
                with open(file_path) as f:
                    return f.read()
            else:
                print(f"::warning file {file_path} does not exist.")
        else:
            print(f"::warning unsupported file_path type: {type(file_path)}")
    except Exception as e:
        print(f"::warning trying to read file {file_path} failed by: {e}")
    return ""
```
- **EN**: This chunk defines `read_file`, which parses or loads structured input into tool-friendly data structures. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `read_file`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 416-427
```python

def unzip_file(path: Path, file_name: str) -> str:
    try:
        with zipfile.ZipFile(path) as zip_file:
            # Read the desired file from the zip archive
            return zip_file.read(name=file_name).decode()
    except Exception as e:
        print(f"::warning trying to download test log {object} failed by: {e}")
        return ""


def parse_args() -> argparse.Namespace:
```
- **EN**: This chunk defines `parse_args`, which parses or loads structured input into tool-friendly data structures. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `parse_args`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 428-451
```python
    """
    Parse command line arguments.

    Returns:
        argparse.Namespace: Parsed arguments.
    """
    parser = argparse.ArgumentParser(description="Upload test stats to s3")
    parser.add_argument(
        "--workflow-run-id",
        type=int,
        required=True,
        help="id of the workflow to get artifacts from",
    )
    parser.add_argument(
        "--workflow-run-attempt",
        type=int,
        required=True,
        help="which retry of the workflow this is",
    )
    parser.add_argument(
        "--workflow-name",
        type=str,
        required=True,
        help="id of the workflow to get artifacts from",
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `parse_args` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 452-472
```python
    )
    parser.add_argument(
        "--job-id",
        type=int,
        required=True,
        help="id of the workflow to get artifacts from",
    )
    parser.add_argument(
        "--job-name",
        type=str,
        required=True,
        help="id of the workflow to get artifacts from",
    )
    parser.add_argument(
        "--repo",
        type=str,
        required=False,
        help="which GitHub repo this workflow run belongs to",
    )
    parser.add_argument("--debug", action="store_true", help="Enable debug mode")
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 473-487
```python
    parser.add_argument("--dry-run", action="store_true", help="Enable dry-run mode")

    parser.add_argument(
        "--artifact-prefix",
        type=str,
        required=False,
        help="artifact prefix to download raw utilizarion data from s3",
    )
    parser.add_argument(
        "--local-path",
        type=str,
        required=False,
        help="path of the raw utilizarion data from local location",
    )
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 488-501
```python
    return parser.parse_args()


if __name__ == "__main__":
    args = parse_args()

    # Flush stdout so that any errors in the upload show up last in the logs.
    sys.stdout.flush()

    repo = PYTORCH_REPO
    if args.repo:
        repo = args.repo
    print(f"repo: {repo}")
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 502-516
```python
    workflow_info = WorkflowInfo(
        workflow_run_id=args.workflow_run_id,
        run_attempt=args.workflow_run_attempt,
        job_id=args.job_id,
        workflow_name=args.workflow_name,
        job_name=args.job_name,
        repo=repo,
    )

    artifact_prefix = JOB_TEST_ARTIFACT_PREFIX
    if args.artifact_prefix:
        artifact_prefix = args.artifact_prefix
        print(f"args.artifact_prefix: {args.artifact_prefix}")
        print(f"artifact_prefix: {artifact_prefix}")
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 517-524
```python
    ud = UploadUtilizationData(
        info=workflow_info,
        dry_run=args.dry_run,
        debug=args.debug,
        artifact_prefix=artifact_prefix,
        local_path=args.local_path,
    )
    ud.start()
```
- **EN**: This chunk continues `parse_args` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `parse_args`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **CI statistics**
  - EN: This file belongs to the ci statistics layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于CI 统计层，应结合同一子目录中的相邻脚本一起理解。
- **Lint enforcement**
  - EN: The code applies policy checks to source files and reports actionable diagnostics.
  - CN: 代码对源码施加策略检查，并产出可执行的诊断信息。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **Pytest integration**
  - EN: The file plugs into pytest collection, execution, or reporting behavior.
  - CN: 该文件接入 pytest 的收集、执行或报告行为。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.upload_stats_lib`, `tools.stats.utilization_stats_lib`
- **Python standard library / Python 标准库**: `os`, `sys`, `pathlib`, `argparse`, `json`, `zipfile`, `dataclasses`, `typing`
- **External packages / 外部依赖包**: `pandas`
- **Primary symbols in this file / 本文件核心符号**: `TEST_USAGE_LOG_FILENAME`, `CMD_PYTHON_LEVEL`, `UTILIZATION_BUCKET`, `PYTORCH_REPO`, `JOB_TEST_ARTIFACT_PREFIX`, `SegmentGenerator`, `UtilizationDbConverter`, `UploadUtilizationData`, `handle_file`, `read_file`, `unzip_file`, `parse_args`
