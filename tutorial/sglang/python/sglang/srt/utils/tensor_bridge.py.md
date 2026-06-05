# tensor_bridge.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/tensor_bridge.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `tensor_bridge` and the surrounding SGLang serving stack. / 提供围绕 `tensor_bridge` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 3-29: Module setup and shared state / 模块设置与共享状态
```python
"""Tensor bridge between MLX and PyTorch.

Provides zero-copy conversion when possible using Apple Silicon's unified memory.
"""

from __future__ import annotations

import logging
from functools import lru_cache
from typing import TYPE_CHECKING, Literal

import torch

from sglang.srt.environ import envs

if TYPE_CHECKING:
    import mlx.core as mx

logger = logging.getLogger(__name__)

_MLX_AVAILABLE: bool = False
try:
    import mlx.core as mx  # noqa: F811

    _MLX_AVAILABLE = True
except ImportError:
    pass
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `logging`, `functools`, `typing`, `torch`, `sglang.srt.environ`. It also defines symbols such as `logger`, `_MLX_AVAILABLE`, `_MLX_AVAILABLE` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `logging`, `functools`, `typing`, `torch`, `sglang.srt.environ`。 同时定义了 `logger`, `_MLX_AVAILABLE`, `_MLX_AVAILABLE` 等符号，供后续逻辑使用。

### Lines 32-34: Function `is_mlx_available` / 函数 `is_mlx_available`
```python
def is_mlx_available() -> bool:
    """Return True when the ``mlx`` package can be imported."""
    return _MLX_AVAILABLE
```
**EN:** This function implements `is_mlx_available`.
**CN:** 该函数实现了 `is_mlx_available`。

### Lines 37-40: Function `use_mlx` / 函数 `use_mlx`
```python
@lru_cache(maxsize=1)
def use_mlx() -> bool:
    """Return True when the user opted-in via ``SGLANG_USE_MLX=1`` **and** MLX is importable."""
    return bool(envs.SGLANG_USE_MLX.get()) and _MLX_AVAILABLE
```
**EN:** This function implements `use_mlx`. It primarily calls `lru_cache`, `bool`, `envs.SGLANG_USE_MLX.get` to complete its work.
**CN:** 该函数实现了 `use_mlx`。 它主要通过调用 `lru_cache`, `bool`, `envs.SGLANG_USE_MLX.get` 来完成任务。

### Lines 47-72: Constants and shared state / 常量与共享状态
```python
_MPS_SAFE_SIZE_BYTES = 1 << 30  # 1GB

# MLX to PyTorch dtype mapping
# TODO(perf): float64 is CPU-only in MLX (see ml-explore/mlx#1843).
# When the target device is GPU/MPS we should auto-downcast float64 → float32
# to avoid a runtime error; when the target is CPU we can keep float64.
# For now float64 is omitted from the mapping so it hits the ValueError
# fallback in mlx_to_torch().
MLX_TO_TORCH_DTYPE = (
    {
        mx.float32: torch.float32,
        mx.float16: torch.float16,
        mx.bfloat16: torch.bfloat16,
        mx.int32: torch.int32,
        mx.int64: torch.int64,
        mx.int16: torch.int16,
        mx.int8: torch.int8,
        mx.uint8: torch.uint8,
        mx.bool_: torch.bool,
    }
    if _MLX_AVAILABLE
    else {}
)

# PyTorch to MLX dtype mapping
TORCH_TO_MLX_DTYPE = {v: k for k, v in MLX_TO_TORCH_DTYPE.items()}
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_MPS_SAFE_SIZE_BYTES`, `MLX_TO_TORCH_DTYPE`, `TORCH_TO_MLX_DTYPE`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_MPS_SAFE_SIZE_BYTES`, `MLX_TO_TORCH_DTYPE`, `TORCH_TO_MLX_DTYPE`。

### Lines 75-83: Function `get_torch_device` / 函数 `get_torch_device`
```python
def get_torch_device() -> torch.device:
    """Get the PyTorch device for Metal/MPS.

    Returns:
        torch.device for MPS if available, else CPU
    """
    if torch.backends.mps.is_available():
        return torch.device("mps")
    return torch.device("cpu")
```
**EN:** This function implements `get_torch_device`. It primarily calls `torch.backends.mps.is_available`, `torch.device` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_torch_device`。 它主要通过调用 `torch.backends.mps.is_available`, `torch.device` 来完成任务。 实现中使用了条件分支。

### Lines 86-95: Function `_get_tensor_size_bytes` / 函数 `_get_tensor_size_bytes`
```python
def _get_tensor_size_bytes(array: mx.array) -> int:
    """Calculate the size of an MLX array in bytes.

    Args:
        array: MLX array

    Returns:
        Size in bytes
    """
    return array.size * array.dtype.size
```
**EN:** This function implements `_get_tensor_size_bytes`.
**CN:** 该函数实现了 `_get_tensor_size_bytes`。

### Lines 98-110: Function `_is_safe_for_mps` / 函数 `_is_safe_for_mps`
```python
def _is_safe_for_mps(array: mx.array) -> bool:
    """Check if an array is safe to transfer to MPS without hitting size limits.

    MPS has a 4GB limit for MPSTemporaryNDArray, but Metal may allocate
    multiple temporary buffers internally. We use a conservative threshold.

    Args:
        array: MLX array to check

    Returns:
        True if safe to transfer to MPS, False if should stay on CPU
    """
    return _get_tensor_size_bytes(array) < _MPS_SAFE_SIZE_BYTES
```
**EN:** This function implements `_is_safe_for_mps`. It primarily calls `_get_tensor_size_bytes` to complete its work.
**CN:** 该函数实现了 `_is_safe_for_mps`。 它主要通过调用 `_get_tensor_size_bytes` 来完成任务。

### Lines 113-134: Function `torch_to_mlx` / 函数 `torch_to_mlx`
```python
def torch_to_mlx(tensor: torch.Tensor) -> mx.array:
    """Convert PyTorch tensor to MLX array.

    Uses numpy as an intermediate to enable zero-copy on unified memory.

    Args:
        tensor: PyTorch tensor (can be on any device)

    Returns:
        MLX array with the same data
    """
    # Move to CPU if on MPS for numpy conversion
    if tensor.device.type != "cpu":
        tensor = tensor.cpu()

    tensor = tensor.detach()

    # Note: numpy does not support bfloat16.
    if tensor.dtype == torch.bfloat16:
        return mx.array(tensor)

    return mx.array(tensor.numpy())
```
**EN:** This function implements `torch_to_mlx`. It primarily calls `tensor.detach`, `mx.array`, `tensor.cpu`, `tensor.numpy` to complete its work. State updates are written into `tensor`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `torch_to_mlx`。 它主要通过调用 `tensor.detach`, `mx.array`, `tensor.cpu`, `tensor.numpy` 来完成任务。 状态更新主要写入 `tensor`。 实现中使用了条件分支。

### Lines 139-197: Function `mlx_to_torch` / 函数 `mlx_to_torch`
```python
def mlx_to_torch(
    array: mx.array,
    device: torch.device | Literal["mps", "cpu"] | None = None,
    already_contiguous: bool = False,
) -> torch.Tensor:
    """Convert MLX array to PyTorch tensor.

    Uses numpy as an intermediate to enable zero-copy on unified memory.

    Args:
        array: MLX array
        device: Target PyTorch device (default: MPS if available)
        already_contiguous: Skip contiguity check if array is known contiguous

    Returns:
        PyTorch tensor with the same data
    """
    if device is None:
        device = get_torch_device()
    elif isinstance(device, str):
        device = torch.device(device)

    # Use memoryview for zero-copy conversion (bypasses numpy for bfloat16)
    # reference: https://github.com/ml-explore/mlx/issues/403
    torch_dtype = MLX_TO_TORCH_DTYPE.get(array.dtype)
    if torch_dtype is not None:
        if already_contiguous:
            # Fast path: skip contiguity check, single eval
# ... omitted for brevity ...
                _MPS_SAFE_SIZE_BYTES,
            )
    elif device.type != "cpu":
        tensor = tensor.to(device)

    return tensor
```
**EN:** This function implements `mlx_to_torch`. It primarily calls `MLX_TO_TORCH_DTYPE.get`, `get_torch_device`, `isinstance`, `torch.frombuffer.reshape`, `ValueError`, `_is_safe_for_mps` to complete its work. State updates are written into `torch_dtype`, `device`, `tensor`, `buffer`, `array`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `mlx_to_torch`。 它主要通过调用 `MLX_TO_TORCH_DTYPE.get`, `get_torch_device`, `isinstance`, `torch.frombuffer.reshape`, `ValueError`, `_is_safe_for_mps` 来完成任务。 状态更新主要写入 `torch_dtype`, `device`, `tensor`, `buffer`, `array`。 实现中使用了条件分支。

### Lines 200-210: Function `sync_mlx` / 函数 `sync_mlx`
```python
def sync_mlx() -> None:
    """Synchronize MLX operations.

    Call this before converting MLX arrays to ensure all operations complete.
    """
    # Prefer an explicit MLX barrier when available; otherwise force evaluation.
    # `mx.eval([])` is a no-op, so we evaluate a tiny scalar as a safe fallback.
    try:
        mx.synchronize()
    except (AttributeError, TypeError):
        mx.eval(mx.array(0, dtype=mx.int32))
```
**EN:** This function implements `sync_mlx`. It primarily calls `mx.synchronize`, `mx.eval`, `mx.array` to complete its work. The implementation relies on error handling.
**CN:** 该函数实现了 `sync_mlx`。 它主要通过调用 `mx.synchronize`, `mx.eval`, `mx.array` 来完成任务。 实现中使用了错误处理。

### Lines 213-219: Function `sync_torch` / 函数 `sync_torch`
```python
def sync_torch() -> None:
    """Synchronize PyTorch MPS operations.

    Call this before converting PyTorch tensors to ensure all operations complete.
    """
    if torch.backends.mps.is_available():
        torch.mps.synchronize()
```
**EN:** This function implements `sync_torch`. It primarily calls `torch.backends.mps.is_available`, `torch.mps.synchronize` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `sync_torch`。 它主要通过调用 `torch.backends.mps.is_available`, `torch.mps.synchronize` 来完成任务。 实现中使用了条件分支。

### Lines 222-228: Constants and shared state / 常量与共享状态
```python
__all__ = [
    "is_mlx_available",
    "use_mlx",
    "mlx_to_torch",
    "torch_to_mlx",
    "get_torch_device",
]
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `__all__`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `__all__`。

## Key Concepts / 关键概念
- **Functions / 函数**: `is_mlx_available`, `use_mlx`, `get_torch_device`, `_get_tensor_size_bytes`, `_is_safe_for_mps`, `torch_to_mlx`, `mlx_to_torch`, `sync_mlx`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`
- **External / 外部依赖**: `torch`, `mlx.core`
- **Standard library / 标准库**: `__future__`, `logging`, `functools`, `typing`
