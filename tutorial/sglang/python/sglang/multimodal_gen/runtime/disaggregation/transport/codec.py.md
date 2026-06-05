# codec.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/transport/codec.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `TensorWrapper`, `TensorDescriptor`, and `dtype_to_str`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Zero-copy tensor codec for ZMQ multipart messages. Frame 0: JSON metadata (tensor descriptors + scalar fields) Frame 1-N: Raw tensor data buffers (one per tensor) / 该文件属于解耦运行时层。它围绕 `TensorWrapper`、`TensorDescriptor` 和 `dtype_to_str` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-14: module setup and imports / 模块初始化与导入
```python
"""Zero-copy tensor codec for ZMQ multipart messages.

Frame 0: JSON metadata (tensor descriptors + scalar fields)
Frame 1-N: Raw tensor data buffers (one per tensor)
"""

import ctypes
import json
import logging
from dataclasses import dataclass

import torch
import zmq
```
**EN:** This block establishes the module context and imports `ctypes`, `json`, `logging`, `dataclasses`, `torch`, and `zmq`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `ctypes`、`json`、`logging`、`dataclasses`、`torch` 和 `zmq`。这些依赖为后续实现提供所需符号。

### Lines 16-30: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)

_DTYPE_TO_STR = {
    torch.float16: "float16",
    torch.float32: "float32",
    torch.float64: "float64",
    torch.bfloat16: "bfloat16",
    torch.int8: "int8",
    torch.int16: "int16",
    torch.int32: "int32",
    torch.int64: "int64",
    torch.uint8: "uint8",
    torch.bool: "bool",
}
_STR_TO_DTYPE = {v: k for k, v in _DTYPE_TO_STR.items()}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `_DTYPE_TO_STR`, and `_STR_TO_DTYPE`. The code collaborates with `logging.getLogger`, and `_DTYPE_TO_STR.items`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`_DTYPE_TO_STR` 和 `_STR_TO_DTYPE` 等名称。 代码会与 `logging.getLogger` 和 `_DTYPE_TO_STR.items` 协同工作。

### Lines 33-37: `dtype_to_str` implementation / `dtype_to_str` 实现
```python
def dtype_to_str(dtype: torch.dtype) -> str:
    s = _DTYPE_TO_STR.get(dtype)
    if s is None:
        raise ValueError(f"Unsupported dtype: {dtype}")
    return s
```
**EN:** This block defines function `dtype_to_str`. It handles dtype to str logic. Key calls include `_DTYPE_TO_STR.get`, and `ValueError`. The implementation branches on conditions. Parameters such as `dtype` drive the behavior in this section.
**CN:** 该代码块定义了函数 `dtype_to_str`。 它用于处理 dtype to str 相关逻辑。 关键调用包括 `_DTYPE_TO_STR.get` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `dtype` 等参数驱动。

### Lines 40-44: `str_to_dtype` implementation / `str_to_dtype` 实现
```python
def str_to_dtype(s: str) -> torch.dtype:
    d = _STR_TO_DTYPE.get(s)
    if d is None:
        raise ValueError(f"Unknown dtype string: {s}")
    return d
```
**EN:** This block defines function `str_to_dtype`. It handles str to dtype logic. Key calls include `_STR_TO_DTYPE.get`, and `ValueError`. The implementation branches on conditions. Parameters such as `s` drive the behavior in this section.
**CN:** 该代码块定义了函数 `str_to_dtype`。 它用于处理 str to dtype 相关逻辑。 关键调用包括 `_STR_TO_DTYPE.get` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `s` 等参数驱动。

### Lines 47-49: `TensorWrapper` class overview / `TensorWrapper` 类概览
```python
class TensorWrapper:
    """Expose a CPU-contiguous tensor's data buffer for zero-copy ZMQ send."""
```
**EN:** This block defines class `TensorWrapper`. Expose a CPU-contiguous tensor's data buffer for zero-copy ZMQ send.
**CN:** 该代码块定义了类 `TensorWrapper`。 它用于封装 tensor wrapper 相关行为。

### Lines 50-59: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, tensor: torch.Tensor):
        if tensor.is_cuda:
            tensor = tensor.cpu()
        if not tensor.is_contiguous():
            tensor = tensor.contiguous()
        self.tensor = tensor
        data_ptr = tensor.data_ptr()
        total_bytes = tensor.numel() * tensor.element_size()
        self._c_buf = (ctypes.c_char * total_bytes).from_address(data_ptr)
        self._view = memoryview(self._c_buf)
```
**EN:** This block defines method `__init__` on `TensorWrapper`. It initializes the instance state. Key calls include `tensor.data_ptr`, `from_address`, `memoryview`, `tensor.cpu`, and `tensor.is_contiguous`. The implementation branches on conditions. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `TensorWrapper` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `tensor.data_ptr`、`from_address`、`memoryview`、`tensor.cpu` 和 `tensor.is_contiguous`。 实现中包含条件分支。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 63-63: `TensorDescriptor` class overview / `TensorDescriptor` 类概览
```python
class TensorDescriptor:
```
**EN:** This block defines class `TensorDescriptor`. It encapsulates tensor descriptor behavior.
**CN:** 该代码块定义了类 `TensorDescriptor`。 它用于封装 tensor descriptor 相关行为。

### Lines 64-67: supporting statements / 辅助语句
```python
    field_name: str
    shape: list[int]
    dtype: str
    list_index: int = -1  # -1 means not part of a list
```
**EN:** This block gathers supporting statements inside `TensorDescriptor`. It updates names such as `field_name`, `shape`, `dtype`, and `list_index`.
**CN:** 该代码块汇集了位于 `TensorDescriptor` 内部的辅助语句。 它会更新 `field_name`、`shape`、`dtype` 和 `list_index` 等名称。

### Lines 69-75: `to_dict` implementation / `to_dict` 实现
```python
    def to_dict(self) -> dict:
        return {
            "field_name": self.field_name,
            "shape": self.shape,
            "dtype": self.dtype,
            "list_index": self.list_index,
        }
```
**EN:** This block defines method `to_dict` on `TensorDescriptor`. It converts to dict.
**CN:** 该代码块定义了 `TensorDescriptor` 的方法 `to_dict`。 它用于转换为dict。

### Lines 77-84: `from_dict` implementation / `from_dict` 实现
```python
    @classmethod
    def from_dict(cls, d: dict) -> "TensorDescriptor":
        return cls(
            field_name=d["field_name"],
            shape=d["shape"],
            dtype=d["dtype"],
            list_index=d.get("list_index", -1),
        )
```
**EN:** This block defines method `from_dict` on `TensorDescriptor`. It constructs from dict. Key calls include `cls`, and `d.get`. Parameters such as `d` drive the behavior in this section.
**CN:** 该代码块定义了 `TensorDescriptor` 的方法 `from_dict`。 它用于从…构造dict。 关键调用包括 `cls` 和 `d.get`。 本段逻辑主要由 `d` 等参数驱动。

### Lines 87-135: `pack_tensors` implementation / `pack_tensors` 实现
```python
def pack_tensors(
    tensor_fields: dict[str, torch.Tensor | list[torch.Tensor] | None],
    scalar_fields: dict | None = None,
) -> tuple[bytes, list[TensorWrapper]]:
    """Pack tensor fields into metadata + buffer list for send_multipart."""
    descriptors = []
    buffers = []

    for field_name, value in tensor_fields.items():
        if value is None:
            continue

        if isinstance(value, torch.Tensor):
            wrapper = TensorWrapper(value)
            descriptors.append(
                TensorDescriptor(
                    field_name=field_name,
                    shape=list(value.shape),
                    dtype=dtype_to_str(value.dtype),
                )
            )
            buffers.append(wrapper)

        elif isinstance(value, list):
            for i, t in enumerate(value):
                if t is None:
                    continue
                if not isinstance(t, torch.Tensor):
                    raise TypeError(
                        f"Expected Tensor in list for field '{field_name}', "
                        f"got {type(t)}"
                    )
                wrapper = TensorWrapper(t)
                descriptors.append(
                    TensorDescriptor(
                        field_name=field_name,
                        shape=list(t.shape),
                        dtype=dtype_to_str(t.dtype),
                        list_index=i,
                    )
                )
                buffers.append(wrapper)

    metadata = {
        "tensor_descriptors": [d.to_dict() for d in descriptors],
        "scalar_fields": scalar_fields or {},
    }
    metadata_bytes = json.dumps(metadata, separators=(",", ":")).encode("utf-8")
    return metadata_bytes, buffers
```
**EN:** This block defines function `pack_tensors`. Pack tensor fields into metadata + buffer list for send_multipart. Key calls include `tensor_fields.items`, `json.dumps.encode`, `isinstance`, `TensorWrapper`, and `descriptors.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `tensor_fields`, and `scalar_fields` drive the behavior in this section.
**CN:** 该代码块定义了函数 `pack_tensors`。 它用于打包tensors。 关键调用包括 `tensor_fields.items`、`json.dumps.encode`、`isinstance`、`TensorWrapper` 和 `descriptors.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `tensor_fields` 和 `scalar_fields` 等参数驱动。

### Lines 138-148: `send_tensors` implementation / `send_tensors` 实现
```python
def send_tensors(
    socket: zmq.Socket,
    tensor_fields: dict[str, torch.Tensor | list[torch.Tensor] | None],
    scalar_fields: dict | None = None,
    flags: int = 0,
) -> None:
    """Send tensors over ZMQ using multipart with zero-copy."""
    metadata_bytes, buffers = pack_tensors(tensor_fields, scalar_fields)
    parts: list = [metadata_bytes]
    parts.extend(w._view if isinstance(w, TensorWrapper) else w for w in buffers)
    socket.send_multipart(parts, flags=flags, copy=True)
```
**EN:** This block defines function `send_tensors`. Send tensors over ZMQ using multipart with zero-copy. Key calls include `pack_tensors`, `parts.extend`, `socket.send_multipart`, and `isinstance`. Parameters such as `socket`, `tensor_fields`, `scalar_fields`, and `flags` drive the behavior in this section.
**CN:** 该代码块定义了函数 `send_tensors`。 它用于处理 send tensors 相关逻辑。 关键调用包括 `pack_tensors`、`parts.extend`、`socket.send_multipart` 和 `isinstance`。 本段逻辑主要由 `socket`、`tensor_fields`、`scalar_fields` 和 `flags` 等参数驱动。

### Lines 151-198: `unpack_tensors` implementation / `unpack_tensors` 实现
```python
def unpack_tensors(
    parts: list,
    device: str | torch.device = "cpu",
) -> tuple[dict[str, torch.Tensor | list[torch.Tensor]], dict]:
    """Unpack multipart message frames into tensor fields and scalar fields."""
    metadata_frame = parts[0]
    metadata_bytes = (
        bytes(metadata_frame.buffer)
        if hasattr(metadata_frame, "buffer")
        else bytes(metadata_frame)
    )
    metadata = json.loads(metadata_bytes)

    descriptors = [
        TensorDescriptor.from_dict(d) for d in metadata["tensor_descriptors"]
    ]
    scalar_fields = metadata.get("scalar_fields", {})

    if len(parts) - 1 != len(descriptors):
        raise ValueError(
            f"Expected {len(descriptors)} tensor frames, got {len(parts) - 1}"
        )

    tensor_fields: dict[str, torch.Tensor | list[torch.Tensor]] = {}
    list_sizes: dict[str, int] = {}
    for desc in descriptors:
        if desc.list_index >= 0:
            current_max = list_sizes.get(desc.field_name, 0)
            list_sizes[desc.field_name] = max(current_max, desc.list_index + 1)

    for field_name, size in list_sizes.items():
        tensor_fields[field_name] = [None] * size

    for i, desc in enumerate(descriptors):
        frame = parts[i + 1]
        buf = frame.buffer if hasattr(frame, "buffer") else bytes(frame)
        dtype = str_to_dtype(desc.dtype)
        # clone() to own the memory (decouple from ZMQ buffer lifetime)
        tensor = torch.frombuffer(buf, dtype=dtype).reshape(desc.shape).clone()
        if device != "cpu" and device != torch.device("cpu"):
            tensor = tensor.to(device)

        if desc.list_index >= 0:
            tensor_fields[desc.field_name][desc.list_index] = tensor
        else:
            tensor_fields[desc.field_name] = tensor

    return tensor_fields, scalar_fields
```
**EN:** This block defines function `unpack_tensors`. Unpack multipart message frames into tensor fields and scalar fields. Key calls include `json.loads`, `metadata.get`, `list_sizes.items`, `enumerate`, and `hasattr`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `parts`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了函数 `unpack_tensors`。 它用于解包tensors。 关键调用包括 `json.loads`、`metadata.get`、`list_sizes.items`、`enumerate` 和 `hasattr`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `parts` 和 `device` 等参数驱动。

## Key Concepts / 关键概念
- `dtype_to_str`: Top-level function that handles dtype to str logic. / 顶层函数，用于处理 dtype to str 相关逻辑。
- `str_to_dtype`: Top-level function that handles str to dtype logic. / 顶层函数，用于处理 str to dtype 相关逻辑。
- `TensorWrapper`: Expose a CPU-contiguous tensor's data buffer for zero-copy ZMQ send. / 核心类，用于封装 tensor wrapper 相关行为。
- `TensorDescriptor`: Primary class that encapsulates tensor descriptor behavior. / 核心类，用于封装 tensor descriptor 相关行为。
- `pack_tensors`: Pack tensor fields into metadata + buffer list for send_multipart. / 顶层函数，用于打包tensors。
- `send_tensors`: Send tensors over ZMQ using multipart with zero-copy. / 顶层函数，用于处理 send tensors 相关逻辑。
- `unpack_tensors`: Unpack multipart message frames into tensor fields and scalar fields. / 顶层函数，用于解包tensors。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `json`, `logging`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`, `zmq`

- **Total lines / 总行数**: 198
