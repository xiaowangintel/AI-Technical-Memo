# azure.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/connector/azure.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the external connector integrations part of the SRT runtime and implements logic centered on `azure`. It exposes primary entry points such as `_filter_allow`, `_filter_ignore`, `_normalize_url`. / 该模块属于 SRT 运行时的外部连接器集成部分，主要实现围绕 `azure` 的逻辑。 它对外提供的主要入口包括 `_filter_allow`, `_filter_ignore`, `_normalize_url`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0

import fnmatch
import os
from pathlib import Path
from typing import Generator, Optional, Tuple

import torch

from sglang.srt.connector import BaseFileConnector


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 13-18: Function _filter_allow
```python
def _filter_allow(paths: list[str], patterns: list[str]) -> list[str]:
    return [
        path
        for path in paths
        if any(fnmatch.fnmatch(path, pattern) for pattern in patterns)
    ]
```
**EN:** This callable implements `_filter_allow`. It takes `paths`, `patterns` and mainly implements filter allow.
**CN:** 这一可调用对象实现了 `_filter_allow`。它接收 `paths`, `patterns`，主要用于实现 filter allow 相关逻辑。

### Lines 21-26: Function _filter_ignore
```python
def _filter_ignore(paths: list[str], patterns: list[str]) -> list[str]:
    return [
        path
        for path in paths
        if not any(fnmatch.fnmatch(path, pattern) for pattern in patterns)
    ]
```
**EN:** This callable implements `_filter_ignore`. It takes `paths`, `patterns` and mainly implements filter ignore.
**CN:** 这一可调用对象实现了 `_filter_ignore`。它接收 `paths`, `patterns`，主要用于实现 filter ignore 相关逻辑。

### Lines 29-31: Function _normalize_url
```python
def _normalize_url(url: str) -> str:
    """Strip trailing slash so blobfile glob/listdir behave consistently."""
    return url.rstrip("/")
```
**EN:** This callable implements `_normalize_url`. It takes `url` and mainly normalizes user-provided values. The docstring states: "Strip trailing slash so blobfile glob/listdir behave consistently."
**CN:** 这一可调用对象实现了 `_normalize_url`。它接收 `url`，主要用于规范化用户提供的值。

### Lines 34-63: Function list_files
```python
def list_files(
    bf,
    path: str,
    allow_pattern: Optional[list[str]] = None,
    ignore_pattern: Optional[list[str]] = None,
) -> Tuple[str, list[str]]:
    """List files from an Azure Blob Storage path and filter by pattern.

    Args:
        bf: The ``blobfile`` module.
        path: An ``az://<account>/<container>/<prefix>`` or
            ``https://<account>.blob.core.windows.net/<container>/<prefix>`` URL.
        allow_pattern: A list of fnmatch patterns of which files to keep.
        ignore_pattern: A list of fnmatch patterns of which files to drop.

    Returns:
        A tuple ``(base_dir, files)`` where ``base_dir`` is the normalized
        prefix used as a directory anchor for relative paths, and ``files``
        is the list of full URLs matched by the patterns.
    """
    base_dir = _normalize_url(path)
    files = [p for p in bf.glob(base_dir + "/**") if not bf.isdir(p)]

    files = _filter_ignore(files, ["*/"])
    if allow_pattern is not None:
        files = _filter_allow(files, allow_pattern)
    if ignore_pattern is not None:
        files = _filter_ignore(files, ignore_pattern)

    return base_dir, files
```
**EN:** This callable implements `list_files`. It takes `bf`, `path`, `allow_pattern`, `ignore_pattern` and mainly implements list files. The docstring states: "List files from an Azure Blob Storage path and filter by pattern." In this range it sets up imports and shared symbols; talks to external storage or service backends.
**CN:** 这一可调用对象实现了 `list_files`。它接收 `bf`, `path`, `allow_pattern`, `ignore_pattern`，主要用于实现 list files 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；与外部存储或服务后端交互。

### Lines 66-74: Class AzureBlobConnector
```python
class AzureBlobConnector(BaseFileConnector):
    """File connector for Azure Blob Storage.

    Accepts both ``az://<account>/<container>/<path>`` URLs and HTTPS URLs of
    the form ``https://<account>.blob.core.windows.net/<container>/<path>``.
    Uses the third-party ``blobfile`` package, which handles authentication via
    standard Azure credential chains (env vars, az CLI, managed identity).
    """

```
**EN:** This range introduces `AzureBlobConnector` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "File connector for Azure Blob Storage." In this range it talks to external storage or service backends.
**CN:** 这一段引入 `AzureBlobConnector`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会与外部存储或服务后端交互。

### Lines 75-85: Method AzureBlobConnector.__init__
```python
    def __init__(self, url: str) -> None:
        try:
            import blobfile as bf
        except ImportError as e:
            raise ImportError(
                "AzureBlobConnector requires the 'blobfile' package. "
                "Install it with `pip install blobfile`."
            ) from e

        super().__init__(url)
        self.bf = bf
```
**EN:** This callable implements `AzureBlobConnector.__init__`. It takes `url` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; talks to external storage or service backends.
**CN:** 这一可调用对象实现了 `AzureBlobConnector.__init__`。它接收 `url`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；与外部存储或服务后端交互。

### Lines 87-89: Method AzureBlobConnector.glob
```python
    def glob(self, allow_pattern: Optional[list[str]] = None) -> list[str]:
        _, files = list_files(self.bf, self.url, allow_pattern=allow_pattern)
        return files
```
**EN:** This callable implements `AzureBlobConnector.glob`. It takes `allow_pattern` and mainly implements glob.
**CN:** 这一可调用对象实现了 `AzureBlobConnector.glob`。它接收 `allow_pattern`，主要用于实现 glob 相关逻辑。

### Lines 91-105: Method AzureBlobConnector.pull_files
```python
    def pull_files(
        self,
        allow_pattern: Optional[list[str]] = None,
        ignore_pattern: Optional[list[str]] = None,
    ) -> None:
        """Download files from Azure Blob Storage to ``self.local_dir``."""
        base_dir, files = list_files(self.bf, self.url, allow_pattern, ignore_pattern)
        if not files:
            return

        for file in files:
            relative = file[len(base_dir) :].lstrip("/")
            destination_file = os.path.join(self.local_dir, relative)
            os.makedirs(Path(destination_file).parent, exist_ok=True)
            self.bf.copy(file, destination_file, overwrite=True)
```
**EN:** This callable implements `AzureBlobConnector.pull_files`. It takes `allow_pattern`, `ignore_pattern` and mainly implements pull files. The docstring states: "Download files from Azure Blob Storage to ``self.local_dir``." In this range it sets up imports and shared symbols; talks to external storage or service backends.
**CN:** 这一可调用对象实现了 `AzureBlobConnector.pull_files`。它接收 `allow_pattern`, `ignore_pattern`，主要用于实现 pull files 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；与外部存储或服务后端交互。

### Lines 107-124: Method AzureBlobConnector.weight_iterator
```python
    def weight_iterator(
        self, rank: int = 0
    ) -> Generator[Tuple[str, torch.Tensor], None, None]:
        from sglang.srt.model_loader.weight_utils import (
            runai_safetensors_weights_iterator,
        )

        # Pull *.safetensors locally first since runai_safetensors_weights_iterator
        # expects local files. blobfile does not provide a streaming safetensors
        # reader compatible with runai_model_streamer.
        self.pull_files(allow_pattern=["*.safetensors"])
        local_files = [
            os.path.join(root, f)
            for root, _, fs in os.walk(self.local_dir)
            for f in fs
            if f.endswith(".safetensors")
        ]
        return runai_safetensors_weights_iterator(local_files)
```
**EN:** This callable implements `AzureBlobConnector.weight_iterator`. It takes `rank` and mainly converts data into another representation. In this range it sets up imports and shared symbols; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `AzureBlobConnector.weight_iterator`。它接收 `rank`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；管理模型权重或检查点。

### Lines 126-127: Method AzureBlobConnector.close
```python
    def close(self):
        super().close()
```
**EN:** This callable implements `AzureBlobConnector.close` and mainly implements close.
**CN:** 这一可调用对象实现了 `AzureBlobConnector.close`，主要用于实现 close 相关逻辑。

## Key Concepts / 关键概念
- `_filter_allow`: implements filter allow / 实现 filter allow 相关逻辑
- `_filter_ignore`: implements filter ignore / 实现 filter ignore 相关逻辑
- `_normalize_url`: normalizes user-provided values / 规范化用户提供的值
- `list_files`: implements list files / 实现 list files 相关逻辑
- `AzureBlobConnector`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `fnmatch`, `os`, `pathlib`, `typing`
- **Third-party / 第三方**: `torch`, `blobfile`
- **Internal modules / 内部模块**: `sglang.srt.connector`, `sglang.srt.model_loader.weight_utils`
