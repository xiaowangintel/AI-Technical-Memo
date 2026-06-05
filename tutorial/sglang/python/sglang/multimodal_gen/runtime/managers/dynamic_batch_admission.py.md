# dynamic_batch_admission.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/managers/dynamic_batch_admission.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for dynamic batch admission in the multimodal generation stack. Key symbols include `AdmissionLimit`, `BatchingRule`, `BatchAdmissionController`. / 该模块包含多模态生成体系中与 dynamic batch admission 相关的运行时支持代码。 关键符号包括 `AdmissionLimit`, `BatchingRule`, `BatchAdmissionController`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""Admission control for native diffusion batching.

Native diffusion batching is model, resolution, device, and implementation
dependent. The scheduler treats `--batching-max-size` as the public ceiling;
`--batching-config` can apply stricter caps for specific model and shape
combinations.
"""

from __future__ import annotations

import json
import os
from dataclasses import dataclass
# ...
        # by admission, but accepted so production configs can explain caps.
        "calibration",
    }
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 45-63: Class `AdmissionLimit` / 类 `AdmissionLimit`
```python
@dataclass(frozen=True)
class AdmissionLimit:
    """Effective batch size and cost caps after matching batching rules."""

    max_batch_size: int
    max_cost: float | None = None
    cap_reason: str | None = None

    def reject_reason(self, *, batch_size: int, batch_cost: float) -> str | None:
        if batch_size > self.max_batch_size:
            return self.cap_reason or f"config_cap:{self.max_batch_size}"
        if self.max_cost is not None and batch_cost > self.max_cost:
            return f"cost_budget:{batch_cost:.0f}>{self.max_cost:.0f}"
        return None

    def stop_reason_for_next_cost(self, next_batch_cost: float) -> str | None:
        if self.max_cost is not None and next_batch_cost > self.max_cost:
            return f"cost_budget_next:{next_batch_cost:.0f}>{self.max_cost:.0f}"
        return None
```
**EN:** This class models `AdmissionLimit`. Effective batch size and cost caps after matching batching rules. Important methods include `reject_reason`, `stop_reason_for_next_cost`.
**CN:** 该类实现 `AdmissionLimit`。 文档字符串指出：Effective batch size and cost caps after matching batching rules. 其中较重要的方法包括 `reject_reason`, `stop_reason_for_next_cost`。

### Lines 66-153: Class `BatchingRule` / 类 `BatchingRule`
```python
@dataclass(frozen=True)
class BatchingRule:
    """One user-provided batching admission rule loaded from batching config."""

    model: str | None = None
    model_contains: str | None = None
    resolution: str | None = None
    device_memory_gb_min: float | None = None
    device_memory_gb_max: float | None = None
    offload: bool | None = None
    max_batch_size: int = 1
    max_cost: float | None = None
    source: str = "user"

# ...
            and device_memory_gb > self.device_memory_gb_max
        ):
            return False
        return True
```
**EN:** This class models `BatchingRule`. One user-provided batching admission rule loaded from batching config. Important methods include `from_dict`, `validate`, `matches`.
**CN:** 该类实现 `BatchingRule`。 文档字符串指出：One user-provided batching admission rule loaded from batching config. 其中较重要的方法包括 `from_dict`, `validate`, `matches`。

### Lines 156-260: Class `BatchAdmissionController` / 类 `BatchAdmissionController`
```python
class BatchAdmissionController:
    """Applies configured caps before adding requests to a batch."""

    def __init__(self, server_args: "ServerArgs", gpu_id: int):
        self._mode = getattr(server_args, "batching_mode", "dynamic")
        self._user_max_batch_size = max(1, int(server_args.batching_max_size))
        self._model_path = server_args.model_path
        self._offload = bool(server_args.layerwise_offload_components)
        self._device_memory_gb = self._get_device_memory_gb(gpu_id)
        self._rules = load_batching_config(server_args.batching_config)
        self._pipeline_config = server_args.pipeline_config

        if self.enabled:
            logger.info(
# ...
        try:
            return current_platform.get_device_total_memory(gpu_id) / BYTES_PER_GB
        except Exception:
            return None
```
**EN:** This class models `BatchAdmissionController`. Applies configured caps before adding requests to a batch. Important methods include `__init__`, `enabled`, `reject_reason_for_candidate`, `batch_is_full`.
**CN:** 该类实现 `BatchAdmissionController`。 文档字符串指出：Applies configured caps before adding requests to a batch. 其中较重要的方法包括 `__init__`, `enabled`, `reject_reason_for_candidate`, `batch_is_full`。

### Lines 263-275: Function `load_batching_config` / 函数 `load_batching_config`
```python
def load_batching_config(path: str | None) -> list[BatchingRule]:
    if path is None:
        return []

    with open(path, encoding="utf-8") as f:
        payload = json.load(f)

    source = os.path.abspath(path)
    entries = _config_entries(payload)
    rules = [BatchingRule.from_dict(entry, source=source) for entry in entries]
    if not rules:
        raise ValueError(f"batching config {source} does not contain any rules")
    return rules
```
**EN:** This function drives `load_batching_config` with inputs such as `path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `load_batching_config`，主要处理 `path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 278-301: Function `_config_entries` / 函数 `_config_entries`
```python
def _config_entries(payload: Any) -> list[dict[str, Any]]:
    if isinstance(payload, dict) and payload.get("schema_version") not in (None, 1):
        raise ValueError("batching config schema_version must be 1")
    if isinstance(payload, dict) and isinstance(payload.get("rules"), list):
        return payload["rules"]
    if isinstance(payload, list):
        return payload
    if isinstance(payload, dict):
        entries: list[dict[str, Any]] = []
        for key, value in payload.items():
            if key == "schema_version" or not isinstance(value, dict):
                continue
            model, _sep, resolution = key.partition("|")
            entry = dict(value)
# ...
    raise ValueError(
        "batching config must be a {'schema_version': 1, 'rules': [...]} object, "
        "a list of rules, or a mapping keyed by model|resolution"
    )
```
**EN:** This function drives `_config_entries` with inputs such as `payload`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_config_entries`，主要处理 `payload` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 304-319: Function `_validate_rule_keys` / 函数 `_validate_rule_keys`
```python
def _validate_rule_keys(data: dict[str, Any], *, source: str) -> None:
    unknown = sorted(set(data) - _BATCHING_RULE_KEYS)
    if not unknown:
        return

    hints = []
    for key in unknown:
        matches = get_close_matches(key, _BATCHING_RULE_KEYS, n=1)
        if matches:
            hints.append(f"{key!r} (did you mean {matches[0]!r}?)")
        else:
            hints.append(repr(key))
    raise ValueError(
        f"batching config rule from {source} contains unknown key(s): "
        f"{', '.join(hints)}"
    )
```
**EN:** This function drives `_validate_rule_keys` with inputs such as `data`, `source`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_validate_rule_keys`，主要处理 `data`, `source` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 322-325: Function `_optional_str` / 函数 `_optional_str`
```python
def _optional_str(value: Any) -> str | None:
    if value is None:
        return None
    return str(value)
```
**EN:** This function drives `_optional_str` with inputs such as `value`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_optional_str`，主要处理 `value` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 328-331: Function `_optional_float` / 函数 `_optional_float`
```python
def _optional_float(value: Any) -> float | None:
    if value is None:
        return None
    return float(value)
```
**EN:** This function drives `_optional_float` with inputs such as `value`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_optional_float`，主要处理 `value` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 334-345: Function `_optional_bool` / 函数 `_optional_bool`
```python
def _optional_bool(value: Any) -> bool | None:
    if value is None:
        return None
    if isinstance(value, bool):
        return value
    if isinstance(value, str):
        lowered = value.strip().lower()
        if lowered in ("1", "true", "yes", "y", "on"):
            return True
        if lowered in ("0", "false", "no", "n", "off"):
            return False
    raise ValueError(f"cannot parse boolean batching config value: {value!r}")
```
**EN:** This function drives `_optional_bool` with inputs such as `value`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_optional_bool`，主要处理 `value` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Symbol `AdmissionLimit` anchors the module API / 符号 `AdmissionLimit` 构成该模块的核心 API
- Symbol `BatchingRule` anchors the module API / 符号 `BatchingRule` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.server_args`
- **External / 外部**: `__future__`, `difflib`
- **Stdlib / 标准库**: `json`, `os`, `dataclasses`, `typing`
