# base_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/connector/base_connector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the external connector integrations part of the SRT runtime and implements logic centered on `base_connector`. It exposes primary entry points such as `BaseConnector`, `BaseKVConnector`, `BaseFileConnector`. / 该模块属于 SRT 运行时的外部连接器集成部分，主要实现围绕 `base_connector` 的逻辑。 它对外提供的主要入口包括 `BaseConnector`, `BaseKVConnector`, `BaseFileConnector`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0

import os
import shutil
import signal
import tempfile
from abc import ABC, abstractmethod
from typing import Generator, List, Optional, Tuple

import torch


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 13-22: Class BaseConnector
```python
class BaseConnector(ABC):
    """
    For fs connector such as s3:
    <connector_type>://<path>/<filename>

    For kv connector such as redis:
    <connector_type>://<host>:<port>/<model_name>/keys/<key>
    <connector_type://<host>:<port>/<model_name>/files/<filename>
    """

```
**EN:** This range introduces `BaseConnector` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "For fs connector such as s3: <connector_type>://<path>/<filename> For kv connector such as redis: <connector_type>://<host>:<port>/<model_name>/keys/<key> <conn" In this range it talks to external storage or service backends.
**CN:** 这一段引入 `BaseConnector`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会与外部存储或服务后端交互。

### Lines 23-29: Method BaseConnector.__init__
```python
    def __init__(self, url: str):
        self.url = url
        self.closed = False
        self.local_dir = tempfile.mkdtemp()
        for sig in (signal.SIGINT, signal.SIGTERM):
            existing_handler = signal.getsignal(sig)
            signal.signal(sig, self._close_by_signal(existing_handler))
```
**EN:** This callable implements `BaseConnector.__init__`. It takes `url` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `BaseConnector.__init__`。它接收 `url`，主要用于初始化实例状态与默认值。

### Lines 31-32: Method BaseConnector.get_local_dir
```python
    def get_local_dir(self):
        return self.local_dir
```
**EN:** This callable implements `BaseConnector.get_local_dir` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `BaseConnector.get_local_dir`，主要用于获取某个值或派生视图。

### Lines 34-38: Method BaseConnector.weight_iterator
```python
    @abstractmethod
    def weight_iterator(
        self, rank: int = 0
    ) -> Generator[Tuple[str, torch.Tensor], None, None]:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseConnector.weight_iterator`. It takes `rank` and mainly converts data into another representation. In this range it performs defensive checks on invalid state; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `BaseConnector.weight_iterator`。它接收 `rank`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查；管理模型权重或检查点。

### Lines 40-46: Method BaseConnector.pull_files
```python
    @abstractmethod
    def pull_files(
        self,
        allow_pattern: Optional[List[str]] = None,
        ignore_pattern: Optional[List[str]] = None,
    ) -> None:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseConnector.pull_files`. It takes `allow_pattern`, `ignore_pattern` and mainly implements pull files. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseConnector.pull_files`。它接收 `allow_pattern`, `ignore_pattern`，主要用于实现 pull files 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 48-54: Method BaseConnector.close
```python
    def close(self):
        if self.closed:
            return

        self.closed = True
        if os.path.exists(self.local_dir):
            shutil.rmtree(self.local_dir)
```
**EN:** This callable implements `BaseConnector.close` and mainly implements close.
**CN:** 这一可调用对象实现了 `BaseConnector.close`，主要用于实现 close 相关逻辑。

### Lines 56-57: Method BaseConnector.__enter__
```python
    def __enter__(self):
        return self
```
**EN:** This callable implements `BaseConnector.__enter__` and mainly implements enter.
**CN:** 这一可调用对象实现了 `BaseConnector.__enter__`，主要用于实现 enter 相关逻辑。

### Lines 59-60: Method BaseConnector.__exit__
```python
    def __exit__(self, exc_type, exc_value, traceback):
        self.close()
```
**EN:** This callable implements `BaseConnector.__exit__`. It takes `exc_type`, `exc_value`, `traceback` and mainly implements exit. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `BaseConnector.__exit__`。它接收 `exc_type`, `exc_value`, `traceback`，主要用于实现 exit 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 62-63: Method BaseConnector.__del__
```python
    def __del__(self):
        self.close()
```
**EN:** This callable implements `BaseConnector.__del__` and mainly implements del.
**CN:** 这一可调用对象实现了 `BaseConnector.__del__`，主要用于实现 del 相关逻辑。

### Lines 65-72: Method BaseConnector._close_by_signal
```python
    def _close_by_signal(self, existing_handler=None):

        def new_handler(signum, frame):
            self.close()
            if existing_handler:
                existing_handler(signum, frame)

        return new_handler
```
**EN:** This callable implements `BaseConnector._close_by_signal`. It takes `existing_handler` and mainly implements close by signal.
**CN:** 这一可调用对象实现了 `BaseConnector._close_by_signal`。它接收 `existing_handler`，主要用于实现 close by signal 相关逻辑。

### Lines 75-76: Class BaseKVConnector
```python
class BaseKVConnector(BaseConnector):

```
**EN:** This range introduces `BaseKVConnector` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `BaseKVConnector`，并定义其后续方法依赖的结构或元数据。

### Lines 77-79: Method BaseKVConnector.get
```python
    @abstractmethod
    def get(self, key: str) -> Optional[torch.Tensor]:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseKVConnector.get`. It takes `key` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseKVConnector.get`。它接收 `key`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 81-83: Method BaseKVConnector.getstr
```python
    @abstractmethod
    def getstr(self, key: str) -> Optional[str]:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseKVConnector.getstr`. It takes `key` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseKVConnector.getstr`。它接收 `key`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 85-87: Method BaseKVConnector.set
```python
    @abstractmethod
    def set(self, key: str, obj: torch.Tensor) -> None:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseKVConnector.set`. It takes `key`, `obj` and mainly applies configuration to mutable state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseKVConnector.set`。它接收 `key`, `obj`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 89-91: Method BaseKVConnector.setstr
```python
    @abstractmethod
    def setstr(self, key: str, obj: str) -> None:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseKVConnector.setstr`. It takes `key`, `obj` and mainly applies configuration to mutable state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseKVConnector.setstr`。它接收 `key`, `obj`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 93-95: Method BaseKVConnector.list
```python
    @abstractmethod
    def list(self, prefix: str) -> List[str]:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseKVConnector.list`. It takes `prefix` and mainly implements list. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseKVConnector.list`。它接收 `prefix`，主要用于实现 list 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 98-108: Class BaseFileConnector
```python
class BaseFileConnector(BaseConnector):
    """
    List full file names from remote fs path and filter by allow pattern.

    Args:
        allow_pattern: A list of patterns of which files to pull.

    Returns:
        list[str]: List of full paths allowed by the pattern
    """

```
**EN:** This range introduces `BaseFileConnector` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "List full file names from remote fs path and filter by allow pattern." In this range it sets up imports and shared symbols.
**CN:** 这一段引入 `BaseFileConnector`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 109-111: Method BaseFileConnector.glob
```python
    @abstractmethod
    def glob(self, allow_pattern: str) -> List[str]:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseFileConnector.glob`. It takes `allow_pattern` and mainly implements glob. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseFileConnector.glob`。它接收 `allow_pattern`，主要用于实现 glob 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `BaseConnector`: core class or state container / 核心类或状态容器
- `BaseKVConnector`: core class or state container / 核心类或状态容器
- `BaseFileConnector`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `shutil`, `signal`, `tempfile`, `abc`, `typing`
- **Third-party / 第三方**: `torch`
