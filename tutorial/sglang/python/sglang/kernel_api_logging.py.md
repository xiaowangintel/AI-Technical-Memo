# kernel_api_logging.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/kernel_api_logging.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Kernel API crash debugging helpers for SGLang. / 该文件围绕上述模块职责展开实现，组织核心流程、辅助逻辑以及与相邻组件的协作。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5: Module-level supporting statements
```python
"""Kernel API crash debugging helpers for SGLang.

This module was developed with reference to FlashInfer's kernel API logging utility:
https://github.com/flashinfer-ai/flashinfer/blob/main/flashinfer/api_logging.py
"""
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-6: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: Module-level supporting statements
```python
from __future__ import annotations
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 8-8: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: Module-level supporting statements
```python
import fnmatch
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-10: Module-level supporting statements
```python
import functools
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Module-level supporting statements
```python
import inspect
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python
import json
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
import logging
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-14: Module-level supporting statements
```python
import os
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-15: Module-level supporting statements
```python
import sys
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-16: Module-level supporting statements
```python
from datetime import datetime
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 17-17: Module-level supporting statements
```python
from pathlib import Path
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 18-18: Module-level supporting statements
```python
from typing import Any, Callable, TypeVar, overload
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 19-19: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 20-20: Module-level supporting statements
```python
import torch
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 21-21: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 22-22: Module-level supporting statements
```python
_logger = logging.getLogger("sglang.kernel_api")
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 23-23: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 24-24: Module-level supporting statements
```python
_T = TypeVar("_T")
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 25-25: Module-level supporting statements
```python
_F = TypeVar("_F", bound=Callable[..., Any])
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 26-27: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 28-31: str with pid function
```python
def _str_with_pid(path: str) -> str:
    if "%i" in path:
        return path.replace("%i", str(os.getpid()))
    return path
```
**EN:** This block uses `_str_with_pid` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_str_with_pid` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 32-33: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 34-48: get env function
```python
def _get_env(key: str, type: Callable[..., _T], default: _T) -> _T:
    value_str = os.environ.get(key, None)
    if value_str is None:
        return default
    try:
        return type(value_str)
    except Exception:
        _logger.warning(
            "Failed to parse environment variable %s=%r as %s, using default %r",
            key,
            value_str,
            type.__name__,
            default,
        )
        return default
```
**EN:** This block uses `_get_env` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_get_env` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 49-50: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 51-52: parse pattern function
```python
def _parse_pattern(value: str) -> list[str]:
    return [p.strip() for p in value.split(",") if p.strip()]
```
**EN:** This block uses `_parse_pattern` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_parse_pattern` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 53-54: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 55-55: Module-level supporting statements
```python
_KERNEL_API_LOG_LEVEL = _get_env("SGLANG_KERNEL_API_LOGLEVEL", int, 0)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 56-56: Module-level supporting statements
```python
_KERNEL_API_LOG_DEST = _get_env("SGLANG_KERNEL_API_LOGDEST", _str_with_pid, "stdout")
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 57-59: Module-level supporting statements
```python
_DUMP_DIR = Path(
    _get_env("SGLANG_KERNEL_API_DUMP_DIR", _str_with_pid, "sglang_kernel_api_dumps")
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 60-60: Module-level supporting statements
```python
_DUMP_INCLUDE_PATTERNS = _get_env("SGLANG_KERNEL_API_DUMP_INCLUDE", _parse_pattern, [])
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 61-61: Module-level supporting statements
```python
_DUMP_EXCLUDE_PATTERNS = _get_env("SGLANG_KERNEL_API_DUMP_EXCLUDE", _parse_pattern, [])
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 62-62: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 63-63: Module-level supporting statements
```python
_dump_call_counter: dict[str, int] = {}
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 64-65: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 66-90: setup logger function
```python
def _setup_logger() -> None:
    for handler in list(_logger.handlers):
        _logger.removeHandler(handler)
        try:
            handler.close()
        except Exception:
            pass

    if _KERNEL_API_LOG_LEVEL == 0:
        _logger.addHandler(logging.NullHandler())
        _logger.setLevel(logging.CRITICAL + 1)
        return

    _logger.setLevel(logging.DEBUG)

    if _KERNEL_API_LOG_DEST == "stdout":
        handler = logging.StreamHandler(sys.stdout)
    elif _KERNEL_API_LOG_DEST == "stderr":
        handler = logging.StreamHandler(sys.stderr)
    else:
        handler = logging.FileHandler(_KERNEL_API_LOG_DEST, mode="a")

    handler.setFormatter(logging.Formatter("%(message)s"))
    _logger.addHandler(handler)
    _logger.propagate = False
```
**EN:** This block uses `_setup_logger` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_setup_logger` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 91-92: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 93-93: Module-level supporting statements
```python
_setup_logger()
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 94-95: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 96-104: is compiling function
```python
def _is_compiling() -> bool:
    try:
        if hasattr(torch, "compiler") and hasattr(torch.compiler, "is_compiling"):
            return bool(torch.compiler.is_compiling())
        if hasattr(torch, "_dynamo") and hasattr(torch._dynamo, "is_compiling"):
            return bool(torch._dynamo.is_compiling())
    except Exception:
        return False
    return False
```
**EN:** This block uses `_is_compiling` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_is_compiling` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 105-106: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 107-108: timestamp function
```python
def _timestamp() -> str:
    return datetime.now().strftime("[%Y-%m-%d %H:%M:%S]")
```
**EN:** This block uses `_timestamp` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_timestamp` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 109-110: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 111-115: is cuda graph capture active function
```python
def _is_cuda_graph_capture_active() -> bool:
    try:
        return torch.cuda.is_available() and torch.cuda.is_current_stream_capturing()
    except Exception:
        return False
```
**EN:** This block uses `_is_cuda_graph_capture_active` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_is_cuda_graph_capture_active` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 116-117: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 118-119: append line function
```python
def _append_line(lines: list[str], indent: int, text: str) -> None:
    lines.append(" " * indent + text)
```
**EN:** This block uses `_append_line` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_append_line` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 120-121: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 122-131: should dump function function
```python
def _should_dump_function(func_name: str) -> bool:
    if _DUMP_INCLUDE_PATTERNS and not any(
        fnmatch.fnmatch(func_name, pattern) for pattern in _DUMP_INCLUDE_PATTERNS
    ):
        return False
    if _DUMP_EXCLUDE_PATTERNS and any(
        fnmatch.fnmatch(func_name, pattern) for pattern in _DUMP_EXCLUDE_PATTERNS
    ):
        return False
    return True
```
**EN:** This block uses `_should_dump_function` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_should_dump_function` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 132-133: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 134-176: serialize tensor function
```python
def _serialize_tensor(tensor: torch.Tensor) -> list[str]:
    lines = ["Tensor("]
    _append_line(lines, 2, f"shape={tuple(tensor.shape)}")
    _append_line(lines, 2, f"dtype={tensor.dtype}")
    _append_line(lines, 2, f"device={tensor.device}")
    _append_line(lines, 2, f"requires_grad={tensor.requires_grad}")
    _append_line(lines, 2, f"is_contiguous={tensor.is_contiguous()}")

    if _KERNEL_API_LOG_LEVEL >= 5:
        if tensor.numel() == 0:
            _append_line(lines, 2, "statistics=[empty tensor]")
        elif tensor.device.type == "cuda" and _is_cuda_graph_capture_active():
            _append_line(
                lines, 2, "statistics=[skipped: CUDA graph capture in progress]"
            )
        else:
            try:
                detached = tensor.detach()
                if detached.is_complex():
                    stats_source = detached.abs().float()
                    nan_count = int(torch.isnan(detached).sum().item())
                    inf_count = int(torch.isinf(detached).sum().item())
                else:
                    stats_source = detached.float()
                    if detached.is_floating_point():
                        nan_count = int(torch.isnan(detached).sum().item())
                        inf_count = int(torch.isinf(detached).sum().item())
                    else:
                        nan_count = 0
                        inf_count = 0

                _append_line(lines, 2, f"min={stats_source.min().item():.6f}")
                _append_line(lines, 2, f"max={stats_source.max().item():.6f}")
                _append_line(lines, 2, f"mean={stats_source.mean().item():.6f}")
                _append_line(lines, 2, f"nan_count={nan_count}")
                _append_line(lines, 2, f"inf_count={inf_count}")
            except Exception as exc:
                _append_line(
                    lines, 2, f"statistics=[unavailable: {type(exc).__name__}]"
                )

    lines.append(")")
    return lines
```
**EN:** This block uses `_serialize_tensor` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_serialize_tensor` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 177-178: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 179-226: serialize value function
```python
def _serialize_value(value: Any, depth: int = 0) -> list[str]:
    if depth >= 2:
        return [f"{type(value).__name__}(...)"]

    if isinstance(value, torch.Tensor):
        return _serialize_tensor(value)

    if isinstance(value, (str, int, float, bool, type(None))):
        return [repr(value)]

    if isinstance(value, (list, tuple)):
        opener = "[" if isinstance(value, list) else "("
        closer = "]" if isinstance(value, list) else ")"
        lines = [opener]
        for idx, item in enumerate(value[:4]):
            item_lines = _serialize_value(item, depth + 1)
            lines.append(f"  [{idx}] {item_lines[0]}")
            for extra in item_lines[1:]:
                lines.append(f"      {extra}")
        if len(value) > 4:
            lines.append(f"  ... ({len(value) - 4} more items)")
        lines.append(closer)
        return lines

    if isinstance(value, dict):
        lines = ["{"]
        items = list(value.items())
        for key, item in items[:8]:
            item_lines = _serialize_value(item, depth + 1)
            lines.append(f"  {key!r}: {item_lines[0]}")
            for extra in item_lines[1:]:
                lines.append(f"      {extra}")
        if len(items) > 8:
            lines.append(f"  ... ({len(items) - 8} more items)")
        lines.append("}")
        return lines

    summary = [f"{type(value).__name__}("]
    for attr in ("shape", "dtype", "device"):
        if hasattr(value, attr):
            try:
                _append_line(summary, 2, f"{attr}={getattr(value, attr)}")
            except Exception:
                pass
    if len(summary) == 1:
        _append_line(summary, 2, f"repr={repr(value)[:200]}")
    summary.append(")")
    return summary
```
**EN:** This block uses `_serialize_value` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_serialize_value` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 227-228: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 229-241: serialize json value function
```python
def _serialize_json_value(value: Any) -> Any:
    if isinstance(value, torch.dtype):
        return {"type": "torch.dtype", "value": str(value)}
    if isinstance(value, (str, int, float, bool, type(None))):
        return value
    if isinstance(value, (list, tuple)):
        return [_serialize_json_value(item) for item in value[:16]]
    if isinstance(value, dict):
        return {
            str(key): _serialize_json_value(item)
            for key, item in list(value.items())[:32]
        }
    return {"type": type(value).__name__, "repr": repr(value)[:200]}
```
**EN:** This block uses `_serialize_json_value` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_serialize_json_value` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 242-243: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 244-276: collect dump entries function
```python
def _collect_dump_entries(
    prefix: str,
    value: Any,
    tensor_entries: dict[str, torch.Tensor],
    metadata_entries: dict[str, Any],
) -> None:
    if isinstance(value, torch.Tensor):
        tensor_entries[prefix] = value.detach().cpu()
        return

    if isinstance(value, (list, tuple)):
        for idx, item in enumerate(value):
            _collect_dump_entries(
                f"{prefix}_{idx}", item, tensor_entries, metadata_entries
            )
        metadata_entries[f"{prefix}__container"] = {
            "type": type(value).__name__,
            "length": len(value),
        }
        return

    if isinstance(value, dict):
        for key, item in value.items():
            _collect_dump_entries(
                f"{prefix}_{str(key)}", item, tensor_entries, metadata_entries
            )
        metadata_entries[f"{prefix}__container"] = {
            "type": "dict",
            "keys": [str(k) for k in value.keys()],
        }
        return

    metadata_entries[prefix] = _serialize_json_value(value)
```
**EN:** This block uses `_collect_dump_entries` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_collect_dump_entries` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 277-278: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 279-280: dump metadata path function
```python
def _dump_metadata_path(dump_dir: Path) -> Path:
    return dump_dir / "metadata.json"
```
**EN:** This block uses `_dump_metadata_path` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_dump_metadata_path` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 281-282: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 283-284: write dump metadata function
```python
def _write_dump_metadata(dump_dir: Path, metadata: dict[str, Any]) -> None:
    _dump_metadata_path(dump_dir).write_text(json.dumps(metadata, indent=2))
```
**EN:** This block uses `_write_dump_metadata` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_write_dump_metadata` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 285-286: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 287-288: read dump metadata function
```python
def _read_dump_metadata(dump_dir: Path) -> dict[str, Any]:
    return json.loads(_dump_metadata_path(dump_dir).read_text())
```
**EN:** This block uses `_read_dump_metadata` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_read_dump_metadata` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 289-290: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 291-330: dump function inputs function
```python
def _dump_function_inputs(
    func_name: str, args: tuple[Any, ...], kwargs: dict[str, Any]
) -> Path | None:
    if not _should_dump_function(func_name):
        return None

    _DUMP_DIR.mkdir(parents=True, exist_ok=True)
    call_index = _dump_call_counter.get(func_name, 0) + 1
    _dump_call_counter[func_name] = call_index
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S_%f")[:-3]
    safe_func_name = func_name.replace("/", "_").replace("<", "_").replace(">", "_")
    dump_dir = (
        _DUMP_DIR
        / f"{timestamp}_pid{os.getpid()}_{safe_func_name}_call{call_index:04d}"
    )
    dump_dir.mkdir(parents=True, exist_ok=True)

    tensor_entries: dict[str, torch.Tensor] = {}
    metadata_entries: dict[str, Any] = {}
    for idx, arg in enumerate(args):
        _collect_dump_entries(f"arg_{idx}", arg, tensor_entries, metadata_entries)
    for key, value in kwargs.items():
        _collect_dump_entries(f"kwarg_{key}", value, tensor_entries, metadata_entries)

    if tensor_entries:
        torch.save(tensor_entries, dump_dir / "inputs.pt")

    metadata = {
        "function_name": func_name,
        "timestamp": timestamp,
        "process_id": os.getpid(),
        "execution_status": "inputs_saved",
        "input_metadata": metadata_entries,
        "input_tensor_keys": list(tensor_entries.keys()),
        "output_metadata": {},
        "output_tensor_keys": [],
    }
    _write_dump_metadata(dump_dir, metadata)
    _logger.debug("Dumped inputs to: %s", dump_dir)
    return dump_dir
```
**EN:** This block uses `_dump_function_inputs` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_dump_function_inputs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 331-332: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 333-345: dump function outputs function
```python
def _dump_function_outputs(dump_dir: Path, result: Any) -> None:
    tensor_entries: dict[str, torch.Tensor] = {}
    metadata_entries: dict[str, Any] = {}
    _collect_dump_entries("result", result, tensor_entries, metadata_entries)
    if tensor_entries:
        torch.save(tensor_entries, dump_dir / "outputs.pt")

    metadata = _read_dump_metadata(dump_dir)
    metadata["execution_status"] = "completed"
    metadata["output_metadata"] = metadata_entries
    metadata["output_tensor_keys"] = list(tensor_entries.keys())
    _write_dump_metadata(dump_dir, metadata)
    _logger.debug("Dumped outputs to: %s", dump_dir)
```
**EN:** This block uses `_dump_function_outputs` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_dump_function_outputs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 346-347: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 348-355: mark dump exception function
```python
def _mark_dump_exception(dump_dir: Path, exc: Exception) -> None:
    metadata = _read_dump_metadata(dump_dir)
    metadata["execution_status"] = "exception"
    metadata["exception"] = {
        "type": type(exc).__name__,
        "message": str(exc),
    }
    _write_dump_metadata(dump_dir, metadata)
```
**EN:** This block uses `_mark_dump_exception` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_mark_dump_exception` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 356-357: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 358-364: log section function
```python
def _log_section(title: str, data: dict[str, Any]) -> None:
    _logger.debug(title)
    for key, value in data.items():
        lines = _serialize_value(value)
        _logger.debug("  %s=%s", key, lines[0])
        for line in lines[1:]:
            _logger.debug("    %s", line)
```
**EN:** This block uses `_log_section` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_log_section` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 365-366: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 367-384: infer func name function
```python
def _infer_func_name(func: Callable) -> str:
    qualname = getattr(func, "__qualname__", getattr(func, "__name__", "unknown"))
    qualname = qualname.replace(".<locals>.", ".").replace("<locals>.", "")

    module = getattr(func, "__module__", "")
    for prefix in ("sglang.", "sgl_kernel."):
        if module.startswith(prefix):
            module = module[len(prefix) :]
            break

    if module and module not in {"__main__", "builtins"}:
        return f"{module}.{qualname}"

    source_path = inspect.getsourcefile(func)
    if source_path is not None:
        return f"{Path(source_path).stem}.{qualname}"

    return qualname
```
**EN:** This block uses `_infer_func_name` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_infer_func_name` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 385-387: Module-level supporting statements
```python


@overload
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 388-392: debug kernel api function
```python
def debug_kernel_api(
    func: _F,
    *,
    op_name: str | None = None,
) -> _F: ...
```
**EN:** This block uses `debug_kernel_api` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `debug_kernel_api` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 393-395: Module-level supporting statements
```python


@overload
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 396-399: debug kernel api function
```python
def debug_kernel_api(
    *,
    op_name: str | None = None,
) -> Callable[[_F], _F]: ...
```
**EN:** This block uses `debug_kernel_api` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `debug_kernel_api` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 400-401: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 402-461: debug kernel api function (part 1/2)
```python
def debug_kernel_api(
    func: Callable | None = None,
    *,
    op_name: str | None = None,
) -> Callable:
    # NOTE: avoid any overhead in the hot path when logging is disabled
    if _KERNEL_API_LOG_LEVEL == 0:
        if func is None:
            return lambda f: f
        return func

    def decorator(f: Callable) -> Callable:
        if hasattr(f, "_debug_kernel_wrapped"):
            return f

        @functools.wraps(f)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            if _is_compiling():
                return f(*args, **kwargs)

            func_name = op_name or _infer_func_name(f)
            dump_dir: Path | None = None
            positional_args = args
            try:
                parameters = tuple(inspect.signature(f).parameters.values())
            except (TypeError, ValueError):
                parameters = ()
            if args and parameters and parameters[0].name in {"self", "cls"}:
                positional_args = args[1:]
            _logger.debug("=" * 80)
            _logger.debug("%s SGLang Kernel API Call: %s", _timestamp(), func_name)

            if _KERNEL_API_LOG_LEVEL >= 3:
                if positional_args:
                    _log_section(
                        "Positional input arguments:",
                        {f"arg[{idx}]": arg for idx, arg in enumerate(positional_args)},
                    )
                if kwargs:
                    _log_section("Keyword input arguments:", kwargs)

            if _KERNEL_API_LOG_LEVEL >= 10:
                if _is_cuda_graph_capture_active():
                    _logger.debug("Tensor dump skipped: CUDA graph capture in progress")
                else:
                    dump_dir = _dump_function_inputs(func_name, positional_args, kwargs)

            try:
                result = f(*args, **kwargs)
            except Exception as exc:
                if dump_dir is not None:
                    _mark_dump_exception(dump_dir, exc)
                _logger.debug(
                    "%s SGLang Kernel API Exception: %s (%s: %s)",
                    _timestamp(),
                    func_name,
                    type(exc).__name__,
                    exc,
                )
                raise
```
**EN:** This block uses `debug_kernel_api` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `debug_kernel_api` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 462-472: debug kernel api function (part 2/2)
```python

            if dump_dir is not None:
                _dump_function_outputs(dump_dir, result)
            if _KERNEL_API_LOG_LEVEL >= 3:
                _log_section("Output:", {"return": result})
            return result

        setattr(wrapper, "_debug_kernel_wrapped", True)
        return wrapper

    return decorator if func is None else decorator(func)
```
**EN:** This block uses `debug_kernel_api` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `debug_kernel_api` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 473-474: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 475-491: debug torch op function
```python
def debug_torch_op(
    op_func: Callable,
    op_name: str,
    *,
    namespace: str = "sglang",
) -> Callable:
    """NOTE: For internal use. Prefer `debug_kernel_api` for general use cases."""
    # NOTE: avoid any overhead in the hot path when logging is disabled
    impl = getattr(getattr(torch.ops, namespace), op_name)
    if _KERNEL_API_LOG_LEVEL == 0:
        return impl
    # NOTE: propagate the marker to avoid double-wrapping
    if hasattr(op_func, "_debug_kernel_wrapped"):
        setattr(impl, "_debug_kernel_wrapped", True)
        return impl
    # NOTE: redirect the function name
    return debug_kernel_api(impl, op_name=_infer_func_name(op_func))
```
**EN:** This block uses `debug_torch_op` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `debug_torch_op` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 492-493: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 494-517: wrap method with debug kernel once function
```python
def wrap_method_with_debug_kernel_once(
    obj: Any,
    method_name: str,
    *,
    op_name: str,
    marker_attr: str | None = None,
) -> Any:
    # NOTE: avoid any overhead in the hot path when logging is disabled
    if _KERNEL_API_LOG_LEVEL == 0:
        return obj

    if marker_attr is None:
        marker_attr = f"_debug_kernel_{method_name}_wrapped"

    if getattr(obj, marker_attr, False):
        return obj

    setattr(
        obj,
        method_name,
        debug_kernel_api(getattr(obj, method_name), op_name=op_name),
    )
    setattr(obj, marker_attr, True)
    return obj
```
**EN:** This block uses `wrap_method_with_debug_kernel_once` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `wrap_method_with_debug_kernel_once` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- None / 无
### External / 外部
- `__future__`
- `fnmatch`
- `torch`
- `datetime` (stdlib)
- `functools` (stdlib)
- `inspect` (stdlib)
- `json` (stdlib)
- `logging` (stdlib)
- `os` (stdlib)
- `pathlib` (stdlib)
- `sys` (stdlib)
- `typing` (stdlib)
