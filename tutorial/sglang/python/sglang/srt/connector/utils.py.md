# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/connector/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the external connector integrations part of the SRT runtime and implements logic centered on `utils`. It exposes primary entry points such as `parse_model_name`, `pull_files_from_db`. / 该模块属于 SRT 运行时的外部连接器集成部分，主要实现围绕 `utils` 的逻辑。 它对外提供的主要入口包括 `parse_model_name`, `pull_files_from_db`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0

import os
from pathlib import Path
from typing import Optional
from urllib.parse import urlparse

from sglang.srt.connector import BaseConnector


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 11-17: Function parse_model_name
```python
def parse_model_name(url: str) -> str:
    """
    Parse the model name from the url.
    Only used for db connector
    """
    parsed_url = urlparse(url)
    return parsed_url.path.lstrip("/")
```
**EN:** This callable implements `parse_model_name`. It takes `url` and mainly parses structured input. The docstring states: "Parse the model name from the url." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `parse_model_name`。它接收 `url`，主要用于解析结构化输入。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 20-35: Function pull_files_from_db
```python
def pull_files_from_db(
    connector: BaseConnector,
    model_name: str,
    allow_pattern: Optional[list[str]] = None,
    ignore_pattern: Optional[list[str]] = None,
) -> None:
    prefix = f"{model_name}/files/"
    local_dir = connector.get_local_dir()
    files = connector.list(prefix)

    for file in files:
        destination_file = os.path.join(local_dir, file.removeprefix(prefix))
        local_dir = Path(destination_file).parent
        os.makedirs(local_dir, exist_ok=True)
        with open(destination_file, "wb") as f:
            f.write(connector.getstr(file).encode("utf-8"))
```
**EN:** This callable implements `pull_files_from_db`. It takes `connector`, `model_name`, `allow_pattern`, `ignore_pattern` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `pull_files_from_db`。它接收 `connector`, `model_name`, `allow_pattern`, `ignore_pattern`，主要用于从外部表示构造数据。

## Key Concepts / 关键概念
- `parse_model_name`: parses structured input / 解析结构化输入
- `pull_files_from_db`: constructs data from an external representation / 从外部表示构造数据

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `pathlib`, `typing`, `urllib.parse`
- **Internal modules / 内部模块**: `sglang.srt.connector`
