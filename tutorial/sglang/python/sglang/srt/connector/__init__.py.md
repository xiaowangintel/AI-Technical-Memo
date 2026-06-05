# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/connector/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the external connector integrations part of the SRT runtime and implements logic centered on `__init__`. It exposes primary entry points such as `ConnectorType`, `_is_azure_blob_url`, `create_remote_connector`. / 该模块属于 SRT 运行时的外部连接器集成部分，主要实现围绕 `__init__` 的逻辑。 它对外提供的主要入口包括 `ConnectorType`, `_is_azure_blob_url`, `create_remote_connector`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0

import enum
import logging

from sglang.srt.connector.base_connector import (
    BaseConnector,
    BaseFileConnector,
    BaseKVConnector,
)
from sglang.srt.connector.redis import RedisConnector
from sglang.srt.connector.remote_instance import RemoteInstanceConnector
from sglang.srt.connector.s3 import S3Connector
from sglang.srt.utils import parse_connector_type

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; talks to external storage or service backends.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；与外部存储或服务后端交互。

### Lines 19-22: Class ConnectorType
```python
class ConnectorType(str, enum.Enum):
    FS = "filesystem"
    KV = "KV"
    INSTANCE = "instance"
```
**EN:** This range introduces `ConnectorType` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ConnectorType`，并定义其后续方法依赖的结构或元数据。

### Lines 25-33: Function _is_azure_blob_url
```python
def _is_azure_blob_url(url: str, connector_type: str) -> bool:
    """Detect Azure Blob Storage URLs.

    Matches ``az://...`` URLs and ``https://<account>.blob.core.windows.net/...``
    URLs, which are the two forms accepted by the ``blobfile`` library.
    """
    if connector_type == "az":
        return True
    return connector_type == "https" and ".blob.core.windows.net" in url
```
**EN:** This callable implements `_is_azure_blob_url`. It takes `url`, `connector_type` and mainly implements is azure blob url. The docstring states: "Detect Azure Blob Storage URLs." In this range it talks to external storage or service backends.
**CN:** 这一可调用对象实现了 `_is_azure_blob_url`。它接收 `url`, `connector_type`，主要用于实现 is azure blob url 相关逻辑。 在这一范围内，它会与外部存储或服务后端交互。

### Lines 36-51: Function create_remote_connector
```python
def create_remote_connector(url, device=None, **kwargs) -> BaseConnector:
    connector_type = parse_connector_type(url)
    if connector_type == "redis":
        return RedisConnector(url)
    elif connector_type == "s3":
        return S3Connector(url)
    elif connector_type == "instance":
        return RemoteInstanceConnector(url, device)
    elif _is_azure_blob_url(url, connector_type):
        # Imported lazily so the optional ``blobfile`` dependency is only
        # required when an Azure URL is actually used.
        from sglang.srt.connector.azure import AzureBlobConnector

        return AzureBlobConnector(url)
    else:
        raise ValueError(f"Invalid connector type: {url}")
```
**EN:** This callable implements `create_remote_connector`. It takes `url`, `device`, `**kwargs` and mainly converts data into another representation. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; talks to external storage or service backends.
**CN:** 这一可调用对象实现了 `create_remote_connector`。它接收 `url`, `device`, `**kwargs`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；与外部存储或服务后端交互。

### Lines 54-62: Function get_connector_type
```python
def get_connector_type(client: BaseConnector) -> ConnectorType:
    if isinstance(client, BaseKVConnector):
        return ConnectorType.KV
    if isinstance(client, BaseFileConnector):
        return ConnectorType.FS
    if isinstance(client, RemoteInstanceConnector):
        return ConnectorType.INSTANCE

    raise ValueError(f"Invalid connector type: {client}")
```
**EN:** This callable implements `get_connector_type`. It takes `client` and mainly converts data into another representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_connector_type`。它接收 `client`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 63-75: Module-level constants and helpers
```python


__all__ = [
    "BaseConnector",
    "BaseFileConnector",
    "BaseKVConnector",
    "RedisConnector",
    "RemoteInstanceConnector",
    "S3Connector",
    "ConnectorType",
    "create_remote_connector",
    "get_connector_type",
]
```
**EN:** This range organizes module-level state and shared setup. In this range it talks to external storage or service backends.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会与外部存储或服务后端交互。

## Key Concepts / 关键概念
- `ConnectorType`: core class or state container / 核心类或状态容器
- `_is_azure_blob_url`: implements is azure blob url / 实现 is azure blob url 相关逻辑
- `create_remote_connector`: converts data into another representation / 将数据转换为另一种表示
- `get_connector_type`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `enum`, `logging`
- **Internal modules / 内部模块**: `sglang.srt.connector.base_connector`, `sglang.srt.connector.redis`, `sglang.srt.connector.remote_instance`, `sglang.srt.connector.s3`, `sglang.srt.utils`, `sglang.srt.connector.azure`
