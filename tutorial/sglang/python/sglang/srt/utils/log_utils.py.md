# log_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/log_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `log_utils` and the surrounding SGLang serving stack. / 提供围绕 `log_utils` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

import json
import logging
import os
import socket
import sys
from datetime import datetime
from logging.handlers import TimedRotatingFileHandler
from typing import List, Optional, Union

import torch.distributed as dist
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `json`, `logging`, `os`, `socket`, `sys`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `json`, `logging`, `os`, `socket`, `sys`。

### Lines 15-20: Function `create_log_targets` / 函数 `create_log_targets`
```python
def create_log_targets(
    *, targets: Optional[List[str]], name_prefix: str
) -> List[logging.Logger]:
    if not targets:
        return [_create_log_target_stdout(name_prefix)]
    return [_create_log_target(t, name_prefix) for t in targets]
```
**EN:** This function implements `create_log_targets`. It primarily calls `_create_log_target`, `_create_log_target_stdout` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `create_log_targets`。 它主要通过调用 `_create_log_target`, `_create_log_target_stdout` 来完成任务。 实现中使用了条件分支。

### Lines 23-26: Function `_create_log_target` / 函数 `_create_log_target`
```python
def _create_log_target(target: str, name_prefix: str) -> logging.Logger:
    if target.lower() == "stdout":
        return _create_log_target_stdout(name_prefix)
    return _create_log_target_file(target, name_prefix)
```
**EN:** This function implements `_create_log_target`. It primarily calls `_create_log_target_file`, `target.lower`, `_create_log_target_stdout` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_create_log_target`。 它主要通过调用 `_create_log_target_file`, `target.lower`, `_create_log_target_stdout` 来完成任务。 实现中使用了条件分支。

### Lines 29-32: Function `_create_log_target_stdout` / 函数 `_create_log_target_stdout`
```python
def _create_log_target_stdout(name_prefix: str) -> logging.Logger:
    return _create_logger_with_handler(
        f"{name_prefix}.stdout", logging.StreamHandler(sys.stdout)
    )
```
**EN:** This function implements `_create_log_target_stdout`. It primarily calls `_create_logger_with_handler`, `logging.StreamHandler` to complete its work.
**CN:** 该函数实现了 `_create_log_target_stdout`。 它主要通过调用 `_create_logger_with_handler`, `logging.StreamHandler` 来完成任务。

### Lines 35-45: Function `_create_log_target_file` / 函数 `_create_log_target_file`
```python
def _create_log_target_file(directory: str, name_prefix: str) -> logging.Logger:
    os.makedirs(directory, exist_ok=True)
    hostname = socket.gethostname()
    rank = dist.get_rank() if dist.is_initialized() else 0
    filename = os.path.join(directory, f"{hostname}_{rank}.log")
    handler = TimedRotatingFileHandler(
        filename, when="H", backupCount=0, encoding="utf-8"
    )
    return _create_logger_with_handler(
        f"{name_prefix}.file.{directory}.{hostname}_{rank}", handler
    )
```
**EN:** This function implements `_create_log_target_file`. It primarily calls `os.makedirs`, `socket.gethostname`, `os.path.join`, `TimedRotatingFileHandler`, `_create_logger_with_handler`, `dist.is_initialized` to complete its work. State updates are written into `hostname`, `rank`, `filename`, `handler`.
**CN:** 该函数实现了 `_create_log_target_file`。 它主要通过调用 `os.makedirs`, `socket.gethostname`, `os.path.join`, `TimedRotatingFileHandler`, `_create_logger_with_handler`, `dist.is_initialized` 来完成任务。 状态更新主要写入 `hostname`, `rank`, `filename`, `handler`。

### Lines 48-57: Function `_create_logger_with_handler` / 函数 `_create_logger_with_handler`
```python
def _create_logger_with_handler(name: str, handler: logging.Handler) -> logging.Logger:
    logger = logging.getLogger(name)
    logger.setLevel(logging.INFO)
    logger.propagate = False
    if not logger.handlers:
        handler.setFormatter(
            logging.Formatter("[%(asctime)s] %(message)s", datefmt="%Y-%m-%d %H:%M:%S")
        )
        logger.addHandler(handler)
    return logger
```
**EN:** This function implements `_create_logger_with_handler`. It primarily calls `logging.getLogger`, `logger.setLevel`, `handler.setFormatter`, `logger.addHandler`, `logging.Formatter` to complete its work. State updates are written into `logger`, `logger.propagate`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_create_logger_with_handler`。 它主要通过调用 `logging.getLogger`, `logger.setLevel`, `handler.setFormatter`, `logger.addHandler`, `logging.Formatter` 来完成任务。 状态更新主要写入 `logger`, `logger.propagate`。 实现中使用了条件分支。

### Lines 60-74: Function `log_json` / 函数 `log_json`
```python
def log_json(
    loggers: Union[logging.Logger, List[logging.Logger]], event: str, data: dict
) -> None:
    log_data = {
        "timestamp": datetime.now().isoformat(),
        "event": event,
        **data,
    }
    msg = json.dumps(log_data, ensure_ascii=False)

    if not isinstance(loggers, list):
        loggers = [loggers]

    for logger in loggers:
        logger.info(msg)
```
**EN:** This function implements `log_json`. It primarily calls `json.dumps`, `datetime.now.isoformat`, `isinstance`, `logger.info`, `datetime.now` to complete its work. State updates are written into `log_data`, `msg`, `loggers`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `log_json`。 它主要通过调用 `json.dumps`, `datetime.now.isoformat`, `isinstance`, `logger.info`, `datetime.now` 来完成任务。 状态更新主要写入 `log_data`, `msg`, `loggers`。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Functions / 函数**: `create_log_targets`, `_create_log_target`, `_create_log_target_stdout`, `_create_log_target_file`, `_create_logger_with_handler`, `log_json`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch.distributed`
- **Standard library / 标准库**: `__future__`, `json`, `logging`, `os`, `socket`, `sys`, `datetime`, `logging.handlers`, `typing`
