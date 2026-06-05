# usage_lib.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/usage/usage_lib.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `usage_lib`-related logic centered around `UsageContext`, `UsageMessage`, `set_runtime_usage_data`, `is_usage_stats_enabled`. / 实现与 `usage_lib` 相关的逻辑，核心符号包括 `UsageContext`, `UsageMessage`, `set_runtime_usage_data`, `is_usage_stats_enabled`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-43)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import datetime
import json
import logging
import os
import platform
import time
from enum import Enum
from pathlib import Path
from threading import Thread
from typing import Any
from uuid import uuid4

import cpuinfo
import psutil
import requests
import torch

import vllm.envs as envs
from vllm.connections import global_http_connection
from vllm.logger import init_logger
from vllm.utils.platform_utils import cuda_get_device_properties
    # ...
    "VLLM_USE_MODELSCOPE",
    "VLLM_USE_FLASHINFER_SAMPLER",
    "VLLM_PP_LAYER_PARTITION",
    "VLLM_USE_TRITON_AWQ",
    "VLLM_ENABLE_V1_MULTIPROCESSING",
]
```
**EN:** Sets up the module with standard-library support such as `datetime`, `json`, `logging`, external packages such as `cpuinfo`, `psutil`, `requests`, vLLM modules such as `vllm.envs`, `vllm.connections`, `vllm.logger`. It prepares the symbols later used by `UsageContext`, `UsageMessage`, `set_runtime_usage_data`, `is_usage_stats_enabled`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.connections`, `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `UsageContext`, `UsageMessage`, `set_runtime_usage_data`, `is_usage_stats_enabled` 提供上下文。

### set_runtime_usage_data (lines 46-48)
```python
def set_runtime_usage_data(key: str, value: str | int | bool) -> None:
    """Set global usage data that will be sent with every usage heartbeat."""
    _GLOBAL_RUNTIME_DATA[key] = value
```
**EN:** `set_runtime_usage_data`: Set global usage data that will be sent with every usage heartbeat. It mainly works with `key`, `value`.
**CN:** `set_runtime_usage_data` 负责把值写入对象或配置结构。 它主要处理 `key`, `value` 等参数。

### is_usage_stats_enabled (lines 51-69)
```python
def is_usage_stats_enabled():
    """Determine whether or not we can send usage stats to the server.
    The logic is as follows:
    - By default, it should be enabled.
    - Three environment variables can disable it:
        - VLLM_DO_NOT_TRACK=1
        - DO_NOT_TRACK=1
        - VLLM_NO_USAGE_STATS=1
    - A file in the home directory can disable it if it exists:
        - $HOME/.config/vllm/do_not_track
    """
    global _USAGE_STATS_ENABLED
    if _USAGE_STATS_ENABLED is None:
        do_not_track = envs.VLLM_DO_NOT_TRACK
        no_usage_stats = envs.VLLM_NO_USAGE_STATS
        do_not_track_file = os.path.exists(_USAGE_STATS_DO_NOT_TRACK_PATH)

        _USAGE_STATS_ENABLED = not (do_not_track or no_usage_stats or do_not_track_file)
    return _USAGE_STATS_ENABLED
```
**EN:** `is_usage_stats_enabled`: Determine whether or not we can send usage stats to the server. Inside the body, it relies on `os.path.exists` to complete the main steps.
**CN:** `is_usage_stats_enabled` 负责检查条件并返回布尔结果。 实现过程中会调用 `os.path.exists` 等函数完成关键步骤。

### _get_current_timestamp_ns (lines 72-73)
```python
def _get_current_timestamp_ns() -> int:
    return int(datetime.datetime.now(datetime.timezone.utc).timestamp() * 1e9)
```
**EN:** `_get_current_timestamp_ns` retrieves data or state needed by the pipeline. Inside the body, it relies on `datetime.datetime.now.timestamp`, `datetime.datetime.now` to complete the main steps.
**CN:** `_get_current_timestamp_ns` 负责获取流水线所需的数据或状态。 实现过程中会调用 `datetime.datetime.now.timestamp`, `datetime.datetime.now` 等函数完成关键步骤。

### _detect_cloud_provider (lines 76-109)
```python
def _detect_cloud_provider() -> str:
    # Try detecting through vendor file
    vendor_files = [
        "/sys/class/dmi/id/product_version",
        "/sys/class/dmi/id/bios_vendor",
        "/sys/class/dmi/id/product_name",
        "/sys/class/dmi/id/chassis_asset_tag",
        "/sys/class/dmi/id/sys_vendor",
    ]
    # Mapping of identifiable strings to cloud providers
    cloud_identifiers = {
        "amazon": "AWS",
        "microsoft corporation": "AZURE",
        "google": "GCP",
        "oraclecloud": "OCI",
    }

    for vendor_file in vendor_files:
        path = Path(vendor_file)
        if path.is_file():
            file_content = path.read_text().lower()
            for identifier, provider in cloud_identifiers.items():
                if identifier in file_content:
                    return provider
    # ...
    }
    for env_var, provider in env_to_cloud_provider.items():
        if os.environ.get(env_var):
            return provider

    return "UNKNOWN"
```
**EN:** `_detect_cloud_provider` implements helper logic used by this module. Inside the body, it relies on `env_to_cloud_provider.items`, `Path`, `path.is_file` to complete the main steps.
**CN:** `_detect_cloud_provider` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `env_to_cloud_provider.items`, `Path`, `path.is_file` 等函数完成关键步骤。

### UsageContext (lines 112-118)
```python
class UsageContext(str, Enum):
    UNKNOWN_CONTEXT = "UNKNOWN_CONTEXT"
    LLM_CLASS = "LLM_CLASS"
    API_SERVER = "API_SERVER"
    OPENAI_API_SERVER = "OPENAI_API_SERVER"
    OPENAI_BATCH_RUNNER = "OPENAI_BATCH_RUNNER"
    ENGINE_CONTEXT = "ENGINE_CONTEXT"
```
**EN:** Defines the `UsageContext` class used by this module. It extends `str`, `Enum`.
**CN:** `UsageContext` 是该文件中的核心类，用于封装与 `UsageContext` 相关的状态和行为。 它继承自 `str`, `Enum`。

### UsageMessage overview (lines 121-279)
```python
class UsageMessage:
    """Collect platform information and send it to the usage stats server."""

    def __init__(self) -> None:
        # NOTE: vLLM's server _only_ support flat KV pair.
        # Do not use nested fields.

        self.uuid = str(uuid4())

        # Environment Information
        self.provider: str | None = None
        self.num_cpu: int | None = None
        self.cpu_type: str | None = None
        self.cpu_family_model_stepping: str | None = None
        self.total_memory: int | None = None
        self.architecture: str | None = None
        self.platform: str | None = None
        self.xpu_runtime: str | None = None
        self.cuda_runtime: str | None = None
        self.gpu_count: int | None = None
        self.gpu_type: str | None = None
        self.gpu_memory_per_device: int | None = None
        self.env_var_json: str | None = None

        # vLLM Information
        self.model_architecture: str | None = None
        self.vllm_version: str | None = None
    # ...
```
**EN:** `UsageMessage`: Collect platform information and send it to the usage stats server. Key methods include `__init__`, `report_usage`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `UsageMessage` 是该文件中的核心类，用于封装与 `UsageMessage` 相关的状态和行为。 关键方法包括 `__init__`, `report_usage`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### UsageMessage.__init__ (lines 124-152)
```python
    def __init__(self) -> None:
        # NOTE: vLLM's server _only_ support flat KV pair.
        # Do not use nested fields.

        self.uuid = str(uuid4())

        # Environment Information
        self.provider: str | None = None
        self.num_cpu: int | None = None
        self.cpu_type: str | None = None
        self.cpu_family_model_stepping: str | None = None
        self.total_memory: int | None = None
        self.architecture: str | None = None
        self.platform: str | None = None
        self.xpu_runtime: str | None = None
        self.cuda_runtime: str | None = None
        self.gpu_count: int | None = None
        self.gpu_type: str | None = None
        self.gpu_memory_per_device: int | None = None
        self.env_var_json: str | None = None
    # ...
        self.vllm_version: str | None = None
        self.context: str | None = None

        # Metadata
        self.log_time: int | None = None
        self.source: str | None = None
```
**EN:** `__init__` initializes state required by the module. Inside the body, it relies on `uuid4` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 实现过程中会调用 `uuid4` 等函数完成关键步骤。

### UsageMessage._report_usage_once (lines 188-247)
```python
    def _report_usage_once(
        self,
        model_architecture: str,
        usage_context: UsageContext,
        extra_kvs: dict[str, Any],
    ) -> None:
        # Platform information
        from vllm.platforms import current_platform

        if current_platform.is_cuda_alike():
            self.gpu_count = current_platform.device_count()
            self.gpu_type, self.gpu_memory_per_device = cuda_get_device_properties(
                0, ("name", "total_memory")
            )
        if current_platform.is_cuda():
            self.cuda_runtime = torch.version.cuda
        if current_platform.is_xpu():
            self.xpu_runtime = torch.version.xpu
            self.gpu_count = torch.xpu.device_count()
            self.gpu_type = torch.xpu.get_device_name(0)
    # ...
        data = vars(self)
        if extra_kvs:
            data.update(extra_kvs)

        self._write_to_file(data)
        self._send_to_server(data)
```
**EN:** `_report_usage_once` implements helper logic used by this module. It mainly works with `model_architecture`, `usage_context`, `extra_kvs`. Inside the body, it relies on `current_platform.is_cuda_alike`, `current_platform.is_cuda`, `current_platform.is_xpu` to complete the main steps.
**CN:** `_report_usage_once` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_architecture`, `usage_context`, `extra_kvs` 等参数。 实现过程中会调用 `current_platform.is_cuda_alike`, `current_platform.is_cuda`, `current_platform.is_xpu` 等函数完成关键步骤。

### UsageMessage.report_usage (lines 154-165)
```python
    def report_usage(
        self,
        model_architecture: str,
        usage_context: UsageContext,
        extra_kvs: dict[str, Any] | None = None,
    ) -> None:
        t = Thread(
            target=self._report_usage_worker,
            args=(model_architecture, usage_context, extra_kvs or {}),
            daemon=True,
        )
        t.start()
```
**EN:** `report_usage` implements helper logic used by this module. It mainly works with `model_architecture`, `usage_context`, `extra_kvs`. Inside the body, it relies on `Thread`, `t.start` to complete the main steps.
**CN:** `report_usage` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_architecture`, `usage_context`, `extra_kvs` 等参数。 实现过程中会调用 `Thread`, `t.start` 等函数完成关键步骤。

### UsageMessage._report_continuous_usage (lines 249-264)
```python
    def _report_continuous_usage(self):
        """Report usage every 10 minutes.

        This helps us to collect more data points for uptime of vLLM usages.
        This function can also help send over performance metrics over time.
        """
        while True:
            time.sleep(600)
            data = {
                "uuid": self.uuid,
                "log_time": _get_current_timestamp_ns(),
            }
            data.update(_GLOBAL_RUNTIME_DATA)

            self._write_to_file(data)
            self._send_to_server(data)
```
**EN:** `_report_continuous_usage`: Report usage every 10 minutes. Inside the body, it relies on `time.sleep`, `data.update`, `self._write_to_file` to complete the main steps.
**CN:** `_report_continuous_usage` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `time.sleep`, `data.update`, `self._write_to_file` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`UsageContext`**: Core class that organizes module behavior. / **`UsageContext`**：组织模块行为的核心类。
- **`UsageMessage`**: Core class that organizes module behavior. / **`UsageMessage`**：组织模块行为的核心类。
- **`set_runtime_usage_data`**: Key helper or entry point in this file. / **`set_runtime_usage_data`**：本文件中的关键辅助函数或入口。
- **`is_usage_stats_enabled`**: Key helper or entry point in this file. / **`is_usage_stats_enabled`**：本文件中的关键辅助函数或入口。
- **`_get_current_timestamp_ns`**: Key helper or entry point in this file. / **`_get_current_timestamp_ns`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: datetime, json, logging, os, platform, time, enum, pathlib, threading, typing, uuid
- **Third-party / 第三方**: cpuinfo, psutil, requests, torch, tpu_inference
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.connections, vllm.logger, vllm.utils.platform_utils, vllm.version, vllm.platforms
