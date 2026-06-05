# fa_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/fa_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `get_flash_attn_version`, `is_fa_version_supported`, `flash_attn_supports_quant_query_input` for the V1 `attention/backends` subsystem. / 为 V1 的 `attention/backends` 子系统实现 `get_flash_attn_version`, `is_fa_version_supported`, `flash_attn_supports_quant_query_input`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from typing import Any

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.platforms import current_platform

logger = init_logger(__name__)

# Track whether upstream flash-attn is available on ROCm.
# Set during module initialization and never modified afterwards.
# This module-level flag avoids repeated import attempts and ensures
# consistent behavior (similar to IS_AITER_FOUND in _aiter_ops.py).
_ROCM_FLASH_ATTN_AVAILABLE = False

if current_platform.is_cuda():
    from vllm._custom_ops import reshape_and_cache_flash
    from vllm.vllm_flash_attn import (  # type: ignore[attr-defined]
        flash_attn_varlen_func,
        get_scheduler_metadata,
    )

elif current_platform.is_xpu():
    from vllm import _custom_ops as ops
    from vllm._xpu_ops import xpu_ops

    reshape_and_cache_flash = ops.reshape_and_cache_flash
    flash_attn_varlen_func = xpu_ops.flash_attn_varlen_func  # type: ignore[assignment]
    get_scheduler_metadata = xpu_ops.get_scheduler_metadata  # type: ignore[assignment]
elif current_platform.is_rocm():
    try:
        from flash_attn import flash_attn_varlen_func  # type: ignore[no-redef]

        # Mark that upstream flash-attn is available on ROCm
        _ROCM_FLASH_ATTN_AVAILABLE = True
    except ImportError:

        def flash_attn_varlen_func(*args: Any, **kwargs: Any) -> Any:  # type: ignore[no-redef,misc]
            raise ImportError(
                "ROCm platform requires upstream flash-attn "
                "to be installed. Please install flash-attn first."
            )

    # ROCm doesn't use scheduler metadata (FA3 feature), provide stub
    def get_scheduler_metadata(*args: Any, **kwargs: Any) -> None:  # type: ignore[misc]
        return None

    # ROCm uses the C++ custom op for reshape_and_cache
    from vllm import _custom_ops as ops

    reshape_and_cache_flash = ops.reshape_and_cache_flash
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `_ROCM_FLASH_ATTN_AVAILABLE`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `_ROCM_FLASH_ATTN_AVAILABLE`。

### `get_flash_attn_version` function / `get_flash_attn_version` 函数
```python
def get_flash_attn_version(
    requires_alibi: bool = False,
    head_size: int | None = None,
    head_size_v: int | None = None,
    has_sinks: bool = False,
) -> int | None:
    if current_platform.is_xpu():
        return 2
    if current_platform.is_rocm():
        # ROCm doesn't use vllm_flash_attn; return None to skip fa_version arg
        return None
    try:
        from vllm.vllm_flash_attn.flash_attn_interface import (
            fa_version_unsupported_reason,
            is_fa_version_supported,
        )

        device_capability = current_platform.get_device_capability()

        assert device_capability is not None

        # 1. default version depending on platform
        if device_capability.major == 9 and is_fa_version_supported(3):
            # Hopper (SM90): prefer FA3
            fa_version = 3
        elif device_capability.major == 10 and is_fa_version_supported(4):
            # Blackwell (SM100+, restrict to SM100 for now): prefer FA4
            fa_version = 4
        else:
            # Fallback to FA2
            fa_version = 2

        # 2. override if passed by environment or config
        from vllm.config import get_current_vllm_config_or_none

        vllm_config = get_current_vllm_config_or_none()
        if (
            vllm_config is not None
            and vllm_config.attention_config.flash_attn_version is not None
        ):
            fa_version = vllm_config.attention_config.flash_attn_version

        # 3. fallback for unsupported combinations
        if device_capability.major >= 10 and fa_version == 3:
            logger.warning_once(
                "Cannot use FA version 3 on Blackwell platform, "
                "defaulting to FA version 4 if supported, otherwise FA2."
            )
            fa_version = 4 if is_fa_version_supported(4) else 2

# ... omitted for brevity ...
        if (
            fa_version == 4
            and device_capability.major >= 10
            and head_size is not None
            and head_size > 128
            and head_size != 192
        ):
            logger.warning_once(
                "FA4 on Blackwell does not support head_size=%d due to TMEM "
                "capacity limits, defaulting to FA version 2.",
                head_size,
            )
            fa_version = 2

        if not is_fa_version_supported(fa_version):
            logger.error(
                "Cannot use FA version %d is not supported due to %s",
                fa_version,
                fa_version_unsupported_reason(fa_version),
            )

        assert is_fa_version_supported(fa_version)
        return fa_version
    except (ImportError, AssertionError):
        return None
```
**EN:** This function returns or derives a value within the module. Key calls include `is_xpu`, `is_rocm`, `get_device_capability`, `get_current_vllm_config_or_none`, `is_fa_version_supported`, `warning_once`. The control flow contains 17 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `is_xpu`, `is_rocm`, `get_device_capability`, `get_current_vllm_config_or_none`, `is_fa_version_supported`, `warning_once`。 控制流包含 17 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `is_fa_version_supported` function / `is_fa_version_supported` 函数
```python
def is_fa_version_supported(fa_version: int) -> bool:
    try:
        from vllm.vllm_flash_attn.flash_attn_interface import (
            is_fa_version_supported as _is_fa_version_supported,
        )

        return _is_fa_version_supported(fa_version)
    except ImportError:
        return False
```
**EN:** This function answers a boolean capability check within the module. Key calls include `_is_fa_version_supported`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。 关键调用包括 `_is_fa_version_supported`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `flash_attn_supports_quant_query_input` function / `flash_attn_supports_quant_query_input` 函数
```python
def flash_attn_supports_quant_query_input() -> bool:
    return not current_platform.is_xpu()
```
**EN:** This function implements `flash_attn_supports_quant_query_input` within the module. Key calls include `is_xpu`.
**CN:** 该函数会实现 `flash_attn_supports_quant_query_input`，其作用域位于the module。 关键调用包括 `is_xpu`。

### `flash_attn_supports_sinks` function / `flash_attn_supports_sinks` 函数
```python
def flash_attn_supports_sinks() -> bool:
    if current_platform.is_xpu():
        return True
    return get_flash_attn_version() in (3, 4)
```
**EN:** This function implements `flash_attn_supports_sinks` within the module. Key calls include `is_xpu`, `get_flash_attn_version`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `flash_attn_supports_sinks`，其作用域位于the module。 关键调用包括 `is_xpu`, `get_flash_attn_version`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `flash_attn_supports_mla` function / `flash_attn_supports_mla` 函数
```python
def flash_attn_supports_mla():
    from vllm.platforms import current_platform

    if current_platform.is_cuda():
        try:
            from vllm.vllm_flash_attn.flash_attn_interface import (
                is_fa_version_supported,
            )

            return is_fa_version_supported(
                3
            ) and current_platform.is_device_capability_family(90)

            # NOTE(Lucas): FA4 CuteDSL does NOT currently support MLA's non-standard
            # head dimensions (576 for qk, 512 for v) due to TMEM capacity limits.

        except (ImportError, AssertionError):
            pass
    return False
```
**EN:** This function implements `flash_attn_supports_mla` within the module. Key calls include `is_cuda`, `is_fa_version_supported`, `is_device_capability_family`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `flash_attn_supports_mla`，其作用域位于the module。 关键调用包括 `is_cuda`, `is_fa_version_supported`, `is_device_capability_family`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `is_flash_attn_varlen_func_available` function / `is_flash_attn_varlen_func_available` 函数
```python
def is_flash_attn_varlen_func_available() -> bool:
    """Check if flash_attn_varlen_func is available.

    This function determines whether the flash_attn_varlen_func imported at module
    level is a working implementation or a stub.

    Platform-specific sources:
    - CUDA: vllm.vllm_flash_attn.flash_attn_varlen_func
    - XPU: xpu_ops.flash_attn_varlen_func
    - ROCm: upstream flash_attn.flash_attn_varlen_func (if available)

    Note: This is separate from the AITER flash attention backend (rocm_aiter_fa.py)
    which uses rocm_aiter_ops.flash_attn_varlen_func. The condition to use AITER is
    handled separately via _aiter_ops.is_aiter_found_and_supported().

    Returns:
        bool: True if a working flash_attn_varlen_func implementation is available.
    """
    if current_platform.is_cuda() or current_platform.is_xpu():
        # CUDA and XPU always have flash_attn_varlen_func available
        return True

    if current_platform.is_rocm():
        # Use the flag set during module import to check if
        # upstream flash-attn was successfully imported
        return _ROCM_FLASH_ATTN_AVAILABLE

    return False
```
**EN:** This function answers a boolean capability check within the module. The docstring frames it as: Check if flash_attn_varlen_func is available. Key calls include `is_rocm`, `is_cuda`, `is_xpu`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。 关键调用包括 `is_rocm`, `is_cuda`, `is_xpu`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `get_flash_attn_version`: top-level helper or orchestration entry point. / `get_flash_attn_version`：顶层辅助函数或编排入口。
- `is_fa_version_supported`: top-level helper or orchestration entry point. / `is_fa_version_supported`：顶层辅助函数或编排入口。
- `flash_attn_supports_quant_query_input`: top-level helper or orchestration entry point. / `flash_attn_supports_quant_query_input`：顶层辅助函数或编排入口。
- `flash_attn_supports_sinks`: top-level helper or orchestration entry point. / `flash_attn_supports_sinks`：顶层辅助函数或编排入口。
- `flash_attn_supports_mla`: top-level helper or orchestration entry point. / `flash_attn_supports_mla`：顶层辅助函数或编排入口。
- `is_flash_attn_varlen_func_available`: top-level helper or orchestration entry point. / `is_flash_attn_varlen_func_available`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `flash_attn`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.logger`, `vllm.platforms`, `vllm._custom_ops`, `vllm.vllm_flash_attn`, `vllm`, `vllm._xpu_ops`, `vllm.vllm_flash_attn.flash_attn_interface`, `vllm.config`
