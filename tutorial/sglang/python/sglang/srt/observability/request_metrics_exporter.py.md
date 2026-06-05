# request_metrics_exporter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/request_metrics_exporter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `request_metrics_exporter`. It exposes primary entry points such as `RequestMetricsExporter`, `FileRequestMetricsExporter`, `RequestMetricsExporterManager`. / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `request_metrics_exporter` 的逻辑。 它对外提供的主要入口包括 `RequestMetricsExporter`, `FileRequestMetricsExporter`, `RequestMetricsExporterManager`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Module imports, constants, and setup
```python
import asyncio
import dataclasses
import json
import logging
import os
from abc import ABC, abstractmethod
from datetime import datetime
from typing import List, Optional, Union

from sglang.srt.constants import HEALTH_CHECK_RID_PREFIX
from sglang.srt.managers.io_struct import EmbeddingReqInput, GenerateReqInput
from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)

# Fields that should always be excluded from request parameters
# because they contain non-JSON-serializable objects (e.g., ImageData, tensors)
ALWAYS_EXCLUDE_FIELDS = {"image_data", "video_data", "audio_data", "input_embeds"}


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 21-23: Class RequestMetricsExporter
```python
class RequestMetricsExporter(ABC):
    """Abstract base class for exporting request-level performance metrics to a data destination."""

```
**EN:** This range introduces `RequestMetricsExporter` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Abstract base class for exporting request-level performance metrics to a data destination." In this range it records metrics or tracing signals.
**CN:** 这一段引入 `RequestMetricsExporter`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 24-32: Method RequestMetricsExporter.__init__
```python
    def __init__(
        self,
        server_args: ServerArgs,
        obj_skip_names: Optional[set[str]],
        out_skip_names: Optional[set[str]],
    ):
        self.server_args = server_args
        self.obj_skip_names = obj_skip_names or set()
        self.out_skip_names = out_skip_names or set()
```
**EN:** This callable implements `RequestMetricsExporter.__init__`. It takes `server_args`, `obj_skip_names`, `out_skip_names` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `RequestMetricsExporter.__init__`。它接收 `server_args`, `obj_skip_names`, `out_skip_names`，主要用于初始化实例状态与默认值。

### Lines 34-62: Method RequestMetricsExporter._format_output_data
```python
    def _format_output_data(
        self, obj: Union[GenerateReqInput, EmbeddingReqInput], out_dict: dict
    ) -> dict:
        """Format request-level output data containing performance metrics. This method
        should be called prior to writing the data record with `self.write_record()`."""

        request_params = {}
        for field in dataclasses.fields(obj):
            field_name = field.name
            # Skip fields in obj_skip_names or fields that are always excluded (not JSON serializable)
            if (
                field_name not in self.obj_skip_names
                and field_name not in ALWAYS_EXCLUDE_FIELDS
            ):
                value = getattr(obj, field_name)
                # Convert to serializable format
                if value is not None:
                    request_params[field_name] = value

        meta_info = out_dict.get("meta_info", {})
        filtered_out_meta_info = {
            k: v for k, v in meta_info.items() if k not in self.out_skip_names
        }

        request_output_data = {
            "request_parameters": json.dumps(request_params),
            **filtered_out_meta_info,
        }
        return request_output_data
```
**EN:** This callable implements `RequestMetricsExporter._format_output_data`. It takes `obj`, `out_dict` and mainly implements format output data. The docstring states: "Format request-level output data containing performance metrics." In this range it serializes or parses JSON payloads; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `RequestMetricsExporter._format_output_data`。它接收 `obj`, `out_dict`，主要用于实现 format output data 相关逻辑。 在这一范围内，它会序列化或解析 JSON 数据；记录指标或追踪信号。

### Lines 64-69: Method RequestMetricsExporter.write_record
```python
    @abstractmethod
    async def write_record(
        self, obj: Union[GenerateReqInput, EmbeddingReqInput], out_dict: dict
    ):
        """Write a data record corresponding to a single request, containing performance metric data."""
        pass
```
**EN:** This callable implements `RequestMetricsExporter.write_record`. It takes `obj`, `out_dict` and mainly implements write record. The docstring states: "Write a data record corresponding to a single request, containing performance metric data." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `RequestMetricsExporter.write_record`。它接收 `obj`, `out_dict`，主要用于实现 write record 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 72-78: Class FileRequestMetricsExporter
```python
class FileRequestMetricsExporter(RequestMetricsExporter):
    """Lightweight `RequestMetricsExporter` implementation that writes records to files on disk.

    Records are written to files in the directory specified by `--export-metrics-to-file-dir`
    server launch flag. File names are of the form `"sglang-request-metrics-{hour_suffix}.log"`.
    """

```
**EN:** This range introduces `FileRequestMetricsExporter` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Lightweight `RequestMetricsExporter` implementation that writes records to files on disk." In this range it records metrics or tracing signals; manages model weights or checkpoints.
**CN:** 这一段引入 `FileRequestMetricsExporter`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号；管理模型权重或检查点。

### Lines 79-92: Method FileRequestMetricsExporter.__init__
```python
    def __init__(
        self,
        server_args: ServerArgs,
        obj_skip_names: Optional[set[str]],
        out_skip_names: Optional[set[str]],
    ):
        super().__init__(server_args, obj_skip_names, out_skip_names)
        self.export_dir = getattr(server_args, "export_metrics_to_file_dir")
        os.makedirs(self.export_dir, exist_ok=True)

        # File handler state management
        self._current_file_handler = None
        self._current_file_lock = asyncio.Lock()
        self._current_hour_suffix = None
```
**EN:** This callable implements `FileRequestMetricsExporter.__init__`. It takes `server_args`, `obj_skip_names`, `out_skip_names` and mainly initializes instance state and defaults. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `FileRequestMetricsExporter.__init__`。它接收 `server_args`, `obj_skip_names`, `out_skip_names`，主要用于初始化实例状态与默认值。 在这一范围内，它会记录指标或追踪信号。

### Lines 94-115: Method FileRequestMetricsExporter._ensure_file_handler
```python
    def _ensure_file_handler(self, hour_suffix: str):
        """Ensure the file handler is open for the current hour suffix."""
        if self._current_hour_suffix != hour_suffix:
            # Close previous file handler if it exists
            if self._current_file_handler is not None:
                try:
                    self._current_file_handler.close()
                except Exception as e:
                    logger.warning(f"Failed to close previous file handler: {e}")

            # Open new file handler
            log_filename = f"sglang-request-metrics-{hour_suffix}.log"
            log_filepath = os.path.join(self.export_dir, log_filename)

            try:
                self._current_file_handler = open(log_filepath, "a", encoding="utf-8")
                self._current_hour_suffix = hour_suffix
            except Exception as e:
                logger.error(f"Failed to open log file {log_filepath}: {e}")
                self._current_file_handler = None
                self._current_hour_suffix = None
                raise
```
**EN:** This callable implements `FileRequestMetricsExporter._ensure_file_handler`. It takes `hour_suffix` and mainly ensures required state exists. The docstring states: "Ensure the file handler is open for the current hour suffix." In this range it emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `FileRequestMetricsExporter._ensure_file_handler`。它接收 `hour_suffix`，主要用于确保所需状态已经存在。 在这一范围内，它会输出日志以便诊断；记录指标或追踪信号。

### Lines 117-126: Method FileRequestMetricsExporter.close
```python
    def close(self):
        """Close the current file handler."""
        if self._current_file_handler is not None:
            try:
                self._current_file_handler.close()
            except Exception as e:
                logger.warning(f"Failed to close file handler: {e}")
            finally:
                self._current_file_handler = None
                self._current_hour_suffix = None
```
**EN:** This callable implements `FileRequestMetricsExporter.close` and mainly implements close. The docstring states: "Close the current file handler." In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `FileRequestMetricsExporter.close`，主要用于实现 close 相关逻辑。 在这一范围内，它会输出日志以便诊断。

### Lines 128-156: Method FileRequestMetricsExporter.write_record
```python
    async def write_record(
        self, obj: Union[GenerateReqInput, EmbeddingReqInput], out_dict: dict
    ):
        # Do not log health check requests, since they don't represent real user requests.
        if isinstance(obj.rid, str) and HEALTH_CHECK_RID_PREFIX in obj.rid:
            return

        try:
            # Get the log file path for the current time.
            current_time = datetime.now()
            hour_suffix = current_time.strftime("%Y%m%d_%H")

            async with self._current_file_lock:
                # Ensure correct file handler is open for current hour
                self._ensure_file_handler(hour_suffix)

                if self._current_file_handler is None:
                    return

                metrics_data = self._format_output_data(obj, out_dict)

                def write_file():
                    json.dump(metrics_data, self._current_file_handler)
                    self._current_file_handler.write("\n")
                    self._current_file_handler.flush()

                await asyncio.to_thread(write_file)
        except Exception as e:
            logger.exception(f"Failed to write perf metrics to file: {e}")
```
**EN:** This callable implements `FileRequestMetricsExporter.write_record`. It takes `obj`, `out_dict` and mainly implements write record. In this range it emits logs for diagnostics; serializes or parses JSON payloads; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `FileRequestMetricsExporter.write_record`。它接收 `obj`, `out_dict`，主要用于实现 write record 相关逻辑。 在这一范围内，它会输出日志以便诊断；序列化或解析 JSON 数据；记录指标或追踪信号。

### Lines 159-161: Class RequestMetricsExporterManager
```python
class RequestMetricsExporterManager:
    """Manager class for creating and managing RequestMetricsExporter instances."""

```
**EN:** This range introduces `RequestMetricsExporterManager` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Manager class for creating and managing RequestMetricsExporter instances." In this range it records metrics or tracing signals.
**CN:** 这一段引入 `RequestMetricsExporterManager`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 162-172: Method RequestMetricsExporterManager.__init__
```python
    def __init__(
        self,
        server_args: ServerArgs,
        obj_skip_names: Optional[set[str]] = None,
        out_skip_names: Optional[set[str]] = None,
    ):
        self.server_args = server_args
        self.obj_skip_names = obj_skip_names or set()
        self.out_skip_names = out_skip_names or set()
        self._exporters: List[RequestMetricsExporter] = []
        self._create_exporters()
```
**EN:** This callable implements `RequestMetricsExporterManager.__init__`. It takes `server_args`, `obj_skip_names`, `out_skip_names` and mainly initializes instance state and defaults. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `RequestMetricsExporterManager.__init__`。它接收 `server_args`, `obj_skip_names`, `out_skip_names`，主要用于初始化实例状态与默认值。 在这一范围内，它会记录指标或追踪信号。

### Lines 174-195: Method RequestMetricsExporterManager._create_exporters
```python
    def _create_exporters(self) -> None:
        """Create and configure RequestMetricsExporter instances based on server args."""
        # Create standard exporters
        self._exporters.extend(
            create_request_metrics_exporters(
                self.server_args, self.obj_skip_names, self.out_skip_names
            )
        )

        # Import additional RequestMetricsExporter from private fork if available; skip otherwise.
        try:
            from sglang.private.managers.request_metrics_exporter_factory import (
                create_private_request_metrics_exporters,
            )

            self._exporters.extend(
                create_private_request_metrics_exporters(
                    self.server_args, self.obj_skip_names, self.out_skip_names
                )
            )
        except ImportError:
            pass
```
**EN:** This callable implements `RequestMetricsExporterManager._create_exporters` and mainly exports processed data. The docstring states: "Create and configure RequestMetricsExporter instances based on server args." In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `RequestMetricsExporterManager._create_exporters`，主要用于导出处理后的数据。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

### Lines 197-199: Method RequestMetricsExporterManager.exporter_enabled
```python
    def exporter_enabled(self) -> bool:
        """Return true if at least one RequestMetricsExporter is enabled."""
        return len(self._exporters) > 0
```
**EN:** This callable implements `RequestMetricsExporterManager.exporter_enabled` and mainly exports processed data. The docstring states: "Return true if at least one RequestMetricsExporter is enabled." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `RequestMetricsExporterManager.exporter_enabled`，主要用于导出处理后的数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 201-204: Method RequestMetricsExporterManager.write_record
```python
    async def write_record(self, obj, out_dict: dict) -> None:
        """Write a record using all configured exporters."""
        for exporter in self._exporters:
            await exporter.write_record(obj, out_dict)
```
**EN:** This callable implements `RequestMetricsExporterManager.write_record`. It takes `obj`, `out_dict` and mainly implements write record. The docstring states: "Write a record using all configured exporters."
**CN:** 这一可调用对象实现了 `RequestMetricsExporterManager.write_record`。它接收 `obj`, `out_dict`，主要用于实现 write record 相关逻辑。

### Lines 207-220: Function create_request_metrics_exporters
```python
def create_request_metrics_exporters(
    server_args: ServerArgs,
    obj_skip_names: Optional[set[str]] = None,
    out_skip_names: Optional[set[str]] = None,
) -> List[RequestMetricsExporter]:
    """Create and configure `RequestMetricsExporter`s based on server args."""
    metrics_exporters = []

    if server_args.export_metrics_to_file:
        metrics_exporters.append(
            FileRequestMetricsExporter(server_args, obj_skip_names, out_skip_names)
        )

    return metrics_exporters
```
**EN:** This callable implements `create_request_metrics_exporters`. It takes `server_args`, `obj_skip_names`, `out_skip_names` and mainly exports processed data. The docstring states: "Create and configure `RequestMetricsExporter`s based on server args." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `create_request_metrics_exporters`。它接收 `server_args`, `obj_skip_names`, `out_skip_names`，主要用于导出处理后的数据。 在这一范围内，它会记录指标或追踪信号。

## Key Concepts / 关键概念
- `ALWAYS_EXCLUDE_FIELDS`: module constant or capability flag / 模块常量或能力标记
- `RequestMetricsExporter`: core class or state container / 核心类或状态容器
- `FileRequestMetricsExporter`: core class or state container / 核心类或状态容器
- `RequestMetricsExporterManager`: core class or state container / 核心类或状态容器
- `create_request_metrics_exporters`: exports processed data / 导出处理后的数据

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `dataclasses`, `json`, `logging`, `os`, `abc`, `datetime`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.constants`, `sglang.srt.managers.io_struct`, `sglang.srt.server_args`, `sglang.private.managers.request_metrics_exporter_factory`
