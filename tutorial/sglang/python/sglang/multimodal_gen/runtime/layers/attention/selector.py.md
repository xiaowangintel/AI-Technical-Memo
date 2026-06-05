# selector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/selector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `ComponentAttnBackendContext`, `backend_name_to_enum`, and `get_env_variable_attn_backend`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `ComponentAttnBackendContext`、`backend_name_to_enum` 和 `get_env_variable_attn_backend` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 6-21: module setup and imports / 模块初始化与导入
```python
import os
from collections.abc import Generator
from contextlib import contextmanager
from contextvars import ContextVar
from functools import cache
from typing import NamedTuple, cast

import torch

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.server_args import get_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import STR_BACKEND_ENV_VAR, resolve_obj_by_qualname
```
**EN:** This block establishes the module context and imports `os`, `collections.abc`, `contextlib`, `contextvars`, `functools`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `os`、`collections.abc`、`contextlib`、`contextvars`、`functools` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 23-23: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 26-40: `backend_name_to_enum` implementation / `backend_name_to_enum` 实现
```python
def backend_name_to_enum(backend_name: str) -> AttentionBackendEnum | None:
    """
    Convert a string backend name to a _Backend enum value.

    Returns:
    * _Backend: enum value if backend_name is a valid in-tree type
    * None: otherwise it's an invalid in-tree type or an out-of-tree platform is
            loaded.
    """
    assert backend_name is not None
    return (
        AttentionBackendEnum[backend_name]
        if backend_name in AttentionBackendEnum.__members__
        else None
    )
```
**EN:** This block defines function `backend_name_to_enum`. Convert a string backend name to a _Backend enum value. Returns: * _Backend: enum value if backend_name is a valid in-tree type * None: otherwise it's an invalid in-tree type or an out-of-tree platform is loaded. Parameters such as `backend_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `backend_name_to_enum`。 它用于处理 backend name to enum 相关逻辑。 本段逻辑主要由 `backend_name` 等参数驱动。

### Lines 43-54: `get_env_variable_attn_backend` implementation / `get_env_variable_attn_backend` 实现
```python
def get_env_variable_attn_backend() -> AttentionBackendEnum | None:
    """
    Get the backend override specified by the sglang-diffusion attention
    backend environment variable, if one is specified.

    Returns:

    * _Backend enum value if an override is specified
    * None otherwise
    """
    backend_name = os.environ.get(STR_BACKEND_ENV_VAR)
    return None if backend_name is None else backend_name_to_enum(backend_name)
```
**EN:** This block defines function `get_env_variable_attn_backend`. Get the backend override specified by the sglang-diffusion attention backend environment variable, if one is specified. Returns: * _Backend enum value if an override is specified * None otherwise Key calls include `os.environ.get`, and `backend_name_to_enum`.
**CN:** 该代码块定义了函数 `get_env_variable_attn_backend`。 它用于获取env variable attn backend。 关键调用包括 `os.environ.get` 和 `backend_name_to_enum`。

### Lines 64-64: supporting statements / 辅助语句
```python
forced_attn_backend: AttentionBackendEnum | None = None
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `forced_attn_backend`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `forced_attn_backend` 等名称。

### Lines 67-67: `ComponentAttnBackendContext` class overview / `ComponentAttnBackendContext` 类概览
```python
class ComponentAttnBackendContext(NamedTuple):
```
**EN:** This block defines class `ComponentAttnBackendContext`. It encapsulates component attn backend context behavior. It inherits from `NamedTuple`.
**CN:** 该代码块定义了类 `ComponentAttnBackendContext`。 它用于封装 component attn backend context 相关行为。 它继承自 `NamedTuple`。

### Lines 68-69: supporting statements / 辅助语句
```python
    backend: AttentionBackendEnum | None
    component_name: str | None
```
**EN:** This block gathers supporting statements inside `ComponentAttnBackendContext`. It updates names such as `backend`, and `component_name`.
**CN:** 该代码块汇集了位于 `ComponentAttnBackendContext` 内部的辅助语句。 它会更新 `backend` 和 `component_name` 等名称。

### Lines 72-74: supporting statements / 辅助语句
```python
component_attn_backend_context: ContextVar[ComponentAttnBackendContext | None] = (
    ContextVar("component_attn_backend_context", default=None)
)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `component_attn_backend_context`. The code collaborates with `ContextVar`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `component_attn_backend_context` 等名称。 代码会与 `ContextVar` 协同工作。

### Lines 77-89: `global_force_attn_backend` implementation / `global_force_attn_backend` 实现
```python
def global_force_attn_backend(attn_backend: AttentionBackendEnum | None) -> None:
    """
    Force all attention operations to use a specified backend.

    Passing `None` for the argument re-enables automatic
    backend selection.,

    Arguments:

    * attn_backend: backend selection (None to revert to auto)
    """
    global forced_attn_backend
    forced_attn_backend = attn_backend
```
**EN:** This block defines function `global_force_attn_backend`. Force all attention operations to use a specified backend. Passing `None` for the argument re-enables automatic backend selection., Arguments: * attn_backend: backend selection (None to revert to auto) Parameters such as `attn_backend` drive the behavior in this section.
**CN:** 该代码块定义了函数 `global_force_attn_backend`。 它用于处理 global force attn backend 相关逻辑。 本段逻辑主要由 `attn_backend` 等参数驱动。

### Lines 92-97: `get_global_forced_attn_backend` implementation / `get_global_forced_attn_backend` 实现
```python
def get_global_forced_attn_backend() -> AttentionBackendEnum | None:
    """
    Get the currently-forced choice of attention backend,
    or None if auto-selection is currently enabled.
    """
    return forced_attn_backend
```
**EN:** This block defines function `get_global_forced_attn_backend`. Get the currently-forced choice of attention backend, or None if auto-selection is currently enabled.
**CN:** 该代码块定义了函数 `get_global_forced_attn_backend`。 它用于获取global forced attn backend。

### Lines 100-101: `get_component_attn_backend_context` implementation / `get_component_attn_backend_context` 实现
```python
def get_component_attn_backend_context() -> ComponentAttnBackendContext | None:
    return component_attn_backend_context.get()
```
**EN:** This block defines function `get_component_attn_backend_context`. It retrieves component attn backend context. Key calls include `component_attn_backend_context.get`.
**CN:** 该代码块定义了函数 `get_component_attn_backend_context`。 它用于获取component attn backend context。 关键调用包括 `component_attn_backend_context.get`。

### Lines 104-106: `get_component_forced_attn_backend` implementation / `get_component_forced_attn_backend` 实现
```python
def get_component_forced_attn_backend() -> AttentionBackendEnum | None:
    context = get_component_attn_backend_context()
    return context.backend if context is not None else None
```
**EN:** This block defines function `get_component_forced_attn_backend`. It retrieves component forced attn backend. Key calls include `get_component_attn_backend_context`.
**CN:** 该代码块定义了函数 `get_component_forced_attn_backend`。 它用于获取component forced attn backend。 关键调用包括 `get_component_attn_backend_context`。

### Lines 109-111: `get_component_attn_backend_name` implementation / `get_component_attn_backend_name` 实现
```python
def get_component_attn_backend_name() -> str | None:
    context = get_component_attn_backend_context()
    return context.component_name if context is not None else None
```
**EN:** This block defines function `get_component_attn_backend_name`. It retrieves component attn backend name. Key calls include `get_component_attn_backend_context`.
**CN:** 该代码块定义了函数 `get_component_attn_backend_name`。 它用于获取component attn backend name。 关键调用包括 `get_component_attn_backend_context`。

### Lines 114-161: `get_attn_backend` implementation / `get_attn_backend` 实现
```python
def get_attn_backend(
    head_size: int,
    dtype: torch.dtype,
    supported_attention_backends: set[AttentionBackendEnum] | None = None,
    selected_attention_backend: AttentionBackendEnum | None = None,
) -> type[AttentionBackend]:
    if supported_attention_backends is None:
        be_tuple = tuple()
    else:
        # Sort the backend names to ensure consistent cache key
        be_tuple = tuple(
            sorted(list(supported_attention_backends), key=lambda b: b.name)
        )

    selected_backend = selected_attention_backend or get_global_forced_attn_backend()
    if selected_backend is None:
        selected_backend = get_component_forced_attn_backend()
    if selected_backend is None:
        server_args = get_global_server_args()
        if server_args.attention_backend is not None:
            try:
                selected_backend = AttentionBackendEnum[
                    server_args.attention_backend.upper()
                ]
            except KeyError:
                raise ValueError(
                    f"Invalid attention backend '{server_args.attention_backend}' specified via command line. "
                    f"Available options are: {[e.name.lower() for e in AttentionBackendEnum]}"
                )

    component_name = get_component_attn_backend_name()
    backend_not_specified = selected_backend is None
    attention_backend_cls = _cached_get_attn_backend(
        head_size,
        dtype,
        be_tuple,
        selected_backend,
    )
    if component_name:
        backend_name = attention_backend_cls.get_enum().name.lower()
        if backend_not_specified:
            logger.info_once(
                f"Attention backend not specified for {component_name}, "
                f"using {backend_name} backend for {component_name}"
            )
        else:
            logger.info_once(f"Using {backend_name} backend for {component_name}")
    return attention_backend_cls
```
**EN:** This block defines function `get_attn_backend`. It retrieves attn backend. Key calls include `get_component_attn_backend_name`, `_cached_get_attn_backend`, `tuple`, `get_global_forced_attn_backend`, and `get_component_forced_attn_backend`. The implementation branches on conditions, handles exceptional paths. Parameters such as `head_size`, `dtype`, `supported_attention_backends`, and `selected_attention_backend` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_attn_backend`。 它用于获取attn backend。 关键调用包括 `get_component_attn_backend_name`、`_cached_get_attn_backend`、`tuple`、`get_global_forced_attn_backend` 和 `get_component_forced_attn_backend`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `head_size`、`dtype`、`supported_attention_backends` 和 `selected_attention_backend` 等参数驱动。

### Lines 164-202: `_cached_get_attn_backend` implementation / `_cached_get_attn_backend` 实现
```python
@cache
def _cached_get_attn_backend(
    head_size: int,
    dtype: torch.dtype,
    supported_attention_backends: tuple[AttentionBackendEnum],
    selected_backend: AttentionBackendEnum | None,
) -> type[AttentionBackend]:
    from sglang.multimodal_gen.runtime.platforms import current_platform

    supported_attention_backends = set(supported_attention_backends)

    # get device-specific attn_backend
    if len(supported_attention_backends) == 0:
        # all attention backends are allowed
        pass
    elif selected_backend is None and len(supported_attention_backends) == 1:
        selected_backend = next(iter(supported_attention_backends))
    elif selected_backend is None:
        logger.debug("Attention backend not specified")
    elif selected_backend not in supported_attention_backends:
        supported_attention_backends_str = [
            supported_attention_backend.__str__()
            for supported_attention_backend in supported_attention_backends
        ]
        logger.debug(
            "Selected attention backend: '%s' not in supported attention backends: %s",
            selected_backend,
            supported_attention_backends_str,
        )
        selected_backend = None

    attention_cls = current_platform.get_attn_backend_cls_str(
        selected_backend, head_size, dtype
    )
    if not attention_cls:
        raise ValueError(
            f"Invalid attention backend for {current_platform.device_name}"
        )
    return cast(type[AttentionBackend], resolve_obj_by_qualname(attention_cls))
```
**EN:** This block defines function `_cached_get_attn_backend`. It handles cached get attn backend logic. Key calls include `set`, `current_platform.get_attn_backend_cls_str`, `cast`, `len`, and `ValueError`. The implementation branches on conditions. Parameters such as `head_size`, `dtype`, `supported_attention_backends`, and `selected_backend` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_cached_get_attn_backend`。 它用于处理 cached get attn backend 相关逻辑。 关键调用包括 `set`、`current_platform.get_attn_backend_cls_str`、`cast`、`len` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `head_size`、`dtype`、`supported_attention_backends` 和 `selected_backend` 等参数驱动。

### Lines 205-220: `component_attn_backend_context_manager` implementation / `component_attn_backend_context_manager` 实现
```python
@contextmanager
def component_attn_backend_context_manager(
    attn_backend: AttentionBackendEnum | None,
    component_name: str | None = None,
) -> Generator[None, None, None]:
    if attn_backend is None and component_name is None:
        yield
        return

    token = component_attn_backend_context.set(
        ComponentAttnBackendContext(attn_backend, component_name)
    )
    try:
        yield
    finally:
        component_attn_backend_context.reset(token)
```
**EN:** This block defines function `component_attn_backend_context_manager`. It handles component attn backend context manager logic. Key calls include `component_attn_backend_context.set`, `ComponentAttnBackendContext`, and `component_attn_backend_context.reset`. The implementation branches on conditions, handles exceptional paths. Parameters such as `attn_backend`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `component_attn_backend_context_manager`。 它用于处理 component attn backend context manager 相关逻辑。 关键调用包括 `component_attn_backend_context.set`、`ComponentAttnBackendContext` 和 `component_attn_backend_context.reset`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `attn_backend` 和 `component_name` 等参数驱动。

### Lines 223-252: `global_force_attn_backend_context_manager` implementation / `global_force_attn_backend_context_manager` 实现
```python
@contextmanager
def global_force_attn_backend_context_manager(
    attn_backend: AttentionBackendEnum,
) -> Generator[None, None, None]:
    """
    Globally force a sglang-diffusion attention backend override within a
    context manager, reverting the global attention backend
    override to its prior state upon exiting the context
    manager.

    Arguments:
    * attn_backend: attention backend to force

    Returns:

    * Generator
    """

    # Save the current state of the global backend override (if any)
    original_value = get_global_forced_attn_backend()

    # Globally force the new backend override
    global_force_attn_backend(attn_backend)

    # Yield control back to the enclosed code block
    try:
        yield
    finally:
        # Revert the original global backend override, if any
        global_force_attn_backend(original_value)
```
**EN:** This block defines function `global_force_attn_backend_context_manager`. Globally force a sglang-diffusion attention backend override within a context manager, reverting the global attention backend override to its prior state upon exiting the context manager. Arguments: * attn_backend: attention backend to force Returns: * Generator Key calls include `get_global_forced_attn_backend`, and `global_force_attn_backend`. The implementation handles exceptional paths. Parameters such as `attn_backend` drive the behavior in this section.
**CN:** 该代码块定义了函数 `global_force_attn_backend_context_manager`。 它用于处理 global force attn backend context manager 相关逻辑。 关键调用包括 `get_global_forced_attn_backend` 和 `global_force_attn_backend`。 实现中处理异常路径。 本段逻辑主要由 `attn_backend` 等参数驱动。

## Key Concepts / 关键概念
- `backend_name_to_enum`: Convert a string backend name to a _Backend enum value. / 顶层函数，用于处理 backend name to enum 相关逻辑。
- `get_env_variable_attn_backend`: Get the backend override specified by the sglang-diffusion attention backend environment variable, if one is specified. / 顶层函数，用于获取env variable attn backend。
- `ComponentAttnBackendContext`: Primary class that encapsulates component attn backend context behavior. / 核心类，用于封装 component attn backend context 相关行为。
- `global_force_attn_backend`: Force all attention operations to use a specified backend. / 顶层函数，用于处理 global force attn backend 相关逻辑。
- `get_global_forced_attn_backend`: Get the currently-forced choice of attention backend, or None if auto-selection is currently enabled. / 顶层函数，用于获取global forced attn backend。
- `get_component_attn_backend_context`: Top-level function that retrieves component attn backend context. / 顶层函数，用于获取component attn backend context。
- `get_component_forced_attn_backend`: Top-level function that retrieves component forced attn backend. / 顶层函数，用于获取component forced attn backend。
- `get_component_attn_backend_name`: Top-level function that retrieves component attn backend name. / 顶层函数，用于获取component attn backend name。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `collections.abc`, `contextlib`, `contextvars`, `functools`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 252
