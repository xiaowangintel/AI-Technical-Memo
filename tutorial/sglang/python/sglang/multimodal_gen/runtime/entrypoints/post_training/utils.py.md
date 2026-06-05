# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/post_training/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `tensor_to_base64`, `base64_to_tensor`, and `_maybe_serialize`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Tensor serialization for post-training / rollout HTTP responses. / 该文件属于服务入口层。它围绕 `tensor_to_base64`、`base64_to_tensor` 和 `_maybe_serialize` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: module setup and imports / 模块初始化与导入
```python
"""Tensor serialization for post-training / rollout HTTP responses."""

from __future__ import annotations

import base64
from typing import Any

import torch
from safetensors.torch import load, save
```
**EN:** This block establishes the module context and imports `__future__`, `base64`, `typing`, `torch`, and `safetensors.torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`base64`、`typing`、`torch` 和 `safetensors.torch`。这些依赖为后续实现提供所需符号。

### Lines 12-15: `tensor_to_base64` implementation / `tensor_to_base64` 实现
```python
def tensor_to_base64(t: torch.Tensor) -> str:
    t = t.detach().contiguous().cpu()
    raw = save({"t": t})
    return base64.b64encode(raw).decode("ascii")
```
**EN:** This block defines function `tensor_to_base64`. It handles tensor to base64 logic. Key calls include `t.detach.contiguous.cpu`, `save`, `base64.b64encode.decode`, `t.detach.contiguous`, and `base64.b64encode`. Parameters such as `t` drive the behavior in this section.
**CN:** 该代码块定义了函数 `tensor_to_base64`。 它用于处理 tensor to base64 相关逻辑。 关键调用包括 `t.detach.contiguous.cpu`、`save`、`base64.b64encode.decode`、`t.detach.contiguous` 和 `base64.b64encode`。 本段逻辑主要由 `t` 等参数驱动。

### Lines 18-20: `base64_to_tensor` implementation / `base64_to_tensor` 实现
```python
def base64_to_tensor(s: str) -> torch.Tensor:
    raw = base64.b64decode(s)
    return load(raw)["t"]
```
**EN:** This block defines function `base64_to_tensor`. It handles base64 to tensor logic. Key calls include `base64.b64decode`, and `load`. Parameters such as `s` drive the behavior in this section.
**CN:** 该代码块定义了函数 `base64_to_tensor`。 它用于处理 base64 to tensor 相关逻辑。 关键调用包括 `base64.b64decode` 和 `load`。 本段逻辑主要由 `s` 等参数驱动。

### Lines 23-35: `_maybe_serialize` implementation / `_maybe_serialize` 实现
```python
def _maybe_serialize(obj: Any) -> Any:
    if isinstance(obj, torch.Tensor):
        return {
            "__tensor__": True,
            "data": tensor_to_base64(obj),
            "shape": list(obj.shape),
            "dtype": str(obj.dtype),
        }
    if isinstance(obj, dict):
        return {k: _maybe_serialize(v) for k, v in obj.items()}
    if isinstance(obj, (list, tuple)):
        return [_maybe_serialize(v) for v in obj]
    return obj
```
**EN:** This block defines function `_maybe_serialize`. It handles maybe serialize logic. Key calls include `isinstance`, `tensor_to_base64`, `list`, `str`, and `_maybe_serialize`. The implementation branches on conditions. Parameters such as `obj` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_maybe_serialize`。 它用于处理 maybe serialize 相关逻辑。 关键调用包括 `isinstance`、`tensor_to_base64`、`list`、`str` 和 `_maybe_serialize`。 实现中包含条件分支。 本段逻辑主要由 `obj` 等参数驱动。

### Lines 38-45: `_maybe_deserialize` implementation / `_maybe_deserialize` 实现
```python
def _maybe_deserialize(obj: Any) -> Any:
    if isinstance(obj, dict):
        if obj.get("__tensor__"):
            return base64_to_tensor(obj["data"])
        return {k: _maybe_deserialize(v) for k, v in obj.items()}
    if isinstance(obj, (list, tuple)):
        return [_maybe_deserialize(v) for v in obj]
    return obj
```
**EN:** This block defines function `_maybe_deserialize`. It handles maybe deserialize logic. Key calls include `isinstance`, `obj.get`, `base64_to_tensor`, `_maybe_deserialize`, and `obj.items`. The implementation branches on conditions. Parameters such as `obj` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_maybe_deserialize`。 它用于处理 maybe deserialize 相关逻辑。 关键调用包括 `isinstance`、`obj.get`、`base64_to_tensor`、`_maybe_deserialize` 和 `obj.items`。 实现中包含条件分支。 本段逻辑主要由 `obj` 等参数驱动。

## Key Concepts / 关键概念
- `tensor_to_base64`: Top-level function that handles tensor to base64 logic. / 顶层函数，用于处理 tensor to base64 相关逻辑。
- `base64_to_tensor`: Top-level function that handles base64 to tensor logic. / 顶层函数，用于处理 base64 to tensor 相关逻辑。
- `_maybe_serialize`: Top-level function that handles maybe serialize logic. / 顶层函数，用于处理 maybe serialize 相关逻辑。
- `_maybe_deserialize`: Top-level function that handles maybe deserialize logic. / 顶层函数，用于处理 maybe deserialize 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `base64`, `typing`
- **Third-party / 第三方依赖**: `torch`, `safetensors.torch`

- **Total lines / 总行数**: 45
