# redis.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/connector/redis.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the external connector integrations part of the SRT runtime and implements logic centered on `redis`. It exposes primary entry points such as `RedisConnector`. / 该模块属于 SRT 运行时的外部连接器集成部分，主要实现围绕 `redis` 的逻辑。 它对外提供的主要入口包括 `RedisConnector`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0

import logging
from typing import Generator, List, Optional, Tuple
from urllib.parse import urlparse

import torch

from sglang.srt.connector import BaseKVConnector
from sglang.srt.connector.serde import create_serde
from sglang.srt.connector.utils import pull_files_from_db

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 16-17: Class RedisConnector
```python
class RedisConnector(BaseKVConnector):

```
**EN:** This range introduces `RedisConnector` and defines the structure or metadata that its methods rely on. In this range it talks to external storage or service backends.
**CN:** 这一段引入 `RedisConnector`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会与外部存储或服务后端交互。

### Lines 18-26: Method RedisConnector.__init__
```python
    def __init__(self, url: str):
        import redis

        super().__init__(url)
        parsed_url = urlparse(url)
        self.connection = redis.Redis(host=parsed_url.hostname, port=parsed_url.port)
        self.model_name = parsed_url.path.lstrip("/")
        # TODO: more serde options
        self.s, self.d = create_serde("safe")
```
**EN:** This callable implements `RedisConnector.__init__`. It takes `url` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols; talks to external storage or service backends.
**CN:** 这一可调用对象实现了 `RedisConnector.__init__`。它接收 `url`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号；与外部存储或服务后端交互。

### Lines 28-35: Method RedisConnector.get
```python
    def get(self, key: str) -> Optional[torch.Tensor]:
        val = self.connection.get(key)

        if val is None:
            logger.error("Key %s not found", key)
            return None

        return self.d.from_bytes(val)
```
**EN:** This callable implements `RedisConnector.get`. It takes `key` and mainly retrieves a value or derived view. In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `RedisConnector.get`。它接收 `key`，主要用于获取某个值或派生视图。 在这一范围内，它会输出日志以便诊断。

### Lines 37-43: Method RedisConnector.getstr
```python
    def getstr(self, key: str) -> Optional[str]:
        val = self.connection.get(key)
        if val is None:
            logger.error("Key %s not found", key)
            return None

        return val.decode("utf-8")
```
**EN:** This callable implements `RedisConnector.getstr`. It takes `key` and mainly retrieves a value or derived view. In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `RedisConnector.getstr`。它接收 `key`，主要用于获取某个值或派生视图。 在这一范围内，它会输出日志以便诊断。

### Lines 45-47: Method RedisConnector.set
```python
    def set(self, key: str, tensor: torch.Tensor) -> None:
        assert tensor is not None
        self.connection.set(key, self.s.to_bytes(tensor))
```
**EN:** This callable implements `RedisConnector.set`. It takes `key`, `tensor` and mainly applies configuration to mutable state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `RedisConnector.set`。它接收 `key`, `tensor`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 49-50: Method RedisConnector.setstr
```python
    def setstr(self, key: str, obj: str) -> None:
        self.connection.set(key, obj)
```
**EN:** This callable implements `RedisConnector.setstr`. It takes `key`, `obj` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `RedisConnector.setstr`。它接收 `key`, `obj`，主要用于将配置写入可变状态。

### Lines 52-65: Method RedisConnector.list
```python
    def list(self, prefix: str) -> List[str]:
        cursor = 0
        all_keys: List[bytes] = []

        while True:
            ret: Tuple[int, List[bytes]] = self.connection.scan(
                cursor=cursor, match=f"{prefix}*"
            )  # type: ignore
            cursor, keys = ret
            all_keys.extend(keys)
            if cursor == 0:
                break

        return [key.decode("utf-8") for key in all_keys]
```
**EN:** This callable implements `RedisConnector.list`. It takes `prefix` and mainly implements list.
**CN:** 这一可调用对象实现了 `RedisConnector.list`。它接收 `prefix`，主要用于实现 list 相关逻辑。

### Lines 67-74: Method RedisConnector.weight_iterator
```python
    def weight_iterator(
        self, rank: int = 0
    ) -> Generator[Tuple[str, bytes], None, None]:
        keys = self.list(f"{self.model_name}/keys/rank_{rank}/")
        for key in keys:
            val = self.get(key)
            key = key.removeprefix(f"{self.model_name}/keys/rank_{rank}/")
            yield key, val
```
**EN:** This callable implements `RedisConnector.weight_iterator`. It takes `rank` and mainly converts data into another representation. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `RedisConnector.weight_iterator`。它接收 `rank`，主要用于将数据转换为另一种表示。 在这一范围内，它会管理模型权重或检查点。

### Lines 76-81: Method RedisConnector.pull_files
```python
    def pull_files(
        self,
        allow_pattern: Optional[List[str]] = None,
        ignore_pattern: Optional[List[str]] = None,
    ) -> None:
        pull_files_from_db(self, self.model_name, allow_pattern, ignore_pattern)
```
**EN:** This callable implements `RedisConnector.pull_files`. It takes `allow_pattern`, `ignore_pattern` and mainly implements pull files.
**CN:** 这一可调用对象实现了 `RedisConnector.pull_files`。它接收 `allow_pattern`, `ignore_pattern`，主要用于实现 pull files 相关逻辑。

### Lines 83-85: Method RedisConnector.close
```python
    def close(self):
        self.connection.close()
        super().close()
```
**EN:** This callable implements `RedisConnector.close` and mainly implements close.
**CN:** 这一可调用对象实现了 `RedisConnector.close`，主要用于实现 close 相关逻辑。

## Key Concepts / 关键概念
- `RedisConnector`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`, `urllib.parse`
- **Third-party / 第三方**: `torch`, `redis`
- **Internal modules / 内部模块**: `sglang.srt.connector`, `sglang.srt.connector.serde`, `sglang.srt.connector.utils`
