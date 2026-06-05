# torch_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/torch_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-73)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import contextlib
import importlib.metadata
import os
import platform
import random
import threading
from collections.abc import Callable, Collection
from typing import TYPE_CHECKING, Any, TypeVar

import numpy as np
import numpy.typing as npt
import torch
from packaging import version
from packaging.version import Version
from torch.library import Library, infer_schema

import vllm.envs as envs
from vllm.logger import init_logger

if TYPE_CHECKING:
    from vllm.config import ModelConfig
    from vllm.sequence import IntermediateTensors
    # ...
T = TypeVar("T")


# Pin memory in non-WSL case.
# Logic duplicated here for now to avoid circular import.
PIN_MEMORY = "microsoft" not in " ".join(platform.uname()).lower()
```
**EN:** Sets up the module with standard-library support such as `contextlib`, `importlib.metadata`, `os`, external packages such as `numpy`, `numpy.typing`, `torch`, vLLM modules such as `vllm.envs`, `vllm.logger`, `vllm.config`. It prepares the symbols later used by `_StreamPlaceholder`, `LayerName`, `is_quantized_kv_cache`, `kv_cache_uses_per_token_head_scales`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.logger`, `vllm.config` 等 vLLM 内部依赖。 这些准备工作为后续的 `_StreamPlaceholder`, `LayerName`, `is_quantized_kv_cache`, `kv_cache_uses_per_token_head_scales` 提供上下文。

### is_strictly_contiguous (lines 89-116)
```python
def is_strictly_contiguous(t: torch.Tensor) -> bool:
    """
    Check if tensor is contiguous AND has no degenerate strides.

    A degenerate stride occurs when a dimension has size 1 but the stride
    doesn't match the canonical contiguous layout. This can cause issues
    in some CUDA kernels that rely on stride values for memory access.

    For a C-contiguous tensor of shape (d0, d1, ..., dn), the expected
    strides are: stride[i] = product(shape[i+1:]) for all i, with stride[-1]=1.

    Example with torch.Size([16, 1, 8, 32]):
        - Canonical strides: (256, 256, 32, 1)
        - Degenerate strides: (256, 1, 32, 1)  # dim=1 has size=1, allowing
                                                  # non-canonical stride in dim=0
    """
    if not t.is_contiguous():
        return False

    # Check that strides match canonical contiguous layout
    shape = t.shape
    strides = t.stride()
    expected_stride = 1
    for i in range(len(shape) - 1, -1, -1):
        if strides[i] != expected_stride:
            return False
        expected_stride *= shape[i]
    return True
```
**EN:** `is_strictly_contiguous`: Check if tensor is contiguous AND has no degenerate strides. It mainly works with `t`. Inside the body, it relies on `t.stride`, `t.is_contiguous` to complete the main steps.
**CN:** `is_strictly_contiguous` 负责检查条件并返回布尔结果。 它主要处理 `t` 等参数。 实现过程中会调用 `t.stride`, `t.is_contiguous` 等函数完成关键步骤。

### set_default_torch_num_threads (lines 155-184)
```python
def set_default_torch_num_threads(num_threads: int | None = None):
    """
    Sets the default number of threads for PyTorch to the given value.

    `None` means using the value of the environment variable `OMP_NUM_THREADS`
    (or `1` if that is not available).
    """
    if num_threads is None:
        num_threads = 1

        try:
            num_threads = int(os.environ["OMP_NUM_THREADS"])
        except KeyError:
            logger.debug_once(
                "OMP_NUM_THREADS is not set; defaulting Torch threads to %d.",
                num_threads,
            )
        except ValueError:
            logger.warning_once(
                "OMP_NUM_THREADS is invalid; defaulting Torch threads to %d.",
                num_threads,
            )

    old_num_threads = torch.get_num_threads()
    torch.set_num_threads(num_threads)

    try:
        yield
    finally:
        torch.set_num_threads(old_num_threads)
```
**EN:** `set_default_torch_num_threads`: Sets the default number of threads for PyTorch to the given value. It mainly works with `num_threads`. Inside the body, it relies on `torch.get_num_threads`, `torch.set_num_threads`, `logger.debug_once` to complete the main steps.
**CN:** `set_default_torch_num_threads` 负责把值写入对象或配置结构。 它主要处理 `num_threads` 等参数。 实现过程中会调用 `torch.get_num_threads`, `torch.set_num_threads`, `logger.debug_once` 等函数完成关键步骤。

### guard_cuda_initialization (lines 188-210)
```python
def guard_cuda_initialization():
    """Avoid unexpected CUDA initialization."""
    from vllm.platforms import current_platform

    if not current_platform.is_cuda():
        yield
        return

    old_value = os.environ.get("CUDA_VISIBLE_DEVICES")
    os.environ["CUDA_VISIBLE_DEVICES"] = ""
    try:
        yield
    except Exception as e:
        if "No CUDA GPUs are available" in str(e):
            err_msg = "CUDA initialization is blocked."
        else:
            err_msg = str(e)
        raise RuntimeError(err_msg) from e
    finally:
        if old_value is None:
            del os.environ["CUDA_VISIBLE_DEVICES"]
        else:
            os.environ["CUDA_VISIBLE_DEVICES"] = old_value
```
**EN:** `guard_cuda_initialization`: Avoid unexpected CUDA initialization. Inside the body, it relies on `os.environ.get`, `current_platform.is_cuda`, `RuntimeError` to complete the main steps.
**CN:** `guard_cuda_initialization` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `os.environ.get`, `current_platform.is_cuda`, `RuntimeError` 等函数完成关键步骤。

### is_lossless_cast (lines 224-253)
```python
def is_lossless_cast(src_dtype: torch.dtype, tgt_dtype: torch.dtype):
    """
    Test whether it is lossless to cast a tensor from
    `src_dtype` to `tgt_dtype`.
    """
    if src_dtype == tgt_dtype:
        return True

    src_level = _get_precision_level(src_dtype)
    tgt_level = _get_precision_level(tgt_dtype)

    if src_level < tgt_level:
        return True
    if src_level > tgt_level:
        return False

    # Compare integral types
    if not src_dtype.is_floating_point and not src_dtype.is_complex:
        src_info = torch.iinfo(src_dtype)
        tgt_info = torch.iinfo(tgt_dtype)
        return src_info.min >= tgt_info.min and src_info.max <= tgt_info.max

    # Compare floating-point types
    src_info = torch.finfo(src_dtype)
    tgt_info = torch.finfo(tgt_dtype)
    return (
        src_info.min >= tgt_info.min
        and src_info.max <= tgt_info.max
        and src_info.resolution >= tgt_info.resolution
    )
```
**EN:** `is_lossless_cast`: Test whether it is lossless to cast a tensor from `src_dtype` to `tgt_dtype`. It mainly works with `src_dtype`, `tgt_dtype`. Inside the body, it relies on `_get_precision_level`, `torch.finfo`, `torch.iinfo` to complete the main steps.
**CN:** `is_lossless_cast` 负责检查条件并返回布尔结果。 它主要处理 `src_dtype`, `tgt_dtype` 等参数。 实现过程中会调用 `_get_precision_level`, `torch.finfo`, `torch.iinfo` 等函数完成关键步骤。

### get_kv_cache_quant_algo_string (lines 311-363)
```python
def get_kv_cache_quant_algo_string(quant_cfg: dict[str, Any]) -> str | None:
    """Get the KV cache quantization algorithm string from the quantization config.

    Maps various FP8 format names to vLLM's standard cache dtype strings.
    Returns None if no kv_cache_quant_algo is specified.
    Returns "auto" if the value is not recognized/supported.
    """
    # Mapping from model config values to vLLM cache_dtype strings

    quant_method = quant_cfg.get("quant_method", "")
    if quant_method.startswith("modelopt"):
        quantization_inner = quant_cfg.get("quantization", quant_cfg)
        # Check if quant config is specified and use kv cache quant algo
        kv_algo = (
            quantization_inner.get("kv_cache_scheme")
            or quant_cfg.get("kv_cache_scheme")
            or quantization_inner.get("kv_cache_quant_algo")
            or quant_cfg.get("kv_cache_quant_algo")
        )
        if isinstance(kv_algo, dict):
            if (
                kv_algo.get("dynamic") is False
                and kv_algo.get("num_bits") == 8
                and kv_algo.get("type") == "float"
    # ...
                    "config. Supported values: %s. Falling back to 'auto'.",
                    kv_algo,
                    list(MODELOPT_TO_VLLM_KV_CACHE_DTYPE_MAP.keys()),
                )
                return "auto"
    return None
```
**EN:** `get_kv_cache_quant_algo_string`: Get the KV cache quantization algorithm string from the quantization config. It mainly works with `quant_cfg`. Inside the body, it relies on `quant_cfg.get`, `quant_method.startswith`, `quantization_inner.get` to complete the main steps.
**CN:** `get_kv_cache_quant_algo_string` 负责获取流水线所需的数据或状态。 它主要处理 `quant_cfg` 等参数。 实现过程中会调用 `quant_cfg.get`, `quant_method.startswith`, `quantization_inner.get` 等函数完成关键步骤。

### _nvfp4_split_data_scale (lines 420-469)
```python
def _nvfp4_split_data_scale(
    kv_side: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Split a single NVFP4 KV-side buffer into data and scale views.

    The input is a 4D tensor for one KV side (K or V) whose last
    dimension is ``full_dim = data_dim + scale_dim``.  The physical
    layout within each side is [data | scale], both packed contiguously.

    Args:
        kv_side: 4D uint8 tensor with shape
            ``(num_pages, dim_1, dim_2, full_dim)``.
            May be in any permutation order (NHD or HND).

    Returns:
        ``(data, scale)`` where
        ``data`` is a uint8 view with shape
        ``(num_pages, dim_1, dim_2, data_dim)``.
        ``scale`` is a float8_e4m3fn view with shape
        ``(num_pages, dim_1, dim_2, scale_dim)``.
    """
    num_pages = kv_side.shape[0]
    dim_1, dim_2 = kv_side.shape[1], kv_side.shape[2]
    full_dim = kv_side.shape[3]
    # ...
    data = torch.as_strided(kv_side, data_shape, data_strides, storage_offset=base)
    scale = torch.as_strided(
        kv_side, scale_shape, scale_strides, storage_offset=base + data_per_kv
    ).view(torch.float8_e4m3fn)

    return data, scale
```
**EN:** `_nvfp4_split_data_scale`: Split a single NVFP4 KV-side buffer into data and scale views. It mainly works with `kv_side`. Inside the body, it relies on `kv_side.stride`, `kv_side.storage_offset`, `torch.as_strided` to complete the main steps.
**CN:** `_nvfp4_split_data_scale` 负责实现本模块使用的辅助逻辑。 它主要处理 `kv_side` 等参数。 实现过程中会调用 `kv_side.stride`, `kv_side.storage_offset`, `torch.as_strided` 等函数完成关键步骤。

### nvfp4_kv_cache_split_views (lines 472-501)
```python
def nvfp4_kv_cache_split_views(kv_cache: torch.Tensor) -> tuple[tuple, tuple]:
    """Split an NVFP4 KV cache tensor into data and scale views.

    Accepts either a 5D tensor ``(num_pages, 2, dim_2, dim_3, full_dim)``
    or a 4D single-side tensor ``(num_pages, dim_2, dim_3, full_dim)``.

    Per-page layout: [K_data | K_scale | V_data | V_scale].
    Each KV side is self-contained (data followed by its scale), so the
    5D case simply splits each side independently.

    The returned views are in the same dim order as the input (NHD or
    HND), so callers get views matching whichever order they passed in.

    Args:
        kv_cache: 5D or 4D uint8 tensor where the last dimension is
            ``full_dim = data_dim + scale_dim = 9 * head_size / 16``.

    Returns:
        For 5D input:
            ``(k_data, v_data), (k_scale, v_scale)``
        For 4D input (single KV side):
            ``(data,), (scale,)``
    """
    if kv_cache.dim() == 4:
        data, scale = _nvfp4_split_data_scale(kv_cache)
        return (data,), (scale,)

    k_data, k_scale = _nvfp4_split_data_scale(kv_cache[:, 0])
    v_data, v_scale = _nvfp4_split_data_scale(kv_cache[:, 1])
    return (k_data, v_data), (k_scale, v_scale)
```
**EN:** `nvfp4_kv_cache_split_views`: Split an NVFP4 KV cache tensor into data and scale views. It mainly works with `kv_cache`. Inside the body, it relies on `_nvfp4_split_data_scale`, `kv_cache.dim` to complete the main steps.
**CN:** `nvfp4_kv_cache_split_views` 负责实现本模块使用的辅助逻辑。 它主要处理 `kv_cache` 等参数。 实现过程中会调用 `_nvfp4_split_data_scale`, `kv_cache.dim` 等函数完成关键步骤。

### create_kv_caches_with_random_flash (lines 504-557)
```python
def create_kv_caches_with_random_flash(
    num_blocks: int,
    block_size: int,
    num_layers: int,
    num_heads: int,
    head_size: int,
    cache_dtype: str | torch.dtype | None,
    model_dtype: str | torch.dtype | None = None,
    seed: int | None = None,
    device: str | None = "cuda",
    cache_layout: str | None = "NHD",
) -> tuple[list[torch.Tensor], list[torch.Tensor]]:
    set_random_seed(seed)

    dtype = get_kv_cache_torch_dtype(cache_dtype, model_dtype)
    generic_kv_cache_shape = (num_blocks, 2, block_size, num_heads, head_size)
    assert cache_layout in ("NHD", "HND")
    stride_order = (0, 1, 2, 3, 4) if cache_layout == "NHD" else (0, 1, 3, 2, 4)

    kv_cache_allocation_shape = tuple(generic_kv_cache_shape[i] for i in stride_order)
    scale = head_size**-0.5

    key_caches: list[torch.Tensor] = []
    value_caches: list[torch.Tensor] = []
    # ...
                _generate_random_fp8(key_value_cache, -scale, scale)
            else:
                raise ValueError(f"Does not support key cache of type {cache_dtype}")
        key_caches.append(key_value_cache[:, 0])
        value_caches.append(key_value_cache[:, 1])
    return key_caches, value_caches
```
**EN:** `create_kv_caches_with_random_flash` constructs new objects or configuration bundles. It mainly works with `num_blocks`, `block_size`, `num_layers`, `num_heads`. Inside the body, it relies on `set_random_seed`, `get_kv_cache_torch_dtype`, `key_caches.append` to complete the main steps.
**CN:** `create_kv_caches_with_random_flash` 负责构造新的对象或配置集合。 它主要处理 `num_blocks`, `block_size`, `num_layers`, `num_heads` 等参数。 实现过程中会调用 `set_random_seed`, `get_kv_cache_torch_dtype`, `key_caches.append` 等函数完成关键步骤。

### create_kv_caches_with_random (lines 560-605)
```python
def create_kv_caches_with_random(
    num_blocks: int,
    block_size: int,
    num_layers: int,
    num_heads: int,
    head_size: int,
    cache_dtype: str | torch.dtype | None,
    model_dtype: str | torch.dtype | None = None,
    seed: int | None = None,
    device: str | None = "cuda",
) -> tuple[list[torch.Tensor], list[torch.Tensor]]:
    if cache_dtype == "fp8" and head_size % 16:
        raise ValueError(
            f"Does not support key cache of type fp8 with head_size {head_size}"
        )

    set_random_seed(seed)

    dtype = get_kv_cache_torch_dtype(cache_dtype, model_dtype)

    scale = head_size**-0.5
    x = 16 // torch.tensor([], dtype=dtype).element_size()
    key_cache_shape = (num_blocks, num_heads, head_size // x, block_size, x)
    key_caches: list[torch.Tensor] = []
    # ...
        elif cache_dtype == "fp8":
            _generate_random_fp8(value_cache, -scale, scale)
        else:
            raise ValueError(f"Does not support value cache of type {cache_dtype}")
        value_caches.append(value_cache)
    return key_caches, value_caches
```
**EN:** `create_kv_caches_with_random` constructs new objects or configuration bundles. It mainly works with `num_blocks`, `block_size`, `num_layers`, `num_heads`. Inside the body, it relies on `set_random_seed`, `get_kv_cache_torch_dtype`, `ValueError` to complete the main steps.
**CN:** `create_kv_caches_with_random` 负责构造新的对象或配置集合。 它主要处理 `num_blocks`, `block_size`, `num_layers`, `num_heads` 等参数。 实现过程中会调用 `set_random_seed`, `get_kv_cache_torch_dtype`, `ValueError` 等函数完成关键步骤。

### current_stream (lines 687-724)
```python
def current_stream() -> torch.cuda.Stream:
    """
    replace `torch.cuda.current_stream()` with `vllm.utils.current_stream()`.
    it turns out that `torch.cuda.current_stream()` is quite expensive,
    as it will construct a new stream object at each call.
    here we patch `torch.cuda.set_stream` to keep track of the current stream
    directly, so that we can avoid calling `torch.cuda.current_stream()`.

    the underlying hypothesis is that we do not call `torch._C._cuda_setStream`
    from C/C++ code.
    """
    from vllm.platforms import current_platform

    if not hasattr(_current_stream_tls, "value") or _current_stream_tls.value is None:
        # when this function is called before any stream is set,
        # we return the default stream.
        # On ROCm using the default 0 stream in combination with RCCL
        # is hurting performance.
        # On CUDA, we capture and replay cudagraph on the same stream,
        # so we need to avoid using the default stream as well. The default
        # stream cannot be used for cudagraph capture, see
        # https://github.com/pytorch/pytorch/blob/42ad9edfb754743fdae3276ade43de000beb4f60/aten/src/ATen/cuda/CUDAGraph.cpp#L77
        # for more details. Therefore, we create a dedicated stream per process.
        if current_platform.is_rocm() or current_platform.is_cuda():
    # ...
            else:
                raise ValueError(
                    "Fail to set current stream, current platform "
                    "may not support current_stream with torch API"
                )
    return _current_stream_tls.value
```
**EN:** `current_stream`: replace `torch.cuda.current_stream()` with `vllm.utils.current_stream()`. Inside the body, it relies on `current_platform.is_rocm`, `current_platform.is_cuda`, `torch.cuda.set_stream` to complete the main steps.
**CN:** `current_stream` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `current_platform.is_rocm`, `current_platform.is_cuda`, `torch.cuda.set_stream` 等函数完成关键步骤。

### LayerName (lines 870-889)
```python
class LayerName(OpaqueBase):  # type: ignore[misc]
    """Wraps a module name string for use as a torch opaque type.

    When torch >= 2.11, this is registered as a hoisted value-type opaque
    object so that torch.compile lifts it as a graph input instead of baking
    it as a constant.  This avoids per-layer recompilation for custom ops
    that accept layer name strings (attention, MOE, KV cache, etc.).
    """

    def __init__(self, value: str):
        self.value = value

    def __eq__(self, other):
        return isinstance(other, LayerName) and self.value == other.value

    def __hash__(self):
        return hash(self.value)

    def __fx_repr__(self):
        return (f"LayerName({self.value!r})", {"LayerName": LayerName})
```
**EN:** `LayerName`: Wraps a module name string for use as a torch opaque type. It extends `OpaqueBase`. Key methods include `__init__`.
**CN:** `LayerName` 是该文件中的核心类，用于封装与 `LayerName` 相关的状态和行为。 它继承自 `OpaqueBase`。 关键方法包括 `__init__`。

### Additional repeated patterns / 其他重复模式
```python
# Large file omitted for brevity
# ...
```
**EN:** The remaining code mostly expands the same abstractions with more wrappers, constants, or schema variants. The analysis above focuses on the control points that shape overall behavior.
**CN:** 剩余代码主要在相同抽象之上继续展开，补充更多包装函数、常量或模式变体。上面的分析聚焦于决定整体行为的关键控制点。

## Key Concepts / 关键概念
- **`_StreamPlaceholder`**: Core class that organizes module behavior. / **`_StreamPlaceholder`**：组织模块行为的核心类。
- **`LayerName`**: Core class that organizes module behavior. / **`LayerName`**：组织模块行为的核心类。
- **`is_quantized_kv_cache`**: Key helper or entry point in this file. / **`is_quantized_kv_cache`**：本文件中的关键辅助函数或入口。
- **`kv_cache_uses_per_token_head_scales`**: Key helper or entry point in this file. / **`kv_cache_uses_per_token_head_scales`**：本文件中的关键辅助函数或入口。
- **`is_strictly_contiguous`**: Key helper or entry point in this file. / **`is_strictly_contiguous`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: contextlib, importlib.metadata, os, platform, random, threading, collections.abc, typing
- **Third-party / 第三方**: numpy, numpy.typing, torch, packaging, packaging.version, torch.library, torch._opaque_base, torch._library.opaque_object
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.logger, vllm.config, vllm.sequence, vllm.platforms, vllm
