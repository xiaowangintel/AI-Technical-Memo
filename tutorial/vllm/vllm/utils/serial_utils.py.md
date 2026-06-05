# serial_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/serial_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-14)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import io
import sys
from collections.abc import Mapping
from dataclasses import dataclass
from typing import Literal, get_args

import numpy as np
import numpy.typing as npt
import pybase64
import torch

sys_byteorder = sys.byteorder
```
**EN:** Sets up the module with standard-library support such as `io`, `sys`, `collections.abc`, external packages such as `numpy`, `numpy.typing`, `pybase64`. It prepares the symbols later used by `DTypeInfo`, `tensor2base64`, `tensor2binary`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `DTypeInfo`, `tensor2base64`, `tensor2binary` 提供上下文。

### DTypeInfo (lines 18-26)
```python
class DTypeInfo:
    torch_dtype: torch.dtype

    torch_view_dtype: torch.dtype
    numpy_view_dtype: npt.DTypeLike

    @property
    def nbytes(self) -> int:
        return self.torch_dtype.itemsize
```
**EN:** Defines the `DTypeInfo` class used by this module. Key methods include `nbytes`.
**CN:** `DTypeInfo` 是该文件中的核心类，用于封装与 `DTypeInfo` 相关的状态和行为。 关键方法包括 `nbytes`。

### tensor2base64 (lines 58-64)
```python
def tensor2base64(x: torch.Tensor) -> str:
    with io.BytesIO() as buf:
        torch.save(x, buf)
        buf.seek(0)
        binary_data = buf.read()

    return pybase64.b64encode(binary_data).decode("utf-8")
```
**EN:** `tensor2base64` implements helper logic used by this module. It mainly works with `x`. Inside the body, it relies on `pybase64.b64encode.decode`, `io.BytesIO`, `torch.save` to complete the main steps.
**CN:** `tensor2base64` 负责实现本模块使用的辅助逻辑。 它主要处理 `x` 等参数。 实现过程中会调用 `pybase64.b64encode.decode`, `io.BytesIO`, `torch.save` 等函数完成关键步骤。

### tensor2binary (lines 67-89)
```python
def tensor2binary(
    tensor: torch.Tensor,
    embed_dtype: "EmbedDType | MmMetadataDType",
    endianness: Endianness,
) -> bytes:
    assert isinstance(tensor, torch.Tensor)
    assert embed_dtype in _ALL_SERIAL_DTYPES
    assert endianness in ENDIANNESS

    dtype_info = _ALL_SERIAL_DTYPES[embed_dtype]

    np_array = (
        tensor.to(dtype_info.torch_dtype)
        .flatten()
        .contiguous()
        .view(dtype_info.torch_view_dtype)
        .numpy()
    )

    if endianness != "native" and endianness != sys_byteorder:
        np_array = np_array.byteswap()

    return np_array.tobytes()
```
**EN:** `tensor2binary` implements helper logic used by this module. It mainly works with `tensor`, `embed_dtype`, `endianness`. Inside the body, it relies on `tensor.to.flatten.contiguous.view.numpy`, `np_array.tobytes`, `np_array.byteswap` to complete the main steps.
**CN:** `tensor2binary` 负责实现本模块使用的辅助逻辑。 它主要处理 `tensor`, `embed_dtype`, `endianness` 等参数。 实现过程中会调用 `tensor.to.flatten.contiguous.view.numpy`, `np_array.tobytes`, `np_array.byteswap` 等函数完成关键步骤。

### binary2tensor (lines 92-108)
```python
def binary2tensor(
    binary: bytes,
    shape: tuple[int, ...],
    embed_dtype: "EmbedDType | MmMetadataDType",
    endianness: Endianness,
) -> torch.Tensor:
    assert embed_dtype in _ALL_SERIAL_DTYPES
    assert endianness in ENDIANNESS

    dtype_info = _ALL_SERIAL_DTYPES[embed_dtype]

    np_array = np.frombuffer(binary, dtype=dtype_info.numpy_view_dtype).reshape(shape)

    if endianness != "native" and endianness != sys_byteorder:
        np_array = np_array.byteswap()

    return torch.from_numpy(np_array).view(dtype_info.torch_dtype)
```
**EN:** `binary2tensor` implements helper logic used by this module. It mainly works with `binary`, `shape`, `embed_dtype`, `endianness`. Inside the body, it relies on `np.frombuffer.reshape`, `torch.from_numpy.view`, `np_array.byteswap` to complete the main steps.
**CN:** `binary2tensor` 负责实现本模块使用的辅助逻辑。 它主要处理 `binary`, `shape`, `embed_dtype`, `endianness` 等参数。 实现过程中会调用 `np.frombuffer.reshape`, `torch.from_numpy.view`, `np_array.byteswap` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`DTypeInfo`**: Core class that organizes module behavior. / **`DTypeInfo`**：组织模块行为的核心类。
- **`tensor2base64`**: Key helper or entry point in this file. / **`tensor2base64`**：本文件中的关键辅助函数或入口。
- **`tensor2binary`**: Key helper or entry point in this file. / **`tensor2binary`**：本文件中的关键辅助函数或入口。
- **`binary2tensor`**: Key helper or entry point in this file. / **`binary2tensor`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: io, sys, collections.abc, dataclasses, typing
- **Third-party / 第三方**: numpy, numpy.typing, pybase64, torch
- **Internal vLLM / vLLM 内部依赖**: None / 无
