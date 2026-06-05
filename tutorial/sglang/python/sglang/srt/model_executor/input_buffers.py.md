# input_buffers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/input_buffers.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `input_buffers`. It exposes primary entry points such as `ForwardInputBuffers`. / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `input_buffers` 的逻辑。 它对外提供的主要入口包括 `ForwardInputBuffers`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module imports, constants, and setup
```python
from __future__ import annotations

import dataclasses
from dataclasses import dataclass, fields
from typing import Dict

import torch

from sglang.srt.utils import is_npu

_forward_input_buffer_pool: Dict[str, torch.Tensor] = {}


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 14-16: Class ForwardInputBuffers
```python
@dataclass
class ForwardInputBuffers:

```
**EN:** This range introduces `ForwardInputBuffers` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ForwardInputBuffers`，并定义其后续方法依赖的结构或元数据。

### Lines 17-34: Method ForwardInputBuffers._share_one_buffer
```python
    def _share_one_buffer(self, name: str, new_buffer: torch.Tensor) -> torch.Tensor:

        buffer_size = new_buffer.size()
        buffer_stride = new_buffer.stride()

        old_buffer = _forward_input_buffer_pool.get(name, None)
        if old_buffer is not None:
            assert (
                new_buffer.dtype == old_buffer.dtype
            ), f"Buffer {name} has different dtype than before."
            assert (
                new_buffer.device == old_buffer.device
            ), f"Buffer {name} has different device than before."
            if old_buffer.numel() > new_buffer.numel():
                new_buffer = old_buffer

        _forward_input_buffer_pool[name] = new_buffer
        return new_buffer.as_strided(buffer_size, buffer_stride)
```
**EN:** This callable implements `ForwardInputBuffers._share_one_buffer`. It takes `name`, `new_buffer` and mainly implements share one buffer. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ForwardInputBuffers._share_one_buffer`。它接收 `name`, `new_buffer`，主要用于实现 share one buffer 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 36-65: Method ForwardInputBuffers.share_buffers
```python
    def share_buffers(self):
        # disable share input buffer on npu due to accuracy issue
        if is_npu():
            return

        for f in fields(self):
            name = f.name
            buffer = getattr(self, name)

            if buffer is None:
                continue

            if dataclasses.is_dataclass(buffer):
                buffer = vars(buffer)

            if isinstance(buffer, dict):
                for sub_name, sub_buffer in buffer.items():
                    assert isinstance(
                        sub_buffer, torch.Tensor
                    ), f"Field {name}.{sub_name} is expected to be a torch.Tensor, but got {type(sub_buffer)}."
                    new_buffer = self._share_one_buffer(
                        f"{name}.{sub_name}", sub_buffer
                    )
                    buffer[sub_name] = new_buffer
            else:
                assert isinstance(
                    buffer, torch.Tensor
                ), f"Field {name} is expected to be a torch.Tensor, a dict of torch.Tensor, or a dataclass of torch.Tensor, but got {type(buffer)}."
                new_buffer = self._share_one_buffer(name, buffer)
                setattr(self, name, new_buffer)
```
**EN:** This callable implements `ForwardInputBuffers.share_buffers` and mainly implements share buffers. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ForwardInputBuffers.share_buffers`，主要用于实现 share buffers 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `ForwardInputBuffers`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.utils`
