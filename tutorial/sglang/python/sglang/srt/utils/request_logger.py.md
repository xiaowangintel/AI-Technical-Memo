# request_logger.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/request_logger.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for logging helpers. / 为 SGLang 运行时提供面向日志辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 14-33: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

import dataclasses
import logging
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Set, Tuple, Union

from sglang.srt.environ import envs
from sglang.srt.utils.log_utils import create_log_targets, log_json

if TYPE_CHECKING:
    import fastapi

    from sglang.srt.managers.io_struct import EmbeddingReqInput, GenerateReqInput

logger = logging.getLogger(__name__)

_DEFAULT_WHITELISTED_HEADERS = ["x-smg-routing-key"]
WHITELISTED_HEADERS = _DEFAULT_WHITELISTED_HEADERS + [
    h.lower() for h in envs.SGLANG_LOG_REQUEST_HEADERS.get()
]
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `dataclasses`, `logging`, `typing`, `sglang.srt.environ`, `sglang.srt.utils.log_utils`. It also defines symbols such as `logger`, `_DEFAULT_WHITELISTED_HEADERS`, `WHITELISTED_HEADERS` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `dataclasses`, `logging`, `typing`, `sglang.srt.environ`, `sglang.srt.utils.log_utils`。 同时定义了 `logger`, `_DEFAULT_WHITELISTED_HEADERS`, `WHITELISTED_HEADERS` 等符号，供后续逻辑使用。

### Lines 36-41: Function `_extract_whitelisted_headers` / 函数 `_extract_whitelisted_headers`
```python
def _extract_whitelisted_headers(
    request: Optional["fastapi.Request"],
) -> Optional[Dict[str, str]]:
    if request is None:
        return None
    return {h: v for h in WHITELISTED_HEADERS if (v := request.headers.get(h))}
```
**EN:** This function implements `_extract_whitelisted_headers`. It primarily calls `request.headers.get` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_extract_whitelisted_headers`。 它主要通过调用 `request.headers.get` 来完成任务。 实现中使用了条件分支。

### Lines 44-44: Class `RequestLogger` declaration / 类 `RequestLogger` 声明
```python
class RequestLogger:
```
**EN:** This class establishes `RequestLogger` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `_setup_targets`, `configure`, `log_received_request`, `log_openai_received_request`, `log_finished_request`.
**CN:** 该类将 `RequestLogger` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `_setup_targets`, `configure`, `log_received_request`, `log_openai_received_request`, `log_finished_request` 等方法。

### Lines 45-62: Method `RequestLogger.__init__` / 方法 `RequestLogger.__init__`
```python
    def __init__(
        self,
        log_requests: bool,
        log_requests_level: int,
        log_requests_format: str,
        log_requests_target: Optional[List[str]],
    ):
        self.log_requests = log_requests
        self.log_requests_level = log_requests_level
        self.log_requests_format = log_requests_format
        self.log_requests_target = log_requests_target

        self.metadata: Tuple[Optional[int], Optional[Set[str]], Optional[Set[str]]] = (
            self._compute_metadata()
        )
        self.targets = self._setup_targets()

        self.log_exceeded_ms = envs.SGLANG_LOG_REQUEST_EXCEEDED_MS.get()
```
**EN:** This method implements `__init__` on `RequestLogger`. It primarily calls `self._compute_metadata`, `self._setup_targets`, `envs.SGLANG_LOG_REQUEST_EXCEEDED_MS.get` to complete its work. State updates are written into `self.log_requests`, `self.log_requests_level`, `self.log_requests_format`, `self.log_requests_target`, `self.metadata`, `self.targets`.
**CN:** 该方法（属于 `RequestLogger`）实现了 `__init__`。 它主要通过调用 `self._compute_metadata`, `self._setup_targets`, `envs.SGLANG_LOG_REQUEST_EXCEEDED_MS.get` 来完成任务。 状态更新主要写入 `self.log_requests`, `self.log_requests_level`, `self.log_requests_format`, `self.log_requests_target`, `self.metadata`, `self.targets`。

### Lines 64-67: Method `RequestLogger._setup_targets` / 方法 `RequestLogger._setup_targets`
```python
    def _setup_targets(self) -> List[logging.Logger]:
        return create_log_targets(
            targets=self.log_requests_target, name_prefix=__name__
        )
```
**EN:** This method implements `_setup_targets` on `RequestLogger`. It primarily calls `create_log_targets` to complete its work.
**CN:** 该方法（属于 `RequestLogger`）实现了 `_setup_targets`。 它主要通过调用 `create_log_targets` 来完成任务。

### Lines 69-86: Method `RequestLogger.configure` / 方法 `RequestLogger.configure`
```python
    def configure(
        self,
        log_requests: Optional[bool] = None,
        log_requests_level: Optional[int] = None,
        log_requests_format: Optional[str] = None,
        log_requests_target: Optional[List[str]] = None,
    ) -> None:
        if log_requests is not None:
            self.log_requests = log_requests
        if log_requests_level is not None:
            self.log_requests_level = log_requests_level
        if log_requests_format is not None:
            self.log_requests_format = log_requests_format
        if log_requests_target is not None:
            self.log_requests_target = log_requests_target

        self.metadata = self._compute_metadata()
        self.targets = self._setup_targets()
```
**EN:** This method implements `configure` on `RequestLogger`. It primarily calls `self._compute_metadata`, `self._setup_targets` to complete its work. State updates are written into `self.metadata`, `self.targets`, `self.log_requests`, `self.log_requests_level`, `self.log_requests_format`, `self.log_requests_target`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `RequestLogger`）实现了 `configure`。 它主要通过调用 `self._compute_metadata`, `self._setup_targets` 来完成任务。 状态更新主要写入 `self.metadata`, `self.targets`, `self.log_requests`, `self.log_requests_level`, `self.log_requests_format`, `self.log_requests_target`。 实现中使用了条件分支。

### Lines 88-129: Method `RequestLogger.log_received_request` / 方法 `RequestLogger.log_received_request`
```python
    def log_received_request(
        self,
        obj: Union["GenerateReqInput", "EmbeddingReqInput"],
        tokenizer: Any = None,
        request: Optional["fastapi.Request"] = None,
    ) -> None:
        if not self.log_requests:
            return

        max_length, skip_names, _ = self.metadata
        headers = _extract_whitelisted_headers(request)
        if self.log_requests_format == "json":
            log_data = {
                "rid": obj.rid,
                "obj": _transform_data_for_logging(obj, max_length, skip_names),
            }
            if headers:
                log_data["headers"] = headers
            log_json(self.targets, "request.received", log_data)
        else:
            headers_str = f", headers={headers}" if headers else ""
            self._log(
                f"Receive: obj={_dataclass_to_string_truncated(obj, max_length, skip_names=skip_names)}{headers_str}"
            )

        # FIXME: This is a temporary fix to get the text from the input ids.
        # We should remove this once we have a proper way.
        if (
# ... omitted for brevity ...
                    tokenizer.decode(_input_ids, skip_special_tokens=False)
                    for _input_ids in obj.input_ids
                ]
            else:
                decoded = tokenizer.decode(obj.input_ids, skip_special_tokens=False)
            obj.text = decoded
```
**EN:** This method implements `log_received_request` on `RequestLogger`. It primarily calls `_extract_whitelisted_headers`, `log_json`, `self._log`, `_transform_data_for_logging`, `isinstance`, `tokenizer.decode` to complete its work. State updates are written into `headers`, `log_data`, `headers_str`, `obj.text`, `decoded`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `RequestLogger`）实现了 `log_received_request`。 它主要通过调用 `_extract_whitelisted_headers`, `log_json`, `self._log`, `_transform_data_for_logging`, `isinstance`, `tokenizer.decode` 来完成任务。 状态更新主要写入 `headers`, `log_data`, `headers_str`, `obj.text`, `decoded`。 实现中使用了条件分支。

### Lines 131-157: Method `RequestLogger.log_openai_received_request` / 方法 `RequestLogger.log_openai_received_request`
```python
    def log_openai_received_request(
        self,
        obj: Any,
        request: Optional["fastapi.Request"] = None,
    ) -> None:
        """Log the raw OpenAI request payload before request adaptation/tokenization."""
        max_length, _, _ = self.metadata
        max_length = max_length if max_length is not None else 2048
        headers = _extract_whitelisted_headers(request)

        if hasattr(obj, "model_dump"):
            obj_to_log = obj.model_dump(exclude_none=True)
        else:
            obj_to_log = obj

        if self.log_requests_format == "json":
            log_data = {
                "obj": _transform_data_for_logging(obj_to_log, max_length=max_length),
            }
            if headers:
                log_data["headers"] = headers
            log_json(self.targets, "request.received.openai", log_data)
        else:
            headers_str = f", headers={headers}" if headers else ""
            self._log(
                f"Receive OpenAI: obj={_dataclass_to_string_truncated(obj_to_log, max_length)}{headers_str}"
            )
```
**EN:** This method implements `log_openai_received_request` on `RequestLogger`. It primarily calls `_extract_whitelisted_headers`, `hasattr`, `obj.model_dump`, `log_json`, `self._log`, `_transform_data_for_logging` to complete its work. State updates are written into `max_length`, `headers`, `obj_to_log`, `log_data`, `headers_str`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `RequestLogger`）实现了 `log_openai_received_request`。 它主要通过调用 `_extract_whitelisted_headers`, `hasattr`, `obj.model_dump`, `log_json`, `self._log`, `_transform_data_for_logging` 来完成任务。 状态更新主要写入 `max_length`, `headers`, `obj_to_log`, `log_data`, `headers_str`。 实现中使用了条件分支。

### Lines 159-191: Method `RequestLogger.log_finished_request` / 方法 `RequestLogger.log_finished_request`
```python
    def log_finished_request(
        self,
        obj: Union["GenerateReqInput", "EmbeddingReqInput"],
        out: Any,
        request: Optional["fastapi.Request"] = None,
    ) -> None:
        if not self.log_requests:
            return

        e2e_latency_ms = out["meta_info"].get("e2e_latency", 0) * 1000
        if self.log_exceeded_ms > 0 and e2e_latency_ms < self.log_exceeded_ms:
            return

        max_length, skip_names, out_skip_names = self.metadata
        headers = _extract_whitelisted_headers(request)
        if self.log_requests_format == "json":
            log_data = {
                "rid": obj.rid,
                "obj": _transform_data_for_logging(obj, max_length, skip_names),
            }
            if headers:
                log_data["headers"] = headers
            log_data["out"] = _transform_data_for_logging(
                out, max_length, out_skip_names
            )
            log_json(self.targets, "request.finished", log_data)
        else:
            obj_str = _dataclass_to_string_truncated(
                obj, max_length, skip_names=skip_names
            )
            out_str = f", out={_dataclass_to_string_truncated(out, max_length, skip_names=out_skip_names)}"
            headers_str = f", headers={headers}" if headers else ""
            self._log(f"Finish: obj={obj_str}{headers_str}{out_str}")
```
**EN:** This method implements `log_finished_request` on `RequestLogger`. It primarily calls `_extract_whitelisted_headers`, `out.get`, `_transform_data_for_logging`, `log_json`, `_dataclass_to_string_truncated`, `self._log` to complete its work. State updates are written into `e2e_latency_ms`, `headers`, `log_data`, `obj_str`, `out_str`, `headers_str`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `RequestLogger`）实现了 `log_finished_request`。 它主要通过调用 `_extract_whitelisted_headers`, `out.get`, `_transform_data_for_logging`, `log_json`, `_dataclass_to_string_truncated`, `self._log` 来完成任务。 状态更新主要写入 `e2e_latency_ms`, `headers`, `log_data`, `obj_str`, `out_str`, `headers_str`。 实现中使用了条件分支。

### Lines 193-233: Method `RequestLogger._compute_metadata` / 方法 `RequestLogger._compute_metadata`
```python
    def _compute_metadata(
        self,
    ) -> Tuple[Optional[int], Optional[Set[str]], Optional[Set[str]]]:
        max_length: Optional[int] = None
        skip_names: Optional[Set[str]] = None
        out_skip_names: Optional[Set[str]] = None
        if self.log_requests:
            if self.log_requests_level == 0:
                max_length = 1 << 30
                skip_names = {
                    "text",
                    "input_ids",
                    "input_embeds",
                    "image_data",
                    "audio_data",
                    "video_data",
                    "lora_path",
                    "sampling_params",
                }
                out_skip_names = {"text", "output_ids", "embedding"}
            elif self.log_requests_level == 1:
                max_length = 1 << 30
                skip_names = {
                    "text",
                    "input_ids",
                    "input_embeds",
                    "image_data",
                    "audio_data",
# ... omitted for brevity ...
                max_length = 1 << 30
            else:
                raise ValueError(
                    f"Invalid --log-requests-level: {self.log_requests_level=}"
                )
        return max_length, skip_names, out_skip_names
```
**EN:** This method implements `_compute_metadata` on `RequestLogger`. It primarily calls `ValueError` to complete its work. State updates are written into `max_length`, `skip_names`, `out_skip_names`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `RequestLogger`）实现了 `_compute_metadata`。 它主要通过调用 `ValueError` 来完成任务。 状态更新主要写入 `max_length`, `skip_names`, `out_skip_names`。 实现中使用了条件分支。

### Lines 235-237: Method `RequestLogger._log` / 方法 `RequestLogger._log`
```python
    def _log(self, msg: str) -> None:
        for target in self.targets:
            target.info(msg)
```
**EN:** This method implements `_log` on `RequestLogger`. It primarily calls `target.info` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `RequestLogger`）实现了 `_log`。 它主要通过调用 `target.info` 来完成任务。 实现中使用了迭代逻辑。

### Lines 241-280: Function `_dataclass_to_string_truncated` / 函数 `_dataclass_to_string_truncated`
```python
def _dataclass_to_string_truncated(
    data: Any, max_length: int = 2048, skip_names: Optional[Set[str]] = None
) -> str:
    if skip_names is None:
        skip_names = set()
    if isinstance(data, str):
        if len(data) > max_length:
            half_length = max_length // 2
            return f"{repr(data[:half_length])} ... {repr(data[-half_length:])}"
        else:
            return f"{repr(data)}"
    elif isinstance(data, (list, tuple)):
        if len(data) > max_length:
            half_length = max_length // 2
            return str(data[:half_length]) + " ... " + str(data[-half_length:])
        else:
            return str(data)
    elif isinstance(data, dict):
        return (
            "{"
            + ", ".join(
                f"'{k}': {_dataclass_to_string_truncated(v, max_length)}"
                for k, v in data.items()
                if k not in skip_names
            )
            + "}"
        )
    elif dataclasses.is_dataclass(data):
# ... omitted for brevity ...
                if f.name not in skip_names
            )
            + ")"
        )
    else:
        return str(data)
```
**EN:** This function implements `_dataclass_to_string_truncated`. It primarily calls `isinstance`, `set`, `len`, `str`, `dataclasses.is_dataclass`, `repr` to complete its work. State updates are written into `skip_names`, `half_length`, `fields`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_dataclass_to_string_truncated`。 它主要通过调用 `isinstance`, `set`, `len`, `str`, `dataclasses.is_dataclass`, `repr` 来完成任务。 状态更新主要写入 `skip_names`, `half_length`, `fields`。 实现中使用了条件分支。

### Lines 283-314: Function `_transform_data_for_logging` / 函数 `_transform_data_for_logging`
```python
def _transform_data_for_logging(
    data: Any, max_length: int = 2048, skip_names: Optional[Set[str]] = None
) -> Any:
    if skip_names is None:
        skip_names = set()
    if isinstance(data, str):
        if len(data) > max_length:
            half_length = max_length // 2
            return data[:half_length] + "..." + data[-half_length:]
        return data
    elif isinstance(data, (list, tuple)):
        if len(data) > max_length:
            half_length = max_length // 2
            return list(data[:half_length]) + ["..."] + list(data[-half_length:])
        return [_transform_data_for_logging(v, max_length) for v in data]
    elif isinstance(data, dict):
        return {
            k: _transform_data_for_logging(v, max_length)
            for k, v in data.items()
            if k not in skip_names
        }
    elif dataclasses.is_dataclass(data):
        fields = dataclasses.fields(data)
        return {
            f.name: _transform_data_for_logging(getattr(data, f.name), max_length)
            for f in fields
            if f.name not in skip_names
        }
    elif isinstance(data, (int, float, bool, type(None))):
        return data
    else:
        return str(data)
```
**EN:** This function implements `_transform_data_for_logging`. It primarily calls `isinstance`, `set`, `len`, `_transform_data_for_logging`, `dataclasses.is_dataclass`, `list` to complete its work. State updates are written into `skip_names`, `half_length`, `fields`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_transform_data_for_logging`。 它主要通过调用 `isinstance`, `set`, `len`, `_transform_data_for_logging`, `dataclasses.is_dataclass`, `list` 来完成任务。 状态更新主要写入 `skip_names`, `half_length`, `fields`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `RequestLogger`
- **Functions / 函数**: `_extract_whitelisted_headers`, `_dataclass_to_string_truncated`, `_transform_data_for_logging`, `__init__`, `_setup_targets`, `configure`, `log_received_request`, `log_openai_received_request`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`, `sglang.srt.utils.log_utils`, `sglang.srt.managers.io_struct`
- **External / 外部依赖**: `fastapi`
- **Standard library / 标准库**: `__future__`, `dataclasses`, `logging`, `typing`
